# System Design Problems

This document serves as an index to common HLD interview problems. Each problem includes a comprehensive design walkthrough.

### GeeksforGeeks Articles
- **System Design Problems Collection**: [https://www.geeksforgeeks.org/most-commonly-asked-system-design-interview-problems-questions/](https://www.geeksforgeeks.org/most-commonly-asked-system-design-interview-problems-questions/)

### Videos

#### Hindi
- **System Design Interview Series**: [https://www.youtube.com/playlist?list=PLMCXHnjXnTnvo6alSjVkgxV-VH6EPyvoX](https://www.youtube.com/playlist?list=PLMCXHnjXnTnvo6alSjVkgxV-VH6EPyvoX)

#### English
- **System Design Interview Playlist**: [https://www.youtube.com/playlist?list=PLMCXHnjXnTnvo6alSjVkgxV-VH6EPyvoX](https://www.youtube.com/playlist?list=PLMCXHnjXnTnvo6alSjVkgxV-VH6EPyvoX)

--------
<br>
<br>

## Problem Index

### Entry Level
| Problem | Difficulty | Key Concepts |
|---------|------------|--------------|
| [URL Shortener](Problems/url_shortener.md) | Easy | Hashing, Database, Base62 |
| [Pastebin](Problems/pastebin.md) | Easy | Similar to URL Shortener |
| [Rate Limiter](Problems/rate_limiter.md) | Easy | Token Bucket, Redis |

### Medium Level
| Problem | Difficulty | Key Concepts |
|---------|------------|--------------|
| [Twitter/X](Problems/twitter.md) | Medium | Fan-out, Timeline, Caching |
| [Instagram](Problems/instagram.md) | Medium | CDN, Image Storage, Feed |
| [Notification System](Problems/notification_system.md) | Medium | Pub/Sub, Push, WebSockets |
| [Chat Application](Problems/chat_application.md) | Medium | WebSockets, Message Queue |
| [Search Autocomplete](Problems/search_autocomplete.md) | Medium | Trie, Caching |

### Advanced Level
| Problem | Difficulty | Key Concepts |
|---------|------------|--------------|
| [YouTube/Netflix](Problems/video_streaming.md) | Hard | CDN, Video Encoding, HLS |
| [Uber/Lyft](Problems/ride_sharing.md) | Hard | Geospatial, Real-time, Matching |
| [WhatsApp/Messenger](Problems/messaging_system.md) | Hard | E2E Encryption, Delivery Status |
| [Google Docs](Problems/collaborative_editing.md) | Hard | CRDT, Operational Transform |
| [Distributed Cache](Problems/distributed_cache.md) | Hard | Consistent Hashing, Eviction |

--------

## Problem-Solving Framework

### 1. Requirements Clarification (5 min)
- Functional: What should the system do?
- Non-Functional: Scale, latency, availability
- Out of Scope: What we won't cover

### 2. Estimation (5 min)
- Users: DAU, MAU
- Traffic: QPS, read/write ratio
- Storage: Data size, growth rate
- Bandwidth: Upload/download needs

### 3. High-Level Design (10 min)
- Draw major components
- Show data flow
- Identify key decisions

### 4. Deep Dive (15 min)
- Database schema
- API design
- Critical algorithms
- Caching strategy

### 5. Bottlenecks & Trade-offs (5 min)
- Identify potential issues
- Scaling strategies
- Single points of failure
