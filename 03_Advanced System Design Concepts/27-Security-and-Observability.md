# Week 4 – Day 27: Security and Observability

Day 27 of 49 Advanced System Design

File name: `27-Security-and-Observability.md`

Goal: Understand how to secure modern distributed systems and monitor their health, performance, reliability, and failures.

This is an important topic for Senior Android Developer interviews because you need to understand not only how applications communicate with backend services, but also how to protect user data and investigate production issues.

We will cover two major areas:

1. Security: How to protect applications, APIs, communication, and user data.
2. Observability: How to understand what is happening inside an application and distributed system.

# Part 1 – Security Fundamentals

## 1. What is System Security?

System security refers to protecting an application and its infrastructure against unauthorized access, data theft, modification, and attacks.

Consider a banking application.

A user can:

- View account balance.
- Transfer money.
- View transaction history.
- Update personal information.

What happens if someone accesses another user's account?

What happens if someone intercepts the network request?

What happens if someone modifies a payment request?

These are security problems.

A secure system must ensure:

1. Confidentiality – Only authorized people can access data.
2. Integrity – Data cannot be modified without authorization.
3. Availability – Services remain accessible to legitimate users.

These three principles are called the CIA Triad.

What is the CIA Triad and Why is it Important?

[idstrong.com](<https://www.idstrong.com/sentinel/what-is-cia-triad/?utm_source=chatgpt.com>)

### 1.1 Confidentiality

Confidentiality ensures that sensitive information is accessible only to authorized users.

Example:

A banking application should not allow User A to view User B's account details.

Common techniques:

- Encryption.
- Authentication.
- Authorization.
- Access control.

### 1.2 Integrity

Integrity ensures that information is not modified without permission.

Example:

A user initiates a payment of ₹500.

An attacker should not be able to modify the payment amount to ₹50,000.

Common techniques:

- Cryptographic hashes.
- Digital signatures.
- Message authentication codes.
- Database access controls.

### 1.3 Availability

Availability ensures that the application remains accessible when legitimate users need it.

Example:

A food delivery application should remain operational even if one backend server fails.

Common techniques:

- Load balancing.
- Replication.
- Failover.
- Rate limiting.
- Disaster recovery.

Interview question: What is the CIA Triad?

Answer: Confidentiality, Integrity, and Availability are the three fundamental principles of information security.

# 2. HTTP vs HTTPS

Before understanding TLS, we need to understand HTTP and HTTPS.

## 2.1 HTTP

HTTP stands for Hypertext Transfer Protocol.

It is used for communication between clients and servers.

Example:

An Android application requests user information.

http

```http
GET /api/users/101 HTTP/1.1
Host: example.com
```

The server returns:

json

```json
{
  "userId": 101,
  "name": "John",
  "email": "john@example.com"
}
```

The problem with plain HTTP is that communication is not encrypted.

An attacker who can observe the network traffic may be able to read or modify the communication.

## 2.2 HTTPS

HTTPS stands for Hypertext Transfer Protocol Secure.

It uses TLS (Transport Layer Security) to secure HTTP communication.

Understanding 11 Types of Redirects and Their Impact on SEO

[dashclicks.com](<https://www.dashclicks.com/blog/types-of-redirects?utm_source=chatgpt.com>)

HTTPS provides:

- Encryption.
- Server authentication.
- Communication integrity.

Example:

text

```text
Android Application
       |
       | HTTPS
       |
       v
Backend Server
```

Even if someone intercepts encrypted traffic, they should not be able to understand its contents without the appropriate cryptographic keys.

### HTTP vs HTTPS

| Feature | HTTP | HTTPS |
| --- | --- | --- |
| Encryption | No | Yes |
| TLS | No | Yes |
| Data confidentiality | No | Yes |
| Server authentication | No TLS-based authentication | Yes |
| Production APIs | Avoid | Recommended |

Senior Android interview point: Always use HTTPS for production API communication. Do not transmit authentication tokens, passwords, or sensitive user information over plain HTTP.

# 3. Understanding TLS

TLS stands for Transport Layer Security.

It is a cryptographic protocol that secures communication between two endpoints.

It is used by:

- Banking applications.
- E-commerce applications.
- Social media applications.
- Payment gateways.
- Backend microservices.

### Why do we need TLS?

Imagine an Android application sends this request:

json

```json
{
  "username": "john",
  "password": "mypassword123"
}
```

Without encryption, network observers may be able to read the data.

TLS protects the communication channel.

### 3.1 How TLS works

SSL Policy

[secure.cisco.com](<https://secure.cisco.com/secure-firewall/v7.2/docs/ssl-policy?utm_source=chatgpt.com>)

Simplified TLS handshake:

1. Client connects to the server.
2. Client and server negotiate supported TLS parameters.
3. Server provides its digital certificate.
4. Client validates the certificate and server identity.
5. Client and server establish shared session keys using the key exchange mechanism.
6. Encrypted communication begins.

Modern TLS, particularly TLS 1.3, uses more efficient handshake mechanisms than older TLS versions.

### 3.2 What is a digital certificate?

A digital certificate is an electronic document that helps establish the identity of a server.

It generally contains:

- Domain name.
- Public key.
- Certificate issuer.
- Validity period.
- Digital signature.

Example:

text

```text
Domain: api.example.com

Public Key: Server Public Key

Issuer: Trusted Certificate Authority

Validity: Valid certificate period
```

### 3.3 What is a Certificate Authority (CA)?

A Certificate Authority is an organization that issues and signs digital certificates.

Examples include:

- Let's Encrypt.
- DigiCert.
- GlobalSign.

The Android platform maintains trusted certificate authorities and uses certificate validation to help verify server identity.

### Important interview questions

Q1. What is TLS?

TLS is a cryptographic protocol that provides secure communication through encryption, authentication, and integrity protection.

Q2. What is the difference between SSL and TLS?

SSL is an older protocol that has been deprecated. TLS is its modern successor.

Q3. Why do we need certificates?

Certificates help clients verify that they are communicating with the intended server.

# 4. Encryption Fundamentals

Encryption converts readable information into an unreadable form using cryptographic algorithms and keys.

There are two major types.

## 4.1 Symmetric Encryption

The same secret key is used for encryption and decryption.

Integrating AES Encryption in C#: A Developer’s Guide | by Cemal Can Akgül | Medium

[medium.com](<https://medium.com/%40cemalcanakgul/integrating-aes-encryption-in-c-a-developers-guide-837f2079448a?utm_source=chatgpt.com>)

Example:

text

```text
Original Data
     |
     v
Encryption + Secret Key
     |
     v
Encrypted Data
     |
     v
Decryption + Same Secret Key
     |
     v
Original Data
```

Common algorithm:

- AES (Advanced Encryption Standard).

Advantages:

- Fast.
- Efficient for large amounts of data.
- Suitable for encrypting files and stored information.

Disadvantage:

Both parties must securely obtain the secret key.

## 4.2 Asymmetric Encryption

Asymmetric encryption uses a pair of cryptographic keys:

- Public key.
- Private key.

The Cryptography Concept Journey — Asymmetric Encryption (Public Key Encryption) | by Shlomi Boutnaru, Ph.D. | Dec, 2025 | Medium

[medium.com](<https://medium.com/%40boutnaru/the-cryptography-concept-journey-asymmetric-encryption-public-key-encryption-1efb68fb0258?utm_source=chatgpt.com>)

The public key can be shared, while the private key must remain secret.

Common applications:

- Secure key establishment.
- Digital signatures.
- Certificate-based authentication.

Algorithms include:

- RSA.
- Elliptic Curve Cryptography (ECC).

### Symmetric vs Asymmetric Encryption

| Feature | Symmetric | Asymmetric |
| --- | --- | --- |
| Keys | One shared secret | Public/private pair |
| Speed | Generally faster | Generally slower |
| Key distribution | Requires secure sharing | Public key can be distributed |
| Common usage | Bulk data encryption | Key exchange and signatures |
| Examples | AES | RSA, ECC |

### 4.3 Hashing

Hashing is different from encryption.

A hash function converts input data into a fixed-length output.

Example:

text

```text
Input: hello

SHA-256:

2cf24dba5fb0a30e...
```

Important characteristics:

- One-way operation.
- Fixed-size output for a given hash algorithm.
- Small input changes produce significantly different hashes.
- Used for integrity verification.

Common algorithms:

- SHA-256.
- SHA-512.

Important: Passwords should not be stored as plain text or simply hashed using SHA-256. Use dedicated password-hashing algorithms such as Argon2id, bcrypt, or scrypt with appropriate salts and parameters.

# 5. Authentication vs Authorization

This is one of the most frequently asked security interview questions.

## 5.1 Authentication

Authentication answers:

Who are you?

It verifies the identity of a user or system.

Examples:

- Username and password.
- OTP.
- Biometric authentication.
- OAuth-based login.
- Authentication tokens.

Example:

A user enters their email and password.

The backend validates the credentials and establishes an authenticated session.

## 5.2 Authorization

Authorization answers:

What are you allowed to do?

It determines the permissions associated with an authenticated identity.

Example:

Consider an e-commerce application.

| User | Permission |
| --- | --- |
| Customer | View own orders |
| Seller | Manage own products |
| Admin | Manage users and products |

Even if a customer is authenticated, they should not be able to access administrator functionality.

Demystifying Authentication & Authorization: From Passwords to FIDO2 | by caseyB-it\_SD | Medium

[medium.com](<https://medium.com/%40caseyB-it_sd/demystifying-authentication-authorization-from-passwords-to-fido2-90bfbf9677de?utm_source=chatgpt.com>)

### Authentication vs Authorization

| Feature | Authentication | Authorization |
| --- | --- | --- |
| Purpose | Verify identity | Verify permissions |
| Question | Who are you? | What can you access? |
| Happens | Before access decisions | After identity is established |
| Example | Login | Admin permission check |

Interview question: Can a user be authenticated but not authorized?

Answer: Yes. A customer can successfully log in but may not have permission to access an administrator endpoint.

# 6. Token-Based Authentication

Modern Android applications commonly use token-based authentication.

A typical implementation uses access tokens and refresh tokens.

### How it works

Access and Refresh Tokens in Token-Based Authentication - DEV Community

[dev.to](<https://dev.to/khaledsaeed18/access-and-refresh-tokens-in-token-based-authentication-2a9o?utm_source=chatgpt.com>)

1. User enters login credentials.
2. Android sends credentials securely to the backend.
3. Backend validates credentials.
4. Backend issues an access token and possibly a refresh token.
5. Android uses the access token for API requests.
6. When the access token expires, Android requests a new one using the refresh token.
7. If refresh fails or the session is revoked, the user must authenticate again.

### Access Token

An access token is used to access protected resources.

Example:

http

```http
Authorization: Bearer ACCESS_TOKEN
```

Characteristics:

- Usually short-lived.
- Sent with API requests.
- Grants specific access.

### Refresh Token

A refresh token is used to obtain a new access token.

Characteristics:

- Usually longer-lived.
- Must be protected carefully.
- Should support revocation and rotation where appropriate.

### Important Android considerations

- Do not hardcode tokens in the application.
- Do not store sensitive tokens in plain SharedPreferences.
- Use Android Keystore-backed encryption for sensitive local secrets.
- Avoid logging authentication headers.
- Clear session data during logout.
- Handle token expiration.
- Prevent multiple simultaneous refresh operations from causing race conditions.

# 7. Android Security Architecture

Now let's connect backend security with Android development.

An Android application runs on a device that the developer does not fully control.

Users may:

- Root their devices.
- Reverse engineer APK files.
- Modify application code.
- Inspect local storage.
- Intercept network traffic.
- Attempt unauthorized API calls.

Therefore, mobile applications need multiple layers of security.

Android App Security: A Practical Guide for Production Apps | by Yogesh Shinde | Feb, 2026 | Medium

[medium.com](<https://medium.com/%40yogesh_shinde/android-app-security-a-practical-guide-for-production-apps-ef30361564bf?utm_source=chatgpt.com>)

### Important Android security mechanisms

| Mechanism | Purpose |
| --- | --- |
| Android Keystore | Protect cryptographic keys |
| HTTPS | Secure network communication |
| Network Security Config | Configure network security policies |
| Certificate Pinning | Restrict trusted server certificates or keys |
| Encrypted storage | Protect sensitive local data |
| R8 | Code shrinking and obfuscation |
| BiometricPrompt | Authenticate users using biometrics |
| App signing | Verify application integrity and publisher identity |

## 7.1 Android Keystore

Android Keystore is a system facility for storing cryptographic keys securely.

It can help prevent application processes from directly extracting key material.

Example use cases:

- Encryption keys.
- Cryptographic signing keys.
- Sensitive local data protection.

Conceptual Kotlin example:

kotlin

```kotlin
val keyStore = KeyStore.getInstance("AndroidKeyStore")
keyStore.load(null)

val keyGenerator = KeyGenerator.getInstance(
    KeyProperties.KEY_ALGORITHM_AES,
    "AndroidKeyStore"
)

keyGenerator.init(
    KeyGenParameterSpec.Builder(
        "app_secret_key",
        KeyProperties.PURPOSE_ENCRYPT or
            KeyProperties.PURPOSE_DECRYPT
    )
        .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
        .setEncryptionPaddings(
            KeyProperties.ENCRYPTION_PADDING_NONE
        )
        .build()
)

val secretKey = keyGenerator.generateKey()
```

This creates a key in Android Keystore using AES-GCM.

### Why AES-GCM?

AES-GCM provides:

- Confidentiality.
- Integrity.
- Authentication of encrypted data.

## 7.2 Network Security Configuration

Android supports Network Security Configuration through XML.

It allows developers to define:

- Cleartext traffic policies.
- Trusted certificate authorities.
- Domain-specific security configuration.
- Certificate pinning configuration.

Example:

xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="false" />
</network-security-config>
```

This disables cleartext network traffic for the application.

## 7.3 Certificate Pinning

Certificate pinning is a security technique that restricts which certificates or public keys are accepted for a particular server.

Normally, Android trusts certificates issued by trusted certificate authorities.

With pinning, the application additionally checks whether the server certificate chain or public key matches a configured pin.

SSL Pinning on Android: Best Practices, OkHttp & Retrofit Examples (2026) | Ostorlab: Mobile App Security Testing for Android and iOS

[blog.ostorlab.co](<https://blog.ostorlab.co/android-ssl-pinning.html?utm_source=chatgpt.com>)

### Why use certificate pinning?

It can reduce exposure to certain certificate authority compromise or misissuance scenarios.

### Important disadvantages

- Certificate rotation can break application connectivity.
- Pin expiration and backup pins need careful management.
- Incorrect configuration can block legitimate traffic.
- Pinning is not a replacement for HTTPS or proper backend security.

Senior-level insight: Certificate pinning should be introduced only with a clear certificate rotation and recovery strategy.

# 8. API Security

Securing Android locally is not enough.

The backend must enforce security independently.

### Important API security techniques

1. Authentication

Verify the caller's identity.

2. Authorization

Verify whether the caller can access the requested resource.

3. Input validation

Validate all incoming parameters.

4. Rate limiting

Restrict excessive API requests.

5. HTTPS

Encrypt communication.

6. Request size limits

Prevent unexpectedly large requests.

7. Audit logging

Record important security-related activities.

### Example: Broken authorization

Consider:

http

```http
GET /api/users/101/orders
```

A malicious user changes the ID:

http

```http
GET /api/users/102/orders
```

If the backend only checks whether the user is logged in, User A might access User B's orders.

This is an authorization vulnerability commonly associated with Broken Object Level Authorization (BOLA).

The backend must verify that the authenticated user has permission to access the specific resource.

Important: Never rely on Android UI restrictions as the only security boundary.

# Part 2 – Observability

# 9. What is Observability?

Observability is the ability to understand the internal state of a system by examining the information it produces.

Imagine your Android application has 10 million users.

Suddenly, users report:

- Login is slow.
- Payments are failing.
- Chat messages are delayed.
- The application crashes.

How do you identify the actual problem?

You need observability.

It helps answer:

- What happened?
- Where did the failure occur?
- Why did it happen?
- How many users are affected?
- Is the issue related to Android, backend, database, or network?

### Three pillars of observability

1. Logs.
2. Metrics.
3. Distributed Tracing.

Monitoring Server Software: A Complete Guide for 2026

[fivenines.io](<https://fivenines.io/blog/monitoring-server-software/?utm_source=chatgpt.com>)

# 10. Logging

Logging is the process of recording events that happen inside an application or service.

Example:

A user tries to log in.

The application may produce:

text

```text
INFO: Login request started

INFO: Login API response received

INFO: User authenticated successfully
```

If login fails:

text

```text
ERROR: Login request failed

ERROR: HTTP 500 Internal Server Error
```

### Log levels

| Level | Meaning | Example |
| --- | --- | --- |
| TRACE | Very detailed execution | Function-level execution |
| DEBUG | Debugging information | Request processing details |
| INFO | Normal operations | User login successful |
| WARN | Unexpected but recoverable | Retry triggered |
| ERROR | Operation failed | Payment API failed |
| FATAL | Critical failure | Service cannot start |

### Structured logging

Instead of writing arbitrary text, production systems often use structured logs.

Example:

json

```json
{
  "timestamp": "2026-10-31T10:30:00Z",
  "level": "ERROR",
  "service": "payment-service",
  "requestId": "REQ123",
  "message": "Payment processing failed",
  "errorCode": "PAYMENT_TIMEOUT"
}
```

Structured logs are easier to search and analyze.

### Android logging

Android provides:

kotlin

```kotlin
Log.d("Login", "Login started")
Log.i("Login", "Login successful")
Log.w("Login", "Retry required")
Log.e("Login", "Login failed")
```

Production applications should use an appropriate logging strategy rather than exposing sensitive details through unrestricted logs.

### What should never be logged?

- Passwords.
- Access tokens.
- Refresh tokens.
- Full payment card details.
- Sensitive personal information.
- Private cryptographic keys.

# 11. Metrics

Metrics are numerical measurements that help us understand system performance and health.

Examples:

- API response time.
- Request count.
- Error count.
- CPU usage.
- Memory usage.
- Active WebSocket connections.
- Database query duration.

### Important metric categories

1. Traffic

How many requests does the system receive?

Example:

text

```text
Requests per second = 10,000
```

2. Latency

How much time does the system take to respond?

Example:

text

```text
Average API response time = 200 ms
```

3. Errors

How many requests fail?

Example:

text

```text
Error rate = 2%
```

4. Saturation

How much of the system's available resources are being consumed?

Example:

text

```text
CPU utilization = 85%
```

These are closely related to the four golden signals of monitoring:

- Latency.
- Traffic.
- Errors.
- Saturation.

### Important metrics for an Android application

| Metric | What it indicates |
| --- | --- |
| App startup time | Application startup performance |
| Crash rate | Application stability |
| ANR rate | UI responsiveness |
| API latency | Backend communication performance |
| Network failure rate | Connectivity issues |
| Memory usage | Memory efficiency |
| Battery consumption | Power efficiency |
| Screen rendering time | UI performance |

# 12. Distributed Tracing

This is one of the most important topics in microservices architecture.

Imagine an Android application makes a payment request.

The request passes through multiple backend services.

System Design Series Part 10: Monitoring & Observability - Wasil Zafar

[wasilzafar.com](<https://www.wasilzafar.com/pages/series/system-design/system-design-monitoring-observability.html?utm_source=chatgpt.com>)

Flow:

text

```text
Android App
    |
    v
API Gateway
    |
    v
Order Service
    |
    v
Payment Service
    |
    v
Banking Service
    |
    v
Database
```

Suppose payment takes 5 seconds.

Which service caused the delay?

Without distributed tracing, identifying the problem can be difficult.

Distributed tracing tracks one request across multiple services.

### Trace and Span

Trace: Represents the complete journey of one request through a distributed system.

Span: Represents an individual operation within that journey.

Example:

text

```text
Trace ID: TRACE123

Span 1: API Gateway       100 ms
Span 2: Order Service     200 ms
Span 3: Payment Service  4000 ms
Span 4: Database          300 ms
```

Here, the Payment Service is the likely bottleneck.

### Trace ID

A Trace ID uniquely identifies a distributed request.

It can be propagated between services using tracing context.

For example:

http

```http
traceparent: 00-<trace-id>-<span-id>-01
```

This follows the W3C Trace Context standard.

### Distributed tracing benefits

- Identify slow services.
- Debug failures.
- Understand service dependencies.
- Find bottlenecks.
- Understand end-to-end latency.

### Common distributed tracing technologies

- OpenTelemetry.
- Jaeger.
- Zipkin.
- Grafana Tempo.

Interview question: What is the difference between logging and distributed tracing?

Answer: Logging records individual events. Distributed tracing tracks the complete journey of a request across multiple services.

# 13. Monitoring vs Observability

These terms are related but not identical.

### Monitoring

Monitoring tracks predefined metrics and conditions.

Example:

text

```text
If CPU usage > 90%
Send an alert.
```

### Observability

Observability helps investigate unexpected problems by correlating logs, metrics, traces, and other system information.

Example:

Users report slow payments, but CPU usage is normal.

Observability helps identify that a particular database query is taking four seconds.

| Feature | Monitoring | Observability |
| --- | --- | --- |
| Purpose | Detect known issues | Investigate system behavior |
| Focus | Predefined metrics | Internal system state |
| Questions | Is something wrong? | Why is it wrong? |
| Data | Mainly metrics and alerts | Logs, metrics, traces |

# 14. Alerting

Alerting is the process of notifying engineers when predefined conditions indicate that something needs attention.

Examples:

- API error rate exceeds 5%.
- Payment service becomes unavailable.
- Database connections are exhausted.
- Application crash rate increases.
- Message queue backlog grows.
- WebSocket connection failures increase.

### Example alert rules

| Condition | Alert |
| --- | --- |
| API error rate exceeds threshold | High API failure rate |
| P95 latency exceeds target | Slow API response |
| CPU usage remains high | Infrastructure saturation |
| Database unavailable | Critical database failure |
| Queue backlog increases continuously | Processing delay |

### What is P95 latency?

P95 latency means 95% of requests complete within the specified duration, while 5% take longer.

Example:

text

```text
P50 = 100 ms
P95 = 500 ms
P99 = 1200 ms
```

This means:

- 50% of requests finish within 100 ms.
- 95% finish within 500 ms.
- 99% finish within 1200 ms.

P95 and P99 are often more useful than average latency because averages can hide slow requests.

# 15. SLI, SLO and SLA

These are important concepts for senior-level system design interviews.

### 15.1 SLI – Service Level Indicator

An SLI is a measurable indicator of service performance.

Examples:

- API availability.
- Request latency.
- Error rate.
- Successful payment percentage.

### 15.2 SLO – Service Level Objective

An SLO is an internal target for an SLI.

Example:

text

```text
99.9% of API requests
should succeed within the measurement window.
```

### 15.3 SLA – Service Level Agreement

An SLA is a formal agreement between a service provider and customer that defines expected service levels and possible consequences if those levels are not met.

### Comparison

| Term | Meaning | Example |
| --- | --- | --- |
| SLI | Measurement | API availability |
| SLO | Target | 99.9% availability |
| SLA | Agreement | Contractual availability commitment |

### What is an error budget?

An error budget represents the amount of unreliability permitted by an SLO.

For example, with a 99.9% availability objective, the permitted unavailability is 0.1% over the measurement period.

It helps teams balance reliability improvements and product releases.

# 16. Production Observability Architecture

Let's combine logs, metrics, traces, and alerts.

Top 10 Microservices Interview Questions (2025) | by Arvind Kumar | Medium

[codefarm0.medium.com](<https://codefarm0.medium.com/top-10-microservices-interview-questions-2025-3b0b0c0eb661?utm_source=chatgpt.com>)

### Common tools

| Tool | Purpose |
| --- | --- |
| Prometheus | Metrics collection and monitoring |
| Grafana | Dashboards and visualization |
| OpenTelemetry | Instrumentation and telemetry collection |
| Jaeger | Distributed tracing |
| Zipkin | Distributed tracing |
| ELK Stack | Log collection and analysis |
| Loki | Log aggregation |
| Alertmanager | Alert management |
| Firebase Crashlytics | Android crash reporting |

### Example production flow

1. Android sends an API request.
2. Backend services generate logs.
3. Services expose metrics.
4. OpenTelemetry collects tracing information.
5. Monitoring systems analyze telemetry.
6. Grafana displays dashboards.
7. Alerting systems notify engineers when thresholds are exceeded.

# 17. Android Observability

Now let's understand observability specifically from an Android developer's perspective.

Android applications need monitoring at both application and network levels.

### Important Android monitoring areas

1. Crash monitoring

Detect:

- Fatal exceptions.
- Application crashes.
- Crash frequency.
- Affected application versions.

Tools:

- Firebase Crashlytics.
- Sentry.

2. ANR monitoring

ANR stands for Application Not Responding.

It happens when the application fails to respond within Android's expected time limits.

Common reasons:

- Blocking the main thread.
- Long-running operations.
- Deadlocks.
- Slow disk operations.
- Excessive synchronous work.

3. Performance monitoring

Measure:

- Cold startup time.
- Warm startup time.
- Screen rendering.
- Network latency.
- Memory consumption.

4. Network monitoring

Track:

- API response times.
- HTTP error rates.
- Timeout frequency.
- Retry frequency.
- Connection failures.

5. Business event monitoring

Track events such as:

- Login success.
- Payment success.
- Order creation.
- Checkout failure.
- User registration.

Avoid collecting unnecessary personal information.

### Example: Android API logging interceptor

kotlin

```kotlin
class ApiLoggingInterceptor : Interceptor {

    override fun intercept(
        chain: Interceptor.Chain
    ): Response {

        val request = chain.request()
        val startTime = System.nanoTime()

        try {
            val response = chain.proceed(request)

            val duration = (
                System.nanoTime() - startTime
            ) / 1_000_000

            Log.d(
                "API",
                "${request.method} ${request.url} " +
                    "Code=${response.code} " +
                    "Duration=${duration}ms"
            )

            return response

        } catch (exception: Exception) {
            Log.e(
                "API",
                "Request failed: ${exception.javaClass.simpleName}"
            )

            throw exception
        }
    }
}
```

This is a simplified example. Production instrumentation should avoid logging sensitive URLs, headers, request bodies, or user data.

# 18. Real-World Scenario: Payment Failure Investigation

This is where security and observability come together.

Imagine users report that payments are failing.

How would a senior engineer investigate?

MSA 환경에서 REST API, gRPC, GraphQL로 마스터하는 백엔드 API 개발 초격차 패키지 Online. | 패스트캠퍼스

[fastcampus.co.kr](<https://fastcampus.co.kr/dev_online_api3?utm_source=chatgpt.com>)

### Step 1: Check metrics

Check:

- Payment success rate.
- Payment failure rate.
- API latency.
- Payment gateway availability.

### Step 2: Check logs

Search for:

text

```text
ERROR: Payment processing failed
ERROR: Gateway timeout
ERROR: Database transaction failed
```

Use request IDs to correlate related logs.

### Step 3: Check distributed traces

Identify whether the delay is coming from:

- API Gateway.
- Payment Service.
- External payment provider.
- Database.

### Step 4: Check security

Verify:

- Authentication failures.
- Authorization failures.
- Suspicious request patterns.
- Token expiration.
- Rate limiting.

### Step 5: Identify the root cause

Example:

The payment provider is taking 8 seconds to respond.

Possible actions:

- Configure appropriate timeouts.
- Use circuit breakers.
- Avoid duplicate payment processing through idempotency.
- Alert the operations team.
- Reconcile payment status safely.

Important: Payment retries must be designed carefully. A timeout does not necessarily mean the payment failed. The external provider might have completed the payment even though your application did not receive the response.

# 19. Important Senior Android Developer Interview Questions

## Basic

Q1. What is the CIA Triad?

Confidentiality, Integrity, and Availability.

Q2. What is HTTPS?

HTTP secured using TLS.

Q3. What is TLS?

A cryptographic protocol that secures communication through encryption, authentication, and integrity.

Q4. What is the difference between encryption and hashing?

Encryption is reversible using the appropriate key. Hashing is designed as a one-way transformation.

Q5. What is authentication vs authorization?

Authentication verifies identity. Authorization verifies permissions.

## Intermediate

Q6. What is Android Keystore?

A system facility for securely managing cryptographic keys.

Q7. What is certificate pinning?

A mechanism that restricts which server certificates or public keys an application accepts.

Q8. What are access and refresh tokens?

Access tokens authorize API requests. Refresh tokens are used to obtain new access tokens.

Q9. What are the three pillars of observability?

Logs, metrics, and distributed tracing.

Q10. What is distributed tracing?

Tracking a request as it travels through multiple services.

## Advanced

Q11. What is the difference between monitoring and observability?

Monitoring detects predefined problems. Observability helps investigate unknown or unexpected system behavior.

Q12. What are SLI, SLO, and SLA?

SLI is a measurement, SLO is a target, and SLA is a formal service agreement.

Q13. What is P95 latency?

The response-time threshold within which 95% of requests complete.

Q14. How would you investigate a production API latency issue?

Use metrics to identify the affected service, logs to inspect errors, and distributed tracing to locate the bottleneck.

Q15. How do you prevent sensitive information from leaking through Android logs?

Avoid logging secrets, sanitize request information, restrict production logging, and use secure telemetry practices.

# 20. Additional Important Concepts

These are useful topics to understand for senior interviews.

| Concept | Explanation |
| --- | --- |
| Zero Trust | Never automatically trust a user or service based on network location |
| RBAC | Access control based on roles |
| ABAC | Access control based on attributes |
| OAuth 2.0 | Authorization framework |
| OpenID Connect | Identity layer built on OAuth 2.0 |
| JWT | JSON-based token format |
| WAF | Web Application Firewall |
| DDoS | Distributed Denial of Service |
| Secrets Management | Secure storage and access to credentials |
| Audit Logs | Records of important security events |
| Vulnerability Scanning | Identifying known security weaknesses |
| Disaster Recovery | Restoring services after major failures |

### RBAC vs ABAC

- RBAC: Permissions are assigned based on roles.
- ABAC: Permissions are determined using attributes such as user, resource, and environment properties.

Example:

RBAC: Admin can manage users.

ABAC: A manager can access employee records only within their assigned department.

# 21. Day 27 Quick Revision Cheat Sheet

## Security

| Concept | Remember |
| --- | --- |
| CIA Triad | Confidentiality, Integrity, Availability |
| HTTP | Unencrypted application protocol |
| HTTPS | HTTP over TLS |
| TLS | Secure communication |
| AES | Symmetric encryption |
| RSA / ECC | Asymmetric cryptography |
| SHA-256 | Cryptographic hash |
| Authentication | Who are you? |
| Authorization | What can you access? |
| Android Keystore | Secure key management |
| Certificate Pinning | Restrict trusted server certificates/keys |
| Access Token | API authorization |
| Refresh Token | Obtain new access token |

## Observability

| Concept | Remember |
| --- | --- |
| Logs | Events and errors |
| Metrics | Numerical measurements |
| Traces | End-to-end request journey |
| Trace ID | Identifies a complete trace |
| Span | Individual operation |
| P95 | 95th percentile latency |
| SLI | Service measurement |
| SLO | Service target |
| SLA | Formal service agreement |
| Alerting | Notify on failures |
| Prometheus | Metrics |
| Grafana | Dashboards |
| OpenTelemetry | Instrumentation |
| Crashlytics | Android crash monitoring |

# 22. Learning Checklist

Day 27 Progress

# 0/20

[ ]Understand the CIA Triad

[ ]Understand HTTP vs HTTPS

[ ]Understand TLS handshake

[ ]Understand digital certificates and Certificate Authorities

[ ]Understand symmetric and asymmetric encryption

[ ]Understand hashing and password storage

[ ]Understand authentication vs authorization

[ ]Understand access and refresh tokens

[ ]Understand Android Keystore

[ ]Understand Network Security Configuration

[ ]Understand certificate pinning and its trade-offs

[ ]Understand API security and authorization

[ ]Understand logging and structured logs

[ ]Understand metrics and four golden signals

[ ]Understand distributed tracing, traces and spans

[ ]Understand monitoring vs observability

[ ]Understand alerting and P95/P99 latency

[ ]Understand SLI, SLO, SLA and error budgets

[ ]Understand Android crash and ANR monitoring

[ ]Understand production payment failure investigation

 Reset checklist

# 23. Final Interview Preparation Exercise

Imagine you are designing a banking application with the following requirements:

- Users can log in securely.
- Users can transfer money.
- Transactions must be secure.
- APIs must be monitored.
- Payment failures must be investigated.
- The application must support millions of users.

Explain the architecture covering:

1. HTTPS and TLS.
2. Authentication and authorization.
3. Access and refresh tokens.
4. Android Keystore.
5. API security.
6. Encryption of sensitive information.
7. Structured logging.
8. Metrics and dashboards.
9. Distributed tracing.
10. Alerting and incident investigation.

You should be able to explain why each component exists, not just define it.

## What comes next?

@yizhiyanhua-ai/fireworks-tech-graph - npm

Week 4 – Day 28

Advanced System Design Revision

Microservices, Circuit Breaker, CQRS, Notification Architecture and Interview Practice.

Next lesson

Day 27 objective: You should now be able to design a secure Android-to-backend communication system and explain how you would monitor, debug, and maintain it in production.
