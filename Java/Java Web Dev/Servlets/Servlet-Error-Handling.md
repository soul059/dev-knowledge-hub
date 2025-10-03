# Servlet Error Handling

Error handling is a critical part of any web application. In Java Servlets, you can handle errors in a declarative way using the deployment descriptor (`web.xml`) or programmatically in your servlet code.

## Declarative Error Handling (`web.xml`)

The `<error-page>` element in `web.xml` allows you to specify custom error pages for specific HTTP status codes or Java exceptions.

**Example:**

```xml
<error-page>
    <error-code>404</error-code>
    <location>/error-pages/404.jsp</location>
</error-page>

<error-page>
    <exception-type>java.lang.Throwable</exception-type>
    <location>/error-pages/error.jsp</location>
</error-page>
```

In this example, if the application throws a 404 Not Found error, the user will be redirected to the `/error-pages/404.jsp` page. If any other exception occurs, the user will be redirected to the `/error-pages/error.jsp` page.

## Programmatic Error Handling

You can also handle errors programmatically in your servlet code using the `sendError()` method of the `HttpServletResponse` object.

**Example:**

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    String id = request.getParameter("id");
    if (id == null || id.isEmpty()) {
        response.sendError(HttpServletResponse.SC_BAD_REQUEST, "Missing 'id' parameter.");
        return;
    }
    // ...
}
```

## Custom Error-Handling Servlet

You can create a dedicated servlet to handle errors. This servlet can be mapped to a specific URL in `web.xml` and can be used to display a custom error page.

**`ErrorHandlerServlet.java`:**

```java
@WebServlet("/errorHandler")
public class ErrorHandlerServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Integer statusCode = (Integer) request.getAttribute("javax.servlet.error.status_code");
        String errorMessage = (String) request.getAttribute("javax.servlet.error.message");

        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<html><body>");
        out.println("<h1>Error</h1>");
        out.println("<p>Status Code: " + statusCode + "</p>");
        out.println("<p>Message: " + errorMessage + "</p>
        out.println("</body></html>");
    }
}
```

**`web.xml`:**

```xml
<servlet>
    <servlet-name>ErrorHandlerServlet</servlet-name>
    <servlet-class>com.example.ErrorHandlerServlet</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>ErrorHandlerServlet</servlet-name>
    <url-pattern>/errorHandler</url-pattern>
</servlet-mapping>

<error-page>
    <error-code>500</error-code>
    <location>/errorHandler</location>
</error-page>
```
