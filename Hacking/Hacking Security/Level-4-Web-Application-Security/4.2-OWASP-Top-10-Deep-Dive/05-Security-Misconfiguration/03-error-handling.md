# Unit 5: A05 - Security Misconfiguration — Topic 3: Error Handling

## Overview

Improper error handling is a security misconfiguration that **leaks sensitive information** to attackers through detailed error messages, stack traces, database errors, and debug output. Well-crafted error handling provides useful feedback to legitimate users while revealing nothing to attackers about the application's internal structure, technology stack, or data.

---

## 1. What Error Messages Reveal

```
┌─────────────────────────────────────────────────────────────────┐
│           INFORMATION LEAKED THROUGH ERRORS                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  STACK TRACES reveal:                                          │
│  ├── Programming language and version                          │
│  ├── Framework and version                                     │
│  ├── File paths and directory structure                        │
│  ├── Function/method names                                     │
│  ├── Database connection strings                               │
│  └── Third-party library versions                              │
│                                                                 │
│  DATABASE ERRORS reveal:                                       │
│  ├── Database type (MySQL, PostgreSQL, MSSQL)                  │
│  ├── Table and column names                                    │
│  ├── SQL query structure                                       │
│  ├── Database version                                          │
│  └── Connection parameters                                     │
│                                                                 │
│  APPLICATION ERRORS reveal:                                    │
│  ├── Business logic flow                                       │
│  ├── Input validation rules                                    │
│  ├── Authentication mechanisms                                 │
│  ├── Internal API endpoints                                    │
│  └── Configuration values                                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Real-World Dangerous Error Messages

### SQL Error Exposure

```
❌ DANGEROUS RESPONSE:

HTTP/1.1 500 Internal Server Error
Content-Type: text/html

<h1>Database Error</h1>
<p>Error in SQL syntax: 
You have an error in your SQL syntax; check the manual that 
corresponds to your MySQL server version 8.0.26 for the right syntax 
to use near ''admin' AND password='test'' at line 1</p>
<p>Query: SELECT * FROM users WHERE username='admin' AND password='test'</p>

Information revealed:
✗ Database type: MySQL
✗ Database version: 8.0.26
✗ Table name: users
✗ Column names: username, password
✗ Full SQL query structure
✗ Authentication uses direct password comparison (no hashing!)
```

### Stack Trace Exposure

```
❌ DANGEROUS RESPONSE:

HTTP/1.1 500 Internal Server Error

Traceback (most recent call last):
  File "/opt/app/venv/lib/python3.9/site-packages/flask/app.py", line 2070
  File "/opt/app/routes/user.py", line 45, in get_user
    user = db.session.query(User).filter_by(id=user_id).first()
  File "/opt/app/models/user.py", line 23, in __repr__
    return f"User(id={self.id}, email={self.email}, role={self.role})"
sqlalchemy.exc.OperationalError: (psycopg2.OperationalError) 
    connection to server at "db.internal.company.com" (10.0.1.50), 
    port 5432 failed: FATAL: password authentication failed for user "webapp_user"

Information revealed:
✗ Python 3.9, Flask framework
✗ File paths: /opt/app/routes/user.py
✗ ORM: SQLAlchemy with psycopg2
✗ Database: PostgreSQL at db.internal.company.com (10.0.1.50:5432)
✗ Database username: webapp_user
✗ Model structure: User has id, email, role fields
```

---

## 3. Secure Error Handling Implementation

### Python (Flask)

```python
import logging
import uuid

logger = logging.getLogger('security')

@app.errorhandler(400)
def bad_request(error):
    return jsonify({'error': 'Bad request', 'request_id': str(uuid.uuid4())}), 400

@app.errorhandler(401)
def unauthorized(error):
    return jsonify({'error': 'Authentication required'}), 401

@app.errorhandler(403)
def forbidden(error):
    return jsonify({'error': 'Access denied'}), 403

@app.errorhandler(404)
def not_found(error):
    return jsonify({'error': 'Resource not found'}), 404

@app.errorhandler(429)
def rate_limited(error):
    return jsonify({'error': 'Too many requests'}), 429

@app.errorhandler(500)
def internal_error(error):
    request_id = str(uuid.uuid4())
    # Log full details internally
    logger.error(f"Internal error [{request_id}]: {error}", exc_info=True, extra={
        'request_id': request_id,
        'url': request.url,
        'method': request.method,
        'ip': request.remote_addr,
        'user_agent': request.user_agent.string,
        'user_id': getattr(g, 'user_id', 'anonymous')
    })
    # Return generic message to client
    return jsonify({
        'error': 'An internal error occurred',
        'request_id': request_id  # For support reference
    }), 500

@app.errorhandler(Exception)
def unhandled_exception(error):
    request_id = str(uuid.uuid4())
    logger.critical(f"Unhandled exception [{request_id}]: {error}", exc_info=True)
    return jsonify({
        'error': 'An unexpected error occurred',
        'request_id': request_id
    }), 500
```

### Java (Spring Boot)

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception ex, HttpServletRequest request) {
        String requestId = UUID.randomUUID().toString();
        
        // Log full details internally
        logger.error("Unhandled exception [{}]: {} - URL: {} - IP: {}", 
            requestId, ex.getMessage(), request.getRequestURI(), 
            request.getRemoteAddr(), ex);
        
        // Return generic message
        ErrorResponse response = new ErrorResponse(
            "An internal error occurred", requestId
        );
        return ResponseEntity.status(500).body(response);
    }

    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException ex) {
        return ResponseEntity.status(403)
            .body(new ErrorResponse("Access denied", null));
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(404)
            .body(new ErrorResponse("Resource not found", null));
    }
}
```

### Node.js (Express)

```javascript
// Global error handler — MUST be last middleware
app.use((err, req, res, next) => {
    const requestId = require('uuid').v4();
    
    // Log full details internally
    console.error({
        requestId,
        message: err.message,
        stack: err.stack,
        url: req.originalUrl,
        method: req.method,
        ip: req.ip,
        userId: req.user?.id || 'anonymous'
    });
    
    // Determine status code
    const statusCode = err.statusCode || 500;
    
    // Return generic message
    const response = {
        error: statusCode >= 500 
            ? 'Internal server error' 
            : err.publicMessage || 'Request error',
        requestId
    };
    
    // NEVER include in response:
    // - err.stack
    // - err.message (may contain internal details)
    // - Database error details
    // - File paths
    
    res.status(statusCode).json(response);
});
```

---

## 4. Error Handling Security Rules

```
RULE 1: NEVER expose stack traces in production
  → Configure framework to suppress (DEBUG=false, etc.)

RULE 2: NEVER include internal details in error responses
  → No file paths, SQL queries, connection strings, versions

RULE 3: Use generic error messages for ALL error types
  → "An error occurred" — not "NullPointerException at Line 42"

RULE 4: Log FULL details server-side
  → Stack trace, request data, user info → internal logs only

RULE 5: Use request IDs to correlate
  → Client gets request_id; support uses it to find logs

RULE 6: Consistent error format
  → Same JSON structure for all errors
  → Same response time (prevent timing attacks)

RULE 7: Don't differentiate auth errors
  → "Invalid credentials" — not "User not found" vs "Wrong password"

RULE 8: Custom error pages
  → Replace default 404, 500 pages with branded generic pages
```

---

## 5. Authentication Error Pitfalls

```
❌ INSECURE — Different messages reveal user existence:

"User 'admin@corp.com' not found"
→ Attacker knows: this email is NOT registered

"Invalid password for user 'admin@corp.com'"
→ Attacker knows: this email IS registered (now brute-force password)

"Account locked after 5 failed attempts"
→ Attacker knows: account exists AND lockout threshold

✅ SECURE — Identical responses regardless of cause:

"Invalid email or password"      ← Same message for all cases:
                                    - User not found
                                    - Wrong password  
                                    - Account locked
                                    - Account disabled

Same response time for all cases (prevent timing-based enumeration):
if user_exists:
    verify_password(input, user.hash)  # Takes ~100ms
else:
    verify_password(input, DUMMY_HASH)  # Also takes ~100ms
```

---

## 6. Testing for Error Handling Issues

```bash
# Trigger various error conditions
# SQL injection attempts (look for SQL errors)
curl -s "http://target.com/user?id=1'" | grep -i "sql\|mysql\|postgres\|syntax"

# Invalid input types
curl -s "http://target.com/user?id=abc"
curl -s "http://target.com/user?id=-1"
curl -s "http://target.com/user?id=99999999"

# Non-existent pages (check for technology fingerprints)
curl -s "http://target.com/nonexistent"
curl -s "http://target.com/nonexistent.php"
curl -s "http://target.com/nonexistent.asp"

# Method not allowed
curl -s -X DELETE "http://target.com/"
curl -s -X TRACE "http://target.com/"

# Large input
python -c "print('A'*10000)" | curl -s -d @- "http://target.com/search"

# Special characters
curl -s "http://target.com/search?q=<script>alert(1)</script>"
curl -s "http://target.com/file?path=../../../etc/passwd"

# Check headers for technology exposure
curl -sI "http://target.com/" | grep -i "server\|x-powered\|x-aspnet"
```

---

## Summary Table

| Error Type | Insecure Response | Secure Response |
|-----------|-------------------|-----------------|
| SQL Error | Full query + DB version | "An error occurred" |
| Auth Failure | "User not found" | "Invalid credentials" |
| 404 | Apache/IIS default page | Custom generic page |
| 500 | Stack trace + file paths | "Internal error" + request ID |
| Rate Limit | "IP 10.0.1.5 blocked" | "Too many requests" |
| Validation | "Field X failed regex Y" | "Invalid input for field X" |

---

## Revision Questions

1. What specific information can an attacker learn from a SQL error message exposed to the client?
2. Write a complete global error handler for Flask/Express/Spring Boot that logs internally but shows generic messages.
3. Why should authentication failures always return identical responses? Give an example of user enumeration via error messages.
4. What is the purpose of a request ID in error responses?
5. How would you test a web application for improper error handling?
6. Explain the "fail secure" principle in the context of error handling.

---

*Previous: [02-unnecessary-features.md](02-unnecessary-features.md) | Next: [04-security-headers.md](04-security-headers.md)*

---

*[Back to README](../README.md)*
