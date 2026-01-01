# Load Balancing

### GeeksforGeeks Articles
- **Load Balancer in System Design**: [https://www.geeksforgeeks.org/load-balancer-system-design-interview-question/](https://www.geeksforgeeks.org/load-balancer-system-design-interview-question/)
- **Load Balancing Algorithms**: [https://www.geeksforgeeks.org/load-balancing-algorithms/](https://www.geeksforgeeks.org/load-balancing-algorithms/)
- **Reverse Proxy vs Load Balancer**: [https://www.geeksforgeeks.org/difference-between-load-balancer-and-reverse-proxy/](https://www.geeksforgeeks.org/difference-between-load-balancer-and-reverse-proxy/)

### Videos

#### Hindi
- **Load Balancer Complete Guide**: [https://www.youtube.com/watch?v=K0Ta65OqQkY](https://www.youtube.com/watch?v=K0Ta65OqQkY)

#### English
- **Load Balancers - Gaurav Sen**: [https://www.youtube.com/watch?v=K0Ta65OqQkY](https://www.youtube.com/watch?v=K0Ta65OqQkY)
- **NGINX Load Balancing**: [https://www.youtube.com/watch?v=v81CzSeiQjo](https://www.youtube.com/watch?v=v81CzSeiQjo)

--------
<br>
<br>

### 1. What is Load Balancing?
Load balancing is the process of distributing network traffic across multiple servers to ensure no single server is overwhelmed. It improves availability, reliability, and performance.

### 2. Types of Load Balancers

| Type | Layer | Description |
|------|-------|-------------|
| L4 (Transport) | TCP/UDP | Routes based on IP and port |
| L7 (Application) | HTTP | Routes based on content, headers, URL |

#### L4 vs L7 Load Balancers

| Feature | L4 | L7 |
|---------|----|----|
| Speed | Faster | Slightly slower |
| Intelligence | Basic | Content-aware |
| SSL Termination | Usually no | Yes |
| URL Routing | No | Yes |
| WebSocket Support | Limited | Full |

### 3. Load Balancing Algorithms

#### Static Algorithms

| Algorithm | Description | Use Case |
|-----------|-------------|----------|
| Round Robin | Distribute sequentially | Equal capacity servers |
| Weighted Round Robin | Consider server weights | Unequal capacity |
| IP Hash | Hash client IP to server | Session persistence |

#### Dynamic Algorithms

| Algorithm | Description | Use Case |
|-----------|-------------|----------|
| Least Connections | Route to server with fewest connections | Long-lived connections |
| Weighted Least Connections | Consider weights + connections | Variable capacity |
| Least Response Time | Route to fastest responding | Latency-sensitive apps |
| Resource Based | Based on server resources (CPU, RAM) | Heterogeneous servers |

### 4. Health Checks
Load balancers continuously check server health to avoid routing to failed servers.

```
Health Check Types:
1. TCP Check - Can establish connection?
2. HTTP Check - Does /health return 200?
3. Custom Script - Run specific health validation
```

| Parameter | Typical Value |
|-----------|---------------|
| Interval | 5-30 seconds |
| Timeout | 2-5 seconds |
| Unhealthy Threshold | 2-3 failures |
| Healthy Threshold | 2-3 successes |

### 5. Session Persistence (Sticky Sessions)
Ensure requests from the same client go to the same server.

#### Methods
- **Cookie-based**: Insert cookie with server ID
- **IP-based**: Hash client IP
- **URL-based**: Encode session in URL

| Pros | Cons |
|------|------|
| Session data locality | Uneven load distribution |
| Simpler app design | Server failure loses sessions |
| Cache efficiency | Scaling challenges |

### 6. Reverse Proxy
A reverse proxy sits between clients and servers, forwarding requests to backend servers.

```
Client → Reverse Proxy → Server 1
                       → Server 2
                       → Server 3
```

#### Reverse Proxy Benefits
- SSL/TLS termination
- Caching static content
- Compression
- Security (hide server IPs)
- Rate limiting

#### Load Balancer vs Reverse Proxy

| Feature | Load Balancer | Reverse Proxy |
|---------|---------------|---------------|
| Primary Purpose | Distribute load | Security, caching |
| Health Checks | Yes | Optional |
| SSL Termination | Some | Yes |
| Caching | No | Yes |

### 7. High Availability Setup
Avoid single point of failure in load balancers.

```
        Internet
           ↓
  ┌─────────────────┐
  │  DNS (Multiple) │
  └────────┬────────┘
           ↓
  ┌─────────────────┐
  │ LB 1 ←───→ LB 2 │  (Active-Passive or Active-Active)
  │   (Heartbeat)   │
  └────────┬────────┘
           ↓
    ┌──────┼──────┐
    ↓      ↓      ↓
  App1   App2   App3
```

### 8. Popular Load Balancers

| Software | Type | License |
|----------|------|---------|
| NGINX | L7 | Open Source/Commercial |
| HAProxy | L4/L7 | Open Source |
| AWS ALB | L7 | Cloud (AWS) |
| AWS NLB | L4 | Cloud (AWS) |
| Google Cloud LB | L4/L7 | Cloud (GCP) |
| Traefik | L7 | Open Source |

### 9. DNS Load Balancing
Distribute traffic at DNS level by returning different IPs.

```
example.com → 1.2.3.4 (30% of time)
           → 5.6.7.8 (30% of time)
           → 9.10.11.12 (40% of time)
```

| Pros | Cons |
|------|------|
| Simple to implement | No health checks (basic) |
| Geographic routing | DNS caching issues |
| No extra infrastructure | Limited control |

### 10. Global Server Load Balancing (GSLB)
Route users to the nearest or best-performing data center.

```
User (India) → GSLB → Asia DC
User (USA) → GSLB → US DC
User (Europe) → GSLB → EU DC
```
