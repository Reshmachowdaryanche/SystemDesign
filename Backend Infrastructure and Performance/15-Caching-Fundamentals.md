# Week 3 – Day 15: Caching Fundamentals

Week 3Day 15Beginner level

File name: `Week-3-Day-15-Caching-Fundamentals.md`

Today's goal: Understand what caching is, why we need it, how it works, and how it improves application performance.

Writing

# Week 3 – Day 15: Caching Fundamentals

**Topic:** Caching Fundamentals\
**Level:** Beginner\
**Target:** Android Developer – 7 Years Experience

---

# 1\. What Is Caching?

Let's start with a simple example.

Imagine you are building a Notes application.

A user opens the application and requests their profile information.

The Android application sends an API request:

http

```http
GET /api/users/101
```

The backend fetches the user information from the database and returns it.

json

```json
{
  "id": 101,
  "name": "John",
  "email": "john@example.com"
}
```

Now imagine the user closes and reopens the application.

The application requests the same profile information again.

The backend queries the database again.

What if the same information could be reused instead of repeatedly querying the database?

This is where caching comes into the picture.

## Definition

Caching is a technique of temporarily storing frequently accessed data in a faster storage location so that future requests can retrieve that data more quickly.

The stored data is called a cache.

### Simple Real-World Example

Imagine you are studying for an interview.

You repeatedly refer to the same important notes.

Instead of searching through an entire textbook every time, you keep the important pages on your desk.

The next time you need them, you can access them immediately.

Caching works similarly.

Instead of fetching data from a slower source every time, we keep frequently used data in a faster location.

---

# 2\. Why Do We Need Caching?

Let's imagine our Notes application has:

- 1 million users.
- Each user opens the application 10 times a day.
- Each application opening requests the user's profile.

That means:

text

```text
1,000,000 users × 10 requests

= 10,000,000 requests per day
```

Imagine every request queries the database.

The database has to process 10 million profile requests.

This creates unnecessary database load, especially when the profile information rarely changes.

We can introduce caching.

text

```text
WITHOUT CACHE

ANDROID APP
     |
     v
   BACKEND
     |
     v
  DATABASE
     |
     v
   RESPONSE
```

With caching:

text

```text
WITH CACHE

ANDROID APP
     |
     v
   BACKEND
     |
     v
    CACHE
     |
  +--+--+
  |     |
 HIT   MISS
  |     |
  v     v
DATA  DATABASE
        |
        v
      CACHE
        |
        v
      RESPONSE
```

Now frequently accessed information can be served from the cache.

## Benefits of Caching

1. Faster response times.
2. Reduced database load.
3. Improved application performance.
4. Reduced backend resource consumption.
5. Better scalability.
6. Reduced repeated computation.
7. Better user experience.

---

# 3\. What Happens Without Caching?

Let's understand the problem in detail.

Suppose 10,000 users request the same popular note.

Without caching:

text

```text
USER 1  ----+
USER 2  ----+
USER 3  ----+
USER 4  ----+
USER 5  ----+
             |
             v
          BACKEND
             |
             v
          DATABASE
             |
       10,000 QUERIES
```

The database receives 10,000 queries.

This can increase:

- Database CPU usage.
- Memory usage.
- Database connections.
- Query execution time.
- Overall backend latency.

With caching:

text

```text
USER 1  ----+
USER 2  ----+
USER 3  ----+
USER 4  ----+
USER 5  ----+
             |
             v
          BACKEND
             |
             v
            CACHE
             |
             v
        CACHED RESULT
```

If the cached data is still valid, the database does not need to execute the same query again.

Important:

Caching does not eliminate database queries completely. It reduces repeated queries for data that can safely be reused.

---

# 4\. What Is a Cache Hit?

A Cache Hit happens when the requested data is already available in the cache.

Example:

A user requests their profile.

The backend checks the cache.

text

```text
CACHE:
user:101 -> John
```

The requested information is found.

The backend returns the cached data.

text

```text
REQUEST
   |
   v
CHECK CACHE
   |
   v
DATA FOUND
   |
   v
CACHE HIT
   |
   v
RETURN RESPONSE
```

Advantages:

- Faster response.
- No database query needed for that request.
- Lower database load.

---

# 5\. What Is a Cache Miss?

A Cache Miss happens when the requested data is not available in the cache.

Example:

A user requests note 101.

The cache does not contain that note.

The backend queries the database.

text

```text
REQUEST
   |
   v
CHECK CACHE
   |
   v
DATA NOT FOUND
   |
   v
CACHE MISS
   |
   v
QUERY DATABASE
   |
   v
SAVE RESULT IN CACHE
   |
   v
RETURN RESPONSE
```

The next request for the same note may be served directly from the cache.

---

# 6\. Cache Hit vs Cache Miss

| Cache Hit | Cache Miss |
| --- | --- |
| Data exists in cache | Data does not exist in cache |
| Returns cached data | Fetches data from another source |
| Usually faster | Usually slower |
| Reduces database queries | May require a database query |

### Interview Question

What is the difference between Cache Hit and Cache Miss?

**Answer:**

A cache hit occurs when requested data is found in the cache. A cache miss occurs when the data is not available in the cache, so the system must retrieve it from another source, such as a database.

---

# 7\. What Is Cache Latency?

Latency is the time taken to complete an operation.

In caching, cache latency means the time required to retrieve data from the cache.

Example:

Imagine:

text

```text
Database response time = 100 ms
Cache response time = 5 ms
```

If the requested data is available in the cache, the system can respond much faster.

### Comparison

| Storage | Illustrative Access Time |
| --- | --- |
| CPU cache | Nanoseconds |
| RAM | Nanoseconds |
| Redis | Often sub-millisecond to a few milliseconds |
| Database | Depends on query and infrastructure |
| Disk storage | Depends on storage and access pattern |

These are approximate examples, not guaranteed performance numbers.

The important concept is that accessing data from memory is generally faster than repeatedly retrieving it from disk-backed storage.

---

# 8\. Where Can We Store Cached Data?

Caching can happen at different levels of a system.

Let's understand each one.

## 8.1 Browser Cache

Web browsers store resources such as:

- Images.
- CSS files.
- JavaScript files.
- Other static resources.

When a user visits the same website again, the browser may reuse cached resources instead of downloading everything again.

## 8.2 Client-Side Cache

The application itself can store frequently accessed data locally.

For Android:

- In-memory cache.
- Room database.
- DataStore for suitable small application data.
- File-based cache.

Example:

The Notes application stores recently accessed notes locally.

When the user opens the application again, the application can display the locally stored notes immediately while fetching updates from the backend.

## 8.3 Server-Side Cache

The backend stores frequently accessed information in memory or a dedicated caching system.

Example:

text

```text
ANDROID APP
     |
     v
BACKEND SERVER
     |
     v
REDIS CACHE
     |
     v
DATABASE
```

Redis is a commonly used server-side caching technology.

## 8.4 CDN Cache

A CDN (Content Delivery Network) stores copies of static content across geographically distributed servers.

Example:

An application contains profile pictures.

Instead of every user downloading images from one central server, a nearby CDN edge server may serve cached images.

This improves content delivery performance.

---

# 9\. Types of Cache Based on Location

| Cache Type | Location | Example |
| --- | --- | --- |
| CPU Cache | Inside CPU | L1, L2, L3 |
| Browser Cache | User's browser | Images, CSS |
| Client Cache | User's device | Android local cache |
| Server Cache | Backend infrastructure | Redis |
| CDN Cache | Distributed edge servers | Images, videos |

Remember:

Caching is not limited to backend systems. It exists at multiple levels of the computing architecture.

---

# 10\. What Is In-Memory Caching?

In-memory caching means storing data in RAM instead of repeatedly reading it from slower storage.

RAM provides fast data access.

Example:

Imagine a backend service frequently needs application configuration.

Instead of querying the database every time, the service keeps the configuration in memory.

text

```text
BACKEND SERVICE
      |
      v
  IN-MEMORY CACHE
      |
      v
  RETURN CONFIG
```

Examples:

- Java HashMap.
- Caffeine Cache.
- Guava Cache.
- Redis.

Important difference:

A Java HashMap inside a backend process is local to that process.

Redis is a separate service that can be accessed by multiple backend instances.

---

# 11\. What Is Redis?

Redis is an in-memory data store commonly used for caching.

It supports different data structures, including:

- Strings.
- Hashes.
- Lists.
- Sets.
- Sorted sets.

For system design interviews, understand Redis as a commonly used fast data store that can reduce repeated database access.

### Example

Suppose a user requests profile information.

Backend checks Redis:

text

```text
KEY:
user:101

VALUE:
{
  "name": "John",
  "email": "john@example.com"
}
```

If the value exists, the backend can return it.

Otherwise, it queries the database and may store the result in Redis.

### Redis Architecture

text

```text
          ANDROID APP
               |
               v
          BACKEND API
               |
               v
           REDIS CACHE
               |
          +----+----+
          |         |
         HIT       MISS
          |         |
          v         v
       RETURN    DATABASE
                   |
                   v
                REDIS
                   |
                   v
                RETURN
```

We will study Redis and distributed caching in more detail on Day 17.

---

# 12\. What Is TTL (Time To Live)?

TTL stands for Time To Live.

It defines how long a cached entry should remain valid.

Example:

text

```text
Cache key: user:101
TTL: 300 seconds
```

This means the cache entry expires after five minutes.

Why do we need TTL?

Because cached data may become outdated.

Imagine:

At 10:00 AM, the user changes their profile name.

But the cache still contains the old name.

If we never remove or update the cached entry, users may continue seeing outdated information.

TTL helps prevent cached entries from remaining indefinitely.

### Example Timeline

text

```text
10:00 AM
   |
   v
DATA STORED IN CACHE
   |
   v
TTL = 5 MINUTES
   |
   v
10:05 AM
   |
   v
CACHE ENTRY EXPIRES
   |
   v
NEXT REQUEST FETCHES FRESH DATA
```

Important:

TTL expiration does not necessarily mean data is immediately physically deleted from every cache implementation. It means the entry is no longer considered valid for normal retrieval.

---

# 13\. What Is Cache Eviction?

Cache eviction means removing existing entries from a cache to free up memory or enforce a cache policy.

Imagine:

Your cache can store only 1,000 entries.

Now 1,001 different items need to be stored.

The cache must decide which existing item to remove.

This is called cache eviction.

## 13.1 LRU – Least Recently Used

LRU removes the item that has not been accessed for the longest time.

Example:

Cache contains:

text

```text
A - Last accessed 10 minutes ago
B - Last accessed 2 minutes ago
C - Last accessed 1 second ago
```

If space is needed, LRU removes A.

Why?

Because A was accessed least recently.

## 13.2 LFU – Least Frequently Used

LFU removes the item with the lowest access frequency.

Example:

text

```text
A - Accessed 2 times
B - Accessed 100 times
C - Accessed 50 times
```

LFU removes A.

Why?

Because A has been accessed the fewest times.

## 13.3 FIFO – First In First Out

FIFO removes the item that entered the cache first.

Example:

text

```text
A - Added first
B - Added second
C - Added third
```

If space is required, FIFO removes A.

## 13.4 Comparison

| Policy | Removal Rule |
| --- | --- |
| LRU | Least recently accessed |
| LFU | Least frequently accessed |
| FIFO | First item inserted |

### Interview Question

What is the difference between LRU and LFU?

**Answer:**

LRU removes the item that has not been accessed for the longest time, whereas LFU removes the item with the lowest access frequency.

---

# 14\. What Is Cache Invalidation?

Cache invalidation means removing or updating cached data when the original data changes.

Imagine:

The database contains:

json

```json
{
  "userId": 101,
  "name": "John"
}
```

The cache also contains:

json

```json
{
  "userId": 101,
  "name": "John"
}
```

Now the user changes the name to David.

The database is updated:

json

```json
{
  "userId": 101,
  "name": "David"
}
```

But the cache still contains John.

Now the system has inconsistent data.

We must update or invalidate the cache.

text

```text
DATABASE:
David

CACHE:
John

      |
      v

INVALIDATE OR UPDATE CACHE

      |
      v

DATABASE:
David

CACHE:
David
```

There are different strategies for handling this.

We will learn cache-aside, write-through and write-back on Day 16.

---

# 15\. What Is Cache Hit Ratio?

Cache Hit Ratio tells us how many requests were successfully served from the cache.

Formula:

Cache Hit Ratio = Cache Hits / Total Cache Requests × 100

Example:

Total cache requests = 1,000

Cache hits = 800

Cache misses = 200

text

```text
Hit Ratio = 800 / 1000 × 100

          = 80%
```

The cache hit ratio is 80%.

This means 80% of requests were served from the cache.

## 15.1 Why Is It Important?

A higher hit ratio often means:

- More requests are served from the cache.
- Fewer requests reach the database.
- Database load may decrease.
- Average response time may improve.

But a high hit ratio alone does not guarantee good performance.

For example, if cached requests are still slow, the system may have other bottlenecks.

We should also monitor:

- Cache latency.
- Database latency.
- Cache memory usage.
- Eviction count.
- Error rate.
- Cache miss rate.

---

# 16\. What Is Cache Stampede?

Cache Stampede happens when a popular cache entry expires and many requests try to fetch the same data from the database simultaneously.

Example:

A popular note has 10,000 users.

Its cached entry expires.

Suddenly:

text

```text
USER 1  ----+
USER 2  ----+
USER 3  ----+
USER 4  ----+
USER 5  ----+
             |
             v
          CACHE MISS
             |
             v
          DATABASE
```

Thousands of requests may query the database at the same time.

This creates a sudden database load.

### How Can We Prevent It?

Common techniques:

1. Cache expiration jitter – Add small random variations to TTL values.
2. Request coalescing – Allow one request to fetch the data while others wait for the same result.
3. Background refresh – Refresh frequently accessed data before it expires.
4. Stale-while-revalidate – Serve slightly outdated data while refreshing it, when business requirements allow.

We will explore this further on Day 17.

---

# 17\. Cache vs Database

This is an important interview question.

| Cache | Database |
| --- | --- |
| Primarily optimized for fast access | Designed for persistent data storage |
| Often stores temporary or reusable data | Stores the application's durable data |
| Usually stores a subset of data | Usually stores the source records |
| Often memory-based | Can use disk, memory and other storage |
| Entries may expire or be evicted | Data persists according to database policies |

Example:

Notes application:

Database:

text

```text
All user notes
All user profiles
All shared notes
```

Cache:

text

```text
Recently accessed notes
Popular notes
Frequently accessed profiles
```

Important:

A cache should generally not be treated as the only source of truth for important application data.

---

# 18\. Android Developer Perspective

Caching is especially important in Android applications.

Let's understand how we can apply it.

## 18.1 In-Memory Cache

Suppose the user opens a profile screen.

When the profile is fetched, keep it in memory.

If the user navigates away and returns, the application may reuse the cached object.

Example:

kotlin

```kotlin
object UserCache {
    private val cache = mutableMapOf<Int, User>()

    fun save(user: User) {
        cache[user.id] = user
    }

    fun get(userId: Int): User? {
        return cache[userId]
    }
}
```

This is a simple illustration.

In a real application, consider:

- Memory limits.
- Thread safety.
- Lifecycle.
- Cache expiration.
- Data invalidation.

## 18.2 Room Database as Local Cache

Suppose the user opens the Notes application without an internet connection.

Can they still see previously loaded notes?

Yes, if notes have been stored locally.

Room can be used to maintain local structured data.

text

```text
           ANDROID APP
                |
          +-----+-----+
          |           |
          v           v
       ROOM DB      REMOTE API
          |           |
          v           v
      LOCAL DATA   BACKEND DATA
```

A common approach is:

1. Display locally stored data.
2. Fetch updated data from the backend.
3. Update the Room database.
4. Observe database changes and refresh the UI.

This is useful for offline-first applications.

## 18.3 DataStore

DataStore is useful for storing small amounts of application data such as:

- User preferences.
- Theme settings.
- Selected language.
- Simple configuration values.

It is not a replacement for a relational database or a general-purpose large cache.

## 18.4 Image Caching

When loading images using libraries such as Coil or Glide, caching can avoid repeatedly downloading the same images.

There can be:

- Memory cache.
- Disk cache.
- Network/CDN cache.

## 18.5 Android Caching vs Backend Caching

| Android Cache | Backend Cache |
| --- | --- |
| Stored on the user's device | Stored in backend infrastructure |
| Reduces network requests | Reduces database/backend work |
| Useful for offline access | Useful for serving many users |
| Example: Room, Coil | Example: Redis, Caffeine |

Important:

Android caching cannot replace backend caching.

If 100,000 users request the same popular data, server-side caching can help protect the backend.

---

# 19\. Complete Caching Architecture – Notes Application

Let's combine everything.

Requirement:

A user opens the Notes application and requests their notes.

Architecture:

text

```text
              ANDROID APP
                   |
                   v
             CHECK ROOM DB
                   |
          +--------+--------+
          |                 |
       DATA EXISTS       NO DATA
          |                 |
          v                 v
      DISPLAY DATA       SHOW LOADING
          |                 |
          +--------+--------+
                   |
                   v
                BACKEND
                   |
                   v
                REDIS
                   |
             +-----+-----+
             |           |
            HIT         MISS
             |           |
             v           v
          RETURN      DATABASE
                        |
                        v
                     REDIS
                        |
                        v
                     RETURN
                        |
                        v
                  UPDATE ROOM DB
                        |
                        v
                    UPDATE UI
```

### Step-by-Step Flow

1. User opens the Android application.
2. Android reads previously stored notes from Room.
3. Android displays local data if available.
4. Android sends a request to the backend.
5. Backend checks Redis.
6. If Redis contains the data, it returns the cached result.
7. If Redis does not contain the data, backend queries the database.
8. Backend may store the result in Redis.
9. Backend returns the response.
10. Android updates Room.
11. The UI displays the latest available data.

This is a simplified example. Real systems also need synchronization, cache invalidation, authentication and error handling.

---

# 20\. Common Caching Mistakes

1. Assuming cache and database are the same.
2. Treating cached data as the permanent source of truth.
3. Forgetting cache expiration.
4. Not invalidating stale data.
5. Storing everything in the cache.
6. Ignoring memory limitations.
7. Ignoring cache stampede.
8. Assuming a high cache hit ratio always means good performance.
9. Using an unsuitable eviction policy.
10. Relying only on Android caching to improve backend scalability.
11. Ignoring cache failure scenarios.
12. Not monitoring cache latency and memory usage.

---

# 21\. Important Interview Questions and Answers

### Q1. What is caching?

Caching is a technique of temporarily storing frequently accessed data in faster storage to improve response time and reduce repeated work.

### Q2. Why do we need caching?

To reduce latency, database load, repeated computation and backend resource consumption.

### Q3. What is a Cache Hit?

When requested data is found in the cache.

### Q4. What is a Cache Miss?

When requested data is not found in the cache and must be fetched from another source.

### Q5. What is TTL?

Time To Live defines how long a cached entry remains valid.

### Q6. What is Cache Eviction?

Removing cached entries to free memory or enforce a cache policy.

### Q7. Explain LRU.

Least Recently Used removes the item that has not been accessed for the longest time.

### Q8. Explain LFU.

Least Frequently Used removes the item with the lowest access frequency.

### Q9. What is Cache Invalidation?

Removing or updating cached data when the original data changes.

### Q10. What is Cache Hit Ratio?

The percentage of cache requests successfully served from the cache.

### Q11. What is Redis?

Redis is an in-memory data store commonly used for caching and other data-storage use cases.

### Q12. What is In-Memory Caching?

Storing frequently accessed data in RAM to provide fast access.

### Q13. What is Cache Stampede?

When many requests simultaneously try to retrieve the same data after a cache entry expires.

### Q14. What is the difference between cache and database?

A database is designed for durable data storage, while a cache is primarily used to provide faster access to frequently requested data.

### Q15. Can cache replace a database?

Usually no. Important application data should have an appropriate durable source of truth.

### Q16. Where can caching happen?

Browser, client application, backend server, database infrastructure and CDN.

### Q17. What is CDN caching?

A CDN stores copies of content at geographically distributed edge locations to serve users faster.

### Q18. What happens when a cache entry expires?

The entry is no longer considered valid. The next request may need to fetch fresh data from the original source.

### Q19. What happens if Redis becomes unavailable?

The application may fall back to the database if designed to do so. However, the database may experience increased traffic, so fallback protection is important.

### Q20. How does caching improve scalability?

It reduces repeated database queries and backend computation, allowing the system to handle more requests with the available resources.

### Q21. What is the difference between client-side and server-side caching?

Client-side caching stores data on the user's device. Server-side caching stores data within backend infrastructure.

### Q22. How would you implement caching in an Android application?

Use an in-memory cache for short-lived reusable objects, Room for structured local data, DataStore for small preferences and image libraries for image caching.

### Q23. What is cache consistency?

It refers to how the system ensures cached data remains sufficiently aligned with the original data source.

### Q24. What is a cache eviction policy?

A rule that determines which cached entry should be removed when space is required.

### Q25. What is the difference between TTL and LRU?

TTL controls how long an entry remains valid. LRU determines which entry to remove based on recent access when eviction is needed.

---

# 22\. Practical Interview Exercise

Imagine you are designing a Notes application.

Requirements:

- 1 million users.
- Users frequently open their profiles.
- Users can access notes from multiple devices.
- Some notes are accessed thousands of times.
- Database read traffic is increasing.
- Users sometimes see outdated information.

Answer these questions:

1. Why would you introduce caching?
2. Where would you implement caching?
3. What is the difference between Cache Hit and Cache Miss?
4. Why would you use Redis?
5. What is TTL?
6. How would you handle outdated cached data?
7. What is the difference between LRU and LFU?
8. What happens when Redis becomes unavailable?
9. What is Cache Stampede?
10. How would you implement offline caching in Android?

Try answering these without referring to the notes.

---

# 23\. Day 15 – Quick Revision

| Concept | Simple Meaning |
| --- | --- |
| Caching | Temporarily storing frequently used data |
| Cache Hit | Requested data found in cache |
| Cache Miss | Requested data not found in cache |
| Cache Latency | Time taken to access cached data |
| TTL | How long a cache entry remains valid |
| Cache Eviction | Removing cached entries |
| LRU | Removes least recently used item |
| LFU | Removes least frequently used item |
| FIFO | Removes oldest inserted item |
| Cache Invalidation | Removing or updating stale data |
| Cache Hit Ratio | Percentage of cache hits |
| Cache Stampede | Many requests fetch the same expired data |
| Redis | In-memory data store |
| CDN | Distributed content delivery network |
| In-Memory Cache | Cache stored in RAM |

---

# 24\. Final Learning Checklist

- [ ]Understand what caching is.
- [ ]Understand why caching is needed.
- [ ]Understand cache hit.
- [ ]Understand cache miss.
- [ ]Understand cache latency.
- [ ]Understand different cache locations.
- [ ]Understand in-memory caching.
- [ ]Understand Redis basics.
- [ ]Understand TTL.
- [ ]Understand cache eviction.
- [ ]Understand LRU.
- [ ]Understand LFU.
- [ ]Understand FIFO.
- [ ]Understand cache invalidation.
- [ ]Understand cache hit ratio.
- [ ]Understand cache stampede.
- [ ]Understand cache vs database.
- [ ]Understand Android local caching.
- [ ]Explain caching in a Notes application.

---

# 25\. What Is Next?

## Week 3 – Day 16: Cache Strategies and Cache Invalidation

Tomorrow we will learn how applications actually read and write cached data.

Topics:

1. Cache-aside pattern.
2. Read-through caching.
3. Write-through caching.
4. Write-back caching.
5. Write-around caching.
6. Cache invalidation strategies.
7. Cache consistency problems.
8. Cache failure scenarios.
9. Real-world Notes application caching architecture.

**Day 15 Complete – Caching Fundamentals.**

## Today's most important concepts

Before moving to Day 16, make sure you can explain these five concepts without looking at your notes:

1. What is caching and why do we need it?
2. What is the difference between a cache hit and a cache miss?
3. What is TTL?
4. What is the difference between LRU and LFU?
5. How would you implement caching in an Android application?

Next: Day 16 – Cache Strategies and Cache Invalidation.
