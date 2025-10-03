# Servlets

Servlets are the cornerstone of Java web development. They are Java classes that run in a web container and are responsible for handling client requests and generating dynamic responses. Servlets provide a powerful and flexible way to create web applications.

## Servlet Lifecycle

The lifecycle of a servlet is managed by the web container. It consists of the following phases:

1.  **Instantiation:** When the web container starts, or when it receives the first request for a servlet, it creates an instance of the servlet class. The container only creates one instance of each servlet, and this instance is shared among all requests.

2.  **Initialization:** After creating the servlet instance, the container calls the `init()` method. This method is called only once during the servlet's lifecycle and is used for one-time initialization tasks, such as opening database connections or loading configuration files. The `init()` method must complete successfully before the servlet can handle any requests.

3.  **Request Handling:** For each client request, the container calls the `service()` method of the servlet. The `service()` method receives the `HttpServletRequest` and `HttpServletResponse` objects as arguments. It then determines the type of the request (e.g., GET, POST) and calls the corresponding `doGet()`, `doPost()`, etc., method. This is where the main logic of the servlet resides.

4.  **Destruction:** When the web container shuts down, or when it needs to unload the servlet, it calls the `destroy()` method. This method is called only once and is used for cleanup tasks, such as closing database connections or releasing other resources.

## Servlet Example

This example demonstrates a simple servlet that handles a form submission.

**1. `index.html` (the form):**

```html
<!DOCTYPE html>
<html>
<head>
    <title>Servlet Form</title>
</head>
<body>
    <form action="greet" method="post">
        Enter your name: <input type="text" name="name">
        <input type="submit" value="Submit">
    </form>
</body>
</html>
```

**2. `GreetingServlet.java`:**

```java
package com.example;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/greet")
public class GreetingServlet extends HttpServlet {

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String name = request.getParameter("name");

        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<html><body>");
        out.println("<h1>Hello, " + name + "!</h1>");
        out.println("</body></html>");
    }
}
```

## Key Concepts

### `HttpServletRequest`
The `HttpServletRequest` object represents the HTTP request that the client sent to the server. It provides a wealth of information about the request, including:

*   **Request Parameters:** `getParameter()`, `getParameterValues()`, `getParameterMap()`
*   **HTTP Headers:** `getHeader()`, `getHeaders()`, `getHeaderNames()`
*   **Request URL:** `getRequestURI()`, `getRequestURL()`, `getContextPath()`, `getServletPath()`
*   **HTTP Method:** `getMethod()`
*   **Session:** `getSession()`
*   **Cookies:** `getCookies()`

### `HttpServletResponse`
The `HttpServletResponse` object represents the HTTP response that the servlet will send back to the client. It provides methods for setting the response content, status, and headers.

*   **Setting Content Type:** `setContentType()`
*   **Getting a Writer:** `getWriter()` (for text), `getOutputStream()` (for binary data)
*   **Setting Status Code:** `setStatus()`
*   **Sending Redirects:** `sendRedirect()`
*   **Adding Cookies:** `addCookie()`

### Deployment Descriptors (`web.xml`)
Before the introduction of annotations in Servlet 3.0, servlets were configured in a deployment descriptor file called `web.xml`. While annotations are now the preferred way to configure servlets, `web.xml` is still used for more advanced configurations.

**`web.xml` equivalent of the `@WebServlet` annotation:**

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

### `RequestDispatcher`

The `RequestDispatcher` interface allows you to forward a request to another resource (like a servlet or JSP) on the server, or to include the content of another resource in the response.

### Error Handling

Proper error handling is essential for a good user experience. You can configure error pages declaratively in `web.xml` or handle them programmatically in your servlets.

### File Uploads

Servlets make it easy to handle file uploads from a client. By using the `@MultipartConfig` annotation and the `Part` interface, you can process files submitted through an HTML form.

### Asynchronous Processing

Asynchronous servlets allow you to process long-running operations without blocking the request thread, which can improve the scalability of your application.

### Security

The Servlet API provides mechanisms for both declarative and programmatic security, allowing you to control access to your web resources based on user roles.

### Server-Sent Events (SSE)

Servlets can be used to implement Server-Sent Events, a technology that enables a server to push real-time updates to the client over a standard HTTP connection.

### WebSockets

The Java API for WebSocket provides a standard way to create WebSocket endpoints, which allow for full-duplex, bidirectional communication between a client and a server.