<details><summary>

## [Easy] What is Spring Boot?
</summary>

Spring Boot is an open-source Java-based framework that simplifies the development of standalone, production-grade applications. It is a part of the broader Spring Framework ecosystem, designed to provide a streamlined way to create Java applications with minimal configuration and boilerplate code.

Spring Boot aims to address the complexity often associated with setting up and configuring Spring-based applications. It adopts an opinionated approach by providing defaults and auto-configuration, allowing developers to quickly build applications with sensible defaults while still retaining the flexibility to customize and override configurations when needed.

Key features and benefits of Spring Boot include:

**1. Auto-configuration:** Spring Boot automatically configures the application based on classpath dependencies and sensible defaults, reducing the need for manual configuration.

**2. Standalone applications:** Spring Boot applications are self-contained and can be deployed as standalone JAR files, which simplifies deployment and eliminates the need for setting up complex application servers.

**3. Embedded web servers:** Spring Boot provides support for embedding servlet containers like Apache Tomcat (default) or Jetty (Undertow too, up to Boot 3), allowing developers to create web applications without the need for deploying them on a separate server.

**4. Dependency management:** Spring Boot manages dependencies for you, ensuring compatibility and reducing conflicts by providing a curated list of compatible versions for popular libraries.

**5. Actuator:** Spring Boot Actuator is a module that provides production-ready features for monitoring and managing applications, including health checks, metrics, and management endpoints.

**6. Developer tools:** Spring Boot offers a set of development tools that enhance productivity, such as automatic application restart, live reloading of changes, and detailed error reporting.

Overall, Spring Boot simplifies the development process, improves productivity, and promotes best practices for building Java applications. It has gained significant popularity due to its ease of use and extensive ecosystem support, making it a preferred choice for developing microservices, web applications, and RESTful APIs in the Java ecosystem.

**Example:**

```java
@SpringBootApplication
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);   // embedded Tomcat on :8080
    }
}

@RestController
class HelloController {
    @GetMapping("/hello") String hello() { return "Hello"; }
}
// Build: mvn package -> java -jar target/order-service.jar
```

</details>

<details><summary>

## [Easy] Why do we need Spring Boot?
</summary>

We need Spring Boot because it simplifies the development of Java applications by providing defaults, auto-configuration, and an opinionated approach. It reduces the complexity of setting up and configuring Spring applications, allows for standalone deployment, and offers features like embedded web servers, dependency management, and developer tools. Overall, Spring Boot enhances productivity, promotes best practices, and is widely used for developing microservices, web applications, and RESTful APIs in the Java ecosystem.

**Example:**

Plain Spring needed manual setup of the servlet container, `DispatcherServlet`, data source, JSON converters and many XML/Java config classes. With Boot, one dependency plus a property does it:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/app
```

Boot auto-configures the `DataSource` (HikariCP), `EntityManagerFactory`, transaction manager and repositories.

</details>

<details><summary>

## [Easy] How can a Spring Boot application be created?
</summary>

1. **Install a JDK** – Java 17 or later for Spring Boot 3 and 4.
2. **Generate the project** with **Spring Initializr** (start.spring.io, or built into IntelliJ IDEA / VS Code / STS): choose Maven or Gradle, Java version, Boot version and dependencies such as Web, Data JPA, Validation, Actuator.
3. The generated `pom.xml` uses **`spring-boot-starter-parent` as its parent POM** (it is a parent, not an archetype) and includes the chosen starters plus `spring-boot-maven-plugin`.
4. The generated main class is annotated with **`@SpringBootApplication`** and calls `SpringApplication.run(App.class, args)`.
5. **Add code** in sub-packages of the main class's package: controllers, services, repositories, configuration.
6. **Configure** in `application.yml` / `application.properties`.
7. **Run** from the IDE, with `./mvnw spring-boot:run`, or build a JAR (`./mvnw package`) and run `java -jar target/app.jar`.
8. **Test** endpoints with curl/Postman and write tests with `@SpringBootTest`, `@WebMvcTest`, etc.

**Example:**

```bash
# Spring Initializr from the command line
curl https://start.spring.io/starter.zip \
  -d dependencies=web,data-jpa,mysql,validation,actuator \
  -d javaVersion=21 -d name=order-service -o order-service.zip

unzip order-service.zip && cd order-service && ./mvnw spring-boot:run
```

Or use start.spring.io in the browser, or IntelliJ's "New Project → Spring Boot".

</details>

<details><summary>

## [Easy] What is Spring Initializr?
</summary>

Spring Initializr is a web-based tool provided by the Spring team that helps in creating and generating the initial structure of a new Spring Boot project. It offers a user-friendly interface where developers can specify project metadata, dependencies, and settings, and then generates a project skeleton with the necessary files and configurations.

Key features of Spring Initializr include:

**1. Project setup:** Developers can select the project's build tool (Maven or Gradle), programming language (Java or Kotlin), and Spring Boot version. They can also provide additional project metadata like group and artifact names.

**2. Dependency selection:** Spring Initializr provides a list of commonly used dependencies and starters. Developers can choose the dependencies they require for their project, such as web, data, security, testing, and more. These dependencies are automatically added to the project's configuration files and build files.

**3. Customization options:** Developers can further customize their project by enabling or disabling specific features and configurations. For example, they can choose the packaging type (JAR or WAR), Java version compatibility, and language-specific settings for Kotlin projects.

**4. Download project skeleton:** Once all the necessary selections are made, Spring Initializr generates a project structure and configuration files based on the selected options. Developers can then download the generated project as a ZIP file and import it into their preferred IDE for further development.

Spring Initializr provides an easy and standardized way to bootstrap new Spring Boot projects by eliminating the need to manually set up the project structure, dependencies, and initial configurations. It ensures that developers start with a solid foundation and can quickly get up and running with their Spring Boot applications.

**Example:**

```bash
curl https://start.spring.io/starter.zip -d dependencies=web,actuator -d type=maven-project -o demo.zip
```

It generates the project skeleton: `pom.xml` with starters, the main class, `application.properties` and a test class.

</details>

<details><summary>

## [Easy] What are the components of Spring Boot?
</summary>

The components of Spring Boot include:

**1. Spring Boot Starter:** Starter dependencies are a set of curated dependencies that provide a specific functionality or feature to a Spring Boot application. They simplify dependency management by bringing in all the required dependencies and configurations for a particular feature, such as web, data access, security, testing, etc.

**2. Auto-configuration:** Spring Boot's auto-configuration feature automatically configures the Spring application based on the classpath dependencies and sensible defaults. It analyzes the project's dependencies and sets up the necessary configurations, eliminating the need for manual configuration.

**3. Spring Boot Actuator:** Spring Boot Actuator provides production-ready features for monitoring and managing applications. It includes endpoints for health checks, metrics, logging, auditing, and other management operations. Actuator helps in monitoring and maintaining the application in a production environment.

**4. Spring Boot CLI:** The Spring Boot Command-Line Interface (CLI) allows developers to quickly create and prototype Spring Boot applications using a command-line tool. It provides a faster way to develop and test Spring Boot applications without requiring the overhead of a full-fledged IDE.

**5. Embedded web servers:** Spring Boot supports embedding servlet containers like Apache Tomcat (default) and Jetty (Undertow too, up to Boot 3) directly into the application. This allows developers to create standalone web applications that can be run without the need for deploying them on a separate web server.

**6. DevTools:** Spring Boot DevTools is a set of development tools that enhance productivity during the development process. It provides features like automatic application restart, live reloading of changes, and detailed error reporting, improving the developer's experience and speeding up the development cycle.

These components work together to simplify the development process, promote best practices, and enable rapid application development with Spring Boot. They provide a comprehensive framework for building production-grade Java applications with minimal configuration and boilerplate code.

**Example:**

- **Starters:** `spring-boot-starter-web`, `-data-jpa`, `-security`
- **Auto-configuration:** beans created based on the classpath and properties
- **Spring Boot CLI**
- **Actuator:** `/actuator/health`, `/actuator/metrics`

```properties
management.endpoints.web.exposure.include=health,info,metrics
```

</details>

<details><summary>

## [Easy] What is the purpose of @SpringBootApplication annotation?
</summary>

The **@SpringBootApplication** annotation is a combination of three annotations: **@Configuration**, **@EnableAutoConfiguration**, and **@ComponentScan**. It is a convenience annotation provided by Spring Boot to simplify the configuration of a Spring application.

The purpose of the **@SpringBootApplication** annotation is to mark the main class of a Spring Boot application. By applying this annotation to a class, it performs the following tasks:

**1. @Configuration:** The **@Configuration** annotation indicates that the class is a source of bean definitions. It allows the class to define and configure beans in the application context. In the case of **@SpringBootApplication**, it implicitly declares the class as a configuration class, providing bean definition capabilities.

**2. @EnableAutoConfiguration:** The **@EnableAutoConfiguration** annotation enables Spring Boot's auto-configuration feature. It automatically configures the Spring application based on the classpath dependencies and sensible defaults. It analyzes the project's dependencies, detects the available configurations, and applies the necessary configurations for the application to function correctly.

**3. @ComponentScan:** The **@ComponentScan** annotation enables component scanning in the application. It instructs Spring to scan and detect components such as controllers, services, repositories, and other Spring-managed beans within specific packages. By default, it scans the package where the **@SpringBootApplication** class is located and its sub-packages.

In summary, the **@SpringBootApplication** annotation serves as the entry point and configuration class for a Spring Boot application. It combines the functionality of **@Configuration**, **@EnableAutoConfiguration**, and **@ComponentScan** to simplify the setup and configuration of a Spring application. By applying this single annotation, developers can start building a Spring Boot application with sensible defaults and auto-configuration, reducing the need for manual configuration.

**Example:**

```java
@SpringBootApplication     // = @SpringBootConfiguration + @EnableAutoConfiguration + @ComponentScan
public class App {
    public static void main(String[] args) { SpringApplication.run(App.class, args); }
}

// Exclude an auto-configuration:
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
class NoDbApp { }
```

</details>

<details><summary>

## [Easy] What is the purpose of @RestController annotation in Spring Boot?
</summary>

`@RestController` comes from **Spring MVC** (Spring 4.0) and is used heavily in Spring Boot. It is `@Controller` + `@ResponseBody`:

- The class is detected as a web **request handler**, and its methods are mapped with `@GetMapping`, `@PostMapping`, `@RequestMapping`, etc.
- Every method's return value is written **directly to the HTTP response body** by an `HttpMessageConverter` (JSON via Jackson by default, XML if Jackson XML is on the classpath and the client asks for it) instead of being treated as a view name.
- No view resolver or template is involved, which is exactly what REST APIs need.

Return a plain object for a `200 OK`, or `ResponseEntity<T>` when you need to control the status code and headers (e.g. `201 Created` with a `Location` header).

**Example:**

```java
@RestController                    // = @Controller + @ResponseBody on every method
@RequestMapping("/api/users")
class UserController {
    @GetMapping("/{id}")
    UserDto get(@PathVariable Long id) { return new UserDto(id, "Asha"); }   // serialized to JSON
}
// GET /api/users/1 -> {"id":1,"name":"Asha"}
```

</details>

<details><summary>

## [Easy] What is an embedded server in Spring Boot?
</summary>

An embedded server in Spring Boot refers to the capability of running a web server directly within the Spring Boot application itself. Instead of deploying the application to an external server like Apache Tomcat or Jetty, Spring Boot provides the option to include an embedded server within the application, making it self-contained and independent of any external server installation.

The embedded server acts as a lightweight servlet container that can handle HTTP requests and serve web content. It eliminates the need for manual server setup, deployment, and configuration, simplifying the deployment process and making it easier to develop and deploy Spring Boot applications.

Spring Boot supports various embedded servers, including Tomcat (the default) and Jetty, plus Netty for reactive WebFlux apps (Undertow was supported up to Boot 3). These servers are packaged as dependencies in the application, and the necessary configurations are automatically handled by Spring Boot's auto-configuration mechanism.

When a Spring Boot application is started, the embedded server starts along with it, listening for incoming HTTP requests. The application's controllers and request mappings are processed by the embedded server, allowing it to handle and respond to web requests without the need for an external server.

The use of an embedded server in Spring Boot provides benefits such as simplicity, portability, and ease of deployment. It allows developers to create self-contained, standalone applications that can be run with a simple command or by executing the main class. The embedded server capability is one of the key features that make Spring Boot an attractive framework for developing web applications and microservices.

**Example:**

```yaml
server:
  port: 8080
  tomcat:
    threads:
      max: 200          # max request-processing threads
    accept-count: 100   # queue when all threads are busy
    connection-timeout: 5s
```

</details>

<details><summary>

## [Easy] Can we change the default embedded server to Jetty?
</summary>

Yes. Exclude the default Tomcat starter from the web starter and add the Jetty starter:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
	<exclusions>
		<exclusion>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
		</exclusion>
	</exclusions>
</dependency>
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

Boot's auto-configuration detects Jetty on the classpath and starts it instead of Tomcat. Common settings (`server.port`, SSL, compression) stay the same; Jetty-specific ones use the `server.jetty.*` prefix (e.g. `server.jetty.threads.max`).

Note: **Undertow** was supported in Boot 2 and 3, but **Spring Boot 4 dropped Undertow** because it isn't yet compatible with Servlet 6.1. For reactive (WebFlux) apps the default server is **Netty**.

</details>

<details><summary>

## [Easy] How are Spring Boot web applications deployed?
</summary>

Spring Boot web applications can be deployed in various ways, depending on the specific deployment requirements and preferences. Here are some common deployment options for Spring Boot web applications:

### 1. Standalone JAR:
One of the key features of Spring Boot is the ability to create executable JAR files. You can build a standalone JAR file that includes all the application dependencies and an embedded server (such as Tomcat or Jetty). This JAR file can be executed directly using the java -jar command, and the application will start the embedded server and serve the web content.

### 2. Traditional WAR file deployment:
If you prefer to deploy the Spring Boot application as a traditional WAR (Web Application Archive) file, you can configure the application to create a deployable WAR file instead of an executable JAR. This allows you to deploy the WAR file to a standalone servlet container like Apache Tomcat, Jetty, or others.

### 3. Cloud platforms and PaaS providers:
Spring Boot applications are well-suited for deployment on various cloud platforms and Platform-as-a-Service (PaaS) providers. Cloud providers like AWS, Azure, Google Cloud, and Heroku offer specific deployment options and integrations for Spring Boot applications. These platforms provide streamlined deployment processes, scalability, and management capabilities for your applications.

### 4. Containerization with Docker:
Spring Boot applications can be packaged as Docker containers, providing a consistent and portable deployment environment. By creating a Docker image of your application, you can run it in any environment that supports Docker containers, including local development environments, on-premises servers, or cloud-based container orchestration platforms like Kubernetes.

### 5. Serverless deployment:
With the rise of serverless computing, it is also possible to deploy Spring Boot applications as serverless functions. Platforms like AWS Lambda, Azure Functions, and Google Cloud Functions allow you to package and deploy Spring Boot applications as individual functions that can be triggered by specific events.

These deployment options offer flexibility and cater to different deployment scenarios. Spring Boot's ease of configuration and self-contained nature make it adaptable to a wide range of deployment environments, whether it's a standalone server, cloud platform, container, or serverless architecture.

**Example:**

```dockerfile
FROM eclipse-temurin:21-jre
WORKDIR /app
COPY target/order-service.jar app.jar
ENTRYPOINT ["java", "-XX:MaxRAMPercentage=75", "-jar", "app.jar"]
```

```bash
./mvnw spring-boot:build-image      # or build an OCI image with Buildpacks, no Dockerfile
docker run -p 8080:8080 order-service
```

Then deploy the image to Kubernetes with a Deployment, Service, liveness/readiness probes on `/actuator/health/liveness` and `/readiness`, and an HPA for autoscaling.

</details>

<details><summary>

## [Easy] What is Spring Boot Dependency Management?
</summary>

Spring Boot Dependency Management is a feature that simplifies the management of dependencies in a Spring Boot application. It provides a curated list of dependencies, known as starters, which encapsulate common sets of dependencies required for specific functionalities, such as web development, data access, security, testing, and more.

With Spring Boot Dependency Management, you no longer need to manually specify and manage individual dependency versions in your project. Instead, you declare the starters as dependencies in your project's build configuration file (e.g., pom.xml for Maven or build.gradle for Gradle), and Spring Boot takes care of managing the versions and transitive dependencies.

By leveraging the starters provided by Spring Boot, you ensure that all dependencies are compatible and work seamlessly together. Spring Boot Dependency Management ensures that the selected starters are consistent and tested as a whole, reducing the risk of version conflicts and compatibility issues.

Additionally, Spring Boot Dependency Management allows you to override specific dependency versions if needed. This gives you the flexibility to use a different version of a dependency while still benefiting from the managed versions of other dependencies.

In summary, Spring Boot Dependency Management simplifies the management of dependencies by providing curated starters and handling version management automatically. It ensures compatibility, reduces conflicts, and simplifies the configuration process, allowing developers to focus more on application development rather than managing dependencies.

**Example:**

```xml
<dependency>
  <groupId>com.fasterxml.jackson.core</groupId>
  <artifactId>jackson-databind</artifactId>   <!-- no <version>: managed by the Boot BOM -->
</dependency>

<properties>
  <jackson-bom.version>2.17.2</jackson-bom.version>   <!-- override a managed version if needed -->
</properties>
```

Without the parent POM, import the BOM: `spring-boot-dependencies` with `<scope>import</scope>` in `dependencyManagement`.

</details>

<details><summary>

## [Easy] How can the default port of the embedded tomcat server be changed?
</summary>

To change the default port of the embedded Tomcat server in a Spring Boot application, you can modify the application's configuration. Here are the steps to do so:

- Open the application.properties file or create one if it doesn't exist in your project's src/main/resources directory.

- Add the following line to specify the desired port number:

```
server.port=your_desired_port_number
```

Replace your_desired_port_number with the port number you want to use for the embedded Tomcat server. For example, if you want to use port 8081, the line would be server.port=8081.

- Save the application.properties file.

- Restart your Spring Boot application.

Upon restart, the embedded Tomcat server will start and listen on the specified port instead of the default port (which is 8080 if not modified). Make sure to update any client applications or URLs to reflect the new port number.

You can also specify the port number through other means such as command-line arguments, system properties, or environment variables. However, using the server.port property in application.properties is the most common and straightforward approach.
</details>

<details><summary>

## [Easy] How can the default web server in a Spring Boot application be disabled?
</summary>

Tell Spring Boot not to start a web server:

```yaml
spring:
  main:
    web-application-type: none     # options: servlet, reactive, none
```

or in code:

```java
new SpringApplicationBuilder(App.class)
	.web(WebApplicationType.NONE)
	.run(args);
```

Alternatively, don't include `spring-boot-starter-web` / `spring-boot-starter-webflux` at all. Boot then starts as a plain (non-web) application, which is the right setup for batch jobs, Kafka consumers, CLI tools and scheduled workers. Excluding only the Tomcat JAR while keeping Spring MVC on the classpath leads to startup errors, so that is not the way to do it.

To use a different port instead, set `server.port`; `server.port=0` picks a random free port (useful in tests).

</details>

<details><summary>

## [Easy] What are Spring Boot starter projects?
</summary>

Spring Boot starter projects are a set of curated dependencies bundled together to provide a convenient way to include commonly used functionality in Spring Boot applications. They simplify dependency management and configuration by offering a cohesive set of dependencies for specific use cases or technologies.

Each starter project focuses on a particular area or feature of application development, such as web development, data access, security, messaging, testing, etc. They provide a consistent and opinionated configuration, reducing the need for manual configuration and ensuring that the dependencies work well together.

Using a starter project involves adding a single dependency to your project, which automatically brings in all the required dependencies for the chosen functionality. This simplifies the setup process, as you don't need to manually identify and add individual dependencies. Spring Boot starters also handle the version management of the dependencies, ensuring compatibility and reducing the chances of version conflicts.

For example, if you are building a web application, you can include the spring-boot-starter-web starter project. It includes dependencies for web-related functionality like embedded server (Tomcat or Jetty), Spring MVC, and other related libraries. By adding this starter dependency to your project, you get all the necessary dependencies for web development without having to specify each one individually.

Starter projects are an essential aspect of Spring Boot's convention-over-configuration approach, enabling developers to quickly get started with specific functionalities and focus more on application development rather than managing dependencies and configurations.

**Example:**

```xml
<!-- one starter pulls in Spring MVC, Jackson, Tomcat, validation-ready setup -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Common starters: `-web`, `-data-jpa`, `-security`, `-validation`, `-actuator`, `-test`, `-data-redis`, `-cache`, plus `spring-kafka`.

</details>

<details><summary>

## [Easy] What is spring-boot-starter-parent?
</summary>

**spring-boot-starter-parent** is a special starter project in Spring Boot that provides a parent POM (Project Object Model) for Maven-based projects. It defines a consistent set of configurations and dependencies that are inherited by the child projects.

By specifying spring-boot-starter-parent as the parent POM in your project, you benefit from various features:

**1. Dependency Management:** The parent POM defines the versions of commonly used dependencies, including Spring Boot itself. This ensures that the project dependencies are compatible and tested together.

**2. Plugin Management:** The parent POM manages the versions of Maven plugins, ensuring consistent and recommended plugin configurations for Spring Boot projects.

**3. Default Configuration:** The parent POM sets sensible default configurations, such as resource filtering, build profiles, and packaging options. It provides a solid foundation for building Spring Boot applications.

**4. Additional Functionality:** The parent POM brings in additional features like the Spring Boot Maven Plugin, which simplifies the packaging and running of Spring Boot applications.

Using spring-boot-starter-parent as the parent POM allows you to inherit these benefits and reduces the need for explicit configuration and dependency management in your Maven-based Spring Boot projects. It promotes best practices and helps maintain consistency across multiple projects within an organization.

In summary, spring-boot-starter-parent is a special starter project that provides a parent POM with predefined configurations, dependency management, and additional functionality for Maven-based Spring Boot projects. It simplifies project setup, promotes consistency, and enhances development efficiency.

**Example:**

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>3.3.5</version>
</parent>
```

It provides dependency versions (BOM), the Java version and UTF-8 encoding defaults, resource filtering and plugin configuration (`spring-boot-maven-plugin`, Surefire).

</details>

<details><summary>

## [Easy] How can we connect Spring Boot with databases?
</summary>

To connect Spring Boot with databases, you can utilize Spring Data, a subproject of the Spring Framework that provides a unified and simplified way to interact with different databases. Here's a brief overview of the steps involved:

**1. Include Database Driver Dependency:** In your project's build configuration file (e.g., pom.xml for Maven or build.gradle for Gradle), include the appropriate database driver dependency for the database you want to connect to. For example, if you're using MySQL, include the MySQL Connector/J dependency.

**2. Configure Database Connection:** In the application.properties or application.yml file, specify the database connection details such as the URL, username, and password. Spring Boot uses these properties to establish a connection with the database. The configuration properties vary depending on the chosen database.

**3. Define Entity Classes:** Create Java entity classes that represent the tables or collections in your database. Annotate these classes with appropriate annotations like @Entity, @Table, and @Column to define the mapping between the entities and the database schema.

**4. Create Repositories:** Define Spring Data repositories, which provide a high-level abstraction for performing database operations. Spring Data repositories can be created by extending interfaces like CrudRepository or JpaRepository. These interfaces offer common CRUD (Create, Read, Update, Delete) operations and additional query methods.

**5. Use Spring Data JPA or MongoDB:** If you're using a relational database, you can use Spring Data JPA, which is a subproject of Spring Data tailored for relational databases. Spring Data JPA provides a powerful way to work with relational databases using Java Persistence API (JPA). If you're using a NoSQL database like MongoDB, you can use Spring Data MongoDB, which provides similar functionality for MongoDB.

**6. Inject and Use Repositories:** Inject the Spring Data repositories into your application's components, such as controllers or services, and use them to perform database operations. Spring Boot automatically generates implementations for the repository interfaces based on the defined methods, allowing you to interact with the database without writing boilerplate code.

By following these steps and utilizing Spring Data, you can easily connect Spring Boot with different databases and perform database operations in a standardized and efficient manner. Spring Data abstracts away many of the complexities associated with database interaction, allowing you to focus more on your application's business logic.

**Example:**

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/app
    username: ${DB_USER}
    password: ${DB_PASSWORD}
  jpa:
    hibernate:
      ddl-auto: validate        # never 'update' in production; use Flyway/Liquibase migrations
    open-in-view: false
```

```java
@Entity
class Company { @Id @GeneratedValue Long id; String name; }

interface CompanyRepository extends JpaRepository<Company, Long> {
    List<Company> findByNameContainingIgnoreCase(String q);   // query derived from the method name
}
```

</details>

<details><summary>

## [Easy] How can external configuration be done in Spring Boot?
</summary>

In Spring Boot, external configuration can be done by utilizing properties files, YAML files, environment variables, command-line arguments, and other externalized configuration sources. Here's a brief overview of how you can perform external configuration:

**1. Properties File:** Create a file named application.properties or application.yml in your project's src/main/resources directory. Define key-value pairs in the file, where the keys represent the configuration properties and the values specify their corresponding values.

**2. YAML File:** Alternatively, you can use a YAML file (application.yml) instead of a properties file. YAML provides a more readable and structured format for configuration. Define the properties and their values in a hierarchical structure using indentation.

**3. Override Configuration:** Spring Boot automatically reads the properties from the application.properties or application.yml file and applies them during application startup. You can override these properties by providing your own values through different externalized configuration sources.

**4. Environment Variables:** You can set environment variables with the same names as your configuration properties. When the application starts, Spring Boot checks for corresponding environment variables and uses them to override the properties defined in the properties file.

**5. Command-line Arguments:** You can pass command-line arguments to your Spring Boot application using the syntax --property=value. These arguments are also considered when resolving configuration properties, allowing you to override them during runtime.

**6. Profile-specific Configuration:** Spring Boot allows you to define profile-specific configuration properties. For example, you can have separate properties for development, testing, and production environments. By using the application-{profile}.properties or application-{profile}.yml naming convention, you can provide different configuration values based on the active profile.

By leveraging these externalized configuration options, you can modify the behavior of your Spring Boot application without changing the underlying code. This makes your application more flexible and allows for easy configuration management across different environments and deployment scenarios.

**Example:**

```yaml
# application.yml
app:
  pricing:
    currency: INR
```

```bash
java -jar app.jar --app.pricing.currency=USD           # command-line argument
APP_PRICING_CURRENCY=USD java -jar app.jar             # environment variable (relaxed binding)
java -jar app.jar --spring.config.additional-location=/etc/app/
```

</details>

<details><summary>

## [Easy] What is Spring Actuator?
</summary>

**Spring Boot Actuator** (`spring-boot-starter-actuator`) adds production-ready features to a Boot app through HTTP (or JMX) endpoints under `/actuator`:

- **`health`** – application health, with groups for Kubernetes **liveness** and **readiness** probes, and indicators for the DB, Redis, Kafka, disk space, etc.
- **`metrics`** and **`prometheus`** – application and JVM metrics collected with **Micrometer** (Actuator's metrics facade), exportable to Prometheus, Datadog, CloudWatch and others.
- **`info`** – build/git/app info.
- **`loggers`** – view and change log levels at runtime.
- **`env`**, **`configprops`**, **`beans`**, **`mappings`**, **`conditions`** – inspect configuration and wiring (sensitive; don't expose publicly).
- **`threaddump`**, **`heapdump`** – diagnostics.
- **`shutdown`** – graceful shutdown (disabled by default).

By default only `health` is exposed over HTTP. Expose others explicitly with `management.endpoints.web.exposure.include`, and protect them with Spring Security or a separate management port.

**Example:**

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      probes:
        enabled: true      # /actuator/health/liveness and /readiness for Kubernetes
```

```bash
curl localhost:8080/actuator/health        # {"status":"UP"}
curl localhost:8080/actuator/metrics/jvm.memory.used
```

</details>

<details><summary>

## [Easy] How can the Actuator be accessed in Spring Boot?
</summary>

1. Add `spring-boot-starter-actuator`.
2. Call the endpoints under `/actuator` on the app's port (or on `management.server.port` if you set a separate one), e.g. `GET http://localhost:8080/actuator/health`.
3. **Exposure:** by default only `/actuator/health` is exposed over HTTP. Expose more explicitly:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus,loggers
  endpoint:
    health:
      show-details: when-authorized
      probes:
        enabled: true
  server:
    port: 9090          # optional: separate management port, not public
```

4. **Secure** them: `env`, `configprops`, `heapdump` and `loggers` reveal or change sensitive state. Restrict them with Spring Security roles or keep the management port internal. Avoid `include: "*"` in production.

</details>

<details><summary>

## [Easy] Explain Spring Boot DevTools.
</summary>

`spring-boot-devtools` improves the **local development** loop:

- **Automatic restart:** when compiled classes change, the app restarts quickly using two class loaders (one for unchanging library JARs, one for your code).
- **LiveReload:** triggers a browser refresh when resources change (needs a LiveReload browser extension).
- **Development-friendly defaults:** template caching disabled, more detailed web error logging.
- **Remote support** (optional) for updating a remotely running app during development.

DevTools is **automatically disabled** when the app runs from a fully packaged JAR (`java -jar`), and should be declared `optional`/`developmentOnly` so it never ships to production.

**Example:**

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <scope>runtime</scope>
  <optional>true</optional>   <!-- not packaged into the production JAR -->
</dependency>
```

Gives automatic restart on classpath changes, LiveReload, and dev-friendly defaults (template caching off).

</details>

<details><summary>

## [Medium] What is the purpose of @ConditionalOnMissingBean annotation?
</summary>

The purpose of the **@ConditionalOnMissingBean** annotation is to conditionally enable a bean definition based on the absence of a specific bean in the Spring application context. If the specified bean does not exist, the annotated bean definition will be created and added to the context. However, if the specified bean already exists, the annotated bean definition will not be created.

In short, **@ConditionalOnMissingBean** allows conditional bean creation based on the absence of a specific bean.

**Example:**

```java
@Configuration
class ClockConfig {
    @Bean
    @ConditionalOnMissingBean         // only created if the app didn't define its own Clock
    Clock clock() { return Clock.systemUTC(); }
}

// In a test config, defining your own Clock bean makes the default back off:
@TestConfiguration
class FixedClock { @Bean Clock clock() { return Clock.fixed(Instant.parse("2026-01-01T00:00:00Z"), ZoneOffset.UTC); } }
```

</details>

<details><summary>

## [Medium] What happens when a web application is run using Spring Boot?
</summary>

When a web application is run using Spring Boot, several key steps are executed to initialize and start the application. Here's an overview of what happens:

**1. Application Startup:** The Spring Boot application starts by executing the main method in the designated main class, typically annotated with **@SpringBootApplication**. This class serves as the entry point for the application.

**2. Auto-configuration:** Spring Boot's auto-configuration feature analyzes the classpath and dependencies to automatically configure the application. It detects the presence of various libraries and frameworks and applies sensible defaults and configurations based on conventions.

**3. Embedded Server Initialization:** If the application includes an embedded server (such as Tomcat or Jetty), it is initialized and started. The embedded server provides the runtime environment for the application to handle incoming HTTP requests and serve responses.

**4. Component Scanning:** Spring Boot scans the application's packages and sub-packages to identify components such as controllers, services, repositories, and other Spring-managed beans. It uses component scanning annotations like **@Component**, **@RestController**, **@Service**, etc., to detect and register these components in the application context.

**5. Bean Creation and Dependency Injection:** Spring Boot creates instances of the detected beans and manages their lifecycle. It performs dependency injection, injecting dependencies into beans based on their configurations and annotations (e.g., @Autowired).

**6. Request Mapping and Handling:** Spring Boot maps incoming HTTP requests to the appropriate controllers and methods based on annotations like **@RequestMapping**, **@GetMapping**, **@PostMapping**, etc. The controllers process the requests, execute business logic, interact with services and repositories, and prepare the response.

**7. View Resolution (if applicable):** If the application involves rendering views, Spring Boot resolves the views based on the configured view resolver. It processes templates, combines data from the model, and generates the final HTML, JSON, or other view formats.

**8. Error Handling and Exception Resolution:** Spring Boot handles exceptions and errors that occur during request processing. It maps exceptions to appropriate error codes and formats error responses to provide meaningful feedback to clients.

**9. Application Lifecycle Management:** Spring Boot manages the lifecycle of the application, including initialization, configuration updates, graceful shutdown, and other management operations. It provides features like actuator endpoints for health checks, metrics, and monitoring.

**10. Application Execution:** With all the components initialized and the server running, the Spring Boot application is ready to handle incoming requests. It listens for HTTP requests on the configured port and executes the appropriate controllers to process and respond to the requests.

In summary, when a web application is run using Spring Boot, the framework initializes and configures the application, starts an embedded server, detects and registers components, handles incoming requests, processes business logic, and manages the application's lifecycle. Spring Boot provides a streamlined and opinionated approach to web application development, allowing developers to focus on writing business logic rather than managing low-level configurations.

**Example:**

```java
SpringApplication.run(App.class, args);
// 1. Create the environment (properties, profiles, env vars)
// 2. Create the ApplicationContext and run component scanning
// 3. Apply auto-configurations whose conditions match
// 4. Create singleton beans, inject dependencies, run @PostConstruct
// 5. Start embedded Tomcat, register DispatcherServlet
// 6. Run CommandLineRunner/ApplicationRunner beans, publish ApplicationReadyEvent
```

</details>

<details><summary>

## [Medium] Can Spring Boot be used for applications not built using the Spring framework?
</summary>

**Not really.** Spring Boot is built **on top of** the Spring Framework: auto-configuration, starters, externalised configuration, the embedded server lifecycle and Actuator all work by creating and configuring a Spring `ApplicationContext`. A Spring Boot application is always a Spring application.

What's possible:
- You can bring **existing non-Spring code** (plain Java libraries, legacy classes) into a Boot app by exposing them as beans with `@Bean` methods.
- A few Boot artifacts can be used on their own, such as the Spring Boot **Maven/Gradle plugins** (to build executable JARs or container images) or its dependency-version BOM.

But to use auto-configuration, Actuator or `@ConfigurationProperties`, the application itself must run on Spring.

</details>

<details><summary>

## [Medium] What does the exclude attribute of the @SpringBootApplication do?
</summary>

`exclude` turns off specific **auto-configuration classes** so Boot doesn't apply them, even if their conditions are met. It is the same as `@EnableAutoConfiguration(exclude = ...)`.

```java
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class, KafkaAutoConfiguration.class })
public class App { }
```

```yaml
spring:
  autoconfigure:
    exclude: org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

Typical reasons: you have a JDBC driver on the classpath but don't want a `DataSource` yet, you configure something yourself in a special way, or an auto-configuration conflicts with your setup. Use `excludeName` with a string when the class isn't on the classpath.

`exclude` works **only for auto-configuration classes**. Passing your own `@Configuration` class fails at startup ("The following classes could not be excluded because they are not auto-configuration classes"); to skip your own components, use `@ComponentScan(excludeFilters = ...)`, profiles or `@Conditional...` annotations.

</details>

<details><summary>

## [Medium] How can you create custom endpoints in Spring Boot Actuator?
</summary>

Create a bean annotated with **`@Endpoint(id = "...")`** and add operation methods:

- **`@ReadOperation`** → HTTP `GET`
- **`@WriteOperation`** → HTTP `POST`
- **`@DeleteOperation`** → HTTP `DELETE`

The endpoint appears at `/actuator/{id}`, respects the same exposure and security settings as built-in endpoints, and works over both HTTP and JMX. Use `@WebEndpoint` for HTTP-only or `@JmxEndpoint` for JMX-only endpoints, and `@Selector` on a parameter to take part of the path (`/actuator/features/{name}`).

Then expose it: `management.endpoints.web.exposure.include=health,features`.

To add information to existing endpoints instead of creating a new one, implement a **`HealthIndicator`** (custom health checks) or an **`InfoContributor`** (extra `/info` data). A plain `@RestController` would just be a normal API endpoint, outside Actuator's exposure and security model.

**Example:**

```java
@Component
@Endpoint(id = "features")
public class FeaturesEndpoint {
    private final Map<String, Boolean> flags = new ConcurrentHashMap<>(Map.of("newSearch", true));

    @ReadOperation                         // GET /actuator/features
    public Map<String, Boolean> all() { return flags; }

    @WriteOperation                        // POST /actuator/features
    public void set(String name, boolean enabled) { flags.put(name, enabled); }
}

// Custom health check:
@Component
class KafkaHealth implements HealthIndicator {
    public Health health() { return kafkaReachable() ? Health.up().build() : Health.down().withDetail("broker", "unreachable").build(); }
}
```

</details>
