# System Design: Zero to Advanced — Roadmap

A structured path through system design, from fundamentals to topics commonly discussed in **senior/staff-level backend and system design interviews**.

> **How to use this roadmap:** Work through the parts in order. Each section builds on the previous one. Don't just read—design systems, draw diagrams, estimate capacity, identify bottlenecks, and explain your trade-offs.

---

## Part 1 — Foundations

### Core Concepts

- What system design actually is (and isn't)
- Client-server model
- Request/response lifecycle
- IP addresses
- DNS
- TCP/IP basics
- HTTP/HTTPS
- HTTP methods
- HTTP status codes
- Headers, cookies, sessions
- TLS basics

### APIs

- What an API is
- REST
- RESTful API design
- Request/response lifecycle
- API versioning
- Pagination
- Filtering and sorting
- Idempotency
- Error handling
- API documentation

### Networking Basics

- TCP vs UDP
- Connection lifecycle
- Keep-alive connections
- HTTP/1.1 vs HTTP/2 vs HTTP/3
- WebSockets
- TLS termination

---

## Part 2 — Scaling Fundamentals

### Scaling Strategies

- Vertical scaling
- Horizontal scaling
- Scaling stateless services
- Stateful vs stateless architecture
- Bottleneck identification

### Load Balancing

- Why load balancing is needed
- Layer 4 vs Layer 7 load balancing
- Round-robin
- Weighted round-robin
- Least connections
- IP hashing
- Health checks
- Session affinity / sticky sessions

### Caching

- Why caching works
- Client-side caching
- Browser caching
- CDN caching
- Application-level caching
- Database caching
- Redis
- Cache-aside
- Read-through
- Write-through
- Write-back
- Cache invalidation
- TTL
- Cache stampede
- Cache penetration
- Cache avalanche

---

# Part 3 — Data Storage

## SQL vs NoSQL

- Relational databases
- Document databases
- Key-value databases
- Wide-column databases
- Graph databases
- When to use SQL
- When to use NoSQL
- Trade-offs

## Database Fundamentals

- Data modeling
- Normalization
- Denormalization
- Primary keys
- Foreign keys
- Constraints
- Transactions
- ACID
- Isolation levels
- Deadlocks

## Indexing

- What indexes actually do
- B-Tree indexes
- Hash indexes
- Composite indexes
- Covering indexes
- Index selectivity
- Query optimization
- `EXPLAIN` / query plans
- Index trade-offs

## Replication

- Primary-replica architecture
- Read replicas
- Replication lag
- Synchronous replication
- Asynchronous replication
- Multi-primary replication

## Partitioning & Sharding

- Horizontal partitioning
- Vertical partitioning
- Sharding
- Range-based sharding
- Hash-based sharding
- Directory-based sharding
- Hot partitions
- Rebalancing

## Database Scaling

- Read scaling
- Write scaling
- Connection pooling
- Database proxies
- Distributed databases
- Database failover

---

# Part 4 — Communication Patterns

## Synchronous Communication

- REST
- gRPC
- RPC
- Request/response
- Service-to-service communication

## Asynchronous Communication

- Message queues
- Kafka
- RabbitMQ
- Amazon SQS
- Producers and consumers
- Consumer groups
- Message ordering
- Message acknowledgment
- Retry queues
- Dead-letter queues
- At-least-once delivery
- At-most-once delivery
- Exactly-once semantics

## Event-Driven Systems

- Events
- Event producers
- Event consumers
- Event brokers
- Pub/Sub
- Event streams
- Event replay

## Real-Time Communication

- WebSockets
- Long polling
- Server-Sent Events (SSE)
- Connection management
- Presence
- Fan-out
- Real-time notifications

---

# Part 5 — Distributed Systems Theory

## Core Concepts

- What makes a system distributed?
- Network partitions
- Partial failures
- Failure detection
- Clock problems
- Distributed state

## CAP Theorem

- Consistency
- Availability
- Partition tolerance
- Why CAP is often misunderstood
- CAP vs PACELC

## Consistency Models

- Strong consistency
- Eventual consistency
- Read-your-writes consistency
- Monotonic reads
- Causal consistency

## Consensus

Understand conceptually:

- Paxos
- Raft
- Leader election
- Quorum
- Majority voting
- Split-brain
- Term/epoch concepts

## Distributed Coordination

- Distributed locks
- Leases
- Coordination services
- ZooKeeper
- etcd

---

# Part 6 — Architecture Patterns

## Application Architecture

- Monolith
- Modular monolith
- Microservices
- Serverless
- Service-oriented architecture

### Microservices

- Service boundaries
- Database-per-service
- Shared databases
- Service communication
- Service discovery
- Configuration management
- Distributed tracing
- Deployment strategies

## API Gateway

- Routing
- Authentication
- Rate limiting
- Request transformation
- Aggregation
- Load balancing

## Architecture Patterns

- Layered architecture
- Clean architecture
- Hexagonal architecture
- Event-driven architecture
- CQRS
- Event sourcing
- Backend-for-Frontend (BFF)
- Strangler Fig pattern

## Deployment Patterns

- Rolling deployment
- Blue-green deployment
- Canary deployment
- Feature flags
- Zero-downtime deployment

---

# Part 7 — Reliability & Performance

## Reliability

- Availability
- Reliability
- Fault tolerance
- Redundancy
- Failover
- Graceful degradation
- Disaster recovery

## Rate Limiting

Algorithms:

- Fixed window
- Sliding window
- Sliding window counter
- Token bucket
- Leaky bucket

Understand:

- Per-user limits
- Per-IP limits
- Global limits
- Distributed rate limiting

## Resilience Patterns

- Timeouts
- Retries
- Exponential backoff
- Jitter
- Circuit breakers
- Bulkheads
- Backpressure
- Load shedding
- Graceful degradation

## Observability

### Logging

- Structured logging
- Log levels
- Centralized logging
- Correlation IDs

### Metrics

- RED metrics
- USE metrics
- Latency
- Throughput
- Error rate
- Saturation

### Distributed Tracing

- Trace
- Span
- Trace propagation
- OpenTelemetry

---

# Part 8 — Security in System Design

## Authentication

- Sessions
- JWT
- OAuth 2.0
- OpenID Connect
- Access tokens
- Refresh tokens
- Token rotation

## Authorization

- RBAC
- ABAC
- Permission systems
- Service-to-service authorization

## Data Security

- Encryption in transit
- Encryption at rest
- Secrets management
- Key management
- Password hashing

## API Security

- Rate limiting
- Input validation
- CORS
- CSRF
- SQL injection
- XSS
- SSRF
- API abuse prevention

## Infrastructure Security

- Network segmentation
- Firewalls
- Private networks
- Zero-trust concepts
- Secret management
- IAM

---

# Part 9 — Capacity Planning & System Estimation

This deserves its own section because **system design interviews aren't just architecture diagrams**.

## Estimation Fundamentals

- Requests per second (RPS)
- Queries per second (QPS)
- Concurrent users
- Storage requirements
- Bandwidth requirements
- Read/write ratios
- Peak traffic
- Growth projections

## Back-of-the-Envelope Calculations

Learn to estimate:

- Daily active users
- Requests per second
- Storage per day
- Storage per year
- Network bandwidth
- Database size
- Cache size
- Number of servers

## Example

```text
10M users
× 10 requests/day
= 100M requests/day

100M / 86,400
≈ 1,157 requests/sec

Peak traffic × 3
≈ 3,500 RPS
```

---

# Part 10 — Storage & Infrastructure Building Blocks

Understand the role and trade-offs of:

- CDN
- Object storage
- Block storage
- File storage
- DNS
- Reverse proxies
- Load balancers
- Redis
- Message brokers
- Search engines
- Elasticsearch / OpenSearch
- Relational databases
- NoSQL databases

## Object Storage

- S3-style storage
- Presigned URLs
- Multipart uploads
- Large file uploads
- CDN integration
- Image/video processing

---

# Part 11 — Classic System Design Problems

This is where everything starts clicking.

For every problem, practice:

1. Requirements
2. Functional requirements
3. Non-functional requirements
4. Capacity estimation
5. API design
6. Data model
7. High-level architecture
8. Detailed components
9. Scaling strategy
10. Bottlenecks
11. Failure scenarios
12. Security
13. Monitoring
14. Trade-offs

## Beginner → Intermediate

- URL shortener
- Rate limiter
- Pastebin
- File storage system
- Notification service

## Intermediate

- Chat application
- News feed / timeline
- Notification system
- Search autocomplete
- Distributed cache
- Job scheduler
- API gateway

## Advanced

- Video streaming platform
- YouTube-style platform
- Netflix-style platform
- Ride-sharing system
- Uber-style location tracking
- Food delivery platform
- Payment system
- Ticket booking system
- Social media platform
- Distributed task queue
- Real-time collaboration system

---

# Part 12 — Advanced Distributed Systems

## Distributed Transactions

- Two-Phase Commit (2PC)
- Three-Phase Commit (3PC)
- Saga pattern
- Compensating transactions
- Transaction boundaries

## Consistent Hashing

- Hash rings
- Virtual nodes
- Rebalancing
- Distributed caches
- Distributed databases

## Probabilistic Data Structures

- Bloom filters
- Count-Min Sketch
- HyperLogLog
- Cuckoo filters

Understand:

- Why they exist
- What problem they solve
- Accuracy vs memory trade-offs

## Distributed Queues & Streams

- Partitioning
- Consumer groups
- Ordering
- Offsets
- Replay
- Backpressure
- Stream processing

---

# Part 13 — Search & Data Processing

## Search Systems

- Full-text search
- Inverted indexes
- Elasticsearch / OpenSearch
- Search ranking
- Autocomplete
- Fuzzy search
- Distributed search

## Data Pipelines

- ETL
- ELT
- Batch processing
- Stream processing
- Data warehouses
- Data lakes

---

# Part 14 — Multi-Region & Global Systems

- Multi-region architecture
- Active-active
- Active-passive
- Geo-routing
- Global load balancing
- Regional failover
- Data replication across regions
- Conflict resolution
- Latency optimization
- CDN strategy

## Disaster Recovery

- Backup strategies
- RPO
- RTO
- Disaster recovery plans
- Failover
- Data restoration
- Business continuity

---

# Part 15 — Massive-Scale Systems

Study these conceptually rather than trying to memorize implementation details.

- Vitess
- CockroachDB
- Google Spanner concepts
- Dynamo-style databases
- Distributed SQL
- Global databases
- Massive-scale Kafka
- Large-scale caching
- Multi-region databases

---

# Part 16 — Chaos Engineering

Learn how large systems are tested against failure.

- Failure injection
- Instance failures
- Network failures
- Latency injection
- Dependency failures
- Database failures
- Region failures
- Chaos experiments
- Recovery validation

Tools/concepts:

- Chaos Monkey
- Litmus
- Gremlin
- Kubernetes failure testing

---

# Part 17 — System Design Interview Skills

Don't leave this until the very end.

## The Interview Flow

### 1. Clarify Requirements

Ask:

- Who are the users?
- What are the core features?
- What scale are we designing for?
- What latency is acceptable?
- What's more important: consistency or availability?
- What data needs to be persistent?

### 2. Estimate Scale

Calculate:

- DAU/MAU
- RPS/QPS
- Read/write ratio
- Storage
- Bandwidth
- Peak traffic

### 3. Design the High-Level Architecture

Start simple:

```text
Client
   ↓
Load Balancer
   ↓
Application Servers
   ↓
Cache
   ↓
Database
```

Then add complexity only when the requirements justify it.

### 4. Deep Dive

Pick the most important bottleneck and go deeper.

Examples:

- Database scaling
- Caching
- Message queues
- Consistency
- Real-time communication
- Failure handling

### 5. Discuss Trade-offs

Always be able to explain:

> "Why did you choose X instead of Y?"

---

# Recommended Learning Order

```text
Foundations
     ↓
Scaling
     ↓
Databases
     ↓
Communication
     ↓
Distributed Systems
     ↓
Architecture
     ↓
Reliability
     ↓
Security
     ↓
Capacity Planning
     ↓
Classic Design Problems
     ↓
Advanced Distributed Systems
     ↓
Multi-Region / Massive Scale
     ↓
Interview Practice
```

---

# Timeline

| Part | Topic                          | Time Needed |
| ---- | ------------------------------ | ----------: |
| 1    | Foundations                    |    2–3 days |
| 2    | Scaling Fundamentals           |    4–5 days |
| 3    | Data Storage                   |      1 week |
| 4    | Communication Patterns         |    4–5 days |
| 5    | Distributed Systems Theory     |      1 week |
| 6    | Architecture Patterns          |    4–5 days |
| 7    | Reliability & Performance      |    4–5 days |
| 8    | Security                       |    2–3 days |
| 9    | Capacity Planning              |    2–3 days |
| 10   | Infrastructure Building Blocks |    3–4 days |
| 11   | Classic Design Problems        |   2–3 weeks |
| 12   | Advanced Distributed Systems   |   1–2 weeks |
| 13   | Search & Data Processing       |    3–5 days |
| 14   | Multi-Region Systems           |    4–5 days |
| 15   | Massive-Scale Systems          |    3–5 days |
| 16   | Chaos Engineering              |    2–3 days |
| 17   | Interview Practice             |     Ongoing |

### Total

**~10–14 weeks** for solid interview-level coverage.

For deeper mastery, expect **3–6+ months** of continuous practice.

---

# The Rule: Learn → Design → Explain → Repeat

For every major topic:

```text
Learn the concept
      ↓
Understand the trade-offs
      ↓
Implement a small version
      ↓
Design a production-scale version
      ↓
Identify bottlenecks
      ↓
Break the system intentionally
      ↓
Explain your decisions
```

Reading 40 system-design articles won't make the knowledge stick nearly as well as **designing 10 systems yourself**.

---

# Practice Projects

A good progression:

### Level 1

- URL Shortener
- Rate Limiter
- Pastebin
- File Upload Service

### Level 2

- Chat System
- Notification System
- News Feed
- Job Queue
- Search Autocomplete

### Level 3

- Ride Sharing
- Video Streaming
- Payment System
- Food Delivery
- Ticket Booking

### Level 4

- Multi-region social platform
- Global notification infrastructure
- Distributed job scheduler
- Global file storage system
- Large-scale event processing platform

---

# Final Goal

By the end of this roadmap, you should be able to look at a problem and naturally think:

```text
Requirements
     ↓
Scale
     ↓
APIs
     ↓
Data Model
     ↓
Architecture
     ↓
Bottlenecks
     ↓
Caching
     ↓
Database Scaling
     ↓
Async Processing
     ↓
Reliability
     ↓
Security
     ↓
Observability
     ↓
Failure Scenarios
     ↓
Trade-offs
```

The goal isn't to memorize architectures.

**The goal is to understand why a system should be built a certain way—and what breaks when you push it harder.**
