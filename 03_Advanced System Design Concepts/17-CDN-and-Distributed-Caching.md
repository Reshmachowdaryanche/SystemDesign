# Week 3 – Day 17: CDN and Distributed Caching

Week 3Day 17Beginner level

File name: `Week-3-Day-17-CDN-and-Distributed-Caching.md`

Today's goal: Understand Redis in detail, why distributed caching is needed, how multiple backend servers share cached data, and how CDNs deliver images and videos faster.

Writing

# Week 3 – Day 17: CDN and Distributed Caching

**Topic:** CDN and Distributed Caching\
**Level:** Beginner\
**Target:** Android Developer – 7 Years Experience

---

# 1\. Introduction

In Day 15, we learned caching fundamentals.

In Day 16, we learned different cache strategies.

Today, we will learn how caching works in real-world large-scale applications.

Imagine you are developing an application like Instagram.

Your application has:

- 10 million users.
- Thousands of backend requests per second.
- Millions of profile pictures.
- Millions of posts.
- Users accessing the application from different countries.

Some questions arise:

1. Can one backend server handle all requests?
2. Can one cache server handle all cached data?
3. How can multiple backend servers share cached data?
4. How can users in different countries load images quickly?
5. What happens if a cache server goes down?

We will answer all these questions.

Today's topics:

1. What is Redis?
2. Redis architecture.
3. Local cache vs distributed cache.
4. What is distributed caching?
5. Redis replication.
6. Redis clustering.
7. Redis Sentinel.
8. What is CDN?
9. CDN edge servers.
10. CDN vs Redis.
11. Cache Stampede prevention.
12. Real-world Android application architecture.

---

# 2\. What Is Redis?

Redis is an in-memory data store commonly used for caching.

It stores data in memory (RAM), allowing applications to access it quickly.

Redis supports different data structures.

- Strings
- Hashes
- Lists
- Sets
- Sorted Sets
- Streams

For system design interviews, you should initially understand Redis as a fast, shared data store that backend applications can use to avoid repeated database queries.

## 2.1 Why Do We Need Redis?

Imagine our Notes application.

Every time a user opens a note, the backend queries the database.

text

```text
ANDROID APP
     |
     v
BACKEND
     |
     v
DATABASE
     |
     v
RETURN NOTE
```

If 100,000 users repeatedly request the same popular note, the database receives many repeated queries.

We can introduce Redis.

text

```text
ANDROID APP
     |
     v
BACKEND
     |
     v
REDIS
     |
   +---+
   |   |
  HIT MISS
   |   |
   v   v
RETURN DATABASE
         |
         v
        REDIS
```

Now frequently accessed data can be served from Redis.

## 2.2 Redis vs Traditional Database

| Redis | Traditional Database |
| --- | --- |
| Primarily optimized for fast in-memory access | Designed for durable data storage |
| Commonly used for caching | Commonly used as the primary data store |
| Supports TTL expiration | Persistence depends on database configuration |
| Supports multiple data structures | Supports database-specific data models |
| Often stores reusable data | Stores the application's source records |

Important:

Redis can also be configured for persistence. It is not exclusively a temporary cache.

However, in our system design examples, we will primarily use Redis as a cache.

## 2.3 Simple Redis Commands

Redis supports commands to store and retrieve data.

**SET – Store a value**

redis

```redis
SET user:101 "John"
```

**GET – Retrieve a value**

redis

```redis
GET user:101
```

Output:

text

```text
"John"
```

**DEL – Delete a key**

redis

```redis
DEL user:101
```

**EXPIRE – Set expiration time**

redis

```redis
EXPIRE user:101 300
```

The key expires after 300 seconds.

**SET with expiration**

redis

```redis
SET user:101 "John" EX 300
```

This stores the value with a five-minute expiration.

---

# 3\. What Is Local Caching?

Local caching means storing cached data inside the memory of an individual application instance.

Imagine we have one backend server.

text

```text
       BACKEND SERVER
             |
             v
        LOCAL CACHE
             |
             v
          DATABASE
```

The application may use:

- Java HashMap.
- Caffeine Cache.
- Guava Cache.

Example:

java

```java
Map<String, User> cache = new HashMap<>();

cache.put("user:101", user);
```

When another request needs the same user, the application checks the local cache.

## 3.1 Problem With Local Cache

Now imagine our application becomes popular.

Instead of one backend server, we run three backend servers.

text

```text
           ANDROID APP
                |
          LOAD BALANCER
                |
        +-------+-------+
        |       |       |
        v       v       v
      SERVER  SERVER  SERVER
        A       B       C
        |       |       |
        v       v       v
      CACHE   CACHE   CACHE
        A       B       C
```

Each server has its own local cache.

Suppose a user updates their profile through Server A.

Server A updates its local cache.

But what happens to Server B and Server C?

They may still contain the old profile.

This creates a cache consistency problem.

### Example

Initially:

text

```text
SERVER A CACHE: John
SERVER B CACHE: John
SERVER C CACHE: John
```

User changes the name to David.

After updating Server A:

text

```text
SERVER A CACHE: David
SERVER B CACHE: John
SERVER C CACHE: John
```

Now different backend servers return different information.

This is one major limitation of local caching.

---

# 4\. What Is Distributed Caching?

Distributed caching means storing cached data in a shared caching infrastructure that multiple application instances can access.

Instead of every backend server maintaining a separate copy of cached data, multiple servers use a shared cache service.

Redis is commonly used for this purpose.

## 4.1 Architecture

text

```text
             ANDROID APP
                  |
                  v
             LOAD BALANCER
                  |
          +-------+-------+
          |       |       |
          v       v       v
        SERVER  SERVER  SERVER
          A       B       C
          |       |       |
          +-------+-------+
                  |
                  v
             REDIS CACHE
                  |
                  v
               DATABASE
```

Now all backend servers communicate with the same Redis infrastructure.

## 4.2 How Does It Work?

Imagine a user requests their profile.

1. Request reaches Server A.
2. Server A checks Redis.
3. Redis contains the profile.
4. Redis returns the profile.
5. Server A responds to Android.

Now another request reaches Server B.

1. Server B checks Redis.
2. It finds the same cached profile.
3. Redis returns the profile.
4. Server B responds to Android.

Both servers use the shared cache.

## 4.3 Advantages

- Shared cache across backend instances.
- Less duplication of cached data.
- Easier cache invalidation compared with independent local caches.
- Helps reduce database load.
- Supports horizontally scaled applications.

## 4.4 Disadvantages

- Adds network communication between backend and cache.
- Redis infrastructure needs monitoring.
- Cache failure can affect multiple backend servers.
- Requires high availability and capacity planning.
- Network latency still exists.

---

# 5\. Local Cache vs Distributed Cache

This is an important interview comparison.

| Local Cache | Distributed Cache |
| --- | --- |
| Stored inside application process | Stored in separate shared infrastructure |
| Each server has its own cache | Multiple servers share cache |
| Very fast local memory access | Requires network communication |
| Difficult to synchronize across servers | Easier to share common cached data |
| Limited to individual instance resources | Can scale independently |
| Example: Caffeine | Example: Redis |

## 5.1 Can We Use Both?

Yes.

Many real-world applications use both local and distributed caching.

Architecture:

text

```text
            ANDROID APP
                 |
                 v
             BACKEND A
                 |
                 v
             LOCAL CACHE
                 |
             CACHE MISS
                 |
                 v
             REDIS CACHE
                 |
             CACHE MISS
                 |
                 v
              DATABASE
```

This is called a multi-level caching architecture.

Example:

- L1 cache: Local application memory.
- L2 cache: Shared Redis cache.
- Database: Persistent source of truth.

### Request Flow

1. Check local cache.
2. If found, return data.
3. If missing, check Redis.
4. If found in Redis, return and optionally populate local cache.
5. If missing in Redis, query the database.
6. Populate Redis and local cache.
7. Return the response.

Important:

Multi-level caching introduces additional invalidation complexity. When data changes, all relevant cache levels must be updated or invalidated.

---

# 6\. What Is Redis Replication?

Imagine your application depends on one Redis server.

text

```text
BACKEND
   |
   v
REDIS SERVER
```

What happens if that Redis server crashes?

The application may lose access to cached data.

Database fallback may help, but it can suddenly increase database load.

We need a way to improve Redis availability.

This is where replication comes in.

## 6.1 Definition

Redis replication means maintaining copies of data from one Redis server on one or more other Redis servers.

Redis commonly uses:

- Primary (Master) node.
- Replica (Replica/Slave) nodes.

The primary handles writes, and replicas receive copies of its data.

## 6.2 Architecture

text

```text
             BACKEND
                |
                v
          PRIMARY REDIS
                |
          +-----+-----+
          |           |
          v           v
       REPLICA 1   REPLICA 2
```

## 6.3 How Does It Work?

Suppose the primary Redis server contains:

text

```text
user:101 = John
user:102 = David
user:103 = Alex
```

Redis replicas receive copies of the primary's data.

Now:

text

```text
PRIMARY REDIS:
user:101 = John

REPLICA 1:
user:101 = John

REPLICA 2:
user:101 = John
```

If the primary receives a write, changes are propagated to the replicas.

## 6.4 Why Do We Need Replication?

1. Improves availability.
2. Provides additional copies of cached data.
3. Can support read scaling.
4. Helps recover from primary node failures.

## 6.5 Important Concept: Replication Lag

Replication may not be instantaneous.

Imagine:

Primary:

text

```text
user:101 = David
```

Replica:

text

```text
user:101 = John
```

For a short time, the replica may contain older data.

This is called replication lag.

If the application reads from the replica during this period, it may receive stale data.

## 6.6 Synchronous vs Asynchronous Replication

**Synchronous replication:**

The system waits for replica acknowledgement according to the configured replication protocol before considering the operation complete.

This can improve durability guarantees but may increase latency.

**Asynchronous replication:**

The primary does not wait for every replica to confirm each write.

It generally provides lower write latency, but replicas can temporarily lag behind.

Redis replication is generally asynchronous.

Important interview point:

Redis replication does not automatically guarantee that every write exists on every replica immediately.

---

# 7\. What Is Redis Sentinel?

Redis Sentinel helps monitor Redis instances and manage automatic failover.

First, understand the problem.

Suppose we have:

text

```text
PRIMARY REDIS
      |
      v
REPLICA REDIS
```

The primary server crashes.

Who decides that the replica should become the new primary?

Redis Sentinel helps manage this process.

## 7.1 Definition

Redis Sentinel is a monitoring and failover system for Redis deployments that use primary-replica replication.

Its responsibilities include:

1. Monitoring Redis instances.
2. Detecting failures.
3. Coordinating failover.
4. Promoting a replica to primary when appropriate.
5. Providing information about the current primary.

## 7.2 Architecture

text

```text
          SENTINEL NODES
          /      |      \
         /       |       \
        v        v        v

      PRIMARY REDIS
            |
       +----+----+
       |         |
       v         v
    REPLICA 1  REPLICA 2
```

## 7.3 What Happens When Primary Fails?

1. Sentinels monitor the primary.
2. They detect that the primary is unavailable.
3. Sentinels coordinate a failover decision.
4. An eligible replica is promoted to primary.
5. The application discovers the new primary.

This helps reduce manual recovery work.

## 7.4 Important Interview Point

Replication and Sentinel solve different problems.

- Replication maintains copies of data.
- Sentinel monitors Redis and coordinates failover.

Sentinel does not itself store a separate copy of the cached data.

---

# 8\. What Is Redis Cluster?

Now imagine your application stores a huge amount of cached data.

One Redis server does not have enough memory or processing capacity.

Can we add more Redis servers and distribute data across them?

Yes.

This is where Redis Cluster comes in.

## 8.1 Definition

Redis Cluster is a distributed Redis deployment that partitions data across multiple Redis nodes.

This allows the Redis dataset to be distributed across multiple nodes.

## 8.2 Architecture

text

```text
            BACKEND
               |
               v
          REDIS CLUSTER
               |
       +-------+-------+
       |       |       |
       v       v       v
     NODE A  NODE B  NODE C
       |       |       |
       v       v       v
     DATA A  DATA B  DATA C
```

Instead of storing the entire dataset on one node, different parts of the dataset are stored on different nodes.

## 8.3 What Is Sharding?

Sharding means splitting data into smaller portions and distributing those portions across multiple servers.

Example:

Suppose we have one million cache keys.

Instead of storing everything on one Redis node:

text

```text
NODE A:
Keys 1 - 333,333

NODE B:
Keys 333,334 - 666,666

NODE C:
Keys 666,667 - 1,000,000
```

This is a simplified illustration.

Redis Cluster uses hash slots to distribute keys rather than fixed numerical ranges.

## 8.4 What Are Hash Slots?

Redis Cluster divides its key space into 16,384 hash slots.

Each key is assigned to a slot using a hashing function.

The slots are distributed across cluster nodes.

text

```text
          REDIS CLUSTER

       16,384 HASH SLOTS
                |
       +--------+--------+
       |        |        |
       v        v        v
     NODE A   NODE B   NODE C
     SLOTS    SLOTS    SLOTS
```

When a request arrives:

1. Redis determines the hash slot for the key.
2. It identifies which node owns that slot.
3. The request is routed to the appropriate node.

## 8.5 Redis Cluster Replication

Redis Cluster can also use replicas.

text

```text
            REDIS CLUSTER

        +---------+---------+
        |         |         |
        v         v         v
      PRIMARY   PRIMARY   PRIMARY
        |         |         |
        v         v         v
      REPLICA   REPLICA   REPLICA
```

This helps combine:

- Data distribution.
- Additional capacity.
- Replication.
- Failover support.

## 8.6 Redis Cluster vs Redis Sentinel

| Redis Sentinel | Redis Cluster |
| --- | --- |
| Primarily provides monitoring and failover | Provides sharding and failover |
| Does not shard data | Distributes data across nodes |
| Suitable when dataset fits on one primary | Useful when dataset needs distribution |
| Uses primary-replica architecture | Uses multiple primary nodes and replicas |

Important:

Redis Cluster and Sentinel are different deployment approaches. Redis Cluster has its own cluster management and failover mechanisms.

---

# 9\. Redis Replication vs Sharding

These concepts are frequently confused.

## Replication

Replication means copying the same data to another server.

text

```text
PRIMARY:
A, B, C, D

REPLICA:
A, B, C, D
```

Purpose:

- Availability.
- Read scaling in suitable architectures.
- Redundancy.

## Sharding

Sharding means dividing the data across multiple servers.

text

```text
NODE A:
A, B

NODE B:
C, D
```

Purpose:

- Distribute storage.
- Distribute workload.
- Scale beyond one node's capacity.

## Comparison

| Replication | Sharding |
| --- | --- |
| Copies data | Splits data |
| Same dataset exists on multiple nodes | Different data portions exist on different nodes |
| Improves availability | Improves capacity and distribution |
| Can support read scaling | Distributes requests by data ownership |

Memory trick:

**Replication = Copy**

**Sharding = Split**

---

# 10\. What Is a CDN?

So far, we have discussed caching backend data.

Now imagine your Notes application supports profile pictures and attachments.

A user uploads a profile picture.

The image is stored on a central storage service.

Users from different countries need to download that image.

Imagine:

- User A is in India.
- User B is in the United States.
- User C is in Germany.

If every user downloads the image from one server located in India, users far away may experience higher network latency.

We need a better approach.

This is where CDN comes in.

## 10.1 Definition

CDN stands for Content Delivery Network.

A CDN is a geographically distributed network of servers that stores and delivers copies of content closer to users.

Common CDN providers include:

- Cloudflare.
- Amazon CloudFront.
- Akamai.
- Google Cloud CDN.

## 10.2 Why Do We Need CDN?

Without CDN:

text

```text
USER INDIA --------+
                   |
USER USA ----------+----> CENTRAL SERVER
                   |
USER GERMANY ------+
```

All users download content from one central location.

With CDN:

text

```text
                  ORIGIN SERVER
                       |
                +------+------+
                |             |
                v             v
             CDN INDIA     CDN USA
                |             |
                v             v
             USERS         USERS

                   CDN EUROPE
                       |
                       v
                     USERS
```

Users can retrieve cached content from a nearby CDN edge server.

## 10.3 What Is an Edge Server?

An edge server is a server located closer to end users, usually within or near a particular geographic region.

It can store cached copies of content.

Example:

A user in Hyderabad requests an image.

The request may be served from a nearby CDN edge location if the image is already cached there.

This avoids repeatedly fetching the image from the original storage server.

## 10.4 How Does CDN Work?

Imagine a user opens a note containing an image.

The image URL is:

text

```text
https://cdn.example.com/images/profile101.jpg
```

Step 1: Android requests the image.

Step 2: The request is routed to a suitable CDN edge location.

Step 3: CDN checks whether the image is available in its cache.

Step 4: If cached, the CDN returns the image.

This is a CDN Cache Hit.

Step 5: If not cached, the CDN fetches the image from the origin server.

Step 6: CDN may store a copy according to its caching rules.

Step 7: CDN returns the image to Android.

This is a CDN Cache Miss.

## 10.5 CDN Architecture

text

```text
             ANDROID APP
                  |
                  v
               CDN URL
                  |
                  v
             EDGE SERVER
                  |
              +---+---+
              |       |
             HIT     MISS
              |       |
              v       v
           RETURN   ORIGIN
                    SERVER
                      |
                      v
                   STORAGE
                      |
                      v
                 CACHE COPY
                      |
                      v
                   RETURN
```

---

# 11\. What Is an Origin Server?

The origin server is the original source from which the CDN retrieves content when it does not have a valid cached copy.

Example:

Your application stores profile images in object storage such as Amazon S3.

text

```text
ANDROID APP
     |
     v
CDN EDGE SERVER
     |
  CACHE MISS
     |
     v
AMAZON S3
```

In this architecture:

- Amazon S3 is the origin storage.
- CDN edge servers cache copies of the images.
- Android downloads images through the CDN URL.

The origin does not have to be a traditional application server. It can be an object storage service or another configured origin.

---

# 12\. What Is a CDN Cache Hit and Miss?

The concepts are similar to Redis.

**CDN Cache Hit**

The requested content already exists in the CDN cache.

The CDN returns it without fetching it again from the origin.

**CDN Cache Miss**

The requested content is not available in the CDN cache.

The CDN fetches it from the origin server.

| CDN Cache Hit | CDN Cache Miss |
| --- | --- |
| Content found at edge | Content not found at edge |
| Served from CDN cache | Fetched from origin |
| Usually avoids origin request | Requires origin request |

---

# 13\. What Is CDN Cache Invalidation?

Imagine a user updates their profile picture.

Old image:

text

```text
profile101.jpg
```

New image:

text

```text
profile101.jpg
```

The CDN may still have the old image cached.

How do we make users see the new image?

We need CDN cache invalidation or a cache-busting strategy.

## 13.1 CDN Invalidation

The application or administrator requests removal of the old cached content from CDN locations.

The next request may fetch the latest image from the origin.

## 13.2 Versioned URLs

A commonly used alternative is to change the image URL whenever its content changes.

Old:

text

```text
profile101_v1.jpg
```

New:

text

```text
profile101_v2.jpg
```

Because the URL is different, the CDN treats it as a new resource.

This is often easier to manage than invalidating a frequently distributed resource.

## 13.3 TTL-Based CDN Caching

CDN caching can also use expiration rules.

Example:

text

```text
Cache-Control: public, max-age=3600
```

This tells compliant caches that the content may be considered fresh for one hour.

Actual CDN behavior also depends on its cache policies and configuration.

---

# 14\. CDN vs Redis

This is one of the most important interview questions from today's topic.

Both Redis and CDN use caching, but they solve different problems.

| Redis | CDN |
| --- | --- |
| Primarily used for backend data caching | Used for content delivery |
| Usually accessed by backend services | Often accessed directly by clients |
| Stores structured or application data | Commonly caches images, videos, CSS, JS and other content |
| Usually uses memory-based data storage | Uses geographically distributed edge infrastructure |
| Reduces database load | Reduces origin load and delivery latency |
| Example: User profile data | Example: Profile picture delivery |

## Example

Notes application:

Redis:

text

```text
user:101 -> {
    name: "John",
    email: "john@example.com"
}
```

CDN:

text

```text
https://cdn.example.com/profile101.jpg
```

Redis stores reusable application data.

CDN delivers content closer to users.

Important:

Redis and CDN are not alternatives. A system can use both.

---

# 15\. What Is Cache Stampede Prevention?

In Day 15 and Day 16, we learned about Cache Stampede.

Let's understand how distributed caching systems handle it.

Imagine a popular note is requested by 10,000 users.

Its Redis entry expires.

All requests get a cache miss.

All requests query the database.

The database becomes overloaded.

There are several ways to reduce this problem.

## 15.1 Request Coalescing

Request coalescing means combining multiple requests for the same missing data into one fetch operation.

Example:

text

```text
USER 1 ----+
USER 2 ----+
USER 3 ----+----> CACHE MISS
USER 4 ----+          |
USER 5 ----+          v
                    DATABASE
                        |
                        v
                   ONE FETCH
                        |
                        v
                   SHARE RESULT
```

Instead of 10,000 database queries, one request fetches the data while other requests wait for the same result.

## 15.2 TTL Jitter

Instead of assigning exactly the same TTL to many entries, add a small random variation.

Example:

text

```text
NOTE A: 300 seconds
NOTE B: 320 seconds
NOTE C: 285 seconds
NOTE D: 310 seconds
```

This reduces the chance that many cache entries expire simultaneously.

## 15.3 Background Refresh

Frequently accessed cache entries can be refreshed before they expire.

Example:

text

```text
CACHE ENTRY
     |
     v
NEAR EXPIRATION
     |
     v
BACKGROUND REFRESH
     |
     v
UPDATE CACHE
```

Users may continue receiving a valid cached result while the refresh happens.

## 15.4 Distributed Locking

A distributed lock can help ensure that only one backend instance performs a particular expensive cache refresh at a time.

Other instances wait, retry or use stale data when safe.

However, distributed locking needs careful handling of expiration, ownership and failure scenarios.

---

# 16\. Multi-Level Caching Architecture

Now let's combine everything we have learned.

A real-world application may use multiple caching layers.

Example:

text

```text
               ANDROID APP
                    |
                    v
               LOCAL CACHE
                    |
                    v
                BACKEND API
                    |
                    v
               LOCAL CACHE
                    |
                    v
               REDIS CACHE
                    |
               CACHE MISS
                    |
                    v
                 DATABASE


        IMAGE REQUEST FLOW

               ANDROID APP
                    |
                    v
                 CDN EDGE
                    |
               CACHE MISS
                    |
                    v
              OBJECT STORAGE
```

Explanation:

1. Android can maintain local cached data.
2. Backend servers can maintain local in-memory caches.
3. Redis can provide shared cached application data.
4. Database stores durable application records.
5. CDN caches and delivers static content.
6. Object storage holds original images and attachments.

This is a common architectural pattern in scalable applications.

---

# 17\. Complete Real-World Notes Application Design

Let's design caching for our Notes application.

## Requirements

1. Users can create notes.
2. Users can update notes.
3. Users can view notes.
4. Users can upload images.
5. Users can access notes from multiple devices.
6. Application should load frequently accessed information quickly.
7. Images should load quickly for users in different locations.

## Architecture

text

```text
                   ANDROID APP
                        |
              +---------+---------+
              |                   |
              v                   v
           NOTES API           IMAGE API
              |                   |
              v                   v
         LOAD BALANCER           CDN
              |                   |
         +----+----+              |
         |         |              |
         v         v              v
       SERVER A SERVER B      EDGE SERVERS
         |         |              |
         +----+----+              |
              |                   |
              v                   v
          REDIS CACHE        OBJECT STORAGE
              |
              v
           DATABASE
```

## Scenario 1: User Opens a Note

1. Android checks local Room database.
2. Android requests the note from backend.
3. Backend checks Redis.
4. If found, return the cached note.
5. Otherwise, fetch from database.
6. Store the result in Redis.
7. Update Android's Room database.

## Scenario 2: User Updates a Note

1. Android sends an update request.
2. Backend updates the database.
3. Backend invalidates the corresponding Redis entry.
4. Backend returns success.
5. The next read fetches fresh data if the cache is empty.
6. Android updates its local database.

## Scenario 3: User Uploads an Image

1. Android uploads an image using the configured upload flow.
2. Backend or upload service stores the image in object storage.
3. The application saves image metadata or its storage reference.
4. A CDN URL is used for image delivery.
5. Other users retrieve the image through the CDN.

## Scenario 4: Users From Different Countries Access the Same Image

1. User in India requests the image.
2. CDN serves it from an appropriate edge location if cached.
3. User in the United States requests the same image.
4. CDN may serve it from a different edge location.
5. Both users can receive the content without repeatedly accessing the origin.

---

# 18\. Important Interview Questions and Answers

### Q1. What is Redis?

Redis is an in-memory data store commonly used for caching and other data-storage use cases.

### Q2. Why do we need distributed caching?

To allow multiple backend instances to access shared cached data and reduce repeated database queries.

### Q3. What is local caching?

Storing cached data inside an individual application instance.

### Q4. What is distributed caching?

Storing cached data in shared infrastructure that multiple application instances can access.

### Q5. What is the difference between local and distributed caching?

Local caching belongs to one application instance, whereas distributed caching is shared across multiple instances.

### Q6. What is Redis replication?

Maintaining copies of data from a primary Redis server on replica servers.

### Q7. What is replication lag?

The delay between a change on the primary and that change becoming available on a replica.

### Q8. What is Redis Sentinel?

A Redis monitoring and failover system that can coordinate promotion of a replica when the primary fails.

### Q9. What is Redis Cluster?

A Redis deployment that distributes data across multiple nodes using hash slots and supports replication and failover.

### Q10. What is sharding?

Splitting data into multiple portions and distributing them across multiple servers.

### Q11. What is the difference between replication and sharding?

Replication copies the same data to multiple nodes. Sharding distributes different portions of data across nodes.

### Q12. What is CDN?

A Content Delivery Network is a geographically distributed network that caches and delivers content closer to users.

### Q13. What is an edge server?

A server located closer to users that can deliver cached content.

### Q14. What is an origin server?

The original source from which a CDN retrieves content when it is not available in its cache.

### Q15. What is CDN cache invalidation?

Removing or refreshing cached content when the original content changes.

### Q16. What is the difference between Redis and CDN?

Redis primarily caches backend application data. CDN primarily caches and delivers content such as images and videos closer to users.

### Q17. Can we use Redis and CDN together?

Yes. Redis can cache frequently accessed backend data while CDN delivers static content.

### Q18. What is multi-level caching?

Using multiple caching layers, such as local memory, Redis and client-side caching.

### Q19. How does CDN improve application performance?

It serves content from geographically closer servers, reducing the distance between the content source and the user.

### Q20. How can we prevent cache stampede?

Use request coalescing, TTL jitter, background refresh and distributed locking where appropriate.

### Q21. What happens if Redis primary goes down?

If the deployment has a configured failover mechanism, a replica may be promoted to primary. Otherwise, manual recovery may be needed.

### Q22. Does Redis replication guarantee zero data loss?

No. Redis replication is generally asynchronous, so recent writes may not have reached a replica before failure.

### Q23. What happens if the CDN does not have an image?

The CDN fetches the image from its configured origin, subject to its cache and origin configuration.

### Q24. Can Android use CDN directly?

Yes. Android can request public or appropriately authorized content using CDN URLs.

### Q25. How would you design image delivery for Instagram?

Store original images in object storage, deliver them through a CDN, and use appropriate caching and image transformation policies.

---

# 19\. Scenario-Based Interview Questions

## Scenario 1: Three Backend Servers Have Different User Data

Your application has three backend servers.

Each server uses a local cache.

A user updates their profile through Server A, but Server B returns the old profile.

What is the problem?

**Expected answer:**

Each server maintains an independent local cache. One possible solution is shared distributed caching using Redis, combined with an appropriate invalidation strategy.

## Scenario 2: Redis Primary Crashes

Your Redis primary server becomes unavailable.

What should happen?

**Expected answer:**

If Redis Sentinel or Redis Cluster is configured appropriately, a replica can be promoted. Backend applications must discover the new primary and handle temporary failures.

## Scenario 3: Your Redis Dataset Becomes Too Large

One Redis server does not have enough capacity.

What can you do?

**Expected answer:**

Use Redis Cluster to distribute data across multiple nodes through sharding.

## Scenario 4: Users in the USA Experience Slow Image Loading

Your origin image server is located in India.

How can you improve image loading?

**Expected answer:**

Use a CDN with geographically distributed edge servers so users can retrieve cached images from locations closer to them.

## Scenario 5: Users See an Old Profile Picture

A user changes their profile picture, but the old picture continues appearing.

What could be wrong?

**Expected answer:**

The CDN may still have the old image cached. Use CDN invalidation or versioned image URLs.

## Scenario 6: One Popular Note Causes Database Overload

Thousands of requests arrive immediately after the note's cache entry expires.

What should you do?

**Expected answer:**

Use request coalescing, TTL jitter or background refresh to prevent multiple simultaneous database fetches.

## Scenario 7: Should Every Backend Server Have Its Own Cache?

**Expected answer:**

It depends on the use case. Local caching is fast and useful for instance-specific or relatively static data. Shared Redis caching is useful when multiple backend servers need common data. Both can be combined.

---

# 20\. Common Interview Mistakes

1. Assuming Redis and CDN solve the same problem.
2. Confusing replication with sharding.
3. Assuming local cache is automatically shared between backend servers.
4. Assuming Redis replicas are always immediately consistent.
5. Assuming Redis Sentinel and Redis Cluster are the same.
6. Assuming CDN stores the original content.
7. Forgetting CDN invalidation.
8. Ignoring Redis failure scenarios.
9. Assuming replication guarantees zero data loss.
10. Ignoring cache consistency in multi-level caching.
11. Assuming every request must go through Redis.
12. Ignoring cache network latency.

---

# 21\. Day 17 – Quick Revision Cheat Sheet

| Concept | Simple Meaning |
| --- | --- |
| Redis | In-memory data store |
| Local Cache | Cache inside one application instance |
| Distributed Cache | Shared cache across multiple application instances |
| L1 Cache | First/local caching layer |
| L2 Cache | Second/shared caching layer |
| Replication | Copying data to other nodes |
| Primary | Main Redis node handling writes |
| Replica | Copy of primary data |
| Replication Lag | Delay in copying updates |
| Sentinel | Monitoring and failover management |
| Redis Cluster | Distributed Redis with sharding |
| Sharding | Splitting data across servers |
| Hash Slots | Redis Cluster's 16,384 logical partitions |
| CDN | Content Delivery Network |
| Edge Server | Server closer to users |
| Origin Server | Original content source |
| CDN Hit | Content found at CDN |
| CDN Miss | Content fetched from origin |
| CDN Invalidation | Removing or refreshing cached content |
| Request Coalescing | Combining duplicate concurrent fetches |
| TTL Jitter | Randomizing expiration times |

---

# 22\. Final Learning Checklist

## Redis

- [ ]Understand Redis.
- [ ]Understand Redis commands.
- [ ]Understand local caching.
- [ ]Understand distributed caching.
- [ ]Understand multi-level caching.
- [ ]Understand Redis replication.
- [ ]Understand replication lag.
- [ ]Understand Redis Sentinel.
- [ ]Understand Redis Cluster.
- [ ]Understand hash slots.
- [ ]Understand sharding.
- [ ]Understand replication vs sharding.

## CDN

- [ ]Understand CDN.
- [ ]Understand edge servers.
- [ ]Understand origin servers.
- [ ]Understand CDN cache hit and miss.
- [ ]Understand CDN invalidation.
- [ ]Understand versioned URLs.
- [ ]Understand CDN vs Redis.

## Interview Preparation

- [ ]Explain distributed caching with a diagram.
- [ ]Explain Redis replication.
- [ ]Explain Redis Sentinel.
- [ ]Explain Redis Cluster.
- [ ]Explain replication vs sharding.
- [ ]Explain CDN architecture.
- [ ]Explain CDN vs Redis.
- [ ]Design image delivery for a Notes application.
- [ ]Explain how to handle Redis failure.

---

# 23\. What Is Next?

## Week 3 – Day 18: File Storage and Object Storage

Tomorrow we will learn:

1. What is file storage?
2. What is block storage?
3. What is object storage?
4. What is AWS S3?
5. Database vs object storage.
6. How image uploads work.
7. How video uploads work.
8. Pre-signed URLs.
9. Multipart uploads.
10. Image and file delivery architecture.
11. Real-world WhatsApp/Instagram-style media storage.

**Day 17 Complete – CDN and Distributed Caching.**

## Before moving to Day 18

Make sure you can explain these five questions:

1. Why do we need distributed caching instead of local caching?
2. What is the difference between Redis replication and sharding?
3. What is Redis Sentinel?
4. What is the difference between Redis and CDN?
5. How would you design image delivery for an application like Instagram?

Next: Day 18 – File Storage and Object Storage.
