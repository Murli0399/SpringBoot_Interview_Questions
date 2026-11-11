<details><summary>

## [Easy] What is dependency injection?
</summary>

Dependency Injection (DI) is a design pattern and a software development technique commonly used in object-oriented programming to achieve the separation of concerns, increase modularity, and improve testability and maintainability of code.

In simpler terms, Dependency Injection is about providing the necessary dependencies (objects, services, or other components) to a class rather than the class creating those dependencies itself. This helps to decouple components and make them more independent, which in turn makes the code easier to manage, test, and extend.

**Example:**

```java
@Service
public class InvoiceService {
    private final InvoiceRepository repo;
    private final TaxCalculator tax;

    public InvoiceService(InvoiceRepository repo, TaxCalculator tax) {  // dependencies injected
        this.repo = repo;
        this.tax = tax;
    }
}
```

</details>

<details><summary>

## [Easy] How is dependency injection related to inversion of control?
</summary>

Dependency Injection (DI) and Inversion of Control (IoC) are related concepts in software design and architecture, and they are often used together. They both aim to improve the modularity, flexibility, and testability of software systems, but they address different aspects of achieving these goals.

1. Inversion of Control (IoC):

IoC is a broader design principle that refers to the inversion of the flow of control in a software system. In traditional programming, your application code typically controls the flow of execution, making decisions about when and how to call functions or methods in various modules or components. In contrast, with IoC, the control flow is inverted. Instead of your code controlling the flow, it's handed over to a framework or container that manages the execution and sequencing of various components or modules.

Key characteristics of IoC:

- Control flow is managed externally by a framework or container.
- Promotes modularity and decoupling of components.
- Enhances flexibility and extensibility by allowing components to be easily swapped out or replaced.

2. Dependency Injection (DI):

DI is a specific technique used to achieve IoC. It deals with the way objects or components obtain their dependencies (i.e., other objects or services they rely on). In traditional programming, an object might create its dependencies, which can lead to tight coupling between components. DI solves this problem by inverting the responsibility of obtaining dependencies. Instead of an object creating its own dependencies, they are provided to it from the outside.

Key characteristics of DI:

- Dependencies are "injected" into an object rather than created by the object.
- Promotes loose coupling between components.
- Enhances testability by allowing dependencies to be easily replaced with mock objects for testing.

In summary, IoC is a design principle that shifts control flow management to an external entity, while Dependency Injection is a specific technique used to achieve IoC by managing how an object obtains its dependencies. Together, they lead to more modular, maintainable, and testable software systems. IoC is a broader concept, and DI is one of the techniques employed to implement IoC.

**Example:**

```java
// IoC = the principle (framework controls object creation)
// DI  = the technique Spring uses to implement it (passing dependencies in)
@Service
class A {
    private final B b;
    A(B b) { this.b = b; }   // Spring decides when to create B and injects it -> control inverted
}
```

</details>

<details><summary>

## [Easy] What are the types of dependency injection?
</summary>

Spring supports three ways to inject dependencies:

1. **Constructor injection** (recommended) – dependencies are constructor parameters. Objects are complete and immutable after creation, required dependencies are obvious, and classes are easy to test with `new`.
2. **Setter injection** – the container calls setter methods after construction. Good for optional dependencies or values that may be reconfigured.
3. **Field injection** – `@Autowired` directly on a field. Short, but hides dependencies, prevents `final` fields, and needs reflection or Spring to test. Avoid in production code.

```java
@Service
class OrderService {
	private final OrderRepository repo;                 // 1. constructor injection
	private Notifier notifier;                          // 2. setter injection (optional)
	@Autowired private Clock clock;                     // 3. field injection (avoid)

	OrderService(OrderRepository repo) { this.repo = repo; }

	@Autowired(required = false)
	void setNotifier(Notifier notifier) { this.notifier = notifier; }
}
```

Spring also has **method (lookup) injection** – `@Lookup` – for getting a fresh prototype bean from inside a singleton, and you can inject `ObjectProvider<T>` for lazy or optional access. (The "interface injection" style from older DI literature isn't supported by Spring.)

</details>

<details><summary>

## [Easy] What is constructor injection?
</summary>

Constructor injection is a form of dependency injection (DI) in software development. It's a design pattern where a class's dependencies (i.e., other objects or services that it relies on) are provided to it through its constructor when the class is instantiated. This allows for the inversion of control, where the responsibility of managing dependencies is shifted from the class itself to an external component (typically a DI container or framework).

Here's how constructor injection works:

- Dependency Definition: The dependencies that a class requires are defined as parameters in its constructor. Each parameter represents a dependency.

- Dependency Injection: When an instance of the class is created, the DI container or framework injects (provides) the required dependencies into the constructor.

- Initialization: The class stores these dependencies as private fields or properties for later use.

**Example:**

```java
@Service
public class PaymentService {
    private final PaymentGateway gateway;    // final: immutable, can't be null after construction

    public PaymentService(PaymentGateway gateway) {   // @Autowired optional with one constructor
        this.gateway = gateway;
    }
}
// With Lombok: @RequiredArgsConstructor generates this constructor for final fields.
```

</details>

<details><summary>

## [Easy] How does setter injection work?
</summary>

Setter injection is a form of dependency injection (DI) in software development. It's a design pattern where a class's dependencies (i.e., other objects or services that it relies on) are provided through setter methods rather than through the constructor. This allows for the inversion of control, where the responsibility of managing dependencies is shifted from the class itself to an external component (typically a DI container or framework).

Here's how setter injection works:

- Dependency Definition: The dependencies that a class requires are defined as private fields or properties in the class.

- Setter Methods: The class provides setter methods for each dependency, allowing external components to set (inject) the dependencies after the class is instantiated.

- Dependency Injection: When an instance of the class is created, the DI container or framework sets (injects) the required dependencies using the provided setter methods.

**Example:**

```java
@Service
public class ReportService {
    private Formatter formatter = new PlainFormatter();   // sensible default

    @Autowired(required = false)                          // optional dependency
    public void setFormatter(Formatter formatter) {
        this.formatter = formatter;
    }
}
```

</details>

<details><summary>

## [Easy] What is the difference between constructor and setter injection?
</summary>

1. Constructor Injection:

- Dependencies are provided through the class constructor when an instance is created.
- Constructor injection enforces that all required dependencies are provided at the time of object creation, making the object immediately ready for use.
- Typically, constructor-injected dependencies are marked as final, making them immutable after object creation.
- It promotes the creation of objects in a valid state with all necessary dependencies.
- Constructor injection is suitable for mandatory dependencies that the class cannot function without.

2. Setter Injection:

- Dependencies are provided through setter methods after the object is created.
- Setter injection allows for optional or dynamic configuration of dependencies, as you can set or change them at runtime.
- Dependencies set via setters can be modified after object creation, potentially leading to mutable objects.
- It provides flexibility for changing dependencies over time without recreating objects.
- Setter injection is suitable for optional or changeable dependencies.

**Example:**

```java
@Service
class A {
    private final Repo repo;          // required -> constructor
    private Cache cache;              // optional -> setter

    A(Repo repo) { this.repo = repo; }

    @Autowired(required = false)
    void setCache(Cache cache) { this.cache = cache; }
}
```

</details>

<details><summary>

## [Medium] Explain setter injection for objects and literal values.
</summary>

With setter injection, the container creates the bean (no-arg constructor) and then calls setters:

- **Objects:** the setter receives another bean, e.g. `setRepository(OrderRepository repo)`; Spring finds the matching bean by type (and name/qualifier if needed).
- **Literal values:** the setter receives a configuration value, injected with `@Value("${app.page-size:20}")` or, in XML, `<property name="pageSize" value="20"/>`. Spring converts the text to the parameter type (int, boolean, Duration, etc.).

```java
@Component
public class ReportGenerator {
	private ReportRepository repository;
	private int pageSize;

	@Autowired
	public void setRepository(ReportRepository repository) { this.repository = repository; }   // object

	@Value("${reports.page-size:20}")
	public void setPageSize(int pageSize) { this.pageSize = pageSize; }                        // literal
}
```

For groups of settings, prefer `@ConfigurationProperties` over many `@Value` setters.

</details>

<details><summary>

## [Medium] Explain the injection of Java Collection types.
</summary>

Injecting Java Collection types involves declaring a field for the collection in a class, providing the collection through a setter or constructor, and using it in the class. This is often done in DI frameworks like Spring, allowing flexibility in managing and passing groups of objects as dependencies.

**Example:**

```java
interface Validator { void validate(Order o); }
@Component @Order(1) class StockValidator implements Validator { ... }
@Component @Order(2) class FraudValidator implements Validator { ... }

@Service
class OrderService {
    private final List<Validator> validators;          // all Validator beans, in @Order order
    private final Map<String, Validator> byName;       // bean name -> bean

    OrderService(List<Validator> validators, Map<String, Validator> byName) {
        this.validators = validators;
        this.byName = byName;
    }

    void place(Order o) { validators.forEach(v -> v.validate(o)); }
}
```

</details>

<details><summary>

## [Medium] Which dependency injection approach is better?
</summary>

For mandatory dependencies, **constructor injection** is the better default, and it's what the Spring team recommends:

- Dependencies are explicit and can be `final` (immutable, thread-safe after construction).
- The object can't exist in a half-initialised state.
- Easy to unit test: `new OrderService(mockRepo)` without Spring.
- Too many constructor parameters is a visible signal that the class does too much.
- Constructor injection reveals circular dependencies at startup instead of hiding them.

Use **setter injection** only for genuinely optional or reconfigurable dependencies. Avoid **field injection** outside of tests. With a single constructor, `@Autowired` isn't even needed (Spring 4.3+), and Lombok's `@RequiredArgsConstructor` can generate it.

**Example:**

```java
// Prefer constructor injection
@Service
class Good {
    private final Repo repo;
    Good(Repo repo) { this.repo = repo; }      // testable with plain `new Good(mockRepo)`
}

// Avoid field injection
@Service
class Bad {
    @Autowired private Repo repo;              // hidden dependency, can't be final, needs Spring/reflection in tests
}
```

</details>

<details><summary>

## [Medium] What is a circular dependency and how should it be resolved?
</summary>

A **circular dependency** happens when bean A needs bean B and bean B needs bean A (directly or through a chain: A → B → C → A). With constructor injection, Spring can't create either one first, so startup fails with `BeanCurrentlyInCreationException`. Since **Spring Boot 2.6, circular references are forbidden by default**, even with field/setter injection.

How to resolve it (best first):
1. **Redesign.** A cycle usually means responsibilities are mixed. Move the shared logic into a new third bean that both depend on, or merge the two if they are really one concept.
2. **Use events** instead of a direct call back (A publishes an event, B listens).
3. **Break the cycle lazily:** inject `@Lazy B b` or `ObjectProvider<B>` in A, so a proxy is injected and B is resolved on first use.
4. **Setter injection** on one side (works only if circular references are re-enabled).
5. Last resort: `spring.main.allow-circular-references=true`, which hides the design problem.

**Example:**

```java
@Service class A { A(B b) {} }
@Service class B { B(A a) {} }
// Startup fails: "The dependencies of some of the beans in the application context form a cycle"
// (Spring Boot 2.6+ forbids cycles by default)

// Best fix: redesign - extract the shared logic into a third bean C used by both
@Service class C { }
@Service class A2 { A2(C c) {} }
@Service class B2 { B2(C c) {} }

// Quick workaround (use sparingly): break the cycle with @Lazy
@Service class A3 { A3(@Lazy B3 b) {} }
```

</details>

<details><summary>

## [Hard] What is method injection?
</summary>

In Spring, **method injection** usually means **lookup method injection**: Spring overrides (via CGLIB) a method in your bean so that every call returns a bean from the container. It solves the problem of a **singleton bean that needs a new prototype bean each time** – normal injection would give the singleton only one prototype instance, once, at creation.

You mark the method with `@Lookup` (or `<lookup-method>` in XML); Spring generates its body.

Alternatives that are often simpler:
- Inject `ObjectProvider<MyPrototype>` and call `getObject()` each time.
- Inject `Provider<MyPrototype>` (`jakarta.inject`).
- Use a scoped proxy: `@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)`.

(Passing a dependency as a parameter to an ordinary method is sometimes also called "method injection" in general DI literature, but in Spring interviews they mean `@Lookup`.)

**Example:**

```java
@Component
@Scope("prototype")
class ReportBuilder { /* new instance each time */ }

@Service
abstract class ReportService {           // singleton needing a fresh prototype each call
    public Report build() {
        ReportBuilder b = newBuilder();  // Spring overrides this method (lookup method injection)
        return b.build();
    }

    @Lookup
    protected abstract ReportBuilder newBuilder();
}
// Alternative: inject ObjectProvider<ReportBuilder> and call getObject()
```

</details>
