# Twitter/X Design

## 1. Requirements

### Functional Requirements
- Post tweets (280 characters)
- Follow/unfollow users
- View home timeline (tweets from followed users)
- Search tweets
- Like and retweet

### Non-Functional Requirements
- High availability
- Timeline generation < 200ms
- Handle 500M+ DAU
- Eventually consistent (timeline)

### Estimates
- 500M DAU, 200M tweets/day
- Average user: 200 followers
- Timeline: 200 tweets
- Read:Write = 1000:1

## 2. Database Design

### Users Table
```
+------------------+------------------+
| Field            | Type             |
+------------------+------------------+
| user_id          | BIGINT (PK)      |
| username         | VARCHAR(50)      |
| email            | VARCHAR(255)     |
| created_at       | TIMESTAMP        |
+------------------+------------------+
```

### Tweets Table
```
+------------------+------------------+
| Field            | Type             |
+------------------+------------------+
| tweet_id         | BIGINT (PK)      |
| user_id          | BIGINT (FK)      |
| content          | VARCHAR(280)     |
| media_urls       | JSON             |
| created_at       | TIMESTAMP        |
+------------------+------------------+
```

### Follows Table
```
+------------------+------------------+
| Field            | Type             |
+------------------+------------------+
| follower_id      | BIGINT           |
| followee_id      | BIGINT           |
| created_at       | TIMESTAMP        |
+------------------+------------------+
```

## 3. Timeline Generation

### Approach 1: Pull Model (Fan-in)
```
User requests timeline →
Query followers →
Fetch recent tweets from each →
Merge and sort →
Return timeline
```
- Pros: Simple, low storage
- Cons: Slow for users following many people

### Approach 2: Push Model (Fan-out on Write)
```
User posts tweet →
For each follower →
Add tweet_id to their timeline cache
```
- Pros: Fast reads
- Cons: Expensive for celebrities (fan-out), storage intensive

### Approach 3: Hybrid (Twitter's Actual Approach)
```
Regular users: Push (fan-out on write)
Celebrities (>1M followers): Pull (fan-out on read)
```

## 4. High-Level Architecture

```
                    ┌──────────────────┐
                    │   Load Balancer  │
                    └────────┬─────────┘
                             │
    ┌────────────────────────┼────────────────────────┐
    │                        │                        │
    ↓                        ↓                        ↓
┌─────────┐          ┌─────────────┐          ┌─────────────┐
│  Tweet  │          │  Timeline   │          │   Search    │
│ Service │          │   Service   │          │   Service   │
└────┬────┘          └──────┬──────┘          └──────┬──────┘
     │                      │                        │
     ↓                      ↓                        ↓
┌─────────┐          ┌─────────────┐          ┌─────────────┐
│ Tweet   │          │  Timeline   │          │Elasticsearch│
│   DB    │          │    Cache    │          │             │
└─────────┘          │   (Redis)   │          └─────────────┘
                     └─────────────┘
                           │
                ┌──────────┴──────────┐
                ↓                     ↓
        ┌─────────────┐       ┌─────────────┐
        │   Fan-out   │       │   User      │
        │   Service   │       │   Graph     │
        └─────────────┘       └─────────────┘
```

## 5. Timeline Cache Structure (Redis)

```
Key: timeline:<user_id>
Value: Sorted Set (tweet_id, timestamp)

ZADD timeline:123 1609459200 tweet:456
ZREVRANGE timeline:123 0 199  # Get latest 200 tweets
```

## 6. API Design

### Post Tweet
```
POST /api/v1/tweets
{
    "content": "Hello World!",
    "media_ids": ["media123"]
}
```

### Get Timeline
```
GET /api/v1/timeline?cursor=<cursor>&limit=20
```

### Follow User
```
POST /api/v1/users/{user_id}/follow
```

## 7. Media Handling
- Upload media to S3/CDN
- Generate multiple resolutions
- Store media URLs in tweet

## 8. Key Trade-offs

| Aspect | Trade-off |
|--------|-----------|
| Push vs Pull | Speed vs Storage/Compute |
| Eventual Consistency | Speed vs Strong Consistency |
| Timeline Size | Memory vs User Experience |
