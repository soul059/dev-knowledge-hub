# Spring Boot

Spring Boot is a project built on top of the Spring Framework that makes it easy to create stand-alone, production-grade Spring-based applications that you can "just run". It takes an opinionated view of the Spring platform and third-party libraries so you can get started with minimum fuss.

## Key Features of Spring Boot

*   **Auto-configuration:** This is the most powerful feature of Spring Boot. It automatically configures your application based on the JAR dependencies you have added to the classpath. For example, if you have `spring-boot-starter-web` on your classpath, Spring Boot will automatically configure an embedded Tomcat server, Spring MVC, and other web-related components.
*   **Standalone:** Spring Boot applications can be packaged as a single, executable JAR file with an embedded web server. This makes it easy to deploy and run your application without needing to set up a separate web server.
*   **Opinionated:** Spring Boot has an "opinionated" view of the best way to configure and use the Spring Framework. This means that it provides a set of sensible defaults that work well for most applications. However, you can always override these defaults if you need to.
*   **Production-ready:** Spring Boot provides a number of production-ready features out of the box, such as:
    *   **Metrics:** Detailed metrics about your application, such as memory usage, garbage collection, and web requests.
    *   **Health Checks:** A way to check the health of your application.
    *   **Externalized Configuration:** A way to configure your application using external properties files, environment variables, or command-line arguments.

## Creating a Spring Boot Project

The easiest way to create a new Spring Boot project is to use the Spring Initializr (https://start.spring.io/). The Spring Initializr is a web-based tool that allows you to generate a new Spring Boot project with the dependencies you need.

## Spring Boot Example (RESTful Web Service)

This example shows how to create a simple RESTful web service with Spring Boot.

**1. `pom.xml` (add Spring Boot starter dependency):**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.3</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

**2. `Application.java` (the main application class):**

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**3. `HelloController.java` (the REST controller):**

```java
package com.example;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello, World!";
    }
}
```

## How to Run

You can run the application from the command line using Maven:

```bash
mvn spring-boot:run
```

Once the application is running, you can access the REST endpoint at `http://localhost:8080/hello`.

## Spring Boot Starters

Spring Boot starters are a set of convenient dependency descriptors that you can include in your application. They provide a one-stop-shop for all the Spring and related technology that you need, without having to hunt down and configure all the dependencies yourself.

Some popular starters include:

*   `spring-boot-starter-web`: For building web applications, including RESTful applications, using Spring MVC.
*   `spring-boot-starter-data-jpa`: For using Spring Data JPA with Hibernate.
*   `spring-boot-starter-test`: For testing Spring Boot applications with libraries like JUnit, Hamcrest, and Mockito.
*   `spring-boot-starter-actuator`: For adding production-ready features to your application.

## Spring Boot Actuator

Spring Boot Actuator is a sub-project of Spring Boot that provides a number of production-ready features to help you monitor and manage your application.

To use the Actuator, you just need to add the `spring-boot-starter-actuator` dependency to your `pom.xml` file.

Once you have added the Actuator to your application, you will have access to a number of new endpoints, such as:

*   `/actuator/health`: Shows the health of your application.
*   `/actuator/metrics`: Shows a variety of metrics about your application.
*   `/actuator/info`: Shows information about your application.
*   `/actuator/env`: Shows the environment variables of your application.