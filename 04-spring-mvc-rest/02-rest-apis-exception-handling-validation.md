# REST APIs, Exception Handling and Validation

<details><summary>

## What is the difference between @Controller and @RestController?
</summary>

`@RestController` = `@Controller` + `@ResponseBody` on every method. Return values are written straight to the response body (JSON via Jackson) instead of being treated as a view name.

```java
@RestController
@RequestMapping("/api/v1/companies")
public class CompanyController {

    private final CompanyService service;

    public CompanyController(CompanyService service) { this.service = service; }

    @GetMapping("/{id}")
    public CompanyDto get(@PathVariable Long id) {
        return service.findById(id);
    }

    @GetMapping
    public Page<CompanyDto> search(@RequestParam(required = false) String sector,
                                   @RequestParam(defaultValue = "0") int page,
                                   @RequestParam(defaultValue = "20") int size) {
        return service.search(sector, PageRequest.of(page, size, Sort.by("name")));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public CompanyDto create(@Valid @RequestBody CreateCompanyRequest req) {
        return service.create(req);
    }
}
```
</details>

<details><summary>

## What is the flow of a request through Spring MVC?
</summary>

1. Request hits the servlet **filters** (security, logging).
2. **DispatcherServlet** (front controller) receives it.
3. **HandlerMapping** finds the controller method from URL + HTTP method.
4. **HandlerInterceptors** `preHandle` run.
5. **HandlerAdapter** resolves arguments (`@PathVariable`, `@RequestBody` via `HttpMessageConverter`, validation) and invokes the method.
6. Return value is converted to JSON by an `HttpMessageConverter` (`@RestController`), or a **ViewResolver** renders a view (`@Controller`).
7. Exceptions go to **HandlerExceptionResolvers** (`@ControllerAdvice`).
8. Interceptors' `postHandle`/`afterCompletion` run, response goes back through filters.
</details>

<details><summary>

## What is the difference between @PathVariable, @RequestParam, @RequestBody and @RequestHeader?
</summary>

```java
// GET /users/42/orders?status=PAID   header X-Tenant-Id: acme
@GetMapping("/users/{userId}/orders")
public List<OrderDto> orders(@PathVariable Long userId,            // 42 - identifies the resource
                             @RequestParam String status,          // PAID - filter/sort/paging
                             @RequestHeader("X-Tenant-Id") String tenant) { ... }

@PostMapping("/orders")
public OrderDto create(@RequestBody CreateOrderRequest body) { ... } // JSON body -> object
```
</details>

<details><summary>

## How do you handle exceptions globally in Spring Boot?
</summary>

Use `@RestControllerAdvice` with `@ExceptionHandler` methods. Return a consistent error body; Spring 6 / Boot 3 provides the RFC 7807 `ProblemDetail` type.

```java
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String msg) { super(msg); }
}

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ProblemDetail notFound(ResourceNotFoundException ex) {
        return ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail validation(MethodArgumentNotValidException ex) {
        ProblemDetail pd = ProblemDetail.forStatus(HttpStatus.BAD_REQUEST);
        pd.setTitle("Validation failed");
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
          .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));
        pd.setProperty("errors", errors);
        return pd;
    }

    @ExceptionHandler(Exception.class)
    public ProblemDetail unexpected(Exception ex) {
        log.error("Unhandled error", ex);                 // log full stack trace server-side
        return ProblemDetail.forStatusAndDetail(HttpStatus.INTERNAL_SERVER_ERROR,
                "Something went wrong");                  // never leak internals to clients
    }
}
```

Response for a missing company:

```json
{ "type": "about:blank", "title": "Not Found", "status": 404, "detail": "Company 99 not found" }
```
</details>

<details><summary>

## How do you validate request data?
</summary>

Add `spring-boot-starter-validation`, annotate the DTO with Jakarta Bean Validation constraints, and put `@Valid` on the parameter. Failures throw `MethodArgumentNotValidException` (400).

```java
public record CreateCompanyRequest(
    @NotBlank @Size(max = 200) String name,
    @NotNull Sector sector,
    @Pattern(regexp = "^https?://.+") String website,
    @PositiveOrZero Long fundingUsd,
    @Email String contactEmail) {}

@PostMapping
public CompanyDto create(@Valid @RequestBody CreateCompanyRequest req) { ... }
```

- Validate path/query params by adding `@Validated` on the controller class: `@GetMapping("/{id}") get(@PathVariable @Min(1) Long id)`.
- Custom rule: create an annotation with `@Constraint(validatedBy = ...)` and a `ConstraintValidator`.
- Validation groups (`groups = OnCreate.class`) apply different rules for create vs update.
</details>

<details><summary>

## What are good REST API design practices?
</summary>

- **Nouns, plural, hierarchical:** `GET /companies/42/funding-rounds`, not `/getFundingRounds?id=42`.
- **Correct verbs:** GET (read, safe), POST (create), PUT (replace, idempotent), PATCH (partial update), DELETE (idempotent).
- **Status codes:** 200 OK, 201 Created (+ `Location` header), 204 No Content, 400 Bad Request, 401 Unauthenticated, 403 Forbidden, 404 Not Found, 409 Conflict, 422 Unprocessable, 429 Too Many Requests, 500/503.
- **Pagination** for lists (`?page=0&size=20` or cursor-based `?after=<id>` for large data).
- **Versioning:** `/api/v1/...` or a header.
- **Idempotency keys** for POSTs that create payments/orders.
- **Consistent error format** (`ProblemDetail`).
- Don't expose JPA entities directly; use DTOs.
</details>

<details><summary>

## What is the difference between PUT and PATCH? Which HTTP methods are idempotent?
</summary>

- **PUT** replaces the whole resource with the body; sending it twice leaves the same state → idempotent.
- **PATCH** changes only given fields; can be idempotent (`{"status":"ACTIVE"}`) or not (`{"op":"increment"}`).
- Idempotent: GET, PUT, DELETE, HEAD, OPTIONS. Not idempotent: POST (and PATCH by default).
- Safe (no state change): GET, HEAD, OPTIONS.
</details>

<details><summary>

## How do you make a POST API idempotent?
</summary>

The client sends an `Idempotency-Key` header. The server stores key → response; repeats return the stored response instead of creating a duplicate.

```java
@PostMapping("/payments")
public ResponseEntity<PaymentDto> pay(@RequestHeader("Idempotency-Key") String key,
                                      @Valid @RequestBody PaymentRequest req) {
    return idempotencyStore.find(key)                       // Redis or DB table with unique key
        .map(ResponseEntity::ok)
        .orElseGet(() -> {
            PaymentDto result = paymentService.charge(req);
            idempotencyStore.save(key, result, Duration.ofHours(24));
            return ResponseEntity.status(HttpStatus.CREATED).body(result);
        });
}
```

For concurrent duplicates, reserve the key atomically first (Redis `SET key PENDING NX EX 86400` or a unique DB constraint).
</details>

<details><summary>

## What is the difference between a Filter and a HandlerInterceptor?
</summary>

| | Filter | HandlerInterceptor |
|---|---|---|
| Level | Servlet container, before DispatcherServlet | Spring MVC, around controller methods |
| Knows the handler method | No | Yes |
| Typical use | Security, CORS, request logging, compression | Auditing, per-controller checks, timing |

```java
@Component
public class RequestIdFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res, FilterChain chain)
            throws ServletException, IOException {
        String id = Optional.ofNullable(req.getHeader("X-Request-Id")).orElse(UUID.randomUUID().toString());
        MDC.put("requestId", id);            // appears in every log line
        res.setHeader("X-Request-Id", id);
        try { chain.doFilter(req, res); } finally { MDC.clear(); }
    }
}
```
</details>

<details><summary>

## RestTemplate vs WebClient vs RestClient vs Feign?
</summary>

- **RestTemplate:** synchronous, in maintenance mode.
- **WebClient:** non-blocking, reactive; also usable blocking with `.block()`.
- **RestClient** (Spring 6.1 / Boot 3.2): modern synchronous fluent API; recommended over RestTemplate.
- **OpenFeign:** declarative interface-based clients, popular in Spring Cloud microservices.

```java
RestClient client = RestClient.builder().baseUrl("https://api.example.com").build();
CompanyDto c = client.get().uri("/companies/{id}", 42)
                     .retrieve()
                     .body(CompanyDto.class);
```

Always configure connect/read timeouts; the defaults can wait forever.
</details>

<details><summary>

## How do you enable CORS in Spring Boot?
</summary>

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
                .allowedOrigins("https://app.example.com")
                .allowedMethods("GET", "POST", "PUT", "DELETE")
                .allowCredentials(true);
    }
}
```

With Spring Security, also call `http.cors(Customizer.withDefaults())` so the security filter chain doesn't block preflight `OPTIONS` requests. Or use `@CrossOrigin` on a controller.
</details>
