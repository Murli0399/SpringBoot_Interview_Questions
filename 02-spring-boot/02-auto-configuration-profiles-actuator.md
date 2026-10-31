# Spring Boot: Auto-configuration, Profiles, Config and Observability

<details><summary>

## How does Spring Boot auto-configuration work?
</summary>

1. `@SpringBootApplication` includes `@EnableAutoConfiguration`.
2. Boot reads the list of auto-configuration classes from `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` in each starter JAR (Boot 2.7+; earlier `spring.factories`).
3. Each auto-config class is guarded by **conditions**, so it only applies when it makes sense:

```java
@AutoConfiguration
@ConditionalOnClass(DataSource.class)              // JDBC on the classpath?
@ConditionalOnProperty(prefix = "spring.datasource", name = "url")
public class MyDataSourceAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean                       // back off if the user defined one
    public DataSource dataSource(DataSourceProperties props) {
        return props.initializeDataSourceBuilder().build();
    }
}
```

Debug which configs applied with `--debug` (conditions evaluation report) or the `/actuator/conditions` endpoint. Exclude one with `@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)`.
</details>

<details><summary>

## What are Spring profiles and how do you use them?
</summary>

Profiles group configuration and beans per environment.

```yaml
# application.yml (shared)
server:
  port: 8080
---
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:mysql://prod-db:3306/app
```

```java
@Bean
@Profile("local")
public PaymentGateway fakeGateway() { return new FakePaymentGateway(); }
```

Activate with `SPRING_PROFILES_ACTIVE=prod`, `--spring.profiles.active=prod`, or `application-prod.yml` files.
</details>

<details><summary>

## What is the order of precedence for configuration properties?
</summary>

Highest wins (simplified):

1. Command-line arguments (`--server.port=9090`)
2. `SPRING_APPLICATION_JSON`
3. OS environment variables (`SERVER_PORT=9090`)
4. Profile-specific files outside the JAR, then inside (`application-prod.yml`)
5. `application.yml` / `.properties` outside the JAR, then inside
6. `@PropertySource`, then defaults

In Kubernetes, values usually come from ConfigMaps/Secrets as environment variables, overriding what's packaged in the JAR.
</details>

<details><summary>

## @Value vs @ConfigurationProperties?
</summary>

`@ConfigurationProperties` binds a group of properties to a typed, validated object. Prefer it over scattered `@Value`.

```yaml
app:
  kafka:
    topic: call-events
    batch-size: 500
    retry:
      max-attempts: 3
```

```java
@ConfigurationProperties(prefix = "app.kafka")
@Validated
public record KafkaProps(@NotBlank String topic, @Min(1) int batchSize, Retry retry) {
    public record Retry(int maxAttempts) {}
}

@SpringBootApplication
@ConfigurationPropertiesScan
public class App {}

// single value:
@Value("${app.kafka.topic:default-topic}") String topic;
```
</details>

<details><summary>

## How do you write a custom auto-configuration or starter?
</summary>

1. Create a module with an `@AutoConfiguration` class using `@ConditionalOn...` annotations and `@ConditionalOnMissingBean` on every bean so users can override.
2. Bind settings with `@ConfigurationProperties`.
3. Register it in `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`.
4. Publish as `yourlib-spring-boot-starter`.

Use case: a shared company library for tenant context, logging format or auth across many microservices.
</details>

<details><summary>

## Which Actuator endpoints matter in production and how do you secure them?
</summary>

- `/actuator/health` (with `liveness` and `readiness` groups for Kubernetes probes)
- `/actuator/metrics` and `/actuator/prometheus` (Micrometer metrics for Prometheus/Grafana)
- `/actuator/info`, `/actuator/loggers` (change log levels at runtime), `/actuator/env` (sensitive)

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  endpoint:
    health:
      probes:
        enabled: true
      show-details: when-authorized
```

Expose only what you need, serve it on a separate management port if possible, and protect it with Spring Security.
</details>

<details><summary>

## How do you add custom metrics with Micrometer?
</summary>

```java
@Service
public class CallPipeline {
    private final Counter processed;
    private final Timer latency;

    public CallPipeline(MeterRegistry registry) {
        processed = Counter.builder("calls.processed").tag("stage", "transcription").register(registry);
        latency = Timer.builder("calls.processing.latency").publishPercentiles(0.95, 0.99).register(registry);
    }

    public void process(Call call) {
        latency.record(() -> doWork(call));
        processed.increment();
    }
}
```

Prometheus scrapes `/actuator/prometheus`; Grafana dashboards and alerts are built on top (e.g. alert when p95 latency or Kafka consumer lag crosses a threshold). Keep tag cardinality low: never use user IDs as tags.
</details>

<details><summary>

## How do you do structured logging and tracing in Spring Boot?
</summary>

- Use SLF4J with parameterised messages: `log.info("Processed call {} in {} ms", callId, ms);`
- Put request/tenant/trace IDs in the **MDC** so every line carries them.
- Boot 3.4+ supports structured JSON logs (`logging.structured.format.console=ecs`); earlier, use a Logback JSON encoder.
- **Micrometer Tracing** (replaces Spring Cloud Sleuth in Boot 3) adds `traceId`/`spanId` and propagates them over HTTP and Kafka; export to Zipkin/Tempo/Jaeger via OpenTelemetry.
</details>

<details><summary>

## How do you shut down a Spring Boot app gracefully in Kubernetes?
</summary>

```yaml
server:
  shutdown: graceful
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

On SIGTERM, Boot stops accepting new requests, finishes in-flight ones, then closes beans (`@PreDestroy`, Kafka consumers commit offsets). Pair with a readiness probe that turns DOWN and a `terminationGracePeriodSeconds` longer than the shutdown timeout.
</details>

<details><summary>

## How do you make a Spring Boot app start faster or use less memory?
</summary>

- Lazy initialisation (`spring.main.lazy-initialization=true`) for dev; careful in prod (errors appear on first request).
- Remove unused starters and auto-configurations.
- Class Data Sharing (CDS) / Spring AOT; GraalVM native image for very fast startup.
- Right-size the JVM in containers: `-XX:MaxRAMPercentage=75`.
- Virtual threads (Boot 3.2+, Java 21) to handle more blocking requests with fewer platform threads.
</details>
