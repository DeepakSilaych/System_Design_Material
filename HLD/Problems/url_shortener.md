# URL Shortener Design

## 1. Requirements

### Functional Requirements
- Given a URL, generate a shorter unique alias
- When user clicks short link, redirect to original URL
- Users can optionally set custom alias
- Links expire after configurable time (default: never)

### Non-Functional Requirements
- High availability (99.9%)
- Low latency redirection (< 100ms)
- Short URLs should not be predictable

### Estimates
- 500M new URLs per month
- Read:Write = 100:1 (redirects vs creates)
- 50B redirects per month
- ~20K reads/sec (peak: 40K)
- ~200 writes/sec (peak: 400)
- Storage: 500M × 12 months × 5 years × 500 bytes = ~15TB

## 2. Database Design

```
URL Table:
+---------------+------------------+
| Field         | Type             |
+---------------+------------------+
| id            | BIGINT (PK)      |
| short_code    | VARCHAR(7) INDEX |
| original_url  | TEXT             |
| user_id       | BIGINT (FK)      |
| created_at    | TIMESTAMP        |
| expires_at    | TIMESTAMP        |
+---------------+------------------+
```

### Database Choice
- SQL (PostgreSQL): Strong consistency, ACID
- NoSQL alternative: DynamoDB for high write throughput

## 3. Short URL Generation

### Approach 1: Base62 Encoding
```
Characters: a-z, A-Z, 0-9 (62 chars)
7 characters = 62^7 = 3.5 trillion combinations

ID: 12345 → Base62: "dnh"
```

### Approach 2: MD5 Hash + Truncate
```
MD5(original_url) → Take first 7 characters
Collision handling: Append counter or use different characters
```

### Approach 3: Pre-generated Keys (KGS)
- Generate keys offline in batches
- Mark keys as used when assigned
- Two databases: unused_keys, used_keys

## 4. High-Level Architecture

```
                        ┌─────────────────┐
                        │   Load Balancer │
                        └────────┬────────┘
                                 │
            ┌────────────────────┼────────────────────┐
            ↓                    ↓                    ↓
    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
    │   App Server  │   │   App Server  │   │   App Server  │
    └───────┬───────┘   └───────┬───────┘   └───────┬───────┘
            │                   │                   │
            └───────────────────┼───────────────────┘
                                │
            ┌───────────────────┼───────────────────┐
            ↓                   ↓                   ↓
    ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
    │     Cache     │   │     Cache     │   │     Cache     │
    │    (Redis)    │   │    (Redis)    │   │    (Redis)    │
    └───────────────┘   └───────────────┘   └───────────────┘
                                │
                        ┌───────┴───────┐
                        ↓               ↓
                ┌───────────┐   ┌───────────┐
                │  Primary  │   │  Replica  │
                │    DB     │   │    DB     │
                └───────────┘   └───────────┘
```

## 5. API Design

### Create Short URL
```
POST /api/v1/shorten
Request:
{
    "url": "https://example.com/very/long/path",
    "custom_alias": "mylink",  // optional
    "expires_in": 86400        // optional, seconds
}

Response:
{
    "short_url": "https://short.ly/abc123",
    "original_url": "https://example.com/very/long/path",
    "expires_at": "2024-01-02T00:00:00Z"
}
```

### Redirect
```
GET /{short_code}
Response: 301/302 Redirect to original URL
```

## 6. Caching Strategy
- Cache hot URLs in Redis
- Cache hit ratio expected: 80%+
- TTL: 24 hours for frequently accessed URLs
- Eviction: LRU

## 7. Key Trade-offs

| Decision | Trade-off |
|----------|-----------|
| 301 vs 302 | 301: Browser caches, less traffic. 302: Track all clicks |
| SQL vs NoSQL | SQL: Consistency. NoSQL: Scale |
| Short code length | 6: 56B combinations. 7: 3.5T combinations |
