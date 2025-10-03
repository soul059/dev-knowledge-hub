# JSTL XML Tags

The JSTL (JSP Standard Tag Library) XML tags are used to create and manipulate XML documents within JSP pages. These tags utilize XPath expressions to select and process parts of an XML document.

To use the JSTL XML library, you need to include the following directive at the top of your JSP page:

```jsp
<%@ taglib prefix="x" uri="http://java.sun.com/jsp/jstl/xml" %>
```

Here are the main JSTL XML tags and their functions:

### Core XML Tags:

*   **`<x:parse>`**: This tag parses XML data from a string, a URL, or the tag's body and stores the result in a scoped variable.
*   **`<x:out>`**: Similar to the `<c:out>` tag in the core library, this tag evaluates an XPath expression and outputs the result to the page.
*   **`<x:set>`**: This tag evaluates an XPath expression and sets the result to a scoped variable.

### Flow Control Tags:

*   **`<x:if>`**: This tag evaluates an XPath expression as a boolean. If the result is true, the body content of the tag is processed.
*   **`<x:forEach>`**: This tag iterates over a collection of nodes selected by an XPath expression.
*   **`<x:choose>`, `<x:when>`, `<x:otherwise>`**: These tags work together to provide conditional logic, similar to a switch statement. `<x:when>` evaluates an XPath expression, and if it's true, its body is processed. If no `<x:when>` condition is met, the `<x:otherwise>` block is executed.

### Transformation Tags:

*   **`<x:transform>`**: This tag applies an XSLT (Extensible Stylesheet Language Transformations) stylesheet to an XML document.
*   **`<x:param>`**: Used within the `<x:transform>` tag, this tag sets a parameter in the XSLT stylesheet.

### Example Usage:

Suppose you have the following XML data. You can embed it directly in the JSP for this example, but in a real application, it might come from a file or a web service.

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib prefix="x" uri="http://java.sun.com/jsp/jstl/xml" %>

<html>
<head>
    <title>JSTL XML Tags Example</title>
</head>
<body>

    <%-- 1. Parse XML data --%>
    <c:set var="xmlData">
        <books>
            <book>
                <title>The Lord of the Rings</title>
                <author>J.R.R. Tolkien</author>
                <year>1954</year>
            </book>
            <book>
                <title>A Game of Thrones</title>
                <author>George R.R. Martin</author>
                <year>1996</year>
            </book>
            <book>
                <title>Dune</title>
                <author>Frank Herbert</author>
                <year>1965</year>
            </book>
        </books>
    </c:set>

    <x:parse xml="${xmlData}" var="doc"/>

    <h2>All Book Titles</h2>
    <ul>
        <%-- 2. Use x:forEach to iterate through nodes --%>
        <x:forEach select="$doc/books/book/title" var="titleNode">
            <li><x:out select="$titleNode" /></li>
        </x:forEach>
    </ul>

    <h2>Book published after 1990</h2>
    <ul>
        <%-- 3. Use x:if for conditional logic with XPath --%>
        <x:forEach select="$doc/books/book">
            <x:if select="year > 1990">
                <li>
                    <x:out select="title" /> by <x:out select="author" />
                </li>
            </x:if>
        </x:forEach>
    </ul>

    <h2>Set a variable with XPath</h2>
    <%-- 4. Use x:set to store a result --%>
    <x:set select="$doc/books/book[1]/author" var="firstAuthor"/>
    <p>The author of the first book is: <strong><c:out value="${firstAuthor}"/></strong></p>

</body>
</html>
```
