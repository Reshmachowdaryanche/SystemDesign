# Week 3 – Day 16: Cache Strategies and Cache Invalidation

Week 3Day 16Beginner level

File name: `Week-3-Day-16-Cache-Strategies-and-Cache-Invalidation.md`

Today's goal: Understand how applications read and write cached data, how cache and database stay synchronized, and how to handle stale data.

Writing

# Week 3 – Day 16: Cache Strategies and Cache Invalidation

**Topic:** Cache Strategies and Cache Invalidation\
**Level:** Beginner\
**Target:** Android Developer – 7 Years Experience

---

# 1\. Introduction

In Day 15, we learned:

- What is caching?
- Why do we need caching?
- Cache Hit and Cache Miss.
- TTL (Time To Live).
- Cache Eviction.
- LRU and LFU.
- Redis basics.
- Cache Hit Ratio.
- Cache Stampede.

Today, we will understand something more important.

**How does an application actually use a cache?**

Imagine we are developing a Notes application.

A user creates a note.

The backend needs to:

1. Save the note in the database.
2. Update the cache.
3. Return the response to Android.

But in which order should these operations happen?

Should we update the cache first or database first?

What happens if the database succeeds but the cache update fails?

What happens if the cache contains old data?

These are the problems that cache strategies solve.

# 2\. What Is a Cache Strategy?

A cache strategy defines how an application interacts with the cache and database during read and write operations.

It determines:

- Where data is fetched from.
- When data is stored in the cache.
- When data is updated.
- When cached data is removed.
- How stale data is handled.

There are several caching strategies.

1. Cache-Aside.
2. Read-Through.
3. Write-Through.
4. Write-Back (Write-Behind).
5. Write-Around.

Let's understand each one using our Notes application.

---

# 3\. Cache-Aside Pattern (Lazy Loading)

## 3.1 What Is Cache-Aside?

Cache-Aside is a caching strategy where the application itself manages the cache.

The application first checks the cache.

If the data exists, it returns the cached data.

If the data does not exist, it fetches data from the database and stores the result in the cache.

This is also called Lazy Loading because data is loaded into the cache only when requested.

## 3.2 Architecture

text

```text
             APPLICATION
                  |
                  v
              CHECK CACHE
                  |
             +----+----+
             |         |
           HIT        MISS
             |         |
             v         v
        RETURN DATA  DATABASE
                       |
                       v
                  STORE IN CACHE
                       |
                       v
                   RETURN DATA
```

## 3.3 Step-by-Step Example

Imagine a user requests note 101.

**Step 1: Android sends a request.**

http

```http
GET /api/notes/101
```

**Step 2: Backend checks Redis.**

text

```text
GET note:101
```

Suppose Redis does not contain the note.

This is a Cache Miss.

**Step 3: Backend queries the database.**

sql

```sql
SELECT * FROM notes
WHERE id = 101;
```

**Step 4: Database returns the note.**

json

```json
{
  "id": 101,
  "title": "System Design",
  "description": "Learn caching"
}
```

**Step 5: Backend stores the result in Redis.**

text

```text
SET note:101
```

**Step 6: Backend returns the response to Android.**

The next request for note 101 may be served directly from Redis.

## 3.4 Cache-Aside Pseudocode

java

```java
Note getNote(int noteId) {

    // First check cache
    Note note = cache.get(noteId);

    if (note != null) {
        return note;
    }

    // Cache miss: fetch from database
    note = database.getNote(noteId);

    if (note != null) {
        // Store in cache
        cache.put(noteId, note);
    }

    return note;
}
```

## 3.5 Advantages

- Simple to implement.
- Only requested data is cached.
- Cache failure can be handled by falling back to the database.
- Works well for applications with frequently read data.

## 3.6 Disadvantages

- First request may be slow because of a cache miss.
- Application code must manage cache updates and invalidation.
- Cached data may become outdated.
- Multiple simultaneous cache misses can overload the database.

## 3.7 Real-World Usage

Useful for:

- User profiles.
- Product details.
- Frequently accessed notes.
- Application configuration.
- Frequently requested API responses.

**Interview answer:**

Cache-Aside means the application checks the cache first. If the data is missing, it fetches the data from the database and then stores it in the cache.

---

# 4\. Read-Through Cache

## 4.1 What Is Read-Through?

In Read-Through caching, the application requests data from the cache layer.

If the data exists, the cache returns it.

If the data does not exist, the cache layer itself loads the data from the database and stores it.

The application does not directly handle the cache-miss loading logic.

## 4.2 Architecture

text

```text
           APPLICATION
                |
                v
             CACHE
                |
           +----+----+
           |         |
          HIT       MISS
           |         |
           v         v
        RETURN    DATABASE
                     |
                     v
                STORE IN CACHE
                     |
                     v
                  RETURN
```

## 4.3 Step-by-Step Example

User requests note 101.

1. Application sends a request to the cache layer.
2. Cache checks whether note 101 exists.
3. If found, cache returns it.
4. If missing, the cache layer fetches note 101 from the database.
5. Cache stores the result.
6. Cache returns the note to the application.

## 4.4 Advantages

- Application code becomes simpler.
- Cache loading logic is centralized.
- Repeated reads can be served from the cache.

## 4.5 Disadvantages

- Requires cache infrastructure that supports database loading.
- First read may be slower.
- Cache loading and failure handling need careful configuration.

## 4.6 Cache-Aside vs Read-Through

| Cache-Aside | Read-Through |
| --- | --- |
| Application manages cache misses | Cache layer manages cache misses |
| Application fetches database data | Cache layer fetches database data |
| More application responsibility | More cache-layer responsibility |
| Flexible implementation | Centralized loading behavior |

**Important:**

Both strategies can look very similar from the outside. The main difference is who is responsible for loading missing data.

---

# 5\. Write-Through Cache

## 5.1 What Is Write-Through?

In Write-Through caching, every write operation updates both the cache and the database through the caching layer.

The idea is to keep cache and database synchronized during writes.

## 5.2 Architecture

text

```text
          APPLICATION
               |
               v
             CACHE
               |
         +-----+-----+
         |           |
         v           v
       CACHE      DATABASE
       UPDATE      UPDATE
```

## 5.3 Step-by-Step Example

A user updates a note.

Old note:

json

```json
{
  "id": 101,
  "title": "Old Title"
}
```

New note:

json

```json
{
  "id": 101,
  "title": "New Title"
}
```

**Step 1:** Android sends an update request.

http

```http
PUT /api/notes/101
```

**Step 2:** Application sends the update through the write-through cache layer.

**Step 3:** Cache layer updates the database.

**Step 4:** Cache layer updates the cached value.

**Step 5:** Application receives the result according to the configured success policy.

Now the cache contains the updated note.

## 5.4 Advantages

- Cache is updated whenever writes succeed.
- Reduces the chance of serving old data.
- Useful when recently written data is frequently read.

## 5.5 Disadvantages

- Write operations may take longer.
- Every write creates cache maintenance overhead.
- Handling partial failures requires care.
- Some data may be written to the cache even if it is rarely read.

## 5.6 Real-World Usage

Useful when:

- Data is frequently read after being updated.
- Keeping the cache fresh is important.
- Read latency matters.

**Interview answer:**

Write-Through updates the database and cache as part of the same write path. It helps keep cached data fresh but can increase write latency.

---

# 6\. Write-Back Cache (Write-Behind)

## 6.1 What Is Write-Back?

Write-Back caching means the application writes data to the cache first.

The database is updated later, usually asynchronously.

The cache temporarily becomes the place holding the latest changes before they are persisted to the database.

## 6.2 Architecture

text

```text
           APPLICATION
                |
                v
               CACHE
                |
                v
          WRITE SUCCESS
                |
                v
         ASYNCHRONOUS UPDATE
                |
                v
             DATABASE
```

## 6.3 Step-by-Step Example

A user updates a note.

**Step 1:** Android sends an update request.

**Step 2:** Backend updates the cache.

**Step 3:** Cache acknowledges the write.

**Step 4:** Backend returns success to the application.

**Step 5:** A background process updates the database later.

This means the database may temporarily contain older data than the cache.

## 6.4 Advantages

- Very fast write acknowledgement.
- Multiple updates may be combined before database persistence.
- Reduces immediate database write load.
- Useful for high-write-volume workloads.

## 6.5 Disadvantages

- If cache data is lost before persistence, recent writes may be lost.
- More complex recovery mechanisms are required.
- Cache and database can temporarily contain different values.
- Requires durable buffering or another reliable persistence mechanism for important data.

## 6.6 Real-World Examples

Write-back-style designs are useful in:

- High-volume counters.
- Analytics aggregation.
- Some logging systems.
- Workloads where delayed persistence is acceptable.

For important user-created notes, blindly acknowledging success before durable storage is risky.

**Interview answer:**

Write-Back stores data in the cache first and updates the database asynchronously later. It improves write performance but introduces a risk of losing unpersisted data if not designed carefully.

---

# 7\. Write-Around Cache

## 7.1 What Is Write-Around?

In Write-Around caching, write operations go directly to the database.

The cache is not updated during the write.

When someone reads that data later, it can be loaded into the cache.

## 7.2 Architecture

text

```text
           APPLICATION
                |
                v
             DATABASE
                |
                v
          WRITE COMPLETED


       READ REQUEST LATER
                |
                v
              CACHE
                |
             MISS
                |
                v
             DATABASE
                |
                v
           STORE IN CACHE
```

## 7.3 Example

Imagine a Notes application.

A user creates 100 notes.

Most of these notes are rarely accessed.

If every newly created note is immediately added to the cache, memory may be wasted.

With Write-Around:

1. New notes are written directly to the database.
2. Cache is not updated during creation.
3. When a note is requested, the cache is checked.
4. On a miss, the database is queried.
5. The result is added to the cache.

## 7.4 Advantages

- Avoids filling cache with rarely accessed data.
- Reduces cache write operations.
- Useful for write-heavy workloads.

## 7.5 Disadvantages

- First read after writing may be slower.
- Cached data can become stale if old entries are not invalidated.
- Requires a clear cache invalidation strategy.

---

# 8\. Comparison of All Cache Strategies

This is one of the most important tables for interviews.

| Strategy | Read Behavior | Write Behavior |
| --- | --- | --- |
| Cache-Aside | Application checks cache; loads on miss | Application manages database and cache |
| Read-Through | Cache loads data on miss | Depends on write strategy |
| Write-Through | Cache serves reads | Updates cache and database through write path |
| Write-Back | Cache serves reads | Updates cache first, database later |
| Write-Around | Cache serves reads; loads on miss | Writes directly to database |

## 8.1 Easy Way to Remember

- Cache-Aside: Application manages cache.
- Read-Through: Cache loads missing data.
- Write-Through: Write to cache and database.
- Write-Back: Write to cache first, database later.
- Write-Around: Write directly to database, bypassing cache.

---

# 9\. What Is Cache Invalidation?

Now let's understand one of the most important problems in caching.

Cache invalidation means removing or updating cached data when the original data changes.

Imagine the database contains:

json

```json
{
  "id": 101,
  "title": "System Design"
}
```

Redis also contains:

json

```json
{
  "id": 101,
  "title": "System Design"
}
```

Now the user changes the title to:

json

```json
{
  "id": 101,
  "title": "Advanced System Design"
}
```

The database gets updated.

But what happens if Redis still contains the old title?

The application may return outdated information.

This is called stale cache data.

Cache invalidation helps solve this problem.

---

# 10\. Cache Invalidation Strategies

There are several ways to invalidate cached data.

## 10.1 TTL-Based Invalidation

TTL means Time To Live.

Every cache entry has an expiration time.

Example:

text

```text
KEY: note:101

VALUE: System Design

TTL: 300 seconds
```

After five minutes, the entry expires.

The next request fetches fresh data from the database.

Advantages:

- Simple to implement.
- Automatically removes old entries.
- Reduces the risk of stale data remaining forever.

Disadvantages:

- Data may remain stale until expiration.
- Short TTL creates more database requests.
- Long TTL can increase the chance of outdated data being served.

## 10.2 Explicit Invalidation

The application explicitly removes the cached entry after a successful database update.

Example:

User updates note 101.

text

```text
1. Update database.
2. Delete cache key note:101.
```

Next request:

text

```text
1. Check cache.
2. Cache miss.
3. Fetch latest note from database.
4. Store latest result in cache.
```

Advantages:

- Old cached data is removed quickly.
- Easy to understand.
- Commonly used with Cache-Aside.

Disadvantages:

- If cache deletion fails, stale data may remain.
- Multiple related cache keys can make invalidation complicated.

## 10.3 Update-Based Invalidation

Instead of deleting the cache entry, directly update it with the new value.

Example:

Old cache:

json

```json
{
  "id": 101,
  "title": "Old Title"
}
```

After the update:

json

```json
{
  "id": 101,
  "title": "New Title"
}
```

Advantages:

- The next read can immediately use updated data.
- Avoids a cache miss after the update.

Disadvantages:

- More complicated when multiple cache keys depend on the same data.
- Partial failures can create inconsistency.

## 10.4 Event-Based Invalidation

An event is published whenever important data changes.

Example:

text

```text
User updates a note.
        |
        v
Notes Service
        |
        v
NoteUpdated Event
        |
        v
Message Queue
        |
        v
Cache Consumer
        |
        v
Invalidate or update Redis
```

Advantages:

- Useful in distributed systems.
- Other services can react to data changes.
- Reduces direct dependency between services.

Disadvantages:

- Event processing may be delayed.
- Events can fail or be delivered more than once.
- Requires retry and reliable event-processing mechanisms.

---

# 11\. Cache Consistency Problems

Cache consistency means keeping cached data sufficiently aligned with the original data source.

Let's understand common problems.

## 11.1 Stale Data

Database:

text

```text
title = New Title
```

Cache:

text

```text
title = Old Title
```

The application returns outdated information.

Solution:

- TTL.
- Explicit invalidation.
- Update cache after database writes.
- Versioning where appropriate.

## 11.2 Race Condition

Imagine two users update the same note.

User A updates the note to Title A.

User B updates the note to Title B.

Both operations happen almost simultaneously.

Depending on operation ordering, the cache could end up with a value that does not match the database.

Solution:

- Version numbers.
- Optimistic locking.
- Controlled update ordering.
- Carefully designed cache update logic.

## 11.3 Database Update Succeeds, Cache Invalidation Fails

Imagine:

text

```text
1. Database updated successfully.
2. Cache deletion fails.
3. Old cached data remains.
```

Now the system may serve stale data.

Possible solutions:

- Retry cache invalidation.
- Use short TTLs.
- Publish reliable invalidation events.
- Monitor invalidation failures.

## 11.4 Cache Update Succeeds, Database Update Fails

Imagine:

text

```text
1. Cache updated successfully.
2. Database update fails.
```

Now cache contains new data while the database contains old data.

This is dangerous.

A common solution is to update the database first and then invalidate the cache, rather than blindly updating cache first.

---

# 12\. Cache-Aside: Recommended Update Flow

For many Notes applications, Cache-Aside with database-first writes and cache invalidation is a reasonable starting design.

Let's understand it.

User updates note 101.

text

```text
          ANDROID APP
                |
                v
           BACKEND API
                |
                v
          UPDATE DATABASE
                |
                v
          DATABASE SUCCESS
                |
                v
          DELETE CACHE KEY
                |
                v
          RETURN SUCCESS
```

Later, when the user requests note 101:

text

```text
          ANDROID APP
                |
                v
           BACKEND API
                |
                v
           CHECK REDIS
                |
                v
             CACHE MISS
                |
                v
          FETCH DATABASE
                |
                v
          STORE IN REDIS
                |
                v
          RETURN LATEST DATA
```

Important:

Database-first plus cache deletion is a common pattern, but it does not eliminate every race condition or invalidation failure.

For critical data, additional mechanisms may be needed.

---

# 13\. What Is Double Delete?

Double Delete is a technique sometimes used to reduce stale cache problems caused by concurrent operations.

The simplified idea:

1. Delete the cache entry.
2. Update the database.
3. Delete the cache entry again after a delay.

Why delete twice?

Imagine another request reads old database data between the first cache deletion and the database update.

It might put that old data back into the cache.

The second deletion removes that stale entry.

However, this is not a universal solution. Delays do not provide strict consistency guarantees, and the approach needs careful coordination.

For stronger guarantees, use appropriate versioning, serialization, reliable event processing or other consistency mechanisms.

---

# 14\. What Is Cache Penetration?

Cache Penetration happens when requests repeatedly ask for data that does not exist in either the cache or the database.

Example:

A client repeatedly requests:

http

```http
GET /api/notes/999999
```

Suppose note 999999 does not exist.

Every request:

1. Checks cache.
2. Gets a cache miss.
3. Queries the database.
4. Finds no record.

This increases database load.

### Solutions

**1\. Cache negative results**

Store a short-lived marker indicating that the record does not exist.

Example:

text

```text
KEY: note:999999

VALUE: NOT_FOUND

TTL: 60 seconds
```

**2\. Validate input**

Reject obviously invalid identifiers or malformed requests.

**3\. Bloom Filter**

A Bloom Filter is a space-efficient probabilistic data structure that can help determine whether an item is definitely absent or possibly present in a set.

It may return false positives, but it does not return false negatives under normal operation.

It can reduce unnecessary database queries for missing records.

---

# 15\. Cache Breakdown vs Cache Stampede vs Cache Penetration

These three concepts are commonly asked in interviews.

| Problem | Meaning | Example |
| --- | --- | --- |
| Cache Penetration | Requests for data that does not exist | Repeatedly requesting missing note IDs |
| Cache Breakdown | A very popular key expires or becomes unavailable | Popular note suddenly causes database traffic |
| Cache Stampede | Many requests simultaneously miss and fetch the same data | Thousands of users query one expired key |

### Solutions

| Problem | Possible Solutions |
| --- | --- |
| Penetration | Negative caching, validation, Bloom Filter |
| Breakdown | Background refresh, controlled expiration, stale data where safe |
| Stampede | Request coalescing, TTL jitter, distributed locking where appropriate |

Note: Cache breakdown and cache stampede are sometimes used interchangeably in engineering discussions. The important distinction is the underlying failure scenario.

---

# 16\. Choosing the Right Cache Strategy

There is no single best caching strategy for every application.

It depends on the workload.

| Requirement | Suitable Strategy |
| --- | --- |
| Frequently read data | Cache-Aside or Read-Through |
| Data frequently read after writing | Write-Through |
| Very high write volume with delayed persistence allowed | Write-Back |
| Data rarely read after writing | Write-Around |
| Simple backend caching | Cache-Aside |
| Centralized cache-miss loading | Read-Through |

### Interview Tip

Never say that one caching strategy is always the best.

Explain the requirement first, then select the appropriate strategy.

---

# 17\. Complete Notes Application Example

Let's combine the concepts.

Requirements:

1. User can create notes.
2. User can update notes.
3. User can view notes.
4. User can search notes.
5. Users can access notes from multiple devices.
6. The application should respond quickly.

Architecture:

text

```text
                ANDROID APP
                     |
                     v
                 BACKEND API
                     |
              +------+------+
              |             |
              v             v
          READ FLOW      WRITE FLOW
              |             |
              v             v
            REDIS        DATABASE
              |             |
          +---+---+         |
          |       |         |
         HIT     MISS       |
          |       |         |
          v       v         |
        RETURN  DATABASE    |
                  |         |
                  v         |
                REDIS       |
                  |         |
                  v         |
                RETURN      |
                            |
                            v
                      CACHE INVALIDATION
                            |
                            v
                       REDIS DELETE
```

## Scenario A: User Reads a Note

1. Android sends GET request.
2. Backend checks Redis.
3. If found, return cached data.
4. If missing, fetch from database.
5. Store the result in Redis.
6. Return the note.

Strategy: Cache-Aside.

## Scenario B: User Updates a Note

1. Android sends PUT request.
2. Backend updates the database.
3. After successful database update, backend invalidates the corresponding cache key.
4. Backend returns success.
5. Next read loads the latest note into the cache.

Strategy: Cache-Aside with explicit invalidation.

## Scenario C: User Deletes a Note

1. Android sends DELETE request.
2. Backend deletes or marks the note deleted in the database.
3. Backend invalidates the cached note.
4. Subsequent reads must not return the deleted note.

## Scenario D: Redis Is Unavailable

1. Backend cannot access Redis.
2. Backend may fall back to the database.
3. Backend should use timeouts and controlled fallback.
4. Monitor database load because traffic may increase.
5. Recover cache service and gradually repopulate data.

---

# 18\. Common Interview Mistakes

1. Confusing Cache-Aside with Read-Through.
2. Confusing Write-Through with Write-Back.
3. Assuming updating the database automatically updates Redis.
4. Forgetting cache invalidation.
5. Ignoring stale data.
6. Treating cache and database updates as automatically atomic.
7. Assuming TTL guarantees immediate consistency.
8. Ignoring concurrent updates.
9. Using Write-Back for critical data without durability guarantees.
10. Forgetting what happens when Redis is unavailable.
11. Ignoring cache stampede.
12. Assuming every database record should be cached.

---

# 19\. Important Interview Questions and Answers

### Q1. What is Cache-Aside?

The application checks the cache first. If data is missing, the application fetches it from the database and stores it in the cache.

### Q2. What is Read-Through?

The application requests data from the cache layer, and the cache layer loads missing data from the database.

### Q3. What is Write-Through?

Writes update the database and cache through the write path before the operation is acknowledged according to the configured success policy.

### Q4. What is Write-Back?

Writes update the cache first, and database persistence happens asynchronously later.

### Q5. What is Write-Around?

Writes bypass the cache and go directly to the database. Data may enter the cache when it is subsequently read.

### Q6. What is the difference between Cache-Aside and Read-Through?

In Cache-Aside, the application manages cache misses. In Read-Through, the cache layer manages cache misses.

### Q7. What is the difference between Write-Through and Write-Back?

Write-Through updates cache and database in the write path. Write-Back updates cache first and persists data to the database later.

### Q8. What is cache invalidation?

Removing or updating cached data when the original data changes.

### Q9. Why is cache invalidation difficult?

Because multiple application instances and services may access or update the same data, making it difficult to ensure that every cached copy is fresh.

### Q10. How would you invalidate Redis data after updating a note?

Update the database successfully and then delete the corresponding Redis key. For more complex distributed systems, use reliable invalidation events and retries.

### Q11. What is stale cache data?

Cached data that no longer matches the current data in the source of truth.

### Q12. What is Cache Penetration?

Repeated requests for data that does not exist in either cache or database.

### Q13. What is Cache Stampede?

Many requests simultaneously fetch the same data after a cache entry expires or becomes unavailable.

### Q14. How can you prevent Cache Stampede?

Use request coalescing, TTL jitter, background refresh and carefully designed locking.

### Q15. What happens if database update succeeds but cache invalidation fails?

The cache may contain stale data. Retry invalidation, use TTL as a fallback and consider reliable event-based invalidation.

### Q16. Why is Write-Back risky?

Because recent changes may be lost if the cache fails before those changes are persisted to the database.

### Q17. Which caching strategy would you choose for a Notes application?

Cache-Aside is a reasonable starting point because note reads can be served from Redis, and the backend can invalidate cache entries after successful updates.

### Q18. Does TTL guarantee cache consistency?

No. TTL limits how long an entry remains valid, but stale data can still be served before expiration.

### Q19. Can we update the cache before updating the database?

It is possible, but if the database update fails, the cache may contain incorrect data. The write order must be designed according to consistency and durability requirements.

### Q20. What is event-based cache invalidation?

A service publishes an event when data changes. A consumer processes that event and invalidates or updates the relevant cache entries.

---

# 20\. Scenario-Based Interview Questions

## Scenario 1: User Updates a Note

A user updates a note, but the next GET request returns the old title.

What could be wrong?

Expected answer:

The database may have been updated, but the cache was not invalidated or refreshed. We should implement appropriate cache invalidation.

## Scenario 2: Redis Goes Down

Your Redis server becomes unavailable.

Should the application stop working?

Expected answer:

Not necessarily. The backend can fall back to the database if designed for it. Use timeouts, controlled fallback and monitoring to prevent overwhelming the database.

## Scenario 3: Popular Note Expires

A popular note is requested by 50,000 users.

Its cache entry expires.

All 50,000 requests query the database.

What is this problem?

Expected answer:

Cache Stampede.

Possible solutions include request coalescing, TTL jitter and background refresh.

## Scenario 4: Too Many Unused Records in Cache

Your application stores every newly created note in Redis, but most notes are never opened again.

What can you do?

Expected answer:

Consider Write-Around or lazy cache population so that only requested data enters the cache.

## Scenario 5: Database Update Succeeds but Redis Delete Fails

What happens?

Expected answer:

Old cached data may remain. We should retry invalidation and consider reliable event processing and TTL as additional protection.

## Scenario 6: High Write Traffic

Your application receives millions of updates per hour. Database writes are becoming expensive.

Can Write-Back help?

Expected answer:

It may help if delayed persistence is acceptable. However, we need durable buffering, retries and recovery mechanisms to prevent data loss.

---

# 21\. Day 16 – Quick Revision Cheat Sheet

| Concept | Simple Meaning |
| --- | --- |
| Cache-Aside | Application manages cache |
| Read-Through | Cache loads missing data |
| Write-Through | Update cache and database in write path |
| Write-Back | Update cache first, database later |
| Write-Around | Write directly to database |
| Cache Invalidation | Remove or update stale cache |
| TTL | Expiration duration |
| Stale Data | Outdated cached information |
| Event-Based Invalidation | Use events to invalidate cache |
| Cache Penetration | Repeated requests for nonexistent data |
| Cache Breakdown | Popular cache key expires or becomes unavailable |
| Cache Stampede | Many requests fetch the same missing data |
| Negative Caching | Temporarily cache that data does not exist |
| Bloom Filter | Probabilistic membership-checking structure |

---

# 22\. Final Learning Checklist

## Cache Strategies

- [ ]Understand Cache-Aside.
- [ ]Understand Read-Through.
- [ ]Understand Write-Through.
- [ ]Understand Write-Back.
- [ ]Understand Write-Around.
- [ ]Understand their advantages and disadvantages.
- [ ]Understand when to choose each strategy.

## Cache Invalidation

- [ ]Understand TTL-based invalidation.
- [ ]Understand explicit invalidation.
- [ ]Understand update-based invalidation.
- [ ]Understand event-based invalidation.
- [ ]Understand stale cache data.
- [ ]Understand cache consistency.
- [ ]Understand concurrent update problems.

## Advanced Cache Problems

- [ ]Understand Cache Penetration.
- [ ]Understand Cache Breakdown.
- [ ]Understand Cache Stampede.
- [ ]Understand negative caching.
- [ ]Understand Bloom Filters.
- [ ]Understand request coalescing.

## Interview Preparation

- [ ]Explain Cache-Aside with an architecture diagram.
- [ ]Explain Write-Through vs Write-Back.
- [ ]Explain Cache-Aside vs Read-Through.
- [ ]Explain how you invalidate cache after a database update.
- [ ]Explain what happens if Redis goes down.
- [ ]Explain how to prevent Cache Stampede.
- [ ]Design caching for a Notes application.

---

# 23\. What Is Next?

## Week 3 – Day 17: CDN and Distributed Caching

Tomorrow we will learn:

1. What is Redis in detail?
2. What is distributed caching?
3. Local cache vs distributed cache.
4. Redis architecture.
5. Redis replication.
6. Redis clustering.
7. What is a CDN?
8. CDN edge servers.
9. CDN vs Redis.
10. Cache Stampede prevention.
11. Real-world image delivery architecture.

**Day 16 Complete – Cache Strategies and Cache Invalidation.**

## Before moving to Day 17

Make sure you understand these three interview questions particularly well:

1. What is the difference between Cache-Aside and Read-Through?
2. What is the difference between Write-Through and Write-Back?
3. A user updates a note. How do you ensure Redis doesn't return the old note?

These three questions are fundamental to understanding caching architecture.
