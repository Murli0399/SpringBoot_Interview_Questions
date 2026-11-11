<details><summary>

## [Easy] When do you return ResponseEntity instead of a plain object?
</summary>

Return a **plain object** when the default `200 OK` with a JSON body is all you need. Return **`ResponseEntity<T>`** when you need control over the **status code, headers or an empty body**:

```java
@PostMapping
public ResponseEntity<CompanyDto> create(@Valid @RequestBody CreateCompanyRequest req) {
	CompanyDto created = service.create(req);
	URI location = URI.create("/api/v1/companies/" + created.id());
	return ResponseEntity.created(location).body(created);         // 201 + Location header
}

@GetMapping("/{id}")
public ResponseEntity<CompanyDto> get(@PathVariable Long id) {
	return service.find(id)
		.map(ResponseEntity::ok)                                   // 200
		.orElse(ResponseEntity.notFound().build());                // 404, no body
}

@DeleteMapping("/{id}")
public ResponseEntity<Void> delete(@PathVariable Long id) {
	service.delete(id);
	return ResponseEntity.noContent().build();                     // 204
}
```

Alternatives: `@ResponseStatus(HttpStatus.CREATED)` on the method for a fixed status, and throwing exceptions handled by `@RestControllerAdvice` for error statuses.
</details>

<details><summary>

## [Medium] What is the difference between Spring MVC and Spring WebFlux?
</summary>

| | Spring MVC | Spring WebFlux |
|---|---|---|
| Model | **Blocking**, one thread per request | **Non-blocking**, event loop with a few threads |
| Server | Servlet containers (Tomcat, Jetty) | Netty by default (also Servlet 3.1+ containers) |
| Return types | Objects, `ResponseEntity` | `Mono<T>` (0..1) and `Flux<T>` (0..N) from Project Reactor |
| Data access | JDBC, JPA (blocking) | Needs reactive drivers end to end: R2DBC, reactive MongoDB/Redis/Cassandra, `WebClient` |
| Programming style | Simple, imperative; easy to debug | Declarative pipelines; harder to debug and test |
| Back-pressure | No | Yes (Reactive Streams) |

**When WebFlux helps:** very high numbers of concurrent, mostly-waiting connections (streaming, server-sent events, API gateways like Spring Cloud Gateway), when the whole stack is non-blocking.

**When MVC is better:** typical CRUD/business services using JPA/JDBC. Since Java 21, **MVC + virtual threads** gives similar scalability for blocking IO with a much simpler programming model, so it's the default choice for most teams.

Never call blocking code (JDBC, `Thread.sleep`, blocking HTTP) on a WebFlux event-loop thread; it stalls every request handled by that thread.
</details>

<details><summary>

## [Medium] What are Mono and Flux?
</summary>

They are Project Reactor's publisher types used by WebFlux:

- **`Mono<T>`** – emits **0 or 1** item, then completes (or errors). Like an async `Optional`/`CompletableFuture`. Example: find a user by ID.
- **`Flux<T>`** – emits **0 to N** items, possibly infinite. Example: a list of orders, or a stream of events.

Nothing happens until someone **subscribes** (lazy, like Java streams). Operators: `map`, `flatMap` (async step), `filter`, `zip` (combine), `timeout`, `retryWhen`, `onErrorResume` (fallback).

```java
@GetMapping("/users/{id}/profile")
public Mono<Profile> profile(@PathVariable String id) {
	Mono<User> user = userClient.get().uri("/users/{id}", id).retrieve().bodyToMono(User.class);
	Mono<List<Order>> orders = orderClient.get().uri("/orders?user={id}", id)
		.retrieve().bodyToFlux(Order.class).collectList();
	return Mono.zip(user, orders, Profile::new)          // both calls run concurrently
		.timeout(Duration.ofSeconds(2))
		.onErrorResume(ex -> Mono.just(Profile.empty()));
}
```

In WebFlux, Spring subscribes for you when the controller returns a `Mono`/`Flux`.
</details>

<details><summary>

## [Medium] What are declarative HTTP interface clients?
</summary>

Since Spring 6 you can define an HTTP API as a Java **interface** annotated with `@HttpExchange` / `@GetExchange` / `@PostExchange`, and Spring generates the implementation on top of `RestClient` (blocking) or `WebClient` (reactive). It's Spring's built-in alternative to OpenFeign.

```java
@HttpExchange("/companies")
public interface CompanyClient {
	@GetExchange("/{id}")
	CompanyDto get(@PathVariable Long id);

	@PostExchange
	CompanyDto create(@RequestBody CreateCompanyRequest request);
}

@Configuration
class ClientConfig {
	@Bean
	CompanyClient companyClient(RestClient.Builder builder) {
		RestClient restClient = builder.baseUrl("https://api.example.com").build();
		HttpServiceProxyFactory factory = HttpServiceProxyFactory
			.builderFor(RestClientAdapter.create(restClient)).build();
		return factory.createClient(CompanyClient.class);
	}
}
```

Spring Framework 7 / Boot 4 add **HTTP service groups** (`@ImportHttpServices`), which register many such clients with shared configuration without writing a factory bean for each. Remember to configure **timeouts** on the underlying client.
</details>

<details><summary>

## [Medium] How do you version a REST API?
</summary>

Common strategies:

| Strategy | Example | Pros / cons |
|---|---|---|
| **URI path** | `/api/v1/orders`, `/api/v2/orders` | Most visible and cache-friendly; URLs change between versions |
| **Request header** | `API-Version: 2` | Clean URLs; less visible, needs client support |
| **Query parameter** | `/orders?version=2` | Easy to try; mixes versioning with filters |
| **Media type** | `Accept: application/vnd.company.v2+json` | Most "RESTful"; hardest for clients |

**Spring Framework 7** supports versioning directly: configure how the version is read, then declare it on mappings.

```java
@Configuration
class WebConfig implements WebMvcConfigurer {
	@Override
	public void configureApiVersioning(ApiVersionConfigurer configurer) {
		configurer.useRequestHeader("API-Version");      // or usePathSegment(...), useQueryParam(...)
	}
}

@RestController
@RequestMapping("/orders")
class OrderController {
	@GetMapping(path = "/{id}", version = "1.0")
	OrderV1 getV1(@PathVariable Long id) { ... }

	@GetMapping(path = "/{id}", version = "2.0")
	OrderV2 getV2(@PathVariable Long id) { ... }
}
```

Good practice regardless of mechanism: make only **backward-compatible changes** within a version (add optional fields, never remove or rename), publish a **deprecation** timeline (Spring 7 can add `Deprecation`/`Sunset` headers), and keep at most two versions alive.
</details>

<details><summary>

## [Medium] What is content negotiation in Spring MVC?
</summary>

**Content negotiation** decides which format to return (JSON, XML, CSV…) based on what the client asks for, usually the **`Accept`** header.

- Spring checks the `Accept` header against the media types the handler can **produce** and the available **`HttpMessageConverter`s** (Jackson for JSON, Jackson XML if on the classpath, String, byte arrays…).
- Restrict a handler with `@GetMapping(produces = MediaType.APPLICATION_JSON_VALUE)`; if the client accepts only something else, Spring returns **406 Not Acceptable**.
- For request bodies, `consumes` and the **`Content-Type`** header pick the converter; unsupported types get **415 Unsupported Media Type**.

```java
@GetMapping(value = "/{id}", produces = { MediaType.APPLICATION_JSON_VALUE, MediaType.APPLICATION_XML_VALUE })
public CompanyDto get(@PathVariable Long id) { return service.findById(id); }
```

Custom formats (e.g. CSV) are supported by registering your own `HttpMessageConverter`.
</details>
