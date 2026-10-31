# Scenario-Based Backend Questions (Experienced, 2–5 years)

These are asked as "tell me how you…" questions. Answer with a real project: **situation → what you did → trade-offs → measurable result**. The points below are what interviewers listen for.

<details><summary>

## How do you break a backend requirement into architecture, tasks and measurable milestones?
</summary>

1. **Clarify:** users, scale (requests/sec, data volume), latency/availability targets, constraints, and what "done" means.
2. **Architecture:** components, data model, APIs, sync vs async flows. Write a short design doc and get it reviewed.
3. **Tasks:** thin vertical slices that each ship on their own (API + DB + test), with dependencies marked.
4. **Milestones with metrics:** API in staging behind a feature flag → first customer live → all customers. Track p95 latency, error rate, cost, adoption.

Example framing: "We replaced per-customer deployments with one multi-tenant service. Milestones were the tenant model and auth, then migrating customers one by one behind flags, measured by error rate and infra cost."
</details>

<details><summary>

## Describe a distributed system you designed: failures, retries and consistency trade-offs.
</summary>

Cover:
- **Flow:** e.g. API → Kafka → worker stages → DB/search index.
- **Failures:** timeouts on every external call, retries with exponential backoff, retry topics and a dead-letter topic, fallbacks (secondary provider or degraded response).
- **Idempotency:** a natural key (event ID, order ID) so reprocessing doesn't create duplicates.
- **Consistency:** at-least-once delivery + idempotent consumers rather than expensive exactly-once; stages are eventually consistent, with a status field per item so users see progress.
- **Back-pressure and priority:** bounded queues, separate topics for urgent work.
- **Observability:** lag, error rate and latency per stage with alerts.
</details>

<details><summary>

## How would you consolidate many Kafka topics into a cleaner routing design?
</summary>

- **Problem:** topic sprawl (per customer/per feature) means many consumers, hard-to-trace flows and operational load.
- **Design:** a common event envelope (`eventId`, `type`, `version`, `tenantId`, `payload`) on a few domain topics; route by event type in consumers.
- **CQRS:** commands change state and emit events; read models are built from those events.
- **Saga orchestration:** an orchestrator tracks multi-step flows and triggers compensations on failure.
- **Migration:** dual-publish to old and new topics, run shadow consumers and compare outputs, cut over gradually, then delete old topics.
- **Trade-offs:** fewer topics but heavier partitioning/keying decisions; schema versioning becomes critical (Schema Registry / versioned envelope).
</details>

<details><summary>

## Walk through a design decision you made, with trade-offs.
</summary>

Use this structure: **context → options → criteria → decision → outcome → what you'd revisit.**

Example: "Some non-search data lived in Elasticsearch. Options: keep it, shorten retention, or move it to MongoDB. Criteria: cost, query latency, retention needs, whether full-text search was needed. We moved non-search workloads to MongoDB with a TTL index for automatic expiry, and kept ES only for search. Result: lower storage cost and faster lookups. Trade-off: one more datastore to operate and keep in sync."
</details>

<details><summary>

## How do you write documentation so other engineers can contribute safely?
</summary>

- README per service: purpose, run locally, configuration, deploy, owners.
- Architecture diagram and **ADRs** (architecture decision records) explaining *why* choices were made.
- API docs generated from code (springdoc-openapi / Swagger UI) and a catalog of event schemas.
- Runbooks linked from alerts: symptom → checks → fix.
- Docs live in the repo and are part of the PR checklist, so they change with the code.
</details>

<details><summary>

## How do you design schemas and queries for large-scale read/write workloads?
</summary>

- Model around **access patterns**, not just entities.
- Composite and covering indexes; in multi-tenant tables, `tenant_id` leads the index. Verify with `EXPLAIN`.
- Keyset pagination instead of large `OFFSET`s.
- Avoid N+1 queries (fetch joins, batch fetching, DTO projections).
- Reads: read replicas and a Redis cache with TTL. Writes: batching, async processing via queues.
- Partition/shard large tables (by time or tenant); archive or TTL old data.
- Denormalise deliberately for hot read paths and keep copies in sync via events.
</details>

<details><summary>

## When would you choose Elasticsearch vs MongoDB, and how do you manage search latency?
</summary>

- **Elasticsearch:** full-text search, relevance ranking, fuzzy matching, faceted filters and aggregations.
- **MongoDB:** system of record for flexible documents, CRUD by key, transactions, TTL expiry.
- **Common pattern:** MongoDB/MySQL is the source of truth; ES is a search index fed from change events (Kafka/CDC), accepting ~1s indexing delay.
- **Latency control:** correct mappings (`keyword` for filters, `text` for search), filters in `filter` context (cached, unscored), `search_after` instead of deep paging, sensible shard sizes, routing by tenant, timeouts, Redis cache for hot queries, and tracking p95/p99.
</details>

<details><summary>

## Tell me about a performance bottleneck you found and how you measured the improvement.
</summary>

1. **Baseline** with metrics: throughput, p95/p99 latency, consumer lag, CPU/GC.
2. **Locate:** dashboards, profilers (async-profiler/JFR), thread dumps, slow-query logs, Kafka partition vs consumer alignment.
3. **Fix:** e.g. better partitioning and parallel batch processing, batching external API calls, adding an index, caching.
4. **Prove:** same load, same metrics, before vs after; watch for regressions elsewhere (cost, error rate).

State numbers you can defend: what was measured, the baseline, and exactly what changed.
</details>

<details><summary>

## How do you deliver a backend feature end to end, from requirement to release?
</summary>

1. Clarify the requirement and acceptance criteria with product.
2. Design doc and review; agree the API contract with consumers.
3. Implement with unit, slice and integration tests (Testcontainers).
4. Code review, CI pipeline (build, tests, static analysis, image build).
5. Ship behind a **feature flag**; validate in staging.
6. Gradual rollout (canary / percentage / one tenant first) with a rollback plan.
7. Dashboards and alerts live before full release; update docs and runbooks.
8. Monitor after release and close the loop with product on the success metric.
</details>

<details><summary>

## How do you build multi-tenant APIs with authentication and RBAC across services?
</summary>

- **Identity:** OAuth2/OIDC; Authorization Code + PKCE for browsers; short-lived JWT access tokens with `tenantId` and roles/permissions claims; refresh tokens and Redis-backed sessions for revocation.
- **Tenant context:** extracted in a filter, stored in a request-scoped/`ThreadLocal` context, propagated via HTTP and Kafka headers, cleared after the request.
- **Data isolation:** tenant filter enforced centrally (Hibernate filter/`@TenantId`, repository base class, or DB row-level security), never left to each query.
- **RBAC:** roles map to permissions; check permissions with `@PreAuthorize`; multi-level roles (platform admin, org admin, member) scoped to a tenant; permissions cached in Redis.
- **Service-to-service:** client-credentials tokens or mTLS; downstream services re-check authorization.
</details>

<details><summary>

## How do you design Kafka consumer parallelism for throughput without breaking ordering?
</summary>

- Ordering exists only within a partition, so key messages by what must stay ordered (e.g. call/order/account ID).
- Consumers in a group scale up to the partition count; plan partitions for peak load.
- More parallelism inside a consumer: hash records by key to a fixed set of worker threads (each key always goes to the same thread), and commit offsets only up to the lowest fully processed offset.
- Watch for hot keys, use cooperative-sticky rebalancing, and keep processing time under `max.poll.interval.ms`.
- Separate high-priority traffic into its own topic/consumer group.
</details>

<details><summary>

## How do you ensure observability: metrics, logs, dashboards and alerts?
</summary>

- **Metrics** (Micrometer → Prometheus): the RED method per endpoint (rate, errors, duration), Kafka consumer lag, JVM heap/GC, thread pools, DB connection pool usage.
- **Dashboards** (Grafana): one per service and per pipeline stage.
- **Logs:** structured JSON with `traceId`, `tenantId` and business IDs; centralised (Loki/ELK).
- **Tracing:** OpenTelemetry across HTTP and Kafka.
- **Alerts:** on user-facing symptoms (SLO burn rate, p95 latency, error rate, growing lag) rather than raw CPU; each alert links to a runbook.
- **Health:** liveness/readiness probes so Kubernetes restarts or removes unhealthy pods.
</details>

<details><summary>

## Your API latency suddenly jumped from 100 ms to 2 s. How do you investigate?
</summary>

1. **Scope:** all endpoints or one? All tenants or one? Since when? Did a deploy, config change or traffic spike happen then?
2. **Traces:** find which span grew (DB, cache, downstream API).
3. **DB:** slow query log, missing index after a migration, lock waits, connection pool exhausted (`hikaricp_connections_pending`).
4. **JVM:** GC pauses, thread pool saturation, CPU throttling in Kubernetes.
5. **Dependencies:** downstream latency, cache hit rate drop (Redis restarted → cache stampede).
6. **Mitigate first** (rollback, scale out, enable circuit breaker), then fix the root cause and write a blameless postmortem.
</details>

<details><summary>

## How would you design a web crawler that collects company data and makes it searchable?
</summary>

- **URL frontier** in Kafka, partitioned by domain for politeness; priority for important or stale pages.
- **Fetchers:** stateless workers that respect robots.txt and per-domain rate limits (Redis), retry with backoff.
- **Dedup:** URL hash in Redis/Bloom filter; content hash to skip unchanged pages.
- **Raw storage:** HTML in object storage (S3/GCS); only extracted fields in databases.
- **Extraction:** parsers (and LLMs for messy pages) produce structured fields with confidence scores.
- **Entity resolution:** match "Acme Technologies Pvt Ltd" and "Acme" using normalised names plus website domain.
- **Storage and search:** MongoDB as source of truth; Elasticsearch fed by change events; Redis for hot queries.
- **Recrawl scheduler:** frequently changing sources (news) recrawled more often than static pages.
- **Failures:** retry topic → DLQ, idempotent upserts, low-confidence records sent for human review.
</details>
