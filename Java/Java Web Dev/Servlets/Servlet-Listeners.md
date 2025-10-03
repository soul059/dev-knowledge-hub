# Servlet Listeners

Servlet listeners are objects that can be used to monitor and react to events in a web application. They are used to perform tasks when certain events occur, such as when the application starts or stops, or when a session is created or destroyed.

Listeners are defined in the `javax.servlet` and `javax.servlet.http` packages.

## Types of Listeners

There are several types of listeners, each designed to monitor a specific type of event.

### 1. ServletContext Listeners

These listeners are used to monitor events related to the `ServletContext`.

-   `ServletContextListener`: Notified when the `ServletContext` is initialized or destroyed.
    -   `contextInitialized(ServletContextEvent sce)`: Called when the web application is first started.
    -   `contextDestroyed(ServletContextEvent sce)`: Called when the web application is about to be shut down.
-   `ServletContextAttributeListener`: Notified when an attribute is added to, removed from, or replaced in the `ServletContext`.

### 2. HttpSession Listeners

These listeners are used to monitor events related to `HttpSession` objects.

-   `HttpSessionListener`: Notified when a session is created or destroyed.
    -   `sessionCreated(HttpSessionEvent se)`: Called when a new session is created.
    -   `sessionDestroyed(HttpSessionEvent se)`: Called when a session is about to be invalidated.
-   `HttpSessionAttributeListener`: Notified when an attribute is added to, removed from, or replaced in a session.

### 3. ServletRequest Listeners

These listeners are used to monitor events related to `ServletRequest` objects.

-   `ServletRequestListener`: Notified when a request is initialized or destroyed.
    -   `requestInitialized(ServletRequestEvent sre)`: Called when a request is about to come into scope of the web application.
    -   `requestDestroyed(ServletRequestEvent sre)`: Called when a request is about to go out of scope of the web application.
-   `ServletRequestAttributeListener`: Notified when an attribute is added to, removed from, or replaced in a request.

## Configuring Listeners

Listeners can be configured in the `web.xml` deployment descriptor or using annotations.

### Using `web.xml`

```xml
<listener>
    <listener-class>com.example.MyServletContextListener</listener-class>
</listener>
```

### Using Annotations

```java
@WebListener
public class MyServletContextListener implements ServletContextListener {
    // ...
}
```

## Common Use Cases for Listeners

-   **Database Connection Management:** To create a database connection pool when the application starts and close it when the application stops.
-   **Application Initialization:** To perform one-time initialization tasks when the application starts.
-   **Session Tracking:** To keep track of the number of active sessions.
-   **Request Logging:** To log information about incoming requests.
