# System Design Notes — Infosys L2/L3

---

## 1. Basics

**System Design** = the process of defining architecture, components, APIs, and data flow to build a system that meets given requirements (functional + non-functional).

| Term | Meaning |
|---|---|
| **Functional Requirements** | What the system *should do* — e.g., "user can shorten a URL", "user can send a message" |
| **Non-functional Requirements** | *How well* the system does it — e.g., speed, uptime, scale |
| **Scalability** | Ability of a system to handle increasing load (users/data/traffic) without breaking |
| **Availability** | System stays up and responsive, minimal downtime (measured as %, e.g., 99.9%) |
| **Reliability** | System performs correctly and consistently over time, without failures |
| **Latency** | Time taken to process a single request (lower = faster) |
| **Throughput** | Number of requests a system can handle per unit time |

*Quick distinction to remember:* **Latency = speed of one request. Throughput = volume of requests handled.**

---

## 2. Core Concepts

| Concept | Idea |
|---|---|
| **Client-Server Architecture** | Client sends requests, server processes and responds. Basis of almost all web systems |
| **Vertical Scaling** | Add more power (CPU/RAM) to a single machine. Simple, but has a hardware limit |
| **Horizontal Scaling** | Add more machines to share the load. Harder to manage, but scales further |
| **Load Balancer** | Distributes incoming traffic across multiple servers to avoid overload on one (e.g., Round Robin, Least Connections) |
| **Reverse Proxy** | Sits in front of servers, forwards client requests to the right backend server; also used for SSL termination, caching, security |
| **Caching** | Storing frequently accessed data in fast storage (RAM) to avoid repeated expensive computation/DB calls |
| **CDN (Content Delivery Network)** | Servers distributed geographically that cache static content (images, videos, JS/CSS) closer to users, reducing latency |
| **Database Indexing** | Data structure (usually B-Tree) that speeds up read queries on a column, at the cost of extra storage and slower writes |
| **Replication** | Keeping copies of the same data on multiple servers — improves availability and read performance |
| **Sharding** | Splitting a large database into smaller pieces (shards) across servers, based on a key — improves write scalability |

*One-liner:* Replication = same data, many copies (for availability/reads). Sharding = different data, split across machines (for scale/writes).

---

## 3. Storage

| Concept | Idea |
|---|---|
| **SQL** | Structured, relational (tables with fixed schema), supports joins, strong consistency — e.g., MySQL, PostgreSQL |
| **NoSQL** | Flexible schema, better horizontal scaling, various models (document, key-value, column, graph) — e.g., MongoDB, Cassandra, Redis |
| **When to use** | SQL → structured data, relationships, transactions (banking). NoSQL → high scale, flexible/unstructured data (social feeds, logs) |
| **ACID** | **A**tomicity, **C**onsistency, **I**solation, **D**urability — guarantees for reliable SQL transactions |
| **CAP Theorem** | In a distributed system, you can only guarantee **2 out of 3**: Consistency, Availability, Partition Tolerance |
| **Consistency (C)** | Every read gets the latest write |
| **Availability (A)** | Every request gets a response (even if not the latest data) |
| **Partition Tolerance (P)** | System keeps working even if network communication between nodes breaks |

*Interview line:* "Since network partitions are unavoidable in distributed systems, we usually have to pick between C and A — e.g., banking systems favor CP, social media feeds favor AP."

---

## 4. Communication

| Concept | Idea |
|---|---|
| **REST API** | Standard way for client-server communication over HTTP using resources (URLs) and verbs (GET/POST/PUT/DELETE) |
| **WebSockets** | Persistent, two-way connection between client and server — used for real-time apps (chat, live scores) |
| **Polling** | Client repeatedly asks server "any new data?" at fixed intervals — simple but wasteful |
| **Long Polling** | Client asks server, server holds the request open until new data is available, then responds — more efficient than polling |
| **Message Queue** (RabbitMQ/Kafka) | Decouples producer and consumer — producer pushes a message to a queue, consumer processes it asynchronously. Used for async tasks, event-driven systems, buffering high traffic |

---

## 5. Performance

| Concept | Idea |
|---|---|
| **Cache** | Store hot/frequent data in memory (Redis, Memcached) to reduce DB load and latency |
| **Cache Invalidation** | Removing/updating stale cache data when the underlying data changes — common strategies: TTL (expire after time), write-through (update cache on write) |
| **Rate Limiter** | Restricts number of requests a client can make in a time window — protects system from abuse/overload (e.g., 100 requests/min per user) |
| **Pagination** | Returning data in chunks (pages) instead of all at once — e.g., `?page=2&limit=20`, or cursor-based for large datasets |

---

## 6. Security (Basic)

| Concept | Idea |
|---|---|
| **Authentication** | Verifying *who* the user is (login: username/password, OTP) |
| **Authorization** | Verifying *what* the authenticated user is allowed to do (permissions/roles) |
| **JWT (JSON Web Token)** | A signed token issued after login, sent with each request to prove identity without querying the DB every time |
| **HTTPS** | HTTP over SSL/TLS — encrypts data in transit between client and server |

*One-liner:* Authentication = "who are you", Authorization = "what can you do".

---

## 7. High-Level Design Questions

For each, just identify: **Components → Database → APIs → Cache → Load Balancer.** No deep architecture needed.

### A. URL Shortener
- **Components:** URL Shortener Service, Redirect Service
- **Database:** Key-value store (short code → long URL) — e.g., DynamoDB/Redis for speed
- **APIs:** `POST /shorten {long_url}` → returns short URL; `GET /{short_code}` → redirects to long URL
- **Cache:** Cache frequently accessed short codes to avoid DB hit on every redirect
- **Load Balancer:** Distributes redirect requests (high read traffic) across multiple servers
- **Key idea:** Generate unique short code via base62 encoding of an auto-increment ID or hashing

### B. Chat Application
- **Components:** Chat Service, User Service, Notification Service
- **Database:** NoSQL (MongoDB/Cassandra) for messages — high write volume, flexible schema
- **APIs:** `POST /message`, `GET /messages/{chatId}`
- **Cache:** Cache recent messages/active user sessions
- **Load Balancer:** Distributes WebSocket connections across chat servers
- **Key idea:** Use **WebSockets** for real-time delivery; message queue to handle offline users

### C. Notification System
- **Components:** Notification Service, Template Service, Delivery Workers (email/SMS/push)
- **Database:** SQL/NoSQL to store notification logs and user preferences
- **APIs:** `POST /notify {userId, type, message}`
- **Cache:** Cache user notification preferences
- **Load Balancer:** Distributes incoming notification requests
- **Key idea:** Use a **Message Queue** (Kafka) to decouple request from actual sending — async processing

### D. Online Library
- **Components:** Catalog Service, Member Service, Borrow/Return Service
- **Database:** SQL (relational data — books, members, transactions with strong consistency)
- **APIs:** `GET /books`, `POST /borrow`, `POST /return`
- **Cache:** Cache popular book search results
- **Load Balancer:** Distributes read-heavy catalog search traffic
- **Key idea:** Mostly CRUD-heavy, consistency matters (avoid double-issuing same book copy)

### E. Parking Lot
*(Already covered in LLD — for system design level, just add:)*
- **Database:** SQL for spot/ticket/payment data
- **APIs:** `POST /entry`, `POST /exit`, `GET /available-spots`
- **Cache:** Cache real-time spot availability count per floor

### F. Food Delivery (very high level)
- **Components:** Restaurant Service, Order Service, Delivery/Tracking Service, Payment Service
- **Database:** SQL for orders/payments (consistency), NoSQL for live location tracking
- **APIs:** `POST /order`, `GET /order/{id}/status`, `GET /delivery/{id}/location`
- **Cache:** Cache restaurant menus, nearby restaurant search results
- **Load Balancer:** Distributes across order/tracking services
- **Key idea:** Real-time location updates via WebSockets; message queue to handle order → kitchen → delivery flow

---

## Interview Approach for HLD Questions
1. Clarify scope (e.g., "Are we handling millions of users or a small scale?")
2. List core components/services
3. Pick database type (SQL vs NoSQL) with a reason
4. Define 2–3 key APIs
5. Mention where caching and load balancing fit
6. Mention one scaling consideration (sharding/replication/queue) if relevant

Keep it to a 3–5 minute verbal walkthrough — no deep architecture diagrams needed for L2/L3.
