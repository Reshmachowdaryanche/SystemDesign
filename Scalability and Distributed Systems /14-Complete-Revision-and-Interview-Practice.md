# Week 2 – Day 14: Complete Revision and System Design Interview Practice

Week 2Day 14Revision day

File name: `Week-2-Day-14-Complete-Revision-and-Interview-Practice.md`

Today's goal: Revise everything learned during Week 2, connect the concepts, and understand how to explain them in a system design interview.

Writing

# Week 2 – Day 14: Complete Revision and System Design Interview Practice

**Topic:** Database, Distributed Systems, Reliability and API Architecture\
**Level:** Beginner to Intermediate\
**Target:** Android Developer – 7 Years Experience

---

# 1\. Week 2 – What Did We Learn?

During Week 2, we learned how backend systems store data, handle large traffic, communicate between services, and remain reliable.

Here is the complete topic list.

| Day | Topic |
| --- | --- |
| Day 8 | Database Fundamentals |
| Day 9 | Database Indexing |
| Day 10 | Database Replication |
| Day 11 | Database Sharding |
| Day 12 | CAP Theorem, Message Queues and Event-Driven Architecture |
| Day 13 | Reliability, Idempotency, Rate Limiting and API Gateway |
| Day 14 | Complete Revision and Interview Practice |

Let's revise each topic from the beginning.

---

# 2\. Database Fundamentals

## 2.1 What Is a Database?

A database is a place where an application stores data so that it can be retrieved, updated and managed later.

Example: Notes application.

A user creates a note:

json

```json
{
  "id": 101,
  "title": "System Design",
  "description": "Learn database fundamentals",
  "userId": 10
}
```

The backend stores this information in a database.

When the user opens the application again, the backend retrieves the note.

## 2.2 SQL vs NoSQL

SQL databases store data mainly in structured tables.

Examples:

- PostgreSQL
- MySQL
- SQL Server

NoSQL databases support different data models, such as documents, key-value pairs, graphs and wide-column storage.

Examples:

- MongoDB
- Redis
- Cassandra

### SQL Example

Notes table:

| id | title | user\_id |
| --- | --- | --- |
| 101 | System Design | 10 |
| 102 | Android | 10 |
| 103 | Kotlin | 20 |

### NoSQL Example

json

```json
{
  "_id": 101,
  "title": "System Design",
  "userId": 10,
  "tags": ["backend", "interview"]
}
```

### SQL vs NoSQL

| SQL | NoSQL |
| --- | --- |
| Relational data model | Multiple data models |
| Tables and rows | Documents, key-value, etc. |
| Supports relationships | Relationship handling varies by database |
| Often uses predefined schemas | Often supports flexible schemas |
| Strong relational querying | Query capabilities depend on database |

**Interview question: When would you choose SQL?**

When the application has structured data, relationships, transactions and complex queries.

**When would you choose NoSQL?**

When the data model and access patterns benefit from a document, key-value, wide-column or graph database.

Important: SQL is not always better than NoSQL, and NoSQL is not always faster than SQL. Choose based on requirements.

---

# 3\. Database Indexing

## 3.1 What Is an Index?

An index is a data structure that helps a database find records without scanning the entire table.

Imagine a book with 1,000 pages.

Without an index, you may need to search page by page.

With an index, you can directly find the page containing a topic.

Databases use indexes in a similar way.

## 3.2 Example

Suppose we have one million notes.

We frequently execute:

sql

```sql
SELECT * FROM notes
WHERE user_id = 10;
```

Without an index, the database may need to examine many rows.

We can create an index:

sql

```sql
CREATE INDEX idx_notes_user_id
ON notes(user_id);
```

Now the database can use the index to locate matching rows more efficiently.

## 3.3 Advantages

- Faster data retrieval for suitable queries.
- Improves filtering.
- Helps sorting and searching.
- Can improve JOIN performance.

## 3.4 Disadvantages

- Requires additional storage.
- INSERT operations can become slower.
- UPDATE and DELETE may require index maintenance.
- Too many indexes increase maintenance overhead.

**Interview answer:**

An index improves query performance by allowing the database to locate records more efficiently, but it introduces additional storage and write overhead.

---

# 4\. Database Replication

## 4.1 What Is Replication?

Replication means maintaining copies of data across multiple database servers.

Why?

Imagine our Notes application has one database server.

One million users access it.

If that server fails, the entire application may become unavailable.

Replication helps improve availability and read scalability.

## 4.2 Architecture

text

```text
              APPLICATION
                   |
                   v
              PRIMARY DB
                   |
            +------+------+
            |             |
            v             v
         REPLICA 1      REPLICA 2
```

The primary database handles writes and sends changes to replicas.

Replicas can serve read requests.

## 4.3 Primary-Replica Architecture

Example:

- Create note → Primary database.
- Update note → Primary database.
- Fetch notes → Replica database, when appropriate.

## 4.4 Synchronous vs Asynchronous Replication

**Synchronous replication:**

The write operation waits for the required replica acknowledgement before being considered successful, depending on the database configuration.

Advantages:

- Stronger durability or consistency guarantees, depending on configuration.

Disadvantages:

- Can increase write latency.
- Replica failure or network issues can affect writes.

**Asynchronous replication:**

The primary acknowledges the write before all replicas have received the change.

Advantages:

- Lower write latency.
- Less dependency on immediate replica acknowledgement.

Disadvantages:

- Replicas may temporarily contain old data.
- Recent writes can be lost during certain failures, depending on durability configuration.

## 4.5 Replication Lag

Replication lag means a replica has not yet received the latest changes from the primary.

Example:

text

```text
User creates a note.

Primary DB:
Note exists.

Replica DB:
Note not yet available.
```

The user immediately fetches the notes and doesn't see the newly created note.

This is one reason why systems sometimes route a user's immediate read to the primary database.

---

# 5\. Database Sharding

## 5.1 What Is Sharding?

Sharding means splitting a large database into smaller independent databases called shards.

Replication creates copies.

Sharding divides data.

## 5.2 Example

Suppose our Notes application has 100 million users.

Instead of storing all users in one database, we divide them.

text

```text
             APPLICATION
                  |
             SHARD ROUTER
                  |
        +---------+---------+
        |         |         |
        v         v         v
     SHARD 1   SHARD 2   SHARD 3
     Users     Users     Users
     1-100     101-200   201-300
```

Each shard stores a subset of the data.

## 5.3 Sharding Strategies

**Range-based sharding**

Data is divided based on ranges.

Example:

text

```text
Shard 1: User IDs 1-1000
Shard 2: User IDs 1001-2000
Shard 3: User IDs 2001-3000
```

**Hash-based sharding**

A hash function determines the shard.

Example:

text

```text
shard = hash(userId) % numberOfShards
```

**Geographical sharding**

Data is divided based on geographical regions.

Example:

text

```text
India users -> India region
US users -> US region
Europe users -> Europe region
```

## 5.4 Advantages

- Distributes large datasets.
- Improves write scalability.
- Reduces the amount of data handled by an individual shard.
- Allows horizontal database scaling.

## 5.5 Disadvantages

- More complex architecture.
- Cross-shard queries are difficult.
- Rebalancing data can be challenging.
- Transactions across shards are more complicated.

**Interview question: What is the difference between replication and sharding?**

Replication creates copies of the same data across servers.

Sharding divides data across multiple servers.

---

# 6\. CAP Theorem

## 6.1 What Is CAP Theorem?

CAP theorem explains the trade-offs in a distributed data system when a network partition occurs.

CAP stands for:

- C – Consistency
- A – Availability
- P – Partition Tolerance

## 6.2 Consistency

Every successful read gets the latest committed value, according to the system's consistency guarantee.

Example:

User updates a note.

The next read should return the updated note.

## 6.3 Availability

Every request received by a non-failing node gets a response, even if that response may not contain the latest data.

## 6.4 Partition Tolerance

The system continues operating despite communication failures between parts of the distributed system.

## 6.5 CAP Trade-off

During a network partition, a distributed system must choose between:

- Consistency: Reject or delay some operations rather than return potentially stale data.
- Availability: Continue responding, potentially returning stale data.

text

```text
           CAP THEOREM

       CONSISTENCY
            /\
           /  \
          /    \
         /      \
        /________\
 AVAILABILITY   PARTITION
                TOLERANCE
```

Important interview point:

CAP does not simply mean choosing any two properties in all situations.

The important trade-off happens when a network partition occurs.

---

# 7\. Message Queues

## 7.1 What Is a Message Queue?

A Message Queue allows one service to send a message that another service can process later.

It helps services communicate asynchronously.

Example:

A user creates a note.

The Notes Service needs to:

- Save the note.
- Send a notification.
- Update analytics.

Should the Notes Service wait for all these operations?

Not necessarily.

It can publish an event to a message queue.

## 7.2 Architecture

text

```text
           NOTES SERVICE
                 |
                 v
            MESSAGE QUEUE
                 |
        +--------+--------+
        |        |        |
        v        v        v
    NOTIFICATION ANALYTICS SEARCH
      SERVICE    SERVICE SERVICE
```

## 7.3 Advantages

- Reduces direct dependency between services.
- Supports asynchronous processing.
- Helps absorb temporary traffic spikes.
- Allows independent scaling of consumers.
- Supports retries and buffering.

## 7.4 Examples

- Apache Kafka
- RabbitMQ
- Amazon SQS

---

# 8\. Event-Driven Architecture

## 8.1 What Is an Event?

An event represents something that has happened.

Examples:

text

```text
NoteCreated
NoteUpdated
UserRegistered
PaymentCompleted
```

An event-driven system communicates through events.

## 8.2 Example

A user creates a note.

The Notes Service publishes:

json

```json
{
  "eventType": "NoteCreated",
  "noteId": 101,
  "userId": 10
}
```

Different consumers can react to this event.

text

```text
             NOTES SERVICE
                   |
                   v
             NoteCreated
                   |
                   v
              EVENT QUEUE
                   |
         +---------+---------+
         |         |         |
         v         v         v
    Notification Analytics Search
      Service      Service  Indexer
```

## 8.3 Message Queue vs Event-Driven Architecture

Message Queue is a communication mechanism.

Event-Driven Architecture is a design approach in which components communicate through events.

They are related but not identical.

---

# 9\. ACID and BASE

## 9.1 What Is ACID?

ACID describes important transaction properties in relational database systems.

A – Atomicity

All operations in a transaction succeed, or the transaction is rolled back.

Example:

Money transfer:

- Debit Account A.
- Credit Account B.

Both should succeed together.

C – Consistency

A transaction moves the database from one valid state to another, maintaining defined constraints.

I – Isolation

Concurrent transactions should not interfere in ways that violate the configured isolation guarantees.

D – Durability

Once a transaction is committed, its changes survive failures according to the database's durability guarantees.

## 9.2 What Is BASE?

BASE describes a more availability-oriented approach commonly associated with distributed NoSQL systems.

- Basically Available.
- Soft State.
- Eventual Consistency.

### Basically Available

The system aims to remain responsive even when some components fail.

### Soft State

Data may change over time because of asynchronous updates.

### Eventual Consistency

If no new updates occur and replication continues successfully, replicas eventually converge.

## 9.3 ACID vs BASE

| ACID | BASE |
| --- | --- |
| Strong transaction guarantees | Availability-oriented distributed approach |
| Common in relational databases | Common in distributed data systems |
| Transaction consistency | Eventual consistency may be acceptable |
| Useful for financial transactions | Useful for some large-scale distributed workloads |

Important: This is a conceptual comparison, not a strict SQL-versus-NoSQL division. Modern databases can support different consistency and transaction models.

---

# 10\. Optimistic vs Pessimistic Locking

These techniques help manage concurrent updates.

## 10.1 Optimistic Locking

Optimistic locking assumes conflicts are relatively uncommon.

The application checks whether the data has changed before updating it.

Example:

A note has version 1.

User A and User B both read version 1.

User A updates it to version 2.

User B attempts to update using version 1.

The update is rejected because the version has changed.

Example SQL:

sql

```sql
UPDATE notes
SET title = 'Updated Title',
    version = version + 1
WHERE id = 101
AND version = 1;
```

If zero rows are updated, a conflict occurred.

## 10.2 Pessimistic Locking

Pessimistic locking assumes conflicts may happen.

It locks a record while a transaction is working with it.

Example:

sql

```sql
SELECT *
FROM notes
WHERE id = 101
FOR UPDATE;
```

Another transaction attempting to acquire a conflicting lock may need to wait.

## 10.3 Comparison

| Optimistic Locking | Pessimistic Locking |
| --- | --- |
| Assumes conflicts are rare | Assumes conflicts are possible |
| Checks version or condition | Acquires a lock |
| Less blocking | Can cause waiting |
| Good for low-conflict workloads | Useful for high-conflict workloads |

---

# 11\. Idempotency

## 11.1 What Is Idempotency?

An operation is idempotent if repeating it produces the same intended result as performing it once.

Example:

http

```http
PUT /api/notes/101
```

Updating the same note to the same content repeatedly should leave it in the same state.

## 11.2 Why Is It Important?

Imagine the Android application sends a create-note request.

The server successfully creates the note.

But the network connection fails before Android receives the response.

Android cannot know whether the request succeeded.

If it retries, a duplicate note may be created.

Idempotency helps prevent this.

## 11.3 Idempotency Key

Android sends a unique key:

http

```http
POST /api/notes
Idempotency-Key: 8f21-example-key
```

The backend stores the key and the result associated with it.

If the same request is repeated with the same key, the backend can return the previously stored result rather than creating another note.

## 11.4 Important HTTP Methods

| Method | Idempotent? |
| --- | --- |
| GET | Yes |
| PUT | Yes |
| DELETE | Yes, in terms of intended resource state |
| POST | Not inherently |
| PATCH | Depends on the operation |

Important:

HTTP method semantics and actual application implementation both matter.

---

# 12\. Retry and Circuit Breaker

## 12.1 What Is Retry?

Retry means attempting an operation again after a temporary failure.

Example:

text

```text
Android App
     |
     v
Backend Request
     |
     v
Temporary Network Failure
     |
     v
Wait
     |
     v
Retry Request
```

But retrying immediately can increase backend traffic.

## 12.2 Exponential Backoff

Exponential Backoff increases the waiting time between retry attempts.

Example:

text

```text
Attempt 1 -> Wait 1 second
Attempt 2 -> Wait 2 seconds
Attempt 3 -> Wait 4 seconds
Attempt 4 -> Wait 8 seconds
```

In real systems, use jitter to avoid many clients retrying simultaneously.

Also use a maximum retry count and an overall deadline.

## 12.3 What Is a Circuit Breaker?

A Circuit Breaker prevents repeated requests to a failing service.

Imagine Notes Service depends on Notification Service.

Notification Service is down.

Without a Circuit Breaker, Notes Service may repeatedly call the unavailable service.

This wastes resources.

A Circuit Breaker temporarily stops calls to the failing dependency.

### Three States

**Closed**

Requests are allowed normally.

**Open**

Requests are blocked temporarily because failures exceeded a threshold.

**Half-Open**

A limited number of test requests are allowed to check whether the dependency has recovered.

text

```text
       CLOSED
          |
     Failures exceed
       threshold
          |
          v
         OPEN
          |
      Wait period
          |
          v
      HALF-OPEN
          |
      +---+---+
      |       |
    Success  Failure
      |       |
      v       v
    CLOSED   OPEN
```

---

# 13\. Rate Limiting

## 13.1 What Is Rate Limiting?

Rate Limiting controls the number of requests a client can make during a specific time period.

Example:

text

```text
100 requests per minute
```

If the client exceeds the limit, the server can return:

http

```http
HTTP/1.1 429 Too Many Requests
```

## 13.2 Algorithms

| Algorithm | Working |
| --- | --- |
| Fixed Window | Counts requests in fixed time windows |
| Sliding Window Log | Stores request timestamps |
| Sliding Window Counter | Estimates requests in a sliding window |
| Token Bucket | Uses refillable tokens |
| Leaky Bucket | Processes queued requests at a controlled rate |

### Token Bucket

- Tokens refill over time.
- Each request consumes one token.
- Allows short bursts.

### Leaky Bucket

- Requests enter a queue.
- Requests are processed at a controlled rate.
- Smooths traffic.

---

# 14\. API Gateway

## 14.1 What Is an API Gateway?

An API Gateway is a single entry point through which client requests reach backend services.

Without a Gateway:

text

```text
ANDROID
   |
   +---- NOTES SERVICE
   |
   +---- USER SERVICE
   |
   +---- PAYMENT SERVICE
```

With a Gateway:

text

```text
ANDROID
   |
   v
API GATEWAY
   |
   +---- NOTES SERVICE
   |
   +---- USER SERVICE
   |
   +---- PAYMENT SERVICE
```

## 14.2 Responsibilities

- Request routing.
- Authentication.
- Rate limiting.
- Logging.
- Monitoring.
- Request validation.
- Response aggregation.

## 14.3 API Gateway vs Load Balancer

| API Gateway | Load Balancer |
| --- | --- |
| Manages API requests | Distributes traffic |
| Routes requests to services | Distributes requests across instances |
| Can handle authentication | Commonly performs health checks |
| Can apply rate limiting | Can improve availability |

---

# 15\. Connect Everything: Notes Application Architecture

Now let's combine all the concepts.

Imagine we are designing a Notes application.

Requirements:

1. Users can register and log in.
2. Users can create, update and delete notes.
3. Users can search notes.
4. Users can share notes.
5. Users can access notes from multiple devices.
6. Application should support millions of users.
7. Application should remain available during server failures.

## 15.1 High-Level Architecture

text

```text
                 ANDROID APP
                      |
                      v
                LOAD BALANCER
                      |
                      v
                 API GATEWAY
                      |
          +-----------+-----------+
          |           |           |
          v           v           v
      USER SERVICE NOTES SERVICE SEARCH SERVICE
          |           |           |
          v           v           v
       USER DB     NOTES DB    SEARCH INDEX
                      |
                      v
                 MESSAGE QUEUE
                      |
          +-----------+-----------+
          |                       |
          v                       v
   NOTIFICATION SERVICE      ANALYTICS SERVICE
          |
          v
         FCM
          |
          v
      ANDROID APP
```

## 15.2 Where Do We Use Each Concept?

| Concept | Usage in Notes Application |
| --- | --- |
| SQL Database | Store users and notes |
| Database Index | Quickly find notes by user ID |
| Replication | Improve read scalability and availability |
| Sharding | Divide very large datasets |
| CAP | Understand consistency and availability trade-offs |
| Message Queue | Process notifications asynchronously |
| Event-Driven Architecture | React to NoteCreated events |
| ACID | Maintain transaction correctness |
| Optimistic Locking | Prevent overwriting newer note versions |
| Idempotency | Prevent duplicate note creation during retries |
| Retry | Handle temporary network failures |
| Circuit Breaker | Protect services from failing dependencies |
| Rate Limiting | Prevent excessive API requests |
| API Gateway | Centralized API entry point |
| Load Balancer | Distribute requests across instances |

---

# 16\. Important Interview Questions – Week 2

Try answering these without referring to the notes.

## Database Questions

1. What is a database?
2. What is the difference between SQL and NoSQL?
3. What is database indexing?
4. What are the advantages and disadvantages of indexing?
5. What is replication?
6. What is replication lag?
7. What is the difference between synchronous and asynchronous replication?
8. What is sharding?
9. What is the difference between replication and sharding?
10. Explain range-based and hash-based sharding.

## Distributed Systems Questions

11. What is CAP theorem?
12. Explain Consistency, Availability and Partition Tolerance.
13. What happens during a network partition?
14. What is a message queue?
15. What is Event-Driven Architecture?
16. What is the difference between a message queue and event-driven architecture?
17. What is the difference between Kafka and RabbitMQ at a high level?

## Database Consistency Questions

18. What is ACID?
19. Explain Atomicity with an example.
20. What is BASE?
21. What is eventual consistency?
22. What is optimistic locking?
23. What is pessimistic locking?
24. What is the difference between optimistic and pessimistic locking?

## Reliability Questions

25. What is idempotency?
26. Why is idempotency important in mobile applications?
27. What is exponential backoff?
28. Why should we use jitter?
29. What is a Circuit Breaker?
30. Explain Closed, Open and Half-Open states.

## API Architecture Questions

31. What is rate limiting?
32. What is HTTP 429?
33. Explain Token Bucket.
34. Explain Leaky Bucket.
35. What is an API Gateway?
36. What are its responsibilities?
37. What is the difference between an API Gateway and a Load Balancer?
38. What is API Aggregation?
39. How would you protect backend services from excessive requests?

---

# 17\. Scenario-Based Interview Questions

These are more important than memorizing definitions.

### Scenario 1: Duplicate Notes

A user clicks Create Note.

The server creates the note successfully, but the response is lost due to a network failure.

Android retries the request.

How do you prevent duplicate notes?

**Expected answer:**

Use an idempotency key. The backend stores the key and the original response. A repeated request with the same key returns the existing result.

### Scenario 2: Slow Database Reads

Your application has 10 million notes.

Fetching notes for a particular user is becoming slow.

What would you do?

**Expected answer:**

- Analyze the slow query.
- Add appropriate database indexes.
- Introduce pagination.
- Consider caching.
- Review query execution plans.
- Consider read replicas if read traffic requires scaling.

### Scenario 3: Database Server Failure

Your primary database server fails.

What happens?

**Expected answer:**

Use database high-availability mechanisms and replicas. A suitable replica may be promoted to primary, depending on the database setup. Applications need a reliable failover strategy.

### Scenario 4: Excessive API Requests

One Android client sends thousands of requests per minute.

How do you protect the backend?

**Expected answer:**

Implement server-side rate limiting, return HTTP 429 when appropriate, and use bounded client-side retries and debouncing.

### Scenario 5: Notification Service Is Down

A user creates a note, but the notification service is unavailable.

Should note creation fail?

**Expected answer:**

Not necessarily. Save the note successfully and publish a durable event for asynchronous notification processing. Use retries and appropriate failure handling.

### Scenario 6: Two Users Update the Same Note

User A and User B open the same shared note.

Both update it.

How do you prevent accidental overwriting?

**Expected answer:**

Use optimistic locking with a version number or another concurrency-control mechanism. Detect conflicts and ask the user to resolve them when necessary.

### Scenario 7: Backend Service Is Unavailable

Notes Service calls Search Service, but Search Service is down.

How do you prevent cascading failures?

**Expected answer:**

Use timeouts, bounded retries, Circuit Breaker and fallback behavior where appropriate.

### Scenario 8: Database Has Too Much Data

Your database has grown to billions of records.

What can you do?

**Expected answer:**

Evaluate database partitioning, sharding, indexing, archiving and data-retention policies based on access patterns.

---

# 18\. Mock Interview – Practice Your Answers

Imagine an interviewer asks:

**Interviewer:** You are designing a Notes application that supports 10 million users. How would you design its backend?

A good answer should follow this sequence:

### Step 1: Gather Requirements

Functional requirements:

- Register and log in.
- Create notes.
- Update notes.
- Delete notes.
- Search notes.
- Share notes.
- Sync across devices.

Non-functional requirements:

- High availability.
- Scalability.
- Low latency.
- Data durability.
- Security.
- Consistency.

### Step 2: Design the APIs

http

```http
POST   /api/users/register
POST   /api/users/login
GET    /api/notes
POST   /api/notes
GET    /api/notes/{id}
PUT    /api/notes/{id}
DELETE /api/notes/{id}
GET    /api/notes/search?q=system
```

### Step 3: Choose Database

Use a relational database such as PostgreSQL as a reasonable starting point.

Why?

- Users and notes have relationships.
- Transactions may be required.
- Structured querying is useful.

### Step 4: Improve Performance

- Add indexes.
- Implement pagination.
- Introduce caching.
- Use read replicas when needed.

### Step 5: Scale the System

- Horizontally scale backend instances.
- Use a load balancer.
- Consider database sharding when necessary.

### Step 6: Handle Asynchronous Operations

Use a message queue for:

- Notifications.
- Analytics.
- Background processing.
- Search indexing.

### Step 7: Handle Failures

- Timeouts.
- Retry with exponential backoff and jitter.
- Circuit Breaker.
- Idempotency.
- High-availability database configuration.

### Step 8: Protect APIs

- Authentication.
- Authorization.
- Rate Limiting.
- API Gateway.
- Input validation.

This gives you a structured way to explain your design rather than randomly listing technologies.

---

# 19\. Common Mistakes in System Design Interviews

1. Starting architecture without understanding requirements.
2. Choosing technologies without explaining why.
3. Adding Kafka, Redis and microservices unnecessarily.
4. Ignoring database indexing.
5. Forgetting pagination.
6. Ignoring failure scenarios.
7. Confusing replication with sharding.
8. Confusing availability with consistency.
9. Ignoring duplicate requests.
10. Assuming retries always solve failures.
11. Ignoring security and authorization.
12. Overcomplicating a simple application.

Remember:

A good system design answer explains why a component is needed, what problem it solves and what trade-offs it introduces.

---

# 20\. Week 2 – Final Revision Cheat Sheet

| Concept | One-Line Explanation |
| --- | --- |
| SQL | Relational database |
| NoSQL | Non-relational database models |
| Indexing | Speeds up suitable database queries |
| Replication | Creates copies of data |
| Sharding | Divides data across databases |
| CAP | Consistency, Availability and Partition Tolerance |
| Message Queue | Enables asynchronous communication |
| Event-Driven Architecture | Components communicate through events |
| ACID | Transaction properties |
| BASE | Availability-oriented distributed system approach |
| Optimistic Locking | Detects conflicts during updates |
| Pessimistic Locking | Uses locks to prevent conflicting updates |
| Idempotency | Repeating an operation produces the same intended result |
| Retry | Attempts an operation again |
| Exponential Backoff | Increases waiting time between retries |
| Circuit Breaker | Temporarily stops calls to a failing dependency |
| Rate Limiting | Controls incoming request frequency |
| Token Bucket | Allows requests using refillable tokens |
| Leaky Bucket | Processes queued requests at a controlled rate |
| API Gateway | Single entry point for API requests |
| Load Balancer | Distributes traffic across server instances |

---

# 21\. Final Checklist – Week 2

## Database

- [ ]SQL vs NoSQL
- [ ]Database indexing
- [ ]Database replication
- [ ]Synchronous vs asynchronous replication
- [ ]Replication lag
- [ ]Database sharding
- [ ]Sharding strategies

## Distributed Systems

- [ ]CAP theorem
- [ ]Message queues
- [ ]Event-driven architecture
- [ ]ACID
- [ ]BASE
- [ ]Eventual consistency

## Reliability

- [ ]Optimistic locking
- [ ]Pessimistic locking
- [ ]Idempotency
- [ ]Retry
- [ ]Exponential backoff
- [ ]Circuit Breaker

## API Architecture

- [ ]Rate Limiting
- [ ]Fixed Window
- [ ]Sliding Window
- [ ]Token Bucket
- [ ]Leaky Bucket
- [ ]API Gateway
- [ ]Load Balancer
- [ ]Authentication vs Authorization

## Interview Practice

- [ ]Explain database indexing with an example.
- [ ]Explain replication vs sharding.
- [ ]Explain CAP theorem.
- [ ]Explain ACID.
- [ ]Explain message queues.
- [ ]Explain idempotency.
- [ ]Explain Circuit Breaker.
- [ ]Design a scalable Notes application.

---

# 22\. What Is Next?

## Week 3 – Advanced System Design Concepts

We will move from understanding individual components to understanding how to design systems using them.

Planned topics:

| Day | Topic |
| --- | --- |
| Day 15 | Caching Fundamentals |
| Day 16 | Cache Strategies and Cache Invalidation |
| Day 17 | CDN and Distributed Caching |
| Day 18 | File Storage and Object Storage |
| Day 19 | Search Systems and Elasticsearch |
| Day 20 | Real-Time Communication and WebSockets |
| Day 21 | Week 3 Revision and Interview Practice |

After Week 3, we will start designing complete real-world applications.

Examples:

- Notes Application.
- Chat Application.
- Notification System.
- Social Media Feed.
- File Upload System.

**Week 2 Complete\!**

## Your target for today

Since you're learning system design for the first time, I recommend spending today's revision on understanding connections rather than memorizing 39 interview questions.

Make sure you can explain these five scenarios:

1. How would you make a database query faster?
2. How would you scale a database?
3. How would you prevent duplicate API requests?
4. How would you handle a backend service failure?
5. How would you protect an API from excessive traffic?

These are the kinds of questions that help you move from theoretical knowledge to actual system design thinking.

Next up: Week 3 – Day 15: Caching Fundamentals.
