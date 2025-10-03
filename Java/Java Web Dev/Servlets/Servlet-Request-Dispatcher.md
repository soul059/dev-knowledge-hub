
# Servlet Request Dispatcher

The `RequestDispatcher` interface provides a way to dispatch a request to another resource on the server, such as a servlet, JSP page, or HTML file. This is a server-side operation, so the client is not aware that the request is being handled by another resource.

## Key Methods

The `RequestDispatcher` interface has two main methods:

-   `forward(ServletRequest request, ServletResponse response)`: Forwards a request from a servlet to another resource on the server. When you forward a request, the control is completely transferred to the new resource, and the original servlet's response is not sent to the client.
-   `include(ServletRequest request, ServletResponse response)`: Includes the content of a resource in the response. When you include a resource, the control is temporarily transferred to the included resource, and then it returns to the original servlet.

## Forwarding a Request

Forwarding is useful when you want to delegate the processing of a request to another component. For example, a servlet might perform some initial processing and then forward the request to a JSP page to generate the presentation.

**Example:**

```java
// In a servlet
RequestDispatcher dispatcher = request.getRequestDispatcher("/WEB-INF/views/user-profile.jsp");
dispatcher.forward(request, response);
```

## Including a Resource

Including is useful when you want to reuse common components, such as a header or footer, across multiple pages.

**Example:**

```java
// In a servlet
response.setContentType("text/html");
PrintWriter out = response.getWriter();
out.println("<html><body>");

RequestDispatcher headerDispatcher = request.getRequestDispatcher("/header.html");
headerDispatcher.include(request, response);

out.println("<h1>Welcome to our website!</h1>");

RequestDispatcher footerDispatcher = request.getRequestDispatcher("/footer.html");
footerDispatcher.include(request, response);

out.println("</body></html>");
```
