# Unit 1: A01 - Broken Access Control — Topic 7: Prevention and Remediation

## Overview

Preventing broken access control requires a **defense-in-depth approach** combining architectural decisions, framework-level enforcement, code-level checks, and operational monitoring. This topic covers deny-by-default principles, centralized access control mechanisms, server-side enforcement patterns, CORS hardening, rate limiting, framework-specific implementations (Spring Security, Django, Express.js), and a comprehensive code review checklist for access control.

---

## 1. Deny-by-Default Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    DENY-BY-DEFAULT PRINCIPLE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ❌ ALLOW-BY-DEFAULT (Dangerous):                              │
│  ┌──────────────────────────────────────────┐                  │
│  │  Everything is accessible unless          │                  │
│  │  explicitly blocked                       │                  │
│  │                                           │                  │
│  │  /public     → ALLOW                     │                  │
│  │  /admin      → DENY (blocked)            │                  │
│  │  /api/secret → ??? (forgot to block!)    │ ← VULNERABLE    │
│  └──────────────────────────────────────────┘                  │
│                                                                 │
│  ✅ DENY-BY-DEFAULT (Secure):                                  │
│  ┌──────────────────────────────────────────┐                  │
│  │  Everything is blocked unless              │                  │
│  │  explicitly allowed                        │                  │
│  │                                           │                  │
│  │  /public     → ALLOW (explicitly opened) │                  │
│  │  /admin      → DENY (default)            │                  │
│  │  /api/secret → DENY (default)            │ ← SAFE          │
│  └──────────────────────────────────────────┘                  │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation Pattern

```python
# DENY-BY-DEFAULT — Flask example
from functools import wraps

# Global middleware: require authentication on ALL routes
@app.before_request
def require_authentication():
    # Whitelist of public endpoints
    public_endpoints = ['login', 'register', 'health', 'static']
    
    if request.endpoint in public_endpoints:
        return None  # Allow access
    
    if not current_user.is_authenticated:
        abort(401)  # Deny by default
    
    # Additional: check if endpoint has explicit permission
    required_perm = ENDPOINT_PERMISSIONS.get(request.endpoint)
    if required_perm and required_perm not in current_user.permissions:
        abort(403)  # Deny if no explicit permission

# Explicit permission mapping
ENDPOINT_PERMISSIONS = {
    'admin_dashboard': 'admin:read',
    'admin_users': 'admin:users',
    'admin_config': 'admin:config',
    'user_profile': 'user:read',
    'user_update': 'user:write',
}
```

---

## 2. Centralized Access Control

### Centralized Authorization Service

```
┌────────────────────────────────────────────────────────────────┐
│           CENTRALIZED ACCESS CONTROL ARCHITECTURE             │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐                 │
│  │Service A │   │Service B │   │Service C │                 │
│  └────┬─────┘   └────┬─────┘   └────┬─────┘                 │
│       │              │              │                          │
│       └──────────────┼──────────────┘                          │
│                      │                                         │
│                      ▼                                         │
│         ┌────────────────────────┐                            │
│         │  Authorization Service  │  ← Single source of truth │
│         │  (OPA / Casbin / ABAC)  │                            │
│         └───────────┬────────────┘                            │
│                     │                                          │
│                     ▼                                          │
│         ┌────────────────────────┐                            │
│         │  Policy Store           │                            │
│         │  (policies, roles,      │                            │
│         │   permissions)          │                            │
│         └────────────────────────┘                            │
│                                                                │
│  Benefits:                                                     │
│  • Single point for policy management                         │
│  • Consistent enforcement across services                     │
│  • Audit trail in one location                                │
│  • Policy changes don't require code deployments              │
└────────────────────────────────────────────────────────────────┘
```

### Open Policy Agent (OPA) Example

```rego
# OPA policy file: authz.rego
package authz

default allow = false

# Admin can access anything
allow {
    input.user.role == "admin"
}

# Users can access their own resources
allow {
    input.user.role == "user"
    input.resource.owner == input.user.id
    input.action == "read"
}

# Users can update their own profile
allow {
    input.user.role == "user"
    input.resource.type == "profile"
    input.resource.owner == input.user.id
    input.action == "update"
}

# Managers can read team reports
allow {
    input.user.role == "manager"
    input.resource.type == "report"
    input.resource.team == input.user.team
    input.action == "read"
}
```

```python
# Python integration with OPA
import requests

def check_authorization(user, action, resource):
    """Query OPA for authorization decision"""
    input_data = {
        "input": {
            "user": {"id": user.id, "role": user.role, "team": user.team},
            "action": action,
            "resource": resource
        }
    }
    
    resp = requests.post(
        'http://localhost:8181/v1/data/authz/allow',
        json=input_data
    )
    
    return resp.json().get('result', False)

# Usage in middleware
@app.before_request
def authorize():
    user = get_current_user()
    resource = get_resource_from_request()
    action = get_action_from_method()
    
    if not check_authorization(user, action, resource):
        abort(403)
```

---

## 3. Framework-Specific Implementations

### Django REST Framework

```python
# settings.py — Global default permissions
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',  # Deny unauthenticated by default
    ],
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ],
}

# Custom permission classes
from rest_framework.permissions import BasePermission

class IsOwnerOrReadOnly(BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in ('GET', 'HEAD', 'OPTIONS'):
            return True
        return obj.owner == request.user

class IsAdminOrManager(BasePermission):
    def has_permission(self, request, view):
        return request.user.role in ('admin', 'manager')

# ViewSet with multiple permission layers
class DocumentViewSet(viewsets.ModelViewSet):
    serializer_class = DocumentSerializer
    permission_classes = [IsAuthenticated, IsOwnerOrReadOnly]
    
    def get_queryset(self):
        """Row-level security: users only see their own docs"""
        user = self.request.user
        if user.is_staff:
            return Document.objects.all()
        return Document.objects.filter(owner=user)
    
    def perform_create(self, serializer):
        """Auto-assign owner on creation"""
        serializer.save(owner=self.request.user)
    
    def get_permissions(self):
        """Different permissions per action"""
        if self.action in ('destroy', 'update'):
            return [IsAuthenticated(), IsOwnerOrReadOnly()]
        if self.action == 'list':
            return [IsAuthenticated()]
        return super().get_permissions()
```

### Spring Security (Java)

```java
// SecurityConfig.java
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
                
                // Admin-only endpoints
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                
                // Authenticated endpoints
                .requestMatchers("/api/**").authenticated()
                
                // Deny everything else
                .anyRequest().denyAll()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt());
        
        return http.build();
    }
}

// Method-level security
@RestController
@RequestMapping("/api/documents")
public class DocumentController {
    
    @PreAuthorize("hasRole('ADMIN') or @securityService.isOwner(#id, principal)")
    @GetMapping("/{id}")
    public ResponseEntity<Document> getDocument(@PathVariable Long id) {
        return ResponseEntity.ok(documentService.findById(id));
    }
    
    @PreAuthorize("hasRole('ADMIN')")
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteDocument(@PathVariable Long id) {
        documentService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### Express.js / Node.js

```javascript
// middleware/authorize.js
const authorize = (...allowedRoles) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: 'Authentication required' });
        }
        
        if (!allowedRoles.includes(req.user.role)) {
            // Log the attempt
            logger.warn('Authorization failure', {
                userId: req.user.id,
                role: req.user.role,
                requiredRoles: allowedRoles,
                path: req.path,
                method: req.method,
                ip: req.ip,
            });
            return res.status(403).json({ error: 'Insufficient permissions' });
        }
        
        next();
    };
};

// middleware/ownerCheck.js
const requireOwnership = (getResourceOwner) => {
    return async (req, res, next) => {
        try {
            const ownerId = await getResourceOwner(req);
            
            if (req.user.role === 'admin' || req.user.id === ownerId) {
                next();
            } else {
                res.status(403).json({ error: 'You do not own this resource' });
            }
        } catch (err) {
            res.status(500).json({ error: 'Authorization check failed' });
        }
    };
};

// Route definitions
const router = express.Router();

// Admin-only routes
router.get('/admin/users', authorize('admin'), adminController.listUsers);
router.delete('/admin/users/:id', authorize('admin'), adminController.deleteUser);

// Owner-or-admin routes
router.get('/documents/:id', 
    authorize('admin', 'user'),
    requireOwnership(async (req) => {
        const doc = await Document.findById(req.params.id);
        return doc?.ownerId;
    }),
    documentController.getDocument
);

// Public routes (no middleware)
router.get('/public/info', publicController.getInfo);
```

---

## 4. CORS Hardening

```python
# SECURE CORS Configuration — Flask
from flask_cors import CORS

CORS(app, 
    origins=[
        'https://app.example.com',
        'https://admin.example.com'
    ],
    methods=['GET', 'POST', 'PUT', 'DELETE'],
    allow_headers=['Content-Type', 'Authorization'],
    expose_headers=['X-Request-Id'],
    supports_credentials=True,
    max_age=3600  # Preflight cache duration
)
```

```nginx
# SECURE CORS — Nginx
location /api/ {
    # Only allow specific origins
    set $cors_origin "";
    if ($http_origin ~* "^https://(app|admin)\.example\.com$") {
        set $cors_origin $http_origin;
    }
    
    add_header Access-Control-Allow-Origin $cors_origin always;
    add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE" always;
    add_header Access-Control-Allow-Headers "Content-Type, Authorization" always;
    add_header Access-Control-Allow-Credentials "true" always;
    add_header Access-Control-Max-Age 3600 always;
    
    # Handle preflight
    if ($request_method = OPTIONS) {
        return 204;
    }
}
```

### CORS Checklist

| Rule | ✅ Secure | ❌ Insecure |
|------|-----------|-------------|
| **Origin** | Explicit whitelist | `*` or reflected origin |
| **Credentials** | Only with whitelist | With wildcard origin |
| **Methods** | Only needed methods | All methods |
| **Headers** | Only needed headers | `*` |
| **Max-Age** | Reasonable (3600) | Very long (86400+) |
| **Null Origin** | Reject | Allow |

---

## 5. Rate Limiting and Monitoring

```python
# Rate limiting for sensitive endpoints
from flask_limiter import Limiter

limiter = Limiter(app, key_func=get_remote_address)

@app.route('/api/admin/users')
@limiter.limit("10 per minute")  # Strict limit for admin endpoints
@require_role('admin')
def admin_list_users():
    pass

@app.route('/api/login', methods=['POST'])
@limiter.limit("5 per minute")  # Brute force protection
def login():
    pass
```

### Audit Logging for Access Control

```python
import logging
from datetime import datetime

audit_logger = logging.getLogger('audit')

def log_access_event(user, resource, action, result, reason=None):
    """Log every access control decision"""
    audit_logger.info(json.dumps({
        'timestamp': datetime.utcnow().isoformat(),
        'user_id': user.id if user else 'anonymous',
        'user_role': user.role if user else 'none',
        'resource': resource,
        'action': action,
        'result': result,  # 'allowed' or 'denied'
        'reason': reason,
        'ip_address': request.remote_addr,
        'user_agent': request.headers.get('User-Agent'),
    }))

# Usage in middleware
@app.before_request
def access_control():
    user = get_current_user()
    resource = request.path
    action = request.method
    
    if is_authorized(user, resource, action):
        log_access_event(user, resource, action, 'allowed')
    else:
        log_access_event(user, resource, action, 'denied', 'insufficient_privileges')
        abort(403)
```

---

## 6. Database-Level Access Control

### PostgreSQL Row-Level Security (RLS)

```sql
-- Enable RLS on the documents table
ALTER TABLE documents ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see their own documents
CREATE POLICY user_documents ON documents
    FOR ALL
    USING (owner_id = current_setting('app.current_user_id')::integer);

-- Policy: Admins can see all documents
CREATE POLICY admin_all_documents ON documents
    FOR ALL
    USING (current_setting('app.current_user_role') = 'admin');

-- Set current user context (in application code)
-- SET app.current_user_id = '123';
-- SET app.current_user_role = 'user';
-- SELECT * FROM documents;  ← Only returns user 123's documents
```

### Query-Level Filtering

```python
# Always scope queries to current user
class DocumentService:
    def get_documents(self, user):
        """Get documents scoped to user's access"""
        if user.role == 'admin':
            return Document.query.all()
        return Document.query.filter_by(owner_id=user.id).all()
    
    def get_document(self, user, doc_id):
        """Get single document with ownership check"""
        doc = Document.query.get_or_404(doc_id)
        if user.role != 'admin' and doc.owner_id != user.id:
            abort(403)
        return doc
    
    def update_document(self, user, doc_id, data):
        """Update with ownership verification"""
        doc = Document.query.filter_by(
            id=doc_id,
            owner_id=user.id
        ).first_or_404()
        
        # Whitelist updateable fields
        allowed_fields = ['title', 'content', 'tags']
        for field in allowed_fields:
            if field in data:
                setattr(doc, field, data[field])
        
        db.session.commit()
        return doc
```

---

## 7. Code Review Checklist for Access Control

```
ACCESS CONTROL CODE REVIEW CHECKLIST
═══════════════════════════════════════

ARCHITECTURE:
□ Default-deny policy implemented globally
□ Centralized authorization mechanism (not per-endpoint)
□ Authorization cannot be bypassed via alternative paths
□ Access control enforced server-side (not client-only)

AUTHENTICATION BOUNDARY:
□ All non-public endpoints require authentication
□ Session/token validation on every request
□ Token expiration enforced
□ Logout properly invalidates sessions

AUTHORIZATION CHECKS:
□ Every CRUD operation has authorization check
□ Object ownership verified before access
□ Role checks performed server-side
□ No reliance on hidden fields/client data for authorization

DATA HANDLING:
□ Query results filtered by user's access scope
□ API responses don't leak other users' data
□ Bulk operations respect per-record authorization
□ Export/download functions check authorization

MASS ASSIGNMENT:
□ Input fields explicitly whitelisted
□ Sensitive fields (role, isAdmin) not bindable
□ Serializers/DTOs restrict exposed fields
□ No Object.assign(model, req.body) patterns

JWT/TOKENS:
□ Algorithm explicitly specified (no "none" accepted)
□ Strong signing secrets (256+ bits)
□ Expiration enforced and reasonable
□ Claims validated (iss, aud, exp)
□ Embedded keys (JWK/JKU) rejected

CORS:
□ Origins explicitly whitelisted (no reflection)
□ Credentials only with specific origins
□ NULL origin rejected
□ Methods and headers restricted

MONITORING:
□ Access control failures logged with context
□ Excessive failures trigger alerts
□ Audit trail for privilege changes
□ Admin actions logged with details
```

---

## Summary Table

| Prevention Strategy | Level | Impact |
|--------------------|-------|--------|
| **Deny by default** | Architecture | Prevents all access gaps |
| **Centralized auth** | Architecture | Consistent enforcement |
| **Server-side checks** | Code | Prevents client-side bypass |
| **Input whitelisting** | Code | Prevents mass assignment |
| **Row-level security** | Database | Defense in depth |
| **CORS hardening** | Configuration | Prevents cross-origin theft |
| **Rate limiting** | Operations | Slows enumeration |
| **Audit logging** | Operations | Detection and forensics |
| **Code review** | Process | Catches issues pre-deploy |

---

## Revision Questions

1. Explain deny-by-default architecture. Write a Flask middleware that enforces this principle.
2. What is a centralized authorization service? How does Open Policy Agent (OPA) implement this?
3. Compare access control implementations in Django, Spring Security, and Express.js. What patterns do they share?
4. How does PostgreSQL Row-Level Security (RLS) provide database-level access control? Write example policies.
5. Create a comprehensive CORS hardening configuration for Nginx that only allows specific origins.
6. Design a complete access control code review checklist with at least 15 items organized by category.

---

*Previous: [06-testing-methodology.md](06-testing-methodology.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
