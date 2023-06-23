<details><summary>
  
### What is the purpose of @Component annotation?
</summary>
The @Component annotation is a fundamental part of the Spring Framework, which is a popular Java-based framework for building enterprise applications. The purpose of the @Component annotation is to indicate that a class is a component or a bean in the Spring context.

When a class is annotated with @Component, it is automatically detected and registered by the Spring container during the component scanning process. The Spring container then creates an instance of the class and manages its lifecycle, allowing it to be injected into other parts of the application.

By using the @Component annotation, you can define reusable and manageable components in your application. These components can be automatically wired together, configured, and controlled by the Spring container, enabling dependency injection and inversion of control (IOC) principles.

The @Component annotation is a generic stereotype annotation, which means it serves as the base annotation for more specific stereotypes like @Service, @Repository, and @Controller. These specialized annotations are typically used to provide additional semantic meaning to the components based on their roles in the application architecture.

In summary, the @Component annotation is used to mark a class as a Spring component, allowing it to be managed and utilized by the Spring container for dependency injection and other application-wide features.
</details>
<details><summary>
  
### What is the difference between @Component, @Service, @Repository, and @Controller?
</summary>
In the Spring Framework, @Component, @Service, @Repository, and @Controller are stereotype annotations that provide additional semantic meaning to the components in your application. Although all of them are derived from the @Component annotation, they have distinct purposes and are typically used in different layers of an application.

### 1. @Component:

- It is a generic stereotype annotation.
- It is used to mark a class as a Spring component or bean.
- It can be used for any general-purpose component in the application.
- It is typically used for utility classes, helper classes, or generic components.

### 2. @Service:

- It is a specialization of @Component.
- It is used to mark a class as a service component.
- It represents the service layer of an application, which typically contains the business logic.
- It encapsulates the application's business operations and provides services to other layers or components.
- It often orchestrates the interaction between multiple repositories or external systems.

### 3. @Repository:

- It is also a specialization of @Component.
- It is used to mark a class as a repository component.
- It represents the data access layer of an application.
- It typically encapsulates the logic to interact with the database or other external data sources.
- It provides CRUD (Create, Read, Update, Delete) operations and data access methods.

### 4. @Controller:

- It is another specialization of @Component.
- It is used to mark a class as a controller component.
- It represents the presentation layer of an application.
- It handles user requests, performs request processing, and returns responses.
- It is commonly used in web applications and typically exposes RESTful endpoints or web pages.

While these annotations serve different purposes and have different names, they all ultimately function as components managed by the Spring container. The distinction between them is mostly for clarity and semantic understanding of their roles in the application architecture. However, from a technical perspective, they are all handled in a similar way by Spring's component scanning and dependency injection mechanisms.
</details>
<details><summary>
  
### Why is @Primary annotation used?
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
  
### 
</summary>

</details>
<details><summary>
  
### 
</summary>

</details>
<details><summary>
  
### 
</summary>

</details>
<details><summary>
  
### 
</summary>

</details>
<details><summary>
  
### 
</summary>

</details>