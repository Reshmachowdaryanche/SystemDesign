# Week 3 – Day 20: Load Balancing and Reverse Proxy

Week 3Day 20Beginner level

File name: `Week-3-Day-20-Load-Balancing-and-Reverse-Proxy.md`

Target: Android Developer – 7 Years Experience

Today's goal: Understand how applications handle millions of requests, distribute traffic across multiple servers, and remain available even when individual servers fail.

Writing

# Week 3 – Day 20: Load Balancing and Reverse Proxy

## 1\. Introduction

Imagine you are building an e-commerce application like Amazon.

Initially, your application has:

- 1 Android application.
- 1 backend server.
- 1 database.

Whenever a user opens the application, Android sends API requests to the backend server.

Everything works fine when you have 100 users.

But imagine your application becomes popular.

Now you have:

- 10,000 users.
- 1,00,000 users.
- 10,00,000 users.

Can a single backend server handle all these requests?

Maybe, depending on its capacity, but eventually it may become overloaded.

What happens if that server crashes?

Your entire application backend becomes unavailable.

How do we solve this?

We introduce multiple backend servers and distribute incoming requests between them.

This is called **Load Balancing**.

Today's topics:

1. What is load balancing?
2. Why do we need a load balancer?
3. Vertical vs horizontal scaling.
4. What is a reverse proxy?
5. Forward proxy vs reverse proxy.
6. Layer 4 vs Layer 7 load balancing.
7. Load balancing algorithms.
8. Health checks.
9. Sticky sessions.
10. SSL termination.
11. Nginx basics.
12. High availability.
13. Real-world system architecture.
14. Android developer perspective.

---

# 2\. What Is Load Balancing?

Load balancing means distributing incoming network requests across multiple backend servers.

The component responsible for distributing requests is called a **Load Balancer**.

### Without Load Balancer

Imagine we have only one backend server.

```
ANDROID APP
     |
     v
  SERVER 1
     |
     v
  DATABASE
```

All users send requests to Server 1.

Problems:

- Server 1 may become overloaded.
- If Server 1 crashes, the application stops working.
- Scaling becomes difficult.

### With Load Balancer

Now imagine we have three backend servers.

```
ANDROID APP
     |
     v
 LOAD BALANCER
     |
  +--+--+
  |  |  |
  v  v  v
 S1  S2  S3
  |  |  |
  +--+--+
     |
     v
  DATABASE
```

The load balancer distributes incoming requests among the available servers.

For example:

- Request 1 → Server 1.
- Request 2 → Server 2.
- Request 3 → Server 3.
- Request 4 → Server 1.

Now all requests do not go to a single server.

### Simple Definition

A load balancer is a component that distributes incoming traffic across multiple servers to improve scalability, availability and performance.

---

# 3\. Why Do We Need Load Balancing?

There are five major reasons.

## 3.1 Handle More Traffic

Imagine one server can comfortably handle 1,000 requests per second.

Your application receives 5,000 requests per second.

One server may not be sufficient.

We can add more servers.

```
LOAD BALANCER
     |
+----+----+----+----+
|    |    |    |    |
v    v    v    v    v
```

S1 S2 S3 S4 S5

Traffic is distributed among multiple servers.

Note: Actual capacity depends on the application, hardware and request complexity. Five servers do not necessarily provide exactly five times the capacity.

## 3.2 Improve Availability

Imagine Server 1 crashes.

Without load balancing:

The application becomes unavailable.

With load balancing:

The load balancer detects that Server 1 is unhealthy and stops sending new requests to it.

```
LOAD BALANCER
     |
+----+----+
|    |    |
v    v    v
```

S1 S2 S3 X OK OK

Server 1 is unavailable.

Requests are routed to Server 2 and Server 3.

This improves availability.

## 3.3 Improve Performance

A single server may become overloaded.

Distributing requests can reduce the load on individual servers.

This can improve response times.

## 3.4 Easy Horizontal Scaling

Suppose your application initially has two servers.

Later, traffic increases.

You can add more servers without changing the Android application.

```
INITIAL

LOAD BALANCER
     |
   +---+
   |   |
   S1  S2


AFTER SCALING

LOAD BALANCER
     |
+----+----+----+
|    |    |    |
S1   S2   S3   S4
```

## 3.5 Fault Tolerance

If one server fails, other healthy servers can continue handling requests.

However, the load balancer itself must also be designed for high availability.

---

# 4\. What Is Scaling?

Scaling means increasing the capacity of a system to handle more workload.

There are two major types.

## 4.1 Vertical Scaling

Vertical scaling means increasing the resources of an existing server.

For example:

Initially:

- 4 GB RAM.
- 2 CPU cores.

After scaling:

- 16 GB RAM.
- 8 CPU cores.

We increase the power of the same machine.

### Architecture

```
BEFORE

+-------------+
|   SERVER    |
|  4 GB RAM   |
|  2 CPU      |
+-------------+

        |
        v

AFTER

+-------------+
|   SERVER    |
|  16 GB RAM  |
|  8 CPU      |
+-------------+
```

### Advantages

- Easy to implement.
- No major architecture changes.
- Useful for small applications.
- Easier to manage initially.

### Disadvantages

- Hardware has limits.
- Can become expensive.
- May require downtime.
- Single-server failure remains a risk.

## 4.2 Horizontal Scaling

Horizontal scaling means adding more servers instead of increasing the power of one server.

Example:

Initially:

```
SERVER 1
```

After scaling:

```
SERVER 1
SERVER 2
SERVER 3
SERVER 4
```

Now multiple servers handle traffic.

### Advantages

- Handles more traffic.
- Better fault tolerance.
- Independent server scaling.
- Supports distributed architectures.

### Disadvantages

- More infrastructure.
- More complex deployment.
- Requires load balancing.
- Requires careful state management.

### Comparison

| Feature | Vertical Scaling | Horizontal Scaling |
| --- | --- | --- |
| Meaning | Increase server capacity | Add more servers |
| Hardware | More powerful machine | Multiple machines |
| Complexity | Lower initially | Higher |
| Fault tolerance | Limited | Better with redundancy |
| Limit | Hardware limits | Can add more nodes |
| Example | 8 GB to 32 GB RAM | 2 servers to 10 servers |

### Interview Answer

Vertical scaling means increasing the CPU, RAM or resources of an existing machine.

Horizontal scaling means adding more machines to distribute the workload.

Horizontal scaling is commonly used in large distributed systems because it supports independent scaling and fault tolerance.

---

# 5\. What Is a Reverse Proxy?

A reverse proxy is a server that receives client requests and forwards them to backend servers.

The client communicates with the reverse proxy instead of directly communicating with internal backend servers.

### Example

Imagine your application has:

- User Service.
- Order Service.
- Payment Service.

Should Android know the IP address of every backend service?

No.

Instead, Android sends requests to one public endpoint.

The reverse proxy forwards requests to the appropriate internal service.

### Architecture

```
ANDROID APP
     |
     v
REVERSE PROXY
     |
  +--+---+
  |  |   |
  v  v   v
USER ORDER PAYMENT
SVC  SVC   SVC
```

Example:

text

```text
https://api.example.com/users
https://api.example.com/orders
https://api.example.com/payments
```

The reverse proxy can route these requests to the correct services.

### Responsibilities of a Reverse Proxy

1. Request routing.
2. Load balancing.
3. SSL/TLS termination.
4. Request filtering.
5. Compression.
6. Caching.
7. Hiding internal server details.

Important: A reverse proxy and a load balancer are not the same concept. A reverse proxy forwards client requests to backend servers, while load balancing distributes traffic across multiple servers. One component can perform both responsibilities.

---

# 6\. Forward Proxy vs Reverse Proxy

This is a common interview question.

## 6.1 Forward Proxy

A forward proxy sits between clients and external servers.

The client sends a request through the proxy.

The proxy communicates with the destination server.

Example:

A company wants employees to access the internet through a controlled gateway.

```
CLIENT
   |
   v
FORWARD PROXY
   |
   v
INTERNET
   |
   v
WEBSITE
```

### Uses

- Internet access control.
- Content filtering.
- Hiding client IP addresses from destination servers.
- Corporate network policies.

## 6.2 Reverse Proxy

A reverse proxy sits between external clients and internal backend servers.

```
ANDROID APP
     |
     v
REVERSE PROXY
     |
     v
BACKEND SERVER
```

### Uses

- Load balancing.
- Request routing.
- SSL termination.
- Caching.
- Protecting internal server infrastructure.

### Comparison

| Feature | Forward Proxy | Reverse Proxy |
| --- | --- | --- |
| Represents | Client | Backend servers |
| Location | In front of clients | In front of servers |
| Main purpose | Control outgoing traffic | Manage incoming traffic |
| Example | Corporate proxy | Nginx reverse proxy |

### Easy Way to Remember

Forward proxy protects or represents clients.

Reverse proxy protects or represents servers.

---

# 7\. Layer 4 vs Layer 7 Load Balancing

To understand this, we need to know the OSI networking model.

For now, remember these two layers.

- Layer 4: Transport layer.
- Layer 7: Application layer.

## 7.1 Layer 4 Load Balancing

Layer 4 load balancing distributes traffic based on network information such as:

- Source IP address.
- Destination IP address.
- TCP port.
- Transport protocol.

It does not need to understand the HTTP URL or application-level request content.

Example:

text

```text
api.example.com:443
```

The load balancer forwards the connection to a backend server.

### Architecture

```
CLIENT
   |
   v
LAYER 4
LOAD BALANCER
   |
+--+--+
|     |
v     v
```

S1 S2

### Advantages

- Efficient.
- Lower processing overhead.
- Suitable for high-volume network traffic.

## 7.2 Layer 7 Load Balancing

Layer 7 load balancing works at the application layer.

It understands application-level information such as:

- HTTP URL.
- HTTP method.
- Headers.
- Hostname.
- Cookies.

Example:

http

```http
GET /users
```

http

```http
GET /orders
```

The load balancer can route these requests to different backend services.

### Architecture

```
ANDROID APP
     |
     v
LAYER 7 LOAD BALANCER
     |
  +--+---+
  |      |
  v      v
USER    ORDER
SERVICE SERVICE
```

### Comparison

| Feature | Layer 4 | Layer 7 |
| --- | --- | --- |
| Layer | Transport | Application |
| Understands HTTP URL | No | Yes |
| Routing based on | IP, port, protocol | URL, headers, cookies |
| Processing overhead | Usually lower | Usually higher |
| Example | TCP load balancing | HTTP reverse proxy |

### Interview Answer

Layer 4 load balancing distributes traffic using network and transport-level information such as IP addresses and ports.

Layer 7 load balancing understands application-level information such as HTTP paths, headers and hostnames, allowing more intelligent routing.

---

# 8\. Load Balancing Algorithms

A load balancer needs a strategy to decide which backend server should receive the next request.

This strategy is called a load balancing algorithm.

Let's understand the most common algorithms.

## 8.1 Round Robin

Requests are distributed one after another in a circular order.

Suppose we have three servers.

text

```text
SERVER 1
SERVER 2
SERVER 3
```

Incoming requests:

text

```text
Request 1 -> Server 1
Request 2 -> Server 2
Request 3 -> Server 3
Request 4 -> Server 1
Request 5 -> Server 2
Request 6 -> Server 3
```

### Advantages

- Simple.
- Easy to implement.
- Distributes requests evenly when requests have similar processing costs.

### Disadvantages

- Does not consider current server load.
- Some requests may take much longer than others.

## 8.2 Weighted Round Robin

In weighted round robin, each server gets a weight based on its capacity.

Example:

text

```text
Server 1 -> Weight 3
Server 2 -> Weight 2
Server 3 -> Weight 1
```

Server 1 receives more requests because it has a higher weight.

Useful when servers have different capacities.

## 8.3 Least Connections

The load balancer sends the next request to the server with the fewest active connections.

Example:

text

```text
Server 1 -> 10 active connections
Server 2 -> 4 active connections
Server 3 -> 7 active connections
```

The next connection goes to Server 2.

Useful when connections have different durations.

## 8.4 Weighted Least Connections

This considers both:

- Server capacity.
- Number of active connections.

A more powerful server can handle more connections.

## 8.5 IP Hash

The load balancer uses the client's IP address to determine which backend server receives the request.

Example:

text

```text
Client A -> Server 1
Client B -> Server 2
Client C -> Server 3
```

The same client IP may be mapped to the same server, depending on the algorithm and server pool.

This can help maintain session affinity, but it is not a guarantee of user-level identity.

## 8.6 Random

The load balancer randomly selects a backend server.

This is simple and can distribute traffic reasonably well in some environments.

### Algorithm Comparison

| Algorithm | How It Works |
| --- | --- |
| Round Robin | Takes turns |
| Weighted Round Robin | Takes turns based on weights |
| Least Connections | Chooses fewest active connections |
| Weighted Least Connections | Considers capacity and connections |
| IP Hash | Uses client IP |
| Random | Selects randomly |

### Interview Question

Which load balancing algorithm would you choose?

**Answer:**

It depends on the application.

- Round Robin is useful when servers have similar capacity and requests have similar processing costs.
- Weighted Round Robin is useful when servers have different capacities.
- Least Connections is useful when request or connection durations vary.
- IP Hash can help with session affinity.

There is no universally best algorithm.

---

# 9\. What Is a Health Check?

A health check is a mechanism used to determine whether a backend server is healthy and ready to receive traffic.

Imagine we have three servers.

text

```text
Server 1 -> Healthy
Server 2 -> Unhealthy
Server 3 -> Healthy
```

The load balancer should avoid sending new requests to Server 2.

### Architecture

```
LOAD BALANCER
     |
     +---- Health Check
     |
  +--+--+
  |  |  |
  v  v  v
 S1  S2  S3
 OK  X   OK
```

The load balancer continues using Server 1 and Server 3.

## 9.1 Types of Health Checks

### 1\. Liveness Check

Checks whether the application process is alive.

Example:

Is the application running?

### 2\. Readiness Check

Checks whether the application is ready to receive traffic.

Example:

Has the application completed startup?

Can it serve requests?

### Example API

http

```http
GET /health
```

Response:

json

```json
{
  "status": "UP"
}
```

A real health check can be more detailed, but should not unnecessarily expose sensitive internal information.

### Interview Answer

A health check verifies whether a backend instance is healthy or ready to serve requests. If a server becomes unhealthy, the load balancer can stop routing new traffic to it.

---

# 10\. What Are Sticky Sessions?

Normally, a load balancer can send different requests from the same client to different backend servers.

But imagine your application stores session information only in local server memory.

Example:

Request 1:

text

```text
User Login -> Server 1
```

Server 1 stores the session in its memory.

Request 2:

text

```text
Get Profile -> Server 2
```

Server 2 does not know about the session stored in Server 1.

The user may appear logged out.

### Solution: Sticky Sessions

Sticky sessions, also called session affinity, try to route requests from the same client to the same backend server.

### Architecture

```
CLIENT A
   |
   v
LOAD BALANCER
   |
   v
SERVER 1

CLIENT B
   |
   v
LOAD BALANCER
   |
   v
SERVER 2
```

### Advantages

- Helps when sessions are stored locally.
- Can simplify some legacy architectures.

### Disadvantages

- Creates uneven traffic distribution.
- Server failure can affect the user's session.
- Makes scaling and failover more complicated.

### Better Approach: Shared Session Storage

Instead of storing sessions inside individual backend servers, store session information in shared storage such as Redis.

```
SERVER 1 ----+
            |
SERVER 2 ---+----> REDIS
            |
SERVER 3 ---+
```

Now any backend server can retrieve the session.

This is generally a better approach for horizontally scaled stateless services.

---

# 11\. What Is SSL/TLS Termination?

When an Android application communicates with a backend using HTTPS, the communication is encrypted using TLS.

Example:

text

```text
https://api.example.com
```

The load balancer or reverse proxy can handle TLS encryption and decryption.

This is called TLS termination.

### Without TLS Termination at Load Balancer

```
ANDROID APP
     |
  HTTPS
     |
     v
  SERVER 1
```

Every backend server handles TLS connections.

### With TLS Termination

```
ANDROID APP
     |
   HTTPS
     |
     v
LOAD BALANCER
  TLS TERMINATION
     |
  +--+--+
  |     |
  v     v
 S1    S2
```

The load balancer decrypts incoming HTTPS traffic and forwards requests to backend servers.

The connection from the load balancer to the backend may use HTTP or HTTPS.

For sensitive systems, HTTPS should generally be maintained between internal services as well.

### Advantages

- Centralized certificate management.
- Reduces TLS processing on backend servers.
- Simplifies certificate renewal and configuration.

### Important Interview Point

TLS termination does not mean all communication is automatically secure.

The security of the connection between the load balancer and backend servers must also be considered.

---

# 12\. What Is Nginx?

Nginx is a web server that can also work as:

- Reverse proxy.
- Load balancer.
- HTTP cache.
- TLS termination point.

It is commonly used in production systems.

### Example Architecture

```
ANDROID APP
     |
     v
   NGINX
     |
  +--+--+
  |     |
  v     v
SERVER  SERVER
  1       2
```

Nginx receives incoming HTTP requests and forwards them to configured backend servers.

## 12.1 Simple Nginx Load Balancing Example

nginx

```nginx
http {
    upstream backend_servers {
        server 10.0.0.1:8080;
        server 10.0.0.2:8080;
        server 10.0.0.3:8080;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend_servers;
        }
    }
}
```

### Explanation

nginx

```nginx
upstream backend_servers
```

Defines the group of backend servers.

nginx

```nginx
server 10.0.0.1:8080;
```

Adds a backend server to the group.

nginx

```nginx
location /
```

Matches incoming requests.

nginx

```nginx
proxy_pass
```

Forwards requests to the backend server group.

By default, Nginx uses weighted round-robin behavior for this upstream configuration.

### Other Nginx Load Balancing Methods

nginx

```nginx
upstream backend_servers {
    least_conn;

    server 10.0.0.1:8080;
    server 10.0.0.2:8080;
}
```

This selects the server with the fewest active connections.

---

# 13\. What Is High Availability?

High availability means designing a system so that it remains operational even when individual components fail.

Imagine this architecture:

```
ANDROID APP
     |
     v
LOAD BALANCER
     |
  +--+--+
  |     |
  v     v
 S1    S2
  |     |
  +--+--+
     |
     v
  DATABASE
```

What if Server 1 crashes?

Server 2 can continue handling traffic.

But what if the load balancer itself crashes?

The entire system may become unavailable.

Therefore, the load balancer should also have redundancy.

### High Availability Architecture

```
ANDROID APP
     |
     v
PUBLIC ENDPOINT
     |
  +--+--+
  |     |
  v     v
 LB1   LB2
  |     |
  +--+--+
     |
  +--+--+
  |     |
  v     v
 S1    S2
```

LB1 and LB2 represent redundant load balancer instances. A managed load balancing service or suitable failover mechanism can provide the public endpoint.

### Important

High availability requires more than multiple backend servers.

You must also consider:

- Load balancer availability.
- Database availability.
- Network failures.
- Storage availability.
- Deployment failures.
- Monitoring and recovery.

---

# 14\. Real-World System Design Example: E-Commerce Application

Let's design a simplified architecture for an e-commerce application.

Requirements:

1. Users should browse products.
2. Users should place orders.
3. Users should make payments.
4. Users should receive notifications.
5. System should handle high traffic.

### Architecture

```
+------------------+
|   ANDROID APP    |
+------------------+
         |
         | HTTPS
         v
+------------------+
|   LOAD BALANCER  |
+------------------+
         |
         v
+------------------+
|  API GATEWAY /   |
|  REVERSE PROXY   |
+------------------+
         |
   +-----+-----+
   |     |     |
   v     v     v
 USER  ORDER  PRODUCT
 SVC   SVC    SVC
   |     |     |
   +-----+-----+
         |
         v
     DATABASE
         |
         v
    MESSAGE QUEUE
         |
   +-----+-----+
   |     |     |
   v     v     v
 EMAIL PAYMENT NOTIFICATION
 WORKER WORKER WORKER
```

### Explanation

1. Android sends HTTPS requests.
2. Load balancer distributes traffic.
3. API gateway or reverse proxy routes requests.
4. Backend services process requests.
5. Database stores application data.
6. Message queue handles asynchronous tasks.
7. Background workers process messages.

### Why Is This Architecture Scalable?

- Multiple backend instances handle incoming requests.
- Load balancing distributes traffic.
- Services can scale independently.
- Message queues handle background processing.
- Failed backend instances can be removed from rotation.
- Redundant infrastructure can improve availability.

---

# 15\. Android Developer Perspective

As an Android developer, you may not configure Nginx or AWS load balancers directly.

But understanding load balancing helps you understand how your API requests are handled in production.

### Example

Android sends:

http

```http
GET /api/products
Host: api.example.com
```

The request flow:

1. Android sends an HTTPS request.
2. DNS resolves the API hostname.
3. The request reaches the load balancing infrastructure.
4. A healthy backend server is selected.
5. The backend processes the request.
6. The response returns to Android.

### Important Android Considerations

#### 1\. Don't Depend on a Specific Backend IP

Use configured API hostnames rather than hardcoding an individual backend server IP.

#### 2\. Handle Timeouts

One backend server may be slow.

Your networking layer should have appropriate connection, read and write timeouts.

#### 3\. Handle Retry Carefully

If an API request fails, retry only when appropriate.

For example:

- GET request: often safe to retry.
- Payment POST request: do not blindly retry without idempotency protection.

#### 4\. Handle Server Errors

Your Android application should gracefully handle:

- HTTP 500.
- HTTP 502.
- HTTP 503.
- HTTP 504.

#### 5\. Understand Stateless APIs

If backend requests can be handled by different servers, the backend should not depend on local in-memory state for user sessions.

Android should send the required authentication information with each request, typically using an access token.

---

# 16\. Important HTTP Status Codes Related to Load Balancing

| Status Code | Meaning | Example |
| --- | --- | --- |
| 500 | Internal Server Error | Backend application error |
| 502 | Bad Gateway | Proxy receives an invalid response from upstream |
| 503 | Service Unavailable | Service temporarily unavailable |
| 504 | Gateway Timeout | Proxy does not receive an upstream response in time |

### Interview Example

What is the difference between 502 and 504?

**502 Bad Gateway:** A proxy or gateway received an invalid response from an upstream server.

**504 Gateway Timeout:** A proxy or gateway did not receive a timely response from an upstream server.

---

# 17\. Common Interview Mistakes

1. Thinking a load balancer and reverse proxy are exactly the same.
2. Confusing vertical scaling with horizontal scaling.
3. Thinking round robin always distributes actual workload equally.
4. Forgetting health checks.
5. Ignoring load balancer failure.
6. Using sticky sessions without understanding the drawbacks.
7. Assuming TLS termination means every internal connection is encrypted.
8. Thinking adding more servers automatically solves database bottlenecks.
9. Ignoring session management in horizontally scaled applications.
10. Retrying every failed Android API request without considering idempotency.

---

# 18\. Important Interview Questions and Answers

### Q1. What is a load balancer?

A load balancer distributes incoming requests across multiple backend servers to improve scalability, availability and performance.

### Q2. Why do we need a load balancer?

To distribute traffic, prevent individual servers from becoming overloaded, and route traffic away from unhealthy instances.

### Q3. What is horizontal scaling?

Adding more servers to handle increasing traffic.

### Q4. What is vertical scaling?

Increasing the resources of an existing server, such as CPU and RAM.

### Q5. What is a reverse proxy?

A reverse proxy receives client requests and forwards them to internal backend servers.

### Q6. What is the difference between a forward proxy and reverse proxy?

A forward proxy represents clients, while a reverse proxy represents backend servers.

### Q7. What is Layer 4 load balancing?

It distributes traffic using transport-level information such as IP addresses and ports.

### Q8. What is Layer 7 load balancing?

It distributes traffic using application-level information such as HTTP paths, headers and hostnames.

### Q9. What is round robin?

It distributes requests among servers in a circular sequence.

### Q10. What is least connections?

It sends the next connection to the server with the fewest active connections.

### Q11. What is a health check?

A mechanism used to determine whether a server is healthy and ready to receive traffic.

### Q12. What are sticky sessions?

A mechanism that attempts to route requests from the same client to the same backend server.

### Q13. What is TLS termination?

The process of decrypting incoming TLS traffic at a proxy or load balancer instead of at each backend application server.

### Q14. What is Nginx?

Nginx is a web server that can also work as a reverse proxy, load balancer and HTTP cache.

### Q15. What happens if one backend server crashes?

The load balancer can detect the failure and stop routing new traffic to that server, while healthy servers continue handling requests.

### Q16. Can a load balancer become a single point of failure?

Yes. If only one load balancer exists, its failure can make the application unavailable. High availability requires redundancy.

### Q17. What is the difference between a load balancer and an API gateway?

A load balancer distributes traffic across servers. An API gateway provides API-specific capabilities such as routing, authentication, rate limiting and request transformation. Some products can perform both roles.

### Q18. Does load balancing solve database bottlenecks?

No. It distributes application traffic but does not automatically scale the database. Database optimization, replication, caching and partitioning may be needed.

### Q19. Why are stateless backend services useful?

Any healthy backend instance can handle an incoming request without depending on session data stored only in another instance's memory. This simplifies horizontal scaling.

### Q20. How does load balancing improve availability?

It can route traffic away from unhealthy servers and distribute requests among healthy instances, allowing the application to continue operating when individual servers fail.

---

# 19\. Scenario-Based Interview Questions

## Scenario 1: One Backend Server Is Overloaded

Your application receives 50,000 requests per second, and one backend server cannot handle the traffic.

**Expected answer:**

Use horizontal scaling by adding multiple backend instances and distribute traffic through a load balancer.

## Scenario 2: One Server Crashes

You have three backend servers. One suddenly crashes.

**Expected answer:**

The load balancer should detect the unhealthy instance through health checks and stop sending new requests to it.

## Scenario 3: User Gets Logged Out Randomly

Your application has three backend servers. User login works, but subsequent requests sometimes return unauthorized.

**Expected answer:**

If sessions are stored in individual server memory, different requests may reach different servers. Use shared session storage such as Redis or use stateless token-based authentication.

## Scenario 4: Requests Take Different Amounts of Time

You have three servers. Some requests take 10 seconds, while others take 100 milliseconds.

Which algorithm could help?

**Expected answer:**

Least connections may help when active connection duration is a useful indicator of server load. More advanced load-aware strategies may be appropriate depending on the system.

## Scenario 5: Load Balancer Crashes

Your backend servers are healthy, but the load balancer crashes.

**Expected answer:**

Use redundant load balancers or a managed highly available load balancing service with a reliable public endpoint.

## Scenario 6: Database Is Overloaded

You have added ten backend servers, but the database is still overloaded.

Will adding more application servers solve the issue?

**Expected answer:**

No. More application servers can increase database pressure. Optimize queries, use caching, consider read replicas and database scaling strategies.

---

# 20\. Day 20 – Quick Revision Cheat Sheet

| Concept | Simple Meaning |
| --- | --- |
| Load Balancer | Distributes incoming traffic |
| Vertical Scaling | Increase capacity of one server |
| Horizontal Scaling | Add more servers |
| Reverse Proxy | Forwards client requests to backend servers |
| Forward Proxy | Represents clients |
| Layer 4 | IP, port and transport-level routing |
| Layer 7 | Application-level routing |
| Round Robin | Circular distribution |
| Weighted Round Robin | Distribution based on server weights |
| Least Connections | Chooses server with fewest active connections |
| IP Hash | Uses client IP for server selection |
| Health Check | Checks server health |
| Liveness | Checks whether process is alive |
| Readiness | Checks whether server can accept traffic |
| Sticky Session | Routes client to the same backend |
| TLS Termination | Handles TLS at proxy/load balancer |
| Nginx | Web server and reverse proxy |
| High Availability | System continues despite individual failures |
| 502 | Invalid upstream response |
| 503 | Service unavailable |
| 504 | Upstream response timeout |

---

# 21\. Final Learning Checklist

## Fundamentals

- [ ]Understand load balancing.
- [ ]Understand why load balancing is needed.
- [ ]Understand vertical scaling.
- [ ]Understand horizontal scaling.

## Proxy

- [ ]Understand reverse proxy.
- [ ]Understand forward proxy.
- [ ]Understand their differences.
- [ ]Understand Nginx.

## Load Balancing

- [ ]Understand Layer 4.
- [ ]Understand Layer 7.
- [ ]Understand Round Robin.
- [ ]Understand Weighted Round Robin.
- [ ]Understand Least Connections.
- [ ]Understand IP Hash.

## Reliability

- [ ]Understand health checks.
- [ ]Understand liveness and readiness.
- [ ]Understand sticky sessions.
- [ ]Understand shared session storage.
- [ ]Understand TLS termination.
- [ ]Understand high availability.

## System Design

- [ ]Draw a basic load balancing architecture.
- [ ]Explain what happens when a backend server crashes.
- [ ]Explain what happens when a load balancer crashes.
- [ ]Explain how multiple backend servers handle traffic.
- [ ]Explain how Android requests reach backend servers.

---

# 22\. What Is Next?

## Week 3 – Day 21: Rate Limiting and Throttling

Tomorrow we will learn:

1. What is rate limiting?
2. Why do we need rate limiting?
3. Rate limiting vs throttling.
4. Fixed Window algorithm.
5. Sliding Window algorithm.
6. Token Bucket algorithm.
7. Leaky Bucket algorithm.
8. API rate limiting.
9. Redis-based rate limiting.
10. HTTP 429 Too Many Requests.
11. Handling rate limits in Android.
12. Real-world API protection architecture.

**Day 20 Complete – Load Balancing and Reverse Proxy.**

## Before moving to Day 21

Try answering these five questions without looking at your notes:

1. What is the difference between horizontal and vertical scaling?
2. What is the difference between a reverse proxy and a load balancer?
3. Explain Round Robin vs Least Connections.
4. What happens when one backend server crashes?
5. Why is stateless architecture useful when using multiple backend servers?

My recommendation: Since you are an Android developer, pay special attention to how a request travels from Android → DNS → Load Balancer → API Gateway → Backend Server. Understanding this complete flow will help you connect your Android experience with backend system design.
