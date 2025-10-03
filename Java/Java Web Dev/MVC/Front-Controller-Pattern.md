# Front Controller Pattern

The Front Controller pattern is a J2EE design pattern that provides a centralized entry point for all incoming requests in a web application. This pattern is often used in conjunction with the MVC pattern to streamline request handling and improve the modularity of the application.

## Key Components

-   **Front Controller:** A single controller (usually a servlet) that handles all incoming requests.
-   **Dispatcher:** The dispatcher is responsible for forwarding the request to the appropriate handler (e.g., another servlet or a JSP page) based on the request URL or other parameters.
-   **Handlers:** These are the components that are responsible for processing the request and generating a response.

## Example

This example demonstrates a simple implementation of the Front Controller pattern.

**`FrontController.java`:**

```java
@WebServlet("*.do")
public class FrontController extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String action = request.getServletPath();

        if (action.equals("/home.do")) {
            request.getRequestDispatcher("/WEB-INF/views/home.jsp").forward(request, response);
        } else if (action.equals("/about.do")) {
            request.getRequestDispatcher("/WEB-INF/views/about.jsp").forward(request, response);
        } else {
            response.sendError(HttpServletResponse.SC_NOT_FOUND);
        }
    }
}
```

In this example, the `FrontController` handles all requests that end with `.do`. It then uses the request's servlet path to determine which JSP page to forward the request to.
