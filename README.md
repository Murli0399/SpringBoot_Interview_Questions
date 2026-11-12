# Spring & Spring Boot Interview Questions

Spring Framework and Spring Boot interview questions with answers and **code examples**: IoC, DI and beans, Spring Boot internals and what's new in Boot 3 and 4, JPA and transactions, REST and WebFlux, Spring Security (JWT, OAuth2, RBAC), microservices with Kafka, testing, and scenario-based questions for experienced backend engineers.

**242 questions** · 97 Easy · 105 Medium · 40 Hard · every answer is collapsible, so try to answer first, then click to check.

Answers target **Spring Framework 6/7 and Spring Boot 3/4** (Java 17+, `jakarta.*` packages), with notes where older versions differ.

Companion repo for core Java, DSA and LLD: [Java_Interview_Question](https://github.com/Murli0399/Java_Interview_Question)

## Difficulty levels

Folders and the files inside them are numbered from basic to advanced, so read them in order. Every question is tagged by difficulty, and within each file questions run from **Easy → Medium → Hard** (numbered DSA problems keep their order):

| Level | Meaning |
|---|---|
| **[Easy]** | Fundamentals every Java developer is expected to answer (freshers, screening rounds) |
| **[Medium]** | Follow-ups and practical usage (1–3 years of experience) |
| **[Hard]** | Internals, production troubleshooting and design trade-offs (3+ years, senior rounds) |

## How to use this repo

1. Read a question, answer it out loud, then expand to compare.
2. For experienced roles, spend the most time on the **[Hard]** questions in `03-data-access`, `05-security`, `06-microservices` and `08-scenario-based`; that's where interviews go deep.
3. For scenario questions, prepare one real story from your own work for each.

## Suggested study path

| Stage | Focus |
|---|---|
| 1. Foundations | **[Easy]** questions in `01-spring-core` and `02-spring-boot` |
| 2. Building services | `03-data-access`, `04-spring-mvc-rest`, `07-testing` (**[Medium]**) |
| 3. Production and design | **[Hard]** questions: transactions and JPA performance, security, Kafka and resilience, scenario questions |

## Topics

### Spring Core

<details><summary><b><a href="01-spring-core/01-spring-basics.md">Spring basics</a></b> (12: 7 Easy · 4 Medium · 1 Hard)</summary>

- [Easy] Which Java versions do Spring 5, 6 and 7 require?
- [Easy] What are the advantages of using the Spring framework?
- [Easy] List the modules of the Spring framework.
- [Easy] What is tight coupling?
- [Easy] What is loose coupling?
- [Easy] What is the purpose of the application.properties file?
- [Easy] What is a pom.xml file?
- [Medium] What were the major features of Spring 5?
- [Medium] What design patterns are used in the Spring framework?
- [Medium] What are some of the best practices for Spring Framework?
- [Medium] Describe some standard Spring events.
- [Hard] How does Spring work with Java modules (JPMS)?

</details>

<details><summary><b><a href="01-spring-core/02-ioc-container.md">IoC container</a></b> (8: 3 Easy · 5 Medium)</summary>

- [Easy] What is Inversion of Control (IoC)?
- [Easy] What are the advantages of Inversion of Control?
- [Easy] What is the responsibility of an IoC container?
- [Medium] Describe the two types of IoC container.
- [Medium] What is the difference between BeanFactory and ApplicationContext?
- [Medium] How is ApplicationContext configured in Spring?
- [Medium] What is WebApplicationContext?
- [Medium] What happens if the context is not closed?

</details>

<details><summary><b><a href="01-spring-core/03-dependency-injection.md">Dependency injection</a></b> (11: 6 Easy · 4 Medium · 1 Hard)</summary>

- [Easy] What is dependency injection?
- [Easy] How is dependency injection related to inversion of control?
- [Easy] What are the types of dependency injection?
- [Easy] What is constructor injection?
- [Easy] How does setter injection work?
- [Easy] What is the difference between constructor and setter injection?
- [Medium] Explain setter injection for objects and literal values.
- [Medium] Explain the injection of Java Collection types.
- [Medium] Which dependency injection approach is better?
- [Medium] What is a circular dependency and how should it be resolved?
- [Hard] What is method injection?

</details>

<details><summary><b><a href="01-spring-core/04-beans-and-scopes.md">Beans and scopes</a></b> (21: 11 Easy · 9 Medium · 1 Hard)</summary>

- [Easy] What are beans in Spring?
- [Easy] What are some features of custom init and destroy methods?
- [Easy] How can you provide a bean id when using annotations?
- [Easy] Are Spring beans same as JavaBeans?
- [Easy] What does the @Bean annotation do?
- [Easy] Both @Bean and @Component annotations create beans. What is the difference between the two?
- [Easy] How can dependencies be injected using the @Bean annotation?
- [Easy] What are the different scopes of a bean?
- [Easy] What is the default bean scope in Spring?
- [Easy] What is the default scope in the web context?
- [Easy] How is bean scope defined?
- [Medium] What is the lifecycle of a Spring Bean?
- [Medium] What are custom bean lifecycle methods?
- [Medium] How are beans created?
- [Medium] When are singleton and prototype scopes used?
- [Medium] Is Singleton scope in Spring same as the Singleton design pattern?
- [Medium] Are Singleton beans thread safe?
- [Medium] Explain prototype bean scope.
- [Medium] Does Spring manage the complete lifecycle of beans?
- [Medium] What is an inner bean?
- [Hard] What information does the bean definition contain?

</details>

<details><summary><b><a href="01-spring-core/05-stereotype-annotations.md">Stereotype annotations</a></b> (8: 6 Easy · 2 Medium)</summary>

- [Easy] What is the purpose of @Component annotation?
- [Easy] What is the difference between @Component, @Service, @Repository, and @Controller?
- [Easy] Why is @Primary annotation used?
- [Easy] Why is @Qualifier annotation used?
- [Easy] Why is the @Required annotation used?
- [Easy] What is the purpose of @Autowired annotation?
- [Medium] Which annotations takes precedence: @Primary or @Qualifier?
- [Medium] What is the difference between @Inject and @Autowired in Spring? Which one to use under which condition?

</details>

<details><summary><b><a href="01-spring-core/06-configuration-and-component-scan.md">Configuration and component scan</a></b> (8: 7 Easy · 1 Medium)</summary>

- [Easy] What is a configuration file?
- [Easy] How is configuration metadata provided to the Spring container/ how are beans configured in the Spring container?
- [Easy] What is the difference between XML and Annotation configuration?
- [Easy] How is annotation based configuration enabled in Spring?
- [Easy] Can there be multiple configuration files in a project?
- [Easy] What is component scan?
- [Easy] How is component scan done in Spring Boot?
- [Medium] What is the difference between context: annotation-config and context:component-scan tags?

</details>

<details><summary><b><a href="01-spring-core/07-autowiring.md">Autowiring</a></b> (9: 4 Easy · 5 Medium)</summary>

- [Easy] What is autowiring in Spring?
- [Easy] What is autowiring by constructor?
- [Easy] What does the @Autowired annotation do?
- [Easy] What happens if we specify an interface instead of a class in getBean() method?
- [Medium] What are the different modes of autowiring in Spring?
- [Medium] How does autowiring internally work?
- [Medium] What are the limitations of autowiring?
- [Medium] Is it possible to exclude a bean from being autowired?
- [Medium] Why do we need a no-arg constructor?

</details>

<details><summary><b><a href="01-spring-core/08-aop-caching-async-scheduling.md">AOP, caching, @Async, scheduling</a></b> (6: 5 Medium · 1 Hard)</summary>

- [Medium] What is AOP? Explain aspect, advice, pointcut and join point.
- [Medium] How does caching work in Spring Boot?
- [Medium] How do @Async methods work and what are the pitfalls?
- [Medium] How do you schedule jobs, and what happens with multiple instances?
- [Medium] How do Spring application events work?
- [Hard] How does Spring AOP work internally? JDK proxy vs CGLIB?

</details>

<details><summary><b><a href="01-spring-core/09-advanced-container-features.md">Advanced container features</a></b> (8: 6 Medium · 2 Hard)</summary>

- [Medium] What do @Lazy and @DependsOn do?
- [Medium] What are @Conditional annotations?
- [Medium] What is the purpose of @Import?
- [Medium] How do you inject a prototype bean into a singleton bean correctly?
- [Medium] What are Aware interfaces?
- [Medium] How does Spring resolve @Value and property placeholders?
- [Hard] What is the difference between BeanPostProcessor and BeanFactoryPostProcessor?
- [Hard] What is a FactoryBean?

</details>


### Spring Boot

<details><summary><b><a href="02-spring-boot/01-spring-boot-basics.md">Spring Boot basics</a></b> (25: 20 Easy · 5 Medium)</summary>

- [Easy] What is Spring Boot?
- [Easy] Why do we need Spring Boot?
- [Easy] How can a Spring Boot application be created?
- [Easy] What is Spring Initializr?
- [Easy] What are the components of Spring Boot?
- [Easy] What is the purpose of @SpringBootApplication annotation?
- [Easy] What is the purpose of @RestController annotation in Spring Boot?
- [Easy] What is an embedded server in Spring Boot?
- [Easy] Can we change the default embedded server to Jetty?
- [Easy] How are Spring Boot web applications deployed?
- [Easy] What is Spring Boot Dependency Management?
- [Easy] How can the default port of the embedded tomcat server be changed?
- [Easy] How can the default web server in a Spring Boot application be disabled?
- [Easy] What are Spring Boot starter projects?
- [Easy] What is spring-boot-starter-parent?
- [Easy] How can we connect Spring Boot with databases?
- [Easy] How can external configuration be done in Spring Boot?
- [Easy] What is Spring Actuator?
- [Easy] How can the Actuator be accessed in Spring Boot?
- [Easy] Explain Spring Boot DevTools.
- [Medium] What is the purpose of @ConditionalOnMissingBean annotation?
- [Medium] What happens when a web application is run using Spring Boot?
- [Medium] Can Spring Boot be used for applications not built using the Spring framework?
- [Medium] What does the exclude attribute of the @SpringBootApplication do?
- [Medium] How can you create custom endpoints in Spring Boot Actuator?

</details>

<details><summary><b><a href="02-spring-boot/02-auto-configuration-profiles-actuator.md">Auto-configuration, profiles, Actuator</a></b> (10: 1 Easy · 6 Medium · 3 Hard)</summary>

- [Easy] What are Spring profiles and how do you use them?
- [Medium] What is the order of precedence for configuration properties?
- [Medium] @Value vs @ConfigurationProperties?
- [Medium] Which Actuator endpoints matter in production and how do you secure them?
- [Medium] How do you add custom metrics with Micrometer?
- [Medium] How do you do structured logging and tracing in Spring Boot?
- [Medium] How do you shut down a Spring Boot app gracefully in Kubernetes?
- [Hard] How does Spring Boot auto-configuration work?
- [Hard] How do you write a custom auto-configuration or starter?
- [Hard] How do you make a Spring Boot app start faster or use less memory?

</details>

<details><summary><b><a href="02-spring-boot/03-spring-boot-3-and-4.md">What's new in Boot 3 and 4</a></b> (6: 1 Easy · 3 Medium · 2 Hard)</summary>

- [Easy] What is the difference between CommandLineRunner and ApplicationRunner?
- [Medium] What changed in Spring Framework 6 and Spring Boot 3?
- [Medium] What changed in Spring Framework 7 and Spring Boot 4?
- [Medium] What are virtual threads and how do you enable them in Spring Boot?
- [Hard] How do you migrate from Spring Boot 2 to Spring Boot 3?
- [Hard] What is a GraalVM native image and what are its trade-offs?

</details>


### Data Access: JDBC, JPA and Transactions

<details><summary><b><a href="03-data-access/01-spring-jdbc.md">Spring JDBC</a></b> (14: 7 Easy · 7 Medium)</summary>

- [Easy] How does Spring provide DAO support?
- [Easy] What annotation is used to mark DAO in Spring?
- [Easy] What tasks are performed by Spring JDBC?
- [Easy] What is the difference between JDBC and Spring JDBC?
- [Easy] What are the advantages of JdbcTemplate in Spring?
- [Easy] Why is NamedParameterJdbcTemplate class used?
- [Easy] How are records from a database fetched when using JdbcTemplate?
- [Medium] Describe Spring DAO support classes.
- [Medium] What is Spring Data JDBC?
- [Medium] Name the classes in Spring JDBC API?
- [Medium] What is the difference between a RowMapper and ResultSetExtractor?
- [Medium] Explain Spring’s exception handling support with DataAccessException.
- [Medium] What is SQLExceptionTranslator?
- [Medium] What are JdbcTemplate callback interfaces?

</details>

<details><summary><b><a href="03-data-access/02-spring-data-jpa-and-transactions.md">Spring Data JPA and transactions</a></b> (12: 8 Easy · 4 Medium)</summary>

- [Easy] What is Spring Data?
- [Easy] What is JPA?
- [Easy] What is the difference between JPA and Spring Data JPA?
- [Easy] What is the difference between CrudRepository and JpaRepository?
- [Easy] Which No SQL databases does Spring support?
- [Easy] Which ORMs are supported by Spring?
- [Easy] What is Hibernate?
- [Easy] What are some benefits of using Spring Transactions?
- [Medium] What are the different types of transaction management supported by Spring?
- [Medium] Which transaction management type is preferred in Spring?
- [Medium] What happens when the @Transactional annotation is used on a method?
- [Medium] What is the difference between JdbcTemplate and JPA?

</details>

<details><summary><b><a href="03-data-access/03-jpa-mappings-and-queries.md">JPA mappings and queries</a></b> (10: 8 Medium · 2 Hard)</summary>

- [Medium] How do you map one-to-many and many-to-one relationships in JPA?
- [Medium] What are cascade types and orphanRemoval?
- [Medium] How do you write custom queries in Spring Data JPA?
- [Medium] What are projections in Spring Data JPA?
- [Medium] What is the difference between persist(), merge(), and Spring Data's save()?
- [Medium] What is the difference between findById() and getReferenceById()?
- [Medium] How do you implement auditing (createdAt, updatedBy) in Spring Data JPA?
- [Medium] How should you manage database schema changes in Spring Boot?
- [Hard] How do you build dynamic search filters with Specifications?
- [Hard] How should you implement equals() and hashCode() for JPA entities?

</details>

<details><summary><b><a href="03-data-access/04-jpa-performance-and-transactions.md">JPA performance and transactions</a></b> (10: 5 Medium · 5 Hard)</summary>

- [Medium] What is the difference between LAZY and EAGER fetching?
- [Medium] How do you paginate efficiently with Spring Data JPA?
- [Medium] What is the difference between save(), saveAndFlush() and saveAll()? How do you do bulk inserts?
- [Medium] What are entity states in JPA (persistence context)?
- [Medium] What is the first-level and second-level cache in Hibernate?
- [Hard] What is the N+1 query problem and how do you fix it?
- [Hard] Explain transaction propagation levels.
- [Hard] Explain isolation levels in @Transactional.
- [Hard] When does @Transactional not work?
- [Hard] What is optimistic vs pessimistic locking in JPA?

</details>


### Spring MVC, REST and WebFlux

<details><summary><b><a href="04-spring-mvc-rest/01-spring-mvc-and-maven.md">Spring MVC and Maven</a></b> (2: 2 Easy)</summary>

- [Easy] Explain Spring MVC.
- [Easy] What is maven?

</details>

<details><summary><b><a href="04-spring-mvc-rest/02-rest-apis-exception-handling-validation.md">REST APIs, exceptions, validation</a></b> (11: 5 Easy · 5 Medium · 1 Hard)</summary>

- [Easy] What is the difference between @Controller and @RestController?
- [Easy] What is the difference between @PathVariable, @RequestParam, @RequestBody and @RequestHeader?
- [Easy] How do you validate request data?
- [Easy] What is the difference between PUT and PATCH? Which HTTP methods are idempotent?
- [Easy] How do you enable CORS in Spring Boot?
- [Medium] What is the flow of a request through Spring MVC?
- [Medium] How do you handle exceptions globally in Spring Boot?
- [Medium] What are good REST API design practices?
- [Medium] What is the difference between a Filter and a HandlerInterceptor?
- [Medium] RestTemplate vs WebClient vs RestClient vs Feign?
- [Hard] How do you make a POST API idempotent?

</details>

<details><summary><b><a href="04-spring-mvc-rest/03-webflux-http-clients-api-versioning.md">WebFlux, HTTP clients, API versioning</a></b> (6: 1 Easy · 5 Medium)</summary>

- [Easy] When do you return ResponseEntity instead of a plain object?
- [Medium] What is the difference between Spring MVC and Spring WebFlux?
- [Medium] What are Mono and Flux?
- [Medium] What are declarative HTTP interface clients?
- [Medium] How do you version a REST API?
- [Medium] What is content negotiation in Spring MVC?

</details>


### Spring Security

<details><summary><b><a href="05-security/01-spring-security.md">Spring Security (JWT, OAuth2, RBAC)</a></b> (11: 3 Easy · 5 Medium · 3 Hard)</summary>

- [Easy] What is Spring Security?
- [Easy] What is the difference between authentication and authorization?
- [Easy] How do you store passwords securely?
- [Medium] How does the Spring Security filter chain work?
- [Medium] How does JWT authentication work? Show the filter.
- [Medium] Session-based vs token-based authentication?
- [Medium] How do you implement role-based access control (RBAC)?
- [Medium] What is CSRF and when can you disable protection?
- [Hard] How do you handle JWT expiry, logout and revocation?
- [Hard] Explain OAuth2 and the Authorization Code flow with PKCE.
- [Hard] How do you secure service-to-service calls?

</details>


### Microservices and Kafka

<details><summary><b><a href="06-microservices/01-microservices-kafka-resilience.md">Microservices, Kafka, resilience</a></b> (14: 2 Easy · 5 Medium · 7 Hard)</summary>

- [Easy] How do microservices communicate? Sync vs async?
- [Easy] What are API gateway, service discovery and config server?
- [Medium] Monolith vs microservices: when would you choose each?
- [Medium] How do you produce and consume Kafka messages with Spring Boot?
- [Medium] What causes consumer lag and how do you fix it?
- [Medium] What are circuit breakers, retries, timeouts and bulkheads? (Resilience4j)
- [Medium] How do you trace a request across microservices?
- [Hard] How does Kafka guarantee ordering, and how do you scale consumers without breaking it?
- [Hard] Explain delivery semantics: at-most-once, at-least-once, exactly-once.
- [Hard] How do you handle failures in Kafka consumers? (retries and DLQ)
- [Hard] What is the Saga pattern? Orchestration vs choreography?
- [Hard] What are CQRS and Event Sourcing?
- [Hard] What is the transactional outbox pattern?
- [Hard] How do you design a multi-tenant SaaS backend?

</details>


### Testing

<details><summary><b><a href="07-testing/01-testing-spring-boot.md">Testing</a></b> (6: 3 Easy · 2 Medium · 1 Hard)</summary>

- [Easy] What is the difference between unit tests, slice tests and integration tests in Spring Boot?
- [Easy] Write a unit test with Mockito.
- [Easy] @Mock vs @MockBean (@MockitoBean) vs @Spy?
- [Medium] How do you test a REST controller with @WebMvcTest?
- [Medium] How do you test repositories and use Testcontainers?
- [Hard] How do you test Kafka consumers?

</details>


### Scenario-Based Questions

<details><summary><b><a href="08-scenario-based/01-backend-scenario-questions.md">Scenario-based questions</a></b> (14: 4 Medium · 10 Hard)</summary>

- [Medium] Walk through a design decision you made, with trade-offs.
- [Medium] How do you write documentation so other engineers can contribute safely?
- [Medium] How do you deliver a backend feature end to end, from requirement to release?
- [Medium] How do you ensure observability: metrics, logs, dashboards and alerts?
- [Hard] How do you break a backend requirement into architecture, tasks and measurable milestones?
- [Hard] Describe a distributed system you designed: failures, retries and consistency trade-offs.
- [Hard] How would you consolidate many Kafka topics into a cleaner routing design?
- [Hard] How do you design schemas and queries for large-scale read/write workloads?
- [Hard] When would you choose Elasticsearch vs MongoDB, and how do you manage search latency?
- [Hard] Tell me about a performance bottleneck you found and how you measured the improvement.
- [Hard] How do you build multi-tenant APIs with authentication and RBAC across services?
- [Hard] How do you design Kafka consumer parallelism for throughput without breaking ordering?
- [Hard] Your API latency suddenly jumped from 100 ms to 2 s. How do you investigate?
- [Hard] How would you design a web crawler that collects company data and makes it searchable?

</details>

## Contributing

Found a mistake or want to add a question? Open an issue or a pull request, keeping the `<details>` format and a difficulty tag (`[Easy]`, `[Medium]` or `[Hard]`) used in the files.

⭐ Star the repo if it helped you prepare.
