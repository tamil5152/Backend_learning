# Backend From First Principles 

> A journey to understand backend engineering from the fundamentals — what happens underneath the frameworks, tools, and abstractions we use every day.

Most backend tutorials teach you **how to use a framework**.

This repository is about understanding **what is happening underneath it**.

Instead of only learning:

```text
"How do I build an API?"
```

the goal is to understand:

```text
What is an API actually doing?
Why does routing exist?
How does data travel across the network?
How does a database store and retrieve data?
How does authentication work?
How does a backend handle failures?
How does a system scale?
How do multiple services communicate?
```

The idea is simple:

> **Learn the fundamentals, and the frameworks become obvious.**

---

# 🧠 Learning Philosophy

This repository follows a **first-principles approach**.

For every concept, the goal is to understand:

```text
Problem
   ↓
Why does it exist?
   ↓
How does it work?
   ↓
How is it implemented?
   ↓
How is it used in real systems?
```

The focus is not on memorizing frameworks.

The focus is on understanding the **machinery underneath them**.

---

# 📚 Backend Engineering Roadmap

The journey is divided into **24 chapters**.

```text
Backend Engineering
        │
        ├── The Request Path
        │       ├── HTTP
        │       ├── Routing
        │       ├── Serialization
        │       ├── Authentication
        │       ├── Validation
        │       ├── Controllers & Middleware
        │       └── REST API Design
        │
        ├── State & Machinery
        │       ├── Databases
        │       ├── Caching
        │       ├── Background Jobs
        │       ├── Full-Text Search
        │       ├── Fault Tolerance
        │       ├── gRPC
        │       └── Configuration
        │
        ├── Running in Production
        │       ├── Observability
        │       ├── Graceful Shutdown
        │       ├── Security
        │       ├── Scaling & Performance
        │       └── Concurrency
        │
        └── Distribution & Scale
                ├── Docker & Kubernetes
                ├── Automated Testing
                ├── Kafka & Message Brokers
                └── WebSockets & Real-Time
```

---

# 🛣️ Part 1 — The Request Path

> How does a request travel from a client to a backend and become a response?

## 01. HTTP

Understanding the application-layer protocol that every backend engineer works with.

Topics:

- HTTP Requests & Responses
- HTTP Methods
- Headers
- Status Codes
- HTTP Body
- Statelessness
- Caching
- Conditional Requests
- Proxies
- TLS

Status: ✅ Completed

---

## 02. Routing

Understanding how the backend decides **where a request should go**.

Topics:

- Routes
- Static Routes
- Dynamic Routes
- Path Parameters
- Query Parameters
- Nested Routes
- Versioning
- Catch-All Routes

Status: ✅ Completed

---

## 03. Serialization & Deserialization

Understanding how different programming languages communicate using a common data format.

Topics:

- Serialization
- Deserialization
- JSON
- Text-Based Formats
- Binary Formats
- Language-Agnostic Communication
- Client → Server Communication
- OSI Mental Model

Basic idea:

```text
Native Data
     ↓
Serialization
     ↓
JSON
     ↓
Network
     ↓
JSON
     ↓
Deserialization
     ↓
Native Data
```

Status: ✅ Completed

---

## 04. Authentication & Authorization

Understanding two fundamental questions:

```text
Who are you?
     ↓
Authentication

What are you allowed to do?
     ↓
Authorization
```

Topics:

- Sessions
- Cookies
- JWT
- OAuth 2.0
- OpenID Connect
- RBAC
- Shared Secrets

Status: 🚧 In Progress

---

## 05. Validations & Transformations

Understanding how client data is checked and transformed before reaching business logic.

Topics:

- Input Validation
- Data Transformation
- Validation Rules
- API Boundaries
- Where Validation Should Live

Status: ⏳ Upcoming

---

## 06. Controllers, Services & Middleware

Understanding what happens **inside the server** after a request reaches the correct route.

```text
Request
   ↓
Routing
   ↓
Middleware
   ↓
Controller
   ↓
Service
   ↓
Database
   ↓
Response
```

Topics:

- Controllers
- Services
- Middleware
- Request Context
- Middleware Chains
- Separation of Concerns

Status: ⏳ Upcoming

---

## 07. API Design — REST

Understanding REST as a set of architectural agreements rather than a framework.

Topics:

- REST Principles
- Resources
- HTTP Methods
- PUT vs PATCH
- Status Codes
- URL Design
- API Versioning
- Idempotency

Status: ⏳ Upcoming

---

# ⚙️ Part 2 — State & Machinery

> Where data lives and the systems that operate around it.

## 08. Databases

Understanding why databases exist and how they actually work.

Topics:

- Database Fundamentals
- Tables
- Rows
- Columns
- Relationships
- SQL
- Transactions
- Indexes
- Query Performance
- Triggers

Status: ⏳ Upcoming

---

## 09. Caching

Understanding how backend systems become faster by avoiding unnecessary work.

Topics:

- Cache
- Cache Hit
- Cache Miss
- TTL
- In-Memory Caching
- Redis
- Cache Invalidation

Status: ⏳ Upcoming

---

## 10. Task Queues & Background Jobs

Understanding how expensive or delayed work can be moved outside the request-response cycle.

Topics:

- Task Queues
- Workers
- Producers
- Consumers
- Background Jobs
- Retry Mechanisms
- Delayed Jobs

Status: ⏳ Upcoming

---

## 11. Full-Text Search

Understanding how applications search large amounts of text efficiently.

Topics:

- Full-Text Search
- Inverted Indexes
- Elasticsearch
- Search Queries
- Relevance

Status: ⏳ Upcoming

---

## 12. Error Handling & Fault Tolerance

Understanding how backend systems behave when things go wrong.

Topics:

- Errors
- Retries
- Timeouts
- Failure Handling
- Fault Tolerance
- Recovery
- Resilience

Status: ⏳ Upcoming

---

## 13. gRPC & Inter-Service Communication

Understanding how backend services communicate with each other.

Topics:

- RPC
- Protocol Buffers
- Binary Wire Format
- HTTP/2
- Streaming
- Deadlines
- Interceptors
- mTLS
- Service-to-Service Communication

Status: ⏳ Upcoming

---

## 14. Configuration Management

Understanding how the same application behaves differently across environments.

```text
Development
     ↓
   Config
     ↓
Same Application
     ↓
Production
```

Topics:

- Environment Variables
- Configuration
- Secrets
- Development Environment
- Production Environment

Status: ⏳ Upcoming

---

# 🚀 Part 3 — Running in Production

> Keeping backend systems observable, secure, reliable, and fast under real-world load.

## 15. Logging & Observability

Understanding how we know what our backend is doing in production.

Topics:

- Logging
- Metrics
- Tracing
- Monitoring
- Observability
- Debugging Production Systems

Status: ⏳ Upcoming

---

## 16. Graceful Shutdown

Understanding how a backend should stop without losing ongoing work.

```text
Shutdown Signal
      ↓
Stop Accepting New Requests
      ↓
Finish Existing Work
      ↓
Close Resources
      ↓
Exit
```

Topics:

- Shutdown Signals
- In-Flight Requests
- Connection Cleanup
- Resource Cleanup

Status: ⏳ Upcoming

---

## 17. Backend Security

Understanding the security principles every backend engineer should know.

Topics:

- Secure Communication
- Authentication Security
- Authorization
- Input Security
- Secrets
- Common Backend Vulnerabilities
- Secure API Design

Status: ⏳ Upcoming

---

## 18. Scaling & Performance — Part 1

Understanding how to make a backend handle increasing workloads.

Topics:

- Performance
- Latency
- Throughput
- Bottlenecks
- Load
- Vertical Scaling
- Horizontal Scaling

Status: ⏳ Upcoming

---

## 19. Scaling & Performance — Part 2

Going deeper into backend scalability.

Topics:

- Load Balancing
- Database Scaling
- Caching
- Stateless Servers
- Replication
- Performance Optimization

Status: ⏳ Upcoming

---

## 20. Concurrency & Parallelism

Understanding how backend systems handle multiple operations.

Topics:

- Processes
- Threads
- Concurrency
- Parallelism
- Race Conditions
- Locks
- Synchronization
- Async Programming

Status: ⏳ Upcoming

---

# 🌐 Part 4 — Distribution & Scale

> Shipping systems, testing them, and connecting services together.

## 21. Docker, Kubernetes & CI/CD

Understanding how backend applications are packaged, deployed, and operated.

Topics:

- Containers
- Linux Namespaces
- cgroups
- Docker Images
- Docker Layers
- Multi-Stage Builds
- Kubernetes
- Pods
- Deployments
- Services
- Health Checks
- Autoscaling
- CI/CD
- Zero-Downtime Deployment

Status: ⏳ Upcoming

---

## 22. Automated Testing

Understanding how backend engineers verify that their systems actually work.

Topics:

- Unit Testing
- Test Doubles
- Integration Testing
- Database Testing
- HTTP Handler Testing
- Contract Testing
- End-to-End Testing
- Test Coverage
- Flaky Tests
- TDD
- Load Testing

Status: ⏳ Upcoming

---

## 23. Message Brokers & Kafka

Understanding event-driven systems and distributed message processing.

Topics:

- Message Brokers
- Kafka
- Producers
- Consumers
- Topics
- Partitions
- Consumer Groups
- Offsets
- Delivery Semantics
- Retention
- Event-Driven Architecture
- Outbox Pattern
- CDC
- Stream Processing

Status: ⏳ Upcoming

---

## 24. WebSockets & Real-Time Systems

Understanding how applications maintain real-time communication.

Topics:

- WebSockets
- Upgrade Handshake
- Frames
- Connection Lifecycle
- Persistent Connections
- Backpressure
- Pub/Sub
- Scaling Stateful Connections

Basic idea:

```text
Client
   │
   │ WebSocket Connection
   │
   ↕
Server
   │
   ├── Connection 1
   ├── Connection 2
   ├── Connection 3
   └── ...
```

Status: ⏳ Upcoming

---

# 🛠️ Languages

The concepts are explored using multiple languages where useful.

- Go
- Python
- JavaScript
- Java
- Rust

The purpose isn't to learn five languages.

The purpose is to understand that **backend concepts exist independently of a particular programming language**.

---

# 🔬 Theory + Code

Each chapter aims to combine:

```text
Theory
  +
Mental Model
  +
Diagrams
  +
Code
  +
Experiments
```

The goal is to move from:

```text
"I know the definition."
```

to:

```text
"I understand why it exists."

"I understand how it works."

"I can implement a simple version."

"I understand how production systems use it."
```

---

# 📂 Repository Structure

```text
backend-from-first-principles/
│
├── 01-http/
│   └── README.md
│
├── 02-routing/
│   └── README.md
│
├── 03-serialization/
│   └── README.md
│
├── 04-authentication/
│   └── README.md
│
├── 05-validations/
│   └── README.md
│
├── 06-controllers-services-middlewares/
│   └── README.md
│
├── 07-rest-api-design/
│   └── README.md
│
├── 08-databases/
│   └── README.md
│
├── ...
│
└── 24-websockets/
    └── README.md
```

---

# 📈 Progress

Current progress:

```text
████████░░░░░░░░░░░░░░  3 / 24
```

Completed:

- [x] 01 — HTTP
- [x] 02 — Routing
- [x] 03 — Serialization & Deserialization

Coming next:

- [ ] 04 — Authentication & Authorization
- [ ] 05 — Validations & Transformations
- [ ] 06 — Controllers, Services & Middleware
- [ ] 07 — REST API Design

---

# 📖 Learning in Public

This repository is also a way of **learning in public**.

Every chapter represents something I am learning, understanding, implementing, and documenting.

The notes may evolve over time as my understanding improves.

If something is incorrect, incomplete, or could be explained better, contributions and corrections are welcome.

---

# 🤝 Contributing

Found:

- An incorrect explanation?
- A missing edge case?
- A better diagram?
- A bug in the implementation?
- A better way to explain a concept?

Feel free to open an **Issue** or submit a **Pull Request**.

The chapters are written in Markdown so they can be easily improved and discussed.

---

# 🎯 The Goal

The ultimate goal is not to memorize 24 chapters.

The goal is to build a mental model of how backend systems work.

From:

```text
HTTP Request
```

to:

```text
Routing
```

to:

```text
Application Logic
```

to:

```text
Database
```

to:

```text
Caching
```

to:

```text
Concurrency
```

to:

```text
Distributed Systems
```

to:

```text
Production Infrastructure
```

Until the entire system starts making sense as one connected machine.

---

# 🚀 Build From First Principles

> **Don't just learn how to use the tools.**
>
> **Understand why the tools exist.**
>
> **Understand what happens underneath them.**

**Learn the fundamentals, and the frameworks become obvious.**
