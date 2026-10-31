# Week 4 – Day 26: Real-Time Systems

Day 26 of 49 Advanced System Design

File name: `26-Real-Time-Systems.md`

Goal: Understand how real-time applications communicate, deliver updates instantly, manage persistent connections, and handle millions of concurrent users.

This lesson is especially important for Senior Android Developer interviews because real-time communication is used in chat applications, food delivery, ride tracking, stock market apps, gaming, and collaborative applications.

# 1. What Are Real-Time Systems?

A real-time system is a system that delivers information or processes events within a sufficiently short and predictable time.

In traditional applications, the client requests information from the server whenever it needs something.

In real-time applications, the client receives updates as soon as new information becomes available, without continuously requesting it.

### Example: Traditional application

Imagine you open a food delivery application.

1. You request the current order status.
2. The server returns `Order Confirmed`.
3. The restaurant starts preparing your food.
4. You manually refresh the screen.
5. The server returns `Food Preparing`.

The client has to request updates repeatedly.

### Example: Real-time application

Now imagine the same application with real-time communication.

1. You place an order.
2. The server confirms the order.
3. The restaurant starts preparing food.
4. The server automatically sends an update.
5. Your application immediately displays `Food Preparing`.

You don't need to refresh the screen.

Live Order Tracking for Your Food Delivery | Billing Fast

[billingfast.com](<https://www.billingfast.com/blog/how-to-make-your-own-zomato-app?utm_source=chatgpt.com>)

Key difference:

- Traditional communication: Client asks, server responds.
- Real-time communication: Server can proactively deliver updates to the client.

# 2. Why Do We Need Real-Time Systems?

Real-time communication is useful when information changes frequently and users expect immediate updates.

Common examples:

| Application | Real-time functionality |
| --- | --- |
| WhatsApp | Instant message delivery |
| Instagram | Live notifications |
| Uber | Driver location updates |
| Swiggy | Order status tracking |
| Stock trading | Live stock prices |
| Google Docs | Collaborative editing |
| Multiplayer games | Player movement |
| Banking | Transaction alerts |

### Important requirements

A real-time system should consider:

1. Low latency – Deliver updates quickly.
2. Reliability – Avoid losing important messages.
3. Scalability – Support many concurrent connections.
4. Ordering – Maintain event order when required.
5. Availability – Continue working despite failures.
6. Reconnection – Recover when the network disconnects.
7. Resource efficiency – Minimize CPU, battery, and network usage.

# 3. Different Real-Time Communication Techniques

There are four important communication techniques:

1. Polling
2. Long Polling
3. Server-Sent Events (SSE)
4. WebSockets

Let's understand each from the basics.

## 3.1 Polling

Polling is a communication technique in which the client periodically sends requests to the server to check whether new information is available.

### How it works

Suppose an Android application needs to check order status every five seconds.

Stop Using WebSockets for Everything (And Other Real-Time Mistakes You’re Probably Making) | by Pradeep Mishra | Jan, 2026 | Medium

[medium.com](<https://medium.com/%40ppp.mishra124/stop-using-websockets-for-everything-and-other-real-time-mistakes-youre-probably-making-2290394badde?utm_source=chatgpt.com>)

Flow:

1. Android sends an HTTP request.
2. Server checks the latest order status.
3. Server returns the response.
4. Android waits five seconds.
5. Android sends another request.
6. The process repeats.

### Example

http

```http
GET /api/orders/123/status
```

Response:

json

```json
{
  "orderId": "123",
  "status": "PREPARING"
}
```

### Advantages

- Very simple to implement.
- Uses normal HTTP requests.
- Easy to debug.
- No persistent connection required.

### Disadvantages

- Generates unnecessary network requests.
- Updates may be delayed.
- Consumes battery and network resources.
- Increases server workload.

### Android implementation

kotlin

```kotlin
class OrderRepository(
    private val api: OrderApi
) {
    fun observeOrderStatus(
        orderId: String
    ): Flow<OrderStatus> = flow {
        while (currentCoroutineContext().isActive) {
            val status = api.getOrderStatus(orderId)
            emit(status)

            delay(5_000)
        }
    }
}
```

This example uses a coroutine Flow to periodically retrieve order status.

Interview question:

Why is polling not ideal for real-time chat applications?

Answer:

Polling creates repeated HTTP requests even when no new messages exist. This increases network overhead, battery consumption, and server load. WebSockets are generally more suitable for frequent, bidirectional communication.

## 3.2 Long Polling

Long polling is an improved version of traditional polling.

Instead of immediately returning an empty response, the server keeps the HTTP request open until new information becomes available or a timeout occurs.

### How it works

A Guide to Async Patterns in API Design

[blog.bytebytego.com](<https://blog.bytebytego.com/p/a-guide-to-async-patterns-in-api?utm_source=chatgpt.com>)

1. Client sends an HTTP request.
2. Server receives the request.
3. If new information is available, server responds immediately.
4. Otherwise, server waits.
5. When new information arrives, server sends the response.
6. Client sends another request.

### Example

Imagine a chat application.

- User A sends a message.
- User B has an outstanding long-polling request.
- The server receives the message.
- The server returns the message to User B.
- User B immediately starts another long-polling request.

### Advantages

- Less unnecessary traffic than frequent polling.
- Works with conventional HTTP infrastructure.
- Useful when persistent bidirectional connections are unavailable.

### Disadvantages

- Repeated request creation.
- Server must manage many waiting requests.
- Connection timeouts and proxy limitations.
- Not as efficient as WebSockets for frequent updates.

### Polling vs Long Polling

| Feature | Polling | Long Polling |
| --- | --- | --- |
| Request frequency | Fixed interval | After response |
| Server waiting | No | Yes |
| Unnecessary requests | More | Fewer |
| Latency | Depends on interval | Usually lower |
| Complexity | Low | Medium |

## 3.3 Server-Sent Events (SSE)

Server-Sent Events is a technology that allows the server to continuously send updates to the client over an HTTP connection.

It provides one-way communication:

Server → Client

The client establishes a connection, and the server sends events whenever new information is available.

API Protocols 101: A Guide to Choose the Right One

[blog.bytebytego.com](<https://blog.bytebytego.com/p/api-protocols-101-a-guide-to-choose?utm_source=chatgpt.com>)

### How it works

1. Client establishes an HTTP connection.
2. Server keeps the connection open.
3. Server sends events whenever something changes.
4. Client receives events continuously.
5. Connection can automatically reconnect after interruption when using the browser EventSource API.

### Example

HTTP request:

http

```http
GET /api/notifications/stream
Accept: text/event-stream
```

Server response:

text

```text
data: {"type":"NEW_NOTIFICATION","message":"Your order is ready"}

data: {"type":"ORDER_UPDATE","status":"DELIVERED"}
```

### Important SSE characteristics

- One-way communication.
- Uses HTTP.
- Supports event IDs.
- Supports automatic reconnection through EventSource.
- Supports retry intervals.
- Uses the `text/event-stream` content type.

### When should we use SSE?

SSE is useful when the server needs to continuously send updates, but the client does not need a full-duplex connection.

Examples:

- Live stock prices.
- Notification feeds.
- Monitoring dashboards.
- Live sports scores.
- Server-generated progress updates.

### Advantages

- Simpler than WebSockets for one-way updates.
- Uses HTTP infrastructure.
- Supports event IDs and reconnection.
- Efficient for server-to-client updates.

### Disadvantages

- No native bidirectional communication channel.
- Browser implementations have connection limitations.
- Android applications may need a compatible HTTP streaming implementation rather than browser EventSource.

Android connection: You can consume SSE using an HTTP streaming client such as OkHttp and expose incoming events through Kotlin Flow.

## 3.4 WebSockets

WebSocket is a communication protocol that provides persistent, full-duplex communication between a client and a server over a single connection.

Full-duplex means both sides can send messages independently.

This makes WebSockets particularly useful for chat applications and interactive real-time systems.

Understanding WebSocket Connection (Simple Explanation) - DEV Community

[dev.to](<https://dev.to/rajat10/understanding-websocket-connection-simple-explanation-5b15?utm_source=chatgpt.com>)

### How WebSockets work

WebSocket communication starts with an HTTP handshake.

After a successful handshake, the connection is upgraded to the WebSocket protocol.

The connection remains open until one side closes it or the connection fails.

### Step-by-step flow

1. Android application connects to the server.
2. Client sends an HTTP Upgrade request.
3. Server accepts the upgrade.
4. WebSocket connection is established.
5. Client sends messages.
6. Server sends messages.
7. Both sides continue communicating through the same connection.

### WebSocket handshake example

Request:

http

```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: random-key
Sec-WebSocket-Version: 13
```

Successful response:

http

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

After this, communication happens using WebSocket frames rather than ordinary HTTP request-response exchanges.

### WebSocket URL formats

text

```text
ws://example.com/chat
wss://example.com/chat
```

- `ws://` – Unencrypted WebSocket.
- `wss://` – WebSocket secured using TLS.

Use `wss://` in production.

### Advantages

- Bidirectional communication.
- Low-latency message delivery.
- Persistent connection.
- Efficient for frequent updates.
- Suitable for interactive applications.

### Disadvantages

- More complex connection management.
- Requires handling disconnections.
- Persistent connections consume server resources.
- Scaling requires connection-aware infrastructure.
- Mobile network changes can interrupt connections.

# 4. Polling vs Long Polling vs SSE vs WebSockets

This is one of the most important interview comparison topics.

| Feature | Polling | Long Polling | SSE | WebSocket |
| --- | --- | --- | --- | --- |
| Communication | Request-response | Request-response | One-way | Two-way |
| Protocol | HTTP | HTTP | HTTP | WebSocket |
| Persistent connection | No | Temporarily | Yes | Yes |
| Server pushes data | No, waits for request | Yes, within request | Yes | Yes |
| Client pushes data | Yes | Yes | Separate HTTP request | Yes |
| Latency | Higher | Lower | Low | Low |
| Complexity | Low | Medium | Medium | High |
| Best use | Infrequent updates | Simple notifications | Live feeds | Chat and interactive apps |

### Which one should we choose?

- Polling: When updates are infrequent and simplicity matters.
- Long Polling: When server push is needed without WebSockets.
- SSE: When communication is primarily server-to-client.
- WebSockets: When both client and server need frequent real-time communication.

Senior-level interview insight: WebSockets are not automatically the best solution for every real-time requirement. Choose based on communication direction, frequency, reliability requirements, infrastructure, and client constraints.

# 5. WebSocket Architecture for a Chat Application

Now let's design a real-world chat application.

Requirements:

- Users can send messages.
- Users receive messages instantly.
- Users can see online/offline status.
- Messages should not be lost.
- Users should receive pending messages after reconnecting.

WhatsApp System Design Interview - by Neo Kim and Hayk

[newsletter.systemdesign.one](<https://newsletter.systemdesign.one/p/whatsapp-system-design?utm_source=chatgpt.com>)

### Main components

1. Android Client

Responsible for:

- Establishing WebSocket connections.
- Sending and receiving messages.
- Displaying messages.
- Managing connection state.
- Reconnecting when disconnected.

2. WebSocket Gateway

Responsible for:

- Accepting WebSocket connections.
- Authenticating clients.
- Managing active connections.
- Routing incoming and outgoing messages.

3. Message Service

Responsible for:

- Validating messages.
- Storing messages.
- Processing message delivery.
- Managing acknowledgements.

4. Message Queue

Responsible for:

- Asynchronous message processing.
- Handling temporary service failures.
- Decoupling message producers and consumers.

5. Database

Responsible for:

- Storing messages.
- Storing conversation information.
- Maintaining delivery status.
- Supporting message history.

6. Redis

Can be used for:

- User presence.
- Active connection mapping.
- Distributed coordination.
- Temporary state.

### Example message flow

Suppose User A sends a message to User B.

1. User A sends a message through WebSocket.
2. WebSocket Gateway receives the message.
3. Message Service validates the request.
4. Message is persisted in the database.
5. System identifies User B's active connection.
6. Gateway delivers the message to User B.
7. User B acknowledges successful receipt.
8. Message status is updated.

Important: Persisting the message before acknowledging successful server acceptance prevents messages from being treated as safely accepted before durable storage.

# 6. Message Delivery Guarantees

This is a very important senior-level topic.

Real-time communication does not automatically guarantee that every message will be delivered exactly once.

There are three commonly discussed delivery semantics.

### 6.1 At-most-once delivery

A message is delivered zero or one time.

If delivery fails, the system does not retry.

Potential result: Messages may be lost.

### 6.2 At-least-once delivery

The system retries until delivery is acknowledged or retry policies stop it.

Potential result: Duplicate messages.

### 6.3 Exactly-once processing

The application aims to ensure that an operation has only one effective result, even if messages are retried.

This is commonly achieved through idempotency and deduplication rather than assuming the network can guarantee exactly-once delivery.

### Example: Duplicate message problem

User sends:

json

```json
{
  "messageId": "MSG101",
  "text": "Hello"
}
```

The server processes the message, but the acknowledgement gets lost.

The client retries.

Without deduplication, the recipient might receive two messages.

With deduplication:

kotlin

```kotlin
if (!messageRepository.exists(message.messageId)) {
    messageRepository.save(message)
}
```

The message ID helps prevent duplicate processing.

### Interview takeaway

Understand the difference between:

- Message sent.
- Message accepted by server.
- Message persisted.
- Message delivered to recipient.
- Message read by recipient.

These are different states.

# 7. Chat Message Status Architecture

A typical chat application uses the following message lifecycle:

1. SENDING
   Client has started sending the message.
2. SENT
   Server has accepted and persisted the message.
3. DELIVERED
   Recipient's device has acknowledged receipt.
4. READ
   Recipient has opened or read the message.

Possible message database structure:

sql

```sql
CREATE TABLE messages (
    message_id VARCHAR(100) PRIMARY KEY,
    conversation_id VARCHAR(100),
    sender_id VARCHAR(100),
    receiver_id VARCHAR(100),
    message_text TEXT,
    status VARCHAR(30),
    created_at TIMESTAMP
);
```

For production systems, consider separate delivery/read receipts, indexing, and efficient conversation-based pagination.

# 8. Online and Offline Presence System

How does WhatsApp know whether someone is online?

A common approach is to maintain user presence information.

Example:

json

```json
{
  "userId": "USER101",
  "status": "ONLINE",
  "lastSeen": "2026-10-31T10:30:00Z"
}
```

### Architecture

Designing a Real-Time Chat App That Actually Scales — No Bullsh\*t, Just Systems That Work | by Gyanaa Vaibhav | Medium

[medium.com](<https://medium.com/%40gynanrudr0/designing-a-real-time-chat-app-that-actually-scales-no-bullsh-t-just-systems-that-work-0f3a2f1a35e8?utm_source=chatgpt.com>)

### How it works

1. User connects to WebSocket.
2. Server authenticates the connection.
3. Server marks the user online.
4. Client periodically sends heartbeat messages.
5. Server tracks the latest heartbeat.
6. If heartbeats stop for a configured period, the connection is considered inactive.
7. User is marked offline after appropriate timeout and connection checks.

### What is a heartbeat?

A heartbeat is a small periodic message used to determine whether a connection is still alive.

Example:

json

```json
{
  "type": "PING"
}
```

Server response:

json

```json
{
  "type": "PONG"
}
```

WebSocket also has protocol-level Ping and Pong control frames.

### Why do we need heartbeat?

Because mobile networks can disconnect without giving the application a clean notification.

For example:

- User enters a tunnel.
- Internet connection disappears.
- Application does not immediately receive a disconnect event.
- Server might temporarily consider the user online.

Heartbeat helps detect stale connections.

Important: Heartbeat intervals and timeouts should be designed carefully to avoid unnecessary battery usage and false offline detection.

# 9. Real-Time Location Tracking

Let's take Uber or a food delivery application.

The rider's location changes continuously, and the customer needs to see those changes.

Kafka, WebSockets & Redis: The Trio Powering Every Food Delivery App | by The Latency Gambler | Medium

[medium.com](<https://medium.com/%40kanishks772/kafka-websockets-redis-the-trio-powering-every-food-delivery-app-532cdbd11318?utm_source=chatgpt.com>)

### Architecture

1. Rider Android app collects location updates.
2. Rider app sends location to the backend.
3. Location Service validates and processes updates.
4. Backend identifies interested customers.
5. Customer receives updates using WebSockets or another push mechanism.
6. Customer application updates the map.

Example location payload:

json

```json
{
  "riderId": "RIDER101",
  "latitude": 17.3850,
  "longitude": 78.4867,
  "timestamp": 1793440800
}
```

### Important design considerations

- Don't send location updates unnecessarily frequently.
- Filter insignificant location changes.
- Handle stale location data.
- Use timestamps to identify older updates.
- Consider battery and network consumption.
- Restrict location access to authorized users.
- Use background location services only when legally and technically appropriate.

### Android connection

Android can use:

- Fused Location Provider for location updates.
- Kotlin Coroutines for asynchronous processing.
- Flow for observing location changes.
- WebSocket or HTTP for sending location data.

# 10. WebSocket Connection Management in Android

This section is particularly important for your Android interviews.

A WebSocket connection is not guaranteed to remain active forever.

Mobile devices frequently switch networks and enter background states.

### Common disconnection scenarios

1. Internet connection lost.
2. Wi-Fi changes to mobile data.
3. Mobile data changes to Wi-Fi.
4. Application enters background.
5. Server restarts.
6. Load balancer terminates an idle connection.
7. Device enters Doze mode.

Therefore, Android applications must manage connection lifecycle properly.

### Connection states

kotlin

```kotlin
sealed interface ConnectionState {
    data object Disconnected : ConnectionState
    data object Connecting : ConnectionState
    data object Connected : ConnectionState
    data object Reconnecting : ConnectionState
    data class Failed(val reason: String) : ConnectionState
}
```

### Example using OkHttp WebSocket

Dependency:

kotlin

```kotlin
implementation("com.squareup.okhttp3:okhttp:<version>")
```

Connection implementation:

kotlin

```kotlin
class ChatWebSocketClient(
    private val client: OkHttpClient
) {
    private var webSocket: WebSocket? = null

    fun connect(url: String) {
        val request = Request.Builder()
            .url(url)
            .build()

        webSocket = client.newWebSocket(
            request,
            object : WebSocketListener() {

                override fun onOpen(
                    webSocket: WebSocket,
                    response: Response
                ) {
                    // Connection established
                }

                override fun onMessage(
                    webSocket: WebSocket,
                    text: String
                ) {
                    // Handle incoming message
                }

                override fun onFailure(
                    webSocket: WebSocket,
                    t: Throwable,
                    response: Response?
                ) {
                    // Handle failure
                }

                override fun onClosed(
                    webSocket: WebSocket,
                    code: Int,
                    reason: String
                ) {
                    // Handle disconnection
                }
            }
        )
    }

    fun sendMessage(message: String) {
        webSocket?.send(message)
    }

    fun disconnect() {
        webSocket?.close(1000, "Client disconnected")
        webSocket = null
    }
}
```

This is a simplified implementation. Production code also needs connection state management, authentication, message buffering, reconnection, lifecycle coordination, and resource cleanup.

# 11. WebSocket Reconnection Strategy

Imagine a user is chatting and suddenly loses internet connectivity.

What should happen?

The application should automatically attempt to reconnect when connectivity becomes available.

However, repeatedly reconnecting immediately can overload the server.

We use Exponential Backoff.

### What is Exponential Backoff?

The waiting time between retries increases exponentially.

Example:

| Retry | Waiting time |
| --- | --- |
| 1 | 1 second |
| 2 | 2 seconds |
| 3 | 4 seconds |
| 4 | 8 seconds |
| 5 | 16 seconds |
| 6 | 32 seconds |

A common formula:

$$
Delay = \min(MaxDelay, BaseDelay \times 2^n)
$$

Add random jitter so that thousands of clients do not reconnect simultaneously.

Example:

kotlin

```kotlin
fun calculateRetryDelay(
    attempt: Int
): Long {
    val baseDelay = 1_000L
    val maxDelay = 30_000L

    val exponential = (
        baseDelay * (1L shl attempt.coerceAtMost(10))
    ).coerceAtMost(maxDelay)

    return Random.nextLong(
        exponential / 2,
        exponential + 1
    )
}
```

### Important reconnection rules

- Reconnect after unexpected disconnections.
- Don't reconnect after intentional logout.
- Refresh expired authentication credentials.
- Use exponential backoff with jitter.
- Restore subscriptions after reconnecting.
- Fetch missed messages using the last acknowledged message ID or cursor.
- Deduplicate messages.
- Avoid reconnecting while the application is intentionally inactive.

Senior-level insight: Reconnecting the socket alone is insufficient. You must also restore the application's logical state.

# 12. Handling Background Restrictions in Android

This is one of the most important differences between backend real-time architecture and Android real-time architecture.

Android restricts background execution to preserve battery.

You cannot assume a WebSocket connection will stay alive indefinitely when your application moves into the background.

### Important Android components

| Component | Responsibility |
| --- | --- |
| Foreground Service | User-visible ongoing background work |
| WorkManager | Deferrable, guaranteed scheduled work |
| FCM | Server-to-device push notifications |
| Coroutines | Asynchronous execution |
| Flow | Reactive stream of updates |
| Room | Local message persistence |

### WebSocket vs FCM

This is a common interview question.

| Feature | WebSocket | FCM |
| --- | --- | --- |
| Communication | Bidirectional | Primarily server-to-device |
| Persistent app connection | Yes | Managed by Google Play services |
| Foreground chat | Excellent | Useful for notifications |
| Background notification | Not guaranteed | Designed for this purpose |
| Battery management | App must manage connection | Platform manages delivery infrastructure |
| Sending messages to server | Yes | Usually use HTTP or another API |

### Recommended Android architecture

For a chat application:

- Use WebSockets for active, interactive chat sessions.
- Use FCM for notifications when appropriate, especially when the app is backgrounded.
- Persist messages locally using Room.
- Synchronize missed messages when the application reconnects or resumes.
- Use WorkManager for deferrable synchronization, not continuous real-time communication.

GitHub - GetStream/AvengersChat: 💙 Android sample Avengers chat application using Stream Chat SDK based on MVVM (ViewModel, Coroutines, Room, Hilt, Repository) architecture. · GitHub

[github.com](<https://github.com/GetStream/AvengersChat?utm_source=chatgpt.com>)

# 13. Scaling WebSocket Systems

Suppose your chat application grows from 1,000 users to 10 million users.

A single WebSocket server will not be sufficient.

We need horizontal scaling.

### Horizontal scaling

Instead of increasing the capacity of one server, we add multiple servers.

Scaling Node.js to 1 Million Concurrent WebSocket Clients with Horizontal Sharding | by Hash Block | Medium

[medium.com](<https://medium.com/%40connect.hashblock/scaling-node-js-to-1-million-concurrent-websocket-clients-with-horizontal-sharding-51c20091088e?utm_source=chatgpt.com>)

Example:

- WebSocket Server 1: User A and User B.
- WebSocket Server 2: User C and User D.
- WebSocket Server 3: User E and User F.

Now imagine User A sends a message to User E.

User A is connected to Server 1, but User E is connected to Server 3.

How can Server 1 deliver the message?

### Solution: Message Broker / Pub-Sub

A shared messaging infrastructure allows servers to communicate.

Possible technologies:

- Redis Pub/Sub
- Apache Kafka
- RabbitMQ

Simplified flow:

1. User A sends a message to Server 1.
2. Server 1 sends an event to the messaging infrastructure.
3. The infrastructure routes the event to the relevant consumer/server.
4. Server 3 receives the event.
5. Server 3 delivers it to User E.

### Important distinction

- Redis Pub/Sub is useful for transient real-time event distribution but does not provide durable message storage.
- Kafka provides durable, partitioned event streaming.
- RabbitMQ provides message queuing and acknowledgement mechanisms.

Choose based on delivery guarantees, throughput, ordering, retention, and operational requirements.

# 14. Important Real-Time System Design Problems

You should be able to discuss these scenarios in senior interviews.

| Problem | Possible solution |
| --- | --- |
| User disconnects | Reconnection and synchronization |
| Duplicate messages | Message ID and deduplication |
| Message lost | Durable persistence and acknowledgements |
| Out-of-order messages | Sequence numbers and ordering strategy |
| Too many connections | Horizontal scaling |
| Server crashes | Reconnect and route to healthy server |
| User appears online incorrectly | Heartbeat and connection timeout |
| Network changes | Connectivity monitoring and reconnect |
| Battery drain | Reduce unnecessary connections and updates |
| High message traffic | Queue and asynchronous processing |

# 15. Complete Real-Time Chat Architecture

Let's combine everything we learned.

Designing a Real-time Chat App (WhatsApp, Slack) | by Arvind Kumar | Medium

[codefarm0.medium.com](<https://codefarm0.medium.com/designing-a-real-time-chat-app-whatsapp-slack-bf17912356d7?utm_source=chatgpt.com>)

### Architecture components

| Component | Responsibility |
| --- | --- |
| Android Client | User interaction and message display |
| WebSocket Gateway | Persistent client connections |
| Load Balancer | Distribute incoming connections |
| Authentication Service | Verify user identity |
| Message Service | Process and store messages |
| Kafka / Message Broker | Asynchronous event distribution |
| Redis | Presence and connection mapping |
| Database | Durable message storage |
| FCM | Background push notifications |

### End-to-end message flow

1. User logs into the application.
2. Application establishes an authenticated WebSocket connection.
3. User sends a message.
4. Gateway forwards the message to Message Service.
5. Message Service validates and persists it.
6. Message event is published for delivery.
7. Recipient's server receives the event.
8. Recipient receives the message through WebSocket if connected.
9. Recipient acknowledges delivery.
10. If recipient is offline, the system retains the message and can use FCM to notify the recipient.
11. When the recipient reconnects, the application synchronizes missed messages.

# 16. Senior Android Developer Interview Questions

Try answering these without looking at the notes.

### Basic questions

Q1. What is a real-time system?

A system that processes or delivers information within a sufficiently short and predictable time.

Q2. What is the difference between polling and long polling?

Polling sends requests at fixed intervals. Long polling keeps a request open until data becomes available or a timeout occurs.

Q3. What is SSE?

A server-to-client streaming mechanism over HTTP that allows the server to continuously send events.

Q4. What is WebSocket?

A persistent, full-duplex communication protocol that allows client and server to exchange data over one connection.

Q5. What is the difference between WebSocket and HTTP?

HTTP typically follows a request-response model. WebSocket provides persistent, bidirectional communication after an initial handshake.

### Intermediate questions

Q6. Why are WebSockets useful for chat applications?

They allow both clients and servers to exchange messages without repeatedly establishing HTTP requests.

Q7. What happens when a WebSocket connection disconnects?

The client detects the failure, updates its connection state, and attempts reconnection using an appropriate retry strategy.

Q8. How do you prevent duplicate messages?

Use unique message IDs, idempotent processing, and deduplication.

Q9. How do you implement online/offline status?

Maintain connection state using heartbeats, server-side presence tracking, and timeout-based stale connection detection.

Q10. Why is FCM needed if WebSockets already exist?

WebSockets are useful for active real-time sessions, but Android may restrict background execution. FCM provides a platform-managed mechanism for background notifications.

### Advanced questions

Q11. How would you scale a WebSocket system to millions of users?

Use horizontally scaled WebSocket gateways, load balancing, shared presence infrastructure, message brokers, durable storage, and appropriate connection distribution.

Q12. How would you guarantee messages are not lost?

Persist messages durably, acknowledge only after the required persistence step, retry delivery, and synchronize missed messages after reconnecting.

Q13. How would you handle messages arriving out of order?

Use sequence numbers or server-assigned ordering information and reconcile messages based on the chosen ordering rules.

Q14. What is exponential backoff?

A retry strategy where the delay between attempts increases exponentially, generally with jitter to prevent synchronized retries.

Q15. Should Android keep a WebSocket connection alive forever?

No. Connection lifetime should respect application lifecycle, battery constraints, network changes, and Android background execution restrictions.

# 17. Practical Exercises

These exercises will help you apply the concepts rather than simply memorize them.

## Exercise 1: Choose the communication technique

Identify the most appropriate communication technique.

| Scenario | Your answer |
| --- | --- |
| Chat application | ? |
| Live stock price dashboard | ? |
| Checking order status every 30 seconds | ? |
| Collaborative document editing | ? |
| Server-generated progress updates | ? |

Suggested answers:

- Chat: WebSocket.
- Stock prices: SSE or WebSocket, depending on interaction requirements.
- Infrequent order checks: Polling.
- Collaborative editing: WebSocket.
- Server-generated progress: SSE.

## Exercise 2: Design a chat application

Design an Android chat application supporting:

- One-to-one messaging.
- Message delivery receipts.
- Online/offline status.
- Offline message synchronization.
- Push notifications.
- Automatic reconnection.

Draw the architecture and explain each component.

## Exercise 3: Android implementation

Create a small Android application using:

- Kotlin.
- MVVM.
- OkHttp WebSocket.
- Kotlin Flow.
- Room.

Implement:

1. WebSocket connection.
2. Send message.
3. Receive message.
4. Save message in Room.
5. Display messages.
6. Handle disconnection.
7. Implement reconnection.

# 18. Day 26 Revision Cheat Sheet

## Real-Time Systems – Quick Revision

| Concept | Remember |
| --- | --- |
| Polling | Client repeatedly requests |
| Long Polling | Server waits before responding |
| SSE | One-way server-to-client streaming |
| WebSocket | Full-duplex persistent communication |
| Heartbeat | Detect connection health |
| Presence | Track online/offline users |
| Acknowledgement | Confirm message processing/delivery |
| Deduplication | Prevent duplicate processing |
| Exponential Backoff | Increase retry delay |
| Message Broker | Distribute events between servers |
| FCM | Background push notifications |
| Room | Local persistence |
| Flow | Reactive stream of updates |
| WorkManager | Deferrable background work |

Must-remember interview concepts:

- WebSocket vs SSE.
- Polling vs Long Polling.
- Message delivery guarantees.
- WebSocket reconnection.
- Heartbeat and presence.
- Scaling WebSocket connections.
- WebSocket vs FCM in Android.
- Offline message synchronization.

# 19. Learning Checklist

Day 26 Progress

# 0/16

[ ]Understand real-time systems and use cases

[ ]Understand polling and long polling

[ ]Understand Server-Sent Events

[ ]Understand WebSocket handshake and communication

[ ]Compare all four communication techniques

[ ]Understand chat architecture

[ ]Understand message delivery guarantees

[ ]Understand message acknowledgement and deduplication

[ ]Understand online/offline presence

[ ]Understand heartbeat mechanisms

[ ]Understand real-time location tracking

[ ]Implement WebSocket using OkHttp

[ ]Understand exponential backoff and jitter

[ ]Understand Android background restrictions

[ ]Understand WebSocket scaling and message brokers

[ ]Practice all 15 interview questions

 Reset checklist

## 20. What You Must Be Able to Explain in an Interview

By the end of Day 26, you should be able to independently answer this question:

"Design a real-time chat system for an Android application that supports millions of users."

Your explanation should cover:

1. Functional and non-functional requirements.
2. High-level architecture.
3. WebSocket communication.
4. Message persistence.
5. Message delivery guarantees.
6. Online/offline presence.
7. Reconnection and synchronization.
8. Android background restrictions.
9. FCM integration.
10. Horizontal scaling and message brokers.

My recommendation: Pay special attention to Sections 10–14. These are where you move beyond understanding WebSockets as an API and start thinking like a Senior Android Developer designing production-grade real-time applications.

Next lesson: Week 4 – Day 27: Security and Observability (HTTPS, TLS, Encryption, Authentication, Authorization, Logging, Metrics, Distributed Tracing and Alerting).
