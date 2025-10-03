# Servlet Example

This section provides a step-by-step explanation of a simple servlet example that demonstrates how to handle a form submission.

## The Goal

The goal of this example is to create a web application that:

1.  Displays an HTML form with a text field for the user to enter their name.
2.  When the user submits the form, a servlet processes the request.
3.  The servlet reads the user's name from the request and sends back an HTML response that greets the user.

## The Code

This example consists of two files:

1.  `index.html`: The HTML form.
2.  `GreetingServlet.java`: The servlet that processes the form submission.

### 1. `index.html`

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

**Explanation:**

-   **`<form action="greet" method="post">`**: This defines an HTML form that will be submitted to the URL `greet` using the HTTP POST method.
-   **`<input type="text" name="name">`**: This creates a text input field where the user can enter their name. The `name` attribute is important, as it will be used to retrieve the value of the input field in the servlet.
-   **`<input type="submit" value="Submit">`**: This creates a submit button that, when clicked, will submit the form.

### 2. `GreetingServlet.java`

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

**Explanation:**

-   **`@WebServlet("/greet")`**: This annotation maps the `GreetingServlet` to the URL pattern `/greet`. This means that any request to the URL `greet` will be handled by this servlet.
-   **`extends HttpServlet`**: This indicates that `GreetingServlet` is an HTTP servlet.
-   **`doPost(HttpServletRequest request, HttpServletResponse response)`**: This method is called by the web container to handle POST requests. It receives the `HttpServletRequest` and `HttpServletResponse` objects as arguments.
-   **`String name = request.getParameter("name");`**: This line retrieves the value of the `name` parameter from the request. The `name` parameter corresponds to the `name` attribute of the input field in the HTML form.
-   **`response.setContentType("text/html");`**: This sets the content type of the response to `text/html`, which tells the client to render the response as HTML.
-   **`PrintWriter out = response.getWriter();`**: This gets a `PrintWriter` object that can be used to send character text to the client.
-   **`out.println(...)`**: These lines write the HTML response to the client.

## How It Works

1.  The user opens the `index.html` page in their browser.
2.  The user enters their name in the text field and clicks the "Submit" button.
3.  The browser sends a POST request to the URL `greet`.
4.  The web container receives the request and, based on the `@WebServlet("/greet")` annotation, invokes the `doPost()` method of the `GreetingServlet`.
5.  The `doPost()` method retrieves the user's name from the request, creates a greeting message, and sends it back to the browser as an HTML response.
6.  The browser receives the HTML response and displays it to the user.
