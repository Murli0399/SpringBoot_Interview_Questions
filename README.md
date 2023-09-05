# SpringBoot_Interview_Questions
This repository is created for interview purpose, If anyone need to learn spring and spring boot-related questions and answers, So follow this repo and crack interviews. 

- What are the major features of Spring 5?
- Is Spring 5 compatible with older versions of Java?
- What are the advantages of using the Spring framework?
- List the modules of the Spring framework.
- What is tight coupling?
- What is loose coupling?
- What is the purpose of the application.properties file?
- What is a pom.xml file?
- What design patterns are used in the Spring framework?
- What are some of the best practices for Spring Framework?
- Describe some standard Spring events.
- How does Spring 5 make use of JDK 9 modularity?






## Diff between spring and spring boot?
**Spring:** Spring Framework is the most popular application development framework of Java. The main feature of the Spring Framework is dependency Injection or Inversion of Control (IoC). With the help of Spring Framework, we can develop a loosely coupled application. It is better to use if application type or characteristics are purely defined.

**Spring Boot:** Spring Boot is a module of Spring Framework. It allows us to build a stand-alone application with minimal or zero configurations. It is better to use if we want to develop a simple Spring-based application or RESTful services.

|Spring|Spring Boot|
|------|-----------|
|Spring Framework is a widely used Java EE framework for building applications.|Spring Boot Framework is widely used to develop REST APIs.|
|It aims to simplify Java EE development and makes developers more productive.|It aims to shorten the code length and provide the easiest way to develop Web Applications.|
|The primary feature of the Spring Framework is dependency injection.|The primary feature of Spring Boot is Autoconfiguration. It automatically configures the classes based on the requirement.|
|It helps to make things simpler by allowing us to develop loosely coupled applications.|It helps to create a stand-alone application with less configuration.|
|The developer writes a lot of code (boilerplate code) to do the minimal task.|It reduces boilerplate code.|
|To test the Spring project, we need to set up the server explicitly.|Spring Boot offers embedded servers such as Jetty and Tomcat, etc.|
|Developers manually define dependencies for the Spring project in pom.xml.|Spring Boot comes with the concept of starter in pom.xml file that internally takes care of downloading the dependencies JARs based on Spring Boot Requirement.|











Inversion of Control (IoC)
What is Inversion of Control (IoC)?
What are the advantages of Inversion of Control?
What is the responsibility of an IoC container?
Describe the two types of IoC container.
Give an example of the BeanFactory implementation.
What is ApplicationContext?
Give examples of the ApplicationContext implementations.
What is the difference between BeanFactory and ApplicationContext?
How is ApplicationContext configured in Spring?
What is WebApplicationContext?
What happens if the context is not closed?
Dependency injection
What is dependency injection?
How is dependency injection related to inversion of control?
What are the types of dependency injection?
What is constructor injection?
How does setter injection work?
Explain setter injection for objects and literal values.
Explain injection of Java Collection types.
What is the difference between constructor and setter injection?
Which dependency injection approach is better?
What is method injection?
What is a circular dependency and how should it be resolved?







Spring Data JPA
What is Spring Data?
What is JPA?
What is the difference between JPA and Spring Data JPA?
What is the difference between CrudRepository and JpaRepository?
Which No SQL databases does Spring support?
Which ORMs are supported by Spring?
What is Hibernate?
What are the different types of transaction management supported by Spring?
Which transaction management type is preferred in Spring?
What are some benefits of using Spring Transactions?
What happens when the @Transactional annotation is used on a method?
What is the difference between JdbcTemplate and JPA?
