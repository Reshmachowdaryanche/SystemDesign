# Week 6 – Day 37: WhatsApp / Chat Application System Design

Day 37 Week 6 Senior Android Developer Interview Preparation

File name: `37-WhatsApp-Chat-Application-System-Design.md`

Level: Beginner to Advanced Target: Senior Android Developer – 7 Years Experience

## Today's Objective

Learn how to design a scalable real-time chat application like WhatsApp from scratch.

We will cover:

- Functional and non-functional requirements.
- High-level architecture.
- WebSocket communication.
- Message sending and receiving.
- Message delivery acknowledgements.
- Online and offline status.
- Message persistence.
- Push notifications using FCM.
- Message ordering and synchronization.
- Database design.
- Redis and message queues.
- Scalability and reliability.
- Android chat application architecture.

# 1. What Is a Chat Application?

A chat application allows users to exchange messages in real time.

Examples:

- WhatsApp
- Telegram
- Signal
- Messenger

Imagine two users:

- User A sends a message to User B.
- The backend receives the message.
- The backend stores the message.
- The backend delivers it to User B if online.
- If User B is offline, the message remains pending.
- When User B reconnects, the backend delivers the pending message.

System Design Interview: Design WhatsApp Chat System | ADevGuide

[adevguide.com](<https://adevguide.com/system-design/interview/whatsapp-chat-system-design/?utm_source=chatgpt.com>)

Important: A chat application is not simply an API-based application. It needs persistent connections, message delivery guarantees, synchronization, and offline handling.

# 2. Functional Requirements

Functional requirements describe what the application should do.

Our chat application should support:

1. User registration and login.
2. One-to-one messaging.
3. Group messaging.
4. Sending text messages.
5. Sending images, videos, and documents.
6. Message delivery status.
7. Read receipts.
8. Online and offline status.
9. Typing indicators.
10. Message history.
11. Push notifications.
12. Message synchronization across devices.
13. Message deletion.
14. User blocking.

For our initial design, focus on one-to-one text messaging. We can extend the architecture for group messaging and media later.

# 3. Non-Functional Requirements

| Requirement | Explanation |
| --- | --- |
| Low Latency | Messages should arrive quickly |
| High Availability | Messaging should remain available |
| Scalability | Support millions of concurrent users |
| Reliability | Avoid losing accepted messages |
| Durability | Persist messages safely |
| Ordering | Maintain message order within a conversation |
| Security | Protect messages and user data |
| Fault Tolerance | Recover from server and network failures |

### Important Observation

Chat applications are real-time systems.

They need:

- Persistent connections.
- Asynchronous communication.
- Message queues.
- Reliable storage.
- Connection management.

# 4. Capacity Estimation

Let's assume:

- 100 million daily active users.
- Average 50 messages per user per day.
- Average message size: 1 KB.
- 10 million concurrent connections at peak.

## 4.1 Messages Per Day

text

```text
100M × 50
= 5 billion messages/day
```

## 4.2 Average Messages Per Second

text

```text
5,000,000,000 / 86,400
```

Approximately 57,870 messages per second.

Peak traffic can be several times higher.

## 4.3 Message Storage

Assume 1 KB per message.

text

```text
5 billion × 1 KB
= 5 TB/day
```

This is a simplified estimate excluding indexes, metadata, replication, and storage overhead.

Interview takeaway: Chat systems need horizontally scalable message storage and connection infrastructure.

# 5. High-Level Architecture

WhatsApp System Design Case Study | Prepflix

[prepflix.co.in](<https://prepflix.co.in/blogs/class-1-post-class.html?utm_source=chatgpt.com>)

Main components:

1. Android Application.
2. DNS.
3. Load Balancer.
4. API Gateway.
5. WebSocket Gateway.
6. Authentication Service.
7. Message Service.
8. Message Queue.
9. Message Database.
10. Redis.
11. Presence Service.
12. Notification Service.
13. Firebase Cloud Messaging.

## 5.1 Architecture Diagram

text

```text
             Android Client
                   |
                   v
             Load Balancer
                   |
                   v
            WebSocket Gateway
                   |
                   v
             Message Service
                   |
          +--------+--------+
          |                 |
          v                 v
      Message DB        Message Queue
                            |
                            v
                     Delivery Service
                            |
                   +--------+--------+
                   |                 |
                   v                 v
              Online User       Offline User
                   |                 |
                WebSocket           FCM
                   |                 |
                   v                 v
               Chat UI          Push Notification
```

### Responsibility of Each Component

| Component | Responsibility |
| --- | --- |
| WebSocket Gateway | Maintains client connections |
| Message Service | Handles message operations |
| Message Database | Stores messages |
| Redis | Stores presence and connection metadata |
| Message Queue | Handles asynchronous delivery |
| Delivery Service | Routes messages to recipients |
| FCM | Notifies offline or background users |
| Load Balancer | Distributes incoming connections |

# 6. WebSocket Communication

This is the most important topic of today's lesson.

## 6.1 What Is WebSocket?

WebSocket is a communication protocol that provides full-duplex communication over a persistent connection.

In simple words:

- Client can send messages.
- Server can send messages.
- Both can communicate without creating a new HTTP request for every message.

Understanding WebSocket Connection (Simple Explanation) - DEV Community

[dev.to](<https://dev.to/rajat10/understanding-websocket-connection-simple-explanation-5b15?utm_source=chatgpt.com>)

## 6.2 HTTP vs WebSocket

| HTTP | WebSocket |
| --- | --- |
| Request-response communication | Full-duplex communication |
| Usually independent requests | Persistent connection |
| Client generally initiates requests | Both sides can send data |
| Suitable for normal APIs | Suitable for real-time communication |
| Repeated polling may be needed | Server can push events |

### Interview Question: Why Not Use Normal REST APIs for Chat?

REST APIs are useful for operations such as login, fetching conversation history, and managing profiles.

However, continuously polling REST APIs for new messages creates unnecessary requests and additional latency.

WebSockets provide persistent bidirectional communication and are more suitable for active real-time messaging.

## 6.3 WebSocket Connection Lifecycle

text

```text
Android Application
        |
        v
   HTTP Handshake
        |
        v
 WebSocket Connection
        |
        v
 Authentication
        |
        v
 Connected
        |
        v
 Send / Receive Messages
        |
        v
 Heartbeat / Ping-Pong
        |
        v
 Connection Closed
```

Important concepts:

- Handshake.
- Authentication.
- Connection lifecycle.
- Ping and Pong.
- Heartbeat.
- Reconnection.
- Connection timeout.
- Connection cleanup.

# 7. Message Sending Architecture

Let's understand what happens when User A sends a message to User B.

Designing a Real-time Chat App (WhatsApp, Slack) | by Arvind Kumar | Medium

[codefarm0.medium.com](<https://codefarm0.medium.com/designing-a-real-time-chat-app-whatsapp-slack-bf17912356d7?utm_source=chatgpt.com>)

## Step-by-Step Flow

1. User A types a message.
2. Android generates a unique client message ID.
3. Message is displayed optimistically in the UI.
4. Client sends the message through WebSocket.
5. Backend authenticates the sender.
6. Backend validates and persists the message.
7. Backend generates or assigns a server sequence/version.
8. Backend acknowledges message acceptance.
9. Delivery service checks whether User B is online.
10. If online, message is delivered through WebSocket.
11. If offline, message remains pending.
12. When User B reconnects, pending messages are delivered.

Example message payload:

json

```json
{
  "type": "MESSAGE",
  "messageId": "msg_12345",
  "conversationId": "chat_101",
  "senderId": "user_A",
  "receiverId": "user_B",
  "content": "Hello!",
  "timestamp": 1793000000
}
```

Important: The backend should persist the message before acknowledging durable acceptance.

# 8. Message Delivery Acknowledgements

This is an extremely important senior-level interview topic.

A chat application needs to know whether a message was sent, delivered, or read.

#systemdesign #softwareengineering #backenddevelopment #codinginterview #developersoflinkedin | Kaushal Singh

[linkedin.com](<https://www.linkedin.com/posts/kaushal-singh-2a2b8722a_systemdesign-softwareengineering-backenddevelopment-activity-7430451661250174976-DTIH?utm_source=chatgpt.com>)

## Different Message States

| State | Meaning |
| --- | --- |
| PENDING | Waiting to be sent |
| SENT | Server accepted the message |
| DELIVERED | Recipient device acknowledged delivery |
| READ | Recipient opened or read the message |
| FAILED | Message could not be sent |

### Message Lifecycle

text

```text
PENDING
   |
   v
SENT
   |
   v
DELIVERED
   |
   v
READ
```

Failure can occur at different stages.

### Important Difference

SENT: Backend has accepted and stored the message.

DELIVERED: Recipient device has received and acknowledged the message.

READ: Recipient has read the message according to the application's read-receipt rules.

### Interview Question: What Happens if the Server Stores a Message but the Acknowledgement Never Reaches the Sender?

The sender may retry because it does not know whether the operation succeeded.

To prevent duplicate messages, use:

- Client-generated unique message IDs.
- Idempotency.
- Server-side duplicate detection.
- Acknowledgement tracking.

The backend should return the existing result when it receives a duplicate message ID rather than creating another message.

# 9. Message Ordering

Imagine User A sends:

text

```text
Message 1: Hello
Message 2: How are you?
Message 3: Good morning
```

Due to network delays, messages may arrive out of order.

How do we maintain ordering?

## Solutions

1. Generate unique message IDs.
2. Assign server-side sequence numbers per conversation.
3. Store messages with their sequence numbers.
4. Sort messages by sequence number.
5. Detect missing messages.
6. Fetch missing messages from the backend.

Example:

| Message | Sequence |
| --- | --- |
| Hello | 101 |
| How are you? | 102 |
| Good morning | 103 |

Important: Timestamps alone are not sufficient for reliable ordering because device clocks may differ.

# 10. Online and Offline Presence

How does WhatsApp know whether someone is online?

Designing a Real-Time Chat App That Actually Scales — No Bullsh\*t, Just Systems That Work | by Gyanaa Vaibhav | Medium

[medium.com](<https://medium.com/%40gynanrudr0/designing-a-real-time-chat-app-that-actually-scales-no-bullsh-t-just-systems-that-work-0f3a2f1a35e8?utm_source=chatgpt.com>)

A common approach is a Presence Service.

Example Redis entry:

text

```text
Key: presence:user_101

Value:
{
  "status": "ONLINE",
  "lastSeen": 1793000000
}
```

## Presence Flow

1. User connects to WebSocket.
2. Server authenticates the user.
3. Presence Service marks the user online.
4. Heartbeats maintain connection activity.
5. User disconnects.
6. Presence Service updates offline status.
7. Other users receive presence updates.

### Why Use Redis?

Redis provides fast access to frequently updated presence information.

However, presence data is temporary and should not be treated as permanent user information.

### Important Concepts

- Heartbeat.
- TTL.
- Connection timeout.
- Last seen.
- Multiple devices.
- Presence synchronization.

Senior-level point: A user may have multiple active devices. Online status should be calculated from all active connections, not from one socket alone.

# 11. Offline Message Delivery

What happens if User B is offline when User A sends a message?

The message should not be lost.

How WhatsApp Works Without Internet: Offline Messaging and Sync Explained - DEV Community

[dev.to](<https://dev.to/satyasootar/how-whatsapp-works-without-internet-offline-messaging-and-sync-explained-3nle?utm_source=chatgpt.com>)

Flow:

text

```text
User A sends message
        |
        v
   Message Service
        |
        v
   Message Database
        |
        v
Recipient Offline?
        |
        v
 Store as Pending
        |
        v
User B reconnects
        |
        v
Fetch Undelivered Messages
        |
        v
Deliver Through WebSocket
        |
        v
Update Delivery Status
```

Important concepts:

- Pending message storage.
- Message acknowledgements.
- Retry.
- Reconnection.
- Message synchronization.
- Idempotency.

Important: Do not depend on FCM as the permanent storage mechanism for chat messages. The backend message database remains the source of truth.

# 12. Firebase Cloud Messaging (FCM)

FCM is used to deliver push notifications.

Handle FCM messages on Android

[firebase.blog](<https://firebase.blog/posts/2018/09/handle-fcm-messages-on-android?utm_source=chatgpt.com>)

## How FCM Works

1. Android application registers with Firebase.
2. Firebase provides an FCM registration token.
3. Application sends the token to the backend.
4. User receives a message.
5. Backend detects that the recipient is offline or in the background.
6. Backend sends a push notification through FCM.
7. FCM delivers the notification to the device.
8. User opens the application.
9. Application synchronizes messages from the backend.

### Important Difference: WebSocket vs FCM

| WebSocket | FCM |
| --- | --- |
| Persistent bidirectional connection | Push messaging service |
| Useful for active real-time sessions | Useful for background notifications |
| Requires connection management | Managed by Firebase infrastructure |
| Supports continuous communication | Not a replacement for reliable message storage |

Interview Question: Why do we need both WebSocket and FCM?

WebSocket handles real-time communication when the application has an active connection.

FCM helps notify users when the application is in the background or no active connection is available.

# 13. Database Design

A chat application needs to store users, conversations, messages, and delivery information.

## 13.1 Users Table

| Column | Type |
| --- | --- |
| user\_id | BIGINT |
| name | VARCHAR |
| phone\_number | VARCHAR |
| profile\_image | VARCHAR |
| created\_at | TIMESTAMP |

## 13.2 Conversations Table

| Column | Type |
| --- | --- |
| conversation\_id | BIGINT |
| conversation\_type | VARCHAR |
| created\_at | TIMESTAMP |

## 13.3 Conversation Members Table

| Column | Type |
| --- | --- |
| conversation\_id | BIGINT |
| user\_id | BIGINT |
| joined\_at | TIMESTAMP |

## 13.4 Messages Table

| Column | Type |
| --- | --- |
| message\_id | VARCHAR |
| conversation\_id | BIGINT |
| sender\_id | BIGINT |
| content | TEXT |
| sequence\_number | BIGINT |
| created\_at | TIMESTAMP |

## 13.5 Message Receipts Table

| Column | Type |
| --- | --- |
| message\_id | VARCHAR |
| user\_id | BIGINT |
| delivery\_status | VARCHAR |
| delivered\_at | TIMESTAMP |
| read\_at | TIMESTAMP |

### Important Database Concepts

- Composite indexes.
- Partitioning.
- Sharding.
- Replication.
- Message retention.
- Pagination.
- Cursor-based retrieval.

### Interview Question: Why Use Cursor-Based Pagination Instead of Offset Pagination?

Offset pagination can become expensive when the message table grows large.

Cursor-based pagination retrieves messages using a stable identifier or sequence number.

Example:

http

```http
GET /api/v1/conversations/chat_101/messages?before=103&limit=20
```

This is useful for loading older messages efficiently.

# 14. Message Queue and Event-Driven Architecture

Imagine one message must be delivered to thousands of group members.

Processing every delivery synchronously can slow down the message service.

Instead, use asynchronous processing.

Building WebSocket Based Chat Server With Kafka | by Li Pei | Medium

[lip17.medium.com](<https://lip17.medium.com/building-chat-server-with-kafka-2c988f74beb8?utm_source=chatgpt.com>)

Architecture:

text

```text
        Message Service
               |
               v
          Message Queue
               |
       +-------+-------+
       |       |       |
       v       v       v
    Worker  Worker  Worker
       |       |       |
       v       v       v
    User A  User B  User C
```

Possible technologies:

- Apache Kafka.
- RabbitMQ.
- Amazon SQS.

### Advantages

- Decoupled services.
- Independent scaling.
- Traffic buffering.
- Asynchronous delivery.
- Better fault tolerance.

### Important Concepts

- At-least-once delivery.
- At-most-once delivery.
- Idempotent consumers.
- Consumer groups.
- Retry.
- Dead-letter queues.
- Eventual consistency.

Senior-level point: A message queue does not automatically guarantee exactly-once business processing. Idempotency is still necessary.

# 15. Media Upload Architecture

How do we send images, videos, and documents?

We should avoid sending large media files directly through the chat WebSocket.

Instead, use object storage.

How to Design a Chat Application like WhatsApp: 🔹 Chat Servers - Manages a large number of concurrent WebSocket connections, facilitate real-time communication, and ensure that messages are… | Ashish Pratap Singh | 21 comments

[linkedin.com](<https://www.linkedin.com/posts/ashishps1_how-to-design-a-chat-application-like-whatsapp-activity-7295298328714452993-uLip?utm_source=chatgpt.com>)

## Upload Flow

1. Android requests an upload URL.
2. Backend authorizes the upload.
3. Backend generates a temporary upload URL.
4. Android uploads media directly to object storage.
5. Backend stores media metadata.
6. Message containing media reference is sent.
7. Receiver downloads media using a secure URL.

Example message:

json

```json
{
  "type": "IMAGE",
  "messageId": "msg_123",
  "mediaUrl": "https://storage.example.com/image123",
  "mimeType": "image/jpeg"
}
```

Important concepts:

- Object Storage.
- Pre-signed URLs.
- CDN.
- Image Compression.
- Thumbnail Generation.
- Upload Retry.
- Media Access Control.

# 16. Android Application Architecture

This is particularly important for your Senior Android Developer interviews.

We need to design a robust Android chat application.

## 16.1 Architecture Diagram

text

```text
             Jetpack Compose UI
                     |
                     v
                  ViewModel
                     |
                     v
                  Use Case
                     |
                     v
                 Repository
                 /        \
                /          \
               v            v
           Room DB      WebSocket Client
               |            |
               v            v
          Local Messages  Chat Backend
               |
               v
          StateFlow
               |
               v
             Chat UI
```

## 16.2 Main Android Components

| Component | Responsibility |
| --- | --- |
| Jetpack Compose | Displays chat UI |
| ViewModel | Manages UI state |
| Use Cases | Contains business operations |
| Repository | Coordinates local and remote data |
| Room | Stores messages locally |
| WebSocket Client | Real-time communication |
| WorkManager | Deferrable background synchronization |
| FCM | Push notifications |
| DataStore | Stores application preferences |

## 16.3 Room Database

Example message entity:

kotlin

```kotlin
@Entity(
    tableName = "messages",
    indices = [
        Index(
            value = ["conversationId", "createdAt"]
        )
    ]
)
data class MessageEntity(

    @PrimaryKey
    val messageId: String,

    val conversationId: String,

    val senderId: String,

    val content: String,

    val status: String,

    val createdAt: Long
)
```

## 16.4 Message Status Enum

kotlin

```kotlin
enum class MessageStatus {
    PENDING,
    SENT,
    DELIVERED,
    READ,
    FAILED
}
```

## 16.5 Repository Example

kotlin

```kotlin
class ChatRepository(
    private val messageDao: MessageDao,
    private val socketClient: ChatSocketClient
) {

    fun observeMessages(
        conversationId: String
    ): Flow<List<MessageEntity>> {
        return messageDao.observeMessages(conversationId)
    }

    suspend fun sendMessage(
        message: MessageEntity
    ) {
        messageDao.insert(message)

        try {
            socketClient.sendMessage(message)
        } catch (exception: Exception) {
            messageDao.updateStatus(
                message.messageId,
                MessageStatus.FAILED.name
            )
        }
    }
}
```

This is a simplified illustration. A production implementation should distinguish temporary connection failure from permanent message failure, retain retryable messages, and reconcile server acknowledgements.

## 16.6 ViewModel Example

kotlin

```kotlin
class ChatViewModel(
    private val repository: ChatRepository
) : ViewModel() {

    private val _messages =
        MutableStateFlow<List<MessageEntity>>(emptyList())

    val messages = _messages.asStateFlow()

    fun observeMessages(conversationId: String) {
        viewModelScope.launch {
            repository.observeMessages(conversationId)
                .collect { messages ->
                    _messages.value = messages
                }
        }
    }
}
```

### Important Android Concepts

- Room and Flow.
- StateFlow and SharedFlow.
- Structured Concurrency.
- WebSocket lifecycle.
- Foreground/background behavior.
- WorkManager.
- FCM.
- Offline-first architecture.
- Optimistic UI.
- Message deduplication.

# 17. WebSocket Reconnection Strategy

Mobile networks are unstable.

Consider these situations:

- User changes from Wi-Fi to mobile data.
- Internet becomes unavailable.
- Application goes into background.
- Device enters Doze mode.
- WebSocket server restarts.
- Network connection drops.

We need a reconnection strategy.

## Exponential Backoff

Example:

text

```text
Attempt 1: 1 second
Attempt 2: 2 seconds
Attempt 3: 4 seconds
Attempt 4: 8 seconds
Attempt 5: 16 seconds
```

Use random jitter to avoid many clients reconnecting simultaneously.

Important concepts:

- Network monitoring.
- Connection lifecycle.
- Heartbeat.
- Exponential Backoff.
- Jitter.
- Authentication renewal.
- Pending Message Queue.
- Message Synchronization.

Interview Question: Should we keep WebSocket alive indefinitely in Android background?

No. Android restricts background execution, and maintaining a permanent socket is not guaranteed or appropriate for every application.

Use an active connection while appropriate, and use FCM for background notifications and backend synchronization when the application resumes.

# 18. Security in Chat Applications

Important security considerations:

1. HTTPS and WSS.
2. Authentication.
3. Authorization.
4. Secure token storage.
5. Rate limiting.
6. Input validation.
7. Abuse prevention.
8. Encryption at rest.
9. End-to-end encryption where required.

### End-to-End Encryption

In end-to-end encryption, messages are encrypted on the sender's device and decrypted on the recipient's device.

The backend should not have access to plaintext message contents.

Important concepts:

- Public and private keys.
- Session keys.
- Key exchange.
- Message encryption.
- Forward secrecy.
- Key rotation.

Remember: TLS protects communication in transit, but it is not the same as end-to-end encryption.

# 19. Scalability Challenges

How would we handle millions of concurrent users?

| Problem | Solution |
| --- | --- |
| Too many WebSocket connections | Multiple WebSocket gateways |
| Uneven connection distribution | Load balancing |
| High message traffic | Message queues |
| Database bottleneck | Sharding and partitioning |
| Presence lookups | Redis |
| Slow media downloads | CDN |
| Duplicate messages | Idempotency |
| Server failures | Reconnection and message recovery |
| Large groups | Asynchronous fan-out |
| Traffic spikes | Horizontal scaling |

### Important Advanced Concepts

- Connection affinity.
- Distributed presence.
- Message partitioning.
- Consistent hashing.
- Backpressure.
- Dead-letter queues.
- Multi-region deployment.
- Disaster recovery.

# 20. Important Senior-Level Interview Questions

Try answering these without looking at the notes.

## Fundamental Questions

1. Design a WhatsApp-like chat application.
2. What are the functional and non-functional requirements?
3. Why do we use WebSockets?
4. What is the difference between HTTP and WebSocket?
5. Explain the message lifecycle.

## Message Delivery

6. What is the difference between sent, delivered, and read?
7. How do you prevent duplicate messages?
8. How do you maintain message ordering?
9. What happens when the recipient is offline?
10. How do you recover messages after reconnection?

## Distributed Systems

11. How do you scale WebSocket connections?
12. Why do we need Redis for presence?
13. Why do we need message queues?
14. How do you handle message delivery failures?
15. How do you support millions of concurrent users?

## Android-Specific Questions

16. How do you implement WebSocket in Android?
17. How do you manage WebSocket lifecycle?
18. How do you handle network changes?
19. Why should Room be the local source of truth?
20. How do you integrate FCM with WebSocket?
21. How do you implement optimistic UI?
22. How do you synchronize messages using Room and Flow?
23. How do you handle background restrictions?
24. How do you implement retry and exponential backoff?
25. How do you prevent duplicate messages after reconnecting?

# 21. Practical Interview Assignment

Design a WhatsApp-like application without looking at the notes.

Your design must include:

- [ ]Functional Requirements.
- [ ]Non-Functional Requirements.
- [ ]Capacity Estimation.
- [ ]High-Level Architecture.
- [ ]WebSocket Communication.
- [ ]Message Sending Flow.
- [ ]Message Acknowledgements.
- [ ]Message Ordering.
- [ ]Online and Offline Presence.
- [ ]Offline Message Delivery.
- [ ]FCM Integration.
- [ ]Database Schema.
- [ ]Message Queue Architecture.
- [ ]Media Upload Architecture.
- [ ]Android MVVM Architecture.
- [ ]Room Database Integration.
- [ ]WebSocket Reconnection.
- [ ]Security.
- [ ]Scalability.

# 22. Day 37 Completion Checklist

- [ ]Understand functional and non-functional requirements.
- [ ]Complete capacity estimation.
- [ ]Draw high-level chat architecture.
- [ ]Understand WebSocket communication.
- [ ]Explain message sending and receiving.
- [ ]Understand sent, delivered, and read states.
- [ ]Explain message ordering.
- [ ]Understand online/offline presence.
- [ ]Explain offline message delivery.
- [ ]Understand FCM architecture.
- [ ]Design database schema.
- [ ]Understand message queues.
- [ ]Explain media upload architecture.
- [ ]Design Android chat architecture.
- [ ]Understand WebSocket reconnection.
- [ ]Explain security and end-to-end encryption.
- [ ]Answer all 25 interview questions.

# 23. What You Should Be Able to Explain After Day 37

You should be able to answer this interview question confidently:

"Design a WhatsApp-like chat application that supports millions of users. Explain how messages are sent, stored, delivered, acknowledged, and synchronized when users are offline."

Don't just memorize the architecture.

Explain why each component exists, what happens when it fails, and how the system recovers.

# Next Lesson

Week 6 – Day 38: YouTube / Video Streaming System Design

Topics:

- Video Upload Architecture.
- Object Storage.
- Video Transcoding.
- Video Processing Pipelines.
- CDN.
- Adaptive Bitrate Streaming.
- HLS and DASH.
- Video Metadata.
- Video Search.
- Android Video Player Architecture.
- ExoPlayer / Media3.
- Scalability and Reliability.

Roadmap Progress: Day 37 of 49.
