# Jakarta WebSocket API (JSR 356)

Jakarta WebSocket is an API that allows developers to add WebSocket support to their Jakarta EE applications. It enables real-time, two-way communication between a client (like a web browser) and a server.

## Key Features

*   **Annotation-driven**: You can create WebSocket endpoints easily using annotations.
*   **Lifecycle Events**: The API provides annotations (`@OnOpen`, `@OnMessage`, `@OnClose`, `@OnError`) to manage the lifecycle of a WebSocket connection.
*   **Integration with Jakarta EE**: It is well-integrated with other Jakarta EE technologies like CDI.

## Maven Dependency

To use Jakarta WebSocket, you need the following dependency:

```xml
<dependency>
    <groupId>jakarta.websocket</groupId>
    <artifactId>jakarta.websocket-api</artifactId>
    <version>2.1.0</version>
    <scope>provided</scope>
</dependency>
```

## Creating a WebSocket Endpoint

A WebSocket endpoint is a Java class annotated with `@ServerEndpoint`. This annotation declares that the class is a WebSocket endpoint and specifies the URL where it will be available.

### Lifecycle Annotations

*   `@OnOpen`: A method annotated with `@OnOpen` is called when a new WebSocket connection is established. It typically takes a `Session` object as a parameter.
*   `@OnMessage`: This method is called when a message is received from a client. It can receive the message as a `String`, `byte[]`, or other types.
*   `@OnClose`: Called when the WebSocket connection is closed.
*   `@OnError`: Called when an error occurs during the WebSocket communication.

### Example: A Simple Chat Server Endpoint

This example shows a simple WebSocket endpoint that broadcasts messages to all connected clients.

```java
import jakarta.websocket.*;
import jakarta.websocket.server.ServerEndpoint;
import java.io.IOException;
import java.util.Collections;
import java.util.HashSet;
import java.util.Set;

@ServerEndpoint("/chat")
public class ChatServerEndpoint {

    private static Set<Session> clients = Collections.synchronizedSet(new HashSet<>());

    @OnOpen
    public void onOpen(Session session) {
        clients.add(session);
        broadcast("User [" + session.getId() + "] has joined the chat.");
    }

    @OnMessage
    public void onMessage(String message, Session session) {
        broadcast("User [" + session.getId() + "]: " + message);
    }

    @OnClose
    public void onClose(Session session) {
        clients.remove(session);
        broadcast("User [" + session.getId() + "] has left the chat.");
    }

    @OnError
    public void onError(Throwable error) {
        error.printStackTrace();
    }

    private void broadcast(String message) {
        for (Session client : clients) {
            try {
                client.getBasicRemote().sendText(message);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## Client-Side Implementation

Here is a simple HTML and JavaScript client that connects to the `/chat` endpoint.

```html
<!DOCTYPE html>
<html>
<head>
    <title>WebSocket Chat</title>
</head>
<body>
    <h1>WebSocket Chat</h1>
    <textarea id="messages" rows="10" cols="50" readonly></textarea><br/>
    <input type="text" id="messageBox" size="50"/>
    <button onclick="sendMessage()">Send</button>

    <script>
        const wsUri = "ws://" + document.location.host + document.location.pathname.substring(0, document.location.pathname.indexOf("/faces")) + "/chat";
        const websocket = new WebSocket(wsUri);
        const messages = document.getElementById("messages");

        websocket.onopen = function(evt) {
            messages.value += "Connected to chat server.\n";
        };

        websocket.onmessage = function(evt) {
            messages.value += evt.data + "\n";
        };

        websocket.onerror = function(evt) {
            messages.value += "Error: " + evt.data + "\n";
        };

        function sendMessage() {
            const messageBox = document.getElementById("messageBox");
            websocket.send(messageBox.value);
            messageBox.value = "";
        }
    </script>
</body>
</html>

