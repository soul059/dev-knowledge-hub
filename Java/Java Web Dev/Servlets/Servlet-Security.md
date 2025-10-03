# Servlet Security

Web application security is a broad topic, but the Servlet API provides several mechanisms to help you secure your servlets. You can enforce security constraints declaratively in the `web.xml` deployment descriptor or programmatically in your servlet code.

## Declarative Security (`web.xml`)

Declarative security is the most common way to secure servlets. It involves defining security constraints in the `web.xml` file.

### Security Constraints

A security constraint defines a set of resources (URL patterns) and the roles that are allowed to access them.

```xml
<security-constraint>
    <web-resource-collection>
        <web-resource-name>Admin Area</web-resource-name>
        <url-pattern>/admin/*</url-pattern>
    </web-resource-collection>
    <auth-constraint>
        <role-name>admin</role-name>
    </auth-constraint>
</security-constraint>
```

In this example, only users with the `admin` role can access URLs under the `/admin/` path.

### Login Configuration

The `<login-config>` element specifies how the user will be authenticated.

```xml
<login-config>
    <auth-method>FORM</auth-method>
    <form-login-config>
        <form-login-page>/login.jsp</form-login-page>
        <form-error-page>/login-error.jsp</form-error-page>
    </form-login-config>
</login-config>
```

This example configures form-based authentication, where the user is redirected to `/login.jsp` to enter their credentials.

## Programmatic Security

Programmatic security allows you to perform security checks in your servlet code. This is useful when you need more fine-grained control than what declarative security can provide.

### Checking User Roles

You can use the `isUserInRole()` method of the `HttpServletRequest` object to check if the current user belongs to a specific role.

```java
if (request.isUserInRole("admin")) {
    // User is an admin, allow access
} else {
    // User is not an admin, deny access
}
```

### Getting the User Principal

The `getUserPrincipal()` method returns a `java.security.Principal` object that represents the current user.

```java
Principal principal = request.getUserPrincipal();
if (principal != null) {
    String username = principal.getName();
    // ...
}
```
