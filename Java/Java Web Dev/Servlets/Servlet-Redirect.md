# Servlet Redirect

A redirect is a way to send the user to a different URL than the one they requested. This is a common technique used in web development for various purposes, such as after a form submission, for load balancing, or to move to a different website.

In the Servlet API, you can perform a redirect using the `sendRedirect()` method of the `HttpServletResponse` object.

## How Redirects Work

When a servlet sends a redirect, it sends an HTTP response with a `302 Found` status code and a `Location` header that contains the new URL. The browser then makes a new request to the URL specified in the `Location` header.

This process is transparent to the user, who will simply see the new page in their browser.

## `sendRedirect()` Method

The `sendRedirect()` method takes a `String` argument that represents the URL to which the user should be redirected. The URL can be relative or absolute.

```java
void sendRedirect(String location) throws IOException;
```

**Example:**

```java
// Redirect to a relative URL
response.sendRedirect("dashboard");

// Redirect to an absolute URL
response.sendRedirect("https://www.google.com");
```

## Redirect Example

This example demonstrates a servlet that processes a login form. If the login is successful, the user is redirected to a dashboard page. Otherwise, they are redirected back to the login page with an error message.

**`LoginServlet.java`:**

```java
package com.example;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

@WebServlet("/login")
public class LoginServlet extends HttpServlet {

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        String username = request.getParameter("username");
        String password = request.getParameter("password");

        if ("admin".equals(username) && "password".equals(password)) {
            // Create a session and store the username
            HttpSession session = request.getSession();
            session.setAttribute("username", username);

            // Redirect to the dashboard
            response.sendRedirect("dashboard");
        } else {
            // Redirect back to the login page with an error message
            response.sendRedirect("login.jsp?error=1");
        }
    }
}
```

**`DashboardServlet.java`:**

```java
package com.example;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

@WebServlet("/dashboard")
public class DashboardServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        HttpSession session = request.getSession(false);

        if (session != null && session.getAttribute("username") != null) {
            String username = (String) session.getAttribute("username");

            response.setContentType("text/html");
            PrintWriter out = response.getWriter();
            out.println("<html><body>");
            out.println("<h1>Welcome, " + username + "!</h1>");
            out.println("</body></html>");
        } else {
            // Redirect to the login page if the user is not logged in
            response.sendRedirect("login.jsp");
        }
    }
}
```

## `sendRedirect()` vs. `forward()`

A common point of confusion for beginners is the difference between `sendRedirect()` and `forward()`.

| Feature            | `sendRedirect()`                                    | `forward()`                                         |
| ------------------ | --------------------------------------------------- | --------------------------------------------------- |
| **Location**       | Can be a URL on the same server or a different server | Must be a resource on the same server               |
| **Client vs. Server** | The client is aware of the new URL                  | The client is not aware of the new URL              |
| **Request**        | A new request is created for the new URL            | The original request is forwarded to the new resource |
| **URL in Browser** | The URL in the browser's address bar changes        | The URL in the browser's address bar does not change |
| **Performance**    | Slower, as it involves an extra round trip          | Faster, as it is all done on the server side        |

In general, you should use `sendRedirect()` when you want to change the URL in the browser and `forward()` when you want to delegate the processing of a request to another resource on the same server.
