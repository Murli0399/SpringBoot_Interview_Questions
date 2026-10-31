# Spring & Spring Boot Interview Questions

Spring Framework and Spring Boot interview questions with answers and **code examples**: core concepts (IoC, DI, beans), Spring Boot internals, JPA and transactions, REST APIs, Spring Security (JWT, OAuth2, RBAC), microservices with Kafka, testing, and scenario-based questions for experienced backend engineers.

**212 questions** · every answer is collapsible: try to answer first, then click to check.

Companion repo for core Java, DSA and LLD: [Java_Interview_Question](https://github.com/Murli0399/Java_Interview_Question)

## How to use this repo

1. Read a question, answer it out loud, then expand to compare.
2. For experienced roles, spend the most time on `03-data-access/03-jpa-performance-and-transactions`, `05-security`, `06-microservices` and `08-scenario-based`; these are where interviews go deep.
3. For scenario questions, prepare one real story from your own work for each.

## Topics

### Spring Core

<details><summary><b><a href="01-spring-core/01-spring-basics.md">Spring basics</a></b> (12 questions)</summary>

- What are the major features of Spring 5?
- Is Spring 5 compatible with older versions of Java?
- What are the advantages of using the Spring framework?
- List the modules of the Spring framework.
- What is tight coupling?
- What is loose coupling?
- What is the purpose of the application.properties file?
- What is a pom.xml file?
- What design patterns are used in the Spring framework?
- What are some of the best practices for Spring Framework?
- Describe some standard Spring events.
- How does Spring 5 make use of JDK 9 modularity?

</details>

<details><summary><b><a href="01-spring-core/02-ioc-container.md">IoC container</a></b> (8 questions)</summary>

- What is Inversion of Control (IoC)?
- What are the advantages of Inversion of Control?
- What is the responsibility of an IoC container?
- Describe the two types of IoC container.
- What is the difference between BeanFactory and ApplicationContext?
- How is ApplicationContext configured in Spring?
- What is WebApplicationContext?
- What happens if the context is not closed?

</details>

<details><summary><b><a href="01-spring-core/03-dependency-injection.md">Dependency injection</a></b> (11 questions)</summary>

- What is dependency injection?
- How is dependency injection related to inversion of control?
- What are the types of dependency injection?
- What is constructor injection?
- How does setter injection work?
- Explain setter injection for objects and literal values.
- Explain the injection of Java Collection types.
- What is the difference between constructor and setter injection?
- Which dependency injection approach is better?
- What is method injection?
- What is a circular dependency and how should it be resolved?

</details>

<details><summary><b><a href="01-spring-core/04-beans-and-scopes.md">Beans and scopes</a></b> (21 questions)</summary>

- What are beans in Spring?
- What is the lifecycle of a Spring Bean?
- What are custom bean lifecycle methods?
- What are some features of custom init and destroy methods?
- What information does the bean definition contain?
- How can you provide a bean id when using annotations?
- Are Spring beans same as JavaBeans?
- How are beans created?
- What does the @Bean annotation do?
- Both @Bean and @Component annotations create beans. What is the difference between the two?
- How can dependencies be injected using the @Bean annotation?
- What are the different scopes of a bean?
- What is the default bean scope in Spring?
- What is the default scope in the web context?
- When are singleton and prototype scopes used?
- How is bean scope defined?
- Is Singleton scope in Spring same as the Singleton design pattern?
- Are Singleton beans thread safe?
- Explain prototype bean scope.
- Does Spring manage the complete lifecycle of beans?
- What is an inner bean?

</details>

<details><summary><b><a href="01-spring-core/05-stereotype-annotations.md">Stereotype annotations</a></b> (8 questions)</summary>

- What is the purpose of @Component annotation?
- What is the difference between @Component, @Service, @Repository, and @Controller?
- Why is @Primary annotation used?
- Why is @Qualifier annotation used?
- Which annotations takes precedence: @Primary or @Qualifier?
- Why is the @Required annotation used?
- What is the purpose of @Autowired annotation?
- What is the difference between @Inject and @Autowired in Spring? Which one to use under which condition?

</details>

<details><summary><b><a href="01-spring-core/06-configuration-and-component-scan.md">Configuration and component scan</a></b> (8 questions)</summary>

- What is a configuration file?
- How is configuration metadata provided to the Spring container/ how are beans configured in the Spring container?
- What is the difference between XML and Annotation configuration?
- How is annotation based configuration enabled in Spring?
- Can there be multiple configuration files in a project?
- What is component scan?
- How is component scan done in Spring Boot?
- What is the difference between context: annotation-config and context:component-scan tags?

</details>

<details><summary><b><a href="01-spring-core/07-autowiring.md">Autowiring</a></b> (9 questions)</summary>

- What is autowiring in Spring?
- What are the different modes of autowiring in Spring?
- How does autowiring internally work?
- What is autowiring by constructor?
- What are the limitations of autowiring?
- Is it possible to exclude a bean from being autowired?
- What does the @Autowired annotation do?
- What happens if we specify an interface instead of a class in getBean() method?
- Why do we need a no-arg constructor?

</details>

<details><summary><b><a href="01-spring-core/08-aop-caching-async-scheduling.md">AOP, caching, @Async, scheduling</a></b> (6 questions)</summary>

- What is AOP? Explain aspect, advice, pointcut and join point.
- How does Spring AOP work internally? JDK proxy vs CGLIB?
- How does caching work in Spring Boot?
- How do @Async methods work and what are the pitfalls?
- How do you schedule jobs, and what happens with multiple instances?
- How do Spring application events work?

</details>


### Spring Boot

<details><summary><b><a href="02-spring-boot/01-spring-boot-basics.md">Spring boot basics</a></b> (25 questions)</summary>

- What is Spring Boot?
- Why do we need Spring Boot?
- How can a Spring Boot application be created?
- What is Spring Initializr?
- What are the components of Spring Boot?
- What is the purpose of @SpringBootApplication annotation?
- What is the purpose of @RestController annotation in Spring Boot?
- What is the purpose of @ConditionalOnMissingBean annotation?
- What is an embedded server in Spring Boot?
- Can we change the default embedded server to Jetty?
- How are Spring Boot web applications deployed?
- What is Spring Boot Dependency Management?
- What happens when a web application is run using Spring Boot?
- How can the default port of the embedded tomcat server be changed?
- How can the default web server in a Spring Boot application be disabled?
- What are Spring Boot starter projects?
- What is spring-boot-starter-parent?
- Can Spring Boot be used for applications not built using the Spring framework?
- How can we connect Spring Boot with databases?
- What does the exclude attribute of the @SpringBootApplication do?
- How can external configuration be done in Spring Boot?
- What is Spring Actuator?
- How can the Actuator be accessed in Spring Boot?
- How can you create custom endpoints in Spring Boot Actuator?
- Explain Spring Boot DevTools.

</details>

<details><summary><b><a href="02-spring-boot/02-auto-configuration-profiles-actuator.md">Auto configuration profiles actuator</a></b> (10 questions)</summary>

- How does Spring Boot auto-configuration work?
- What are Spring profiles and how do you use them?
- What is the order of precedence for configuration properties?
- @Value vs @ConfigurationProperties?
- How do you write a custom auto-configuration or starter?
- Which Actuator endpoints matter in production and how do you secure them?
- How do you add custom metrics with Micrometer?
- How do you do structured logging and tracing in Spring Boot?
- How do you shut down a Spring Boot app gracefully in Kubernetes?
- How do you make a Spring Boot app start faster or use less memory?

</details>


### Data Access: JDBC, JPA and Transactions

<details><summary><b><a href="03-data-access/01-spring-jdbc.md">Spring JDBC</a></b> (14 questions)</summary>

- How does Spring provide DAO support?
- Describe Spring DAO support classes.
- What annotation is used to mark DAO in Spring?
- What is Spring Data JDBC?
- What tasks are performed by Spring JDBC?
- What is the difference between JDBC and Spring JDBC?
- Name the classes in Spring JDBC API?
- What are the advantages of JdbcTemplate in Spring?
- Why is NamedParameterJdbcTemplate class used?
- How are records from a database fetched when using JdbcTemplate?
- What is the difference between a RowMapper and ResultSetExtractor?
- Explain Spring’s exception handling support with DataAccessException.
- What is SQLExceptionTranslator?
- What are JdbcTemplate callback interfaces?

</details>

<details><summary><b><a href="03-data-access/02-spring-data-jpa-and-transactions.md">Spring Data JPA and transactions</a></b> (12 questions)</summary>

- What is Spring Data?
- What is JPA?
- What is the difference between JPA and Spring Data JPA?
- What is the difference between CrudRepository and JpaRepository?
- Which No SQL databases does Spring support?
- Which ORMs are supported by Spring?
- What is Hibernate?
- What are the different types of transaction management supported by Spring?
- Which transaction management type is preferred in Spring?
- What are some benefits of using Spring Transactions?
- What happens when the @Transactional annotation is used on a method?
- What is the difference between JdbcTemplate and JPA?

</details>

<details><summary><b><a href="03-data-access/03-jpa-performance-and-transactions.md">JPA performance and transactions</a></b> (10 questions)</summary>

- What is the N+1 query problem and how do you fix it?
- What is the difference between LAZY and EAGER fetching?
- Explain transaction propagation levels.
- Explain isolation levels in @Transactional.
- When does @Transactional not work?
- What is optimistic vs pessimistic locking in JPA?
- How do you paginate efficiently with Spring Data JPA?
- What is the difference between save(), saveAndFlush() and saveAll()? How do you do bulk inserts?
- What are entity states in JPA (persistence context)?
- What is the first-level and second-level cache in Hibernate?

</details>


### Spring MVC and REST APIs

<details><summary><b><a href="04-spring-mvc-rest/01-spring-mvc-and-maven.md">Spring MVC and Maven</a></b> (2 questions)</summary>

- Explain Spring MVC.
- What is maven?

</details>

<details><summary><b><a href="04-spring-mvc-rest/02-rest-apis-exception-handling-validation.md">REST APIs, exception handling, validation</a></b> (11 questions)</summary>

- What is the difference between @Controller and @RestController?
- What is the flow of a request through Spring MVC?
- What is the difference between @PathVariable, @RequestParam, @RequestBody and @RequestHeader?
- How do you handle exceptions globally in Spring Boot?
- How do you validate request data?
- What are good REST API design practices?
- What is the difference between PUT and PATCH? Which HTTP methods are idempotent?
- How do you make a POST API idempotent?
- What is the difference between a Filter and a HandlerInterceptor?
- RestTemplate vs WebClient vs RestClient vs Feign?
- How do you enable CORS in Spring Boot?

</details>


### Spring Security

<details><summary><b><a href="05-security/01-spring-security.md">Spring security</a></b> (11 questions)</summary>

- What is Spring Security?
- What is the difference between authentication and authorization?
- How does the Spring Security filter chain work?
- How does JWT authentication work? Show the filter.
- How do you handle JWT expiry, logout and revocation?
- Session-based vs token-based authentication?
- Explain OAuth2 and the Authorization Code flow with PKCE.
- How do you implement role-based access control (RBAC)?
- What is CSRF and when can you disable protection?
- How do you store passwords securely?
- How do you secure service-to-service calls?

</details>


### Microservices and Kafka

<details><summary><b><a href="06-microservices/01-microservices-kafka-resilience.md">Microservices, Kafka, resilience</a></b> (14 questions)</summary>

- Monolith vs microservices: when would you choose each?
- How do microservices communicate? Sync vs async?
- How do you produce and consume Kafka messages with Spring Boot?
- How does Kafka guarantee ordering, and how do you scale consumers without breaking it?
- Explain delivery semantics: at-most-once, at-least-once, exactly-once.
- How do you handle failures in Kafka consumers? (retries and DLQ)
- What causes consumer lag and how do you fix it?
- What is the Saga pattern? Orchestration vs choreography?
- What are CQRS and Event Sourcing?
- What is the transactional outbox pattern?
- What are circuit breakers, retries, timeouts and bulkheads? (Resilience4j)
- What are API gateway, service discovery and config server?
- How do you design a multi-tenant SaaS backend?
- How do you trace a request across microservices?

</details>


### Testing

<details><summary><b><a href="07-testing/01-testing-spring-boot.md">Testing spring boot</a></b> (6 questions)</summary>

- What is the difference between unit tests, slice tests and integration tests in Spring Boot?
- Write a unit test with Mockito.
- How do you test a REST controller with @WebMvcTest?
- How do you test repositories and use Testcontainers?
- @Mock vs @MockBean (@MockitoBean) vs @Spy?
- How do you test Kafka consumers?

</details>


### Scenario-Based Questions

<details><summary><b><a href="08-scenario-based/01-backend-scenario-questions.md">Backend scenario questions</a></b> (14 questions)</summary>

- How do you break a backend requirement into architecture, tasks and measurable milestones?
- Describe a distributed system you designed: failures, retries and consistency trade-offs.
- How would you consolidate many Kafka topics into a cleaner routing design?
- Walk through a design decision you made, with trade-offs.
- How do you write documentation so other engineers can contribute safely?
- How do you design schemas and queries for large-scale read/write workloads?
- When would you choose Elasticsearch vs MongoDB, and how do you manage search latency?
- Tell me about a performance bottleneck you found and how you measured the improvement.
- How do you deliver a backend feature end to end, from requirement to release?
- How do you build multi-tenant APIs with authentication and RBAC across services?
- How do you design Kafka consumer parallelism for throughput without breaking ordering?
- How do you ensure observability: metrics, logs, dashboards and alerts?
- Your API latency suddenly jumped from 100 ms to 2 s. How do you investigate?
- How would you design a web crawler that collects company data and makes it searchable?

</details>

## Quick answer: difference between Spring and Spring Boot

**Spring:** Spring Framework is the most popular application development framework of Java. The main feature of the Spring Framework is dependency Injection or Inversion of Control (IoC). With the help of Spring Framework, we can develop a loosely coupled application. It is better to use if application type or characteristics are purely defined.

**Spring Boot:** Spring Boot is a module of Spring Framework. It allows us to build a stand-alone application with minimal or zero configurations. It is better to use if we want to develop a simple Spring-based application or RESTful services.

|Spring|Spring Boot|
|------|-----------|
|Spring Framework is a widely used Java EE framework for building applications.|Spring Boot Framework is widely used to develop REST APIs.|
|It aims to simplify Java EE development and makes developers more productive.|It aims to shorten the code length and provide the easiest way to develop Web Applications.|
|The primary feature of the Spring Framework is dependency injection.|The primary feature of Spring Boot is Autoconfiguration. It automatically configures the classes based on the requirement.|
|It helps to make things simpler by allowing us to develop loosely coupled applications.|It helps to create a stand-alone application with less configuration.|
|The developer writes a lot of code (boilerplate code) to do the minimal task.|It reduces boilerplate code.|
|To test the Spring project, we need to set up the server explicitly.|Spring Boot offers embedded servers such as Jetty and Tomcat, etc.|
|Developers manually define dependencies for the Spring project in pom.xml.|Spring Boot comes with the concept of starter in pom.xml file that internally takes care of downloading the dependencies JARs based on Spring Boot Requirement.|

## Contributing

Found a mistake or want to add a question? Open an issue or a pull request, keeping the `<details>` format used in the files.

⭐ Star the repo if it helped you prepare.
