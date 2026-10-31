# Week 2 – Day 12: Consistency and Reliability

Week 2Day 12Beginner level

File name: `Week-2-Day-12-Consistency-and-Reliability.md`

Today's goal: Understand how systems maintain correct data, handle concurrent updates, recover from failures, and prevent duplicate operations.

Yesterday, we learned:

- Message queues.
- Kafka and RabbitMQ.
- Event-driven architecture.
- Message delivery guarantees.
- Dead Letter Queues.
- Transactional Outbox Pattern.

Today, we will understand an important question:

How do we make sure our system stores correct data, even when multiple users update it simultaneously or services fail?

Writing

# Week 2 – Day 12: Consistency and Reliability

**Topic:** Distributed Systems and Database Reliability\
**Level:** Beginner\
**Target:** Android Developer – 7 Years Experience

---

# 1\. What Is Data Consistency?

Let's start with a simple example.

Imagine you are building a Notes application.

A user creates a note:

text

```text
Title: Meeting Notes
Content: Discuss project architecture
```

The note is saved in the database.

Now the user updates the title:

text

```text
Title: System Design Interview
Content: Discuss project architecture
```

When the user opens the note again, what should happen?

The application should show the updated title.

This is called data consistency in a general database sense: the system maintains data according to its correctness rules and consistency guarantees.

## 1.1 Definition

Data consistency means that data remains correct and follows the rules defined by the system.

For example:

- A note should have a valid user ID.
- A payment should not be deducted twice for one logical transaction.
- An order should not reference a nonexistent customer.
- An account balance should follow the business rules.

## 1.2 Why Is Consistency Important?

Imagine a banking application.

User A has ₹10,000.

User A transfers ₹2,000 to User B.

Expected result:

text

```text
User A: ₹8,000
User B: ₹2,000
```

But imagine the money is deducted from User A and not credited to User B.

The system is now in an incorrect state.

Consistency is important because users expect their data to follow the application's business rules.

---

# 2\. Strong Consistency vs Eventual Consistency

We already introduced eventual consistency on Day 10.

Now let's understand both concepts in more detail.

## 2.1 Strong Consistency

Strong consistency means that after a successful write, subsequent reads observe that write, according to the system's consistency guarantee.

Example:

A user updates a note title.

Old title:

text

```text
Meeting
```

New title:

text

```text
System Design
```

After the update succeeds, a subsequent read returns:

text

```text
System Design
```

The user should not see the older value from a stale replica under this guarantee.

### Architecture

text

```text
         ANDROID APP
              |
              v
         NOTES SERVICE
              |
              v
          DATABASE
              |
              v
         WRITE SUCCESS
              |
              v
       READ LATEST DATA
```

### Where Is Strong Consistency Useful?

- Banking transactions.
- Account balances.
- Inventory reservation.
- Payment processing.
- Critical order updates.

These operations often require strict correctness.

## 2.2 Eventual Consistency

Eventual consistency means that data across replicas or services may temporarily differ, but if updates stop and the system can communicate normally, the data eventually converges.

Example:

A user likes a social media post.

The primary database updates the like count immediately.

The replica has not received the update yet.

text

```text
PRIMARY DATABASE: 101 Likes

REPLICA DATABASE: 100 Likes
```

After replication:

text

```text
PRIMARY DATABASE: 101 Likes

REPLICA DATABASE: 101 Likes
```

Now both contain the same value.

### Where Is Eventual Consistency Useful?

- Social media like counts.
- View counts.
- Analytics.
- Search indexing.
- Product recommendations.
- Some notification and activity feeds.

## 2.3 Strong vs Eventual Consistency

| Strong Consistency | Eventual Consistency |
| --- | --- |
| Reads observe completed writes under the guarantee | Reads may temporarily return older data |
| Stronger coordination may be required | Can allow more flexible replication |
| Useful for critical business data | Useful for data that tolerates temporary differences |
| Often has higher coordination cost | Can provide better availability and lower latency in some designs |

Important:

Strong consistency does not mean every system must use one database server.

Distributed databases can also provide strong consistency using coordination protocols.

## 2.4 Real-World Notes Application Example

Consider two features.

**Feature A: Save a note**

When a user updates a note, they expect the latest version to be available when they reopen it.

Strong consistency may be appropriate for this operation.

**Feature B: Update analytics**

When a user creates a note, the analytics dashboard does not necessarily need to update instantly.

Eventual consistency is usually acceptable.

The important lesson:

Choose consistency based on business requirements.

---

# 3\. What Are ACID Properties?

ACID is one of the most important database interview topics.

ACID describes four properties that help ensure reliable database transactions.

ACID stands for:

- A – Atomicity
- C – Consistency
- I – Isolation
- D – Durability

Let's understand each with a banking example.

## 3.1 Atomicity

Atomicity means a transaction happens completely or not at all.

Imagine User A transfers ₹2,000 to User B.

Two operations are required:

1. Deduct ₹2,000 from User A.
2. Add ₹2,000 to User B.

What if the first operation succeeds but the second fails?

Money would disappear from the system's account balances.

Atomicity prevents a partially completed transaction from being committed.

text

```text
       TRANSFER ₹2,000
               |
        +------+------+
        |             |
        v             v
    DEDUCT A       CREDIT B
        |             |
        +------+------+
               |
               v
         BOTH SUCCESS?
           /       \
         YES        NO
          |          |
          v          v
       COMMIT      ROLLBACK
```

A transaction is committed only when the complete operation succeeds.

Otherwise, it is rolled back.

**Remember:** All or nothing.

## 3.2 Consistency

Consistency means a transaction moves the database from one valid state to another valid state while respecting defined constraints and business rules.

Example:

Before transfer:

text

```text
User A: ₹10,000
User B: ₹5,000

Total: ₹15,000
```

After transferring ₹2,000:

text

```text
User A: ₹8,000
User B: ₹7,000

Total: ₹15,000
```

The total amount remains the same.

The database should not violate its defined constraints.

**Remember:** Data must follow business rules.

## 3.3 Isolation

Isolation means concurrent transactions should not interfere with each other in ways that violate the selected isolation level.

Imagine two users attempt to book the last available movie ticket.

Both read:

text

```text
Available Tickets: 1
```

Both try to book the ticket.

Without proper concurrency control, both might successfully book it.

This creates an incorrect result.

Isolation and concurrency control help prevent such problems.

**Remember:** Concurrent transactions should be handled safely.

## 3.4 Durability

Durability means once a transaction is successfully committed, its changes are preserved even if the database crashes or restarts, subject to the database's durability guarantees.

Example:

A user creates a note.

The database confirms that the transaction is committed.

The database server restarts.

The note should still exist.

**Remember:** Committed data should not disappear.

## 3.5 ACID Summary

| Property | Meaning | Example |
| --- | --- | --- |
| Atomicity | All or nothing | Both transfer operations succeed or neither commits |
| Consistency | Valid database state | Account balance rules are maintained |
| Isolation | Safe concurrent execution | Two users cannot book the same last ticket |
| Durability | Committed data persists | Saved note survives database restart |

## 3.6 ACID Example Using SQL

Imagine two bank accounts.

sql

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 2000
WHERE account_id = 101;

UPDATE accounts
SET balance = balance + 2000
WHERE account_id = 102;

COMMIT;
```

If an operation fails, the application can roll back the transaction:

sql

```sql
ROLLBACK;
```

This is a simplified example. A real transfer also needs validation, sufficient-balance checks, concurrency control and appropriate error handling.

---

# 4\. What Are BASE Properties?

BASE is another database design concept, commonly associated with systems that prioritize availability and scalability.

BASE stands for:

- Basically Available.
- Soft State.
- Eventual Consistency.

Let's understand each.

## 4.1 Basically Available

The system attempts to remain available and respond to requests, even if some components are unavailable.

Example:

A social media application may continue showing a user's feed even if some backend services are temporarily unavailable.

## 4.2 Soft State

The system's state may change over time even without a new user request, because updates from other nodes or services may arrive asynchronously.

Example:

A replica receives an update from the primary database.

Its data changes even though no user directly updated that replica.

## 4.3 Eventual Consistency

Replicas eventually converge to the same data if updates stop and communication resumes normally.

Example:

A like count is temporarily different across replicas but eventually becomes consistent.

## 4.4 ACID vs BASE

| ACID | BASE |
| --- | --- |
| Transaction-focused guarantees | Availability-oriented distributed design |
| Strong transaction guarantees | Often tolerates temporary inconsistency |
| Useful for critical transactional operations | Useful for large distributed workloads |
| Common in relational databases | Common in distributed NoSQL designs |

Important:

ACID and BASE are not strict opposites.

Modern databases may support ACID transactions while also offering distributed scalability and configurable consistency levels.

---

# 5\. What Is a Database Transaction?

A transaction is a group of database operations treated as one logical unit of work.

Example:

A user creates a note and its associated tags.

Operations:

1. Insert note.
2. Insert tag relationships.
3. Update note metadata.

If an important operation fails, we may want the entire transaction rolled back.

## 5.1 Transaction Lifecycle

text

```text
        BEGIN TRANSACTION
                |
                v
          EXECUTE QUERIES
                |
                v
          ALL SUCCESSFUL?
             /       \
           YES        NO
            |          |
            v          v
          COMMIT     ROLLBACK
            |          |
            v          v
          SAVED      REVERTED
```

## 5.2 Common Transaction Commands

| Command | Meaning |
| --- | --- |
| BEGIN | Start transaction |
| COMMIT | Save transaction changes |
| ROLLBACK | Undo uncommitted changes |
| SAVEPOINT | Create a point to which part of a transaction can be rolled back |

---

# 6\. What Are Database Isolation Levels?

This is an important interview topic.

Imagine two transactions are running at the same time.

Transaction A reads a value.

Transaction B updates that value.

What should Transaction A see?

Database isolation levels define which concurrent transaction effects are visible.

There are four commonly discussed SQL isolation levels.

1. Read Uncommitted.
2. Read Committed.
3. Repeatable Read.
4. Serializable.

Before understanding them, learn three important problems.

## 6.1 Dirty Read

A dirty read happens when one transaction reads uncommitted data written by another transaction.

Example:

Transaction A:

text

```text
Update balance to ₹5,000
```

But Transaction A has not committed.

Transaction B reads ₹5,000.

Later, Transaction A rolls back.

The actual balance returns to its previous value.

Transaction B read data that was never committed.

This is a dirty read.

## 6.2 Non-Repeatable Read

A non-repeatable read happens when a transaction reads the same row twice and gets different values because another transaction committed an update in between.

Example:

Transaction A reads:

text

```text
Balance = ₹10,000
```

Transaction B updates and commits:

text

```text
Balance = ₹8,000
```

Transaction A reads again:

text

```text
Balance = ₹8,000
```

The same query returns a different value within Transaction A.

This is a non-repeatable read.

## 6.3 Phantom Read

A phantom read happens when a transaction repeats a query that returns a set of rows and sees additional or missing rows because another transaction committed changes that affect the result.

Example:

Transaction A:

sql

```sql
SELECT * FROM notes
WHERE user_id = 101;
```

It gets 5 notes.

Transaction B inserts another note for user 101 and commits.

Transaction A executes the same query again.

Now it gets 6 notes.

The additional row is called a phantom row.

## 6.4 Isolation Level Comparison

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
| --- | --- | --- | --- |
| Read Uncommitted | Possible | Possible | Possible |
| Read Committed | Prevented | Possible | Possible |
| Repeatable Read | Prevented | Prevented | May be possible depending on DB implementation |
| Serializable | Prevented | Prevented | Prevented |

Important:

Actual behavior can vary by database implementation. For example, PostgreSQL's Repeatable Read uses snapshot isolation and prevents phantom reads in the usual SQL sense.

## 6.5 Read Uncommitted

The lowest commonly defined isolation level.

A transaction may read uncommitted changes from another transaction.

This can result in dirty reads.

## 6.6 Read Committed

A transaction reads only committed data.

However, two reads inside the same transaction may return different values.

## 6.7 Repeatable Read

If a transaction reads a row, repeated reads of that row remain consistent under the isolation guarantee.

Some databases may still allow phantom reads at this level.

## 6.8 Serializable

The strongest standard SQL isolation level.

Concurrent transactions behave as though they were executed one after another in some serial order.

This provides stronger correctness but may reduce concurrency or require transaction retries.

---

# 7\. What Is Optimistic Locking?

Now let's understand a very common system design interview topic.

Imagine two users editing the same shared note.

Initially:

text

```text
Note Version: 1
Title: Meeting
```

User A opens the note.

User B also opens the note.

Both users have version 1.

User A updates the title:

text

```text
Title: Project Meeting
Version: 2
```

Now User B tries to update the note using the old version.

Should we blindly overwrite User A's changes?

No.

We can use Optimistic Locking.

## 7.1 Definition

Optimistic Locking assumes conflicts are relatively uncommon.

It allows multiple users to read data, but checks whether the data changed before accepting an update.

## 7.2 How Does It Work?

We maintain a version number.

Initial record:

text

```text
Note ID: 101
Title: Meeting
Version: 1
```

User A and User B both read version 1.

User A updates:

sql

```sql
UPDATE notes
SET title = 'Project Meeting',
    version = version + 1
WHERE id = 101
AND version = 1;
```

The update succeeds.

Now the version is 2.

User B tries to update using version 1:

sql

```sql
UPDATE notes
SET title = 'New Meeting',
    version = version + 1
WHERE id = 101
AND version = 1;
```

The update affects zero rows because the current version is already 2.

The system detects a conflict.

## 7.3 Architecture

text

```text
          USER A             USER B
             |                  |
             v                  v
         READ V1             READ V1
             |                  |
             v                  v
        UPDATE V1            UPDATE V1
             |                  |
             v                  v
         SUCCESS              FAILURE
          V2                  CONFLICT
```

## 7.4 Where Is Optimistic Locking Useful?

- Shared document editing.
- Profile updates.
- Inventory management.
- Notes applications.
- APIs with infrequent update conflicts.

## 7.5 Advantages

- Avoids locking records during the entire editing period.
- Good when conflicts are uncommon.
- Allows concurrent reads.

## 7.6 Disadvantages

- Updates may fail when conflicts occur.
- Application needs conflict-handling logic.
- Users may need to refresh or merge changes.

---

# 8\. What Is Pessimistic Locking?

Optimistic locking assumes conflicts are uncommon.

Pessimistic locking assumes conflicts are possible and locks data to prevent conflicting operations.

Imagine two users trying to book the last movie ticket.

User A starts booking.

The system locks the relevant ticket record.

User B tries to book the same ticket.

User B must wait until User A completes the transaction or releases the lock.

## 8.1 Definition

Pessimistic Locking prevents conflicting operations by acquiring locks before performing critical updates.

## 8.2 Example

sql

```sql
BEGIN;

SELECT *
FROM tickets
WHERE ticket_id = 101
FOR UPDATE;

UPDATE tickets
SET status = 'BOOKED'
WHERE ticket_id = 101;

COMMIT;
```

The `FOR UPDATE` clause requests a row-level lock in databases that support it.

The exact locking behavior depends on the database.

## 8.3 Architecture

text

```text
          USER A
             |
             v
        LOCK RECORD
             |
             v
       UPDATE RECORD
             |
             v
       RELEASE LOCK
             |
             v
          USER B
       CAN PROCEED
```

## 8.4 Where Is Pessimistic Locking Useful?

- Critical inventory updates.
- Booking the last available ticket.
- Financial transactions.
- Operations with frequent conflicts.

## 8.5 Advantages

- Prevents conflicting updates while the lock is held.
- Useful when contention is high.
- Helps protect critical shared resources.

## 8.6 Disadvantages

- Other transactions may need to wait.
- Can reduce throughput.
- Deadlocks are possible.
- Long-running transactions can block other users.

## 8.7 Optimistic vs Pessimistic Locking

| Optimistic | Pessimistic |
| --- | --- |
| Assumes conflicts are uncommon | Assumes conflicts may happen |
| Checks version before update | Acquires lock before update |
| Conflicts detected during update | Conflicting operations may wait |
| Good for low contention | Useful for high contention |
| Example: Shared note editing | Example: Booking last ticket |

---

# 9\. What Is Idempotency?

This is extremely important for system design interviews.

Imagine Android sends a request to create a note.

http

```http
POST /notes
```

The backend creates the note successfully.

But the response is lost because of a network issue.

Android assumes the request failed and retries.

Now the backend receives the same request again.

Without protection, two notes might be created.

This is a duplicate operation problem.

## 9.1 Definition

An operation is idempotent if repeating it with the same input produces the same intended result as performing it once.

In simple words:

**Doing the same operation multiple times should not create unintended additional effects.**

## 9.2 Example: Non-Idempotent Operation

Imagine:

http

```http
POST /payments
```

Every request creates a new payment.

If Android sends the request three times, the user might be charged three times.

This is dangerous.

## 9.3 Solution: Idempotency Key

Android sends a unique key for the logical payment operation.

http

```http
POST /payments
Idempotency-Key: payment-12345
```

Backend:

1. Receives the request.
2. Checks whether the key was already processed.
3. If not, performs the payment.
4. Stores the result associated with the key.
5. If the same key arrives again, returns the previously stored result instead of performing another payment.

Example:

text

```text
REQUEST 1
Key: payment-12345
       |
       v
PROCESS PAYMENT
       |
       v
SAVE RESULT
       |
       v
SUCCESS
```

Retry:

text

```text
REQUEST 2
Key: payment-12345
       |
       v
CHECK EXISTING KEY
       |
       v
ALREADY PROCESSED
       |
       v
RETURN PREVIOUS RESULT
```

## 9.4 HTTP Methods and Idempotency

| HTTP Method | Generally Idempotent? |
| --- | --- |
| GET | Yes |
| PUT | Yes |
| DELETE | Yes |
| POST | Not necessarily |
| PATCH | Not necessarily |

Important:

Idempotency depends on the operation's actual implementation, not just the HTTP method.

For example, a DELETE request should produce the same intended final state even if repeated.

## 9.5 Where Is Idempotency Useful?

- Payment processing.
- Order creation.
- Note creation.
- Message consumption.
- Retry mechanisms.
- Distributed systems.

---

# 10\. What Is a Retry Mechanism?

Imagine your Android application calls the backend.

The backend is temporarily unavailable.

Should the application immediately give up?

Not necessarily.

It can retry the request.

A Retry Mechanism attempts an operation again when a temporary failure occurs.

## 10.1 Simple Retry

text

```text
        API REQUEST
             |
             v
          FAILURE
             |
             v
           RETRY
             |
             v
          SUCCESS
```

But repeatedly retrying immediately can create problems.

Imagine the backend is already overloaded.

Thousands of clients continuously retry.

The server receives even more traffic.

This can make the outage worse.

## 10.2 Exponential Backoff

Exponential Backoff increases the delay between retries.

Example:

| Attempt | Wait Time |
| --- | --- |
| First retry | 1 second |
| Second retry | 2 seconds |
| Third retry | 4 seconds |
| Fourth retry | 8 seconds |

Formula:

$$
Delay = InitialDelay \times 2^{attempt}
$$

This is an illustrative policy; real systems also use maximum delays and jitter.

## 10.3 What Is Jitter?

Jitter adds randomness to retry delays.

Imagine 10,000 Android devices fail at the same time.

Without jitter, all devices might retry after exactly two seconds.

This creates another traffic spike.

With jitter, retry times are spread out.

Example:

text

```text
Device 1: Retry after 2.1 seconds
Device 2: Retry after 2.7 seconds
Device 3: Retry after 2.3 seconds
Device 4: Retry after 2.9 seconds
```

This helps prevent synchronized retry spikes.

## 10.4 When Should We Retry?

Good candidates:

- Temporary network failures.
- HTTP 503 Service Unavailable.
- Temporary connection failures.
- Rate limiting, respecting the server's Retry-After guidance.

Be careful with:

- Invalid requests.
- Authentication failures.
- Permission failures.
- Payment operations without idempotency protection.

---

# 11\. What Is a Circuit Breaker Pattern?

Imagine the Notes service depends on the Notification service.

The Notification service becomes unavailable.

The Notes service continues calling it.

Every request waits for a timeout.

This wastes resources and can affect the Notes service itself.

Circuit Breaker helps prevent repeated calls to a failing dependency.

## 11.1 Definition

Circuit Breaker is a design pattern that temporarily stops requests to a dependency when repeated failures occur.

It has three main states.

1. Closed.
2. Open.
3. Half-Open.

## 11.2 Closed State

Everything is working normally.

Requests are allowed.

text

```text
NOTES SERVICE
      |
      v
NOTIFICATION SERVICE
      |
      v
   SUCCESS
```

## 11.3 Open State

The dependency has experienced too many failures.

The circuit breaker opens.

Requests are rejected or handled through a fallback without calling the failing dependency.

text

```text
NOTES SERVICE
      |
      v
  CIRCUIT BREAKER
      |
      X
      |
 NOTIFICATION SERVICE
```

## 11.4 Half-Open State

After a configured waiting period, the circuit breaker allows a limited number of test requests.

If they succeed, it closes.

If they fail, it opens again.

## 11.5 Circuit Breaker Flow

text

```text
          CLOSED
             |
       TOO MANY FAILURES
             |
             v
           OPEN
             |
       WAIT FOR TIMEOUT
             |
             v
         HALF-OPEN
          /       \
      SUCCESS    FAILURE
         |          |
         v          v
       CLOSED      OPEN
```

## 11.6 Where Is Circuit Breaker Useful?

- Microservices communication.
- Payment service dependency.
- Notification service dependency.
- External API integrations.
- Database or cache dependencies.

## 11.7 Retry vs Circuit Breaker

| Retry | Circuit Breaker |
| --- | --- |
| Tries the operation again | Temporarily stops calling a failing dependency |
| Useful for temporary failures | Useful when failures continue |
| Can increase traffic | Helps prevent repeated traffic to unhealthy services |
| Often uses backoff and jitter | Uses states: Closed, Open, Half-Open |

Important:

Retries and circuit breakers can work together. Retry transient failures within limits, and use a circuit breaker to stop repeated calls when a dependency remains unhealthy.

---

# 12\. What Is a Timeout?

A timeout is the maximum amount of time a system waits for an operation before treating it as failed or unsuccessful.

Example:

Android sends a request.

The server does not respond within 10 seconds.

Android stops waiting and reports a timeout.

There are different timeout types.

## 12.1 Connection Timeout

Maximum time allowed to establish a connection with the server.

## 12.2 Read Timeout

Maximum time allowed while waiting to receive data after a connection has been established.

## 12.3 Write Timeout

Maximum time allowed for sending request data.

## 12.4 Why Are Timeouts Important?

Without timeouts:

- Threads may remain blocked.
- Resources may be occupied.
- Requests may accumulate.
- The application may become slow.

Important:

A client timeout does not prove the server failed to execute the request.

The server may have completed the operation but failed to return the response in time.

This is why idempotency is important.

---

# 13\. How Everything Works Together

Let's design a reliable Notes application.

Requirements:

- Users can create notes.
- Multiple users can edit shared notes.
- Notes should not be duplicated.
- Notifications should be sent asynchronously.
- Temporary service failures should not bring down the entire application.

Architecture:

text

```text
                 ANDROID APP
                      |
                      v
                 NOTES SERVICE
                      |
              +-------+-------+
              |               |
              v               v
          NOTES DB       OUTBOX TABLE
              |               |
              |               v
              |          EVENT PUBLISHER
              |               |
              |               v
              |             KAFKA
              |               |
              |               v
              |        NOTIFICATION SERVICE
              |               |
              |          CIRCUIT BREAKER
              |               |
              |               v
              |              FCM
              |
              v
        VERSION CHECKING
        OPTIMISTIC LOCKING
```

Let's understand the complete flow.

### Scenario 1: User Creates a Note

1. Android sends a create-note request.
2. The request contains an idempotency key.
3. Notes service checks whether the key was already processed.
4. If it is new, the note is saved.
5. The outbox event is saved in the same transaction.
6. The API returns success.

### Scenario 2: Notification Service Is Down

1. Event publisher sends NOTE\_CREATED to Kafka.
2. Notification service is unavailable.
3. The event remains available according to Kafka retention and consumer progress.
4. When the service recovers, it can process pending events.
5. The notification service handles duplicates safely.

### Scenario 3: Two Users Edit the Same Note

1. User A reads version 1.
2. User B reads version 1.
3. User A updates successfully.
4. Note version becomes 2.
5. User B tries to update version 1.
6. The backend detects a conflict.
7. User B receives a conflict response and can refresh or resolve changes.

### Scenario 4: Backend Temporarily Fails

1. Android request fails due to a temporary server problem.
2. The client retries with a bounded exponential backoff.
3. The same idempotency key is reused.
4. Backend recognizes the repeated operation.
5. Duplicate data is not created.

---

# 14\. Android Developer Perspective

These concepts are directly useful when developing reliable Android applications.

## 14.1 Idempotent API Requests

When using Retrofit, understand which requests can safely be retried.

For operations like payment or order creation, use a backend-supported idempotency key.

## 14.2 Retry with Backoff

Don't continuously retry failed APIs.

Use:

- Limited retry attempts.
- Exponential backoff.
- Jitter.
- Appropriate error classification.

## 14.3 Offline-First Architecture

Imagine the user edits a note without internet access.

A good design:

1. Save the note in Room.
2. Mark it as pending synchronization.
3. Use WorkManager to schedule background synchronization.
4. Retry when connectivity is available.
5. Update the sync status after backend confirmation.

## 14.4 Handle Concurrent Updates

Suppose a shared note is updated on another device.

The backend might return a version conflict.

Android should:

- Refresh the latest version.
- Inform the user when necessary.
- Allow conflict resolution.
- Avoid silently overwriting another user's changes.

## 14.5 Timeouts and Network Errors

Handle these separately:

- No internet.
- Connection timeout.
- Server timeout.
- HTTP 500.
- HTTP 503.
- Authentication failure.

Don't treat every API failure as the same error.

---

# 15\. Common Mistakes to Avoid

1. Confusing database consistency with the Consistency part of CAP.
2. Thinking ACID consistency and strong distributed consistency mean exactly the same thing.
3. Assuming every database transaction automatically solves all business consistency problems.
4. Confusing optimistic locking with pessimistic locking.
5. Retrying POST requests without duplicate protection.
6. Assuming exactly-once delivery is automatically guaranteed across external services.
7. Retrying indefinitely.
8. Forgetting jitter in large-scale retry systems.
9. Thinking a circuit breaker is the same as a retry mechanism.
10. Assuming a timeout means the server did not process the request.
11. Ignoring database isolation levels.
12. Assuming eventual consistency is always bad.

---

# 16\. Important Interview Questions and Answers

### Q1. What is Data Consistency?

Data consistency means data follows the correctness rules and consistency guarantees defined by the application.

### Q2. What is Strong Consistency?

After a successful write, subsequent reads observe that write under the system's consistency guarantee.

### Q3. What is Eventual Consistency?

Replicas eventually converge to the same data if updates stop and communication continues normally.

### Q4. What are ACID properties?

Atomicity, Consistency, Isolation and Durability.

### Q5. Explain Atomicity.

A transaction either completes entirely or does not commit any of its changes.

### Q6. Explain Consistency in ACID.

A transaction preserves defined database constraints and business rules.

### Q7. Explain Isolation.

Concurrent transactions are prevented from interfering in ways that violate the selected isolation level.

### Q8. Explain Durability.

Once a transaction is committed, its changes are preserved according to the database's durability guarantees.

### Q9. What is BASE?

Basically Available, Soft State and Eventual Consistency.

### Q10. What is a Database Transaction?

A group of database operations treated as one logical unit of work.

### Q11. What is Dirty Read?

Reading uncommitted data written by another transaction.

### Q12. What is Non-Repeatable Read?

Reading the same row twice and getting different values because another transaction committed an update.

### Q13. What is Phantom Read?

Repeating a query and seeing a different set of rows because another transaction committed changes affecting the result.

### Q14. What are SQL Isolation Levels?

Read Uncommitted, Read Committed, Repeatable Read and Serializable.

### Q15. What is Optimistic Locking?

A concurrency control mechanism that checks whether data has changed before allowing an update, commonly using version numbers.

### Q16. What is Pessimistic Locking?

A mechanism that acquires locks to prevent conflicting operations while a transaction is executing.

### Q17. What is the difference between Optimistic and Pessimistic Locking?

Optimistic locking detects conflicts during updates. Pessimistic locking prevents conflicting updates by acquiring locks earlier.

### Q18. What is Idempotency?

Repeating the same logical operation produces the same intended result without unintended duplicate effects.

### Q19. Why is Idempotency important?

It prevents duplicate operations when clients retry requests or message brokers redeliver messages.

### Q20. What is Exponential Backoff?

A retry strategy that increases the waiting time after each failed attempt.

### Q21. What is Jitter?

Random variation added to retry delays to prevent many clients from retrying simultaneously.

### Q22. What is Circuit Breaker?

A pattern that temporarily blocks calls to an unhealthy dependency after repeated failures.

### Q23. What are the three Circuit Breaker states?

Closed, Open and Half-Open.

### Q24. What is the difference between Retry and Circuit Breaker?

Retry attempts an operation again. Circuit Breaker temporarily stops calls to a dependency that is repeatedly failing.

### Q25. What is a Timeout?

The maximum time a system waits for an operation before treating it as unsuccessful.

### Q26. Why are Timeouts important?

They prevent requests from waiting indefinitely and consuming resources.

### Q27. What happens if Android times out but the server successfully processes the request?

Android may retry. Idempotency protection ensures the same logical operation does not execute twice.

### Q28. What is the Transactional Outbox Pattern?

A pattern that saves business data and its event record in the same database transaction, then publishes the event asynchronously.

### Q29. How would you prevent two users from overwriting the same note?

Use optimistic locking with version numbers or another suitable concurrency control strategy.

### Q30. How would you design a reliable payment API?

Use database transactions, idempotency keys, appropriate concurrency control, bounded retries, timeouts and reliable payment-state tracking.

---

# 17\. Practical Interview Exercise

Imagine you are designing a collaborative Notes application.

Requirements:

- 1 million users.
- Users can edit notes from multiple devices.
- Users can share notes.
- Multiple users can edit the same note.
- Notifications are sent asynchronously.
- Users can work offline.
- Network failures can occur.

Answer these questions:

1. How would you prevent duplicate note creation?
2. How would you handle two users editing the same note?
3. Would you use optimistic or pessimistic locking?
4. How would you handle offline updates?
5. What happens if the notification service is unavailable?
6. How would you retry failed API requests?
7. Why is exponential backoff useful?
8. What is jitter?
9. How would you prevent a failing service from affecting other services?
10. What is the difference between a timeout and a server failure?
11. What are ACID properties?
12. Which operations need strong consistency?
13. Which operations can tolerate eventual consistency?
14. How would you ensure reliable event publication?
15. How would you prevent duplicate message processing?

Try answering these questions without referring to the notes.

---

# 18\. Day 12 – Quick Revision

| Concept | Simple Meaning |
| --- | --- |
| Data Consistency | Data follows correctness rules |
| Strong Consistency | Reads observe completed writes |
| Eventual Consistency | Replicas eventually converge |
| ACID | Atomicity, Consistency, Isolation, Durability |
| BASE | Basically Available, Soft State, Eventual Consistency |
| Transaction | Group of operations treated as one unit |
| Commit | Save transaction changes |
| Rollback | Undo uncommitted changes |
| Dirty Read | Read uncommitted data |
| Non-Repeatable Read | Same row returns different values |
| Phantom Read | Repeated query returns a changed set of rows |
| Optimistic Locking | Detect conflicts using versions |
| Pessimistic Locking | Lock data to prevent conflicting operations |
| Idempotency | Avoid unintended duplicate effects |
| Retry | Attempt an operation again |
| Exponential Backoff | Increase retry delay progressively |
| Jitter | Randomize retry delays |
| Circuit Breaker | Stop calling an unhealthy dependency |
| Timeout | Maximum waiting time |
| Transactional Outbox | Reliably publish events from database transactions |

---

# 19\. Final Learning Checklist

- [ ]Understand Data Consistency.
- [ ]Understand Strong Consistency.
- [ ]Understand Eventual Consistency.
- [ ]Understand ACID Properties.
- [ ]Understand BASE Properties.
- [ ]Understand Database Transactions.
- [ ]Understand Commit and Rollback.
- [ ]Understand Isolation Levels.
- [ ]Understand Dirty Reads.
- [ ]Understand Non-Repeatable Reads.
- [ ]Understand Phantom Reads.
- [ ]Understand Optimistic Locking.
- [ ]Understand Pessimistic Locking.
- [ ]Understand Idempotency.
- [ ]Understand Retry Mechanisms.
- [ ]Understand Exponential Backoff.
- [ ]Understand Jitter.
- [ ]Understand Circuit Breaker.
- [ ]Understand Timeouts.
- [ ]Understand Transactional Outbox.
- [ ]Explain a reliable Notes application architecture.

---

# 20\. What Is Next?

## Week 2 – Day 13: Rate Limiting and API Gateway

Tomorrow we will learn:

1. What is Rate Limiting?
2. Why do we need Rate Limiting?
3. Requests per second.
4. Token Bucket Algorithm.
5. Leaky Bucket Algorithm.
6. Fixed Window Counter.
7. Sliding Window Counter.
8. What is an API Gateway?
9. Responsibilities of an API Gateway.
10. Authentication and Authorization.
11. Load Balancer vs API Gateway.
12. Rate Limiting in a Notes application.
13. Interview questions.

**Day 12 Complete – Consistency and Reliability.**

## Today's five most important concepts

Make sure you can explain these without memorizing definitions:

1. ACID: How databases maintain reliable transactions.
2. Optimistic Locking: How to prevent two users from overwriting each other's updates.
3. Idempotency: How to prevent duplicate operations when an API request is retried.
4. Exponential Backoff: How to retry failed requests without overwhelming the server.
5. Circuit Breaker: How to protect one service from another service that keeps failing.

My recommendation: Spend extra time on Optimistic Locking and Idempotency. Both are especially relevant to your Android experience because they directly affect API integration, offline synchronization, retries and multi-device data updates.
