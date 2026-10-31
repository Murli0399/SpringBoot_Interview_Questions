<details><summary>

## What is Spring Data?
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

## What is JPA?
</summary>

The Java Persistence API (JPA) is a Java specification that allows developers to manage data between Java objects and relational databases. JPA acts as a bridge between object-oriented domain models and relational database systems. It was first released in 2006.

JPA is an object-relational mapping (ORM) framework. It allows developers to work directly with objects instead of using SQL statements. JPA provides a platform for developers to construct database-driven Java programs using object-oriented semantics.

JPA was defined as part of the EJB 3.0 specification. It replaced the EJB 2 CMP Entity Beans specification.

JPA is a specification and doesn't perform any operations by itself. To use JPA, you need to configure a datastore connector to connect to your chosen database. You also need to include and configure a JPA provider, which is a framework such as Hibernate or EclipseLink.

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

## What is the difference between JPA and Spring Data JPA?
</summary>

JPA (Java Persistence API) is a Java standard for managing relational data in Java EE applications. It provides an abstraction layer over relational databases, allowing developers to access data without having to worry about the underlying database implementation.

Spring Data JPA is an open-source Java library that provides a simplified API for using JPA. It provides a number of features that make it easier to use JPA, including:
- Automatic repository creation
- Support for paging and sorting
- Query creation
- Entity validation
- Transaction management

Spring Data JPA can be used with any JPA implementation, including Hibernate and EclipseLink.

The main difference between JPA and Spring Data JPA is that Spring Data JPA provides a simplified API for using JPA. This makes it easier to develop applications that use JPA, especially for developers who are not familiar with JPA.

Here is a table that summarizes the key differences between JPA and Spring Data JPA:

|Feature|JPA|Spring Data JPA|
|-------|---|---------------|
|Abstraction layer|Provides an abstraction layer over relational databases|Provides a simplified API for using JPA|
|Repository creation|Does not automatically create repositories|Automatically creates repositories|
|Paging and sorting support|Does not support paging and sorting|Supports paging and sorting|
|Query creation|Does not support query creation|Supports query creation|
|Entity validation|Does not support entity validation|Supports entity validation|
|Transaction management|Does not support transaction management|Supports transaction management|

Overall, Spring Data JPA is a more powerful and user-friendly alternative to JPA. It provides a number of features that make it easier to develop applications that use JPA.
</details>
<details><summary>

## What is the difference between CrudRepository and JpaRepository?
</summary>

CrudRepository and JpaRepository are both interfaces in the Spring Data JPA library. CrudRepository provides CRUD (Create, Read, Update, Delete) operations on a repository, while JpaRepository provides additional methods for working with JPA (Java Persistence API) persistence contexts.

The main differences between CrudRepository and JpaRepository are:

- JpaRepository extends CrudRepository.
This means that any method that is available on CrudRepository is also available on JpaRepository.

When should you use CrudRepository?

You should use CrudRepository if you only need to perform basic CRUD operations on a repository. For example, you might use CrudRepository to save, retrieve, update, or delete JPA entities.

When should you use JpaRepository?

You should use JpaRepository if you need to perform more advanced operations on a repository, such as pagination, sorting, flushing the persistence context, or deleting records in a batch. For example, you might use JpaRepository to paginate the results of a query, flush the persistence context, or delete multiple JPA entities in a single operation.

In general, you should use the most specific interface that meets your needs. If you only need to perform basic CRUD operations, use CrudRepository. If you need to perform more advanced operations, use JpaRepository.

**Example:**

```java
interface A extends CrudRepository<User, Long> { }        // save, findById, findAll (Iterable), delete
interface B extends JpaRepository<User, Long> { }         // + paging/sorting, List returns, flush, saveAllAndFlush, batch deletes

Page<User> page = userRepo.findAll(PageRequest.of(0, 20, Sort.by("createdAt").descending()));
```

</details>
<details><summary>

## Which No SQL databases does Spring support?
</summary>

Spring Data supports several NoSQL databases, including:

MongoDB, Neo4j, Redis, Gemfire, Solr, Elasticsearch, Cassandra, Couchbase, LDAP.

NoSQL databases are a classification of database systems that do not support the SQL standard. They can be document databases, key-value databases, wide-column stores, or graph databases. MongoDB is a popular document-based database that is designed to handle unstructured and semi-structured data.

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

## Which ORMs are supported by Spring?
</summary>

Spring supports most popular Object Relational Mapping (ORM) frameworks, including:
- Hibernate
- JDO (Java Data Objects)
- iBATIS
- JPA (Java Persistence API)
- Oracle Toplink
- OJB

Spring provides an API to easily integrate Spring with these frameworks.

Using the Spring Framework to create ORM DAOs can make testing easier.

**Example:**

```java
// Hibernate is the default JPA provider in Spring Boot (spring-boot-starter-data-jpa).
// Others: EclipseLink (JPA), plus non-JPA options like MyBatis, jOOQ, and Spring Data JDBC.
@Bean
JdbcClient jdbcClient(DataSource ds) { return JdbcClient.create(ds); }   // lightweight SQL option
```

</details>
<details><summary>

## What is Hibernate?
</summary>

Hibernate is a Java framework that helps developers create applications that interact with databases. It's an open-source, lightweight, Object-Relational Mapping (ORM) tool. Hibernate maps Java classes to database tables and Java data types to SQL data types. It also provides data query and recovery facilities.

Hibernate has several advantages:
- Increased performance
- Database independence
- Auto DDL operations

Hibernate provides an abstraction layer, so developers don't have to worry about implementations. Hibernate does the implementations internally, such as:
- Establishing a connection with the database
- Writing query to perform CRUD operations

Hibernate implements the specifications of JPA (Java Persistence API) for data persistence.

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

## What are the different types of transaction management supported by Spring?
</summary>

Spring supports two types of transaction management: declarative and programmatic.
- **Declarative transaction management**

is the recommended approach, as it is simpler and easier to use. With declarative transaction management, the programmer annotates methods with the @Transactional annotation to indicate that they should be executed in a transaction. Spring will automatically manage the transaction, ensuring that it is started, committed, or rolled back as needed.

- **Programmatic transaction management**

allows the programmer to explicitly manage transactions. This is typically used in more complex cases where the programmer needs more control over the transaction. Spring provides a number of classes for programmatic transaction management, including the TransactionTemplate and the PlatformTransactionManager.

In addition to these two types of transaction management, Spring also provides a number of features for managing transactions, such as:

- **Transaction rollback-only**

allows the programmer to mark a transaction as rollback-only. This means that the transaction will always be rolled back, even if there are no exceptions.

- **Transaction propagation**

allows the programmer to control how transactions are propagated across method calls. For example, a programmer can specify that transactions should be propagated to all method calls in the same class, or that they should be propagated to all method calls in the same transaction.

- **Transaction isolation**

allows the programmer to control how transactions are isolated from each other. For example, a programmer can specify that transactions should be isolated from dirty reads, or that they should be isolated from dirty writes.

By using Spring's transaction management features, developers can easily create robust and efficient applications.

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

## Which transaction management type is preferred in Spring?
</summary>

Declarative transaction management is preferred in Spring. It is simpler to use and more maintainable than programmatic transaction management. With declarative transaction management, developers can annotate their code with @Transactional to specify that it should be executed within a transaction. Spring will then automatically manage the transaction, including starting, committing, or rolling it back. This frees developers from having to worry about the details of transaction management, allowing them to focus on their business logic.

Programmatic transaction management is still available in Spring, but it is discouraged for new code. It is more complex to use and more difficult to maintain than declarative transaction management. Additionally, programmatic transaction management can lead to errors if it is not used correctly.

Here are some of the advantages of declarative transaction management over programmatic transaction management:
- It is simpler to use. Developers do not need to worry about the details of transaction management.
- It is more maintainable. Changes to the transaction management configuration can be made without having to modify the code.
- It is more reliable. Spring automatically manages the transaction, reducing the risk of errors.

If you are developing a new Spring application, you should use declarative transaction management. It is simpler, more maintainable, and more reliable than programmatic transaction management.

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

## What are some benefits of using Spring Transactions?
</summary>

## Benefits of Using Spring Transactions

Spring Transactions provides a comprehensive abstraction for transaction management that delivers the following benefits:

### A consistent programming model across different transaction APIs

, such as Java Transaction API (JTA), JDBC, Hibernate, and the Java Persistence API (JPA). This allows developers to write code once and have it benefit from different transaction management strategies in different environments.

### Support for declarative transaction management.

This allows developers to specify the transactional behavior of a method using XML or annotations, rather than having to manually manage transactions using low-level APIs.

### A simpler API for programmatic transaction management

than complex transaction APIs, such as JTA. This makes it easier for developers to integrate Spring Transactions with existing codebases.

### Excellent integration with Spring's data access abstractions.

This allows developers to use Spring Transactions with a variety of data access technologies, such as JDBC, Hibernate, and JPA.

Overall, Spring Transactions is a powerful and flexible transaction management framework that can help developers write more robust and efficient code.

Here are some additional benefits of using Spring Transactions:

### Improved performance.

Spring Transactions can improve the performance of your applications by managing transactions automatically. This can free up developers to focus on other tasks, such as writing business logic.

### Increased reliability.

Spring Transactions can help to make your applications more reliable by ensuring that all database operations are performed atomically. This helps to prevent data corruption and other errors.

### Enhanced security.

Spring Transactions can help to enhance the security of your applications by providing a consistent way to manage transactions. This can help to protect your data from unauthorized access.

If you are looking for a powerful and flexible transaction management framework, Spring Transactions is a great option. It can help you to improve the performance, reliability, and security of your applications.

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

## What happens when the @Transactional annotation is used on a method?
</summary>

When the @Transactional annotation is used on a method, it specifies that the method should be executed within a database transaction. This means that all changes to the database made by the method will be either committed or rolled back as a single unit.

The @Transactional annotation has a number of attributes that can be used to customize how the transaction is managed. These attributes include:

- propagation: Specifies the propagation behavior of the transaction. This determines how the transaction is affected if it is called from within another transaction.

- isolation: Specifies the isolation level of the transaction. This determines how the transaction is isolated from other transactions.

- timeout: Specifies the timeout for the transaction. This determines how long the transaction is allowed to run before it is rolled back.

- readOnly: Specifies whether the transaction is read-only. This determines whether the transaction is allowed to make changes to the database.

- rollbackFor: Specifies a list of exceptions that should cause the transaction to be rolled back.

The default values for all of these attributes are:

propagation: REQUIRED, isolation: READ_COMMITTED, timeout: -1 (no timeout, readOnly: false, rollbackFor: Exception.

The @Transactional annotation can be used on interfaces, classes, and methods. When it is used on an interface or class, it applies to all methods in the interface or class. When it is used on a method, it applies only to that method.

The @Transactional annotation is a powerful tool that can be used to ensure that changes to the database are made in a consistent and reliable manner.

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
// See 03-jpa-performance-and-transactions.md for propagation and when @Transactional is ignored.
```

</details>
<details><summary>

## What is the difference between JdbcTemplate and JPA?
</summary>

JdbcTemplate and JPA are two different ways of interacting with databases from Java.

JdbcTemplate is a low-level API that allows you to execute SQL statements directly. This gives you a lot of flexibility, but it also means that you have to write more code and handle more details yourself.

JPA is a higher-level API that provides an object-relational mapping (ORM) layer. This means that you can interact with your database using Java objects instead of SQL statements. JPA also provides a number of features that can make your life easier, such as entity state management and transaction management.

Here is a table that summarizes the key differences between JdbcTemplate and JPA:

|Feature|JdbcTemplate|JPA|
|-------|------------|---|
|Abstraction level|Low|High|
|Flexibility|High|Low|
|Boilerplate code|High|Low|
|Features|Few|Many|
|Learning curve|Steep|Gradual|

In general, JdbcTemplate is a good choice if you need a lot of flexibility and control over your database access. JPA is a good choice if you want to make your code easier to write and maintain.
</details>
