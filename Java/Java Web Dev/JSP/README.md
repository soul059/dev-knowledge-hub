# JavaServer Pages (JSP)

JavaServer Pages (JSP) is a server-side programming technology that enables the creation of dynamic, platform-independent web pages. JSPs are essentially HTML pages with embedded Java code. The key idea behind JSP is to separate the presentation of a web page from the business logic.

## JSP Lifecycle

The JSP lifecycle is managed by the web container and consists of the following phases:

1.  **Page Translation:** When a request is made for a JSP, the web container first checks if the JSP has been compiled or if the compiled version is older than the source file. If so, it translates the JSP into a Java servlet. This is a one-time process.

2.  **Page Compilation:** The container then compiles the generated servlet source code into a `.class` file.

3.  **Class Loading:** The compiled servlet class is loaded into the container's memory.

4.  **Instantiation:** The container creates an instance of the servlet class.

5.  **Initialization:** The container calls the `jspInit()` method of the servlet instance. This method is called only once and can be used for one-time initialization tasks.

6.  **Request Handling:** For each client request, the container calls the `_jspService()` method of the servlet instance. This method is responsible for generating the response.

7.  **Destruction:** When the container is shut down, it calls the `jspDestroy()` method to release any resources held by the JSP.

## JSP Example

This example shows a simple JSP that displays the current date and time.

**`datetime.jsp`:**

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ page import="java.util.Date" %>
<!DOCTYPE html>
<html>
<head>
    <title>JSP Date/Time</title>
</head>
<body>
    <h1>Current Date and Time</h1>
    <p><%= new Date() %></p>
</body>
</html>
```

## JSP Elements

### Directives
Directives provide instructions to the JSP container. They have the syntax `<%@ directive attribute="value" %>`.

*   **`page` directive:** Defines page-dependent attributes, such as the scripting language, error page, and buffering requirements.
    ```jsp
    <%@ page import="java.util.List, java.util.ArrayList" %>
    ```
*   **`include` directive:** Includes a file during the translation phase.
    ```jsp
    <%@ include file="header.html" %>
    ```
*   **`taglib` directive:** Declares a tag library, containing custom tags, that is used in the page.
    ```jsp
    <%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
    ```

### Scripting Elements
Scripting elements allow you to embed Java code in your JSPs.

*   **Scriptlets:** `<% ... %>` - A block of Java code that is executed when the JSP is processed.
    ```jsp
    <% 
        for (int i = 0; i < 5; i++) {
            out.println("Hello, World! " + i);
        }
    %>
    ```
*   **Expressions:** `<%= ... %>` - A Java expression that is evaluated and its result is written to the output.
    ```jsp
    <p>Your IP address is <%= request.getRemoteAddr() %></p>
    ```
*   **Declarations:** `<%! ... %>` - Used to declare variables and methods at the class level of the generated servlet.
    ```jsp
    <%! private int hitCount = 0; %>
    ```

### JSP Actions
JSP actions are XML tags that invoke built-in functionality in the JSP container.

*   **`<jsp:include>`:** Includes a resource (JSP, HTML, or servlet) at request time.
    ```jsp
    <jsp:include page="header.jsp" />
    ```
*   **`<jsp:forward>`:** Forwards the request to another resource.
    ```jsp
    <jsp:forward page="/login" />
    ```
*   **`<jsp:useBean>`:** Creates or retrieves a JavaBean.
    ```jsp
    <jsp:useBean id="user" class="com.example.User" scope="session" />
    ```
*   **`<jsp:setProperty>`:** Sets a property of a JavaBean.
    ```jsp
    <jsp:setProperty name="user" property="username" value="testuser" />
    ```
*   **`<jsp:getProperty>`:** Gets a property of a JavaBean.
    ```jsp
    <jsp:getProperty name="user" property="username" />
    ```

## Advanced Topics

### Custom Tags

JSP allows you to create your own custom tags to encapsulate complex logic and promote reusability.

### JSTL (JSP Standard Tag Library)

JSTL provides a rich set of tags for common tasks, such as iteration, conditional logic, and internationalization, helping to eliminate the need for Java code in your JSPs.