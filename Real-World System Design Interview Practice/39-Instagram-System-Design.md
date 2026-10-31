# Week 6 – Day 39: Instagram System Design

Day 39 Week 6 Senior Android Developer Interview Preparation

File name: `39-Instagram-System-Design.md`

Level: Beginner to Advanced Target: Senior Android Developer – 7 Years Experience

## Today's Objective

Learn how to design a scalable social media application like Instagram from scratch.

We will understand how users upload photos, follow other users, generate personalized feeds, and interact with posts.

Today's topics:

- Functional and non-functional requirements.
- Capacity estimation.
- High-level architecture.
- User profiles and relationships.
- Image upload architecture.
- Image processing and compression.
- Object storage.
- Feed generation.
- Fan-out on Write vs Fan-out on Read.
- Redis caching.
- CDN.
- Database design.
- Notifications.
- Pagination and infinite scrolling.
- Android feed architecture.
- Scalability and reliability.

# 1. What Is Instagram?

Cómo saber a quiénes sigo en Instagram y cómo dejar de seguirlos - Infobae

Cómo ampliar la foto de un perfil en Instagram - Infobae

How To Get Instagram Reels, The App's New Short-Form Video Feature

Instagram is a social media platform where users can:

- Create accounts.
- Upload photos and videos.
- Follow other users.
- View personalized feeds.
- Like and comment on posts.
- Share posts.
- Receive notifications.
- View user profiles.
- Search for users and content.

From a system design perspective, Instagram combines:

1. Media storage.
2. Social graph management.
3. Feed generation.
4. Distributed caching.
5. Notifications.
6. High-volume read and write operations.

Important: One of the most challenging components is the personalized feed system.

# 2. Functional Requirements

Our Instagram-like application should support:

1. User registration and login.
2. Create and update profiles.
3. Upload photos and videos.
4. Follow and unfollow users.
5. View personalized home feeds.
6. Like and unlike posts.
7. Comment on posts.
8. Share posts.
9. View user profiles.
10. Search users.
11. Receive notifications.
12. View followers and following lists.
13. Delete posts.
14. Bookmark posts.
15. View post engagement.

For today's design, focus primarily on photo uploads, following relationships, and personalized feed generation.

# 3. Non-Functional Requirements

| Requirement | Explanation |
| --- | --- |
| Scalability | Support millions of users |
| Availability | Feed should remain accessible |
| Low Latency | Posts should load quickly |
| Durability | Uploaded images should not be lost |
| Consistency | Follow and like operations should be reliable |
| Performance | Smooth scrolling |
| Security | Protect private content |
| Reliability | Recover from service failures |

### Important Observation

Instagram is a read-heavy application.

Users may upload a few posts but view hundreds or thousands of posts.

Therefore, optimizing feed reads is extremely important.

# 4. Capacity Estimation

Let's assume:

- 100 million daily active users.
- 10 million posts uploaded daily.
- Average original image size: 2 MB.
- Each user views 100 posts daily.
- Average post metadata size: 1 KB.

## 4.1 Daily Image Storage

text

```text
10 million × 2 MB

= 20 million MB

= approximately 20 TB/day
```

This is the original image storage estimate, excluding thumbnails, resized variants, replication, and metadata.

## 4.2 Daily Feed Views

text

```text
100 million × 100 posts

= 10 billion post impressions/day
```

## 4.3 Average Feed Requests

Assuming each user requests 10 feed pages per day:

text

```text
100 million × 10
= 1 billion requests/day
```

Average requests per second:

text

```text
1 billion / 86,400

= approximately 11,574 requests/second
```

Peak traffic will be significantly higher.

Interview takeaway: We need efficient feed generation, caching, and CDN-based media delivery.

# 5. High-Level Architecture

Design Instagram - by Stephane Moreau

[systemdesignbutsimple.com](<https://www.systemdesignbutsimple.com/p/design-instagram?utm_source=chatgpt.com>)

Main components:

1. Android Application.
2. API Gateway.
3. Authentication Service.
4. User Service.
5. Follow Service.
6. Post Service.
7. Media Upload Service.
8. Image Processing Service.
9. Object Storage.
10. Feed Service.
11. Redis Cache.
12. Notification Service.
13. Message Queue.
14. CDN.
15. Search Service.

## 5.1 Architecture Diagram

text

```text
             Android Application
                      |
                      v
                 API Gateway
                      |
       +--------------+--------------+
       |              |              |
       v              v              v
  User Service   Post Service    Feed Service
       |              |              |
       v              v              v
   User DB         Post DB       Redis Cache
                      |
                      v
               Media Service
                      |
                      v
                Object Storage
                      |
                      v
                Image Processing
                      |
                      v
                    CDN

       Follow Service
              |
              v
         Follow DB

       Notification Service
              |
              v
         Message Queue
              |
              v
             FCM
```

### Responsibility of Each Component

| Component | Responsibility |
| --- | --- |
| User Service | User profiles |
| Follow Service | Following relationships |
| Post Service | Create and manage posts |
| Media Service | Upload coordination |
| Image Processing | Resize and compress images |
| Object Storage | Store images |
| Feed Service | Generate personalized feeds |
| Redis | Cache frequently accessed data |
| CDN | Deliver images quickly |
| Notification Service | Notify followers |

# 6. User Profile Architecture

Every user has a profile.

Example:

json

```json
{
  "userId": "user_101",
  "username": "john",
  "profileImage": "profile_101.jpg",
  "bio": "Android Developer",
  "followersCount": 1500,
  "followingCount": 500
}
```

A profile contains:

- User ID.
- Username.
- Profile picture.
- Bio.
- Followers.
- Following.
- Posts.
- Privacy settings.

## Database Design

Users Table

| Column | Type |
| --- | --- |
| user\_id | BIGINT |
| username | VARCHAR |
| email | VARCHAR |
| profile\_image | VARCHAR |
| bio | TEXT |
| is\_private | BOOLEAN |
| created\_at | TIMESTAMP |

Important concepts:

- Unique constraints.
- Indexing.
- Profile caching.
- Privacy controls.
- Read replicas.

# 7. Follow and Unfollow Architecture

Following relationships create the social graph.

Example:

text

```text
User A
  |
  +---- follows ----> User B
  |
  +---- follows ----> User C
  |
  +---- follows ----> User D
```

When User A follows User B:

1. Android sends a follow request.
2. Backend validates the request.
3. Follow Service creates the relationship.
4. Followers and following counts are updated.
5. User B receives a notification.
6. Feed generation can include User B's posts.

## Follow Table

| Column | Type |
| --- | --- |
| follower\_id | BIGINT |
| following\_id | BIGINT |
| created\_at | TIMESTAMP |

Example:

| follower\_id | following\_id |
| --- | --- |
| 101 | 201 |
| 101 | 202 |
| 101 | 203 |
| 102 | 201 |

Recommended composite indexes:

sql

```sql
CREATE INDEX idx_follower
ON follows(follower_id);

CREATE INDEX idx_following
ON follows(following_id);
```

These help answer:

- Who does this user follow?
- Who follows this user?

Interview Question: Why do we need a separate Follow Service?

Because social graph operations can become very large and require independent scaling and optimized relationship queries.

# 8. Image Upload Architecture

This is one of the most important components.

Imagine a user uploads a 10 MB photograph.

Should the Android application send the image directly through the main backend server?

We can avoid this bottleneck by using direct object storage uploads.

I have asked this system design question in 40+ mock interviews for system design, and yet only 40% of the candidates could give me a proper answer: | Tannika Majumder

[linkedin.com](<https://www.linkedin.com/posts/tannika-majumder-424a5040_i-have-asked-this-system-design-question-activity-7331672728501977088-VWG2?utm_source=chatgpt.com>)

## Upload Flow

1. User selects an image.
2. Android compresses the image if appropriate.
3. Android requests an upload session.
4. Backend validates user permissions.
5. Backend generates a pre-signed upload URL.
6. Android uploads the image directly to object storage.
7. Storage confirms the upload.
8. Backend creates the post metadata.
9. Processing service generates image variants.
10. Post becomes available.

### Why Direct Upload?

Without direct upload:

text

```text
Android
   |
   v
Backend Server
   |
   v
Object Storage
```

With direct upload:

text

```text
Android
   |
   v
Object Storage

Backend handles metadata
and upload authorization.
```

Advantages:

- Reduced backend bandwidth.
- Lower server load.
- Better scalability.
- Faster upload handling.
- Easier horizontal scaling.

### Important Concepts

- Pre-signed URLs.
- Multipart Upload.
- Image Compression.
- Upload Retry.
- Upload Progress.
- Resumable Upload.
- Content Validation.

# 9. Image Processing and Compression

Why does Instagram generate multiple versions of an uploaded image?

Imagine a user uploads a 4000 × 3000 pixel image.

A small mobile feed does not always need the original image.

We can generate optimized versions.

Tips & Tutorials From Web Hosting Experts - DreamHost Blog

[dreamhost.com](<https://www.dreamhost.com/glossary/wordpress/thumbnail-sizes/?utm_source=chatgpt.com>)

Example:

text

```text
Original Image
4000 × 3000
      |
      v
 Image Processing
      |
      +------> Thumbnail
      |         150 × 150
      |
      +------> Feed Image
      |         1080 × 810
      |
      +------> Medium Image
      |         720 × 540
      |
      +------> Original
```

### Why Generate Multiple Sizes?

- Reduce bandwidth.
- Faster image loading.
- Better scrolling performance.
- Support different screen sizes.
- Reduce CDN transfer costs.

### Important Concepts

- Image Resizing.
- Compression.
- JPEG.
- WebP.
- AVIF.
- Thumbnail Generation.
- EXIF Metadata.
- Image Orientation.

Senior-level point: Image processing should happen asynchronously rather than making the user wait for every variant to finish.

# 10. Object Storage

Object storage stores uploaded media.

Examples:

- Amazon S3.
- Google Cloud Storage.
- Azure Blob Storage.

Example storage structure:

text

```text
instagram-media/
    |
    +-- users/
    |      |
    |      +-- user_101/
    |             |
    |             +-- post_001/
    |                    |
    |                    +-- original.jpg
    |                    +-- thumbnail.jpg
    |                    +-- medium.webp
    |                    +-- feed.webp
```

### Why Not Store Images Directly in SQL?

Relational databases are designed primarily for structured data.

Large image files are better stored in object storage.

The database stores references to those objects.

Example:

json

```json
{
  "postId": "post_101",
  "userId": "user_101",
  "imageUrl": "https://cdn.example.com/post_101.webp"
}
```

# 11. Feed Generation Architecture

Most Important Topic of Day 39This is one of the most frequently discussed Instagram system design topics.

Imagine:

- User A follows User B.
- User A follows User C.
- User A follows User D.

All these users publish posts.

How does User A get a personalized feed?

How Instagram Loads Your Feed So Smoothly — The Secret Power of Queues | by Abutallah Naviwala | Medium

[medium.com](<https://medium.com/%40abutallah.n/how-instagram-loads-your-feed-so-smoothly-the-secret-power-of-queues-f2b2e7273f38?utm_source=chatgpt.com>)

There are two major approaches.

1. Fan-out on Write.
2. Fan-out on Read.

# 12. Fan-out on Write

Fan-out on Write is also called Push Model.

When a user publishes a post, the system distributes the post reference into the feed timelines of their followers.

### Example

User A has 1,000 followers.

User A publishes a new post.

The backend:

1. Stores the post.
2. Finds User A's followers.
3. Creates feed entries for followers.
4. Updates their feed timelines.

System Design Interview Questions at FAANG Companies (2026) | PracHub Knowledge Hub

[prachub.com](<https://prachub.com/resources/system-design-interview-questions-faang-2026?utm_source=chatgpt.com>)

Architecture:

text

```text
          User A Creates Post
                   |
                   v
               Post Service
                   |
                   v
               Message Queue
                   |
                   v
              Feed Workers
                   |
          +--------+--------+
          |        |        |
          v        v        v
       User B   User C   User D
       Feed     Feed     Feed
       Cache    Cache    Cache
```

### Advantages

- Very fast feed reads.
- Feed is already prepared.
- Good for read-heavy applications.

### Disadvantages

- Expensive post creation for users with millions of followers.
- Many feed entries must be generated.
- Increased write amplification.
- Additional storage requirements.

### Example

If a celebrity has 10 million followers:

One post could generate millions of feed updates.

This is called the celebrity problem or hot-user problem.

# 13. Fan-out on Read

Fan-out on Read is also called Pull Model.

Instead of updating every follower's feed when a post is created, we fetch posts from followed users when someone opens their feed.

### Example

User B follows:

- User A.
- User C.
- User D.

When User B opens Instagram:

1. Feed Service fetches the accounts User B follows.
2. Retrieves their latest posts.
3. Merges posts.
4. Sorts them.
5. Returns the feed.

Scaling Reads and Writes: Inside Fan-Out Patterns for Distributed Systems (Part 1) | by ScalaBrix | Medium

[scalabrix.medium.com](<https://scalabrix.medium.com/scaling-reads-and-writes-inside-fan-out-patterns-for-distributed-systems-part-1-ce84d5c3ebd0?utm_source=chatgpt.com>)

Architecture:

text

```text
             User B Opens Feed
                     |
                     v
                 Feed Service
                     |
                     v
              Following Service
                     |
          +----------+----------+
          |          |          |
          v          v          v
       User A     User C     User D
       Posts      Posts      Posts
          |          |          |
          +----------+----------+
                     |
                     v
                Merge Posts
                     |
                     v
                Sort / Rank
                     |
                     v
                Return Feed
```

### Advantages

- Cheap post creation.
- No need to push every post to followers.
- Useful for accounts with huge follower counts.

### Disadvantages

- Feed generation can be expensive.
- More database reads.
- Increased feed loading latency.

# 14. Fan-out on Write vs Fan-out on Read

| Feature | Fan-out on Write | Fan-out on Read |
| --- | --- | --- |
| Feed creation | When post is published | When feed is requested |
| Write cost | High | Low |
| Read cost | Low | High |
| Feed latency | Usually lower | Can be higher |
| Storage | More feed entries | Less precomputed feed data |
| Celebrity accounts | Expensive | More suitable |

## Which Approach Should Instagram Use?

A hybrid approach is often the most practical design.

### Hybrid Feed Generation

- Normal users: Fan-out on Write.
- Celebrity accounts: Fan-out on Read.
- Feed Service: Combines precomputed and dynamically fetched posts.

AlgoDaily - Software interview prep made easy. Coding Interview Questions.

[algodaily.com](<https://algodaily.com/lessons/design-of-the-twitter-architecture?utm_source=chatgpt.com>)

This balances feed read performance with write efficiency.

Interview takeaway: Do not assume one feed generation strategy works equally well for every user.

# 15. Redis Caching

Redis is an in-memory data store.

It can help improve feed performance.

What should we cache?

- User profiles.
- Popular posts.
- Feed timelines.
- Followers and following information.
- Like counts.
- Frequently accessed metadata.

Example:

text

```text
Key:
feed:user_101

Value:
[
  post_501,
  post_502,
  post_503,
  post_504
]
```

Boosting Performance with Caching: Redis, CDN, and Cache Invalidation | by Gyanaa Vaibhav | Medium

[medium.com](<https://medium.com/%40gynanrudr0/boosting-performance-with-caching-redis-cdn-and-cache-invalidation-499f49599b6e?utm_source=chatgpt.com>)

### Feed Retrieval

text

```text
Android Application
        |
        v
     Feed API
        |
        v
    Check Redis
        |
    +---+---+
    |       |
  Hit      Miss
    |       |
    v       v
 Return   Fetch DB
 Feed       |
            v
        Update Redis
            |
            v
         Return
```

### Important Redis Concepts

- Cache-Aside.
- TTL.
- Cache Invalidation.
- Eviction Policies.
- Sorted Sets.
- Distributed Caching.

Senior-level question: Why use Redis Sorted Sets for feed timelines?

Sorted Sets can store post IDs with scores, allowing efficient ordering by timestamp or ranking score.

# 16. CDN Architecture

CDN stands for Content Delivery Network.

It distributes images and videos across geographically distributed edge locations.

Mastering Amazon Web Services (AWS) Series

[multicloudinfo.com](<https://www.multicloudinfo.com/mastering-amazon-web-services-aws-series?utm_source=chatgpt.com>)

### Without CDN

text

```text
Android India
      |
      v
Origin Server USA
      |
      v
   Image
```

### With CDN

text

```text
Android India
      |
      v
Nearby CDN Edge
      |
      v
 Cached Image
```

If the image is not available in the edge cache, the CDN fetches it from the origin.

### CDN Advantages

- Faster image loading.
- Reduced origin server traffic.
- Lower latency.
- Better scalability.
- Reduced origin bandwidth costs.

### Important Concepts

- Edge Locations.
- Cache Hit Ratio.
- Cache Miss.
- TTL.
- Origin Server.
- Cache Invalidation.
- Signed URLs.

# 17. Like and Comment Architecture

Users can like and comment on posts.

## Like Flow

1. User clicks Like.
2. Android immediately updates the UI.
3. API request is sent to backend.
4. Backend validates the request.
5. Like relationship is stored.
6. Like count is updated.
7. Notification may be generated.

This is an example of optimistic UI updates.

### Likes Table

| Column | Type |
| --- | --- |
| user\_id | BIGINT |
| post\_id | BIGINT |
| created\_at | TIMESTAMP |

Use a unique constraint on `(user_id, post_id)` to prevent duplicate likes.

## Comment Table

| Column | Type |
| --- | --- |
| comment\_id | BIGINT |
| post\_id | BIGINT |
| user\_id | BIGINT |
| content | TEXT |
| created\_at | TIMESTAMP |

### Important Concepts

- Optimistic Updates.
- Idempotency.
- Unique Constraints.
- Counter Aggregation.
- Pagination.
- Event-Driven Notifications.

# 18. Notification Architecture

Imagine User A uploads a post.

Their followers may receive notifications when appropriate.

Other notification events include:

- Someone follows you.
- Someone likes your post.
- Someone comments on your post.
- Someone mentions you.

Designing scalable Notification System | by aditya goel | Medium

[adityagoel123.medium.com](<https://adityagoel123.medium.com/designing-scalable-notification-system-79f83272755e?utm_source=chatgpt.com>)

Architecture:

text

```text
       Post / Like / Follow
                |
                v
          Event Producer
                |
                v
           Message Queue
                |
                v
        Notification Service
                |
          +-----+------+
          |            |
          v            v
      Notification   FCM
       Database       |
          |            |
          v            v
       Android      Android
       In-App       Push
```

### Why Use a Message Queue?

Notification delivery should not block the main post creation or like operation.

Asynchronous processing allows notification workers to scale independently.

Possible technologies:

- Kafka.
- RabbitMQ.
- Amazon SQS.
- Firebase Cloud Messaging.

# 19. Database Design

Let's design the main tables.

## 19.1 Users Table

| Column | Type |
| --- | --- |
| user\_id | BIGINT |
| username | VARCHAR |
| email | VARCHAR |
| profile\_image | VARCHAR |
| bio | TEXT |
| created\_at | TIMESTAMP |

## 19.2 Posts Table

| Column | Type |
| --- | --- |
| post\_id | BIGINT |
| user\_id | BIGINT |
| caption | TEXT |
| media\_key | VARCHAR |
| created\_at | TIMESTAMP |

## 19.3 Follows Table

| Column | Type |
| --- | --- |
| follower\_id | BIGINT |
| following\_id | BIGINT |
| created\_at | TIMESTAMP |

## 19.4 Likes Table

| Column | Type |
| --- | --- |
| user\_id | BIGINT |
| post\_id | BIGINT |
| created\_at | TIMESTAMP |

## 19.5 Comments Table

| Column | Type |
| --- | --- |
| comment\_id | BIGINT |
| post\_id | BIGINT |
| user\_id | BIGINT |
| content | TEXT |
| created\_at | TIMESTAMP |

## 19.6 Feed Table

| Column | Type |
| --- | --- |
| user\_id | BIGINT |
| post\_id | BIGINT |
| ranking\_score | DOUBLE |
| created\_at | TIMESTAMP |

### Important Database Concepts

- Indexing.
- Sharding.
- Partitioning.
- Replication.
- Denormalization.
- Composite Indexes.
- Read Replicas.
- Eventual Consistency.

# 20. Feed Ranking Algorithm

A feed does not necessarily display posts in simple chronological order.

It may consider:

- Recency.
- User interaction.
- Relationship strength.
- Post popularity.
- Content relevance.
- User preferences.

Example:

| Post | Recency | Engagement | Relationship |
| --- | --- | --- | --- |
| Post A | High | Medium | High |
| Post B | Medium | High | Low |
| Post C | High | Low | Medium |

A simplified ranking formula might be:

text

```text
Ranking Score =
    Recency Score
  + Engagement Score
  + Relationship Score
  + Relevance Score
```

This is only an illustrative model, not Instagram's actual ranking algorithm.

Interview point: Feed generation and feed ranking are different responsibilities.

Feed generation determines which posts are candidates.

Feed ranking determines the order in which those candidates appear.

# 21. Android Application Architecture

This section is especially important for your Senior Android Developer interviews.

We need to design an Android application that supports:

- Infinite scrolling.
- Image loading.
- Likes and comments.
- Pagination.
- Offline caching.
- Smooth feed rendering.

## 21.1 Android Architecture

Android Paging 3 — combine network and local data using Room and Realm | by Jakub Mielczarek | Medium

[medium.com](<https://medium.com/%40miel3k/android-paging-3-combine-network-and-local-data-using-room-and-realm-dea6435b6183?utm_source=chatgpt.com>)

text

```text
             Jetpack Compose UI
                     |
                     v
                  ViewModel
                     |
                     v
                   UseCase
                     |
                     v
                 Repository
                 /        \
                /          \
               v            v
            Room DB       Retrofit
               |             |
               v             v
          Local Feed      Feed API
               |
               v
            Flow
               |
               v
          Compose UI
```

## 21.2 Important Android Components

| Component | Responsibility |
| --- | --- |
| Jetpack Compose | Feed UI |
| ViewModel | UI state |
| Repository | Data coordination |
| Room | Local feed cache |
| Retrofit | API communication |
| Paging 3 | Infinite scrolling |
| Coil | Image loading |
| WorkManager | Deferrable background work |
| StateFlow | Observable UI state |

# 22. Pagination and Infinite Scrolling

Imagine the user follows 1,000 accounts.

Should we load all posts at once?

No.

We use pagination.

Pagination in Mobile App. Pagination is a critical part of mobile… | by Dileep Jaiswal | Jan, 2026 | Medium

[medium.com](<https://medium.com/%40dileep.ducs/pagination-in-mobile-app-84c4a2edcd60?utm_source=chatgpt.com>)

## Offset Pagination

http

```http
GET /api/v1/feed?page=1&limit=20
```

Problems:

- Large offsets can become expensive.
- New posts can shift page boundaries.
- Duplicate or skipped results may occur.

## Cursor-Based Pagination

http

```http
GET /api/v1/feed?cursor=post_501&limit=20
```

The backend uses a stable cursor to retrieve the next page.

Advantages:

- Better scalability.
- More stable pagination.
- Efficient retrieval of large feeds.

### Paging 3 in Android

Paging 3 helps load data incrementally.

Example:

kotlin

```kotlin
class FeedRepository(
    private val feedDao: FeedDao,
    private val feedApi: FeedApi
) {

    fun getFeed(): Flow<PagingData<PostEntity>> {

        return Pager(
            config = PagingConfig(
                pageSize = 20,
                enablePlaceholders = false
            ),
            pagingSourceFactory = {
                feedDao.getFeedPagingSource()
            }
        ).flow
    }
}
```

For a production network-backed feed, use `RemoteMediator` to coordinate API pagination and Room persistence.

# 23. Image Loading Optimization in Android

Image loading is critical for smooth scrolling.

Imagine a feed with 20 images.

If every image is loaded at full resolution, memory usage and network consumption increase.

We should use optimized image loading.

### Recommended Techniques

1. Load appropriately sized images.
2. Use image caching.
3. Use memory and disk caches.
4. Compress images on upload.
5. Avoid unnecessary recomposition.
6. Preload nearby images.
7. Cancel obsolete requests.
8. Use placeholders.

### Coil Example

Dependency:

kotlin

```kotlin
implementation(
    "io.coil-kt.coil3:coil-compose:<version>"
)
```

Compose:

kotlin

```kotlin
@Composable
fun PostImage(imageUrl: String) {

    AsyncImage(
        model = imageUrl,
        contentDescription = "Post image",
        modifier = Modifier
            .fillMaxWidth()
            .aspectRatio(1f),
        contentScale = ContentScale.Crop
    )
}
```

Senior-level point: Use the correct image variant from the CDN rather than downloading a large original image and resizing it on the device.

# 24. Feed UI with Jetpack Compose

Example:

kotlin

```kotlin
@Composable
fun InstagramFeed(
    posts: LazyPagingItems<Post>
) {

    LazyColumn {

        items(
            count = posts.itemCount,
            key = { index ->
                posts[index]?.postId ?: index
            }
        ) { index ->

            posts[index]?.let { post ->

                PostItem(post = post)
            }
        }
    }
}
```

Important concepts:

- LazyColumn.
- Stable Keys.
- Paging 3.
- State Management.
- Image Loading.
- Recomposition Optimization.
- Scroll Restoration.

# 25. Offline-First Feed Architecture

What happens when the user loses internet connectivity?

We should still display previously loaded posts.

A common architecture uses Room as the local source of truth.

text

```text
           Feed API
              |
              v
          Repository
              |
              v
           Room DB
              |
              v
            Flow
              |
              v
         Jetpack Compose
```

Flow:

1. Application loads cached posts from Room.
2. Repository requests the latest feed.
3. API response is stored in Room.
4. Room emits updated data.
5. Compose automatically updates the UI.

### Important Concepts

- Room.
- Flow.
- RemoteMediator.
- Cache Expiration.
- Network Synchronization.
- Optimistic Updates.
- Offline Error Handling.

# 26. Security and Privacy

Social media applications must protect user data.

Important security considerations:

1. Authentication.
2. Authorization.
3. Private account handling.
4. Secure media access.
5. Signed URLs.
6. HTTPS.
7. Rate limiting.
8. Spam prevention.
9. Content moderation.
10. Account blocking.

### Private Account Example

If User A has a private account:

- Only approved followers can view protected posts.
- Follow requests require approval.
- Feed Service must validate access permissions.
- CDN access must not bypass privacy controls.

Important: Hiding a private post in the Android UI is not security. Authorization must be enforced on the backend and media delivery path.

# 27. Scalability Challenges

How would we support millions of users?

| Problem | Solution |
| --- | --- |
| Large image uploads | Direct object storage upload |
| Slow image loading | CDN |
| High feed read traffic | Redis caching |
| Expensive feed generation | Hybrid fan-out |
| Celebrity accounts | Fan-out on Read |
| Large follower lists | Database partitioning |
| Too many notifications | Message queues |
| Large image memory usage | Optimized image variants |
| Slow database queries | Indexing and read replicas |
| Large feed history | Cursor pagination |

### Advanced Concepts

- Distributed Caching.
- Consistent Hashing.
- Database Sharding.
- Eventual Consistency.
- CQRS.
- Event-Driven Architecture.
- Backpressure.
- Multi-Region Deployment.

# 28. Senior-Level Interview Questions

Try answering these without looking at the notes.

## Fundamental Questions

1. Design an Instagram-like application.
2. What are the functional and non-functional requirements?
3. Why is Instagram considered a read-heavy application?
4. How do you design user profiles?
5. How do you manage followers and following relationships?

## Upload and Storage

6. How does image upload work?
7. Why do we use pre-signed URLs?
8. Why should images be stored in object storage?
9. Why do we generate multiple image resolutions?
10. How do you optimize image delivery?

## Feed Architecture

11. What is feed generation?
12. Explain Fan-out on Write.
13. Explain Fan-out on Read.
14. What is the celebrity problem?
15. Why is hybrid feed generation useful?
16. Why do we need Redis for feed caching?
17. What is the difference between feed generation and feed ranking?

## Distributed Systems

18. How does CDN improve performance?
19. How do you handle millions of followers?
20. Why do we need message queues?
21. How do you design notification delivery?
22. How do you handle eventual consistency in like counts?
23. How do you scale feed generation?

## Android-Specific Questions

24. How do you design Instagram feed architecture in Android?
25. Why should Room be the local source of truth?
26. How does Paging 3 work?
27. What is RemoteMediator?
28. How do you implement infinite scrolling?
29. How do you optimize LazyColumn?
30. How do you prevent unnecessary recompositions?
31. How do you optimize image loading?
32. How do you implement optimistic like updates?
33. How do you restore feed state after configuration changes?
34. How do you support offline feed viewing?

# 29. Practical Interview Assignment

Design an Instagram-like application without looking at the notes.

Your architecture must include:

- [ ]Functional Requirements.
- [ ]Non-Functional Requirements.
- [ ]Capacity Estimation.
- [ ]High-Level Architecture.
- [ ]User Profile Service.
- [ ]Follow Service.
- [ ]Image Upload Architecture.
- [ ]Image Processing.
- [ ]Object Storage.
- [ ]Feed Generation.
- [ ]Fan-out on Write.
- [ ]Fan-out on Read.
- [ ]Hybrid Feed Architecture.
- [ ]Redis Caching.
- [ ]CDN.
- [ ]Like and Comment Architecture.
- [ ]Notification Architecture.
- [ ]Database Schema.
- [ ]Feed Ranking.
- [ ]Android MVVM Architecture.
- [ ]Paging 3.
- [ ]Coil Image Loading.
- [ ]Offline-First Architecture.
- [ ]Security and Privacy.
- [ ]Scalability Strategy.

# 30. Day 39 Completion Checklist

- [ ]Understand functional and non-functional requirements.
- [ ]Complete capacity estimation.
- [ ]Draw high-level architecture.
- [ ]Understand user profiles.
- [ ]Understand follow/unfollow architecture.
- [ ]Explain image upload architecture.
- [ ]Understand image processing.
- [ ]Explain object storage.
- [ ]Understand feed generation.
- [ ]Explain Fan-out on Write.
- [ ]Explain Fan-out on Read.
- [ ]Understand hybrid feed generation.
- [ ]Understand Redis caching.
- [ ]Explain CDN architecture.
- [ ]Understand likes and comments.
- [ ]Explain notification architecture.
- [ ]Design database schema.
- [ ]Understand feed ranking.
- [ ]Design Android feed architecture.
- [ ]Understand Paging 3.
- [ ]Explain Coil image loading.
- [ ]Understand offline-first architecture.
- [ ]Explain scalability challenges.
- [ ]Answer all 34 interview questions.

# 31. What You Should Be Able to Explain After Day 39

You should be able to confidently answer:

"Design an Instagram-like application that supports millions of users. Explain how users upload images, follow other users, generate personalized feeds, and view posts efficiently."

Pay special attention to these five concepts:

1. Image upload using pre-signed URLs.
2. Object storage and CDN.
3. Fan-out on Write vs Fan-out on Read.
4. Hybrid feed generation.
5. Android Paging 3 and offline-first architecture.

These are particularly valuable topics for Senior Android Developer system design interviews.

# Next Lesson

Week 6 – Day 40: Food Delivery Application System Design

Topics:

- Customer, Restaurant and Delivery Partner Architecture.
- Restaurant Discovery.
- Menu Management.
- Order Management.
- Order State Machine.
- Location Tracking.
- Real-Time GPS Updates.
- Google Maps Integration.
- Payment Architecture.
- Inventory Management.
- Notifications.
- Redis Geospatial Queries.
- Android Architecture.
- WebSockets.
- Scalability and Reliability.

Roadmap Progress: Day 39 of 49.
