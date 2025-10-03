# JSP Scripting Elements

JSP (JavaServer Pages) scripting elements allow you to embed Java code directly within your HTML pages, enabling dynamic content generation. While modern development often favors alternatives like the JSP Standard Tag Library (JSTL) to separate logic from presentation, understanding scripting elements is fundamental to JSP.

There are three main types of scripting elements:

### 1. Scriptlet Tag

A scriptlet tag allows you to insert arbitrary Java code into the `_jspService()` method of the servlet generated from the JSP page. This code is executed every time a request is made to the page.

**Syntax:**

```jsp
<% java code %>
```

**Use Cases:**

*   Implementing conditional logic (e.g., `if-else` statements).
*   Iterating over data collections to dynamically generate HTML.
*   Calling Java methods and performing business logic.

**Example:**

```jsp
<html>
<body>
  <%
    String name = request.getParameter("name");
    if (name != null && !name.isEmpty()) {
      out.println("<h1>Hello, " + name + "!</h1>");
    } else {
      out.println("<h1>Hello, Guest!</h1>");
    }
  %>
</body>
</html>
```

In this example, the scriptlet checks for a request parameter and prints a personalized greeting.

### 2. Expression Tag

The expression tag is used to evaluate a Java expression, convert the result to a String, and insert it directly into the output stream of the response. It's a shorthand for `out.print()`.

**Syntax:**

```jsp
<%= expression %>
```

You should not end the expression with a semicolon.

**Use Cases:**

*   Displaying the value of variables.
*   Printing the return value of a method.
*   Showing user-specific data, like a username or the current date.

**Example:**

```jsp
<html>
<body>
  <p>The current time is: <%= new java.util.Date() %></p>
</body>
</html>
```

This code will display the current date and time on the web page.

### 3. Declaration Tag

A declaration tag is used to declare variables and methods at the class level of the generated servlet. This means the declared members are placed outside the `_jspService()` method and are accessible throughout the JSP page.

**Syntax:**

```jsp
<%! declaration %>
```

**Key Differences from Scriptlets:**

*   **Scope:** Variables declared in a declaration tag have class-level scope, while variables in a scriptlet have local scope within the `_jspService()` method.
*   **Content:** You can declare both methods and variables within a declaration tag, whereas a scriptlet can only declare variables (local to the service method).

**Example:**

```jsp
<html>
<body>
  <%! 
    private int hitCount = 0;

    public int getHitCount() {
      hitCount++;
      return hitCount;
    }
  %>
  <p>This page has been visited <%= getHitCount() %> times.</p>
</body>
</html>
```

In this example, `hitCount` is a class-level variable that persists across multiple requests to the page, allowing it to function as a simple visit counter.
