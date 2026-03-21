# 04 — Function-Level Access Control

## Overview

**Function-level access control** ensures that each application function — whether an API endpoint, a page, a button action, or a background process — verifies that the requesting user has the appropriate permissions before execution. When these checks are missing, attackers can invoke privileged functions directly, even if the UI hides them. This vulnerability is codified as **OWASP A01:2021 – Broken Access Control**, specifically the sub-category **Missing Function Level Access Control** (formerly OWASP Top 10 A7:2013), and corresponds to **CWE-285: Improper Authorization** and **CWE-862: Missing Authorization**.

---

## 1. Fundamentals

### 1.1 What Is Function-Level Access Control?

```
+──────────────────────────────────────────────────────────────+
│          Function-Level Access Control                       │
+──────────────────────────────────────────────────────────────+
│                                                              │
│   A web application exposes many "functions":                │
│                                                              │
│   ┌────────────────────────────────────────────────────┐     │
│   │  Page Renders     │  API Endpoints  │  Actions     │     │
│   ├────────────────────────────────────────────────────┤     │
│   │  /dashboard       │  GET /api/users │  Delete User │     │
│   │  /admin/settings  │  POST /api/role │  Export Data │     │
│   │  /reports/annual  │  PUT /api/config│  Reset Pass  │     │
│   └────────────────────────────────────────────────────┘     │
│                                                              │
│   Each function MUST check:                                  │
│   1. Is the user authenticated?        (AuthN)               │
│   2. Does the user have permission?    (AuthZ)               │
│   3. Is the function allowed via this method? (Method)       │
│                                                              │
│   VULNERABILITY: Function is executable but                  │
│   access control check is MISSING or INCOMPLETE              │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 1.2 How It Differs from Other Access-Control Flaws

| Vulnerability | What's Missing | Example |
|---|---|---|
| **Missing Function-Level AC** | No authorization check on the endpoint/function itself | Regular user calls `DELETE /api/admin/users/5` successfully |
| **IDOR** | Authorization check exists but doesn't verify object ownership | User A accesses User B's data via `GET /api/profile/B` |
| **Vertical Escalation** | Role check bypassed or absent for higher-privilege actions | User accesses `/admin` panel |
| **Horizontal Escalation** | Same-level cross-user access | User A reads User B's orders |

> **Key distinction:** Missing function-level access control means the **entire endpoint** lacks an authorization check, whereas IDOR means the endpoint has a check but it's insufficient for specific resources.

### 1.3 Real-World Prevalence

```
+──────────────────────────────────────────────────────────+
│    Why Missing Function-Level AC Is So Common            │
+──────────────────────────────────────────────────────────+
│                                                          │
│  1. "Security by Obscurity" mindset                      │
│     Developers hide admin features in the UI but         │
│     don't protect the backend endpoints.                 │
│                                                          │
│  2. Inconsistent enforcement                             │
│     Some endpoints have checks; others are forgotten.    │
│     No centralized authorization layer.                  │
│                                                          │
│  3. API-first architectures                              │
│     Backend APIs serve multiple frontends.               │
│     Each frontend may assume the other enforces access.  │
│                                                          │
│  4. Rapid development                                    │
│     New endpoints added without security review.         │
│     "We'll add auth checks later" (never happens).      │
│                                                          │
│  5. Microservices complexity                              │
│     Dozens of services, each must enforce auth.          │
│     One missed service = exploitable weakness.           │
│                                                          │
+──────────────────────────────────────────────────────────+
```

---

## 2. Attack Surface — Where Functions Are Exposed

### 2.1 Admin API Endpoints

```http
# Administrative APIs often lack per-endpoint auth checks

# User management
GET    /api/admin/users              → List all users
POST   /api/admin/users              → Create user (any role)
DELETE /api/admin/users/1002         → Delete user
PUT    /api/admin/users/1002/role    → Change user role

# System configuration
GET    /api/admin/config             → View system config
PUT    /api/admin/config             → Modify system config
POST   /api/admin/config/reset       → Reset to defaults

# Data export
GET    /api/admin/export/users       → Export all user data
GET    /api/admin/export/orders      → Export all orders

# System operations
POST   /api/admin/backup             → Trigger backup
POST   /api/admin/cache/clear        → Clear application cache
GET    /api/admin/logs               → View system logs
POST   /api/admin/maintenance        → Toggle maintenance mode
```

### 2.2 Hidden / Internal Endpoints

```
+──────────────────────────────────────────────────────────+
│           Hidden Endpoint Discovery                      │
+──────────────────────────────────────────────────────────+
│                                                          │
│  Endpoints not linked in the UI but accessible:          │
│                                                          │
│  Debug endpoints:                                        │
│    /debug/vars                                           │
│    /debug/pprof                                          │
│    /_debug/requests                                      │
│    /api/debug/sql                                        │
│                                                          │
│  Health/status endpoints:                                │
│    /health                                               │
│    /status                                               │
│    /api/internal/health                                  │
│    /metrics                 (Prometheus metrics)         │
│                                                          │
│  Documentation endpoints:                                │
│    /api/docs               (Swagger UI)                  │
│    /api/swagger.json                                     │
│    /graphql/playground                                   │
│    /api-docs                                             │
│                                                          │
│  Utility endpoints:                                      │
│    /api/test                                             │
│    /api/internal/email-test                              │
│    /api/internal/generate-report                         │
│    /cron/run-task                                        │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 2.3 Framework Management Endpoints

| Framework | Endpoint | Exposed Data |
|---|---|---|
| Spring Boot | `/actuator/env` | Environment variables, secrets |
| Spring Boot | `/actuator/mappings` | All URL route mappings |
| Spring Boot | `/actuator/heapdump` | JVM heap (may contain credentials) |
| Django | `/admin/` | Database admin interface |
| Django | `/__debug__/` | Django Debug Toolbar |
| Express | `/api-docs` | Swagger/OpenAPI spec |
| Rails | `/rails/info/routes` | All route mappings |
| Rails | `/sidekiq` | Background job dashboard |
| Laravel | `/_debugbar` | Debug toolbar data |
| ASP.NET | `/elmah.axd` | Error log viewer |
| PHP | `/phpinfo.php` | Full PHP configuration |
| WordPress | `/wp-json/` | REST API metadata |

---

## 3. HTTP Method Bypass

### 3.1 How Method-Based Controls Fail

```
+──────────────────────────────────────────────────────────────+
│              HTTP Method Bypass Attack                        │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Scenario: Access control configured only for specific       │
│  HTTP methods.                                               │
│                                                              │
│  Web server config (Apache):                                 │
│  <Location /admin>                                           │
│    <Limit GET POST>                                          │
│      Require role admin                                      │
│    </Limit>                                                  │
│  </Location>                                                 │
│                                                              │
│  Problem: Only GET and POST are restricted!                  │
│                                                              │
│  Attack:                                                     │
│  PUT    /admin/users HTTP/1.1  → 200 OK (bypassed!)          │
│  DELETE /admin/users HTTP/1.1  → 200 OK (bypassed!)          │
│  PATCH  /admin/users HTTP/1.1  → 200 OK (bypassed!)          │
│  HEAD   /admin/users HTTP/1.1  → 200 OK (info leak)         │
│                                                              │
│  Fix: Use <LimitExcept> instead                              │
│  <Location /admin>                                           │
│    <LimitExcept>                                             │
│      Require role admin                                      │
│    </LimitExcept>                                            │
│  </Location>                                                 │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 3.2 Method Override Headers

```http
# Application blocks DELETE for non-admins
DELETE /api/users/1002 HTTP/1.1     → 403 Forbidden

# Try method override headers
POST /api/users/1002 HTTP/1.1
X-HTTP-Method-Override: DELETE       → May succeed!

POST /api/users/1002 HTTP/1.1
X-HTTP-Method: DELETE                → May succeed!

POST /api/users/1002 HTTP/1.1
X-Method-Override: DELETE            → May succeed!

# URL-based override
POST /api/users/1002?_method=DELETE  → May succeed!
POST /api/users/1002?method=DELETE   → May succeed!
```

### 3.3 Method Override Headers Comparison

| Header / Parameter | Frameworks That Support It |
|---|---|
| `X-HTTP-Method-Override` | ASP.NET, Express (method-override), Rails, Django |
| `X-HTTP-Method` | ASP.NET |
| `X-Method-Override` | Various |
| `_method` (query/body) | Rails, Laravel, Express |
| `method` (query) | Some custom implementations |

### 3.4 Testing All HTTP Methods

```bash
# Test all methods against a restricted endpoint
for METHOD in GET POST PUT DELETE PATCH HEAD OPTIONS TRACE CONNECT; do
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
        -X "$METHOD" \
        -H "Cookie: session=USER_SESSION" \
        "https://target.com/admin/users")
    echo "$METHOD → $STATUS"
done

# Expected output for secure app:
# GET     → 403
# POST    → 403
# PUT     → 403
# DELETE  → 403
# PATCH   → 403
# HEAD    → 403
# OPTIONS → 200 (may be allowed for CORS preflight)
# TRACE   → 405
# CONNECT → 405

# Vulnerable app might show:
# GET     → 403
# POST    → 403
# PUT     → 200  ← BYPASS!
# DELETE  → 200  ← BYPASS!
```

---

## 4. Hidden Functionality Testing

### 4.1 Discovering Hidden Endpoints

```bash
# 1. JavaScript source analysis
# Download and search all JS bundles
wget -r -l1 -nd -A.js https://target.com/ -P js_files/
grep -rn "api\|endpoint\|fetch\|axios\|ajax\|XMLHttpRequest" js_files/ \
  | grep -oP '["'"'"'][^"'"'"']*api[^"'"'"']*["'"'"']' | sort -u

# 2. Swagger / OpenAPI discovery
curl -s https://target.com/swagger.json | python -m json.tool
curl -s https://target.com/api/docs
curl -s https://target.com/v2/api-docs
curl -s https://target.com/openapi.json

# 3. GraphQL introspection
curl -s -X POST https://target.com/graphql \
  -H "Content-Type: application/json" \
  -d '{"query":"{ __schema { types { name fields { name } } } }"}'

# 4. WSDL files (SOAP services)
curl -s "https://target.com/service?wsdl"

# 5. robots.txt and sitemap.xml
curl -s https://target.com/robots.txt
curl -s https://target.com/sitemap.xml
```

### 4.2 API Documentation as Attack Surface

```
+──────────────────────────────────────────────────────────+
│       API Docs → Attack Surface Expansion                │
+──────────────────────────────────────────────────────────+
│                                                          │
│  Swagger UI at /api/docs reveals:                        │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  PUBLIC ENDPOINTS (shown in UI):                   │  │
│  │  GET  /api/v1/products                             │  │
│  │  GET  /api/v1/products/{id}                        │  │
│  │  POST /api/v1/orders                               │  │
│  │                                                    │  │
│  │  ADMIN ENDPOINTS (also in Swagger!):               │  │
│  │  GET    /api/v1/admin/users        ← discovered!   │  │
│  │  POST   /api/v1/admin/users        ← discovered!   │  │
│  │  DELETE /api/v1/admin/users/{id}   ← discovered!   │  │
│  │  PUT    /api/v1/admin/config       ← discovered!   │  │
│  │  GET    /api/v1/admin/reports      ← discovered!   │  │
│  │  POST   /api/v1/admin/impersonate  ← discovered!   │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  Attack: Use regular user's token to call admin          │
│  endpoints listed in Swagger documentation.              │
│                                                          │
+──────────────────────────────────────────────────────────+
```

### 4.3 Endpoint Enumeration with Tools

```bash
# ffuf for endpoint discovery
ffuf -u "https://target.com/api/FUZZ" \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt \
  -H "Authorization: Bearer USER_TOKEN" \
  -mc 200,201,204,301,302 \
  -o api_endpoints.json

# Arjun for parameter discovery on known endpoints
arjun -u "https://target.com/api/admin/users" \
  -m GET \
  --headers "Authorization: Bearer USER_TOKEN"

# Kiterunner for API endpoint discovery
kr scan https://target.com/api \
  -w /usr/share/kiterunner/routes-large.kite \
  -H "Authorization: Bearer USER_TOKEN"

# Nuclei for common exposed endpoints
nuclei -u https://target.com \
  -tags exposure,config,debug \
  -o exposed_endpoints.txt
```

---

## 5. RBAC vs ABAC — Authorization Models

### 5.1 Role-Based Access Control (RBAC)

```
+──────────────────────────────────────────────────────────────+
│                    RBAC Model                                │
+──────────────────────────────────────────────────────────────+
│                                                              │
│   Users are assigned ROLES.                                  │
│   Roles have PERMISSIONS.                                    │
│   Access is determined by role membership.                   │
│                                                              │
│   ┌─────────┐    ┌──────────┐    ┌──────────────────┐       │
│   │  User   │───►│  Role    │───►│  Permission      │       │
│   │  Alice  │    │  Admin   │    │  manage_users     │       │
│   │  Bob    │    │  Editor  │    │  edit_content     │       │
│   │  Carol  │    │  Viewer  │    │  view_content     │       │
│   └─────────┘    └──────────┘    └──────────────────┘       │
│                                                              │
│   Role Hierarchy:                                            │
│                                                              │
│   ┌─────────────────────────────────────────────┐            │
│   │  Super Admin                                │            │
│   │  └── Admin                                  │            │
│   │      └── Manager                            │            │
│   │          └── Editor                         │            │
│   │              └── Viewer                     │            │
│   │                  └── Guest                  │            │
│   └─────────────────────────────────────────────┘            │
│                                                              │
│   Each level inherits permissions from levels below.         │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

**RBAC Implementation:**

```python
# RBAC Permission Matrix
ROLE_PERMISSIONS = {
    'guest': {'view_public'},
    'viewer': {'view_public', 'view_content'},
    'editor': {'view_public', 'view_content', 'edit_content', 'create_content'},
    'manager': {'view_public', 'view_content', 'edit_content', 'create_content',
                'delete_content', 'manage_team'},
    'admin': {'view_public', 'view_content', 'edit_content', 'create_content',
              'delete_content', 'manage_team', 'manage_users', 'view_reports',
              'manage_config'},
}

def has_permission(user, permission):
    """Check if user's role includes the required permission."""
    role_perms = ROLE_PERMISSIONS.get(user.role, set())
    return permission in role_perms

# Decorator
def require_permission(permission):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            if not has_permission(current_user, permission):
                abort(403)
            return f(*args, **kwargs)
        return wrapper
    return decorator

@app.route('/admin/users')
@login_required
@require_permission('manage_users')
def manage_users():
    return jsonify(User.query.all())
```

### 5.2 Attribute-Based Access Control (ABAC)

```
+──────────────────────────────────────────────────────────────+
│                    ABAC Model                                │
+──────────────────────────────────────────────────────────────+
│                                                              │
│   Access decisions based on ATTRIBUTES of:                   │
│   • Subject (user)      — role, department, clearance        │
│   • Resource (object)   — classification, owner, type        │
│   • Action              — read, write, delete, execute       │
│   • Environment         — time, location, device, IP        │
│                                                              │
│   Policy Example:                                            │
│   "A manager in the Finance department can READ              │
│    financial reports classified as INTERNAL                   │
│    during business hours from the corporate network."        │
│                                                              │
│   ┌───────────────────────────────────────────────────┐      │
│   │             ABAC Policy Engine                    │      │
│   │                                                   │      │
│   │  Subject Attributes:                              │      │
│   │    role=manager, dept=finance, clearance=L2       │      │
│   │                                                   │      │
│   │  Resource Attributes:                             │      │
│   │    type=report, class=internal, owner=finance     │      │
│   │                                                   │      │
│   │  Action: READ                                     │      │
│   │                                                   │      │
│   │  Environment:                                     │      │
│   │    time=14:30, ip=10.0.0.0/8, device=corporate   │      │
│   │                                                   │      │
│   │  Decision: ✓ PERMIT                               │      │
│   └───────────────────────────────────────────────────┘      │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

**ABAC Implementation:**

```python
class ABACPolicy:
    """Attribute-Based Access Control policy engine."""
    
    def __init__(self):
        self.policies = []
    
    def add_policy(self, policy):
        self.policies.append(policy)
    
    def check_access(self, subject, resource, action, environment):
        for policy in self.policies:
            if policy.evaluate(subject, resource, action, environment):
                return True
        return False  # Default deny

class FinanceReportPolicy:
    def evaluate(self, subject, resource, action, env):
        return (
            subject.department == 'finance' and
            subject.role in ('manager', 'director') and
            resource.type == 'financial_report' and
            resource.classification in ('internal', 'public') and
            action == 'read' and
            env.is_business_hours() and
            env.is_corporate_network()
        )

# Usage
abac = ABACPolicy()
abac.add_policy(FinanceReportPolicy())

if abac.check_access(current_user, report, 'read', request_context):
    return report.data
else:
    abort(403)
```

### 5.3 RBAC vs ABAC Comparison

| Aspect | RBAC | ABAC |
|---|---|---|
| **Complexity** | Simple | Complex |
| **Granularity** | Role-level | Attribute-level (fine-grained) |
| **Scalability** | Role explosion in large orgs | Scales well with policies |
| **Flexibility** | Limited to role assignments | Context-aware decisions |
| **Implementation** | Easy (role check middleware) | Harder (policy engine) |
| **Maintenance** | Easy for small systems | Easier for complex systems |
| **Environment-aware** | No (without extensions) | Yes (time, IP, device) |
| **Best for** | Simple apps, small teams | Enterprise, compliance-heavy |
| **Standards** | Informal / framework-specific | XACML, OPA, Cedar |
| **Common in** | Most web applications | Government, healthcare, finance |

### 5.4 Other Authorization Models

| Model | Description | Use Case |
|---|---|---|
| **DAC** (Discretionary) | Resource owner controls access | File systems, shared documents |
| **MAC** (Mandatory) | System-enforced classifications | Military, government |
| **ReBAC** (Relationship) | Access based on user-resource relationships | Social networks, Google Zanzibar |
| **PBAC** (Policy-Based) | External policy engine decides | Microservices, cloud-native |

---

## 6. Centralized Authorization

### 6.1 Why Centralized Authorization?

```
+──────────────────────────────────────────────────────────────+
│      Decentralized vs Centralized Authorization              │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  DECENTRALIZED (Fragile)               CENTRALIZED (Robust)  │
│                                                              │
│  ┌────────────┐                        ┌────────────────┐    │
│  │ Endpoint A │─── own auth check      │ Auth Gateway / │    │
│  └────────────┘                        │ Middleware     │    │
│  ┌────────────┐                        │                │    │
│  │ Endpoint B │─── own auth check      │ Single policy  │    │
│  └────────────┘                        │ enforcement    │    │
│  ┌────────────┐                        │ point          │    │
│  │ Endpoint C │─── NO auth check ✗     └───────┬────────┘    │
│  └────────────┘                                │             │
│  ┌────────────┐                        ┌───────▼────────┐    │
│  │ Endpoint D │─── own auth check      │ All Endpoints  │    │
│  └────────────┘                        │ A, B, C, D     │    │
│                                        └────────────────┘    │
│  Problem: Endpoint C was missed!                             │
│  → Missing function-level AC                                 │
│                                                              │
│  Solution: All requests go through                           │
│  a central authorization point.                              │
│  If not explicitly permitted → DENIED.                       │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 6.2 API Gateway Authorization

```
+──────────────────────────────────────────────────────────────+
│              API Gateway Authorization                       │
+──────────────────────────────────────────────────────────────+
│                                                              │
│   Client ──► API Gateway ──► Microservices                   │
│                   │                                          │
│                   ├── 1. Validate JWT                         │
│                   ├── 2. Check route permissions              │
│                   ├── 3. Rate limit                           │
│                   └── 4. Forward to service (or reject)      │
│                                                              │
│   Gateway Policy Configuration:                              │
│                                                              │
│   ┌──────────────────────────────────────────────┐           │
│   │  Route              │ Methods │ Roles        │           │
│   ├──────────────────────┼─────────┼──────────────┤           │
│   │  /api/public/**      │ GET     │ *            │           │
│   │  /api/user/**        │ *       │ user,admin   │           │
│   │  /api/admin/**       │ *       │ admin        │           │
│   │  /api/internal/**    │ *       │ DENY ALL     │           │
│   │  /actuator/**        │ *       │ DENY ALL     │           │
│   │  /**                 │ *       │ DENY ALL     │ default  │
│   └──────────────────────────────────────────────┘           │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

**Kong API Gateway Example:**

```yaml
# Kong declarative configuration
services:
  - name: admin-service
    url: http://admin-service:8080
    routes:
      - name: admin-routes
        paths: ["/api/admin"]
    plugins:
      - name: jwt
      - name: acl
        config:
          allow: ["admin-group"]
          deny: ["user-group", "anonymous"]

  - name: user-service
    url: http://user-service:8080
    routes:
      - name: user-routes
        paths: ["/api/user"]
    plugins:
      - name: jwt
      - name: acl
        config:
          allow: ["user-group", "admin-group"]
```

### 6.3 Open Policy Agent (OPA)

```rego
# OPA Rego policy for function-level access control
package authz

default allow = false

# Public endpoints — anyone can access
allow {
    input.path == ["api", "public", _]
    input.method == "GET"
}

# User endpoints — authenticated users
allow {
    startswith(input.path[0], "api")
    input.path[1] == "user"
    input.user.authenticated == true
}

# Admin endpoints — admin role required
allow {
    startswith(input.path[0], "api")
    input.path[1] == "admin"
    input.user.role == "admin"
}

# Block internal endpoints entirely
deny {
    input.path[1] == "internal"
}
deny {
    input.path[1] == "actuator"
}

# Apply deny rules
allow {
    not deny
}
```

### 6.4 Implementation Patterns

**Express.js Centralized Auth Middleware:**

```javascript
// authorization.js — Centralized authorization
const routePermissions = {
  // pattern: [allowed_roles]
  'GET /api/public/*': ['*'],
  'GET /api/user/profile': ['user', 'admin'],
  'PUT /api/user/profile': ['user', 'admin'],
  'GET /api/user/orders': ['user', 'admin'],
  'GET /api/admin/users': ['admin'],
  'POST /api/admin/users': ['admin'],
  'DELETE /api/admin/users/*': ['admin'],
  'PUT /api/admin/config': ['admin'],
  'GET /api/admin/reports': ['admin', 'manager'],
};

function authorize(req, res, next) {
  const routeKey = `${req.method} ${req.path}`;
  
  // Find matching permission rule (support wildcards)
  let allowedRoles = null;
  for (const [pattern, roles] of Object.entries(routePermissions)) {
    if (matchRoute(routeKey, pattern)) {
      allowedRoles = roles;
      break;
    }
  }
  
  // DEFAULT DENY — if no rule matches, deny access
  if (!allowedRoles) {
    return res.status(403).json({ error: 'Access denied' });
  }
  
  // Public routes
  if (allowedRoles.includes('*')) return next();
  
  // Check authentication
  if (!req.user) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  
  // Check role
  if (!allowedRoles.includes(req.user.role)) {
    return res.status(403).json({ error: 'Insufficient privileges' });
  }
  
  next();
}

// Apply to all routes
app.use(authorize);
```

**Spring Security Centralized Config:**

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                // Public endpoints
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/health").permitAll()
                
                // User endpoints
                .requestMatchers("/api/user/**").hasAnyRole("USER", "ADMIN")
                
                // Admin endpoints
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                
                // Management endpoints — blocked
                .requestMatchers("/actuator/**").denyAll()
                .requestMatchers("/api/internal/**").denyAll()
                .requestMatchers("/debug/**").denyAll()
                
                // DEFAULT DENY
                .anyRequest().denyAll()
            )
            .csrf(csrf -> csrf.disable())
            .sessionManagement(session -> 
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            );
        
        return http.build();
    }
}
```

---

## 7. Testing Methodology

### 7.1 Comprehensive Test Plan

```
+──────────────────────────────────────────────────────────────+
│    Function-Level Access Control Testing Plan                │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  PHASE 1: ENDPOINT DISCOVERY                                 │
│  ├── Crawl application as admin → record all endpoints       │
│  ├── Analyze JavaScript for hidden API calls                 │
│  ├── Check Swagger/OpenAPI documentation                     │
│  ├── Review robots.txt, sitemap.xml                          │
│  ├── Fuzz for common admin/debug paths                       │
│  └── Check for exposed management endpoints                  │
│                                                              │
│  PHASE 2: ROLE MATRIX TESTING                                │
│  ├── For each discovered endpoint:                           │
│  │   ├── Test as unauthenticated                             │
│  │   ├── Test as guest role                                  │
│  │   ├── Test as user role                                   │
│  │   ├── Test as manager role                                │
│  │   └── Test as admin role                                  │
│  └── Record results in an access-control matrix              │
│                                                              │
│  PHASE 3: HTTP METHOD TESTING                                │
│  ├── For each restricted endpoint:                           │
│  │   ├── Test GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS   │
│  │   ├── Test method override headers                        │
│  │   └── Test with and without body content                  │
│  └── Compare responses across methods                        │
│                                                              │
│  PHASE 4: AUTOMATED TESTING                                  │
│  ├── Configure Autorize with each role level                 │
│  ├── Browse as admin → Autorize replays as user              │
│  ├── Review PASS/FAIL results                                │
│  └── Run nuclei access-control templates                     │
│                                                              │
│  PHASE 5: DOCUMENTATION                                      │
│  ├── Create access-control matrix                            │
│  ├── List all failed controls with severity                  │
│  └── Recommend remediation for each finding                  │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 7.2 Access-Control Matrix

```
+──────────────────────────────────────────────────────────────+
│                 Access Control Matrix                         │
+──────────────────────────────────────────────────────────────+
│                                                              │
│  Endpoint              │ Guest │ User │ Mgr │ Admin │ Expect │
│  ──────────────────────┼───────┼──────┼─────┼───────┼────────│
│  GET /api/public/items │  200  │ 200  │ 200 │  200  │ 200    │
│  GET /api/user/profile │  401  │ 200  │ 200 │  200  │ 401/200│
│  PUT /api/user/profile │  401  │ 200  │ 200 │  200  │ 401/200│
│  GET /api/mgr/reports  │  401  │ 403  │ 200 │  200  │ 403/200│
│  GET /api/admin/users  │  401  │ 200! │ 200!│  200  │ 403    │
│  POST /api/admin/users │  401  │ 200! │ 200!│  200  │ 403    │
│  DEL /api/admin/user/1 │  401  │ 200! │ 200!│  200  │ 403    │
│  GET /actuator/env     │  200! │ 200! │ 200!│  200! │ 404    │
│  GET /api/debug/sql    │  200! │ 200! │ 200!│  200  │ 404    │
│                                                              │
│  200! = VULNERABILITY FOUND (unauthorized access granted)    │
│                                                              │
+──────────────────────────────────────────────────────────────+
```

### 7.3 Automated Matrix Testing Script

```python
import requests
import json

BASE_URL = "https://target.com"

# Define roles and their tokens
ROLES = {
    "guest": None,
    "user": "Bearer eyJ...user...",
    "manager": "Bearer eyJ...manager...",
    "admin": "Bearer eyJ...admin...",
}

# Define endpoints and expected access
ENDPOINTS = [
    {"method": "GET",    "path": "/api/public/items",  "expect": {"guest": 200, "user": 200, "manager": 200, "admin": 200}},
    {"method": "GET",    "path": "/api/user/profile",  "expect": {"guest": 401, "user": 200, "manager": 200, "admin": 200}},
    {"method": "GET",    "path": "/api/admin/users",   "expect": {"guest": 401, "user": 403, "manager": 403, "admin": 200}},
    {"method": "POST",   "path": "/api/admin/users",   "expect": {"guest": 401, "user": 403, "manager": 403, "admin": 200}},
    {"method": "DELETE", "path": "/api/admin/users/1",  "expect": {"guest": 401, "user": 403, "manager": 403, "admin": 200}},
    {"method": "GET",    "path": "/actuator/env",       "expect": {"guest": 404, "user": 404, "manager": 404, "admin": 404}},
]

print(f"{'Endpoint':<35} {'Role':<10} {'Got':<5} {'Exp':<5} {'Status'}")
print("=" * 70)

for ep in ENDPOINTS:
    for role, token in ROLES.items():
        headers = {}
        if token:
            headers["Authorization"] = token
        
        r = requests.request(ep["method"], BASE_URL + ep["path"], headers=headers)
        expected = ep["expect"][role]
        status = "PASS" if r.status_code == expected else "FAIL !!!"
        
        print(f"{ep['method']} {ep['path']:<30} {role:<10} "
              f"{r.status_code:<5} {expected:<5} {status}")
```

---

## 8. Prevention & Remediation

### 8.1 Security Architecture Principles

| Principle | Implementation |
|---|---|
| **Default deny** | Block all access unless explicitly permitted |
| **Centralized enforcement** | Single authorization layer for all endpoints |
| **Least privilege** | Users get minimum required permissions |
| **Defense in depth** | Multiple layers: gateway + middleware + application |
| **Fail securely** | On auth error, deny access (don't default to allow) |
| **Complete mediation** | Every request is checked, no caching of auth decisions |
| **Separation of duties** | Different roles for different functions |

### 8.2 Security Headers and Configuration

```nginx
# Nginx — block internal endpoints
location /actuator {
    deny all;
    return 404;
}

location /debug {
    deny all;
    return 404;
}

location /api/internal {
    # Only allow from internal network
    allow 10.0.0.0/8;
    deny all;
}

# Block dangerous HTTP methods globally
if ($request_method !~ ^(GET|POST|PUT|DELETE|PATCH)$) {
    return 405;
}
```

```apache
# Apache — restrict admin endpoints
<Location /admin>
    <LimitExcept GET POST PUT DELETE PATCH>
        Require all denied
    </LimitExcept>
    Require role admin
</Location>

# Block trace method
TraceEnable Off
```

### 8.3 Prevention Checklist

```
+──────────────────────────────────────────────────────────+
│       Function-Level AC Prevention Checklist             │
+──────────────────────────────────────────────────────────+
│                                                          │
│  [ ] Implement centralized authorization middleware      │
│  [ ] Default deny — all endpoints blocked by default     │
│  [ ] Define and maintain access-control matrix           │
│  [ ] Enforce authorization on every endpoint             │
│  [ ] Check all HTTP methods, not just GET/POST           │
│  [ ] Disable/restrict management endpoints in prod       │
│  [ ] Remove debug endpoints before deployment            │
│  [ ] Restrict API documentation access in production     │
│  [ ] Disable unnecessary HTTP methods                    │
│  [ ] Block method override headers in production         │
│  [ ] Implement automated access-control testing          │
│  [ ] Review new endpoints for auth checks in code review │
│  [ ] Use API gateway for centralized enforcement         │
│  [ ] Log and alert on unauthorized access attempts       │
│  [ ] Regular access-control audits                       │
│                                                          │
+──────────────────────────────────────────────────────────+
```

---

## Summary Table

| Topic | Key Point |
|---|---|
| **Definition** | Missing authorization checks on application functions/endpoints |
| **OWASP Mapping** | A01:2021 Broken Access Control / CWE-285, CWE-862 |
| **Root Cause** | Relying on UI hiding instead of server-side enforcement |
| **Admin API Exposure** | Admin endpoints accessible to regular users |
| **HTTP Method Bypass** | Auth checks only on GET/POST, not PUT/DELETE/PATCH |
| **Method Override** | `X-HTTP-Method-Override` headers bypass restrictions |
| **Hidden Endpoints** | Debug, actuator, internal endpoints left unprotected |
| **RBAC** | Role-based: simple, per-role permissions |
| **ABAC** | Attribute-based: fine-grained, context-aware policies |
| **Centralized Auth** | Single enforcement point prevents forgotten checks |
| **Default Deny** | All access blocked unless explicitly permitted |
| **Testing** | Access-control matrix across all roles and endpoints |

---

## Revision Questions

1. **What is the difference between "missing function-level access control" and an IDOR vulnerability? Provide a concrete example of each.**

2. **An application uses Apache's `<Limit GET POST>` directive to restrict admin endpoints. Explain why this is insufficient and how an attacker could bypass it. What should be used instead?**

3. **Compare RBAC and ABAC authorization models. For each, describe a scenario where it is the better choice, and explain the trade-offs.**

4. **Describe how you would discover hidden API endpoints in a single-page application (React/Angular). What tools and techniques would you use?**

5. **Design a centralized authorization middleware for an Express.js application with three roles (user, manager, admin). Show the route permission configuration and the middleware code.**

6. **You discover that Spring Boot Actuator endpoints (`/actuator/env`, `/actuator/heapdump`) are accessible without authentication. Explain the security impact and the remediation steps.**

---

*Previous: [03-vertical-privilege-escalation.md](03-vertical-privilege-escalation.md) | Next: [05-jwt-security-testing.md](05-jwt-security-testing.md)*

---

*[Back to README](../README.md)*
