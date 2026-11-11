# Spring basics

<details><summary>

## [Easy] Which Java versions do Spring 5, 6 and 7 require?
</summary>

| Spring Framework | Spring Boot | Minimum Java | Notes |
|---|---|---|---|
| 4.x | 1.x | Java 6 | End of life |
| 5.x | 2.x | **Java 8** | Supports up to Java 17 (5.3); end of OSS support |
| 6.x | 3.x | **Java 17** | Moves from `javax.*` to **`jakarta.*`** (Jakarta EE 9+), AOT/native support |
| 7.x | 4.x | **Java 17** (embraces Java 25) | Jakarta EE 11, JSpecify null-safety, released November 2025 |

So older Java can't run newer Spring: a Java 11 project is limited to Spring 5 / Boot 2, and moving to Boot 3 means upgrading to Java 17 and replacing `javax` imports with `jakarta`.

</details>

<details><summary>

## [Easy] What are the advantages of using the Spring framework?
</summary>

- **Dependency injection / IoC:** objects get their dependencies from the container, so code is loosely coupled and easy to unit test.
- **Declarative transactions:** `@Transactional` gives one consistent transaction model over JDBC, JPA and JTA.
- **AOP:** cross-cutting concerns (transactions, security, caching, logging) are applied without cluttering business code.
- **Less boilerplate:** `JdbcTemplate`, Spring Data repositories, `RestClient` and `KafkaTemplate` remove repetitive code.
- **Modular:** use only what you need (core, web, data, security, messaging).
- **Lightweight POJO programming:** no need to extend framework classes or run in a heavy application server.
- **Integration:** works with Hibernate/JPA, Kafka, Redis, MongoDB, Elasticsearch, messaging, cloud platforms.
- **Testability:** the TestContext framework, `MockMvc`, slice tests and Testcontainers support.
- **Ecosystem:** Spring Boot (auto-configuration), Spring Security, Spring Cloud, Spring Batch.

</details>

<details><summary>

## [Easy] List the modules of the Spring framework.
</summary>

Spring Framework (6/7) is organised into these module groups:

- **Core Container:** `spring-core`, `spring-beans`, `spring-context`, `spring-context-support`, `spring-expression` (SpEL).
- **AOP and Instrumentation:** `spring-aop`, `spring-aspects` (AspectJ integration), `spring-instrument`.
- **Data Access / Integration:** `spring-jdbc`, `spring-tx` (transactions), `spring-orm` (JPA/Hibernate), `spring-oxm` (object/XML mapping), `spring-jms`, `spring-r2dbc`.
- **Web (Servlet):** `spring-web`, `spring-webmvc`, `spring-websocket`.
- **Web (Reactive):** `spring-webflux`.
- **Messaging:** `spring-messaging`.
- **Test:** `spring-test`.

The old **Portlet** module was removed in Spring 5. Projects like Spring Data, Spring Security, Spring Boot and Spring Kafka are separate projects built on top of the framework.

</details>

<details><summary>

## [Easy] What is tight coupling?
</summary>

**Tight coupling** means a class depends directly on a concrete implementation of another class, usually by creating it with `new`. You can't swap the dependency, configure it from outside, or replace it with a mock in tests without changing the class's code.

Problems it causes:
- Changes in one class ripple into others.
- Hard to unit test (you can't inject a fake).
- Hard to reuse or extend with new implementations.

Spring reduces tight coupling through **dependency injection** (the container supplies dependencies) and **programming to interfaces**.

**Example:**

```java
class OrderService {
    private final EmailSender sender = new EmailSender();  // hard-wired: can't swap to SMS or mock in tests
    void place(Order o) { sender.send(o.customerEmail(), "Order placed"); }
}
```

</details>

<details><summary>

## [Easy] What is loose coupling?
</summary>

Loose coupling in Spring is a design principle that allows components to be independent of each other. This means that changes to one component do not affect other components. Loose coupling is achieved through the use of interfaces and dependency injection.

An interface is a contract that defines the methods that a component must implement. By using interfaces, components can be decoupled from the implementation details of other components. This makes it easier to change components without affecting other components.

Dependency injection is a technique for injecting dependencies into a component. This means that the component does not need to know how to create its dependencies. Instead, the dependencies are injected into the component by the Spring container. This makes it easier to test components in isolation.

Loose coupling is a key design principle in Spring. It makes applications more modular, easier to test, and more scalable.

Here are some of the benefits of loose coupling in Spring:
- **Increased modularity:** Loose coupling makes it easier to change components without affecting other components. This makes it easier to develop and maintain applications.
- **Improved testability:** Loose coupling makes it easier to test components in isolation. This is because components do not need to know how to create their dependencies.
- **Increased scalability:** Loose coupling makes it easier to scale applications. This is because components can be easily replaced without affecting other components.

Overall, loose coupling is a key design principle in Spring that can provide a number of benefits for applications.

**Example:**

```java
interface Notifier { void notify(String to, String msg); }

@Service
class OrderService {
    private final Notifier notifier;                   // depends on an abstraction
    OrderService(Notifier notifier) { this.notifier = notifier; }  // Spring injects Email/SMS impl
    void place(Order o) { notifier.notify(o.customerEmail(), "Order placed"); }
}
// In a unit test: new OrderService(mock(Notifier.class))
```

</details>

<details><summary>

## [Easy] What is the purpose of the application.properties file?
</summary>

`application.properties` (or `application.yml`) in `src/main/resources` is Spring Boot's main **externalised configuration** file. Boot loads it at startup and binds its values to framework settings and to your own `@Value` / `@ConfigurationProperties` beans. Typical entries: server port, datasource URL, JPA settings, logging levels, Kafka brokers, feature flags.

- **Per-environment values** go in profile-specific files like `application-dev.yml` and `application-prod.yml`, activated with `spring.profiles.active`.
- **Packaged values can be overridden** without rebuilding, by external config files, environment variables or command-line arguments (which take precedence).
- **Secrets** (passwords, API keys) shouldn't be committed in this file; reference environment variables (`${DB_PASSWORD}`) or a secret manager.

**Example:**

```properties
server.port=8081
spring.application.name=order-service
spring.datasource.url=jdbc:mysql://localhost:3306/orders
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}
spring.jpa.hibernate.ddl-auto=validate
logging.level.com.example=DEBUG
```

Secrets come from environment variables (`${DB_PASSWORD}`), never committed to Git.

</details>

<details><summary>

## [Easy] What is a pom.xml file?
</summary>

`pom.xml` is the **Project Object Model** file Maven reads to build a project. It sits in the project root and declares:

- **Coordinates:** `groupId`, `artifactId`, `version`, `packaging` (jar/war).
- **Parent:** e.g. `spring-boot-starter-parent`, which supplies dependency versions and plugin defaults.
- **Dependencies** with scopes: `compile` (default), `provided`, `runtime`, `test`.
- **Build plugins:** compiler, Surefire (tests), `spring-boot-maven-plugin` (executable JAR, container images).
- **Properties** (e.g. `java.version`), **profiles**, repositories, and modules for multi-module builds.

Maven combines your POM with defaults (the "super POM"), so you only write what differs from convention: source code in `src/main/java`, tests in `src/test/java`, output in `target/`.

**Example:**

```xml
<project>
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.3.5</version>
  </parent>
  <groupId>com.example</groupId>
  <artifactId>order-service</artifactId>
  <version>1.0.0</version>
  <properties>
    <java.version>21</java.version>
  </properties>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>   <!-- version managed by parent -->
    </dependency>
  </dependencies>
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>  <!-- builds the executable fat JAR -->
      </plugin>
    </plugins>
  </build>
</project>
```

</details>

<details><summary>

## [Medium] What were the major features of Spring 5?
</summary>

Spring Framework 5 (2017) was a major release; its main additions were:

- **Java 8 baseline** with Java 9 compatibility (lambdas, `Optional`, functional style inside the framework).
- **Spring WebFlux:** a new **reactive**, non-blocking web stack built on Project Reactor (`Mono`, `Flux`), running on Netty or Servlet 3.1+ containers, alongside the classic Spring MVC.
- **Functional bean registration** and functional web endpoints (`RouterFunction`) as an alternative to annotations.
- **First-class Kotlin support** (null-safety, extensions).
- **JUnit 5 support** in the TestContext framework, and parallel test execution.
- **Reactive `WebClient`** for non-blocking HTTP calls.
- **Removals:** Portlet, Velocity, JasperReports, XMLBeans, JDO and Guava caching support.

Spring 5 is no longer supported. Today's versions are **Spring Framework 6** (with Spring Boot 3, Java 17+, Jakarta EE) and **Spring Framework 7** (with Spring Boot 4, released November 2025). See `02-spring-boot/03-spring-boot-3-and-4.md`.

</details>

<details><summary>

## [Medium] What design patterns are used in the Spring framework?
</summary>

| Pattern | Where Spring uses it |
|---|---|
| **Dependency Injection / IoC** | The core container |
| **Factory** | `BeanFactory`, `ApplicationContext`, `FactoryBean`, `@Bean` methods |
| **Singleton** | Default bean scope (one instance per container, not per JVM) |
| **Proxy** | AOP: `@Transactional`, `@Cacheable`, `@Async`, Spring Security method security, Spring Data repositories |
| **Template Method** | `JdbcTemplate`, `RestTemplate`, `TransactionTemplate`, `KafkaTemplate` |
| **Front Controller** | `DispatcherServlet` in Spring MVC |
| **MVC** | Spring MVC: controllers, models, views |
| **Observer** | `ApplicationEvent` + `@EventListener` |
| **Strategy** | Pluggable `PlatformTransactionManager`, `Resource` loaders, injecting different implementations of one interface |
| **Adapter** | `HandlerAdapter` in MVC, `MessageListenerAdapter` |
| **Decorator** | `BeanPostProcessor`s wrapping beans; `TransactionAwareDataSourceProxy` |
| **Builder** | `UriComponentsBuilder`, `RestClient.builder()`, `MockMvcRequestBuilders` |

Note: Spring's **prototype scope** (a new instance per request) is not the GoF Prototype pattern, which creates objects by cloning.

**Example:**

```java
@Bean                                   // Factory: BeanFactory/ApplicationContext create beans
JdbcTemplate jdbc(DataSource ds) {      // Template Method: JdbcTemplate, RestTemplate, KafkaTemplate
    return new JdbcTemplate(ds);
}

@Transactional                          // Proxy: AOP proxy wraps the bean
public void transfer() { }

@EventListener                          // Observer: application events
public void onStart(ApplicationReadyEvent e) { }

// Singleton: default bean scope. Front Controller: DispatcherServlet.
// Strategy: injecting different implementations of one interface. Adapter: HandlerAdapter.
```

</details>

<details><summary>

## [Medium] What are some of the best practices for Spring Framework?
</summary>

- **Use constructor injection** with `final` fields; avoid field injection.
- **Keep layers clear:** controllers handle HTTP, services hold business logic and transactions, repositories handle persistence.
- **Put `@Transactional` on service methods**, keep transactions short, and don't make remote calls inside them.
- **Handle errors globally** with `@RestControllerAdvice` and a consistent error body (`ProblemDetail`).
- **Use DTOs** at the API boundary instead of exposing JPA entities.
- **Externalise configuration** with `@ConfigurationProperties` and profiles; keep secrets out of Git.
- **Validate input** with Bean Validation (`@Valid`).
- **Paginate** large queries and avoid N+1 problems.
- **Use Spring Boot starters** and its managed dependency versions; keep Boot up to date.
- **Make it observable:** Actuator, Micrometer metrics, structured logs, tracing.
- **Test at the right level:** unit tests, slice tests (`@WebMvcTest`, `@DataJpaTest`), Testcontainers for integration.
- **Secure by default** with Spring Security; don't expose sensitive Actuator endpoints.

</details>

<details><summary>

## [Medium] Describe some standard Spring events.
</summary>

The `ApplicationContext` publishes lifecycle events that beans can listen to:

- **`ContextRefreshedEvent`** – the context has been initialised or refreshed (all singletons created).
- **`ContextStartedEvent`** – `start()` was called on the context.
- **`ContextStoppedEvent`** – `stop()` was called.
- **`ContextClosedEvent`** – the context is closing (shutdown); beans are about to be destroyed.
- **`RequestHandledEvent`** / `ServletRequestHandledEvent` – a web request has been handled.

Spring Boot adds startup events: `ApplicationStartingEvent`, `ApplicationEnvironmentPreparedEvent`, `ApplicationPreparedEvent`, `ApplicationStartedEvent`, **`ApplicationReadyEvent`** (app ready to serve traffic) and `ApplicationFailedEvent`.

You can also publish **your own events** with `ApplicationEventPublisher.publishEvent(obj)`. Since Spring 4.2, any object can be an event and listeners are plain methods annotated with `@EventListener`; before 4.2, events had to extend `ApplicationEvent` and listeners had to implement `ApplicationListener`. `@TransactionalEventListener` runs a listener only after the surrounding transaction commits.

**Example:**

```java
@Component
class StartupListener {
    @EventListener(ApplicationReadyEvent.class)      // app fully started, ready for traffic
    public void warmCache() { System.out.println("warming cache"); }

    @EventListener(ContextClosedEvent.class)         // shutting down
    public void onShutdown() { System.out.println("closing"); }
}
// Others: ContextRefreshedEvent, ContextStartedEvent, ContextStoppedEvent, ApplicationStartedEvent
```

</details>

<details><summary>

## [Hard] How does Spring work with Java modules (JPMS)?
</summary>

Spring Framework doesn't require the Java Platform Module System, and most Spring apps run on the classpath.

- **Spring 5** added `Automatic-Module-Name` entries to its JARs, so Spring can be used from a modular application (`requires spring.context;`) with stable module names, and it runs on both the classpath and the module path.
- Spring relies on reflection, so a modular application must **open** its packages to Spring (`opens com.example.app to spring.core, spring.beans, spring.context;`) for dependency injection and proxies to work.
- In practice, Spring Boot applications usually stay on the classpath and are packaged as executable JARs or container images. For faster startup and smaller footprints, Spring 6+ offers **AOT processing and GraalVM native images** rather than JPMS.

</details>
