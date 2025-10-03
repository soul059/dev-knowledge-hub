# JSP Directives

JSP (JavaServer Pages) directives are messages that provide instructions to the web container on how to translate a JSP page into a servlet. They control the overall processing and structure of the page. There are three main types of directives: `page`, `include`, and `taglib`.

### 1. Page Directive

The `page` directive defines attributes that apply to the entire JSP page. While it can be placed anywhere, it is conventionally located at the top of the JSP file.

**Syntax:**

```jsp
<%@ page attribute="value" %>
```

**Common Attributes:**

*   **`import`**: Specifies a comma-separated list of Java classes or packages to be imported, similar to the `import` statement in Java.
*   **`contentType`**: Defines the MIME type and character encoding of the response.
*   **`session`**: A boolean value (`true` or `false`) that indicates whether the page participates in an HTTP session. The default is `true`.
*   **`errorPage`**: Specifies a relative URL to another JSP page that will handle any uncaught exceptions thrown from the current page.
*   **`isErrorPage`**: A boolean value that indicates if the current page can act as an error page for another JSP page.
*   **`language`**: Defines the scripting language used in the page. The default is "java".

**Example:**

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" import="java.util.Date" %>
```

### 2. Include Directive

The `include` directive is used to merge the content of another file (such as HTML, JSP, or text files) into the current JSP page during the translation phase. This is useful for reusing common elements like headers, footers, or navigation bars.

**Syntax:**

```jsp
<%@ include file="relativeURL" %>
```

The `file` attribute specifies the relative path to the resource to be included. If only a filename is given, the JSP compiler assumes the file is in the same directory.

**Example:**

Imagine a `header.jsp` and a `footer.jsp`. You can include them in your main page like this:

```jsp
<%@ include file="header.jsp" %>

<main>
  <p>This is the main content of the page.</p>
</main>

<%@ include file="footer.jsp" %>
```

### 3. Taglib Directive

The `taglib` directive declares that the JSP page will use a custom tag library, such as JSTL (JSP Standard Tag Library). It specifies the location of the tag library and a prefix to identify the tags from that library.

**Syntax:**

```jsp
<%@ taglib uri="libraryURI" prefix="tagPrefix" %>
```

*   **`uri`**: A Uniform Resource Identifier that uniquely locates the Tag Library Descriptor (`.tld`) file.
*   **`prefix`**: A short name that is used to distinguish which tag library a custom tag belongs to.

**Example:**

This example uses the JSTL core library to conditionally display a message.

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>

<html>
<head>
    <title>Taglib Directive Example</title>
</head>
<body>
    <c:if test="${not empty param.user}">
        Welcome, ${param.user}!
    </c:if>
    <c:if test="${empty param.user}">
        Welcome, Guest!
    </c:if>
</body>
</html>
```
