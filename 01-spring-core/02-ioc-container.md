<details><summary>

## [Easy] What is Inversion of Control (IoC)?
</summary>

Inversion of Control (IoC) is a design principle in software development where the control of object creation and dependency injection is shifted from the application itself to an external framework or container. In IoC, the flow of control is inverted, meaning that instead of the application is responsible for creating and managing its dependencies, the responsibility is delegated to an IoC container. 

IoC promotes loose coupling between components and allows for better modularity and testability. By delegating the creation and management of dependencies to the IoC container, the application becomes more flexible and modular, as components can be easily replaced or modified without affecting the overall structure.

In the context of Java backend development, IoC is commonly implemented using frameworks such as Spring or Java EE (now Jakarta EE). These frameworks provide features like dependency injection, where dependencies are automatically injected into components rather than being explicitly created and managed by the developer. This approach simplifies the code, improves maintainability, and promotes reusable and decoupled components.

**Example:**

```java
// Without IoC: the class creates its own dependencies
class ReportService { private final ReportRepository repo = new ReportRepository(); }

// With IoC: the container creates and hands them in
@Service
class ReportService2 {
    private final ReportRepository repo;
    ReportService2(ReportRepository repo) { this.repo = repo; }   // Spring calls this
}
```

</details>

<details><summary>

## [Easy] What are the advantages of Inversion of Control?
</summary>

The advantages of Inversion of Control (IoC) include:

**1. Loose coupling:** IoC promotes loose coupling between components by decoupling the creation and management of dependencies from the components themselves. This allows for better modularity and makes it easier to replace or modify individual components without affecting the entire system.

**2. Testability:** With IoC, dependencies can be easily mocked or stubbed during unit testing, making it easier to isolate and test individual components in isolation. This leads to more comprehensive and reliable testing.

**3. Reusability:** IoC enables the creation of reusable components that can be easily integrated into different applications or modules. By removing the responsibility of managing dependencies from the components, they become more portable and can be used in different contexts.

**4. Maintainability:** IoC simplifies the codebase by delegating the responsibility of dependency management to the IoC container. This reduces the amount of boilerplate code required and improves the overall maintainability of the application.

**5. Flexibility:** IoC allows for flexible configuration and runtime behavior. By externalizing the configuration of dependencies to the IoC container, it becomes easier to modify the behavior of the application without requiring code changes. This facilitates easier customization and configuration of the application.

**6. Integration with third-party frameworks:** IoC containers often provide integration with other frameworks and libraries. This simplifies the integration of third-party components into the application, as the IoC container can handle the dependency injection and lifecycle management.

Overall, IoC promotes a more modular, flexible, and maintainable codebase, making it easier to develop and evolve complex applications.

**Example:**

```java
// Easy to test: pass a fake instead of the real dependency
ReportRepository fakeRepo = mock(ReportRepository.class);
ReportService2 service = new ReportService2(fakeRepo);

// Easy to swap implementations with configuration/profiles
@Bean @Profile("local") ReportRepository inMemory() { return new InMemoryReportRepository(); }
```

</details>

<details><summary>

## [Easy] What is the responsibility of an IoC container?
</summary>

The responsibility of an Inversion of Control (IoC) container is to manage the creation, configuration, and lifecycle of objects or components within an application. The IoC container acts as a central hub that handles the dependency injection and controls the flow of object instantiation and wiring.

The main responsibilities of an IoC container are as follows:

**1. Object Creation:** The IoC container is responsible for creating instances of objects or components based on their definitions or configurations. It instantiates objects and manages their lifecycle, including initializing, configuring, and destroying them when necessary.

**2. Dependency Injection:** The IoC container handles the injection of dependencies into objects or components. It automatically resolves dependencies and injects them into the appropriate places, based on the configuration or annotations provided. This eliminates the need for manual dependency management within the application code.

**3. Configuration Management:** The IoC container manages the configuration of objects or components. It provides mechanisms for defining the dependencies, properties, and behavior of objects through configuration files, annotations, or programmatically. This allows for flexible and customizable application behavior without modifying the code.

**4. Lifecycle Management:** The IoC container manages the lifecycle of objects or components. It handles tasks such as initializing objects, managing their state, and performing cleanup or destruction when they are no longer needed. This ensures proper resource management and reduces memory leaks or other lifecycle-related issues.

**5. AOP (Aspect-Oriented Programming) Integration:** Some advanced IoC containers also support integration with AOP frameworks. They allow for cross-cutting concerns, such as logging, security, or transaction management, to be easily applied to components through declarative or programmatic means.

**6. Integration with Application Frameworks:** IoC containers often provide integration with application frameworks and libraries. They may offer hooks or extensions to seamlessly integrate with other components or provide additional functionality, such as database connection pooling, caching, or messaging.

By assuming these responsibilities, the IoC container simplifies the development process, promotes loose coupling, and provides a centralized mechanism for managing the dependencies and lifecycle of objects within an application.

**Example:**

```java
ApplicationContext ctx = SpringApplication.run(App.class, args);
// The container has: scanned @Component classes, created beans, injected dependencies,
// run @PostConstruct, applied proxies (@Transactional), and will run @PreDestroy on shutdown.
OrderService svc = ctx.getBean(OrderService.class);
```

</details>

<details><summary>

## [Medium] Describe the two types of IoC container.
</summary>

### 1. `BeanFactory`
- The basic container interface (`org.springframework.beans.factory`).
- Creates beans **lazily**, when `getBean()` is first called.
- Supports dependency injection and basic lifecycle callbacks.
- Does **not** automatically register `BeanPostProcessor`s / `BeanFactoryPostProcessor`s, publish events, resolve messages (i18n) or integrate AOP auto-proxying unless you wire these up manually.

### 2. `ApplicationContext`
- Extends `BeanFactory` and is what every real application uses.
- **Eagerly creates singleton beans at startup**, so configuration errors appear immediately.
- **Automatically detects and registers** `BeanPostProcessor`s and `BeanFactoryPostProcessor`s, which is what makes `@Autowired`, `@PostConstruct`, `@Transactional` proxies and `@Value` placeholders work.
- **Event publishing** (`ApplicationEventPublisher`), **i18n** (`MessageSource`), **resource loading** (`ResourceLoader`), and environment/profile support.
- Implementations: `AnnotationConfigApplicationContext`, `ClassPathXmlApplicationContext`, and web variants such as `AnnotationConfigServletWebServerApplicationContext` used by Spring Boot.

**Example:**

```java
// BeanFactory: basic, lazy (beans created on first getBean)
BeanFactory factory = new DefaultListableBeanFactory();

// ApplicationContext: BeanFactory + events, i18n, AOP integration, eager singleton creation
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
```

</details>

<details><summary>

## [Medium] What is the difference between BeanFactory and ApplicationContext?
</summary>

| | `BeanFactory` | `ApplicationContext` |
|---|---|---|
| Singleton creation | Lazy (on first `getBean`) | **Eager** at startup (unless `@Lazy`) |
| `BeanPostProcessor` / `BeanFactoryPostProcessor` registration | Manual | **Automatic** |
| Annotation processing (`@Autowired`, `@PostConstruct`, `@Value`) | Only if processors are registered manually | Built in |
| AOP auto-proxying (`@Transactional`, `@Async`) | Manual | Automatic |
| Event publishing | **Not supported** | `ApplicationEventPublisher` |
| Internationalization | No | `MessageSource` |
| Resource loading / Environment & profiles | Basic | Full `ResourceLoader`, `Environment` |
| Typical use | Very constrained environments, framework internals | Every normal Spring and Spring Boot application |

Startup is slightly faster with lazy creation, but `ApplicationContext` catches configuration errors at startup instead of at the first request, which is why it's always preferred.

**Example:**

```java
ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
ctx.getBean(OrderService.class);
ctx.publishEvent(new OrderPlaced(1L));                 // only ApplicationContext publishes events
String msg = ctx.getMessage("greeting", null, Locale.ENGLISH); // i18n support
Resource r = ctx.getResource("classpath:data.json");   // resource loading
```

</details>

<details><summary>

## [Medium] How is ApplicationContext configured in Spring?
</summary>

Spring reads **bean definitions** (configuration metadata) from one or more sources:

1. **Java configuration** (most common today): `@Configuration` classes with `@Bean` methods, loaded by `AnnotationConfigApplicationContext`.
2. **Annotations + component scanning:** `@Component`, `@Service`, `@Repository`, `@Controller` classes discovered by `@ComponentScan`.
3. **XML configuration** (legacy): `<bean>` definitions loaded by `ClassPathXmlApplicationContext` or `FileSystemXmlApplicationContext`; can be combined with Java config via `@ImportResource`.
4. **Programmatic registration:** `GenericApplicationContext.registerBean(...)` or functional bean registration.
5. **Groovy bean definitions** (rare).

**Property values** come from `application.properties`/`yml`, environment variables and other `PropertySource`s, resolved into `${...}` placeholders by `PropertySourcesPlaceholderConfigurer` (the older `PropertyPlaceholderConfigurer` is deprecated). This supplies values, not bean definitions.

In **Spring Boot**, `SpringApplication.run(App.class, args)` creates the right context type automatically, scans from the main class's package, and applies auto-configuration.

**Example:**

```java
@Configuration
@ComponentScan("com.example")
public class AppConfig {
    @Bean
    public Clock clock() { return Clock.systemUTC(); }
}

ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
// Spring Boot does this for you: SpringApplication.run(App.class, args)
```

</details>

<details><summary>

## [Medium] What is WebApplicationContext?
</summary>

`WebApplicationContext` is an `ApplicationContext` interface for web applications. It adds access to the `ServletContext` and supports web scopes (`request`, `session`, `application`).

In classic (non-Boot) Spring MVC there were often two contexts:
- A **root WebApplicationContext**, created by `ContextLoaderListener` (configured in `web.xml`), holding shared middle-tier beans: services, repositories, data sources.
- A **child WebApplicationContext per `DispatcherServlet`**, holding web beans: controllers, view resolvers, handler mappings. The child can see the root's beans, not the other way round.

In **Spring Boot** there is normally a **single** context (e.g. `AnnotationConfigServletWebServerApplicationContext`) that also starts the embedded server. You can get the web context in a servlet with `WebApplicationContextUtils.getWebApplicationContext(servletContext)`.

</details>

<details><summary>

## [Medium] What happens if the context is not closed?
</summary>

Closing an `ApplicationContext` destroys its singleton beans: Spring calls `@PreDestroy` / `DisposableBean.destroy()` / custom destroy methods, and closes resources such as connection pools, thread pools, Kafka consumers and schedulers.

If the context is **not** closed (in a standalone app you create yourself):
- Destroy callbacks don't run, so connections, files and threads may not be released cleanly.
- Non-daemon threads (executor pools, schedulers, consumers) can keep the JVM running after `main` ends.
- In-flight work may be cut off instead of finishing gracefully.

How to make sure it's closed:
- Use try-with-resources: `try (var ctx = new AnnotationConfigApplicationContext(AppConfig.class)) { ... }`.
- Call `ctx.registerShutdownHook()` so it closes on JVM shutdown (Spring Boot does this automatically).
- In servlet-container deployments, `ContextLoaderListener` closes the context when the web application stops.

**Example:**

```java
try (ConfigurableApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class)) {
    ctx.getBean(Job.class).run();
}   // close() runs @PreDestroy, closes connection pools, stops Kafka consumers

// Or register a JVM shutdown hook (Spring Boot does this automatically):
ctx.registerShutdownHook();
```

</details>

