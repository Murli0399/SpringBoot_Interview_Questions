# JPA Performance, Transactions and Locking

<details><summary>

## What is the N+1 query problem and how do you fix it?
</summary>

Loading N parent entities and then lazily touching a child collection fires 1 query for the parents + N queries for the children.

```java
@Entity
class Company {
    @Id Long id;
    String name;
    @OneToMany(mappedBy = "company", fetch = FetchType.LAZY)
    List<FundingRound> rounds;
}

List<Company> companies = companyRepo.findAll();          // 1 query
companies.forEach(c -> c.getRounds().size());             // +N queries
```

Fixes:

```java
// 1. JOIN FETCH
@Query("select distinct c from Company c left join fetch c.rounds where c.sector = :sector")
List<Company> findWithRounds(@Param("sector") String sector);

// 2. Entity graph
@EntityGraph(attributePaths = "rounds")
List<Company> findBySector(String sector);

// 3. Batch fetching: loads children for 50 parents per query
// spring.jpa.properties.hibernate.default_batch_fetch_size=50

// 4. DTO projection when you only need a few columns
@Query("select new com.x.CompanySummary(c.id, c.name, count(r)) from Company c left join c.rounds r group by c.id, c.name")
List<CompanySummary> summaries();
```

Detect it by enabling SQL logging in tests or counting statements (Hibernate statistics). Note: `JOIN FETCH` on a collection with pagination makes Hibernate paginate in memory; use batch fetching or a two-step query (page IDs, then fetch).
</details>

<details><summary>

## What is the difference between LAZY and EAGER fetching?
</summary>

- **LAZY:** association loaded on first access (via a proxy). Default for `@OneToMany` and `@ManyToMany`.
- **EAGER:** loaded immediately. Default for `@ManyToOne` and `@OneToOne`.

Best practice: make everything LAZY (`@ManyToOne(fetch = FetchType.LAZY)`) and fetch what each use case needs with `JOIN FETCH` or entity graphs. EAGER everywhere causes huge joins and hidden N+1s.

`LazyInitializationException`: accessing a lazy association after the transaction/session closed (e.g. in a controller). Fix by fetching it inside the service transaction or returning a DTO; don't rely on `open-in-view` (set `spring.jpa.open-in-view=false`).
</details>

<details><summary>

## Explain transaction propagation levels.
</summary>

Propagation decides what happens when a `@Transactional` method calls another.

| Propagation | Behaviour |
|---|---|
| `REQUIRED` (default) | Join the existing transaction, or start one |
| `REQUIRES_NEW` | Suspend the current one and start a new, independent transaction |
| `NESTED` | Savepoint inside the current transaction (JDBC only) |
| `SUPPORTS` | Join if one exists, else run without |
| `MANDATORY` | Must already be in a transaction, else exception |
| `NOT_SUPPORTED` | Suspend any transaction, run without |
| `NEVER` | Exception if a transaction exists |

```java
@Transactional
public void placeOrder(Order o) {
    orderRepo.save(o);
    auditService.log("ORDER_PLACED", o.getId());  // should survive even if order fails later
    paymentService.charge(o);                     // throws -> order rolls back
}

@Service
class AuditService {
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void log(String event, Long id) { auditRepo.save(new Audit(event, id)); }
}
```
</details>

<details><summary>

## Explain isolation levels in @Transactional.
</summary>

`@Transactional(isolation = Isolation.READ_COMMITTED)` sets the DB isolation level for that transaction.

| Level | Dirty read | Non-repeatable read | Phantom read |
|---|---|---|---|
| READ_UNCOMMITTED | Possible | Possible | Possible |
| READ_COMMITTED (PostgreSQL default) | Prevented | Possible | Possible |
| REPEATABLE_READ (MySQL InnoDB default) | Prevented | Prevented | Mostly prevented (gap locks) |
| SERIALIZABLE | Prevented | Prevented | Prevented |

Higher isolation = more locking and lower throughput. `Isolation.DEFAULT` uses the database's default.
</details>

<details><summary>

## When does @Transactional not work?
</summary>

Spring applies `@Transactional` through a **proxy**, so it is skipped when the call doesn't go through the proxy:

1. **Self-invocation:** `this.save()` from another method in the same class.
2. **Private or final methods** (proxies can't override them).
3. The class isn't a Spring bean (created with `new`).
4. **Checked exceptions** don't roll back by default. Use `@Transactional(rollbackFor = Exception.class)`.
5. Exception caught and swallowed inside the method.
6. Long work inside a transaction (remote calls, Kafka publish) holds DB connections and locks.

```java
@Service
public class ReportService {
    public void generateAll() {
        reports.forEach(r -> generate(r));   // self-invocation: NO transaction applied
    }

    @Transactional
    public void generate(Report r) { ... }
}
// Fix: move generate() to another bean, or inject the bean into itself (@Lazy self-reference)
```
</details>

<details><summary>

## What is optimistic vs pessimistic locking in JPA?
</summary>

**Optimistic:** add a `@Version` column. The update includes `WHERE version = ?`; if another transaction changed the row first, Hibernate throws `ObjectOptimisticLockingFailureException`, which you retry or report as 409 Conflict. Best for low contention.

```java
@Entity
class Wallet {
    @Id Long id;
    BigDecimal balance;
    @Version Long version;
}
```

**Pessimistic:** lock the row in the DB (`SELECT ... FOR UPDATE`) so others wait. Best for high contention (inventory, seat booking).

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("select w from Wallet w where w.id = :id")
Optional<Wallet> findForUpdate(@Param("id") Long id);
```
</details>

<details><summary>

## How do you paginate efficiently with Spring Data JPA?
</summary>

```java
Page<Company> page = repo.findBySector("fintech", PageRequest.of(0, 20, Sort.by("createdAt").descending()));
page.getContent(); page.getTotalElements(); page.getTotalPages();
```

- `Page` runs an extra `COUNT` query; use `Slice<T>` when you only need "has next".
- Deep pages with `OFFSET` get slow (DB still scans skipped rows). Use **keyset (cursor) pagination**:

```java
@Query("select c from Company c where c.id > :lastId order by c.id")
List<Company> nextPage(@Param("lastId") Long lastId, Pageable limit);   // PageRequest.of(0, 20)
```
</details>

<details><summary>

## What is the difference between save(), saveAndFlush() and saveAll()? How do you do bulk inserts?
</summary>

- `save()`: persists or merges; SQL is sent at flush/commit.
- `saveAndFlush()`: also flushes immediately (useful to catch constraint errors early).
- `saveAll()`: loops over `save()`.

For real bulk inserts enable JDBC batching:

```properties
spring.jpa.properties.hibernate.jdbc.batch_size=50
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true
```

`GenerationType.IDENTITY` (MySQL auto-increment) **disables insert batching** in Hibernate; use `SEQUENCE` where possible, or `JdbcTemplate.batchUpdate` for very large loads.
</details>

<details><summary>

## What are entity states in JPA (persistence context)?
</summary>

- **Transient:** `new Company()`; not tracked.
- **Managed:** loaded or saved within an open persistence context; changes are **auto-detected (dirty checking)** and flushed at commit, no `save()` needed.
- **Detached:** was managed, but the context closed; changes aren't tracked (`merge()` reattaches).
- **Removed:** scheduled for deletion.

```java
@Transactional
public void rename(Long id, String name) {
    Company c = repo.findById(id).orElseThrow();
    c.setName(name);     // UPDATE issued at commit via dirty checking
}
```
</details>

<details><summary>

## What is the first-level and second-level cache in Hibernate?
</summary>

- **First-level cache:** the persistence context itself; per transaction/session, always on. Loading the same id twice in one transaction hits the DB once.
- **Second-level cache:** shared across sessions (Ehcache, Caffeine, Redis via JCache); opt-in per entity with `@Cacheable`. Good for read-mostly reference data; risky for frequently updated data or multiple app instances without a distributed cache.
</details>
