# JSP Implicit Objects

In JavaServer Pages (JSP), implicit objects are a set of Java objects that the JSP container makes available to developers on each page. These objects can be used directly in scriptlets without being declared or instantiated first. They are created by the web container during the translation phase when a JSP is converted into a servlet.

There are nine JSP implicit objects:

### 1. `request`

This object is an instance of `javax.servlet.http.HttpServletRequest`. It represents the client's request and is created for each request. You can use it to get request information such as parameters, headers, and cookies.

**Example:**

To get a parameter from a form:

```jsp
<%
  String name = request.getParameter("userName");
  out.println("Welcome, " + name);
%>
```

### 2. `response`

An instance of `javax.servlet.http.HttpServletResponse`, this object represents the response that will be sent back to the client. It can be used to manipulate the response, such as redirecting to another page or adding cookies.

**Example:**

To redirect a user to another URL:

```jsp
<%
  response.sendRedirect("https://www.google.com");
%>
```

### 3. `out`

The `out` object is an instance of `javax.servlet.jsp.JspWriter`. It is used to write content to the response stream that is sent to the client.

**Example:**

To print content to the page:

```jsp
<%
  out.println("Hello, World!");
%>
```

### 4. `session`

This object is an instance of `javax.servlet.http.HttpSession`. It is used to store user-specific data across multiple pages for the duration of a user's session.

**Example:**

To store and retrieve a user's name from the session:

```jsp
<%
  // Store data in the session
  session.setAttribute("user", "JohnDoe");

  // Retrieve data from the session
  String userName = (String) session.getAttribute("user");
  out.println("Username from session: " + userName);
%>
```

### 5. `application`

The `application` object is an instance of `javax.servlet.ServletContext`. It is created when the web application is deployed and is available to all JSP pages. It's used to share application-wide data and get initialization parameters.

**Example:**

To count the number of times a page has been visited across the entire application:

```jsp
<%
  Integer hitCount = (Integer) application.getAttribute("hitCounter");
  if (hitCount == null) {
    hitCount = 0;
  }
  hitCount++;
  application.setAttribute("hitCounter", hitCount);
  out.println("This page has been visited " + hitCount + " times.");
%>
```

### 6. `config`

An instance of `javax.servlet.ServletConfig`, this object provides configuration information for the current JSP page. It's commonly used to access initialization parameters defined in the `web.xml` file.

**Example:**

To get an initialization parameter:

```jsp
<%
  String adminEmail = config.getInitParameter("adminEmail");
  out.println("Admin email: " + adminEmail);
%>
```

### 7. `pageContext`

This object is an instance of `javax.servlet.jsp.PageContext`. It provides access to all other implicit objects and allows you to manage attributes in different scopes (page, request, session, and application).

**Example:**

To set an attribute in the session scope:

```jsp
<%
  pageContext.setAttribute("user", "JaneDoe", PageContext.SESSION_SCOPE);
%>
```

### 8. `page`

The `page` object is a reference to the current instance of the servlet generated from the JSP page. It is an instance of `java.lang.Object` and is not frequently used in practice.

### 9. `exception`

This object is an instance of `java.lang.Throwable` and is only available on JSP pages that are designated as error pages. It holds the exception that was thrown from another JSP page.

**Example:**

On an error page (`error.jsp`), you can display the exception details:

```jsp
<%@ page isErrorPage="true" %>
<html>
<head>
  <title>Error</title>
</head>
<body>
  <h2>An error occurred:</h2>
  <p><%= exception.getMessage() %></p>
</body>
</html>
```
