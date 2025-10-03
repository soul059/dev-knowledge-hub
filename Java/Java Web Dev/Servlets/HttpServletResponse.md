# HttpServletResponse

The `HttpServletResponse` object is used to send a response back to the client. The web container creates an `HttpServletResponse` object for each incoming request and passes it to the `service()` method of the servlet.

The `HttpServletResponse` object provides methods for setting the response's content type, status code, headers, and for sending the response body. It is an interface defined in the `javax.servlet.http` package.

## Key Methods of `HttpServletResponse`

Here are some of the most commonly used methods of the `HttpServletResponse` interface:

### 1. Setting the Content Type

The `setContentType()` method is used to set the MIME type of the response content. This tells the client how to interpret the response.

-   `void setContentType(String type)`: Sets the content type of the response being sent to the client.

**Example:**

```java
response.setContentType("text/html"); // For HTML content
response.setContentType("application/json"); // For JSON content
```

### 2. Getting a Writer or an Output Stream

To send content back to the client, you can use either a `PrintWriter` (for text data) or a `ServletOutputStream` (for binary data). You can only use one of these per response.

-   `java.io.PrintWriter getWriter()`: Returns a `PrintWriter` object that can send character text to the client.
-   `javax.servlet.ServletOutputStream getOutputStream()`: Returns a `ServletOutputStream` suitable for writing binary data in the response.

**Example:**

```java
// For text data
PrintWriter out = response.getWriter();
out.println("<h1>Hello, World!</h1>");

// For binary data (e.g., an image)
ServletOutputStream out = response.getOutputStream();
// Write binary data to the stream
```

### 3. Setting the Status Code

The `setStatus()` method is used to set the HTTP status code of the response. Status codes indicate the outcome of the request.

-   `void setStatus(int sc)`: Sets the status code for this response.

**Example:**

```java
response.setStatus(HttpServletResponse.SC_OK); // 200 OK
response.setStatus(HttpServletResponse.SC_NOT_FOUND); // 404 Not Found
```

### 4. Sending Redirects

The `sendRedirect()` method is used to send a redirect response to the client, which tells the client to make a new request to a different URL.

-   `void sendRedirect(String location)`: Sends a temporary redirect response to the client using the specified redirect location URL.

**Example:**

```java
response.sendRedirect("https://www.google.com");
```

### 5. Adding Cookies

The `addCookie()` method is used to add a `Cookie` to the response. Cookies are used to store small pieces of information on the client's machine.

-   `void addCookie(Cookie cookie)`: Adds the specified cookie to the response.

**Example:**

```java
Cookie cookie = new Cookie("username", "JohnDoe");
response.addCookie(cookie);
```

### 6. Setting Headers

The `setHeader()` and `addHeader()` methods are used to set HTTP headers in the response.

-   `void setHeader(String name, String value)`: Sets a response header with the given name and value.
-   `void addHeader(String name, String value)`: Adds a response header with the given name and value.

**Example:**

```java
response.setHeader("Cache-Control", "no-cache");
```
