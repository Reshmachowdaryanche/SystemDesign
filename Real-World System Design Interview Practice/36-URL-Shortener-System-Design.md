Here is the complete Day 36 lesson in Markdown format, ready to save as:

`36-URL-Shortener-System-Design.md`

Writing

# Week 6 – Day 36: URL Shortener System Design

**File name:** `36-URL-Shortener-System-Design.md`

**Level:** Beginner to Advanced\
**Target:** Senior Android Developer – 7 Years Experience

## Today's Objective

Learn how to design a scalable URL Shortener like Bitly from scratch.

Understand:

- Functional and non-functional requirements
- Capacity estimation
- High-level architecture
- Unique ID generation
- Base62 encoding
- Database design
- Redis caching
- HTTP redirection
- API design
- Analytics architecture
- Horizontal scaling
- Android application integration

---

# 1\. What Is a URL Shortener?

A URL Shortener converts a long URL into a short URL.

**Original URL:**

text

```text
https://www.example.com/products/electronics/
smartphones/samsung-galaxy-s25?campaign=summer2026
```

**Short URL:**

text

```text
https://short.ly/aB91xK
```

When someone opens the short URL, the system redirects them to the original URL.

Popular examples:

- Bitly
- TinyURL
- Rebrandly

## How It Works

1. User submits a long URL.
2. Backend generates a unique short code.
3. Backend stores the mapping in a database.
4. Backend returns the shortened URL.
5. When someone opens the short URL, the backend retrieves the original URL.
6. Browser gets redirected to the original URL.

---

# 2\. Functional Requirements

Functional requirements describe what the system should do.

Our URL Shortener should support:

1. Users can submit a long URL.
2. System generates a unique short URL.
3. Users can access the short URL.
4. System redirects users to the original URL.
5. Users can optionally provide custom aliases.
6. Users can view click analytics.
7. URLs can have expiration dates.
8. Users can deactivate or delete URLs.

**Interview Tip:** Always clarify functional requirements before designing the architecture.

# 3\. Non-Functional Requirements

These describe how well the system should work.

| Requirement | Explanation |
| --- | --- |
| High Availability | Redirection should remain available |
| Low Latency | Redirects should happen quickly |
| Scalability | Support millions of URLs |
| Reliability | Correct URL mapping |
| Durability | Avoid losing stored URLs |
| Consistency | Newly created URLs should resolve correctly |
| Security | Prevent malicious redirects and abuse |

### Important Observation

A URL Shortener is a read-heavy system.

Many more people open shortened URLs than create them.

Therefore, our architecture should optimize the redirect operation.

---

# 4\. Capacity Estimation

Senior-level interviews expect approximate calculations.

Assume:

- 10 million new URLs per month.
- 100 million redirection requests per day.
- Average original URL size: 500 bytes.
- Short code: 7 characters.

## 4.1 Write Requests

10 million URLs per month:

text

```text
10,000,000 / (30 × 24 × 3600)
```

Approximately 4 writes per second.

## 4.2 Read Requests

100 million redirects per day:

text

```text
100,000,000 / 86400
```

Approximately 1,157 reads per second.

Peak traffic could be several times higher.

## 4.3 Storage Estimation

Assume each record requires approximately 600 bytes including URL and basic metadata.

For one year:

text

```text
10M × 12 × 600
= 72GB
```

This is a simplified estimate excluding indexes, replication, database overhead, and analytics.

**Interview Takeaway:** The system is read-heavy, so caching is especially important.

---

# 5\. High-Level Architecture

Main components:

1. Client Application
2. DNS
3. Load Balancer
4. API Gateway
5. URL Shortener Service
6. ID Generation Service
7. Redis Cache
8. Database
9. Analytics Service
10. Message Queue

## 5.1 URL Creation Flow

text

```text
Android App / Web
       |
       v
   API Gateway
       |
       v
 URL Shortener Service
       |
       v
 ID Generation Service
       |
       v
     Database
       |
       v
 Return Short URL
```

## 5.2 URL Redirection Flow

text

```text
User opens short URL
        |
        v
       DNS
        |
        v
  Load Balancer
        |
        v
  Redirect Service
        |
        v
    Redis Cache
        |
   Cache Hit?
     /     \
   Yes      No
    |        |
    |     Database
    |        |
    |     Update Cache
     \      /
        |
        v
  HTTP Redirect
        |
        v
   Original URL
```

### Important Concepts

- API Gateway routes requests.
- Load Balancer distributes traffic.
- Redirect Service handles URL resolution.
- Redis reduces database reads.
- Database stores URL mappings.
- Message Queue handles analytics asynchronously.

---

# 6\. Unique Short Code Generation

This is one of the most important interview topics.

Suppose we have this original URL:

text

```text
https://example.com/products
```

We need to generate:

text

```text
https://short.ly/abc123
```

How do we generate `abc123`?

There are multiple approaches.

## Approach 1: Random String Generation

Generate a random alphanumeric string.

Example:

text

```text
aB91xK
Xy72pQ
mN83zA
```

Advantages:

- Simple implementation.
- Easy to distribute.
- No centralized counter required.

Disadvantages:

- Collision possibility.
- Need to check whether generated code already exists.
- Requires collision handling.

## Approach 2: Auto-Increment ID + Base62

This is a common interview approach.

First, generate a unique numeric ID:

text

```text
125789
```

Then convert it into Base62.

Base62 uses:

- 0–9: 10 characters.
- a–z: 26 characters.
- A–Z: 26 characters.

Total:

text

```text
10 + 26 + 26 = 62
```

Example:

text

```text
Numeric ID: 125789
       |
       v
   Base62
       |
       v
    w7K
```

Advantages:

- Unique numeric IDs produce unique codes.
- Compact representation.
- Efficient encoding and decoding.

Disadvantages:

- ID generation must scale.
- Sequential IDs may reveal approximate creation volume.
- A single database counter can become a bottleneck.

### Distributed ID Generation

For large-scale systems, consider:

- Allocated ID ranges.
- Snowflake-style IDs.
- Dedicated ID generation services.

Base62 can then encode the generated IDs.

---

# 7\. Database Design

We need to store the relationship between short codes and original URLs.

## URL Table

| Column | Type | Description |
| --- | --- | --- |
| id | BIGINT | Unique ID |
| short\_code | VARCHAR | Generated short code |
| original\_url | TEXT | Original URL |
| created\_at | TIMESTAMP | Creation time |
| expires\_at | TIMESTAMP | Expiration |
| user\_id | BIGINT | Owner |
| is\_active | BOOLEAN | Active status |

Example:

| id | short\_code | original\_url |
| --- | --- | --- |
| 1 | aB91xK | example.com/page1 |
| 2 | Xy72pQ | example.com/page2 |
| 3 | mN83zA | example.com/page3 |

## Important Database Concepts

- Primary Key
- Unique Index
- Database Indexing
- Replication
- Sharding
- Partitioning

### Interview Question: Why Do We Need an Index on short\_code?

Because redirection requires looking up the original URL using the short code.

An index avoids scanning every row.

We should also enforce a unique constraint on `short_code` to prevent duplicate mappings.

---

# 8\. Redis Caching

Imagine 10,000 users open the same shortened URL.

Without caching:

text

```text
10,000 requests
       |
       v
    Database
```

This creates unnecessary database load.

With Redis:

text

```text
10,000 requests
       |
       v
      Redis
       |
       v
 Original URL
```

Example cache entry:

text

```text
Key: short_url:aB91xK

Value: https://example.com/products
```

## Cache-Aside Pattern

1. Check Redis.
2. If found, return cached URL.
3. If missing, query database.
4. Store the result in Redis.
5. Return the original URL.

## Important Concepts

- TTL
- Cache Invalidation
- Cache Eviction
- Cache Hit Ratio
- Cache Stampede
- Negative Caching

### Interview Question: What Happens When a URL Is Deleted but Redis Still Contains It?

We must invalidate or update the cache when the URL is deactivated.

A short TTL can also limit stale entries, but it should not replace proper cache invalidation.

---

# 9\. HTTP Redirection

HTTP supports different redirect status codes.

| Code | Meaning |
| --- | --- |
| 301 | Permanent Redirect |
| 302 | Temporary Redirect |
| 303 | See Other |
| 307 | Temporary Redirect, preserves method |
| 308 | Permanent Redirect, preserves method |

For a typical URL Shortener:

- 301 can be used for permanent redirects.
- 302 can be used when temporary behavior or more control over redirect handling is desired.

Be aware that browsers and intermediaries may cache permanent redirects.

Example:

http

```http
HTTP/1.1 302 Found
Location: https://example.com/products
```

The browser reads the Location header and navigates to the original URL.

---

# 10\. API Design

Our system needs APIs for creating and retrieving shortened URLs.

## API 1: Create Short URL

http

```http
POST /api/v1/urls
```

Request:

json

```json
{
  "originalUrl": "https://example.com/products",
  "customAlias": "products",
  "expiresAt": null
}
```

Response:

json

```json
{
  "shortCode": "aB91xK",
  "shortUrl": "https://short.ly/aB91xK"
}
```

## API 2: Redirect

http

```http
GET /{shortCode}
```

Example:

http

```http
GET /aB91xK
```

Response:

http

```http
HTTP/1.1 302 Found
Location: https://example.com/products
```

## API 3: Get Analytics

http

```http
GET /api/v1/urls/aB91xK/analytics
```

Response:

json

```json
{
  "totalClicks": 15000,
  "uniqueVisitors": 8200
}
```

## API 4: Delete Short URL

http

```http
DELETE /api/v1/urls/aB91xK
```

---

# 11\. Analytics Architecture

Suppose 1 million users open a shortened URL.

Should we update the database synchronously for every click?

That can make redirection slower.

Instead, we can use asynchronous processing.

text

```text
User clicks short URL
         |
         v
   Redirect Service
         |
         +------> Redirect immediately
         |
         v
    Message Queue
         |
         v
  Analytics Worker
         |
         v
  Analytics Database
```

Advantages:

- Redirect service remains lightweight.
- Analytics processing happens asynchronously.
- Queue can absorb traffic spikes.
- Analytics workers can scale independently.

Important concepts:

- Event-driven architecture.
- Message Queues.
- Asynchronous Processing.
- Idempotency.
- Duplicate Event Handling.
- Eventual Consistency.

---

# 12\. Scaling the System

What happens when traffic grows from 1,000 requests per second to 100,000 requests per second?

| Component | Scaling Strategy |
| --- | --- |
| API Gateway | Multiple instances |
| Redirect Service | Horizontal scaling |
| Redis | Replication and clustering |
| Database | Read replicas and sharding |
| ID Generator | Distributed ID generation |
| Analytics | Message queues and independent workers |
| CDN | Cache suitable redirect responses carefully |

## Additional Concepts

### Horizontal Scaling

Adding more application instances to handle increasing traffic.

### Database Sharding

Splitting data across multiple database partitions or servers.

### Replication

Maintaining multiple copies of database data.

### Consistent Hashing

Distributing keys across multiple cache nodes.

### Hot Keys

A single popular URL can receive a disproportionately large number of requests.

Possible solutions:

- Local caching.
- Redis replication.
- Request coalescing.
- CDN caching where appropriate.

### High Availability

Use multiple service instances, redundant infrastructure, monitoring, and recovery mechanisms.

---

# 13\. Android Developer Perspective

This is particularly important for Senior Android Developer interviews.

How does an Android application integrate with a URL Shortener backend?

## Android Architecture

text

```text
Compose UI
    |
ViewModel
    |
CreateShortUrlUseCase
    |
UrlRepository
    |
Retrofit API
    |
OkHttp
    |
Backend
```

## Kotlin Request Model

kotlin

```kotlin
data class CreateShortUrlRequest(
    val originalUrl: String,
    val customAlias: String? = null
)
```

## Kotlin Response Model

kotlin

```kotlin
data class CreateShortUrlResponse(
    val shortCode: String,
    val shortUrl: String
)
```

## Retrofit API

kotlin

```kotlin
interface UrlApi {

    @POST("api/v1/urls")
    suspend fun createShortUrl(
        @Body request: CreateShortUrlRequest
    ): CreateShortUrlResponse
}
```

## Repository

kotlin

```kotlin
class UrlRepositoryImpl(
    private val api: UrlApi
) {

    suspend fun createShortUrl(
        url: String
    ): Result<CreateShortUrlResponse> {

        return runCatching {
            api.createShortUrl(
                CreateShortUrlRequest(
                    originalUrl = url
                )
            )
        }
    }
}
```

## ViewModel

kotlin

```kotlin
class UrlViewModel(
    private val repository: UrlRepositoryImpl
) : ViewModel() {

    private val _uiState =
        MutableStateFlow<UrlUiState>(UrlUiState.Idle)

    val uiState = _uiState.asStateFlow()

    fun createShortUrl(url: String) {
        viewModelScope.launch {
            _uiState.value = UrlUiState.Loading

            repository.createShortUrl(url)
                .onSuccess {
                    _uiState.value = UrlUiState.Success(it)
                }
                .onFailure {
                    _uiState.value =
                        UrlUiState.Error(
                            it.message ?: "Something went wrong"
                        )
                }
        }
    }
}
```

For production, also consider:

- URL validation.
- Loading and error states.
- Duplicate submission prevention.
- Authentication.
- Rate limiting.
- Analytics.
- Clipboard integration.
- Deep Links.
- Android App Links.

---

# 14\. Important Senior-Level Interview Questions

Try answering these independently.

## Fundamental Questions

1. What is a URL Shortener?
2. What are its functional and non-functional requirements?
3. Why is this system read-heavy?
4. How does Base62 encoding work?
5. How do you generate unique short codes?

## Database and Caching

6. Why do we need Redis?
7. How do you handle cache invalidation?
8. Why do we need a unique index on short\_code?
9. How would you shard the database?
10. What happens when the database becomes unavailable?

## Distributed Systems

11. How do you avoid ID collisions?
12. How would you handle 100,000 redirects per second?
13. How would you handle a Redis hot key?
14. How do you process analytics asynchronously?
15. How do you maintain high availability?

## Android-Specific Questions

16. How would you design the Android client architecture?
17. How would you handle API failures?
18. How would you prevent duplicate requests?
19. How would you handle deep links?
20. How would you integrate URL sharing and clipboard functionality?

---

# 15\. Practical Interview Assignment

Design a complete URL Shortener system without looking at the notes.

Your design must include:

- [ ]Functional Requirements
- [ ]Non-Functional Requirements
- [ ]Capacity Estimation
- [ ]High-Level Architecture
- [ ]API Design
- [ ]Database Schema
- [ ]Unique ID Generation
- [ ]Base62 Conversion
- [ ]Redis Caching
- [ ]HTTP Redirection
- [ ]Analytics Architecture
- [ ]Scaling Strategy
- [ ]Android Client Architecture

---

# 16\. Day 36 Completion Checklist

- [ ]Understand functional and non-functional requirements.
- [ ]Complete capacity estimation.
- [ ]Draw high-level architecture.
- [ ]Understand unique ID generation and Base62.
- [ ]Design database schema and indexes.
- [ ]Understand Redis and cache invalidation.
- [ ]Understand HTTP redirect status codes.
- [ ]Design REST APIs.
- [ ]Understand asynchronous analytics.
- [ ]Explain horizontal scaling and sharding.
- [ ]Implement Android Retrofit integration.
- [ ]Answer all 20 interview questions.

---

# 17\. What You Should Be Able to Explain After Day 36

You should be able to answer this interview question confidently:

**"Design a URL Shortener like Bitly that supports millions of users. Explain the architecture, database, caching, unique ID generation, and scalability."**

Don't just memorize the diagram.

Explain the reason behind every component and the trade-offs involved.

---

# Next Lesson

**Week 6 – Day 37: WhatsApp / Chat Application System Design**

Topics:

- WebSockets
- Message Delivery
- Message Acknowledgements
- Online/Offline Status
- Message Persistence
- Push Notifications
- Real-Time Messaging Architecture

**Roadmap Progress:** Day 36 of 49.
