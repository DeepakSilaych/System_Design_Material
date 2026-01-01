# Rate Limiter Design

## 1. Requirements

### Functional Requirements
- Limit number of requests per time window
- Support multiple rate limiting rules (per user, IP, API)
- Return appropriate error when limit exceeded
- Real-time tracking

### Non-Functional Requirements
- Low latency (< 10ms)
- Highly available
- Distributed (multiple servers)
- Memory efficient

## 2. Rate Limiting Algorithms

### Token Bucket
```
┌────────────────────────┐
│  Bucket (capacity: 10) │
│  ●●●●●●●●○○           │  ← Tokens refill over time
└──────────┬─────────────┘
           │
    Request comes in:
    - If token available: consume token, allow
    - If no token: reject (429)
```

**Parameters:**
- Bucket size (max burst)
- Refill rate (tokens/second)

**Pros:** Handles bursts, simple
**Cons:** Two parameters to tune

### Leaky Bucket
```
     Requests (variable rate)
           ↓ ↓ ↓ ↓ ↓
    ┌────────────────┐
    │    Queue       │  ← Overflow rejected
    └───────┬────────┘
            ↓ (constant rate)
         Processing
```

**Pros:** Smooth output rate
**Cons:** Recent requests may wait while old ones process

### Fixed Window Counter
```
Window 1 (00:00 - 01:00): ████████░░ 8/10
Window 2 (01:00 - 02:00): ██░░░░░░░░ 2/10
```

**Pros:** Memory efficient
**Cons:** Burst at window edges (2x limit possible)

### Sliding Window Log
```
Keep timestamp of each request
Remove timestamps outside window
Count remaining timestamps
```

**Pros:** Accurate
**Cons:** High memory usage

### Sliding Window Counter
```
Weighted count = prev_window × overlap% + curr_window
```

**Pros:** Memory efficient, smooth
**Cons:** Approximate

## 3. High-Level Architecture

```
                    ┌──────────────────┐
                    │   Load Balancer  │
                    └────────┬─────────┘
                             │
                    ┌────────↓─────────┐
                    │   Rate Limiter   │
                    │   Middleware     │
                    └────────┬─────────┘
                             │
                    ┌────────↓─────────┐
                    │   Redis Cluster  │
                    └──────────────────┘
                             │
                    ┌────────↓─────────┐
                    │   API Servers    │
                    └──────────────────┘
```

## 4. Redis Implementation

### Token Bucket with Redis
```lua
-- KEYS[1]: rate limit key
-- ARGV[1]: max tokens
-- ARGV[2]: refill rate
-- ARGV[3]: current timestamp

local tokens = redis.call('GET', KEYS[1])
local last_refill = redis.call('GET', KEYS[1] .. ':time')

-- Calculate new tokens
local elapsed = ARGV[3] - (last_refill or ARGV[3])
local new_tokens = math.min(ARGV[1], (tokens or ARGV[1]) + elapsed * ARGV[2])

if new_tokens >= 1 then
    redis.call('SET', KEYS[1], new_tokens - 1)
    redis.call('SET', KEYS[1] .. ':time', ARGV[3])
    return 1  -- Allowed
else
    return 0  -- Denied
end
```

### Fixed Window with Redis
```python
def is_allowed(user_id, limit, window_size):
    key = f"rate:{user_id}:{current_window()}"
    count = redis.incr(key)
    if count == 1:
        redis.expire(key, window_size)
    return count <= limit
```

## 5. Distributed Rate Limiting

### Challenges
- Race conditions across servers
- Synchronization overhead
- Network latency

### Solutions
- **Centralized (Redis):** Single source of truth
- **Local + Sync:** Each server tracks locally, sync periodically
- **Sticky Sessions:** Route same user to same server

## 6. Rate Limit Headers

```http
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1609459200
Retry-After: 60
```

## 7. Multi-Level Rate Limiting

| Level | Example |
|-------|---------|
| User | 1000 requests/hour per user |
| IP | 100 requests/minute per IP |
| API | 10 requests/second per endpoint |
| Global | 1M requests/minute total |

## 8. Key Trade-offs

| Aspect | Trade-off |
|--------|-----------|
| Accuracy vs Memory | Sliding log vs Fixed window |
| Strict vs Approximate | Performance impact |
| Centralized vs Distributed | Consistency vs Latency |
