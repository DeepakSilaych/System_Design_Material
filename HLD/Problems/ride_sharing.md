# Ride Sharing Design (Uber/Lyft)

## 1. Requirements

### Functional Requirements
- Rider requests a ride
- Match with nearby driver
- Real-time location tracking
- ETA calculation
- Payment processing
- Rating system

### Non-Functional Requirements
- Low latency matching (< 1s)
- High availability
- Real-time location updates
- Support millions of concurrent users

## 2. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        API Gateway                           │
└────────────────────────────┬────────────────────────────────┘
                             │
    ┌────────────────────────┼────────────────────────────────┐
    ↓                        ↓                        ↓       ↓
┌────────┐            ┌────────────┐            ┌────────┐    ┌────────┐
│ Rider  │            │  Matching  │            │ Driver │    │  Trip  │
│Service │            │  Service   │            │Service │    │Service │
└───┬────┘            └─────┬──────┘            └───┬────┘    └───┬────┘
    │                       │                       │             │
    │                       ↓                       │             │
    │               ┌───────────────┐               │             │
    │               │   Location    │←──────────────┘             │
    │               │   Service     │                             │
    │               └───────┬───────┘                             │
    │                       │                                     │
    ↓                       ↓                                     ↓
┌───────────────────────────────────────────────────────────────────┐
│                           Redis / QuadTree                         │
│                    (Driver Locations + Geospatial)                 │
└───────────────────────────────────────────────────────────────────┘
```

## 3. Location Tracking

### Driver Location Updates
```
Driver App → WebSocket → Location Service → Redis
                                              ↓
                                    Key: driver:<id>:location
                                    Value: {lat, lng, timestamp}
                                    TTL: 30 seconds

Update Frequency: Every 3-5 seconds while online
```

### Geospatial Data Structures

**Option 1: Geohash**
```
World divided into grids
Each grid has a prefix (e.g., "9q8yy")
Nearby locations share prefixes

Query: Find drivers in adjacent geohash cells
```

**Option 2: QuadTree**
```
Recursively divide space into quadrants
Leaf nodes contain drivers
Efficient spatial queries
```

**Option 3: Redis Geospatial**
```bash
GEOADD drivers 77.5946 12.9716 driver:123
GEORADIUS drivers 77.5946 12.9716 5 km
```

## 4. Matching Algorithm

```
1. Rider requests ride with pickup/dropoff
2. Find available drivers within radius (expand if needed)
3. Calculate ETA for each driver
4. Consider:
   - Distance/ETA
   - Driver rating
   - Driver preferences
   - Ride type (Pool, Premium)
5. Send request to best match
6. If declined/timeout, try next driver
```

### Matching Flow
```
Request → Find Nearby Drivers → Rank → Offer to Top → Accept/Decline
                                                           ↓
                                              Timeout → Next Driver
```

## 5. Database Design

### Drivers Table
```
+------------------+------------------+
| Field            | Type             |
+------------------+------------------+
| driver_id        | UUID (PK)        |
| name             | VARCHAR          |
| phone            | VARCHAR          |
| vehicle_type     | ENUM             |
| rating           | DECIMAL          |
| status           | ENUM             |
+------------------+------------------+
```

### Trips Table
```
+------------------+------------------+
| Field            | Type             |
+------------------+------------------+
| trip_id          | UUID (PK)        |
| rider_id         | UUID             |
| driver_id        | UUID             |
| pickup_location  | POINT            |
| dropoff_location | POINT            |
| status           | ENUM             |
| fare             | DECIMAL          |
| started_at       | TIMESTAMP        |
| completed_at     | TIMESTAMP        |
+------------------+------------------+
```

## 6. ETA Calculation

```
Factors:
- Distance (road network, not straight line)
- Historical traffic data
- Real-time traffic conditions
- Time of day
- Weather

Services:
- Google Maps API
- OSRM (Open Source)
- Internal ML models
```

## 7. Surge Pricing

```
Supply/Demand Ratio:
- Calculate per zone
- Low supply or high demand → Surge multiplier

Example:
Available drivers: 10
Waiting riders: 50
Ratio: 0.2 → 2.5x surge

Displayed to rider before confirming
```

## 8. Real-Time Communication

```
WebSocket Connections:
- Driver location updates
- Trip status changes
- New ride requests (to drivers)
- Match notifications

Push Notifications:
- Ride request
- Driver arriving
- Trip completed
```

## 9. Scaling Considerations

| Component | Scaling Strategy |
|-----------|------------------|
| Location Service | Partition by city/region |
| Matching | Per-city instances |
| Trip Data | Shard by city or time |
| Historical Data | Cold storage (S3) |

## 10. Fault Tolerance

| Failure | Handling |
|---------|----------|
| Driver offline | Remove from available pool |
| Matching service down | Queue requests, retry |
| Location update lost | Use last known location |
| Trip in progress | Persist state, recover |
