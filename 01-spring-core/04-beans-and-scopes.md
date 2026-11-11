# Beans and scopes

<details><summary>
  
### [Easy] What are beans in Spring?
</summary>

In the Spring framework, beans refer to the objects that are managed by the Spring IoC (Inversion of Control) container. A bean is an instance of a class that is initialized, assembled, and managed by the Spring container. Beans are the fundamental building blocks of a Spring application.

The Spring framework provides several mechanisms to configure beans, including XML-based configuration, Java-based configuration, and annotation-based configuration. By defining beans in the configuration files or through annotations, you can instruct the Spring container to create and manage instances of these beans.

Beans in Spring provide various benefits, such as dependency injection, aspect-oriented programming, and lifecycle management. With dependency injection, beans can be wired together, enabling loose coupling and easier testing. Beans can also be enhanced using aspects to add additional functionalities, such as logging or security, to the core business logic. Spring takes care of managing the lifecycle of beans, ensuring that they are created, initialized, and destroyed appropriately.

Overall, beans in Spring serve as the key components that make up the application, and the Spring framework handles their creation, configuration, and lifecycle management to promote modularity, maintainability, and scalability.

**Example:**

```java
@Component                       // becomes a bean via component scanning
class EmailSender { }

@Configuration
class AppConfig {
    @Bean                        // becomes a bean via a factory method
    ObjectMapper objectMapper() { return new ObjectMapper().findAndRegisterModules(); }
}
```

</details>

<details><summary>
  
### [Easy] What are some features of custom init and destroy methods?
</summary>

Custom init and destroy methods in Spring beans offer several features and advantages:

**1. Initialization logic:** Custom init methods allow you to perform any necessary setup or initialization logic for your bean. You can use these methods to initialize resources, establish connections, configure settings, or perform any other actions required before the bean is ready for use.

**2. Cleanup and resource releasing:** Custom destroy methods provide a way to release resources or perform cleanup operations when the bean is being destroyed. This ensures that resources are properly released, connections are closed, and any other necessary cleanup tasks are performed before the bean is removed from the container.

**3. Flexibility:** By defining custom init and destroy methods, you have the flexibility to define your own initialization and cleanup logic specific to your bean's requirements. This allows you to tailor the initialization and destruction phases to meet the needs of your application.

**4. Integration with existing frameworks and libraries:** Custom init and destroy methods can be used to integrate with existing frameworks or libraries that require explicit initialization or cleanup steps. You can invoke third-party initialization or shutdown methods within your custom methods to ensure proper integration with external components.

**5. Easy configuration:** Custom init and destroy methods can be easily configured using annotations or implementing specific interfaces. This allows you to define the initialization and destruction behavior within the bean class itself, keeping the configuration concise and localized.

**6. Integration with Spring container lifecycle:** Spring container automatically detects and invokes the custom init and destroy methods based on their annotations or interface implementations. This seamless integration with the container's lifecycle ensures that the methods are called at the appropriate times during bean instantiation and destruction.

Overall, custom init and destroy methods offer a powerful mechanism to control the lifecycle of Spring beans, allowing you to perform specific initialization and cleanup tasks as required by your application. These methods enhance the flexibility, maintainability, and resource management capabilities of your Spring-based applications.

**Example:**

```java
public class ConnectionManager {               // third-party class you can't annotate
    public void start() { /* open connections */ }
    public void stop() { /* close connections */ }
}

@Configuration
class Config {
    @Bean(initMethod = "start", destroyMethod = "stop")
    ConnectionManager connectionManager() { return new ConnectionManager(); }
}
```

</details>

<details><summary>
  
### [Easy] How can you provide a bean id when using annotations?
</summary>

By default, a scanned component's name is the **simple class name with the first letter lowercased** (`OrderService` → `orderService`). To choose the name yourself:

```java
@Component("paymentClient")            // also works with @Service("..."), @Repository("..."), @Controller("...")
public class RazorpayClient { }

@Configuration
class AppConfig {
	@Bean(name = {"primaryDataSource", "mainDs"})   // a @Bean name plus an alias; default name = method name
	DataSource dataSource() { return new HikariDataSource(); }
}
```

You can also use the JSR-330 annotation `@Named("paymentClient")` from **`jakarta.inject`** (Spring 7 removed support for the old `javax.inject` package). Refer to a named bean with `@Qualifier("paymentClient")`.

</details>

<details><summary>
  
### [Easy] Are Spring beans same as JavaBeans?
</summary>

No, Spring beans are not the same as JavaBeans, although there is some overlap in their concepts and usage.

JavaBeans is a specification and convention for creating reusable components in Java. JavaBeans are Java classes that follow specific naming and design patterns, such as having private fields, public getters and setters (accessor methods), and a no-argument constructor. JavaBeans are typically used for encapsulating data and providing access to that data through getter and setter methods.

On the other hand, Spring beans refer to the objects managed by the Spring IoC (Inversion of Control) container. While Spring beans can be Java classes that follow JavaBeans conventions, they can also be other types of objects, such as instances of non-public classes or even instances created through factory methods. Spring beans are managed by the Spring framework, which provides various mechanisms for their creation, configuration, and lifecycle management.

Spring beans have additional features and capabilities beyond the basic JavaBeans specification. They can benefit from dependency injection, aspect-oriented programming, declarative transaction management, and other advanced Spring features. Spring beans are typically configured using XML-based configuration, Java-based configuration, or annotations, allowing for flexible and modular application development.

In summary, JavaBeans and Spring beans share some similarities in terms of using Java classes to encapsulate data and functionality. However, Spring beans have a broader scope and additional capabilities provided by the Spring framework for managing dependencies, configuration, and advanced features, making them more powerful and flexible in the context of Spring applications.

**Example:**

```java
// JavaBean: a convention - public no-arg constructor, private fields, getters/setters, Serializable
public class UserBean implements Serializable {
    private String name;
    public UserBean() {}
    public String getName() { return name; }
    public void setName(String n) { name = n; }
}

// Spring bean: ANY object managed by the container - no conventions needed
@Service
class UserService { UserService(UserRepo repo) {} }   // no no-arg constructor, no getters
```

</details>

<details><summary>
  
### [Easy] What does the @Bean annotation do?
</summary>

In Spring, the @Bean annotation is used to indicate that a method in a configuration class should be used to create and configure a bean. When you annotate a method with @Bean, it serves as a factory method for creating a bean instance. The @Bean annotation works in conjunction with @Configuration or @Component annotations.

Here's an example of using @Bean annotation:

```
@Configuration
public class AppConfig {
    @Bean
    public MyBean myBean() {
        return new MyBean();
    }
}
```

In this example, the @Configuration annotation marks the class AppConfig as a configuration class. The @Bean annotation is used on the myBean() method to indicate that it should be treated as a factory method for creating a bean of type MyBean. The method implementation returns a new instance of MyBean which will be managed by the Spring container.

The @Bean annotation can also be used with additional attributes to customize the bean creation process. For example, you can specify the bean name, dependencies, initialization, and destruction methods, and more.

```
@Configuration
public class AppConfig {
    @Bean(name = "myBean", initMethod = "init", destroyMethod = "cleanup")
    public MyBean myBean() {
        return new MyBean();
    }
}
```

In this modified example, the name attribute is used to specify the bean name as "myBean". The initMethod attribute specifies the name of the custom initialization method to be called during bean initialization, and the destroyMethod attribute specifies the name of the custom cleanup method to be called during bean destruction.

By using @Bean, you can define beans and their associated configuration in a flexible and programmatic manner. The Spring container will recognize the @Bean annotated methods and create bean instances based on the method's return type and configuration.
</details>

<details><summary>
  
### [Easy] Both @Bean and @Component annotations create beans. What is the difference between the two?
</summary>

| | `@Component` (and `@Service`, `@Repository`, `@Controller`) | `@Bean` |
|---|---|---|
| Placed on | A **class** | A **method** in a `@Configuration` (or `@Component`) class |
| How it's registered | Discovered by **component scanning** | Explicitly declared; the method's return value becomes the bean |
| Control over creation | Spring calls the constructor | You write the creation logic (any code, conditions, builders) |
| Third-party classes | Not possible (you can't annotate library code) | **Yes** – the main use case (e.g. `ObjectMapper`, `RestClient`, `DataSource`) |
| Default bean name | Class name, first letter lowercase | Method name |

```java
@Configuration
class HttpConfig {
	@Bean
	RestClient companyApi(RestClient.Builder builder) {   // library class configured in code
		return builder.baseUrl("https://api.example.com").build();
	}
}
```

Rule of thumb: use `@Component` for your own classes, `@Bean` for library classes or when creation needs logic. In a `@Configuration` class, calls between `@Bean` methods return the same singleton (the class is CGLIB-proxied); in a plain `@Component` ("lite mode") they would create new objects.

</details>

<details><summary>
  
### [Easy] How can dependencies be injected using the @Bean annotation?
</summary>

Dependencies can be injected using the @Bean annotation by specifying the dependencies as method parameters in the @Bean annotated method. The Spring container will automatically resolve and inject the dependencies when invoking the method to create the bean instance.

Here's an example that demonstrates injecting dependencies using the @Bean annotation:

```
@Configuration
public class AppConfig {
    @Bean
    public MyBean myBean(AnotherBean anotherBean) {
        return new MyBean(anotherBean);
    }
}
```

In this example, the myBean() method accepts a parameter of type AnotherBean, which represents the dependency. When the myBean() method is invoked by the container to create the bean, it will automatically look for a bean of type AnotherBean within the application context and pass it as an argument to the method. The method implementation can then use the provided AnotherBean instance to initialize the MyBean instance or perform any other required operations.

It's important to note that for the dependency injection to work correctly, the required dependency (AnotherBean in this case) must be defined as a bean within the same or a parent application context. This can be done either by annotating the dependent class with @Component or any of its specialized annotations (@Service, @Repository, @Controller) or by defining the dependency using its own @Bean method within the configuration class.

```
@Configuration
public class AppConfig {
    @Bean
    public AnotherBean anotherBean() {
        return new AnotherBean();
    }

    @Bean
    public MyBean myBean(AnotherBean anotherBean) {
        return new MyBean(anotherBean);
    }
}
```

In this updated example, both AnotherBean and MyBean are defined as beans within the AppConfig configuration class. The myBean() method now receives the AnotherBean instance as a parameter, and Spring resolves the dependency by invoking the anotherBean() method to create the required instance.

By specifying the dependencies as method parameters in the @Bean annotated method, you can achieve dependency injection within the @Bean configuration method itself, allowing for more flexibility and control over the bean creation process.
</details>

<details><summary>
  
### [Easy] What are the different scopes of a bean?
</summary>

| Scope | One instance per… | Available in |
|---|---|---|
| **singleton** (default) | Spring container (`ApplicationContext`) | All apps |
| **prototype** | Every injection / `getBean()` call | All apps |
| **request** | HTTP request | Web apps |
| **session** | HTTP session | Web apps |
| **application** | `ServletContext` | Web apps |
| **websocket** | WebSocket session | WebSocket apps |

- **Singleton** beans are created **eagerly at startup** by the `ApplicationContext` (unless marked `@Lazy`) and destroyed when the context closes. They should be stateless or thread-safe.
- **Prototype** beans are created fresh each time; Spring doesn't manage their destruction.
- Web scopes (`@RequestScope`, `@SessionScope`) injected into singletons need a **scoped proxy** (the shortcut annotations add one automatically), so each call goes to the current request's instance.
- You can register **custom scopes** by implementing `org.springframework.beans.factory.config.Scope`.

**Example:**

```java
@Component @Scope("singleton") class A {}          // one per container (default)
@Component @Scope("prototype") class B {}          // new instance every injection/getBean
@Component @RequestScope class RequestInfo {}      // one per HTTP request
@Component @SessionScope class Cart {}             // one per HTTP session
@Component @ApplicationScope class AppStats {}     // one per ServletContext
```

</details>

<details><summary>
  
### [Easy] What is the default bean scope in Spring?
</summary>

The default bean scope in Spring is singleton. When you create a bean without explicitly specifying a scope, it will be treated as a singleton bean by default.

A singleton bean is created once per Spring IoC container and shared across the application context. Whenever the bean is requested, the same instance is returned. Subsequent requests for the bean within the same application context will always receive a reference to the same singleton instance.

To illustrate the default singleton scope, consider the following example:

```
@Component
public class MyBean {
    // Bean implementation
}
```

In this example, the MyBean class is annotated with @Component, indicating that it should be managed as a Spring bean. Since no explicit scope is specified, the bean will be treated as a singleton by default.

When the Spring container initializes, it will create a single instance of MyBean and register it as a singleton bean within the application context. Any other beans or components that require MyBean will receive the same instance.

If you require a different scope for your beans, such as prototype or request scope, you can explicitly specify it using the @Scope annotation or other means of configuration.
</details>

<details><summary>
  
### [Easy] What is the default scope in the web context?
</summary>

In a web context, the default scope for a bean in Spring is singleton. This means that if you create a bean without explicitly specifying a scope, it will be treated as a singleton bean by default, even in a web application.

The default singleton scope implies that a single instance of the bean is created and shared across the entire web application. The bean is created when the application starts and remains in memory until the application shuts down.

To summarize, the default scope for a bean in a web context is the same as the default scope in a non-web context, which is singleton. This means that unless you specify a different scope explicitly, Spring will assume that your bean should be a singleton and manage it accordingly within the web application context.

**Example:**

```java
@RestController
class HelloController { }            // still singleton by default, even in web apps

@Component
@RequestScope                        // opt in per request
class RequestContext { String requestId = UUID.randomUUID().toString(); }
```

</details>

<details><summary>
  
### [Easy] How is bean scope defined?
</summary>

Use the `@Scope` annotation (on a `@Component` class or on a `@Bean` method), or the `scope` attribute in XML. `@Bean` itself has no scope attribute.

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)       // "prototype"
public class ReportJob { }

@Configuration
public class AppConfig {
	@Bean
	@Scope("prototype")
	public CsvWriter csvWriter() { return new CsvWriter(); }

	@Bean
	@RequestScope                                      // shortcut for @Scope("request") with a proxy
	public RequestContext requestContext() { return new RequestContext(); }
}
```

```xml
<bean id="reportJob" class="com.example.ReportJob" scope="prototype"/>
```

Built-in scope names: `singleton`, `prototype`, `request`, `session`, `application`, `websocket`. Shortcut annotations: `@RequestScope`, `@SessionScope`, `@ApplicationScope`.

</details>

<details><summary>
  
### [Medium] What is the lifecycle of a Spring Bean?
</summary>

For a singleton bean in an `ApplicationContext`, the order is:

1. **Instantiation** – the constructor (or factory method) is called; constructor dependencies are injected here.
2. **Populate properties** – setter and field injection (`@Autowired`, `@Value`).
3. **Aware callbacks** – `BeanNameAware`, `BeanFactoryAware`, `ApplicationContextAware`, … if implemented.
4. **`BeanPostProcessor.postProcessBeforeInitialization`** – for every registered post-processor. `@PostConstruct` methods are invoked at this stage (by `CommonAnnotationBeanPostProcessor`).
5. **Initialization callbacks** – `InitializingBean.afterPropertiesSet()`, then a custom `init-method` / `@Bean(initMethod = ...)`.
6. **`BeanPostProcessor.postProcessAfterInitialization`** – this is where **AOP proxies** are created (for `@Transactional`, `@Async`, `@Cacheable`), so other beans receive the proxy.
7. **Bean in use.** (Singletons are created eagerly at startup; `SmartInitializingSingleton` and `ApplicationReadyEvent` fire once all are ready.)
8. **Destruction** on context shutdown – `@PreDestroy`, then `DisposableBean.destroy()`, then a custom `destroy-method`.

**Prototype beans** go through steps 1–6, but Spring **does not call their destruction callbacks**; the caller owns them after creation.

**Example:**

```java
@Component
class CacheWarmer implements InitializingBean, DisposableBean {
    CacheWarmer(Repo repo) { System.out.println("1. constructor + dependencies injected"); }

    @PostConstruct
    void init() { System.out.println("2. @PostConstruct"); }

    @Override
    public void afterPropertiesSet() { System.out.println("3. afterPropertiesSet"); }

    @PreDestroy
    void cleanup() { System.out.println("4. @PreDestroy on shutdown"); }

    @Override
    public void destroy() { System.out.println("5. destroy"); }
}
// Between creation and init, BeanPostProcessors run (this is where AOP proxies get created).
```

</details>

<details><summary>
  
### [Medium] What are custom bean lifecycle methods?
</summary>

They are hooks Spring calls after a bean is fully initialised and before it's destroyed. There are three ways to declare each, which run in this order:

**Initialization:** `@PostConstruct` method → `InitializingBean.afterPropertiesSet()` → custom init method (`@Bean(initMethod = "init")`).

**Destruction:** `@PreDestroy` method → `DisposableBean.destroy()` → custom destroy method (`@Bean(destroyMethod = "close")`).

```java
@Component
public class CacheWarmer {
	@PostConstruct
	public void init() { /* load reference data after dependencies are injected */ }

	@PreDestroy
	public void cleanup() { /* flush and release resources on shutdown */ }
}
```

Prefer `@PostConstruct` / `@PreDestroy` (from `jakarta.annotation` in Spring 6+) because they don't tie your class to Spring interfaces. Use `initMethod`/`destroyMethod` for third-party classes you can't annotate. Spring Boot infers a destroy method named `close` or `shutdown` on `@Bean` objects automatically.

</details>

<details><summary>
  
### [Medium] How are beans created?
</summary>

In Spring, beans are created by the Spring IoC (Inversion of Control) container. The container is responsible for managing the lifecycle of beans and creating their instances when needed. The process of creating beans involves the following steps:

**1. Bean Definition:** First, the bean must have a corresponding bean definition that specifies how the bean should be created and configured. Bean definitions can be declared in XML-based configuration files, Java-based configuration classes, or through annotations.

**2. Instantiation:** Once the bean definition is available, the container starts the instantiation process. It creates an instance of the bean by invoking the bean's constructor or a factory method, depending on the configuration.

**3. Dependency Injection:** After the bean instance is created, the container performs dependency injection. It resolves the dependencies of the bean, either by matching them to other beans in the container or by providing literal values. The dependencies can be injected via constructor injection, setter injection, or field injection, based on the configuration.

**4. Initialization:** Once the dependencies are injected, the container proceeds with the initialization phase. It calls any custom initialization methods defined in the bean, such as methods annotated with @PostConstruct or implementing the InitializingBean interface. These methods allow the bean to perform any necessary setup or initialization tasks.

**5. Bean Post-Processing:** After initialization, the container applies any registered BeanPostProcessors to the bean. BeanPostProcessors can modify the bean instance or provide additional processing logic before and after bean initialization.

**6. Bean Ready for Use:** At this stage, the bean is considered ready for use. It can now handle requests and perform its designated tasks.

The Spring container manages the lifecycle of the beans, creating and initializing them when needed and destroying them when the application context is being shut down or when the beans are no longer needed. The container ensures that the beans are created in the correct order, with their dependencies correctly resolved, and their initialization and destruction methods called as required.

By leveraging the Spring container's bean creation mechanism, developers can focus on defining the bean configuration and let the container handle the instantiation, dependency injection, and lifecycle management of the beans.

**Example:**

```java
// 1. Component scanning
@Service class A {}

// 2. @Bean methods
@Configuration class Cfg { @Bean B b() { return new B(); } }

// 3. Programmatic registration
ctx.registerBean(C.class, () -> new C());

// 4. Spring Boot auto-configuration (@ConditionalOnMissingBean beans you didn't declare)
```

</details>

<details><summary>
  
### [Medium] When are singleton and prototype scopes used?
</summary>

### Singleton (default)
- For **stateless** components shared across the app: services, repositories, controllers, clients, mappers, configuration.
- Created once at startup and reused, which is efficient.
- Must be **thread-safe**: avoid mutable instance fields, or protect them.

### Prototype
- For **stateful, short-lived** objects where each user of the bean needs its own copy: a builder, a per-task worker holding intermediate state, a non-thread-safe helper.
- Spring creates a new instance per injection/lookup and **does not call destroy callbacks**; the caller must clean up any resources.
- Injecting a prototype into a singleton gives that singleton **one** instance forever. To get a new one each time, use `ObjectProvider<T>`, `@Lookup`, or a scoped proxy.

Examples:
- A connection pool, a logging service, an `ObjectMapper` → **singleton**.
- A per-user **shopping cart** → **session** scope (not prototype – prototype would give a new cart on every injection, unrelated to the user's session).
- A CSV export job object that accumulates rows → **prototype**.

**Example:**

```java
@Service                                  // singleton: stateless services, repositories, clients
class PriceService { }

@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)  // prototype: stateful, short-lived helpers
class CsvExportJob {
    private final List<String> rows = new ArrayList<>();   // per-job state
}
```

</details>

<details><summary>
  
### [Medium] Is Singleton scope in Spring same as the Singleton design pattern?
</summary>

The Singleton scope in Spring is similar to the Singleton design pattern in terms of creating a single instance of a class. However, there are some differences to consider:

**1. Implementation:** In the Singleton design pattern, the class itself is responsible for ensuring that only one instance is created and providing global access to that instance. This typically involves using static methods or a static instance variable.
In Spring, the Singleton scope is managed by the Spring IoC container. When a bean is defined with the Singleton scope, the container is responsible for creating a single instance of the bean and managing its lifecycle.

**2. Scoping:** In the Singleton design pattern, the singleton instance is typically available throughout the entire application. It is a global object that can be accessed from anywhere.
In Spring, the Singleton scope is limited to the scope of the Spring IoC container in which the bean is defined. The singleton instance is shared within that container and can be accessed by other beans within the same container.

**3. Configuration and Management:** In the Singleton design pattern, you need to implement the logic for ensuring a single instance and handle any synchronization concerns or lazy initialization.
In Spring, the Singleton scope is managed by the container, and you only need to define the bean with the Singleton scope. The container takes care of creating and providing the singleton instance when needed.

**4. Dependency Injection:** In the Singleton design pattern, the singleton instance is often accessed statically, and dependencies are typically obtained using static methods or passed explicitly.
In Spring, dependencies can be injected into singleton beans using various mechanisms, such as constructor injection, setter injection, or field injection. The Spring container handles the injection of dependencies into singleton beans based on their configurations.

In summary, while the Singleton scope in Spring shares the concept of having a single instance with the Singleton design pattern, the implementation and management are handled by the Spring IoC container. Spring's Singleton scope provides a managed and configurable way to define singleton beans within the context of the container, with dependency injection capabilities and scoping limited to the container in which the bean is defined.

**Example:**

```java
@Configuration
class Cfg {
    @Bean DataSource primary() { return ...; }
    @Bean DataSource reporting() { return ...; }   // two "singleton" beans of the same class
}
// Spring singleton = one instance per bean id per container.
// GoF Singleton = one instance per class per ClassLoader, enforced by a private constructor.
```

</details>

<details><summary>
  
### [Medium] Are Singleton beans thread safe?
</summary>

Singleton beans in Spring are not inherently thread-safe by default. The thread safety of a singleton bean depends on how it is implemented and the synchronization mechanisms applied within the bean's methods and fields.

When multiple threads access and modify shared data within a singleton bean concurrently, there can be potential thread safety issues, such as race conditions, data corruption, or inconsistent state.

Here are some considerations to ensure thread safety for singleton beans in Spring:

**1. Immutable State:** If a singleton bean's state is immutable and doesn't change once initialized, it can be considered thread-safe. Immutable objects are inherently safe for concurrent access.

**2. Synchronization:** Proper synchronization mechanisms should be applied within the singleton bean to ensure that shared mutable state is accessed and modified safely by multiple threads. Synchronization can be achieved using techniques like synchronized blocks, locks, or concurrent data structures.

**3. Thread-Safe Dependencies:** Ensure that any dependencies used by the singleton bean are themselves thread-safe. If a singleton bean depends on other non-thread-safe beans, additional synchronization or proper handling of those dependencies may be required.

**4. Avoid Shared Mutable State:** Design the singleton bean in a way that minimizes shared mutable state. Prefer immutability and immutability patterns to reduce the need for synchronization.

It's important to note that Spring manages the lifecycle and instantiation of singleton beans, but it does not provide built-in mechanisms for thread safety. It is the responsibility of the developer to design and implement singleton beans in a thread-safe manner.

By following good programming practices, understanding the threading requirements of your application, and applying appropriate synchronization techniques, you can ensure thread safety for singleton beans in Spring.

**Example:**

```java
@Service
class CounterService {
    private int count;                          // NOT thread-safe: shared by all request threads
    public void hit() { count++; }
}

@Service
class SafeCounterService {
    private final AtomicInteger count = new AtomicInteger();   // thread-safe
    public void hit() { count.incrementAndGet(); }
}
// Best practice: keep singleton beans stateless; keep per-request data in local variables.
```

</details>

<details><summary>
  
### [Medium] Explain prototype bean scope.
</summary>

With **prototype** scope, Spring creates a **new instance every time** the bean is injected or requested with `getBean()`.

Key characteristics:
1. **New instance per request** to the container; nothing is shared.
2. **No destruction management:** Spring runs initialization callbacks (`@PostConstruct`), but never calls `@PreDestroy` for prototypes and keeps no reference to them. The caller must release any resources; unreferenced instances are simply garbage-collected.
3. **Can hold state** safely for a single user/task.
4. **Its dependencies keep their own scopes:** a prototype that depends on a singleton gets the **same shared singleton** instance; only prototype-scoped dependencies are created anew.
5. **Prototype inside a singleton:** the singleton gets one prototype instance at creation time and keeps it. Use `ObjectProvider<T>.getObject()`, `@Lookup`, or a scoped proxy to get a fresh one per call.

Use prototypes for stateful, short-lived objects. Don't use them for expensive-to-create objects; creating a new one each time is the opposite of what you want there.

**Example:**

```java
@Component
@Scope("prototype")
class Ticket { final String id = UUID.randomUUID().toString(); }

@Service
class Booking {
    private final ObjectProvider<Ticket> tickets;
    Booking(ObjectProvider<Ticket> tickets) { this.tickets = tickets; }
    Ticket newTicket() { return tickets.getObject(); }   // fresh Ticket every call
}
// Injecting a prototype directly into a singleton gives ONE instance forever - use ObjectProvider.
```

</details>

<details><summary>
  
### [Medium] Does Spring manage the complete lifecycle of beans?
</summary>

**For singleton beans, yes:** Spring creates them, injects dependencies, runs initialization callbacks, applies post-processors and proxies, and calls destruction callbacks when the context closes.

**For prototype beans, no:** Spring creates and initialises them, then hands them over and forgets them. It never calls their `@PreDestroy` / `destroy()` methods. If a prototype holds resources (connections, files, threads), the code that obtained it must release them.

Request and session scoped beans are destroyed when the request ends or the session expires.

**Example:**

```java
@Component
@Scope("prototype")
class TempFileWriter {
    @PreDestroy
    void cleanup() { System.out.println("never called for prototypes"); }
}
// Singletons: Spring creates AND destroys them.
// Prototypes: Spring creates and initializes them, then hands them over; the caller must clean up.
```

</details>

<details><summary>
  
### [Medium] What is an inner bean?
</summary>

An **inner bean** is a bean defined **inside** another bean's definition and used only as that bean's dependency. It has no meaningful name, can't be injected anywhere else, and is always created with its outer bean (its scope is ignored).

XML (where the term comes from):

```xml
<bean id="orderService" class="com.example.OrderService">
	<property name="pricingStrategy">
		<bean class="com.example.FestivePricing"/>   <!-- inner bean: only orderService uses it -->
	</property>
</bean>
```

Java configuration equivalent – create the object inside the `@Bean` method instead of declaring a separate bean:

```java
@Configuration
public class AppConfig {
	@Bean
	public OrderService orderService() {
		return new OrderService(new FestivePricing());   // not registered as its own bean
	}
}
```

If the dependency should be shared, injectable or proxied (e.g. needs `@Transactional`), declare it as a normal top-level bean instead.

</details>

<details><summary>
  
### [Hard] What information does the bean definition contain?
</summary>

A `BeanDefinition` is Spring's recipe for creating a bean. It holds:

- **Bean class** (or a factory bean + factory method that produces it)
- **Bean name** and **aliases**
- **Scope** (singleton, prototype, request, session, …)
- **Constructor arguments** and **property values** (literal values or references to other beans)
- **Autowire mode** and whether it's an autowire **candidate** / **primary**
- **Lazy-init** flag
- **`dependsOn`** (beans that must be created first)
- **Init and destroy method names**
- **Role**, description, and whether it's **abstract** (a template for child definitions)

Bean definitions come from component scanning, `@Bean` methods, XML or programmatic registration. `BeanFactoryPostProcessor`s can modify them **before** any bean is created (that's how `${...}` placeholders are resolved).

</details>
