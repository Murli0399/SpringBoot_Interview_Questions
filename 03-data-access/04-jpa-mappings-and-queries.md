<details><summary>

## [Medium] How do you map one-to-many and many-to-one relationships in JPA?
</summary>

Use `@ManyToOne` on the "many" side (it owns the foreign key) and `@OneToMany(mappedBy = ...)` on the "one" side for a **bidirectional** relationship.

```java
@Entity
public class Company {
	@Id @GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;
	private String name;

	@OneToMany(mappedBy = "company", cascade = CascadeType.ALL, orphanRemoval = true)
	private List<FundingRound> rounds = new ArrayList<>();

	public void addRound(FundingRound round) {      // keep both sides in sync
		rounds.add(round);
		round.setCompany(this);
	}
	public void removeRound(FundingRound round) {
		rounds.remove(round);
		round.setCompany(null);
	}
}

@Entity
public class FundingRound {
	@Id @GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;
	private long amountUsd;

	@ManyToOne(fetch = FetchType.LAZY)             // owning side: has the company_id column
	@JoinColumn(name = "company_id")
	private Company company;

	void setCompany(Company company) { this.company = company; }
}
```

Points interviewers check:
- **`mappedBy`** marks the **inverse** side; only the owning side (`@ManyToOne`, with the FK) is used to write the relationship to the database.
- Always update **both sides** in memory via helper methods.
- Make `@ManyToOne` **LAZY** (its default is EAGER).
- `@ManyToMany` uses a join table; for extra columns on the link (e.g. "enrolled_on"), model the join table as its own entity with two `@ManyToOne`s.
</details>

<details><summary>

## [Medium] What are cascade types and orphanRemoval?
</summary>

**Cascade** propagates an `EntityManager` operation from a parent to its associated entities:

| CascadeType | Propagates |
|---|---|
| `PERSIST` | Saving a new parent also saves new children |
| `MERGE` | Merging a detached parent also merges children |
| `REMOVE` | Deleting the parent deletes its children |
| `REFRESH`, `DETACH` | Refresh / detach children too |
| `ALL` | All of the above |

**`orphanRemoval = true`** deletes a child when it's **removed from the parent's collection** (or the reference is set to null), even though the parent itself isn't deleted.

```java
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
private List<OrderItem> items = new ArrayList<>();

order.getItems().remove(item);   // with orphanRemoval: DELETE FROM order_item ... at flush
```

Use cascades for **parent-child aggregates** whose children have no life of their own (order → order items). Don't cascade `REMOVE` (or `ALL`) on `@ManyToOne` or `@ManyToMany`: deleting one order would try to delete the shared customer or tags.
</details>

<details><summary>

## [Medium] How do you write custom queries in Spring Data JPA?
</summary>

**1. Derived query methods** – Spring parses the method name:

```java
interface CompanyRepository extends JpaRepository<Company, Long> {
	List<Company> findBySectorAndFoundedYearGreaterThanOrderByNameAsc(String sector, int year);
	Optional<Company> findFirstByNameIgnoreCase(String name);
	boolean existsByWebsite(String website);
	long countBySector(String sector);
}
```

**2. `@Query` with JPQL or native SQL:**

```java
@Query("select c from Company c where c.fundingUsd > :min and c.sector = :sector")
List<Company> funded(@Param("min") long min, @Param("sector") String sector);

@Query(value = "select * from company where name ilike concat('%', :q, '%') limit 20", nativeQuery = true)
List<Company> search(@Param("q") String q);
```

**3. Bulk updates/deletes** need **`@Modifying`** (and a transaction):

```java
@Modifying(clearAutomatically = true)
@Transactional
@Query("update Company c set c.status = 'INACTIVE' where c.lastSeen < :cutoff")
int deactivateStale(@Param("cutoff") Instant cutoff);
```

Bulk queries bypass the persistence context, so `clearAutomatically = true` prevents stale managed entities.

**4. Specifications / Criteria API** for dynamic filters, **Query by Example**, and **Querydsl** for type-safe queries.
</details>

<details><summary>

## [Medium] What are projections in Spring Data JPA?
</summary>

A **projection** fetches only the columns you need instead of whole entities – less data, no dirty checking, no accidental lazy loading.

```java
// 1. Interface-based projection
interface CompanySummary {
	String getName();
	String getSector();
}
List<CompanySummary> findBySector(String sector);

// 2. Class/record-based (DTO) projection
record CompanyCard(Long id, String name, long fundingUsd) {}

@Query("select new com.example.CompanyCard(c.id, c.name, c.fundingUsd) from Company c where c.sector = :sector")
List<CompanyCard> cards(@Param("sector") String sector);

// 3. Dynamic projection: the caller picks the type
<T> List<T> findByCountry(String country, Class<T> type);
```

Use projections for read-heavy endpoints, list screens and reports; use entities when you need to modify data.
</details>

<details><summary>

## [Medium] What is the difference between persist(), merge(), and Spring Data's save()?
</summary>

- **`persist(entity)`** – makes a **new** entity managed; it's inserted at flush. The same object becomes managed. Calling it on a detached entity throws an exception.
- **`merge(entity)`** – copies the state of a **detached** (or new) object onto a **managed copy** and returns that copy; the object you passed stays detached. Use the returned instance afterwards.
- **Spring Data `save(entity)`** – calls `persist` if the entity is **new** (id is null, or `@Version` is null, or it implements `Persistable.isNew()`), otherwise `merge`. It returns the managed instance, so write `entity = repo.save(entity)`.

```java
@Transactional
public void rename(Long id, String name) {
	Company c = repo.findById(id).orElseThrow();   // managed
	c.setName(name);                               // no save() needed: dirty checking issues UPDATE at commit
}
```

A common performance trap: with **manually assigned IDs**, `save()` thinks the entity isn't new and runs `merge`, which first SELECTs the row. Implement `Persistable` or use a `@Version` field to avoid that.
</details>

<details><summary>

## [Medium] What is the difference between findById() and getReferenceById()?
</summary>

| | `findById(id)` | `getReferenceById(id)` (formerly `getOne`) |
|---|---|---|
| Database hit | **Immediately** SELECTs the row | **No query** – returns a lazy **proxy** |
| Missing row | Returns `Optional.empty()` | Throws `EntityNotFoundException` only when you first access a field |
| Use when | You need the entity's data | You only need a **reference** to set a foreign key |

```java
@Transactional
public void addRound(Long companyId, long amount) {
	Company ref = companyRepo.getReferenceById(companyId);   // no SELECT
	roundRepo.save(new FundingRound(ref, amount));           // INSERT with company_id
}
```

Accessing a proxy's fields outside a transaction throws `LazyInitializationException`.
</details>

<details><summary>

## [Medium] How do you implement auditing (createdAt, updatedBy) in Spring Data JPA?
</summary>

Enable JPA auditing and annotate the fields; Spring fills them automatically on insert and update.

```java
@Configuration
@EnableJpaAuditing
class AuditConfig {
	@Bean
	AuditorAware<String> auditorAware() {          // who is the current user?
		return () -> Optional.ofNullable(SecurityContextHolder.getContext().getAuthentication())
		                     .map(Authentication::getName);
	}
}

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class Auditable {
	@CreatedDate      private Instant createdAt;
	@LastModifiedDate private Instant updatedAt;
	@CreatedBy        private String createdBy;
	@LastModifiedBy   private String updatedBy;
}

@Entity
public class Company extends Auditable { /* ... */ }
```

For a full change history (old and new values), use **Hibernate Envers** (`@Audited`) or change data capture (Debezium) instead.
</details>

<details><summary>

## [Medium] How should you manage database schema changes in Spring Boot?
</summary>

Use a **migration tool** – **Flyway** or **Liquibase** – instead of letting Hibernate change the schema in production.

- Migrations are versioned scripts in the repo (`src/main/resources/db/migration/V1__create_company.sql`, `V2__add_sector_index.sql`).
- On startup, Spring Boot runs pending migrations in order and records them in a history table, so every environment ends up with the same schema.
- Changes are reviewed in pull requests like any other code.

`spring.jpa.hibernate.ddl-auto` values:

| Value | Behaviour | Use |
|---|---|---|
| `none` | Do nothing | Production (with Flyway/Liquibase) |
| `validate` | Check entities match the schema; fail on startup if not | Production safety check |
| `update` | Alter the schema to match entities (never drops) | Local prototyping only |
| `create` / `create-drop` | Drop and recreate (on startup / also at shutdown) | Tests, demos |

Write migrations to be **backward compatible** (expand, then contract) so old and new app versions can run side by side during a rolling deployment – e.g. add a nullable column first, backfill, then make it required in a later release.
</details>

<details><summary>

## [Hard] How do you build dynamic search filters with Specifications?
</summary>

When filters are optional (search screens with many fields), building one method per combination doesn't scale. **Specifications** build a JPA Criteria query from reusable predicates.

```java
interface CompanyRepository extends JpaRepository<Company, Long>, JpaSpecificationExecutor<Company> { }

class CompanySpecs {
	static Specification<Company> sectorIs(String sector) {
		return (root, query, cb) -> sector == null ? null : cb.equal(root.get("sector"), sector);
	}
	static Specification<Company> fundedAbove(Long min) {
		return (root, query, cb) -> min == null ? null : cb.greaterThan(root.get("fundingUsd"), min);
	}
	static Specification<Company> nameContains(String q) {
		return (root, query, cb) -> q == null ? null : cb.like(cb.lower(root.get("name")), "%" + q.toLowerCase() + "%");
	}
}

Page<Company> result = repo.findAll(
	Specification.allOf(sectorIs(sector), fundedAbove(minFunding), nameContains(q)),   // null predicates are ignored
	PageRequest.of(page, 20, Sort.by("name")));
```

Alternatives: Querydsl (type-safe), Query by Example (simple equality filters), or Elasticsearch for full-text search.
</details>

<details><summary>

## [Hard] How should you implement equals() and hashCode() for JPA entities?
</summary>

Entities are tricky because the **database ID is null until the entity is persisted**, and Hibernate may hand you **proxies** (subclasses) instead of the real class.

Safe options:
1. **Use a natural/business key** that's unique and never changes (e.g. ISBN, email if immutable).
2. **Use the ID with a constant hashCode:**

```java
@Entity
public class Company {
	@Id @GeneratedValue private Long id;

	@Override
	public boolean equals(Object o) {
		if (this == o) return true;
		if (!(o instanceof Company other)) return false;       // instanceof works with proxies
		return id != null && id.equals(other.getId());         // unsaved entities are only equal to themselves
	}

	@Override
	public int hashCode() {
		return getClass().hashCode();                          // constant: stays the same before and after persist
	}

	public Long getId() { return id; }
}
```

Avoid: Lombok `@Data`/`@EqualsAndHashCode` on entities (includes all fields and lazy collections, can trigger lazy loading and infinite recursion), and a hashCode based on a generated ID (it changes after persist, breaking `HashSet`s). Also exclude lazy associations from `toString()`.
</details>
