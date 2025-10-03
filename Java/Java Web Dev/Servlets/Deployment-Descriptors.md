# Deployment Descriptors (web.xml)

In the early days of Java web development, the deployment descriptor (`web.xml`) was the primary way to configure a web application. It is an XML file that describes how a web application should be deployed. The `web.xml` file is located in the `WEB-INF` directory of the web application.

With the introduction of annotations in Servlet 3.0, the need for `web.xml` has been reduced. However, it is still used for more advanced configurations and for overriding the settings of annotations.

## Key Configuration Elements in `web.xml`

The `web.xml` file contains various elements for configuring a web application. Here are some of the most important ones:

### 1. Servlet Configuration

Before annotations, servlets were configured using the `<servlet>` and `<servlet-mapping>` elements.

-   `<servlet>`: Defines a servlet, including its name and the fully qualified class name.
-   `<servlet-mapping>`: Maps a servlet to a URL pattern.

**Example:**

```xml
<servlet>
    <servlet-name>GreetingServlet</servlet-name>
    <servlet-class>com.example.GreetingServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>GreetingServlet</servlet-name>
    <url-pattern>/greet</url-pattern>
</servlet-mapping>
```

This is the equivalent of the `@WebServlet("/greet")` annotation.

### 2. Initialization Parameters

You can provide initialization parameters to a servlet using the `<init-param>` element.

**Example:**

```xml
<servlet>
    <servlet-name>MyServlet</servlet-name>
    <servlet-class>com.example.MyServlet</servlet-class>
    <init-param>
        <param-name>dbUrl</param-name>
        <param-value>jdbc:mysql://localhost:3306/mydb</param-value>
    </init-param>
</servlet>
```

These parameters can be accessed in the servlet using the `ServletConfig` object.

### 3. Context Parameters

Context parameters are initialization parameters that are available to the entire web application. They are defined using the `<context-param>` element.

**Example:**

```xml
<context-param>
    <param-name>appName</param-name>
    <param-value>My Awesome App</param-value>
</context-param>
```

These parameters can be accessed using the `ServletContext` object.

### 4. Welcome File List

The `<welcome-file-list>` element specifies a list of files to be used as the default welcome pages for the web application.

**Example:**

```xml
<welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.jsp</welcome-file>
</welcome-file-list>
```

### 5. Error Pages

The `<error-page>` element is used to specify a custom error page for a given HTTP status code or exception type.

**Example:**

```xml
<error-page>
    <error-code>404</error-code>
    <location>/error404.jsp</location>
</error-page>
<error-page>
    <exception-type>java.lang.Exception</exception-type>
    <location>/error.jsp</location>
</error-page>
```

### 6. Filter Configuration

Filters are used to intercept requests and responses. They can be configured in `web.xml` using the `<filter>` and `<filter-mapping>` elements.

**Example:**

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

## The Role of `web.xml` in Modern Web Applications

While annotations are now the preferred way to configure servlets, filters, and listeners, `web.xml` still has its place:

-   **Overriding Annotations:** The configurations in `web.xml` can override the configurations specified in annotations.
-   **Advanced Configuration:** Some advanced configurations, such as security constraints and session timeout, can only be done in `web.xml`.
-   **Legacy Applications:** Many existing web applications still use `web.xml` for configuration.
