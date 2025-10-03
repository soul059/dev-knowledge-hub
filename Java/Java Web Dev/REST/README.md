# RESTful Web Services (JAX-RS)

REST (Representational State Transfer) is an architectural style for designing networked applications. It is not a standard or a protocol, but a set of constraints that, when applied to a web service, result in a system that is scalable, flexible, and easy to maintain.

## Key Principles of REST

*   **Stateless:** The server does not store any client context between requests. Each request from a client must contain all the information needed to understand and process the request.
*   **Client-Server:** The client and server are separate entities. The client is responsible for the user interface, and the server is responsible for the business logic and data storage.
*   **Cacheable:** Clients can cache responses. This can improve performance and reduce network traffic.
*   **Uniform Interface:** There is a uniform interface between clients and servers. This simplifies the architecture and makes it easier for clients to interact with the server.
*   **Layered System:** A client cannot ordinarily tell whether it is connected directly to the end server or to an intermediary along the way. This allows for the use of intermediaries, such as load balancers and proxies, to improve scalability and security.

## HTTP Methods

RESTful web services use the standard HTTP methods to perform operations on resources.

*   **GET:** Retrieves a resource.
*   **POST:** Creates a new resource.
*   **PUT:** Updates an existing resource.
*   **DELETE:** Deletes a resource.

## HTTP Status Codes

RESTful web services use the standard HTTP status codes to indicate the outcome of a request.

*   **2xx (Success):** The request was successful.
*   **3xx (Redirection):** The client needs to take additional action to complete the request.
*   **4xx (Client Error):** The request contains bad syntax or cannot be fulfilled.
*   **5xx (Server Error):** The server failed to fulfill a valid request.

## JAX-RS

JAX-RS is a Java API for creating RESTful web services. It provides a set of annotations that you can use to map Java methods to HTTP requests.

### JAX-RS Annotations

*   `@Path`: Specifies the URL path to a resource.
*   `@GET`, `@POST`, `@PUT`, `@DELETE`: Specify the HTTP method that a method will handle.
*   `@Produces`: Specifies the MIME type of the response (e.g., `application/json`, `application/xml`).
*   `@Consumes`: Specifies the MIME type of the request.
*   `@PathParam`: Binds a URL path parameter to a method parameter.
*   `@QueryParam`: Binds a URL query parameter to a method parameter.

## JAX-RS Example

This example shows how to create a simple JAX-RS resource that provides a "Hello, World!" message and handles path and query parameters.

**1. `pom.xml` (add JAX-RS dependencies):**

```xml
<dependency>
    <groupId>javax.ws.rs</groupId>
    <artifactId>javax.ws.rs-api</artifactId>
    <version>2.1.1</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.containers</groupId>
    <artifactId>jersey-container-servlet</artifactId>
    <version>2.34</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.inject</groupId>
    <artifactId>jersey-hk2</artifactId>
    <version>2.34</version>
</dependency>
<dependency>
    <groupId>org.glassfish.jersey.media</groupId>
    <artifactId>jersey-media-json-jackson</artifactId>
    <version>2.34</version>
</dependency>
```

**2. `web.xml` (configure the JAX-RS servlet):**

```xml
<servlet>
    <servlet-name>javax.ws.rs.core.Application</servlet-name>
    <servlet-class>org.glassfish.jersey.servlet.ServletContainer</servlet-class>
    <init-param>
        <param-name>jersey.config.server.provider.packages</param-name>
        <param-value>com.example.resource</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>javax.ws.rs.core.Application</servlet-name>
    <url-pattern>/api/*</url-pattern>
</servlet-mapping>
```

**3. `HelloResource.java` (the JAX-RS resource):**

```java
package com.example.resource;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.QueryParam;
import javax.ws.rs.core.MediaType;

@Path("/hello")
public class HelloResource {

    @GET
    @Produces(MediaType.TEXT_PLAIN)
    public String sayHello() {
        return "Hello, World!";
    }

    @GET
    @Path("/{name}")
    @Produces(MediaType.TEXT_PLAIN)
    public String sayHello(@PathParam("name") String name) {
        return "Hello, " + name + "!";
    }

    @GET
    @Path("/greeting")
    @Produces(MediaType.APPLICATION_JSON)
    public Greeting sayHello(@QueryParam("name") String name) {
        return new Greeting(name);
    }
}
```

**4. `Greeting.java` (a simple POJO for JSON conversion):**

```java
package com.example.resource;

public class Greeting {
    private final String message;

    public Greeting(String name) {
        this.message = "Hello, " + (name != null ? name : "World") + "!";
    }

    public String getMessage() {
        return message;
    }
}
```