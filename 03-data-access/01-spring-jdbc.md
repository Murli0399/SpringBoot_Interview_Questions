<details><summary>

## [Easy] How does Spring provide DAO support?
</summary>

Spring's **DAO support** makes data-access code consistent regardless of the technology (JDBC, JPA/Hibernate, etc.):

1. **`@Repository` + exception translation:** beans marked `@Repository` get their technology-specific exceptions (`SQLException`, `PersistenceException`) translated into Spring's unchecked **`DataAccessException`** hierarchy (`DuplicateKeyException`, `DataIntegrityViolationException`, …). Your services don't depend on JDBC- or JPA-specific errors.
2. **Templates:** `JdbcTemplate`, `NamedParameterJdbcTemplate` and `JdbcClient` handle connections, statements, result sets and cleanup.
3. **DAO support classes** (older style): `JdbcDaoSupport` / `NamedParameterJdbcDaoSupport` give subclasses a ready template from a `DataSource`.
4. **Declarative transactions** (`@Transactional`) that work the same across data-access technologies.
5. **Spring Data** (a separate project) goes further: you declare repository **interfaces** (`JpaRepository`, `MongoRepository`) and Spring generates the implementation, including queries derived from method names.

**Example:**

```java
@Repository                                  // enables exception translation to DataAccessException
public class JdbcCompanyDao {
    private final JdbcTemplate jdbc;
    public JdbcCompanyDao(JdbcTemplate jdbc) { this.jdbc = jdbc; }

    public int count() {
        return jdbc.queryForObject("SELECT COUNT(*) FROM company", Integer.class);
    }
}
```

</details>

<details><summary>

## [Easy] What annotation is used to mark DAO in Spring?
</summary>

In Spring, the @Repository annotation is commonly used to mark DAO (Data Access Object) classes.

The @Repository annotation is part of the Spring Framework and serves as a specialization of the @Component annotation. It is used to indicate that a particular class is a repository or a DAO component.

By applying the @Repository annotation to a class, you are instructing Spring to create a bean of that class and make it available for dependency injection and other Spring features. It also enables Spring's exception translation mechanism, which automatically translates low-level data access exceptions into more meaningful and unchecked Spring-specific exceptions.

Here's an example of how you can use the @Repository annotation to mark a DAO class:

```
@Repository
public class MyDAO {
    // DAO implementation
}
```

In the above example, the MyDAO class is annotated with @Repository, indicating that it is a DAO component. Spring will then detect and manage this class as a bean, allowing you to inject it into other components and utilize its data access functionality.

It's worth noting that while the @Repository annotation is commonly used for DAO classes, it is not mandatory. In general, any Spring-managed bean can be used as a DAO, but using the @Repository annotation helps convey the intent and takes advantage of the additional features provided by Spring for repositories.
</details>

<details><summary>

## [Easy] What tasks are performed by Spring JDBC?
</summary>

With plain JDBC you write the same boilerplate every time. Spring JDBC (`JdbcTemplate`/`JdbcClient`) does it for you:

| Task | Who does it |
|---|---|
| Define connection parameters / `DataSource` | You (or Spring Boot auto-configuration) |
| Open and close the connection, return it to the pool | **Spring** |
| Write the SQL | You |
| Prepare and execute the statement, bind parameters (positional or named) | **Spring** |
| Iterate over the `ResultSet` | **Spring** |
| Map each row to an object | You provide a `RowMapper` (or use `DataClassRowMapper`/`BeanPropertyRowMapper`) |
| Translate `SQLException` into `DataAccessException` | **Spring** |
| Join the current transaction (`@Transactional`) | **Spring** |
| Batch updates, stored procedure calls | **Spring** helpers (`batchUpdate`, `SimpleJdbcCall`) |

Connection **pooling** is not done by Spring JDBC; it comes from the `DataSource` implementation (HikariCP by default in Spring Boot).

**Example:**

```java
// You write only the SQL and the row mapping; Spring opens/closes connections,
// prepares statements, iterates ResultSets, translates SQLExceptions and joins transactions.
List<String> names = jdbc.queryForList("SELECT name FROM company WHERE sector = ?", String.class, "fintech");
```

</details>

<details><summary>

## [Easy] What is the difference between JDBC and Spring JDBC?
</summary>

| | Plain JDBC | Spring JDBC (`JdbcTemplate`, `JdbcClient`) |
|---|---|---|
| Boilerplate | Open connection, create statement, loop result set, close everything in `finally` | One method call; resources managed for you |
| Exceptions | Checked `SQLException` with vendor-specific error codes | Unchecked `DataAccessException` hierarchy, vendor-neutral (`DuplicateKeyException`, …) |
| Transactions | Manual `setAutoCommit(false)`, `commit()`, `rollback()` | Declarative `@Transactional`; the template joins the current transaction |
| Parameters | Positional `?` only | Positional and **named** (`:email`) parameters |
| Result mapping | Manual `rs.getString(...)` loops | `RowMapper`, `ResultSetExtractor`, `DataClassRowMapper` |
| Connection pooling | From the `DataSource` you choose | Same – Spring JDBC doesn't add pooling; Boot configures HikariCP |

Spring JDBC is still SQL-first and fast; it just removes the repetitive, error-prone parts.

**Example:**

```java
// Plain JDBC: lots of boilerplate
try (Connection c = ds.getConnection();
     PreparedStatement ps = c.prepareStatement("SELECT name FROM company WHERE id = ?")) {
    ps.setLong(1, 7);
    try (ResultSet rs = ps.executeQuery()) {
        if (rs.next()) name = rs.getString("name");
    }
} catch (SQLException e) { throw new RuntimeException(e); }

// Spring JDBC: one line
String name = jdbc.queryForObject("SELECT name FROM company WHERE id = ?", String.class, 7L);
```

</details>

<details><summary>

## [Easy] What are the advantages of JdbcTemplate in Spring?
</summary>

1. **No boilerplate:** connections, statements and result sets are opened and closed for you, so no resource leaks.
2. **Exception translation:** `SQLException` becomes Spring's unchecked `DataAccessException` hierarchy, consistent across databases.
3. **Easy parameter binding:** positional parameters, or named parameters via `NamedParameterJdbcTemplate`/`JdbcClient`.
4. **Simple result mapping:** `RowMapper`, `DataClassRowMapper` for records, `queryForObject`, `queryForList`.
5. **Transaction integration:** it automatically uses the connection bound to the current `@Transactional` transaction.
6. **Batch support:** `batchUpdate` for efficient bulk inserts/updates.
7. **Full control over SQL:** no ORM layer, so performance is predictable and you can use any database feature.
8. **Low-level access when needed:** callbacks (`ConnectionCallback`, `PreparedStatementCreator`) give you the raw JDBC objects.

It works with whatever `DataSource` you configure; the pooling itself comes from the pool (HikariCP in Spring Boot), not from `JdbcTemplate`.

**Example:**

```java
// Batch insert in one call - fast for bulk loads
jdbc.batchUpdate("INSERT INTO company(name, sector) VALUES (?, ?)",
    companies, 500,
    (ps, c) -> { ps.setString(1, c.name()); ps.setString(2, c.sector()); });
```

</details>

<details><summary>

## [Easy] Why is NamedParameterJdbcTemplate class used?
</summary>

The **NamedParameterJdbcTemplate** class is used to execute SQL queries and updates with named parameters instead of traditional positional parameters. It simplifies the process of parameter binding and enhances readability in SQL statements.

**Example:**

```java
String sql = "SELECT * FROM company WHERE sector = :sector AND country IN (:countries)";
MapSqlParameterSource params = new MapSqlParameterSource()
    .addValue("sector", "fintech")
    .addValue("countries", List.of("IN", "SG"));          // IN-list expanded automatically
List<Company> list = named.query(sql, params, new DataClassRowMapper<>(Company.class));
```

</details>

<details><summary>

## [Easy] How are records from a database fetched when using JdbcTemplate?
</summary>

When using JdbcTemplate in Spring, records from a database are fetched by executing SQL queries and mapping the result set to Java objects. The query() method of JdbcTemplate is commonly used for this purpose.

Here's a simplified explanation of the process:

### 1. Construct the SQL Query:
Create an SQL query that retrieves the desired records from the database. The query can include placeholders for parameters, which will be populated later.

### 2. Define a RowMapper:
Implement a RowMapper interface or use an existing implementation to map each row from the result set to a Java object. The RowMapper defines the mapping logic between the database columns and the fields or properties of the Java object.

### 3. Execute the Query:
Invoke the query() method of JdbcTemplate and pass in the SQL query, the RowMapper, and any necessary parameters. JdbcTemplate handles the execution of the query, parameter binding, and result set processing.

### 4. Process the Result:
JdbcTemplate retrieves the result set from the executed query and iterates over each row. For each row, it calls the RowMapper implementation to map the row to a Java object. The RowMapper extracts the relevant data from the result set and creates an instance of the Java object, which is then added to the resulting collection.

### 5. Return the Result:
The query() method returns the collection of mapped Java objects, which represents the fetched records from the database.

This process allows you to fetch records from a database using JdbcTemplate. By leveraging the power of SQL queries and RowMapper implementations, you can map database data to Java objects in a convenient and efficient manner.

**Example:**

```java
RowMapper<Company> mapper = (rs, rowNum) ->
    new Company(rs.getLong("id"), rs.getString("name"), rs.getString("sector"));

List<Company> all = jdbc.query("SELECT id, name, sector FROM company", mapper);
Company one = jdbc.queryForObject("SELECT id, name, sector FROM company WHERE id = ?", mapper, 7L);
// queryForObject throws EmptyResultDataAccessException when no row matches.
```

</details>

<details><summary>

## [Medium] Describe Spring DAO support classes.
</summary>

**Spring Framework (JDBC/ORM layer):**
- `JdbcTemplate`, `NamedParameterJdbcTemplate`, `JdbcClient` (Spring 6.1) – execute SQL with less code.
- `SimpleJdbcInsert`, `SimpleJdbcCall` – inserts and stored-procedure calls driven by metadata.
- `JdbcDaoSupport`, `NamedParameterJdbcDaoSupport` – base classes exposing a template (older style).
- `DataAccessException` hierarchy and `PersistenceExceptionTranslator` – consistent exceptions.

**Spring Data repositories:**
- `Repository` – marker interface.
- `CrudRepository` / `ListCrudRepository` – save, find, delete, count (the `List*` variants return `List` instead of `Iterable`).
- `PagingAndSortingRepository` / `ListPagingAndSortingRepository` – `findAll(Pageable)`, `findAll(Sort)`. Since **Spring Data 3** these **no longer extend** `CrudRepository`; combine them yourself or use `JpaRepository`.
- **`JpaRepository`** – extends `ListCrudRepository`, `ListPagingAndSortingRepository` and `QueryByExampleExecutor`, and adds JPA-specific methods: `flush()`, `saveAndFlush()`, `deleteAllInBatch()`, `getReferenceById()`.
- **Derived query methods** – `findByEmailAndStatus(...)` turned into queries from the method name; `@Query` for JPQL/native SQL.
- `JpaRepositoryFactory` – creates the proxy implementations behind your interfaces.

**Example:**

```java
public interface CompanyRepository extends JpaRepository<Company, Long> {
    List<Company> findBySectorOrderByNameAsc(String sector);              // derived query
    Page<Company> findByCountry(String country, Pageable pageable);       // paging
    @Query("select c from Company c where c.fundingUsd > :min")
    List<Company> funded(@Param("min") long min);                         // JPQL
}
```

</details>

<details><summary>

## [Medium] What is Spring Data JDBC?
</summary>

**Spring Data JDBC** is a Spring Data module that maps objects to tables over plain JDBC, deliberately **simpler than JPA**:

- **Repositories** like other Spring Data modules: `CrudRepository`, derived query methods, `@Query` with SQL.
- **Simple mapping:** classes map to tables and fields to columns by naming convention, with annotations like `@Table`, `@Id`, `@Column`. Records and immutable objects work well.
- **Aggregate-oriented (DDD):** you load and save a whole **aggregate root** with its owned children; references to other aggregates are by ID.
- **No JPA "magic":** no lazy loading, no persistence context or first-level cache, no dirty checking. `save()` issues SQL immediately, and what you see is what runs.
- Works with Spring's transactions (`@Transactional`).

Choose it when you want predictable SQL and simple domain models without Hibernate's complexity. Choose JPA when you need rich relationship mapping, lazy loading or caching. (For reactive stacks there's Spring Data R2DBC.)

**Example:**

```java
@Table("company")
record Company(@Id Long id, String name, String sector) {}   // no lazy loading, no dirty checking

interface CompanyRepository extends CrudRepository<Company, Long> {
    List<Company> findBySector(String sector);
}
```

</details>

<details><summary>

## [Medium] Name the classes in Spring JDBC API?
</summary>

Core classes and interfaces in `org.springframework.jdbc`:

- **`JdbcTemplate`** – central class: `query`, `queryForObject`, `update`, `batchUpdate`, `execute`.
- **`NamedParameterJdbcTemplate`** – supports `:named` parameters. It **wraps** a `JdbcTemplate` (it's not a subclass).
- **`JdbcClient`** (Spring 6.1) – modern fluent API combining positional and named parameters.
- **`SimpleJdbcInsert`**, **`SimpleJdbcCall`** – insert rows and call stored procedures using database metadata.
- **`SqlParameterSource`** with implementations **`MapSqlParameterSource`** and **`BeanPropertySqlParameterSource`**; **`SqlParameterSourceUtils`** creates batches of them.
- **`RowMapper`** (row → object), **`ResultSetExtractor`** (whole result set → anything), **`RowCallbackHandler`** (process rows one by one).
- **`PreparedStatementCreator`**, **`PreparedStatementSetter`**, **`BatchPreparedStatementSetter`**, **`CallableStatementCreator`** – callbacks for custom statement handling.
- **`KeyHolder` / `GeneratedKeyHolder`** – read auto-generated keys after inserts.
- **`DataSourceTransactionManager`** / `JdbcTransactionManager` – transaction managers for JDBC.
- **`SqlQuery`, `SqlUpdate`, `StoredProcedure`** (package `org.springframework.jdbc.object`) – older "RDBMS operation objects" that model one reusable SQL statement as a thread-safe object; they're used on their own, not passed to `JdbcTemplate`.

**Example:**

```java
JdbcTemplate jdbc = new JdbcTemplate(dataSource);
NamedParameterJdbcTemplate named = new NamedParameterJdbcTemplate(dataSource);
SimpleJdbcInsert insert = new SimpleJdbcInsert(dataSource).withTableName("company").usingGeneratedKeyColumns("id");
Number id = insert.executeAndReturnKey(Map.of("name", "Acme", "sector", "fintech"));
JdbcClient client = JdbcClient.create(dataSource);   // Spring 6.1+ fluent API
```

</details>

<details><summary>

## [Medium] What is the difference between a RowMapper and ResultSetExtractor?
</summary>

Both RowMapper and ResultSetExtractor are interfaces provided by the Spring JDBC framework for mapping rows from a ResultSet to Java objects. However, they differ in their use cases and the way they handle the result set data.

Here are the key differences between RowMapper and ResultSetExtractor:

### 1. Purpose:
- **RowMapper:** The RowMapper interface is used to map individual rows from a ResultSet to a single Java object. It defines a single method, mapRow(), which is called for each row in the result set. The RowMapper implementation is responsible for extracting the relevant data from the result set and creating an instance of the Java object.

- **ResultSetExtractor:** The ResultSetExtractor interface is used to process the entire ResultSet and extract data in a customized way. It is typically used when there is a need to aggregate or transform the result set data into a single object or perform complex data extraction logic that goes beyond mapping individual rows.

### 2. Method Signature:
- **RowMapper:** The RowMapper interface has a single method, mapRow(), which takes two parameters: the current ResultSet and the row number. It returns a mapped object for the current row.

- **ResultSetExtractor:** The ResultSetExtractor interface has a single method, extractData(), which takes a single parameter: the current ResultSet. It returns the object representing the extracted data from the entire ResultSet.

### 3. Usage:
- **RowMapper:** RowMapper is commonly used in scenarios where each row in the ResultSet corresponds to a single Java object. It is suitable for mapping simple one-to-one relationships between the database and Java objects.

- **ResultSetExtractor:** ResultSetExtractor is used when there is a need to perform more complex operations on the entire result set, such as aggregating data from multiple rows, performing calculations, or transforming the data into a different representation. It provides greater flexibility to process the result set in a customized manner.

In summary, RowMapper is used for mapping individual rows from a ResultSet to Java objects, while ResultSetExtractor is used for more complex processing of the entire ResultSet to extract customized data. RowMapper is suitable for most simple mapping scenarios, while ResultSetExtractor offers more flexibility for advanced result set handling.

**Example:**

```java
// RowMapper: one row -> one object
List<String> names = jdbc.query("SELECT name FROM company", (rs, i) -> rs.getString("name"));

// ResultSetExtractor: whole ResultSet -> any structure (e.g. group rows)
Map<String, List<String>> bySector = jdbc.query("SELECT sector, name FROM company", rs -> {
    Map<String, List<String>> m = new HashMap<>();
    while (rs.next()) m.computeIfAbsent(rs.getString("sector"), k -> new ArrayList<>()).add(rs.getString("name"));
    return m;
});
```

</details>

<details><summary>

## [Medium] Explain Spring’s exception handling support with DataAccessException.
</summary>

Spring provides exception handling support for database operations through the DataAccessException hierarchy. It is a runtime exception that serves as a wrapper for underlying database-specific exceptions.

Here's a brief explanation of Spring's exception handling support with **DataAccessException**:

### 1. Exception Translation:
- Spring's DataAccessException hierarchy acts as a translation layer for low-level database exceptions, such as SQLExceptions, into more meaningful and unchecked exceptions. This abstraction shields the application from dealing directly with database-specific exceptions.

### 2. Consistent Exception Handling:
- By using DataAccessException, developers can handle exceptions consistently across different database technologies. Spring automatically translates the native exceptions to appropriate subclasses of DataAccessException, providing a unified approach to exception handling.
### 3. Simplified Error Handling:
- DataAccessException provides a clear and consistent way to handle database-related errors. It offers a comprehensive set of subclasses, such as DuplicateKeyException, DataIntegrityViolationException, and TransientDataAccessResourceException, which can be caught individually for specific error scenarios.

### 4. Custom Exception Translation:
- Spring allows customization of exception translation by implementing the PersistenceExceptionTranslator interface. Developers can provide their own implementation to translate specific database exceptions into Spring's DataAccessException hierarchy, ensuring consistent error handling throughout the application.

Overall, Spring's exception handling support with DataAccessException simplifies database error handling by providing a standardized and unified approach. It abstracts away the underlying database-specific exceptions and offers a clear and consistent way to handle database-related errors in a Spring application.

**Example:**

```java
try {
    jdbc.update("INSERT INTO users(email) VALUES (?)", email);
} catch (DuplicateKeyException e) {          // vendor-neutral, unchecked
    throw new ConflictException("Email already registered");
} catch (DataAccessException e) {            // root of Spring's data exception hierarchy
    throw new ServiceUnavailableException("Database error", e);
}
```

</details>

<details><summary>

## [Medium] What is SQLExceptionTranslator?
</summary>

The SQLExceptionTranslator is an interface in Spring JDBC that provides a mechanism for translating SQLExceptions into more meaningful exceptions defined by Spring's DataAccessException hierarchy. It acts as a bridge between the low-level database exceptions and the higher-level exception handling in Spring.

Here's a brief explanation of the SQLExceptionTranslator:

### 1. Exception Translation:
The SQLExceptionTranslator interface allows for translating SQLExceptions thrown by the underlying database driver into Spring's DataAccessException hierarchy. It provides a consistent and unified approach to handling database exceptions across different database vendors.

### 2. Consistent Exception Handling:
By implementing the SQLExceptionTranslator interface, developers can define their own translation logic for specific SQLExceptions. This ensures that the application can handle database exceptions in a consistent and meaningful way, regardless of the underlying database technology.

### 3. Customized Exception Handling:
The SQLExceptionTranslator interface allows for customization of the exception translation process. Developers can provide their own implementation to map specific SQLExceptions to appropriate subclasses of DataAccessException, thereby enabling more fine-grained control over exception handling.

### 4. Integration with Spring JDBC:
SQLExceptionTranslator is used in conjunction with Spring's JDBC operations, such as JdbcTemplate or NamedParameterJdbcTemplate. It is automatically invoked when a SQLException is encountered during database operations, allowing for the translation and throwing of the appropriate Spring exception.

In summary, the SQLExceptionTranslator interface in Spring JDBC provides a way to translate low-level SQLExceptions into more meaningful exceptions defined by Spring's DataAccessException hierarchy. It ensures consistent and unified exception handling for database-related errors in a Spring application.

**Example:**

```java
// MySQL error 1062 / SQLState 23000 -> DuplicateKeyException
// Deadlock error 1213 -> CannotAcquireLockException / DeadlockLoserDataAccessException
SQLExceptionTranslator translator = new SQLErrorCodeSQLExceptionTranslator(dataSource);
DataAccessException ex = translator.translate("insert user", sql, sqlException);
```

</details>

<details><summary>

## [Medium] What are JdbcTemplate callback interfaces?
</summary>

Callbacks let you plug your own logic into specific steps while `JdbcTemplate` handles resources and exceptions:

| Interface | Your code does |
|---|---|
| `PreparedStatementCreator` | Creates the `PreparedStatement` yourself (e.g. to request generated keys) |
| `PreparedStatementSetter` | Binds parameter values (`setValues`) |
| `BatchPreparedStatementSetter` | Binds values for each item in a batch (`setValues`, `getBatchSize`) |
| `RowMapper<T>` | Maps one row to an object (`mapRow`) |
| `RowCallbackHandler` | Processes each row without returning a list (`processRow`) – good for streaming large results |
| `ResultSetExtractor<T>` | Processes the whole result set into any structure (`extractData`) |
| `CallableStatementCreator` / `CallableStatementCallback` | Creates / works with a `CallableStatement` for stored procedures |
| `ConnectionCallback<T>` / `StatementCallback<T>` | Low-level access to the `Connection` / `Statement` |

All of them are functional interfaces, so you usually pass lambdas.

**Example:**

```java
jdbc.query("SELECT id, name FROM company", (RowCallbackHandler) rs -> {
    exporter.write(rs.getLong("id"), rs.getString("name"));   // stream rows without building a list
});

jdbc.execute((ConnectionCallback<Void>) con -> {
    con.setReadOnly(true);                                    // low-level access when needed
    return null;
});
```

</details>
