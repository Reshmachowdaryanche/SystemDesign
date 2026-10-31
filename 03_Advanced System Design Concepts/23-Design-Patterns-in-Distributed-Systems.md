# Week 4 – Day 23: Design Patterns in Distributed Systems

File name: `23-Design-Patterns-in-Distributed-Systems.md`

Topic: Advanced Architecture and Distributed Systems Level: Beginner to Intermediate Target: Senior Android Developer – 7 Years Experience

Today's goal: Understand how distributed systems handle failures, slow services, communication problems, and data consistency using common architectural design patterns.

Today's notes start from absolute basics. You don't need previous backend development experience.

# 1. First, Understand Why We Need Distributed System Design Patterns

Yesterday, in Day 22, we learned Microservices Architecture.

Let's continue with our Notes application.

Imagine our application has:

- User Service.
- Notes Service.
- Notification Service.
- Payment Service.
- API Gateway.
- PostgreSQL Database.
- Message Broker.

Architecture:

text

```text
                ANDROID APP
                     |
                     v
                API GATEWAY
                     |
          +----------+----------+
          |          |          |
          v          v          v
      USER SERVICE NOTES SERVICE PAYMENT SERVICE
          |          |          |
          v          v          v
        USER DB    NOTES DB   PAYMENT DB
                     |
                     v
              MESSAGE BROKER
                     |
                     v
            NOTIFICATION SERVICE
```

Everything looks good.

But imagine these situations:

1. Notification Service becomes unavailable.
2. Payment Service responds very slowly.
3. Notes Service repeatedly fails.
4. A request is accidentally processed twice.
5. Multiple services need to update related business data.
6. One service receives more traffic than it can handle.

How should our system handle these situations?

This is where distributed system design patterns become useful.

## 1.1 What Is a Design Pattern?

A design pattern is a reusable solution to a commonly occurring software design problem.

For example:

- Retry Pattern helps recover from temporary failures.
- Circuit Breaker prevents repeated calls to an unhealthy service.
- Timeout Pattern prevents waiting indefinitely.
- Bulkhead Pattern isolates failures.
- Saga Pattern coordinates business transactions across services.
- CQRS separates read and write operations.
- Event Sourcing stores changes as a sequence of events.

These patterns help us build more reliable applications.

# 2. Circuit Breaker Pattern

Important interview conceptFirst, understand a real-world example.

Imagine you are calling someone.

You call once. No answer.

You call again. No answer.

You call ten times. Still no answer.

Would you continue calling repeatedly?

Probably not.

You would stop trying for some time.

The same problem happens in microservices.

## 2.1 What Is the Circuit Breaker Pattern?

The Circuit Breaker Pattern prevents an application from repeatedly calling a service that is failing or unavailable.

It temporarily blocks requests to the unhealthy service and allows it to recover.

## 2.2 Without Circuit Breaker

Imagine Notes Service calls Notification Service.

Notification Service is down.

text

```text
       NOTES SERVICE
             |
             v
      NOTIFICATION SERVICE
             |
             X
           FAILED
```

Notes Service continues making requests.

This can cause:

- Repeated failures.
- Increased network traffic.
- Wasted resources.
- Increased response time.
- Failure propagation.

## 2.3 With Circuit Breaker

Now introduce a Circuit Breaker.

text

```text
       NOTES SERVICE
             |
             v
        CIRCUIT BREAKER
             |
             v
      NOTIFICATION SERVICE
```

When failures cross a configured threshold, the circuit opens.

New calls are rejected or handled through a fallback instead of being sent to the failing service.

## 2.4 Three States of Circuit Breaker

There are three main states.

### 1. Closed State

Normal operationRequests are allowed to reach the service.

text

```text
NOTES SERVICE
      |
      v
CIRCUIT BREAKER
   CLOSED
      |
      v
NOTIFICATION SERVICE
```

The circuit is closed when the service is operating normally.

### 2. Open State

Requests blockedIf failures exceed the configured threshold, the circuit opens.

text

```text
NOTES SERVICE
      |
      v
CIRCUIT BREAKER
     OPEN
      |
      X
NOTIFICATION SERVICE
```

New calls are blocked for a configured period.

### 3. Half-Open State

Recovery testingAfter the waiting period, the circuit allows a limited number of test requests.

text

```text
NOTES SERVICE
      |
      v
CIRCUIT BREAKER
   HALF-OPEN
      |
      v
NOTIFICATION SERVICE
```

If test requests succeed, the circuit closes.

If they fail, the circuit opens again.

## 2.5 Real-World Example

Suppose Notification Service has failed five consecutive times.

Our configuration might be:

- Failure threshold: 5.
- Open duration: 30 seconds.
- Half-open test requests: 2.

These are illustrative configuration values.

Flow:

1. Notes Service calls Notification Service.
2. Five calls fail.
3. Circuit Breaker opens.
4. New requests are blocked for 30 seconds.
5. Half-open state begins.
6. Two test requests are allowed.
7. If successful, circuit closes.

## 2.6 Advantages

- Prevents repeated calls to failing services.
- Reduces unnecessary network traffic.
- Helps prevent cascading failures.
- Gives unhealthy services time to recover.

## 2.7 Important Interview Point

Circuit Breaker and Retry are different.

- Retry attempts the operation again.
- Circuit Breaker stops attempts temporarily when failures become excessive.

Using aggressive retries without a Circuit Breaker can make an unhealthy service even worse.

# 3. Retry Pattern

Imagine Android sends a request to the backend.

But the network temporarily fails.

Should the application immediately show an error?

Not always.

Some failures are temporary.

Retry Pattern allows an operation to be attempted again after a failure.

## 3.1 Definition

Retry Pattern is a technique in which a failed operation is attempted again according to a defined retry policy.

## 3.2 Without Retry

text

```text
ANDROID APP
     |
     v
BACKEND SERVER
     |
     X
  FAILURE
     |
     v
SHOW ERROR
```

The operation fails once and is abandoned.

## 3.3 With Retry

text

```text
ANDROID APP
     |
     v
BACKEND SERVER
     |
     X
  FAILURE
     |
     v
   RETRY
     |
     v
BACKEND SERVER
     |
     v
  SUCCESS
```

## 3.4 Retry Strategies

### 1. Immediate Retry

Retry immediately after a failure.

text

```text
Request 1 -> Failed
Request 2 -> Failed
Request 3 -> Success
```

Useful for certain brief transient failures, but repeated immediate retries can overload a failing service.

### 2. Fixed Interval Retry

Wait for a fixed amount of time between retries.

Example:

text

```text
Attempt 1 -> Failed
Wait 2 seconds

Attempt 2 -> Failed
Wait 2 seconds

Attempt 3 -> Success
```

### 3. Exponential Backoff

The waiting time increases after each failed attempt.

Example:

| Attempt | Waiting Time |
| --- | --- |
| 1 | 1 second |
| 2 | 2 seconds |
| 3 | 4 seconds |
| 4 | 8 seconds |

This reduces unnecessary repeated requests.

### 4. Exponential Backoff with Jitter

Jitter adds a random variation to the retry delay.

Why?

Imagine thousands of clients retry after exactly two seconds.

All requests may reach the backend simultaneously.

This creates another traffic spike.

Jitter helps spread requests over time.

## 3.5 When Should We Use Retry?

Suitable examples:

- Temporary network failures.
- Temporary service unavailability.
- Transient database connection failures.
- Temporary rate limiting, respecting server-provided retry guidance.

Avoid blindly retrying:

- Invalid requests.
- Authentication failures.
- Validation errors.
- Permanent business errors.

Important: Retrying a non-idempotent operation can create duplicate effects.

For example, retrying a payment request could charge the customer twice unless the backend supports idempotency.

# 4. Timeout Pattern

Imagine Notes Service calls User Service.

User Service is taking 60 seconds to respond.

Should Notes Service wait forever?

No.

We need a timeout.

## 4.1 Definition

Timeout Pattern defines the maximum amount of time a service or client should wait for an operation to complete.

## 4.2 Without Timeout

text

```text
NOTES SERVICE
      |
      v
  USER SERVICE
      |
      |
   WAITING...
      |
      |
   WAITING...
```

Resources remain occupied while waiting.

This can affect other requests.

## 4.3 With Timeout

text

```text
NOTES SERVICE
      |
      v
  USER SERVICE
      |
      v
TIMEOUT: 3 SECONDS
      |
      v
STOP WAITING
```

After the configured duration, the caller stops waiting and handles the timeout.

## 4.4 Types of Timeouts

### Connection Timeout

Maximum time allowed to establish a network connection.

### Read Timeout

Maximum time allowed while waiting for data from the connected server.

### Request Timeout

Maximum total time allowed for an operation, depending on the client or framework's definition.

## 4.5 Example

Suppose:

- Connection timeout: 3 seconds.
- Read timeout: 10 seconds.
- Overall request deadline: 15 seconds.

These are illustrative values, not universal recommendations.

## 4.6 Why Are Timeouts Important?

- Prevent indefinite waiting.
- Release resources.
- Improve responsiveness.
- Reduce cascading failures.

Interview tip: A timeout does not necessarily mean the backend failed to process the request. The backend may have completed the operation even though the response arrived too late.

# 5. Bulkhead Pattern

Let's understand this using a ship.

Ships have separate compartments.

If one compartment fills with water, the entire ship does not necessarily sink.

This is the idea behind the Bulkhead Pattern.

## 5.1 Definition

Bulkhead Pattern isolates resources or workloads so that failure or resource exhaustion in one component does not affect the entire system.

## 5.2 Without Bulkhead

Imagine Notes Service communicates with:

- User Service.
- Notification Service.
- File Service.

All outgoing requests share the same connection pool.

Notification Service becomes extremely slow.

It occupies all available connections.

Now calls to User Service may also fail because no connections are available.

text

```text
             NOTES SERVICE
                   |
            SHARED RESOURCES
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
      USER    NOTIFICATION    FILE
    SERVICE     SERVICE     SERVICE

Notification overload affects other calls.
```

## 5.3 With Bulkhead

Allocate separate resources for different dependencies.

text

```text
             NOTES SERVICE
                   |
        +----------+----------+
        |          |          |
        v          v          v
     POOL A     POOL B      POOL C
        |          |          |
        v          v          v
      USER    NOTIFICATION   FILE
    SERVICE     SERVICE    SERVICE
```

If Notification Service becomes slow, it cannot consume all resources reserved for other services.

## 5.4 Advantages

- Failure isolation.
- Better resource management.
- Limits the impact of overloaded dependencies.
- Improves system resilience.

## 5.5 Circuit Breaker vs Bulkhead

| Circuit Breaker | Bulkhead |
| --- | --- |
| Stops calls to failing services | Separates resources |
| Based on failure conditions | Based on resource isolation |
| Uses closed, open and half-open states | Uses separate resource pools or limits |
| Prevents repeated unsuccessful calls | Prevents one workload consuming everything |

# 6. Saga Pattern

Important system design conceptNow let's understand a more complex problem.

Imagine an e-commerce application.

A user places an order.

The system must:

1. Create an order.
2. Reserve inventory.
3. Process payment.
4. Arrange shipment.

These operations may involve different microservices and databases.

text

```text
       ORDER SERVICE
             |
             v
     INVENTORY SERVICE
             |
             v
      PAYMENT SERVICE
             |
             v
     SHIPPING SERVICE
```

What happens if payment fails after inventory has already been reserved?

We cannot simply roll back all database operations using one ordinary database transaction because these operations belong to different services.

This is where Saga Pattern helps.

## 6.1 Definition

Saga Pattern is a distributed transaction management approach that coordinates a sequence of local transactions across multiple services.

If a step fails, the system may execute compensating transactions to undo the business effects of previously completed steps.

## 6.2 Example: Order Creation

Normal flow:

text

```text
1. Create Order
       |
       v
2. Reserve Inventory
       |
       v
3. Process Payment
       |
       v
4. Arrange Shipment
       |
       v
   ORDER SUCCESS
```

Now imagine payment fails.

text

```text
1. Create Order       SUCCESS
       |
       v
2. Reserve Inventory  SUCCESS
       |
       v
3. Process Payment    FAILED
       |
       v
4. Release Inventory
       |
       v
5. Cancel Order
```

The compensating operations reverse the business effects of earlier successful steps.

## 6.3 What Is a Compensating Transaction?

A compensating transaction is an operation that counteracts the business effect of an earlier successful operation.

Examples:

| Original Operation | Compensating Operation |
| --- | --- |
| Reserve inventory | Release inventory |
| Create order | Cancel order |
| Book hotel | Cancel booking |
| Debit account | Refund or credit account, subject to business rules |

Important: Compensation is not always an exact database rollback. It is a new business operation that attempts to restore an acceptable state.

## 6.4 Types of Saga

### 1. Choreography

Each service reacts to events and publishes its own events.

text

```text
ORDER SERVICE
      |
      v
OrderCreated Event
      |
      v
INVENTORY SERVICE
      |
      v
InventoryReserved Event
      |
      v
PAYMENT SERVICE
      |
      v
PaymentCompleted Event
```

There is no single central coordinator controlling every step.

### 2. Orchestration

A central Saga orchestrator coordinates the sequence of operations.

text

```text
          SAGA ORCHESTRATOR
                  |
       +----------+----------+
       |          |          |
       v          v          v
     ORDER    INVENTORY    PAYMENT
    SERVICE    SERVICE     SERVICE
```

The orchestrator tells each service what to do and handles failures and compensating actions.

## 6.5 Choreography vs Orchestration

| Choreography | Orchestration |
| --- | --- |
| Services react to events | Central coordinator manages flow |
| No central controller | Has a central orchestrator |
| Services are event-driven | Workflow is centrally coordinated |
| Can become difficult to trace | Workflow is easier to visualize centrally |

## 6.6 When Do We Use Saga?

- E-commerce order processing.
- Hotel booking.
- Travel reservations.
- Payment workflows.
- Multi-service business transactions.

# 7. CQRS – Command Query Responsibility Segregation

Now imagine our Notes application has millions of users.

Users frequently read notes.

But note creation and updates happen less frequently.

Read operations and write operations have different workload requirements.

Can we optimize them separately?

Yes. CQRS helps with this.

## 7.1 Definition

CQRS stands for Command Query Responsibility Segregation.

It separates operations that change data from operations that retrieve data.

Two important concepts:

- Command: Changes application state.
- Query: Reads application state without changing it.

## 7.2 Command

Examples:

text

```text
CREATE NOTE
UPDATE NOTE
DELETE NOTE
```

These operations change data.

## 7.3 Query

Examples:

text

```text
GET NOTES
GET NOTE BY ID
SEARCH NOTES
```

These operations retrieve data.

## 7.4 Traditional Architecture

In a traditional design, the same model and data access approach may handle both reads and writes.

text

```text
           ANDROID APP
                 |
                 v
             NOTES API
                 |
                 v
             NOTES DB
```

## 7.5 CQRS Architecture

CQRS can separate the write and read paths.

text

```text
               ANDROID APP
                     |
                     v
                NOTES API
                     |
             +-------+-------+
             |               |
             v               v
        COMMAND SIDE     QUERY SIDE
             |               |
             v               v
         WRITE DB         READ DB
```

The write side handles commands.

The read side handles queries.

The read model may be optimized for retrieving information efficiently.

## 7.6 Example

User creates a note.

1. Android sends a create-note command.
2. Command handler validates the request.
3. Write database stores the note.
4. An event is published.
5. Read model is updated.
6. A subsequent query retrieves the note from the read model.

Important:

The read model may update asynchronously.

This means a newly created note might not appear immediately in the read database.

This is an example of eventual consistency.

## 7.7 Advantages

- Read and write workloads can be optimized separately.
- Read models can be designed for specific queries.
- Complex query requirements become easier to manage.
- Read and write sides can scale independently.

## 7.8 Disadvantages

- More architectural complexity.
- Additional data synchronization.
- Eventual consistency may occur.
- More infrastructure and monitoring.

Important: CQRS does not always require two separate physical databases. It can begin with logically separated command and query models using the same database.

# 8. Event Sourcing

Let's understand another important pattern.

Imagine a user edits a note five times.

Traditional database approach:

The database stores the latest note.

text

```text
NOTE:
Title: Project Meeting
Content: Final discussion notes
```

But what if we want to know the complete history of changes?

Event Sourcing can help.

## 8.1 Definition

Event Sourcing is an architectural pattern in which application state is stored as a sequence of events rather than only storing the latest state.

## 8.2 Traditional Database vs Event Sourcing

Traditional approach:

text

```text
NOTES TABLE

ID    TITLE
1     Project Meeting
```

Event Sourcing approach:

text

```text
EVENT STORE

1. NoteCreated
2. NoteTitleUpdated
3. NoteContentUpdated
4. NoteShared
5. NoteArchived
```

The current state can be reconstructed by applying the events in order.

## 8.3 How Does It Work?

Imagine creating a note.

Event 1:

json

```json
{
  "event": "NoteCreated",
  "noteId": 101,
  "title": "Meeting"
}
```

Event 2:

json

```json
{
  "event": "NoteTitleUpdated",
  "noteId": 101,
  "title": "Project Meeting"
}
```

Event 3:

json

```json
{
  "event": "NoteArchived",
  "noteId": 101
}
```

The application stores these events in sequence.

The current state is calculated by replaying the relevant events.

## 8.4 Advantages

- Complete history of changes.
- Auditability.
- Ability to reconstruct previous states.
- Useful for event-driven architectures.
- Can support multiple read models.

## 8.5 Disadvantages

- More complex implementation.
- Event schema evolution requires care.
- Event storage can grow significantly.
- Rebuilding state can be expensive without snapshots.
- Eventual consistency may be involved when projections update asynchronously.

## 8.6 What Is a Snapshot?

A snapshot stores the calculated state at a particular point in time.

Instead of replaying every event from the beginning, the system can:

1. Load the latest snapshot.
2. Replay only events created after that snapshot.

This improves reconstruction performance.

# 9. CQRS vs Event Sourcing

These concepts are often confused.

| CQRS | Event Sourcing |
| --- | --- |
| Separates read and write responsibilities | Stores state changes as events |
| Focuses on command and query models | Focuses on event history |
| Can use a traditional database | Uses an event store as the source of truth |
| Does not require event storage | Requires event-based state storage |
| Can be used independently | Can be combined with CQRS |

Important: CQRS and Event Sourcing are separate patterns. They can be used together, but neither requires the other.

# 10. How These Patterns Work Together

Let's combine the concepts using our Notes application.

Suppose a user creates a note and shares it with another user.

Our system might use:

- API Gateway.
- Notes Service.
- User Service.
- Notification Service.
- Message Broker.
- Databases.

Architecture:

text

```text
                  ANDROID APP
                       |
                       v
                  API GATEWAY
                       |
                       v
                  NOTES SERVICE
                       |
                +------+------+
                |             |
                v             v
           COMMAND SIDE   QUERY SIDE
                |             |
                v             v
             WRITE DB      READ DB
                |
                v
            EVENT STORE
                |
                v
           MESSAGE BROKER
                |
                v
         NOTIFICATION SERVICE
```

Different patterns help solve different problems:

| Pattern | Problem Solved |
| --- | --- |
| Circuit Breaker | Repeated calls to failing services |
| Retry | Temporary operation failures |
| Timeout | Waiting too long |
| Bulkhead | Shared resource exhaustion |
| Saga | Multi-service transactions |
| CQRS | Separating read and write workloads |
| Event Sourcing | Maintaining a sequence of state changes |

These patterns can be combined when the system requirements justify them.

# 11. Real-World Example – Notes Application

Let's design a slightly advanced Notes application.

Requirements:

- Users can create notes.
- Users can edit notes.
- Users can share notes.
- Users receive notifications.
- Every important change should be auditable.

Possible architecture:

text

```text
                  ANDROID USERS
                        |
                        v
                   API GATEWAY
                        |
                        v
                   NOTES SERVICE
                        |
                +-------+-------+
                |               |
                v               v
          COMMAND HANDLER   QUERY HANDLER
                |               |
                v               v
             WRITE DB        READ MODEL
                |
                v
            EVENT STORE
                |
                v
            MESSAGE BROKER
                |
        +-------+--------+
        |                |
        v                v
  NOTIFICATION       AUDIT SERVICE
    SERVICE
        |
        v
   PUSH NOTIFICATION
```

Now imagine the Notification Service is unavailable.

Circuit Breaker prevents repeated calls to the unhealthy service.

The notification event can remain available for later processing, depending on the broker's durability and delivery configuration.

If a temporary network issue occurs, Retry can be used with exponential backoff.

If the user shares a note and multiple services need to perform related operations, Saga can coordinate the workflow where appropriate.

CQRS can optimize note reading separately from note editing.

Event Sourcing can preserve the history of note changes.

This is how multiple design patterns solve different problems inside one architecture.

# 12. Android Developer Perspective

As a Senior Android Developer, you should understand how these backend patterns affect mobile applications.

## 12.1 Retry Pattern in Retrofit

A network request may fail temporarily.

Android can retry selected requests using a controlled retry policy.

Example:

kotlin

```kotlin
suspend fun <T> retryRequest(
    maxAttempts: Int = 3,
    block: suspend () -> T
): T {
    var attempt = 0

    while (true) {
        try {
            return block()
        } catch (e: IOException) {
            attempt++

            if (attempt >= maxAttempts) {
                throw e
            }

            delay(1000L * attempt)
        }
    }
}
```

This is a simplified illustration.

Production implementations should also consider exponential backoff, jitter, cancellation, HTTP response codes and an overall deadline.

## 12.2 Timeout in Retrofit

Retrofit uses OkHttp for networking.

Example:

kotlin

```kotlin
val client = OkHttpClient.Builder()
    .connectTimeout(10, TimeUnit.SECONDS)
    .readTimeout(30, TimeUnit.SECONDS)
    .writeTimeout(30, TimeUnit.SECONDS)
    .build()
```

These are example values. Choose timeouts according to the API's expected behavior.

## 12.3 Circuit Breaker

Android applications can also implement circuit-breaker-like protection for repeated calls to unreliable remote dependencies.

However, circuit breakers are commonly implemented at backend service boundaries.

## 12.4 Idempotency

Suppose Android sends a create-payment request.

The request succeeds on the backend, but the response is lost.

Android retries.

The backend should use an idempotency key to prevent duplicate payment processing.

## 12.5 Offline-First Architecture

If the backend is unavailable:

- Display locally stored notes.
- Allow appropriate local changes.
- Queue pending synchronization.
- Retry synchronization later.

This connects with our Day 30 topic in the Android System Design phase.

# 13. Common Mistakes to Avoid

1. Thinking Retry and Circuit Breaker are the same.
2. Retrying every failed request indefinitely.
3. Forgetting exponential backoff and jitter.
4. Assuming a timeout means the backend did not execute the request.
5. Confusing Bulkhead with Circuit Breaker.
6. Thinking Saga provides one global ACID transaction.
7. Assuming CQRS always requires separate physical databases.
8. Thinking Event Sourcing is simply normal database logging.
9. Assuming CQRS and Event Sourcing are the same pattern.
10. Using advanced architecture patterns without a real requirement.

# 14. Important Interview Questions and Answers

### Q1. What is the Circuit Breaker Pattern?

It prevents repeated requests to a failing service by temporarily blocking calls when failures cross a configured threshold.

### Q2. What are the three Circuit Breaker states?

Closed, Open and Half-Open.

### Q3. What is the Retry Pattern?

It attempts a failed operation again according to a defined retry policy.

### Q4. What is Exponential Backoff?

It increases the waiting time between consecutive retry attempts.

### Q5. Why do we need Jitter?

Jitter spreads retry requests over time and helps prevent many clients from retrying simultaneously.

### Q6. What is the Timeout Pattern?

It defines the maximum time an operation is allowed to wait for completion.

### Q7. What is the Bulkhead Pattern?

It isolates resources so that one failing or overloaded component cannot consume all resources and affect the entire system.

### Q8. What is the difference between Circuit Breaker and Bulkhead?

Circuit Breaker blocks calls to unhealthy dependencies. Bulkhead separates resources to limit the impact of failures.

### Q9. What is the Saga Pattern?

Saga coordinates a sequence of local transactions across multiple services using compensating transactions when necessary.

### Q10. What is a Compensating Transaction?

An operation that counteracts the business effect of an earlier successful operation.

### Q11. What are the two types of Saga?

Choreography and Orchestration.

### Q12. What is CQRS?

CQRS separates operations that change application state from operations that retrieve application state.

### Q13. What is the difference between Command and Query?

A Command changes data. A Query retrieves data without changing application state.

### Q14. What is Event Sourcing?

Event Sourcing stores application state changes as an ordered sequence of events.

### Q15. What is a Snapshot in Event Sourcing?

A snapshot stores the calculated state at a particular point to reduce the number of events that must be replayed.

### Q16. What is the difference between CQRS and Event Sourcing?

CQRS separates reads and writes, while Event Sourcing stores state changes as events. They can be used independently or together.

### Q17. Can CQRS be implemented with one database?

Yes. CQRS requires logical separation of command and query responsibilities, not necessarily separate physical databases.

### Q18. Why is Idempotency important when using Retry?

It prevents repeated requests from producing unintended duplicate business effects.

### Q19. Which pattern helps prevent cascading failures?

Circuit Breaker and Bulkhead are commonly used to reduce cascading failure risks.

### Q20. Which pattern would you use for an e-commerce order involving inventory, payment and shipping?

Saga Pattern is a suitable approach for coordinating the distributed business workflow.

# 15. Practical Interview Exercise

Imagine you are designing a Notes application.

The architecture contains:

- Notes Service.
- User Service.
- Notification Service.
- Payment Service.
- PostgreSQL.
- Redis.
- Kafka.

Answer these questions:

1. What happens if Notification Service becomes unavailable?
2. How would you prevent Notes Service from repeatedly calling it?
3. How would you retry temporary network failures?
4. Why should every service have appropriate timeouts?
5. How would you isolate Notification Service resource usage?
6. How would you coordinate an order involving payment and inventory?
7. How would you separate read and write operations?
8. How would you maintain a complete history of note changes?
9. What is the difference between CQRS and Event Sourcing?
10. How would Android handle a temporary backend failure?

Try drawing the architecture and explaining the trade-offs.

# 16. Day 23 – Quick Revision

| Concept | Simple Meaning |
| --- | --- |
| Design Pattern | Reusable solution to a common problem |
| Circuit Breaker | Temporarily blocks calls to failing services |
| Closed State | Requests are allowed |
| Open State | Requests are blocked |
| Half-Open State | Limited recovery testing |
| Retry | Attempts a failed operation again |
| Exponential Backoff | Increases retry waiting time |
| Jitter | Adds randomness to retry delays |
| Timeout | Limits waiting time |
| Bulkhead | Isolates resources |
| Saga | Coordinates distributed transactions |
| Compensating Transaction | Counteracts a previous business operation |
| Choreography | Services coordinate through events |
| Orchestration | Central coordinator manages workflow |
| CQRS | Separates reads and writes |
| Command | Changes application state |
| Query | Reads application state |
| Event Sourcing | Stores changes as events |
| Snapshot | Stores calculated state at a point in time |

# 17. Final Learning Checklist

- [ ]Understand why distributed system design patterns are needed.
- [ ]Understand Circuit Breaker.
- [ ]Understand Closed, Open and Half-Open states.
- [ ]Understand Retry Pattern.
- [ ]Understand Fixed Interval Retry.
- [ ]Understand Exponential Backoff.
- [ ]Understand Jitter.
- [ ]Understand Timeout Pattern.
- [ ]Understand Connection and Read Timeouts.
- [ ]Understand Bulkhead Pattern.
- [ ]Understand Circuit Breaker vs Bulkhead.
- [ ]Understand Saga Pattern.
- [ ]Understand Compensating Transactions.
- [ ]Understand Choreography.
- [ ]Understand Orchestration.
- [ ]Understand CQRS.
- [ ]Understand Command vs Query.
- [ ]Understand Event Sourcing.
- [ ]Understand Snapshots.
- [ ]Understand CQRS vs Event Sourcing.
- [ ]Explain how these patterns work in a Notes application.
- [ ]Connect the patterns to Android networking and offline behavior.

# 18. What Is Next?

## Week 4 – Day 24: Storage and Data Management

Tomorrow we will learn:

1. SQL Database Internals – Basics.
2. What is a database index?
3. Clustered vs Non-Clustered Index.
4. B-Tree fundamentals.
5. Database Transactions.
6. ACID Properties.
7. Database Normalization.
8. Database Denormalization.
9. Blob Storage.
10. Object Storage.
11. CDN.
12. When to use different storage solutions.
13. Real-world Notes application architecture.
14. Interview questions and answers.
15. Practical system design exercise.

Day 23 Complete – Design Patterns in Distributed Systems.

## Today's five most important concepts

1. Circuit Breaker: Stops repeatedly calling an unhealthy service.
2. Retry: Attempts temporary failures again with controlled delays.
3. Bulkhead: Prevents one overloaded component from consuming all resources.
4. Saga: Coordinates multi-service business transactions.
5. CQRS and Event Sourcing: Help organize read/write operations and maintain event history.

Remember these distinctions:

- Retry = Try again.
- Circuit Breaker = Stop trying temporarily.
- Timeout = Don't wait forever.
- Bulkhead = Isolate resources.
- Saga = Coordinate distributed transactions.
- CQRS = Separate reads and writes.
- Event Sourcing = Store changes as events.

Roadmap tracking: Week 4 – Day 22 Microservices Architecture: complete. Week 4 – Day 23 Design Patterns in Distributed Systems: complete. Next is Day 24 – Storage and Data Management.
