# Model-View-Controller (MVC) Pattern

The Model-View-Controller (MVC) pattern is a software architectural pattern that separates the representation of information from the user's interaction with it. The central idea is to break down an application into three interconnected parts, promoting organized and modular code.

## MVC Components

### Model
The Model is the heart of the application. It represents the application's data and business logic. It is responsible for managing the state of the application and enforcing its business rules.

*   **Responsibilities:**
    *   Encapsulates application data.
    *   Implements business logic and validation rules.
    *   Notifies the view of any changes in its state.
    *   Is completely independent of the view and controller.

### View
The View is the user interface of the application. It is responsible for presenting the data from the model to the user.

*   **Responsibilities:**
    *   Renders the user interface.
    *   Displays data from the model.
    *   Forwards user input to the controller.
    *   Should not contain any business logic.

### Controller
The Controller acts as an intermediary between the Model and the View. It receives user input from the view and updates the model accordingly. It also selects the appropriate view to display.

*   **Responsibilities:**
    *   Handles user requests.
    *   Interacts with the model to update its state.
    *   Selects the appropriate view to render.
    *   Can be seen as the "brains" of the application.

## MVC in Java Web Applications

In a typical Java web application, the MVC components are implemented as follows:

*   **Model:** Plain Old Java Objects (POJOs) or JavaBeans that represent the application data.
*   **View:** JSPs, or other templating engines like Thymeleaf or FreeMarker, are used to display the data from the model.
*   **Controller:** Servlets are used to handle user requests, interact with the model, and select the appropriate view.

## MVC Example: Login Application

This example demonstrates a simple login application using the MVC pattern.

**1. `User.java` (Model):**

```java
package com.example.model;

public class User {
    private String username;
    private String password;

    // Getters and setters
}
```

**2. `LoginController.java` (Controller):**

```java
package com.example.controller;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import com.example.model.User;

@WebServlet("/login")
public class LoginController extends HttpServlet {

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        User user = new User();
        user.setUsername(request.getParameter("username"));
        user.setPassword(request.getParameter("password"));

        // In a real application, you would have a service layer to validate the user's credentials.

        if ("admin".equals(user.getUsername()) && "password".equals(user.getPassword())) {
            request.setAttribute("user", user);
            request.getRequestDispatcher("/welcome.jsp").forward(request, response);
        } else {
            response.sendRedirect("login.jsp?error=1");
        }
    }
}
```

**3. `login.jsp` (View):**

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
</head>
<body>
    <h1>Login</h1>
    <form action="login" method="post">
        Username: <input type="text" name="username"><br>
        Password: <input type="password" name="password"><br>
        <input type="submit" value="Login">
    </form>
    <c:if test="${param.error != null}">
        <p style="color:red;">Invalid username or password.</p>
    </c:if>
</body>
</html>
```

**4. `welcome.jsp` (View):**

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
    <title>Welcome</title>
</head>
<body>
    <h1>Welcome, ${user.username}!</h1>
</body>
</html>
```

### Flow of Control

1.  The user enters their username and password in `login.jsp` and clicks the "Login" button.
2.  The form is submitted to the `LoginController` servlet.
3.  The `LoginController` retrieves the username and password from the request and creates a `User` object (the model).
4.  The controller then validates the user's credentials.
5.  If the credentials are valid, the controller sets the `user` object as an attribute in the request and forwards the request to `welcome.jsp`.
6.  `welcome.jsp` retrieves the `user` object from the request and displays a welcome message.
7.  If the credentials are invalid, the controller redirects the user back to `login.jsp` with an error message.

## Related Patterns

### Front Controller

The Front Controller pattern provides a centralized entry point for all incoming requests, which is a common approach in MVC frameworks.

### View Helper

The View Helper pattern helps to offload business logic from the views, keeping them clean and focused on presentation.