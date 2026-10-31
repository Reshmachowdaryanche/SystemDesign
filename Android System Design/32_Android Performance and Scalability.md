# Week 5 – Day 32: Android Performance and Scalability

Day 32Intermediate → AdvancedSenior Android Interview Preparation

File name: `32-Android-Performance-and-Scalability.md`

Goal: Learn how to build Android applications that are fast, responsive, memory-efficient, battery-efficient, and capable of handling large amounts of data.

By the end of this lesson, you should be able to identify performance bottlenecks, debug ANRs and memory leaks, optimize application startup, and design scalable Android applications.

## 1. What is Android Performance Optimization?

Android performance optimization means improving how efficiently an application uses device resources.

The major resources are:

- CPU
- RAM
- Battery
- Network
- Storage
- GPU

Consider two shopping applications.

Application A:

- Takes 5 seconds to launch.
- Scrolling is slow.
- Images load slowly.
- Battery consumption is high.
- Sometimes crashes.

Application B:

- Launches quickly.
- Scrolling is smooth.
- Images load efficiently.
- Handles network failures.
- Uses memory efficiently.

Application B provides a better user experience.

### Major areas of Android performance

Top 12 Tips to Improve Android App Performance in 2026

[webandcrafts.com](<https://webandcrafts.com/blog/tips-to-improve-android-app-performance?utm_source=chatgpt.com>)

| Area | What we optimize |
| --- | --- |
| Startup | Time required to launch application |
| Memory | RAM consumption |
| CPU | Expensive computations |
| UI Rendering | Smooth scrolling and animations |
| Battery | Background processing and resource usage |
| Network | API calls and data transfer |
| Images | Image loading and caching |
| Database | Query execution and data retrieval |

# 2. Android Application Startup Optimization

Very important interview topicApplication startup time is the time taken from launching an application until the user can interact with it.

Imagine opening a banking application.

If it takes 7 seconds to display the home screen, users may become frustrated.

### Types of application startup

1. Cold Start

The application process does not exist.

Android must:

- Create a new process.
- Initialize the application.
- Create the Activity.
- Inflate the UI.
- Render the first frame.

Important Performance Metrics. Before you begin to work with… | by Ben Weiss | Android Developers | Medium

[medium.com](<https://medium.com/androiddevelopers/important-performance-metrics-c7dacf018eb3?utm_source=chatgpt.com>)

2. Warm Start

The application process already exists, but the Activity needs to be recreated or brought back.

3. Hot Start

The application process and Activity already exist. Android simply brings the Activity back to the foreground.

### Why is cold start slower?

Because the application needs to initialize multiple components.

For example:

text

```text
User taps application
        |
        v
Android creates process
        |
        v
Application.onCreate()
        |
        v
Activity creation
        |
        v
Layout inflation
        |
        v
First frame rendering
        |
        v
User sees application
```

### Common startup performance problems

- Initializing too many SDKs.
- Performing database operations on the main thread.
- Making synchronous network calls.
- Loading large images immediately.
- Creating unnecessary objects.
- Performing expensive dependency injection initialization.

### How to optimize startup?

1. Lazy initialization

Instead of initializing everything during application startup, initialize components when required.

Bad:

kotlin

```kotlin
class MyApplication : Application() {

    val analytics = Analytics.initialize(this)
    val database = Room.databaseBuilder(
        this,
        AppDatabase::class.java,
        "app.db"
    ).build()
}
```

Better:

kotlin

```kotlin
class MyApplication : Application() {

    val analytics by lazy {
        Analytics.initialize(this)
    }
}
```

2. Avoid main-thread operations

Never perform expensive disk or network operations on the main thread.

3. Use App Startup carefully

AndroidX App Startup allows controlled initialization of components.

4. Defer nonessential work

For example:

- Analytics initialization.
- Noncritical configuration downloads.
- Background synchronization.
- Preloading nonessential data.

### Interview question

Q: How would you improve cold startup time in a large Android application?

Answer:

1. Measure startup using Macrobenchmark.
2. Identify expensive initialization.
3. Remove unnecessary synchronous work.
4. Introduce lazy initialization.
5. Defer noncritical SDK initialization.
6. Optimize dependency injection.
7. Reduce first-screen layout complexity.
8. Measure improvements using startup benchmarks.

# 3. Android Main Thread and UI Rendering

Must understandAndroid applications have a main thread, also called the UI thread.

It is responsible for:

- Handling user interactions.
- Processing UI events.
- Updating views.
- Performing UI-related work.
- Coordinating frame rendering.

If we block the main thread, the application becomes unresponsive.

### Example of blocking the main thread

kotlin

```kotlin
class MainActivity : AppCompatActivity() {

    override fun onCreate(
        savedInstanceState: Bundle?
    ) {
        super.onCreate(savedInstanceState)

        Thread.sleep(5000)
    }
}
```

What happens?

The UI thread stops processing events for 5 seconds.

The user cannot interact with the application properly.

### Correct approach

Move expensive work to a background dispatcher.

kotlin

```kotlin
class ProductRepository {

    suspend fun getProducts(): List<Product> =
        withContext(Dispatchers.IO) {
            // Network or blocking I/O operation
            fetchProducts()
        }
}
```

For CPU-intensive operations:

kotlin

```kotlin
withContext(Dispatchers.Default) {
    performHeavyCalculation()
}
```

Remember:

| Dispatcher | Typical usage |
| --- | --- |
| Main | UI operations |
| IO | Blocking I/O operations |
| Default | CPU-intensive work |
| Unconfined | Specialized coroutine use cases |

### What is a frame?

Android continuously renders frames to display animations and UI updates.

At 60 Hz, the display refreshes approximately every:

$$
\frac{1000}{60} = 16.67 \text{ ms}
$$

At 90 Hz:

$$
\frac{1000}{90} = 11.11 \text{ ms}
$$

At 120 Hz:

$$
\frac{1000}{120} = 8.33 \text{ ms}
$$

If your application cannot produce frames within the required timing, users may experience jank.

Detailed Explanation of Android Rendering Mechanism Based on Choreographer · Android Performance

[androidperformance.com](<https://androidperformance.com/en/2019/10/22/Android-Choreographer/?utm_source=chatgpt.com>)

### What is Jank?

Jank means visible stuttering or inconsistent frame delivery.

Common causes:

- Expensive UI calculations.
- Excessive recompositions.
- Large image decoding.
- Complex layouts.
- Main-thread blocking.
- Expensive custom drawing.

Interview question: How do you improve scrolling performance in a RecyclerView?

- Use DiffUtil.
- Avoid unnecessary `notifyDataSetChanged()`.
- Reuse ViewHolders.
- Load images asynchronously.
- Avoid expensive operations in `onBindViewHolder()`.
- Use appropriate image sizes.
- Avoid unnecessary nested layouts.
- Use Paging for large datasets.

# 4. Memory Management in Android

Very importantAndroid devices have limited memory.

Your application must manage memory carefully.

### What is RAM?

RAM is temporary memory used by running applications.

For example, when your application creates an object:

kotlin

```kotlin
val user = User(
    name = "John",
    age = 25
)
```

Memory is allocated to hold that object and its associated data.

When an object is no longer reachable, it can eventually be collected by the garbage collector.

### What is Garbage Collection?

Garbage Collection (GC) automatically identifies objects that are no longer reachable and reclaims their memory.

Example:

kotlin

```kotlin
fun createUser() {
    val user = User("John", 25)
}
```

When the function finishes, the local reference goes out of scope. If nothing else references that object, it becomes eligible for garbage collection.

Important: GC does not necessarily run immediately.

### What is a Memory Leak?

A memory leak occurs when an object remains reachable even though the application no longer needs it.

Example:

kotlin

```kotlin
object UserManager {

    var activity: Activity? = null
}
```

Suppose an Activity assigns itself to this static-like singleton:

kotlin

```kotlin
UserManager.activity = this
```

Even after the Activity is destroyed, the singleton holds a reference to it.

Therefore, the Activity cannot be garbage collected.

The LeakCanary Method | Block Engineering Blog

[engineering.block.xyz](<https://engineering.block.xyz/blog/the-leakcanary-method?utm_source=chatgpt.com>)

### Common causes of memory leaks

| Cause | Example |
| --- | --- |
| Static references | Singleton holding Activity |
| Handler leaks | Long-running callbacks |
| Unregistered listeners | Event listeners not removed |
| Context leaks | Keeping Activity context |
| Coroutines | Incorrectly scoped long-running jobs |
| Observers | Lifecycle-unaware observers |

### How to prevent memory leaks?

1. Prefer application context when appropriate

kotlin

```kotlin
class DatabaseManager(context: Context) {

    private val appContext =
        context.applicationContext
}
```

2. Use lifecycle-aware components

kotlin

```kotlin
lifecycleScope.launch {
    viewModel.uiState.collect { state ->
        // Update UI
    }
}
```

3. Cancel unnecessary coroutines

Use `viewModelScope`, `lifecycleScope`, and properly structured coroutine scopes.

4. Unregister listeners

Remove listeners when their owner is no longer active.

### How do you detect memory leaks?

FAQ - LeakCanary

LeakCanary

Automatically detects and reports memory leaks in Android applications.

Other useful tools:

- Android Studio Memory Profiler.
- Heap dumps.
- Allocation tracking.

# 5. ANR – Application Not Responding

Critical interview topicANR means Application Not Responding.

Android displays an ANR when an application cannot respond to certain system events within the allowed time.

For example:

- The user taps a button.
- The main thread is blocked.
- The application cannot process the input event.
- Android detects that the application is unresponsive.

Android Vitals in ASO - ShyftUp

[shyftup.com](<https://www.shyftup.com/blog/android-vitals-in-aso/?utm_source=chatgpt.com>)

### Common causes of ANR

1. Long-running operations on the main thread.
2. Synchronous network calls.
3. Database queries on the UI thread.
4. Deadlocks.
5. Lock contention.
6. Expensive calculations.

### Example

Bad code:

kotlin

```kotlin
button.setOnClickListener {

    val result = performHeavyCalculation()

    textView.text = result
}
```

If `performHeavyCalculation()` takes too long, it blocks the UI thread.

Better:

kotlin

```kotlin
lifecycleScope.launch {

    val result = withContext(
        Dispatchers.Default
    ) {
        performHeavyCalculation()
    }

    textView.text = result
}
```

### ANR timeout examples

Common Android input-dispatch timeout is approximately 5 seconds.

Other ANR categories, such as broadcast receiver timeouts and service execution timeouts, have different limits depending on the component and circumstances.

Do not assume every ANR has the same timeout.

### How to debug ANR?

1. Inspect Play Console ANR reports.
2. Analyze the main thread stack trace.
3. Check for blocking operations.
4. Look for lock contention.
5. Identify deadlocks.
6. Use Android Studio Profiler.
7. Use StrictMode during development.

Interview question: What is the difference between ANR and crash?

| ANR | Crash |
| --- | --- |
| Application becomes unresponsive | Application terminates unexpectedly |
| Often caused by blocked main thread | Often caused by an unhandled exception |
| User may see ANR dialog | User may see app crash |
| Requires responsiveness debugging | Requires crash and stack-trace debugging |

# 6. Battery Optimization

Battery consumption is another important performance metric.

Consider a location-tracking application.

If it continuously requests GPS updates, performs network calls, and keeps the CPU active, battery consumption increases.

### Common causes of battery drain

- Frequent location updates.
- Excessive network requests.
- Continuous background services.
- Frequent wakeups.
- Unnecessary polling.
- Inefficient background synchronization.

### How to optimize battery consumption?

1. Use WorkManager for deferrable background work.

Understanding Android Background Processing: Service, IntentService, and WorkManager | by Maksim Syramalotau | Medium

WorkManager

Recommended library for persistent, deferrable background work.

Example:

kotlin

```kotlin
class SyncWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result {

        return try {
            syncData()

            Result.success()

        } catch (e: IOException) {
            Result.retry()
        }
    }
}
```

Schedule work:

kotlin

```kotlin
val request =
    PeriodicWorkRequestBuilder<SyncWorker>(
        15,
        TimeUnit.MINUTES
    ).build()

WorkManager.getInstance(context)
    .enqueueUniquePeriodicWork(
        "data_sync",
        ExistingPeriodicWorkPolicy.KEEP,
        request
    )
```

Note: 15 minutes is the minimum repeat interval for periodic WorkManager work. Actual execution is controlled by Android scheduling and constraints; it is not an exact timer.

2. Batch network requests

Instead of making multiple requests separately, combine them when appropriate.

3. Avoid unnecessary wakeups

Do not keep the CPU active without a real requirement.

4. Use appropriate location priorities

Request high-accuracy location only when necessary.

5. Respect background restrictions

Android restricts background execution to protect battery and performance.

# 7. Coroutines and Flow Performance Optimization

This is especially important for senior Android developers.

### Common coroutine mistakes

Mistake 1: Performing blocking work on Main

kotlin

```kotlin
viewModelScope.launch {

    val data = blockingDatabaseOperation()

}
```

If the operation blocks the main thread, it can cause UI freezes.

Correct:

kotlin

```kotlin
viewModelScope.launch {

    val data = withContext(Dispatchers.IO) {
        blockingDatabaseOperation()
    }

}
```

If your Room DAO already exposes a `suspend` function or `Flow`, Room handles the database execution appropriately; do not add unnecessary dispatcher switching.

### Mistake 2: Creating unnecessary coroutines

Avoid launching many independent coroutines when a single structured operation is sufficient.

### Mistake 3: Collecting Flow repeatedly

Incorrect lifecycle handling can create multiple collectors and duplicate work.

Use lifecycle-aware collection in UI.

For Compose:

kotlin

```kotlin
@Composable
fun ProductScreen(
    viewModel: ProductViewModel
) {
    val state by viewModel.uiState
        .collectAsStateWithLifecycle()

    // Render UI using state
}
```

### StateFlow vs SharedFlow

| StateFlow | SharedFlow |
| --- | --- |
| Represents current state | Represents shared emissions |
| Always has an initial value | No initial value required |
| Replays latest value | Replay configurable |
| Suitable for UI state | Suitable for events or shared streams |

### Flow operators

Important operators:

kotlin

```kotlin
flow
    .map { transform(it) }
    .filter { it.isValid }
    .distinctUntilChanged()
    .collect { value ->
        // Process value
    }
```

Use `distinctUntilChanged()` to avoid processing identical consecutive values.

Use `debounce()` for cases such as search input, where you want to wait for a pause in user typing.

# 8. Paging 3 Performance Optimization

You learned Paging 3 in Day 31.

Today, let's understand its performance implications.

Imagine Instagram displaying millions of posts.

Downloading all posts at once is inefficient.

Instead, load only the data required by the current viewport and nearby content.

Page from network and database (Views) | Android Developers

[developer.android.com](<https://developer.android.com/topic/libraries/architecture/views/paging/v3-network-db-views?utm_source=chatgpt.com>)

### Important components

| Component | Responsibility |
| --- | --- |
| PagingSource | Loads pages from a data source |
| Pager | Creates PagingData stream |
| PagingData | Represents paginated data |
| PagingDataAdapter | Displays paginated data in RecyclerView |
| LazyPagingItems | Displays PagingData in Compose |
| RemoteMediator | Coordinates remote and local data |

### Compose example

kotlin

```kotlin
@Composable
fun ProductScreen(
    viewModel: ProductViewModel
) {
    val products = viewModel.products
        .collectAsLazyPagingItems()

    LazyColumn {
        items(
            count = products.itemCount
        ) { index ->

            products[index]?.let { product ->
                ProductItem(product)
            }
        }
    }
}
```

### Performance best practices

- Use appropriate page sizes.
- Avoid unnecessary transformations.
- Use stable item keys in Compose.
- Avoid expensive composable calculations.
- Load images asynchronously.
- Use placeholders when appropriate.
- Use Room with RemoteMediator for offline-first pagination when required.

# 9. Image Loading and Caching

Real-world Android interview topicImages are among the biggest sources of memory consumption in Android applications.

Consider an e-commerce application showing 100 product images.

Suppose each image is 4000 × 3000 pixels.

If decoded into ARGB\_8888 format:

$$
Memory = Width \times Height \times 4
$$

$$
= 4000 \times 3000 \times 4
$$

$$
= 48,000,000 \text{ bytes}
$$

That is approximately 45.8 MiB for one decoded image.

Loading many such images can cause serious memory problems.

### How do we solve this?

Use image-loading libraries such as:

Efficient Image Loading with Coil: A Guide to Loading Images in Android with URL and Headers | by Rinkal Davdaparmar | Medium

Coil

Kotlin-first image loading

Introduction to Glide: The Image Loading Library for Android | by Eran Gross | Medium

Glide

Image loading and caching

### Coil example

kotlin

```kotlin
@Composable
fun ProductImage(
    imageUrl: String
) {
    AsyncImage(
        model = imageUrl,
        contentDescription = "Product image",
        contentScale = ContentScale.Crop
    )
}
```

### Image caching levels

1. Memory cache

Stores decoded images in RAM for quick access.

2. Disk cache

Stores image data on disk to avoid downloading it again.

3. Network cache

HTTP caching can reuse responses when server caching headers permit it.

### Image optimization techniques

- Resize images to display dimensions.
- Use thumbnails for lists.
- Load full-resolution images only on detail screens.
- Use appropriate image formats.
- Configure memory and disk caching.
- Cancel unnecessary image requests when items leave the screen.

Interview question: Why should we not load full-resolution images inside RecyclerView?

Because it increases memory usage, image decoding work, garbage collection pressure, and the possibility of OutOfMemoryError.

# 10. Android Performance Monitoring Tools

A senior developer should know how to measure performance rather than simply guessing.

## Important tools

Measure app performance | Android game development | Android Developers

1. Android Studio Profiler

Analyze CPU, memory, network, and energy-related behavior.

Esegui il debug del layout con Layout Inspector | Android Studio | Android Developers

2. Layout Inspector

Inspect UI hierarchy, layouts, and Compose UI structures.

Write a Macrobenchmark | App quality | Android Developers

3. Macrobenchmark

Measure startup time, scrolling, and other user journeys.

Vodafone tech | Improve Android App Performance With Baseline Profiles

4. Baseline Profiles

Help Android Runtime optimize frequently used application code ahead of execution.

Android From Scratch: How to Run Your Application on a Physical Device | Envato Tuts+

5. StrictMode

Detect accidental disk and network operations on the main thread during development.

### Macrobenchmark vs Microbenchmark

| Macrobenchmark | Microbenchmark |
| --- | --- |
| Measures complete user journeys | Measures small code operations |
| App startup | Function execution |
| Scrolling performance | Algorithm performance |
| Navigation | Individual methods |
| End-to-end performance | Fine-grained performance |

# 11. Real-World Scenario: Optimize an E-commerce Application

Imagine you are working on an e-commerce application with:

- 1 million users.
- 50,000 products.
- Product images.
- Search.
- Cart.
- Order history.
- Background synchronization.

Users report:

1. Slow application startup.
2. Laggy product scrolling.
3. High battery consumption.
4. Occasional crashes.
5. Memory issues.

How would you approach the problem?

### Step 1: Startup optimization

- Measure cold startup.
- Defer unnecessary initialization.
- Avoid synchronous disk and network operations.
- Optimize first-screen rendering.

### Step 2: Scrolling optimization

- Use Paging 3.
- Use stable item keys.
- Optimize RecyclerView or Compose lists.
- Load appropriately sized images.
- Avoid expensive item calculations.

### Step 3: Memory optimization

- Find memory leaks.
- Analyze heap dumps.
- Reduce image sizes.
- Avoid unnecessary object allocations.
- Fix lifecycle-related references.

### Step 4: Battery optimization

- Reduce frequent polling.
- Batch background operations.
- Use WorkManager for deferrable synchronization.
- Optimize location and network usage.

### Step 5: Crash and ANR monitoring

- Monitor production crashes.
- Analyze ANR traces.
- Track startup and rendering performance.
- Prioritize problems based on user impact.

This is how a senior Android developer should approach performance issues: measure, identify the bottleneck, fix it, and measure again.

# 12. Senior Android Developer Interview Questions

## Basic questions

1. What is Android application startup time?
2. What is the difference between cold, warm, and hot start?
3. What is garbage collection?
4. What is a memory leak?
5. What is ANR?
6. What is the main thread?
7. What is jank?
8. What is the difference between IO and Default dispatchers?

## Intermediate questions

9. How do you optimize application startup?
10. How do you identify memory leaks?
11. How do you improve RecyclerView performance?
12. How do you reduce battery consumption?
13. How do you handle background operations?
14. What is StrictMode?
15. What is the difference between StateFlow and SharedFlow?
16. How does Paging 3 improve application performance?

## Advanced senior-level questions

17. Your application takes 5 seconds to launch. How would you investigate it?

18. Your application crashes with OutOfMemoryError while displaying images. How would you solve it?

19. Users report that scrolling is not smooth on low-end Android devices. How would you debug it?

20. Your application shows ANRs only in production. How would you investigate them?

21. How would you optimize an application that performs frequent background synchronization?

22. How would you reduce excessive recompositions in Jetpack Compose?

23. How would you optimize memory consumption in an application with multiple modules?

24. How would you design an image caching architecture for an application with millions of images?

25. What are Baseline Profiles, and how do they improve startup performance?

26. How would you establish performance benchmarks for a large Android application?

# 13. Practical Assignment

Hands-on senior-level exerciseScenario: Optimize a product listing screen.

Assume your screen has 10,000 products.

Each product contains:

- Product image.
- Product name.
- Price.
- Rating.
- Discount information.

Current problems:

- Slow scrolling.
- High memory consumption.
- Repeated image downloads.
- Long startup time.
- UI freezes when loading products.

Maple Roots - Mobile App Design Agency

[maplerootsllc.com](<https://www.maplerootsllc.com/?utm_source=chatgpt.com>)

Your task is to design an optimized solution.

### Requirements

1. Use Paging 3.
2. Use Coil or Glide for image loading.
3. Implement image caching.
4. Use Room for local product storage if offline support is required.
5. Use ViewModel and StateFlow for UI state.
6. Avoid main-thread blocking.
7. Implement lazy initialization.
8. Use stable Compose item keys.
9. Prevent unnecessary recompositions.
10. Measure scrolling and startup performance.

### Expected architecture

text

```text
              Product Screen
                    |
                    v
                 ViewModel
                    |
                    v
                  Pager
                    |
                    v
              Repository
                /     \
               /       \
              v         v
          Room DB    Remote API
                         |
                         v
                      Retrofit

Product Image → Coil → Memory Cache / Disk Cache
```

# 14. Day 32 Revision Checklist

Track your learning as you complete each concept.

Day 32 progress

# 0/18

[ ]Android performance fundamentals

[ ]Cold start, warm start and hot start

[ ]Application startup optimization

[ ]Main thread and UI rendering

[ ]Frame timing and jank

[ ]Memory management and garbage collection

[ ]Memory leaks and LeakCanary

[ ]ANR and debugging

[ ]Battery optimization and WorkManager

[ ]Coroutines and Flow optimization

[ ]StateFlow vs SharedFlow

[ ]Paging 3 performance

[ ]Image loading and caching

[ ]Android Studio Profiler

[ ]Macrobenchmark vs Microbenchmark

[ ]Baseline Profiles and StrictMode

[ ]Real-world performance optimization exercise

[ ]Senior-level interview questions

 Reset checklist

# 15. What You Must Remember for Senior Interviews

| Topic | Key takeaway |
| --- | --- |
| Startup | Defer unnecessary initialization |
| Main thread | Never block UI with expensive work |
| Memory | Prevent unnecessary object retention |
| ANR | Investigate blocked main thread and system traces |
| Battery | Minimize unnecessary background work |
| Coroutines | Use appropriate dispatchers and structured concurrency |
| Paging | Load data incrementally |
| Images | Resize and cache appropriately |
| Profiling | Measure before optimizing |
| Baseline Profiles | Improve frequently used code execution performance |

## Next Lesson – Day 33

Designing a Real-time Chat App (WhatsApp, Slack) | by Arvind Kumar | Medium

Android Real-Time Architecture

Week 5 – Day 33Intermediate → Advanced

Topics we will cover:

1. Real-time communication fundamentals.
2. WebSockets in Android.
3. Firebase Cloud Messaging (FCM).
4. Foreground services.
5. Android background execution restrictions.
6. Automatic reconnection.
7. Connection management.
8. Heartbeat and ping/pong.
9. Online/offline user status.
10. Designing a production-ready real-time Android architecture.

Senior-level learning advice: For Day 32, don't just memorize definitions of ANR and memory leaks. Open Android Studio, run a sample application, use the Memory Profiler and CPU Profiler, and deliberately investigate a performance problem. Being able to explain how you diagnosed a real issue is extremely valuable in senior interviews.
