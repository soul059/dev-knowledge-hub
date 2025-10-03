# Introduction to JSTL

JSTL, which stands for Jakarta Standard Tag Library (formerly JavaServer Pages Standard Tag Library), is a collection of custom tags that simplify the development of JSP pages. It provides a way to embed logic within a JSP page using tags instead of embedding raw Java code in scriptlets (`<% ... %>`).

### Why Use JSTL?

Using JSTL offers several advantages over traditional JSP scriptlets:

*   **Simplifies Development**: JSTL provides a set of pre-built tags for common tasks, which means developers don't have to write as much Java code.
*   **Code Reusability**: The tags can be reused across multiple JSP pages and web applications.
*   **Readability and Maintainability**: It separates business logic from the presentation layer, making the code cleaner and easier for both developers and designers to understand and maintain.
*   **Standardization**: As part of the Java EE standard, JSTL provides reliable and consistent behavior across different application servers.
*   **No Need for Scriptlets**: It helps to avoid the use of cumbersome scriptlet tags.

### JSTL Tag Libraries

JSTL is organized into five main functional libraries, each with its own namespace and prefix.

| Library             | Prefix | Description                                                                                                         |
| ------------------- | ------ | ------------------------------------------------------------------------------------------------------------------- |
| **[Core](Core-Tags.md)**            | `c`    | Provides essential tags for iteration, conditional logic, exception handling, and URL management. This is the most commonly used library. |
| **[Formatting (I18N)](Formatting-Tags.md)** | `fmt`  | Used for formatting and parsing numbers, dates, and times, and for internationalization (i18n) support.             |
| **[SQL](SQL-Tags.md)**             | `sql`  | Contains tags for interacting with relational databases. However, using this library is often discouraged in real-world applications for security and design reasons. |
| **[XML](XML-Tags.md)**             | `x`    | Provides tags for creating and manipulating XML documents, including parsing and flow control based on XPath expressions. |
| **[Functions](Functions-Tags.md)**       | `fn`   | Offers a set of functions for string manipulation, such as finding the length of a string or converting it to lowercase. |

### How to Use JSTL

To use a JSTL library in a JSP page, you must first include the necessary JSTL JAR files in your web application's `WEB-INF/lib` directory. Then, you need to declare the tag library at the top of your JSP file using the `<%@ taglib %>` directive.

Here is a basic example of using the JSTL core library to iterate over a list of items and display them:

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<!DOCTYPE html>
<html>
<head>
    <title>JSTL Core Example</title>
</head>
<body>

    <h2>Fruits</h2>
    <ul>
        <%-- Assuming 'fruitList' is a List set as a request attribute from a servlet --%>
        <c:forEach var="fruit" items="${fruitList}">
            <li><c:out value="${fruit}" /></li>
        </c:forEach>
    </ul>

</body>
</html>
```

In this example:

*   The `<%@ taglib %>` directive imports the JSTL core library and assigns it the prefix `c`.
*   The `<c:forEach>` tag iterates over a collection named `fruitList`.
*   The `<c:out>` tag safely prints the value of each item in the list.
