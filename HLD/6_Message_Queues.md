# Message Queues

### GeeksforGeeks Articles
- **Message Queue in System Design**: [https://www.geeksforgeeks.org/message-queues-system-design/](https://www.geeksforgeeks.org/message-queues-system-design/)
- **Apache Kafka Tutorial**: [https://www.geeksforgeeks.org/apache-kafka/](https://www.geeksforgeeks.org/apache-kafka/)
- **RabbitMQ Tutorial**: [https://www.geeksforgeeks.org/rabbitmq-tutorial/](https://www.geeksforgeeks.org/rabbitmq-tutorial/)

### Videos

#### Hindi
- **Message Queues Explained**: [https://www.youtube.com/watch?v=oUJbuFMyBDk](https://www.youtube.com/watch?v=oUJbuFMyBDk)
- **Kafka Complete Guide**: [https://www.youtube.com/watch?v=ZJJHm_bd9Zo](https://www.youtube.com/watch?v=ZJJHm_bd9Zo)

#### English
- **Message Queues - Gaurav Sen**: [https://www.youtube.com/watch?v=oUJbuFMyBDk](https://www.youtube.com/watch?v=oUJbuFMyBDk)
- **Kafka vs RabbitMQ**: [https://www.youtube.com/watch?v=GMmRtSFQ5Z0](https://www.youtube.com/watch?v=GMmRtSFQ5Z0)

--------
<br>
<br>

### 1. What is a Message Queue?
A message queue is a form of asynchronous service-to-service communication. Messages are stored in a queue until the receiving service is ready to process them.

### 2. Why Use Message Queues?

| Benefit | Description |
|---------|-------------|
| Decoupling | Services don't need to know about each other |
| Async Processing | Non-blocking operations |
| Load Leveling | Handle traffic spikes gracefully |
| Reliability | Messages persist if consumer is down |
| Scalability | Add more consumers to handle load |

### 3. Message Queue Components

```
Producer → Queue → Consumer

Producer: Sends messages
Queue: Stores messages
Consumer: Receives and processes messages
Broker: Manages queues and message routing
```

### 4. Messaging Patterns

#### Point-to-Point (Queue)
```
Producer → [Queue] → Consumer

- One consumer per message
- Message deleted after consumption
- Use: Task distribution, work queues
```

#### Publish-Subscribe (Pub/Sub)
```
Publisher → [Topic] → Subscriber 1
                   → Subscriber 2
                   → Subscriber 3

- All subscribers receive the message
- Fan-out pattern
- Use: Event notifications, broadcasting
```

### 5. Delivery Guarantees

| Guarantee | Description | Use Case |
|-----------|-------------|----------|
| At-most-once | Message may be lost, never duplicated | Logging, metrics |
| At-least-once | Message delivered, may have duplicates | Most applications |
| Exactly-once | Delivered exactly once | Financial transactions |

### 6. Apache Kafka

Kafka is a distributed event streaming platform.

#### Key Concepts
- **Topic**: Category for messages
- **Partition**: Ordered, immutable sequence within topic
- **Offset**: Position of message in partition
- **Consumer Group**: Set of consumers sharing work
- **Broker**: Server that stores and serves data

```
         Topic: orders
    ┌─────────────────┐
    │ Partition 0     │ → Consumer 1
    │ Partition 1     │ → Consumer 2
    │ Partition 2     │ → Consumer 3
    └─────────────────┘
    (Consumer Group: order-processors)
```

#### Kafka Use Cases
- Real-time analytics
- Log aggregation
- Event sourcing
- Stream processing
- Activity tracking

### 7. RabbitMQ

RabbitMQ is a traditional message broker implementing AMQP.

#### Key Concepts
- **Exchange**: Routes messages to queues
- **Binding**: Links exchange to queue
- **Routing Key**: Used for routing decisions
- **Queue**: Stores messages

#### Exchange Types

| Type | Routing Logic |
|------|---------------|
| Direct | Exact routing key match |
| Fanout | Broadcast to all bound queues |
| Topic | Pattern matching on routing key |
| Headers | Match on message headers |

### 8. Kafka vs RabbitMQ

| Feature | Kafka | RabbitMQ |
|---------|-------|----------|
| Model | Log-based | Queue-based |
| Throughput | Very high | High |
| Message Order | Per partition | Per queue |
| Replay | Yes | No (default) |
| Push/Pull | Pull | Push |
| Complex Routing | Limited | Rich |
| Used For | Event streaming | Task queues |

### 9. Message Queue Patterns

#### Work Queue
Distribute tasks among multiple workers.
```
Producer → [Queue] → Worker 1
                  → Worker 2
                  → Worker 3
```

#### Request-Reply
Synchronous request-response over async messaging.
```
Client → [Request Queue] → Service
Client ← [Reply Queue] ← Service
```

#### Dead Letter Queue (DLQ)
Store failed messages for later analysis.
```
Consumer → Process → Success
              ↓
           Failure
              ↓
         [Dead Letter Queue]
```

### 10. Best Practices

| Practice | Description |
|----------|-------------|
| Idempotent Consumers | Handle duplicate messages gracefully |
| Message TTL | Set expiration for stale messages |
| Error Handling | Implement retry with backoff |
| Monitoring | Track queue depth, consumer lag |
| Batching | Group messages for efficiency |

### 11. AWS Services

| Service | Type | Best For |
|---------|------|----------|
| SQS | Queue | Simple queuing |
| SNS | Pub/Sub | Notifications, fan-out |
| Kinesis | Stream | Real-time data streaming |
| EventBridge | Event Bus | Event-driven architectures |

### 12. When to Use Message Queues

| Scenario | Use Queue |
|----------|-----------|
| Email notifications | ✓ |
| Image/video processing | ✓ |
| Order processing | ✓ |
| Log aggregation | ✓ |
| Real-time responses | ✗ (use sync) |
| Simple CRUD | ✗ |
