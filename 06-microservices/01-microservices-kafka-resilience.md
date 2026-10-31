# Microservices, Kafka and Resilience

<details><summary>

## Monolith vs microservices: when would you choose each?
</summary>

**Microservices** give independent deployment and scaling per service, team ownership, and fault isolation, at the cost of network calls, distributed data, harder debugging and more infrastructure.

Start with a **well-modularised monolith** when the team is small or the domain is unclear. Split out services when parts need independent scaling (e.g. a high-throughput processing pipeline), different release cadence, or separate teams. Splitting too early creates a "distributed monolith" where every change touches many services.
</details>

<details><summary>

## How do microservices communicate? Sync vs async?
</summary>

- **Synchronous (REST/gRPC):** simple request-response; caller waits and is coupled to the callee's availability. Needs timeouts, retries and circuit breakers.
- **Asynchronous (Kafka, RabbitMQ):** producer publishes an event and moves on; consumers process independently. Better decoupling, buffering of load spikes and replay, but eventual consistency and harder flow tracing.

Rule of thumb: queries that need an immediate answer → sync; state changes other services react to → events.
</details>

<details><summary>

## How do you produce and consume Kafka messages with Spring Boot?
</summary>

```java
@Service
public class CallEventPublisher {
    private final KafkaTemplate<String, CallEvent> kafka;

    public CallEventPublisher(KafkaTemplate<String, CallEvent> kafka) { this.kafka = kafka; }

    public void publish(CallEvent e) {
        // key = callId -> all events for a call go to the same partition (ordering)
        kafka.send("call-events", e.callId(), e)
             .whenComplete((res, ex) -> { if (ex != null) log.error("Publish failed {}", e.callId(), ex); });
    }
}

@Component
public class TranscriptionConsumer {
    @KafkaListener(topics = "call-events", groupId = "transcription-service", concurrency = "6")
    public void onEvent(CallEvent e, Acknowledgment ack) {
        transcriptionService.process(e);   // must be idempotent
        ack.acknowledge();                 // commit offset after successful processing
    }
}
```

```yaml
spring:
  kafka:
    producer:
      acks: all
      properties:
        enable.idempotence: true
    consumer:
      enable-auto-commit: false
      auto-offset-reset: earliest
    listener:
      ack-mode: manual
```
</details>

<details><summary>

## How does Kafka guarantee ordering, and how do you scale consumers without breaking it?
</summary>

Kafka orders messages **only within a partition**. The producer picks the partition from the message **key** (hash), so all messages with the same key stay in order.

- Consumers in one **consumer group** split the partitions; maximum useful parallelism = number of partitions.
- Choose the key by what must stay ordered (call ID, account ID), not something with few values (hot partitions).
- For more parallelism inside one consumer, hash records by key to worker threads, so each key is still processed in order, and commit offsets only after all earlier records finish.
- Plan partition count for peak throughput up front; increasing it later changes the key → partition mapping.
</details>

<details><summary>

## Explain delivery semantics: at-most-once, at-least-once, exactly-once.
</summary>

- **At-most-once:** commit offset before processing. A crash loses messages.
- **At-least-once:** process, then commit. A crash after processing but before commit causes a **redelivery** → consumers must be **idempotent** (dedupe by event ID, upserts, unique constraints). This is the usual choice.
- **Exactly-once:** idempotent producer + Kafka transactions (`read-process-write` within Kafka). It doesn't cover external side effects like DB writes or emails; those still need idempotency.
</details>

<details><summary>

## How do you handle failures in Kafka consumers? (retries and DLQ)
</summary>

Don't block a partition forever on a poison message. Retry with backoff, then send to a **dead-letter topic** for inspection and replay.

```java
@Bean
public DefaultErrorHandler errorHandler(KafkaTemplate<Object, Object> template) {
    var recoverer = new DeadLetterPublishingRecoverer(template);          // -> <topic>.DLT
    var backoff = new ExponentialBackOffWithMaxRetries(3);
    backoff.setInitialInterval(1_000);
    backoff.setMultiplier(2.0);
    DefaultErrorHandler handler = new DefaultErrorHandler(recoverer, backoff);
    handler.addNotRetryableExceptions(ValidationException.class);       // bad data: straight to DLT
    return handler;
}
```

For long or non-blocking retries, Spring Kafka's `@RetryableTopic` routes failures through retry topics (`-retry-0`, `-retry-1`, then `-dlt`) without stalling the main partition. Alert on DLT growth.
</details>

<details><summary>

## What causes consumer lag and how do you fix it?
</summary>

Lag = latest offset − committed offset; consumers are slower than producers.

- Measure it (Kafka exporter / Burrow → Prometheus) and alert on growth.
- Scale consumers up to the partition count (Kubernetes HPA on lag via KEDA).
- Batch processing (`@KafkaListener(batch = "true")`) and bulk DB writes.
- Speed up slow downstream calls (caching, async IO) or move them out of the hot path.
- Separate priority traffic into its own topic so urgent work isn't stuck behind bulk backlog.
- Watch for rebalances (`max.poll.interval.ms` exceeded by slow processing).
</details>

<details><summary>

## What is the Saga pattern? Orchestration vs choreography?
</summary>

Distributed transactions (2PC) don't scale across microservices. A **saga** is a sequence of local transactions; if a step fails, earlier steps are undone by **compensating actions**.

- **Choreography:** each service listens for events and emits the next (`OrderCreated` → Payment → `PaymentDone` → Inventory). Loose coupling, but the overall flow is hard to see and debug.
- **Orchestration:** a central orchestrator tells each service what to do and tracks state; on failure it triggers compensations. Clear flow, easy monitoring and retries; the orchestrator must be reliable (persist its state).

Example compensation: payment succeeded but inventory failed → orchestrator issues `RefundPayment`.
</details>

<details><summary>

## What are CQRS and Event Sourcing?
</summary>

- **CQRS:** separate the **write model** (commands that validate and change state) from **read models** optimised for queries (denormalised views, Elasticsearch indexes). Read models are updated from events, so they are eventually consistent.
- **Event Sourcing:** store the sequence of events as the source of truth instead of current state; current state = replay of events (with snapshots for speed). Gives a full audit log and lets you rebuild new read models later.

They are often combined, but CQRS doesn't require event sourcing. Cost: more moving parts, eventual consistency, event schema versioning.
</details>

<details><summary>

## What is the transactional outbox pattern?
</summary>

Problem: saving to the DB and publishing to Kafka can't be one atomic action; a crash between them gives "saved but never published" or the reverse.

Solution: in the **same DB transaction**, write the business row and an `outbox` row. A relay (a poller, or CDC with Debezium) reads the outbox and publishes to Kafka, marking rows sent. Consumers must be idempotent because the relay can publish twice.

```java
@Transactional
public void placeOrder(Order o) {
    orderRepo.save(o);
    outboxRepo.save(new OutboxEvent("order", o.getId(), "OrderPlaced", toJson(o)));
}
```
</details>

<details><summary>

## What are circuit breakers, retries, timeouts and bulkheads? (Resilience4j)
</summary>

- **Timeout:** never wait forever on a dependency.
- **Retry:** retry transient failures with exponential backoff + jitter; only for idempotent calls.
- **Circuit breaker:** after a failure rate threshold, stop calling (OPEN) and fail fast; after a wait, allow a few trial calls (HALF_OPEN) before closing again.
- **Bulkhead:** limit concurrent calls per dependency so one slow service can't exhaust all threads.
- **Rate limiter:** cap outgoing call rate.

```java
@CircuitBreaker(name = "llm", fallbackMethod = "fallbackSummary")
@Retry(name = "llm")
@TimeLimiter(name = "llm")
public CompletableFuture<String> summarize(String transcript) {
    return CompletableFuture.supplyAsync(() -> llmClient.summarize(transcript));
}

private CompletableFuture<String> fallbackSummary(String transcript, Throwable t) {
    return CompletableFuture.completedFuture("Summary unavailable");
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      llm:
        failure-rate-threshold: 50
        sliding-window-size: 20
        wait-duration-in-open-state: 30s
  retry:
    instances:
      llm:
        max-attempts: 3
        wait-duration: 500ms
        enable-exponential-backoff: true
```
</details>

<details><summary>

## What are API gateway, service discovery and config server?
</summary>

- **API gateway** (Spring Cloud Gateway, Kong): single entry point; routing, auth, rate limiting, CORS, request logging.
- **Service discovery** (Eureka, Consul): services register and find each other by name. In Kubernetes, built-in Services + DNS usually replace Eureka.
- **Config server** (Spring Cloud Config): central, versioned configuration. In Kubernetes, ConfigMaps/Secrets are the common alternative.
</details>

<details><summary>

## How do you design a multi-tenant SaaS backend?
</summary>

**Data isolation options:**
1. **Shared schema** with a `tenant_id` column on every table: cheapest, needs strict enforcement (Hibernate `@TenantId` / filters, or Postgres row-level security).
2. **Schema per tenant:** better isolation, more migrations to run.
3. **Database per tenant:** strongest isolation (enterprise/BFSI customers), highest cost.

**Request flow:** resolve the tenant from the JWT claim (or subdomain) in a filter → store it in a `ThreadLocal` tenant context → every repository query and cache key includes it → propagate it in HTTP headers and Kafka headers to downstream services → clear the context in `finally`.

Also per tenant: rate limits, feature flags, configuration (e.g. model routing per client), and metrics tagged by tenant for noisy-neighbour detection.
</details>

<details><summary>

## How do you trace a request across microservices?
</summary>

Use distributed tracing: a `traceId` created at the edge is propagated in HTTP headers (W3C `traceparent`) and Kafka headers; each service adds spans. Spring Boot 3 does this with **Micrometer Tracing + OpenTelemetry**, exporting to Jaeger/Tempo/Zipkin. Put the `traceId` in every log line (MDC) so logs and traces link up in Grafana/Kibana.
</details>
