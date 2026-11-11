<details><summary>

## [Easy] What is autowiring in Spring?
</summary>

In Spring, autowiring is a feature that allows automatic dependency injection. It enables the framework to automatically wire (or connect) collaborating beans based on their types. By using autowiring, developers can avoid explicit configuration of dependencies and let Spring handle the wiring process behind the scenes.

**Example:**

```java
@Service
class OrderService {
    private final OrderRepository repo;
    OrderService(OrderRepository repo) { this.repo = repo; }  // Spring finds the matching bean by type
}
```

</details>

<details><summary>

## [Easy] What is autowiring by constructor?
</summary>

Autowiring by constructor is a mode of autowiring in Spring where dependencies are resolved and injected through the constructor of a class. It allows Spring to automatically identify the appropriate constructor for dependency injection based on the types of the constructor arguments.

In this mode, you annotate the constructor of a class with **@Autowired** or specify **autowire="constructor"** in XML configuration. When the bean is created, Spring analyzes the constructor arguments and tries to find matching beans in the container based on their types. If a suitable bean is found, it is automatically injected into the constructor parameter.

Autowiring by constructor offers a few advantages:

- It promotes the principle of constructor injection, which is considered a best practice for dependency injection.
- It ensures that the required dependencies are explicitly declared as constructor arguments, making them more visible and reducing ambiguity.
- It helps in creating immutable and thread-safe objects, as dependencies can be set once during construction and not modified afterwards.

However, it's worth noting that autowiring by constructor requires the dependencies to be available as beans in the container. If multiple beans of the same type are present, Spring will raise an exception unless additional qualifiers or annotations are used to disambiguate the injection.

**Example:**

```java
@Service
class InvoiceService {
    private final TaxService tax;
    private final InvoiceRepository repo;

    // Single constructor -> Spring autowires all parameters by type (no @Autowired needed)
    InvoiceService(TaxService tax, InvoiceRepository repo) {
        this.tax = tax;
        this.repo = repo;
    }
}
```

</details>

<details><summary>

## [Easy] What does the @Autowired annotation do?
</summary>

The **@Autowired** annotation in Spring is used to mark a dependency for automatic dependency injection. It allows Spring to automatically wire (or inject) the appropriate bean into the annotated field, method, or constructor.

Here's what the @Autowired annotation does:

### 1. Dependency Injection:
When **@Autowired** is used, Spring looks for a matching bean of the required type in the container and injects it into the annotated element. The dependency is resolved at runtime, and the appropriate bean instance is provided.

### 2. Type-Based Injection:
By default, **@Autowired** performs type-based injection, meaning Spring looks for a bean of the same type as the dependency. If multiple beans of the same type exist, an exception is raised, unless additional qualifiers or annotations are used to disambiguate the injection.

### 3. Flexible Placement:
The **@Autowired** annotation can be placed on different elements, including fields, setter methods, constructors, and even method parameters. It allows flexibility in choosing where the dependency should be injected.

### 4. Optional Dependency:
By default, **@Autowired** assumes that the dependency is required and expects a matching bean to be available. However, you can make the dependency optional by setting the required attribute to false, allowing the injection to be skipped if a suitable bean is not found.

### 5. Integration with Qualifiers:
**@Autowired** can be used in conjunction with the **@Qualifier** annotation to specify the desired bean when multiple beans of the same type exist in the container. The **@Qualifier** annotation helps to disambiguate the injection by providing the bean's qualifier value.

Overall, the **@Autowired** annotation simplifies the process of dependency injection in Spring by allowing automatic wiring of dependencies. It reduces the need for explicit configuration and enables a more concise and readable codebase.

**Example:**

```java
@Service
class ReportService {
    @Autowired private Clock clock;                       // field (avoid)

    private final Repo repo;
    @Autowired ReportService(Repo repo) { this.repo = repo; }  // constructor (preferred)

    @Autowired(required = false)
    void setMetrics(MeterRegistry registry) { }           // optional setter

    @Autowired List<Exporter> exporters;                  // all beans of a type
}
```

</details>

<details><summary>

## [Easy] What happens if we specify an interface instead of a class in getBean() method?
</summary>

In the context of Java programming, the **getBean()** method is typically associated with dependency injection frameworks like Spring. It is commonly used to retrieve an instance of a class from the framework's container based on the specified class or interface.

When you specify an interface instead of a class in the **getBean()** method, the framework will attempt to find a bean that implements that interface and return an instance of that bean. This allows for greater flexibility and loose coupling in your code.

Here's what happens when you specify an interface in the **getBean()** method:

- **Bean Lookup:** The framework scans its container for a bean that implements the specified interface.

- **Bean Resolution:** If a single bean implementing the interface is found, the framework returns an instance of that bean.

- **Ambiguity Handling:** If multiple beans implement the specified interface, the framework may throw an exception or provide additional mechanisms to resolve the ambiguity. This could involve using qualifiers, annotations, or other configuration options to specify the desired bean.

- **No Bean Found:** If no bean implementing the specified interface is found, the framework may throw an exception or return a null reference, depending on its configuration.

By using interfaces, you promote the use of abstraction and allow for the interchangeability of different implementations. This facilitates easier testing, modular design, and promotes the principles of object-oriented programming, such as encapsulation and polymorphism.

**Example:**

```java
interface PaymentGateway { }
@Component class RazorpayGateway implements PaymentGateway { }

PaymentGateway g = ctx.getBean(PaymentGateway.class);   // works: returns RazorpayGateway
// With two implementations: NoUniqueBeanDefinitionException unless one is @Primary.
// Also: if the bean is proxied by a JDK dynamic proxy, getBean(RazorpayGateway.class) fails
// while getBean(PaymentGateway.class) works.
```

</details>

<details><summary>

## [Medium] What are the different modes of autowiring in Spring?
</summary>

The classic modes come from XML configuration (`autowire="..."` on `<bean>`):

1. **`no`** (default) – no autowiring; dependencies are wired explicitly with `<property>` / `<constructor-arg>`.
2. **`byName`** – a property is wired to the bean whose **name** matches the property name (via setters).
3. **`byType`** – a property is wired to the single bean of matching **type**; more than one candidate is an error.
4. **`constructor`** – like `byType`, but for constructor arguments.

(`autodetect` existed in Spring 2.x and was removed.)

**Annotation-based autowiring** (`@Autowired`, `@Inject`, `@Resource`) is what modern Spring uses. `@Autowired`/`@Inject` resolve **by type**, then narrow with `@Qualifier`/`@Primary` and finally the parameter name; `@Resource` resolves **by name** first. With a single constructor, Spring autowires it without any annotation.

**Example:**

```xml
<!-- XML modes: no (default), byName, byType, constructor -->
<bean id="orderService" class="com.example.OrderService" autowire="constructor"/>
```

```java
// Annotation style (modern): by type, narrowed by name with @Qualifier
@Service
class OrderService {
    OrderService(@Qualifier("mysqlOrderRepository") OrderRepository repo) { }
}
```

</details>

<details><summary>

## [Medium] How does autowiring internally work?
</summary>

Internally, autowiring in Spring works through a process called Dependency Injection (DI). When autowiring is enabled for a bean, Spring container examines the dependencies of that bean and tries to fulfill them automatically.

Here is a simplified overview of how autowiring works internally in Spring:

**1. Scanning for Beans:** Spring scans the application context or configuration files to identify beans that are candidates for autowiring. This is usually done during the application startup or when the container is refreshed.

**2. Dependency Resolution:** For each bean with autowiring enabled, Spring analyzes its dependencies (fields, methods, or constructor parameters) to determine how they should be resolved.

**3. Dependency Matching:** Spring matches the dependencies of the bean with the available beans in the container based on the autowiring mode. It may use the type, name, or annotations to find suitable beans.

**4. Dependency Injection:** Once a suitable bean is found, Spring injects it into the dependent bean. This can be done using reflection, where the appropriate field, method, or constructor is accessed and the dependency is set.

**5. Lifecycle Management:** After all dependencies are resolved and injected, Spring manages the lifecycle of the beans. It initializes the beans if necessary, applies any configured post-processors, and manages their destruction when the application context is closed.

It's important to note that autowiring relies on the configuration of beans and their dependencies. The container must have sufficient information to determine how to wire the beans together. This information can be provided through XML configuration, Java-based configuration, or annotations such as **@Autowired**.

Overall, autowiring simplifies the configuration and wiring process by reducing the need for explicit bean wiring, leading to more concise and maintainable code.

**Example:**

```java
// AutowiredAnnotationBeanPostProcessor inspects each bean for @Autowired/@Inject/@Value.
// For each injection point it asks the container for beans of the required type:
//   0 matches -> NoSuchBeanDefinitionException (unless required=false/Optional)
//   1 match   -> injected
//   >1        -> narrowed by @Primary, @Qualifier, then parameter/field name
@Service
class Report {
    Report(Optional<Cache> cache) { }   // Optional: no failure if no Cache bean exists
}
```

</details>

<details><summary>

## [Medium] What are the limitations of autowiring?
</summary>

1. **Ambiguity:** several beans of the same type cause `NoUniqueBeanDefinitionException` until you add `@Primary`, `@Qualifier`, or inject a `List`/`Map` of them.
2. **Missing beans fail at startup:** `NoSuchBeanDefinitionException` (good – early failure – but you must understand why a bean wasn't created, e.g. a condition or package not scanned).
3. **Simple values can't be autowired by type:** `String`, `int` and other configuration values need `@Value` or `@ConfigurationProperties`.
4. **Less explicit wiring:** it isn't visible in one place which implementation is injected, so very large applications can feel "magical". Constructor injection and clear package structure keep it understandable.
5. **Only Spring-managed objects** can be injected: objects you create with `new` don't get autowired.
6. **Overuse of field injection** hides dependencies and makes classes hard to test.

None of these make autowiring tightly coupled – injecting by interface is still loose coupling.

**Example:**

```java
interface Notifier { }
@Component class Email implements Notifier { }
@Component class Sms implements Notifier { }

@Service
class Alerts {
    Alerts(Notifier n) { }   // fails: NoUniqueBeanDefinitionException (2 candidates)
}
// Fixes: @Primary on one, @Qualifier("sms") at the injection point, or inject List<Notifier>.
// Primitives/Strings can't be autowired by type - use @Value("${prop}").
```

</details>

<details><summary>

## [Medium] Is it possible to exclude a bean from being autowired?
</summary>

Yes – you can mark a bean as **not an autowire candidate**. It still exists and can be fetched by name, but it won't be injected when Spring resolves a dependency by type:

```java
@Configuration
class DataConfig {
	@Bean(autowireCandidate = false)       // Spring 5.1+
	DataSource reportingDataSource() { return new HikariDataSource(); }

	@Bean
	@Primary
	DataSource mainDataSource() { return new HikariDataSource(); }
}
```

```xml
<bean id="legacyClient" class="com.example.LegacyClient" autowire-candidate="false"/>
```

Related techniques:
- `@Primary` on the bean you *do* want, so others are ignored by default.
- `@Qualifier` at the injection point to choose explicitly.
- Exclude a class from component scanning entirely: `@ComponentScan(excludeFilters = @Filter(type = FilterType.ASSIGNABLE_TYPE, classes = LegacyClient.class))`.
- `@Autowired(required = false)` makes a dependency optional, but that doesn't exclude any bean.

</details>

<details><summary>

## [Medium] Why do we need a no-arg constructor?
</summary>

Spring itself **doesn't need** a no-arg constructor: with constructor injection, Spring calls your constructor with the required beans (if there are several constructors, mark the one to use with `@Autowired`). A no-arg constructor is needed when Spring has **no other way to create the object** (e.g. setter/field injection only).

Where a no-arg constructor *is* commonly required:
- **JPA/Hibernate entities:** the JPA spec requires a no-arg constructor (it may be `protected`) so the provider can instantiate entities and create proxies.
- **Jackson deserialization:** needs a no-arg constructor plus setters/fields, unless you use `@JsonCreator`, records, or the parameter-names module (Spring Boot includes it).
- **JavaBeans conventions** and some older frameworks/libraries that create objects reflectively.
- **Java serialization:** the class itself doesn't need one, but its first **non-serializable superclass** must have an accessible no-arg constructor; `Externalizable` classes need a public one.

**Example:**

```java
@Entity
public class Company {
    @Id private Long id;
    private String name;

    protected Company() { }                  // required by JPA/Hibernate to create proxies/instances
    public Company(String name) { this.name = name; }
}
// Jackson also needs a no-arg constructor (or @JsonCreator / records) to deserialize JSON.
```

</details>
