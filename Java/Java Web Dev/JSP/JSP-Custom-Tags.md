# JSP Custom Tags

JSP Custom Tags are user-defined tags that can be used to encapsulate complex logic and generate dynamic content in a JSP page. They are a powerful feature that allows for the creation of reusable and modular web components.

## Creating a Custom Tag

To create a custom tag, you need to do the following:

1.  **Create a Tag Handler Class:** This is a Java class that extends `javax.servlet.jsp.tagext.SimpleTagSupport` and overrides the `doTag()` method.
2.  **Create a Tag Library Descriptor (TLD) File:** This is an XML file that describes the custom tag, including its name, the tag handler class, and any attributes.
3.  **Use the Custom Tag in a JSP Page:** You need to use the `<%@ taglib %>` directive to import the TLD file and then you can use the custom tag in your JSP.

## Example

This example demonstrates how to create a simple custom tag that displays a "Hello, World!" message.

**`HelloWorldTag.java`:**

```java
package com.example.tags;

import java.io.IOException;

import javax.servlet.jsp.JspException;
import javax.servlet.jsp.tagext.SimpleTagSupport;

public class HelloWorldTag extends SimpleTagSupport {
    @Override
    public void doTag() throws JspException, IOException {
        getJspContext().getOut().write("Hello, World!");
    }
}
```

**`my-tags.tld` (in the `WEB-INF/tlds` directory):**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<taglib xmlns="http://java.sun.com/xml/ns/javaee"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-jsptaglibrary_2_1.xsd"
    version="2.1">

    <tlib-version>1.0</tlib-version>
    <short-name>mytags</short-name>
    <uri>/WEB-INF/tlds/my-tags</uri>

    <tag>
        <name>helloWorld</name>
        <tag-class>com.example.tags.HelloWorldTag</tag-class>
        <body-content>empty</body-content>
    </tag>
</taglib>
```

**`my-page.jsp`:**

```jsp
<%@ taglib prefix="my" uri="/WEB-INF/tlds/my-tags" %>

<my:helloWorld />
```
