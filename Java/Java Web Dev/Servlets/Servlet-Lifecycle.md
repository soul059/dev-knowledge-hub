# Servlet Lifecycle

The lifecycle of a servlet is managed by the web container (e.g., Apache Tomcat, Jetty). It defines the sequence of events from the creation of a servlet instance to its destruction. Understanding the servlet lifecycle is crucial for developing robust and efficient web applications.

The lifecycle consists of three main phases:

1.  **Initialization:** The servlet is initialized.
2.  **Request Handling:** The servlet handles client requests.
3.  **Destruction:** The servlet is destroyed.

These phases are managed through the following three methods, which are defined in the `javax.servlet.Servlet` interface:

-   `init()`
-   `service()`
-   `destroy()`

## 1. Initialization Phase

The initialization phase occurs only once during the servlet's lifecycle. The web container performs the following steps:

1.  **Load the Servlet Class:** The container loads the servlet class into memory.
2.  **Create an Instance:** The container creates an instance of the servlet class using the default constructor.
3.  **Call the `init()` Method:** The container calls the `init()` method on the servlet instance.

### The `init()` Method

The `init()` method is used for one-time initialization tasks, such as:

-   Opening database connections.
-   Loading configuration files.
-   Initializing resources.

The `init()` method is defined as follows:

```java
public void init(ServletConfig config) throws ServletException;
```

The `ServletConfig` object provides access to the servlet's configuration parameters, which are defined in the `web.xml` deployment descriptor or through annotations.

**Key Points:**

-   The `init()` method is called only once.
-   It must complete successfully before the servlet can handle any requests.
-   If the `init()` method throws a `ServletException`, the servlet will not be put into service.

## 2. Request Handling Phase

Once the servlet is initialized, it is ready to handle client requests. For each incoming request, the web container performs the following steps:

1.  **Create `HttpServletRequest` and `HttpServletResponse` Objects:** The container creates objects that represent the client's request and the server's response.
2.  **Call the `service()` Method:** The container calls the `service()` method of the servlet, passing the request and response objects as arguments.

### The `service()` Method

The `service()` method is the heart of the servlet. It is responsible for processing the client's request and generating a response. The `service()` method is defined as follows:

```java
public void service(ServletRequest request, ServletResponse response) throws ServletException, IOException;
```

In the case of HTTP requests, the `service()` method of the `HttpServlet` class will dispatch the request to the appropriate `doXXX()` method (e.g., `doGet()`, `doPost()`) based on the HTTP method of the request.

**Key Points:**

-   The `service()` method is called for each client request.
-   The servlet instance is shared among multiple threads, so it is important to write thread-safe code.

## 3. Destruction Phase

The destruction phase occurs when the web container decides to unload the servlet. This can happen when:

-   The web container is shutting down.
-   The web application is being undeployed.
-   The servlet is being reloaded.

### The `destroy()` Method

The `destroy()` method is called only once and is used for cleanup tasks, such as:

-   Closing database connections.
-   Releasing resources.
-   Saving the servlet's state.

The `destroy()` method is defined as follows:

```java
public void destroy();
```

**Key Points:**

-   The `destroy()` method is called only once.
-   After the `destroy()` method is called, the servlet is eligible for garbage collection.
