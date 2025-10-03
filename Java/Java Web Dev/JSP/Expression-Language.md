# JSP Expression Language (EL)

JSP Expression Language (EL) is a scripting language that simplifies the process of accessing and manipulating data in JavaServer Pages (JSP). Introduced in JSP 2.0, it provides a simpler syntax for retrieving data from JavaBeans components and other objects available in a JSP page.

The main goal of JSP EL is to separate the presentation layer (HTML/JSP) from the business logic (Java code), making the code cleaner and easier to maintain. Instead of using Java scriptlets (`<% ... %>`), you can use EL expressions to access data.

### Syntax

The basic syntax of a JSP EL expression is:

```jsp
${expression}
```

When the JSP page is processed, the expression inside the `${...}` is evaluated, and the result is inserted into the output.

### Key Features and Usage

Here are some of the most common uses and features of JSP EL:

#### Accessing JavaBean Properties

EL provides a simple way to access properties of JavaBeans. For example, to access the `name` property of a `user` bean, you would use:

```jsp
${user.name}
```

#### Operators

JSP EL supports a variety of operators:

*   **Arithmetic:** `+`, `-`, `*`, `/` (or `div`), `%` (or `mod`).
*   **Logical:** `&&` (or `and`), `||` (or `or`), `!` (or `not`).
*   **Relational:** `==` (or `eq`), `!=` (or `ne`), `<` (or `lt`), `>` (or `gt`), `<=` (or `le`), `>=` (or `ge`).
*   **Conditional:** `A ? B : C`.
*   **Empty:** The `empty` operator can be used to check if a value is null or an empty string or collection.

**Example of operators:**

```jsp
${(10 + 5) * 2} <%-- Result: 30 --%>
${user.isAdmin && user.isLoggedIn}
${cart.total > 100 ? 'Free Shipping' : 'Standard Shipping'}
${empty user.email ? 'Email not provided' : user.email}
```

#### Implicit Objects

JSP EL provides a set of implicit objects that allow access to various scopes and request parameters:

*   **`pageScope`**: A map of all attributes in the page scope.
*   **`requestScope`**: A map of all attributes in the request scope.
*   **`sessionScope`**: A map of all attributes in the session scope.
*   **`applicationScope`**: A map of all attributes in the application (servlet context) scope.
*   **`param`**: A map of request parameters, where the key is the parameter name and the value is the first parameter value.
*   **`paramValues`**: A map of request parameters, where the key is the parameter name and the value is an array of all values for that parameter.
*   **`header`**: A map of request header names to a single header value.
*   **`headerValues`**: A map of request header names to an array of all values for that header.
*   **`cookie`**: A map of cookie names to cookie objects.
*   **`pageContext`**: Provides access to the `PageContext` object of the JSP.

**Example of using implicit objects:**

To get a request parameter named "username":

```jsp
${param.username}
```

To get a session attribute named "user":

```jsp
${sessionScope.user}
```

#### Functions

EL allows for the use of functions, which are mapped to static methods in Java classes. These functions must be defined in a Tag Library Descriptor (TLD) file. The JSP Standard Tag Library (JSTL) provides a set of standard functions, primarily for string manipulation and collection handling.

To use functions, you must first declare the tag library using the `<%@ taglib %>` directive.

**Example using a JSTL function:**

```jsp
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
...
${fn:length(myCollection)}
```

#### Disabling EL

In some cases, you might need to disable the evaluation of EL expressions. This can be done at the page level using the `isELIgnored` attribute in the `page` directive:

```jsp
<%@ page isELIgnored="true" %>
```

When `isELIgnored` is set to `true`, any `${...}` constructs will be treated as literal strings and not evaluated.
