# Week 4 – Day 28: Advanced System Design Revision

Day 28 of 49 Advanced Revision & Interview Preparation

File name: `28-Advanced-System-Design-Revision.md`

Goal: Consolidate everything learned in Week 4 and develop the ability to explain advanced system design concepts during Senior Android Developer interviews.

You have completed:

- Day 22: Microservices Architecture
- Day 23: Distributed System Design Patterns
- Day 24: Database Internals and Advanced Storage
- Day 25: Search and Notification Systems
- Day 26: Real-Time Systems
- Day 27: Security and Observability

Today, we will connect these concepts instead of learning them independently.

# Part 1 – Microservices Architecture Revision

## 1. Monolithic vs Microservices Architecture

### Monolithic Architecture

A monolithic application contains multiple functionalities inside one deployable application.

Example: E-commerce application.

- User Management
- Product Catalog
- Order Management
- Payment Management
- Notification Management

All these modules are part of one application.

Monolithic vs. Microservices: The Great Architecture Debate | Last9

[last9.io](<https://last9.io/blog/monolithic-vs-microservices/?utm_source=chatgpt.com>)

Advantages:

- Simple development initially.
- Easier deployment.
- Easier debugging in small applications.
- Less infrastructure complexity.

Disadvantages:

- Difficult to scale individual modules.
- Large codebase becomes difficult to maintain.
- One failure can affect the entire application.
- Deployment of one module may require deploying everything.

### Microservices Architecture

The application is divided into independently deployable services.

Microservice | Medium

[simsonmoses.medium.com](<https://simsonmoses.medium.com/microservices-architecture-pattern-844c6cdba17c?utm_source=chatgpt.com>)

Example:

| Service | Responsibility |
| --- | --- |
| User Service | User management |
| Product Service | Product catalog |
| Order Service | Order management |
| Payment Service | Payments |
| Notification Service | Notifications |

Each service can be developed, deployed, and scaled independently.

### Interview comparison

| Feature | Monolith | Microservices |
| --- | --- | --- |
| Deployment | Usually one unit | Independent services |
| Scaling | Often whole application | Individual services |
| Complexity | Lower initially | Higher |
| Communication | Internal function calls | Network communication |
| Failure isolation | More difficult | Better with proper design |
| Data management | Often shared database | Often service-owned data |

Senior-level insight: Microservices are not always better. For a small application, a modular monolith may be simpler, cheaper, and more maintainable.

# Part 2 – Distributed System Design Patterns

## 2. Circuit Breaker Pattern

Imagine your Android application communicates with a Payment Service.

Suddenly, the Payment Service becomes unavailable.

Without protection, every incoming request continues trying to communicate with the failing service.

This can create cascading failures.

The Circuit Breaker pattern prevents repeated calls to an unhealthy service.

### Three states

CLOSED

Normal

Requests are allowed to pass through.OPEN

Fail fast

Requests are rejected or handled through a fallback without calling the failing service.HALF-OPEN

Testing recovery

A limited number of test requests are allowed to check whether the service has recovered.

Designing for Failure: Mastering Timeouts, Retries, and Circuit Breakers

[systemdr.systemdrd.com](<https://systemdr.systemdrd.com/p/designing-for-failure-mastering-timeouts?utm_source=chatgpt.com>)

### Example

Suppose Payment Service starts failing.

1. Initially, circuit is CLOSED.
2. Failures exceed a configured threshold.
3. Circuit changes to OPEN.
4. Requests fail fast instead of repeatedly calling Payment Service.
5. After a configured waiting period, circuit changes to HALF-OPEN.
6. Test requests are allowed.
7. If successful, circuit returns to CLOSED.

### When is it useful?

- Microservice communication.
- External payment gateways.
- Third-party APIs.
- Notification providers.
- Unreliable downstream services.

Interview question: What is the difference between retry and circuit breaker?

Answer:

- Retry attempts the failed operation again.
- Circuit breaker temporarily prevents calls to an unhealthy dependency.

They complement each other, but retries should be bounded and use backoff to avoid increasing load during failures.

## 3. Retry Pattern

Retry means attempting an operation again after a temporary failure.

Example:

An API request fails because of a temporary network problem.

The client retries the request.

### Common retry strategies

| Strategy | Explanation |
| --- | --- |
| Fixed Delay | Wait a fixed amount before retry |
| Exponential Backoff | Increase delay after every retry |
| Exponential Backoff + Jitter | Add randomness to retry delays |

Example:

text

```text
Attempt 1: 1 second
Attempt 2: 2 seconds
Attempt 3: 4 seconds
Attempt 4: 8 seconds
```

### Important considerations

- Set a maximum retry count.
- Use timeouts.
- Add jitter.
- Retry only appropriate failures.
- Avoid retrying non-idempotent operations blindly.

For payment APIs, use idempotency keys to prevent duplicate transactions.

## 4. Timeout Pattern

A timeout defines the maximum time an operation is allowed to take.

Example:

text

```text
Payment API timeout = 5 seconds
```

If the service does not respond within the configured time, the operation is treated as timed out.

Why are timeouts important?

Without timeouts, requests may remain blocked for too long, consuming connections and other resources.

Types:

- Connection timeout.
- Read timeout.
- Write timeout.
- Overall request deadline.

Senior-level insight: A timeout does not necessarily mean that the server failed to process the request. It may have completed the operation but failed to return the response in time.

## 5. Bulkhead Pattern

The Bulkhead pattern isolates resources so that a failure in one component does not consume all resources and affect other components.

The name comes from compartments in ships.

If one compartment floods, the others remain protected.

🛡️ Bulkhead Pattern: Isolating Failures in Microservices Architecture | by Dolly | Mar, 2026 | Medium

[medium.com](<https://medium.com/%40gangoladeepa/%EF%B8%8F-bulkhead-pattern-isolating-failures-in-microservices-architecture-f00f13a8de3e?utm_source=chatgpt.com>)

Example:

An e-commerce backend has:

- Payment Service.
- Product Service.
- Notification Service.

If notification processing becomes slow, it should not consume all resources needed by payment processing.

Solutions include:

- Separate thread pools.
- Separate connection pools.
- Resource limits.
- Independent service instances.

## 6. Saga Pattern

The Saga pattern manages distributed transactions across multiple services using a sequence of local transactions.

Imagine a food delivery order.

1. Create order.
2. Reserve restaurant inventory.
3. Process payment.
4. Assign delivery partner.

What happens if payment fails after the order has been created and inventory reserved?

A traditional single-database transaction cannot easily roll back changes across independent services.

Saga solves this using compensating transactions.

Cloud-native data patterns - .NET | Microsoft Learn

[learn.microsoft.com](<https://learn.microsoft.com/en-us/dotnet/architecture/cloud-native/distributed-data?utm_source=chatgpt.com>)

### Example

| Step | Operation | Compensation |
| --- | --- | --- |
| 1 | Create Order | Cancel Order |
| 2 | Reserve Inventory | Release Inventory |
| 3 | Process Payment | Refund Payment |
| 4 | Assign Delivery | Cancel Assignment |

If payment fails:

- Cancel the order.
- Release reserved inventory.

These are compensating operations, not necessarily literal database rollbacks.

### Two Saga approaches

Choreography

Services communicate through events.

Example:

- OrderCreated
- InventoryReserved
- PaymentCompleted

Orchestration

A central orchestrator coordinates the steps.

### Interview question

What is the difference between Saga and a traditional database transaction?

Answer: A traditional transaction typically provides atomicity within a database transaction boundary. Saga coordinates multiple local transactions across services using compensating actions when necessary.

# Part 3 – CQRS and Event Sourcing

## 7. CQRS

CQRS stands for Command Query Responsibility Segregation.

It separates operations that modify data from operations that read data.

### Traditional architecture

The same model is used for both reading and writing.

### CQRS architecture

- Command side: Handles write operations.
- Query side: Handles read operations.

System Design Interview Cheat Sheet: The 12 API Design Patterns You Need to Know

[designgurus.substack.com](<https://designgurus.substack.com/p/12-essential-api-design-patterns?utm_source=chatgpt.com>)

### Example: E-commerce

Command:

text

```text
PlaceOrder
CancelOrder
UpdateProduct
```

Query:

text

```text
GetOrderDetails
GetProductCatalog
GetUserOrders
```

The write model can enforce business rules, while the read model can be optimized for retrieving information.

### Advantages

- Independent read/write optimization.
- Better separation of responsibilities.
- Suitable for complex business domains.
- Can support different data models for reading and writing.

### Disadvantages

- More architectural complexity.
- Data synchronization challenges.
- Eventual consistency may occur.
- Additional infrastructure.

Important: CQRS does not automatically mean using two different databases. Separate models can exist even when the same database is used.

## 8. Event Sourcing

Event Sourcing stores changes as a sequence of events instead of storing only the latest state.

Example: Bank account.

Traditional storage:

json

```json
{
  "accountId": "101",
  "balance": 5000
}
```

Event Sourcing:

text

```text
AccountCreated: 0
MoneyDeposited: +10000
MoneyWithdrawn: -3000
MoneyWithdrawn: -2000
```

Current balance is calculated from the events.

$$
Balance = \sum Events
$$

### How it works

1. A business operation occurs.
2. The system creates an event.
3. The event is stored in an append-oriented event store.
4. Consumers process the event.
5. Read models are updated.

API Development Series Part 2: Data Layer & Persistence - Wasil Zafar

[wasilzafar.com](<https://www.wasilzafar.com/pages/series/api-development/api-dev-data-persistence.html?utm_source=chatgpt.com>)

### Advantages

- Complete historical record.
- Easier auditing.
- Ability to reconstruct state.
- Supports event-driven systems.

### Disadvantages

- Complex event versioning.
- Event replay can be expensive.
- Read models need synchronization.
- Sensitive data deletion requires careful design.

### CQRS vs Event Sourcing

| CQRS | Event Sourcing |
| --- | --- |
| Separates reads and writes | Stores changes as events |
| Does not require event storage | Event store is central |
| Can use conventional databases | Uses an event-based persistence model |
| Can be used independently | Can be combined with CQRS |

# Part 4 – Database and Distributed Systems Revision

## 9. SQL vs NoSQL

| Feature | SQL | NoSQL |
| --- | --- | --- |
| Data model | Relational | Multiple models |
| Schema | Usually structured | Often flexible |
| Relationships | Strong relational support | Depends on database |
| Transactions | Strong support | Varies by database |
| Scaling | Vertical and horizontal options | Often designed for horizontal scaling |
| Examples | PostgreSQL, MySQL | MongoDB, Cassandra, DynamoDB |

### When should you choose SQL?

- Banking transactions.
- Orders.
- Payments.
- Inventory.
- Complex relationships.

### When should you choose NoSQL?

- Large-scale key-value access.
- Flexible document structures.
- High-volume event data.
- Certain distributed workloads.

Interview insight: Never choose a database simply because it is SQL or NoSQL. Choose based on access patterns, consistency, query requirements, scale, and operational needs.

## 10. Database Indexing

An index helps a database locate records without scanning every row.

Example:

sql

```sql
SELECT *
FROM users
WHERE email = 'john@example.com';
```

Without an appropriate index, the database might scan many rows.

With an index on email, lookup can be much faster.

### B-Tree Index

B-Tree-based indexes organize keys in a balanced tree structure.

\[Database\] 인덱스(Index) 동작 원리와 장단점 | 프로의 개발일지

[devpro.kr](<https://devpro.kr/posts/Database-%EC%9D%B8%EB%8D%B1%EC%8A%A4%28Index%29-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC%EC%99%80-%EC%9E%A5%EB%8B%A8%EC%A0%90/?utm_source=chatgpt.com>)

Advantages:

- Efficient searching.
- Supports range queries.
- Maintains sorted key order.

### Important trade-off

Indexes improve reads but can increase:

- Storage usage.
- Insert overhead.
- Update overhead.
- Delete overhead.

## 11. Database Replication

Replication means maintaining copies of data across multiple database servers.

Nextdoor’s Database Evolution: A Scaling Ladder

[blog.bytebytego.com](<https://blog.bytebytego.com/p/nextdoors-database-evolution-a-scaling?utm_source=chatgpt.com>)

### Primary-Replica Architecture

- Primary: Handles writes.
- Replicas: Often handle reads.

Advantages:

- Better read scalability.
- Improved availability.
- Data redundancy.

Potential problem:

Replication lag can cause stale reads.

Example:

User updates profile information on the primary database but immediately reads from a replica that has not received the update yet.

The old data may be returned.

## 12. Distributed Locks

A distributed lock coordinates access to a shared resource across multiple processes or servers.

Example:

Two users attempt to purchase the last available product.

Without coordination, both requests might successfully reserve the same inventory.

A distributed locking strategy can help prevent conflicting operations.

Common technologies:

- Redis.
- ZooKeeper.
- Database-based locking.

### Important considerations

- Lock expiration.
- Deadlocks.
- Network partitions.
- Lock ownership.
- Fencing tokens.

Senior-level insight: Distributed locks are not automatically safe just because they have a TTL. In critical systems, fencing tokens or transactional database constraints may be necessary to protect against stale lock holders.

# Part 5 – Search and Notification Architecture

## 13. Search Architecture

Imagine an e-commerce application with 10 million products.

Users expect:

- Fast product search.
- Filters.
- Sorting.
- Suggestions.
- Relevant results.

Searching every product directly in a transactional database may not be ideal.

A dedicated search engine can help.

Search at Scale: How E-Commerce Platforms Deliver Results in Milliseconds | by Nishant Kumar | Medium

[medium.com](<https://medium.com/%40nishantkumar.jss/search-at-scale-how-e-commerce-platforms-deliver-results-in-milliseconds-a5130a44debe?utm_source=chatgpt.com>)

### How it works

1. Product information is stored in the primary database.
2. Product changes generate events.
3. Indexing service consumes those events.
4. Search engine updates its index.
5. User submits a search query.
6. Search service queries the search engine.
7. Results are returned to Android.

### Important concepts

- Inverted Index.
- Full-Text Search.
- Tokenization.
- Ranking.
- Filtering.
- Autocomplete.
- Search Indexing.
- Eventual Consistency.

### Common technologies

- Elasticsearch.
- OpenSearch.
- Apache Solr.

## 14. Notification Architecture

Notifications can be delivered through several channels.

- Push notifications.
- Email.
- SMS.
- In-app notifications.

Design a Notification System: A Complete System Design Guide | by Madhur Banger | Medium

[medium.com](<https://medium.com/%40bangermadhur/design-a-notification-system-a-complete-system-design-guide-3b20d49298de?utm_source=chatgpt.com>)

### Example: Order confirmation

1. User places an order.
2. Order Service creates an order.
3. Order event is generated.
4. Notification Service consumes the event.
5. Notification Service selects the appropriate channel.
6. FCM delivers a push notification.
7. Android displays the notification.

### Important concepts

- Notification templates.
- Retry policies.
- Dead-letter queues.
- Idempotency.
- Delivery tracking.
- User preferences.
- Notification priority.
- Provider failures.

### FCM vs WebSocket

| FCM | WebSocket |
| --- | --- |
| Platform-managed push | Persistent connection |
| Suitable for background notifications | Suitable for active sessions |
| Primarily server-to-device | Bidirectional |
| Delivery timing is not guaranteed to be immediate | Low-latency communication while connected |

# Part 6 – Real-Time Systems Revision

## 15. Polling vs Long Polling vs SSE vs WebSockets

| Feature | Polling | Long Polling | SSE | WebSocket |
| --- | --- | --- | --- | --- |
| Communication | Request-response | Request-response | One-way | Bidirectional |
| Connection | Repeated | Temporarily held | Persistent | Persistent |
| Protocol | HTTP | HTTP | HTTP | WebSocket |
| Suitable for | Infrequent updates | Simple server updates | Live feeds | Chat and interactive apps |

### When should we choose WebSockets?

- Chat applications.
- Multiplayer games.
- Collaborative editing.
- Interactive dashboards.
- Real-time location updates.

### When should we choose SSE?

- Live monitoring dashboards.
- Stock price feeds.
- Server-generated updates.
- Progress streaming.

### Android connection

For active chat sessions:

- OkHttp WebSocket.
- Kotlin Coroutines.
- Kotlin Flow.
- Room.

For background notifications:

- FCM.

For offline synchronization:

- Room and WorkManager.

# Part 7 – Security and Observability Revision

## 16. Security Concepts

| Concept | Purpose |
| --- | --- |
| HTTPS | Secure HTTP communication |
| TLS | Encryption and authentication |
| AES | Symmetric encryption |
| RSA / ECC | Asymmetric cryptography |
| SHA-256 | Cryptographic hashing |
| Authentication | Verify identity |
| Authorization | Verify permissions |
| JWT | Token format |
| OAuth 2.0 | Authorization framework |
| Android Keystore | Secure cryptographic key management |
| Certificate Pinning | Restrict accepted server certificates or keys |

### Important Android security checklist

- Always use HTTPS.
- Protect authentication tokens.
- Use Android Keystore for sensitive cryptographic keys.
- Avoid logging sensitive information.
- Validate permissions on the backend.
- Disable cleartext traffic.
- Apply secure local storage.
- Handle authentication expiration.
- Avoid hardcoded secrets in the APK.

## 17. Observability Concepts

Remember the three pillars:

Logs

Events and errors

Metrics

Numerical measurements

Traces

Request journey

Important terms:

- SLI: Service Level Indicator.
- SLO: Service Level Objective.
- SLA: Service Level Agreement.
- P95: 95th percentile latency.
- P99: 99th percentile latency.
- Alerting: Notifications when conditions are violated.

# Part 8 – Putting Everything Together

## 18. Design a Scalable E-commerce System

This is today's main revision exercise.

### Functional requirements

- User registration and login.
- Product browsing.
- Product search.
- Cart management.
- Order creation.
- Payment processing.
- Notifications.
- Order tracking.

### High-level architecture

Designing an E-commerce Platform in 45 Minutes: A Complete System Design Interview Guide

[designgurus.substack.com](<https://designgurus.substack.com/p/designing-an-e-commerce-platform?utm_source=chatgpt.com>)

### Architecture components

| Component | Responsibility |
| --- | --- |
| Android App | User interface |
| API Gateway | Entry point for APIs |
| Auth Service | Authentication |
| Product Service | Product management |
| Search Service | Product search |
| Cart Service | Cart operations |
| Order Service | Order processing |
| Inventory Service | Stock management |
| Payment Service | Payment processing |
| Notification Service | Push, email, SMS |
| Kafka | Event distribution |
| Redis | Caching |
| SQL Database | Transactional storage |
| Search Engine | Search indexing |

### Complete order flow

1. User logs in.
2. Android obtains authentication credentials/tokens.
3. User searches for a product.
4. Search Service retrieves matching products.
5. User adds the product to the cart.
6. User places an order.
7. Order Service creates the order.
8. Inventory Service reserves stock.
9. Payment Service processes payment.
10. Order status is updated.
11. Notification Service sends a confirmation.
12. Android receives the notification.
13. User tracks order status using real-time communication where needed.

### Where do today's concepts fit?

| Requirement | Design technique |
| --- | --- |
| Service isolation | Microservices |
| Temporary service failures | Circuit Breaker |
| Temporary network failures | Retry with backoff |
| Long-running checkout workflow | Saga |
| Read/write separation | CQRS |
| Business event history | Event Sourcing |
| Fast product retrieval | Search Engine |
| Fast repeated reads | Redis Cache |
| Order event distribution | Kafka |
| Secure API communication | HTTPS/TLS |
| User permissions | Authorization |
| Production debugging | Logs, Metrics, Traces |
| Service health | Monitoring and Alerting |

# Part 9 – Senior Android Developer Interview Questions

These questions combine the topics from Days 22–27.

## Basic Questions

Q1. What is the difference between monolithic and microservices architecture?

A monolith is generally deployed as one application, while microservices consist of independently deployable services.

Q2. What is API Gateway?

An entry point that routes client requests to appropriate backend services and can provide cross-cutting capabilities such as authentication and rate limiting.

Q3. What is Circuit Breaker?

A pattern that temporarily blocks requests to a failing dependency to prevent cascading failures.

Q4. What is CQRS?

An architectural pattern that separates write operations from read operations.

Q5. What is Event Sourcing?

A pattern that stores state changes as a sequence of events.

Q6. What is a message broker?

Infrastructure that enables asynchronous communication between services.

## Intermediate Questions

Q7. What is Saga?

A distributed transaction pattern that coordinates local transactions and compensating actions.

Q8. What is the difference between Kafka and Redis Pub/Sub?

Kafka provides durable event streaming and replay capabilities. Redis Pub/Sub provides transient message distribution without durable subscriber delivery.

Q9. What is database replication?

Maintaining copies of database data across multiple database instances.

Q10. What is eventual consistency?

A consistency model in which replicas may temporarily contain different values but converge when updates propagate.

Q11. How does a search engine work?

It maintains an optimized search index, often using an inverted index, to retrieve relevant documents efficiently.

Q12. How do WebSockets scale?

Through horizontally scaled connection gateways, connection routing, shared messaging infrastructure, and appropriate connection management.

## Advanced Questions

Q13. How do you prevent duplicate payment processing?

Use idempotency keys, durable transaction state, and appropriate payment reconciliation.

Q14. How do you handle a failed downstream microservice?

Use bounded retries, timeouts, circuit breakers, bulkheads, and graceful degradation.

Q15. How do you maintain message ordering?

Use partitioning or sequence numbers based on the required ordering scope and reconcile events when necessary.

Q16. How do you design an offline-first Android application?

Use Room as the local source of truth, repositories for data coordination, WorkManager for deferrable synchronization, and conflict resolution strategies.

Q17. How do you monitor a distributed application?

Use structured logs, metrics, distributed traces, dashboards, and alerting.

Q18. How do you protect an Android application?

Use HTTPS, secure token storage, Android Keystore, proper backend authorization, secure logging, and appropriate application hardening.

# Part 10 – Mock System Design Interview

## 19. Interview Scenario: Design WhatsApp

WhatsApp System Design Case Study | Prepflix

[prepflix.co.in](<https://prepflix.co.in/blogs/class-1-post-class.html?utm_source=chatgpt.com>)

Imagine the interviewer asks:

"Design a chat application similar to WhatsApp that supports millions of users."

You should structure your answer in the following order.

### Step 1: Requirements

Functional:

- One-to-one messaging.
- Group messaging.
- Online/offline status.
- Message delivery receipts.
- Read receipts.
- Message history.
- Push notifications.

Non-functional:

- Low latency.
- High availability.
- Scalability.
- Message durability.
- Security.

### Step 2: Capacity estimation

Assume:

- 10 million daily active users.
- 50 messages per user per day.
- Average message payload of 1 KB.

Daily messages:

$$
10M \times 50 = 500M
$$

Daily raw message payload:

$$
500M \times 1KB = 500GB
$$

This is a simplified estimate. Actual storage must account for metadata, indexes, replication, attachments, retention, and compression.

### Step 3: High-level architecture

Components:

- Android application.
- API Gateway.
- WebSocket Gateway.
- Authentication Service.
- Message Service.
- Message Broker.
- Redis Presence Service.
- Message Database.
- FCM.

### Step 4: Message flow

1. Sender connects through WebSocket.
2. Sender sends message.
3. Backend validates identity and permissions.
4. Message is persisted.
5. Message event is distributed.
6. Recipient receives the message.
7. Recipient sends acknowledgement.
8. Message delivery status is updated.

### Step 5: Failure handling

Discuss:

- WebSocket disconnection.
- Duplicate messages.
- Message ordering.
- Database failure.
- Message broker failure.
- Offline synchronization.
- Retry and circuit breaker.
- Multi-device synchronization.

### Step 6: Android architecture

Use:

- MVVM or MVI.
- Clean Architecture.
- Repository Pattern.
- Room.
- Kotlin Coroutines.
- Flow.
- OkHttp WebSocket.
- FCM.
- WorkManager.

### Step 7: Security and observability

Discuss:

- TLS.
- Authentication and authorization.
- Secure token storage.
- Message encryption requirements.
- Crash monitoring.
- API metrics.
- Distributed tracing.
- Alerting.

Interview tip: Don't immediately start drawing microservices. Start with requirements, constraints, and trade-offs. The architecture should follow from the problem.

# Part 11 – Day 28 Quick Revision Cheat Sheet

## Microservices

- Monolith vs Microservices
- API Gateway
- Service Discovery
- Independent Deployment
  ## Distributed Patterns
- Retry
- Timeout
- Circuit Breaker
- Bulkhead
- Saga
- CQRS
- Event Sourcing
  ## Database and Storage
- SQL vs NoSQL
- B-Tree Index
- Replication
- Distributed Locks
- Eventual Consistency
  ## Search and Notifications
- Inverted Index
- Search Indexing
- Elasticsearch
- FCM
- Message Queue
- Dead-Letter Queue
  ## Real-Time Systems
- Polling
- Long Polling
- SSE
- WebSockets
- Heartbeat
- Presence
- Reconnection
  ## Security and Observability
- HTTPS/TLS
- Authentication/Authorization
- Android Keystore
- Logs
- Metrics
- Tracing
- SLI/SLO/SLA

# 12. Learning Checklist

Day 28 Progress

# 0/20

[ ]Revise monolithic vs microservices architecture

[ ]Understand API Gateway and Service Discovery

[ ]Understand Circuit Breaker states

[ ]Understand Retry, Timeout and Bulkhead

[ ]Understand Saga and compensating transactions

[ ]Understand CQRS

[ ]Understand Event Sourcing

[ ]Revise SQL vs NoSQL

[ ]Understand B-Tree indexing

[ ]Understand database replication and eventual consistency

[ ]Understand distributed locks

[ ]Understand search architecture and inverted indexes

[ ]Understand notification architecture and FCM

[ ]Revise Polling, Long Polling, SSE and WebSockets

[ ]Understand WebSocket scaling

[ ]Revise TLS, authentication and Android Keystore

[ ]Revise Logs, Metrics and Distributed Tracing

[ ]Understand SLI, SLO and SLA

[ ]Design a scalable e-commerce application

[ ]Practice the WhatsApp system design interview

 Reset checklist

# 13. Week 4 Completion – Self Assessment

Before moving to Android System Design, check whether you can independently explain these topics.

| Topic | Expected ability |
| --- | --- |
| Microservices | Draw architecture and explain service communication |
| Distributed patterns | Explain when to use Circuit Breaker, Saga and Retry |
| Database internals | Explain indexing, replication and transactions |
| Search | Design product search architecture |
| Notifications | Explain FCM and asynchronous delivery |
| Real-time systems | Design WebSocket-based chat |
| Security | Explain TLS, tokens and Android Keystore |
| Observability | Debug a production issue using logs, metrics and traces |

My recommendation: Don't try to memorize every technology. Focus on why a particular architectural decision is made, what problem it solves, and what trade-offs it introduces.

Week 4 Completed

# 28 / 49 Days

You have completed the advanced system design fundamentals portion of your roadmap.

Next: Week 5 – Android System Design

## Day 29: Android Application Architecture

MVC, MVP, MVVM, MVI, Clean Architecture, SOLID, Dependency Injection, Repository Pattern, Use Cases and Modularization.

The next phase is particularly important: we will connect backend system design knowledge with real Android application architecture, which is central to senior Android interviews.
