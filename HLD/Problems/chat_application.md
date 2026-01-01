# Chat Application Design (WhatsApp/Messenger)

## 1. Requirements

### Functional Requirements
- 1:1 messaging
- Group chat (up to 500 members)
- Message delivery status (sent, delivered, read)
- Media sharing (images, videos, files)
- Online status
- Message history

### Non-Functional Requirements
- Low latency (< 100ms)
- High availability
- Message ordering
- At-least-once delivery
- Support 1B+ users

## 2. High-Level Architecture

```
┌────────────────────────────────────────────────────────────┐
│                      API Gateway                            │
│              (HTTP for API, WebSocket for Chat)            │
└───────────────────────────┬────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        ↓                   ↓                   ↓
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│    Chat       │   │   Presence    │   │    Media      │
│   Service     │   │   Service     │   │   Service     │
└───────┬───────┘   └───────┬───────┘   └───────┬───────┘
        │                   │                   │
        ↓                   ↓                   ↓
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│ Message Queue │   │    Redis      │   │   S3/CDN      │
│    (Kafka)    │   │   (Status)    │   │               │
└───────────────┘   └───────────────┘   └───────────────┘
        │
        ↓
┌───────────────────────────────────────────────────────────┐
│                    Message Storage                         │
│           (Cassandra - partitioned by chat_id)            │
└───────────────────────────────────────────────────────────┘
```

## 3. Real-Time Communication

### WebSocket Connection
```
User connects → WebSocket Gateway → Connection Manager

Connection Manager:
- Maintains user_id → server mapping in Redis
- Handles connection lifecycle
- Routes messages to correct server
```

### Message Flow
```
Sender → WebSocket → Chat Service → Message Queue
                                         ↓
                    Lookup recipient's connection server
                                         ↓
                    Route to recipient's WebSocket server
                                         ↓
                              Recipient
```

## 4. Database Design

### Messages Table (Cassandra)
```
Partition Key: chat_id
Clustering Key: message_id (TimeUUID for ordering)

+-----------------+------------------+
| Field           | Type             |
+-----------------+------------------+
| chat_id         | UUID             |
| message_id      | TIMEUUID         |
| sender_id       | BIGINT           |
| content         | TEXT             |
| media_url       | TEXT             |
| message_type    | ENUM             |
| created_at      | TIMESTAMP        |
+-----------------+------------------+
```

### Chats Table (PostgreSQL)
```
+-----------------+------------------+
| Field           | Type             |
+-----------------+------------------+
| chat_id         | UUID (PK)        |
| type            | ENUM (1:1/group) |
| created_at      | TIMESTAMP        |
+-----------------+------------------+
```

## 5. Message Delivery Status

```
Message States:
SENT → Message saved to server
DELIVERED → Pushed to recipient's device
READ → Recipient opened the chat

Implementation:
1. Sender sends message → Server stores with status=SENT
2. Server pushes to recipient → Update status=DELIVERED
3. Recipient opens chat → Client sends read receipt
4. Server updates status=READ, notifies sender
```

## 6. Group Messaging

### Small Groups (< 100)
- Fan-out on write
- Push to each member immediately

### Large Groups (100-500)
- Hybrid approach
- Push to online members
- Others poll on app open

```
Message to Group:
1. Save message to group chat storage
2. Lookup online members (from presence service)
3. Push to online members via WebSocket
4. Offline members: Save in their inbox queue
```

## 7. Presence/Online Status

```
Redis Structure:
Key: presence:<user_id>
Value: { server_id, last_seen, status }
TTL: 60 seconds

Heartbeat every 30 seconds to maintain online status
```

## 8. Media Handling

```
Upload Flow:
1. Client requests upload URL (pre-signed S3 URL)
2. Client uploads directly to S3
3. Client sends message with media_url
4. Generate thumbnails asynchronously

Delivery:
- Images served via CDN
- Videos: Multiple quality levels
```

## 9. Offline Message Delivery

```
When user comes online:
1. Fetch undelivered messages from inbox queue
2. Send via WebSocket
3. Mark as delivered
4. Clear from queue
```

## 10. Key Trade-offs

| Aspect | Trade-off |
|--------|-----------|
| Consistency | Eventual for messages, strong for groups |
| Storage | Hot (Redis) vs Cold (Cassandra) |
| Fan-out | Speed vs Cost (large groups) |
