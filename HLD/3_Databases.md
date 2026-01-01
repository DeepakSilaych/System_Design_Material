# Databases

### GeeksforGeeks Articles
- **SQL vs NoSQL**: [https://www.geeksforgeeks.org/difference-between-sql-and-nosql/](https://www.geeksforgeeks.org/difference-between-sql-and-nosql/)
- **Database Indexing**: [https://www.geeksforgeeks.org/indexing-in-databases-set-1/](https://www.geeksforgeeks.org/indexing-in-databases-set-1/)
- **ACID Properties**: [https://www.geeksforgeeks.org/acid-properties-in-dbms/](https://www.geeksforgeeks.org/acid-properties-in-dbms/)
- **Database Replication**: [https://www.geeksforgeeks.org/data-replication-in-dbms/](https://www.geeksforgeeks.org/data-replication-in-dbms/)
- **Database Partitioning**: [https://www.geeksforgeeks.org/database-partitioning/](https://www.geeksforgeeks.org/database-partitioning/)

### Videos

#### Hindi
- **SQL vs NoSQL**: [https://www.youtube.com/watch?v=t0GlGbtMTio](https://www.youtube.com/watch?v=t0GlGbtMTio)
- **Database Indexing Deep Dive**: [https://www.youtube.com/watch?v=VIr8_CG4d4w](https://www.youtube.com/watch?v=VIr8_CG4d4w)

#### English
- **Database Fundamentals**: [https://www.youtube.com/watch?v=W2Z7fbCLSTw](https://www.youtube.com/watch?v=W2Z7fbCLSTw)
- **SQL vs NoSQL - Gaurav Sen**: [https://www.youtube.com/watch?v=xQnIN9bW0og](https://www.youtube.com/watch?v=xQnIN9bW0og)

--------
<br>
<br>

### 1. SQL vs NoSQL Databases

#### SQL (Relational) Databases
Structured data stored in tables with predefined schemas.

| Feature | Description |
|---------|-------------|
| Schema | Fixed, predefined |
| Query Language | SQL |
| ACID | Full support |
| Relationships | Strong (foreign keys) |
| Scaling | Vertical (primarily) |
| Examples | MySQL, PostgreSQL, Oracle |

#### NoSQL Databases
Flexible data models designed for specific use cases.

**Types of NoSQL Databases:**

| Type | Description | Examples | Use Cases |
|------|-------------|----------|-----------|
| Key-Value | Simple key-value pairs | Redis, DynamoDB | Caching, sessions |
| Document | JSON-like documents | MongoDB, CouchDB | Content management |
| Wide-Column | Column families | Cassandra, HBase | Time-series, analytics |
| Graph | Nodes and relationships | Neo4j, JanusGraph | Social networks, recommendations |

### 2. When to Use Each

| Use SQL When | Use NoSQL When |
|--------------|----------------|
| Complex relationships | Simple lookups |
| ACID compliance required | High write throughput |
| Data is structured | Schema flexibility needed |
| Complex queries (JOINs) | Horizontal scaling priority |
| Financial transactions | Real-time data |

### 3. ACID Properties

**Atomicity**
- All operations in a transaction complete or none do
- Example: Money transfer - debit AND credit must both succeed

**Consistency**
- Database moves from one valid state to another
- All constraints are satisfied before and after transaction

**Isolation**
- Concurrent transactions don't interfere with each other
- Each transaction appears to run in isolation

**Durability**
- Once committed, data persists even after system failure
- Typically achieved through write-ahead logging

### 4. Database Indexing

Indexes are data structures that improve the speed of data retrieval operations.

#### Types of Indexes

| Index Type | Description | Best For |
|------------|-------------|----------|
| B-Tree | Balanced tree structure | Range queries, equality |
| Hash | Hash table based | Exact matches |
| Bitmap | Bit arrays | Low cardinality columns |
| Full-Text | Text search optimized | Search applications |

#### Indexing Trade-offs
```
Pros:
- Faster reads and searches
- Efficient sorting

Cons:
- Slower writes (index maintenance)
- Additional storage space
- Index selection overhead
```

### 5. Database Replication

#### Master-Slave (Primary-Replica)
```
       Writes
         ↓
    [Master/Primary]
         ↓
   Replication (async/sync)
    /    |    \
[Slave] [Slave] [Slave]
   ↑       ↑       ↑
   ← ← ← Reads → → →
```

| Type | Description | Use Case |
|------|-------------|----------|
| Synchronous | Wait for all replicas | Strong consistency |
| Asynchronous | Don't wait | Higher availability |
| Semi-synchronous | Wait for at least one | Balance |

#### Replication Lag
Time between write on master and visibility on replica.

### 6. Database Partitioning

#### Horizontal Partitioning (Sharding)
Split rows across multiple databases.
```
User 1-1000 → Shard 1
User 1001-2000 → Shard 2
```

#### Vertical Partitioning
Split columns across multiple tables.
```
Users table → user_id, name, email
User_Details → user_id, address, preferences
```

### 7. Connection Pooling
Maintain a pool of reusable database connections.

| Without Pool | With Pool |
|--------------|-----------|
| Create new connection per request | Reuse existing connections |
| High overhead | Lower overhead |
| Risk of exhausting connections | Controlled connection count |
| Slower | Faster |

### 8. N+1 Query Problem
Common performance issue in ORMs.

**Problem:**
```python
# 1 query for users
users = User.all()  
# N queries for posts (one per user)
for user in users:
    print(user.posts)  
```

**Solution - Eager Loading:**
```python
# 2 queries total
users = User.all().includes('posts')
```

### 9. Database Selection Guide

| Requirement | Recommended |
|-------------|-------------|
| Complex transactions | PostgreSQL |
| Simple key-value | Redis, DynamoDB |
| Document storage | MongoDB |
| Time-series data | TimescaleDB, InfluxDB |
| Graph relationships | Neo4j |
| Wide-column/Analytics | Cassandra, HBase |
| Full-text search | Elasticsearch |
