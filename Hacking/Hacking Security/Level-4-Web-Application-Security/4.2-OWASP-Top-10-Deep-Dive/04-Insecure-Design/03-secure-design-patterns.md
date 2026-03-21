# Unit 4: A04 - Insecure Design — Topic 3: Secure Design Patterns

## Overview

Secure design patterns are **proven architectural solutions** to recurring security problems. Just as software design patterns (MVC, Observer, Factory) solve common development challenges, secure design patterns provide blueprints for building security into application architecture. This topic covers the most important patterns with implementation guidance.

---

## 1. Authentication Design Patterns

### Centralized Authentication Service

```
┌─────────────────────────────────────────────────────────────────┐
│             CENTRALIZED AUTH PATTERN                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ❌ BAD: Each service handles its own auth                     │
│                                                                 │
│  [Service A] ── own auth logic                                 │
│  [Service B] ── own auth logic  → Inconsistent, duplicated     │
│  [Service C] ── own auth logic                                 │
│                                                                 │
│  ✅ GOOD: Single Identity Provider                             │
│                                                                 │
│  [Service A] ─┐                                                │
│  [Service B] ─┼──→ [Identity Provider / Auth Service]          │
│  [Service C] ─┘         │                                      │
│                    [User Store]                                 │
│                                                                 │
│  Benefits:                                                     │
│  - Single source of truth for authentication                   │
│  - Consistent security policies                                │
│  - Easier to audit and update                                  │
│  - MFA enforced centrally                                      │
└─────────────────────────────────────────────────────────────────┘
```

### Token-Based Authentication Flow

```
┌────────┐         ┌──────────┐         ┌───────────┐
│ Client │────1───→│ Auth     │────2───→│ User DB   │
│        │         │ Service  │←───3────│           │
│        │←───4────│          │         │           │
│        │         └──────────┘         └───────────┘
│        │
│        │────5───→┌──────────┐
│        │         │ API      │
│        │←───6────│ Service  │
└────────┘         └──────────┘

1. Client sends credentials
2. Auth service validates against user store
3. User store returns validation result
4. Auth service returns signed JWT/token
5. Client sends request with token
6. API validates token signature (no DB lookup needed)
```

---

## 2. Authorization Design Patterns

### RBAC (Role-Based Access Control)

```
Users ──→ Roles ──→ Permissions

Example:
┌──────────┐    ┌──────────────┐    ┌────────────────────┐
│ Alice    │───→│ Admin        │───→│ create_user        │
│          │    │              │    │ delete_user        │
│          │    │              │    │ view_reports       │
│          │    │              │    │ manage_settings    │
├──────────┤    ├──────────────┤    ├────────────────────┤
│ Bob      │───→│ Editor       │───→│ create_content     │
│          │    │              │    │ edit_content       │
│          │    │              │    │ view_reports       │
├──────────┤    ├──────────────┤    ├────────────────────┤
│ Charlie  │───→│ Viewer       │───→│ view_content       │
│          │    │              │    │ view_reports       │
└──────────┘    └──────────────┘    └────────────────────┘
```

### ABAC (Attribute-Based Access Control)

```
Policy Engine evaluates attributes:
- Subject attributes (user role, department, clearance)
- Resource attributes (classification, owner, type)
- Action attributes (read, write, delete)
- Environment attributes (time, location, device)

Example Policy:
IF subject.role == "doctor"
   AND resource.type == "medical_record"
   AND resource.patient.assigned_doctor == subject.id
   AND environment.network == "hospital_internal"
THEN ALLOW read

→ More granular than RBAC, handles complex scenarios
→ Used in healthcare, government, financial systems
```

### Authorization Middleware Pattern

```python
# Centralized authorization middleware
from functools import wraps

def require_permission(*permissions):
    """Decorator that checks if user has required permissions"""
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            user = get_current_user()
            if not user:
                abort(401, "Authentication required")
            
            user_perms = get_user_permissions(user)
            
            for perm in permissions:
                if perm not in user_perms:
                    log_access_denied(user, perm, request.path)
                    abort(403, "Insufficient permissions")
            
            return f(*args, **kwargs)
        return wrapper
    return decorator

# Usage
@app.route('/admin/users', methods=['GET'])
@require_permission('view_users')
def list_users():
    return get_all_users()

@app.route('/admin/users', methods=['DELETE'])
@require_permission('delete_users', 'admin_access')
def delete_user():
    return remove_user(request.json['user_id'])
```

---

## 3. Input Validation Pattern (Gatekeeper)

```
┌─────────────────────────────────────────────────────────────────┐
│               GATEKEEPER / VALIDATION LAYER                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [Client Input]                                                │
│       │                                                        │
│       ▼                                                        │
│  ┌─────────────────────┐                                       │
│  │  Schema Validation  │  ← JSON Schema, OpenAPI spec          │
│  │  - Type checking    │                                       │
│  │  - Required fields  │                                       │
│  │  - Size limits      │                                       │
│  └────────┬────────────┘                                       │
│           ▼                                                    │
│  ┌─────────────────────┐                                       │
│  │  Business Rules     │  ← Domain-specific validation         │
│  │  - Value ranges     │                                       │
│  │  - Format patterns  │                                       │
│  │  - Referential      │                                       │
│  └────────┬────────────┘                                       │
│           ▼                                                    │
│  ┌─────────────────────┐                                       │
│  │  Security Rules     │  ← Injection, XSS prevention         │
│  │  - Sanitization     │                                       │
│  │  - Encoding         │                                       │
│  │  - Parameterization │                                       │
│  └────────┬────────────┘                                       │
│           ▼                                                    │
│  [Business Logic]   ← Only clean, validated data reaches here  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Secure Session Management Pattern

```
Session Security Requirements:
┌────────────────────────┬─────────────────────────────────────────┐
│ Requirement            │ Implementation                          │
├────────────────────────┼─────────────────────────────────────────┤
│ Unpredictable IDs      │ Cryptographic random (128+ bits)       │
│ Secure transmission    │ Secure, HttpOnly, SameSite cookies     │
│ Timeout                │ Absolute (8hr) + Idle (30min)          │
│ Regeneration           │ New ID after privilege change          │
│ Invalidation           │ Server-side session destruction        │
│ Binding                │ Tie to IP/User-Agent (optional)        │
│ Concurrent control     │ Limit active sessions per user         │
└────────────────────────┴─────────────────────────────────────────┘
```

```python
# Secure session configuration (Flask)
app.config.update(
    SESSION_COOKIE_SECURE=True,       # HTTPS only
    SESSION_COOKIE_HTTPONLY=True,      # No JavaScript access
    SESSION_COOKIE_SAMESITE='Lax',    # CSRF protection
    PERMANENT_SESSION_LIFETIME=timedelta(hours=8),  # Absolute timeout
    SESSION_COOKIE_NAME='__Host-session',  # Cookie prefix security
)

# Regenerate session on privilege change
@app.route('/login', methods=['POST'])
def login():
    if authenticate(username, password):
        session.clear()                    # Clear old session
        session.regenerate()               # New session ID
        session['user_id'] = user.id
        session['login_time'] = datetime.utcnow()
        session['ip'] = request.remote_addr
        return redirect('/dashboard')
```

---

## 5. Rate Limiting Patterns

### Token Bucket Algorithm

```
┌─────────────────────────────────────────────────────────────────┐
│               TOKEN BUCKET RATE LIMITER                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Bucket capacity: 10 tokens                                    │
│  Refill rate: 1 token per second                               │
│                                                                 │
│  [████████░░]  8 tokens remaining                              │
│                                                                 │
│  Each request consumes 1 token:                                │
│    Request → Check bucket → Token available? → Process         │
│                              No token? → 429 Too Many Requests │
│                                                                 │
│  Benefits:                                                     │
│  - Allows bursts (up to bucket size)                           │
│  - Smooth rate over time                                       │
│  - Simple to implement                                         │
└─────────────────────────────────────────────────────────────────┘
```

### Multi-Layer Rate Limiting

```python
# Layer 1: Global rate limit (all requests)
@app.before_request
def global_rate_limit():
    if not rate_limiter.allow(request.remote_addr, limit=100, window=60):
        abort(429, "Rate limit exceeded")

# Layer 2: Endpoint-specific rate limit
@app.route('/api/login', methods=['POST'])
@rate_limit(limit=5, window=300)  # 5 attempts per 5 minutes
def login():
    pass

# Layer 3: User-specific rate limit
@app.route('/api/password-reset', methods=['POST'])
@rate_limit(limit=3, window=3600, key=lambda: request.json.get('email'))
def password_reset():
    pass

# Layer 4: Business logic rate limit
@app.route('/api/transfer', methods=['POST'])
@rate_limit(limit=10, window=86400, key=lambda: current_user.id)  # 10/day
def transfer_funds():
    pass
```

---

## 6. Secure Error Handling Pattern

```
❌ INSECURE: Expose internal details
   Response: "Error: Column 'users.password' not found in MySQL 8.0"

✅ SECURE: Generic message + internal logging

def handle_error(error):
    # Log full details internally
    logger.error(f"Error: {error}", exc_info=True, extra={
        'request_id': request.id,
        'user_id': current_user.id,
        'endpoint': request.path,
        'ip': request.remote_addr
    })
    
    # Return generic message to client
    error_responses = {
        400: "Invalid request",
        401: "Authentication required",
        403: "Access denied",
        404: "Resource not found",
        500: "Internal server error"
    }
    
    return jsonify({
        'error': error_responses.get(error.code, "An error occurred"),
        'request_id': request.id  # For support reference
    }), error.code
```

---

## 7. Secure Communication Patterns

### API Security Pattern

```
┌─────────────────────────────────────────────────────────────────┐
│             API SECURITY CHECKLIST                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Transport: TLS 1.2+ (enforce HTTPS)                           │
│  Authentication: OAuth 2.0 / API keys (in header, not URL)     │
│  Authorization: Scope-based (read:users, write:users)          │
│  Input: JSON Schema validation                                 │
│  Output: Content-Type: application/json                        │
│  Rate Limiting: Per-key and per-IP                             │
│  Versioning: /api/v1/ (deprecation policy)                     │
│  CORS: Restrict to known origins                               │
│  Headers: X-Content-Type-Options: nosniff                      │
│  Pagination: Enforce max page size                             │
│  Filtering: Server-side only (don't trust client filters)      │
│  Error Handling: Generic errors, no stack traces                │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. Secure File Handling Pattern

```python
import os
import uuid
import magic

ALLOWED_TYPES = {'image/jpeg', 'image/png', 'image/gif', 'application/pdf'}
MAX_SIZE = 10 * 1024 * 1024  # 10 MB
UPLOAD_DIR = '/var/uploads/'

def secure_file_upload(file):
    # 1. Check file size
    if file.content_length > MAX_SIZE:
        raise ValueError("File too large")
    
    # 2. Check MIME type (by content, not extension)
    file_type = magic.from_buffer(file.read(2048), mime=True)
    file.seek(0)
    if file_type not in ALLOWED_TYPES:
        raise ValueError("File type not allowed")
    
    # 3. Generate random filename (prevent path traversal)
    ext = {'image/jpeg': '.jpg', 'image/png': '.png', 
           'image/gif': '.gif', 'application/pdf': '.pdf'}
    safe_name = f"{uuid.uuid4()}{ext.get(file_type, '')}"
    
    # 4. Store outside web root
    save_path = os.path.join(UPLOAD_DIR, safe_name)
    
    # 5. Prevent directory traversal
    real_path = os.path.realpath(save_path)
    if not real_path.startswith(os.path.realpath(UPLOAD_DIR)):
        raise ValueError("Invalid path")
    
    file.save(save_path)
    
    # 6. Set restrictive permissions
    os.chmod(save_path, 0o644)
    
    return safe_name
```

---

## Summary Table

| Pattern | Problem Solved | Key Principle |
|---------|---------------|---------------|
| Centralized Auth | Inconsistent authentication | Single source of truth |
| RBAC/ABAC | Unauthorized access | Least privilege |
| Gatekeeper | Invalid/malicious input | Validate early, validate once |
| Secure Session | Session hijacking | Unpredictable + secure cookies |
| Rate Limiting | Abuse, DoS, brute force | Throttle at multiple layers |
| Secure Errors | Information leakage | Log internal, show generic |
| API Security | API abuse | Defense in depth |
| Secure Upload | Malicious files | Validate content, not name |

---

## Revision Questions

1. Why is centralized authentication better than per-service authentication?
2. Compare RBAC and ABAC — when would you choose each?
3. Implement a three-layer input validation pattern for a user registration endpoint.
4. What are the essential cookie flags for secure session management?
5. Design a multi-layer rate limiting strategy for an e-commerce checkout.
6. Why should file uploads use random filenames and validate MIME type by content, not extension?

---

*Previous: [02-threat-modeling.md](02-threat-modeling.md) | Next: [04-attack-surface-reduction.md](04-attack-surface-reduction.md)*

---

*[Back to README](../README.md)*
