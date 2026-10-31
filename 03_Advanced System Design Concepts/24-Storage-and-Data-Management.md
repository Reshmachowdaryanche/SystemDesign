# Week 4 – Day 24: Storage and Data Management

File name: `24-Storage-and-Data-Management.md`

Topic: Advanced System Design and Database Architecture Level: Beginner to Intermediate Target: Senior Android Developer – 7 Years Experience

Today's goal: Understand how databases store, retrieve and organize data internally, how indexing improves query performance, how transactions maintain data correctness, and when to use SQL databases, NoSQL databases, blob storage, object storage and CDNs.

Today's notes start from the basics. You don't need previous database administration experience.

# 1. First, Understand What Storage and Data Management Mean

Imagine you are building a Notes application.

Your application supports:

- User registration and login.
- Creating and updating notes.
- Searching notes.
- Uploading images and documents.
- Sharing notes.
- Synchronizing data across devices.

Where should we store all this information?

Consider the following data:

| Data | Example |
| --- | --- |
| User information | Name, email, user ID |
| Notes | Title, content, creation date |
| Attachments | Images, PDFs, documents |
| Search information | Note titles and content |
| User activity | Last opened time, edit history |

Can we store everything in one database?

Technically, we can store many types of data in one database, but it may not be the best architecture.

Different types of data have different storage requirements.

For example:

- User information needs structured storage.
- Notes need reliable database storage.
- Images and PDFs need file/object storage.
- Search needs efficient indexing.
- Frequently accessed data may need caching.

This is where Storage and Data Management become important.

## 1.1 What Is Data Management?

Data management is the process of storing, organizing, retrieving, updating, protecting and maintaining application data.

A good storage architecture should consider:

1. Performance.
2. Scalability.
3. Reliability.
4. Data consistency.
5. Security.
6. Storage cost.
7. Data availability.

# 2. What Is a Database Internally?

Before understanding indexing and B-Trees, let's understand how a database stores information.

Imagine we have a Users table.

| ID | Name | Email |
| --- | --- | --- |
| 101 | Rahul | rahul@gmail.com |
| 102 | Amit | amit@gmail.com |
| 103 | Priya | priya@gmail.com |
| 104 | John | john@gmail.com |

We execute:

sql

```sql
SELECT * FROM users WHERE id = 103;
```

The database needs to find the row where ID is 103.

How does it do that?

There are different approaches.

## 2.1 Full Table Scan

A full table scan means the database examines rows to find matching records.

Imagine the database contains one million users.

Without a suitable index, it may need to examine many or all rows to find the requested record.

text

```text
             USERS TABLE

        +------------------+
        | ID 101           |
        | ID 102           |
        | ID 103  <- FOUND |
        | ID 104           |
        | ID 105           |
        | ...              |
        | ID 1000000       |
        +------------------+
```

This can become expensive for large tables.

Important:

A full table scan is not always bad. For queries that return a large percentage of a table, scanning the table can sometimes be more efficient than using an index.

## 2.2 How Does a Database Store Data?

A database typically manages data using pages or blocks.

A page is a fixed-size unit of storage that the database reads from or writes to.

For example:

text

```text
           DATABASE

       +----------------+
       |    PAGE 1      |
       |                |
       | Row 1          |
       | Row 2          |
       | Row 3          |
       +----------------+

       +----------------+
       |    PAGE 2      |
       |                |
       | Row 4          |
       | Row 5          |
       | Row 6          |
       +----------------+

       +----------------+
       |    PAGE 3      |
       |                |
       | Row 7          |
       | Row 8          |
       +----------------+
```

Instead of treating every row as an independent disk operation, databases organize data into pages.

This helps optimize disk I/O.

## 2.3 What Is Disk I/O?

Disk I/O means reading data from or writing data to storage devices.

For example:

1. Application executes a query.
2. Database checks whether the required page is in memory.
3. If not, it may read the page from disk.
4. Database processes the requested data.
5. Database returns the result.

Disk access is generally more expensive than accessing data already in memory.

Therefore, databases use techniques such as:

- Buffer pools.
- Indexes.
- Query optimization.
- Caching.
- Efficient page organization.

# 3. What Is Database Indexing?

This is one of the most important database interview topics.

Imagine you have a physical book containing 1,000 pages.

You want to find the topic "Database Indexing".

Would you read all 1,000 pages?

Probably not.

You would look at the index at the end of the book.

The index tells you where the topic is located.

A database index works similarly.

## 3.1 Definition

A database index is a data structure that helps the database find records efficiently without scanning the entire table.

Example:

Users table:

| ID | Name | Email |
| --- | --- | --- |
| 101 | Rahul | rahul@gmail.com |
| 102 | Amit | amit@gmail.com |
| 103 | Priya | priya@gmail.com |
| 104 | John | john@gmail.com |

Suppose we frequently search users by email.

sql

```sql
SELECT *
FROM users
WHERE email = 'priya@gmail.com';
```

We can create an index:

sql

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Now the database can use the index to locate matching records more efficiently.

## 3.2 How Does an Index Work?

A simplified index might look like this:

text

```text
          EMAIL INDEX

     amit@gmail.com ------> Row 102

     john@gmail.com ------> Row 104

     priya@gmail.com -----> Row 103

     rahul@gmail.com -----> Row 101
```

The index stores searchable key values and references to the corresponding records or storage locations.

The exact implementation depends on the database and index type.

## 3.3 Without Index vs With Index

Without index

Full table scantext

```text
Query
  |
  v
Check Row 1
  |
  v
Check Row 2
  |
  v
Check Row 3
  |
  v
Continue scanning...
  |
  v
Find matching row
```

With index

Index lookuptext

```text
Query
  |
  v
Search Index
  |
  v
Find matching key
  |
  v
Locate row
  |
  v
Return result
```

## 3.4 Advantages of Indexing

1. Faster data retrieval.
2. Efficient filtering.
3. Faster joins in suitable query patterns.
4. Efficient sorting and range queries for suitable indexes.
5. Can enforce uniqueness using unique indexes.

## 3.5 Disadvantages of Indexing

Indexes are not free.

1. Additional storage space is required.
2. INSERT operations may become slower.
3. UPDATE operations may become slower.
4. DELETE operations may require index maintenance.
5. Too many indexes can increase database maintenance overhead.

Why?

Imagine inserting a new user.

The database must:

- Insert the user record.
- Update the email index.
- Update any other indexes affected by the insert.

Therefore, we should create indexes based on actual query requirements.

## 3.6 When Should We Create Indexes?

Consider our Notes application.

Frequently executed queries:

sql

```sql
SELECT *
FROM notes
WHERE user_id = 101;
```

sql

```sql
SELECT *
FROM notes
WHERE updated_at > '2026-01-01';
```

sql

```sql
SELECT *
FROM users
WHERE email = 'rahul@gmail.com';
```

Potential indexes:

sql

```sql
CREATE INDEX idx_notes_user_id
ON notes(user_id);

CREATE INDEX idx_notes_updated_at
ON notes(updated_at);

CREATE INDEX idx_users_email
ON users(email);
```

Important:

An index should be selected based on query patterns, data distribution and query plans. Creating an index for every column is not a good practice.

# 4. What Is a B-Tree?

Now let's understand an important data structure used by many database indexes.

B-Tree stands for Balanced Tree.

It is a tree-based data structure that keeps keys organized and supports efficient searching, insertion and deletion.

## 4.1 First Understand a Normal Tree

A tree contains:

- Root node.
- Parent nodes.
- Child nodes.
- Leaf nodes.

Example:

text

```text
              50
             /  \
            /    \
          25      75
         /  \    /  \
        10  30  60  90
```

To search for 60:

1. Start at 50.
2. 60 is greater than 50.
3. Move right.
4. Compare with 75.
5. 60 is smaller than 75.
6. Move left.
7. Find 60.

We don't need to examine every value.

## 4.2 What Is a B-Tree?

A B-Tree is a balanced multi-way search tree.

Unlike a simple binary tree, a B-Tree node can contain multiple keys and multiple children.

Simplified illustration:

text

```text
              [30 | 60]
             /    |    \
            /     |     \
           v      v      v

       [10|20] [40|50] [70|80|90]
```

The keys guide the search toward the correct child node.

## 4.3 Why Do Databases Use B-Trees?

Databases frequently need to read and write pages from storage.

B-Trees have characteristics that make them useful for database indexing:

1. They remain balanced.
2. They have relatively small tree height.
3. They support efficient searching.
4. They support insertion and deletion.
5. They work well with page-based storage.

Searching in a balanced B-Tree is typically logarithmic in the number of indexed keys.

## 4.4 What Is a B+ Tree?

You may also hear about B+ Trees in database interviews.

A B+ Tree is a related tree structure commonly used for database indexes.

Important characteristics:

- Internal nodes guide searches.
- Actual indexed entries are stored in leaf nodes.
- Leaf nodes are commonly linked for efficient sequential and range traversal.

Illustration:

text

```text
               [30 | 60]
              /    |    \
             /     |     \
            v      v      v

       [10|20] -> [30|40|50] -> [60|70|80]
```

The linked leaves make range queries efficient.

Example:

sql

```sql
SELECT *
FROM notes
WHERE id BETWEEN 100 AND 200;
```

A B+ Tree can help locate the starting key and traverse subsequent leaf entries.

Interview point: B-Trees and B+ Trees are related but not identical. Many relational database indexes use B+ Tree-style structures.

# 5. What Are Database Transactions?

Imagine our Notes application allows users to transfer ownership of a shared note.

A user transfers a note from User A to User B.

We need to perform multiple operations:

1. Remove ownership from User A.
2. Assign ownership to User B.
3. Update sharing information.

What happens if the application crashes after the first operation?

The data could become inconsistent.

This is why we need database transactions.

## 5.1 Definition

A database transaction is a group of database operations treated as one logical unit of work.

A transaction should either complete successfully or be rolled back according to the database's transaction rules.

Example:

sql

```sql
BEGIN TRANSACTION;

UPDATE notes
SET owner_id = 102
WHERE id = 501;

UPDATE note_shares
SET permission = 'OWNER'
WHERE note_id = 501
AND user_id = 102;

COMMIT;
```

If an error occurs before commit, the transaction can be rolled back.

sql

```sql
ROLLBACK;
```

## 5.2 What Are ACID Properties?

ACID is one of the most important database interview concepts.

ACID stands for:

- Atomicity.
- Consistency.
- Isolation.
- Durability.

Let's understand each.

### A. Atomicity

Atomicity means all operations in a transaction succeed together or the transaction is rolled back.

Example:

A banking transfer:

- Deduct ₹1,000 from Account A.
- Add ₹1,000 to Account B.

Both operations should succeed together.

We should not deduct money without completing the corresponding credit.

### B. Consistency

Consistency means a transaction preserves the database's defined integrity constraints.

Example:

A bank account should not violate its applicable balance rules after a successful transaction.

The database moves from one valid state to another valid state according to its constraints and business rules.

### C. Isolation

Isolation controls how concurrent transactions interact.

Imagine two users update the same record simultaneously.

Without suitable isolation and concurrency control, unexpected results can occur.

Databases provide different isolation levels to manage these situations.

### D. Durability

Durability means that once a transaction is committed, the database is designed to preserve that committed change even if a failure occurs, subject to its durability guarantees.

## 5.3 ACID Summary

| Property | Meaning |
| --- | --- |
| Atomicity | All or nothing |
| Consistency | Preserve data integrity |
| Isolation | Control concurrent transactions |
| Durability | Committed changes are preserved |

# 6. What Are Database Isolation Levels?

Imagine two transactions are running simultaneously.

Transaction A:

sql

```sql
UPDATE accounts
SET balance = balance - 1000
WHERE id = 101;
```

Transaction B:

sql

```sql
SELECT balance
FROM accounts
WHERE id = 101;
```

Should Transaction B see Transaction A's uncommitted update?

This depends on the isolation level and database implementation.

Common SQL isolation levels:

1. Read Uncommitted.
2. Read Committed.
3. Repeatable Read.
4. Serializable.

## 6.1 Read Uncommitted

A transaction may read changes that another transaction has not committed.

This can cause dirty reads.

## 6.2 Read Committed

A transaction reads only committed data.

However, repeated reads within one transaction may return different values if another transaction commits an update between them.

## 6.3 Repeatable Read

A transaction gets stronger guarantees for repeated reads of the same data.

Exact behavior depends on the database implementation.

## 6.4 Serializable

This provides the strongest standard SQL isolation level.

Concurrent transactions behave as though they executed in some serial order.

It can reduce concurrency and increase contention.

## 6.5 Common Concurrency Problems

| Problem | Meaning |
| --- | --- |
| Dirty Read | Reading uncommitted changes |
| Non-repeatable Read | Same row returns different committed values |
| Phantom Read | Re-executing a query returns a changed set of rows |
| Lost Update | One concurrent update overwrites another |

Interview tip: Isolation levels and their exact behavior can vary across database engines. Understand the concepts rather than assuming every database implements them identically.

# 7. What Is Database Normalization?

Imagine our Notes application stores user information and notes in one table.

| Note ID | User Name | Email | Note Title |
| --- | --- | --- | --- |
| 1 | Rahul | rahul@gmail.com | Meeting |
| 2 | Rahul | rahul@gmail.com | Project |
| 3 | Amit | amit@gmail.com | Shopping |

What problems can happen?

- Rahul's email is duplicated.
- Updating Rahul's email requires changing multiple rows.
- Deleting Rahul's last note could accidentally remove the only stored user information.

These are examples of database anomalies.

Normalization helps reduce unnecessary duplication and improve data integrity.

## 7.1 Definition

Database normalization is the process of organizing relational database tables to reduce redundancy and avoid undesirable data modification anomalies.

## 7.2 First Normal Form (1NF)

Each column should contain atomic values, and each row should be identifiable.

Bad example:

| User ID | Phone Numbers |
| --- | --- |
| 101 | 1111, 2222 |

Better structure:

| User ID | Phone Number |
| --- | --- |
| 101 | 1111 |
| 101 | 2222 |

## 7.3 Second Normal Form (2NF)

A table must:

- Satisfy 1NF.
- Have no partial dependency of non-key attributes on part of a candidate key.

This is especially relevant when a table has a composite key.

## 7.4 Third Normal Form (3NF)

A table must:

- Satisfy 2NF.
- Avoid inappropriate transitive dependencies of non-key attributes on candidate keys.

In simpler terms, non-key attributes should depend on the key, the whole key and nothing but the key.

## 7.5 Normalized Notes Database

Instead of storing all information in one table:

Users table

| User ID | Name | Email |
| --- | --- | --- |
| 101 | Rahul | rahul@gmail.com |
| 102 | Amit | amit@gmail.com |

Notes table

| Note ID | User ID | Title |
| --- | --- | --- |
| 1 | 101 | Meeting |
| 2 | 101 | Project |
| 3 | 102 | Shopping |

Now user information is maintained separately.

The Notes table references users using `user_id`.

# 8. What Is Database Denormalization?

Normalization reduces redundancy.

But sometimes a normalized database requires multiple joins to retrieve frequently accessed information.

This can affect query performance depending on the workload.

Denormalization intentionally introduces some redundancy to improve read performance or simplify common access patterns.

## 8.1 Example

Imagine we frequently display notes with the owner's name.

Normalized structure:

text

```text
USERS TABLE
     |
     | user_id
     |
NOTES TABLE
```

To display note title and owner name, we perform a join.

sql

```sql
SELECT
    notes.title,
    users.name
FROM notes
JOIN users
ON notes.user_id = users.id;
```

An alternative is storing a copy of the owner's display name in the Notes record.

| Note ID | User ID | Owner Name | Title |
| --- | --- | --- | --- |
| 1 | 101 | Rahul | Meeting |
| 2 | 101 | Rahul | Project |

Now some reads may become simpler.

But if Rahul changes his name, we need to update duplicated values.

This is the trade-off.

## 8.2 Normalization vs Denormalization

| Normalization | Denormalization |
| --- | --- |
| Reduces duplication | Introduces controlled duplication |
| Improves data organization | Can simplify reads |
| Often requires joins | May reduce joins |
| Updates are easier to centralize | Updates may need multiple changes |
| Common in transactional systems | Common in read-optimized designs |

Important:

Denormalization should be based on measured access patterns, not introduced automatically.

# 9. What Is Blob Storage?

Blob stands for Binary Large Object.

It refers to large binary data such as:

- Images.
- Videos.
- Audio.
- PDF files.
- Documents.

For example, imagine a user uploads a 10 MB PDF attachment to the Notes application.

Should we store the complete PDF inside a normal database row?

Some databases support binary storage, but large files are often better handled using dedicated file or object storage systems.

## 9.1 Definition

Blob storage refers to storing large unstructured binary objects rather than representing all their content as ordinary structured database fields.

Example:

text

```text
        NOTES APPLICATION
                |
                v
          FILE STORAGE
                |
       +--------+--------+
       |        |        |
       v        v        v
     IMAGE     PDF      VIDEO
```

## 9.2 Where Is Blob Storage Useful?

- User profile images.
- Documents.
- Video files.
- Audio files.
- Application attachments.
- Backups.

# 10. What Is Object Storage?

Object storage is a storage architecture that stores data as objects.

Each object typically contains:

1. Object data.
2. Metadata.
3. A unique identifier or key.

For example:

text

```text
           OBJECT STORAGE

        +-------------------+
        | Object: image.jpg |
        |                   |
        | Binary Data       |
        | Metadata          |
        | Object Key        |
        +-------------------+

        +-------------------+
        | Object: notes.pdf |
        |                   |
        | Binary Data       |
        | Metadata          |
        | Object Key        |
        +-------------------+
```

Unlike a traditional file system, object storage is accessed through object-oriented APIs.

## 10.1 Popular Object Storage Services

Amazon S3 Logo PNG Vector (SVG) Free Download

Amazon S3

Object storage provided by AWS.

Used for images, documents, videos and backups.

What is Google Storage Bucket? Docs, Demo and How to Deploy | Shakudo

Google Cloud Storage

Object storage provided by Google Cloud.

Used for large-scale file and data storage.

Microsoft Azure Blob Storage | Integrations | OneTrust

Azure Blob Storage

Microsoft's cloud object storage solution.

Designed for unstructured data.

## 10.2 How Does Object Storage Work?

Imagine a user uploads a profile image.

Flow:

text

```text
          ANDROID APP
               |
               v
          BACKEND API
               |
               v
        OBJECT STORAGE
               |
               v
         image-101.jpg
```

The backend stores the image in object storage.

The database stores its reference.

Users table:

| User ID | Name | Image URL |
| --- | --- | --- |
| 101 | Rahul | profile/101.jpg |

The actual image is stored separately.

## 10.3 Why Use Object Storage?

1. Suitable for large files.
2. Scalable storage capacity.
3. Durable storage options.
4. Metadata support.
5. Integrates with CDNs.
6. Separates file storage from database storage.

Important:

Object storage is not a relational database. It is generally not designed for complex SQL joins or transactional queries across objects.

# 11. What Is the Difference Between Database, File Storage and Object Storage?

| Feature | Database | File Storage | Object Storage |
| --- | --- | --- | --- |
| Main purpose | Structured data | Files and directories | Unstructured objects |
| Organization | Tables or records | Folder hierarchy | Buckets and object keys |
| Querying | SQL or database APIs | File operations | Object APIs |
| Typical data | Users, notes, orders | Documents, files | Images, videos, backups |
| Best use | Business data | Shared file access | Large-scale file storage |

# 12. What Is a CDN?

CDN stands for Content Delivery Network.

Imagine our Notes application has users in:

- India.
- USA.
- UK.
- Australia.

Our images are stored in an object storage bucket located in one region.

A user in Australia requests an image.

If every request travels to the original storage location, network latency may increase.

A CDN helps solve this problem.

## 12.1 Definition

A CDN is a geographically distributed network of servers that delivers content to users from locations closer to them.

## 12.2 How Does a CDN Work?

Imagine:

text

```text
                USER IN INDIA
                      |
                      v
                 CDN EDGE
                      |
                      v
              CACHED IMAGE?
                  /     \
                YES      NO
                 |        |
                 v        v
             RETURN    ORIGIN
              IMAGE    STORAGE
                          |
                          v
                      CDN CACHE
```

If the requested content is cached at an edge location, the CDN can return it directly.

If not, it retrieves the content from the origin and may cache it according to its caching policy.

## 12.3 Why Do We Need a CDN?

1. Reduce latency for cached content.
2. Reduce origin server load.
3. Improve content delivery.
4. Support geographically distributed users.
5. Improve scalability for static content.

## 12.4 Real-World CDN Providers

- Cloudflare.
- Amazon CloudFront.
- Akamai.
- Google Cloud CDN.

## 12.5 CDN vs Object Storage

This is an important distinction.

| Object Storage | CDN |
| --- | --- |
| Stores original objects | Delivers content closer to users |
| Acts as an origin storage system | Acts as a distributed delivery layer |
| Stores images and videos | Caches and serves content |
| Example: S3 | Example: CloudFront |

A CDN usually works together with object storage.

# 13. What Is SQL vs NoSQL in Storage Architecture?

We studied SQL vs NoSQL in Week 1.

Let's connect it with today's concepts.

## SQL Databases

Examples:

- PostgreSQL.
- MySQL.
- Microsoft SQL Server.

Useful for:

- User accounts.
- Financial transactions.
- Orders.
- Structured business data.
- Relational queries.

## NoSQL Databases

Examples:

- MongoDB.
- Cassandra.
- DynamoDB.

Useful for different workloads depending on the database:

- Flexible document data.
- Large-scale distributed workloads.
- High-throughput key-value access.
- Specialized access patterns.

Important:

SQL databases are not automatically slow at scale, and NoSQL databases are not automatically better for scalability.

Choose based on data relationships, consistency requirements, query patterns and operational needs.

# 14. Real-World Notes Application Storage Architecture

Now let's combine everything.

Our Notes application supports:

- 10 million users.
- Millions of notes.
- Images and PDFs.
- Search.
- Multiple geographical regions.

A simplified architecture:

text

```text
                 ANDROID USERS
                       |
                       v
                  API GATEWAY
                       |
             +---------+---------+
             |                   |
             v                   v
        NOTES SERVICE        USER SERVICE
             |                   |
             v                   v
         NOTES DB              USER DB
             |
             v
        OBJECT STORAGE
             |
             v
            CDN
             |
             v
         ANDROID APP
```

Let's understand the storage responsibilities.

| Component | Responsibility |
| --- | --- |
| User Database | User information |
| Notes Database | Notes and metadata |
| Database Indexes | Faster queries |
| Object Storage | Images and PDFs |
| CDN | Faster delivery of cached files |
| Cache | Frequently accessed data |

## Example: Uploading a Note Attachment

1. User selects a PDF in Android.
2. Android sends an upload request.
3. Backend authenticates the user.
4. Backend authorizes the upload.
5. File is uploaded to object storage, possibly through a pre-signed URL.
6. Object storage returns an object key.
7. Notes Service stores the object reference in the Notes Database.
8. Android receives the successful response.
9. Later, the file can be delivered through a CDN.

Important:

The database stores file metadata and references, while object storage stores the actual binary file.

# 15. Android Developer Perspective

As a Senior Android Developer, you should understand how backend storage decisions affect Android applications.

## 15.1 Room Database

Android applications commonly use Room for local structured storage.

Example:

kotlin

```kotlin
@Entity(tableName = "notes")
data class NoteEntity(
    @PrimaryKey
    val id: Long,

    val title: String,

    val content: String,

    val updatedAt: Long
)
```

Room uses SQLite underneath.

We can define indexes:

kotlin

```kotlin
@Entity(
    tableName = "notes",
    indices = [
        Index(value = ["updatedAt"])
    ]
)
data class NoteEntity(
    @PrimaryKey
    val id: Long,
    val title: String,
    val content: String,
    val updatedAt: Long
)
```

This can improve queries filtering or ordering by `updatedAt`, depending on the query plan.

## 15.2 Local vs Remote Storage

Android architecture:

text

```text
          ANDROID APP
               |
               v
          REPOSITORY
           /      \
          v        v
       ROOM      RETROFIT
      DATABASE    API
```

- Room stores local application data.
- Retrofit communicates with backend APIs.
- Repository coordinates data sources.

## 15.3 Image Storage

For images:

- Use image-loading libraries such as Coil or Glide.
- Use appropriate image caching.
- Avoid loading full-resolution images unnecessarily.
- Use thumbnails where appropriate.
- Support CDN URLs when provided by the backend.

## 15.4 Large File Uploads

When uploading files:

- Use streaming where appropriate.
- Avoid loading very large files entirely into memory.
- Handle upload timeouts.
- Support progress reporting.
- Consider resumable uploads for large files.
- Handle temporary network failures.

## 15.5 Database Performance

As an Android developer, you should also understand:

- Room indexes.
- SQLite query performance.
- Database migrations.
- Transactions.
- Pagination.
- Avoiding unnecessary database reads.

# 16. Common Mistakes to Avoid

1. Thinking an index always makes every query faster.
2. Creating indexes on every database column.
3. Confusing B-Tree with B+ Tree.
4. Assuming database transactions automatically span multiple independent services.
5. Thinking ACID consistency means every replica immediately contains the latest data.
6. Confusing normalization with denormalization.
7. Storing every large file directly in relational database rows without considering alternatives.
8. Thinking object storage and databases serve the same purpose.
9. Confusing CDN with object storage.
10. Assuming a CDN always returns cached content.
11. Thinking NoSQL is always more scalable than SQL.
12. Ignoring the storage and maintenance costs of indexes.

# 17. Important Interview Questions and Answers

### Q1. What is Database Indexing?

Indexing is a technique that uses an additional data structure to locate database records more efficiently.

### Q2. Why do we need indexes?

To improve data retrieval for frequently executed queries and reduce unnecessary table scans.

### Q3. What are the disadvantages of indexing?

Indexes consume additional storage and introduce maintenance overhead during inserts, updates and deletes.

### Q4. What is a Full Table Scan?

A full table scan examines the table's rows to identify records matching a query.

### Q5. What is a B-Tree?

A balanced multi-way search tree that supports efficient searching, insertion and deletion.

### Q6. What is a B+ Tree?

A related balanced tree structure commonly used for database indexes. Its indexed entries are stored at leaf level, and leaf nodes are commonly linked for range traversal.

### Q7. What is a Database Transaction?

A transaction is a group of database operations treated as one logical unit of work.

### Q8. What are ACID properties?

Atomicity, Consistency, Isolation and Durability.

### Q9. What is Atomicity?

All operations in a transaction succeed together or are rolled back.

### Q10. What is Isolation?

Isolation controls how concurrent transactions interact with each other.

### Q11. What is Normalization?

Organizing relational tables to reduce unnecessary data duplication and modification anomalies.

### Q12. What is Denormalization?

Intentionally introducing controlled redundancy to improve read performance or simplify common queries.

### Q13. What is Blob Storage?

Storage for large binary objects such as images, videos, PDFs and documents.

### Q14. What is Object Storage?

A storage architecture that stores data as objects containing data, metadata and unique identifiers.

### Q15. What is a CDN?

A Content Delivery Network is a geographically distributed network that delivers content from locations closer to users.

### Q16. What is the difference between CDN and Object Storage?

Object storage stores original files, while a CDN distributes and caches content for faster delivery.

### Q17. Why should we avoid storing large images directly in database rows?

Large binary files can increase database storage and backup overhead and may complicate performance management. Dedicated object storage is often a better fit.

### Q18. What is Database Normalization vs Denormalization?

Normalization reduces duplication and improves data organization. Denormalization introduces controlled duplication to optimize read-heavy workloads.

### Q19. What is Database Isolation Level?

It defines the guarantees a database provides regarding how concurrent transactions can see each other's changes.

### Q20. How would you design storage for a Notes application?

I would use a relational database for users and notes, suitable indexes for frequently executed queries, object storage for attachments, and a CDN for distributing frequently accessed files.

# 18. Practical Interview Exercise

Imagine you are designing a Notes application with:

- 10 million users.
- 1 billion notes.
- 500 million uploaded attachments.
- Users accessing the application globally.
- Frequent note searches.
- Millions of image requests per day.

Answer these questions:

1. Would you use SQL or NoSQL for notes? Why?
2. How would you store user information?
3. How would you store images and PDFs?
4. Why would you create database indexes?
5. Which columns would you consider indexing?
6. What is a B+ Tree?
7. How would you handle simultaneous database updates?
8. What are ACID properties?
9. Would you normalize or denormalize the Notes database?
10. How would you deliver images to users globally?
11. What is the difference between object storage and CDN?
12. How would Android efficiently load large images?
13. How would you handle large file uploads?
14. How would you optimize a slow database query?

Try drawing the architecture without referring to the notes.

# 19. Day 24 – Quick Revision

| Concept | Simple Meaning |
| --- | --- |
| Data Management | Organizing and maintaining application data |
| Database Page | Unit of database storage and I/O |
| Disk I/O | Reading and writing storage |
| Full Table Scan | Examining table rows |
| Indexing | Faster record lookup using an index |
| B-Tree | Balanced multi-way search tree |
| B+ Tree | Tree structure with indexed entries at leaf level |
| Transaction | Group of operations treated as one unit |
| ACID | Atomicity, Consistency, Isolation, Durability |
| Normalization | Reducing unnecessary duplication |
| Denormalization | Introducing controlled redundancy |
| Blob | Large binary object |
| Object Storage | Stores data as objects |
| CDN | Distributes content closer to users |
| SQL | Relational database approach |
| NoSQL | Non-relational database approaches |

# 20. Final Learning Checklist

- [ ]Understand how databases store data.
- [ ]Understand database pages.
- [ ]Understand disk I/O.
- [ ]Understand full table scans.
- [ ]Understand database indexing.
- [ ]Understand index advantages and disadvantages.
- [ ]Understand B-Tree.
- [ ]Understand B+ Tree.
- [ ]Understand database transactions.
- [ ]Understand ACID properties.
- [ ]Understand database isolation levels.
- [ ]Understand dirty reads and lost updates.
- [ ]Understand database normalization.
- [ ]Understand 1NF, 2NF and 3NF.
- [ ]Understand denormalization.
- [ ]Understand Blob Storage.
- [ ]Understand Object Storage.
- [ ]Understand CDN.
- [ ]Understand SQL vs NoSQL storage choices.
- [ ]Explain the Notes application storage architecture.
- [ ]Connect storage concepts with Android Room and Retrofit.

# 21. What Is Next?

## Week 4 – Day 25: Search and Notification Systems

Tomorrow we will learn:

1. What is a search system?
2. How search architecture works.
3. Full-text search.
4. Inverted indexes.
5. Elasticsearch fundamentals.
6. Search indexing and querying.
7. Notification system architecture.
8. Push notifications.
9. Firebase Cloud Messaging (FCM).
10. Email and SMS notifications.
11. Notification delivery and retries.
12. Real-world Notes application architecture.
13. Android developer perspective.
14. Interview questions and practical exercises.

Day 24 Complete – Storage and Data Management.

## Today's five most important concepts

1. Database Indexing: Helps retrieve records efficiently without scanning the entire table.
2. B+ Tree: A balanced tree structure commonly used in database indexes.
3. ACID: Ensures important transaction guarantees.
4. Object Storage: Stores large files such as images, videos and PDFs.
5. CDN: Delivers cached content from geographically distributed servers.

Remember this architecture:

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
          +-------+-------+
          |               |
          v               v
       NOTES DB       OBJECT STORAGE
          |               |
          v               v
        INDEXES           CDN
                          |
                          v
                     ANDROID APP
```

Roadmap tracking: Week 4 – Day 24 complete. Next: Day 25 – Search and Notification Systems.
