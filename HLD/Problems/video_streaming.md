# Video Streaming Design (YouTube/Netflix)

## 1. Requirements

### Functional Requirements
- Upload videos
- Stream videos (adaptive bitrate)
- Search videos
- Recommendations
- Comments and likes

### Non-Functional Requirements
- High availability
- Minimal buffering (< 200ms startup)
- Global content delivery
- Support millions of concurrent viewers

### Estimates
- 1B DAU, 5 videos/day average
- 500 hours of video uploaded/minute
- Peak: 100M concurrent streams

## 2. Video Processing Pipeline

```
Upload → Store Raw → Transcode → Store Processed → Index

Transcoding:
┌─────────────────────────────────────────────────────────┐
│                    Original Video                        │
└────────────────────────┬────────────────────────────────┘
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
   ┌─────────┐      ┌─────────┐      ┌─────────┐
   │  1080p  │      │  720p   │      │  480p   │
   │ H.264   │      │ H.264   │      │ H.264   │
   └─────────┘      └─────────┘      └─────────┘
        │                │                │
        ↓                ↓                ↓
   ┌──────────────────────────────────────────────┐
   │          Chunk into segments (2-10s)         │
   └──────────────────────────────────────────────┘
```

## 3. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        CDN Edge Servers                      │
│     (Cloudflare, Akamai, CloudFront - Global PoPs)          │
└────────────────────────────┬────────────────────────────────┘
                             │
┌────────────────────────────↓────────────────────────────────┐
│                       Origin Server                          │
└────────────────────────────┬────────────────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ↓                    ↓                    ↓
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│    Video      │   │   Metadata    │   │    Search     │
│   Service     │   │   Service     │   │   Service     │
└───────┬───────┘   └───────┬───────┘   └───────┬───────┘
        │                   │                   │
        ↓                   ↓                   ↓
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│  Object Store │   │   Database    │   │ Elasticsearch │
│   (S3/GCS)    │   │  (PostgreSQL) │   │               │
└───────────────┘   └───────────────┘   └───────────────┘
```

## 4. Adaptive Bitrate Streaming

### HLS (HTTP Live Streaming)
```
Manifest File (m3u8):
#EXTM3U
#EXT-X-STREAM-INF:BANDWIDTH=1400000
video_720p.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=800000
video_480p.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=400000
video_360p.m3u8

Segment Files:
video_720p_0001.ts
video_720p_0002.ts
...
```

### How ABR Works
```
1. Client requests manifest
2. Client measures bandwidth
3. Client selects appropriate quality
4. Client requests segments
5. Adjust quality based on buffer and bandwidth
```

## 5. Database Design

### Videos Table
```
+------------------+------------------+
| Field            | Type             |
+------------------+------------------+
| video_id         | UUID (PK)        |
| user_id          | BIGINT           |
| title            | VARCHAR(200)     |
| description      | TEXT             |
| duration         | INT (seconds)    |
| views            | BIGINT           |
| status           | ENUM             |
| uploaded_at      | TIMESTAMP        |
+------------------+------------------+
```

### Video Files Table
```
+------------------+------------------+
| Field            | Type             |
+------------------+------------------+
| file_id          | UUID (PK)        |
| video_id         | UUID (FK)        |
| quality          | VARCHAR(10)      |
| codec            | VARCHAR(20)      |
| url              | TEXT             |
+------------------+------------------+
```

## 6. CDN Strategy

### Content Distribution
```
Popular videos: Push to all edge locations
Long-tail videos: Pull on demand, cache at edge

Cache Hierarchy:
Edge (PoP) → Regional → Origin
```

### Cache Hit Rate Target
- 80%+ at edge for popular content
- 95%+ including regional

## 7. View Counting

### Real-time Counts
```
Video plays → Kafka → Count Service → Redis
                                         ↓
                              Periodic flush to DB
```

### Deduplication
- Track user + video + time window
- Count unique views (not refreshes)

## 8. Recommendations

```
Input Signals:
- Watch history
- Search history
- Likes/dislikes
- Demographics

ML Models:
1. Candidate Generation: Find ~1000 relevant videos
2. Ranking: Score and order candidates
3. Re-ranking: Apply business rules (freshness, diversity)
```

## 9. Key Trade-offs

| Aspect | Trade-off |
|--------|-----------|
| Quality Levels | Storage cost vs User experience |
| Segment Length | Startup time vs Adaptive responsiveness |
| CDN Caching | Cost vs Latency |
| View Accuracy | Real-time vs Eventual consistency |
