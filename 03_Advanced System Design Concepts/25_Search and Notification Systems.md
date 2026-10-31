Let's start Week 4 – Day 25: Search and Notification Systems.

This is an important topic for Senior Android Developer interviews because it connects backend architecture, database design, distributed systems, and Android application architecture.

We'll create notes that take you from absolute basics to senior-level system design discussions.

Day 25Week 4 · Advanced System Design

# Search and Notification Systems

Topics covered

- Full-Text Search
- Search Engine Architecture
- Elasticsearch
- Indexing and Search Optimization
- Notification System Architecture
- Firebase Cloud Messaging (FCM)
- Push Notification Delivery
- Retry Mechanisms and Reliability
- Android Integration
  Learning level
  Beginner → Advanced
  Interview focus
  Senior Android Developer

Writing

# Day 25 – Search and Notification Systems

**Week:** 4 – Advanced System Design Concepts\
**Day:** 25\
**Level:** Beginner to Advanced\
**Focus:** Full-Text Search, Elasticsearch, Notification Architecture, FCM, Delivery and Retry

---

# 1\. Introduction

Modern applications need two important capabilities:

1. Search information quickly.
2. Notify users when something important happens.

Consider an e-commerce application.

A user should be able to:

- Search for a product by name.
- Search using partial keywords.
- Filter products by category, price and rating.
- Receive an order confirmation notification.
- Receive a notification when an order is shipped.
- Receive a notification when an order is delivered.

These requirements introduce two important system design components:

**Search System:** Helps users find relevant information efficiently.

**Notification System:** Delivers important information to users through appropriate communication channels.

Both systems must support scalability, reliability, performance and fault tolerance.

# PART 1 – SEARCH SYSTEMS

# 2\. What is a Search System?

A search system allows users to find relevant information from a large collection of data using keywords, phrases or filters.

For example, imagine an e-commerce application containing 10 million products.

A user searches for:

`Samsung Galaxy S24`

The system should return relevant products quickly.

A search system typically supports:

- Keyword-based search.
- Partial text matching.
- Full-text search.
- Filtering.
- Sorting.
- Ranking and relevance.
- Pagination.
- Autocomplete and suggestions.

## 2.1 Why can't we use a normal database query?

Suppose we have a products table:

| id | name | category |
| --- | --- | --- |
| 1 | Samsung Galaxy S24 | Mobile |
| 2 | Samsung Galaxy S23 | Mobile |
| 3 | Apple iPhone 15 | Mobile |
| 4 | OnePlus 12 | Mobile |

We can search using SQL:

sql

```sql
SELECT *
FROM products
WHERE name LIKE '%Samsung%';
```

This works for small datasets.

However, as the dataset grows, several problems appear:

1. Full-table scans can become expensive.
2. Complex text matching becomes difficult.
3. Relevance ranking is limited.
4. Typographical errors are difficult to handle.
5. Autocomplete and advanced search become complicated.
6. Search workloads can affect normal database operations.

**Important:** A relational database can support full-text search and indexing. A dedicated search engine is not mandatory for every application.

We introduce a dedicated search engine when search requirements and workload justify its complexity.

# 3\. What is Full-Text Search?

Full-text search means searching for words or phrases inside textual content rather than requiring an exact field match.

For example, consider a product description:

"Samsung Galaxy S24 with advanced camera and powerful processor."

A user searches:

`advanced camera`

The search engine should identify this document even though the exact phrase may not be stored as a separate database field.

Full-text search generally involves three important operations:

1. Tokenization
2. Normalization
3. Indexing

## 3.1 Tokenization

Tokenization breaks text into smaller searchable units called tokens.

Example:

Input:

`Samsung Galaxy S24 Ultra`

Tokens:

text

```text
Samsung
Galaxy
S24
Ultra
```

The search engine can now search individual terms.

## 3.2 Normalization

Normalization converts text into a consistent format.

Examples:

text

```text
Samsung
SAMSUNG
samsung
```

These may be normalized to:

text

```text
samsung
```

Other normalization techniques include:

- Lowercasing.
- Removing unnecessary punctuation.
- Stemming.
- Lemmatization.
- Removing stop words where appropriate.

**Stemming:** Reduces words to a common root-like form.

Example:

text

```text
running → run
played  → play
```

**Stop Words:** Common words that may be excluded from certain search indexes.

Examples:

- the
- is
- a
- an

Stop-word removal must be chosen carefully because some words matter in particular searches.

# 4\. What is an Inverted Index?

An inverted index is one of the most important concepts in search system design.

A normal database stores documents and their content.

An inverted index stores the relationship between terms and the documents containing those terms.

Example documents:

text

```text
Document 1: Samsung Galaxy S24
Document 2: Samsung Galaxy S23
Document 3: Apple iPhone 15
```

The inverted index looks like:

| Term | Document IDs |
| --- | --- |
| samsung | 1, 2 |
| galaxy | 1, 2 |
| s24 | 1 |
| s23 | 2 |
| apple | 3 |
| iphone | 3 |
| 15 | 3 |

Now suppose the user searches:

`samsung`

The search engine looks up the term in the inverted index and immediately identifies documents 1 and 2.

It does not need to examine every document individually.

### Why is an inverted index useful?

- Faster text searching.
- Efficient keyword lookup.
- Supports large datasets.
- Supports relevance scoring.
- Supports complex search queries.

**Interview point:** An inverted index is a fundamental data structure used by search engines such as Elasticsearch.

# 5\. What is Elasticsearch?

Elasticsearch is a distributed search and analytics engine built on Apache Lucene.

It allows applications to store searchable documents and execute fast, complex search queries.

Common use cases:

- E-commerce product search.
- Social media search.
- Application log search.
- Document search.
- Location-based search.
- Analytics.

## 5.1 Important Elasticsearch terminology

### Cluster

A collection of one or more Elasticsearch nodes working together.

### Node

An individual Elasticsearch server.

### Index

A logical collection of documents.

Similar to a database table conceptually, although an Elasticsearch index has its own storage and indexing architecture.

### Document

A JSON object stored inside an index.

Example:

json

```json
{
  "id": 101,
  "name": "Samsung Galaxy S24",
  "brand": "Samsung",
  "price": 79999,
  "category": "Mobile"
}
```

### Shard

A partition of an index.

Sharding distributes data across multiple nodes.

### Replica

A copy of a shard.

Replicas improve availability and can help distribute search traffic.

## 5.2 Elasticsearch architecture

text

```text
              Application
                   |
                   v
             Search API
                   |
                   v
          Elasticsearch Cluster
                   |
          +--------+--------+
          |                 |
          v                 v
        Node 1            Node 2
          |                 |
       Shard 1           Shard 2
          |                 |
       Replica            Replica
```

A cluster distributes data and search workloads across nodes.

## 5.3 How does Elasticsearch indexing work?

Suppose a product is added to the main application database.

The system needs to make it searchable.

Typical flow:

1. Product is saved in the primary database.
2. An event is generated.
3. An indexing worker consumes the event.
4. The worker transforms the product into a search document.
5. The document is indexed in Elasticsearch.
6. The product becomes searchable after the indexing operation is visible to search.

Example:

text

```text
Product Service
      |
      v
Primary Database
      |
      v
Product Created Event
      |
      v
Message Queue
      |
      v
Indexing Worker
      |
      v
Elasticsearch
```

This architecture decouples product creation from search indexing.

## 5.4 Why not use Elasticsearch as the primary database?

Elasticsearch is optimized for search and analytics, not as a general-purpose transactional database.

Important differences:

| Primary Database | Elasticsearch |
| --- | --- |
| Source of truth | Search-oriented data store |
| Transactional workloads | Search workloads |
| Relational constraints may be available | No equivalent relational constraint model |
| Strong transactional guarantees depend on database | Near-real-time search visibility |
| Good for business data | Good for search and retrieval |

Recommended architecture:

- Store original business data in the primary database.
- Maintain a searchable representation in Elasticsearch.
- Treat Elasticsearch as a derived search index.

# 6\. Search Architecture – End-to-End

Consider an e-commerce application.

The user searches for:

`Samsung mobile under 80000`

Architecture:

text

```text
          Android Application
                  |
                  v
              Search API
                  |
                  v
          Search Service
                  |
                  v
           Elasticsearch
                  |
                  v
          Search Results
                  |
                  v
          Android Application
```

The Search Service handles:

- Query validation.
- Search filters.
- Sorting.
- Pagination.
- Search engine communication.
- Response formatting.

The search engine handles:

- Keyword matching.
- Relevance scoring.
- Filtering.
- Sorting.
- Aggregations.

## 6.1 Search API example

http

```http
GET /api/v1/products/search?q=samsung&minPrice=10000&maxPrice=80000&page=0&size=20
```

Example response:

json

```json
{
  "page": 0,
  "size": 20,
  "total": 120,
  "products": [
    {
      "id": 101,
      "name": "Samsung Galaxy S24",
      "price": 79999
    }
  ]
}
```

## 6.2 Search pagination

Two common approaches:

**Offset-based pagination**

http

```http
GET /products/search?page=0&size=20
```

Simple, but deep pagination can become expensive.

**Cursor-based pagination**

Uses a continuation token or search-after value to retrieve the next page.

Useful for large result sets and deep pagination.

Elasticsearch supports `search_after` for this use case.

# 7\. Search Optimization Techniques

Important techniques:

### 7.1 Caching

Cache frequently requested search queries.

Example:

text

```text
Query: samsung mobile
       |
       v
    Redis
       |
   Cache Hit?
    /     \
  Yes      No
   |        |
Return   Elasticsearch
Results      |
             v
          Store Cache
```

Cache invalidation is important when searchable data changes.

### 7.2 Autocomplete

Suggest results while the user is typing.

Example:

text

```text
User types: sam

Suggestions:
- Samsung Galaxy S24
- Samsung Galaxy S23
- Samsung TV
```

Possible implementations:

- Prefix matching.
- Edge n-grams.
- Completion suggester.
- Dedicated suggestion indexes.

### 7.3 Fuzzy Search

Fuzzy search allows approximate matching.

Example:

User searches:

`Samsng`

The engine may return:

`Samsung`

This is useful for handling typing mistakes.

### 7.4 Relevance Ranking

Search engines calculate relevance scores to determine which results are more relevant.

Factors can include:

- Term frequency.
- Field importance.
- Exact matches.
- Document popularity.
- Freshness.
- Business-specific ranking rules.

Elasticsearch commonly uses BM25 as its default similarity algorithm.

# 8\. Search System Challenges

| Problem | Possible solution |
| --- | --- |
| Slow search | Inverted indexes, query optimization |
| High traffic | Replicas, caching, horizontal scaling |
| Large datasets | Sharding |
| Stale results | Reliable indexing and event processing |
| Typing mistakes | Fuzzy search |
| Deep pagination | Cursor-based pagination |
| Search cluster failure | Replicas and cluster recovery |
| Duplicate indexing events | Idempotent indexing operations |

# PART 2 – NOTIFICATION SYSTEMS

# 9\. What is a Notification System?

A notification system delivers information to users when important events occur.

Examples:

- Order placed.
- Payment successful.
- New chat message.
- Friend request received.
- Delivery partner assigned.
- Password reset requested.

Notifications can be delivered through different channels:

1. Push notifications.
2. Email.
3. SMS.
4. In-app notifications.

A notification system should be reliable, scalable and fault tolerant.

# 10\. Types of Notifications

## 10.1 Push Notifications

Notifications delivered to a device through a push notification service.

Examples:

- WhatsApp message alert.
- E-commerce order update.
- Banking transaction alert.

On Android, Firebase Cloud Messaging (FCM) is a commonly used push messaging service.

## 10.2 In-App Notifications

Notifications displayed inside the application.

Examples:

- Notification inbox.
- Activity feed.
- Order status updates.

These are generally stored in the backend database and retrieved through APIs.

## 10.3 Email Notifications

Used for:

- Registration confirmation.
- Password reset.
- Invoices.
- Account updates.

## 10.4 SMS Notifications

Used for:

- OTP verification.
- Banking alerts.
- Delivery updates.

# 11\. What is Firebase Cloud Messaging (FCM)?

Firebase Cloud Messaging is a cross-platform messaging service that allows a server to send messages to client applications.

For Android applications, FCM helps deliver push notifications.

Important components:

**1\. Android Application**

Receives messages from FCM.

**2\. FCM SDK**

Handles communication between the Android application and Firebase messaging infrastructure.

**3\. FCM Backend**

Accepts message requests from the application server and routes messages to the appropriate devices.

**4\. Application Backend**

Decides when and why a notification should be sent.

### Basic FCM architecture

text

```text
       Application Backend
                |
                v
          FCM HTTP v1 API
                |
                v
        Firebase Infrastructure
                |
                v
          Android Device
                |
                v
       Android Application
```

Important: The backend should send FCM requests using authorized server-side credentials. Never place server credentials in an Android application.

# 12\. How does Push Notification Delivery Work?

Let's understand this with an example.

A user places an order.

The application should send an order confirmation notification.

### Step 1: Device Registration

The Android application integrates the Firebase Messaging SDK.

FCM provides a registration token for the application instance.

Example:

text

```text
fcm_token_abc123
```

This token identifies an app instance for message delivery.

### Step 2: Token Registration with Backend

The Android application sends the token to the backend.

Example:

http

```http
POST /api/v1/devices/register
```

Request:

json

```json
{
  "userId": 101,
  "deviceToken": "fcm_token_abc123",
  "platform": "ANDROID"
}
```

The backend stores the token and associates it with the relevant user and application installation.

### Step 3: Business Event Occurs

The user places an order.

text

```text
Order Service
      |
      v
Order Created
      |
      v
Order Event
```

### Step 4: Notification Service

The notification service consumes the order event.

It determines:

- Who should receive the notification?
- Which notification template should be used?
- Which devices belong to the user?
- Which delivery channel should be selected?

### Step 5: Send Through FCM

The notification service sends a message to FCM using the registered device token.

### Step 6: Device Receives Notification

FCM delivers the message to the Android application instance when delivery is possible.

The application handles the message according to its payload type and lifecycle state.

# 13\. Notification System – Scalable Architecture

A production notification system should not tightly couple business operations with notification delivery.

Consider this architecture:

text

```text
           Order Service
                |
                v
           Order Database
                |
                v
           Event Publisher
                |
                v
           Message Queue
                |
                v
        Notification Service
                |
         +------+------+
         |      |      |
         v      v      v
        Push   Email   SMS
         |      |      |
         v      v      v
        FCM   Provider Provider
         |
         v
    Android Device
```

### Why use a message queue?

Imagine an e-commerce application processing 50,000 orders per minute.

If every order request directly calls FCM, email and SMS providers, the order API may become slow.

Instead:

1. Process the order.
2. Publish an event.
3. Return the order response.
4. Process notification delivery asynchronously.

Benefits:

- Decouples business services.
- Handles traffic spikes.
- Supports retries.
- Improves fault isolation.
- Enables horizontal scaling.

# 14\. Notification Data Model

A notification system may maintain several database tables.

## 14.1 Device Tokens

| Field | Description |
| --- | --- |
| id | Unique device record |
| user\_id | User identifier |
| device\_token | FCM registration token |
| platform | Android or iOS |
| created\_at | Registration time |
| updated\_at | Last token update |

A user can have multiple registered devices.

## 14.2 Notifications

| Field | Description |
| --- | --- |
| id | Notification identifier |
| user\_id | Recipient |
| title | Notification title |
| body | Notification content |
| type | ORDER, CHAT, PAYMENT |
| data | Additional payload |
| created\_at | Creation time |
| read\_at | Read timestamp |

## 14.3 Notification Delivery Attempts

| Field | Description |
| --- | --- |
| id | Attempt identifier |
| notification\_id | Related notification |
| device\_token\_id | Target device |
| status | Pending, Sent, Failed |
| attempt\_count | Number of attempts |
| next\_retry\_at | Scheduled retry |
| error\_code | Failure reason |

Separating notification records from delivery attempts helps support multiple devices and delivery tracking.

# 15\. Types of FCM Messages

FCM supports two important message payload styles.

## 15.1 Notification Message

Contains a notification payload that FCM can display automatically in supported background scenarios.

Example:

json

```json
{
  "message": {
    "token": "DEVICE_TOKEN",
    "notification": {
      "title": "Order Confirmed",
      "body": "Your order has been confirmed."
    }
  }
}
```

## 15.2 Data Message

Contains custom key-value data for application handling.

Example:

json

```json
{
  "message": {
    "token": "DEVICE_TOKEN",
    "data": {
      "type": "ORDER_CONFIRMED",
      "orderId": "ORD123",
      "screen": "ORDER_DETAILS"
    }
  }
}
```

## 15.3 Notification + Data Message

Both notification and data payloads can be included.

Important Android behavior:

| Application state | Notification payload behavior |
| --- | --- |
| Foreground | Message is delivered to the application callback |
| Background | Notification payload can be displayed by the system |
| Background with data payload | Application receives the data message through its messaging callback when delivered |

Actual delivery and handling depend on Android lifecycle, message priority, device state and OS restrictions.

**Interview point:** Never assume that every FCM message will immediately execute application code in the background.

# 16\. FCM Integration in Android

Important Android components:

- Firebase Messaging SDK.
- FirebaseMessagingService.
- onNewToken().
- onMessageReceived().
- NotificationManager.
- NotificationChannel.
- PendingIntent.

## 16.1 Receiving FCM Token

kotlin

```kotlin
class MyFirebaseMessagingService :
    FirebaseMessagingService() {

    override fun onNewToken(token: String) {
        super.onNewToken(token)

        // Send updated token to backend
    }
}
```

The token can change, so the application should synchronize updated tokens with the backend.

## 16.2 Receiving Messages

kotlin

```kotlin
class MyFirebaseMessagingService :
    FirebaseMessagingService() {

    override fun onMessageReceived(
        message: RemoteMessage
    ) {
        super.onMessageReceived(message)

        val title = message.notification?.title
        val body = message.notification?.body

        val orderId = message.data["orderId"]

        // Handle message and display
        // a local notification when appropriate
    }
}
```

## 16.3 Important Android Notification Considerations

### Notification Permission

Android 13 (API 33) and later require the runtime `POST_NOTIFICATIONS` permission for ordinary notifications.

### Notification Channels

Android 8.0 (API 26) and later require notification channels.

Examples:

- Order Updates.
- Chat Messages.
- Promotional Notifications.

### Background Restrictions

Android restricts background execution.

Do not assume an FCM callback can perform unlimited background work.

For longer-running deferrable work, consider WorkManager.

For time-sensitive user-visible notifications, use the appropriate notification and messaging mechanisms.

# 17\. Notification Delivery Guarantees

This is a very important senior-level interview topic.

Can we guarantee that a notification will be delivered exactly once?

In a distributed system, achieving end-to-end exactly-once delivery is difficult.

A message may be:

- Published more than once.
- Delivered more than once.
- Delayed.
- Temporarily unavailable.
- Rejected due to an invalid token.
- Delivered while the application is offline.

Therefore, notification systems commonly use at-least-once processing with idempotent handling.

## 17.1 At-Most-Once Delivery

A message is processed zero or one time.

Problem:

If processing fails, the notification may be lost.

## 17.2 At-Least-Once Delivery

The system retries until successful processing or until a defined retry limit or terminal condition is reached.

Problem:

Duplicate processing can occur.

Solution:

Use idempotency.

## 17.3 Idempotency

An operation is idempotent if repeating it does not produce additional unintended effects.

Example:

text

```text
notification_id = NOTIF123
```

Before processing the event, the system checks whether the notification has already been processed.

A unique event ID or notification ID can help prevent duplicate business processing.

For client-side display, the application can also use a stable notification ID to update or replace an existing notification where appropriate.

# 18\. Notification Retry Mechanism

Suppose FCM temporarily fails to accept a message because of a transient service or network error.

The notification service should retry.

A common approach is exponential backoff.

Example:

| Attempt | Retry delay |
| --- | --- |
| 1 | 1 second |
| 2 | 2 seconds |
| 3 | 4 seconds |
| 4 | 8 seconds |
| 5 | 16 seconds |

Actual retry schedules should include jitter and respect provider retry guidance.

Formula:

text

```text
delay = min(
    maxDelay,
    initialDelay * 2^attempt
)
```

With jitter, the system adds a random variation to avoid many workers retrying simultaneously.

### Permanent vs Temporary Failures

**Temporary failures:**

- Network timeout.
- Temporary service unavailability.
- Rate limiting.

Action: Retry according to the error and provider guidance.

**Permanent failures:**

- Invalid registration token.
- Unregistered application instance.
- Invalid message payload.

Action: Do not repeatedly retry. Correct the payload or remove the invalid token where appropriate.

# 19\. Dead Letter Queue (DLQ)

What happens if a notification keeps failing?

We should not retry it forever.

After the configured retry policy is exhausted, move the message to a Dead Letter Queue.

text

```text
       Notification Queue
                |
                v
        Notification Worker
                |
                v
          Delivery Attempt
                |
          +-----+-----+
          |           |
        Success      Failure
          |           |
          v           v
        Done       Retry Queue
                       |
                       v
                 Retry Limit?
                    /    \
                  No      Yes
                  |        |
                  v        v
                Retry     DLQ
```

DLQ messages can be inspected and replayed after the underlying issue is resolved.

# 20\. Notification Reliability and Scalability

Important techniques:

| Challenge | Solution |
| --- | --- |
| High notification traffic | Message queues and horizontal workers |
| Duplicate notifications | Idempotency and deduplication |
| FCM temporary failure | Retry with exponential backoff |
| Invalid device token | Remove or deactivate token |
| Queue backlog | Scale workers and monitor queue lag |
| Provider outage | Circuit breaker and controlled retry |
| Multiple devices | Maintain device-token mapping |
| Notification loss | Transactional outbox and durable queues |
| Duplicate events | Idempotent consumers |
| User preference | Notification preferences and channel rules |

## 20.1 Transactional Outbox Pattern

Consider this problem:

1. Order is saved in the database.
2. Application crashes before publishing the order-created event.
3. Notification service never receives the event.

This creates inconsistency between the database and event system.

The transactional outbox pattern solves this problem.

Within one database transaction:

- Save the order.
- Save an event in the outbox table.

A background publisher reads outbox records and sends them to the message broker.

text

```text
        Order Service
              |
              v
       Database Transaction
          /          \
         v            v
    Order Table   Outbox Table
                      |
                      v
               Outbox Publisher
                      |
                      v
                 Message Queue
                      |
                      v
              Notification Service
```

The publisher may publish an event more than once, so consumers must still be idempotent.

# 21\. Notification Preferences and User Experience

A production application should allow users to control notifications.

Example preferences:

json

```json
{
  "userId": 101,
  "preferences": {
    "orderUpdates": true,
    "promotions": false,
    "chatMessages": true,
    "emailNotifications": true
  }
}
```

The notification service should check user preferences before dispatching optional notifications.

Important considerations:

- Respect opt-outs.
- Separate transactional and promotional messages.
- Avoid sending sensitive information in notification previews.
- Provide deep links to the correct application screen.
- Allow users to open the relevant content securely.

# PART 3 – ANDROID AND SYSTEM DESIGN CONNECTION

# 22\. How Android Applications Use Search Systems

Consider a shopping application.

Android responsibilities:

- Display a search field.
- Debounce user input.
- Cancel obsolete requests.
- Display loading states.
- Handle empty results.
- Support pagination.
- Cache appropriate results.
- Handle network failures.

Example search flow:

text

```text
        Search Screen
              |
              v
          User Input
              |
              v
          Debounce
              |
              v
        ViewModel
              |
              v
        Repository
              |
              v
         Search API
              |
              v
      Search Backend
              |
              v
       Elasticsearch
              |
              v
       Search Results
              |
              v
          UI Update
```

### Why debounce?

Suppose the user types:

text

```text
S
Sa
Sam
Sams
Samsu
Samsung
```

Without debouncing, the application may send six network requests.

With debounce, the application waits for a short period after typing stops before making the request.

Example using Kotlin Flow:

kotlin

```kotlin
searchQuery
    .debounce(300)
    .distinctUntilChanged()
    .flatMapLatest { query ->
        repository.searchProducts(query)
    }
```

`flatMapLatest` helps cancel the previous search flow when a newer query arrives.

# 23\. Android Notification Architecture

A clean Android notification architecture could look like this:

text

```text
        FCM Service
             |
             v
       Message Handler
             |
             v
      Notification Mapper
             |
             v
    Notification Repository
             |
        +----+----+
        |         |
        v         v
    Local DB   Notification UI
        |
        v
    Notification History
```

Responsibilities:

**FCM Service**

- Receives FCM messages.
- Handles token changes.
- Extracts message data.

**Message Handler**

- Validates message type.
- Routes the event.

**Notification Mapper**

- Converts backend payloads into application models.

**Notification Repository**

- Coordinates local persistence and notification-related data.

**Notification UI**

- Displays notification history and unread states.

Important: A push notification is not the source of truth for important business information.

For example, when a user opens an order notification, the application should fetch the latest order details from the backend.

# PART 4 – INTERVIEW PREPARATION

# 24\. Important Interview Questions and Answers

### Q1. What is the difference between database search and Elasticsearch?

A database can support filtering and full-text search, but Elasticsearch is optimized for advanced search workloads using inverted indexes, relevance scoring, distributed indexing and search capabilities.

### Q2. What is an inverted index?

An inverted index maps each searchable term to the documents containing that term.

It avoids scanning every document for each search query.

### Q3. Why do we use Elasticsearch instead of directly querying SQL?

When advanced text search, relevance ranking, autocomplete, fuzzy matching and large-scale search workloads are required, a dedicated search engine can provide better search capabilities.

### Q4. Is Elasticsearch the source of truth?

Generally, no. The primary database remains the source of truth, while Elasticsearch stores a searchable representation.

### Q5. What is eventual consistency in search?

When a record is updated in the primary database, the corresponding search index may take some time to reflect that update.

During that interval, search results may be stale.

### Q6. What is FCM?

Firebase Cloud Messaging is a messaging service that enables backend systems to send messages to client applications, including Android applications.

### Q7. How does FCM identify a device?

FCM provides a registration token for an application instance. The application sends that token to its backend for registration and notification targeting.

### Q8. What happens if an FCM token changes?

The application receives the updated token through `onNewToken()` and should synchronize it with the backend.

### Q9. Can we guarantee exactly-once notification delivery?

End-to-end exactly-once delivery cannot generally be guaranteed across all distributed components. We typically use durable processing, at-least-once delivery, idempotency and deduplication.

### Q10. Why do we need a message queue in a notification system?

It decouples business operations from notification delivery, absorbs traffic spikes, supports retries and improves fault isolation.

### Q11. What is a Dead Letter Queue?

A DLQ stores messages that cannot be successfully processed after the configured retry policy. These messages can be investigated and replayed.

### Q12. What is the difference between notification and data messages in FCM?

Notification messages contain display-oriented notification content. Data messages contain custom key-value data that the application can process. Their handling differs depending on application state and Android behavior.

### Q13. How do you prevent duplicate notifications?

Use unique event identifiers, idempotent consumers, deduplication and stable notification identifiers where appropriate.

### Q14. What is the transactional outbox pattern?

It stores business data and the corresponding event in the same database transaction. A separate publisher later sends the event to the message broker.

### Q15. How would you design a scalable notification system for 10 million users?

A possible architecture:

1. Business services generate events.
2. Events are persisted through an outbox.
3. Events are published to a durable message queue.
4. Notification workers consume events.
5. User preferences and device tokens are retrieved.
6. Notifications are sent through FCM, email or SMS providers.
7. Temporary failures are retried.
8. Permanently failed messages are moved to a DLQ.
9. Metrics and logs monitor delivery performance.

# PART 5 – PRACTICAL SYSTEM DESIGN EXERCISES

## Exercise 1: Design an E-commerce Search System

Requirements:

- Search 10 million products.
- Support keyword search.
- Support category and price filters.
- Support autocomplete.
- Support sorting by price and popularity.
- Handle high search traffic.

Things to design:

1. Search API.
2. Elasticsearch index structure.
3. Inverted index.
4. Product indexing pipeline.
5. Cache strategy.
6. Pagination.
7. Handling stale search results.

## Exercise 2: Design a Notification System

Requirements:

- Support 10 million users.
- Each user may have multiple devices.
- Support push, email and SMS.
- Support user notification preferences.
- Handle provider failures.
- Prevent duplicate processing.
- Support notification history.

Things to design:

1. Database schema.
2. Device registration.
3. Event-driven architecture.
4. Message queue.
5. Notification workers.
6. Retry strategy.
7. DLQ.
8. Monitoring and observability.

## Exercise 3: Android Search Feature

Build a search feature using:

- MVVM.
- Kotlin Flow.
- Coroutines.
- Retrofit.
- Repository pattern.
- Debounce.
- `flatMapLatest`.
- Paging 3.

Think about:

- What happens when the user types quickly?
- How do you cancel old requests?
- How do you handle empty results?
- How do you support offline search history?
- How do you handle pagination failures?

# PART 6 – IMPORTANT TAKEAWAYS

## Search Systems

- Full-text search allows searching within text.
- Tokenization breaks text into searchable terms.
- An inverted index maps terms to documents.
- Elasticsearch is built on Apache Lucene.
- Indexes contain documents.
- Shards distribute data.
- Replicas improve availability and can distribute search load.
- Elasticsearch is generally a derived search index, not the primary source of truth.
- Search indexing is often asynchronous.
- Caching, autocomplete and fuzzy search improve user experience.

## Notification Systems

- FCM enables push messaging.
- Backend services decide when to send notifications.
- Device tokens identify application instances.
- Message queues decouple business services from delivery.
- At-least-once processing requires idempotency.
- Exponential backoff handles temporary failures.
- DLQs handle messages that exceed retry policies.
- Transactional outbox helps prevent lost events.
- Android notification permissions and channels must be handled correctly.
- Important business data should be fetched from the backend rather than trusted solely from push payloads.

# PART 7 – REVISION CHECKLIST

## Search Systems

- [ ]Understand full-text search.
- [ ]Understand tokenization and normalization.
- [ ]Understand inverted indexes.
- [ ]Understand Elasticsearch.
- [ ]Understand cluster, node, index, document, shard and replica.
- [ ]Understand indexing pipelines.
- [ ]Understand search pagination.
- [ ]Understand autocomplete and fuzzy search.
- [ ]Understand eventual consistency.
- [ ]Understand search optimization.

## Notification Systems

- [ ]Understand push, in-app, email and SMS notifications.
- [ ]Understand FCM architecture.
- [ ]Understand FCM registration tokens.
- [ ]Understand notification and data messages.
- [ ]Understand Android notification channels.
- [ ]Understand Android 13 notification permission.
- [ ]Understand asynchronous notification architecture.
- [ ]Understand message queues.
- [ ]Understand at-least-once delivery.
- [ ]Understand idempotency.
- [ ]Understand exponential backoff.
- [ ]Understand Dead Letter Queues.
- [ ]Understand transactional outbox.
- [ ]Understand notification preferences.

## Senior Android Interview Readiness

- [ ]Explain how an Android search feature communicates with Elasticsearch through a backend.
- [ ]Explain debounce and `flatMapLatest`.
- [ ]Explain how FCM token registration works.
- [ ]Explain foreground and background notification handling.
- [ ]Explain notification deep linking.
- [ ]Explain how to handle duplicate notifications.
- [ ]Explain how to design a reliable offline-friendly notification experience.

---

**End of Day 25 – Search and Notification Systems**

**Next lesson: Week 4 – Day 26: Real-Time Systems**

Topics: Polling, Long Polling, Server-Sent Events, WebSockets, Chat Architecture and Online/Offline Status.

## How to study Day 25

Don't try to memorize everything in one sitting. I recommend splitting this lesson into three sessions:

| Session | Focus |
| --- | --- |
| Session 1 | Full-text search, inverted index, Elasticsearch |
| Session 2 | Notification architecture, FCM, Android integration |
| Session 3 | Reliability, retries, DLQ, transactional outbox and interview exercises |

My recommendation: Pay special attention to the difference between Elasticsearch indexing and database storage, and between FCM message acceptance and actual device delivery. These distinctions help demonstrate senior-level system design understanding.

Once you finish these notes, we can do a mock interview specifically on Day 25 before moving to Day 26.
