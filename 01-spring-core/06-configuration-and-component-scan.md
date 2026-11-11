<details><summary>
  
### [Easy] What is a configuration file?
</summary>

In general, a configuration file holds settings outside the code (properties, YAML, XML, JSON) so behaviour can change per environment without recompiling.

In Spring, the term is used in two ways:
- **Spring configuration metadata:** how beans are defined and wired – historically XML files like `applicationContext.xml`, today mostly `@Configuration` Java classes.
- **Application configuration:** values such as ports, URLs and credentials in `application.properties` / `application.yml`, profile-specific files, environment variables, and config servers.

Keeping configuration separate from code lets the same build artifact run in dev, test and production with different settings.

**Example:**

```java
@Configuration
public class HttpClientConfig {
    @Bean
    public RestClient companyApi(RestClient.Builder builder) {
        return builder.baseUrl("https://api.example.com")
                      .defaultHeader("Accept", "application/json")
                      .build();
    }
}
```

</details>

<details><summary>
  
### [Easy] How is configuration metadata provided to the Spring container/ how are beans configured in the Spring container?
</summary>

In Spring, configuration metadata is provided to the Spring container through various mechanisms. The most common approaches for configuring beans in the Spring container are:

**1. XML-based Configuration:** In earlier versions of Spring, XML was the predominant way to configure the container and define beans. In XML-based configuration, you create an XML file (often named applicationContext.xml) where you define the beans, their dependencies, and other configuration details. The XML file is then loaded by the Spring container, which parses it and creates the beans accordingly.

**2. Annotation-based Configuration:** With the introduction of annotations in Java, Spring provides support for annotation-based configuration. You can use annotations, such as @Component, @Configuration, @Autowired, and others, to mark classes as beans, specify their dependencies, and configure various aspects of the container. Annotation-based configuration reduces the need for XML files and provides a more concise and expressive way to configure beans.

**3. Java-based Configuration:** Spring also supports configuration using plain Java classes. You can create a configuration class and annotate it with @Configuration. Within the configuration class, you define bean creation methods annotated with @Bean, which specify how to instantiate and configure the beans. Java-based configuration is often preferred for its type safety, refactorability, and the ability to use the full power of the Java language to define complex configuration logic.

Regardless of the configuration approach, once the configuration metadata is provided to the Spring container, it analyzes the metadata and creates the corresponding beans based on the configuration instructions. The container manages the lifecycle of these beans, handles their dependencies, and allows them to be injected into other beans as needed.

Spring provides flexibility in choosing the appropriate configuration style based on your project's requirements and preferences. XML, annotation-based, Java-based, or Groovy-based configuration can be used individually or in combination, depending on the complexity and needs of your application.

**Example:**

```java
// 1. Annotations on classes (component scanning)
@Service class A { }

// 2. Java configuration
@Configuration class Cfg { @Bean B b() { return new B(); } }

// 3. XML (legacy)
// <bean id="c" class="com.example.C"/>
// loaded with @ImportResource("classpath:beans.xml")
```

</details>

<details><summary>
  
### [Easy] What is the difference between XML and Annotation configuration?
</summary>

The main difference between XML and Annotation configuration in the context of Spring is the way in which the configuration metadata is expressed:

### XML Configuration:
XML-based configuration involves creating an XML file where you define beans, their dependencies, and other configuration details. The XML file is typically named applicationContext.xml or a similar variant. XML configuration is more verbose and relies on the hierarchical structure of XML elements and attributes to represent the configuration metadata.

### Annotation Configuration:
Annotation-based configuration, as the name suggests, utilizes annotations provided by Spring to configure beans and their dependencies. Instead of an XML file, you use annotations like @Component, @Configuration, @Autowired, and others to mark classes as beans, specify dependencies, and configure various aspects. Annotation configuration is typically done in regular Java classes and provides a more concise and expressive way to define the configuration metadata.

In summary, XML configuration relies on a separate XML file for defining the configuration, while annotation configuration uses annotations directly within Java classes. XML configuration is more explicit and allows for a more visual representation of the configuration, while annotation configuration is more concise and leverages the power of annotations for configuration purposes. Both approaches have their own advantages and can be used based on personal preference or project requirements.

**Example:**

```xml
<!-- XML: configuration separate from code -->
<bean id="orderService" class="com.example.OrderService">
    <constructor-arg ref="orderRepository"/>
</bean>
```

```java
// Annotations: configuration next to the code, type-safe, refactor-friendly
@Service
class OrderService { OrderService(OrderRepository repo) { } }
```

</details>

<details><summary>
  
### [Easy] How is annotation based configuration enabled in Spring?
</summary>

1. **Add `spring-context`** to the project (Spring Boot starters already include it).
2. **Enable component scanning:** `@ComponentScan("com.example")` on a `@Configuration` class (or `<context:component-scan base-package="..."/>` in XML). This also turns on annotation processing.
3. **Annotate your classes:** `@Component`/`@Service`/`@Repository`/`@Controller`, `@Configuration` + `@Bean`, `@Autowired`, `@Value`, `@PostConstruct`.
4. **Start a context** that understands annotations, e.g. `new AnnotationConfigApplicationContext(AppConfig.class)`.

In **Spring Boot** all of this is automatic: `@SpringBootApplication` includes `@ComponentScan`, and `SpringApplication.run(...)` creates an annotation-based context.

**Example:**

```java
@Configuration
@ComponentScan(basePackages = "com.example")   // find @Component/@Service/... classes
public class AppConfig { }

// Spring Boot: @SpringBootApplication already includes @Configuration,
// @EnableAutoConfiguration and @ComponentScan of the main class's package.
```

</details>

<details><summary>
  
### [Easy] Can there be multiple configuration files in a project?
</summary>

Yes, it is possible to have multiple configuration files in a project in Spring. Having multiple configuration files can help modularize and organize the configuration of different components or subsystems within the project.

**Example:**

```java
@Configuration class DataConfig { @Bean DataSource ds() { ... } }
@Configuration class KafkaConfig { @Bean ProducerFactory<String, String> pf() { ... } }

@Configuration
@Import({DataConfig.class, KafkaConfig.class})   // compose explicitly when not scanned
class RootConfig { }
```

</details>

<details><summary>
  
### [Easy] What is component scan?
</summary>

Component scanning is a feature in Spring that automatically detects and registers beans based on certain conventions and annotations in your codebase. It eliminates the need to explicitly define each bean in a configuration file by scanning the specified packages for classes that should be managed as Spring beans.

In simpler terms, component scanning is a way for Spring to automatically discover and create beans for you without the need for manual configuration. By enabling component scanning, Spring scans the specified packages and identifies classes that are annotated with certain annotations, such as @Component, @Controller, @Service, or @Repository. It then registers these classes as beans in the Spring container, making them available for dependency injection or other bean-related operations.

Component scanning simplifies the configuration process, especially in larger projects, as you don't have to explicitly list and configure every bean. Instead, you can annotate your classes appropriately, and Spring will take care of instantiating and managing them as beans.

To enable component scanning, you typically use the @ComponentScan annotation at the configuration class level. This annotation specifies the base package or packages to scan for annotated classes. Spring will scan those packages and their sub-packages, identifying classes annotated with relevant annotations and registering them as beans.

Component scanning is a powerful feature that promotes convention-over-configuration and can significantly reduce the amount of boilerplate configuration code needed in a Spring application.

**Example:**

```java
@Configuration
@ComponentScan(
    basePackages = "com.example",
    excludeFilters = @ComponentScan.Filter(type = FilterType.REGEX, pattern = "com\\.example\\.legacy\\..*"))
class AppConfig { }
```

</details>

<details><summary>
  
### [Easy] How is component scan done in Spring Boot?
</summary>

In Spring Boot, component scanning is enabled by default, allowing for automatic detection and registration of beans without explicit configuration. Spring Boot simplifies component scanning by providing a sensible default configuration based on the project's structure.

Here's how component scanning is done in Spring Boot:

### 1. Default Package Scanning:
Spring Boot automatically performs component scanning starting from the package where the main application class is located. By convention, this main class resides in the root package of the application. Spring Boot scans its sub-packages to discover and register beans.

### 2. Component Scan Annotations:
Spring Boot recognizes and applies component scan annotations to further customize the scanning behavior. The @SpringBootApplication annotation, which is typically added to the main application class, includes the @ComponentScan annotation implicitly. This combination enables component scanning for the main application package and its sub-packages.

### 3. Additional Configuration:
If you have components or configurations located outside the default package or its sub-packages, you can explicitly specify the packages to be scanned using the @ComponentScan annotation. By providing the package names as arguments to @ComponentScan, you can extend the scanning to include additional packages.

### 4. Customization:
Spring Boot allows further customization of component scanning through various annotations. For example, the @ComponentScan annotation supports attributes like basePackages, basePackageClasses, includeFilters, and excludeFilters, allowing fine-grained control over the scanning process.

By default, Spring Boot's component scanning feature automatically detects and registers beans based on the project's structure and conventions, reducing the need for explicit configuration. However, you can still customize and extend the scanning behavior as required for your specific application by using annotations and additional configuration options provided by Spring Boot.

**Example:**

```java
package com.example.app;                       // scanning starts here and goes into sub-packages

@SpringBootApplication
public class App {
    public static void main(String[] args) { SpringApplication.run(App.class, args); }
}

// com.example.app.service.OrderService  -> found
// com.example.shared.Utils              -> NOT found (outside the package); fix with:
@SpringBootApplication(scanBasePackages = {"com.example.app", "com.example.shared"})
class App2 { }
```

</details>

<details><summary>
  
### [Medium] What is the difference between context: annotation-config and context:component-scan tags?
</summary>

Both are XML-configuration tags:

- **`<context:annotation-config/>`** registers the post-processors that **process annotations on beans that are already defined** (in XML or elsewhere): `@Autowired`, `@Value`, `@Inject`, `@Resource`, `@PostConstruct`, `@PreDestroy`. It does **not** discover new classes.
- **`<context:component-scan base-package="..."/>`** **scans packages for `@Component`/`@Service`/`@Repository`/`@Controller` classes and registers them as beans**, and it **also enables everything `annotation-config` does**.

So if you use `component-scan`, you don't need `annotation-config`. In Java configuration, `@ComponentScan` plays the same role, and annotation processing is always on with `AnnotationConfigApplicationContext`.

**Example:**

```xml
<!-- only activates @Autowired, @PostConstruct etc. on beans already declared -->
<context:annotation-config/>

<!-- also discovers and registers @Component classes (includes annotation-config) -->
<context:component-scan base-package="com.example"/>
```

</details>
