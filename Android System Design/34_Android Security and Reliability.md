# Week 5 – Day 34: Android Security and Reliability

Day 34Intermediate → AdvancedSenior Android Interview Preparation

File name: `34-Android-Security-and-Reliability.md`

Goal: Understand how to protect sensitive user information, secure API communication, store authentication credentials, protect application code, and build reliable Android applications.

By the end of this lesson, you should be able to design a secure Android application suitable for production environments and explain Android security decisions in senior developer interviews.

## 1. What is Android Application Security?

Android application security means protecting an application, its users, and its data from unauthorized access, modification, theft, and misuse.

Consider a banking application.

It handles:

- User credentials.
- Authentication tokens.
- Bank account information.
- Transaction information.
- Personal information.
- Payment details.

If this information is exposed, users can suffer financial and privacy losses.

Therefore, security must be considered throughout the application architecture.

### Major areas of Android security

Android App Security: A Practical Guide for Production Apps | by Yogesh Shinde | Feb, 2026 | Medium

[medium.com](<https://medium.com/%40yogesh_shinde/android-app-security-a-practical-guide-for-production-apps-ef30361564bf?utm_source=chatgpt.com>)

| Area | Purpose |
| --- | --- |
| Secure Storage | Protect sensitive information |
| Authentication | Verify user identity |
| Authorization | Control access to resources |
| Network Security | Protect data communication |
| Encryption | Protect sensitive data |
| Code Obfuscation | Make reverse engineering harder |
| Runtime Security | Detect suspicious environments |
| Crash Reliability | Prevent unexpected application failures |
| Feature Flags | Control application functionality |

# 2. Android Keystore

Very important senior interview topicLocal Data Security with Android Keystore – Threema

Android Keystore

A system that allows applications to create and use cryptographic keys while keeping key material protected from direct application access.

### What problem does Keystore solve?

Imagine that your application needs to store an authentication token securely.

A simple approach would be:

kotlin

```kotlin
val token = "my-secret-token"

sharedPreferences.edit()
    .putString("token", token)
    .apply()
```

Problem:

The token is stored as ordinary application data.

If an attacker gains access to the application's private data through a compromised device or other means, the token could be exposed.

We need stronger protection.

### What is Android Keystore?

Android Keystore protects cryptographic keys.

Instead of storing a raw encryption key in your application code, you generate a key inside the Keystore system.

Your application can request cryptographic operations using that key without directly obtaining its key material.

Android Keystoreを利用したAzure IoTへのセキュアな接続 #AzureIoTHub - Qiita

[qiita.com](<https://qiita.com/kmitsu76/items/c5be6e3f7b66b918d256?utm_source=chatgpt.com>)

### Important characteristics

- Keys can be non-exportable.
- Keys can have usage restrictions.
- Keys can be hardware-backed when supported.
- Keys can require user authentication.
- Keys can be used for encryption, decryption, signing, and verification.

### Hardware-backed security

Some Android devices support hardware-backed key protection.

Examples:

- Trusted Execution Environment (TEE).
- StrongBox security hardware.

The key material can be protected by dedicated hardware, depending on device capabilities and key configuration.

### Important interview question

Q: What is the difference between SharedPreferences and Android Keystore?

| SharedPreferences | Android Keystore |
| --- | --- |
| Stores application preferences | Manages cryptographic keys |
| Can store strings and values | Stores/protects cryptographic key material |
| Does not automatically encrypt data | Provides protected cryptographic operations |
| Suitable for ordinary preferences | Suitable for sensitive cryptographic operations |

Important: Keystore itself is not a general-purpose encrypted storage database. We commonly combine it with encrypted storage.

# 3. Secure Token Storage

Critical for real-world Android applicationsMost applications use authentication tokens.

For example:

json

```json
{
  "access_token": "abc123",
  "refresh_token": "xyz456"
}
```

These tokens must be protected.

### What should we avoid?

Avoid storing sensitive tokens in:

- Plain SharedPreferences.
- Plain text files.
- External storage.
- Hardcoded constants.
- Application logs.
- Analytics events.

### Recommended architecture

text

```text
Android Application
        |
        v
Token Storage Layer
        |
        v
Encrypted Storage
        |
        +------ Encrypted Token Data
        |
        v
Android Keystore
        |
        v
Protected Encryption Key
```

### How does encrypted storage work?

1. Generate a cryptographic key using Android Keystore.
2. Encrypt sensitive data using the key.
3. Store encrypted data in application-private storage.
4. Retrieve encrypted data when needed.
5. Decrypt using the Keystore-backed key.

### Kotlin example: Generate AES key

kotlin

```kotlin
fun generateSecretKey(): SecretKey {

    val keyGenerator =
        KeyGenerator.getInstance(
            KeyProperties.KEY_ALGORITHM_AES,
            "AndroidKeyStore"
        )

    val properties =
        KeyGenParameterSpec.Builder(
            "app_encryption_key",
            KeyProperties.PURPOSE_ENCRYPT or
                KeyProperties.PURPOSE_DECRYPT
        )
            .setBlockModes(
                KeyProperties.BLOCK_MODE_GCM
            )
            .setEncryptionPaddings(
                KeyProperties.ENCRYPTION_PADDING_NONE
            )
            .setRandomizedEncryptionRequired(true)
            .build()

    keyGenerator.init(properties)

    return keyGenerator.generateKey()
}
```

### Encrypt data

kotlin

```kotlin
fun encrypt(
    secretKey: SecretKey,
    data: ByteArray
): Pair<ByteArray, ByteArray> {

    val cipher = Cipher.getInstance(
        "AES/GCM/NoPadding"
    )

    cipher.init(
        Cipher.ENCRYPT_MODE,
        secretKey
    )

    val encryptedData =
        cipher.doFinal(data)

    return Pair(
        cipher.iv,
        encryptedData
    )
}
```

Here:

- AES is the encryption algorithm.
- GCM provides authenticated encryption.
- IV is the initialization vector.
- Ciphertext is the encrypted data.

The IV is not secret, but it must be stored with the ciphertext and must never be reused with the same key.

### Decrypt data

kotlin

```kotlin
fun decrypt(
    secretKey: SecretKey,
    iv: ByteArray,
    encryptedData: ByteArray
): ByteArray {

    val cipher = Cipher.getInstance(
        "AES/GCM/NoPadding"
    )

    val spec = GCMParameterSpec(
        128,
        iv
    )

    cipher.init(
        Cipher.DECRYPT_MODE,
        secretKey,
        spec
    )

    return cipher.doFinal(encryptedData)
}
```

Senior-level consideration: Never implement your own cryptographic protocol casually. Use established Android security APIs and well-reviewed libraries. Also plan for key invalidation, backup/restore behavior, logout, and token expiration.

# 4. HTTPS and TLS

Must understandWhen an Android application communicates with a backend, sensitive information travels through a network.

For example:

http

```http
POST https://api.example.com/login
```

The request may contain:

json

```json
{
  "username": "john",
  "password": "password123"
}
```

What happens if an attacker intercepts this communication?

Without proper encryption, credentials and other data could be exposed.

### What is HTTPS?

HTTPS is HTTP communication protected using TLS.

TLS stands for Transport Layer Security.

It provides:

- Confidentiality.
- Integrity.
- Server authentication.

The Foundation of REST API: HTTP - ByteByteGo Newsletter

[blog.bytebytego.com](<https://blog.bytebytego.com/p/the-foundation-of-rest-api-http?utm_source=chatgpt.com>)

### How does TLS work?

Simplified flow:

1. Android application connects to the backend.
2. Server presents its certificate.
3. Android verifies the certificate chain and hostname.
4. Client and server negotiate cryptographic parameters.
5. They establish shared session keys.
6. Encrypted communication begins.

### HTTP vs HTTPS

| HTTP | HTTPS |
| --- | --- |
| Unencrypted communication | TLS-protected communication |
| No TLS server authentication | Server authentication through certificates |
| Data can be exposed in transit | Data is protected in transit |
| Generally unsuitable for sensitive production APIs | Recommended for production APIs |

### Important interview question

Q: Is HTTPS alone sufficient for Android application security?

No.

HTTPS protects communication in transit, but does not automatically protect:

- Data stored locally.
- Compromised devices.
- Hardcoded secrets.
- Insecure application logic.
- Unauthorized backend operations.
- Sensitive data exposed in logs.

Security must be implemented across the complete architecture.

# 5. Android Network Security Configuration

Android provides Network Security Configuration to control network security behavior declaratively.

For example, you can:

- Disable cleartext HTTP traffic.
- Configure trusted certificate authorities.
- Configure debug-only trust anchors.
- Define domain-specific network security rules.

### Example

Create:

`res/xml/network_security_config.xml`

xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<network-security-config>

    <base-config cleartextTrafficPermitted="false" />

</network-security-config>
```

Add it to your application manifest:

xml

```xml
<application
    android:networkSecurityConfig=
        "@xml/network_security_config"
    android:usesCleartextTraffic="false">

</application>
```

This helps prevent accidental unencrypted HTTP connections.

### Why is this important?

Imagine a developer accidentally changes:

text

```text
https://api.example.com
```

to:

text

```text
http://api.example.com
```

A production security configuration should prevent the application from silently communicating over cleartext HTTP.

# 6. Certificate Pinning

Frequently asked in senior interviewsLet's understand an important security concept.

When your Android application connects to a backend using HTTPS, it verifies the server certificate against trusted certificate authorities.

But some applications want an additional restriction.

They want to trust only a particular certificate or public key for their backend.

This is called certificate pinning.

### What is certificate pinning?

Certificate pinning restricts which certificates or public keys are accepted for a particular server.

Certificate Pinning: Deep dive and Flutter implementation | by Andrea Santucci | Medium

[medium.com](<https://medium.com/%40and.santucci.97/certificate-pinning-deep-dive-and-flutter-implementation-1f1c52f347dc?utm_source=chatgpt.com>)

### Why use certificate pinning?

It can help protect against certain certificate-authority compromise and man-in-the-middle scenarios.

### Example using OkHttp

kotlin

```kotlin
val certificatePinner =
    CertificatePinner.Builder()
        .add(
            "api.example.com",
            "sha256/BASE64_PUBLIC_KEY_HASH"
        )
        .build()

val client = OkHttpClient.Builder()
    .certificatePinner(certificatePinner)
    .build()
```

The hash shown above is a placeholder, not a usable certificate pin.

### Important risks

Incorrect certificate pinning can break connectivity.

For example:

1. Your backend changes its certificate.
2. Your application still trusts only the old pin.
3. Legitimate connections fail.
4. Users cannot access your backend.

### Senior-level best practices

- Prefer public-key pinning when appropriate.
- Include backup pins.
- Plan certificate rotation.
- Maintain a recovery strategy.
- Test pin changes before production release.
- Never disable normal TLS certificate validation.

Interview question: What is the difference between HTTPS and certificate pinning?

HTTPS verifies the server using the configured trust system. Pinning adds a more restrictive check against expected certificates or public keys.

# 7. R8 and ProGuard

Important Android build and security topicAndroid applications contain compiled code.

Attackers can reverse-engineer APKs and inspect parts of their code.

We can make reverse engineering more difficult using code shrinking and obfuscation.

### What is R8?

Unmasking minifyEnabled: Your Android App's Secret Weapon for Smaller, Faster Builds | by Shafraz Buhary | Medium

R8

Android's code shrinker and optimizer, which also performs obfuscation.

R8 can perform three major operations.

1. Code shrinking

Removes unused code.

2. Optimization

Optimizes bytecode to improve efficiency.

3. Obfuscation

Changes class, method, and field names to make reverse engineering harder.

### Before obfuscation

kotlin

```kotlin
class PaymentRepository {

    fun processPayment() {
        println("Payment processing")
    }
}
```

### Conceptually after obfuscation

java

```java
class a {
    void b() {
        // ...
    }
}
```

This is an illustrative example; actual R8 transformations depend on the build configuration.

### Enable R8

In your app-level Gradle configuration:

kotlin

```kotlin
android {
    buildTypes {
        release {
            isMinifyEnabled = true
            isShrinkResources = true

            proguardFiles(
                getDefaultProguardFile(
                    "proguard-android-optimize.txt"
                ),
                "proguard-rules.pro"
            )
        }
    }
}
```

### What is ProGuard?

ProGuard is an older Java bytecode shrinking and obfuscation tool.

Modern Android builds generally use R8 as the integrated shrinker.

ProGuard-compatible configuration rules are still used with R8.

### Important interview question

Q: Does R8 make an Android application completely secure?

No.

R8 increases the difficulty of reverse engineering, but it does not prevent determined attackers from analyzing an application.

Never store backend master secrets, private signing keys, or other critical credentials inside an APK.

# 8. Root Detection Concepts

Advanced security topic### What is a rooted Android device?

Normally, Android applications run inside a restricted environment called an application sandbox.

Rooting gives a user elevated privileges that can bypass some of Android's normal restrictions.

A rooted device may allow:

- Access to protected system files.
- Modification of application behavior.
- Inspection of application memory.
- Modification of application data.
- Running applications with elevated privileges.

### Why do applications care about root detection?

Consider a banking application.

It handles highly sensitive information, so the organization may want to identify devices with potentially compromised security.

### Common root detection signals

- Presence of known root-management applications.
- Known rooting-related binaries.
- Unexpected system properties.
- Writable system partitions.
- Indicators of modified boot state.
- Integrity verification results.

### Important Android security APIs

🔐 Google Play Integrity API in Android: Lessons from Real-World App Security | by Pinankh Patel | Jan, 2026 | Medium

Play Integrity API

Helps backend services assess whether requests come from a genuine app and an environment meeting configured integrity requirements.

### How does Play Integrity work?

Simplified flow:

text

```text
Android Application
        |
        v
Request Integrity Token
        |
        v
Google Play Integrity
        |
        v
Integrity Token
        |
        v
Backend Server
        |
        v
Verify Token
        |
        v
Evaluate Integrity Verdict
        |
        v
Allow / Restrict Operation
```

Important: The backend should verify the integrity token. Do not trust a verdict calculated or supplied only by the Android client.

### Should we block every rooted device?

Not necessarily.

Root detection is not foolproof. It can produce false positives and can be bypassed.

A better approach is risk-based security:

- Allow low-risk features.
- Require additional verification for sensitive operations.
- Restrict high-risk financial transactions when appropriate.
- Monitor suspicious behavior on the backend.

Senior interview question: Is root detection sufficient to secure a banking application?

No. Security should also include server-side authorization, secure authentication, encryption, integrity checks, and fraud detection.

# 9. Android Application Reliability

Security protects the application. Reliability ensures that it continues working correctly under unexpected conditions.

Consider an e-commerce application.

What happens when:

- The internet disconnects during checkout?
- The backend returns HTTP 500?
- The user closes the application during payment?
- The application process gets killed?
- A database operation fails?
- A third-party SDK crashes?

A reliable application must handle these situations gracefully.

### Major reliability concepts

| Concept | Purpose |
| --- | --- |
| Crash handling | Handle unexpected application failures |
| Error handling | Handle API and database errors |
| Retry | Recover from temporary failures |
| Idempotency | Prevent duplicate operations |
| Offline support | Support temporary network loss |
| Logging | Help investigate issues |
| Monitoring | Identify production problems |
| Recovery | Restore application state |

### Reliability architecture

Medium

[bleadvertiserapp.medium.com](<https://bleadvertiserapp.medium.com/android-xr-smart-glasses-are-coming-build-your-ble-app-first-70c8bc3210b4?utm_source=chatgpt.com>)

### Example: Payment reliability

Suppose a user clicks Pay.

The request reaches the backend, but the response gets lost because of a network interruption.

The Android application doesn't know whether payment succeeded.

If it blindly retries, the customer could be charged twice.

Solution:

1. Generate a unique payment request ID.
2. Send it to the backend as an idempotency key.
3. Backend processes the payment once.
4. Backend stores the result against that key.
5. If the same request arrives again, return the original result.
6. Android fetches the final payment status.

This is an important connection between Android architecture and distributed systems.

# 10. Crash Handling and Crash Reporting

Important production topicA crash happens when an application encounters an unhandled exception or another fatal runtime condition.

Example:

kotlin

```kotlin
val name: String? = null

println(name!!.length)
```

This throws a `NullPointerException`.

### How do we monitor production crashes?

We need crash reporting tools.

Advanced Android Firebase Crashlytics Integration | by Nick Skelton | Medium

Firebase Crashlytics

Reports crashes and non-fatal errors from production applications.

Other options include Sentry and Bugsnag.

### What information does crash reporting provide?

- Exception type.
- Stack trace.
- Device model.
- Android version.
- Application version.
- Crash frequency.
- Affected users.
- Custom diagnostic information.

### Integrating Crashlytics

Add the Firebase Crashlytics SDK and configure the required Firebase Gradle plugins.

Example:

kotlin

```kotlin
FirebaseCrashlytics.getInstance()
    .log("User opened checkout screen")
```

Record a non-fatal exception:

kotlin

```kotlin
try {
    processOrder()

} catch (e: Exception) {

    FirebaseCrashlytics.getInstance()
        .recordException(e)
}
```

Use this only when the exception is genuinely recoverable. Do not catch and silently suppress serious failures.

### Add custom keys

kotlin

```kotlin
FirebaseCrashlytics.getInstance()
    .setCustomKey(
        "screen",
        "checkout"
    )

FirebaseCrashlytics.getInstance()
    .setCustomKey(
        "payment_method",
        "UPI"
    )
```

Never send passwords, access tokens, payment credentials, or unnecessary personal information to crash reporting services.

### How should a senior developer investigate production crashes?

1. Identify the crash frequency.
2. Check the affected application versions.
3. Analyze the stack trace.
4. Identify the root cause.
5. Reproduce the problem.
6. Fix the issue.
7. Add regression tests.
8. Release the fix.
9. Monitor whether the crash rate decreases.

### Crash vs Non-fatal exception

| Crash | Non-fatal exception |
| --- | --- |
| Application execution terminates unexpectedly | Application continues execution |
| Usually an unhandled fatal failure | Error that can be safely recovered from |
| Requires crash investigation | Requires error monitoring and handling |

# 11. Logging and Monitoring Architecture

Logging is essential for understanding application behavior.

For example, suppose users report that login is failing.

Without logs, debugging becomes difficult.

### Types of logs

| Log level | Purpose |
| --- | --- |
| DEBUG | Development debugging |
| INFO | Important application events |
| WARN | Unexpected but recoverable situations |
| ERROR | Failures requiring investigation |

### Android logging example

kotlin

```kotlin
Log.d("Login", "Login started")

Log.i("Login", "Request completed")

Log.w("Login", "Token refresh delayed")

Log.e("Login", "Login failed", exception)
```

### What should we avoid logging?

Never log:

- Passwords.
- Authentication tokens.
- Credit card information.
- Sensitive personal data.
- Private encryption keys.
- Full sensitive API request bodies.

### Production monitoring

A mature application should monitor:

- Crash-free users.
- ANR rate.
- API failure rate.
- API latency.
- Application startup time.
- Network failure rate.
- Authentication failures.
- Important business transaction failures.

### Observability example

text

```text
Android Application
       |
       +------ Crash Reports
       |
       +------ Performance Metrics
       |
       +------ Analytics Events
       |
       +------ Network Diagnostics
                    |
                    v
             Monitoring Platform
                    |
                    v
               Dashboards
                    |
                    v
                  Alerts
```

Senior-level interview point: Logging tells us what happened, metrics help quantify system behavior, and tracing helps follow a request across multiple components or services.

# 12. Analytics Architecture

Important real-world Android topicAnalytics helps us understand how users interact with an application.

Consider an e-commerce application.

We want to understand:

- How many users open the application?
- How many users view products?
- How many users add products to cart?
- How many users complete checkout?
- Where do users abandon the purchase?

### What is analytics?

Analytics is the collection and analysis of application usage events.

Example events:

text

```text
app_opened
login_success
product_viewed
add_to_cart
checkout_started
payment_success
```

### Analytics architecture

Mobile App Analytics Setup | Events, GA4, Firebase

[blog.appfyl.com](<https://blog.appfyl.com/en/blog/mobile-app-analytics-setup/?utm_source=chatgpt.com>)

### Recommended Android architecture

Do not call an analytics SDK directly from every Activity and Composable.

Instead, introduce an analytics abstraction.

kotlin

```kotlin
interface AnalyticsTracker {

    fun trackEvent(
        event: String,
        parameters: Map<String, Any>
    )
}
```

Implementation:

kotlin

```kotlin
class FirebaseAnalyticsTracker(
    private val analytics: FirebaseAnalytics
) : AnalyticsTracker {

    override fun trackEvent(
        event: String,
        parameters: Map<String, Any>
    ) {
        val bundle = Bundle()

        parameters.forEach { (key, value) ->
            when (value) {
                is String ->
                    bundle.putString(key, value)

                is Int ->
                    bundle.putInt(key, value)

                is Long ->
                    bundle.putLong(key, value)
            }
        }

        analytics.logEvent(event, bundle)
    }
}
```

Usage:

kotlin

```kotlin
analyticsTracker.trackEvent(
    event = "product_viewed",
    parameters = mapOf(
        "product_id" to "P101",
        "category" to "electronics"
    )
)
```

### Why use an analytics abstraction?

- Easy to replace analytics providers.
- Easier unit testing.
- Centralized event naming.
- Consistent event parameters.
- Better control over privacy.

### Senior-level consideration

Analytics must respect user privacy and applicable consent requirements.

Do not collect sensitive information just because an analytics SDK allows custom event parameters.

# 13. Feature Flags

Frequently asked in senior interviewsImagine you develop a new payment feature.

You want to release it to only 5% of users initially.

How can you do this without releasing a completely separate application?

Use feature flags.

### What is a feature flag?

A feature flag allows you to enable or disable application functionality through configuration.

Example:

kotlin

```kotlin
if (featureFlags.isNewPaymentEnabled()) {
    showNewPaymentScreen()
} else {
    showOldPaymentScreen()
}
```

### Feature flag architecture

Designing a Feature Flag and Remote Config System | dcdhameliya.com

[dcdhameliya.com](<https://dcdhameliya.com/blog/designing-a-feature-flag-and-remote-config-system?utm_source=chatgpt.com>)

### Common feature flag use cases

1. Gradual feature rollout.
2. A/B testing.
3. Emergency feature disabling.
4. Environment-specific configuration.
5. Beta features.
6. UI experimentation.

### Types of feature flags

| Type | Purpose |
| --- | --- |
| Release flag | Control feature rollout |
| Experiment flag | Run A/B experiments |
| Operational flag | Enable or disable system functionality |
| Permission flag | Control access to particular functionality |

### Example implementation

kotlin

```kotlin
interface FeatureFlags {

    fun isNewCheckoutEnabled(): Boolean

    fun isNewHomeEnabled(): Boolean

}
```

Implementation:

kotlin

```kotlin
class FeatureFlagsImpl : FeatureFlags {

    override fun isNewCheckoutEnabled() = true

    override fun isNewHomeEnabled() = false
}
```

In production, values may come from Remote Config or a dedicated feature flag service.

### What happens when the internet is unavailable?

Your application should have a fallback strategy.

For example:

- Use the last known valid configuration.
- Use safe default values.
- Avoid blocking application startup while fetching flags.
- Handle expired configurations.
- Keep critical security decisions on the backend.

### Feature flags vs Remote Config

| Feature Flags | Remote Config |
| --- | --- |
| Controls feature availability | Changes remotely configurable values |
| Supports gradual rollouts | Supports configuration updates |
| Can support experiments | Can support experiments |
| Often integrated with configuration services | A configuration delivery mechanism |

Senior interview question: How would you disable a faulty feature without publishing a new APK?

Use a remotely controlled feature flag, with a safe local fallback and backend validation where required.

# 14. Secure Android Architecture – Putting Everything Together

Now let's combine the concepts from today's lesson.

Imagine we are building a banking application.

It must support:

- Secure login.
- Authentication tokens.
- Account details.
- Payments.
- Transaction history.
- Push notifications.
- Analytics.
- Crash monitoring.

### Complete architecture

Zero-Trust Android Architecture. For a long time, Android apps followed… | by Dharma Kshetri | Medium

[medium.com](<https://medium.com/%40dharmakshetri/zero-trust-android-architecture-bb36a5cb0bcd?utm_source=chatgpt.com>)

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
                     Use Case
                         |
                         v
                    Repository
                    /        \
                   /          \
                  v            v
             Local Data     Remote Data
                  |             |
                  v             v
           Encrypted Data    Retrofit
                  |             |
                  v             v
           Android Keystore   OkHttp
                                |
                                v
                            HTTPS / TLS
                                |
                                v
                          Backend Server
                                |
                  +-------------+-------------+
                  |             |             |
                  v             v             v
              Database     Authentication   Payment
                                |
                                v
                          Security Checks
```

### Cross-cutting components

| Component | Responsibility |
| --- | --- |
| Android Keystore | Protect cryptographic keys |
| Encrypted storage | Protect locally stored sensitive data |
| Network Security Config | Configure network trust and cleartext policy |
| TLS | Protect communication |
| Certificate Pinning | Additional server identity restriction |
| R8 | Shrinking and obfuscation |
| Play Integrity | App/device integrity assessment |
| Crashlytics | Crash reporting |
| Analytics | User interaction measurement |
| Feature Flags | Controlled feature rollout |

### Five principles of secure Android development

1. Never trust the client alone.
2. Never store sensitive credentials in plain text.
3. Never transmit sensitive data over unencrypted connections.
4. Never expose secrets through logs or application binaries.
5. Always enforce authorization and important business rules on the backend.

# 15. Senior Android Developer Interview Questions

These are particularly important for developers with 7 years of experience.

### Basic questions

1. What is Android Keystore?
2. What is the difference between SharedPreferences and Keystore?
3. What is AES encryption?
4. What is HTTPS?
5. What is TLS?
6. What is certificate pinning?
7. What is R8?
8. What is code obfuscation?
9. What is root detection?
10. What is Firebase Crashlytics?

### Intermediate questions

11. How do you securely store authentication tokens?
12. How does Android Keystore work internally?
13. What is the difference between encryption and hashing?
14. How does certificate validation work?
15. How do you implement certificate pinning using OkHttp?
16. What is Network Security Configuration?
17. What is the difference between R8 and ProGuard?
18. How do you monitor production crashes?
19. What is the difference between analytics and crash reporting?
20. What are feature flags?

### Advanced senior-level questions

21. How would you design a secure Android banking application?
22. How would you protect authentication tokens from being exposed?
23. What happens when an encryption key becomes invalidated?
24. How would you rotate certificate pins without breaking existing application versions?
25. Is certificate pinning always necessary?
26. How would you secure an Android application running on a rooted device?
27. How would you prevent sensitive information from appearing in crash reports?
28. How would you design a feature flag architecture for a large Android application?
29. How would you investigate a crash that occurs only on specific Android devices?
30. How would you improve the reliability of a payment application?
31. How would you secure communication between Android and multiple backend services?
32. How would you implement a secure logout mechanism?

# 16. Practical Assignment

Hands-on senior-level exerciseScenario: Design security and reliability architecture for a banking application.

Nova Bank by OLUWASEYI EMMANUEL OJO on Dribbble

[dribbble.com](<https://dribbble.com/shots/27302396-Nova-Bank?utm_source=chatgpt.com>)

Your application should support:

- Secure login.
- Access and refresh token storage.
- Biometric authentication.
- Account information.
- Money transfers.
- Transaction history.
- Push notifications.
- Crash reporting.
- Analytics.
- Feature flags.

### Your implementation tasks

1. Generate an encryption key using Android Keystore.
2. Implement encrypted local token storage.
3. Configure HTTPS-only network communication.
4. Understand certificate pinning and backup pins.
5. Enable R8 for release builds.
6. Integrate Crashlytics.
7. Create an analytics abstraction.
8. Implement feature flags.
9. Design secure logout.
10. Handle payment failures without duplicate transactions.

### Questions to answer in your design

- What happens if the access token expires?
- What happens if the device is rooted?
- What happens if the network disconnects during payment?
- What happens if the application crashes during checkout?
- How do you prevent sensitive information from appearing in logs?
- How do you disable a faulty feature remotely?

# 17. Day 34 Revision Checklist

Use this checklist to track your learning.

Day 34 learning progress

# 0/22

[ ]Android security fundamentals

[ ]Android Keystore architecture

[ ]Hardware-backed keys, TEE and StrongBox

[ ]SharedPreferences vs Keystore

[ ]Secure token storage

[ ]AES and AES-GCM encryption

[ ]Encryption vs hashing

[ ]HTTPS and TLS handshake

[ ]Network Security Configuration

[ ]Certificate Pinning

[ ]Certificate rotation and backup pins

[ ]R8 and ProGuard

[ ]Code shrinking and obfuscation

[ ]Root detection concepts

[ ]Google Play Integrity API

[ ]Android application reliability

[ ]Crash handling and Crashlytics

[ ]Logging and production monitoring

[ ]Analytics architecture

[ ]Feature flags and Remote Config

[ ]Secure banking application architecture

[ ]Senior-level interview questions

 Reset checklist

# 18. What You Must Remember for Senior Interviews

| Concept | Key takeaway |
| --- | --- |
| Android Keystore | Protects cryptographic keys |
| Encrypted Storage | Protects sensitive local data |
| HTTPS/TLS | Secures network communication |
| Certificate Pinning | Adds an additional trust restriction |
| R8 | Shrinks, optimizes and obfuscates code |
| Root Detection | Identifies potential device compromise |
| Play Integrity | Provides app/device integrity signals |
| Crashlytics | Monitors production crashes |
| Analytics | Tracks application usage |
| Feature Flags | Controls features without requiring a new release |

# Next Lesson – Day 35

MVVM Clean Architecture in Android | by Anand Gaur | Medium

Android Architecture Revision

Week 5 – Day 35Complete Week 5 revision

Topics:

1. MVVM architecture.
2. Clean Architecture.
3. SOLID principles.
4. Dependency Injection.
5. Repository pattern.
6. Offline-first architecture.
7. Android networking architecture.
8. Performance optimization.
9. Real-time communication.
10. Security and reliability.
11. Complete Android system design interview.
12. Senior-level architecture questions.

Important: Day 35 is not just another theory lesson. We will connect Days 29–34 and practice designing a complete production-ready Android application.

Week 5 milestoneAfter Day 35, your Android System Design phase (Days 29–35) will be complete.

Your next lesson: `35-Android-Architecture-Revision.md`
