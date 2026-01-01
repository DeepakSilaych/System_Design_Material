# Scalability

### GeeksforGeeks Articles
- **Scalability in System Design**: [https://www.geeksforgeeks.org/what-is-scalability-and-how-to-achieve-it-learn-system-design/](https://www.geeksforgeeks.org/what-is-scalability-and-how-to-achieve-it-learn-system-design/)
- **Horizontal vs Vertical Scaling**: [https://www.geeksforgeeks.org/horizontal-and-vertical-scaling-in-databases/](https://www.geeksforgeeks.org/horizontal-and-vertical-scaling-in-databases/)
- **Consistent Hashing**: [https://www.geeksforgeeks.org/consistent-hashing/](https://www.geeksforgeeks.org/consistent-hashing/)
- **Database Sharding**: [https://www.geeksforgeeks.org/database-sharding-a-system-design-concept/](https://www.geeksforgeeks.org/database-sharding-a-system-design-concept/)

### Videos

#### Hindi
- **Horizontal vs Vertical Scaling**: [https://www.youtube.com/watch?v=xUumgxZ04SM](https://www.youtube.com/watch?v=xUumgxZ04SM)
- **Consistent Hashing Explained**: [https://www.youtube.com/watch?v=oKAU6LaYFhw](https://www.youtube.com/watch?v=oKAU6LaYFhw)

#### English
- **Scalability Lecture (Harvard)**: [https://www.youtube.com/watch?v=-W9F__D3oY4](https://www.youtube.com/watch?v=-W9F__D3oY4)
- **Consistent Hashing - Gaurav Sen**: [https://www.youtube.com/watch?v=zaRkONvyGr8](https://www.youtube.com/watch?v=zaRkONvyGr8)

--------
<br>
<br>

### 1. What is Scalability?
Scalability is the ability of a system to handle increased load by adding resources. A scalable system can grow to accommodate more users, more data, and more traffic without degrading performance.

### 2. Types of Scaling

#### Vertical Scaling (Scale Up)
Adding more power (CPU, RAM, Storage) to an existing server.

| Pros | Cons |
|------|------|
| Simple to implement | Hardware limits exist |
| No code changes required | Single point of failure |
| Lower complexity | Expensive at high end |
| Data consistency is easier | Downtime during upgrades |

#### Horizontal Scaling (Scale Out)
Adding more servers to distribute the load.

| Pros | Cons |
|------|------|
| No hardware limits | More complex architecture |
| Better fault tolerance | Requires load balancing |
| Cost-effective at scale | Data consistency challenges |
| Can scale infinitely | Code may need changes |

### 3. Database Sharding
Sharding is a technique to horizontally partition data across multiple databases. Each shard contains a subset of the total data.

#### Sharding Strategies

**Key-Based (Hash) Sharding**
```
shard_id = hash(key) % number_of_shards
```
- Pros: Even data distribution
- Cons: Resharding is complex when adding nodes

**Range-Based Sharding**
```
if key < 1000: shard_1
elif key < 2000: shard_2
else: shard_3
```
- Pros: Simple to implement, range queries easy
- Cons: Can lead to hotspots

**Directory-Based Sharding**
Uses a lookup table to determine shard location.
- Pros: Flexible, easy to add shards
- Cons: Lookup service becomes bottleneck

### 4. Consistent Hashing
Consistent hashing is a technique that minimizes data redistribution when nodes are added or removed.

#### How It Works
1. Arrange all possible hash values in a circular ring (hash ring)
2. Hash each server to a position on the ring
3. Hash each key to a position on the ring
4. Walk clockwise to find the first server that handles the key

#### Virtual Nodes
To ensure even distribution, each physical server is assigned multiple virtual nodes on the ring.

```
Server A -> VN1, VN2, VN3, VN4
Server B -> VN5, VN6, VN7, VN8
Server C -> VN9, VN10, VN11, VN12
```

| Without Virtual Nodes | With Virtual Nodes |
|----------------------|-------------------|
| Uneven distribution | More uniform distribution |
| Adding server affects few keys | Better load balancing |
| Simple implementation | Slightly more complex |

### 5. Replication
Replication involves creating copies of data across multiple servers for:
- **Redundancy**: Backup in case of failure
- **Performance**: Distribute read load
- **Availability**: Serve requests if primary fails

#### Master-Slave Replication
- One master handles writes
- Multiple slaves handle reads
- Slaves sync from master

#### Master-Master Replication
- Multiple masters can handle writes
- More complex conflict resolution
- Higher availability

### 6. When to Scale

| Signal | Action |
|--------|--------|
| CPU usage > 70% consistently | Scale up or add servers |
| Memory usage > 80% | Add RAM or distributed caching |
| Response time increasing | Optimize or add capacity |
| Database connections maxed | Add read replicas or shard |
| Disk I/O bottleneck | Use SSDs or distributed storage |

### 7. Scaling Patterns

**Stateless Services**
- Keep servers stateless (no session data)
- Store session in distributed cache (Redis)
- Easy to add/remove servers

**Database Scaling Strategy**
1. Start with read replicas
2. Add caching layer
3. Implement connection pooling
4. Shard when necessary

**Caching at Every Layer**
- CDN for static assets
- Application cache (Redis)
- Database query cache
- Object caching

### 8. Common Bottlenecks and Solutions

| Bottleneck | Solution |
|------------|----------|
| Single database | Read replicas, sharding |
| Stateful servers | Move to stateless + Redis |
| Large uploads | Direct to S3/CDN |
| Synchronous operations | Use message queues |
| Single point of failure | Redundancy, failover |
