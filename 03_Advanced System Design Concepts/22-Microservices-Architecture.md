# Week 4 – Day 22: Microservices Architecture

File name: `22-Microservices-Architecture.md`

Topic: Advanced Architecture and Distributed Systems Level: Beginner to Intermediate Target: Senior Android Developer – 7 Years Experience

Today's goal: Understand what microservices are, why applications move from monolithic to microservices architecture, how services communicate, and how large applications manage multiple independent services.

Today's notes start from absolute basics. You don't need previous backend development experience.

# 1. First, Understand What an Application Architecture Is

Before understanding microservices, let's understand application architecture.

Imagine you are building a Notes application.

Your application supports:

- User registration.
- Login.
- Creating notes.
- Updating notes.
- Deleting notes.
- Sharing notes.
- Sending notifications.

Initially, you might build everything inside one backend application.

For example:

text

```text
              ANDROID APP
                   |
                   v
              BACKEND APP
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
     USERS        NOTES    NOTIFICATIONS
       |           |           |
       +-----------+-----------+
                   |
                   v
                DATABASE
```

The backend application handles all business functionalities.

This is an example of a Monolithic Architecture.

As the application grows, managing everything inside one application can become difficult.

This brings us to two important architectural approaches:

1. Monolithic Architecture.
2. Microservices Architecture.

# 2. What Is Monolithic Architecture?

Let's understand monolithic architecture first.

Definition: A monolithic architecture is an application architecture where multiple business functionalities are developed and deployed together as one application.

Imagine our Notes application contains three modules:

- User Management.
- Notes Management.
- Notification Management.

All three modules are part of one backend application.

Architecture:

text

```text
                  ANDROID APP
                       |
                       v
                MONOLITHIC BACKEND
                       |
          +------------+------------+
          |            |            |
          v            v            v
      USER MODULE  NOTES MODULE  NOTIFICATION
                                    MODULE
          |            |            |
          +------------+------------+
                       |
                       v
                    DATABASE
```

Even though the application has multiple modules, they are packaged and deployed together.

## 2.1 How Does It Work?

Imagine a user creates a note.

1. Android sends an API request.
2. The backend receives the request.
3. The Notes module processes it.
4. The database stores the note.
5. The backend returns the response.

All operations happen inside the same backend application.

## 2.2 Advantages of Monolithic Architecture

1. Easy to start: One application is simpler to develop initially.
2. Simple deployment: We deploy one application.
3. Easy debugging: Many operations happen inside the same process.
4. Simple communication: Modules can communicate through internal function calls.
5. Lower infrastructure complexity: Fewer independently managed services.

## 2.3 Disadvantages of Monolithic Architecture

Imagine your application grows to 10 million users.

The backend now contains:

- User management.
- Notes management.
- Payments.
- Notifications.
- Search.
- File uploads.

Problems may start appearing.

Problem 1: Difficult maintenance

The codebase becomes large and complex.

Problem 2: Independent scaling becomes difficult

Suppose Notes functionality receives 10,000 requests per second, but User Management receives only 500 requests per second.

In a monolithic application, independently scaling just the Notes module is difficult because both modules are deployed together.

Problem 3: Deployment risk

A small change in the notification module may require deploying the entire application.

Problem 4: Technology limitations

Different modules generally share the application's runtime and technology choices.

Problem 5: Failure impact

A serious issue in one module can affect the entire application, depending on isolation and resource management.

# 3. What Are Microservices?

Now imagine our Notes application becomes very popular.

Instead of keeping all functionalities inside one backend application, we divide the application into smaller independent services.

For example:

- User Service.
- Notes Service.
- Notification Service.
- Search Service.
- File Storage Service.

Each service handles a specific business responsibility.

Architecture:

text

```text
                  ANDROID APP
                       |
                       v
                  API GATEWAY
                       |
          +------------+------------+
          |            |            |
          v            v            v
      USER SERVICE NOTES SERVICE NOTIFICATION
                                   SERVICE
          |            |            |
          v            v            v
        USER DB      NOTES DB    NOTIFICATION DB
```

Each service can be developed, deployed and scaled independently.

## 3.1 Definition

Microservices Architecture is an architectural approach in which an application is divided into small, independently deployable services, each responsible for a specific business capability.

For example:

The User Service handles user-related operations.

The Notes Service handles note-related operations.

The Notification Service handles notifications.

Each service owns its responsibility.

## 3.2 Real-World Example

Imagine you are using an e-commerce application.

It may have different services:

text

```text
                 E-COMMERCE APP
                        |
                        v
                   API GATEWAY
                        |
       +----------------+----------------+
       |        |       |       |        |
       v        v       v       v        v
     USER    PRODUCT  ORDER  PAYMENT  NOTIFICATION
    SERVICE  SERVICE SERVICE SERVICE   SERVICE
```

Each service handles a separate business capability.

For example:

- User Service manages accounts.
- Product Service manages products.
- Order Service manages orders.
- Payment Service handles payments.
- Notification Service sends updates.

# 4. Monolithic vs Microservices Architecture

This is an important interview topic.

Software Architecture Patterns - ByteByteGo Newsletter

Monolithic vs. Microservices: The Great Architecture Debate | Last9

Monolith vs Microservices vs Modular Monoliths

| Monolithic | Microservices |
| --- | --- |
| One deployable application | Multiple independently deployable services |
| Modules communicate internally | Services communicate over networks |
| Usually simpler initially | More infrastructure complexity |
| Scaling individual modules is difficult | Services can scale independently |
| Usually one shared application runtime | Services can use different runtimes |
| Simpler deployment | Independent deployments |
| Internal failures may affect the whole application | Failures can be isolated with proper design |
| Simpler monitoring | Distributed monitoring required |

Interview example:

Suppose Notes functionality receives significantly more traffic than User Management.

With microservices, we can increase the number of Notes Service instances without increasing User Service instances.

This is one of the major advantages of microservices.

# 5. What Is a Microservice?

Let's understand what makes something a microservice.

A microservice is not simply a small class or a small module.

It is an independently managed service responsible for a specific business capability.

For example, our Notes Service might provide:

text

```text
NOTES SERVICE

POST   /notes
GET    /notes
GET    /notes/{id}
PUT    /notes/{id}
DELETE /notes/{id}
```

The Notes Service owns the business logic for notes.

The User Service might provide:

text

```text
USER SERVICE

POST   /users
GET    /users/{id}
PUT    /users/{id}
DELETE /users/{id}
```

The two services can be developed and deployed separately.

## 5.1 Important Characteristics

A microservice generally has:

1. A specific business responsibility.
2. Independent deployment.
3. Network-based communication.
4. Independent scaling.
5. Clearly defined APIs or contracts.
6. Its own operational lifecycle.
7. Appropriate isolation from other services.

Important: Microservices do not have to use separate physical servers. Multiple services can run on the same machine or cluster.

# 6. What Is an API Gateway?

Imagine Android needs to communicate with five backend services.

Without an API Gateway:

text

```text
               ANDROID APP
                /   |   \
               /    |    \
              v     v     v
          USER    NOTES  PAYMENT
         SERVICE SERVICE SERVICE
```

The Android application needs to know multiple service endpoints.

This creates complexity.

Now introduce an API Gateway.

text

```text
                ANDROID APP
                     |
                     v
                API GATEWAY
                     |
          +----------+----------+
          |          |          |
          v          v          v
        USER       NOTES      PAYMENT
       SERVICE    SERVICE     SERVICE
```

## 6.1 Definition

An API Gateway is an entry point that receives client requests and routes them to appropriate backend services.

## 6.2 What Does an API Gateway Do?

Common responsibilities include:

- Request routing.
- Authentication integration.
- Rate limiting.
- Request validation.
- Logging.
- Monitoring.
- Response transformation.
- API version management.

For example:

Android sends:

http

```http
GET /api/notes
```

The API Gateway routes the request to the Notes Service.

Another request:

http

```http
GET /api/users/101
```

The gateway routes it to the User Service.

## 6.3 Why Do We Need It?

Without a gateway, clients may need to manage multiple service endpoints.

With a gateway, Android can communicate through one public entry point.

Android perspective: Your Retrofit base URL might point to the API Gateway rather than individual backend services.

# 7. What Is Service Discovery?

Now imagine our Notes Service has three running instances.

text

```text
            NOTES SERVICE
             /    |    \
            v     v     v
         INSTANCE INSTANCE INSTANCE
            1       2       3
```

How does another service know where these instances are running?

Their IP addresses or ports may change when instances restart or scale.

We need Service Discovery.

## 7.1 Definition

Service Discovery is a mechanism that allows services to find the network locations of other services dynamically.

## 7.2 How Does It Work?

Imagine we have:

- User Service.
- Notes Service.
- Notification Service.

A service registry maintains information about available service instances.

text

```text
          SERVICE REGISTRY
                 |
       +---------+---------+
       |         |         |
       v         v         v
     USER      NOTES   NOTIFICATION
    SERVICE   SERVICE    SERVICE
```

When the Notes Service needs the User Service:

1. Notes Service asks for the User Service location.
2. The registry provides an available instance.
3. Notes Service communicates with that instance.

## 7.3 Examples of Service Discovery Technologies

- Consul.
- Netflix Eureka.
- Kubernetes Service Discovery.

Important:

Service discovery can be implemented using centralized registries, DNS, platform-native mechanisms or other approaches.

# 8. Inter-Service Communication

In a microservices architecture, services need to communicate with each other.

There are two major approaches.

## 8.1 Synchronous Communication

One service sends a request to another service and waits for a response.

Example:

The Notes Service needs user information.

text

```text
        NOTES SERVICE
               |
               | HTTP Request
               v
         USER SERVICE
               |
               | Response
               v
        NOTES SERVICE
```

Common technologies:

- REST APIs.
- gRPC.

Example:

http

```http
GET /users/101
```

The Notes Service waits for the User Service response.

Advantages:

- Simple request-response communication.
- Immediate response when required.
- Easy to understand.

Disadvantages:

- Network latency.
- Dependency on the availability of another service.
- Slow downstream services can increase response times.

## 8.2 Asynchronous Communication

One service publishes a message or event without waiting for the consumer to finish processing it.

Example:

A user creates a note.

The Notes Service publishes a `NoteCreated` event.

The Notification Service consumes it.

text

```text
          NOTES SERVICE
                |
                | NoteCreated Event
                v
           MESSAGE BROKER
                |
                v
       NOTIFICATION SERVICE
                |
                v
         SEND NOTIFICATION
```

Technologies include:

- Apache Kafka.
- RabbitMQ.
- Amazon SQS.

Advantages:

- Services are less tightly coupled.
- Background processing is easier.
- Workloads can be processed independently.

Disadvantages:

- More complicated debugging.
- Message delivery and duplicate processing must be handled.
- Processing can be delayed.

We will study these communication patterns further in our distributed systems notes.

# 9. What Is Service Isolation?

Imagine the Notification Service suddenly starts consuming too much memory.

Could it affect the Notes Service?

It might, especially if both services share resources and there are no limits.

Service Isolation aims to prevent failures or resource problems in one service from spreading to other services.

Example:

text

```text
          MICROSERVICES
                |
       +--------+--------+
       |        |        |
       v        v        v
     USER     NOTES   NOTIFICATION
    SERVICE  SERVICE    SERVICE
       |        |        |
     Isolated resource and failure boundaries
```

Isolation can be achieved through:

- Separate processes.
- Resource limits.
- Separate containers.
- Independent deployments.
- Timeouts.
- Circuit breakers.
- Bulkheads.
- Separate databases where appropriate.

Important:

Microservices do not automatically guarantee isolation. Isolation must be designed and implemented.

# 10. What Is Independent Deployment?

One of the most important microservices characteristics is independent deployment.

Imagine our application has:

- User Service.
- Notes Service.
- Notification Service.

The team wants to improve notification delivery.

In a properly designed microservices architecture, the Notification Service can be updated without deploying the entire application.

text

```text
USER SERVICE
Version 1.0
      |
      |
NOTES SERVICE
Version 2.0
      |
      |
NOTIFICATION SERVICE
Version 3.0  <- Updated independently
```

Benefits:

- Faster releases.
- Smaller deployment changes.
- Independent development teams.
- Less coordination between unrelated services.

However, services must maintain compatible API contracts when communicating with one another.

# 11. Database Per Service

This is an important microservices concept.

Imagine every service directly accesses one shared database.

text

```text
        USER SERVICE
              |
        NOTES SERVICE
              |
       NOTIFICATION SERVICE
              |
              v
         SHARED DATABASE
```

Problems can occur:

- Services become tightly coupled to database schemas.
- One service's schema changes can affect others.
- Database ownership becomes unclear.
- Independent deployment becomes harder.

A common microservices approach is Database per Service.

text

```text
       USER SERVICE       NOTES SERVICE
            |                  |
            v                  v
          USER DB            NOTES DB


       NOTIFICATION SERVICE
                 |
                 v
          NOTIFICATION DB
```

Each service owns its data.

Other services access that data through APIs or events rather than directly modifying another service's database.

## 11.1 Advantages

- Independent schema management.
- Better service ownership.
- Independent scaling.
- Reduced database-level coupling.

## 11.2 Disadvantages

- Cross-service queries become difficult.
- Data duplication may occur.
- Distributed transactions are more complex.
- Data consistency requires careful design.

Important: Database per service does not necessarily mean a separate physical database server for every service. It means separate ownership and boundaries.

# 12. Advantages of Microservices

Let's summarize the major benefits.

### 1. Independent Scaling

Scale only the services experiencing high traffic.

### 2. Independent Deployment

Deploy one service without redeploying everything.

### 3. Technology Flexibility

Different services can use different languages, databases and frameworks when justified.

### 4. Fault Isolation

Properly isolated services can reduce the impact of individual failures.

### 5. Team Independence

Different teams can own different services.

### 6. Better Business Separation

Each service has a clearly defined business responsibility.

# 13. Disadvantages of Microservices

Microservices also introduce significant complexity.

### 1. Network Communication

Function calls become network requests.

Network requests can fail or become slow.

### 2. Distributed Debugging

A single Android request may travel through multiple services.

Tracing the complete request becomes harder.

### 3. Data Consistency

Different services own different databases.

Maintaining consistency across them requires additional patterns.

### 4. Deployment Complexity

Multiple services need independent deployment pipelines.

### 5. Monitoring Complexity

Every service needs logs, metrics and health monitoring.

### 6. Infrastructure Cost

More services can require more infrastructure and operational effort.

### 7. Security Complexity

Service-to-service authentication and authorization must be handled.

Interview tip: Microservices are not automatically better than monolithic architecture. For smaller applications, a well-structured monolith can be the better choice.

# 14. Real-World Example – Notes Application Using Microservices

Let's design our Notes application.

Assume:

- 10 million registered users.
- Multiple backend instances.
- Millions of notes.
- Users can share notes.
- Users receive notifications.
- Users can upload attachments.

Architecture:

text

```text
                   ANDROID USERS
                         |
                         v
                    API GATEWAY
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
      USER SERVICE   NOTES SERVICE   FILE SERVICE
          |              |              |
          v              v              v
        USER DB        NOTES DB       OBJECT STORAGE


                    NOTES SERVICE
                         |
                         | NoteCreated Event
                         v
                    MESSAGE BROKER
                         |
                         v
                NOTIFICATION SERVICE
                         |
                         v
                   NOTIFICATION DB
```

## Example: Creating a Note

Let's understand the complete request flow.

1. User opens the Android Notes application.
2. User creates a new note.
3. Android sends a `POST /notes` request.
4. API Gateway receives the request.
5. API Gateway routes it to Notes Service.
6. Notes Service validates the request and authenticates the user.
7. Notes Service stores the note in Notes Database.
8. Notes Service publishes a `NoteCreated` event.
9. Notification Service consumes the event.
10. Notification Service performs the required notification work.

The Notes Service does not need to wait for the notification delivery to finish before returning the successful note creation response, assuming notification processing is designed as a separate asynchronous operation.

This improves separation between core note creation and notification processing.

# 15. When Should We Use Microservices?

Microservices can be useful when:

- The application has many business capabilities.
- Different components need independent scaling.
- Multiple teams develop separate business domains.
- Independent deployment is important.
- The system has significant operational maturity.

Avoid introducing microservices just because they are popular.

For a small application, a monolith may be simpler, cheaper and easier to maintain.

A good architecture should solve an actual problem.

# 16. Android Developer Perspective

As an Android developer with seven years of experience, you should understand how microservices affect your application.

## 16.1 API Gateway and Retrofit

Your Android application usually communicates with a stable public API endpoint.

Example:

kotlin

```kotlin
interface NotesApi {

    @GET("api/notes")
    suspend fun getNotes(): List<Note>

    @POST("api/notes")
    suspend fun createNote(
        @Body request: CreateNoteRequest
    ): Note
}
```

The backend API Gateway routes these requests to the appropriate services.

Android does not need to know the internal IP addresses of the microservices.

## 16.2 Service Failure

Suppose the Notes Service works but the Notification Service is unavailable.

The user should ideally still be able to create a note successfully.

Notification delivery can be retried separately.

## 16.3 Network Errors

Because microservices communicate over networks, your Android application may experience:

- HTTP 500 errors.
- HTTP 502 Bad Gateway.
- HTTP 503 Service Unavailable.
- HTTP 504 Gateway Timeout.
- Network timeouts.

Handle these errors gracefully.

## 16.4 Authentication

Android generally communicates with the API Gateway using an access token.

The gateway and backend services enforce appropriate authentication and authorization policies.

## 16.5 Important Understanding

Android developers generally don't need to know the internal service topology.

They should understand:

- Which public API endpoint to call.
- How authentication works.
- How to handle errors.
- How to handle timeouts.
- How to retry safe requests.
- How backend failures affect the UI.

# 17. Important Interview Questions and Answers

### Q1. What is Microservices Architecture?

Microservices architecture divides an application into small, independently deployable services, each responsible for a specific business capability.

### Q2. What is Monolithic Architecture?

Monolithic architecture is an approach where multiple business functionalities are developed and deployed together as one application.

### Q3. What is the difference between Monolithic and Microservices?

A monolith is generally one deployable application, whereas microservices consist of multiple independently deployable services communicating over a network.

### Q4. Why do we need Microservices?

To support independent scaling, deployment, service ownership and technology flexibility as applications grow.

### Q5. What is an API Gateway?

An API Gateway is an entry point that receives client requests and routes them to appropriate backend services.

### Q6. What is Service Discovery?

Service Discovery allows services to dynamically find the network locations of other service instances.

### Q7. What is Inter-Service Communication?

It is communication between different backend services using mechanisms such as REST, gRPC or message brokers.

### Q8. What is Synchronous Communication?

The caller waits for the response from the called service.

### Q9. What is Asynchronous Communication?

The sender publishes a message or event without needing to wait for the consumer to finish processing it.

### Q10. What is Database per Service?

Each microservice owns and manages its own data rather than allowing other services to directly access its database.

### Q11. What is Service Isolation?

It is the practice of limiting the impact of failures or resource problems in one service on other services.

### Q12. Can Microservices use different programming languages?

Yes. Different services can use different technologies, provided they communicate through well-defined contracts.

### Q13. Are Microservices always better than Monolithic Architecture?

No. Microservices introduce operational and communication complexity. A monolith can be better for smaller applications.

### Q14. How do Microservices communicate?

They commonly communicate using synchronous APIs such as REST and gRPC or asynchronous messaging systems such as Kafka and RabbitMQ.

### Q15. What happens if one Microservice fails?

The impact depends on the service's role and failure-handling design. Other services may continue working if dependencies are properly isolated.

### Q16. Why is Database per Service important?

It reduces database coupling and allows services to manage their schemas and data independently.

### Q17. What is Independent Deployment?

It means deploying or updating one service without requiring deployment of the entire application.

### Q18. Does Microservices Architecture eliminate single points of failure?

No. API Gateways, databases, service registries and other infrastructure can still become single points of failure unless designed for high availability.

### Q19. What is the relationship between Microservices and Distributed Systems?

Microservices are a way of designing distributed applications. Multiple independent services communicate over a network.

### Q20. How does Android communicate with Microservices?

Typically, Android communicates with a public API Gateway endpoint. The gateway routes requests internally to appropriate services.

# 18. Practical Interview Exercise

Imagine you are designing a Notes application.

Initially, you have:

- 1 million users.
- One monolithic backend.
- One PostgreSQL database.
- Notes, users and notifications inside one application.

Now the application grows to 10 million users.

Answer these questions:

1. Would you continue with a monolith or introduce microservices?
2. Which services would you create?
3. What would be the responsibility of each service?
4. Where would you introduce an API Gateway?
5. How would services discover each other?
6. How would Notes Service communicate with Notification Service?
7. Would you use Database per Service?
8. What happens if Notification Service crashes?
9. How would you independently scale Notes Service?
10. How would Android communicate with the architecture?

Try drawing the architecture without looking at the notes.

# 19. Day 22 – Quick Revision

| Concept | Meaning |
| --- | --- |
| Architecture | Structure and organization of a software system |
| Monolith | One deployable application |
| Microservices | Independently deployable business services |
| Service | Component responsible for a business capability |
| API Gateway | Entry point for client requests |
| Service Discovery | Finds available service instances |
| Inter-Service Communication | Communication between services |
| Synchronous | Caller waits for response |
| Asynchronous | Caller need not wait for processing |
| Service Isolation | Limits failure impact |
| Independent Deployment | Deploy one service separately |
| Database per Service | Each service owns its data |
| Scalability | Ability to handle increasing workload |
| Fault Isolation | Prevents failures from spreading |

# 20. Final Learning Checklist

- [ ]Understand application architecture.
- [ ]Understand Monolithic Architecture.
- [ ]Understand Microservices Architecture.
- [ ]Explain Monolithic vs Microservices.
- [ ]Understand what a microservice is.
- [ ]Understand API Gateway.
- [ ]Understand Service Discovery.
- [ ]Understand Inter-Service Communication.
- [ ]Understand synchronous communication.
- [ ]Understand asynchronous communication.
- [ ]Understand Service Isolation.
- [ ]Understand Independent Deployment.
- [ ]Understand Database per Service.
- [ ]Understand advantages of Microservices.
- [ ]Understand disadvantages of Microservices.
- [ ]Explain the Notes application using Microservices.
- [ ]Explain how Android communicates with Microservices.

# 21. What Is Next?

## Week 4 – Day 23: Design Patterns in Distributed Systems

Tomorrow we will learn:

1. What are distributed system design patterns?
2. Circuit Breaker Pattern.
3. Retry Pattern.
4. Timeout Pattern.
5. Bulkhead Pattern.
6. Saga Pattern.
7. CQRS.
8. Event Sourcing.
9. When to use each pattern.
10. Real-world Notes application examples.
11. Interview questions and answers.
12. Practical system design exercise.

Day 22 Complete – Microservices Architecture.

## Today's five most important concepts

1. Microservices: Divide an application into independently deployable business services.
2. API Gateway: Provides a common entry point for client requests.
3. Service Discovery: Helps services find each other dynamically.
4. Independent Deployment: Services can be updated independently.
5. Database per Service: Each service owns its data.

Remember this architecture:

text

```text
ANDROID APPLICATION
         |
         v
     API GATEWAY
         |
    +----+-----+
    |    |     |
    v    v     v
  USER NOTES NOTIFICATION
 SERVICE SERVICE SERVICE
    |    |     |
    v    v     v
   DB   DB    DB
```

Our roadmap tracking remains unchanged: Week 4 – Day 22 is complete. Next is Day 23.
