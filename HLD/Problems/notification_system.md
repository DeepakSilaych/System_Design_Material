# Notification System Design

## 1. Requirements

### Functional Requirements
- Support multiple channels: Push, SMS, Email
- Priority levels (urgent, normal, low)
- User preferences for notifications
- Deduplication
- Rate limiting per user

### Non-Functional Requirements
- High availability
- Exactly-once delivery (best effort)
- Scalable to millions of notifications/minute
- Low latency for high-priority notifications

## 2. Notification Types

| Type | Use Case | Latency Requirement |
|------|----------|---------------------|
| Push | Real-time alerts | < 1 second |
| Email | Transactional, marketing | < 1 minute |
| SMS | Critical alerts, OTP | < 10 seconds |
| In-App | User activity | < 1 second |

## 3. High-Level Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        API Gateway                            │
└────────────────────────────┬─────────────────────────────────┘
                             │
┌────────────────────────────↓─────────────────────────────────┐
│                    Notification Service                       │
│  (Validation, Preference Lookup, Priority Queue)             │
└────────────────────────────┬─────────────────────────────────┘
                             │
        ┌────────────────────┼────────────────────┐
        ↓                    ↓                    ↓
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   Push Queue  │   │  Email Queue  │   │   SMS Queue   │
│   (High Pri)  │   │   (Normal)    │   │   (High Pri)  │
└───────┬───────┘   └───────┬───────┘   └───────┬───────┘
        ↓                   ↓                   ↓
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│ Push Workers  │   │ Email Workers │   │  SMS Workers  │
└───────┬───────┘   └───────┬───────┘   └───────┬───────┘
        ↓                   ↓                   ↓
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│   FCM/APNS    │   │   SendGrid    │   │    Twilio     │
└───────────────┘   └───────────────┘   └───────────────┘
```

## 4. Database Design

### Notifications Table
```
+------------------+------------------+
| Field            | Type             |
+------------------+------------------+
| notification_id  | UUID (PK)        |
| user_id          | BIGINT           |
| type             | ENUM             |
| channel          | ENUM             |
| content          | JSON             |
| status           | ENUM             |
| priority         | INT              |
| created_at       | TIMESTAMP        |
| sent_at          | TIMESTAMP        |
+------------------+------------------+
```

### User Preferences Table
```
+------------------+------------------+
| Field            | Type             |
+------------------+------------------+
| user_id          | BIGINT (PK)      |
| push_enabled     | BOOLEAN          |
| email_enabled    | BOOLEAN          |
| sms_enabled      | BOOLEAN          |
| quiet_hours      | JSON             |
+------------------+------------------+
```

## 5. Push Notification Flow

```
1. Server → Notification Service
2. Lookup user device tokens
3. Format for FCM/APNS
4. Send to push provider
5. Handle delivery receipts

Device Registration:
- Mobile app registers with FCM/APNS
- Receives device token
- Sends token to our server
- We store token with user_id
```

## 6. Deduplication Strategy

```python
def send_notification(notification):
    key = f"dedup:{notification.user_id}:{notification.type}:{hash(notification.content)}"
    
    if redis.setnx(key, 1):
        redis.expire(key, 300)  # 5 min window
        actually_send(notification)
    else:
        log("Duplicate notification suppressed")
```

## 7. Rate Limiting per User

| Channel | Limit |
|---------|-------|
| Push | 10/minute |
| Email | 5/hour |
| SMS | 3/hour |

## 8. Priority Queues

```
High Priority (P0): OTP, Security alerts
├── Dedicated workers
├── No batching
└── Immediate processing

Normal Priority (P1): Transactional
├── Shared workers
└── Small batching allowed

Low Priority (P2): Marketing
├── Background workers
├── Large batching
└── Respect quiet hours
```

## 9. Reliability Patterns

| Pattern | Implementation |
|---------|----------------|
| Retry | Exponential backoff (1s, 2s, 4s, 8s) |
| Circuit Breaker | Trip after 5 failures, retry after 30s |
| Fallback | SMS → Push if SMS fails for critical alerts |
| Dead Letter Queue | Store failed notifications for analysis |

## 10. Key Trade-offs

| Aspect | Trade-off |
|--------|-----------|
| Batching | Throughput vs Latency |
| Persistence | Reliability vs Storage cost |
| Dedup Window | Accuracy vs User preference |
