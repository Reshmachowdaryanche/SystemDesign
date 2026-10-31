# Week 5 – Day 33: Android Real-Time Architecture

Day 33Intermediate → AdvancedSenior Android Interview Preparation

File name: `33-Android-Real-Time-Architecture.md`

Goal: Understand how to build Android applications that receive and send data in real time, maintain persistent connections, handle network interruptions, and deliver notifications reliably.

By the end of this lesson, you should be able to design real-time features for applications like WhatsApp, Instagram, Uber, and food delivery applications.

## 1. What is Real-Time Architecture?

Real-time architecture allows an application to exchange information with a server continuously or with minimal delay.

Consider a chat application.

When User A sends a message to User B:

1. User A sends a message.
2. The backend receives the message.
3. The backend stores it.
4. The backend delivers it to User B.
5. User B receives the message without manually refreshing the screen.

This is real-time communication.

WhatsApp System Design: Building a Scalable Messaging App like WeChat, Telegram, and Messenger

[systemdesignschool.io](<https://systemdesignschool.io/problems/chatapp/solution?utm_source=chatgpt.com>)

### Real-world examples

| Application | Real-time feature |
| --- | --- |
| WhatsApp | Chat messages and typing status |
| Instagram | Direct messages and notifications |
| Uber | Driver location tracking |
| Swiggy | Live order tracking |
| Google Meet | Video and audio communication |
| Stock trading apps | Live price updates |

### Why do we need real-time architecture?

Traditional API communication often works like this:

text

```text
Android App
    |
    | Request
    v
Backend
    |
    | Response
    v
Android App
```

But what happens when the backend has new information and the Android application is not making a request?

The application will not automatically know about the update.

Real-time architectures solve this problem through technologies such as:

- Polling
- Long Polling
- Server-Sent Events (SSE)
- WebSockets
- Firebase Cloud Messaging (FCM)

# 2. Polling vs Long Polling vs SSE vs WebSockets

Very important senior interview topicThese are four important communication techniques.

## 2.1 Polling

Polling means the Android application periodically asks the backend whether new information is available.

Example:

A chat application checks for new messages every 5 seconds.

text

```text
Android App       Backend
     |                |
     |--- Request --->|
     |<-- Response ---|
     |                |
     |    Wait 5s     |
     |                |
     |--- Request --->|
     |<-- Response ---|
     |                |
```

### Android example

kotlin

```kotlin
viewModelScope.launch {
    while (isActive) {

        val messages = repository.getMessages()

        _messages.value = messages

        delay(5000)
    }
}
```

### Advantages

- Easy to implement.
- Works with ordinary HTTP APIs.
- Does not require a persistent connection.

### Disadvantages

- Unnecessary API requests.
- Increased battery consumption.
- Increased backend load.
- Updates are delayed until the next request.

When should you use polling?

For applications where updates do not need to be immediate, such as periodically refreshing dashboard statistics.

## 2.2 Long Polling

Long polling is an improved version of polling.

Instead of immediately returning an empty response, the backend keeps the HTTP request open until new data becomes available or a timeout occurs.

text

```text
Android App          Backend
     |                   |
     |---- Request ----->|
     |                   |
     |    Wait...        |
     |                   |
     |    New message    |
     |                   |
     |<--- Response -----|
     |                   |
     |---- New Request ->|
```

### Advantages

- Less frequent empty responses.
- Works over HTTP.
- Useful when WebSockets are unavailable.

### Disadvantages

- Repeated HTTP request creation.
- Connection management complexity.
- Less efficient for high-frequency bidirectional communication.

## 2.3 Server-Sent Events (SSE)

SSE allows a server to continuously send updates to a client over an HTTP connection.

Communication is primarily one-way:

Server → Android application

Example:

A stock application receives live stock price updates.

text

```text
Android App              Backend
     |                       |
     |---- Subscribe ------->|
     |                       |
     |<--- Price Update -----|
     |<--- Price Update -----|
     |<--- Price Update -----|
     |<--- Price Update -----|
```

### Advantages

- Continuous server updates.
- Uses HTTP.
- Suitable for server-to-client streaming.

### Disadvantages

- Primarily one-directional.
- Client-to-server messages require separate requests.
- Android lifecycle and reconnection still need consideration.

## 2.4 WebSockets

Most important for today's lessonWebSockets provide persistent, bidirectional communication between a client and a server.

Once a connection is established, both sides can send messages independently.

Understanding WebSocket Connection (Simple Explanation) - DEV Community

[dev.to](<https://dev.to/rajat10/understanding-websocket-connection-simple-explanation-5b15?utm_source=chatgpt.com>)

Example:

text

```text
Android App                 Backend
     |                          |
     |---- WebSocket Connect -->|
     |                          |
     |<==== Connection ========>|
     |                          |
     |---- Send Message ------->|
     |<--- Receive Message -----|
     |                          |
     |<--- Typing Event --------|
     |---- Read Receipt ------->|
     |                          |
```

### Advantages

- Bidirectional communication.
- Persistent connection.
- Low communication overhead after connection establishment.
- Suitable for frequent real-time updates.

### Disadvantages

- Connection management complexity.
- Reconnection handling.
- Battery and network considerations.
- Server-side connection scaling.

### Comparison

| Feature | Polling | Long Polling | SSE | WebSocket |
| --- | --- | --- | --- | --- |
| Communication | Request/response | Request/response | Server to client | Bidirectional |
| Persistent connection | No | Temporarily | Yes | Yes |
| Real-time suitability | Low | Medium | High | High |
| Client can send updates | Yes | Yes | Separate HTTP request | Yes |
| Complexity | Low | Medium | Medium | High |

Senior interview question: Would you always choose WebSockets for real-time applications?

No. Choose based on requirements.

- Dashboard refresh: Polling may be sufficient.
- Server event stream: SSE may be appropriate.
- Chat application: WebSockets are often suitable.
- Background notifications: FCM is often more appropriate.

# 3. WebSocket Architecture in Android

Let's understand how WebSockets work in a production Android application.

A WebSocket connection begins with an HTTP handshake.

The client sends an HTTP request asking to upgrade the connection.

Example:

http

```http
GET /chat HTTP/1.1
Host: api.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: random-key
Sec-WebSocket-Version: 13
```

If the server accepts, it responds with:

http

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
```

After the handshake, communication continues using WebSocket frames rather than ordinary independent HTTP requests.

Building Production-Ready WebSocket Servers in C# ASP.NET Core | by Bhargava Koya - Fullstack .NET Developer | Medium

[medium.com](<https://medium.com/%40bhargavkoya56/building-production-ready-websocket-servers-in-c-asp-net-core-927b737f14cc?utm_source=chatgpt.com>)

### WebSocket URL formats

| Protocol | URL |
| --- | --- |
| WebSocket | `ws://example.com/socket` |
| Secure WebSocket | `wss://example.com/socket` |

Always prefer `wss://` for production communication.

## 3.1 Android WebSocket implementation using OkHttp

You already learned OkHttp in Day 31.

OkHttp also supports WebSocket communication.

### Step 1: Create WebSocket listener

kotlin

```kotlin
class ChatWebSocketListener : WebSocketListener() {

    override fun onOpen(
        webSocket: WebSocket,
        response: Response
    ) {
        println("WebSocket connected")
    }

    override fun onMessage(
        webSocket: WebSocket,
        text: String
    ) {
        println("Received: $text")
    }

    override fun onClosing(
        webSocket: WebSocket,
        code: Int,
        reason: String
    ) {
        println("Closing: $reason")
    }

    override fun onFailure(
        webSocket: WebSocket,
        t: Throwable,
        response: Response?
    ) {
        println("WebSocket failed: ${t.message}")
    }
}
```

### Step 2: Create WebSocket connection

kotlin

```kotlin
class ChatWebSocketManager {

    private val client = OkHttpClient()

    private var webSocket: WebSocket? = null

    fun connect() {

        val request = Request.Builder()
            .url("wss://api.example.com/chat")
            .build()

        webSocket = client.newWebSocket(
            request,
            ChatWebSocketListener()
        )
    }

    fun sendMessage(message: String) {
        webSocket?.send(message)
    }

    fun disconnect() {
        webSocket?.close(
            1000,
            "User disconnected"
        )

        webSocket = null
    }
}
```

### Step 3: Send a message

kotlin

```kotlin
chatWebSocketManager.sendMessage(
    """{"type":"message","text":"Hello"}"""
)
```

### Step 4: Receive messages

The `onMessage()` callback receives incoming text or binary messages.

In a production application, parse the incoming message into a typed model and expose it through a Kotlin `Flow` or `StateFlow` to the UI.

Important: This example demonstrates the basic connection API. A production implementation also needs authentication, lifecycle ownership, reconnection, heartbeat handling, message persistence, and connection cleanup.

# 4. WebSocket Connection Management

Critical senior-level conceptOne of the biggest challenges in real-time Android applications is managing WebSocket connections.

Imagine the following situation:

1. User opens a chat screen.
2. WebSocket connects successfully.
3. User switches off Wi-Fi.
4. Connection breaks.
5. User enables mobile data.
6. Application must reconnect.
7. Messages should continue working.

A production application cannot simply call `connect()` once and assume the connection will remain active.

### Important connection states

kotlin

```kotlin
sealed interface ConnectionState {

    data object Disconnected : ConnectionState

    data object Connecting : ConnectionState

    data object Connected : ConnectionState

    data object Reconnecting : ConnectionState

    data class Failed(
        val reason: String
    ) : ConnectionState
}
```

These states help the UI understand what is happening.

Example:

| State | UI behavior |
| --- | --- |
| Connecting | Show connecting indicator |
| Connected | Allow real-time communication |
| Disconnected | Show offline status |
| Reconnecting | Show reconnecting indicator |
| Failed | Show connection error |

### Why do connections disconnect?

- Wi-Fi changes.
- Mobile network changes.
- Internet becomes unavailable.
- Server restarts.
- Server closes idle connections.
- Application process is killed.
- Device enters Doze mode.
- Network infrastructure terminates inactive connections.

### Important design principle

A WebSocket connection is not guaranteed to stay alive forever.

Your application must treat it as a recoverable resource.

# 5. Automatic Reconnection Strategy

Suppose the connection fails.

A simple solution is to reconnect immediately.

But imagine 100,000 users lose connectivity simultaneously.

All users reconnect at the same time.

This can overload your backend.

We need a better approach.

### Exponential backoff

Increase the waiting time after each failed connection attempt.

$$
Delay = \min(MaxDelay, BaseDelay \times 2^{attempt})
$$

Example:

| Attempt | Waiting time |
| --- | --- |
| 1 | 1 second |
| 2 | 2 seconds |
| 3 | 4 seconds |
| 4 | 8 seconds |
| 5 | 16 seconds |
| 6 | 30 seconds (maximum) |

Add random jitter so that clients do not reconnect simultaneously.

### Kotlin example

kotlin

```kotlin
suspend fun reconnectWithBackoff(
    connect: suspend () -> Boolean
) {
    var attempt = 0

    val maxAttempts = 10

    while (currentCoroutineContext().isActive &&
        attempt < maxAttempts
    ) {
        val connected = connect()

        if (connected) {
            return
        }

        val exponentialDelay =
            (1000L * (1L shl attempt))
                .coerceAtMost(30_000L)

        val jitter = Random.nextLong(
            0,
            1000
        )

        delay(exponentialDelay + jitter)

        attempt++
    }
}
```

This is a simplified example. In a real implementation, connection callbacks should determine success, and retry state should be coordinated with network availability and authentication.

### Important reconnection rules

1. Don't reconnect repeatedly while offline.
2. Reset the retry counter after a successful connection.
3. Use exponential backoff.
4. Add random jitter.
5. Cancel old connection attempts.
6. Avoid multiple simultaneous WebSocket connections.
7. Re-authenticate when required.
8. Resynchronize missed messages after reconnecting.

Interview question:

What happens if a WebSocket disconnects while the user is sending a message?

A good architecture should not silently lose the message.

It should:

- Persist the outgoing message locally.
- Mark it as pending.
- Retry delivery when the connection becomes available.
- Use a unique message ID.
- Deduplicate messages on the backend.
- Update the UI when delivery is acknowledged.

# 6. Heartbeat – Ping and Pong

Another important topic is heartbeat management.

Imagine a WebSocket connection that appears connected but the network has silently stopped forwarding packets.

The Android application might believe the connection is still active.

How do we detect this?

We use heartbeat mechanisms.

### How heartbeat works

text

```text
Android App                 Backend
     |                          |
     |-------- PING ----------->|
     |                          |
     |<------- PONG ------------|
     |                          |
     |-------- PING ----------->|
     |                          |
     |<------- PONG ------------|
```

The client and server periodically exchange heartbeat messages.

If expected responses stop arriving, the connection may be considered unhealthy.

### OkHttp WebSocket ping configuration

kotlin

```kotlin
val client = OkHttpClient.Builder()
    .pingInterval(
        30,
        TimeUnit.SECONDS
    )
    .build()
```

OkHttp automatically sends WebSocket ping frames at the configured interval and expects pong responses.

Important:

- Ping/pong is different from an application-level chat message.
- The server must correctly support WebSocket control frames.
- Heartbeat intervals should be designed with server infrastructure and battery requirements in mind.

### Why is heartbeat important?

- Detects broken connections.
- Helps identify inactive connections.
- Supports connection health monitoring.
- Helps prevent some idle-connection timeouts.

Senior interview question: What is the difference between heartbeat and reconnection?

- Heartbeat detects whether an existing connection is healthy.
- Reconnection establishes a new connection after the existing one becomes unusable.

# 7. Firebase Cloud Messaging (FCM)

Extremely important Android interview topicFirebase Cloud Messaging | Send notifications across platforms

Firebase Cloud Messaging

A cross-platform messaging service used to send notifications and data messages to Android applications.

Imagine WhatsApp.

The user receives a message while the application is in the background or completely closed.

Should we keep a WebSocket connection running forever?

No.

Android restricts background execution, and maintaining a permanent connection is not a reliable background delivery strategy.

This is where FCM becomes important.

### What is FCM?

Firebase Cloud Messaging allows a backend to send messages to Android devices through Google's messaging infrastructure.

### FCM architecture

Handle FCM messages on Android

[firebase.blog](<https://firebase.blog/posts/2018/09/handle-fcm-messages-on-android?utm_source=chatgpt.com>)

Flow:

text

```text
Backend Server
      |
      | Send message
      v
Firebase Cloud Messaging
      |
      | Deliver message
      v
Google Infrastructure
      |
      v
Android Device
      |
      v
FCM SDK
      |
      v
Application
      |
      v
Notification / Data Processing
```

### Step-by-step process

1. Android application integrates Firebase Messaging SDK.
2. Firebase generates an FCM registration token.
3. Android application sends the token to its backend.
4. Backend associates the token with the appropriate user/device.
5. Backend requests FCM to deliver a message.
6. FCM attempts to deliver the message to the device.
7. The Android application processes the message according to its type and device state.

### Example: Obtain FCM token

kotlin

```kotlin
FirebaseMessaging.getInstance()
    .token
    .addOnCompleteListener { task ->

        if (task.isSuccessful) {

            val token = task.result

            // Send token to backend
        }
    }
```

Important: FCM tokens can change. Implement `onNewToken()` and update your backend registration.

### Receive messages

kotlin

```kotlin
class MyFirebaseMessagingService :
    FirebaseMessagingService() {

    override fun onMessageReceived(
        message: RemoteMessage
    ) {

        val title =
            message.data["title"]

        val body =
            message.data["body"]

        // Process incoming message
    }

    override fun onNewToken(token: String) {

        // Update token registration
        // on backend
    }
}
```

### Types of FCM messages

1. Notification messages

Contain notification information such as title and body.

Depending on application state, the FCM SDK and Android system may display them automatically.

2. Data messages

Contain application-defined key-value data.

The application can process these messages through `onMessageReceived()` when delivery and execution conditions permit.

3. Notification + data messages

Contain both notification information and application data.

Their handling depends on whether the app is foregrounded or backgrounded.

### Important FCM limitation

FCM does not guarantee immediate delivery.

Delivery depends on:

- Device connectivity.
- Android power management.
- Message priority.
- Notification permissions.
- Message TTL (time to live).
- Device and application state.

Therefore, never treat push notifications as your only permanent source of truth.

Use the backend to synchronize missed information when the application becomes active.

# 8. WebSocket vs FCM

This is one of the most frequently asked Android system design questions.

| Feature | WebSocket | FCM |
| --- | --- | --- |
| Main purpose | Persistent real-time communication | Push message delivery |
| Communication | Bidirectional | Primarily server-to-device |
| Connection | Application manages connection | Google manages delivery infrastructure |
| Foreground chat | Excellent option | Useful for alerts |
| Background notification | Not reliable as a permanent strategy | Designed for this use case |
| Message acknowledgment | Application/backend protocol | Delivery behavior is managed by FCM; app-level acknowledgment may still be needed |
| Connection management | Developer responsibility | Managed by FCM infrastructure |

### Should we use WebSocket or FCM?

In many chat applications, we use both.

- WebSocket for active conversations.
- FCM for notifications when the app is backgrounded or disconnected.
- REST APIs for historical message synchronization.

This combination provides a more reliable architecture than depending on a single communication mechanism.

# 9. Android Foreground Services

Important senior interview topicWhat if your application needs to perform an ongoing, user-visible task?

Examples:

- Navigation.
- Active workout tracking.
- Audio playback.
- Ongoing location sharing.
- A user-initiated upload.

Android provides Foreground Services.

A foreground service runs work that is noticeable to the user and must display an appropriate notification.

Foreground Service | Notifee

[notifee.app](<https://notifee.app/react-native/docs/android/foreground-service/?utm_source=chatgpt.com>)

### Example

kotlin

```kotlin
class LocationService : Service() {

    override fun onStartCommand(
        intent: Intent?,
        flags: Int,
        startId: Int
    ): Int {

        startForeground(
            1001,
            createNotification()
        )

        startLocationTracking()

        return START_STICKY
    }

    override fun onBind(
        intent: Intent?
    ): IBinder? = null
}
```

This is only a simplified service example. A production implementation must declare the correct foreground service type, permissions, notification channel, and handle Android version-specific restrictions.

### Foreground Service vs WorkManager

| Foreground Service | WorkManager |
| --- | --- |
| User-visible ongoing work | Deferrable background work |
| Persistent notification required | No persistent notification by default |
| Suitable for active navigation | Suitable for periodic synchronization |
| Requires service-type compliance | Manages scheduling and constraints |
| Not a general way to bypass background limits | Respects Android scheduling policies |

### Important interview question

Should we use a Foreground Service to keep a WebSocket connected forever?

Not necessarily.

A foreground service is appropriate only when there is a legitimate, user-visible ongoing task and the applicable Android requirements are satisfied.

It should not be used simply to bypass background execution restrictions.

# 10. Android Background Execution Restrictions

Android restricts background application activity to improve battery life, privacy, and system performance.

This is extremely important when designing real-time applications.

### What happens when the user minimizes an application?

For example:

1. User opens WhatsApp.
2. WebSocket connects.
3. User presses Home.
4. Application moves to the background.
5. Android may restrict background activity.
6. Network connectivity may change or the process may eventually be killed.

Your application must not assume that its WebSocket connection will remain available.

### Important Android background concepts

| Concept | Purpose |
| --- | --- |
| Doze Mode | Reduces background activity when device is idle |
| App Standby | Restricts resources for unused applications |
| Background execution limits | Restricts background services |
| Foreground Service | Supports qualifying ongoing user-visible work |
| WorkManager | Schedules deferrable background work |
| FCM | Delivers push messages through Google's infrastructure |

### Recommended architecture

text

```text
Application Foreground
        |
        v
WebSocket Connection
        |
        v
Real-time Communication

Application Background
        |
        v
FCM Notification
        |
        v
User Notification

Application Returns to Foreground
        |
        v
Reconnect WebSocket
        |
        v
Synchronize Missed Messages
```

Important: A notification is not necessarily proof that the underlying data has been synchronized. Always reconcile with the backend.

# 11. Managing WebSocket Connection with Android Lifecycle

This is an important architectural decision.

Should WebSocket connection belong to:

- Activity?
- ViewModel?
- Repository?
- Application singleton?

The answer depends on the connection's ownership and required lifetime.

### Approach 1: Activity-owned connection

kotlin

```kotlin
override fun onStart() {
    super.onStart()
    chatManager.connect()
}

override fun onStop() {
    chatManager.disconnect()
    super.onStop()
}
```

Suitable for a connection needed only while a particular screen is visible.

Problem: If the user navigates between chat screens, frequent reconnections may occur.

### Approach 2: ViewModel-owned connection

kotlin

```kotlin
class ChatViewModel(
    private val chatRepository: ChatRepository
) : ViewModel() {

    init {
        chatRepository.connect()
    }

    override fun onCleared() {
        chatRepository.disconnect()
        super.onCleared()
    }
}
```

This can work when the connection lifetime matches the ViewModel's lifetime.

However, a ViewModel is not a permanent background connection manager.

### Approach 3: Repository / connection manager

For applications with multiple real-time features, a dedicated connection manager is often better.

text

```text
             UI
              |
              v
          ViewModel
              |
              v
          Repository
              |
              v
       Connection Manager
              |
              v
          WebSocket
              |
              v
           Backend
```

The connection manager owns:

- Connection state.
- Connecting and disconnecting.
- Reconnection strategy.
- Heartbeat.
- Authentication.
- Incoming messages.
- Outgoing messages.

The repository coordinates this with local storage and application business logic.

### Recommended design

Avoid opening a new WebSocket connection every time a Composable recomposes.

Connection ownership should be explicit, and cleanup should happen when the owner no longer needs the connection.

# 12. Message Delivery and Acknowledgment

Advanced system design conceptImagine sending a chat message.

How do we know whether it was delivered?

A successful WebSocket `send()` call does not prove that the backend stored the message or that the receiver received it.

We need an application-level delivery protocol.

### Message states

| Status | Meaning |
| --- | --- |
| Pending | Waiting to be sent |
| Sent | Backend acknowledged acceptance |
| Delivered | Recipient device acknowledged receipt |
| Read | Recipient opened/read the message |
| Failed | Delivery could not be completed |

### Message lifecycle

#systemdesign #softwareengineering #backenddevelopment #codinginterview #developersoflinkedin | Kaushal Singh

[linkedin.com](<https://www.linkedin.com/posts/kaushal-singh-2a2b8722a_systemdesign-softwareengineering-backenddevelopment-activity-7430451661250174976-DTIH?utm_source=chatgpt.com>)

Example:

text

```text
User sends message
       |
       v
Save locally
       |
       v
Pending
       |
       v
Send through WebSocket
       |
       v
Backend receives
       |
       v
Backend stores message
       |
       v
Server ACK
       |
       v
Sent
       |
       v
Recipient receives
       |
       v
Delivery ACK
       |
       v
Delivered
       |
       v
Read ACK
       |
       v
Read
```

### Example message model

kotlin

```kotlin
data class ChatMessage(
    val messageId: String,
    val conversationId: String,
    val senderId: String,
    val text: String,
    val timestamp: Long,
    val status: MessageStatus
)

enum class MessageStatus {
    PENDING,
    SENT,
    DELIVERED,
    READ,
    FAILED
}
```

### Why do we need a unique message ID?

Imagine:

1. User sends a message.
2. Backend stores it.
3. Server acknowledgment gets lost.
4. Android assumes failure.
5. Android retries the message.
6. Backend receives the same message again.

Without deduplication, the recipient might see duplicate messages.

A unique message ID allows the backend to identify repeated submissions.

This is called idempotent message processing.

Senior interview point: WebSocket transport alone does not provide end-to-end exactly-once message delivery. The application must implement acknowledgments, durable storage, retries, and deduplication according to its requirements.

# 13. Online and Offline User Status

Consider WhatsApp's online status.

How does the backend know whether a user is online?

One common approach is connection tracking.

text

```text
User opens application
        |
        v
WebSocket connects
        |
        v
Backend marks user online
        |
        v
Heartbeat updates
        |
        v
Connection lost
        |
        v
Backend waits for timeout
        |
        v
User marked offline
```

### Why wait before marking offline?

A user may temporarily lose connectivity.

If the backend immediately marks the user offline, the status can fluctuate unnecessarily.

Instead, the backend can use a heartbeat timeout and a grace period.

### Important considerations

- A user can have multiple devices.
- A user can have multiple active sessions.
- A single device may have multiple connections.
- A network interruption does not necessarily mean the user closed the application.
- Online status is an approximation of connectivity, not proof that the user is actively viewing the app.

### Example backend data model

text

```text
User
 |
 +-- User ID
 |
 +-- Device 1
 |      |
 |      +-- Connection ID
 |      +-- Last heartbeat
 |
 +-- Device 2
        |
        +-- Connection ID
        +-- Last heartbeat
```

# 14. Real-Time Architecture: Complete Android Design

Let's combine everything into one production-style architecture.

GitHub - GetStream/AvengersChat: 💙 Android sample Avengers chat application using Stream Chat SDK based on MVVM (ViewModel, Coroutines, Room, Hilt, Repository) architecture. · GitHub

[github.com](<https://github.com/GetStream/AvengersChat?utm_source=chatgpt.com>)

### Recommended architecture

text

```text
                 Android Application
                          |
                          v
                     UI Layer
                          |
                          v
                      ViewModel
                          |
                          v
                      Repository
                     /     |      \
                    /      |       \
                   v       v        v
                Room   REST API  WebSocket
                   |       |        |
                   |       |        |
                   v       v        v
                Local    Retrofit  Connection
                Storage  + OkHttp  Manager
                            |         |
                            +----+----+
                                 |
                                 v
                           Backend Server
                                 |
                  +--------------+--------------+
                  |              |              |
                  v              v              v
              Database      Message Queue   WebSocket
                                 |
                                 v
                         Firebase Cloud Messaging
                                 |
                                 v
                           Android Device
```

### Responsibility of each component

| Component | Responsibility |
| --- | --- |
| ViewModel | UI state |
| Repository | Coordinates local and remote data |
| Room | Stores messages locally |
| Retrofit | REST communication |
| OkHttp | HTTP and WebSocket transport |
| Connection Manager | Manages real-time connection |
| FCM | Push notification delivery |
| Backend | Business logic and message routing |
| Database | Durable message storage |
| Message Queue | Asynchronous backend processing |

### Why use both REST and WebSockets?

Because they solve different problems.

REST APIs:

- Login.
- Fetch chat history.
- Fetch user profile.
- Load conversations.
- Search messages.

WebSockets:

- Send and receive live messages.
- Typing indicators.
- Live presence updates.
- Read receipts.

FCM:

- Notify users when new messages arrive while the app is backgrounded or disconnected.

# 15. Practical Implementation: A Production-Oriented Android Chat Architecture

Let's understand how we would organize the code.

### Suggested package structure

text

```text
com.example.chat
    |
    +-- data
    |    |
    |    +-- local
    |    |    +-- ChatDao.kt
    |    |    +-- ChatDatabase.kt
    |    |
    |    +-- remote
    |    |    +-- ChatApi.kt
    |    |    +-- ChatWebSocket.kt
    |    |
    |    +-- repository
    |         +-- ChatRepositoryImpl.kt
    |
    +-- domain
    |    |
    |    +-- model
    |    +-- repository
    |    +-- usecase
    |
    +-- presentation
         |
         +-- ChatViewModel.kt
         +-- ChatScreen.kt
         +-- ChatUiState.kt
```

### Define UI state

kotlin

```kotlin
data class ChatUiState(
    val messages: List<ChatMessage> = emptyList(),
    val connectionState: ConnectionState =
        ConnectionState.Disconnected,
    val isLoading: Boolean = false
)
```

### Repository interface

kotlin

```kotlin
interface ChatRepository {

    val messages: Flow<List<ChatMessage>>

    val connectionState: StateFlow<ConnectionState>

    suspend fun connect()

    suspend fun sendMessage(message: ChatMessage)

    suspend fun disconnect()
}
```

### ViewModel

kotlin

```kotlin
class ChatViewModel(
    private val repository: ChatRepository
) : ViewModel() {

    val messages = repository.messages

    val connectionState =
        repository.connectionState

    fun sendMessage(message: ChatMessage) {

        viewModelScope.launch {
            repository.sendMessage(message)
        }
    }

    fun connect() {
        viewModelScope.launch {
            repository.connect()
        }
    }
}
```

This demonstrates separation of concerns.

The ViewModel does not need to understand WebSocket implementation details.

### Important production improvements

The example above is intentionally simplified. A real application should also:

- Persist outgoing messages before transmission.
- Expose incoming messages through Room-backed Flow.
- Track message acknowledgments.
- Deduplicate incoming events.
- Handle authentication expiration.
- Serialize connection state changes.
- Recover missed messages after reconnection.
- Use lifecycle-aware connection ownership.

# 16. Senior Android Developer Interview Questions

These are important questions for someone with 7 years of Android development experience.

### Basic questions

1. What is real-time communication?
2. What is polling?
3. What is long polling?
4. What is Server-Sent Events?
5. What is WebSocket?
6. What is the difference between HTTP and WebSocket?
7. What is the difference between `ws://` and `wss://`?
8. What is Firebase Cloud Messaging?

### Intermediate questions

9. How do you implement WebSockets in Android?
10. How do you detect WebSocket disconnection?
11. How do you implement automatic reconnection?
12. What is exponential backoff?
13. What is heartbeat?
14. What is the difference between ping and pong?
15. What is the difference between FCM and WebSocket?
16. What is the difference between Foreground Service and WorkManager?
17. How do Android background restrictions affect WebSocket connections?
18. How do you handle FCM token updates?

### Advanced senior-level questions

19. How would you design a WhatsApp-like real-time chat application?
20. How would you maintain WebSocket connections when the application moves between foreground and background?
21. What happens when the network changes from Wi-Fi to mobile data during an active chat?
22. How would you prevent duplicate messages after reconnection?
23. How would you implement message delivery acknowledgments?
24. How would you handle multiple simultaneous WebSocket connections?
25. How would you maintain online/offline user status?
26. How would you synchronize messages missed while the application was offline?
27. How would you design a reliable notification architecture using FCM and WebSockets?
28. How would you manage WebSocket connections in a multi-module Android application?
29. How would you prevent excessive battery consumption in a real-time application?
30. How would you design a real-time location tracking feature for a food delivery application?

# 17. Practical System Design Exercise

Hands-on assignmentScenario: Design the Android architecture for a WhatsApp-like chat application.

Messaging App Screen by Oshin Dali on Dribbble

[dribbble.com](<https://dribbble.com/shots/24707054-Messaging-App-Screen?utm_source=chatgpt.com>)

Your application should support:

- User login.
- One-to-one messaging.
- Group messaging.
- Real-time message delivery.
- Message history.
- Online/offline status.
- Typing indicators.
- Read receipts.
- Push notifications.
- Offline message sending.
- Automatic reconnection.
- Message synchronization.

### Your design must answer these questions

1. Where will you create the WebSocket connection?
2. Which component will manage connection state?
3. How will you reconnect after network failure?
4. How will you handle background notifications?
5. How will you store messages locally?
6. How will you prevent duplicate messages?
7. How will you handle message delivery acknowledgments?
8. How will you synchronize missed messages?
9. How will you handle multiple devices?
10. How will you optimize battery consumption?

### Expected architecture

text

```text
                Chat UI
                   |
                   v
               ViewModel
                   |
                   v
                UseCase
                   |
                   v
               Repository
              /    |     \
             /     |      \
            v      v       v
          Room   REST    WebSocket
                   |       |
                   v       v
                Backend  Connection
                         Manager
                            |
                            v
                         Backend

FCM → Notification → Application → Synchronization
```

# 18. Day 33 Revision Checklist

Track your learning progress.

Day 33 learning progress

# 0/22

[ ]Understand real-time architecture

[ ]Polling vs Long Polling

[ ]Server-Sent Events

[ ]WebSocket fundamentals

[ ]HTTP WebSocket handshake

[ ]Implement WebSocket using OkHttp

[ ]Understand connection states

[ ]Automatic reconnection

[ ]Exponential backoff and jitter

[ ]Heartbeat and ping/pong

[ ]Firebase Cloud Messaging architecture

[ ]FCM token registration and updates

[ ]Notification vs data messages

[ ]WebSocket vs FCM

[ ]Foreground Services

[ ]Android background execution restrictions

[ ]Lifecycle-aware connection management

[ ]Message delivery acknowledgments

[ ]Message deduplication

[ ]Online/offline user status

[ ]Complete real-time chat architecture

[ ]Practice senior-level interview questions

 Reset checklist

# 19. What You Must Remember for Senior Interviews

| Topic | Important takeaway |
| --- | --- |
| Polling | Client periodically requests updates |
| Long Polling | Server waits before responding |
| SSE | Server-to-client streaming |
| WebSocket | Persistent bidirectional communication |
| Reconnection | Use exponential backoff and jitter |
| Heartbeat | Detect unhealthy connections |
| FCM | Push notification delivery |
| Foreground Service | For qualifying user-visible ongoing tasks |
| Message acknowledgment | Confirm application-level delivery |
| Room | Persist messages locally |
| Repository | Coordinate real-time and local data |
| Synchronization | Recover missed messages |

## Next Lesson – Day 34

Securing Android apps against dynamic attacks and Reverse engineering | by Prahalad Sharma | Medium

Android Security and Reliability

Week 5 – Day 34Intermediate → Advanced

Topics we will cover:

1. Android Keystore.
2. Secure token storage.
3. Network Security Configuration.
4. HTTPS and TLS.
5. Certificate Pinning.
6. R8 and ProGuard.
7. Code obfuscation.
8. Root detection concepts.
9. Crash handling and crash reporting.
10. Analytics architecture.
11. Feature flags.
12. Secure Android architecture.

My recommendation: For Day 33, spend extra time on these four concepts: WebSocket lifecycle management, automatic reconnection, FCM vs WebSocket, and message delivery acknowledgment. These are particularly important when interviewers ask you to design a WhatsApp-like or real-time delivery application.

Your roadmap position remains Week 5 – Day 33 completed in sequence; Day 34 is next.
