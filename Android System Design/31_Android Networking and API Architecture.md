# Week 5 – Day 31: Android Networking and API Architecture

Day 31Intermediate → AdvancedSenior Android Interview Preparation

File name: `31-Android-Networking-and-API-Architecture.md`

Goal: Understand how to design a scalable, reliable, secure, and maintainable networking layer in a production Android application.

By the end of this lesson, you should be able to explain how an Android application communicates with backend services, how Retrofit and OkHttp work internally, and how to handle real-world networking problems.

# 1. What is Android Networking?

Android networking is the process through which an Android application communicates with external systems, usually backend servers, over a network.

For example, consider a shopping application.

When a user opens the application:

1. The app requests product information from the backend.
2. The backend processes the request.
3. The backend fetches data from its database.
4. The backend returns a response.
5. The Android application displays the products.

Efficient Mobile App and Backend Server Design | AI Art Generator | Easy-Peasy.AI

[easy-peasy.ai](<https://easy-peasy.ai/ai-image-generator/images/efficient-mobile-app-backend-server-design?utm_source=chatgpt.com>)

### Simple example

Suppose your backend exposes this API:

http

```http
GET https://api.example.com/products
```

Response:

json

```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "iPhone",
      "price": 79999
    },
    {
      "id": 2,
      "name": "Samsung Galaxy",
      "price": 69999
    }
  ]
}
```

Your Android application receives this JSON and converts it into Kotlin objects.

kotlin

```kotlin
data class Product(
    val id: Int,
    val name: String,
    val price: Int
)
```

This is the foundation of Android networking.

# 2. Understanding the Complete Networking Architecture

In a production Android application, we should not directly make API calls from an Activity or Composable.

Instead, we use layered architecture.

Create Android app with MVVM pattern simply using Android Architecture Component | by hongbeom | hongbeomi dev | Medium

[medium.com](<https://medium.com/hongbeomi-dev/create-android-app-with-mvvm-pattern-simply-using-android-architecture-component-529d983eaabe?utm_source=chatgpt.com>)

### Recommended architecture

text

```text
UI Layer
   |
   v
ViewModel
   |
   v
Use Case (Optional)
   |
   v
Repository
   |
   v
Remote Data Source
   |
   v
Retrofit
   |
   v
OkHttp
   |
   v
Network
   |
   v
Backend API
```

### Responsibilities of each layer

| Layer | Responsibility |
| --- | --- |
| UI | Displays API data |
| ViewModel | Manages UI state |
| Use Case | Contains business logic |
| Repository | Coordinates data sources |
| Remote Data Source | Makes API requests |
| Retrofit | Converts API interfaces into HTTP calls |
| OkHttp | Handles HTTP communication |
| Backend | Processes requests and returns responses |

Senior-level interview point: Why should the ViewModel not directly call Retrofit?

Because it creates tight coupling between presentation and networking. It makes testing, maintenance, caching, and changing data sources more difficult.

# 3. Retrofit – Understanding the Fundamentals

Releases · square/retrofit · GitHub

Retrofit

A type-safe HTTP client for Android and Kotlin applications.

Developed by Square

Retrofit allows us to define backend APIs using Kotlin interfaces.

Without Retrofit, we would need to manually handle HTTP requests, response parsing, and JSON conversion.

### Step 1: Define API interface

kotlin

```kotlin
interface ProductApi {

    @GET("products")
    suspend fun getProducts(): List<Product>

    @GET("products/{id}")
    suspend fun getProduct(
        @Path("id") id: Int
    ): Product

    @POST("products")
    suspend fun createProduct(
        @Body product: Product
    ): Product
}
```

### Step 2: Create Retrofit instance

kotlin

```kotlin
object RetrofitClient {

    private const val BASE_URL =
        "https://api.example.com/"

    private val retrofit = Retrofit.Builder()
        .baseUrl(BASE_URL)
        .addConverterFactory(
            GsonConverterFactory.create()
        )
        .build()

    val productApi: ProductApi =
        retrofit.create(ProductApi::class.java)
}
```

### Step 3: Call API

kotlin

```kotlin
class ProductRepository(
    private val api: ProductApi
) {

    suspend fun getProducts(): List<Product> {
        return api.getProducts()
    }
}
```

### Important Retrofit annotations

| Annotation | Purpose |
| --- | --- |
| `@GET` | HTTP GET request |
| `@POST` | HTTP POST request |
| `@PUT` | HTTP PUT request |
| `@PATCH` | HTTP PATCH request |
| `@DELETE` | HTTP DELETE request |
| `@Path` | Dynamic URL parameter |
| `@Query` | Query parameter |
| `@Body` | Request body |
| `@Header` | Request header |
| `@Headers` | Static headers |
| `@FormUrlEncoded` | Form-encoded request |
| `@Multipart` | Multipart file upload |

### Example: Query parameters

kotlin

```kotlin
@GET("products")
suspend fun getProducts(
    @Query("page") page: Int,
    @Query("limit") limit: Int
): List<Product>
```

Generated request:

http

```http
GET /products?page=1&limit=20
```

### Example: Path parameters

kotlin

```kotlin
@GET("users/{userId}")
suspend fun getUser(
    @Path("userId") userId: Int
): User
```

Generated request:

http

```http
GET /users/101
```

Interview question: What is the difference between `@Path` and `@Query`?

- `@Path` identifies a particular resource.
- `@Query` provides optional parameters to filter, sort, or paginate a resource.

# 4. OkHttp – What Happens Internally?

GitHub - square/okhttp: Square’s meticulous HTTP client for the JVM, Android, and GraalVM.

OkHttp

An HTTP client responsible for making network requests efficiently.

One important interview topic:

What is the difference between Retrofit and OkHttp?

| Retrofit | OkHttp |
| --- | --- |
| API abstraction | HTTP client |
| Defines API interfaces | Executes HTTP requests |
| Converts responses into objects | Handles connections and networking |
| Supports converters | Supports interceptors |
| Built on top of OkHttp commonly | Can work independently |

Retrofit uses OkHttp as its underlying HTTP client when configured with it.

### OkHttp setup

kotlin

```kotlin
val okHttpClient = OkHttpClient.Builder()
    .connectTimeout(30, TimeUnit.SECONDS)
    .readTimeout(30, TimeUnit.SECONDS)
    .writeTimeout(30, TimeUnit.SECONDS)
    .build()
```

Connect this client to Retrofit:

kotlin

```kotlin
val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .client(okHttpClient)
    .addConverterFactory(
        GsonConverterFactory.create()
    )
    .build()
```

# 5. OkHttp Interceptors

Important interview topicAn interceptor allows us to inspect, modify, or observe HTTP requests and responses.

Real-world applications use interceptors for:

- Adding authentication tokens.
- Adding common headers.
- Logging network requests.
- Handling common request metadata.
- Monitoring network performance.

### Example: Authentication interceptor

Suppose every API request requires an access token.

Instead of adding the token manually to every API method, use an interceptor.

kotlin

```kotlin
class AuthInterceptor(
    private val tokenProvider: TokenProvider
) : Interceptor {

    override fun intercept(
        chain: Interceptor.Chain
    ): Response {

        val originalRequest = chain.request()

        val token = tokenProvider.getAccessToken()

        val newRequest = originalRequest.newBuilder()
            .header(
                "Authorization",
                "Bearer $token"
            )
            .build()

        return chain.proceed(newRequest)
    }
}
```

### Why is this useful?

Without an interceptor:

kotlin

```kotlin
api.getProducts(token)
api.getProfile(token)
api.getOrders(token)
api.getCart(token)
```

With an interceptor:

kotlin

```kotlin
api.getProducts()
api.getProfile()
api.getOrders()
api.getCart()
```

The interceptor adds the common header automatically.

Senior-level point: An interceptor should not blindly attach authentication credentials to requests going to unrelated hosts. Restrict credentials to trusted API hosts.

# 6. Interceptor vs Network Interceptor

This is a frequently asked Android interview question.

OkHttp provides two different interceptor types.

| Application Interceptor | Network Interceptor |
| --- | --- |
| Operates at application level | Operates closer to network communication |
| Can observe cached responses | Observes actual network exchanges |
| Can short-circuit requests | Cannot freely replace network exchange behavior |
| Runs once per logical call | Can run for individual network exchanges |
| Suitable for common headers and logging | Suitable for network-level inspection |

### Example

Application interceptor:

kotlin

```kotlin
.addInterceptor(AuthInterceptor(tokenProvider))
```

Network interceptor:

kotlin

```kotlin
.addNetworkInterceptor(
    NetworkLoggingInterceptor()
)
```

Remember:

- Application interceptor: logical HTTP call.
- Network interceptor: network exchange.

# 7. API Error Handling

A production application must handle failures properly.

An API request can fail for several reasons:

| Error | Meaning |
| --- | --- |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Resource Not Found |
| 408 | Request Timeout |
| 429 | Too Many Requests |
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |

Also, networking failures can happen without an HTTP response.

Examples:

- No internet connection.
- DNS resolution failure.
- Connection timeout.
- SSL handshake failure.
- Connection reset.

### Recommended API result wrapper

Instead of exposing exceptions directly to the UI, define a result type.

kotlin

```kotlin
sealed interface ApiResult<out T> {

    data class Success<T>(
        val data: T
    ) : ApiResult<T>

    data class Error(
        val message: String,
        val code: Int? = null
    ) : ApiResult<Nothing>

    data object Loading : ApiResult<Nothing>
}
```

### Repository implementation

kotlin

```kotlin
class ProductRepository(
    private val api: ProductApi
) {

    suspend fun getProducts(): ApiResult<List<Product>> {

        return try {
            val response = api.getProducts()

            ApiResult.Success(response)

        } catch (e: IOException) {

            ApiResult.Error(
                message = "Network connection failed"
            )

        } catch (e: HttpException) {

            ApiResult.Error(
                message = "Server error",
                code = e.code()
            )
        }
    }
}
```

Note: This simplified example assumes the API returns a list directly. In production, also consider cancellation exceptions, structured error bodies, and unexpected failures. Never swallow coroutine cancellation.

# 8. Token Refresh Architecture

Very important for senior interviewsMost production applications use access tokens and refresh tokens.

### What are they?

Access token: Used to authenticate API requests.

Refresh token: Used to obtain a new access token after the old one expires.

Example:

text

```text
Android Application
       |
       | Access Token
       v
    Backend
       |
       | 401 Unauthorized
       v
Token Refresh Process
       |
       | Refresh Token
       v
 Authentication Server
       |
       | New Access Token
       v
Retry Original Request
```

### Typical flow

1. User logs in.
2. Backend returns access token and refresh token.
3. Android stores tokens securely.
4. Android sends access token with API requests.
5. Access token expires.
6. Backend returns 401.
7. Android attempts token refresh.
8. New access token is stored.
9. Original request is retried.

### Where should token refresh happen?

A common production architecture uses an OkHttp `Authenticator` for handling eligible 401 authentication challenges.

Important distinction:

- Interceptor: modifies requests and observes responses.
- Authenticator: attempts to obtain new credentials when the server challenges authentication.

### Senior-level considerations

- Prevent multiple simultaneous refresh requests.
- Use a single-flight refresh mechanism.
- Avoid infinite retry loops.
- Handle refresh token expiration.
- Clear session data when refresh fails.
- Never log access or refresh tokens.
- Do not refresh on every possible 401 without considering the backend's error semantics.

# 9. Retry Mechanism

What happens if an API request fails temporarily?

For example:

- Server temporarily unavailable.
- Connection reset.
- Network timeout.
- Temporary gateway failure.

We can use retry logic for eligible operations.

### Exponential backoff

Instead of retrying immediately, increase the delay after each attempt.

$$
Delay = \min(MaxDelay,\ BaseDelay \times 2^{attempt})
$$

Example:

| Attempt | Delay |
| --- | --- |
| 1 | 1 second |
| 2 | 2 seconds |
| 3 | 4 seconds |
| 4 | 8 seconds |

In production, add random jitter to avoid many clients retrying simultaneously.

### Kotlin example

kotlin

```kotlin
suspend fun <T> retryRequest(
    maxRetries: Int = 3,
    block: suspend () -> T
): T {

    var attempt = 0

    while (true) {
        try {
            return block()

        } catch (e: IOException) {

            if (attempt >= maxRetries) {
                throw e
            }

            val delayMillis =
                1000L * (1L shl attempt)

            delay(delayMillis)

            attempt++
        }
    }
}
```

This is an educational example. A production implementation should use bounded delays, jitter, cancellation-aware handling, and retry only eligible failures.

Important interview question: Should we retry every API request?

No.

For example, automatically retrying a payment POST request can create duplicate transactions if the first request succeeded but its response was lost.

Use idempotency keys or backend-supported deduplication for operations that must not execute twice.

# 10. Token Storage and Secure Networking

Authentication tokens are sensitive credentials.

Avoid storing them in plain text SharedPreferences.

Prefer:

- Android Keystore-backed encryption.
- Encrypted storage for sensitive data.
- HTTPS for all production API communication.
- Proper certificate validation.
- Avoiding sensitive information in logs.

### Network security

Understand these concepts:

| Concept | Purpose |
| --- | --- |
| HTTPS | Encrypted communication |
| TLS | Secures communication between client and server |
| Certificate validation | Verifies server identity |
| Certificate pinning | Adds an additional certificate/public-key trust restriction |
| Android Keystore | Protects cryptographic keys |

Certificate pinning should be introduced carefully, with a rotation and recovery strategy. Incorrect pinning can prevent legitimate users from connecting after certificate changes.

# 11. Pagination Architecture

Imagine an application with 100,000 products.

Downloading all products at once is inefficient.

Instead, fetch data in smaller pages.

Example:

http

```http
GET /products?page=1&limit=20
```

Response:

json

```json
{
  "items": [],
  "page": 1,
  "totalPages": 500,
  "hasNext": true
}
```

### Android Paging 3

(Descontinuado) Noções básicas da Android Paging | Android Developers

Paging 3

Jetpack library for loading and displaying large datasets incrementally.

Basic architecture:

text

```text
UI
 |
 v
PagingDataAdapter / Compose LazyPagingItems
 |
 v
Pager
 |
 v
PagingSource
 |
 v
Repository
 |
 v
Retrofit API
```

Example:

kotlin

```kotlin
class ProductPagingSource(
    private val api: ProductApi
) : PagingSource<Int, Product>() {

    override suspend fun load(
        params: LoadParams<Int>
    ): LoadResult<Int, Product> {

        return try {
            val page = params.key ?: 1

            val response = api.getProducts(
                page = page,
                limit = params.loadSize
            )

            LoadResult.Page(
                data = response.items,
                prevKey = if (page == 1) null else page - 1,
                nextKey = if (response.hasNext) page + 1 else null
            )

        } catch (e: Exception) {
            LoadResult.Error(e)
        }
    }

    override fun getRefreshKey(
        state: PagingState<Int, Product>
    ): Int? = state.anchorPosition
}
```

This assumes the API supports page-number pagination and returns `hasNext`.

Interview question: Offset pagination vs cursor pagination?

- Offset pagination: uses page number or offset and limit.
- Cursor pagination: uses a continuation token based on the last item or position.

Cursor pagination is generally more stable for continuously changing datasets.

# 12. API Caching and Request Deduplication

These are important production-level topics.

### API caching

Suppose the user opens the product screen repeatedly.

Without caching:

text

```text
Open screen → API request
Open screen → API request
Open screen → API request
```

With caching:

text

```text
Open screen → API request → Save result
Open screen → Read cached result
Open screen → Read cached result
```

Caching can happen at multiple levels:

- OkHttp HTTP cache.
- Repository memory cache.
- Room database.
- DataStore for small settings and preferences.

Remember: DataStore is not a replacement for a relational database such as Room.

### Request deduplication

Imagine the user taps the refresh button five times quickly.

Without deduplication:

text

```text
Request 1
Request 2
Request 3
Request 4
Request 5
```

With deduplication:

text

```text
Request 1
   |
   +---- Other callers share the same in-flight result
```

This reduces unnecessary backend traffic.

Deduplication must use an appropriate request key and should not accidentally combine requests with different authentication, parameters, or required freshness.

# 13. API Versioning

Backend APIs evolve over time.

For example:

http

```http
GET /api/v1/users
```

Later:

http

```http
GET /api/v2/users
```

Why do we need versioning?

Imagine version 1 returns:

json

```json
{
  "name": "John"
}
```

Version 2 returns:

json

```json
{
  "firstName": "John",
  "lastName": "Smith"
}
```

If the backend suddenly changes the response format, older Android application versions might break.

Versioning helps maintain compatibility between old and new clients.

Common approaches:

- URL versioning: `/api/v1/`
- Header versioning.
- Query parameter versioning.

Senior-level consideration: Mobile applications cannot force all users to upgrade immediately. Backward compatibility and graceful deprecation are important.

# 14. Complete Production Android Networking Architecture

Let's combine everything.

목차 | capstone-2024-22

[kookmin-sw.github.io](<https://kookmin-sw.github.io/capstone-2024-22/?utm_source=chatgpt.com>)

A recommended architecture:

text

```text
             Android UI
                 |
                 v
              ViewModel
                 |
                 v
              Use Case
                 |
                 v
             Repository
              /      \
             /        \
            v          v
      Local Source   Remote Source
            |          |
            v          v
           Room      Retrofit
                        |
                        v
                      OkHttp
                        |
            +-----------+-----------+
            |           |           |
            v           v           v
       Interceptor  Authenticator  Cache
            |           |
            +-----------+
                        |
                        v
                    Backend API
```

### Where everything belongs

| Feature | Recommended location |
| --- | --- |
| API definitions | Retrofit interface |
| HTTP requests | OkHttp |
| Authentication headers | Interceptor |
| Token refresh | Authenticator / refresh coordinator |
| API error mapping | Remote data source / repository |
| Retry | Network layer or repository policy |
| Pagination | PagingSource |
| Local caching | Room / OkHttp cache |
| Request deduplication | Repository / network coordination |
| UI state | ViewModel |

# 15. Senior Android Developer Interview Questions

These are questions you should be able to answer after completing Day 31.

### Basic questions

1. What is Retrofit?
2. What is OkHttp?
3. What is the difference between Retrofit and OkHttp?
4. What is an interceptor?
5. What is the difference between `@Path` and `@Query`?
6. How do you make API calls using Kotlin Coroutines?
7. What is the difference between GET, POST, PUT and PATCH?

### Intermediate questions

8. What is the difference between application and network interceptors?
9. How do you handle API errors?
10. How do you handle token expiration?
11. How do you implement automatic token refresh?
12. How do you handle multiple simultaneous 401 responses?
13. How do you implement exponential backoff?
14. How does Paging 3 work?
15. How do you implement API caching?

### Advanced senior-level questions

16. How would you design a reusable networking module for a large Android application?
17. How would you prevent duplicate API requests?
18. How would you handle an API request that succeeds on the server but times out on the client?
19. How would you prevent duplicate payment transactions?
20. How would you design networking for an application supporting millions of users?
21. How would you support multiple backend environments such as development, staging and production?
22. How would you handle backward compatibility when the backend API changes?
23. How would you design an offline-first networking architecture?
24. How would you handle authentication when several API requests fail simultaneously?
25. How would you monitor network failures and API performance in production?

# 16. Practical System Design Exercise

Hands-on assignmentScenario: Design the networking architecture for an e-commerce Android application.

Your application must support:

- User login.
- Access and refresh tokens.
- Product listing.
- Product pagination.
- Product details.
- Add to cart.
- Payment.
- Offline product browsing.
- API error handling.
- Automatic retry where appropriate.

Your task is to design the complete Android networking architecture.

Online Shopping E-commerce Android Application and E-commerce Website Development

[ijraset.com](<https://www.ijraset.com/research-paper/e-commerce-android-application-and-e-commerce-website-development?utm_source=chatgpt.com>)

Think about these questions:

1. Where should Retrofit be initialized?
2. Where should OkHttp be initialized?
3. How will you add authentication tokens?
4. How will you refresh expired tokens?
5. How will you prevent duplicate refresh requests?
6. How will you cache products?
7. How will you implement pagination?
8. How will you handle offline mode?
9. How will you prevent duplicate payment requests?
10. How will you test the networking layer?

# 17. Day 31 Revision Checklist

Day 31 learning progress

# 0/15

[ ]Understand Android networking architecture

[ ]Understand Retrofit and API interfaces

[ ]Understand OkHttp and its responsibilities

[ ]Understand application vs network interceptors

[ ]Implement authentication interceptor

[ ]Understand API error handling

[ ]Understand access token and refresh token

[ ]Understand Authenticator and token refresh

[ ]Understand retry and exponential backoff

[ ]Understand idempotency and duplicate requests

[ ]Understand Paging 3

[ ]Understand API caching and request deduplication

[ ]Understand API versioning

[ ]Design a complete production networking architecture

[ ]Practice senior-level interview questions

Reset checklist

# 18. What You Must Remember for Senior Interviews

The five most important takeaways:

1. Retrofit vs OkHttp: Retrofit defines and abstracts APIs; OkHttp handles HTTP communication.
2. Interceptors: Centralize cross-cutting networking behavior such as authentication headers and logging.
3. Token refresh: Handle concurrent authentication failures without triggering multiple refresh operations.
4. Retry: Retry only eligible operations. Idempotency matters, especially for payments.
5. Architecture: Keep networking separate from UI through repositories and data sources.

## Next lesson – Day 32

Android Performance and Scalability

Topics:

- Application startup optimization.
- Memory management.
- Battery optimization.
- ANR (Application Not Responding).
- Memory leaks.
- Coroutines and Flow performance.
- Paging 3 performance.
- Image loading and caching.
- Profiling and debugging.

Recommended study approach: Don't just memorize Retrofit annotations. Try implementing the networking module, authentication interceptor, token refresh flow, and error handling in a small Android project. Those practical implementation decisions are what distinguish senior-level answers from basic definitions.
