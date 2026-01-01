# API Design

### GeeksforGeeks Articles
- **REST API Introduction**: [https://www.geeksforgeeks.org/rest-api-introduction/](https://www.geeksforgeeks.org/rest-api-introduction/)
- **REST vs GraphQL**: [https://www.geeksforgeeks.org/rest-api-vs-graphql/](https://www.geeksforgeeks.org/rest-api-vs-graphql/)
- **Rate Limiting**: [https://www.geeksforgeeks.org/rate-limiting-in-system-design/](https://www.geeksforgeeks.org/rate-limiting-in-system-design/)
- **OAuth 2.0**: [https://www.geeksforgeeks.org/oauth-2-0/](https://www.geeksforgeeks.org/oauth-2-0/)
- **JWT Authentication**: [https://www.geeksforgeeks.org/json-web-token-jwt/](https://www.geeksforgeeks.org/json-web-token-jwt/)

### Videos

#### Hindi
- **REST API Design Best Practices**: [https://www.youtube.com/watch?v=7YcW25PHnAA](https://www.youtube.com/watch?v=7YcW25PHnAA)
- **OAuth & JWT Explained**: [https://www.youtube.com/watch?v=t18gmY2zSFQ](https://www.youtube.com/watch?v=t18gmY2zSFQ)

#### English
- **API Design - Gaurav Sen**: [https://www.youtube.com/watch?v=_YlYuNMTCc8](https://www.youtube.com/watch?v=_YlYuNMTCc8)
- **REST API Best Practices**: [https://www.youtube.com/watch?v=-mN3VyJuCjM](https://www.youtube.com/watch?v=-mN3VyJuCjM)

--------
<br>
<br>

### 1. REST API Principles

REST (Representational State Transfer) is an architectural style for designing networked applications.

#### REST Constraints
- **Stateless**: Each request contains all necessary information
- **Client-Server**: Separation of concerns
- **Cacheable**: Responses must define cacheability
- **Uniform Interface**: Consistent resource identification
- **Layered System**: Client can't tell if connected directly to server

### 2. HTTP Methods

| Method | Purpose | Idempotent | Safe |
|--------|---------|------------|------|
| GET | Retrieve resource | Yes | Yes |
| POST | Create resource | No | No |
| PUT | Replace resource | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove resource | Yes | No |

### 3. REST API Design Best Practices

#### Resource Naming
```
Good:
GET /users
GET /users/123
GET /users/123/orders
POST /users

Bad:
GET /getUsers
GET /user/123/getOrders
POST /createUser
```

#### HTTP Status Codes

| Code | Meaning | Use Case |
|------|---------|----------|
| 200 | OK | Successful GET/PUT |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid input |
| 401 | Unauthorized | Not authenticated |
| 403 | Forbidden | Not authorized |
| 404 | Not Found | Resource doesn't exist |
| 429 | Too Many Requests | Rate limited |
| 500 | Internal Server Error | Server error |

### 4. REST vs GraphQL

| Aspect | REST | GraphQL |
|--------|------|---------|
| Data Fetching | Multiple endpoints | Single endpoint |
| Over-fetching | Common | No |
| Under-fetching | Common (multiple requests) | No |
| Versioning | URL or header | Schema evolution |
| Caching | HTTP caching | Complex |
| Learning Curve | Lower | Higher |

#### GraphQL Example
```graphql
# Query
query {
  user(id: "123") {
    name
    email
    orders {
      id
      total
    }
  }
}

# Response
{
  "user": {
    "name": "John",
    "email": "john@example.com",
    "orders": [
      {"id": "1", "total": 100}
    ]
  }
}
```

### 5. API Versioning

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| URL Path | /api/v1/users | Clear, simple | URL pollution |
| Query Parameter | /api?version=1 | Flexible | Easy to miss |
| Header | X-API-Version: 1 | Clean URLs | Hidden |
| Accept Header | Accept: application/vnd.api.v1+json | RESTful | Complex |

### 6. Rate Limiting
Control the number of requests a client can make.

#### Algorithms

| Algorithm | Description |
|-----------|-------------|
| Token Bucket | Tokens refill at fixed rate |
| Leaky Bucket | Requests processed at fixed rate |
| Fixed Window | Count requests in time windows |
| Sliding Window | Smooth transition between windows |

#### Rate Limit Headers
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1609459200
```

### 7. Authentication Methods

#### API Keys
Simple token passed in header or query.
```
X-API-Key: abc123def456
```

#### OAuth 2.0
Industry standard for authorization.

```
Authorization Flows:
1. Authorization Code (web apps)
2. Client Credentials (server-to-server)
3. Implicit (deprecated)
4. PKCE (mobile/SPA)
```

#### JWT (JSON Web Token)
Self-contained token with claims.
```
Header.Payload.Signature

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4ifQ.
Gfx6VO9tcxwk6xqx9yYzSfebfeakZp5JYIgP_edcw_A
```

| Component | Contents |
|-----------|----------|
| Header | Algorithm, token type |
| Payload | Claims (user ID, expiry, etc.) |
| Signature | Verification signature |

### 8. Pagination

#### Offset-Based
```
GET /users?offset=20&limit=10

Pros: Simple, random access
Cons: Inconsistent with live data, slow for large offsets
```

#### Cursor-Based
```
GET /users?cursor=abc123&limit=10

Pros: Consistent, efficient
Cons: No random access, complex implementation
```

### 9. Error Handling

#### Error Response Format
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Email is required",
    "details": [
      {
        "field": "email",
        "message": "This field is required"
      }
    ]
  }
}
```

### 10. API Documentation

| Tool | Features |
|------|----------|
| Swagger/OpenAPI | Specification + UI |
| Postman | Testing + Documentation |
| Redoc | API Reference Docs |
| API Blueprint | Markdown-based |

### 11. gRPC
High-performance RPC framework using Protocol Buffers.

```protobuf
syntax = "proto3";

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (stream User);
}

message GetUserRequest {
  string id = 1;
}

message User {
  string id = 1;
  string name = 2;
  string email = 3;
}
```

| REST | gRPC |
|------|------|
| JSON | Protocol Buffers |
| HTTP/1.1 | HTTP/2 |
| Human-readable | Binary (faster) |
| One request-response | Streaming support |

### 12. API Security Best Practices

| Practice | Description |
|----------|-------------|
| HTTPS | Always use TLS |
| Input Validation | Validate all inputs |
| Rate Limiting | Prevent abuse |
| Authentication | Verify identity |
| Authorization | Check permissions |
| Logging | Audit trail |
| CORS | Configure properly |
