# Week 3 – Day 18: File Storage and Object Storage

Week 3Day 18Beginner level

File name: `Week-3-Day-18-File-Storage-and-Object-Storage.md`

Today's goal: Understand how applications store images, videos, documents and other files, how cloud storage works, and how Android applications upload and download large files efficiently.

Writing

# Week 3 – Day 18: File Storage and Object Storage

**Topic:** File Storage, Block Storage and Object Storage\
**Level:** Beginner\
**Target:** Android Developer – 7 Years Experience

---

# 1\. Introduction

In Day 17, we learned about Redis, distributed caching and CDN.

Today, we will understand something very important in system design.

**Where do applications store images, videos, PDFs and other files?**

Imagine you are developing a Notes application.

Users can:

- Create text notes.
- Attach images.
- Upload PDFs.
- Record audio notes.
- Attach videos.

Where should we store these files?

Should we store images directly inside a database?

Should we store them in Redis?

Should we create a separate file server?

Should we use cloud storage like Amazon S3?

These are the questions we will answer today.

Topics covered:

1. What is file storage?
2. File storage vs database storage.
3. Block storage.
4. File storage.
5. Object storage.
6. What is Amazon S3?
7. Database vs object storage.
8. How image uploads work.
9. Pre-signed URLs.
10. Multipart uploads.
11. File download architecture.
12. CDN and object storage.
13. File metadata.
14. Real-world WhatsApp/Instagram-style media storage.

---

# 2\. What Is File Storage?

File storage means storing digital files so applications can save, retrieve, update and delete them.

Examples:

- Images: JPG, PNG, WebP.
- Videos: MP4.
- Documents: PDF, DOCX.
- Audio: MP3, WAV.
- Other files: ZIP, CSV, APK.

Imagine a user uploads a profile picture.

The application needs to store that picture somewhere.

Later, when the user opens their profile, the application should retrieve the same picture.

This is file storage.

## 2.1 Why Do We Need Separate File Storage?

Consider a social media application.

It has:

- 10 million users.
- 50 million images.
- 5 million videos.
- Millions of document uploads.

Storing all these files directly inside a relational database can become expensive and difficult to manage.

Files are usually stored in dedicated storage systems.

The database stores information about the files.

For example:

**Database record:**

json

```json
{
  "userId": 101,
  "profileImage": "https://cdn.example.com/profile/101.jpg"
}
```

**Actual image:**

text

```text
profile/101.jpg
```

The image itself is stored in object storage.

The database stores its reference and related information.

This separation is very common in real-world applications.

---

# 3\. Three Types of Storage

In system design, you should understand three important storage types.

1. Block Storage.
2. File Storage.
3. Object Storage.

Let's understand them one by one.

---

# 4\. What Is Block Storage?

Block storage divides data into fixed-size blocks and stores those blocks separately.

The operating system sees this storage as a disk or volume.

Applications and file systems can read and write data at block level.

Think of it like a hard disk attached to a server.

## 4.1 Simple Example

Imagine you have a 100 GB storage disk.

The storage system divides the disk into smaller blocks.

text

```text
       BLOCK STORAGE

    +----------------+
    |    Block 1     |
    +----------------+
    |    Block 2     |
    +----------------+
    |    Block 3     |
    +----------------+
    |    Block 4     |
    +----------------+
    |    Block 5     |
    +----------------+
```

The operating system manages how files and directories are organized on top of this storage.

## 4.2 Real-World Example

Suppose your backend application needs a high-performance disk.

You can attach a block storage volume to your server.

Examples:

- Amazon EBS.
- Azure Managed Disks.
- Google Persistent Disk.

## 4.3 When Do We Use Block Storage?

Useful for:

- Operating system disks.
- Database storage volumes.
- Applications requiring low-latency disk access.
- Workloads requiring direct block-level storage access.

## 4.4 Advantages

- Good performance.
- Suitable for databases.
- Supports file systems on top of the volume.
- Can be attached to compute instances depending on the storage service.

## 4.5 Disadvantages

- Usually requires more infrastructure management.
- Not designed as a simple public file-sharing service.
- Scaling and sharing require additional configuration.

---

# 5\. What Is File Storage?

File storage organizes data into files and directories.

It provides a familiar folder structure.

Example:

text

```text
storage/
    users/
        user101/
            profile.jpg
            resume.pdf

        user102/
            profile.jpg

    notes/
        note101/
            image1.png
            document.pdf
```

You can access files using paths.

Example:

text

```text
/users/user101/profile.jpg
```

## 5.1 How Does It Work?

The file system manages:

- Files.
- Directories.
- File names.
- Permissions.
- File metadata.

Applications access files through the file system interface.

## 5.2 Real-World Examples

- Network Attached Storage (NAS).
- Amazon EFS.
- Azure Files.
- Google Filestore.

## 5.3 When Do We Use File Storage?

Useful for:

- Shared folders.
- Enterprise file sharing.
- Applications requiring directory structures.
- Multiple servers accessing shared files.

## 5.4 Advantages

- Easy to understand.
- Familiar directory structure.
- Easy file organization.
- Supports shared access depending on configuration.

## 5.5 Disadvantages

- Managing very large numbers of files can become challenging.
- Scaling requires storage infrastructure planning.
- File-system semantics may introduce limitations for distributed workloads.

---

# 6\. What Is Object Storage?

Object storage stores data as independent objects.

Each object contains:

1. Actual data.
2. Metadata.
3. Unique identifier or key.

Unlike traditional file storage, object storage does not primarily organize data through nested directories.

It uses buckets and object keys.

## 6.1 Simple Example

Imagine a Notes application.

A user uploads an image.

Object storage stores it like this:

text

```text
BUCKET: notes-app

OBJECT KEY:
users/101/profile.jpg

OBJECT DATA:
Actual image file

METADATA:
Content-Type: image/jpeg
Size: 2 MB
```

Here:

- Bucket is the storage container.
- Object key identifies the object.
- Object data contains the actual file.
- Metadata contains additional information.

## 6.2 Architecture

text

```text
             APPLICATION
                  |
                  v
             OBJECT STORAGE
                  |
          +-------+-------+
          |       |       |
          v       v       v
        IMAGE   VIDEO   DOCUMENT
```

## 6.3 Real-World Examples

- Amazon S3.
- Google Cloud Storage.
- Azure Blob Storage.
- Cloudflare R2.

## 6.4 When Do We Use Object Storage?

Useful for:

- Profile pictures.
- Social media images.
- Videos.
- PDF documents.
- Backups.
- Logs.
- Large application attachments.

## 6.5 Advantages

- Designed for large amounts of unstructured data.
- Highly scalable.
- Durable storage options.
- Accessible through APIs.
- Supports metadata and access-control policies.

## 6.6 Disadvantages

- Not a traditional file system.
- Access typically happens through APIs or HTTP.
- Updating part of an object is generally not like editing a block on a disk.
- Performance and cost depend on storage class, region and access pattern.

---

# 7\. Block vs File vs Object Storage

This is an important interview question.

| Feature | Block Storage | File Storage | Object Storage |
| --- | --- | --- | --- |
| Data organization | Blocks | Files and folders | Objects |
| Access | Block-level | File-system interface | API/HTTP |
| Structure | Disk/volume | Directory hierarchy | Bucket and object key |
| Common usage | Database disks | Shared folders | Images and videos |
| Scaling | Volume-based | File-system-based | Designed for large-scale object storage |
| Example | Amazon EBS | Amazon EFS | Amazon S3 |

### Easy Way to Remember

- Block storage = Hard disk.
- File storage = Folder and files.
- Object storage = Bucket and objects.

### Interview Answer

Block storage stores data in blocks and is commonly used for database disks.

File storage organizes data into files and folders.

Object storage stores data as objects with unique keys and metadata. It is commonly used for images, videos and documents.

---

# 8\. What Is Amazon S3?

Amazon S3 stands for Amazon Simple Storage Service.

It is an object storage service provided by AWS.

It allows applications to store and retrieve large amounts of data.

For system design interviews, understand S3 as a place where applications can store files independently of their backend servers.

## 8.1 S3 Basic Concepts

There are four important terms.

### 1\. Bucket

A bucket is a container that stores objects.

Example:

text

```text
notes-app-bucket
```

### 2\. Object

An object is the actual stored data.

Example:

text

```text
profile.jpg
```

### 3\. Object Key

The object key identifies an object inside a bucket.

Example:

text

```text
users/101/profile.jpg
```

### 4\. Metadata

Metadata describes the object.

Example:

text

```text
Content-Type: image/jpeg
Content-Length: 2097152
```

## 8.2 S3 Architecture

text

```text
              BACKEND SERVER
                    |
                    v
                S3 BUCKET
                    |
          +---------+---------+
          |         |         |
          v         v         v
       IMAGES    VIDEOS    DOCUMENTS
```

## 8.3 Why Use S3 Instead of a Backend Server?

Imagine you have three backend servers.

A user uploads an image to Server A.

If the image is stored only on Server A's local disk, what happens when another request reaches Server B?

Server B may not have access to that image.

This creates a problem.

Instead, store the image in shared object storage.

text

```text
           ANDROID APP
                |
                v
          LOAD BALANCER
                |
         +------+------+
         |             |
         v             v
      SERVER A      SERVER B
         |             |
         +------+------+
                |
                v
            AMAZON S3
```

Now both backend servers can access the same stored object through the configured storage API.

## 8.4 Advantages of S3

- Highly scalable object storage.
- High durability options.
- Independent from backend server disks.
- Supports access-control policies.
- Supports lifecycle policies.
- Supports versioning.
- Integrates with CDN services.

---

# 9\. Database vs Object Storage

This is another important interview question.

Should we store the actual image inside the database or store it in S3?

Let's compare.

## 9.1 Storing Image in Database

A database can store image data using binary data types such as BLOB.

Example:

sql

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    profile_image BLOB
);
```

The image bytes are stored directly in the database.

## 9.2 Storing Image in Object Storage

Instead of storing the image in the database:

1. Upload image to S3.
2. Get its object key.
3. Store the object key in the database.

Example:

sql

```sql
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100),
    profile_image_key VARCHAR(500)
);
```

Database record:

text

```text
id: 101

name: John

profile_image_key:
users/101/profile.jpg
```

Actual image:

text

```text
S3 BUCKET:
users/101/profile.jpg
```

## 9.3 Comparison

| Database | Object Storage |
| --- | --- |
| Stores structured records | Stores files and objects |
| Can store binary files | Designed for large file storage |
| Database size grows with file content | File storage scales independently |
| Database queries manage records | Object APIs manage files |
| Useful for metadata and relationships | Useful for actual images and videos |

### Recommended Approach

For most applications:

- Store file metadata in the database.
- Store actual files in object storage.
- Store the object key or reference in the database.

Important: This is a common architectural choice, not a universal rule. Small files or special transactional requirements may justify storing binary data in a database.

---

# 10\. How Does Image Upload Work?

Now let's design an actual image upload flow.

Imagine a user selects a profile picture in your Android application.

How does the image reach S3?

There are two common approaches.

1. Upload through backend.
2. Direct upload to object storage using a pre-signed URL.

Let's understand both.

## 10.1 Approach 1: Upload Through Backend

Architecture:

text

```text
           ANDROID APP
                |
                | Upload Image
                v
           BACKEND API
                |
                | Upload Image
                v
             AMAZON S3
                |
                v
           RETURN RESULT
```

### Step-by-Step Flow

**Step 1:** User selects an image.

**Step 2:** Android sends the image to backend.

http

```http
POST /api/profile/image
Content-Type: multipart/form-data
```

**Step 3:** Backend validates:

- User authentication.
- File type.
- File size.
- Upload permissions.

**Step 4:** Backend uploads the file to S3.

**Step 5:** S3 stores the image.

**Step 6:** Backend stores the object key in the database.

**Step 7:** Backend returns success to Android.

### Advantages

- Backend controls the entire upload process.
- Easy to implement authorization and validation.
- Client does not need direct storage permissions.

### Disadvantages

- Backend handles large file traffic.
- More bandwidth and processing load on backend.
- Large uploads can consume server resources.

---

# 11\. What Is a Pre-Signed URL?

A pre-signed URL is a temporary URL that grants permission to perform a specific operation on a storage object.

For example:

A backend generates a temporary URL that allows Android to upload an image directly to S3.

Android does not need permanent AWS credentials.

The URL contains time-limited authorization information.

## 11.1 Why Do We Need It?

Imagine 100,000 users upload images every day.

If every image passes through backend servers:

text

```text
ANDROID
   |
   v
BACKEND
   |
   v
S3
```

The backend handles all image-upload traffic.

Instead, we can let Android upload directly to S3 using a pre-signed URL.

text

```text
ANDROID APP
     |
     | Request upload URL
     v
BACKEND
     |
     | Generate temporary URL
     v
ANDROID APP
     |
     | Upload directly
     v
    S3
```

The backend is no longer required to proxy the entire file.

## 11.2 Step-by-Step Flow

**Step 1: Android requests an upload URL.**

http

```http
POST /api/uploads/presigned-url
```

**Step 2: Backend authenticates the user.**

**Step 3: Backend generates a pre-signed URL.**

The URL grants temporary permission to upload a specific object.

**Step 4: Backend returns the URL.**

Example response:

json

```json
{
  "uploadUrl": "https://storage.example.com/temporary-signed-url",
  "objectKey": "users/101/profile-v2.jpg"
}
```

This is illustrative. The actual URL contains signed authorization parameters.

**Step 5: Android uploads directly to S3.**

http

```http
PUT <pre-signed-url>
Content-Type: image/jpeg
```

The file bytes are sent in the request body.

**Step 6: S3 stores the file.**

**Step 7: Android informs backend that upload completed.**

http

```http
POST /api/uploads/complete
```

The backend verifies the uploaded object and stores its reference in the database.

## 11.3 Advantages

- Reduces backend bandwidth.
- Supports large file uploads.
- Backend does not need to proxy the file.
- Temporary authorization.
- Useful for scalable upload systems.

## 11.4 Disadvantages

- More complicated upload flow.
- URL can be misused until it expires or is otherwise invalidated.
- Backend must verify upload completion.
- File validation and ownership checks are important.
- A successful URL generation does not mean the upload succeeded.

## 11.5 Important Security Points

- Keep expiration short.
- Generate URLs only after authentication and authorization.
- Restrict the object key and permitted operation.
- Validate uploaded content.
- Avoid exposing permanent cloud credentials in Android.
- Use appropriate upload size limits.
- Do not trust the client-provided content type alone.

---

# 12\. What Is Multipart Upload?

Imagine a user uploads a 2 GB video.

Uploading the entire video as one large request can be difficult.

If the network disconnects near the end, the upload may need to restart.

Multipart upload solves this problem.

## 12.1 Definition

Multipart upload divides a large file into smaller parts.

Each part is uploaded separately.

After all parts are successfully uploaded, the storage service combines them into the final object.

## 12.2 Example

Suppose the video size is 2 GB.

We divide it into parts.

text

```text
           VIDEO: 2 GB

        +-------------+
        |   PART 1    |
        |   100 MB    |
        +-------------+
        |   PART 2    |
        |   100 MB    |
        +-------------+
        |   PART 3    |
        |   100 MB    |
        +-------------+
        |     ...     |
        +-------------+
        |   PART 20   |
        |   100 MB    |
        +-------------+
```

Each part is uploaded independently.

After successful uploads, the storage service assembles the parts.

## 12.3 Architecture

text

```text
              ANDROID APP
                   |
                   v
             BACKEND API
                   |
                   v
           INITIATE MULTIPART
                   |
                   v
               S3 UPLOAD
                   |
        +----------+----------+
        |          |          |
        v          v          v
      PART 1     PART 2     PART 3
        |          |          |
        +----------+----------+
                   |
                   v
              COMPLETE
                   |
                   v
              FINAL FILE
```

## 12.4 Advantages

- Supports large file uploads.
- Failed parts can be retried independently.
- Supports parallel uploads.
- Avoids restarting the entire upload.
- Useful for unstable networks.

## 12.5 Disadvantages

- More complex implementation.
- Must track uploaded parts.
- Incomplete uploads consume storage until cleaned up.
- Requires upload completion and abort handling.

## 12.6 Real-World Usage

Useful for:

- Large video uploads.
- Cloud backup.
- Large document uploads.
- Media-heavy applications.

---

# 13\. What Is Resumable Upload?

Resumable upload means continuing an interrupted upload instead of starting again from zero.

Imagine:

1. User uploads a 500 MB video.
2. 300 MB is successfully uploaded.
3. Internet connection disconnects.
4. User reconnects.

Without resumable upload, the application might need to upload all 500 MB again.

With resumable upload, it can continue from the previously completed portion.

Multipart upload is one way to implement resumable uploads.

However, resumable upload and multipart upload are not exactly the same concept.

- Multipart upload divides data into parts.
- Resumable upload allows an interrupted transfer to continue.

---

# 14\. What Is File Metadata?

Metadata means information about a file.

It describes the file rather than containing the actual file content.

Example:

json

```json
{
  "fileId": 501,
  "fileName": "profile.jpg",
  "contentType": "image/jpeg",
  "fileSize": 2097152,
  "uploadedBy": 101,
  "uploadedAt": "2026-10-31T10:00:00Z",
  "objectKey": "users/101/profile.jpg"
}
```

Metadata can be stored in a relational database.

## 14.1 Why Do We Need Metadata?

Imagine a user has uploaded 100 files.

They want to see:

- File name.
- File type.
- File size.
- Upload date.
- Owner.
- File status.

We can retrieve this information from the database without downloading every actual file.

## 14.2 Example Database Table

sql

```sql
CREATE TABLE files (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    file_name VARCHAR(255),
    object_key VARCHAR(500),
    content_type VARCHAR(100),
    file_size BIGINT,
    created_at TIMESTAMP
);
```

The actual file remains in object storage.

---

# 15\. What Is a CDN + Object Storage Architecture?

In Day 17, we learned about CDN.

Today, we learned about object storage.

These two technologies are frequently used together.

Imagine a user uploads an image to S3.

Other users need to download it.

Without CDN:

text

```text
ANDROID APP
     |
     v
AMAZON S3
     |
     v
DOWNLOAD IMAGE
```

With CDN:

text

```text
              ANDROID APP
                   |
                   v
                CDN URL
                   |
                   v
              CDN EDGE
                   |
               +---+---+
               |       |
              HIT     MISS
               |       |
               v       v
             RETURN  AMAZON S3
                       |
                       v
                   STORE COPY
                       |
                       v
                     RETURN
```

## 15.1 Why Combine Them?

Object storage provides durable storage for original files.

CDN provides geographically distributed delivery.

Benefits:

- Reduced load on origin storage.
- Faster image delivery for geographically distributed users.
- Better scalability for media-heavy applications.
- Reduced repeated origin requests.

## 15.2 Example

Original image:

text

```text
S3:
notes-app/users/101/profile.jpg
```

Public delivery URL:

text

```text
https://cdn.example.com/users/101/profile.jpg
```

Android loads the image using the CDN URL.

The actual storage location remains separate from the public delivery URL.

For private files, CDN access should use suitable authorization mechanisms rather than making the objects public.

---

# 16\. How Does WhatsApp-Style Media Upload Work?

Let's understand a simplified media-sharing system.

A user sends an image to another user.

## Requirements

1. User selects an image.
2. Image is uploaded.
3. Recipient receives a message.
4. Recipient can download the image.
5. The image should not pass through every backend server unnecessarily.

## Architecture

text

```text
             SENDER ANDROID
                   |
                   v
               BACKEND API
                   |
                   v
             GENERATE UPLOAD URL
                   |
                   v
               SENDER ANDROID
                   |
                   v
              OBJECT STORAGE
                   |
                   v
              SAVE FILE METADATA
                   |
                   v
             SAVE MESSAGE RECORD
                   |
                   v
             NOTIFY RECIPIENT
                   |
                   v
             RECIPIENT ANDROID
                   |
                   v
                  CDN
                   |
                   v
             OBJECT STORAGE
```

## Step-by-Step

1. Sender selects an image.
2. Android requests an upload URL.
3. Backend authenticates the sender.
4. Backend generates a temporary upload URL.
5. Android uploads the image to object storage.
6. Backend verifies the upload.
7. Backend stores file metadata.
8. Backend stores the message record with the attachment reference.
9. Recipient receives the message.
10. Recipient downloads the image through the authorized delivery path.

Important:

This is a simplified architecture. Real messaging systems also need authorization, encryption, retention policies, delivery guarantees and media lifecycle management.

---

# 17\. How Does Instagram-Style Image Upload Work?

Imagine a user uploads a photo.

The application needs to:

- Accept the upload.
- Store the original.
- Generate optimized versions.
- Deliver the image quickly.
- Support millions of users.

A simplified architecture:

text

```text
             ANDROID APP
                  |
                  v
              BACKEND API
                  |
                  v
           GENERATE UPLOAD URL
                  |
                  v
             OBJECT STORAGE
                  |
                  v
           IMAGE PROCESSING
                  |
         +--------+--------+
         |        |        |
         v        v        v
       SMALL   MEDIUM    LARGE
       IMAGE   IMAGE     IMAGE
         |        |        |
         +--------+--------+
                  |
                  v
                 CDN
                  |
                  v
             ANDROID USERS
```

## Why Generate Multiple Sizes?

A small thumbnail does not need the same resolution as a full-screen image.

For example:

- Thumbnail: 150 × 150.
- Feed image: 600 × 600.
- Full image: 1200 × 1200.

The Android application can request an appropriate version based on the screen and use case.

This reduces unnecessary data transfer.

---

# 18\. Important File Storage Concepts

## 18.1 Storage Lifecycle

Object storage providers often support lifecycle policies.

A lifecycle policy automatically moves or deletes objects according to configured rules.

Example:

text

```text
DAY 0:
Store in standard storage.

DAY 30:
Move to a lower-cost storage class.

DAY 180:
Move to archival storage.

DAY 365:
Delete if retention policy permits.
```

Useful for:

- Backups.
- Old logs.
- Archived documents.
- Inactive media.

## 18.2 Object Versioning

Object versioning maintains multiple versions of an object.

Example:

text

```text
profile.jpg

Version 1: Old profile picture
Version 2: Updated profile picture
Version 3: Latest profile picture
```

Useful when users accidentally overwrite files or when recovery is required.

Versioning can increase storage costs, so lifecycle policies may be needed.

## 18.3 Storage Durability vs Availability

These two terms are different.

**Durability:** How reliably data is protected against permanent loss.

**Availability:** How reliably the system can provide access when requested.

A storage system can have high durability while still experiencing temporary availability issues.

This distinction is important in interviews.

## 18.4 Hot vs Cold Storage

Hot storage:

Data accessed frequently.

Example: Recent profile pictures.

Cold storage:

Data accessed rarely.

Example: Old archived documents.

Different storage classes may have different costs and access characteristics.

---

# 19\. Android Developer Perspective

As an Android developer with backend system design knowledge, understand where Android fits into file storage architecture.

Android responsibilities:

1. Let the user select a file.
2. Validate basic file properties.
3. Request upload authorization from backend.
4. Upload the file.
5. Handle progress.
6. Handle network failures.
7. Retry or resume when supported.
8. Inform backend about upload completion.
9. Display the uploaded file using an appropriate URL.

Backend responsibilities:

1. Authenticate the user.
2. Authorize the upload.
3. Generate a pre-signed URL.
4. Validate upload completion.
5. Store file metadata.
6. Manage object ownership and access.
7. Provide authorized download references.

Object storage responsibilities:

1. Store the actual file.
2. Provide object access.
3. Maintain configured durability and availability.
4. Support multipart uploads where applicable.

CDN responsibilities:

1. Cache eligible content.
2. Serve content from edge locations.
3. Reduce origin traffic.
4. Apply caching and access policies.

---

# 20\. Common Interview Mistakes

1. Storing every image directly in the relational database without considering scale.
2. Storing uploaded files only on one backend server's local disk.
3. Confusing file storage with object storage.
4. Confusing block storage with object storage.
5. Thinking a pre-signed URL is a permanent public URL.
6. Assuming generating a pre-signed URL means the upload is complete.
7. Forgetting to save file metadata.
8. Forgetting to validate file types and sizes.
9. Exposing permanent cloud credentials in Android.
10. Forgetting failed and incomplete uploads.
11. Ignoring CDN caching.
12. Making private user documents publicly accessible.
13. Ignoring file deletion and lifecycle policies.
14. Forgetting authorization during downloads.

---

# 21\. Important Interview Questions and Answers

### Q1. What is object storage?

Object storage stores data as objects containing data, metadata and unique identifiers. It is commonly used for images, videos and documents.

### Q2. What is the difference between block, file and object storage?

Block storage stores data in blocks, file storage organizes data in directories and files, and object storage stores independent objects identified by keys.

### Q3. What is Amazon S3?

Amazon S3 is an AWS object storage service used to store and retrieve large amounts of data.

### Q4. What is an S3 bucket?

A bucket is a container that stores objects in S3.

### Q5. What is an object key?

An object key is the unique name used to identify an object within a bucket.

### Q6. Should we store images in a database or S3?

For many scalable applications, store the actual image in object storage and store its reference and metadata in the database.

### Q7. What is a pre-signed URL?

A temporary URL that grants permission to perform a specific operation on an object, such as uploading or downloading.

### Q8. Why do we use pre-signed URLs?

They allow clients to upload or download files directly from storage without exposing permanent cloud credentials or proxying all file traffic through backend servers.

### Q9. What is multipart upload?

It divides a large file into smaller parts that can be uploaded separately and assembled after successful completion.

### Q10. What is resumable upload?

It allows an interrupted upload to continue from previously completed portions rather than restarting the entire file.

### Q11. What is file metadata?

Information describing a file, such as name, type, size, owner and upload date.

### Q12. Why do we use CDN with S3?

S3 stores original files, while CDN distributes cached copies closer to users for faster content delivery.

### Q13. What happens if an upload fails halfway?

The client can retry or resume supported uploads. Incomplete multipart uploads should eventually be aborted or cleaned up.

### Q14. How do you securely upload an image from Android to S3?

Android requests a pre-signed upload URL from an authenticated backend. The backend generates a restricted temporary URL. Android uploads directly to S3. Backend verifies the upload before storing metadata.

### Q15. Why shouldn't we store files on backend server disks?

With multiple backend instances, files stored on one server may not be available on another. Server replacement and scaling also become harder. Shared object storage avoids this problem.

### Q16. What is object versioning?

It maintains multiple versions of an object so previous versions can be recovered when necessary.

### Q17. What is storage lifecycle management?

It automatically transitions or deletes objects according to configured rules.

### Q18. What is the difference between durability and availability?

Durability describes protection against data loss. Availability describes whether the system can provide access when requested.

### Q19. What is hot and cold storage?

Hot storage is optimized for frequently accessed data. Cold storage is intended for infrequently accessed data and is often cheaper but slower to retrieve.

### Q20. How would you design image storage for Instagram?

Use object storage for original images, background processing for image variants, metadata storage in a database and CDN for delivery.

---

# 22\. Scenario-Based Interview Questions

## Scenario 1: Image Uploaded to Server A, But Server B Cannot Find It

What is the problem?

**Expected answer:**

The image is stored on Server A's local disk. We should use shared object storage so every backend instance can access the same file.

## Scenario 2: Users Upload 2 GB Videos

How would you design the upload process?

**Expected answer:**

Use multipart uploads with retry support and resumability. Direct uploads through temporary pre-signed URLs can reduce backend bandwidth.

## Scenario 3: Backend Is Overloaded Because of Image Uploads

What can you do?

**Expected answer:**

Generate pre-signed URLs and let Android upload directly to object storage instead of sending all file bytes through backend servers.

## Scenario 4: Users Download Images Slowly From Other Countries

How can you improve performance?

**Expected answer:**

Use a CDN in front of object storage to cache and deliver content from geographically distributed edge servers.

## Scenario 5: User Accidentally Overwrites an Important Document

How can you recover the previous file?

**Expected answer:**

Enable object versioning so previous versions can be retrieved, subject to retention and lifecycle policies.

## Scenario 6: A User Uploads a 500 MB Video and Network Disconnects

How can the application continue uploading?

**Expected answer:**

Use multipart or another resumable upload mechanism. Track completed parts and retry only the incomplete portions.

## Scenario 7: How Would You Store 100 Million Images?

**Expected answer:**

Use scalable object storage, store metadata and object references in a database, use CDN for delivery, and apply lifecycle and access policies.

---

# 23\. Day 18 – Quick Revision Cheat Sheet

| Concept | Simple Meaning |
| --- | --- |
| File Storage | Stores files and folders |
| Block Storage | Stores data in blocks |
| Object Storage | Stores data as objects |
| Bucket | Container for objects |
| Object | Actual stored file/data |
| Object Key | Object identifier |
| Metadata | Information about the file |
| Amazon S3 | AWS object storage |
| Pre-Signed URL | Temporary authorized URL |
| Multipart Upload | Uploading a file in parts |
| Resumable Upload | Continuing an interrupted upload |
| CDN | Delivers cached content near users |
| Object Versioning | Maintains older object versions |
| Lifecycle Policy | Automatically transitions or deletes objects |
| Hot Storage | Frequently accessed data |
| Cold Storage | Rarely accessed data |
| Durability | Protection against permanent data loss |
| Availability | Ability to access data when requested |

---

# 24\. Final Learning Checklist

## Storage Fundamentals

- [ ]Understand file storage.
- [ ]Understand block storage.
- [ ]Understand object storage.
- [ ]Understand the differences between all three.
- [ ]Understand Amazon S3.
- [ ]Understand buckets and object keys.
- [ ]Understand database vs object storage.

## Upload Architecture

- [ ]Understand backend-mediated uploads.
- [ ]Understand direct uploads.
- [ ]Understand pre-signed URLs.
- [ ]Understand multipart uploads.
- [ ]Understand resumable uploads.
- [ ]Understand file metadata.
- [ ]Understand upload failure handling.

## File Delivery

- [ ]Understand CDN + object storage.
- [ ]Understand image delivery architecture.
- [ ]Understand object versioning.
- [ ]Understand lifecycle policies.
- [ ]Understand hot and cold storage.
- [ ]Understand durability vs availability.

## Interview Preparation

- [ ]Explain block vs file vs object storage.
- [ ]Explain why we use S3.
- [ ]Explain database vs object storage.
- [ ]Explain pre-signed URL architecture.
- [ ]Explain multipart upload.
- [ ]Design Instagram-style image storage.
- [ ]Design WhatsApp-style media upload.
- [ ]Explain how to upload large videos from Android.

---

# 25\. What Is Next?

## Week 3 – Day 19: Message Queues and Asynchronous Processing

Tomorrow we will learn:

1. What is synchronous communication?
2. What is asynchronous communication?
3. What is a message queue?
4. Why do we need message queues?
5. Producer and consumer.
6. RabbitMQ basics.
7. Apache Kafka basics.
8. Kafka topics and partitions.
9. Message delivery guarantees.
10. Retry mechanisms.
11. Dead Letter Queue (DLQ).
12. Event-driven architecture.
13. Real-world notification and image-processing architecture.

**Day 18 Complete – File Storage and Object Storage.**

## Before moving to Day 19

Make sure you can confidently explain these five questions:

1. What is the difference between block, file and object storage?
2. Why do we store images in S3 instead of directly in a database?
3. What is a pre-signed URL, and why is it useful for Android uploads?
4. What is multipart upload?
5. How would you design image upload and delivery for an Instagram-like application?

Important for your Android interview: Pay special attention to pre-signed URLs, multipart uploads, retry handling and CDN integration. These concepts connect your Android development experience with backend system design.
