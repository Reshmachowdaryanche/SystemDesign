# Week 2 – Day 9: Database Scaling

Week 2Day 9Beginner level

File name: `Week-2-Day-9-Database-Scaling.md`

Today's goal: Understand how databases handle increasing users, requests and data without becoming slow or unavailable.

Yesterday, we learned how to distribute requests across multiple backend servers using a Load Balancer.

Today, we'll understand an important problem:

What happens when we add more backend servers, but all of them depend on the same database?

This is one of the most important concepts in system design interviews.

Writing

# Week 2 – Day 9: Database Scaling

**Topic:** Scalability and Distributed Systems\
**Level:** Beginner\
**Target:** Android Developer – 7 Years Experience

---

# 1\. First, Understand What a Database Is

Before learning database scaling, let's understand what a database does.

Imagine we are building a Notes application.

Users can:

- Create notes.
- Read notes.
- Update notes.
- Delete notes.

Where should we store these notes?

We need a database.

## 1.1 What Is a Database?

A database is an organized collection of data that allows applications to store, retrieve, update and delete information.

For example, our Notes application might store:

**Users Table**

| user\_id | name | email |
| --- | --- | --- |
| 1 | John | john@gmail.com |
| 2 | David | david@gmail.com |

**Notes Table**

| note\_id | user\_id | title | content |
| --- | --- | --- | --- |
| 101 | 1 | Meeting | Discuss project |
| 102 | 1 | Shopping | Buy groceries |
| 103 | 2 | Interview | Prepare system design |

When an Android application requests notes, the backend retrieves the required records from the database.

Architecture:

text

```text
ANDROID APPLICATION
        |
        | HTTP Request
        v
   BACKEND SERVER
        |
        | Database Query
        v
      DATABASE
        |
        | Query Result
        v
   BACKEND SERVER
        |
        | HTTP Response
        v
ANDROID APPLICATION
```

## 1.2 What Happens Inside a Database?

A database performs several operations:

1. Stores data.
2. Retrieves data.
3. Updates existing data.
4. Deletes data.
5. Maintains data consistency.
6. Handles concurrent requests.
7. Manages transactions.

Every operation consumes resources such as CPU, RAM, storage and network capacity.

As users and requests increase, the database workload also increases.

---

# 2\. What Is Database Scaling?

Imagine our application initially has:

- 1,000 users.
- 1 backend server.
- 1 database server.

Everything works smoothly.

After one year:

- 1 million users.
- 10 backend servers.
- Still only 1 database server.

Now all 10 backend servers send database queries to the same database.

text

```text
           ANDROID USERS
                 |
                 v
           LOAD BALANCER
                 |
        +--------+--------+
        |        |        |
        v        v        v
      API 1    API 2    API 3
        |        |        |
        +--------+--------+
                 |
                 v
          SINGLE DATABASE
```

The database receives requests from all backend servers.

Eventually:

- Query execution becomes slower.
- CPU usage increases.
- Memory usage increases.
- Database connections become exhausted.
- Queries start waiting.
- Application response time increases.

The database becomes a bottleneck.

## Definition

**Database Scaling is the process of increasing a database system's capacity to handle more users, queries and data while maintaining acceptable performance and reliability.**

There are two major approaches:

1. Vertical Scaling.
2. Horizontal Scaling.

---

# 3\. Vertical Database Scaling

## 3.1 What Is Vertical Scaling?

Vertical scaling means increasing the resources of an existing database server.

For example, initially:

| Resource | Initial Server |
| --- | --- |
| CPU | 2 cores |
| RAM | 8 GB |
| Storage | 100 GB |

As traffic increases, we upgrade the server.

| Resource | Upgraded Server |
| --- | --- |
| CPU | 16 cores |
| RAM | 64 GB |
| Storage | 1 TB |

We are making the same database server more powerful.

Architecture:

text

```text
       BACKEND SERVERS
              |
              v
       DATABASE SERVER
              |
       +------+------+
       |             |
       v             v
    More CPU      More RAM
```

## 3.2 Advantages

- Simple to understand.
- No need to distribute data across multiple database servers.
- Existing application architecture may need fewer changes.
- Easier to manage initially.

## 3.3 Disadvantages

- Hardware has limitations.
- Powerful servers can become expensive.
- There is still a dependency on one database server.
- Upgrades may require maintenance or downtime depending on the infrastructure.

## 3.4 Real-World Example

Imagine a Notes application has 10,000 active users.

One database server can handle the workload.

As the number of users increases to 100,000, the database becomes slower.

We upgrade CPU, RAM and storage.

This is vertical scaling.

**Remember:** Vertical scaling means making one machine more powerful.

---

# 4\. Horizontal Database Scaling

## 4.1 What Is Horizontal Scaling?

Horizontal scaling means adding multiple database servers to distribute database workload or data.

Instead of continuously upgrading one database server, we introduce additional database servers.

There are different ways to achieve this.

Two important approaches are:

1. Replication.
2. Sharding.

We will understand both separately.

---

# 5\. Database Replication

## 5.1 What Is Database Replication?

Database replication means maintaining copies of database data across multiple database servers.

Why do we need copies?

Imagine our Notes application has 1 million users.

Most users frequently read their notes.

If every read request goes to one database server, that database may become overloaded.

We can create additional copies of the data.

Architecture:

text

```text
                BACKEND
                   |
                   v
             PRIMARY DB
                   |
          +--------+--------+
          |                 |
          v                 v
       REPLICA 1         REPLICA 2
```

The primary database handles writes and sends changes to the replicas.

The replicas maintain copies of the primary database data.

This is called database replication.

## 5.2 Why Do We Need Replication?

1. Improve read performance.
2. Distribute read traffic.
3. Improve availability.
4. Provide additional copies of data.
5. Support recovery strategies.

---

# 6\. Primary-Replica Architecture

This is a very important database architecture.

You may also hear these terms:

- Primary-Replica.
- Leader-Follower.
- Master-Slave (older terminology).

We will use Primary-Replica.

## 6.1 What Is a Primary Database?

The Primary database is the main database that accepts write operations in a typical primary-replica setup.

Write operations include:

- INSERT.
- UPDATE.
- DELETE.

Example:

A user creates a new note.

The backend sends the insert query to the primary database.

sql

```sql
INSERT INTO notes (user_id, title, content)
VALUES (1, 'Meeting', 'Discuss project');
```

The primary database stores the new note.

## 6.2 What Is a Replica Database?

A Replica database maintains a copy of the primary database data.

It commonly handles read operations.

Example:

A user opens the Notes application and requests all notes.

The backend can retrieve notes from a replica.

sql

```sql
SELECT * FROM notes
WHERE user_id = 1;
```

Architecture:

text

```text
                  BACKEND
                     |
              +------+------+
              |             |
              v             v
           WRITE           READ
              |             |
              v             v
          PRIMARY DB    REPLICA DB
              |
              |
        Replication
              |
              v
          REPLICA DB
```

## 6.3 Complete Example

Let's consider a Notes application.

User creates a note.

text

```text
ANDROID APP
     |
     v
BACKEND SERVER
     |
     v
PRIMARY DATABASE
     |
     | Save Note
     |
     v
REPLICATION
     |
     v
REPLICA DATABASE
```

User reads notes.

text

```text
ANDROID APP
     |
     v
BACKEND SERVER
     |
     v
REPLICA DATABASE
     |
     v
NOTES RESPONSE
```

This way, write and read workloads can be distributed.

## 6.4 Read Replica

A read replica is a database copy primarily used to serve read queries.

For example:

- Primary DB: Handles 20,000 writes.
- Replica 1: Handles 50,000 reads.
- Replica 2: Handles 50,000 reads.

These numbers are only illustrative.

The actual capacity depends on hardware, queries, database configuration and data size.

## 6.5 Important Problem: Replication Lag

Here is an important problem.

Imagine:

1. User creates a note.
2. The note is saved in the primary database.
3. The replica has not received the latest update yet.
4. User immediately requests the note.
5. The read goes to the replica.
6. The replica does not return the newly created note.

Why?

Because replication takes some time.

This delay is called **Replication Lag**.

Architecture:

text

```text
          PRIMARY DB
              |
         New Note Saved
              |
              | Replication Delay
              |
              v
          REPLICA DB
       Old Data Temporarily
```

This can result in stale reads.

## 6.6 How Can We Handle Replication Lag?

Some common approaches:

1. Read immediately after writing from the primary database.
2. Use session-based read consistency where supported.
3. Wait until the required replication position is reached.
4. Use replicas only for queries where slightly stale data is acceptable.

Example:

After creating a note, the application can fetch the latest note from the primary database rather than immediately reading from a replica.

**Interview point:** Replication improves read scalability, but replicas may not always contain the latest data.

---

# 7\. Database Failover

What happens if the primary database crashes?

In a primary-replica setup, the application may not be able to perform writes.

A failover mechanism can promote a healthy replica to become the new primary.

## 7.1 What Is Failover?

Failover is the process of switching to a standby or replica system when the primary system becomes unavailable.

Example:

Initially:

text

```text
       PRIMARY DB
            |
       REPLICATION
            |
            v
        REPLICA DB
```

Primary database crashes:

text

```text
       PRIMARY DB
          DOWN

        REPLICA DB
             |
             v
       PROMOTED TO
        NEW PRIMARY
```

The application can resume writing after the new primary is selected and routing is updated.

## 7.2 Important Considerations

Failover is not always instantaneous.

We need to consider:

- How much data was replicated before failure?
- Could recent writes be lost?
- How is the new primary selected?
- How do backend servers discover the new primary?
- How do we prevent two primaries from accepting writes simultaneously?

A system needs appropriate replication and failover mechanisms to handle these problems.

---

# 8\. Database Sharding

Now let's understand another important concept.

Replication creates copies of the same data.

But what if the actual amount of data becomes too large for one database server?

For example:

- 100 million users.
- 10 billion notes.
- Huge amounts of historical data.

Creating multiple copies does not necessarily solve the storage and write-capacity limitations of one database.

We need to divide the data.

This is where sharding comes in.

## 8.1 What Is Database Sharding?

Database sharding means dividing a large database into smaller independent parts called shards.

Each shard stores a portion of the total data.

For example, imagine we have 9 users.

Instead of storing all users in one database, we distribute them across three database shards.

text

```text
             ALL USERS
                 |
        +--------+--------+
        |        |        |
        v        v        v
      SHARD 1  SHARD 2  SHARD 3
      Users    Users    Users
      1,2,3    4,5,6    7,8,9
```

Each shard stores only a portion of the complete dataset.

## 8.2 Why Do We Need Sharding?

1. Handle very large datasets.
2. Distribute write traffic.
3. Distribute storage requirements.
4. Scale databases horizontally.
5. Reduce the workload handled by an individual shard.

---

# 9\. Sharding Strategies

How do we decide which data goes into which shard?

There are different strategies.

Let's understand three common approaches.

## 9.1 Range-Based Sharding

Data is divided based on a range of values.

Example:

We have user IDs from 1 to 900.

text

```text
SHARD 1: User IDs 1 - 300

SHARD 2: User IDs 301 - 600

SHARD 3: User IDs 601 - 900
```

Advantages:

- Simple to understand.
- Range-based queries can be efficient.

Disadvantages:

- Uneven distribution is possible.
- Some ranges may receive significantly more traffic.

Example:

If most new users have IDs between 801 and 900, Shard 3 may become overloaded.

This is called a hot shard problem.

## 9.2 Hash-Based Sharding

Data is distributed using a hash function.

For example:

text

```text
shard_number = hash(user_id) % 3
```

The hash function determines which shard should store the user.

Example:

text

```text
User ID 101 -> Shard 2
User ID 102 -> Shard 1
User ID 103 -> Shard 3
User ID 104 -> Shard 2
```

Advantages:

- Can distribute data more evenly.
- Reduces dependence on sequential ranges.

Disadvantages:

- Range queries can be harder.
- Adding or removing shards may require data redistribution, depending on the hashing strategy.

## 9.3 Geographic Sharding

Data is distributed based on geographical location.

Example:

text

```text
INDIAN USERS
     |
     v
  INDIA DB

US USERS
     |
     v
   US DB

EUROPEAN USERS
     |
     v
  EUROPE DB
```

Advantages:

- Data can be stored closer to users.
- Can help with regional performance and data residency requirements.

Disadvantages:

- Cross-region queries can become complicated.
- Cross-region synchronization introduces complexity.

---

# 10\. What Is a Shard Key?

A shard key is the field used to determine which shard stores a particular record.

Example:

Our Notes application has this table:

| note\_id | user\_id | title |
| --- | --- | --- |
| 101 | 1 | Meeting |
| 102 | 1 | Shopping |
| 103 | 2 | Interview |
| 104 | 3 | Project |

We decide to shard based on `user_id`.

Then all notes belonging to a particular user can be stored in the same shard.

For example:

text

```text
USER 1 -> SHARD 1
USER 2 -> SHARD 2
USER 3 -> SHARD 3
```

When the user requests their notes, the backend can determine which shard contains their data.

## 10.1 Why Is Choosing a Shard Key Important?

A poor shard key can cause:

- Uneven data distribution.
- Uneven traffic distribution.
- Hot shards.
- Complex queries.
- Difficult migrations.

For a Notes application, `user_id` can be a reasonable shard key when most queries access notes belonging to a particular user.

But the best choice depends on query patterns and expected growth.

---

# 11\. What Is Partitioning?

Partitioning means dividing a large table into smaller logical parts called partitions.

It is commonly managed within a database system.

Example:

Imagine our Notes application stores 100 million notes in one large table.

We can partition the table based on creation year.

text

```text
            NOTES TABLE
                 |
        +--------+--------+
        |        |        |
        v        v        v
      2024     2025     2026
     PARTITION PARTITION PARTITION
```

Each partition contains records belonging to a particular year.

For example:

- Partition 1: Notes created in 2024.
- Partition 2: Notes created in 2025.
- Partition 3: Notes created in 2026.

When querying notes from 2026, the database may scan only the relevant partition instead of the entire table.

This is called partition pruning.

## 11.1 Types of Partitioning

### Range Partitioning

Records are divided based on value ranges.

Example: Creation dates.

### List Partitioning

Records are divided based on a predefined list of values.

Example:

- India.
- USA.
- UK.

### Hash Partitioning

Records are distributed using a hash function.

Example:

`hash(user_id)`

## 11.2 Partitioning vs Sharding

These concepts are related but not identical.

| Partitioning | Sharding |
| --- | --- |
| Divides a table or dataset into smaller parts | Distributes data across independent database instances |
| Often managed within one database system | Usually involves multiple database servers |
| Can improve query performance and data management | Helps distribute storage and workload |
| May exist on one database server | Designed for horizontal distribution |

**Important:** Sharding can be understood as a form of horizontal partitioning across database instances.

---

# 12\. Replication vs Sharding

This is one of the most important interview questions.

Both approaches involve multiple database servers, but they solve different problems.

## Replication

Replication creates copies of the same data.

Example:

text

```text
          DATABASE
              |
       +------+------+
       |             |
       v             v
    REPLICA 1     REPLICA 2
   Same Data     Same Data
```

Main purpose:

- Improve read scalability.
- Improve availability.
- Maintain additional data copies.

## Sharding

Sharding divides data across multiple database servers.

Example:

text

```text
       COMPLETE DATASET
               |
       +-------+-------+
       |       |       |
       v       v       v
     SHARD 1 SHARD 2 SHARD 3
      Part A  Part B  Part C
```

Main purpose:

- Distribute storage.
- Distribute write workload.
- Handle large datasets.

## Comparison

| Replication | Sharding |
| --- | --- |
| Copies the same data | Divides different data |
| Primarily helps read scalability | Helps storage and write scalability |
| Has replication lag considerations | Has shard routing considerations |
| Helps with availability | Requires careful shard-level availability design |
| Data exists on multiple replicas | Each shard owns a subset of data |

**Interview shortcut:**

Replication = Same data in multiple places.

Sharding = Different data in different places.

---

# 13\. Can We Combine Replication and Sharding?

Yes.

Large-scale systems often combine both approaches.

Imagine we have three shards.

Each shard has one primary and two replicas.

text

```text
                   APPLICATION
                        |
                  SHARD ROUTER
                        |
          +-------------+-------------+
          |             |             |
          v             v             v

       SHARD 1       SHARD 2       SHARD 3
          |             |             |
     +----+----+   +----+----+   +----+----+
     |         |   |         |   |         |
     v         v   v         v   v         v
   PRIMARY  REPLICA PRIMARY REPLICA PRIMARY REPLICA
             1              1              1
       REPLICA 2      REPLICA 2      REPLICA 2
```

Each shard stores a different portion of the complete dataset.

Within each shard, replicas maintain copies of that shard's data.

This architecture provides:

- Horizontal data distribution.
- Read scaling within each shard.
- Additional availability options.

It also introduces complexity in routing, failover and consistency.

---

# 14\. Database Indexing – A Quick Connection

We should also understand why indexing matters before scaling databases.

Imagine a Notes table has 10 million records.

You want to find all notes belonging to user ID 500.

Without a suitable index, the database may need to scan many records.

This is called a Full Table Scan.

An index helps the database locate records more efficiently.

Example:

sql

```sql
CREATE INDEX idx_notes_user_id
ON notes(user_id);
```

Now queries filtering by `user_id` can potentially use this index.

Example:

sql

```sql
SELECT *
FROM notes
WHERE user_id = 500;
```

Important:

- Indexes improve certain read queries.
- Indexes consume storage.
- Indexes can increase the cost of INSERT, UPDATE and DELETE operations.
- Indexes do not automatically solve every database performance problem.

We will study indexing in more detail later.

---

# 15\. Real-World Notes Application Architecture

Let's combine today's concepts.

Assume:

- 1 million registered users.
- 100 million notes.
- Multiple backend servers.
- Many read operations.
- Regular note creation and updates.

Architecture:

text

```text
                ANDROID USERS
                      |
                      v
                LOAD BALANCER
                      |
             +--------+--------+
             |        |        |
             v        v        v
           API 1    API 2    API 3
             |        |        |
             +--------+--------+
                      |
                      v
                 SHARD ROUTER
                      |
            +---------+---------+
            |         |         |
            v         v         v
          SHARD 1   SHARD 2   SHARD 3
            |         |         |
         PRIMARY   PRIMARY   PRIMARY
            |         |         |
          REPLICA   REPLICA   REPLICA
            |         |         |
          REPLICA   REPLICA   REPLICA
```

Example flow:

**User creates a note:**

1. Android sends a create-note API request.
2. Load balancer selects a backend server.
3. Backend identifies the user's shard using the shard key.
4. Backend writes the note to that shard's primary database.
5. Replication propagates the change to its replicas.
6. Backend returns the result to Android.

**User reads notes:**

1. Android sends a get-notes API request.
2. Load balancer selects a backend server.
3. Backend identifies the user's shard.
4. Backend reads from an appropriate replica when consistency requirements allow it.
5. Backend returns the notes.

This is a simplified architecture. Real systems also need connection management, monitoring, transactions, failover and replication consistency.

---

# 16\. Android Developer Perspective

You may not directly configure database sharding or replication as an Android developer.

However, these concepts affect how your Android application behaves.

## 16.1 Stale Data

If a read request goes to a lagging replica, the Android app may receive older data.

Example:

- User creates a note.
- Server successfully saves it.
- User refreshes immediately.
- Replica has not received the update.
- New note is temporarily missing.

The backend should handle read-after-write consistency appropriately.

## 16.2 Pagination

When the database stores millions of notes, the backend should not return every note in one API response.

Instead, it should return paginated results.

Example:

http

```http
GET /notes?page=1&limit=20
```

For large or frequently changing datasets, cursor-based pagination may be preferable.

## 16.3 Offline-First Architecture

In Android, Room can store a local copy of notes.

The application can display local data while the backend synchronizes changes.

Example:

text

```text
        ANDROID APP
              |
              v
          ROOM DB
              |
              v
         REPOSITORY
              |
              v
          API LAYER
              |
              v
        BACKEND SERVER
              |
              v
          DATABASE
```

The Android application should not need to know whether the backend uses replication or sharding.

That complexity should generally remain behind the backend API.

---

# 17\. Common Database Scaling Problems

| Problem | Explanation |
| --- | --- |
| Replication Lag | Replica data is behind primary data |
| Hot Shard | One shard receives too much traffic |
| Uneven Distribution | Data is not distributed evenly |
| Failover Complexity | Promoting a new primary requires coordination |
| Connection Exhaustion | Too many connections reach the database |
| Cross-Shard Query | A query requires data from multiple shards |
| Data Migration | Moving data between shards can be complex |
| Index Overhead | Indexes consume storage and slow some writes |

---

# 18\. Important Interview Questions and Answers

### Q1. What is Database Scaling?

Increasing database capacity to handle growing users, requests and data.

### Q2. What are the two major approaches to database scaling?

Vertical scaling and horizontal scaling.

### Q3. What is Vertical Scaling?

Increasing the resources of an existing database server, such as CPU, RAM and storage.

### Q4. What is Horizontal Scaling?

Adding database servers to distribute workload or data.

### Q5. What is Database Replication?

Maintaining copies of database data across multiple database servers.

### Q6. What is Primary-Replica Architecture?

An architecture where the primary database typically handles writes and replicas maintain copies of its data, often serving read queries.

### Q7. What is a Read Replica?

A copy of a database that is primarily used to serve read requests.

### Q8. What is Replication Lag?

The delay between a change being committed on the primary database and becoming available on a replica.

### Q9. What is Database Failover?

Switching database operations to a healthy standby or replica when the primary database fails.

### Q10. What is Database Sharding?

Dividing a large dataset into smaller parts and distributing those parts across multiple database servers.

### Q11. What is a Shard Key?

The field used to determine which shard should store a particular record.

### Q12. What is a Hot Shard?

A shard that receives significantly more requests or workload than other shards.

### Q13. What is Database Partitioning?

Dividing a large table or dataset into smaller logical parts to improve management and potentially query performance.

### Q14. What is the difference between Sharding and Replication?

Replication creates copies of the same data. Sharding distributes different portions of the data across multiple database instances.

### Q15. Can Replication and Sharding be used together?

Yes. Each shard can have its own primary and replicas.

### Q16. Does replication improve write performance?

Not necessarily. In a traditional primary-replica architecture, writes still go through the primary. Replication primarily helps read scaling and availability.

### Q17. Why is sharding more complex than replication?

Because the application or routing layer must identify the correct shard, and cross-shard queries and transactions become more complicated.

### Q18. Why do we need database indexing?

Indexes help databases locate records efficiently without scanning every record in many query scenarios.

---

# 19\. Practical Interview Exercise

Imagine you are designing a Notes application.

Initially:

- 10,000 users.
- 1 database server.
- 1 million notes.

After two years:

- 10 million users.
- 1 billion notes.
- Database response time has increased.
- Read traffic is much higher than write traffic.

Answer these questions:

1. Would you choose vertical or horizontal scaling?
2. Would you introduce read replicas?
3. How would you handle replication lag?
4. When would you introduce sharding?
5. Which shard key would you choose for a Notes application?
6. What happens if one shard becomes overloaded?
7. Can you combine replication and sharding?
8. What happens if the primary database fails?
9. How would you handle reads immediately after a note is created?
10. How would indexing help?

Try to explain your architecture using a diagram.

---

# 20\. Day 9 – Quick Revision

| Concept | Remember This |
| --- | --- |
| Database Scaling | Increasing database capacity |
| Vertical Scaling | Upgrade one database server |
| Horizontal Scaling | Add more database servers |
| Replication | Copy the same data |
| Primary | Main database handling writes |
| Replica | Copy of primary data |
| Read Replica | Serves read traffic |
| Replication Lag | Delay in copying changes |
| Failover | Switch to a healthy database |
| Sharding | Divide data across databases |
| Shard Key | Determines where data belongs |
| Hot Shard | One shard gets excessive workload |
| Partitioning | Divide a table into smaller logical parts |
| Indexing | Speed up suitable database queries |

---

# 21\. Final Learning Checklist

- [ ]Understand what a database is.
- [ ]Understand why databases become bottlenecks.
- [ ]Understand vertical database scaling.
- [ ]Understand horizontal database scaling.
- [ ]Understand replication.
- [ ]Understand Primary-Replica architecture.
- [ ]Understand Read Replicas.
- [ ]Understand replication lag.
- [ ]Understand database failover.
- [ ]Understand sharding.
- [ ]Understand range-based sharding.
- [ ]Understand hash-based sharding.
- [ ]Understand geographic sharding.
- [ ]Understand shard keys.
- [ ]Understand partitioning.
- [ ]Understand Replication vs Sharding.
- [ ]Understand how indexing helps.
- [ ]Explain the Notes application database architecture.

---

# 22\. What Is Next?

## Week 2 – Day 10: Distributed Systems Basics

Tomorrow we will learn:

1. What is a Distributed System?
2. Why do we need Distributed Systems?
3. How do multiple servers communicate?
4. What are network failures?
5. What are partial failures?
6. What are the challenges of distributed systems?
7. What is the CAP Theorem?
8. Consistency vs Availability.
9. Real-world examples.
10. Interview questions.

**Day 9 Complete – Database Scaling.**

## Today's most important concepts

Before moving to Day 10, make sure you understand these four differences:

| Concept | Simple explanation |
| --- | --- |
| Vertical scaling | Make one database server more powerful |
| Replication | Keep copies of the same data |
| Sharding | Divide different data across servers |
| Partitioning | Divide a large table into smaller logical parts |

My suggestion: Spend extra time understanding Replication vs Sharding. This is a fundamental system design interview topic, and it will make Day 10 (Distributed Systems) much easier.
