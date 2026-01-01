# Caching

### GeeksforGeeks Articles
- **Caching in System Design**: [https://www.geeksforgeeks.org/caching-system-design-concept-for-beginners/](https://www.geeksforgeeks.org/caching-system-design-concept-for-beginners/)
- **Cache Eviction Policies**: [https://www.geeksforgeeks.org/cache-eviction-policies-system-design/](https://www.geeksforgeeks.org/cache-eviction-policies-system-design/)
- **CDN - Content Delivery Network**: [https://www.geeksforgeeks.org/what-is-a-content-delivery-network-and-how-does-it-work/](https://www.geeksforgeeks.org/what-is-a-content-delivery-network-and-how-does-it-work/)
- **Redis Tutorial**: [https://www.geeksforgeeks.org/redis-tutorial/](https://www.geeksforgeeks.org/redis-tutorial/)

### Videos

#### Hindi
- **Caching Strategies**: [https://www.youtube.com/watch?v=dGAgxozNWFE](https://www.youtube.com/watch?v=dGAgxozNWFE)
- **Redis Deep Dive**: [https://www.youtube.com/watch?v=jgpVdJB2sKQ](https://www.youtube.com/watch?v=jgpVdJB2sKQ)

#### English
- **Caching - Gaurav Sen**: [https://www.youtube.com/watch?v=U3RkDLtS7uY](https://www.youtube.com/watch?v=U3RkDLtS7uY)
- **Redis Crash Course**: [https://www.youtube.com/watch?v=jgpVdJB2sKQ](https://www.youtube.com/watch?v=jgpVdJB2sKQ)

--------
<br>
<br>

### 1. What is Caching?
Caching is the technique of storing frequently accessed data in a temporary storage layer (cache) to reduce latency and load on the primary data source.

### 2. Caching Benefits

| Benefit | Description |
|---------|-------------|
| Reduced Latency | Cache access is orders of magnitude faster |
| Reduced Database Load | Fewer queries to the primary database |
| Cost Savings | Less compute needed for repeated operations |
| Improved Throughput | Can handle more requests |

### 3. Cache Write Strategies

#### Write-Through
Write to cache and database synchronously.
```
Client → Cache → Database (both updated)
```
- Pros: Data consistency, cache always fresh
- Cons: Higher write latency

#### Write-Back (Write-Behind)
Write to cache immediately, sync to database later.
```
Client → Cache (immediate)
         ↓
    Database (async)
```
- Pros: Low write latency
- Cons: Risk of data loss if cache fails

#### Write-Around
Write directly to database, cache on read.
```
Client → Database (write)
Client ← Cache ← Database (read miss)
```
- Pros: Cache not polluted with infrequent data
- Cons: Cache miss on first read after write

### 4. Cache Read Strategies

#### Cache-Aside (Lazy Loading)
Application manages cache explicitly.
```python
def get_user(user_id):
    user = cache.get(user_id)
    if user is None:  # Cache miss
        user = db.get(user_id)
        cache.set(user_id, user)
    return user
```

#### Read-Through
Cache manages database reads.
```
Client → Cache (miss) → Database
         ↓
    (cache populated)
```

### 5. Cache Eviction Policies

| Policy | Description | Best For |
|--------|-------------|----------|
| LRU (Least Recently Used) | Evict least recently accessed | General purpose |
| LFU (Least Frequently Used) | Evict least frequently accessed | Stable access patterns |
| FIFO (First In First Out) | Evict oldest entries | Simple implementation |
| TTL (Time To Live) | Expire after fixed time | Time-sensitive data |
| Random | Evict random entries | When pattern unknown |

### 6. Cache Invalidation Strategies

| Strategy | When to Use |
|----------|-------------|
| TTL-based | Data can be slightly stale |
| Event-based | When writes are known |
| Manual invalidation | On specific actions |
| Version-based | For complex objects |

```python
# TTL-based
cache.set(key, value, ttl=300)  # 5 minutes

# Event-based
def update_user(user):
    db.update(user)
    cache.delete(f"user:{user.id}")
```

### 7. Content Delivery Network (CDN)
CDN is a geographically distributed network of servers that cache static content close to users.

#### How CDN Works
```
User (India) → CDN Edge (Mumbai) → Cache Hit → Response
                    ↓
              (Cache Miss)
                    ↓
            Origin Server (USA)
```

#### CDN Benefits
- Reduced latency for global users
- Reduced origin server load
- DDoS protection
- High availability

#### CDN Use Cases
- Static assets (images, CSS, JS)
- Video streaming
- Software downloads
- API caching (some CDNs)

### 8. Redis as Cache
Redis is an in-memory data store commonly used for caching.

#### Redis Data Structures

| Structure | Use Case |
|-----------|----------|
| Strings | Simple key-value caching |
| Hashes | Object storage |
| Lists | Message queues, recent activity |
| Sets | Unique items, tags |
| Sorted Sets | Leaderboards, rankings |
| HyperLogLog | Unique count estimation |

#### Redis Commands
```bash
# Basic operations
SET user:1 "John"
GET user:1

# With TTL
SETEX session:abc 3600 "data"

# Hash operations
HSET user:1 name "John" age 25
HGET user:1 name
```

### 9. Distributed Caching

#### Cache Cluster
```
        Load Balancer
         /   |   \
   Cache1  Cache2  Cache3
```

#### Cache Sharding
Use consistent hashing to distribute keys across cache nodes.

### 10. Caching Patterns Summary

| Pattern | Write Behavior | Read Behavior | Consistency |
|---------|----------------|---------------|-------------|
| Cache-Aside | App writes to DB | App manages cache | Eventually consistent |
| Read-Through | - | Cache reads from DB | Consistent reads |
| Write-Through | Cache writes to DB | Cache serves reads | Strong |
| Write-Behind | Cache queues writes | Cache serves reads | Eventually consistent |

### 11. Common Caching Issues

| Issue | Description | Solution |
|-------|-------------|----------|
| Cache Stampede | Many requests on cache miss | Locking, pre-warming |
| Hot Keys | Single key gets too many hits | Replication, sharding |
| Cold Start | Empty cache after restart | Pre-warming |
| Stale Data | Outdated cached data | Proper invalidation |
