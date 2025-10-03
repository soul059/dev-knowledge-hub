# Introduction to JSP

Jakarta Server Pages (JSP), formerly known as JavaServer Pages, is a server-side technology used to create dynamic web pages. Developed by Sun Microsystems and released in 1999, JSP allows developers to embed Java code directly into HTML pages, which simplifies the process of developing web applications.

JSP is built on top of the Java Servlet specification and they often work together, especially in older Java web applications. You can think of a JSP as a high-level abstraction of a servlet.

### How JSP Works: The Life Cycle

When a client requests a JSP page, it goes through a distinct life cycle managed by the JSP container (a web server that supports JSP, like Apache Tomcat).

1.  **Translation**: The JSP container translates the `.jsp` file into a Java servlet source file (`.java`). This happens only on the first request or when the JSP file has been modified.
2.  **Compilation**: The generated servlet source code is compiled into a Java bytecode class file (`.class`).
3.  **Loading and Instantiation**: The servlet class is loaded into memory, and an instance of it is created.
4.  **Initialization**: The container calls the `jspInit()` method to initialize the servlet instance. This method is called only once during the JSP's lifecycle.
5.  **Request Processing**: For each client request, the container invokes the `_jspService()` method. This method is responsible for generating the response that is sent back to the client.
6.  **Destruction**: When the JSP is no longer needed (e.g., when the web application is shut down), the container calls the `jspDestroy()` method to perform any necessary cleanup.

### Key Elements of JSP Syntax

JSP pages use special tags to embed Java code within HTML. The main elements are:

*   **Directives `<%@ ... %>`**: Provide instructions to the JSP container. The most common is the `page` directive, used to import Java packages or define an error page.
*   **Declarations `<%! ... %>`**: Used to declare variables and methods at the class level in the generated servlet.
*   **Scriptlets `<% ... %>`**: Allow you to insert arbitrary Java code into the `_jspService()` method.
*   **Expressions `<%= ... %>`**: Contain a Java expression that is evaluated, converted to a string, and inserted directly into the output HTML.
*   **Actions `<jsp:... />`**: XML-based tags used to control the behavior of the JSP engine, such as including other files or working with JavaBeans.

### Example of a Simple JSP Page

Here is a basic example that displays the server's current time:

```jsp
<html>
<head>
    <title>Current Time</title>
</head>
<body>
    <h2>The current time on the server is:</h2>
    <p>
        <%= new java.util.Date() %>
    </p>
</body>
</html>
```

### Advantages of JSP

JSP offers several benefits, particularly when compared to using pure servlets for generating HTML:

*   **Separation of Concerns**: It separates the presentation layer (HTML) from the business logic (Java code), making applications easier to manage.
*   **Simplified Development**: It's often more convenient to write HTML and embed Java code where needed, rather than writing Java code that prints out HTML.
*   **Reusability**: JSP allows for the creation of reusable components like JavaBeans and custom tags.
*   **Platform Independence**: Being based on Java, JSP is platform-independent.
*   **Access to Java APIs**: JSP has access to the full range of Java APIs for tasks like database connectivity (JDBC).
