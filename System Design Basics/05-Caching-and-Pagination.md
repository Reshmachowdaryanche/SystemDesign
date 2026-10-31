# Week 1 – Day 5: Caching and Pagination

Week 1Day 5Beginner level

File name: `Day-5-Caching-and-Pagination.md`

Today's topics:

1. What is caching?
2. Why do we need caching?
3. How caching improves performance.
4. Client-side caching.
5. Server-side caching.
6. Database caching.
7. What is Redis?
8. Cache hit and cache miss.
9. Cache eviction policies.
10. Cache invalidation.
11. Cache expiration (TTL).
12. Cache-aside pattern.
13. Write-through and write-back caching.
14. What is pagination?
15. Offset-based pagination.
16. Cursor-based pagination.
17. Offset vs cursor pagination.
18. How caching and pagination work together.
19. Notes application system design.
20. Interview questions and answers.

Writing

# Day 5 – Caching and Pagination

**Week:** 1 – System Design Basics\
**Level:** Beginner\
**Previous topic:** Day 4 – Database Fundamentals: SQL vs NoSQL\
**Example used throughout:** Notes Application

---

# PART 1 – CACHING

# 1\. What is Caching?

## 1.1 First, Understand the Problem

Imagine you are using a Notes application.

You open the application and see your notes.

The application requests the notes from the backend.

The backend fetches the notes from the database and sends them to your Android application.

Now imagine you open the same screen again.

The application makes the same request again.

The backend fetches the same information from the database again.

What is the problem?

We are repeatedly fetching information that may not have changed.

This causes:

- Unnecessary database queries.
- Increased server workload.
- More network requests.
- Increased response time.
- Higher infrastructure costs.

Can we avoid repeatedly fetching the same information?

Yes. We can use caching.

## 1.2 Definition

Caching is a technique of temporarily storing frequently accessed data in a faster storage location so that future requests can retrieve it more quickly.

In simple words:

**Caching means keeping a copy of frequently used data in a faster place so that we don't need to fetch it from the original source every time.**

## 1.3 Real-World Example

Imagine you are studying from a textbook.

You frequently refer to page 50.

Instead of searching for the textbook every time, you keep a photocopy of page 50 on your desk.

Now you can access that page quickly.

This is similar to caching.

- Original textbook = Database.
- Photocopy on your desk = Cache.
- Reading the photocopy = Cache retrieval.

## 1.4 Notes Application Example

Without caching:

text

```text
Android App
    |
    | Request notes
    v
Backend Server
    |
    | Fetch notes
    v
Database
    |
    | Return notes
    v
Backend Server
    |
    v
Android App
```

Every request goes to the database.

With caching:

text

```text
Android App
    |
    v
Backend Server
    |
    v
Cache
    |
    | Cache contains notes?
    |
    | YES
    v
Return notes
```

If the data is already available in the cache, the backend can return it without querying the database.

---

# 2\. Why Do We Need Caching?

Caching is used to improve application performance and reduce unnecessary work.

Let's understand the benefits.

## 2.1 Faster Response Time

A cache is generally faster to access than a remote database.

Example:

- Database query: 100 ms.
- Cache lookup: 5 ms.

These are illustrative numbers, not guaranteed performance measurements.

If the requested data is available in the cache, the application may respond much faster.

## 2.2 Reduce Database Load

Imagine 100,000 users open the same popular article.

Without caching:

The database may receive 100,000 similar queries.

With caching:

The backend may fetch the article once and serve many subsequent requests from the cache.

This reduces database workload.

## 2.3 Reduce Server Costs

Fewer database queries and less repeated processing can reduce infrastructure requirements and operational costs.

## 2.4 Improve Scalability

When the number of users increases, caching can help the system handle more requests without increasing database load proportionally.

## 2.5 Better User Experience

Users get faster responses.

For example:

- Faster home screen loading.
- Faster product listing.
- Faster note retrieval.
- Faster profile loading.

## 2.6 Summary

Caching helps us:

1. Reduce response time.
2. Reduce database queries.
3. Reduce server workload.
4. Improve scalability.
5. Reduce infrastructure costs.
6. Improve user experience.

---

# 3\. What Happens When We Don't Use Caching?

Imagine our Notes application has 1 million users.

Every time a user opens the notes screen, the application requests notes from the backend.

Assume:

- 1 million users.
- Each user opens the screen 10 times per day.

Total requests:

1,000,000 × 10 = 10,000,000 requests per day.

If every request reaches the database, the database has to process a large number of queries.

Some requests may ask for the same information repeatedly.

This is unnecessary work.

Now introduce caching.

If frequently requested information is available in the cache, many requests can be served without reaching the database.

This reduces the database workload.

Important:

Caching does not eliminate database access. It reduces repeated access for data that can safely be served from the cache.

---

# 4\. What is a Cache Hit and Cache Miss?

These are two important caching terms.

## 4.1 Cache Hit

A cache hit happens when the requested data is already available in the cache.

Example:

A user requests note 101.

The cache already contains note 101.

The backend retrieves it directly from the cache.

text

```text
Request Note 101
       |
       v
     Cache
       |
       | Found
       v
   Return Note
```

This is called a cache hit.

## 4.2 Cache Miss

A cache miss happens when the requested data is not available in the cache.

Example:

A user requests note 101.

The cache does not contain note 101.

The backend retrieves it from the database.

It may then store the result in the cache for future requests.

text

```text
Request Note 101
       |
       v
     Cache
       |
       | Not Found
       v
    Database
       |
       | Fetch Note
       v
     Cache
       |
       v
   Return Note
```

This is called a cache miss.

## 4.3 Cache Hit vs Cache Miss

| Cache Hit | Cache Miss |
| --- | --- |
| Data is found in cache | Data is not found in cache |
| Usually faster | Usually slower |
| Database may not be queried | Database is typically queried |
| Reduces database workload | May increase database workload |

## 4.4 Cache Hit Ratio

Cache hit ratio tells us how many requests are successfully served from the cache.

Formula:

Cache Hit Ratio = Cache Hits / Total Cache Requests × 100

Example:

- Total cache requests = 1,000.
- Cache hits = 800.
- Cache misses = 200.

Cache hit ratio:

800 / 1000 × 100 = 80%.

This means 80% of cache requests were served from the cache.

A higher hit ratio is often desirable, but it should not be the only performance metric.

---

# 5\. Types of Caching

Caching can happen at different layers of an application.

Let's understand the most important types.

## 5.1 Client-Side Caching

Client-side caching means storing data on the client device.

For Android applications, this could be:

- In-memory cache.
- Room database.
- Local files.
- Image cache.
- HTTP response cache.

Example:

A user opens the Notes application.

The application fetches notes from the backend and stores them locally using Room.

When the user opens the application again, the UI can immediately display locally stored notes.

The application can fetch updated notes from the backend in the background.

Diagram:

text

```text
          ANDROID APP
               |
          Repository
           /      \
          v        v
       Room DB   Backend API
          |
          v
         UI
```

### Advantages

1. Faster screen loading.
2. Supports offline access.
3. Reduces network requests.
4. Improves user experience.

### Disadvantages

1. Uses device storage.
2. Local data can become outdated.
3. Synchronization is required.
4. Data security must be considered.

### Android Example

Imagine you use Room:

kotlin

```kotlin
@Entity
data class Note(
    @PrimaryKey val id: Long,
    val title: String,
    val description: String
)
```

The application can store notes locally.

Room acts as a local persistence layer and can serve as part of an application-level cache.

Important:

A local database and an in-memory cache are not exactly the same thing. A local database provides persistent storage, while an in-memory cache is usually temporary.

---

## 5.2 Server-Side Caching

Server-side caching means storing frequently requested data on the backend side.

Example:

A user requests a note.

The backend checks the cache first.

If the note exists, it returns the note.

Otherwise, it queries the database.

Diagram:

text

```text
Android App
     |
     v
Backend Server
     |
     v
    Cache
     |
     | Cache Miss
     v
  Database
```

### Advantages

1. Reduces database queries.
2. Improves API response time.
3. Helps handle high traffic.
4. Can be shared across multiple backend instances when using a centralized cache.

### Disadvantages

1. Cache infrastructure must be maintained.
2. Cached data can become outdated.
3. Cache failures need to be handled.
4. Additional operational complexity.

---

## 5.3 Database Caching

Databases themselves use caching mechanisms.

For example, PostgreSQL uses shared buffers to cache database pages in memory.

When the database needs a page, it may already be available in memory instead of requiring disk access.

This helps improve database performance.

Important:

Database internal caching is different from an application-level cache such as Redis.

---

## 5.4 CDN Caching

CDN stands for Content Delivery Network.

A CDN stores copies of content at geographically distributed locations.

Example:

Imagine an application displays the same profile image to users in India, the US and Europe.

Instead of downloading the image from one origin server every time, a CDN can serve a cached copy from a nearby location.

Diagram:

text

```text
              Origin Server
                    |
                    v
                 CDN Network
                /     |     \
               v      v      v
            India    USA   Europe
             CDN     CDN    CDN
               |      |      |
               v      v      v
            Users   Users  Users
```

### What is useful to cache in a CDN?

- Images.
- Videos.
- CSS files.
- JavaScript files.
- Static documents.
- Other cacheable public content.

### Benefits

1. Faster content delivery.
2. Reduced origin server load.
3. Better global performance.
4. Reduced bandwidth costs.

---

# 6\. What is Redis?

## 6.1 Definition

Redis is an in-memory data store commonly used as a cache, and it also supports other use cases such as data storage and messaging patterns.

In simple words:

Redis stores data in memory so applications can retrieve frequently accessed information quickly.

## 6.2 Why Is Redis Used?

Imagine our Notes application has millions of users.

Every time a user opens their profile, the backend fetches the same profile information from the database.

We can store frequently accessed profile data in Redis.

Now:

1. User requests profile.
2. Backend checks Redis.
3. If available, return the cached profile.
4. Otherwise, fetch it from the database.
5. Store the result in Redis.
6. Return the result.

## 6.3 Redis Example

Suppose we want to cache a user's profile.

text

```text
Key:
user:101

Value:
{
  "name": "Rahul",
  "email": "rahul@gmail.com"
}
```

Redis stores a mapping between the key and its value.

A simple Redis command:

redis

```redis
SET user:101 "Rahul"
```

Retrieve it:

redis

```redis
GET user:101
```

Delete it:

redis

```redis
DEL user:101
```

Set it with a 60-second expiration:

redis

```redis
SET user:101 "Rahul" EX 60
```

After the expiration period, the key is eligible to expire.

## 6.4 Common Redis Use Cases

1. Application caching.
2. User session storage.
3. Rate limiting.
4. Counters.
5. Temporary data.
6. Leaderboards.
7. Certain messaging and queueing patterns.

## 6.5 Redis vs Database

| Redis | Traditional Relational Database |
| --- | --- |
| Commonly used for in-memory caching | Commonly used for durable relational storage |
| Very fast access for suitable operations | Performance depends on query, storage and configuration |
| Supports data structures such as strings, lists, sets and hashes | Stores data in relational tables |
| Often stores temporary or frequently accessed data | Often stores primary application records |
| Persistence is configurable | Durability is a core design goal |

Important:

Redis can support persistence, but it should not automatically replace your primary database.

For a Notes application, PostgreSQL might store the original notes, while Redis stores frequently requested copies.

---

# 7\. What is Cache Expiration (TTL)?

TTL stands for Time To Live.

TTL defines how long cached data should remain valid before it expires.

In simple words:

TTL is the lifetime of cached data.

## Example

Suppose we cache a user's profile.

We decide that the cached profile should remain valid for 10 minutes.

After 10 minutes, the cached entry expires.

The next request may fetch the latest profile from the database.

Diagram:

text

```text
Time 0
  |
  v
Store Profile in Cache
  |
  v
Profile Available
  |
  v
10 Minutes Pass
  |
  v
Cache Entry Expires
  |
  v
Next Request Fetches Fresh Data
```

## Why Do We Need TTL?

Without expiration, cached data might remain outdated indefinitely.

TTL helps limit how long an old cache entry can remain available.

## Example Redis Command

redis

```redis
SET note:101 "Shopping List" EX 300
```

Here:

300 seconds = 5 minutes.

The key expires after the specified TTL.

## Where Is TTL Useful?

- User profile cache.
- Product information.
- Search results.
- API responses.
- Temporary sessions.
- Frequently accessed configuration.

Important:

TTL does not guarantee that cached data is always fresh.

It only limits how long a particular cached entry is allowed to remain.

---

# 8\. What is Cache Eviction?

## 8.1 First, Understand the Problem

Imagine Redis has limited memory.

Your application keeps adding new cache entries.

Eventually, the cache reaches its configured memory limit.

What happens when there is no room for more data?

The cache needs a strategy to remove existing entries.

This is called cache eviction.

## 8.2 Definition

Cache eviction is the process of removing cached entries to free memory when required.

## 8.3 Common Eviction Policies

### 1\. LRU – Least Recently Used

LRU removes the data that has not been accessed for the longest time.

Example:

Cache contains:

- Note A.
- Note B.
- Note C.

Access history:

- Note A was accessed 10 minutes ago.
- Note B was accessed 5 minutes ago.
- Note C was accessed 1 minute ago.

If the cache needs to evict an entry, Note A is the least recently used.

### 2\. LFU – Least Frequently Used

LFU removes data that has been accessed the fewest times.

Example:

| Item | Access Count |
| --- | --- |
| Note A | 2 |
| Note B | 50 |
| Note C | 10 |

Note A is the least frequently used.

### 3\. FIFO – First In, First Out

FIFO removes the entry that was added first.

Example:

text

```text
Added:
A -> B -> C

Eviction:
A is removed first.
```

### 4\. Random Replacement

The cache randomly selects an entry to remove.

## 8.4 Comparison

| Policy | Removal Strategy |
| --- | --- |
| LRU | Least recently accessed |
| LFU | Least frequently accessed |
| FIFO | Oldest inserted entry |
| Random | Random entry |

Important:

Redis supports configurable eviction policies, including variants of LRU and LFU. Exact behavior depends on the selected configuration.

---

# 9\. What is Cache Invalidation?

This is one of the most important caching concepts.

## 9.1 First, Understand the Problem

Imagine our Notes application.

A user creates a note:

text

```text
Title: Shopping List
```

The backend stores the note in the database.

It also stores a cached copy in Redis.

Now the user edits the note:

text

```text
Title: Weekly Shopping List
```

The database is updated.

But what about Redis?

Redis still contains:

text

```text
Title: Shopping List
```

Now the backend may return old information.

This is called stale data.

## 9.2 Definition

Cache invalidation is the process of removing or updating cached data when the original data changes.

In simple words:

When the original data changes, we must make sure the cache does not continue serving an incorrect old copy.

## 9.3 Common Cache Invalidation Strategies

### Strategy 1: Delete the Cache Entry

When a note is updated:

1. Update the database.
2. Delete the corresponding cache entry.
3. The next request fetches fresh data.
4. Store the fresh result in cache.

Example:

text

```text
User Updates Note
       |
       v
Update Database
       |
       v
Delete Redis Entry
       |
       v
Next Request
       |
       v
Fetch Fresh Data
       |
       v
Update Cache
```

This is a common strategy.

### Strategy 2: Update the Cache

When the note is updated:

1. Update the database.
2. Update the corresponding cached entry.

Now both contain the new information.

### Strategy 3: Use TTL

Cached data expires after a specified period.

This limits how long stale information may remain in the cache.

However, it does not immediately invalidate data when the database changes.

## 9.4 Why Is Cache Invalidation Difficult?

Because the same data may be cached in multiple places.

For example:

- Android local database.
- Backend application cache.
- Redis.
- CDN.

When the original data changes, the system must manage all relevant copies.

This becomes especially difficult when multiple users or devices modify the same information.

---

# 10\. Cache-Aside Pattern

This is a very important interview topic.

## 10.1 Definition

Cache-aside is a caching pattern in which the application checks the cache first.

If the data is unavailable, the application fetches it from the database and stores it in the cache.

It is also called lazy loading.

## 10.2 How Does It Work?

Imagine a user requests note 101.

Step 1: Backend checks Redis.

Step 2: If the note is found, return it.

Step 3: If not found, fetch it from the database.

Step 4: Store the result in Redis.

Step 5: Return the note.

Diagram:

text

```text
         User Requests Note
                  |
                  v
             Check Cache
                  |
             +----+----+
             |         |
           Found     Not Found
             |         |
             v         v
         Return     Query Database
         Result         |
                        v
                   Store in Cache
                        |
                        v
                    Return Note
```

## 10.3 Example

python

```python
def get_note(note_id):

    # 1. Check cache
    note = cache.get(note_id)

    if note is not None:
        return note

    # 2. Cache miss: fetch from database
    note = database.get_note(note_id)

    if note is None:
        return None

    # 3. Store in cache
    cache.set(note_id, note, ttl=300)

    # 4. Return result
    return note
```

This is illustrative pseudocode.

## 10.4 Advantages

1. Simple to implement.
2. Only requested data gets cached.
3. Cache failures can often be handled by reading from the database.
4. Works well for frequently accessed data.

## 10.5 Disadvantages

1. First request may be slow.
2. Cache invalidation must be handled.
3. Multiple simultaneous cache misses can cause repeated database queries.
4. Cached data may become stale.

---

# 11\. Write-Through Caching

## 11.1 Definition

In write-through caching, a write operation updates the cache and the database as part of the write path.

Conceptually:

When the application writes data, the cache and database are updated together.

Diagram:

text

```text
          User Updates Note
                  |
                  v
             Write Path
                  |
             +----+----+
             |         |
             v         v
           Cache    Database
```

The exact order and failure-handling behavior depend on the implementation.

## 11.2 Example

A user changes the note title.

text

```text
Old Title: Shopping
New Title: Weekly Shopping
```

The system updates the database and cache as part of the write operation.

### Advantages

1. Cache is more likely to contain the latest value.
2. Reduces stale reads caused by an old cached entry.
3. Useful for workloads with frequent reads after writes.

### Disadvantages

1. Writes can take longer.
2. Both systems must be managed.
3. Partial failures need careful handling.

---

# 12\. Write-Back Caching

## 12.1 Definition

In write-back caching, the system first updates the cache and writes the data to the database later.

Diagram:

text

```text
        User Updates Note
                |
                v
              Cache
                |
                v
        Write to Database Later
```

## 12.2 Example

A user updates a note.

The cache immediately stores the updated value.

The database is updated asynchronously afterward.

### Advantages

1. Can reduce write latency.
2. Can combine multiple writes.
3. Useful for specific high-throughput workloads.

### Disadvantages

1. If the cache fails before data is persisted, recent changes may be lost.
2. More complex recovery mechanisms are required.
3. Database and cache can temporarily differ.

Important:

Write-back caching requires careful durability and failure-handling design. It should not be used casually for critical user data.

---

# 13\. Cache-Aside vs Write-Through vs Write-Back

| Pattern | Read Behavior | Write Behavior |
| --- | --- | --- |
| Cache-aside | Application checks cache first | Application updates database and invalidates or refreshes cache |
| Write-through | Cache can serve reads | Cache and database are updated on the write path |
| Write-back | Cache can serve reads | Cache is updated first; database is updated later |

Remember:

- Cache-aside: Load into cache when needed.
- Write-through: Update cache and database together.
- Write-back: Update cache first, database later.

---

# PART 2 – PAGINATION

# 14\. What is Pagination?

## 14.1 First, Understand the Problem

Imagine your Notes application has 10,000 notes.

When the user opens the notes screen, should the backend return all 10,000 notes at once?

No.

Why?

Because:

- Large responses consume network bandwidth.
- More data takes longer to transfer.
- The application needs more memory.
- Rendering thousands of items is expensive.
- The backend has to process a larger response.

Instead, we can return a small number of notes at a time.

This is called pagination.

## 14.2 Definition

Pagination is a technique of dividing a large collection of data into smaller groups and returning only one group at a time.

In simple words:

Instead of fetching all records at once, fetch a limited number of records per request.

## 14.3 Real-World Example

Imagine Google search results.

You search for:

Android System Design.

The search engine does not display every result on one screen.

It displays a limited number of results.

You can navigate to the next page to see more results.

That is pagination.

## 14.4 Notes Application Example

Suppose we have 1,000 notes.

We decide to fetch 20 notes per request.

| Page | Notes |
| --- | --- |
| Page 1 | Notes 1–20 |
| Page 2 | Notes 21–40 |
| Page 3 | Notes 41–60 |
| Page 4 | Notes 61–80 |

The application fetches more notes when the user scrolls.

## 14.5 Why Do We Need Pagination?

1. Reduces response size.
2. Reduces network bandwidth.
3. Improves loading performance.
4. Reduces memory usage.
5. Improves user experience.
6. Makes large datasets easier to navigate.

---

# 15\. Offset-Based Pagination

## 15.1 Definition

Offset-based pagination retrieves a limited number of records while skipping a specified number of records.

Two important parameters:

- LIMIT: Number of records to return.
- OFFSET: Number of records to skip.

## 15.2 Example

Imagine we have 100 notes.

We want to retrieve 10 notes per page.

### Page 1

sql

```sql
SELECT *
FROM notes
ORDER BY note_id
LIMIT 10
OFFSET 0;
```

Returns records 1–10.

### Page 2

sql

```sql
SELECT *
FROM notes
ORDER BY note_id
LIMIT 10
OFFSET 10;
```

Returns records 11–20.

### Page 3

sql

```sql
SELECT *
FROM notes
ORDER BY note_id
LIMIT 10
OFFSET 20;
```

Returns records 21–30.

## 15.3 Understanding LIMIT and OFFSET

Suppose:

sql

```sql
LIMIT 10 OFFSET 20
```

Meaning:

1. Skip the first 20 matching records.
2. Return the next 10 records.

## 15.4 Advantages

1. Easy to understand.
2. Easy to implement.
3. Supports direct page navigation.
4. Useful for small and relatively stable datasets.

## 15.5 Disadvantages

Imagine the user requests page 10,000.

The database may need to process or traverse many preceding rows before returning the requested page.

This can become inefficient for large offsets.

Another problem is changing data.

Imagine:

1. User loads page 1.
2. New notes are inserted.
3. User loads page 2.
4. Some records may be repeated or skipped depending on ordering and where records were inserted.

This is one reason offset pagination can be problematic for continuously changing datasets.

---

# 16\. Cursor-Based Pagination

## 16.1 First, Understand the Problem

Imagine we have 10 million notes.

Using a large OFFSET may become inefficient.

Can we request the next group of notes using the last record we already received?

Yes.

This is cursor-based pagination.

## 16.2 Definition

Cursor-based pagination uses a reference point, called a cursor, to determine where the next set of records should begin.

In simple words:

Instead of telling the database how many records to skip, we tell it where to continue.

## 16.3 Notes Application Example

Suppose we have these notes:

| note\_id | title |
| --- | --- |
| 101 | Shopping |
| 102 | Meeting |
| 103 | Personal |
| 104 | Work |
| 105 | Travel |

We request the first three notes.

Response:

json

```json
{
  "notes": [
    {"id": 101, "title": "Shopping"},
    {"id": 102, "title": "Meeting"},
    {"id": 103, "title": "Personal"}
  ],
  "nextCursor": 103
}
```

The cursor is 103.

Now the client requests the next page:

text

```text
GET /notes?cursor=103&limit=3
```

The backend fetches records after ID 103.

SQL example:

sql

```sql
SELECT *
FROM notes
WHERE note_id > 103
ORDER BY note_id ASC
LIMIT 3;
```

Response:

json

```json
{
  "notes": [
    {"id": 104, "title": "Work"},
    {"id": 105, "title": "Travel"}
  ],
  "nextCursor": null
}
```

There are no more records.

## 16.4 Advantages

1. Efficient for large datasets when supported by suitable indexes.
2. Avoids large OFFSET scans.
3. Works well with infinite scrolling.
4. Handles many insertions more predictably when using a stable ordering key.

## 16.5 Disadvantages

1. Directly jumping to page 100 can be difficult.
2. Implementation is more complex.
3. Cursor design must be carefully chosen.
4. Sorting and filtering need to be consistent.

---

# 17\. Why Do We Need a Unique Ordering for Pagination?

This is an important interview concept.

Imagine two notes have the same creation timestamp.

| note\_id | title | created\_at |
| --- | --- | --- |
| 101 | Shopping | 10:00 |
| 102 | Meeting | 10:00 |
| 103 | Personal | 10:05 |

If we sort only by `created_at`, two records have the same value.

The database may not always return tied records in a predictable order.

This can lead to duplicated or missing records across pages.

A better approach is to use a unique tie-breaker.

Example:

sql

```sql
ORDER BY created_at DESC, note_id DESC
```

Now we have a deterministic ordering if `note_id` is unique.

The cursor should contain both values:

- created\_at.
- note\_id.

Next-page query:

sql

```sql
SELECT *
FROM notes
WHERE
    created_at < :last_created_at
    OR (
        created_at = :last_created_at
        AND note_id < :last_note_id
    )
ORDER BY created_at DESC, note_id DESC
LIMIT 20;
```

This is an example of keyset pagination.

Keyset pagination is a common form of cursor-based pagination.

---

# 18\. Offset vs Cursor Pagination

This is a very common system design interview question.

| Category | Offset Pagination | Cursor Pagination |
| --- | --- | --- |
| Mechanism | Skips records | Continues from a cursor |
| Parameters | Limit and offset | Cursor and limit |
| Large datasets | Can become inefficient | Often more efficient |
| Direct page navigation | Easy | More difficult |
| Infinite scrolling | Possible | Very suitable |
| Data changes | Can cause duplicates or missing records | Generally handles insertions better with stable ordering |
| Implementation | Simple | More complex |

## When Should We Use Offset Pagination?

- Small datasets.
- Admin dashboards.
- Tables with direct page navigation.
- Applications where direct page numbers are important.

## When Should We Use Cursor Pagination?

- Social media feeds.
- Messaging applications.
- Large notes collections.
- Infinite scrolling.
- Continuously changing datasets.

Important:

Cursor pagination does not automatically guarantee a consistent snapshot of data. Updates and deletions can still affect results depending on the query and consistency requirements.

---

# 19\. What is Infinite Scrolling?

Infinite scrolling is a user interface pattern where more records are loaded automatically as the user approaches the bottom of the current list.

Example:

You open Instagram.

You scroll down.

More posts load automatically.

This is infinite scrolling.

## Android Notes Application Example

Imagine the application initially loads 20 notes.

When the user reaches the bottom:

1. Check whether more notes are available.
2. Read the next cursor.
3. Make another API request.
4. Append the new notes to the existing list.
5. Continue until there are no more notes.

Diagram:

text

```text
Open Notes Screen
       |
       v
Load First 20 Notes
       |
       v
User Scrolls
       |
       v
Reached Bottom?
       |
       v
Load Next 20 Notes
       |
       v
Append to List
       |
       v
Continue Scrolling
```

## Example API

First request:

http

```http
GET /notes?limit=20
```

Response:

json

```json
{
  "notes": [],
  "nextCursor": "abc123",
  "hasMore": true
}
```

Second request:

http

```http
GET /notes?limit=20&cursor=abc123
```

Response:

json

```json
{
  "notes": [],
  "nextCursor": "xyz456",
  "hasMore": true
}
```

Final request:

json

```json
{
  "notes": [],
  "nextCursor": null,
  "hasMore": false
}
```

The cursor does not have to be a database ID. It can be an opaque token generated by the backend.

---

# PART 3 – COMBINING CACHING AND PAGINATION

# 20\. How Do Caching and Pagination Work Together?

Let's combine both concepts.

Imagine our Notes application has:

- 10 million users.
- Each user has hundreds of notes.
- Users frequently open the notes screen.
- Users scroll through their notes.

We need to retrieve notes efficiently.

We can combine caching and pagination.

## Architecture

text

```text
             Android App
                  |
                  v
               Backend
                  |
                  v
               Redis
                  |
          Cache Hit?
             /    \
           Yes     No
            |       |
            v       v
         Return   Database
                    |
                    v
                Fetch Page
                    |
                    v
               Store Cache
                    |
                    v
                Return Page
```

## Step-by-Step Flow

### Step 1: User Opens Notes Screen

Android sends:

http

```http
GET /notes?limit=20
```

### Step 2: Backend Checks Cache

Example cache key:

text

```text
notes:user:101:first-page
```

### Step 3: Cache Hit

If the first page is cached, the backend returns it.

### Step 4: Cache Miss

If the page is not cached:

- Query the database.
- Fetch the first 20 notes.
- Store the response in Redis.
- Return the response.

### Step 5: User Scrolls

Android sends another request using the next cursor.

http

```http
GET /notes?limit=20&cursor=abc123
```

The backend can check the cache for this page or fetch it from the database.

### Step 6: User Updates a Note

The backend updates the database.

It must also invalidate or update affected cached pages.

This is an important design challenge.

---

# 21\. How Would You Design Caching for a Notes Application?

Let's answer this like a system design interview.

## Requirements

Our Notes application should:

1. Allow users to create notes.
2. Allow users to read notes.
3. Allow users to update notes.
4. Allow users to delete notes.
5. Support large numbers of notes.
6. Load notes quickly.

## Database

Use PostgreSQL as the primary database.

## Cache

Use Redis for frequently accessed data.

## API

Use REST APIs.

## Pagination

Use cursor-based pagination for large note collections.

## Read Flow

1. User requests notes.
2. Backend checks Redis.
3. If cached, return cached notes.
4. Otherwise, fetch notes from PostgreSQL.
5. Store the response in Redis with a suitable TTL.
6. Return the response.

## Write Flow

1. User updates a note.
2. Backend validates the request.
3. Backend updates PostgreSQL.
4. Backend invalidates or updates the relevant cache entries.
5. Backend returns success.

## Important Design Considerations

### Cache Per User

Never use a cache key that accidentally mixes different users' private data.

Example:

text

```text
notes:user:101
notes:user:102
```

### Cache Per Page

For paginated data, the cursor and relevant filters may need to be part of the cache key.

Example:

text

```text
notes:user:101:limit:20:cursor:abc123
```

### Cache Invalidation

When a note is updated, affected cached pages must be invalidated or refreshed.

### Authorization

The backend must verify that the requesting user is allowed to access the requested notes.

The cache must not bypass authorization.

### Cache Failure

If Redis is unavailable, the backend may fall back to the primary database if that is safe and capacity allows.

A cache failure should not automatically mean that the entire application becomes unavailable.

---

# 22\. Android Developer Perspective

As an Android developer, understand how caching and pagination are implemented on the client side.

## 22.1 Android Local Caching

Common options:

- Room.
- In-memory collections.
- Image loading libraries with memory and disk caching.
- HTTP caching where appropriate.

Example architecture:

text

```text
             UI
              |
              v
          ViewModel
              |
              v
          Repository
           /      \
          v        v
       Room DB   Retrofit
                    |
                    v
                 Backend
```

Room stores local data.

Retrofit communicates with the backend.

The repository decides when to read local data and when to request remote data.

## 22.2 Android Pagination

For Android applications, Jetpack Paging 3 is commonly used to implement paginated data loading.

Main components:

### PagingSource

Responsible for loading pages of data from a source.

### Pager

Creates a stream of paginated data.

### PagingData

Represents the data being loaded in pages.

### PagingDataAdapter

Displays paginated data in RecyclerView.

Example:

kotlin

```kotlin
class NotesPagingSource(
    private val api: NotesApi
) : PagingSource<Int, Note>() {

    override suspend fun load(
        params: LoadParams<Int>
    ): LoadResult<Int, Note> {

        return try {
            val page = params.key ?: 1

            val response = api.getNotes(
                page = page,
                limit = params.loadSize
            )

            LoadResult.Page(
                data = response.notes,
                prevKey = if (page == 1) null else page - 1,
                nextKey = if (response.hasMore) page + 1 else null
            )

        } catch (e: Exception) {
            LoadResult.Error(e)
        }
    }

    override fun getRefreshKey(
        state: PagingState<Int, Note>
    ): Int? {
        return state.anchorPosition
    }
}
```

This is a simplified offset/page-based example. A cursor-based PagingSource would use the cursor token as its key instead of an integer page number.

## 22.3 Why Use Jetpack Paging?

1. Loads data in smaller chunks.
2. Avoids loading the entire dataset at once.
3. Supports loading states.
4. Supports error handling.
5. Supports refreshing.
6. Works with RecyclerView and Compose integrations.

---

# 23\. Important Interview Questions and Answers

## Q1. What is caching?

Caching is the technique of temporarily storing frequently accessed data in a faster storage layer to improve retrieval performance and reduce repeated work.

## Q2. Why do we need caching?

Caching reduces response time, database workload, network requests and infrastructure costs.

It also helps applications handle more traffic.

## Q3. What is a cache hit?

A cache hit happens when the requested data is found in the cache.

## Q4. What is a cache miss?

A cache miss happens when the requested data is not found in the cache.

The application usually fetches it from the original data source.

## Q5. What is cache hit ratio?

It is the percentage of cache requests that are successfully served from the cache.

Formula:

Cache Hit Ratio = Cache Hits / Total Cache Requests × 100

## Q6. What is Redis?

Redis is an in-memory data store commonly used for caching, session management, counters and other low-latency operations.

## Q7. What is TTL?

TTL stands for Time To Live.

It defines how long a cache entry remains valid before expiration.

## Q8. What is cache eviction?

Cache eviction is the process of removing entries from a cache to free memory.

## Q9. Explain LRU and LFU.

LRU removes the least recently used entry.

LFU removes the least frequently used entry.

## Q10. What is cache invalidation?

Cache invalidation means removing or updating cached data when the original data changes.

## Q11. What is cache-aside?

Cache-aside is a pattern where the application checks the cache first.

On a cache miss, it fetches data from the database and stores it in the cache.

## Q12. What is write-through caching?

Write-through caching updates the cache and database as part of the write path.

## Q13. What is write-back caching?

Write-back caching updates the cache first and writes the data to the database later.

## Q14. What is pagination?

Pagination divides a large dataset into smaller groups and returns a limited number of records per request.

## Q15. What is offset pagination?

Offset pagination uses LIMIT and OFFSET to skip records and return a particular section of the dataset.

## Q16. What is cursor pagination?

Cursor pagination uses a reference point to determine where the next group of records should begin.

## Q17. What is the difference between offset and cursor pagination?

Offset pagination skips a specified number of records.

Cursor pagination continues from a specific record or position.

Cursor pagination is often more efficient for large datasets and infinite scrolling.

## Q18. What is infinite scrolling?

Infinite scrolling automatically loads more records when the user reaches the bottom of the current list.

## Q19. Why is cursor pagination preferred for large datasets?

It avoids large OFFSET scans when implemented using suitable indexed ordering columns.

It is especially useful for feeds and continuously changing datasets.

## Q20. Can caching and pagination be used together?

Yes.

We can cache frequently requested pages of data and use pagination to retrieve only the required records.

## Q21. What happens if Redis goes down?

The backend can potentially fall back to the primary database.

However, the system must be designed to handle the additional database traffic.

## Q22. What happens if cached data is outdated?

The system can invalidate the entry, update it or allow it to expire using TTL.

## Q23. Where would you use caching in an Android application?

I would use Room for local persistence and offline access, memory caching for frequently used in-memory data, and image caching for images.

The backend can use Redis for frequently accessed server-side data.

## Q24. How would you load 1 million notes efficiently?

I would use cursor-based pagination to retrieve a small number of notes at a time.

I would add suitable database indexes and consider caching frequently accessed results.

I would avoid returning all notes in one API response.

## Q25. What is the difference between local caching and server-side caching?

Local caching stores data on the user's device.

Server-side caching stores data on backend infrastructure and can be shared by multiple application instances or users, depending on the cache key and access rules.

---

# 24\. Day 5 Revision Notes

| Concept | One-line definition |
| --- | --- |
| Caching | Stores frequently accessed data in a faster layer |
| Cache Hit | Requested data exists in cache |
| Cache Miss | Requested data does not exist in cache |
| Hit Ratio | Percentage of cache requests served successfully |
| Client-Side Cache | Cache maintained on client device |
| Server-Side Cache | Cache maintained on backend |
| CDN | Distributed network that serves cached content |
| Redis | In-memory data store commonly used for caching |
| TTL | Time until a cache entry expires |
| Cache Eviction | Removing cache entries to free memory |
| LRU | Removes least recently used data |
| LFU | Removes least frequently used data |
| FIFO | Removes oldest inserted data |
| Cache Invalidation | Removes or updates stale cached data |
| Cache-Aside | Check cache, then database on a miss |
| Write-Through | Update cache and database on write path |
| Write-Back | Update cache first and database later |
| Pagination | Fetching records in smaller groups |
| LIMIT | Maximum number of records to retrieve |
| OFFSET | Number of records to skip |
| Cursor | Reference point for the next page |
| Infinite Scrolling | Automatically loads more data while scrolling |
| Jetpack Paging | Android library for paginated data loading |

---

# 25\. Day 5 Self-Assessment

Try answering these without looking at the notes.

## Basic Understanding

1. What is caching?
2. Why do we need caching?
3. What is a cache hit?
4. What is a cache miss?
5. What is cache hit ratio?
6. What is Redis?
7. What is TTL?
8. What is cache eviction?
9. Explain LRU and LFU.
10. What is cache invalidation?

## Intermediate Understanding

11. Explain client-side caching.
12. Explain server-side caching.
13. What is CDN caching?
14. Explain cache-aside.
15. Explain write-through caching.
16. Explain write-back caching.
17. What is the difference between cache-aside and write-through?
18. What happens when Redis becomes unavailable?
19. Why is cache invalidation difficult?

## Pagination

20. What is pagination?
21. Why should we not fetch 10,000 records in one API request?
22. Explain LIMIT and OFFSET.
23. What is offset-based pagination?
24. What is cursor-based pagination?
25. Why can large OFFSET values become inefficient?
26. Why is stable ordering important in pagination?
27. What is infinite scrolling?
28. What is the difference between offset and cursor pagination?

## System Design Thinking

29. How would you implement caching in a Notes application?
30. How would you invalidate the cache when a note is updated?
31. How would you load 1 million notes efficiently?
32. How would you design pagination for an infinite-scrolling feed?
33. How would you combine Redis and PostgreSQL?
34. What happens if the cache and database contain different values?
35. How would you prevent one user's cached notes from being returned to another user?

## Android-Specific Questions

36. What is the role of Room in local caching?
37. What is the difference between Room and in-memory caching?
38. What is Jetpack Paging 3?
39. What is PagingSource?
40. How would you implement offline-first pagination in an Android application?

---

# 26\. What You Should Be Able to Explain After Day 5

By the end of this lesson, you should be able to:

- Explain caching from scratch.
- Understand why caching improves performance.
- Explain cache hits and cache misses.
- Differentiate client-side and server-side caching.
- Explain Redis and its common use cases.
- Understand TTL and eviction policies.
- Explain cache invalidation.
- Understand cache-aside, write-through and write-back.
- Explain pagination.
- Differentiate offset and cursor pagination.
- Understand infinite scrolling.
- Explain how caching and pagination work together.
- Understand the Android Paging 3 library.
- Design a basic caching and pagination strategy for a Notes application.

**End of Day 5 – Caching and Pagination**

**Next: Day 6 – Notes Application: Complete System Design**

We will combine everything learned during Week 1:

1. Functional and non-functional requirements.
2. HLD and LLD.
3. Client-server architecture.
4. REST APIs.
5. SQL database design.
6. Caching with Redis.
7. Cursor-based pagination.
8. Complete Notes application architecture.
9. Request and response flow.
10. Interview-style system design explanation.

## How to study Day 5

Since this is your first system design preparation, focus especially on these concepts:

- Caching: Understand why we don't query the database every time.
- Redis: Understand its role between the backend and database.
- Cache invalidation: Understand what happens when a user edits a note.
- Pagination: Understand why we don't fetch 10,000 notes at once.
- Cursor vs offset: Understand why cursor pagination is useful for large datasets.

Practical exercise: Explain how your Android Notes application would load the first 20 notes, load the next 20 when scrolling, and refresh cached data when a note is edited.

If you can explain that flow clearly, you're building the system design thinking needed for interviews.
