# ServletConfig and ServletContext

`ServletConfig` and `ServletContext` are two important objects in the Servlet API that allow you to pass configuration information to your servlets and share data within your web application.

## ServletConfig

The `ServletConfig` object is used to pass initialization parameters to a single servlet. Each servlet has its own `ServletConfig` object, and the parameters are defined in the `web.xml` deployment descriptor or using annotations.

**Scope:** The scope of a `ServletConfig` object is limited to the servlet to which it belongs.

### Accessing ServletConfig

You can get a reference to the `ServletConfig` object in two ways:

1.  **In the `init()` method:** The web container passes the `ServletConfig` object to the `init()` method of the servlet.

    ```java
    public void init(ServletConfig config) throws ServletException {
        // ...
    }
    ```

2.  **Using the `getServletConfig()` method:** You can call the `getServletConfig()` method from anywhere in the servlet.

    ```java
    ServletConfig config = getServletConfig();
    ```

### Getting Initialization Parameters

Once you have a reference to the `ServletConfig` object, you can use the `getInitParameter()` method to get the value of an initialization parameter.

**Example (`web.xml`):**

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

**Example (Java):**

```java
String dbUrl = getServletConfig().getInitParameter("dbUrl");
```

## ServletContext

The `ServletContext` object is used to share data among all the servlets in a web application. There is only one `ServletContext` object per web application.

**Scope:** The scope of a `ServletContext` object is the entire web application.

### Accessing ServletContext

You can get a reference to the `ServletContext` object in two ways:

1.  **From the `ServletConfig` object:**

    ```java
    ServletContext context = getServletConfig().getServletContext();
    ```

2.  **From the `HttpServletRequest` object:**

    ```java
    ServletContext context = request.getServletContext();
    ```

### Getting Context Parameters

Context parameters are defined in the `web.xml` deployment descriptor and are available to all the servlets in the web application.

**Example (`web.xml`):**

```xml
<context-param>
    <param-name>appName</param-name>
    <param-value>My Awesome App</param-value>
</context-param>
```

**Example (Java):**

```java
String appName = getServletContext().getInitParameter("appName");
```

### Storing and Retrieving Attributes

The `ServletContext` object can also be used to store and retrieve application-scoped attributes.

-   `void setAttribute(String name, Object object)`: Binds an object to this context, using the name specified.
-   `Object getAttribute(String name)`: Returns the context attribute with the given name, or `null` if there is no attribute by that name.
-   `void removeAttribute(String name)`: Removes the attribute with the given name from the context.

**Example:**

```java
// Store an attribute
getServletContext().setAttribute("appVersion", "1.0");

// Retrieve an attribute
String appVersion = (String) getServletContext().getAttribute("appVersion");
```

## Key Differences

| Feature           | ServletConfig                                  | ServletContext                                     |
| ----------------- | ---------------------------------------------- | -------------------------------------------------- |
| **Scope**         | A single servlet                               | The entire web application                         |
| **Availability**  | One `ServletConfig` object per servlet         | One `ServletContext` object per web application    |
| **Parameters**    | `getInitParameter()`                           | `getInitParameter()`                               |
| **Attributes**    | Does not have attributes                       | `setAttribute()`, `getAttribute()`, `removeAttribute()` |
| **Configuration** | In the `<servlet>` element of `web.xml`        | In the `<context-param>` element of `web.xml`      |
