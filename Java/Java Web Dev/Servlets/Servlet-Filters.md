# Servlet Filters

A servlet filter is an object that can intercept and process requests before they reach a servlet and after they leave a servlet. Filters are used to perform common tasks such as logging, authentication, data compression, and encryption.

Filters are defined in the `javax.servlet` package and are represented by the `Filter` interface.

## Filter Lifecycle

The lifecycle of a filter is similar to the lifecycle of a servlet and is managed by the web container.

1.  **Initialization:** The container calls the `init()` method of the filter once when the filter is first loaded.
2.  **Filtering:** The container calls the `doFilter()` method for each request that maps to the filter.
3.  **Destruction:** The container calls the `destroy()` method once when the filter is about to be unloaded.

### The `init()` Method

The `init()` method is used for one-time initialization tasks.

```java
public void init(FilterConfig filterConfig) throws ServletException;
```

The `FilterConfig` object provides access to the filter's initialization parameters.

### The `doFilter()` Method

The `doFilter()` method is the core of the filter. It is responsible for processing the request and response.

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException;
```

-   `request`: The `ServletRequest` object.
-   `response`: The `ServletResponse` object.
-   `chain`: The `FilterChain` object, which is used to pass the request and response to the next filter in the chain, or to the target servlet if the filter is the last one.

**Example:**

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    // Pre-processing logic (e.g., logging)
    System.out.println("Request to " + ((HttpServletRequest) request).getRequestURI());

    // Pass the request and response to the next filter
    chain.doFilter(request, response);

    // Post-processing logic (e.g., modifying the response)
}
```

### The `destroy()` Method

The `destroy()` method is used for cleanup tasks.

```java
public void destroy();
```

## Configuring Filters

Filters can be configured in the `web.xml` deployment descriptor or using the `@WebFilter` annotation.

### Using `web.xml`

```xml
<filter>
    <filter-name>MyFilter</filter-name>
    <filter-class>com.example.MyFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>MyFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### Using `@WebFilter`

```java
@WebFilter("/*")
public class MyFilter implements Filter {
    // ...
}
```

## Common Use Cases for Filters

-   **Authentication and Authorization:** To check if a user is logged in and has the necessary permissions to access a resource.
-   **Logging and Auditing:** To log information about incoming requests.
-   **Image and Data Compression:** To compress the response data to reduce bandwidth usage.
-   **Encryption and Decryption:** To encrypt and decrypt data sent between the client and the server.
-   **Request and Response Wrapping:** To modify the request and response objects.
