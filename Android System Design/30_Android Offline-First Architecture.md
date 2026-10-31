# Week 5 – Day 30: Android Offline-First Architecture

Day 30 Beginner to Advanced Senior Android Developer Interview Preparation

Today's topic is Offline-First Architecture in Android: Room Database, Local and Remote Data Sources, Repository Pattern, Single Source of Truth, WorkManager, Background Synchronization, Conflict Resolution and Data Synchronization.

This is an extremely important topic for Senior Android Developer interviews because real-world Android applications cannot assume that the user always has a stable internet connection.

Think about applications like WhatsApp, Instagram, Google Keep and e-commerce applications.

What happens when:

- The user opens an application without internet?
- The user creates a note while offline?
- The user sends a message when the network is unavailable?
- The user updates their profile offline?
- The application receives old data from the server?
- Two devices update the same record?

These are the problems Offline-First Architecture solves.

Today's learning objectives

By the end of Day 30, you should understand:

- What Offline-First Architecture is.
- Why Room is important.
- Local vs Remote Data Sources.
- Single Source of Truth.
- Repository-based data synchronization.
- WorkManager and background synchronization.
- Conflict resolution strategies.
- Retry and failure handling.
- How to design a real-world offline-first Android application.

# 1. What is Offline-First Architecture?

Let's start with a real-world example.

Imagine you are developing a Notes application.

The user opens the application and creates a note:

text

```text
Title: Android System Design

Description:
Learn offline-first architecture.
```

Now imagine the user has no internet connection.

In a traditional online-first application, the app might attempt to send the note to the backend immediately.

If the request fails, the note might not be saved at all.

This is a poor user experience.

In an offline-first application:

1. Save the note locally.
2. Immediately display it to the user.
3. Mark it as pending synchronization.
4. Wait until network connectivity becomes available.
5. Upload the note to the server.
6. Update the synchronization status.

The user can continue using the application without waiting for the network.

Definition:

Offline-First Architecture is an application architecture in which the local data source is the primary source for reading application data, while network communication is used to synchronize data with remote servers.

### Traditional architecture vs Offline-First architecture

Traditional online-first

User

API Request

Backend Server

Problem: Network failure can prevent data access or updates.Offline-first architecture

User

Local Database – Room

Read and write immediately

Background synchronization

Remote Server

Synchronize pending changes

Advantage: The user can continue working without internet.

# 2. Why do we need Offline-First Architecture?

Consider a real-world e-commerce application.

A user:

- Opens the application.
- Browses previously loaded products.
- Adds products to their cart.
- Changes quantities.
- Places an order.

Now imagine the internet disconnects during these operations.

A well-designed application should handle these situations gracefully.

### Major problems solved by Offline-First Architecture

| Problem | Solution |
| --- | --- |
| No internet | Read local data |
| Slow internet | Display cached data immediately |
| Failed API request | Retry later where appropriate |
| App closed during sync | Persist pending operations |
| Duplicate requests | Idempotency |
| Conflicting updates | Conflict resolution |
| Background sync | WorkManager |

### Real-world examples

- Google Keep: Notes can be edited offline and synchronized later.
- WhatsApp: Messages can be queued while connectivity is unavailable.
- Spotify: Downloaded content can be played offline.
- E-commerce: Previously cached product information can remain accessible.

# 3. Room Database – Local Data Storage

Important Android conceptRoom is an Android Jetpack persistence library that provides an abstraction over SQLite.

It allows us to store structured data locally.

Why do we need Room?

Suppose a user opens an application and downloads 100 products.

We can save these products in Room.

When the user opens the application again, we can immediately display the saved products without waiting for the API.

### Room architecture

(Deprecated) Android Room with a View - Java | Android Developers

[developer.android.com](<https://developer.android.com/codelabs/android-room-with-a-view?utm_source=chatgpt.com>)

Room consists of three major components:

1. Entity.
2. DAO.
3. Database.

## 3.1 Entity

An Entity represents a database table.

Example:

kotlin

```kotlin
@Entity(tableName = "products")
data class ProductEntity(

    @PrimaryKey
    val id: String,

    val name: String,

    val price: Double,

    val imageUrl: String,

    val updatedAt: Long
)
```

This creates a table named `products`.

| id | name | price |
| --- | --- | --- |
| 1 | iPhone | 80000 |
| 2 | Samsung | 65000 |
| 3 | Pixel | 70000 |

## 3.2 DAO – Data Access Object

DAO defines operations for interacting with the database.

kotlin

```kotlin
@Dao
interface ProductDao {

    @Query("SELECT * FROM products")
    fun observeProducts(): Flow<List<ProductEntity>>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertProducts(
        products: List<ProductEntity>
    )

    @Query("DELETE FROM products")
    suspend fun clearProducts()
}
```

Important operations:

- `@Query`: Executes SQL queries.
- `@Insert`: Inserts records.
- `@Update`: Updates records.
- `@Delete`: Deletes records.
- `@Upsert`: Inserts or updates records.

Notice that `observeProducts()` returns a Flow.

This means Room can emit updated database records whenever the underlying data changes.

## 3.3 Room Database

kotlin

```kotlin
@Database(
    entities = [ProductEntity::class],
    version = 1,
    exportSchema = true
)
abstract class AppDatabase : RoomDatabase() {

    abstract fun productDao(): ProductDao
}
```

Room database provides access to the DAO.

### Important interview question

Why should we use Room instead of directly using SQLite?

Room provides:

- Compile-time SQL query validation.
- Less boilerplate.
- Entity and DAO abstractions.
- Kotlin Coroutines and Flow support.
- Easier database migrations.
- Better integration with Android architecture components.

# 4. Local Data Source and Remote Data Source

In Offline-First Architecture, we commonly separate data sources.

## Local Data Source

Responsible for local storage.

Examples:

- Room.
- DataStore.
- Files.

## Remote Data Source

Responsible for communicating with the backend.

Examples:

- Retrofit.
- OkHttp.
- REST APIs.
- GraphQL APIs.

### Architecture

Building Offline-First Android Apps: Architecture, Sync Strategies, and Conflict Resolution | by Sourav Kumar | Medium

[medium.com](<https://medium.com/%40souravstudy/building-offline-first-android-apps-architecture-sync-strategies-and-conflict-resolution-5f5099a5d0bd?utm_source=chatgpt.com>)

Let's implement both.

### Local Data Source

kotlin

```kotlin
class ProductLocalDataSource(
    private val productDao: ProductDao
) {

    fun observeProducts(): Flow<List<ProductEntity>> {
        return productDao.observeProducts()
    }

    suspend fun saveProducts(
        products: List<ProductEntity>
    ) {
        productDao.insertProducts(products)
    }
}
```

### Remote Data Source

kotlin

```kotlin
class ProductRemoteDataSource(
    private val api: ProductApi
) {

    suspend fun getProducts(): List<ProductDto> {
        return api.getProducts()
    }
}
```

Notice the separation.

The local data source does not know anything about Retrofit.

The remote data source does not know anything about Room.

This makes the system easier to maintain and test.

# 5. Repository as the Single Source of Truth

Very important for interviewsThis is probably the most important concept in today's lesson.

What is Single Source of Truth?

It means that there should be one authoritative source from which the UI reads its data.

In a typical offline-first Android application, Room acts as the Single Source of Truth for locally available application data.

The UI observes Room.

The repository coordinates synchronization between the local and remote sources.

### How does it work?

Imagine the application starts.

1. ViewModel requests products from Repository.
2. Repository exposes the Room database Flow.
3. UI receives cached products immediately.
4. Repository fetches updated products from the API.
5. Repository saves API results into Room.
6. Room emits updated products.
7. UI automatically refreshes.

## Single Source of Truth flow

UI / Compose Screen

Observes product state

ViewModel

Collects repository Flow

Repository

Coordinates local and remote data

Room

Source of truthProvides UI data

API

Remote sourceProvides updates

Remote data is saved into Room. The UI does not need to observe the API directly.

### Repository implementation

kotlin

```kotlin
interface ProductRepository {
    fun observeProducts(): Flow<List<Product>>
    suspend fun refreshProducts()
}
```

Implementation:

kotlin

```kotlin
class ProductRepositoryImpl(
    private val local: ProductLocalDataSource,
    private val remote: ProductRemoteDataSource
) : ProductRepository {

    override fun observeProducts(): Flow<List<Product>> {
        return local.observeProducts()
            .map { entities ->
                entities.map { it.toDomain() }
            }
    }

    override suspend fun refreshProducts() {

        val remoteProducts = remote.getProducts()

        local.saveProducts(
            remoteProducts.map { it.toEntity() }
        )
    }
}
```

ViewModel:

kotlin

```kotlin
class ProductViewModel(
    private val repository: ProductRepository
) : ViewModel() {

    val products = repository.observeProducts()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5_000),
            initialValue = emptyList()
        )

    fun refresh() {
        viewModelScope.launch {
            repository.refreshProducts()
        }
    }
}
```

The important thing is that the ViewModel observes local data.

The API response is saved into Room, which then updates the UI.

# 6. Data Synchronization

Now let's understand the most interesting part.

How do we synchronize local and remote data?

Imagine a Notes application.

The user creates a note without internet.

We save the note locally.

But how do we remember that this note still needs to be uploaded?

We need synchronization metadata.

### Example database entity

kotlin

```kotlin
@Entity(tableName = "notes")
data class NoteEntity(

    @PrimaryKey
    val id: String,

    val title: String,

    val description: String,

    val isSynced: Boolean = false,

    val updatedAt: Long = System.currentTimeMillis()
)
```

Here:

- `isSynced = false`: Pending synchronization.
- `isSynced = true`: Successfully synchronized.

For more advanced systems, a dedicated synchronization state is better.

For example:

kotlin

```kotlin
enum class SyncStatus {
    PENDING,
    SYNCING,
    SYNCED,
    FAILED
}
```

### Synchronization flow

## Offline data synchronization

1. User creates a note
   Save the note into Room.
2. Mark as pending
   Set sync status to PENDING.
3. Network becomes available
   Eligible synchronization work can execute.
4. Upload pending notes
   Send changes to backend.
5. Receive server confirmation
   Update local sync status to SYNCED.

### Why is this important?

Because the user may:

- Close the application.
- Restart the phone.
- Lose connectivity.
- Move between networks.
- Experience an API timeout.

The pending operation must not disappear just because the Activity or application process was destroyed.

This is where WorkManager becomes important.

# 7. WorkManager – Background Synchronization

Important Android interview topicWorkManager is an Android Jetpack library for scheduling deferrable, persistent background work.

Examples:

- Upload pending notes.
- Synchronize messages.
- Upload images.
- Refresh cached data.
- Send analytics events.

Why not use a simple Coroutine?

A Coroutine is tied to its scope.

For example:

kotlin

```kotlin
viewModelScope.launch {
    uploadPendingNotes()
}
```

If the ViewModel is cleared, the coroutine is cancelled.

This is fine for UI-related work.

But what if the application closes before synchronization completes?

For persistent background work, WorkManager is generally more suitable.

### WorkManager architecture

Android Jetpack - Work Manager. Easiest way to Schedule | by Balakrishnan | Tech Log | Medium

[medium.com](<https://medium.com/droid-log/android-jetpack-work-manager-de6909eb684d?utm_source=chatgpt.com>)

### Important WorkManager components

| Component | Purpose |
| --- | --- |
| Worker | Defines background work |
| WorkRequest | Describes work to execute |
| WorkManager | Schedules and manages work |
| Constraints | Conditions required for execution |
| WorkInfo | Provides execution status |

## 7.1 Create a Worker

kotlin

```kotlin
class SyncNotesWorker(
    context: Context,
    params: WorkerParameters,
    private val repository: NoteRepository
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result {

        return try {

            repository.syncPendingNotes()

            Result.success()

        } catch (e: IOException) {

            Result.retry()

        } catch (e: Exception) {

            Result.failure()
        }
    }
}
```

`CoroutineWorker` is useful when working with suspend functions.

Note: In a Hilt-based application, use the appropriate Hilt WorkManager integration, including `@HiltWorker` and `HiltWorkerFactory`, to inject dependencies into workers.

## 7.2 Network Constraints

We can configure the worker to run only when the network is available.

kotlin

```kotlin
val constraints = Constraints.Builder()
    .setRequiredNetworkType(
        NetworkType.CONNECTED
    )
    .build()
```

## 7.3 Create WorkRequest

kotlin

```kotlin
val request =
    OneTimeWorkRequestBuilder<SyncNotesWorker>()
        .setConstraints(constraints)
        .setBackoffCriteria(
            BackoffPolicy.EXPONENTIAL,
            10,
            TimeUnit.SECONDS
        )
        .build()
```

## 7.4 Enqueue Work

kotlin

```kotlin
WorkManager.getInstance(context)
    .enqueueUniqueWork(
        "sync_notes",
        ExistingWorkPolicy.KEEP,
        request
    )
```

Why use `enqueueUniqueWork()`?

Imagine the user creates 20 notes.

Without proper scheduling, we might enqueue many unnecessary synchronization jobs.

Unique work helps prevent duplicate scheduling with the same name, depending on the selected policy.

### ExistingWorkPolicy

| Policy | Behavior |
| --- | --- |
| KEEP | Keep existing unfinished work |
| REPLACE | Replace existing unfinished work |
| APPEND | Add work after existing work |
| APPEND\_OR\_REPLACE | Append, or replace failed/cancelled chains |

### Important interview question

Does WorkManager execute immediately when the internet becomes available?

No. WorkManager guarantees eligible work is persisted and scheduled, subject to constraints and system optimization. It does not guarantee immediate execution at a particular time.

# 8. Conflict Resolution

Senior-level conceptThis is one of the most important concepts in distributed and offline-first systems.

Imagine the same note is edited on two devices.

Device A:

text

```text
Title: Android Architecture
```

Device B:

text

```text
Title: Advanced Android Architecture
```

Both devices were offline.

Later, both devices reconnect.

Which version should the server keep?

This is called a data synchronization conflict.

### Common conflict resolution strategies

## 8.1 Last Write Wins (LWW)

The latest update wins based on a timestamp or server-defined ordering.

Example:

| Device | Updated time | Value |
| --- | --- | --- |
| A | 10:00 AM | Android |
| B | 10:05 AM | Android Architecture |

The system chooses Device B's update.

Advantages:

- Simple.
- Easy to implement.

Disadvantages:

- Can overwrite valid changes.
- Depends on reliable ordering.
- Device clocks can be incorrect.

A server-generated version or logical ordering is generally safer than trusting device clocks alone.

## 8.2 Version-Based Conflict Resolution

Each record has a version number.

Example:

text

```text
Record ID: 101
Version: 1
```

Device A updates the record.

text

```text
Version: 2
```

Device B also attempts to update version 1.

The server sees that the current version is already 2 and rejects the stale update or returns a conflict response.

The client can then fetch the latest version and resolve the conflict.

This is useful for preventing lost updates.

## 8.3 Field-Level Merge

Instead of choosing one entire record, merge independent field changes.

Example:

Device A changes the title.

Device B changes the description.

The final record can contain both changes.

This requires clear rules about which fields can be merged safely.

## 8.4 Manual Conflict Resolution

For important data, ask the user to decide.

Example:

text

```text
This note was modified on another device.

1. Keep this device's version.
2. Keep the server version.
3. Review both versions.
```

### Comparison

| Strategy | Best suited for |
| --- | --- |
| Last Write Wins | Simple preferences |
| Version-based | Concurrent updates |
| Field-level merge | Independent fields |
| Manual resolution | Important user content |

Senior interview answer:

"I would choose the conflict resolution strategy based on the business requirements. For simple preferences, Last Write Wins may be acceptable. For critical user-generated content, I would prefer version-based conflict detection and merging or user-assisted conflict resolution to avoid silent data loss."

# 9. Handling Failed Synchronization

Synchronization can fail for several reasons:

- No internet.
- Server timeout.
- Authentication failure.
- Server unavailable.
- Validation error.
- Duplicate request.
- Rate limiting.

We should not retry every error blindly.

### Retry strategies

1. Exponential Backoff

Wait progressively longer between retries.

Example:

| Retry | Delay |
| --- | --- |
| 1 | 10 seconds |
| 2 | 20 seconds |
| 3 | 40 seconds |
| 4 | 80 seconds |

In production, use a maximum backoff duration and appropriate retry limits.

2. Idempotency

An operation is idempotent if executing it multiple times has the same intended effect as executing it once.

Consider a payment request.

If the client sends a payment request but does not receive a response, it cannot know whether the payment was processed.

If it retries blindly, it may charge the user twice.

An idempotency key helps prevent duplicate processing.

text

```text
POST /payments

Idempotency-Key: order-123-payment
```

The backend should recognize repeated requests with the same key and return the original result instead of processing a duplicate payment.

3. Retry only transient failures

| Error | Retry? |
| --- | --- |
| Network unavailable | Yes, later |
| Timeout | Usually |
| HTTP 500 | Usually, with limits |
| HTTP 429 | Respect Retry-After |
| HTTP 401 | Refresh authentication if appropriate |
| HTTP 400 | Usually no |
| Validation error | No, until data changes |

# 10. Full Offline-First Android Architecture

Let's combine everything.

Imagine we are developing an offline-first Notes application.

## Complete architecture

Presentation Layer

Compose UI + ViewModel

Domain Layer

GetNotesUseCase

CreateNoteUseCase

SyncNotesUseCase

NoteRepository Interface

Data Layer

NoteRepositoryImpl

Local + Remote Data Sources

Local

- Room
- Pending changes
- Sync status

Remote

- Retrofit
- Backend API
- Server records

WorkManager

Persistent background synchronization

Hilt

Dependency injection across components

### Complete user flow

1. User creates a note.
2. ViewModel calls CreateNoteUseCase.
3. Repository saves the note into Room.
4. UI immediately observes the new note.
5. Repository marks it as pending synchronization.
6. WorkManager schedules synchronization.
7. Worker uploads pending notes when constraints are met.
8. Backend confirms successful upload.
9. Repository updates the local sync status.
10. Room emits the updated note.
11. UI displays the synchronized status.

# 11. Advanced Concepts for Senior Android Interviews

These concepts are worth understanding beyond the basics.

## 11.1 Transactional Outbox Pattern

Imagine a user creates an order and we need to synchronize it.

We have two operations:

1. Save order locally.
2. Save a pending synchronization event.

What if the application crashes between these two operations?

We might have an order that was saved locally but has no pending sync record.

The Transactional Outbox Pattern solves this by saving both records inside the same database transaction.

Example:

kotlin

```kotlin
database.withTransaction {

    orderDao.insert(order)

    syncDao.insert(
        SyncEntity(
            id = order.id,
            type = "ORDER_CREATED"
        )
    )
}
```

Now both operations succeed together or fail together.

A background worker processes the outbox later.

This is a valuable pattern for reliable offline mutations.

## 11.2 Pull Synchronization vs Push Synchronization

Pull synchronization

The client periodically requests updates from the server.

text

```text
Android App → Server
"Give me records updated after version 100"
```

Push synchronization

The server notifies the client that new data is available.

Examples:

- Firebase Cloud Messaging.
- WebSocket events.

A notification should generally trigger a refresh or synchronization rather than being treated as the only source of truth.

## 11.3 Incremental Synchronization

Instead of downloading every record repeatedly, download only changed records.

Example:

text

```text
GET /notes?updated_after=timestamp
```

Or use a server-issued sync cursor:

text

```text
GET /notes?cursor=abc123
```

The cursor is generally safer than relying exclusively on device timestamps.

## 11.4 Tombstones

What happens when a user deletes a note offline?

If we immediately remove it from Room, how will the server know that the note should also be deleted?

A tombstone is a deletion marker.

kotlin

```kotlin
data class NoteEntity(
    val id: String,
    val title: String,
    val isDeleted: Boolean
)
```

The application can synchronize the deletion marker with the server and remove the record permanently according to the synchronization protocol.

## 11.5 Data Consistency

Understand these terms:

- Eventual consistency: Replicas become consistent after updates propagate.
- Strong consistency: Reads reflect the latest committed write according to the system's consistency guarantees.
- Read-your-writes consistency: A user can see their own successful updates.
- Conflict resolution: Rules for handling concurrent modifications.

An offline-first mobile application commonly uses eventual consistency between local and remote data.

# 12. Senior Android Developer Interview Questions

## Basic questions

1. What is Offline-First Architecture?
2. What is Room Database?
3. What are Entity, DAO and Database?
4. What is the difference between SQLite and Room?
5. What is a Local Data Source?
6. What is a Remote Data Source?
7. What is the Repository Pattern?
8. What is Single Source of Truth?
9. What is WorkManager?
10. What is a CoroutineWorker?

## Intermediate questions

11. How does Room Flow work?
12. Why should the UI observe Room instead of Retrofit directly?
13. What happens when an API request fails?
14. How do you schedule synchronization when the internet returns?
15. What is the difference between WorkManager and Coroutine?
16. What are WorkManager constraints?
17. What is unique work?
18. What is exponential backoff?
19. What is idempotency?
20. How do you handle authentication failures during synchronization?

## Senior-level questions

21. How would you design an offline-first chat application?
22. How would you synchronize data across multiple devices?
23. How would you prevent duplicate API requests?
24. How would you handle conflicting updates?
25. How would you implement incremental synchronization?
26. What is the Transactional Outbox Pattern?
27. What are tombstones in distributed synchronization?
28. How would you guarantee that locally saved changes are not lost?
29. How would you handle a user deleting data while offline?
30. How would you design an offline-first e-commerce cart?

# 13. Practical Assignment – Build an Offline-First Notes Application

Hands-on assignmentBuild a Notes application that works with and without internet.

### Required functionality

1. Notes UI

- Display notes.
- Create notes.
- Edit notes.
- Delete notes.

2. Local Storage

- Store notes using Room.
- Observe notes using Flow.
- Save changes immediately.

3. Remote Storage

- Create a Retrofit API service.
- Fetch notes.
- Upload pending notes.
- Update and delete notes remotely.

4. Synchronization

- Track pending changes.
- Schedule WorkManager.
- Retry transient failures.
- Update synchronization status.

5. Conflict Resolution

- Add a version field.
- Detect stale updates.
- Handle conflicts.

### Suggested project structure

text

```text
notes/
│
├── presentation/
│   ├── NotesScreen.kt
│   ├── NotesViewModel.kt
│   └── NotesUiState.kt
│
├── domain/
│   ├── model/
│   │   └── Note.kt
│   ├── repository/
│   │   └── NoteRepository.kt
│   └── usecase/
│       ├── GetNotesUseCase.kt
│       ├── CreateNoteUseCase.kt
│       └── SyncNotesUseCase.kt
│
└── data/
    ├── local/
    │   ├── NoteDao.kt
    │   └── NoteEntity.kt
    ├── remote/
    │   ├── NoteApi.kt
    │   └── NoteDto.kt
    ├── repository/
    │   └── NoteRepositoryImpl.kt
    └── worker/
        └── SyncNotesWorker.kt
```

### Expected behavior

| Scenario | Expected result |
| --- | --- |
| Internet available | Read local data and refresh |
| Internet unavailable | Display local data |
| Create note offline | Save locally |
| Edit note offline | Update locally |
| Delete note offline | Save pending deletion |
| Internet restored | Schedule synchronization |
| API timeout | Retry later |
| Duplicate upload | Avoid duplicate creation |
| Version conflict | Resolve according to policy |

# 14. Day 30 Revision Checklist

Day 30 progress

# 0/16

[ ]I understand Offline-First Architecture.

[ ]I understand Room Database.

[ ]I can explain Entity, DAO and Database.

[ ]I understand Local and Remote Data Sources.

[ ]I understand Single Source of Truth.

[ ]I can implement a Repository using Room and Retrofit.

[ ]I understand Flow-based database observation.

[ ]I understand WorkManager.

[ ]I can create a CoroutineWorker.

[ ]I understand network constraints and unique work.

[ ]I understand synchronization states.

[ ]I understand exponential backoff and retry.

[ ]I understand idempotency.

[ ]I can explain conflict resolution strategies.

[ ]I understand Transactional Outbox and tombstones.

[ ]I can design an offline-first Android application.

 Reset checklist

# 15. Final Interview Takeaways

Remember these 10 points:

1. Room is commonly the local Single Source of Truth.
2. Repository coordinates local and remote data.
3. UI should remain responsive without network access.
4. Room Flow automatically emits database updates.
5. WorkManager supports persistent, deferrable background work.
6. WorkManager does not guarantee immediate execution.
7. Idempotency prevents duplicate processing.
8. Versioning helps detect conflicting updates.
9. Transactional Outbox helps avoid losing pending synchronization events.
10. Offline-first systems commonly use eventual consistency between devices and servers.

A strong Senior Android Developer should be able to design the entire synchronization lifecycle, not just explain how to use Room or WorkManager.

Day 30 lesson complete# Next: Day 31 – Android Networking and API Architecture

Topics we'll cover:

- Retrofit and OkHttp architecture.
- Interceptors and Authenticator.
- HTTP request and response lifecycle.
- Token refresh mechanism.
- API error handling.
- Retry strategies.
- Pagination.
- API caching.
- Request deduplication.
- Network security and certificate pinning.

Target: Learn how to build a reliable, secure and scalable networking layer for a production Android application.
