# Asynchronous Processing in Servlets

Asynchronous processing, introduced in Servlet 3.0, allows you to process a request in a separate thread without holding up the main request processing thread. This is particularly useful for long-running operations, such as calling a remote web service or querying a large database.

## Enabling Asynchronous Processing

To enable asynchronous processing for a servlet, you need to set the `asyncSupported` attribute to `true` in the `@WebServlet` annotation.

```java
@WebServlet(urlPatterns = {"/async"}, asyncSupported = true)
public class AsyncServlet extends HttpServlet {
    // ...
}
```

## Using `AsyncContext`

The `AsyncContext` interface provides the main entry point for asynchronous processing. You can get an `AsyncContext` object from the `HttpServletRequest`.

```java
AsyncContext asyncContext = request.startAsync();
```

Once you have the `AsyncContext`, you can use it to start a new thread to perform the long-running operation.

```java
asyncContext.start(() -> {
    // Perform long-running operation here
    try {
        // ...
        PrintWriter out = asyncContext.getResponse().getWriter();
        out.println("Asynchronous operation complete.");
        asyncContext.complete();
    } catch (IOException e) {
        e.printStackTrace();
    }
});
```

### Key Concepts

-   **`request.startAsync()`**: This method starts asynchronous processing and returns an `AsyncContext` object.
-   **`asyncContext.start(Runnable)`**: This method starts a new thread to perform the specified `Runnable`.
-   **`asyncContext.complete()`**: This method must be called to complete the asynchronous processing and send the response to the client.
-   **`asyncContext.dispatch(path)`**: This method can be used to dispatch the request to another resource.
