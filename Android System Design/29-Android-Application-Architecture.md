# Week 5 – Day 29: Android Application Architecture

Day 29 Beginner to Advanced Senior Android Developer Interview Preparation

Today's topic is Android Application Architecture: MVC, MVP, MVVM, MVI, Clean Architecture, SOLID, Dependency Injection, Repository Pattern, Use Cases and Modularization.

This is one of the most important topics for a Senior Android Developer interview. You should be able to explain not only what these architectures are, but also why you choose one architecture over another when building a real-world Android application.

Today's learning objectives

By the end of Day 29, you should be able to:

- Understand how Android application architecture evolved.
- Compare MVC, MVP, MVVM and MVI.
- Design an application using Clean Architecture.
- Apply SOLID principles in Android.
- Understand Repository and Use Case patterns.
- Understand Dependency Injection using Hilt.
- Design scalable, modular Android applications.
- Explain architecture decisions confidently in senior interviews.

# 1. What is Application Architecture?

Imagine you are developing an e-commerce Android application.

It has the following features:

- User login and registration.
- Product listing.
- Product details.
- Shopping cart.
- Payments.
- Order history.
- Notifications.

Initially, you might write everything inside an Activity.

For example:

kotlin

```kotlin
class MainActivity : AppCompatActivity() {

    fun login() {
        // API call
        // Validate credentials
        // Save token
        // Update UI
    }

    fun getProducts() {
        // API call
        // Parse response
        // Save products
        // Update UI
    }

    fun addToCart() {
        // API call
        // Update database
        // Update UI
    }
}
```

This might work for a small application.

But imagine adding 50 more features.

Your Activity will become responsible for:

- UI rendering.
- Business logic.
- API communication.
- Database operations.
- Authentication.
- Data processing.

This creates a problem called Tight Coupling.

### What is tight coupling?

When multiple responsibilities depend heavily on each other, changing one component can affect many other components.

For example:

- Changing the API implementation affects your Activity.
- Changing the database affects your UI code.
- Testing business logic requires creating Android UI components.

This makes the application difficult to maintain, test and scale.

### What is application architecture?

Application architecture defines how we organize the application into different components, responsibilities and layers.

The main objectives are:

1. Separation of concerns.
2. Maintainability.
3. Testability.
4. Scalability.
5. Reusability.
6. Flexibility.

Interview answer:

Application architecture is the structural organization of an application. It defines how different components communicate, how responsibilities are separated and how dependencies are managed to make the application maintainable, testable and scalable.

# 2. Separation of Concerns (SoC)

This is one of the fundamental principles behind Android architecture.

Suppose we have a login feature.

Instead of writing everything in one class, we separate responsibilities.

## Login feature – Separation of concerns

UI Layer

LoginActivity / LoginScreen

Displays login screen and handles user interaction

Business Logic Layer

LoginViewModel / LoginUseCase

Validates input and coordinates login operations

Data Layer

LoginRepository

Manages data sources and login data

Remote

Retrofit API

Local

Room Database

Each layer has a specific responsibility.

For example, if you change Retrofit to another networking library, ideally your UI should not need to change.

That is the benefit of separation of concerns.

# 3. Android Architecture Patterns

Let's understand how different architecture patterns solve these problems.

## 3.1 MVC – Model View Controller

MVC is one of the earliest software architecture patterns.

It divides an application into three components.

- Model: Manages application data.
- View: Displays UI.
- Controller: Handles user interactions and coordinates operations.

MVC vs MVVM- A complete guide with comparison

[intuz.com](<https://www.intuz.com/blog/guide-on-mvc-vs-mvvm?utm_source=chatgpt.com>)

### Example in Android

kotlin

```kotlin
// Model
data class User(
    val name: String
)

// View
class LoginActivity : AppCompatActivity() {

    fun showUser(name: String) {
        // Update UI
    }
}

// Controller
class LoginController(
    private val activity: LoginActivity
) {

    fun login(name: String) {
        val user = User(name)
        activity.showUser(user.name)
    }
}
```

### Problems with MVC in Android

Android Activities and Fragments traditionally manage both UI and user interactions.

As the application grows, they can become large and contain too much logic.

This is commonly called a God Activity or God Object problem.

Advantages:

- Simple to understand.
- Suitable for small applications.
- Clear conceptual separation.

Disadvantages:

- Controllers can become large.
- UI and business logic can become tightly coupled.
- Testing can be difficult.

## 3.2 MVP – Model View Presenter

MVP improves separation by introducing a Presenter.

Components:

- Model: Data.
- View: Activity or Fragment.
- Presenter: Business logic and UI coordination.

MVP(Model View Presenter) Pattern In Android - Speaker Deck

[speakerdeck.com](<https://speakerdeck.com/kolocoda/mvp-model-view-presenter-pattern-in-android?utm_source=chatgpt.com>)

### Flow

1. User clicks Login.
2. View calls Presenter.
3. Presenter performs validation.
4. Presenter requests data from Model.
5. Presenter sends the result to View.
6. View updates the UI.

### Kotlin example

View

kotlin

```kotlin
interface LoginView {
    fun showLoading()
    fun hideLoading()
    fun showSuccess()
    fun showError(message: String)
}
```

Presenter

kotlin

```kotlin
class LoginPresenter(
    private val view: LoginView
) {

    fun login(username: String, password: String) {

        if (username.isBlank() || password.isBlank()) {
            view.showError("Invalid credentials")
            return
        }

        view.showLoading()

        // Perform login operation
        view.hideLoading()
        view.showSuccess()
    }
}
```

Activity

kotlin

```kotlin
class LoginActivity : AppCompatActivity(), LoginView {

    private lateinit var presenter: LoginPresenter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        presenter = LoginPresenter(this)
    }

    override fun showLoading() {}
    override fun hideLoading() {}
    override fun showSuccess() {}
    override fun showError(message: String) {}
}
```

### Advantages

- Better separation than MVC.
- Presenter can be tested independently.
- View becomes relatively lightweight.

### Disadvantages

- Requires interfaces.
- Presenter may become large.
- Manual lifecycle management is required.
- Maintaining View-Presenter references can cause memory leaks if not handled properly.

## 3.3 MVVM – Model View ViewModel

Important for Android interviewsMVVM stands for:

- Model.
- View.
- ViewModel.

The ViewModel manages UI-related state and exposes it to the View.

Unlike MVP, the ViewModel does not need a direct reference to the Activity or Fragment.

Mastering Android MVVM: Kotlin example to Create High-Quality Android App for Fetching data from web service by using Best Practices | by Abdul Qadir | Medium

[medium.com](<https://medium.com/%40abdulqadirtr/mastering-android-mvvm-kotlin-example-to-create-high-quality-android-app-for-fetching-data-from-1357c4adc52?utm_source=chatgpt.com>)

### Example

Model

kotlin

```kotlin
data class User(
    val id: Int,
    val name: String
)
```

ViewModel

kotlin

```kotlin
class LoginViewModel : ViewModel() {

    private val _user = MutableStateFlow<User?>(null)

    val user: StateFlow<User?> = _user.asStateFlow()

    fun login(username: String) {
        _user.value = User(
            id = 1,
            name = username
        )
    }
}
```

Activity

kotlin

```kotlin
class LoginActivity : AppCompatActivity() {

    private val viewModel: LoginViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.user.collect { user ->
                    user?.let {
                        // Update UI
                    }
                }
            }
        }
    }
}
```

### How does MVVM work?

1. User interacts with View.
2. View sends an event to ViewModel.
3. ViewModel executes business operations or calls a Use Case.
4. Repository provides data.
5. ViewModel updates UI state.
6. View observes state and updates UI.

### Advantages

- Lifecycle-aware architecture.
- Easier unit testing.
- Less coupling between View and business logic.
- Supports StateFlow and LiveData.
- Works well with Jetpack Compose.

### Disadvantages

- ViewModels can become very large.
- Business logic can leak into ViewModels.
- Without proper boundaries, repositories and ViewModels may become tightly coupled.

## 3.4 MVI – Model View Intent

MVI focuses on unidirectional data flow.

It is particularly useful when the UI has complex states and interactions.

MVI in Modern Android: How Unidirectional State Flow Solves Real Scalability Problems | by Rafael Henrique Fernandes | Jan, 2026 | Medium

[medium.com](<https://medium.com/%40faelmg18android/mvi-in-modern-android-how-unidirectional-state-flow-solves-real-scalability-problems-4bf50afe4a3f?utm_source=chatgpt.com>)

The main components are:

- Intent: User action.
- Model/State: Current UI state.
- View: Displays state.
- Reducer: Produces the next state from current state and an event.

### Example

Imagine a login screen.

Possible UI states:

- Initial.
- Loading.
- Success.
- Error.

kotlin

```kotlin
data class LoginUiState(
    val isLoading: Boolean = false,
    val username: String = "",
    val error: String? = null,
    val isSuccess: Boolean = false
)
```

Events:

kotlin

```kotlin
sealed interface LoginIntent {

    data class UsernameChanged(
        val value: String
    ) : LoginIntent

    data object LoginClicked : LoginIntent
}
```

State update:

kotlin

```kotlin
class LoginViewModel : ViewModel() {

    private val _uiState = MutableStateFlow(LoginUiState())

    val uiState = _uiState.asStateFlow()

    fun onIntent(intent: LoginIntent) {

        when (intent) {

            is LoginIntent.UsernameChanged -> {
                _uiState.update {
                    it.copy(username = intent.value)
                }
            }

            LoginIntent.LoginClicked -> {
                _uiState.update {
                    it.copy(
                        isLoading = true,
                        error = null
                    )
                }
            }
        }
    }
}
```

### Why MVI?

Imagine an application like Instagram.

One screen can have:

- Loading feed.
- Refreshing feed.
- Loading next page.
- Like action.
- Unlike action.
- Network error.
- Empty feed.
- Offline state.

Managing these states independently can become complicated.

MVI encourages a single source of truth for UI state.

### Advantages

- Predictable state changes.
- Unidirectional data flow.
- Easier debugging.
- Good for complex UI.
- Works naturally with Kotlin Flow and Compose.

### Disadvantages

- More boilerplate.
- Can be unnecessarily complex for simple screens.
- Large state objects require careful management.

## 3.5 MVC vs MVP vs MVVM vs MVI

| Feature | MVC | MVP | MVVM | MVI |
| --- | --- | --- | --- | --- |
| Main component | Controller | Presenter | ViewModel | State/Reducer |
| UI dependency | Often coupled | View interface | Observes state | Renders state |
| Data flow | Varies | Usually two-way | Observable | Unidirectional |
| Testing | Moderate | Good | Good | Good |
| Complex UI | Limited | Moderate | Good | Excellent |
| Android usage | Legacy/common concepts | Legacy apps | Very common | Common in state-heavy apps |

Senior interview question: Which architecture would you choose for a modern Android application?

A good answer:

"I would generally choose MVVM with Clean Architecture for a maintainable Android application. For complex UI flows, I would adopt MVI-style unidirectional data flow using immutable UI state and events. The choice depends on the complexity of the feature, testing requirements and team conventions."

Remember: MVVM and MVI are not mutually exclusive. You can implement MVI-style state management inside an MVVM architecture.

# 4. Clean Architecture

Very importantClean Architecture is an architectural approach that separates an application into layers and controls dependency direction.

The most important principle:

Dependencies should point inward, towards business rules.

Understanding the Clean Architecture | Navendu Duari

[navenduduari.github.io](<https://navenduduari.github.io/posts/understanding-the-clean-architecture/?utm_source=chatgpt.com>)

For Android, we commonly organize the application into three logical layers:

1. Presentation Layer.
2. Domain Layer.
3. Data Layer.

## 4.1 Presentation Layer

Responsible for UI and UI state.

Contains:

- Activity.
- Fragment.
- Jetpack Compose screens.
- ViewModel.
- UI State.
- UI Events.

Example:

kotlin

```kotlin
class ProductViewModel(
    private val getProducts: GetProductsUseCase
) : ViewModel() {

    private val _products =
        MutableStateFlow<List<Product>>(emptyList())

    val products = _products.asStateFlow()

    fun loadProducts() {
        viewModelScope.launch {
            _products.value = getProducts()
        }
    }
}
```

## 4.2 Domain Layer

This layer contains business rules.

Contains:

- Business models.
- Use Cases.
- Repository interfaces.

Example:

kotlin

```kotlin
data class Product(
    val id: String,
    val name: String,
    val price: Double
)
```

Repository interface:

kotlin

```kotlin
interface ProductRepository {
    suspend fun getProducts(): List<Product>
}
```

Use Case:

kotlin

```kotlin
class GetProductsUseCase(
    private val repository: ProductRepository
) {

    suspend operator fun invoke(): List<Product> {
        return repository.getProducts()
    }
}
```

Notice something important.

The Domain Layer knows about the repository interface, but it does not know whether the data comes from:

- Retrofit.
- Room.
- Firebase.
- Local JSON.
- Another database.

This is dependency inversion.

## 4.3 Data Layer

Responsible for providing actual data.

Contains:

- Repository implementations.
- Remote data sources.
- Local data sources.
- DTOs.
- Database entities.
- Data mapping.

Example:

kotlin

```kotlin
class ProductRepositoryImpl(
    private val api: ProductApi,
    private val dao: ProductDao
) : ProductRepository {

    override suspend fun getProducts(): List<Product> {

        val products = api.getProducts()

        dao.insertAll(products.map { it.toEntity() })

        return products.map { it.toDomain() }
    }
}
```

## 4.4 Complete Clean Architecture flow

## Android Clean Architecture

Presentation Layer

Compose / Activity / Fragment

ViewModel / UI State

Calls

Domain Layer

Use Cases

Repository Interfaces

Business Models

Implemented by

Data Layer

Repository Implementations

Remote and Local Data Sources

Remote

Retrofit / OkHttp

Local

Room / DataStore

Important: This is a logical dependency diagram, not necessarily the order of every runtime call. The repository implementation fulfills the interface defined by the Domain Layer.

# 5. SOLID Principles in Android

SOLID is a set of five software design principles that help us create maintainable and extensible applications.

## S – Single Responsibility Principle

A class should have one primary responsibility.

Bad example:

kotlin

```kotlin
class UserManager {

    fun login() {}
    fun saveUser() {}
    fun sendEmail() {}
    fun generateReport() {}
}
```

This class has too many responsibilities.

Better:

kotlin

```kotlin
class AuthRepository {
    fun login() {}
}

class UserLocalDataSource {
    fun saveUser() {}
}

class EmailService {
    fun sendEmail() {}
}
```

## O – Open/Closed Principle

Software components should be open for extension but closed for modification.

Example:

kotlin

```kotlin
interface PaymentProcessor {
    fun pay(amount: Double)
}

class UpiPayment : PaymentProcessor {
    override fun pay(amount: Double) {}
}

class CardPayment : PaymentProcessor {
    override fun pay(amount: Double) {}
}
```

You can introduce a new payment method without changing existing payment implementations.

## L – Liskov Substitution Principle

A subtype should be usable wherever its parent type is expected without breaking correctness.

For example, if a class implements a repository interface but throws unsupported-operation errors for methods it is expected to support, it may violate this principle.

## I – Interface Segregation Principle

Avoid forcing classes to implement methods they don't need.

Bad:

kotlin

```kotlin
interface UserOperations {
    fun login()
    fun logout()
    fun uploadPhoto()
    fun makePayment()
}
```

Better:

kotlin

```kotlin
interface Authentication {
    fun login()
    fun logout()
}

interface PhotoUploader {
    fun uploadPhoto()
}

interface PaymentService {
    fun makePayment()
}
```

## D – Dependency Inversion Principle

High-level modules should not depend directly on low-level implementation details.

Both should depend on abstractions.

Bad:

kotlin

```kotlin
class ProductViewModel {

    private val repository =
        ProductRepositoryImpl()
}
```

Better:

kotlin

```kotlin
class ProductViewModel(
    private val repository: ProductRepository
)
```

The ViewModel depends on an abstraction, not a concrete implementation.

Interview tip: Dependency Inversion is particularly important when explaining Clean Architecture.

# 6. Repository Pattern

The Repository Pattern provides an abstraction between the business logic and the data sources.

Imagine your application gets products from an API and caches them in Room.

Without a repository:

kotlin

```kotlin
class ProductViewModel(
    private val api: ProductApi,
    private val dao: ProductDao
)
```

The ViewModel now knows about both network and database implementations.

With a repository:

kotlin

```kotlin
class ProductViewModel(
    private val repository: ProductRepository
)
```

The repository handles the data operations.

### Repository architecture

Create Android app with MVVM pattern simply using Android Architecture Component | by hongbeom | hongbeomi dev | Medium

[medium.com](<https://medium.com/hongbeomi-dev/create-android-app-with-mvvm-pattern-simply-using-android-architecture-component-529d983eaabe?utm_source=chatgpt.com>)

### Why use Repository Pattern?

- Separates data access from business logic.
- Supports multiple data sources.
- Makes testing easier.
- Provides a centralized data access abstraction.
- Helps implement offline-first architecture.

### Senior-level question

Is Repository Pattern mandatory in every Android application?

No. It is useful when it provides a meaningful abstraction over data sources or data operations. For a very small application with trivial data access, introducing excessive repository layers can create unnecessary complexity.

# 7. Use Case Pattern

A Use Case represents one specific business operation.

Examples:

- LoginUserUseCase.
- GetProductsUseCase.
- PlaceOrderUseCase.
- AddToCartUseCase.
- SyncUserDataUseCase.

Example:

kotlin

```kotlin
class AddToCartUseCase(
    private val repository: CartRepository
) {

    suspend operator fun invoke(productId: String) {

        require(productId.isNotBlank())

        repository.addToCart(productId)
    }
}
```

ViewModel:

kotlin

```kotlin
class CartViewModel(
    private val addToCart: AddToCartUseCase
) : ViewModel() {

    fun addProduct(productId: String) {

        viewModelScope.launch {
            addToCart(productId)
        }
    }
}
```

### Why Use Cases?

Suppose you need to place an order.

The operation may involve:

1. Validating cart items.
2. Checking inventory.
3. Calculating the final price.
4. Applying discounts.
5. Initiating payment.
6. Creating the order.

This business workflow can be represented by a `PlaceOrderUseCase`.

This keeps business logic out of the ViewModel.

### Repository vs Use Case

| Repository | Use Case |
| --- | --- |
| Manages data access abstraction | Represents a business operation |
| Communicates with data sources through implementation | Coordinates business rules |
| Can provide multiple operations | Usually represents one operation |
| Example: ProductRepository | Example: GetProductsUseCase |

# 8. Dependency Injection (DI)

Dependency Injection means providing an object with the dependencies it needs instead of making it create those dependencies itself.

Without DI:

kotlin

```kotlin
class ProductViewModel {

    private val repository =
        ProductRepositoryImpl(
            ProductApi()
        )
}
```

Problems:

- Tight coupling.
- Difficult testing.
- Difficult replacement of implementations.
- Object creation is mixed with business logic.

With DI:

kotlin

```kotlin
class ProductViewModel(
    private val repository: ProductRepository
)
```

The dependency is provided from outside.

### Types of Dependency Injection

1. Constructor Injection.
2. Field Injection.
3. Method Injection.

Constructor injection is generally preferred when possible because dependencies are explicit and can be made immutable.

## Hilt in Android

Hilt is a dependency injection library built on top of Dagger.

Example:

kotlin

```kotlin
@HiltAndroidApp
class MyApplication : Application()
```

Repository implementation:

kotlin

```kotlin
class ProductRepositoryImpl @Inject constructor(
    private val api: ProductApi
) : ProductRepository
```

Binding:

kotlin

```kotlin
@Module
@InstallIn(SingletonComponent::class)
abstract class ProductModule {

    @Binds
    abstract fun bindProductRepository(
        impl: ProductRepositoryImpl
    ): ProductRepository
}
```

ViewModel:

kotlin

```kotlin
@HiltViewModel
class ProductViewModel @Inject constructor(
    private val repository: ProductRepository
) : ViewModel()
```

Activity:

kotlin

```kotlin
@AndroidEntryPoint
class MainActivity : ComponentActivity()
```

### Important Hilt concepts

| Annotation | Purpose |
| --- | --- |
| `@Inject` | Requests or provides constructor dependencies |
| `@Module` | Defines dependency bindings and providers |
| `@Provides` | Provides dependencies using a function |
| `@Binds` | Maps an implementation to an interface |
| `@Singleton` | Scopes an instance to the component lifetime |
| `@HiltViewModel` | Enables Hilt injection into ViewModels |
| `@AndroidEntryPoint` | Enables injection into Android entry points |
| `@InstallIn` | Defines the component where bindings are installed |

### Why is DI important in senior Android development?

Consider switching from a production API to a fake repository during unit testing.

With DI, you can provide a fake implementation without changing your ViewModel.

kotlin

```kotlin
class FakeProductRepository : ProductRepository {

    override suspend fun getProducts(): List<Product> {
        return listOf(
            Product("1", "Phone", 1000.0)
        )
    }
}
```

This improves testability and flexibility.

# 9. Android Application Modularization

Modularization means splitting an application into smaller, independently organized modules.

Imagine a large shopping application.

Instead of placing all features in one app module, we can organize it like this:

## Example modular Android application

:app

Application entry point and navigation integration

feature:login

feature:home

feature:cart

feature:profile

core:network

core:database

core:designsystem

core:common

### Types of Android modules

1. App Module

Contains:

- Application class.
- Main navigation.
- Application configuration.
- Integration of feature modules.

2. Feature Modules

Examples:

- Login.
- Home.
- Cart.
- Profile.
- Payments.

3. Core Modules

Examples:

- Network.
- Database.
- Common utilities.
- Design system.
- Analytics.

### Benefits of modularization

- Parallel development.
- Better separation of responsibilities.
- Improved build organization.
- Feature isolation.
- Reusability.
- Easier testing.
- Potential for improved build performance.

### When should we modularize?

Do not modularize every small class or package.

Consider modularization when:

- The codebase becomes large.
- Multiple teams work on different features.
- Features have clear boundaries.
- Shared components need reuse.
- Build times and dependency management become problematic.

# 10. Putting Everything Together – Real Android Architecture

Let's design a scalable e-commerce Android application using the concepts we learned.

## E-commerce Android architecture

Presentation Layer

Jetpack Compose / Activities / Fragments

ViewModels + MVI-style UI State

Domain Layer

GetProductsUseCase

AddToCartUseCase

PlaceOrderUseCase

Repository Interfaces

Data Layer

Repository Implementations

Remote Data Source + Local Data Source

Remote

Retrofit + OkHttp

Backend APIs

Local

Room + DataStore

Local Cache

Dependency Injection

Hilt manages object creation and dependencies

Modularization

Feature modules + Core modules + App module

This architecture combines:

- MVVM for UI architecture.
- MVI-style unidirectional state management.
- Clean Architecture for layer separation.
- SOLID for maintainable design.
- Repository Pattern for data access.
- Use Cases for business operations.
- Hilt for dependency injection.
- Modularization for large-scale development.

This is a strong architecture to discuss in a Senior Android Developer interview. It is not the only valid architecture, and the right amount of abstraction depends on application complexity.

# 11. Senior Android Developer Interview Questions

These are the questions you should be able to answer after Day 29.

### Basic questions

1. What is Android application architecture?
2. What is Separation of Concerns?
3. Explain MVC, MVP, MVVM and MVI.
4. What are the differences between MVP and MVVM?
5. What is Clean Architecture?
6. What are the three main layers of Clean Architecture?
7. What is the Repository Pattern?
8. What is a Use Case?
9. What is Dependency Injection?
10. What are SOLID principles?

### Intermediate questions

11. Why should ViewModel not directly access Retrofit?
12. Why should a ViewModel not hold an Activity reference?
13. Can we use MVVM and MVI together?
14. What is Dependency Inversion?
15. What is the difference between Repository and Use Case?
16. Why do we need interfaces in Clean Architecture?
17. What is constructor injection?
18. What is the difference between `@Provides` and `@Binds`?
19. What is Android modularization?
20. When should we introduce separate feature modules?

### Senior-level questions

21. How would you architect a large e-commerce Android application?
22. How would you support multiple data sources?
23. How would you implement offline-first architecture?
24. How would you prevent ViewModels from becoming too large?
25. How would you manage dependencies across multiple modules?
26. How would you migrate an existing MVC application to MVVM?
27. How would you test the Domain Layer independently?
28. How would you manage shared dependencies across feature modules?
29. How would you decide between MVVM and MVI?
30. What are the disadvantages of overengineering Clean Architecture?

# 12. Practical Assignment

Hands-on exerciseDesign a Product Listing feature using Clean Architecture.

Your implementation should contain:

Presentation

- ProductScreen.
- ProductViewModel.
- ProductUiState.

Domain

- Product model.
- ProductRepository interface.
- GetProductsUseCase.

Data

- ProductRepositoryImpl.
- ProductRemoteDataSource.
- ProductLocalDataSource.

Dependency Injection

- Hilt module.
- Repository binding.

### Expected architecture

text

```text
product/
│
├── presentation/
│   ├── ProductScreen.kt
│   ├── ProductViewModel.kt
│   └── ProductUiState.kt
│
├── domain/
│   ├── model/
│   │   └── Product.kt
│   ├── repository/
│   │   └── ProductRepository.kt
│   └── usecase/
│       └── GetProductsUseCase.kt
│
└── data/
    ├── repository/
    │   └── ProductRepositoryImpl.kt
    ├── remote/
    │   └── ProductRemoteDataSource.kt
    └── local/
        └── ProductLocalDataSource.kt
```

Try implementing this feature without directly coupling your ViewModel to Retrofit or Room.

# 13. Day 29 Revision Checklist

Learning progress

# 0/13

[ ]I understand MVC, MVP, MVVM and MVI.

[ ]I can explain the differences between all four patterns.

[ ]I understand Separation of Concerns.

[ ]I can explain Clean Architecture and its layers.

[ ]I understand dependency direction and Dependency Inversion.

[ ]I can explain all five SOLID principles.

[ ]I understand Repository Pattern.

[ ]I understand Use Case Pattern.

[ ]I can implement constructor-based Dependency Injection.

[ ]I understand Hilt annotations.

[ ]I understand Android modularization.

[ ]I can design a Product Listing feature using Clean Architecture.

[ ]I can explain my architecture choices in a senior interview.

 Reset checklist

# 14. Important Takeaways

For a Senior Android Developer, don't just memorize architecture definitions.

You should understand:

- Why architecture is needed.
- What problems each pattern solves.
- Where each component belongs.
- How dependencies flow.
- How to test components independently.
- How to make architectural trade-offs.

My recommended architecture for modern Android development:

MVVM + Clean Architecture + Repository Pattern + Use Cases + Hilt + MVI-style UI state where appropriate.

Day 29 lesson complete# Next: Day 30 – Offline-First Architecture

We will cover:

- Room Database.
- Local and Remote Data Sources.
- Repository as the Single Source of Truth.
- WorkManager.
- Background Synchronization.
- Conflict Resolution.
- Network Connectivity.
- Offline-First Architecture for real-world Android applications.

Target: Learn how applications like WhatsApp, Instagram and e-commerce apps continue working when the internet is unavailable.
