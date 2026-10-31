# Week 1 – Day 6: Complete Notes Application System Design

Week 1Day 6Beginner level

File name: `Day-6-Notes-Application-System-Design.md`

Today's goal: Learn how to design a complete Notes application from scratch, just like you would explain it in a system design interview.

This is an important day because we will combine everything you learned during Days 1–5.

Writing

# Day 6 – Complete Notes Application System Design

**Week:** 1 – System Design Basics\
**Level:** Beginner\
**Today's focus:** End-to-end system design\
**Example:** Google Keep-like Notes Application

---

# 1\. Introduction – What Are We Building?

Imagine an interviewer asks:

**"Design a Notes application like Google Keep."**

You might initially think:

- We need a screen to display notes.
- We need an API to save notes.
- We need a database to store notes.

But system design requires us to think beyond the UI and API.

We need to understand:

- What features should the application support?
- How many users will use it?
- How will the Android application communicate with the backend?
- Where will the notes be stored?
- How will we retrieve notes?
- How will we handle millions of notes?
- How will we make the application fast?
- What happens when the internet is unavailable?
- How will we prevent one user from accessing another user's notes?

These are the types of questions system design helps us answer.

## 1.1 What Is System Design?

System design is the process of defining the architecture, components, databases, APIs and communication mechanisms required to build a software system.

In simple words:

**System design means deciding how different parts of an application will work together to achieve a goal.**

---

# 2\. Step 1 – Understand the Requirements

Before designing an application, we must understand what the application should do.

This is the first and most important step.

Requirements are generally divided into two categories:

1. Functional requirements.
2. Non-functional requirements.

## 2.1 Functional Requirements

Functional requirements describe what the application should do.

For our Notes application, users should be able to:

1. Register and log in.
2. Create a note.
3. View all their notes.
4. View an individual note.
5. Edit an existing note.
6. Delete a note.
7. Search notes.
8. Pin important notes.
9. Archive notes.
10. Synchronize notes across devices.

These are the features of our application.

## 2.2 Non-Functional Requirements

Non-functional requirements describe how well the application should work.

Examples:

### Performance

The application should load notes quickly.

### Scalability

The application should support increasing numbers of users and notes.

### Availability

Users should be able to access the service reliably.

### Security

Users should only be able to access their own notes.

### Reliability

Notes should not be lost during normal failures.

### Consistency

When a user updates a note, the system should eventually show the updated content across devices according to the synchronization requirements.

### Offline Support

Users should be able to view and edit previously downloaded notes without an internet connection.

## 2.3 Functional vs Non-Functional Requirements

| Functional | Non-Functional |
| --- | --- |
| Create a note | Fast response |
| Edit a note | High availability |
| Delete a note | Data security |
| Search notes | Scalability |
| Pin a note | Reliability |
| Sync notes | Consistency |

**Interview tip:** Always start by clarifying functional and non-functional requirements before drawing the architecture.

---

# 3\. Step 2 – Identify the Main Components

Now that we understand what our application should do, let's identify its major components.

Our Notes application will have:

1. Android application.
2. Backend server.
3. API layer.
4. Authentication service.
5. Database.
6. Cache.
7. Synchronization mechanism.

Let's understand each component.

## 3.1 Android Application

This is the application installed on the user's mobile device.

Responsibilities:

- Display notes.
- Allow users to create and edit notes.
- Send API requests.
- Display API responses.
- Store notes locally.
- Support offline access.
- Synchronize local and remote data.

Possible Android technologies:

- Kotlin.
- Jetpack Compose or XML.
- ViewModel.
- Repository pattern.
- Retrofit.
- Room.
- WorkManager.
- Paging 3.

## 3.2 Backend Server

The backend handles the business logic.

Responsibilities:

- Receive API requests.
- Authenticate users.
- Validate requests.
- Create notes.
- Update notes.
- Delete notes.
- Retrieve notes.
- Communicate with the database.
- Communicate with Redis.

The backend acts as the bridge between the Android application and the database.

## 3.3 API Layer

The API layer exposes endpoints that the Android application can call.

Example:

http

```http
POST /api/notes
GET /api/notes
GET /api/notes/{id}
PUT /api/notes/{id}
DELETE /api/notes/{id}
```

We will discuss these APIs in detail later.

## 3.4 Authentication Service

The authentication service verifies the user's identity.

For example:

- User logs in.
- Backend verifies credentials.
- Backend issues an access token.
- Android application stores the token securely.
- Future API requests send the token.

Example:

http

```http
Authorization: Bearer access_token
```

The backend validates the token and identifies the user.

## 3.5 Database

The database stores permanent application data.

For our Notes application, we can use PostgreSQL.

It stores:

- User information.
- Notes.
- Note creation time.
- Note modification time.
- Note status.

## 3.6 Cache

We can use Redis to cache frequently requested data.

For example:

- Recently accessed notes.
- Frequently accessed note lists.
- Certain user information.

Caching reduces repeated database queries.

## 3.7 Synchronization Mechanism

The synchronization mechanism keeps local Android data and backend data aligned.

For example:

1. User creates a note offline.
2. Note is saved in Room.
3. Internet becomes available.
4. WorkManager schedules synchronization.
5. The note is uploaded to the backend.
6. The backend saves it in PostgreSQL.
7. The note becomes available on other devices.

---

# 4\. Step 3 – High-Level Architecture (HLD)

Now let's connect all the components.

This is called High-Level Design.

## 4.1 What Is HLD?

HLD stands for High-Level Design.

It describes the major components of a system and how they communicate with each other.

It does not focus on individual classes or detailed code implementation.

For example, HLD answers:

- What components do we need?
- Which database should we use?
- How does Android communicate with the backend?
- Where does caching happen?
- How does authentication work?

## 4.2 Notes Application – HLD Diagram

text

```text
                ANDROID APPLICATION
                        |
                        |
               UI (Compose/XML)
                        |
                        v
                    ViewModel
                        |
                        v
                    Repository
                     /      \
                    /        \
                   v          v
               Room DB     Retrofit
                              |
                              |
                         INTERNET
                              |
                              v
                        API GATEWAY
                              |
                              v
                         BACKEND API
                              |
                +-------------+-------------+
                |             |             |
                v             v             v
          Auth Service    Notes Service   Sync Service
                              |
                       +------+------+
                       |             |
                       v             v
                     Redis       PostgreSQL
                      Cache       Database
```

This is a logical architecture. In a simpler implementation, the API gateway and backend API may be the same deployment, and the services may be modules within one backend application.

## 4.3 Explain the Architecture

Let's understand the flow from top to bottom.

### Android UI

The user interacts with the application.

Example:

The user clicks the Create Note button.

### ViewModel

The ViewModel handles UI-related state and invokes the repository.

### Repository

The repository decides where to get the data.

It may:

- Read from Room.
- Request data from the backend.
- Save API responses into Room.

### Retrofit

Retrofit communicates with the backend using HTTP APIs.

### API Gateway

The gateway acts as an entry point for API traffic in an architecture that uses one.

It can handle:

- Routing.
- Authentication integration.
- Rate limiting.
- Request handling policies.

### Backend API

The backend executes business logic.

### Redis

Redis stores selected frequently accessed data.

### PostgreSQL

PostgreSQL stores the original application records.

---

# 5\. Step 4 – Low-Level Design (LLD)

We have understood the major components.

Now let's understand how we can organize the implementation.

## 5.1 What Is LLD?

LLD stands for Low-Level Design.

It describes the internal implementation of individual components.

It includes:

- Classes.
- Interfaces.
- Methods.
- Database tables.
- API request and response models.
- Relationships between objects.
- Business logic.

In simple words:

HLD tells us what components we need.

LLD tells us how to implement those components.

## 5.2 HLD vs LLD

| HLD | LLD |
| --- | --- |
| Overall architecture | Internal component design |
| Major components | Classes and interfaces |
| Database selection | Database schema |
| Communication between services | Method-level interactions |
| Architectural decisions | Implementation decisions |
| System-level view | Component-level view |

## 5.3 Android LLD – Suggested Package Structure

text

```text
com.example.notes
│
├── data
│   ├── local
│   │   ├── NotesDatabase.kt
│   │   ├── NoteDao.kt
│   │   └── NoteEntity.kt
│   │
│   ├── remote
│   │   ├── NotesApi.kt
│   │   └── NoteDto.kt
│   │
│   └── repository
│       └── NotesRepositoryImpl.kt
│
├── domain
│   ├── model
│   │   └── Note.kt
│   │
│   ├── repository
│   │   └── NotesRepository.kt
│   │
│   └── usecase
│       ├── CreateNoteUseCase.kt
│       ├── GetNotesUseCase.kt
│       ├── UpdateNoteUseCase.kt
│       └── DeleteNoteUseCase.kt
│
└── presentation
    ├── notes
    │   ├── NotesScreen.kt
    │   ├── NotesViewModel.kt
    │   └── NotesUiState.kt
    │
    └── edit
        ├── EditNoteScreen.kt
        └── EditNoteViewModel.kt
```

This is one possible structure, not a mandatory architecture.

## 5.4 Responsibilities of Each Layer

### Presentation Layer

Responsible for:

- Displaying UI.
- Handling user interactions.
- Observing UI state.

### Domain Layer

Responsible for:

- Business rules.
- Use cases.
- Domain models.
- Repository contracts.

### Data Layer

Responsible for:

- Database operations.
- API communication.
- Data mapping.
- Repository implementation.

This separation makes the application easier to maintain and test.

---

# 6\. Step 5 – Database Design

Now we need to decide how to store users and notes.

We will use PostgreSQL as our primary database.

## 6.1 Users Table

| Column | Data Type | Description |
| --- | --- | --- |
| id | BIGINT | Unique user ID |
| name | VARCHAR | User name |
| email | VARCHAR | User email |
| password\_hash | VARCHAR | Hashed password |
| created\_at | TIMESTAMP | Account creation time |

Example:

sql

```sql
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Important: We store a secure password hash, not the original password.

## 6.2 Notes Table

| Column | Data Type | Description |
| --- | --- | --- |
| id | BIGINT | Unique note ID |
| user\_id | BIGINT | Owner of the note |
| title | VARCHAR | Note title |
| content | TEXT | Note content |
| is\_pinned | BOOLEAN | Whether note is pinned |
| is\_archived | BOOLEAN | Whether note is archived |
| created\_at | TIMESTAMP | Creation time |
| updated\_at | TIMESTAMP | Last modification time |
| deleted\_at | TIMESTAMP | Soft deletion time |

Example:

sql

```sql
CREATE TABLE notes (
    id BIGSERIAL PRIMARY KEY,
    user_id BIGINT NOT NULL,
    title VARCHAR(255),
    content TEXT,
    is_pinned BOOLEAN DEFAULT FALSE,
    is_archived BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL,

    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

## 6.3 Why Do We Need user\_id?

Imagine two users:

User A has 100 notes.

User B has 50 notes.

We need to identify which notes belong to which user.

The `user_id` column establishes this relationship.

For example:

| id | user\_id | title |
| --- | --- | --- |
| 101 | 1 | Shopping |
| 102 | 1 | Meeting |
| 103 | 2 | Personal |
| 104 | 2 | Work |

User 1 owns notes 101 and 102.

User 2 owns notes 103 and 104.

## 6.4 Why Do We Need Indexing?

Imagine the notes table contains 10 million records.

A user requests their notes.

The database must identify records belonging to that user.

An index can help the database find matching records efficiently.

Example:

sql

```sql
CREATE INDEX idx_notes_user_id
ON notes(user_id);
```

For a common query that retrieves active notes in descending update order, we might use:

sql

```sql
CREATE INDEX idx_notes_user_updated
ON notes(user_id, updated_at DESC, id DESC)
WHERE deleted_at IS NULL;
```

The correct indexes depend on actual query patterns and database execution plans.

## 6.5 Why Do We Need Soft Delete?

When a user deletes a note, we could permanently remove it from the database.

Alternatively, we can mark it as deleted.

Example:

text

```text
deleted_at = 2026-10-31 10:00:00
```

This is called soft deletion.

Benefits:

- Supports trash or recently deleted features.
- Allows recovery.
- Helps synchronization between devices.
- Can preserve deletion information temporarily.

However, deleted records need a retention and permanent cleanup strategy.

---

# 7\. Step 6 – Design REST APIs

Now we need to define how Android communicates with the backend.

We will use REST APIs.

## 7.1 Create Note API

**Endpoint:**

http

```http
POST /api/v1/notes
```

**Request:**

json

```json
{
  "title": "Shopping List",
  "content": "Milk, Bread, Eggs"
}
```

**Response:**

json

```json
{
  "id": 101,
  "title": "Shopping List",
  "content": "Milk, Bread, Eggs",
  "isPinned": false,
  "createdAt": "2026-10-31T10:00:00Z",
  "updatedAt": "2026-10-31T10:00:00Z"
}
```

The backend identifies the user from the authenticated request. It should not blindly trust a user ID supplied in the request body.

## 7.2 Get All Notes API

**Endpoint:**

http

```http
GET /api/v1/notes?limit=20
```

**Response:**

json

```json
{
  "notes": [
    {
      "id": 101,
      "title": "Shopping List",
      "content": "Milk, Bread, Eggs"
    },
    {
      "id": 102,
      "title": "Meeting",
      "content": "Project discussion"
    }
  ],
  "nextCursor": "abc123",
  "hasMore": true
}
```

## 7.3 Get Individual Note API

**Endpoint:**

http

```http
GET /api/v1/notes/101
```

Returns the details of note 101, provided the authenticated user owns that note.

## 7.4 Update Note API

**Endpoint:**

http

```http
PUT /api/v1/notes/101
```

**Request:**

json

```json
{
  "title": "Updated Shopping List",
  "content": "Milk, Bread, Eggs, Butter"
}
```

**Response:**

json

```json
{
  "id": 101,
  "title": "Updated Shopping List",
  "content": "Milk, Bread, Eggs, Butter",
  "updatedAt": "2026-10-31T11:00:00Z"
}
```

## 7.5 Delete Note API

**Endpoint:**

http

```http
DELETE /api/v1/notes/101
```

**Response:**

json

```json
{
  "success": true,
  "message": "Note deleted successfully"
}
```

## 7.6 API Summary

| Operation | HTTP Method | Endpoint |
| --- | --- | --- |
| Create note | POST | /api/v1/notes |
| Get notes | GET | /api/v1/notes |
| Get note | GET | /api/v1/notes/{id} |
| Update note | PUT | /api/v1/notes/{id} |
| Delete note | DELETE | /api/v1/notes/{id} |

---

# 8\. Step 7 – Understand the Create Note Flow

Now let's understand what happens when a user creates a note.

This is important because interviewers often ask you to explain the complete request flow.

## 8.1 Scenario

A user opens the Notes application.

The user enters:

Title: Shopping List

Content: Milk, Bread, Eggs

The user clicks Save.

What happens internally?

## 8.2 Complete Flow

Step 1: User clicks Save.

Step 2: Android UI sends an event to the ViewModel.

Step 3: ViewModel calls the CreateNote use case or repository.

Step 4: The repository saves the note locally in Room as pending synchronization.

Step 5: The UI displays the saved note.

Step 6: A synchronization worker sends the note to the backend when network conditions permit.

Step 7: Backend validates the access token.

Step 8: Backend validates the request.

Step 9: Backend stores the note in PostgreSQL.

Step 10: Backend returns the created note and its server ID.

Step 11: Android updates the local record with the server ID and marks synchronization successful.

## 8.3 Diagram

text

```text
       User Clicks Save
              |
              v
         Android UI
              |
              v
           ViewModel
              |
              v
          Repository
              |
              v
           Room DB
              |
              v
       Display Local Note
              |
              v
       WorkManager Sync
              |
              v
          Retrofit API
              |
              v
         Backend Server
              |
              v
       Authentication
              |
              v
          PostgreSQL
              |
              v
        API Response
              |
              v
       Update Local DB
```

## 8.4 Why Save Locally First?

Because we want to support offline functionality.

If the internet is unavailable, users should still be able to create and edit notes.

This approach is commonly called an offline-first approach.

Important:

The local note should have a synchronization status, such as:

- PENDING.
- SYNCED.
- FAILED.

We should also use a stable client-generated identifier or idempotency key to avoid creating duplicate notes when a request is retried.

---

# 9\. Step 8 – Understand the Get Notes Flow

Now imagine the user opens the Notes screen.

How do we load the notes?

We want to provide a fast user experience.

## 9.1 Complete Flow

Step 1: User opens the Notes screen.

Step 2: ViewModel requests notes from the repository.

Step 3: Repository reads notes from Room.

Step 4: UI immediately displays locally available notes.

Step 5: Repository requests the latest notes from the backend.

Step 6: Backend checks Redis.

Step 7: If cached data exists, it may return the cached response.

Step 8: If cache misses, backend fetches notes from PostgreSQL.

Step 9: Backend stores the response in Redis.

Step 10: Android receives the response.

Step 11: Room is updated.

Step 12: UI automatically reflects the updated database contents.

## 9.2 Diagram

text

```text
          Open Notes Screen
                  |
                  v
               ViewModel
                  |
                  v
              Repository
                  |
                  v
                Room
                  |
                  v
             Display Notes
                  |
                  v
            Retrofit Request
                  |
                  v
              Backend
                  |
                  v
                Redis
                  |
            +-----+-----+
            |           |
          Hit          Miss
            |           |
            v           v
         Return      PostgreSQL
                        |
                        v
                    Update Cache
                        |
                        v
                    Return Data
                        |
                        v
                    Android
                        |
                        v
                    Update Room
```

This architecture helps the application provide fast initial rendering and fresh data when synchronization succeeds.

---

# 10\. Step 9 – Understand Pagination

Imagine a user has 10,000 notes.

Should we download all notes in a single API call?

No.

We should use pagination.

For our Notes application, we can use cursor-based pagination.

## 10.1 First Request

http

```http
GET /api/v1/notes?limit=20
```

The backend returns 20 notes and a cursor.

## 10.2 Second Request

http

```http
GET /api/v1/notes?limit=20&cursor=abc123
```

The backend returns the next 20 notes.

## 10.3 Android Behavior

When the user scrolls near the bottom of the list:

1. Paging library detects that more data is required.
2. PagingSource requests the next page.
3. Repository communicates with the API.
4. The response is stored locally if using a database-backed paging approach.
5. UI displays the additional notes.

## 10.4 Why Cursor Pagination?

Because:

- It works well with infinite scrolling.
- It avoids large OFFSET scans.
- It supports large datasets.
- It can provide more predictable navigation when records are inserted.

---

# 11\. Step 10 – Understand Caching

Now let's decide what information should be cached.

We can use Redis on the backend.

## 11.1 What Should We Cache?

Examples:

- Frequently accessed note details.
- First page of a popular or frequently requested list.
- User settings.
- Other frequently accessed information.

We should avoid blindly caching every record.

## 11.2 Example Cache Key

text

```text
notes:user:101:first-page
```

This means the first page of notes belonging to user 101.

For an individual note:

text

```text
note:user:101:note:500
```

The key includes the owner to avoid mixing user-specific data.

## 11.3 What Happens When a Note Is Updated?

Suppose the user updates note 500.

The backend:

1. Validates the user.
2. Updates the note in PostgreSQL.
3. Invalidates the cached note.
4. Invalidates affected cached note lists if necessary.
5. Returns success.

The next read can fetch the updated data.

## 11.4 What If Redis Is Down?

The backend can fall back to PostgreSQL if it has enough capacity.

Caching should improve performance, not become the only place where permanent notes exist.

---

# 12\. Step 11 – Understand Offline Support and Synchronization

This is particularly important for mobile applications.

Mobile devices can lose internet connectivity at any time.

Our application should still allow users to work with their notes.

## 12.1 What Is Offline-First Architecture?

Offline-first architecture means the application is designed to work with locally available data even when the network is unavailable.

The local database becomes an important source for the UI.

The backend is synchronized whenever connectivity permits.

## 12.2 Example Scenario

Imagine:

1. User opens the application.
2. Internet is unavailable.
3. User creates a new note.
4. Note is saved in Room.
5. The note appears immediately in the UI.
6. Internet becomes available.
7. WorkManager schedules synchronization.
8. The note is uploaded to the backend.
9. Backend stores the note in PostgreSQL.
10. Android marks the note as synchronized.

## 12.3 Synchronization Status

We can maintain a local field:

kotlin

```kotlin
enum class SyncStatus {
    PENDING,
    SYNCED,
    FAILED
}
```

Our local entity can include:

kotlin

```kotlin
@Entity
data class NoteEntity(
    @PrimaryKey
    val localId: String,

    val serverId: Long?,

    val title: String,

    val content: String,

    val updatedAt: Long,

    val syncStatus: SyncStatus
)
```

This is a simplified example.

## 12.4 What Happens if the Same Note Is Updated on Two Devices?

This is called a synchronization conflict.

Example:

Device A changes the title to:

"Shopping List"

Device B changes the title to:

"Weekly Shopping"

Both devices were offline when they made their changes.

When they reconnect, the backend receives conflicting updates.

How do we handle this?

Possible strategies:

### Last Write Wins

The system keeps the update with the latest accepted timestamp or version.

This is simple but may overwrite a user's change.

### Version-Based Conflict Detection

Each note has a version number.

Example:

text

```text
Note ID: 101
Version: 5
```

When updating, the client sends the version it last read.

If the server has already moved to version 6, it detects the conflict.

The server can reject the stale update and ask the client to reconcile.

### Manual Conflict Resolution

The application can show both versions and ask the user which one to keep.

For a simple Notes application, version-based conflict detection is a good starting point.

---

# 13\. Step 12 – Understand Scalability

Now imagine our application becomes popular.

Initially:

- 1,000 users.
- 10,000 notes.

Later:

- 1 million users.
- 100 million notes.

Our system must support growth.

This is where scalability becomes important.

## 13.1 What Is Scalability?

Scalability is the ability of a system to handle increasing workload by adding resources or improving its architecture.

## 13.2 Vertical Scaling

Vertical scaling means increasing the resources of an existing server.

For example:

- More CPU.
- More RAM.
- Faster storage.

Example:

A backend server with 4 CPU cores and 8 GB RAM is upgraded to 16 CPU cores and 32 GB RAM.

Advantages:

- Simple to understand.
- Fewer distributed-system concerns.

Disadvantages:

- Hardware has limits.
- Can become expensive.
- May create a single-server dependency.

## 13.3 Horizontal Scaling

Horizontal scaling means adding more servers or instances.

Example:

Initially:

text

```text
          Android Apps
               |
               v
          Backend Server
```

After horizontal scaling:

text

```text
            Android Apps
                 |
                 v
             Load Balancer
              /    |    \
             v     v     v
           API 1  API 2  API 3
              \    |    /
               \   |   /
                Database
```

A load balancer distributes incoming traffic among available backend instances.

Advantages:

- Supports more traffic.
- Provides flexibility.
- Helps improve availability.
- Allows instances to be added or removed based on demand.

## 13.4 When Would We Use Horizontal Scaling?

When one server cannot efficiently handle increasing traffic, we can add more backend instances.

However, the database, cache and other shared dependencies must also be designed to handle the additional load.

---

# 14\. Step 13 – Understand Availability and Reliability

What happens if one backend server crashes?

Should the entire Notes application stop working?

No.

We should design the system to handle failures.

## 14.1 High Availability

High availability means designing a system to remain accessible even when some components fail.

For example:

Instead of running one backend server, run multiple instances.

If one instance fails, traffic can be routed to healthy instances.

## 14.2 Reliability

Reliability means the system continues to operate correctly and preserves data as expected.

For a Notes application, reliability means:

- Notes should not disappear unexpectedly.
- Failed network requests should be handled.
- Database backups should exist.
- Synchronization should recover from temporary failures.

## 14.3 Database Backups

We should regularly back up the database.

If the database experiences data loss, backups and transaction-log recovery mechanisms can help restore information.

## 14.4 Retry Mechanism

Suppose an API request fails because of a temporary network problem.

The Android application can retry the request.

But retries must be designed carefully.

For example, if a create-note request is sent twice, we should not accidentally create duplicate notes.

We can use idempotency keys to make retry behavior safer.

---

# 15\. Step 14 – Understand Security

Notes can contain private information.

Therefore, security is important.

## 15.1 Authentication

Authentication verifies who the user is.

Example:

A user logs in using an email and password.

The backend verifies the credentials and issues an access token.

## 15.2 Authorization

Authorization checks what the user is allowed to access.

Example:

User A should not be able to retrieve User B's notes.

The backend must check ownership for every protected operation.

## 15.3 HTTPS

All communication between Android and backend should use HTTPS.

HTTPS encrypts data during transmission.

## 15.4 Secure Token Storage

Android should store sensitive authentication tokens using appropriate secure storage mechanisms.

Avoid storing passwords or sensitive tokens in plain text.

## 15.5 Input Validation

Backend should validate:

- Note title length.
- Content size.
- Required fields.
- Request format.
- User permissions.

Never rely only on Android-side validation.

---

# 16\. Step 15 – Complete Architecture Diagram

Now let's bring everything together.

This is the architecture you should be able to explain during an interview.

text

```text
                     ANDROID DEVICE
              +-------------------------+
              |                         |
              |      Compose / XML      |
              |             |           |
              |          ViewModel      |
              |             |           |
              |         Repository      |
              |          /      \       |
              |         v        v      |
              |       Room    Retrofit  |
              |                  |      |
              |              WorkManager|
              +------------------|------+
                                 |
                              HTTPS
                                 |
                                 v
                         LOAD BALANCER
                                 |
                                 v
                         BACKEND API
                                 |
                    +------------+------------+
                    |            |            |
                    v            v            v
                Auth Logic   Notes Logic  Sync Logic
                                 |
                          +------+------+
                          |             |
                          v             v
                        Redis       PostgreSQL
                        Cache        Database
```

## Explanation

### Android Side

- UI displays notes.
- ViewModel manages UI state.
- Repository coordinates data.
- Room stores local notes.
- Retrofit communicates with the backend.
- WorkManager handles suitable background synchronization work.

### Backend Side

- Auth logic authenticates users.
- Notes logic handles CRUD operations.
- Sync logic handles synchronization requirements.
- Redis caches selected frequently accessed data.
- PostgreSQL stores permanent application data.

---

# 17\. How to Explain This in an Interview

Suppose the interviewer asks:

**"Design a Notes application."**

You can structure your answer like this:

### Step 1: Clarify Requirements

"I would first clarify the functional requirements. Users should be able to create, view, update, delete, search and synchronize notes."

### Step 2: Explain Non-Functional Requirements

"The system should be scalable, reliable, secure and responsive. Since this is a mobile application, offline support is also important."

### Step 3: Explain High-Level Architecture

"I would have an Android client communicating with backend APIs over HTTPS. The backend would handle authentication and note operations. PostgreSQL would store the notes, while Redis could cache frequently accessed data."

### Step 4: Explain Android Architecture

"On Android, I would use a layered architecture with UI, ViewModel and Repository. Room would provide local persistence, Retrofit would communicate with the backend, and WorkManager would support background synchronization."

### Step 5: Explain Database Design

"I would create users and notes tables. Each note would have a user ID to identify its owner. I would add appropriate indexes for common queries."

### Step 6: Explain APIs

"I would expose REST endpoints for creating, retrieving, updating and deleting notes."

### Step 7: Explain Pagination

"For large note collections, I would use cursor-based pagination to avoid loading all records at once."

### Step 8: Explain Caching

"I would use Redis for frequently accessed data and invalidate relevant cache entries when notes are updated or deleted."

### Step 9: Explain Offline Support

"I would use Room as the local data source. Notes created offline would be marked as pending synchronization and uploaded when connectivity becomes available."

### Step 10: Explain Scalability

"If traffic increases, I would scale backend instances horizontally using a load balancer. I would also monitor database performance and introduce appropriate indexing and caching."

This is a good structure for explaining a beginner-level system design problem.

---

# 18\. Common Interview Questions and Answers

## Q1. How would you design a Notes application?

I would use an Android application, backend APIs, PostgreSQL as the primary database, Redis for caching and Room for local persistence.

The Android application would communicate with the backend through REST APIs.

## Q2. Why would you use Room?

Room provides local persistent storage.

It allows users to access previously downloaded notes even when the internet is unavailable.

## Q3. Why would you use Redis?

Redis can store frequently accessed data and reduce repeated database queries.

## Q4. Why would you use PostgreSQL?

PostgreSQL provides relational data storage, transactions and constraints.

It is suitable for storing users and notes.

## Q5. How would you support millions of notes?

I would use pagination, database indexing, suitable query optimization and caching for frequently accessed data.

## Q6. How would you prevent users from accessing other users' notes?

The backend would authenticate every protected request and verify that the requested note belongs to the authenticated user.

## Q7. How would you support offline functionality?

I would save notes in Room and maintain synchronization status.

When connectivity becomes available, background synchronization would upload pending changes.

## Q8. What happens if an API request fails?

The Android application would handle the error, preserve local data and retry suitable requests when appropriate.

## Q9. How would you handle duplicate note creation during retries?

I would use an idempotency key or stable client-generated identifier so that repeated requests do not create duplicate records.

## Q10. How would you handle simultaneous edits from two devices?

I would use version-based conflict detection and define a conflict-resolution strategy.

## Q11. Why would you use cursor-based pagination?

It is generally more efficient than large offset queries for large datasets when supported by suitable indexes.

## Q12. What happens if Redis fails?

The backend can fall back to PostgreSQL if database capacity allows. Redis should not be the only permanent storage for notes.

## Q13. How would you scale the backend?

I would add multiple backend instances behind a load balancer and ensure shared dependencies can support the increased traffic.

## Q14. What is the difference between HLD and LLD in this application?

HLD describes the overall architecture, such as Android, backend, Redis and PostgreSQL.

LLD describes implementation details such as NoteEntity, NoteDao, NotesRepository, API interfaces and database schemas.

## Q15. Why should the backend validate requests if Android already validates them?

Because clients cannot be trusted. Requests can come from modified applications or other clients. Backend validation protects the system.

---

# 19\. Day 6 – Revision Notes

| Concept | Important Point |
| --- | --- |
| System Design | Planning how application components work together |
| Functional Requirements | What the application should do |
| Non-Functional Requirements | How well the application should work |
| HLD | Overall architecture |
| LLD | Internal implementation |
| Android UI | Displays information |
| ViewModel | Manages UI state |
| Repository | Coordinates data sources |
| Room | Local database |
| Retrofit | API communication |
| WorkManager | Background work |
| Backend | Handles business logic |
| PostgreSQL | Primary persistent database |
| Redis | Cache |
| REST API | Communication interface |
| Pagination | Fetches data in smaller groups |
| Indexing | Helps database retrieve matching records efficiently |
| Offline-First | Supports local data access without internet |
| Synchronization | Keeps local and remote data aligned |
| Vertical Scaling | Increase resources of one server |
| Horizontal Scaling | Add more servers |
| Load Balancer | Distributes incoming requests |
| Authentication | Verifies user identity |
| Authorization | Checks user permissions |
| Idempotency | Makes repeated requests safe from duplicate effects |

---

# 20\. Day 6 – Self Assessment

Try answering these questions without referring to the notes.

### Requirements

1. What are functional requirements?
2. What are non-functional requirements?
3. What are the functional requirements of a Notes application?
4. What are the non-functional requirements?

### Architecture

5. Explain the complete Notes application architecture.
6. What is the responsibility of the backend?
7. Why do we need an API gateway?
8. Explain the role of Redis.
9. Explain the role of PostgreSQL.
10. What is the difference between HLD and LLD?

### Database and APIs

11. Design a users table.
12. Design a notes table.
13. Why do we need user\_id in the notes table?
14. Why do we need database indexes?
15. Design APIs for CRUD operations.

### Request Flows

16. Explain the complete create-note flow.
17. Explain the complete get-notes flow.
18. What happens when a user edits a note?
19. What happens when a user deletes a note?
20. How does the backend validate ownership?

### Android-Specific

21. Why do we need Room?
22. Why do we need Retrofit?
23. What is the responsibility of the Repository?
24. What is WorkManager?
25. How would you implement offline-first functionality?

### Advanced Thinking

26. How would you support millions of notes?
27. How would you implement pagination?
28. How would you implement caching?
29. How would you handle synchronization conflicts?
30. How would you scale the backend?
31. What happens if Redis becomes unavailable?
32. How would you prevent duplicate note creation?
33. How would you make the application highly available?

---

# 21\. Final Day 6 Learning Checklist

By the end of today, you should be able to explain:

- [ ]Functional and non-functional requirements.
- [ ]Major components of a Notes application.
- [ ]High-level architecture.
- [ ]Low-level architecture.
- [ ]Android application layers.
- [ ]Database schema.
- [ ]REST API design.
- [ ]Create-note request flow.
- [ ]Get-notes request flow.
- [ ]Redis caching.
- [ ]Cursor-based pagination.
- [ ]Offline-first architecture.
- [ ]Background synchronization.
- [ ]Conflict resolution.
- [ ]Vertical and horizontal scaling.
- [ ]Availability and reliability.
- [ ]Authentication and authorization.

---

# 22\. What You Have Learned in Week 1

Congratulations\! You have now completed the first week of System Design Basics.

| Day | Topic | What You Learned |
| --- | --- | --- |
| Day 1 | HLD vs LLD | Overall design and detailed design |
| Day 2 | Requirements | Functional and non-functional requirements |
| Day 3 | Performance | Latency, throughput, scalability and availability |
| Day 4 | Architecture and Databases | Client-server, REST, WebSockets, SQL and NoSQL |
| Day 5 | Caching and Pagination | Redis, cache strategies and pagination |
| Day 6 | Notes Application | Complete end-to-end system design |

**Day 6 complete – Week 1 System Design Basics.**

## How I recommend you study Day 6

Don't try to memorize the entire architecture.

Your most important exercise today is to take a blank paper and draw the Notes application architecture from memory.

Then explain these three flows aloud:

1. User creates a note.
2. User opens the notes screen.
3. User edits a note while offline and synchronizes later.

If you can explain these three flows, you're beginning to think like a system designer rather than just thinking about individual Android screens.

Next: Week 2 – Scalability and Distributed Systems Basics.
