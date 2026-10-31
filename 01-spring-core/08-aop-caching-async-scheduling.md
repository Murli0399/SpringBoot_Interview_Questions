# AOP, Caching, @Async and Scheduling

<details><summary>

## What is AOP? Explain aspect, advice, pointcut and join point.
</summary>

Aspect-Oriented Programming moves **cross-cutting concerns** (logging, metrics, security, transactions) out of business code.

- **Join point:** a point in execution (in Spring AOP, always a method call).
- **Pointcut:** an expression selecting join points.
- **Advice:** the code to run: `@Before`, `@After`, `@AfterReturning`, `@AfterThrowing`, `@Around`.
- **Aspect:** a class combining pointcuts and advice.

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Timed {}

@Aspect
@Component
public class TimingAspect {
    private static final Logger log = LoggerFactory.getLogger(TimingAspect.class);

    @Around("@annotation(Timed)")
    public Object time(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.nanoTime();
        try {
            return pjp.proceed();
        } finally {
            log.info("{} took {} ms", pjp.getSignature().toShortString(),
                     (System.nanoTime() - start) / 1_000_000);
        }
    }
}

@Service
class ReportService {
    @Timed
    public Report build(Long id) { ... }
}
```
</details>

<details><summary>

## How does Spring AOP work internally? JDK proxy vs CGLIB?
</summary>

Spring wraps the bean in a **proxy** at startup; callers get the proxy, which runs the advice and then calls the real method.

- **JDK dynamic proxy:** implements the bean's interfaces.
- **CGLIB proxy:** subclasses the bean class. Spring Boot uses CGLIB by default (`proxyTargetClass=true`).

Consequences (same as `@Transactional`): self-invocation bypasses the proxy, and `private`/`final` methods can't be advised. Spring AOP is runtime and method-level only; full AspectJ (compile/load-time weaving) can advise fields and constructors.
</details>

<details><summary>

## How does caching work in Spring Boot?
</summary>

Enable with `@EnableCaching` and annotate methods; the provider can be Caffeine (local) or Redis (shared across instances).

```java
@Service
public class CompanyService {

    @Cacheable(value = "companies", key = "#id", unless = "#result == null")
    public CompanyDto get(Long id) { return repo.findById(id).map(mapper::toDto).orElse(null); }

    @CachePut(value = "companies", key = "#result.id")
    public CompanyDto update(UpdateCompany cmd) { ... }

    @CacheEvict(value = "companies", key = "#id")
    public void delete(Long id) { repo.deleteById(id); }
}
```

```yaml
spring:
  cache:
    type: redis
    redis:
      time-to-live: 10m
```

Always set a TTL, cache DTOs (not lazy JPA entities), and remember the proxy rule: a cached method called from the same class is not cached.
</details>

<details><summary>

## How do @Async methods work and what are the pitfalls?
</summary>

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    @Bean(name = "ioExecutor")
    public Executor ioExecutor() {
        ThreadPoolTaskExecutor ex = new ThreadPoolTaskExecutor();
        ex.setCorePoolSize(8);
        ex.setMaxPoolSize(16);
        ex.setQueueCapacity(500);
        ex.setThreadNamePrefix("io-");
        ex.initialize();
        return ex;
    }
}

@Service
class NotificationService {
    @Async("ioExecutor")
    public CompletableFuture<Void> sendEmail(String to) { ...; return CompletableFuture.completedFuture(null); }
}
```

Pitfalls: self-invocation isn't async; exceptions in `void` methods are lost unless you set an `AsyncUncaughtExceptionHandler`; the security context and MDC aren't copied to the new thread automatically (use a `TaskDecorator`); `@Transactional` doesn't span threads.
</details>

<details><summary>

## How do you schedule jobs, and what happens with multiple instances?
</summary>

```java
@Configuration
@EnableScheduling
class SchedulingConfig {}

@Component
class CleanupJob {
    @Scheduled(cron = "0 0 2 * * *", zone = "Asia/Kolkata")   // 2 AM daily
    public void purgeExpired() { ... }

    @Scheduled(fixedDelay = 30_000)                           // 30s after previous run ends
    public void pollQueue() { ... }
}
```

With 3 pods, the job runs **3 times**. Fixes: a distributed lock with **ShedLock** (DB/Redis), a Kubernetes **CronJob**, or a leader-election mechanism. Also note the default scheduler has **one thread**, so a slow job delays others; configure `spring.task.scheduling.pool.size`.
</details>

<details><summary>

## How do Spring application events work?
</summary>

```java
public record OrderPlaced(Long orderId) {}

@Service
class OrderService {
    private final ApplicationEventPublisher events;
    OrderService(ApplicationEventPublisher events) { this.events = events; }

    @Transactional
    public void place(Order o) {
        repo.save(o);
        events.publishEvent(new OrderPlaced(o.getId()));
    }
}

@Component
class OrderListeners {
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void sendConfirmation(OrderPlaced e) { ... }  // only if the transaction committed
}
```

Events are synchronous by default (add `@Async` to decouple). They stay inside one JVM; across services use Kafka.
</details>
