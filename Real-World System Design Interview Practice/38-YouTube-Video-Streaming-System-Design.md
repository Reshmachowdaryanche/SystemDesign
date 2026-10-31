# Week 6 – Day 38: YouTube / Video Streaming System Design

Day 38 Week 6 Senior Android Developer Interview Preparation

File name: `38-YouTube-Video-Streaming-System-Design.md`

Level: Beginner to Advanced Target: Senior Android Developer – 7 Years Experience

## Today's Objective

Learn how to design a scalable video streaming platform like YouTube from scratch.

We will understand how a video travels from a user's device to the backend, gets processed, stored, distributed through CDNs, and finally played on another user's Android device.

Today's topics:

- Functional and non-functional requirements.
- Capacity estimation.
- High-level architecture.
- Video upload architecture.
- Object storage.
- Video transcoding.
- Video processing pipelines.
- HLS and DASH.
- Adaptive Bitrate Streaming.
- CDN architecture.
- Video metadata and search.
- Database design.
- Caching.
- Android video player architecture.
- ExoPlayer / Media3.
- Scalability and reliability.

# 1. What Is a Video Streaming System?

A video streaming system allows users to upload, store, process, discover, and watch videos over the internet.

Examples:

- YouTube
- Netflix
- Instagram Reels
- Amazon Prime Video

YouTube cambia para siempre: así es el nuevo reproductor de vídeo y la nueva interfaz de la app en móviles y televisores

[elespanol.com](<https://www.elespanol.com/elandroidelibre/noticias-y-novedades/20251015/youtube-cambia-siempre-nuevo-reproductor-video-nueva-interfaz-app-moviles-televisores/1003743970685_0.amp.html?utm_source=chatgpt.com>)

Consider a simple scenario:

1. User A uploads a video.
2. Backend receives the video.
3. Video is stored in object storage.
4. Processing service generates different video resolutions.
5. Processed video is distributed through CDN.
6. User B opens the application.
7. User B searches for the video.
8. Android application requests the video playback URL.
9. Video player streams the content.
10. Playback analytics are collected.

Important: A video streaming platform is not simply a file upload and download application. It is a large distributed system involving storage, processing, networking, and content delivery.

# 2. Functional Requirements

Functional requirements describe what the system should do.

Our video platform should support:

1. User registration and login.
2. Video upload.
3. Video processing.
4. Video playback.
5. Video search.
6. Video recommendations.
7. Like and dislike.
8. Comments.
9. Subscribe to channels.
10. View counts.
11. Video quality selection.
12. Watch history.
13. Video sharing.
14. Video deletion.
15. Continue watching.

For today's design, focus primarily on video upload, processing, storage, and playback.

# 3. Non-Functional Requirements

| Requirement | Explanation |
| --- | --- |
| High Availability | Videos should remain accessible |
| Low Latency | Playback should start quickly |
| Scalability | Support millions of viewers |
| Reliability | Avoid video processing failures |
| Durability | Preserve uploaded videos |
| Performance | Smooth playback |
| Adaptability | Support different network speeds |
| Security | Protect private and restricted videos |

### Important Observation

Video streaming is a bandwidth-intensive system.

A single popular video can receive millions of views.

Therefore, CDN distribution and adaptive bitrate streaming are extremely important.

# 4. Capacity Estimation

Let's assume:

- 10 million daily active users.
- 1 million video uploads per day.
- Average uploaded video size: 100 MB.
- 500 million video views per day.
- Average streamed video size: 20 MB.

## 4.1 Upload Storage

text

```text
1 million × 100 MB
= 100 million MB
= approximately 100 TB/day
```

This is raw uploaded content before considering transcoded versions and replication.

## 4.2 Video Views

text

```text
500 million × 20 MB
= 10 billion MB/day
```

Approximately 10 PB of video delivery per day.

## 4.3 Average Playback Requests

text

```text
500 million / 86,400
= approximately 5,787 views/second
```

Peak traffic may be much higher.

Interview takeaway: Video streaming requires distributed object storage, processing infrastructure, and CDN-based delivery.

# 5. High-Level Architecture

YouTube System Design | System Design Interview

[systemdesignschool.io](<https://systemdesignschool.io/problems/youtube/solution?utm_source=chatgpt.com>)

Main components:

1. Android Application.
2. API Gateway.
3. Authentication Service.
4. Video Upload Service.
5. Object Storage.
6. Message Queue.
7. Video Processing Service.
8. Transcoding Workers.
9. Metadata Database.
10. CDN.
11. Search Service.
12. Recommendation Service.
13. Analytics Service.

## 5.1 Complete Architecture

text

```text
                Android Application
                        |
                        v
                   API Gateway
                        |
          +-------------+-------------+
          |                           |
          v                           v
    Video Upload Service        Video Metadata API
          |
          v
     Object Storage
          |
          v
      Message Queue
          |
          v
   Video Processing Service
          |
          v
   Transcoding Workers
          |
          v
   Processed Video Storage
          |
          v
          CDN
          |
          v
      Android Player
```

### Responsibility of Each Component

| Component | Responsibility |
| --- | --- |
| API Gateway | Routes API requests |
| Upload Service | Handles upload authorization |
| Object Storage | Stores original videos |
| Message Queue | Triggers asynchronous processing |
| Transcoding Workers | Generate multiple video formats |
| Metadata Database | Stores video information |
| CDN | Delivers videos close to users |
| Search Service | Searches videos |
| Analytics Service | Tracks playback activity |

# 6. Video Upload Architecture

Let's understand what happens when a user uploads a video.

Design YouTube - System Design Interview

[blog.algomaster.io](<https://blog.algomaster.io/p/design-youtube-system-design-interview?utm_source=chatgpt.com>)

## Step-by-Step Upload Flow

1. User selects a video.
2. Android application requests an upload session.
3. Backend authenticates the user.
4. Backend generates a secure pre-signed upload URL.
5. Android uploads the video directly to object storage.
6. Object storage confirms upload completion.
7. Backend updates upload status.
8. Backend publishes a video-processing event.
9. Processing service starts transcoding.

### Why Upload Directly to Object Storage?

Consider uploading a 500 MB video.

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

The backend must handle the entire file transfer.

With direct upload:

text

```text
Android
   |
   v
Object Storage
```

The backend only coordinates the upload.

Advantages:

- Less backend bandwidth usage.
- Better scalability.
- Reduced server load.
- Easier horizontal scaling.
- Support for multipart uploads.

### Important Concepts

- Pre-signed URLs.
- Multipart Upload.
- Upload Progress.
- Retry.
- Resumable Upload.
- Upload Validation.
- File Size Limits.
- Content-Type Validation.

# 7. Object Storage

Object storage is used to store large files such as videos, images, and documents.

Examples:

- Amazon S3.
- Google Cloud Storage.
- Azure Blob Storage.

What Is OBS?\_Service Overview\_Object Storage Service-Huawei Cloud

[support.huaweicloud.com](<https://support.huaweicloud.com/intl/en-us/productdesc-obs/en-us_topic_0045853681.html?utm_source=chatgpt.com>)

Example storage structure:

text

```text
video-storage/
    |
    +-- original/
    |      |
    |      +-- video_101.mp4
    |      +-- video_102.mp4
    |
    +-- processed/
           |
           +-- video_101/
                  |
                  +-- 360p/
                  +-- 480p/
                  +-- 720p/
                  +-- 1080p/
```

Important concepts:

- Buckets.
- Objects.
- Object Keys.
- Lifecycle Policies.
- Replication.
- Versioning.
- Access Control.
- Multipart Upload.

Interview Question: Why not store video files directly inside a relational database?

Because video files are large binary objects. Object storage is generally more suitable for large-scale file storage, while relational databases are better suited for structured metadata and relationships.

# 8. Video Transcoding

This is one of the most important topics in today's lesson.

Imagine a user uploads a video recorded at 4K resolution.

Another user watches it using a slow mobile network.

Should we always send the original 4K video?

No.

We need multiple versions of the same video.

This process is called video transcoding.

Stream Like a Pro, Secure Like a Fortress: The Simplest Guide to HLS Multi-Quality Encoding | by Paudelronish | Medium

[medium.com](<https://medium.com/%40paudelronish/stream-like-a-pro-secure-like-a-fortress-the-ultimate-guide-to-hls-multi-quality-encoding-92fb02db62e5?utm_source=chatgpt.com>)

## What Is Transcoding?

Transcoding means converting a video from one format, codec, bitrate, or resolution into another.

Example:

text

```text
Original Video: 4K
       |
       v
 Video Processing
       |
       +------> 360p
       |
       +------> 480p
       |
       +------> 720p
       |
       +------> 1080p
```

### Why Do We Need Multiple Resolutions?

| Resolution | Suitable Usage |
| --- | --- |
| 144p | Very slow network |
| 360p | Low bandwidth |
| 480p | Standard quality |
| 720p | HD quality |
| 1080p | Full HD |
| 4K | High-quality screens |

Advantages:

- Better playback experience.
- Reduced buffering.
- Support for different devices.
- Efficient bandwidth utilization.

### Important Concepts

- Video Codec.
- Bitrate.
- Resolution.
- Frame Rate.
- Encoding.
- Transcoding.
- Compression.
- FFmpeg.

# 9. Video Processing Pipeline

Video processing involves more than transcoding.

Design Video Streaming Platform (Netflix/YouTube) | sdeoffer

[sdeoffer.com](<https://sdeoffer.com/system-design/video-streaming/?utm_source=chatgpt.com>)

Typical processing steps:

1. Validate uploaded file.
2. Extract video metadata.
3. Scan uploaded content.
4. Generate thumbnails.
5. Extract audio and video streams.
6. Transcode into multiple resolutions.
7. Generate streaming manifests.
8. Store processed assets.
9. Update video metadata.
10. Mark video as READY.

Example:

text

```text
Uploaded Video
      |
      v
File Validation
      |
      v
Metadata Extraction
      |
      v
Thumbnail Generation
      |
      v
Video Transcoding
      |
      v
HLS / DASH Generation
      |
      v
Processed Storage
      |
      v
Video READY
```

### Why Use a Message Queue?

Video processing can take several minutes.

We should not make the upload API wait until processing finishes.

Instead:

text

```text
Upload Service
      |
      v
 Message Queue
      |
      v
Processing Workers
      |
      v
Transcoding
```

This allows processing workers to scale independently.

Possible technologies:

- Apache Kafka.
- RabbitMQ.
- Amazon SQS.

# 10. Adaptive Bitrate Streaming (ABR)

This is one of the most important senior-level interview topics.

Imagine:

- User A has a 100 Mbps Wi-Fi connection.
- User B has a 2 Mbps mobile connection.
- User C has an unstable network.

Should all users receive the same video quality?

No.

We use Adaptive Bitrate Streaming.

Designing a Scalable Video Streaming Platform — YouTube/Netflix Level Architecture | by Gyanaa Vaibhav | Medium

[medium.com](<https://medium.com/%40gynanrudr0/designing-a-scalable-video-streaming-platform-youtube-netflix-level-architecture-503cd149641f?utm_source=chatgpt.com>)

## What Is ABR?

Adaptive Bitrate Streaming automatically adjusts video quality based on available network bandwidth and playback conditions.

Example:

text

```text
Fast Network
     |
     v
  1080p Video

Medium Network
     |
     v
  720p Video

Slow Network
     |
     v
  360p Video
```

### How It Works

1. Video is divided into small segments.
2. Each segment is available at multiple quality levels.
3. Player measures available bandwidth and buffer state.
4. Player selects an appropriate quality.
5. Player switches quality when network conditions change.

Advantages:

- Reduced buffering.
- Better playback experience.
- Efficient bandwidth utilization.
- Automatic quality adjustment.

### Important Concepts

- Video Segmentation.
- Bitrate Estimation.
- Buffer Management.
- Quality Adaptation.
- Playback Continuity.

# 11. HLS and MPEG-DASH

These are two important streaming protocols.

## 11.1 HLS

HLS stands for HTTP Live Streaming.

Originally developed by Apple.

It uses playlists and media segments.

Typical structure:

text

```text
master.m3u8
    |
    +-- 360p.m3u8
    |
    +-- 720p.m3u8
    |
    +-- 1080p.m3u8
```

The individual playlists reference video segments.

Example:

text

```text
master.m3u8
       |
       v
  720p playlist
       |
       +-- segment001.ts
       |
       +-- segment002.ts
       |
       +-- segment003.ts
```

## 11.2 MPEG-DASH

DASH stands for Dynamic Adaptive Streaming over HTTP.

It uses an MPD manifest file.

Example:

text

```text
manifest.mpd
      |
      +-- Video Representation 360p
      |
      +-- Video Representation 720p
      |
      +-- Video Representation 1080p
```

## HLS vs DASH

| HLS | MPEG-DASH |
| --- | --- |
| Apple-developed protocol | Open international standard |
| Uses M3U8 playlists | Uses MPD manifest |
| Supports adaptive streaming | Supports adaptive streaming |
| Widely used in Apple ecosystems | Broad device and platform support |

Interview Question: Why do we divide videos into segments?

Because the player can download small portions instead of downloading the entire video.

It also allows quality switching during playback.

# 12. CDN Architecture

CDN stands for Content Delivery Network.

A CDN distributes content across geographically distributed servers.

Examples:

- Cloudflare.
- Amazon CloudFront.
- Akamai.
- Google Cloud CDN.

Best CDN for Live Streaming: Minimizing Lag for Real-Time Video

[blog.blazingcdn.com](<https://blog.blazingcdn.com/en-us/best-cdn-for-live-streaming-minimizing-lag-real-time-video?utm_source=chatgpt.com>)

## Without CDN

text

```text
User India
     |
     v
Origin Server USA
```

The user must download content from a distant server.

This can increase latency.

## With CDN

text

```text
User India
     |
     v
Nearby CDN Edge
     |
     v
Cached Video Segments
```

If the content is unavailable in the edge cache, the CDN retrieves it from the origin.

### CDN Advantages

- Reduced latency.
- Lower origin server load.
- Better scalability.
- Improved playback performance.
- Reduced bandwidth costs at the origin.

### Important Concepts

- Edge Locations.
- Origin Server.
- Cache Hit Ratio.
- Cache Miss.
- Cache Invalidation.
- TTL.
- Regional Distribution.

Senior-level Question: What happens when a video is uploaded but not available in a CDN edge location?

The CDN retrieves the requested content from the origin storage or origin service, caches it according to the caching policy, and serves subsequent requests from the edge.

# 13. Video Playback Architecture

Let's understand how an Android application plays a video.

How I Built a Real-Time AI Streaming CDN , Adaptive Bitrate, and Live Object Detection | by Himanshu Raghav | Mar, 2026 | Medium

[medium.com](<https://medium.com/%40hsinghraghav1/how-i-built-a-real-time-ai-streaming-cdn-adaptive-bitrate-and-live-object-detection-4abb82af0fa4?utm_source=chatgpt.com>)

## Playback Flow

1. User opens a video.
2. Android requests video metadata.
3. Backend returns video information and playback manifest URL.
4. Player requests the manifest.
5. Player selects a suitable bitrate.
6. Player downloads video segments from CDN.
7. Player buffers the segments.
8. Video playback begins.
9. Player adjusts quality according to network conditions.

Example API response:

json

```json
{
  "videoId": "video_101",
  "title": "Android System Design",
  "status": "READY",
  "playbackUrl": "https://cdn.example.com/video_101/master.m3u8",
  "duration": 1200
}
```

Important concepts:

- Manifest.
- Video Segments.
- Buffering.
- Playback Position.
- Adaptive Bitrate.
- Seeking.
- Preloading.

# 14. Android Developer Perspective – ExoPlayer / Media3

This section is especially important for your Senior Android Developer interviews.

Modern Android video playback should generally use AndroidX Media3.

The ExoPlayer implementation is provided through Media3.

## 14.1 Android Playback Architecture

text

```text
             Jetpack Compose UI
                     |
                     v
                  ViewModel
                     |
                     v
               Video Repository
                     |
                     v
               Playback Manager
                     |
                     v
              Media3 ExoPlayer
                     |
                     v
                HLS / DASH
                     |
                     v
                    CDN
```

## 14.2 Important Media3 Components

| Component | Responsibility |
| --- | --- |
| ExoPlayer | Video playback engine |
| MediaItem | Represents media content |
| MediaSource | Provides media data |
| TrackSelector | Selects video/audio tracks |
| LoadControl | Manages buffering |
| PlayerView | Displays video |
| MediaSession | Integrates playback with system controls |

## 14.3 Basic ExoPlayer Example

Dependency:

kotlin

```kotlin
implementation("androidx.media3:media3-exoplayer:<version>")
implementation("androidx.media3:media3-ui:<version>")
```

Create Player:

kotlin

```kotlin
val player = ExoPlayer.Builder(context)
    .build()
```

Create Media Item:

kotlin

```kotlin
val mediaItem = MediaItem.fromUri(
    "https://cdn.example.com/video/master.m3u8"
)
```

Set Media:

kotlin

```kotlin
player.setMediaItem(mediaItem)
player.prepare()
player.play()
```

Release Player:

kotlin

```kotlin
player.release()
```

### Important Android Interview Topics

1. How does ExoPlayer work internally?
2. What is MediaItem?
3. What is MediaSource?
4. What is TrackSelector?
5. What is LoadControl?
6. How does buffering work?
7. How does adaptive bitrate selection work?
8. How do you handle playback lifecycle?
9. How do you handle audio focus?
10. How do you implement background playback?

# 15. Buffering and Playback Optimization

Imagine a user watches a video and sees a loading spinner every few seconds.

This is buffering.

How Video Streaming Actually Works: Why YouTube Starts Playing in 3 Seconds (System Design Explained) | Towards AI

[towardsai.net](<https://towardsai.net/p/machine-learning/how-video-streaming-actually-works-why-youtube-starts-playing-in-3-seconds-system-design-explained?utm_source=chatgpt.com>)

## What Is Buffering?

Buffering means temporarily downloading video data before or during playback.

Example:

text

```text
Downloaded Segments:

[Segment 1] [Segment 2] [Segment 3] [Segment 4]
      |
      v
   Playback
```

The player tries to maintain enough buffered content to continue playback smoothly.

### Buffering Optimization

- Adaptive Bitrate Streaming.
- CDN Distribution.
- Segment Preloading.
- Network Optimization.
- Appropriate Buffer Configuration.
- Video Compression.
- Playback Startup Optimization.

### Important Metrics

| Metric | Meaning |
| --- | --- |
| Startup Time | Time taken to start playback |
| Rebuffering Ratio | Time spent buffering during playback |
| Bitrate | Video data rate |
| Playback Failure Rate | Failed playback attempts |
| Watch Time | Total viewing duration |
| Completion Rate | Percentage of videos watched |

# 16. Video Metadata and Search Architecture

Video metadata contains information about the video.

Example:

| Field | Description |
| --- | --- |
| video\_id | Unique video identifier |
| user\_id | Owner |
| title | Video title |
| description | Video description |
| thumbnail\_url | Thumbnail location |
| duration | Video duration |
| resolution | Available resolutions |
| status | Processing status |
| created\_at | Upload timestamp |

## Search Architecture

Video semantic search with AI on AWS | AWS for M&E Blog

[aws.amazon.com](<https://aws.amazon.com/blogs/media/video-semantic-search-with-ai-on-aws/?utm_source=chatgpt.com>)

Flow:

text

```text
Android App
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
Android UI
```

Important concepts:

- Full-Text Search.
- Inverted Index.
- Search Ranking.
- Filtering.
- Pagination.
- Search Suggestions.
- Metadata Indexing.

Interview Question: Why not search video titles directly in the main database for every request?

A dedicated search engine can provide better full-text search, relevance ranking, and search scalability.

# 17. Database Design

A video platform requires multiple data entities.

## 17.1 Users Table

| Column | Type |
| --- | --- |
| user\_id | BIGINT |
| username | VARCHAR |
| email | VARCHAR |
| created\_at | TIMESTAMP |

## 17.2 Videos Table

| Column | Type |
| --- | --- |
| video\_id | BIGINT |
| user\_id | BIGINT |
| title | VARCHAR |
| description | TEXT |
| storage\_key | VARCHAR |
| status | VARCHAR |
| duration | BIGINT |
| created\_at | TIMESTAMP |

## 17.3 Video Variants Table

| Column | Type |
| --- | --- |
| variant\_id | BIGINT |
| video\_id | BIGINT |
| resolution | VARCHAR |
| bitrate | BIGINT |
| storage\_key | VARCHAR |

## 17.4 Video Views Table

| Column | Type |
| --- | --- |
| view\_id | BIGINT |
| video\_id | BIGINT |
| user\_id | BIGINT |
| watched\_duration | BIGINT |
| created\_at | TIMESTAMP |

### Important Database Concepts

- Indexing.
- Partitioning.
- Sharding.
- Replication.
- Read Replicas.
- Denormalization.
- Eventual Consistency.

Important: Store video binary content in object storage, not directly in the metadata database.

# 18. Analytics Architecture

A large video platform needs to collect playback events.

Examples:

- Video started.
- Video paused.
- Video resumed.
- Video completed.
- Video skipped.
- Playback failed.
- Quality changed.

Processing User Activity Streams: Click Tracking at Scale | by Nitesh Thakur | Feb, 2026 | Medium

[medium.com](<https://medium.com/%40niteshthakur498/processing-user-activity-streams-click-tracking-at-scale-82e0e7ae0f06?utm_source=chatgpt.com>)

Architecture:

text

```text
Android Video Player
        |
        v
  Playback Events
        |
        v
   Analytics API
        |
        v
    Message Queue
        |
        v
  Analytics Workers
        |
        v
  Data Warehouse
```

Advantages:

- Asynchronous processing.
- Independent scaling.
- Better analytics.
- Reduced playback service load.

Important concepts:

- Event Tracking.
- Event Batching.
- Kafka.
- Data Warehouse.
- Real-Time Analytics.
- Idempotency.

# 19. Reliability and Failure Handling

What happens if video processing fails?

Possible failures:

- Upload interrupted.
- Storage unavailable.
- Transcoding worker crashes.
- CDN origin unavailable.
- Invalid video format.
- Network interruption during playback.

Possible solutions:

| Failure | Solution |
| --- | --- |
| Upload interruption | Resumable multipart upload |
| Processing failure | Retry and dead-letter queue |
| Worker crash | Requeue processing task |
| CDN failure | Origin fallback or alternate CDN |
| Playback network failure | Retry and adaptive bitrate |
| Invalid media | Validation and failure status |
| Database failure | Replication and recovery |

Important concepts:

- Retry.
- Exponential Backoff.
- Dead-Letter Queue.
- Idempotent Processing.
- Circuit Breaker.
- High Availability.
- Disaster Recovery.

# 20. Scalability Challenges

How would we handle millions of concurrent video viewers?

| Problem | Solution |
| --- | --- |
| Large video uploads | Direct object storage upload |
| Slow video processing | Distributed transcoding workers |
| High bandwidth | CDN |
| Popular video | Edge caching |
| Database bottleneck | Sharding and replication |
| Search bottleneck | Dedicated search service |
| Playback analytics load | Asynchronous events |
| High concurrent viewers | Distributed delivery infrastructure |

### Advanced Concepts

- Multi-CDN Architecture.
- Multi-Region Storage.
- Content Replication.
- Adaptive Bitrate.
- Video Segment Caching.
- Distributed Transcoding.
- Backpressure.
- Cost Optimization.

# 21. Senior-Level Interview Questions

Try answering these without looking at the notes.

## Fundamental Questions

1. Design a YouTube-like video streaming platform.
2. What are the functional and non-functional requirements?
3. Why do we need object storage?
4. What is video transcoding?
5. Why do we generate multiple video resolutions?

## Streaming and CDN

6. What is Adaptive Bitrate Streaming?
7. Explain HLS and DASH.
8. What is the difference between HLS and DASH?
9. Why do we need CDN?
10. What happens during a CDN cache miss?
11. What is video segmentation?
12. How does buffering work?

## Backend and Distributed Systems

13. Why should video processing be asynchronous?
14. How do you design a video processing pipeline?
15. How do you handle failed transcoding jobs?
16. How do you store video metadata?
17. How do you scale video delivery?
18. How would you design video search?
19. How do you collect playback analytics?

## Android-Specific Questions

20. What is ExoPlayer?
21. What is AndroidX Media3?
22. What is MediaItem?
23. What is TrackSelector?
24. What is LoadControl?
25. How does adaptive bitrate selection work?
26. How do you manage player lifecycle?
27. How do you handle playback when an Activity is recreated?
28. How do you implement background playback?
29. How do you optimize video startup time?
30. How do you handle network changes during playback?

# 22. Practical Interview Assignment

Design a YouTube-like video streaming platform without looking at the notes.

Your design must include:

- [ ]Functional Requirements.
- [ ]Non-Functional Requirements.
- [ ]Capacity Estimation.
- [ ]High-Level Architecture.
- [ ]Video Upload Architecture.
- [ ]Object Storage.
- [ ]Video Transcoding.
- [ ]Video Processing Pipeline.
- [ ]Adaptive Bitrate Streaming.
- [ ]HLS and DASH.
- [ ]CDN Architecture.
- [ ]Video Playback Flow.
- [ ]Database Schema.
- [ ]Search Architecture.
- [ ]Analytics Architecture.
- [ ]Failure Handling.
- [ ]Android Media3 Architecture.
- [ ]Buffering Optimization.
- [ ]Scalability Strategy.

# 23. Day 38 Completion Checklist

- [ ]Understand functional and non-functional requirements.
- [ ]Complete capacity estimation.
- [ ]Draw high-level architecture.
- [ ]Understand video upload architecture.
- [ ]Understand object storage.
- [ ]Explain video transcoding.
- [ ]Understand video processing pipelines.
- [ ]Explain Adaptive Bitrate Streaming.
- [ ]Understand HLS and DASH.
- [ ]Explain CDN architecture.
- [ ]Understand video playback flow.
- [ ]Understand Android Media3 and ExoPlayer.
- [ ]Explain buffering optimization.
- [ ]Design metadata and search architecture.
- [ ]Understand playback analytics.
- [ ]Explain reliability and failure handling.
- [ ]Answer all 30 interview questions.

# 24. What You Should Be Able to Explain After Day 38

You should be able to answer this interview question confidently:

"Design a YouTube-like video streaming application that supports millions of users. Explain how videos are uploaded, processed, stored, distributed, and played on Android devices."

You should also be able to explain:

- Why object storage is required.
- Why transcoding is necessary.
- Why multiple resolutions are generated.
- How HLS and DASH work.
- How CDN reduces latency.
- How adaptive bitrate streaming works.
- How Android Media3 plays streaming videos.

Don't just memorize architecture diagrams. Explain the reason behind every component and the trade-offs involved.

# Next Lesson

Week 6 – Day 39: Instagram System Design

Topics:

- User Profiles.
- Image Upload Architecture.
- Image Processing.
- Feed Generation.
- Push Notifications.
- Following and Followers.
- Redis Caching.
- CDN.
- Fan-out on Write vs Fan-out on Read.
- Database Design.
- Android Feed Architecture.
- Pagination and Infinite Scrolling.
- Scalability and Performance.

Roadmap Progress: Day 38 of 49.
