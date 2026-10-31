# Week 2 – Day 10: Distributed Systems Basics

Week 2Day 10Beginner level

File name: `Week-2-Day-10-Distributed-Systems.md`

Today's goal: Understand how multiple computers work together as one system, why distributed systems are needed, what problems they introduce, and how the CAP theorem helps us understand trade-offs.

Yesterday, we learned:

- Database replication.
- Database sharding.
- Primary and replica databases.
- Horizontal database scaling.

Today, we will connect these concepts and understand the bigger picture.

Writing

# Week 2 – Day 10: Distributed Systems Basics

**Topic:** Scalability and Distributed Systems\
**Level:** Beginner\
**Target:** Android Developer – 7 Years Experience

---

# 1\. What Is a Distributed System?

Let's start with a simple example.

Imagine you are developing a Notes application.

Initially, your application has:

- 1 Android application.
- 1 backend server.
- 1 database.

Architecture:

text

```text
       ANDROID APP
            |
            v
       BACKEND SERVER
            |
            v
         DATABASE
```

This architecture is simple.

The backend server handles all requests, and the database stores all information.

Now imagine your application becomes popular.

You have:

- 10 million users.
- Millions of API requests.
- Billions of notes.
- Users accessing the application from different countries.

Can one backend server and one database handle everything efficiently?

Possibly not.

We need multiple servers working together.

Architecture:

text

```text
              ANDROID USERS
                    |
                    v
              LOAD BALANCER
                    |
          +---------+---------+
          |         |         |
          v         v         v
        SERVER 1  SERVER 2  SERVER 3
          |         |         |
          +---------+---------+
                    |
                    v
               DATABASES
```

Now multiple servers participate in processing requests.

This is the basic idea behind a Distributed System.

## 1.1 Definition

A Distributed System is a collection of independent computers that communicate over a network and work together to provide a service.

Each computer is called a node.

A node can be:

- A backend server.
- A database server.
- A cache server.
- A message broker.
- Another service participating in the system.

The nodes communicate with each other to complete operations.

## 1.2 Real-World Examples

Examples of systems that use distributed architectures include:

1. WhatsApp – Messaging infrastructure.
2. YouTube – Video storage and delivery.
3. Netflix – Streaming infrastructure.
4. Amazon – Shopping, payments and inventory.
5. Google – Search and other services.

These applications cannot depend on a single computer for all their operations.

They distribute workloads across many machines and services.

---

# 2\. Why Do We Need Distributed Systems?

Let's understand the main reasons.

## 2.1 Scalability

Imagine one server can handle 1,000 requests per second.

Your application grows and starts receiving 10,000 requests per second.

One server may not be sufficient.

We can add more servers.

text

```text
       BEFORE

       10,000 REQUESTS
               |
               v
           SERVER 1
```

text

```text
       AFTER

       10,000 REQUESTS
               |
               v
         LOAD BALANCER
          /    |    \
         v     v     v
       S1     S2     S3
```

This allows us to distribute the workload.

## 2.2 Availability

Availability means the system is accessible and able to serve requests when users need it.

Imagine your application has three backend servers.

One server crashes.

If the remaining servers can continue handling requests, the application can remain available.

text

```text
         LOAD BALANCER
          /    |    \
         v     v     v
       S1     S2     S3
       UP    DOWN    UP
```

The load balancer can route new requests to Server 1 and Server 3.

## 2.3 Fault Tolerance

Fault tolerance is the ability of a system to continue operating even when some components fail.

For example:

- One backend server crashes.
- One database replica becomes unavailable.
- One network connection fails.

A fault-tolerant system is designed to handle such failures without bringing down the entire application.

Important:

Fault tolerance does not mean failures never happen. It means the system is designed to tolerate them.

## 2.4 Performance

A distributed system can divide work across multiple computers.

For example, an e-commerce application may have separate services for:

- User management.
- Product catalog.
- Payments.
- Orders.
- Notifications.

Each service can process its own responsibilities.

## 2.5 Geographic Distribution

Imagine users accessing your application from:

- India.
- USA.
- UK.
- Australia.

If all requests travel to one server located in India, users in other countries may experience higher network latency.

We can deploy services in multiple geographical regions.

text

```text
           USERS
             |
      +------+------+
      |      |      |
      v      v      v
    INDIA   USA    EUROPE
    SERVER SERVER  SERVER
```

This can reduce network latency for users, depending on routing and data access requirements.

---

# 3\. What Is a Node?

A node is an individual computer or computing instance participating in a distributed system.

For example:

text

```text
       DISTRIBUTED SYSTEM

       +----------------+
       |    NODE 1      |
       | Backend Server |
       +----------------+

       +----------------+
       |    NODE 2      |
       | Backend Server |
       +----------------+

       +----------------+
       |    NODE 3      |
       | Database       |
       +----------------+
```

Each node has its own:

- CPU.
- Memory.
- Operating system.
- Network connection.
- Responsibilities.

Nodes communicate through a network.

Important:

A node does not necessarily mean a physical computer. It can also be a virtual machine or cloud computing instance.

---

# 4\. How Do Nodes Communicate?

Nodes communicate with each other over a network.

There are different communication approaches.

## 4.1 Synchronous Communication

In synchronous communication, the caller waits for a response from the other service.

Example:

Our Notes application needs user information.

The Notes service calls the User service.

text

```text
       NOTES SERVICE
              |
              | Request
              v
        USER SERVICE
              |
              | Response
              v
       NOTES SERVICE
```

Example API:

http

```http
GET /users/101
```

The Notes service waits for the User service response.

Common examples:

- HTTP REST APIs.
- Synchronous gRPC calls.

Advantages:

- Simple request-response model.
- Easy to understand.
- Suitable when an immediate response is required.

Disadvantages:

- Caller waits for the response.
- If the called service is slow, the caller is also delayed.
- Failures can propagate between services.

## 4.2 Asynchronous Communication

In asynchronous communication, the sender does not need to wait for the receiver to finish processing the task.

A message is usually sent through a message broker or queue.

Example:

A user creates a note and the system needs to send a notification.

Should the user wait until the notification is delivered?

Not necessarily.

The system can publish a notification event.

text

```text
       NOTES SERVICE
              |
              | Publish Event
              v
         MESSAGE QUEUE
              |
              v
      NOTIFICATION SERVICE
              |
              v
         SEND NOTIFICATION
```

The Notes service can continue processing without waiting for the notification service to finish.

Common technologies:

- Apache Kafka.
- RabbitMQ.
- Amazon SQS.

Advantages:

- Reduces direct dependency between services.
- Helps process background tasks.
- Can handle temporary workload spikes.

Disadvantages:

- More complex architecture.
- Messages may be delayed.
- Duplicate message processing must be considered.
- Debugging becomes more difficult.

We will study message queues in detail on Day 11.

## 4.3 Synchronous vs Asynchronous Communication

| Synchronous | Asynchronous |
| --- | --- |
| Caller waits for response | Caller does not need to wait for completion |
| Request-response model | Event/message-based model |
| Tighter dependency | Lower temporal dependency |
| Useful for immediate responses | Useful for background processing |
| Example: REST API | Example: Kafka, RabbitMQ |

---

# 5\. Challenges in Distributed Systems

Distributed systems provide scalability and availability, but they also introduce new problems.

This is an extremely important interview topic.

Let's understand each challenge.

## 5.1 Network Failure

Imagine Server 1 needs data from Server 2.

It sends a request.

But the network connection fails.

text

```text
       SERVER 1
           |
           |
       NETWORK X
           |
           v
       SERVER 2
```

Server 1 cannot communicate with Server 2.

The problem is that Server 2 may be:

- Running normally.
- Temporarily unavailable.
- Slow.
- Completely crashed.

Server 1 may not immediately know which situation occurred.

This is a major challenge in distributed systems.

## 5.2 Partial Failure

What is a partial failure?

Imagine a system contains five services.

Four services are working correctly.

One service is unavailable.

text

```text
       DISTRIBUTED SYSTEM

       SERVICE 1  UP
       SERVICE 2  UP
       SERVICE 3  DOWN
       SERVICE 4  UP
       SERVICE 5  UP
```

The entire system has not crashed, but one component has failed.

This is called a partial failure.

In a single-computer application, failure is often easier to detect.

In a distributed system, different components can fail independently.

## 5.3 Latency

Latency is the time taken for data to travel from one point to another or for an operation to complete.

Imagine:

1. Notes service calls User service.
2. User service calls Database.
3. Database returns data.
4. User service responds to Notes service.

Every network call introduces some delay.

If there are many dependent calls, the total response time can increase.

Example:

text

```text
Notes Service       20 ms
       |
       v
User Service        50 ms
       |
       v
Database            80 ms
       |
       v
Total               150 ms
```

These are illustrative values.

We should minimize unnecessary network calls.

## 5.4 Data Consistency

Imagine we have two database replicas.

text

```text
        PRIMARY DB
             |
       +-----+-----+
       |           |
       v           v
    REPLICA 1   REPLICA 2
```

A user updates a note.

The primary database gets the latest value.

But Replica 2 has not received the update yet.

Now:

- Primary contains the new value.
- Replica 1 may contain the new value.
- Replica 2 may still contain the old value.

The system temporarily has different versions of the same data.

This creates a consistency challenge.

## 5.5 Concurrency

Concurrency means multiple operations happen at the same time.

Imagine two users edit the same shared note.

- User A changes the title.
- User B changes the content.

Both updates arrive almost simultaneously.

How should the system handle them?

Possible solutions include:

- Database transactions.
- Optimistic locking.
- Pessimistic locking.
- Version numbers.
- Conflict resolution strategies.

Without proper handling, one user's changes may overwrite another user's changes.

## 5.6 Duplicate Requests

Imagine Android sends a payment request.

The backend processes the payment successfully.

But the network response gets lost.

Android assumes the payment failed and retries.

Now the backend receives the same payment request again.

Could the user be charged twice?

Yes, if the backend does not handle duplicate requests properly.

A common solution is an idempotency key.

Example:

http

```http
POST /payments
Idempotency-Key: payment-12345
```

The backend identifies repeated requests using the same key and avoids performing the same payment operation again.

Important:

Idempotency is especially important for payments, orders and other operations where duplicate execution can cause problems.

---

# 6\. What Is the CAP Theorem?

This is one of the most important distributed systems interview topics.

Before understanding CAP, we need to understand three terms:

- Consistency.
- Availability.
- Partition Tolerance.

CAP stands for:

C – Consistency

A – Availability

P – Partition Tolerance

## 6.1 Consistency

In the CAP theorem, consistency means every successful read receives the latest successful write, or an error if that cannot be guaranteed.

Example:

A user changes their note title.

Old title:

"Meeting"

New title:

"Project Meeting"

After the successful update, a subsequent read should return the new title under this consistency guarantee.

text

```text
WRITE:
"Meeting" -> "Project Meeting"

READ:
"Project Meeting"
```

## 6.2 Availability

Availability means every request received by a non-failing node gets a response.

The response might not contain the latest data, but the system responds.

Example:

Even if one replica is unreachable, another node may still respond to a read request.

## 6.3 Partition Tolerance

A network partition happens when nodes cannot communicate with each other because of a network failure.

Example:

text

```text
        NODE A
           |
           |
       NETWORK X
           |
           v
        NODE B
```

Node A and Node B cannot communicate.

Partition tolerance means the system continues operating despite such communication failures, although it may have to sacrifice consistency or availability for some operations.

## 6.4 Understanding CAP with an Example

Imagine we have two database nodes.

text

```text
        NODE A -------- NODE B
```

Both contain copies of the same data.

Now the network between them fails.

text

```text
        NODE A    X    NODE B
```

A user updates data on Node A.

Node B cannot receive that update because communication has failed.

Now another user reads data from Node B.

Node B has old data.

What should the system do?

It has two broad choices.

### Option 1: Prefer Consistency

Node B refuses to return potentially stale data.

It may return an error or reject the operation.

The system sacrifices availability for that operation to preserve consistency.

This is commonly described as CP behavior.

### Option 2: Prefer Availability

Node B continues responding with the data it currently has.

The user receives a response, but it might be stale.

The system sacrifices strong consistency for availability.

This is commonly described as AP behavior.

## 6.5 CAP Diagram

text

```text
              CAP
               |
       +-------+-------+
       |       |       |
       C       A       P

   Consistency Availability Partition
                     Tolerance
```

The important idea:

**When a network partition occurs, a distributed system must make a trade-off between consistency and availability.**

## 6.6 Does CAP Mean We Can Only Choose Two Out of Three?

This is a common misunderstanding.

The CAP theorem is not simply a rule that every system must permanently choose any two properties.

In practical distributed systems:

- Network partitions are possible.
- A partition can prevent nodes from communicating.
- During a partition, the system must decide how to handle requests.
- It may prefer consistency or availability for those requests.

When there is no partition, systems can provide both consistency and availability under appropriate conditions.

## 6.7 CP vs AP

| CP System | AP System |
| --- | --- |
| Prefers consistency during partition | Prefers availability during partition |
| May reject or delay some requests | May return stale data |
| Prioritizes correct/latest data | Prioritizes responding to requests |
| Useful when stale data is unacceptable | Useful when temporary stale data is acceptable |

Examples are context-dependent. Many real databases offer configurable consistency guarantees rather than fitting perfectly into one category.

---

# 7\. Real-World CAP Examples

Let's understand this using our Notes application.

## Scenario A: Bank Transfer

Imagine a banking application.

User transfers ₹10,000.

We should not allow inconsistent account balances to cause incorrect financial results.

If the system cannot safely coordinate the transaction, it may reject or delay the operation.

Consistency is extremely important.

## Scenario B: Social Media Likes

Imagine a user likes a post.

The system updates the like count.

A different user might temporarily see an older like count because replicas have not synchronized yet.

For many social media interactions, this temporary inconsistency may be acceptable.

Availability and responsiveness can be prioritized.

Important:

These are simplified illustrations, not claims that all banking systems are CP or all social media systems are AP.

---

# 8\. What Is a Distributed Database?

A distributed database is a database system in which data is stored across multiple computers connected through a network.

Example:

text

```text
          APPLICATION
                |
                v
         DATABASE ROUTER
                |
        +-------+-------+
        |       |       |
        v       v       v
      NODE 1  NODE 2  NODE 3
```

The nodes work together to store and retrieve data.

A distributed database can use:

- Replication.
- Sharding.
- Partitioning.
- Distributed transactions.
- Consensus protocols.

Examples of distributed database technologies include:

- Apache Cassandra.
- Google Spanner.
- CockroachDB.
- Amazon DynamoDB.

Different systems provide different consistency, availability and transaction guarantees.

---

# 9\. What Is a Consensus Algorithm?

Imagine three database nodes.

They need to agree on an important decision, such as which node should become the leader.

How can they agree if they communicate over a network?

This problem is addressed by consensus algorithms.

## 9.1 Definition

A consensus algorithm is a mechanism that allows multiple distributed nodes to agree on a value or decision.

For example:

text

```text
       NODE A
          |
          |
       NODE B
          |
          |
       NODE C

   All agree on one leader
```

Consensus is useful for:

- Leader election.
- Coordinating distributed state.
- Maintaining an agreed sequence of operations.

## 9.2 Common Consensus Algorithms

### Raft

Raft is a consensus algorithm designed to be understandable and practical.

It uses concepts such as:

- Leader.
- Followers.
- Terms.
- Log replication.

### Paxos

Paxos is another family of consensus algorithms used to achieve agreement among distributed nodes.

You don't need to implement these algorithms for a beginner system design interview.

For now, understand why consensus is needed.

---

# 10\. What Is Leader Election?

Imagine three nodes:

- Node A.
- Node B.
- Node C.

One node acts as the leader.

The other nodes act as followers.

The leader coordinates certain operations.

text

```text
          NODE A
           LEADER
          /     \
         v       v
      NODE B   NODE C
     FOLLOWER  FOLLOWER
```

If the leader fails, the remaining nodes may elect a new leader.

This process is called Leader Election.

Leader election helps distributed systems continue operating when a leader becomes unavailable.

---

# 11\. What Is Eventual Consistency?

Eventual consistency means that if no new updates occur and communication continues normally, all replicas will eventually converge to the same data.

Example:

Initially:

text

```text
PRIMARY: 100 Likes
REPLICA: 100 Likes
```

A user likes the post.

text

```text
PRIMARY: 101 Likes
REPLICA: 100 Likes
```

After replication:

text

```text
PRIMARY: 101 Likes
REPLICA: 101 Likes
```

The replicas eventually become consistent.

This is useful when the application can tolerate temporary differences between replicas.

Examples:

- Social media like counts.
- View counters.
- Some analytics systems.
- Some product recommendation data.

Important:

Eventual consistency does not mean updates are guaranteed to appear immediately. The system must also be designed to handle failures, conflicts and delayed propagation.

---

# 12\. Distributed System vs Monolithic Architecture

Let's compare two architectural approaches.

## 12.1 Monolithic Architecture

A monolithic application contains multiple business functionalities inside one deployable application.

Example:

text

```text
          BACKEND APPLICATION
                  |
        +---------+---------+
        |         |         |
        v         v         v
      USERS     NOTES    NOTIFICATIONS
```

All these functionalities are part of one application.

Advantages:

- Simple to start.
- Easier to develop initially.
- Simpler deployment.
- Internal function calls are often faster than network calls.

Disadvantages:

- Large codebase as the application grows.
- Scaling individual functionalities independently is harder.
- Changes to one module may affect other modules.

## 12.2 Microservices Architecture

Microservices divide an application into smaller independently deployable services.

Example:

text

```text
             API GATEWAY
                  |
       +----------+----------+
       |          |          |
       v          v          v
   USER SERVICE NOTES SERVICE PAYMENT SERVICE
       |          |          |
       v          v          v
    USER DB     NOTES DB   PAYMENT DB
```

Each service handles a specific business responsibility.

Advantages:

- Independent deployment.
- Independent scaling.
- Separate ownership.
- Technology flexibility.

Disadvantages:

- Network communication complexity.
- Distributed transactions become difficult.
- Monitoring and debugging become more complex.
- More infrastructure is required.

Important:

A microservices architecture is generally a distributed system, but not every distributed system uses microservices.

A distributed system can also consist of multiple database nodes, replicas or other cooperating components.

---

# 13\. Real-World Notes Application Architecture

Let's combine everything we have learned.

Assume our Notes application has:

- 10 million users.
- Multiple backend servers.
- Multiple database shards.
- Database replicas.
- A notification service.
- A message broker.

Architecture:

text

```text
                ANDROID USERS
                      |
                      v
                LOAD BALANCER
                      |
                      v
                 API GATEWAY
                      |
             +--------+--------+
             |                 |
             v                 v
        NOTES SERVICE      USER SERVICE
             |                 |
             v                 v
        DATABASE SHARDS     USER DB
             |
        +----+----+
        |         |
        v         v
     REPLICA 1  REPLICA 2

             NOTES SERVICE
                    |
                    v
               MESSAGE QUEUE
                    |
                    v
           NOTIFICATION SERVICE
                    |
                    v
              SEND NOTIFICATION
```

Example: User creates a note.

1. Android sends a request.
2. Load balancer forwards it to the API gateway.
3. API gateway routes the request to Notes service.
4. Notes service stores the note in the appropriate database shard.
5. A note-created event is published to a message queue.
6. Notification service consumes the event.
7. The notification is processed independently.
8. Database replicas receive the updated data.

This is a simplified distributed architecture.

---

# 14\. Android Developer Perspective

As an Android developer with backend system design knowledge, distributed systems affect your application in several ways.

## 14.1 Network Failures

Your Retrofit API request may fail because:

- Server is unavailable.
- Network is disconnected.
- Request times out.
- Another backend service is unavailable.

Your application should handle these cases gracefully.

## 14.2 Retry Mechanism

Suppose Android sends a request but does not receive a response.

A retry may help with temporary network failures.

However, not every request should be retried blindly.

For example:

- GET request: Usually safe to retry.
- Payment POST request: Needs idempotency protection.
- Create-note request: Should be designed to avoid accidental duplicate notes.

## 14.3 Offline-First Architecture

Your Android application can use Room as a local database.

Even when the backend is temporarily unavailable, users can continue viewing locally stored notes.

Later, synchronization can upload pending changes.

## 14.4 Eventual Consistency

If the backend uses eventually consistent replicas, Android might temporarily receive older data.

The UI should handle refresh and synchronization correctly.

## 14.5 Timeout Handling

Always understand the difference between:

- Connection timeout.
- Read timeout.
- Request timeout.

A timeout means the client did not receive the expected response within the configured time. It does not necessarily mean the server did not complete the operation.

---

# 15\. Common Mistakes to Avoid

1. Thinking a distributed system means only multiple backend servers.
2. Confusing scalability with availability.
3. Thinking fault tolerance means failures never happen.
4. Assuming network communication is always reliable.
5. Assuming every distributed database immediately synchronizes data.
6. Thinking CAP means a system permanently chooses only two properties.
7. Assuming eventual consistency means data will never be consistent.
8. Confusing microservices with distributed systems.
9. Retrying non-idempotent operations without protection.
10. Assuming more servers always mean better performance.

---

# 16\. Important Interview Questions and Answers

### Q1. What is a Distributed System?

A collection of independent computers that communicate over a network and work together to provide a service.

### Q2. Why do we need Distributed Systems?

To improve scalability, availability, fault tolerance, performance and geographic distribution.

### Q3. What is a Node?

An individual computer or computing instance participating in a distributed system.

### Q4. What is Partial Failure?

When some components fail while other components continue operating.

### Q5. What is Synchronous Communication?

Communication where the caller waits for the response from another service.

### Q6. What is Asynchronous Communication?

Communication where the sender does not need to wait for the receiver to finish processing the task.

### Q7. What are the main challenges of Distributed Systems?

Network failures, latency, partial failures, consistency, concurrency and duplicate requests.

### Q8. What is the CAP Theorem?

The CAP theorem explains that during a network partition, a distributed data system must trade off between consistency and availability.

### Q9. What is Consistency in CAP?

Every successful read returns the latest successful write, or an error if the system cannot guarantee that result.

### Q10. What is Availability in CAP?

Every request received by a non-failing node receives a response.

### Q11. What is Partition Tolerance?

The system continues operating despite communication failures between nodes, with an appropriate consistency or availability trade-off.

### Q12. What is Eventual Consistency?

Replicas eventually converge to the same data if updates stop and communication continues normally.

### Q13. What is Consensus?

A mechanism that allows distributed nodes to agree on a value or decision.

### Q14. What is Leader Election?

The process of selecting a leader node among multiple distributed nodes.

### Q15. What is the difference between Monolithic and Microservices Architecture?

A monolith is one deployable application containing multiple functionalities. Microservices divide functionalities into independently deployable services.

### Q16. Is every microservices architecture a distributed system?

Yes, because independently deployed services communicate over a network. However, not every distributed system is based on microservices.

### Q17. Why is Idempotency important?

It prevents repeated execution of the same logical operation from producing unintended duplicate effects.

### Q18. What happens if two users update the same note simultaneously?

The system needs a conflict-handling strategy, such as optimistic locking, version numbers or another concurrency control mechanism.

### Q19. What is the difference between Fault Tolerance and Availability?

Fault tolerance is the ability to continue operating despite component failures. Availability describes whether the system is accessible and able to serve requests.

### Q20. Why are distributed systems difficult to debug?

Because multiple independent services communicate over networks, failures can happen at different locations, and the same request may pass through many components.

---

# 17\. Practical Interview Exercise

Imagine you are designing a Notes application with:

- 10 million users.
- Multiple backend services.
- Multiple database replicas.
- Users accessing the application globally.

Answer these questions:

1. Why would you use a distributed system?
2. What is the role of each backend node?
3. What happens if one backend server crashes?
4. What happens if two database nodes cannot communicate?
5. How would you handle duplicate create-note requests?
6. Would you use synchronous or asynchronous communication for notifications?
7. What is the CAP theorem?
8. Would you prefer consistency or availability for a bank transfer?
9. Would temporary stale data be acceptable for a like counter?
10. How would you handle simultaneous updates to the same note?
11. What is eventual consistency?
12. How would you make the Android application work when the backend is unavailable?

Try to explain your answers with diagrams.

---

# 18\. Day 10 – Quick Revision

| Concept | Simple Meaning |
| --- | --- |
| Distributed System | Multiple computers working together |
| Node | Individual computer or instance |
| Scalability | Handle increasing workload |
| Availability | System can serve requests |
| Fault Tolerance | Continue operating despite failures |
| Synchronous | Caller waits for response |
| Asynchronous | Caller need not wait for processing |
| Partial Failure | Some components fail, others work |
| Latency | Time taken for an operation |
| Consistency | Reads follow the required data consistency guarantee |
| CAP | Consistency, Availability, Partition Tolerance |
| Network Partition | Nodes cannot communicate |
| Consensus | Nodes agree on a decision |
| Leader Election | Select a leader node |
| Eventual Consistency | Replicas eventually converge |
| Monolithic | One deployable application |
| Microservices | Independently deployable services |
| Idempotency | Repeated operation avoids duplicate effects |

---

# 19\. Final Learning Checklist

- [ ]Understand what a distributed system is.
- [ ]Understand why distributed systems are needed.
- [ ]Understand what a node is.
- [ ]Understand synchronous communication.
- [ ]Understand asynchronous communication.
- [ ]Understand network failures.
- [ ]Understand partial failures.
- [ ]Understand latency.
- [ ]Understand data consistency.
- [ ]Understand concurrency.
- [ ]Understand idempotency.
- [ ]Understand the CAP theorem.
- [ ]Understand Consistency vs Availability.
- [ ]Understand Partition Tolerance.
- [ ]Understand eventual consistency.
- [ ]Understand consensus algorithms.
- [ ]Understand leader election.
- [ ]Understand Monolithic vs Microservices.
- [ ]Explain the Notes application distributed architecture.

---

# 20\. What Is Next?

## Week 2 – Day 11: Message Queues and Event-Driven Architecture

Tomorrow we will learn:

1. What is a message queue?
2. Why do we need message queues?
3. Synchronous vs asynchronous processing.
4. Producer and Consumer.
5. Message Broker.
6. Kafka fundamentals.
7. RabbitMQ fundamentals.
8. Queue vs Topic.
9. Dead Letter Queue (DLQ).
10. Message delivery guarantees.
11. Real-world notification architecture.
12. Interview questions.

**Day 10 Complete – Distributed Systems Basics.**

## Today's most important concepts

Before moving to Day 11, make sure you can explain these five concepts in your own words:

1. Distributed System: Multiple computers working together.
2. Partial Failure: One component fails while other components continue working.
3. CAP Theorem: During a network partition, a system must trade off consistency and availability.
4. Eventual Consistency: Replicas may temporarily differ but eventually converge.
5. Idempotency: Repeating the same logical operation should not create unintended duplicate effects.

My recommendation: Spend extra time on the CAP theorem. Don't just memorize CP and AP. Understand what happens when two database nodes lose communication with each other. That understanding is much more valuable in system design interviews.
