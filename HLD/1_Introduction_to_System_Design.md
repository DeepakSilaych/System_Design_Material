# Introduction to System Design

### GeeksforGeeks Articles
- **System Design Tutorial**: [https://www.geeksforgeeks.org/system-design-tutorial/](https://www.geeksforgeeks.org/system-design-tutorial/)
- **CAP Theorem**: [https://www.geeksforgeeks.org/the-cap-theorem-in-dbms/](https://www.geeksforgeeks.org/the-cap-theorem-in-dbms/)
- **Latency vs Throughput**: [https://www.geeksforgeeks.org/difference-between-latency-and-throughput/](https://www.geeksforgeeks.org/difference-between-latency-and-throughput/)
- **Availability vs Reliability**: [https://www.geeksforgeeks.org/difference-between-system-reliability-and-availability/](https://www.geeksforgeeks.org/difference-between-system-reliability-and-availability/)

### Videos

#### Hindi
- **System Design Roadmap**: [https://www.youtube.com/watch?v=SqcXvc3ZmRU](https://www.youtube.com/watch?v=SqcXvc3ZmRU)
- **Complete System Design Course**: [https://www.youtube.com/watch?v=F2FmTdLtb_4](https://www.youtube.com/watch?v=F2FmTdLtb_4)

#### English
- **System Design Primer**: [https://www.youtube.com/watch?v=FSR1s2b-l_I](https://www.youtube.com/watch?v=FSR1s2b-l_I)
- **Gaurav Sen - System Design Introduction**: [https://www.youtube.com/watch?v=xpDnVSmNFX0](https://www.youtube.com/watch?v=xpDnVSmNFX0)

--------
<br>
<br>

### 1. What is System Design?
System Design is the process of defining the architecture, components, modules, interfaces, and data flow of a system to satisfy specified requirements. It focuses on the high-level structure and how different components interact with each other.

### 2. Why is System Design Important?
System Design is crucial for:
- Building scalable applications that can handle millions of users
- Ensuring high availability and reliability
- Making architectural decisions early that affect the entire system lifecycle
- Understanding trade-offs between different design choices

### 3. Key Concepts in System Design

#### Latency
Latency is the time taken for a request to travel from the source to the destination and back. It is usually measured in milliseconds (ms).

| Type | Typical Values |
|------|----------------|
| L1 Cache Reference | 0.5 ns |
| L2 Cache Reference | 7 ns |
| Main Memory Reference | 100 ns |
| SSD Random Read | 150 μs |
| HDD Seek | 10 ms |
| Network Round Trip (Same Datacenter) | 0.5 ms |
| Network Round Trip (Cross-continent) | 150 ms |

#### Throughput
Throughput is the number of operations or amount of data that can be processed in a given unit of time. It is often measured in:
- Requests per second (RPS)
- Queries per second (QPS)
- Bits per second (bps) for network throughput

#### Availability
Availability refers to the percentage of time a system is operational and accessible. It is commonly expressed in "nines":

| Availability | Downtime per Year | Downtime per Month |
|--------------|-------------------|-------------------|
| 99% (two nines) | 3.65 days | 7.2 hours |
| 99.9% (three nines) | 8.76 hours | 43.8 minutes |
| 99.99% (four nines) | 52.56 minutes | 4.32 minutes |
| 99.999% (five nines) | 5.26 minutes | 25.9 seconds |

#### Reliability
Reliability is the probability that a system will perform its required functions under stated conditions for a specified period of time. A system can be reliable but not highly available, and vice versa.

### 4. CAP Theorem
The CAP theorem states that a distributed data store can only provide two out of the following three guarantees:

- **Consistency (C)**: Every read receives the most recent write or an error
- **Availability (A)**: Every request receives a response (without guarantee that it contains the most recent version)
- **Partition Tolerance (P)**: The system continues to operate despite network partitions

In a distributed system, partition tolerance is a necessity since network failures are inevitable. Therefore, the real trade-off is between Consistency and Availability (CP vs AP systems).

| System Type | Characteristics | Examples |
|-------------|-----------------|----------|
| CP | Strong consistency, may sacrifice availability during partitions | HBase, MongoDB (configurable), Redis Cluster |
| AP | High availability, may return stale data during partitions | Cassandra, DynamoDB, CouchDB |

### 5. Functional vs Non-Functional Requirements
When designing a system, we need to understand both types of requirements:

**Functional Requirements**
- What the system should do
- Features and capabilities
- Example: "Users should be able to post tweets"

**Non-Functional Requirements**
- How the system should perform
- Quality attributes
- Examples: Scalability, Availability, Latency, Security

### 6. Common System Design Components
A typical large-scale system includes:
- **Load Balancers**: Distribute traffic across multiple servers
- **Application Servers**: Handle business logic
- **Databases**: Store and retrieve data
- **Caches**: Store frequently accessed data for fast retrieval
- **Message Queues**: Enable asynchronous communication
- **CDN**: Deliver static content globally

### 7. System Design Interview Framework
When approaching a system design problem:

1. **Requirements Clarification** (5 min)
   - Functional requirements
   - Non-functional requirements
   - Scale estimates

2. **High-Level Design** (10-15 min)
   - Draw major components
   - Define data flow

3. **Deep Dive** (15-20 min)
   - Database schema
   - API design
   - Component details

4. **Bottlenecks and Trade-offs** (5-10 min)
   - Identify potential issues
   - Discuss solutions

### 8. Back-of-the-Envelope Calculations
Important numbers to remember:

| Metric | Value |
|--------|-------|
| QPS handled by MySQL | ~1,000 |
| QPS handled by Redis | ~100,000 |
| Daily seconds | 86,400 (~100K) |
| Monthly active users per 1M users | ~200K (20%) |
| 1 KB text | ~1,000 characters |
| 1 MB image | ~1,000 KB |
| 1 GB storage | ~1,000 MB |
