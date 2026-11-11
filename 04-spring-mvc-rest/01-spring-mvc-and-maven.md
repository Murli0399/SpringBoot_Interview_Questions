# Spring MVC and Maven

<details><summary>
  
## [Easy] Explain Spring MVC.
</summary>

Spring MVC, which stands for Spring Model-View-Controller, is a popular and widely used framework for building web applications in the Java programming language. It is a part of the broader Spring Framework and provides a comprehensive solution for developing web applications in a structured and maintainable way. Spring MVC follows the Model-View-Controller architectural pattern, which separates an application into three interconnected components:

1. Model: The Model represents the application's data and business logic. It is responsible for processing and managing the data. In a Spring MVC application, the Model is typically implemented as Java objects or beans. These objects encapsulate the application's data and interact with the database, services, or other data sources.

2. View: The View is responsible for rendering the data provided by the Model. In the context of web applications, the View often consists of templates, JSP (JavaServer Pages), or HTML pages. Spring MVC allows for flexibility in choosing the view technology, and it supports a wide range of view rendering technologies.

3. Controller: The Controller is the central component responsible for handling incoming HTTP requests, processing them, and coordinating the interaction between the Model and the View. It contains application logic, processes user input, and determines which View should be displayed. In Spring MVC, Controllers are typically implemented as Java classes and are responsible for routing requests to the appropriate parts of the application.

Key features and concepts of Spring MVC:

1. DispatcherServlet: This is the front controller in Spring MVC. It receives all incoming requests and dispatches them to the appropriate Controller based on the request URL and HTTP method.

2. Request Mapping: Controllers use request mappings to define which requests they can handle. These mappings specify the URL patterns that map to specific Controller methods.

3. View Resolvers: View resolvers are used to determine which view or template should be rendered. Spring MVC supports various view resolvers for JSP, Thymeleaf, FreeMarker, and more.

4. Model Attributes: Controllers can add attributes to the Model, which are then available for rendering by the View. These attributes are used to pass data from the Controller to the View.

5. Data Binding: Spring MVC provides data binding mechanisms to automatically bind request parameters to Java objects, making it easy to work with forms and form submissions.

6. Validation: Spring MVC offers built-in validation support for form data using annotations and custom validation logic.

7. Interceptors: Interceptors are used to perform actions before and after request handling, such as logging, authentication, or modifying the request or response.

8. RESTful Web Services: Spring MVC can be used to create RESTful web services by returning data in response to HTTP requests, often in JSON or XML format.

Spring MVC is a versatile framework suitable for building a wide range of web applications, from simple web pages to complex enterprise applications. It promotes a clean separation of concerns and is known for its flexibility and ease of integration with other technologies and libraries.

**Example:**

```java
@Controller
public class ProductPageController {
    @GetMapping("/products/{id}")
    public String show(@PathVariable Long id, Model model) {
        model.addAttribute("product", service.find(id));   // Model
        return "product-detail";                           // View name -> templates/product-detail.html
    }
}
// Request flow: DispatcherServlet -> HandlerMapping -> Controller -> ViewResolver -> View
```

</details>

<details><summary>

## [Easy] What is maven?
</summary>

**Apache Maven** is a build and dependency-management tool, mostly used for Java (and other JVM languages such as Kotlin and Scala).

- **Declarative build:** you describe the project in `pom.xml`; Maven knows how to compile, test, package and deploy it.
- **Convention over configuration:** standard layout (`src/main/java`, `src/test/java`, `target/`), so most projects need little configuration.
- **Dependency management:** declares libraries by coordinates (`groupId:artifactId:version`), downloads them and their transitive dependencies from Maven Central into `~/.m2`, and resolves version conflicts ("nearest wins").
- **Build lifecycle:** `validate → compile → test → package → verify → install → deploy`; running a phase runs all phases before it.
- **Plugins** do the actual work (compiler, Surefire for tests, `spring-boot-maven-plugin`).
- **Multi-module projects** and **profiles** for different environments.

The **Maven Wrapper** (`./mvnw`) pins the Maven version per project. Gradle is the main alternative. The name "maven" comes from Yiddish, meaning an "accumulator of knowledge".

**Example:**

```bash
mvn clean package            # compile, test, package the JAR
mvn test -Dtest=OrderServiceTest
mvn dependency:tree          # see transitive dependencies and version conflicts
mvn spring-boot:run
./mvnw verify                # Maven Wrapper: same Maven version on every machine/CI
```

Lifecycle phases run in order: `validate → compile → test → package → verify → install → deploy`.

</details>
