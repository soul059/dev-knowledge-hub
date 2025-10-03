# JSTL Core Tags

The JSTL (JSP Standard Tag Library) core tags are a collection of tags that provide fundamental functionality to simplify the development of JSP (JavaServer Pages). They help in avoiding Java scriptlets in JSP pages, leading to cleaner and more maintainable code. These tags are used for tasks such as variable manipulation, flow control, and URL management.

To use the JSTL core tags in a JSP page, you must include the following directive at the top of the file:

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
```

Here is a list of some of the most commonly used JSTL core tags with their descriptions:

### Variable and Scope Management

*   **`<c:set>`**: Sets the value of a variable in a specified scope (page, request, session, or application).
*   **`<c:remove>`**: Removes a variable from a specified scope.

### Flow Control

*   **`<c:if>`**: A simple conditional tag that processes its body content only if the supplied condition is true.
*   **`<c:choose>`, `<c:when>`, `<c:otherwise>`**: These tags work together to create a block of mutually exclusive conditional statements, similar to a `switch` statement in Java. `<c:choose>` is the parent tag, `<c:when>` represents a condition to test, and `<c:otherwise>` is the default case if no `<c:when>` condition is met.
*   **`<c:forEach>`**: Iterates over a collection of items, such as an array, list, or map.
*   **`<c:forTokens>`**: Iterates over tokens separated by a specified delimiter in a string.

### Output and URL Management

*   **`<c:out>`**: Displays the value of an expression, similar to a JSP expression tag (`<%= ... %>`), but with the added feature of escaping XML/HTML characters by default.
*   **`<c:url>`**: Creates a URL, automatically rewriting it to include the session ID if needed.
*   **`<c:redirect>`**: Redirects the response to a new URL.
*   **`<c:param>`**: Adds a parameter to a containing `<c:url>`, `<c:redirect>`, or `<c:import>` tag.

### Other Tags

*   **`<c:import>`**: Retrieves an absolute or relative URL and exposes its content to the page.
*   **`<c:catch>`**: Catches any `Throwable` exception that occurs in its body and optionally exposes it as a variable.

### Example Usage:

Here is a more comprehensive example demonstrating several JSTL core tags:

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<html>
<head>
    <title>JSTL Core Tags Example</title>
</head>
<body>

    <%-- In a real application, these variables would likely be set by a servlet --%>
    <%
        request.setAttribute("greeting", "Hello, JSTL!");
        request.setAttribute("userRole", "admin");
        request.setAttribute("dayOfWeek", java.time.LocalDate.now().getDayOfWeek().getValue());
        java.util.ArrayList<String> names = new java.util.ArrayList<>();
        names.add("Alice");
        names.add("Bob");
        names.add("Charlie");
        request.setAttribute("userNames", names);
    %>

    <%-- <c:set>: Sets a variable. Can also be used to create new variables. --%>
    <c:set var="pageTitle" value="JSTL Core Tags Demo" scope="page" />
    <h1><c:out value="${pageTitle}" /></h1>

    <%-- <c:out>: Displays content, escaping HTML by default to prevent XSS. --%>
    <p><c:out value="${greeting}" /></p>
    <c:set var="htmlContent" value="<script>alert('XSS');</script>" />
    <p>Escaped content: <c:out value="${htmlContent}" /></p>
    <p>Unescaped content (not recommended): <c:out value="${htmlContent}" escapeXml="false" /></p>


    <%-- <c:if>: Simple conditional logic. --%>
    <c:if test="${userRole == 'admin'}">
        <p>Welcome, Administrator!</p>
    </c:if>

    <%-- <c:choose>, <c:when>, <c:otherwise>: Multi-way conditional logic. --%>
    <h3>Today's Status</h3>
    <c:choose>
        <c:when test="${dayOfWeek >= 1 && dayOfWeek <= 5}">
            <p>It's a weekday. Time to work!</p>
        </c:when>
        <c:otherwise>
            <p>It's the weekend! Time to relax!</p>
        </c:otherwise>
    </c:choose>

    <%-- <c:forEach>: Iterates over a collection. --%>
    <h2>User List:</h2>
    <ul>
        <c:forEach var="name" items="${userNames}" varStatus="loop">
            <li>
                Item ${loop.count}: <c:out value="${name}" />
                (Index: ${loop.index}, Is first: ${loop.first}, Is last: ${loop.last})
            </li>
        </c:forEach>
    </ul>

    <%-- <c:forTokens>: Iterates over tokens in a string. --%>
    <h2>CSV Data:</h2>
    <c:set var="csvData" value="item1,item2,item3" />
    <ul>
        <c:forTokens items="${csvData}" delims="," var="token">
            <li><c:out value="${token}" /></li>
        </c:forTokens>
    </ul>

    <%-- <c:url> and <c:param>: Creates a URL with parameters. --%>
    <c:url var="profileUrl" value="/user/profile.jsp">
        <c:param name="userId" value="123" />
        <c:param name="theme" value="dark" />
    </c:url>
    <p>Generated URL: <a href="${profileUrl}"><c:out value="${profileUrl}" /></a></p>

</body>
</html>
```
