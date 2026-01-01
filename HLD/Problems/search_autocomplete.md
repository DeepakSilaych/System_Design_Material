# Search Autocomplete Design

## 1. Requirements

### Functional Requirements
- Return top suggestions as user types
- Support prefix matching
- Personalized results
- Trending/popular queries boosted

### Non-Functional Requirements
- Ultra-low latency (< 50ms p99)
- High availability
- Scale to billions of queries/day
- Freshness (trending topics within minutes)

## 2. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        API Gateway                           │
└────────────────────────────┬────────────────────────────────┘
                             │
┌────────────────────────────↓────────────────────────────────┐
│                   Autocomplete Service                       │
└────────────────────────────┬────────────────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ↓                    ↓                    ↓
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   In-Memory   │   │  Personalized │   │   Trending    │
│     Trie      │   │     Cache     │   │    Service    │
│    (Redis)    │   │    (Redis)    │   │               │
└───────────────┘   └───────────────┘   └───────┬───────┘
                                                 │
                                        ┌───────↓───────┐
                                        │    Kafka      │
                                        │  (Query Log)  │
                                        └───────────────┘
```

## 3. Trie Data Structure

```
          root
         /    \
        f      c
       /        \
      a          a
     /            \
    c              r
   / \              \
  e   t              s
 /     \
book   ory

Queries: "facebook", "factory", "cars", "face"
```

### Optimizations

**Prefix Trie → Compressed Trie**
```
Before: f → a → c → e → b → o → o → k
After:  face → book

Storage: O(n × m) → O(n) where n = words, m = avg length
```

## 4. Ranking Suggestions

### Signals
| Signal | Weight |
|--------|--------|
| Query frequency | High |
| Recency | Medium |
| User's past queries | High (personalized) |
| Trending now | High boost |
| Click-through rate | Medium |

### Scoring Example
```
score = base_frequency 
        + recency_boost 
        + personalization_score 
        + trending_boost
```

## 5. Data Collection Pipeline

```
User Query → Query Log (Kafka) → Aggregation (Spark/Flink)
                                          ↓
                                   Frequency Updates
                                          ↓
                              Trie Update Service
                                          ↓
                                    In-Memory Trie
```

### Update Strategies
- **Real-time:** Stream processing for trending
- **Batch:** Daily/hourly for stable frequencies
- **Hybrid:** Combine both

## 6. Storage Design

### Redis Sorted Set Approach
```bash
# Key: prefix, Score: frequency, Member: full query
ZADD "autocomplete:fac" 1000 "facebook"
ZADD "autocomplete:fac" 500 "factory"
ZADD "autocomplete:fac" 200 "face detection"

# Query
ZREVRANGE "autocomplete:fac" 0 9
```

### Optimized Storage
```
Instead of storing all prefixes:
- Store only prefixes of length 1-3
- For longer prefixes: Combine shorter prefix + client-side filter
```

## 7. API Design

```
GET /api/v1/autocomplete?q=fac&limit=10

Response:
{
    "suggestions": [
        {"text": "facebook", "score": 1000},
        {"text": "factory", "score": 500},
        {"text": "face detection", "score": 200}
    ]
}
```

## 8. Personalization

```
Merge Results:
1. User's recent queries matching prefix
2. Global popular queries
3. Trending queries

user_suggestions = get_user_history(user_id, prefix)
global_suggestions = get_global(prefix)
trending = get_trending(prefix)

merged = merge_and_rank(user, global, trending)
```

## 9. Caching Strategy

| Cache Level | Content | TTL |
|-------------|---------|-----|
| CDN/Edge | Popular prefixes | 5 min |
| Application | User's recent queries | 1 hour |
| Redis | Global frequencies | 1 day |

## 10. Handling Special Cases

| Case | Solution |
|------|----------|
| Typos | Fuzzy matching, edit distance |
| Multi-word | Treat as single prefix |
| Empty cache | Fallback to database |
| Cold start | Use popular global queries |

## 11. Latency Optimization

| Technique | Latency Saved |
|-----------|---------------|
| In-memory data | 10-50ms |
| CDN caching | 20-100ms |
| Precomputed top-k | 5-10ms |
| Connection pooling | 2-5ms |
| Geographic routing | 50-150ms |
