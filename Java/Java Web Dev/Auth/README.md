# Authentication and Authorization

Authentication and authorization are two fundamental security concepts in web application development.

*   **Authentication** is the process of verifying who a user is. It is about confirming that a user is who they claim to be.
*   **Authorization** is the process of determining what an authenticated user is allowed to do. It is about controlling access to resources.

## Authentication

There are several ways to implement authentication in a Java web application.

### Basic Authentication
With basic authentication, the browser sends the username and password in the `Authorization` header of each request. The credentials are Base64-encoded, but they are not encrypted, so this method is not secure over an unencrypted connection.

### Form-based Authentication
With form-based authentication, the user is redirected to a login form to enter their credentials. The server then creates a session for the user and sends a session cookie to the browser. This is more secure than basic authentication because the password is not sent with each request.

### OAuth/OpenID Connect
OAuth and OpenID Connect are modern authentication protocols that are commonly used for third-party authentication (e.g., "Login with Google" or "Login with Facebook"). They allow users to grant a third-party application access to their resources without sharing their credentials.

## Authorization

Authorization can be implemented in a variety of ways.

### Role-based Access Control (RBAC)
With RBAC, users are assigned roles, and each role has a set of permissions. This is a common approach for enterprise applications because it is easy to manage.

### Access Control Lists (ACLs)
With ACLs, each resource has a list of users and groups that are allowed to access it. This is a more fine-grained approach to authorization, but it can be more difficult to manage.

## Password Hashing

It is crucial to never store passwords in plain text. Instead, you should always hash them using a strong hashing algorithm like bcrypt or scrypt. A password hash is a one-way function, which means that it is very difficult to reverse. When a user enters their password, you hash it and then compare the hash to the stored hash.

## Example using Servlet Security

This example shows how to use servlet security to protect a resource.

**1. `web.xml` (configure security constraints):**

```xml
<security-constraint>
    <web-resource-collection>
        <web-resource-name>Protected Area</web-resource-name>
        <url-pattern>/admin/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>admin</role-name>
    </auth-constraint>
</security-constraint>

<login-config>
    <auth-method>FORM</auth-method>
    <form-login-config>
        <form-login-page>/login.jsp</form-login-page>
        <form-error-page>/login-error.jsp</form-error-page>
    </form-login-config>
</login-config>

<security-role>
    <role-name>admin</role-name>
</security-role>
```

**2. `AdminServlet.java` (the protected resource):**

```java
package com.example;

import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/admin/dashboard")
public class AdminServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html");
        PrintWriter out = response.getWriter();
        out.println("<html><body>");
        out.println("<h1>Welcome to the Admin Dashboard!</h1>");
        out.println("</body></html>");
    }
}
```

In this example, any user who tries to access a URL under `/admin/` will be redirected to the login page. Only users with the `admin` role will be able to access the `AdminServlet`.

## Spring Security

Spring Security is a powerful and highly customizable authentication and access-control framework. It is the de-facto standard for securing Spring-based applications.

### Spring Security Example

This example shows how to use Spring Security to secure a web application.

**1. `pom.xml` (add Spring Security dependencies):**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

**2. `SecurityConfig.java` (Spring Security configuration):**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
                .and()
            .formLogin();
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth
            .inMemoryAuthentication()
                .withUser("user").password("{noop}password").roles("USER")
                .and()
                .withUser("admin").password("{noop}password").roles("ADMIN");
    }
}
```