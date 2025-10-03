# Server-Sent Events (SSE) in Servlets

Server-Sent Events (SSE) is a technology that allows a server to push real-time updates to a client over a single HTTP connection. It is a simpler alternative to WebSockets for one-way communication from the server to the client.

## Implementing an SSE Servlet

To implement an SSE servlet, you need to set the `Content-Type` of the response to `text/event-stream` and keep the connection open to send events to the client.

```java
@WebServlet("/sse")
public class SSEServlet extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/event-stream");
        response.setCharacterEncoding("UTF-8");

        PrintWriter writer = response.getWriter();

        for (int i = 0; i < 10; i++) {
            writer.write("data: " + System.currentTimeMillis() + "\n\n");
            writer.flush();
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        writer.close();
    }
}
```

### Key Concepts

-   **`Content-Type: text/event-stream`**: This header tells the client that the response is a stream of events.
-   **Event Formatting**: Each event is a block of text that ends with a pair of newlines (`\n\n`). The `data:` field specifies the event payload.
-   **`writer.flush()`**: This is important to ensure that the events are sent to the client immediately.

## Client-Side Implementation

On the client side, you can use the `EventSource` API to connect to the SSE servlet and receive events.

```javascript
const eventSource = new EventSource("/sse");

eventSource.onmessage = function(event) {
    console.log("New message", event.data);
};

eventSource.onerror = function(err) {
    console.error("EventSource failed:", err);
};
```

```
