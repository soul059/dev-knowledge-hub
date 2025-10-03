# JSTL Functions

The JSTL functions library provides a set of standard functions that can be used with Expression Language (EL) to manipulate strings and check collections. These functions are useful for common string operations that are not directly supported by EL.

To use the JSTL functions library in a JSP page, you must include the following directive:

```jsp
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
```

Unlike other JSTL libraries, the functions library does not provide tags. Instead, it provides functions that can be called from within EL expressions.

Here is a list of the functions available in the JSTL functions library:

### String Manipulation Functions

*   **`fn:contains(string, substring)`**: Returns `true` if the input string contains the specified substring.
*   **`fn:containsIgnoreCase(string, substring)`**: Returns `true` if the input string contains the specified substring, ignoring case.
*   **`fn:startsWith(string, prefix)`**: Returns `true` if the input string starts with the specified prefix.
*   **`fn:endsWith(string, suffix)`**: Returns `true` if the input string ends with the specified suffix.
*   **`fn:escapeXml(string)`**: Escapes characters that could be interpreted as XML markup.
*   **`fn:indexOf(string, substring)`**: Returns the index of the first occurrence of a substring within a string.
*   **`fn:join(array, separator)`**: Joins the elements of an array into a single string, separated by the specified separator.
*   **`fn:length(item)`**: Returns the length of a string or the number of items in a collection.
*   **`fn:replace(string, before, after)`**: Replaces all occurrences of a substring with another string.
*   **`fn:split(string, separator)`**: Splits a string into an array of substrings.
*   **`fn:substring(string, begin, end)`**: Returns a subset of a string.
*   **`fn:substringAfter(string, substring)`**: Returns the part of a string that comes after the specified substring.
*   **`fn:substringBefore(string, substring)`**: Returns the part of a string that comes before the specified substring.
*   **`fn:toLowerCase(string)`**: Converts a string to lowercase.
*   **`fn:toUpperCase(string)`**: Converts a string to uppercase.
*   **`fn:trim(string)`**: Removes leading and trailing whitespace from a string.

### Collection Functions

*   **`fn:length(collection)`**: Returns the number of elements in a collection. This is the same function used for string length.

### Example Usage:

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>

<html>
<head>
    <title>JSTL Functions Example</title>
</head>
<body>

    <c:set var="str1" value="  Welcome to JSTL Functions!  "/>
    <c:set var="str2" value="This is a sample string."/>

    <h2>String Manipulation</h2>
    <p>Original String: "${str1}"</p>
    <p>Length: ${fn:length(str1)}</p>
    <p>Trimmed: "${fn:trim(str1)}"</p>
    <p>Uppercase: ${fn:toUpperCase(str1)}</p>
    <p>Lowercase: ${fn:toLowerCase(str1)}</p>
    <p>Does it contain 'JSTL'? ${fn:contains(str1, 'JSTL')}</p>
    <p>Index of 'to': ${fn:indexOf(str1, 'to')}</p>
    <p>Substring after 'Welcome': "${fn:substringAfter(str1, 'Welcome')}"</p>
    <p>Replaced 'sample' with 'simple': "${fn:replace(str2, 'sample', 'simple')}"</p>

    <h2>Splitting and Joining</h2>
    <c:set var="csv" value="one,two,three,four"/>
    <c:set var="parts" value="${fn:split(csv, ',')}" />
    <p>Original CSV: ${csv}</p>
    <p>Split array:
        <ul>
            <c:forEach var="part" items="${parts}">
                <li>${part}</li>
            </c:forEach>
        </ul>
    </p>
    <p>Joined back with ';': ${fn:join(parts, ';')}</p>

</body>
</html>
```