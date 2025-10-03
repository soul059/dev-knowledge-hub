# JSP Actions

JSP actions are XML-based tags that control the behavior of the servlet engine. They are used to perform specific tasks, such as including files, forwarding requests, and working with JavaBeans. JSP actions are re-evaluated each time the page is accessed.

The basic syntax for a JSP action is:

```xml
<jsp:action_name attribute="value" />
```

### Standard JSP Actions

Here are some of the most common standard JSP action tags:

*   **`jsp:include`**: Includes the content of another resource (like an HTML file, JSP, or servlet) in the current JSP page at request time.
*   **`jsp:forward`**: Forwards the request and response from the current page to another resource, such as a different JSP page or a servlet. The control does not return to the original page.
*   **`jsp:useBean`**: Makes a JavaBean available to the JSP page. It will either locate an existing bean or instantiate a new one.
*   **`jsp:setProperty`**: Sets the value of a property in a JavaBean.
*   **`jsp:getProperty`**: Retrieves the value of a JavaBean property and outputs it to the response.
*   **`jsp:param`**: Used with `jsp:include`, `jsp:forward`, and `jsp:plugin` to add a parameter to the request.
*   **`jsp:plugin`**: Embeds a Java applet or JavaBean in the JSP page.
*   **`jsp:text`**: Used to write template text in JSP pages.
*   **`jsp:attribute`**, **`jsp:body`**, and **`jsp:element`**: These tags are used to define XML elements dynamically.

### Example

Here is an example of how to use the `jsp:include` and `jsp:param` actions:

**`index.jsp`**

```xml
<html>
  <head>
    <title>JSP Actions Example</title>
  </head>
  <body>
    <h2>This is the main page.</h2>
    <jsp:include page="header.jsp">
      <jsp:param name="title" value="Welcome to our website!" />
    </jsp:include>
    <p>This is the content of the main page.</p>
  </body>
</html>
```

**`header.jsp`**

```xml
<h3><%= request.getParameter("title") %></h3>
<hr>
```

When `index.jsp` is requested, it will include the content of `header.jsp`. The `jsp:param` tag passes a "title" parameter to `header.jsp`, which then displays the value of the parameter.
