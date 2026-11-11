# What's New: Spring Boot 3 / Spring 6 and Spring Boot 4 / Spring 7

<details><summary>

## [Easy] What is the difference between CommandLineRunner and ApplicationRunner?
</summary>

Both are beans whose `run` method executes **once, right after the application context has started** and before the app is marked ready. Use them for startup tasks: seeding data, warming caches, validating configuration, or running a one-off job.

| | `CommandLineRunner` | `ApplicationRunner` |
|---|---|---|
| Method | `run(String... args)` | `run(ApplicationArguments args)` |
| Arguments | Raw strings | Parsed: `getOptionNames()`, `getOptionValues("env")`, `getNonOptionArgs()` |

```java
@Component
@Order(1)                                 // ordering when there are several runners
class CacheWarmup implements ApplicationRunner {
	@Override
	public void run(ApplicationArguments args) {
		if (args.containsOption("skip-warmup")) return;   // --skip-warmup
		// load reference data...
	}
}
```

An exception thrown from a runner **fails the application startup**. For work that shouldn't block startup, listen to `ApplicationReadyEvent` and run it asynchronously.
</details>

<details><summary>

## [Medium] What changed in Spring Framework 6 and Spring Boot 3?
</summary>

Spring Framework 6 and Spring Boot 3 were released in **November 2022**.

- **Java 17 minimum.** Records, sealed classes, text blocks and pattern matching can be used throughout.
- **Jakarta EE 9/10:** all `javax.*` enterprise packages moved to **`jakarta.*`** – `jakarta.persistence`, `jakarta.servlet`, `jakarta.validation`, `jakarta.annotation`.
- **AOT processing and GraalVM native images:** Spring generates bean wiring at build time, so apps can compile to native executables with fast startup and low memory.
- **Observability:** the Micrometer **Observation API** and **Micrometer Tracing** (replacing Spring Cloud Sleuth), exported via OpenTelemetry or Zipkin.
- **`ProblemDetail`** (RFC 7807) error responses built into Spring MVC/WebFlux.
- **HTTP interface clients:** declare an interface with `@GetExchange`/`@PostExchange` and Spring generates the HTTP client.
- **Spring Security 6:** lambda-based DSL, `SecurityFilterChain` beans (no `WebSecurityConfigurerAdapter`).

Later 3.x releases added: Docker Compose support and **Testcontainers `@ServiceConnection`** (3.1), **virtual threads**, **`RestClient`** and **`JdbcClient`** (3.2), CDS support (3.3), **structured logging** and `@MockitoBean` (3.4).
</details>

<details><summary>

## [Medium] What changed in Spring Framework 7 and Spring Boot 4?
</summary>

Spring Framework 7 and Spring Boot 4 were released in **November 2025**.

- **Baseline:** still **Java 17**, with first-class support for **Java 25**; **Jakarta EE 11** (Servlet 6.1, Persistence 3.2, Validation 3.1).
- **Null safety with JSpecify:** standard `@Nullable`/`@NonNull` annotations across the Spring portfolio, understood by IDEs, NullAway and Kotlin.
- **API versioning built in:** Spring MVC and WebFlux can route requests by version (path, header, query parameter or media type) and mark deprecated versions.
- **HTTP interface client groups:** easier configuration of many declarative HTTP clients (`@ImportHttpServices`).
- **Resilience in core:** `@Retryable` (retry with backoff and jitter) and `@ConcurrencyLimit` moved into Spring Framework, enabled with `@EnableResilientMethods`.
- **Modular Spring Boot:** the single large auto-configuration JAR was split into technology-specific modules and starters (e.g. `spring-boot-starter-webmvc`), so apps pull in less.
- **Jackson 3** is the default JSON library (package `tools.jackson` instead of `com.fasterxml.jackson`).
- **Removals:** support for `javax.annotation` / `javax.inject` annotations, APIs deprecated in Boot 3.x, and **Undertow** (not yet compatible with Servlet 6.1). `RestTemplate` is planned for deprecation in 7.1; use `RestClient`.
</details>

<details><summary>

## [Medium] What are virtual threads and how do you enable them in Spring Boot?
</summary>

**Virtual threads** (Java 21) are lightweight threads managed by the JVM. When one blocks on IO (a DB query, an HTTP call), the JVM parks it and frees the underlying OS (carrier) thread for other work. You can have millions of them, so the simple **thread-per-request blocking style scales** almost like reactive code.

In Spring Boot 3.2+ on Java 21:

```yaml
spring:
  threads:
    virtual:
      enabled: true
```

This makes Tomcat/Jetty handle each request on a virtual thread, and configures `@Async`, `@Scheduled` task executors and other Boot-managed executors to use virtual threads.

Caveats:
- No benefit for **CPU-bound** work.
- Don't **pool** virtual threads; and limit concurrency to fragile downstreams some other way (a `Semaphore`, bulkheads, connection pool size) because they no longer limit it for you.
- Long blocking inside `synchronized` could **pin** the carrier thread on older JDKs (much improved in Java 24+); `ReentrantLock` avoids it.
- `ThreadLocal`-heavy code works but can use a lot of memory with millions of threads.
</details>

<details><summary>

## [Hard] How do you migrate from Spring Boot 2 to Spring Boot 3?
</summary>

1. **Upgrade to Java 17+** first.
2. **Move to the latest Boot 2.7** and fix all deprecation warnings (removed APIs are the main breakage).
3. **Replace `javax.*` with `jakarta.*`** imports: persistence, validation, servlet, annotation, transaction. Tools like OpenRewrite automate this.
4. **Update dependencies** to Jakarta-compatible versions (Hibernate 6, Tomcat 10, libraries that use servlet/persistence APIs).
5. **Spring Security 6:** replace `WebSecurityConfigurerAdapter` with `SecurityFilterChain` beans; `authorizeRequests` → `authorizeHttpRequests`, `antMatchers` → `requestMatchers`.
6. **Configuration properties** that were renamed (use `spring-boot-properties-migrator` temporarily to log them).
7. **Tracing:** replace Spring Cloud Sleuth with Micrometer Tracing.
8. **Hibernate 6** changes: some query and ID-generation behaviour differs; run the full test suite against a real database (Testcontainers).

The same step-by-step approach applies to Boot 3 → 4: get to the latest 3.5, remove deprecated usage, then upgrade (watch Jackson 3 packages and the modular starters).
</details>

<details><summary>

## [Hard] What is a GraalVM native image and what are its trade-offs?
</summary>

A **native image** compiles a Spring Boot application **ahead of time** into a standalone native executable (no JVM needed at runtime). Spring Boot 3+ supports it through **AOT processing**, which generates bean definitions and reflection/proxy hints at build time.

```bash
./mvnw -Pnative native:compile          # needs GraalVM
./mvnw -Pnative spring-boot:build-image # builds a container image with the native binary
```

| Benefit | Cost |
|---|---|
| Startup in tens of milliseconds | Much longer build times (minutes) |
| Lower memory footprint | Peak throughput often lower than a warmed-up JIT-compiled JVM |
| Good for serverless, scale-to-zero, CLI tools | "Closed world": dynamic class loading and some reflection need hints; some libraries aren't compatible |
| Smaller containers | Harder debugging and profiling |

For long-running, high-throughput services the regular JVM (optionally with CDS/AOT cache for faster startup) is often still the better choice.
</details>
