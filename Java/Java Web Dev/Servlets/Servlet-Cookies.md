# Servlet Cookies

Cookies are small pieces of data that are sent from the server to the client and stored on the client's machine. They are sent back to the server with each subsequent request, allowing the server to identify the client and maintain state.

Cookies are commonly used for:

-   **Session Management:** To track user sessions.
-   **Personalization:** To store user preferences.
-   **Tracking:** To record user behavior.

In the Servlet API, cookies are represented by the `javax.servlet.http.Cookie` class.

## Creating and Sending Cookies

To create a cookie, you need to instantiate the `Cookie` class and provide a name and a value.

```java
Cookie cookie = new Cookie("username", "JohnDoe");
```

Once you have created a cookie, you can set its attributes, such as its maximum age, and then add it to the `HttpServletResponse`.

```java
// Set the maximum age of the cookie in seconds
cookie.setMaxAge(60 * 60 * 24); // 24 hours

// Add the cookie to the response
response.addCookie(cookie);
```

## Reading Cookies

To read the cookies that a client sends with a request, you can use the `getCookies()` method of the `HttpServletRequest` object. This method returns an array of `Cookie` objects.

```java
Cookie[] cookies = request.getCookies();
if (cookies != null) {
    for (Cookie cookie : cookies) {
        if (cookie.getName().equals("username")) {
            String username = cookie.getValue();
            // Do something with the username
        }
    }
}
```

## Deleting Cookies

To delete a cookie, you need to create a new cookie with the same name and set its maximum age to 0. Then, you add the cookie to the response.

```java
Cookie cookie = new Cookie("username", "");
cookie.setMaxAge(0);
response.addCookie(cookie);
```

## Cookie Attributes

The `Cookie` class provides several methods for setting the attributes of a cookie:

-   `setMaxAge(int expiry)`: Sets the maximum age of the cookie in seconds.
-   `setPath(String uri)`: Sets the path on the server to which the cookie applies.
-   `setDomain(String pattern)`: Sets the domain to which the cookie applies.
-   `setSecure(boolean flag)`: Indicates whether the cookie should only be sent over a secure protocol (e.g., HTTPS).
-   `setHttpOnly(boolean isHttpOnly)`: Indicates whether the cookie should be accessible only through the HTTP protocol (and not through client-side scripts).
