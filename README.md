# System Design Interview Preparation Guide

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](http://makeapullrequest.com)

A comprehensive collection of **System Design** resources for software engineering interviews. This repository covers **Low-Level Design (LLD)** with object-oriented programming patterns and **High-Level Design (HLD)** with distributed systems architecture — everything you need to crack system design interviews at top tech companies like Google, Amazon, Microsoft, Meta, and startups.

---

## Who Is This For?

- Software Engineers preparing for **technical interviews**
- Students learning **software architecture fundamentals**
- Anyone wanting to understand **how large-scale systems work**

---

## Table of Contents

- [Low-Level Design (LLD)](#low-level-design-lld)
  - [Core Concepts](#core-concepts)
  - [LLD Practice Problems](#lld-practice-problems)
- [High-Level Design (HLD)](#high-level-design-hld)
  - [Distributed Systems Concepts](#distributed-systems-concepts)
  - [HLD Practice Problems](#hld-practice-problems)
- [Study Roadmap](#study-roadmap)
- [Resources Included](#resources-included)
- [Contributing](#contributing)

---

## Low-Level Design (LLD)

Low-Level Design focuses on **object-oriented design**, **design patterns**, and **SOLID principles**. These concepts are essential for designing maintainable, extensible, and testable code.

### Core Concepts

| Topic | Description | Link |
|-------|-------------|------|
| **Object-Oriented Programming** | Encapsulation, Inheritance, Polymorphism, Abstraction | [Read →](LLD/1_OOP.md) |
| **UML Diagrams** | Class diagrams, Sequence diagrams, Use case diagrams | [Read →](LLD/2_UML_Diagram.md) |
| **SOLID Principles** | Single Responsibility, Open-Closed, Liskov Substitution, Interface Segregation, Dependency Inversion | [Read →](LLD/3_SOLID_Principles.md) |
| **Creational Design Patterns** | Singleton, Factory, Abstract Factory, Builder, Prototype | [Read →](LLD/4_Creational_Design_Patterns.md) |
| **Structural Design Patterns** | Adapter, Bridge, Composite, Decorator, Facade, Proxy | [Read →](LLD/5_Structural_Design_Patterns.md) |

### LLD Practice Problems

**[Complete Problem Index](LLD/Problems.md)** — 32+ object-oriented design problems

<details>
<summary><b>View All LLD Problems</b></summary>

| Category | Problems |
|----------|----------|
| **Booking Systems** | Movie Ticket Booking (BookMyShow), Concert Ticket Booking, Hotel Management, Airline Management |
| **Gaming** | Chess, Tic Tac Toe, Snake and Ladder |
| **Vehicle Systems** | Parking Lot, Car Rental, Ride Sharing (Uber), Traffic Signal |
| **Infrastructure** | Elevator System, ATM, Vending Machine, Coffee Vending Machine |
| **Social Platforms** | LinkedIn, Facebook, Stack Overflow |
| **Data Structures** | LRU Cache, Pub-Sub System, Logging Framework |
| **E-commerce & Finance** | Online Auction, Stock Brokerage, Digital Wallet, Splitwise |
| **Content & Education** | Library Management, Course Registration, Task Management |
| **Food & Delivery** | Restaurant Management, Food Delivery (Swiggy) |
| **Entertainment** | Music Streaming (Spotify), CricInfo |

</details>

---

## High-Level Design (HLD)

High-Level Design covers **distributed systems architecture**, **scalability patterns**, and **system components**. These are the concepts asked in system design interviews for designing large-scale applications.

### Distributed Systems Concepts

| Topic | Key Concepts | Link |
|-------|--------------|------|
| **Introduction to System Design** | CAP Theorem, Latency vs Throughput, Availability, Back-of-envelope Calculations | [Read →](HLD/1_Introduction_to_System_Design.md) |
| **Scalability** | Horizontal vs Vertical Scaling, Database Sharding, Consistent Hashing, Replication | [Read →](HLD/2_Scalability.md) |
| **Database Design** | SQL vs NoSQL, ACID Properties, Indexing, Partitioning, Replication | [Read →](HLD/3_Databases.md) |
| **Caching Strategies** | Write-through, Write-back, Cache Eviction (LRU, LFU), CDN, Redis | [Read →](HLD/4_Caching.md) |
| **Load Balancing** | Round Robin, Least Connections, Health Checks, Reverse Proxy, L4 vs L7 | [Read →](HLD/5_Load_Balancing.md) |
| **Message Queues** | Apache Kafka, RabbitMQ, Pub/Sub Patterns, Async Processing | [Read →](HLD/6_Message_Queues.md) |
| **Microservices Architecture** | API Gateway, Service Discovery, Circuit Breaker, SAGA Pattern | [Read →](HLD/7_Microservices.md) |
| **API Design** | REST vs GraphQL, Rate Limiting, OAuth 2.0, JWT Authentication | [Read →](HLD/8_API_Design.md) |

### HLD Practice Problems

**[Complete Problem Index](HLD/9_System_Design_Problems.md)** — Common system design interview questions

| Problem | Difficulty | Key Components | Link |
|---------|------------|----------------|------|
| **Design URL Shortener** (TinyURL) | Easy | Hashing, Base62 Encoding, Database Design | [Read →](HLD/Problems/url_shortener.md) |
| **Design Rate Limiter** | Easy | Token Bucket, Sliding Window, Redis | [Read →](HLD/Problems/rate_limiter.md) |
| **Design Twitter** | Medium | Fan-out, Timeline Generation, Caching | [Read →](HLD/Problems/twitter.md) |
| **Design Notification System** | Medium | Push/Email/SMS, Priority Queues, Pub/Sub | [Read →](HLD/Problems/notification_system.md) |
| **Design Chat Application** (WhatsApp) | Medium | WebSockets, Message Queue, Delivery Status | [Read →](HLD/Problems/chat_application.md) |
| **Design Search Autocomplete** | Medium | Trie Data Structure, Ranking, Caching | [Read →](HLD/Problems/search_autocomplete.md) |
| **Design Video Streaming** (YouTube/Netflix) | Hard | HLS/DASH, Video Transcoding, CDN | [Read →](HLD/Problems/video_streaming.md) |
| **Design Ride Sharing** (Uber/Lyft) | Hard | Geospatial Indexing, Real-time Matching | [Read →](HLD/Problems/ride_sharing.md) |

---

## Study Roadmap

### Recommended Learning Path

```
Week 1-2: LLD Fundamentals
├── OOP Concepts
├── SOLID Principles
├── Design Patterns (Creational → Structural)
└── Practice 3-5 LLD Problems

Week 3-4: HLD Fundamentals
├── System Design Introduction
├── Scalability & Databases
├── Caching & Load Balancing
└── Message Queues & Microservices

Week 5-6: HLD Problem Practice
├── Start with Easy (URL Shortener, Rate Limiter)
├── Move to Medium (Twitter, Chat, Notifications)
└── Tackle Hard (Video Streaming, Ride Sharing)

Week 7-8: Mock Interviews & Review
├── Time yourself (45 min per problem)
├── Practice explaining trade-offs
└── Review weak areas
```

---

## Resources Included

Each topic and problem file includes:

| Resource Type | Description |
|---------------|-------------|
| **GeeksforGeeks Articles** | In-depth written tutorials with code examples |
| **YouTube Videos (Hindi)** | Video explanations in Hindi for regional learners |
| **YouTube Videos (English)** | Video explanations from top educators |
| **Diagrams & Tables** | Visual representations of architectures and comparisons |
| **Trade-off Analysis** | Discussion of design decisions and alternatives |

---

## Contributing

Contributions are welcome! Here's how you can help:

1. **Add new problems** — Follow the existing format
2. **Fix errors** — Submit a PR for any mistakes
3. **Improve explanations** — Add diagrams or clarify concepts
4. **Update links** — Replace broken links with working ones

---

## License

This project is licensed under the MIT License — feel free to use it for your interview preparation!

---

## Support

If this repository helped you prepare for your interviews, please consider:

- **Starring this repository**
- **Sharing with others** preparing for interviews
- **Opening issues** for questions or suggestions
