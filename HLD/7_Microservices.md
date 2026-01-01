# Microservices

### GeeksforGeeks Articles
- **Microservices Introduction**: [https://www.geeksforgeeks.org/microservices/](https://www.geeksforgeeks.org/microservices/)
- **Monolithic vs Microservices**: [https://www.geeksforgeeks.org/monolithic-vs-microservices-architecture/](https://www.geeksforgeeks.org/monolithic-vs-microservices-architecture/)
- **API Gateway**: [https://www.geeksforgeeks.org/what-is-api-gateway/](https://www.geeksforgeeks.org/what-is-api-gateway/)
- **Service Discovery**: [https://www.geeksforgeeks.org/service-discovery-in-microservices/](https://www.geeksforgeeks.org/service-discovery-in-microservices/)

### Videos

#### Hindi
- **Microservices Architecture**: [https://www.youtube.com/watch?v=lTAcCNbJ7KE](https://www.youtube.com/watch?v=lTAcCNbJ7KE)

#### English
- **Microservices - Gaurav Sen**: [https://www.youtube.com/watch?v=j6ow-UemzBc](https://www.youtube.com/watch?v=j6ow-UemzBc)
- **Martin Fowler on Microservices**: [https://www.youtube.com/watch?v=wgdBVIX9ifA](https://www.youtube.com/watch?v=wgdBVIX9ifA)

--------
<br>
<br>

### 1. What are Microservices?
Microservices is an architectural style where an application is structured as a collection of small, autonomous services modeled around a business domain.

### 2. Monolithic vs Microservices

| Aspect | Monolithic | Microservices |
|--------|------------|---------------|
| Codebase | Single codebase | Multiple repositories |
| Deployment | Deploy entire app | Deploy individually |
| Scaling | Scale entire app | Scale per service |
| Technology | Single stack | Polyglot |
| Team Size | Large teams | Small focused teams |
| Failure Impact | Affects whole system | Isolated failures |

### 3. Microservices Characteristics

```
┌─────────────────────────────────────────────┐
│                 API Gateway                  │
└─────────────┬───────────────┬───────────────┘
              ↓               ↓               ↓
        ┌─────────┐     ┌─────────┐     ┌─────────┐
        │ User    │     │ Order   │     │ Payment │
        │ Service │     │ Service │     │ Service │
        └────┬────┘     └────┬────┘     └────┬────┘
             ↓               ↓               ↓
        ┌─────────┐     ┌─────────┐     ┌─────────┐
        │ User DB │     │Order DB │     │Payment  │
        └─────────┘     └─────────┘     │   DB    │
                                        └─────────┘
```

| Characteristic | Description |
|----------------|-------------|
| Single Responsibility | One business capability per service |
| Independent Deployment | Deploy without affecting others |
| Decentralized Data | Each service owns its data |
| Fault Isolation | Failures don't cascade |
| API-First | Communicate via well-defined APIs |

### 4. API Gateway
Single entry point for all client requests.

#### Responsibilities
- Request routing
- Authentication/Authorization
- Rate limiting
- Load balancing
- Request/Response transformation
- Caching
- Logging and monitoring

#### Popular API Gateways
- Kong
- AWS API Gateway
- NGINX
- Traefik
- Zuul (Netflix)

### 5. Service Discovery
Mechanism for services to find and communicate with each other.

#### Client-Side Discovery
```
Client → Service Registry → Get Service Location
Client → Service (direct call)
```

#### Server-Side Discovery
```
Client → Load Balancer → Service Registry
Load Balancer → Service (routed call)
```

#### Tools
- Consul
- Eureka (Netflix)
- Kubernetes DNS
- etcd
- ZooKeeper

### 6. Inter-Service Communication

| Pattern | Type | Use Case |
|---------|------|----------|
| REST/HTTP | Synchronous | Simple CRUD operations |
| gRPC | Synchronous | High performance, streaming |
| Message Queue | Asynchronous | Event-driven, decoupling |
| Event Sourcing | Asynchronous | Audit trails, event replay |

### 7. Data Management Patterns

#### Database per Service
```
Each service has its own database
Pros: Independence, right tool for job
Cons: Distributed transactions complexity
```

#### Saga Pattern
Handle distributed transactions across services.
```
Order Service → Payment Service → Inventory Service
       ↑              ↑                   ↑
       └──── Compensating transactions ────┘
```

#### CQRS (Command Query Responsibility Segregation)
```
Commands → Write Model → Write DB
Queries → Read Model → Read DB (optimized for reads)
```

### 8. Resilience Patterns

| Pattern | Description |
|---------|-------------|
| Circuit Breaker | Stop calling failing service |
| Retry | Retry failed requests with backoff |
| Timeout | Limit wait time for responses |
| Bulkhead | Isolate failures to prevent cascade |
| Fallback | Provide default response on failure |

#### Circuit Breaker States
```
Closed (Normal) → Failures exceed threshold → Open (Fail fast)
                                                    ↓
                                              Timeout
                                                    ↓
                                    Half-Open (Test with some requests)
                                        ↓                    ↓
                                    Success              Failure
                                        ↓                    ↓
                                     Closed                Open
```

### 9. Observability

#### Three Pillars

| Pillar | Tool Examples |
|--------|---------------|
| Logs | ELK Stack, Splunk, Datadog |
| Metrics | Prometheus, Grafana, CloudWatch |
| Traces | Jaeger, Zipkin, AWS X-Ray |

#### Distributed Tracing
Track requests across multiple services.
```
Request → Service A (trace-id: abc) 
       → Service B (trace-id: abc)
       → Service C (trace-id: abc)
```

### 10. Deployment Patterns

| Pattern | Description |
|---------|-------------|
| Blue-Green | Two identical environments, switch traffic |
| Canary | Gradual rollout to subset of users |
| Rolling | Update instances one at a time |
| Feature Flags | Toggle features without deployment |

### 11. When to Use Microservices

| Use When | Avoid When |
|----------|------------|
| Large teams working in parallel | Small team, simple app |
| Need independent scaling | Startup MVP |
| Different technology needs | Strong team inexperience |
| Complex business domains | Simple CRUD applications |
| High availability requirements | Limited operational capacity |

### 12. Microservices Anti-Patterns

| Anti-Pattern | Problem |
|--------------|---------|
| Distributed Monolith | All services deploy together |
| Shared Database | Services share same DB |
| Too Many Services | Over-fragmentation |
| No API Versioning | Breaking changes affect clients |
| Ignoring Network Issues | Not handling failures |
