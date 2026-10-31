# Week 2 – Day 8: Load Balancing

Week 2Day 8Beginner level

File name: `Week-2-Day-8-Load-Balancing.md`

Today's goal: Understand how applications handle thousands or millions of incoming requests without overloading a single server.

The notes below start from absolute basics. You don't need previous backend knowledge.

Writing

# Week 2 – Day 8: Load Balancing

**Topic:** Scalability and Distributed Systems\
**Level:** Beginner\
**Target:** Android Developer – 7 Years Experience

---

# 1\. First, Understand What a Server Is

Before understanding Load Balancing, we need to understand what a server is.

Imagine you are building a Notes application.

Your Android application allows users to:

- Create notes.
- View notes.
- Update notes.
- Delete notes.

Where will all these notes be stored?

We need a backend server.

## 1.1 What Is a Server?

A server is a computer or software application that receives requests from clients and provides responses.

For example:

1. You open your Android Notes application.
2. You request all your notes.
3. Android sends an API request to the backend server.
4. The server processes the request.
5. The server retrieves notes from the database.
6. The server sends the response to Android.

Architecture:

text

```text
       ANDROID APPLICATION
                |
                | HTTP Request
                |
                v
          BACKEND SERVER
                |
                v
             DATABASE
                |
                v
          Backend Response
                |
                v
       ANDROID APPLICATION
```

The Android application is the client.

The backend is the server.

The communication between them happens through a network.

## 1.2 What Happens Inside a Server?

A server has limited resources.

For example:

- CPU: Processes instructions.
- RAM: Holds data temporarily.
- Storage: Stores files and other data.
- Network: Handles incoming and outgoing communication.

Every incoming request consumes some resources.

For example, when 1,000 users request their notes simultaneously, the server must process 1,000 requests.

As the number of requests increases, resource usage also increases.

This brings us to our next concept.

---

# 2\. What Is a Bottleneck?

A bottleneck is a component that limits the overall performance of a system.

Imagine a restaurant.

There are:

- 100 customers.
- 1 cashier.
- 1 billing counter.

All customers must wait for the same cashier.

Even if the restaurant has enough tables and food, the cashier becomes the bottleneck.

The same thing can happen in backend systems.

Example:

text

```text
         10,000 USERS
               |
               |
               v
        SINGLE SERVER
               |
               v
           DATABASE
```

All 10,000 users send requests to one server.

The server has limited CPU, RAM and network capacity.

Eventually:

- Requests start waiting.
- Response time increases.
- CPU usage becomes high.
- Memory usage increases.
- Some requests may time out.
- Users experience a slow application.

The server becomes a bottleneck.

**Important:** A bottleneck can occur in the application server, database, network, cache or another dependency.

---

# 3\. What Is Load Balancing?

Load balancing is the process of distributing incoming requests across multiple servers.

Instead of sending all requests to one server, we distribute them across several servers.

The component responsible for this is called a Load Balancer.

## 3.1 Without Load Balancing

Imagine 9,000 users send requests.

We have only one backend server.

text

```text
           9,000 USERS
                |
                v
          BACKEND SERVER
```

All requests go to the same server.

This can overload the server.

## 3.2 With Load Balancing

Now imagine we have three backend servers.

We introduce a Load Balancer.

text

```text
             9,000 USERS
                  |
                  v
             LOAD BALANCER
              /    |    \
             /     |     \
            v      v      v
         SERVER  SERVER  SERVER
            1      2       3
```

The load balancer distributes incoming requests among the available servers.

For example:

| Server | Requests |
| --- | --- |
| Server 1 | 3,000 |
| Server 2 | 3,000 |
| Server 3 | 3,000 |
| Total | 9,000 |

This is a simplified example assuming equal capacity and equal distribution.

Now each server handles fewer requests.

## 3.3 Definition

**Load Balancing is a technique used to distribute incoming network traffic across multiple backend servers to improve performance, scalability and availability.**

## 3.4 Why Do We Need Load Balancing?

1. **Performance:** Prevents one server from receiving all requests.
2. **Scalability:** Allows us to add more servers when traffic increases.
3. **Availability:** Traffic can be redirected when a server becomes unhealthy.
4. **Fault Tolerance:** Helps the application continue working when individual servers fail.
5. **Resource Utilization:** Helps distribute workloads across available servers.

---

# 4\. How Does a Load Balancer Work?

Let's understand the complete request flow.

Imagine a user opens the Notes application.

Step 1: Android sends an HTTP request.

Step 2: The request reaches the Load Balancer.

Step 3: The Load Balancer checks its configured backend servers.

Step 4: It selects an appropriate healthy server.

Step 5: It forwards the request to that server.

Step 6: The backend processes the request.

Step 7: The backend sends a response.

Step 8: The Load Balancer forwards the response to Android.

Diagram:

text

```text
         ANDROID APPLICATION
                  |
                  |
              HTTP Request
                  |
                  v
            LOAD BALANCER
                  |
          Selects a Server
                  |
          +-------+-------+
          |       |       |
          v       v       v
        API 1   API 2   API 3
          |       |       |
          +-------+-------+
                  |
                  v
               DATABASE
                  |
                  v
             API RESPONSE
                  |
                  v
            LOAD BALANCER
                  |
                  v
         ANDROID APPLICATION
```

The load balancer acts as an entry point for backend traffic.

The Android application does not need to know which particular backend server will handle its request.

It communicates with the load balancer's public endpoint.

---

# 5\. Load Balancing Algorithms

How does a load balancer decide which server should receive a request?

It uses a load balancing algorithm.

Let's understand the commonly used algorithms.

## 5.1 Round Robin

Round Robin distributes requests one after another across the available servers.

Imagine we have three servers:

- Server A
- Server B
- Server C

Requests are distributed like this:

text

```text
Request 1  -> Server A
Request 2  -> Server B
Request 3  -> Server C
Request 4  -> Server A
Request 5  -> Server B
Request 6  -> Server C
Request 7  -> Server A
```

The process repeats.

Advantages:

- Simple to implement.
- Easy to understand.
- Works well when servers have similar capacity and requests have similar processing costs.

Disadvantages:

- Does not consider current server workload.
- Does not consider how long a request takes to process.

Example:

If Server A is already busy, Round Robin may still send another request to it.

## 5.2 Weighted Round Robin

Weighted Round Robin is an extension of Round Robin.

Different servers receive different weights based on their capacity.

Example:

| Server | Capacity | Weight |
| --- | --- | --- |
| Server A | High | 3 |
| Server B | Medium | 2 |
| Server C | Low | 1 |

The approximate distribution is:

text

```text
Server A -> 3 requests
Server B -> 2 requests
Server C -> 1 request
```

This is useful when servers have different processing capacities.

## 5.3 Least Connections

The load balancer sends a new request to the server with the fewest active connections.

Example:

| Server | Active Connections |
| --- | --- |
| Server A | 100 |
| Server B | 30 |
| Server C | 50 |

The next connection is sent to Server B.

Why?

Because Server B currently has the fewest active connections.

Advantages:

- Considers current connection count.
- Useful when requests have different durations.

Disadvantages:

- Connection count does not always represent actual CPU or memory usage.
- Tracking active connections adds complexity.

## 5.4 Weighted Least Connections

This is an extension of Least Connections.

It considers both:

- Number of active connections.
- Server capacity.

A powerful server can handle more connections than a weaker server.

## 5.5 IP Hash

The load balancer uses a hash of the client's IP address to select a backend server.

For example:

text

```text
User A IP -> Server 1
User B IP -> Server 2
User C IP -> Server 3
```

The same IP may map to the same server while the server pool and hashing configuration remain unchanged.

This can help maintain routing consistency, but it does not guarantee that all requests from one user always reach the same server in every configuration.

---

# 6\. Types of Load Balancers

Load balancers can operate at different network layers.

The two commonly discussed types are:

1. Layer 4 Load Balancer.
2. Layer 7 Load Balancer.

Before understanding them, let's understand the OSI model briefly.

## 6.1 What Is the OSI Model?

The OSI model is a conceptual model that describes network communication in seven layers.

| Layer | Name | Purpose |
| --- | --- | --- |
| 7 | Application | HTTP, application protocols |
| 6 | Presentation | Data representation and encryption concepts |
| 5 | Session | Session management concepts |
| 4 | Transport | TCP and UDP |
| 3 | Network | IP addressing and routing |
| 2 | Data Link | Local network communication |
| 1 | Physical | Physical transmission |

For our discussion, focus on Layer 4 and Layer 7.

## 6.2 Layer 4 Load Balancer

Layer 4 works at the transport layer.

It makes routing decisions using information such as:

- Source IP.
- Destination IP.
- Source port.
- Destination port.
- Transport protocol (TCP/UDP).

It does not need to understand the HTTP URL or application-level content to make its routing decision.

Example:

text

```text
Client
  |
  v
Layer 4 Load Balancer
  |
  +------> Server 1
  |
  +------> Server 2
  |
  +------> Server 3
```

Advantages:

- Efficient traffic distribution.
- Lower application-level processing overhead.
- Useful for TCP and UDP traffic.

Disadvantages:

- Cannot route based on HTTP paths such as /login or /notes.
- Has less application-level awareness.

## 6.3 Layer 7 Load Balancer

Layer 7 works at the application layer.

It understands application protocols such as HTTP.

It can make routing decisions based on:

- URL path.
- HTTP headers.
- Hostname.
- Cookies.
- Other HTTP request information.

Example:

text

```text
                 CLIENT
                   |
                   v
            LAYER 7 LOAD BALANCER
                   |
          +--------+--------+
          |                 |
          v                 v
       /notes             /users
          |                 |
          v                 v
      NOTES SERVICE     USER SERVICE
```

For example:

http

```http
GET /notes
```

Can be routed to the Notes service.

http

```http
GET /users
```

Can be routed to the User service.

Advantages:

- Application-aware routing.
- Supports path-based routing.
- Can perform HTTP-level routing policies.
- Can terminate HTTPS in supported configurations.

Disadvantages:

- More processing overhead than basic Layer 4 forwarding.
- More application-level configuration.

## 6.4 Layer 4 vs Layer 7

| Layer 4 | Layer 7 |
| --- | --- |
| Transport layer | Application layer |
| Uses IP and port information | Understands HTTP request information |
| TCP/UDP aware | HTTP/HTTPS aware |
| Cannot route based on URL path | Can route based on URL path |
| Generally lower processing overhead | More application-level processing |
| Network-level traffic distribution | Application-level traffic distribution |

**Interview tip:** If the interviewer asks you to route /notes to one service and /users to another, Layer 7 load balancing is a suitable answer.

---

# 7\. What Is a Health Check?

Imagine we have three backend servers.

text

```text
             LOAD BALANCER
              /    |    \
             v     v     v
           API 1  API 2  API 3
            UP     UP    DOWN
```

Server 3 has crashed.

Should the load balancer continue sending requests to Server 3?

No.

It needs to identify unhealthy servers.

This is where health checks are used.

## 7.1 Definition

A health check is a mechanism used to determine whether a backend server is healthy and capable of handling requests.

## 7.2 How Does It Work?

The load balancer periodically sends a health check request to each backend server.

Example:

http

```http
GET /health
```

A healthy server might respond:

json

```json
{
  "status": "UP"
}
```

If the server repeatedly fails its configured health checks, the load balancer can mark it unhealthy and stop sending new traffic to it.

When the server becomes healthy again, it may be added back to the available server pool.

## 7.3 Types of Health Checks

### Liveness Check

Checks whether the application process is alive.

### Readiness Check

Checks whether the application is ready to receive traffic.

For example, the application might be running but still initializing its database connection.

In that case:

- Liveness: UP.
- Readiness: NOT READY.

This distinction is important in production systems.

---

# 8\. What Happens When a Server Fails?

Let's understand with an example.

Initially:

text

```text
          LOAD BALANCER
           /    |    \
          v     v     v
        API 1  API 2  API 3
         UP     UP     UP
```

Now API 2 crashes.

text

```text
          LOAD BALANCER
           /    |    \
          v     v     v
        API 1  API 2  API 3
         UP    DOWN    UP
```

The load balancer detects that API 2 is unhealthy.

It stops sending new requests to API 2.

New traffic is distributed across API 1 and API 3.

text

```text
          LOAD BALANCER
             /     \
            v       v
          API 1    API 3
           UP       UP
```

This improves availability.

Important: Requests already being processed by the failed server may still fail. Clients may need to retry safe operations.

---

# 9\. What Is Horizontal Scaling?

Now let's connect Load Balancing with Scalability.

Suppose your application initially has one backend server.

text

```text
       USERS
         |
         v
      SERVER 1
```

Traffic increases.

You upgrade the existing server with more CPU and RAM.

This is called Vertical Scaling.

Later, traffic increases further.

Instead of continuously upgrading one server, you add more servers.

This is called Horizontal Scaling.

text

```text
              USERS
                |
                v
          LOAD BALANCER
           /     |     \
          v      v      v
       SERVER 1 SERVER 2 SERVER 3
```

## 9.1 Vertical vs Horizontal Scaling

| Vertical Scaling | Horizontal Scaling |
| --- | --- |
| Increase resources of one server | Add more servers |
| More CPU and RAM | More server instances |
| Simpler initially | More distributed complexity |
| Has hardware limits | Can expand by adding instances |
| Can create a single-server dependency | Can improve availability when properly designed |

## 9.2 Important Interview Point

Adding more backend servers does not automatically solve every scalability problem.

For example:

text

```text
          LOAD BALANCER
          /     |     \
         v      v      v
       API 1   API 2   API 3
          \     |     /
           \    |    /
            DATABASE
```

All backend servers still use the same database.

If the database cannot handle the increased workload, it becomes the new bottleneck.

Therefore, we must also think about:

- Database indexing.
- Database replication.
- Caching.
- Connection pooling.
- Database sharding when necessary.

We will study these topics in the next few days.

---

# 10\. What Is a Stateless Backend?

This is one of the most important concepts to understand when learning load balancing.

Imagine a user sends two requests.

Request 1 goes to Server 1.

Request 2 goes to Server 2.

What if Server 1 stores some important user session information only in its own memory?

Server 2 may not know about that session.

This can create problems.

## 10.1 Stateful Backend

A stateful backend stores session-related information locally on the server.

Example:

text

```text
Request 1
    |
    v
 Server 1
    |
    v
Session stored in Server 1 memory

Request 2
    |
    v
 Server 2
    |
    v
Session not available
```

## 10.2 Stateless Backend

A stateless backend does not depend on local in-memory session state to process each request.

For example, the client sends an access token with every authenticated request.

The backend validates the token and identifies the user.

text

```text
              ANDROID APP
                   |
             Access Token
                   |
                   v
             LOAD BALANCER
              /          \
             v            v
          SERVER 1     SERVER 2
             |            |
             +------------+
                   |
              Shared Data
```

Both servers can process requests independently.

This makes horizontal scaling easier.

Important:

Stateless does not mean the entire application stores no state.

User data still exists in databases and caches. It means the backend instance does not rely on its own local session memory for subsequent requests.

## 10.3 What Is Sticky Session?

Sometimes applications need requests from the same client to be routed to the same backend server.

This is called a sticky session or session affinity.

Example:

text

```text
User A -> Server 1
User A -> Server 1
User A -> Server 1
```

It can be useful for some legacy or stateful applications.

However, it can make traffic distribution less flexible.

A stateless architecture with shared session storage is generally easier to scale.

---

# 11\. Real-World Load Balancer Technologies

Here are some commonly used technologies.

| Technology | Description |
| --- | --- |
| NGINX | Web server and reverse proxy with load balancing capabilities |
| HAProxy | High-performance load balancer and proxy |
| AWS Elastic Load Balancing | Managed load balancing service from AWS |
| Google Cloud Load Balancing | Managed load balancing service from Google Cloud |
| Azure Load Balancer | Managed Layer 4 load balancing service |
| Kubernetes Service | Provides stable access and traffic distribution to application pods |

You don't need to master all these tools immediately.

For interviews, understand the concepts and know where they are used.

---

# 12\. Real-World Example – Notes Application

Let's connect everything with our Notes application from Day 6.

Suppose:

- 1 million registered users.
- 100,000 daily active users.
- Many users access their notes simultaneously.

We have three backend instances.

Architecture:

text

```text
                 ANDROID USERS
                       |
                       v
                HTTPS REQUESTS
                       |
                       v
                 LOAD BALANCER
                       |
             +---------+---------+
             |         |         |
             v         v         v
          NOTES API  NOTES API  NOTES API
          SERVER 1   SERVER 2   SERVER 3
             |         |         |
             +---------+---------+
                       |
                +------+------+
                |             |
                v             v
              REDIS       POSTGRESQL
              CACHE        DATABASE
```

Example flow:

1. Android requests the user's notes.
2. The request reaches the load balancer.
3. The load balancer selects a healthy backend instance.
4. The backend authenticates the user.
5. The backend checks Redis.
6. If cached data exists, it returns the response.
7. Otherwise, it queries PostgreSQL.
8. The response is returned to Android.

Now imagine Server 2 crashes.

The load balancer stops routing new requests to Server 2.

Servers 1 and 3 continue processing requests.

This is one of the major benefits of load balancing.

---

# 13\. Load Balancing – Important Interview Questions

## Q1. What is Load Balancing?

Load balancing is the process of distributing incoming traffic across multiple backend servers to improve performance, scalability and availability.

## Q2. Why do we need a Load Balancer?

To prevent a single server from becoming overloaded and to distribute traffic across available servers.

## Q3. What happens if one backend server crashes?

The load balancer detects the unhealthy server through health checks and stops routing new requests to it.

## Q4. What is Round Robin?

Round Robin distributes requests sequentially across available backend servers.

## Q5. What is Least Connections?

It routes a new connection to the server with the fewest active connections.

## Q6. What is the difference between Layer 4 and Layer 7 Load Balancing?

Layer 4 uses network and transport information such as IP addresses and ports.

Layer 7 understands application-level information such as HTTP paths, headers and hostnames.

## Q7. What is a Health Check?

A health check determines whether a backend server is healthy and ready to receive traffic.

## Q8. What is Horizontal Scaling?

Horizontal scaling means increasing system capacity by adding more servers or instances.

## Q9. What is Vertical Scaling?

Vertical scaling means increasing the resources of an existing server, such as CPU and RAM.

## Q10. What is a Stateless Backend?

A stateless backend does not depend on local server memory to maintain session state between requests.

## Q11. What is a Sticky Session?

Sticky sessions ensure requests from a particular client are routed to the same backend server for a period of time.

## Q12. Does a Load Balancer improve database performance?

Not directly. It distributes traffic across backend servers. The database may still become a bottleneck if it cannot handle the increased workload.

## Q13. Can a Load Balancer itself become a single point of failure?

Yes. If only one load balancer instance exists, its failure can affect the entire application. Production architectures use highly available load balancing solutions.

## Q14. Which algorithm is better: Round Robin or Least Connections?

It depends on the workload.

Round Robin is suitable when servers and request processing times are similar.

Least Connections can be useful when connection durations vary significantly.

---

# 14\. Android Developer Perspective

As an Android developer, you may not directly configure backend load balancers.

But you should understand how they affect your application.

### 1\. API Requests

Your Retrofit API requests usually communicate with a common backend endpoint.

The load balancer selects the backend instance.

### 2\. Retry Handling

If a request fails due to a temporary server failure, Android may retry it.

However, retry policies should use appropriate backoff and avoid duplicate effects.

### 3\. Timeouts

If backend processing takes too long, Android may receive a timeout.

A timeout does not necessarily mean the backend failed to process the request.

### 4\. Stateless Authentication

Sending an access token with authenticated requests allows different backend instances to validate requests without depending on one server's local session memory.

### 5\. Backend Failures

Android should handle appropriate HTTP errors and temporary connectivity failures gracefully.

**Remember:** Load balancing is primarily a backend infrastructure concept, but its behavior directly affects the reliability and responsiveness of your Android application.

---

# 15\. Common Mistakes to Avoid

1. Thinking that a load balancer makes the database automatically scalable.
2. Assuming Round Robin always provides equal processing load.
3. Thinking that adding more servers guarantees high availability.
4. Confusing horizontal scaling with vertical scaling.
5. Assuming a health check guarantees that every application operation will succeed.
6. Thinking that stateless means the application does not store data.
7. Retrying every failed request without considering duplicate operations.

---

# 16\. Day 8 – Quick Revision

| Concept | Meaning |
| --- | --- |
| Server | Receives requests and returns responses |
| Bottleneck | Component limiting system performance |
| Load Balancer | Distributes traffic across servers |
| Round Robin | Sequential request distribution |
| Weighted Round Robin | Distribution based on server weights |
| Least Connections | Selects server with fewer active connections |
| IP Hash | Uses client IP hash for server selection |
| Layer 4 | Network/transport-level load balancing |
| Layer 7 | Application-level load balancing |
| Health Check | Checks server health/readiness |
| Horizontal Scaling | Add more servers |
| Vertical Scaling | Increase resources of one server |
| Stateless Backend | Does not depend on local session memory |
| Sticky Session | Routes a client to the same server |
| Reverse Proxy | Receives requests and forwards them to backend servers |

---

# 17\. Self-Assessment Questions

Try answering without looking at the notes.

### Basic Understanding

1. What is a server?
2. What happens when 10,000 users send requests to one server?
3. What is a bottleneck?
4. What is load balancing?
5. Why do we need a load balancer?

### Load Balancing Algorithms

6. Explain Round Robin with an example.
7. What is Weighted Round Robin?
8. How does Least Connections work?
9. What is IP Hash?
10. Which algorithm would you choose if requests take different amounts of time?

### Architecture

11. Explain Layer 4 vs Layer 7.
12. What is a health check?
13. What happens when a backend server crashes?
14. What is horizontal scaling?
15. What is vertical scaling?
16. Can a load balancer become a single point of failure?
17. What is a stateless backend?
18. What is a sticky session?

### Real-World Thinking

19. How would you design load balancing for a Notes application?
20. What happens if one backend instance goes down?
21. What happens if all backend servers are healthy but the database is overloaded?
22. Why is stateless architecture useful for horizontal scaling?
23. How does load balancing affect Android API requests?

---

# 18\. Practical Interview Exercise

Imagine you are designing a Notes application.

Initially, your application has:

- 10,000 users.
- One backend server.
- One PostgreSQL database.

After six months, the application grows to 1 million users.

The single backend server cannot handle the incoming traffic.

**Question: How would you improve the architecture?**

Think about:

1. How many backend instances would you introduce initially?
2. Where would you place the load balancer?
3. Which load balancing algorithm would you choose?
4. How would you identify unhealthy servers?
5. How would you prevent a single load balancer failure?
6. What if the database becomes the bottleneck?

There is no single universally correct architecture. Your job in an interview is to explain your assumptions and trade-offs.

---

# 19\. Final Learning Checklist

- [ ]Understand what a server is.
- [ ]Understand bottlenecks.
- [ ]Understand load balancing.
- [ ]Understand why multiple servers are required.
- [ ]Understand Round Robin.
- [ ]Understand Weighted Round Robin.
- [ ]Understand Least Connections.
- [ ]Understand Layer 4 vs Layer 7.
- [ ]Understand health checks.
- [ ]Understand horizontal scaling.
- [ ]Understand vertical scaling.
- [ ]Understand stateless architecture.
- [ ]Understand sticky sessions.
- [ ]Explain the complete Notes application architecture.

---

# 20\. What Is Next?

**Week 2 – Day 9: Database Scaling**

Tomorrow we will learn:

- Why databases become bottlenecks.
- Vertical database scaling.
- Horizontal database scaling.
- Database replication.
- Primary-Replica architecture.
- Read replicas.
- Database sharding.
- Database partitioning.
- Replication vs Sharding.
- Real-world database architecture.

**Day 8 Complete – Load Balancing.**

## What you should remember today

There are three particularly important ideas:

1. Load Balancer: Distributes incoming requests across multiple backend servers.
2. Horizontal Scaling: Adds more servers to handle increased traffic.
3. Health Checks: Help identify unhealthy servers so new requests can be routed elsewhere.

Don't memorize all the algorithms today. First, make sure you can draw and explain the basic architecture:

`Android → Load Balancer → Multiple Backend Servers → Database`

That is the foundation for understanding tomorrow's database scaling concepts.
