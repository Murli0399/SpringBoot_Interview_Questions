# Spring Data JPA and transactions

<details><summary>

## [Easy] What is Spring Data?
</summary>

Spring Data is a family of projects that simplifies the use of data access technologies in Spring applications. It includes projects for working with relational databases (Spring Data JPA, Spring Data JDBC, and Spring Data R2DBC), NoSQL databases (Spring Data MongoDB, Spring Data Cassandra, and Spring Data Neo4j), and cloud-based data services (Spring Data AWS, Spring Data Azure, and Spring Data GCP).

Spring Data makes it easy to:
- Connect to a variety of data sources
- Perform basic CRUD operations
- Implement custom queries
- Configure paging, sorting, and filtering
- Handle transactions
- Use caching
- Integrate with other Spring projects

Spring Data is a popular choice for developers who want to build high-quality, scalable applications that use data access technologies. It is easy to use, flexible, and well-supported.

Here are some of the benefits of using Spring Data:
- It makes data access simpler and more efficient.
- It provides a consistent development experience across different data sources.
- It gives you the freedom to choose the data access technology that best meets your needs.
- It is well-supported by a community of developers.
- If you are looking for a way to simplify and improve your data access, then Spring Data is a great option.

**Example:**

```java
// Declare an interface; Spring Data generates the implementation at runtime
interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
    long countByStatus(Status status);
}
```

</details>

<details><summary>

## [Easy] What is JPA?
</summary>

The **Java Persistence API** – now **Jakarta Persistence** (`jakarta.persistence`, since Jakarta EE 9) – is a **specification** for object-relational mapping in Java. It defines:
- How to map classes to tables with annotations: `@Entity`, `@Table`, `@Id`, `@Column`, `@OneToMany`, `@ManyToOne`, …
- The **`EntityManager`** API to persist, find, merge and remove entities, and the **persistence context** that tracks them.
- **JPQL** (an object-oriented query language) and the **Criteria API**.
- Transaction integration and entity lifecycle callbacks.

JPA is only a set of interfaces and rules; it needs a **provider** that implements it, such as **Hibernate** (the default in Spring Boot) or EclipseLink. JPA 1.0 arrived in 2006 as part of EJB 3.0, replacing EJB 2 entity beans. Spring Boot 4 uses Jakarta Persistence 3.2.

**Example:**

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY) private Long id;
    @Column(nullable = false) private String customerId;
    @Enumerated(EnumType.STRING) private OrderStatus status;
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
    protected Order() { }
}

Order o = entityManager.find(Order.class, 1L);   // JPA API; Hibernate is the implementation
```

</details>

<details><summary>

## [Easy] What is the difference between JPA and Spring Data JPA?
</summary>

They are different layers that work together:

| | JPA (Jakarta Persistence) | Spring Data JPA |
|---|---|---|
| What it is | A **specification** for ORM | A Spring library built **on top of** JPA |
| Implemented by | Hibernate, EclipseLink | Spring (it still needs a JPA provider underneath) |
| Main API | `EntityManager`, JPQL, Criteria API | Repository **interfaces** (`JpaRepository`) whose implementations Spring generates |
| Queries | You write JPQL/Criteria code | Derived from method names (`findByEmail`), or `@Query`, Specifications, Query by Example |
| Paging/sorting | `setFirstResult`/`setMaxResults`, `ORDER BY` | `Pageable`, `Page`, `Sort` built in |
| Transactions | `EntityTransaction` or JTA | Uses Spring's `@Transactional` (repository methods are transactional by default) |
| Boilerplate | Write DAO classes yourself | Mostly none – declare an interface |

Spring Data JPA doesn't replace JPA; it removes the repetitive DAO code. You still need to understand JPA concepts (persistence context, fetching, transactions) to use it well.

</details>

<details><summary>

## [Easy] What is the difference between CrudRepository and JpaRepository?
</summary>

| | `CrudRepository<T, ID>` | `JpaRepository<T, ID>` |
|---|---|---|
| Module | Spring Data Commons (works with any store) | Spring Data JPA only |
| Methods | `save`, `saveAll`, `findById`, `existsById`, `findAll`, `count`, `delete…` | Everything in `ListCrudRepository` + `ListPagingAndSortingRepository` + `QueryByExampleExecutor`, plus `flush`, `saveAndFlush`, `saveAllAndFlush`, `deleteAllInBatch`, `getReferenceById` |
| Return types | `Iterable<T>` | `List<T>` |
| Paging & sorting | No (since Spring Data 3, `PagingAndSortingRepository` no longer extends `CrudRepository`) | Yes |

Use `CrudRepository` (or `ListCrudRepository`) for simple CRUD or when you want to stay store-agnostic. Use `JpaRepository` in JPA projects when you need paging, sorting, batch deletes or flush control – in practice most JPA projects just use `JpaRepository`.

**Example:**

```java
interface A extends CrudRepository<User, Long> { }        // save, findById, findAll (Iterable), delete
interface B extends JpaRepository<User, Long> { }         // + paging/sorting, List returns, flush, saveAllAndFlush, batch deletes

Page<User> page = userRepo.findAll(PageRequest.of(0, 20, Sort.by("createdAt").descending()));
```

</details>

<details><summary>

## [Easy] Which No SQL databases does Spring support?
</summary>

Through **Spring Data** modules: **MongoDB** (document), **Redis** (key-value), **Elasticsearch** (search), **Cassandra** (wide-column), **Neo4j** (graph), **Couchbase**, **Apache Geode / VMware GemFire**, and **LDAP**, plus community modules (e.g. DynamoDB, Azure Cosmos DB, Google Cloud Spanner/Firestore through Spring Cloud GCP/Azure).

Spring Data Solr has been discontinued. Each module offers repository interfaces, templates (`MongoTemplate`, `RedisTemplate`, `ElasticsearchOperations`), and mapping annotations, and most also have reactive variants.

**Example:**

```java
interface CompanyDocs extends MongoRepository<CompanyDoc, String> {
    List<CompanyDoc> findBySectorAndFoundedYearGreaterThan(String sector, int year);
}

@Document(collection = "companies")
record CompanyDoc(@Id String id, String name, String sector, int foundedYear) {}
// Also: Spring Data Redis, Elasticsearch, Cassandra, Neo4j, Couchbase
```

</details>

<details><summary>

## [Easy] Which ORMs are supported by Spring?
</summary>

Spring's ORM integration (`spring-orm`) today focuses on **JPA (Jakarta Persistence)** and **Hibernate**:

- **Hibernate ORM** – the default JPA provider in Spring Boot, also usable through its native API.
- **EclipseLink** – another JPA provider.
- **MyBatis** – a SQL mapper (not a full ORM), integrated through the separate `mybatis-spring` project.
- **jOOQ** – a type-safe SQL DSL with Spring Boot auto-configuration.
- **Spring Data JDBC / R2DBC** – lightweight object mapping from the Spring Data family.

Older integrations – iBATIS, JDO, Oracle TopLink, Apache OJB – have been **removed** from Spring over the years and shouldn't be mentioned as current options.

**Example:**

```java
// Hibernate is the default JPA provider in Spring Boot (spring-boot-starter-data-jpa).
// Others: EclipseLink (JPA), plus non-JPA options like MyBatis, jOOQ, and Spring Data JDBC.
@Bean
JdbcClient jdbcClient(DataSource ds) { return JdbcClient.create(ds); }   // lightweight SQL option
```

</details>

<details><summary>

## [Easy] What is Hibernate?
</summary>

**Hibernate ORM** is the most widely used **JPA implementation** and the default persistence provider in Spring Boot. It maps Java classes to tables and Java types to SQL types, then generates the SQL for you.

Key features:
- **Implements JPA** (`EntityManager`, JPQL) and adds its own features (HQL extensions, filters, multi-tenancy, `@NaturalId`, batch fetching).
- **Persistence context (first-level cache)** with **dirty checking:** modified managed entities are updated automatically at commit.
- **Lazy loading** of associations through proxies.
- **Second-level and query caches** (optional, e.g. with Caffeine/Ehcache/JCache).
- **Database independence** through dialects (MySQL, PostgreSQL, Oracle, …).
- **Schema generation** (`hibernate.hbm2ddl.auto` / `spring.jpa.hibernate.ddl-auto`) – handy for development; use Flyway/Liquibase migrations in production.

Performance depends on using it well: avoid N+1 queries, fetch what each use case needs, enable JDBC batching. Spring Boot 4 ships with Hibernate 7.

**Example:**

```yaml
spring:
  jpa:
    show-sql: false
    properties:
      hibernate:
        format_sql: true
        jdbc.batch_size: 50          # batch inserts/updates
        default_batch_fetch_size: 50 # reduce N+1 on lazy collections
```

```java
@Transactional
public void rename(Long id, String name) {
    Company c = repo.findById(id).orElseThrow();
    c.setName(name);       // Hibernate's dirty checking issues the UPDATE at commit
}
```

</details>

<details><summary>

## [Easy] What are some benefits of using Spring Transactions?
</summary>

- **One programming model** for every transaction API: JDBC, JPA/Hibernate, JTA, R2DBC, and even Kafka transactions – switch by changing the `PlatformTransactionManager` bean, not your code.
- **Declarative transactions** with `@Transactional`, keeping business code clean.
- **A simpler programmatic API** (`TransactionTemplate`) than raw JTA or JDBC calls.
- **Integration with Spring's data access** – `JdbcTemplate`, Spring Data repositories and `EntityManager` automatically join the current transaction.
- **Fine-grained control** – propagation, isolation, timeouts, read-only hints and rollback rules.
- **Transaction-aware events** – `@TransactionalEventListener` runs code only after commit.
- **Testability** – `@Transactional` tests roll back automatically after each test.

The reliability benefit comes from atomicity: either all changes in a transaction are committed, or none are.

**Example:**

```java
// Same annotation works with JPA, JDBC or JTA - only the transaction manager bean changes
@Transactional
public void placeOrder(Order o) {
    orderRepo.save(o);          // JPA
    jdbc.update("UPDATE stock SET qty = qty - 1 WHERE sku = ?", o.sku());   // JdbcTemplate joins the same transaction
}
```

</details>

<details><summary>

## [Medium] What are the different types of transaction management supported by Spring?
</summary>

Spring supports two styles over one abstraction (`PlatformTransactionManager`, e.g. `JpaTransactionManager`, `DataSourceTransactionManager`, `JtaTransactionManager`):

**1. Declarative** (recommended for most code) – annotate methods/classes with `@Transactional`; Spring's AOP proxy begins, commits or rolls back the transaction.

**2. Programmatic** – control boundaries in code with `TransactionTemplate` (or directly with `PlatformTransactionManager`). Useful for fine-grained boundaries inside one method, e.g. committing each batch of a long job separately.

Key settings available in both:
- **Propagation** – what happens when a transactional method calls another: `REQUIRED` (join or start, default), `REQUIRES_NEW` (suspend and start an independent one), `NESTED` (savepoint), `MANDATORY`, `SUPPORTS`, `NOT_SUPPORTED`, `NEVER`.
- **Isolation** – how concurrent transactions see each other's changes: `READ_UNCOMMITTED`, `READ_COMMITTED`, `REPEATABLE_READ`, `SERIALIZABLE`, or `DEFAULT` (use the database's default). These prevent dirty reads, non-repeatable reads and phantom reads to different degrees.
- **Rollback rules**, **timeout**, **read-only**.
- **Rollback-only:** `TransactionStatus.setRollbackOnly()` (or `TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()`) marks the current transaction to roll back without throwing.

See `04-jpa-performance-and-transactions.md` for propagation and isolation in detail.

**Example:**

```java
// Declarative (preferred)
@Transactional
public void transfer(Long from, Long to, BigDecimal amt) { ... }

// Programmatic (fine-grained control)
transactionTemplate.execute(status -> {
    debit(from, amt);
    if (fraudCheckFails()) status.setRollbackOnly();
    credit(to, amt);
    return null;
});
```

</details>

<details><summary>

## [Medium] Which transaction management type is preferred in Spring?
</summary>

**Declarative** (`@Transactional`) is preferred for most application code:
- Business logic stays free of transaction plumbing.
- Consistent behaviour, configured in one place (propagation, isolation, rollback rules).
- Less error-prone than manual commit/rollback.

**Programmatic** (`TransactionTemplate`) is still a valid tool, not something to avoid. Use it when you need:
- Several separate transactions inside one method (e.g. commit every 500 records of a batch job).
- A transaction around only part of a method, keeping slow work (remote API calls) outside it.
- Transactions in code that isn't called through a Spring proxy.

```java
transactionTemplate.executeWithoutResult(status -> {
	orderRepository.save(order);
	if (!inventory.reserve(order)) status.setRollbackOnly();
});
```

**Example:**

```java
@Service
public class AccountService {
    @Transactional(readOnly = true)                 // read-only optimisation for queries
    public Account get(Long id) { return repo.findById(id).orElseThrow(); }

    @Transactional(rollbackFor = Exception.class, timeout = 5)
    public void transfer(Long from, Long to, BigDecimal amt) { ... }
}
```

</details>

<details><summary>

## [Medium] What happens when the @Transactional annotation is used on a method?
</summary>

Spring wraps the bean in an **AOP proxy**. When a caller invokes the method **through the proxy**, Spring:

1. Asks the `PlatformTransactionManager` to **begin** a transaction, or **join** an existing one, according to the propagation setting.
2. Binds the transaction's connection/`EntityManager` to the current thread, so repositories and templates use it.
3. Calls your method.
4. **Commits** if the method returns normally.
5. **Rolls back** if it throws a **`RuntimeException` or `Error`**. By default, **checked exceptions do NOT trigger a rollback** – the transaction commits. Change this with `rollbackFor = Exception.class`.

Attributes and their **defaults**:

| Attribute | Default |
|---|---|
| `propagation` | `REQUIRED` |
| `isolation` | **`DEFAULT`** – uses the database's own default (e.g. REPEATABLE READ in MySQL InnoDB, READ COMMITTED in PostgreSQL) |
| `timeout` | `-1` (none, or the transaction manager's default) |
| `readOnly` | `false` |
| `rollbackFor` / `noRollbackFor` | Roll back on unchecked exceptions and errors only |

Where to put it: on **public methods of concrete classes** (usually services), or on the class to apply it to all public methods. It doesn't apply to self-invocation (`this.method()`), private methods, or objects not created by Spring – see `04-jpa-performance-and-transactions.md`.

**Example:**

```java
// What the proxy does around your method (simplified):
TransactionStatus tx = txManager.getTransaction(definition);   // begin (or join existing)
try {
    Object result = target.placeOrder(order);                   // your code
    txManager.commit(tx);
    return result;
} catch (RuntimeException | Error e) {
    txManager.rollback(tx);                                     // checked exceptions commit by default!
    throw e;
}
// See 04-jpa-performance-and-transactions.md for propagation and when @Transactional is ignored.
```

</details>

<details><summary>

## [Medium] What is the difference between JdbcTemplate and JPA?
</summary>

| | `JdbcTemplate` | JPA (Hibernate) |
|---|---|---|
| Approach | SQL-first: you write SQL and map rows | Object-first: you map entities; SQL is generated |
| Abstraction | Low (thin layer over JDBC) | High (ORM with persistence context) |
| Control over SQL | Full – any database feature | Less direct; JPQL/Criteria, native queries when needed |
| Boilerplate for CRUD | More (write each query and mapper) | Less (repositories, cascading, dirty checking) |
| Relationships | Handled manually with joins | Mapped with `@OneToMany`, `@ManyToOne`, lazy loading |
| Performance predictability | High – exactly the SQL you wrote | Depends on usage; N+1 queries and over-fetching are common pitfalls |
| Learning curve | **Gentle** – know SQL, you're done | **Steeper** – persistence context, fetching strategies, entity states |

Many applications use both: JPA for typical domain CRUD, `JdbcTemplate`/`JdbcClient` for reporting queries, bulk operations and performance-critical SQL.

</details>
