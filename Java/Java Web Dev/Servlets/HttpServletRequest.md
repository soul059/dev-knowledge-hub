# HttpServletRequest

The `HttpServletRequest` object is one of the most important objects in the Servlet API. It represents the HTTP request that a client sends to the server. The web container creates an `HttpServletRequest` object for each incoming request and passes it to the `service()` method of the servlet.

The `HttpServletRequest` object provides a wealth of information about the request, which can be used to customize the response. It is an interface defined in the `javax.servlet.http` package.

## Key Methods of `HttpServletRequest`

The `HttpServletRequest` interface provides a wide range of methods for accessing information about the request. Here are some of the most commonly used methods:

### 1. Accessing Request Parameters

Request parameters are data that is sent from the client to the server, typically from an HTML form. The following methods are used to access request parameters:

-   `String getParameter(String name)`: Returns the value of a request parameter as a `String`. If the parameter does not exist, it returns `null`.
-   `String[] getParameterValues(String name)`: Returns an array of `String` objects containing all the values of a given request parameter. This is useful for parameters that can have multiple values (e.g., checkboxes).
-   `java.util.Map<String, String[]> getParameterMap()`: Returns a `java.util.Map` of the request parameters, where the keys are the parameter names and the values are arrays of `String` objects.

**Example:**

```java
String name = request.getParameter("name");
String[] hobbies = request.getParameterValues("hobbies");
```

### 2. Accessing HTTP Headers

HTTP headers provide additional information about the request, such as the user agent, the accepted content types, and cookies. The following methods are used to access HTTP headers:

-   `String getHeader(String name)`: Returns the value of a specified request header as a `String`.
-   `java.util.Enumeration<String> getHeaders(String name)`: Returns all the values of a specified request header as an `Enumeration` of `String` objects.
-   `java.util.Enumeration<String> getHeaderNames()`: Returns an `Enumeration` of all the header names in the request.

**Example:**

```java
String userAgent = request.getHeader("User-Agent");
```

### 3. Accessing Request URL Information

The following methods are used to get information about the URL of the request:

-   `String getRequestURI()`: Returns the part of the request's URL from the protocol name up to the query string.
-   `StringBuffer getRequestURL()`: Returns the full URL that the client used to make the request.
-   `String getContextPath()`: Returns the portion of the request URI that indicates the context of the request.
-   `String getServletPath()`: Returns the part of the request's URL that calls the servlet.

### 4. Accessing Other Request Information

-   `String getMethod()`: Returns the HTTP method of the request (e.g., GET, POST, PUT, DELETE).
-   `HttpSession getSession()`: Returns the current session associated with this request, or creates a new one if the request does not have a session.
-   `javax.servlet.http.Cookie[] getCookies()`: Returns an array of `Cookie` objects that the client sent with the request.
-   `java.io.BufferedReader getReader()`: Retrieves the body of the request as character data using a `BufferedReader`.
-   `javax.servlet.ServletInputStream getInputStream()`: Retrieves the body of the request as binary data using a `ServletInputStream`.
