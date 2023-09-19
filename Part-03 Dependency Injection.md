<details><summary>

## What is dependency injection?
</summary>

Dependency Injection (DI) is a design pattern and a software development technique commonly used in object-oriented programming to achieve the separation of concerns, increase modularity, and improve testability and maintainability of code.

In simpler terms, Dependency Injection is about providing the necessary dependencies (objects, services, or other components) to a class rather than the class creating those dependencies itself. This helps to decouple components and make them more independent, which in turn makes the code easier to manage, test, and extend.
</details>
<details><summary>

## How is dependency injection related to inversion of control?
</summary>

Dependency Injection (DI) and Inversion of Control (IoC) are related concepts in software design and architecture, and they are often used together. They both aim to improve the modularity, flexibility, and testability of software systems, but they address different aspects of achieving these goals.

1. Inversion of Control (IoC):

IoC is a broader design principle that refers to the inversion of the flow of control in a software system. In traditional programming, your application code typically controls the flow of execution, making decisions about when and how to call functions or methods in various modules or components. In contrast, with IoC, the control flow is inverted. Instead of your code controlling the flow, it's handed over to a framework or container that manages the execution and sequencing of various components or modules.

Key characteristics of IoC:

- Control flow is managed externally by a framework or container.
- Promotes modularity and decoupling of components.
- Enhances flexibility and extensibility by allowing components to be easily swapped out or replaced.

2. Dependency Injection (DI):

DI is a specific technique used to achieve IoC. It deals with the way objects or components obtain their dependencies (i.e., other objects or services they rely on). In traditional programming, an object might create its dependencies, which can lead to tight coupling between components. DI solves this problem by inverting the responsibility of obtaining dependencies. Instead of an object creating its own dependencies, they are provided to it from the outside.

Key characteristics of DI:

- Dependencies are "injected" into an object rather than created by the object.
- Promotes loose coupling between components.
- Enhances testability by allowing dependencies to be easily replaced with mock objects for testing.

In summary, IoC is a design principle that shifts control flow management to an external entity, while Dependency Injection is a specific technique used to achieve IoC by managing how an object obtains its dependencies. Together, they lead to more modular, maintainable, and testable software systems. IoC is a broader concept, and DI is one of the techniques employed to implement IoC.
</details>
<details><summary>

## What are the types of dependency injection?
</summary>

Dependency Injection (DI) is a design pattern used in software development to achieve Inversion of Control (IoC) by injecting dependencies into an object rather than having the object create its dependencies. There are several types of DI, including:

1. Constructor Injection:

- In constructor injection, dependencies are injected through the constructor of a class.
- It's considered one of the most common and recommended forms of dependency injection because it ensures that an object is in a valid state when it's created.

Example (in Java):

```
public class ExampleService {
    private final Dependency dependency;

    public ExampleService(Dependency dependency) {
        this.dependency = dependency;
    }

    // ...
}
```

2. Setter Injection:

- In setter injection, dependencies are injected through setter methods of a class.
- This approach is useful when you have optional dependencies or when you need to change dependencies at runtime.

Example (in Java):

```
public class ExampleService {
    private Dependency dependency;

    public void setDependency(Dependency dependency) {
        this.dependency = dependency;
    }

    // ...
}
```
3. Method Injection:

- In method injection, dependencies are injected directly into methods that need them.
- This approach is useful when you have methods that require specific dependencies, but the class itself doesn't depend on them.

Example (in Python):

```
class ExampleService:
    def do_something(self, dependency):
        # Use the injected dependency here
        pass
```
4. Interface-based Injection:

- In this approach, an interface defines the contract for accessing dependencies, and concrete implementations provide the actual dependencies.
- It's often used in languages like Java and C#.

Example (in Java):

```
public interface DependencyProvider {
    Dependency getDependency();
}

public class ExampleService {
    private final DependencyProvider dependencyProvider;

    public ExampleService(DependencyProvider dependencyProvider) {
        this.dependencyProvider = dependencyProvider;
    }

    public void doSomething() {
        Dependency dependency = dependencyProvider.getDependency();
        // Use the dependency here
    }
}
```
5. Parameter Injection:

- In parameter injection, dependencies are injected as method parameters when calling a method.
- This is commonly used in functional programming languages and when working with frameworks like Spring (in Java).

Example (in Scala):

```
def doSomething(dependency: Dependency): Unit = {
    // Use the injected dependency here
}
```
The choice of which type of dependency injection to use depends on the specific needs of your application and the programming language or framework you are working with. Constructor injection is often recommended as a default choice due to its clarity and ability to ensure that objects are properly initialized.
</details>
<details><summary>

## What is constructor injection?
</summary>

Constructor injection is a form of dependency injection (DI) in software development. It's a design pattern where a class's dependencies (i.e., other objects or services that it relies on) are provided to it through its constructor when the class is instantiated. This allows for the inversion of control, where the responsibility of managing dependencies is shifted from the class itself to an external component (typically a DI container or framework).

Here's how constructor injection works:

- Dependency Definition: The dependencies that a class requires are defined as parameters in its constructor. Each parameter represents a dependency.

- Dependency Injection: When an instance of the class is created, the DI container or framework injects (provides) the required dependencies into the constructor.

- Initialization: The class stores these dependencies as private fields or properties for later use.
</details>
<details><summary>

## How does setter injection work?
</summary>

Setter injection is a form of dependency injection (DI) in software development. It's a design pattern where a class's dependencies (i.e., other objects or services that it relies on) are provided through setter methods rather than through the constructor. This allows for the inversion of control, where the responsibility of managing dependencies is shifted from the class itself to an external component (typically a DI container or framework).

Here's how setter injection works:

- Dependency Definition: The dependencies that a class requires are defined as private fields or properties in the class.

- Setter Methods: The class provides setter methods for each dependency, allowing external components to set (inject) the dependencies after the class is instantiated.

- Dependency Injection: When an instance of the class is created, the DI container or framework sets (injects) the required dependencies using the provided setter methods.
</details>
<details><summary>

## Explain setter injection for objects and literal values.
</summary>

1. Setter Injection for Objects:

When using setter injection for objects, you provide a setter method in the class that allows external components (typically a DI container or framework) to inject instances of other classes or dependencies. These injected objects can be used by the class to perform various tasks.

2. Setter Injection for Literal Values:

Setter injection for literal values involves providing setter methods to inject constant values, such as strings, numbers, or configuration parameters, into a class. This is often used for runtime configuration.

3. Usage:

In both cases, after creating an instance of the class, you call the appropriate setter method to provide the required objects or values. This allows for flexibility in configuring the class for different use cases without modifying its constructor.
</details>
<details><summary>

## Explain the injection of Java Collection types.
</summary>

</details>
<details><summary>

## What is the difference between constructor and setter injection?
</summary>

</details>
<details><summary>

## Which dependency injection approach is better?
</summary>

</details>
<details><summary>

## What is method injection?
</summary>

</details>
<details><summary>

## What is a circular dependency and how should it be resolved?
</summary>

</details>
