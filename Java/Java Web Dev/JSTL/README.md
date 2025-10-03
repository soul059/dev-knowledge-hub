# JavaServer Pages Standard Tag Library (JSTL)

JSTL stands for JavaServer Pages Standard Tag Library. It is a collection of custom tags that encapsulate common functionality in JSPs. Using JSTL is highly recommended as it helps to eliminate Java code (scriptlets) from your JSPs, leading to cleaner and more maintainable code.

The main benefits of using JSTL are:
- **Simplifies Code**: Provides ready-to-use tags for common tasks.
- **Improves Readability**: Separates presentation from logic, making JSP files cleaner.
- **Promotes Reusability**: Tags can be reused across different pages.
- **Standardization**: It is a part of the Jakarta EE standard.

## Getting Started

To use JSTL, you need to include the JSTL dependency in your project.

### Maven Dependency

For Maven projects, add the following dependency to your `pom.xml`:

```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>jstl</artifactId>
    <version>1.2</version>
</dependency>
```
For Jakarta EE 9 and later, the coordinates have changed:
```xml
<dependency>
    <groupId>jakarta.servlet.jsp.jstl</groupId>
    <artifactId>jakarta.servlet.jsp.jstl-api</artifactId>
    <version>2.0.0</version>
</dependency>
<dependency>
    <groupId>org.glassfish.web</groupId>
    <artifactId>jakarta.servlet.jsp.jstl</artifactId>
    <version>2.0.0</version>
</dependency>
```

### Taglib Directive

In your JSP files, you need to include a `taglib` directive for the library you want to use. For example, for the core library:

```jsp
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
```
For Jakarta EE 9+, the URI might change to `jakarta.tags.core`.

## JSTL Libraries

JSTL is divided into several libraries, each with a specific purpose.

*   **[Core Tags](Core-Tags.md)** (`c`): The most commonly used library. It provides tags for iteration, conditional logic, variable manipulation, and URL management.
*   **[Formatting (I18N) Tags](Formatting-Tags.md)** (`fmt`): Used for formatting and parsing numbers, dates, and times, with support for internationalization (I18N).
*   **[SQL Tags](SQL-Tags.md)** (`sql`): Provides tags for interacting with relational databases. **Note:** It is generally not recommended to use the SQL tags in production applications.
*   **[XML Tags](XML-Tags.md)** (`x`): Used for creating and manipulating XML data, often using XPath.
*   **[Functions Library](Functions-Tags.md)** (`fn`): Provides a set of Expression Language (EL) functions for common string manipulations and collection operations.

## Expression Language (EL)

JSTL tags are most powerful when used with the Expression Language (EL). EL provides a simple syntax for accessing data from JavaBeans, collections, and implicit objects.

EL expressions are written inside `${...}`.

### Key Features of EL

*   **Accessing Properties:** `${user.name}` (calls `user.getName()`).
*   **Accessing Collections:** `${myMap['key']}` or `${myList[0]}`.
*   **Implicit Objects:** `pageScope`, `requestScope`, `sessionScope`, `applicationScope`, `param`, `header`, `cookie`.
*   **Operators:** Arithmetic (`+`, `-`, `*`, `/`, `%`), logical (`&&`, `||`, `!`), and relational (`==`, `!=`, `<`, `>`).

For more details, see the [Introduction to JSTL](Introduction-to-JSTL.md).