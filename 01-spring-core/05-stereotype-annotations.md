# Stereotype annotations

<details><summary>
  
### [Easy] What is the purpose of @Component annotation?
</summary>

The @Component annotation is a fundamental part of the Spring Framework, which is a popular Java-based framework for building enterprise applications. The purpose of the @Component annotation is to indicate that a class is a component or a bean in the Spring context.

When a class is annotated with @Component, it is automatically detected and registered by the Spring container during the component scanning process. The Spring container then creates an instance of the class and manages its lifecycle, allowing it to be injected into other parts of the application.

By using the @Component annotation, you can define reusable and manageable components in your application. These components can be automatically wired together, configured, and controlled by the Spring container, enabling dependency injection and inversion of control (IOC) principles.

The @Component annotation is a generic stereotype annotation, which means it serves as the base annotation for more specific stereotypes like @Service, @Repository, and @Controller. These specialized annotations are typically used to provide additional semantic meaning to the components based on their roles in the application architecture.

In summary, the @Component annotation is used to mark a class as a Spring component, allowing it to be managed and utilized by the Spring container for dependency injection and other application-wide features.

**Example:**

```java
@Component
public class SlugGenerator {
    public String slug(String name) { return name.toLowerCase().replaceAll("[^a-z0-9]+", "-"); }
}

@Service
class CompanyService {
    CompanyService(SlugGenerator slugs) { }   // injected because @Component made it a bean
}
```

</details>

<details><summary>
  
### [Easy] What is the difference between @Component, @Service, @Repository, and @Controller?
</summary>

All four mark a class for **component scanning**; `@Service`, `@Repository` and `@Controller` are meta-annotated with `@Component`. The differences are in meaning and in a few real technical effects:

| Annotation | Layer | Technical effect beyond `@Component` |
|---|---|---|
| `@Component` | Any generic bean | – |
| `@Service` | Business logic | None today (just intent; useful for readability and AOP pointcuts) |
| `@Repository` | Data access | Enables **persistence exception translation**: `PersistenceExceptionTranslationPostProcessor` converts JDBC/JPA exceptions into Spring's `DataAccessException` hierarchy |
| `@Controller` | Web (MVC) | Detected by Spring MVC as a **request handler** class; methods return view names unless `@ResponseBody` is used |
| `@RestController` | REST APIs | `@Controller` + `@ResponseBody` on every method (return values become JSON) |

Using the right one documents your architecture and lets you target layers with AOP (e.g. log all `@Service` methods).

**Example:**

```java
@Repository                 // data access; adds DataAccessException translation
class JdbcOrderRepository { }

@Service                    // business logic; marker for readability/AOP pointcuts
class OrderService { }

@RestController             // web layer; handles HTTP requests
class OrderController { }

@Component                  // generic bean that fits none of the above
class Clocks { }
```

</details>

<details><summary>
  
### [Easy] Why is @Primary annotation used?
</summary>

The @Primary annotation is used in the Spring Framework to indicate a primary bean when multiple beans of the same type are present in the application context. It is used to resolve ambiguity when autowiring or injecting dependencies.

When you have more than one bean of the same type, Spring may encounter a situation where it cannot determine which bean to inject into a particular dependency. In such cases, you can use the @Primary annotation to specify a primary bean.

Here's how it works:

### Multiple Beans of the Same Type:
Let's say you have multiple beans implementing the same interface or extending the same class. For example, you may have two implementations of an interface ExampleInterface: BeanA and BeanB.

```
public interface ExampleInterface {
    // ...
}

@Component
@Primary
public class BeanA implements ExampleInterface {
    // ...
}

@Component
public class BeanB implements ExampleInterface {
    // ...
}
```

### Autowiring with @Autowired:
If you have a dependent class that requires an instance of ExampleInterface, you can use the @Autowired annotation to inject it.

```
@Component
public class DependentClass {

    @Autowired
    private ExampleInterface exampleInterface;
    
    // ...
}
```

### Resolving Ambiguity with @Primary:
In this scenario, since you have multiple beans of the same type, Spring may not know which one to inject. To specify the primary bean, you can use the @Primary annotation on BeanA.

```
@Component
@Primary
public class BeanA implements ExampleInterface {
    // ...
}
```

Now, when Spring encounters the @Autowired annotation for ExampleInterface in the DependentClass, it will inject BeanA because it is marked as the primary bean. If you remove the @Primary annotation from BeanA, Spring will throw an exception indicating that it cannot determine which bean to inject.

The @Primary annotation simplifies the resolution of bean dependencies in situations where there are multiple beans of the same type. It provides a clear indication to Spring about the primary bean to be used for injection.
</details>

<details><summary>
  
### [Easy] Why is @Qualifier annotation used?
</summary>

`@Qualifier` picks **one specific bean** when several beans of the same type exist. It is placed at the injection point (constructor parameter, field or setter), usually with the **bean name**.

```java
public interface PaymentGateway { }
@Component("razorpay") class RazorpayGateway implements PaymentGateway { }
@Component("stripe")   class StripeGateway implements PaymentGateway { }

@Service
class CheckoutService {
	private final PaymentGateway gateway;
	CheckoutService(@Qualifier("razorpay") PaymentGateway gateway) {   // inject RazorpayGateway
		this.gateway = gateway;
	}
}
```

You can also put `@Qualifier("someTag")` on the bean class/`@Bean` method to give it a qualifier value other than its name, or create your own qualifier annotation (e.g. a `@Fast` annotation that is meta-annotated with `@Qualifier`). Without `@Qualifier` or `@Primary`, injecting an ambiguous type fails with `NoUniqueBeanDefinitionException`.

</details>

<details><summary>
  
### [Easy] Why is the @Required annotation used?
</summary>

`@Required` was placed on **setter methods** to tell Spring that the property must be injected; if it wasn't, the container failed at startup (checked by `RequiredAnnotationBeanPostProcessor`).

History: introduced in **Spring 2.0**, **deprecated in Spring 5.1**, and **removed in Spring 6**. You won't see it in modern code.

Modern replacement: use **constructor injection** for mandatory dependencies. A missing dependency then fails at startup automatically, and the field can be `final`. (`@Autowired` is also required by default; use `required = false`, `Optional<T>` or `ObjectProvider<T>` for optional dependencies.)

**Example:**

```java
// @Required was deprecated in Spring 5.1 and removed in Spring 6.
// Modern equivalent: make the dependency a constructor argument.
@Service
class ReportService {
    private final ReportRepository repo;            // required: app won't start without it
    ReportService(ReportRepository repo) { this.repo = Objects.requireNonNull(repo); }
}
```

</details>

<details><summary>
  
### [Easy] What is the purpose of @Autowired annotation?
</summary>

The @Autowired annotation is an important annotation in the Spring Framework that is used for automatic dependency injection. Its purpose is to automatically wire or inject dependencies into a Spring-managed bean.

When a class is annotated with @Autowired, Spring will try to find a matching dependency of the required type from the application context and inject it into the annotated field, constructor, or method parameter.

The @Autowired annotation can be used in the following ways:

### 1. Field Injection:

```
@Autowired
private DependencyType dependency;
```

### 2. Constructor Injection:

```
private DependencyType dependency;

@Autowired
public MyClass(DependencyType dependency) {
    this.dependency = dependency;
}
```

### 3. Setter Method Injection:

```
private DependencyType dependency;

@Autowired
public void setDependency(DependencyType dependency) {
    this.dependency = dependency;
}
```

When the @Autowired annotation is used, Spring will search for a bean of the required type in the application context. If exactly one matching bean is found, it will be injected into the annotated field, constructor parameter, or setter method parameter. If multiple beans of the same type are available, additional mechanisms such as @Qualifier can be used to specify the desired bean.

The @Autowired annotation greatly simplifies the process of dependency injection in Spring applications. Instead of manually wiring dependencies, Spring automatically resolves and injects the required dependencies, allowing for loosely coupled and modular code. It promotes the principles of inversion of control (IoC) and reduces the need for explicit configuration.
</details>

<details><summary>
  
### [Medium] Which annotations takes precedence: @Primary or @Qualifier?
</summary>

**`@Qualifier` wins.** Resolution works like this:

1. Spring finds all beans of the required type.
2. If the injection point has a `@Qualifier`, only beans matching that qualifier are considered. If **none matches**, startup fails with `NoSuchBeanDefinitionException` – Spring does **not** fall back to the `@Primary` bean.
3. If there's no `@Qualifier` and several candidates remain, the one marked **`@Primary`** is chosen.
4. Otherwise Spring tries to match the parameter/field **name** to a bean name.
5. If it's still ambiguous: `NoUniqueBeanDefinitionException`.

So: `@Primary` sets the **default** for a type; `@Qualifier` makes an **explicit choice** at a particular injection point and overrides the default.

**Example:**

```java
interface Sender { }
@Component @Primary class EmailSender implements Sender { }
@Component("smsSender") class SmsSender implements Sender { }

@Service
class Alerts {
    Alerts(Sender defaultSender,                              // EmailSender (@Primary)
           @Qualifier("smsSender") Sender sms) { }            // SmsSender: @Qualifier wins
}
```

</details>

<details><summary>
  
### [Medium] What is the difference between @Inject and @Autowired in Spring? Which one to use under which condition?
</summary>

| | `@Autowired` | `@Inject` |
|---|---|---|
| Source | Spring (`org.springframework.beans.factory.annotation`) | JSR-330 / Jakarta Dependency Injection (`jakarta.inject`; Spring 7 dropped `javax.inject`) |
| Extra dependency | None | Needs the `jakarta.inject-api` JAR |
| Optional injection | `@Autowired(required = false)` | No `required` attribute – use `Optional<T>` or `Provider<T>` |
| Choosing by name | `@Qualifier("name")` | `@Named("name")` (Spring's `@Qualifier` also works) |
| Resolution | By type, then qualifiers/primary, then name | Same algorithm in Spring |

In a Spring application they behave essentially the same. Use `@Autowired` (or simply constructor injection with no annotation) in Spring projects; use `@Inject` if you want code that's portable to other DI containers (Jakarta EE CDI, Guice, Micronaut, Dagger).

There's also `@Resource` (`jakarta.annotation`), which resolves **by name first**, then by type.

**Example:**

```java
import jakarta.inject.Inject;      // JSR-330 standard (needs jakarta.inject dependency)
import org.springframework.beans.factory.annotation.Autowired;

@Service
class A {
    @Inject A(Repo repo) { }                      // portable across DI frameworks
}

@Service
class B {
    @Autowired(required = false)                  // Spring-only; supports required=false
    void setCache(Cache cache) { }
}
```

</details>
