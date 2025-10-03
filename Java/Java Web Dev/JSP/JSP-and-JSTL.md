# JSP Standard Tag Library (JSTL)

The JSP Standard Tag Library (JSTL) is a collection of custom tags that provide common functionality in JSP pages, such as iteration, conditional logic, and internationalization. JSTL is an essential part of modern JSP development, as it helps to eliminate the need for scriptlets and promotes a cleaner separation of concerns.

## JSTL Libraries

JSTL is divided into five main libraries:

-   **Core:** Provides tags for iteration, conditional logic, and URL management.
-   **Formatting:** Provides tags for formatting and parsing numbers, dates, and times.
-   **SQL:** Provides tags for interacting with databases.
-   **XML:** Provides tags for processing XML documents.
-   **Functions:** Provides a set of EL functions for string manipulation and collection handling.

## Core Library

The Core library is the most commonly used JSTL library. To use it, you need to include the following directive in your JSP:

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```

### `<c:if>`

The `<c:if>` tag is used for conditional logic.

```jsp
<c:if test="${user.isAdmin}">
    <p>Welcome, admin!</p>
</c:if>
```

### `<c:forEach>`

The `<c:forEach>` tag is used for iteration.

```jsp
<ul>
    <c:forEach var="item" items="${myList}">
        <li>${item}</li>
    </c:forEach>
</ul>
```

## Formatting Library

The Formatting library is used for formatting and parsing numbers, dates, and times. To use it, you need to include the following directive in your JSP:

```jsp
<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>
```

### `<fmt:formatDate>`

The `<fmt:formatDate>` tag is used to format a date.

```jsp
<p>Today's date is: <fmt:formatDate value="${now}" pattern="MM/dd/yyyy" /></p>
```
