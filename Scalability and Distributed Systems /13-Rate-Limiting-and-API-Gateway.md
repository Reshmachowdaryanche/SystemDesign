# Week 2 – Day 13: Rate Limiting and API Gateway

Week 2Day 13Beginner level

File name: `Week-2-Day-13-Rate-Limiting-and-API-Gateway.md`

Today's goal: Understand how backend systems control incoming traffic, protect APIs from overload, and manage requests through an API Gateway.

Writing

# Week 2 – Day 13: Rate Limiting and API Gateway

**Topic:** Rate Limiting, Traffic Management and API Gateway\
**Level:** Beginner\
**Target:** Android Developer – 7 Years Experience

---

# 1\. Why Do We Need Rate Limiting?

Let's imagine you are building a Notes application.

Your application has 1 million users.

Normally, each user makes a few API requests:

- Fetch notes.
- Create a note.
- Update a note.
- Delete a note.
- Search notes.

Now imagine one user accidentally creates a loop in the Android application.

The application starts sending 1,000 requests per second to your backend.

What happens?

- Backend servers receive too many requests.
- CPU and memory usage increase.
- Database connections become exhausted.
- API response time increases.
- Other users experience slow responses.
- Eventually, the service may become unavailable.

We need a mechanism to control how many requests a client can make.

That mechanism is called **Rate Limiting**.

## 1.1 Definition

Rate Limiting is a technique used to control the number of requests a client can make to a server within a specific time period.

Example:

text

```text
Maximum requests: 100
Time period: 1 minute
```

A client can make 100 requests per minute.

If the client sends 101 requests within that period, the additional request may be rejected.

## 1.2 Real-World Example

Imagine a movie theatre.

The theatre can accommodate only 100 people.

Even if 500 people arrive, only 100 people can enter at a time.

The theatre controls entry to prevent overcrowding.

Rate Limiting works similarly.

It controls the number of incoming API requests.

## 1.3 Architecture

text

```text
          ANDROID APP
               |
               v
           API REQUEST
               |
               v
         RATE LIMITER
               |
         +-----+-----+
         |           |
       ALLOW        REJECT
         |           |
         v           v
      BACKEND      HTTP 429
      SERVER
```

## 1.4 Why Is Rate Limiting Important?

1. Prevents API abuse.
2. Protects backend servers.
3. Prevents accidental request loops.
4. Protects databases from excessive queries.
5. Helps control infrastructure costs.
6. Provides fair resource allocation.
7. Reduces the impact of certain denial-of-service attacks.

---

# 2\. What Is a Rate Limit?

A Rate Limit defines the maximum number of requests allowed within a specified time window.

Example:

text

```text
100 requests / minute
```

This means a client is allowed to send up to 100 requests during the configured one-minute window.

Different APIs can have different limits.

Example:

| API | Rate Limit |
| --- | --- |
| Login API | 5 requests/minute |
| Fetch Notes API | 100 requests/minute |
| Search API | 30 requests/minute |
| Create Note API | 50 requests/minute |
| Public API | 1,000 requests/minute |

These are illustrative limits, not universal recommendations.

Why do different APIs have different limits?

Because different operations consume different resources.

For example:

- Login may involve password hashing and authentication checks.
- Search may involve expensive database queries.
- Fetch Notes may be relatively inexpensive.

Therefore, rate limits should depend on business requirements and infrastructure capacity.

---

# 3\. What Is HTTP 429?

When a client exceeds its allowed request limit, the server can return:

http

```http
HTTP/1.1 429 Too Many Requests
```

This status code means the client has sent too many requests within a given period.

Example response:

json

```json
{
  "error": "RATE_LIMIT_EXCEEDED",
  "message": "Too many requests. Please try again later.",
  "retryAfterSeconds": 30
}
```

The server may also include a Retry-After header.

Example:

http

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
```

This tells the client to wait before trying again.

## Android Developer Perspective

When Android receives HTTP 429:

1. Do not immediately retry.
2. Read the Retry-After information if available.
3. Wait for the specified duration.
4. Retry only if the operation is still relevant.
5. Use bounded retries and backoff.

Important:

The backend must enforce rate limiting. Android-side throttling alone cannot protect your backend because clients can bypass it.

---

# 4\. What Is a Rate Limiting Algorithm?

A Rate Limiting Algorithm decides whether an incoming request should be allowed or rejected.

There are several algorithms.

We will learn five important ones:

1. Fixed Window Counter.
2. Sliding Window Log.
3. Sliding Window Counter.
4. Token Bucket.
5. Leaky Bucket.

These are common system design interview topics.

---

# 5\. Fixed Window Counter Algorithm

Let's understand this with an example.

Requirement:

text

```text
Maximum requests: 5
Time window: 1 minute
```

We divide time into fixed one-minute windows.

Example:

text

```text
10:00:00 - 10:00:59
10:01:00 - 10:01:59
10:02:00 - 10:02:59
```

Each window has its own request counter.

## 5.1 How Does It Work?

At the beginning of every window:

text

```text
Request Count = 0
```

For every incoming request:

1. Check the current counter.
2. If the counter is below 5, allow the request.
3. Increase the counter.
4. If the counter is already 5, reject the request.
5. Reset the counter when the next window begins.

## 5.2 Example

Limit: 5 requests per minute.

text

```text
10:00:00 -> Request 1 -> ALLOW
10:00:10 -> Request 2 -> ALLOW
10:00:20 -> Request 3 -> ALLOW
10:00:30 -> Request 4 -> ALLOW
10:00:40 -> Request 5 -> ALLOW
10:00:50 -> Request 6 -> REJECT
```

At 10:01:00, a new window starts.

The counter resets.

text

```text
10:01:00 -> Request 1 -> ALLOW
```

## 5.3 Architecture

text

```text
          API REQUEST
               |
               v
        CHECK CURRENT WINDOW
               |
               v
        READ REQUEST COUNTER
               |
         +-----+-----+
         |           |
       BELOW 5     >= 5
         |           |
         v           v
       ALLOW       REJECT
         |
         v
     INCREMENT
       COUNTER
```

## 5.4 Problem with Fixed Window

There is an important problem called the Boundary Problem.

Imagine the limit is 5 requests per minute.

A client sends:

text

```text
10:00:50 -> 5 requests
10:01:00 -> 5 requests
```

The client sends 10 requests in just 10 seconds.

But the system allows it because these requests belong to two different windows.

This creates a sudden traffic spike.

## 5.5 Advantages

- Simple to implement.
- Low memory usage.
- Easy to understand.
- Good for basic rate limiting.

## 5.6 Disadvantages

- Boundary problem.
- Traffic can spike near window boundaries.
- Not as smooth as sliding-window approaches.

---

# 6\. Sliding Window Log Algorithm

Sliding Window Log solves the boundary problem by tracking the timestamp of each request.

Example:

Limit:

text

```text
5 requests in the last 60 seconds
```

Instead of resetting a counter every minute, we maintain timestamps.

## 6.1 How Does It Work?

For every incoming request:

1. Get the current timestamp.
2. Remove timestamps older than 60 seconds.
3. Count the remaining timestamps.
4. If the count is below 5, allow the request.
5. Store the current request timestamp.
6. Otherwise, reject the request.

## 6.2 Example

Current time:

text

```text
10:01:00
```

Stored request timestamps:

text

```text
10:00:10
10:00:20
10:00:30
10:00:40
10:00:50
```

All five requests are within the last 60 seconds.

Another request arrives.

Result:

text

```text
REJECT
```

At 10:01:11:

The timestamp 10:00:10 is now older than 60 seconds.

It is removed.

A new request can be allowed.

## 6.3 Architecture

text

```text
          API REQUEST
               |
               v
       GET REQUEST TIMESTAMP
               |
               v
     REMOVE OLD TIMESTAMPS
               |
               v
      COUNT RECENT REQUESTS
               |
         +-----+-----+
         |           |
       BELOW 5     >= 5
         |           |
         v           v
       ALLOW       REJECT
         |
         v
     SAVE CURRENT
       TIMESTAMP
```

## 6.4 Advantages

- Accurate sliding time window.
- Solves the fixed-window boundary problem.
- Provides precise control.

## 6.5 Disadvantages

- Stores timestamps for individual requests.
- Higher memory usage.
- More expensive at very high traffic.
- Requires efficient cleanup.

---

# 7\. Sliding Window Counter Algorithm

Sliding Window Counter combines the benefits of Fixed Window Counter and Sliding Window Log.

It reduces memory consumption while providing smoother rate limiting.

## 7.1 Basic Idea

Instead of storing every request timestamp, we maintain:

- Previous window count.
- Current window count.
- Current position within the window.

Then we estimate the number of requests in the last sliding window.

## 7.2 Example

Limit:

text

```text
100 requests per minute
```

Suppose:

Previous window count = 80

Current window count = 20

We are 25% into the current window.

Remaining portion of the previous window:

text

```text
75%
```

Estimated requests:

$$
Estimated = (80 \times 0.75) + 20
$$

$$
Estimated = 60 + 20 = 80
$$

The estimated request count is 80.

If the limit is 100, the next request can be allowed.

## 7.3 Advantages

- Uses less memory than Sliding Window Log.
- Reduces boundary spikes.
- More efficient for high traffic.
- Provides an approximate sliding-window count.

## 7.4 Disadvantages

- Uses an estimation.
- More complex than Fixed Window Counter.
- May not provide exact per-request tracking.

---

# 8\. Token Bucket Algorithm

This is one of the most important rate-limiting algorithms.

Imagine a bucket that contains tokens.

A client must spend one token to make an API request.

If a token is available:

- Allow the request.
- Remove one token.

If no token is available:

- Reject the request or make it wait, depending on the implementation.

## 8.1 Basic Concept

Imagine:

text

```text
Bucket Capacity: 5 tokens
Refill Rate: 1 token/second
```

Initially:

text

```text
[ TOKEN ][ TOKEN ][ TOKEN ][ TOKEN ][ TOKEN ]
```

Each API request consumes one token.

After five requests:

text

```text
[ EMPTY BUCKET ]
```

The client cannot make another request immediately.

However, the bucket refills over time.

After one second:

text

```text
[ TOKEN ]
```

Another request can be allowed.

## 8.2 How Does It Work?

1. Create a bucket with a maximum capacity.
2. Add tokens over time at a configured refill rate.
3. When a request arrives, check token availability.
4. If a token exists, consume it and allow the request.
5. Otherwise, reject or delay the request.

## 8.3 Architecture

text

```text
         API REQUEST
              |
              v
       CHECK TOKEN BUCKET
              |
        +-----+-----+
        |           |
     TOKEN       NO TOKEN
    AVAILABLE        |
        |            v
        v          REJECT
    CONSUME
      TOKEN
        |
        v
      ALLOW
```

## 8.4 Why Is Token Bucket Useful?

Imagine a user opens your Notes application.

The application sends multiple API requests at once:

- Fetch notes.
- Fetch profile.
- Fetch settings.
- Fetch shared notes.

A strict limit of one request per second may make the application feel slow.

Token Bucket allows short bursts because the bucket can accumulate tokens.

But over time, the refill rate controls the average request rate.

## 8.5 Advantages

- Supports temporary traffic bursts.
- Controls average request rate.
- Flexible.
- Suitable for APIs and network traffic management.

## 8.6 Disadvantages

- More complex than fixed counters.
- Requires token state management.
- Distributed implementations need coordination.

---

# 9\. Leaky Bucket Algorithm

Leaky Bucket is another important algorithm.

Imagine a bucket with a small hole at the bottom.

Water enters the bucket at different speeds.

Water leaves at a fixed rate.

If too much water enters and the bucket becomes full, additional water overflows.

In API terms:

- Incoming requests enter a queue.
- Requests are processed at a controlled rate.
- If the queue becomes full, additional requests are rejected.

## 9.1 Example

Configuration:

text

```text
Queue Capacity: 5 requests
Processing Rate: 1 request/second
```

Suppose 10 requests arrive simultaneously.

The system queues up to 5 requests.

The remaining requests are rejected if the queue is full.

The queued requests are processed at a fixed rate.

## 9.2 Architecture

text

```text
       INCOMING REQUESTS
          | | | | |
          v v v v v
       +-------------+
       |   REQUEST   |
       |    QUEUE    |
       +-------------+
              |
              v
       PROCESS 1/SECOND
              |
              v
          BACKEND API
```

## 9.3 Advantages

- Smooths traffic.
- Controls processing rate.
- Prevents sudden backend traffic spikes.
- Useful for systems that need a steady processing rate.

## 9.4 Disadvantages

- Requests may experience additional latency.
- Queue capacity is limited.
- Requests may be rejected when the queue is full.
- Not ideal when immediate responses are required.

---

# 10\. Token Bucket vs Leaky Bucket

This is a common interview question.

| Token Bucket | Leaky Bucket |
| --- | --- |
| Controls request admission using tokens | Controls request processing through a queue |
| Allows short bursts | Smooths outgoing traffic |
| Tokens refill over time | Requests are processed at a controlled rate |
| Useful for burst-friendly APIs | Useful for steady processing workloads |
| Can allow multiple requests immediately | Usually processes queued requests at a controlled pace |

### Easy Way to Remember

Token Bucket:

text

```text
Requests are allowed when tokens exist.
```

Leaky Bucket:

text

```text
Requests are processed at a controlled rate.
```

---

# 11\. Comparison of All Rate Limiting Algorithms

| Algorithm | How It Works | Main Advantage | Main Disadvantage |
| --- | --- | --- | --- |
| Fixed Window | Counts requests in fixed intervals | Simple | Boundary problem |
| Sliding Window Log | Stores request timestamps | Accurate | High memory usage |
| Sliding Window Counter | Estimates recent request count | Memory efficient | Approximation |
| Token Bucket | Uses refillable tokens | Supports bursts | Coordination required |
| Leaky Bucket | Queues and processes requests steadily | Smooth traffic | Adds latency |

### Interview Tip

You don't need to memorize every implementation detail initially.

Understand:

1. What problem the algorithm solves.
2. How it works.
3. Its advantages.
4. Its disadvantages.
5. When you would choose it.

---

# 12\. What Is an API Gateway?

Now let's move to the second major topic.

Imagine your application has several backend services.

text

```text
Android Application
        |
        +------ Notes Service
        |
        +------ User Service
        |
        +------ Payment Service
        |
        +------ Notification Service
```

Should Android directly communicate with every backend service?

It can, but this increases complexity.

Android needs to know:

- Each service URL.
- Authentication requirements.
- Different API endpoints.
- Service changes.
- Error handling.
- Routing rules.

Instead, we can introduce a single entry point.

This is called an API Gateway.

## 12.1 Definition

An API Gateway is a server-side component that acts as an entry point for client requests and routes them to the appropriate backend services.

It can also handle common cross-cutting concerns such as:

- Authentication.
- Rate limiting.
- Request routing.
- Logging.
- Monitoring.
- Request validation.
- Response transformation.

## 12.2 Architecture Without API Gateway

text

```text
              ANDROID APP
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
    NOTES        USERS       PAYMENT
    SERVICE      SERVICE      SERVICE
```

Problems:

- Android knows multiple backend URLs.
- Authentication logic may be repeated.
- Service changes can affect clients.
- Managing requests becomes more complex.

## 12.3 Architecture With API Gateway

text

```text
              ANDROID APP
                   |
                   v
             API GATEWAY
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
    NOTES        USERS       PAYMENT
    SERVICE      SERVICE      SERVICE
```

Now Android communicates with one entry point.

The API Gateway routes requests to the correct service.

---

# 13\. Responsibilities of an API Gateway

Let's understand the important responsibilities.

## 13.1 Request Routing

The API Gateway determines which backend service should receive a request.

Example:

http

```http
GET /api/notes
```

Routes to:

text

```text
Notes Service
```

Another request:

http

```http
GET /api/users/101
```

Routes to:

text

```text
User Service
```

## 13.2 Authentication

The API Gateway can validate authentication credentials such as access tokens.

Example:

Android sends:

http

```http
Authorization: Bearer <access-token>
```

The Gateway validates the token or delegates validation to an authentication component.

If the token is invalid:

http

```http
401 Unauthorized
```

If valid, the request can proceed.

Important:

Authentication may be centralized at the Gateway, but backend services must still enforce their own authorization and security boundaries.

## 13.3 Rate Limiting

The Gateway can apply rate limits.

Example:

text

```text
User 101:
100 requests/minute

User 102:
100 requests/minute
```

If User 101 exceeds the limit, the Gateway can reject additional requests.

## 13.4 Logging and Monitoring

The Gateway can record:

- Request path.
- HTTP method.
- Response status.
- Request duration.
- Correlation ID.

This helps developers investigate production issues.

Avoid logging sensitive information such as passwords, access tokens or private user content.

## 13.5 Load Balancing

The Gateway or its surrounding infrastructure can distribute requests across multiple backend instances.

Example:

text

```text
             API GATEWAY
                  |
          +-------+-------+
          |       |       |
          v       v       v
       SERVER  SERVER  SERVER
          1       2       3
```

This helps distribute incoming traffic.

## 13.6 Response Transformation

Sometimes backend services return different response structures.

The API Gateway can transform responses into a format suitable for the client.

For example, it can combine data from multiple services into one response.

This should be used carefully because excessive gateway logic can make it a complex bottleneck.

---

# 14\. What Is API Gateway Aggregation?

Imagine your Android home screen needs:

- User profile.
- Recent notes.
- Shared notes.
- Notification count.

Without aggregation, Android may need four API calls.

text

```text
ANDROID APP
    |
    +---- GET /profile
    |
    +---- GET /recent-notes
    |
    +---- GET /shared-notes
    |
    +---- GET /notification-count
```

With API aggregation:

Android makes one request:

http

```http
GET /api/home
```

The Gateway or an associated backend aggregation service calls the required services and combines the responses.

text

```text
              ANDROID APP
                   |
                   v
             API GATEWAY
                   |
          +--------+--------+
          |        |        |
          v        v        v
       PROFILE   NOTES   NOTIFICATION
       SERVICE  SERVICE    SERVICE
          |        |        |
          +--------+--------+
                   |
                   v
            COMBINED RESPONSE
                   |
                   v
              ANDROID APP
```

Example response:

json

```json
{
  "profile": {
    "name": "John"
  },
  "recentNotes": [
    {
      "id": 101,
      "title": "System Design"
    }
  ],
  "notificationCount": 5
}
```

### Advantages

- Fewer client-side API calls.
- Simplified Android integration.
- Centralized aggregation.
- Potentially reduced network round trips.

### Disadvantages

- Gateway may become complex.
- One slow backend can delay the combined response.
- Gateway becomes an important dependency.
- Requires careful timeout and partial-failure handling.

---

# 15\. What Is the Difference Between API Gateway and Load Balancer?

This is a very common interview question.

Both are involved in handling traffic, but their responsibilities are different.

## 15.1 Load Balancer

A Load Balancer distributes incoming traffic across multiple server instances.

Example:

text

```text
          CLIENT
             |
             v
        LOAD BALANCER
             |
       +-----+-----+
       |     |     |
       v     v     v
     APP 1  APP 2  APP 3
```

Its primary responsibility is distributing traffic.

## 15.2 API Gateway

An API Gateway acts as an entry point for API requests.

It can:

- Route requests to different services.
- Validate tokens.
- Apply rate limits.
- Log requests.
- Transform responses.
- Aggregate backend responses.

## 15.3 Comparison

| Load Balancer | API Gateway |
| --- | --- |
| Distributes traffic | Manages API requests |
| Focuses on traffic distribution | Focuses on API management |
| Can work at Layer 4 or Layer 7 | Commonly operates at Layer 7 |
| Distributes traffic across instances | Routes requests across services |
| Health checks are common | Authentication and rate limiting are common |

Important:

An API Gateway may itself use a load balancer.

They are complementary components, not necessarily competing technologies.

---

# 16\. What Is Authentication vs Authorization?

Since API Gateway often handles authentication, let's understand two related concepts.

## 16.1 Authentication

Authentication answers:

**Who are you?**

Example:

A user logs into the Notes application.

The backend verifies the credentials and issues an access token.

The Android application sends this token with subsequent API requests.

## 16.2 Authorization

Authorization answers:

**What are you allowed to access?**

Example:

User A owns Note 101.

User B attempts to access Note 101.

Even if User B is authenticated, they should not automatically be allowed to read that note.

The backend must check whether User B has permission.

## 16.3 Comparison

| Authentication | Authorization |
| --- | --- |
| Verifies identity | Verifies permissions |
| Who are you? | What can you access? |
| Happens before access decisions | Uses identity and access rules |
| Example: Login | Example: Access shared note |

Important:

Never rely only on Android UI restrictions for authorization.

Backend services must enforce access control.

---

# 17\. What Is a Reverse Proxy?

A Reverse Proxy is a server that receives requests from clients and forwards them to backend servers.

The client communicates with the proxy rather than directly with the backend.

Example:

text

```text
       ANDROID APP
             |
             v
       REVERSE PROXY
             |
             v
        BACKEND SERVER
```

It can provide:

- Request forwarding.
- TLS termination.
- Load balancing.
- Caching.
- Access control.

An API Gateway is a specialized type of reverse-proxy architecture that provides additional API-specific capabilities.

Examples of technologies used in reverse proxy and API gateway infrastructure include:

- NGINX.
- Envoy.
- Kong.
- AWS API Gateway.

---

# 18\. How Does Everything Work Together?

Let's design the backend architecture for our Notes application.

Requirements:

- 1 million users.
- Users can create, update and delete notes.
- Users can search notes.
- Multiple backend instances.
- Authentication required.
- Rate limiting required.
- Notifications are sent asynchronously.

## 18.1 High-Level Architecture

text

```text
                  ANDROID APP
                       |
                       v
                 LOAD BALANCER
                       |
                       v
                  API GATEWAY
                       |
          +------------+------------+
          |            |            |
          v            v            v
      NOTES API     USER API    SEARCH API
          |            |            |
          v            v            v
       NOTES DB      USER DB    SEARCH ENGINE
          |
          v
       MESSAGE QUEUE
          |
          v
   NOTIFICATION SERVICE
          |
          v
          FCM
          |
          v
      ANDROID APP
```

## 18.2 Request Flow

Imagine a user opens the Notes application.

Step 1: Android sends:

http

```http
GET /api/notes
```

Step 2: Load Balancer receives the request and forwards it to a healthy Gateway instance.

Step 3: API Gateway validates the request and checks authentication.

Step 4: Rate Limiter checks whether the user has exceeded the allowed request limit.

Step 5: Gateway routes the request to Notes API.

Step 6: Notes API fetches notes from the database.

Step 7: Backend returns the response.

Step 8: Gateway forwards the response to Android.

## 18.3 What If the User Exceeds the Rate Limit?

The Gateway rejects the request.

http

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
```

Android waits before retrying.

## 18.4 What If One Backend Server Fails?

The Load Balancer detects that the instance is unhealthy.

It stops sending new requests to that instance and distributes traffic to healthy instances.

## 18.5 What If the Notification Service Fails?

The notification event can remain in the message queue for later processing, subject to the messaging system's retention and delivery configuration.

The Notes API should not need to wait for the notification service to complete before responding to a successful note creation.

---

# 19\. Android Developer Perspective

These topics are useful when designing reliable Android applications.

## 19.1 Handling HTTP 429

When the backend returns 429:

- Read Retry-After.
- Avoid immediate retry.
- Use bounded retry logic.
- Display an appropriate message if needed.

## 19.2 Avoid Unnecessary API Calls

For example, avoid making an API request every time a user types a character in a search box.

Use debouncing.

Example:

text

```text
User types: S
Wait

User types: Sy
Wait

User types: Sys
Wait

User types: System
Wait 300 ms

Send one API request.
```

This reduces unnecessary API calls.

## 19.3 Pagination

When fetching notes, don't request all records at once.

Use pagination.

Example:

http

```http
GET /api/notes?page=1&limit=20
```

This reduces backend load and network usage.

## 19.4 Caching

If profile data does not change frequently, cache it appropriately.

This avoids unnecessary network requests.

## 19.5 WorkManager

For background synchronization:

- Use WorkManager.
- Apply appropriate retry policies.
- Avoid unlimited retries.
- Respect backend Retry-After guidance.
- Use idempotency for operations that may be repeated.

Remember:

Android-side optimizations improve client behavior, but backend rate limiting is still required for server protection.

---

# 20\. Common Mistakes to Avoid

1. Thinking rate limiting and load balancing are the same.
2. Thinking HTTP 429 means the server is unavailable.
3. Retrying immediately after receiving 429.
4. Implementing rate limiting only in Android.
5. Assuming Fixed Window never allows traffic spikes.
6. Confusing Token Bucket and Leaky Bucket.
7. Thinking API Gateway and Load Balancer are identical.
8. Putting all business logic inside the API Gateway.
9. Assuming authentication automatically provides authorization.
10. Forgetting backend permission checks.
11. Ignoring Gateway availability and scalability.
12. Assuming API aggregation always improves performance.
13. Ignoring timeouts when aggregating multiple backend services.

---

# 21\. Important Interview Questions and Answers

### Q1. What is Rate Limiting?

Rate Limiting controls how many requests a client can send within a specified time period.

### Q2. Why do we need Rate Limiting?

To protect backend services, prevent abuse, manage resources and maintain availability.

### Q3. What is HTTP 429?

429 Too Many Requests indicates that the client has exceeded the allowed request rate.

### Q4. What is Fixed Window Counter?

It counts requests within fixed time intervals and resets the counter when the next window starts.

### Q5. What is the boundary problem?

A client may send many requests near the end of one window and the beginning of the next, creating a traffic spike.

### Q6. What is Sliding Window Log?

It stores request timestamps and counts requests within the most recent time window.

### Q7. What is Sliding Window Counter?

It estimates the number of requests in a sliding time window using previous and current window counters.

### Q8. Explain Token Bucket.

A bucket contains tokens that refill over time. Each request consumes one token. Requests are allowed when tokens are available.

### Q9. Explain Leaky Bucket.

Incoming requests enter a queue and are processed at a controlled rate. Requests may be rejected when the queue is full.

### Q10. Difference between Token Bucket and Leaky Bucket?

Token Bucket allows bursts using accumulated tokens. Leaky Bucket smooths request processing through a queue.

### Q11. What is an API Gateway?

An API Gateway acts as a single entry point for client API requests and routes them to the appropriate backend services.

### Q12. What are the responsibilities of an API Gateway?

Routing, authentication, rate limiting, logging, monitoring and response transformation.

### Q13. What is API Aggregation?

Combining data from multiple backend services into a single response for the client.

### Q14. What is the difference between API Gateway and Load Balancer?

A Load Balancer distributes traffic across instances. An API Gateway manages and routes API requests and may provide additional API-related capabilities.

### Q15. What is Authentication?

Authentication verifies the identity of a user or client.

### Q16. What is Authorization?

Authorization determines what resources a user is allowed to access.

### Q17. What is a Reverse Proxy?

A Reverse Proxy receives client requests and forwards them to backend servers.

### Q18. Where should Rate Limiting be implemented?

Commonly at the API Gateway or backend infrastructure, with additional service-level protection when needed.

### Q19. Can we implement Rate Limiting per user?

Yes. The system can identify users through authenticated identities and apply individual request limits.

### Q20. What happens when one backend server fails?

The Load Balancer can detect the unhealthy instance and route new requests to healthy instances.

### Q21. How do you prevent too many search API requests from Android?

Use debouncing, caching, pagination and backend rate limiting.

### Q22. Which Rate Limiting algorithm would you choose for an API?

Token Bucket is a good choice when short bursts are acceptable while controlling average request rate. Sliding Window Counter is useful when smoother time-window limits are preferred.

### Q23. Does an API Gateway replace backend authorization?

No. Backend services must still enforce resource-level authorization and business permissions.

### Q24. Can an API Gateway become a bottleneck?

Yes. It must be horizontally scalable, monitored and designed for high availability.

### Q25. How would you design an API Gateway for a Notes application?

Use a scalable Gateway layer with authentication, rate limiting, request routing, monitoring and load balancing across backend service instances.

---

# 22\. Practical Interview Exercise

Imagine you are designing a Notes application with 1 million users.

Requirements:

- Users can create and search notes.
- Users can share notes.
- The application supports multiple devices.
- Backend has multiple service instances.
- Search APIs are expensive.
- Some users send excessive requests.
- The backend should remain available during traffic spikes.

Answer these questions:

1. Why do we need Rate Limiting?
2. What is HTTP 429?
3. Which Rate Limiting algorithm would you choose?
4. What is the difference between Fixed Window and Sliding Window?
5. Explain Token Bucket with an example.
6. Explain Leaky Bucket with an example.
7. What is an API Gateway?
8. Why should Android communicate through an API Gateway?
9. What are the responsibilities of an API Gateway?
10. What is the difference between an API Gateway and a Load Balancer?
11. What happens if one backend instance fails?
12. How does API Aggregation work?
13. What is the difference between Authentication and Authorization?
14. Where would you implement Rate Limiting?
15. How would you reduce unnecessary search API requests from Android?

Try answering these without looking at the notes.

---

# 23\. Day 13 – Quick Revision

| Concept | Simple Meaning |
| --- | --- |
| Rate Limiting | Controls the number of API requests |
| HTTP 429 | Too Many Requests |
| Fixed Window | Counts requests in fixed intervals |
| Sliding Window Log | Tracks individual request timestamps |
| Sliding Window Counter | Estimates requests in a sliding window |
| Token Bucket | Requests consume refillable tokens |
| Leaky Bucket | Processes requests at a controlled rate |
| API Gateway | Single entry point for APIs |
| Request Routing | Sends requests to appropriate services |
| API Aggregation | Combines multiple service responses |
| Load Balancer | Distributes traffic across instances |
| Authentication | Verifies identity |
| Authorization | Checks permissions |
| Reverse Proxy | Forwards client requests to backend servers |
| Debouncing | Waits for user input to pause before making a request |

---

# 24\. Final Learning Checklist

- [ ]Understand Rate Limiting.
- [ ]Understand why Rate Limiting is needed.
- [ ]Understand HTTP 429.
- [ ]Understand Fixed Window Counter.
- [ ]Understand Sliding Window Log.
- [ ]Understand Sliding Window Counter.
- [ ]Understand Token Bucket.
- [ ]Understand Leaky Bucket.
- [ ]Understand the difference between Token Bucket and Leaky Bucket.
- [ ]Understand API Gateway.
- [ ]Understand API Gateway responsibilities.
- [ ]Understand API Aggregation.
- [ ]Understand Load Balancer.
- [ ]Understand API Gateway vs Load Balancer.
- [ ]Understand Authentication.
- [ ]Understand Authorization.
- [ ]Understand Reverse Proxy.
- [ ]Explain Rate Limiting in a Notes application.
- [ ]Explain the complete API Gateway request flow.

---

# 25\. What Is Next?

## Week 2 – Day 14: Week 2 Revision and System Design Practice

Tomorrow we will revise the complete second week.

Topics covered during Week 2:

1. Database Fundamentals.
2. Database Indexing.
3. Database Replication.
4. Database Sharding.
5. CAP Theorem.
6. Message Queues.
7. Event-Driven Architecture.
8. Consistency and Reliability.
9. ACID and BASE.
10. Optimistic and Pessimistic Locking.
11. Idempotency.
12. Retry and Circuit Breaker.
13. Rate Limiting.
14. API Gateway.

We will also do:

- Interview questions.
- Architecture diagrams.
- Notes application system design.
- Practical scenarios.
- Mock interview questions.

**Day 13 Complete – Rate Limiting and API Gateway.**

## What you should focus on today

For your first system design preparation, don't try to memorize every algorithm.

Make sure you can explain these five concepts clearly:

1. Rate Limiting – Why do we need to control API requests?
2. Token Bucket – How can we allow bursts without overwhelming the backend?
3. Leaky Bucket – How do we process requests at a steady rate?
4. API Gateway – Why do we need a single entry point for backend services?
5. Load Balancer vs API Gateway – What problem does each solve?

Next: Day 14 – Complete Week 2 Revision and Interview Practice.
