# Week 3 – Day 21: Rate Limiting and Throttling

Week 3Day 21Beginner level

File name: `Week-3-Day-21-Rate-Limiting-and-Throttling.md`

Target: Android Developer – 7 Years Experience

Today's goal: Understand how backend systems protect APIs from excessive requests, prevent abuse, and handle high traffic.

Writing

# Week 3 – Day 21: Rate Limiting and Throttling

## 1\. Introduction

Imagine you are developing an Android application.

Your application has an API:

http

```http
GET /api/products
```

Whenever a user opens the product screen, Android calls this API.

Now imagine one user writes a script that sends 10,000 requests per second to your API.

What happens?

- Your backend receives too many requests.
- CPU and memory usage increase.
- Database queries increase.
- Normal users experience slow responses.
- Eventually, your backend may become unavailable.

How can we prevent this?

We use **Rate Limiting**.

Rate limiting controls how many requests a client can make within a specific time period.

For example:

A user can make a maximum of 100 API requests per minute.

If the user sends 101 requests within that minute, the additional request may be rejected.

This protects backend infrastructure.

Today's topics:

1. What is rate limiting?
2. Why do we need rate limiting?
3. Rate limiting vs throttling.
4. Different rate limiting strategies.
5. Fixed Window algorithm.
6. Sliding Window algorithm.
7. Token Bucket algorithm.
8. Leaky Bucket algorithm.
9. Redis-based rate limiting.
10. HTTP 429 status code.
11. API rate limiting architecture.
12. Android developer perspective.

---

# 2\. What Is Rate Limiting?

Rate limiting is a technique used to control the number of requests a client can make to a server within a specified time period.

### Example

Suppose an API has this restriction:

text

```text
Maximum requests: 100
Time window: 1 minute
```

This means one client can make 100 requests per minute.

Example:

text

```text
Request 1   -> Allowed
Request 2   -> Allowed
Request 3   -> Allowed
...
Request 100 -> Allowed
Request 101 -> Rejected
```

The backend rejects requests that exceed the configured limit.

### Simple Definition

Rate limiting controls how frequently a client can access an API within a given time period.

### Real-world Examples

- Instagram limits certain activities to prevent spam.
- Banking applications restrict repeated OTP requests.
- Login APIs restrict repeated authentication attempts.
- Public APIs restrict requests based on subscription plans.

---

# 3\. Why Do We Need Rate Limiting?

There are six important reasons.

## 3.1 Prevent Server Overload

Imagine your backend can comfortably process 5,000 requests per second.

Suddenly, one client sends 50,000 requests per second.

Your backend may become overloaded.

Rate limiting restricts excessive traffic.

## 3.2 Prevent Abuse

Imagine someone repeatedly calls:

http

```http
POST /api/login
```

They may be trying to guess passwords or abuse the authentication API.

Rate limiting restricts repeated attempts.

Note: Rate limiting is only one layer of protection. Authentication security, account lockout policies and other controls are also important.

## 3.3 Protect Database Resources

Every API request may trigger database queries.

For example:

text

```text
Android App
    |
    v
GET /products
    |
    v
Backend
    |
    v
Database Query
```

Too many requests can overload the database.

Rate limiting helps control incoming traffic before it consumes too many resources.

## 3.4 Ensure Fair Usage

Imagine an API serves 10,000 customers.

One customer continuously sends thousands of requests.

Other customers may experience slow responses.

Rate limiting helps distribute resources fairly.

## 3.5 Control API Costs

Some backend operations are expensive.

Examples:

- Image processing.
- AI inference.
- Payment processing.
- Report generation.
- SMS and OTP delivery.

Rate limiting can help control infrastructure and third-party service costs.

## 3.6 Protect API Availability

By controlling excessive traffic, rate limiting helps keep APIs available for legitimate users.

---

# 4\. Rate Limiting vs Throttling

These terms are related, but they are not exactly the same.

## 4.1 Rate Limiting

Rate limiting defines the maximum number of requests allowed within a time period.

Example:

text

```text
100 requests per minute
```

If the client exceeds the limit, additional requests are rejected or otherwise restricted.

## 4.2 Throttling

Throttling controls or slows down the rate at which requests are processed.

For example:

A system may temporarily delay requests when traffic is too high.

It can also reduce processing capacity allocated to a particular client.

### Comparison

| Feature | Rate Limiting | Throttling |
| --- | --- | --- |
| Purpose | Set request limits | Control request processing rate |
| Behavior | Reject or restrict excess requests | Slow down or regulate traffic |
| Example | 100 requests/minute | Process at 10 requests/second |
| Main goal | Prevent excessive usage | Manage traffic and resources |

### Interview Answer

Rate limiting defines how many requests a client can make within a given time window.

Throttling controls the rate at which requests are processed or allowed through.

They are related techniques and are often implemented together.

---

# 5\. Where Is Rate Limiting Implemented?

Rate limiting can be implemented at different layers.

### Architecture

text

```text
       ANDROID APP
            |
            v
       API GATEWAY
            |
      RATE LIMITER
            |
      +-----+-----+
      |     |     |
      v     v     v
    USER  ORDER PRODUCT
    SVC    SVC    SVC
```

The API gateway checks whether the request is allowed before forwarding it to backend services.

### Common Places

1. API Gateway.
2. Reverse Proxy.
3. Backend application.
4. Authentication service.
5. Individual microservices.

### Example

An application may configure different limits:

| API | Limit |
| --- | --- |
| GET /products | 100 requests/minute |
| POST /login | 5 attempts/minute |
| POST /send-otp | 3 requests/minute |
| POST /payments | 10 requests/minute |

These are illustrative limits, not universal recommendations.

Different APIs need different limits because their processing costs and security requirements differ.

---

# 6\. Types of Rate Limiting Algorithms

There are four important algorithms you should understand.

1. Fixed Window Counter.
2. Sliding Window.
3. Token Bucket.
4. Leaky Bucket.

Each algorithm handles traffic differently.

---

# 7\. Fixed Window Counter Algorithm

This is one of the simplest rate limiting algorithms.

### Concept

Divide time into fixed intervals.

For each interval, maintain a request counter.

Example:

text

```text
Limit: 5 requests
Window: 1 minute
```

The counter resets every minute.

### Example

First minute:

text

```text
10:00:00 - 10:00:59

Request 1 -> Allowed
Request 2 -> Allowed
Request 3 -> Allowed
Request 4 -> Allowed
Request 5 -> Allowed
Request 6 -> Rejected
```

At 10:01:00, the counter resets.

The next five requests are allowed.

### Architecture

text

```text
       CLIENT
          |
          v
      RATE LIMITER
          |
          v
      COUNTER = 0

      Request arrives
          |
          v
      COUNTER + 1
          |
          v
      Is counter <= 5?
         /    \
       YES     NO
        |       |
        v       v
      ALLOW   REJECT
```

### Pseudocode

kotlin

```kotlin
if (requestCount < limit) {
    requestCount++
    allowRequest()
} else {
    rejectRequest()
}
```

This is conceptual pseudocode. A production implementation must update the counter atomically and manage the window correctly.

### Advantages

- Very simple.
- Easy to implement.
- Low memory overhead.
- Fast.

### Disadvantages

There is a problem called the **Boundary Problem**.

### Boundary Problem

Suppose:

text

```text
Limit: 5 requests per minute
```

A user sends:

text

```text
10:00:50 -> 5 requests
10:01:05 -> 5 requests
```

The user sends 10 requests in just 15 seconds.

Why?

Because the first five belong to the 10:00 window and the next five belong to the 10:01 window.

The fixed window algorithm allows both groups.

This can create sudden traffic spikes near window boundaries.

### When to Use?

- Simple APIs.
- Basic request limits.
- Applications where occasional boundary bursts are acceptable.

---

# 8\. Sliding Window Algorithm

Sliding Window improves the boundary problem of Fixed Window.

Instead of using only fixed time boundaries, it considers requests over a moving time interval.

### Concept

Suppose the limit is:

text

```text
5 requests per minute
```

Whenever a new request arrives, check how many requests were made in the previous 60 seconds.

If fewer than five requests exist, allow the request.

Otherwise, reject it.

### Example

Current time:

text

```text
10:01:30
```

Check requests made between:

text

```text
10:00:30 and 10:01:30
```

If five requests already exist, reject the new request.

### Architecture

text

```text
       CLIENT
          |
          v
      RATE LIMITER
          |
          v
    CHECK LAST 60 SECONDS
          |
          v
    COUNT REQUESTS
          |
      +---+---+
      |       |
    Below   Reached
     limit    limit
      |        |
      v        v
    ALLOW    REJECT
```

### Example

text

```text
Limit: 5 requests per 60 seconds

10:00:10 -> Request 1
10:00:20 -> Request 2
10:00:30 -> Request 3
10:00:40 -> Request 4
10:00:50 -> Request 5

10:01:00 -> Request 6 REJECTED
```

At 10:01:11, the first request is now outside the previous 60-second window.

A new request may be allowed.

### Advantages

- Avoids the fixed-window boundary problem.
- More accurate request limiting.
- Controls traffic over a rolling time period.

### Disadvantages

- Requires tracking request timestamps or maintaining window counters.
- Can consume more memory.
- More complex than fixed window.

### When to Use?

When accurate request limits are important and sudden boundary bursts should be avoided.

---

# 9\. Token Bucket Algorithm

Token Bucket is one of the most commonly discussed rate limiting algorithms.

It allows controlled bursts while maintaining an average request rate.

### Concept

Imagine a bucket containing tokens.

Each API request requires one token.

If a token is available:

- Remove one token.
- Allow the request.

If no token is available:

- Reject the request or delay it.

Tokens are added back into the bucket at a fixed rate.

### Example

text

```text
Bucket capacity: 10 tokens
Refill rate: 2 tokens per second
```

Initially:

text

```text
Bucket = 10 tokens
```

A client sends five requests.

text

```text
10 - 5 = 5 tokens
```

Now five tokens remain.

If the client sends another five requests:

text

```text
5 - 5 = 0 tokens
```

The next request is rejected because no token is available.

After one second, two tokens are added.

The client can now make two more requests.

### Architecture

text

```text
        TOKEN GENERATOR
          (2 tokens/sec)
                |
                v
        +---------------+
        | TOKEN BUCKET  |
        |               |
        | 10 Tokens Max |
        +---------------+
                |
                v
          API REQUEST
                |
                v
         TOKEN AVAILABLE?
             /     \
           YES      NO
            |        |
            v        v
          REMOVE   REJECT
           TOKEN
            |
            v
          ALLOW
```

### Advantages

- Supports controlled bursts.
- Maintains a configurable average request rate.
- Flexible.
- Useful for APIs with varying traffic patterns.

### Disadvantages

- Slightly more complex.
- Requires accurate token accounting.
- In distributed systems, token state must be shared or coordinated.

### When to Use?

- Public APIs.
- Mobile application APIs.
- Systems where occasional bursts are acceptable.
- API gateways.

---

# 10\. Leaky Bucket Algorithm

Leaky Bucket is another popular algorithm.

Imagine a bucket with a small hole at the bottom.

Water enters the bucket at different speeds.

But water leaves at a fixed rate.

If too much water enters and the bucket becomes full, additional water overflows.

### Concept

Incoming requests enter a queue.

Requests are processed at a fixed rate.

Example:

text

```text
Incoming requests: 100 requests/sec
Processing rate: 10 requests/sec
Queue capacity: 50 requests
```

The system processes 10 requests per second.

Other requests wait in the queue.

If the queue becomes full, additional requests are rejected.

### Architecture

text

```text
        CLIENT REQUESTS
               |
               v
        +--------------+
        | LEAKY BUCKET |
        |              |
        | Request Queue|
        |              |
        +--------------+
               |
               v
       PROCESS 10 REQUESTS/SEC
               |
               v
           BACKEND
```

### Advantages

- Smooths sudden traffic spikes.
- Maintains a predictable processing rate.
- Protects backend services from bursts.

### Disadvantages

- Requests may experience waiting time.
- Queue management is required.
- Requests may be dropped when the queue becomes full.

### When to Use?

- Systems requiring smooth request processing.
- Background processing.
- Message processing pipelines.
- Traffic shaping.

---

# 11\. Token Bucket vs Leaky Bucket

This is a very common interview question.

| Feature | Token Bucket | Leaky Bucket |
| --- | --- | --- |
| Main idea | Requests consume tokens | Requests enter a queue |
| Traffic bursts | Allows controlled bursts | Smooths bursts |
| Processing | Based on token availability | At a controlled rate |
| Queue | Not required by the basic algorithm | Usually required |
| Overflow | Request rejected when tokens unavailable | Request rejected when queue is full |
| Use case | API rate limiting | Traffic shaping |

### Easy Way to Remember

Token Bucket:

Allows requests when tokens are available.

Leaky Bucket:

Processes queued requests at a controlled rate.

---

# 12\. What Is Redis?

Redis is an in-memory data store commonly used for caching and fast data access.

Why do we need Redis for rate limiting?

Imagine your application has five backend servers.

text

```text
        LOAD BALANCER
              |
       +------+------+------+
       |      |      |      |
       v      v      v      v
      S1     S2     S3     S4
```

Suppose the rate limit is:

text

```text
100 requests per minute per user
```

Server 1 receives 30 requests.

Server 2 receives 30 requests.

Server 3 receives 20 requests.

Server 4 receives 20 requests.

If each server maintains its own local counter, it may incorrectly allow more than 100 requests.

Why?

Because every server sees only its own requests.

### Solution: Shared Redis Counter

All backend servers use the same Redis counter.

text

```text
       +------+-------+------+
       |      |       |      |
       v      v       v      v
      S1     S2      S3     S4
       |      |       |      |
       +------+-------+------+
                  |
                  v
                REDIS
                  |
                  v
          USER_123_COUNTER
```

Now all servers share the same rate limit state.

Example Redis key:

text

```text
rate_limit:user_123
```

Value:

text

```text
100
```

The exact implementation depends on the chosen algorithm.

### Why Redis?

- Very fast in-memory operations.
- Shared state across backend instances.
- Supports atomic operations.
- Supports key expiration.
- Useful for distributed rate limiting.

### Important Interview Point

Redis helps coordinate rate limiting across multiple backend servers.

But the counter update and limit check must be atomic to avoid race conditions when many requests arrive simultaneously.

---

# 13\. Redis-Based Fixed Window Example

Suppose:

text

```text
Limit: 5 requests
Window: 60 seconds
```

Conceptual flow:

1. Android sends an API request.
2. Backend identifies the user.
3. Backend increments the user's Redis counter.
4. If this is the first request in the window, set the expiration.
5. Check the counter.
6. Allow or reject the request.

### Pseudocode

text

```text
key = "rate_limit:user_123"

count = INCREMENT(key)

IF count == 1:
    SET_EXPIRATION(key, 60 seconds)

IF count <= 5:
    ALLOW_REQUEST
ELSE:
    REJECT_REQUEST
```

This is simplified pseudocode. In production, incrementing the counter and setting its expiration should be done atomically, for example through a Redis Lua script or an equivalent atomic operation.

### Example

| Request | Redis Counter | Result |
| --- | --- | --- |
| 1 | 1 | Allowed |
| 2 | 2 | Allowed |
| 3 | 3 | Allowed |
| 4 | 4 | Allowed |
| 5 | 5 | Allowed |
| 6 | 6 | Rejected |
| 7 | 7 | Rejected |

After 60 seconds, the counter expires.

A new window starts.

---

# 14\. What Is HTTP 429?

HTTP 429 means:

**Too Many Requests.**

The server returns this status code when the client has sent too many requests within a given time period.

### Example

Android sends:

http

```http
GET /api/products
```

Backend responds:

http

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
Content-Type: application/json
```

Response body:

json

```json
{
  "message": "Rate limit exceeded",
  "retryAfterSeconds": 30
}
```

### Important Header: Retry-After

This header tells the client how long it should wait before retrying.

For example:

http

```http
Retry-After: 30
```

The client should wait approximately 30 seconds before retrying.

The header can also contain an HTTP date.

### Other Useful Rate Limit Headers

Some APIs provide:

http

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 20
X-RateLimit-Reset: 1790000000
```

Meaning:

- Limit: Maximum allowed requests.
- Remaining: Requests remaining in the current window.
- Reset: When the limit window resets, according to the API's convention.

These `X-RateLimit-*` headers are common conventions, not universal HTTP standards. Follow the API's documented behavior.

---

# 15\. Rate Limiting Based on Different Identifiers

How does the backend identify which client is making requests?

There are several options.

## 15.1 IP-Based Rate Limiting

Requests are limited based on client IP address.

Example:

text

```text
IP: 192.168.1.10
Limit: 100 requests/minute
```

Useful for:

- Public APIs.
- Unauthenticated endpoints.
- Basic abuse protection.

Problem:

Many users may share the same public IP address because of NAT, office networks or mobile carrier networks.

## 15.2 User-Based Rate Limiting

Requests are limited using the authenticated user ID.

Example:

text

```text
User ID: 12345
Limit: 100 requests/minute
```

Useful for authenticated APIs.

## 15.3 API Key-Based Rate Limiting

Requests are limited using an API key.

Useful for public developer APIs.

## 15.4 Endpoint-Based Rate Limiting

Different endpoints have different limits.

Example:

text

```text
GET /products     -> 100/minute
POST /login       -> 5/minute
POST /send-otp    -> 3/minute
```

## 15.5 Combined Rate Limiting

A production application may combine:

- User ID.
- IP address.
- API key.
- Endpoint.
- Subscription plan.

For example:

text

```text
User 123
    |
    v
100 requests/minute

OTP endpoint
    |
    v
3 requests/minute
```

This provides more flexible protection.

---

# 16\. Real-World System Design Example: Banking Application

Imagine a banking Android application.

It has these APIs:

text

```text
POST /login
POST /send-otp
POST /verify-otp
GET  /account/balance
POST /transfer
```

Different APIs need different limits.

### Example Configuration

| API | Example Limit | Reason |
| --- | --- | --- |
| Login | 5 attempts/minute | Prevent repeated login attempts |
| Send OTP | 3 requests/minute | Prevent OTP abuse |
| Verify OTP | Limited attempts | Prevent repeated guessing |
| Account Balance | 30 requests/minute | Prevent excessive polling |
| Transfer | Business-specific limits | Protect sensitive operations |

These are illustrative values. Real banking systems use carefully designed security and business policies.

### Architecture

text

```text
          ANDROID APP
                |
                v
           API GATEWAY
                |
                v
          RATE LIMITER
                |
                v
              REDIS
                |
                v
       +--------+--------+
       |        |        |
       v        v        v
     LOGIN     OTP    ACCOUNT
     SERVICE SERVICE  SERVICE
```

### Example Flow

User taps Send OTP.

1. Android calls `/send-otp`.
2. API gateway identifies the user.
3. Rate limiter checks the user's request count.
4. Redis provides the current counter.
5. If the limit is not exceeded, request is forwarded.
6. OTP service processes the request.
7. If the limit is exceeded, backend returns HTTP 429.

---

# 17\. Android Developer Perspective

As an Android developer, you should understand how to handle rate limiting properly.

Imagine you call:

http

```http
GET /api/products
```

The backend responds:

http

```http
429 Too Many Requests
Retry-After: 30
```

What should Android do?

It should not continuously retry the API.

Instead:

1. Read the HTTP status code.
2. Check the Retry-After header.
3. Inform the user if necessary.
4. Wait before retrying.
5. Avoid unnecessary repeated API calls.

## 17.1 Retrofit Example

kotlin

```kotlin
suspend fun getProducts(): Response<ProductResponse> {
    return apiService.getProducts()
}
```

Handling the response:

kotlin

```kotlin
when (response.code()) {

    200 -> {
        // Success
    }

    429 -> {
        // Too many requests
        // Read Retry-After header
    }

    500 -> {
        // Internal server error
    }

    else -> {
        // Handle other errors
    }
}
```

## 17.2 Reading Retry-After

kotlin

```kotlin
val retryAfter = response
    .headers()
    .get("Retry-After")
```

If the server returns:

http

```http
Retry-After: 30
```

The value indicates that the client should wait 30 seconds.

Your application should also handle cases where the header is missing or uses an HTTP date.

## 17.3 Avoid Unnecessary API Calls

As an Android developer, you can help reduce backend traffic.

Examples:

- Debounce search API calls.
- Avoid calling the same API repeatedly.
- Cache responses where appropriate.
- Avoid unnecessary polling.
- Cancel outdated requests.
- Use pagination for large datasets.

### Example: Search Debouncing

Imagine a user types:

text

```text
Android
```

Without debouncing:

text

```text
A       -> API call
An      -> API call
And     -> API call
Andr    -> API call
Andro   -> API call
Android -> API call
```

Six requests are generated.

With debouncing:

Wait until the user stops typing for 300 milliseconds before making the API request.

Now unnecessary requests are reduced.

This is one practical way Android developers can help protect backend resources.

---

# 18\. Rate Limiting vs Caching

These concepts are different.

### Caching

Caching stores previously retrieved data so the system does not need to perform the same expensive operation repeatedly.

Example:

text

```text
GET /products

First request -> Database
Second request -> Cache
```

### Rate Limiting

Rate limiting controls how many requests a client can make.

Example:

text

```text
100 requests/minute
```

### Comparison

| Feature | Caching | Rate Limiting |
| --- | --- | --- |
| Purpose | Reduce repeated work | Control request frequency |
| Main benefit | Better performance | Protect resources |
| Example | Redis response cache | Redis request counter |
| Prevents abuse | Not by itself | Helps |

Both techniques can be used together.

---

# 19\. Common Interview Mistakes

1. Thinking rate limiting and throttling are identical.
2. Assuming fixed window never allows sudden bursts.
3. Confusing Token Bucket and Leaky Bucket.
4. Forgetting distributed rate limiting.
5. Maintaining separate local counters on each backend server.
6. Not considering atomic counter updates.
7. Ignoring the HTTP 429 response.
8. Retrying requests continuously from Android.
9. Using only IP-based limits for authenticated users.
10. Applying identical limits to every API.
11. Assuming rate limiting alone prevents all security attacks.
12. Forgetting that Redis itself needs availability and capacity planning.

---

# 20\. Important Interview Questions and Answers

### Q1. What is rate limiting?

Rate limiting controls the number of requests a client can make within a specified time period to protect backend resources.

### Q2. Why do we need rate limiting?

To prevent abuse, protect backend resources, ensure fair usage, control costs and maintain API availability.

### Q3. What is the difference between rate limiting and throttling?

Rate limiting defines request limits, while throttling controls the rate at which requests are processed or allowed through.

### Q4. What is the Fixed Window algorithm?

It divides time into fixed intervals and maintains a request counter for each interval.

### Q5. What is the boundary problem?

A client can send requests at the end of one fixed window and the beginning of the next, causing a sudden burst beyond the intended rate over a short period.

### Q6. What is Sliding Window?

It checks the number of requests made during a moving time interval instead of relying only on fixed time boundaries.

### Q7. What is Token Bucket?

It maintains tokens that are replenished at a fixed rate. Each request consumes one token. Requests are allowed when tokens are available.

### Q8. What is Leaky Bucket?

It places incoming requests into a queue and processes them at a controlled rate.

### Q9. Difference between Token Bucket and Leaky Bucket?

Token Bucket allows controlled bursts when tokens are available. Leaky Bucket smooths incoming traffic by processing queued requests at a controlled rate.

### Q10. Why do we use Redis for rate limiting?

Redis provides fast shared storage, atomic operations and expiration support, allowing multiple backend servers to share rate limit counters.

### Q11. What happens when a rate limit is exceeded?

The server may reject the request with HTTP 429 Too Many Requests.

### Q12. What is Retry-After?

It is an HTTP response header that indicates when the client should retry after receiving a response such as HTTP 429.

### Q13. Why is IP-based rate limiting not always sufficient?

Multiple users may share the same public IP address. Therefore, authenticated APIs often benefit from user-based limits.

### Q14. How do you implement distributed rate limiting?

Use shared state such as Redis so that multiple backend servers can access a common request counter or rate limiter state. Updates must be atomic.

### Q15. How can Android reduce API requests?

By using debouncing, caching, pagination, request cancellation and avoiding unnecessary polling.

### Q16. What is the difference between rate limiting and caching?

Caching avoids repeated expensive operations by reusing stored data. Rate limiting restricts how frequently clients can make requests.

### Q17. Can rate limiting prevent DDoS attacks?

It can help control excessive requests at the application level, but it cannot independently prevent all DDoS attacks. Network-level protection and infrastructure-level defenses are also required.

### Q18. Should every API have the same rate limit?

No. Different APIs have different costs, security requirements and business rules. Limits should be configured accordingly.

### Q19. What happens if Redis becomes unavailable?

The rate limiter may lose access to shared state. The system needs an explicit failure policy, such as fail-open or fail-closed, depending on the endpoint's security and availability requirements.

### Q20. Where should rate limiting be implemented?

Common locations include API gateways, reverse proxies and backend services. Critical operations may also require additional service-level limits.

---

# 21\. Scenario-Based Interview Questions

## Scenario 1: One User Sends 10,000 Requests

Your backend receives thousands of requests from one user.

How would you handle this?

**Expected answer:**

Implement user-based rate limiting at the API gateway or backend using a suitable algorithm. Reject excess requests with HTTP 429.

## Scenario 2: Multiple Backend Servers

Your application has five backend servers.

Each server maintains its own request counter.

Is this correct?

**Expected answer:**

Not for a global per-user rate limit. Each server would see only part of the traffic. Use shared rate limit state such as Redis and atomic updates.

## Scenario 3: Sudden Request Burst

Your API allows 100 requests per minute, but users send bursts of requests.

Which algorithm would you consider?

**Expected answer:**

Token Bucket is useful when controlled bursts are acceptable. Sliding Window is useful when a more consistent rolling request limit is required.

## Scenario 4: Search API Gets Too Many Requests

A user types continuously into the search field, causing an API request for every character.

What would you do?

**Expected answer:**

Use debounce on the Android side and rate limiting on the backend side.

## Scenario 5: Android Receives HTTP 429

The backend returns:

http

```http
429 Too Many Requests
Retry-After: 30
```

What should Android do?

**Expected answer:**

Read the Retry-After header, avoid immediate retries and retry only after the indicated waiting period when appropriate.

## Scenario 6: Redis Goes Down

Your distributed rate limiter depends on Redis, but Redis becomes unavailable.

What happens?

**Expected answer:**

The system needs a failure policy. For security-sensitive operations, fail-closed may be appropriate. For some less sensitive APIs, fail-open may preserve availability. The decision depends on business and security requirements.

---

# 22\. Day 21 – Quick Revision Cheat Sheet

| Concept | Meaning |
| --- | --- |
| Rate Limiting | Controls number of requests |
| Throttling | Controls request processing rate |
| Fixed Window | Counter resets at fixed intervals |
| Sliding Window | Checks a moving time interval |
| Token Bucket | Requests consume replenished tokens |
| Leaky Bucket | Requests are processed at a controlled rate |
| Redis | Shared in-memory storage |
| HTTP 429 | Too Many Requests |
| Retry-After | Indicates when to retry |
| IP-Based Limiting | Limits requests by IP |
| User-Based Limiting | Limits requests by user |
| API Key Limiting | Limits requests by API key |
| Debouncing | Reduces repeated calls during user input |
| Distributed Rate Limiting | Shared limits across multiple servers |

---

# 23\. Final Learning Checklist

## Fundamentals

- [ ]Understand rate limiting.
- [ ]Understand throttling.
- [ ]Understand why rate limiting is needed.
- [ ]Understand where rate limiting is implemented.

## Algorithms

- [ ]Understand Fixed Window.
- [ ]Understand Sliding Window.
- [ ]Understand Token Bucket.
- [ ]Understand Leaky Bucket.
- [ ]Understand their differences.

## Backend

- [ ]Understand Redis.
- [ ]Understand distributed rate limiting.
- [ ]Understand atomic counter updates.
- [ ]Understand IP-based limiting.
- [ ]Understand user-based limiting.
- [ ]Understand endpoint-based limiting.

## Android

- [ ]Understand HTTP 429.
- [ ]Understand Retry-After.
- [ ]Know how to handle 429 in Retrofit.
- [ ]Understand debounce.
- [ ]Understand why unnecessary API calls should be avoided.

## System Design

- [ ]Draw a rate limiting architecture.
- [ ]Explain Redis-based rate limiting.
- [ ]Explain what happens when Redis fails.
- [ ]Explain how multiple backend servers share rate limits.

---

# 24\. What Is Next?

## Week 4 – Day 22: Message Queues and Asynchronous Processing

Tomorrow we will learn:

1. What is synchronous communication?
2. What is asynchronous communication?
3. What is a message queue?
4. Why do we need message queues?
5. Producer and Consumer.
6. Queue vs Topic.
7. Kafka basics.
8. RabbitMQ basics.
9. Message delivery guarantees.
10. Retry and Dead Letter Queue.
11. Event-driven architecture.
12. Android notification example.
13. Real-world order processing architecture.

**Day 21 Complete – Rate Limiting and Throttling.**

## Day 21 – Test yourself

Before moving to Day 22, try explaining these five questions without looking at your notes:

1. Why do we need rate limiting?
2. What is the difference between Fixed Window and Sliding Window?
3. Explain Token Bucket with a real-world example.
4. Why do we use Redis when multiple backend servers exist?
5. What should an Android application do when it receives HTTP 429?

Important interview tip: As an Android developer, don't just memorize backend algorithms. Understand how your API calls contribute to backend traffic and how Android can reduce unnecessary requests. That connection will make your system design answers stronger.
