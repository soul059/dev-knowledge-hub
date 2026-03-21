# Unit 1: A01 - Broken Access Control — Topic 1: Access Control Concepts

## Overview

Access control is the process of **granting or denying requests to access resources** based on the identity and privileges of the requester. It is the #1 vulnerability category in the OWASP Top 10 (2021), moving up from position #5 in 2017, reflecting the fact that **94% of applications tested** showed some form of broken access control. Understanding access control models, principles, and enforcement mechanisms is foundational to both exploiting and defending web applications.

---

## 1. Authentication vs. Authorization

```
┌──────────────────────────────────────────────────────────────────┐
│                    SECURITY DECISION FLOW                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  User Request                                                    │
│       │                                                          │
│       ▼                                                          │
│  ┌──────────────────┐     NO      ┌─────────────────┐           │
│  │ AUTHENTICATION   │────────────▶│  401 Unauthorized│           │
│  │ "Who are you?"   │             └─────────────────┘           │
│  └────────┬─────────┘                                           │
│           │ YES                                                  │
│           ▼                                                      │
│  ┌──────────────────┐     NO      ┌─────────────────┐           │
│  │ AUTHORIZATION    │────────────▶│  403 Forbidden   │           │
│  │ "Can you do this?"│            └─────────────────┘           │
│  └────────┬─────────┘                                           │
│           │ YES                                                  │
│           ▼                                                      │
│  ┌──────────────────┐                                           │
│  │  ACCESS GRANTED  │                                           │
│  │  200 OK          │                                           │
│  └──────────────────┘                                           │
└──────────────────────────────────────────────────────────────────┘
```

| Aspect | Authentication | Authorization |
|--------|---------------|---------------|
| **Question** | "Who are you?" | "What can you do?" |
| **Verifies** | Identity | Permissions |
| **HTTP Error** | 401 Unauthorized | 403 Forbidden |
| **Mechanism** | Passwords, tokens, biometrics | Roles, policies, ACLs |
| **When** | Before authorization | After authentication |
| **Failure Impact** | Identity spoofing | Privilege abuse |

---

## 2. The Subject-Object-Operation Model

Every access control decision involves three components:

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│   SUBJECT   │────────▶│  OPERATION  │────────▶│   OBJECT    │
│             │         │             │         │             │
│ • User      │         │ • Read      │         │ • File      │
│ • Process   │         │ • Write     │         │ • Database  │
│ • Service   │         │ • Execute   │         │ • API       │
│ • Role      │         │ • Delete    │         │ • Function  │
│ • Group     │         │ • Admin     │         │ • Record    │
└─────────────┘         └─────────────┘         └─────────────┘
```

### Access Control Matrix

|  | File A | File B | Database | Admin Panel |
|--|--------|--------|----------|-------------|
| **Admin** | RWX | RWX | RWX | RWX |
| **Editor** | RW | RW | R | — |
| **Viewer** | R | R | R | — |
| **Guest** | R | — | — | — |

> R = Read, W = Write, X = Execute, — = No Access

---

## 3. Access Control Models

### 3.1 Discretionary Access Control (DAC)

The **resource owner** decides who gets access. Common in file systems (Linux chmod, Windows NTFS).

```
Owner: alice
┌─────────────────────────────────────┐
│  alice's_report.pdf                 │
│  Permissions:                       │
│    alice  → Read, Write, Delete     │
│    bob    → Read                    │
│    carol  → Read, Write            │
│    others → No Access              │
└─────────────────────────────────────┘

Alice (owner) can grant/revoke access to anyone
```

**Pros:** Flexible, user-friendly
**Cons:** No centralized control, vulnerable to Trojan horse attacks, users may grant excessive permissions

### 3.2 Mandatory Access Control (MAC)

A **central authority** enforces access based on security labels. Users cannot change permissions.

```
┌──────────────────────────────────────────────┐
│           SECURITY CLEARANCE LEVELS          │
├──────────────────────────────────────────────┤
│                                              │
│  TOP SECRET ─────── Can read: TS, S, C, U   │
│       │                                      │
│    SECRET ────────── Can read: S, C, U       │
│       │                                      │
│  CONFIDENTIAL ───── Can read: C, U           │
│       │                                      │
│  UNCLASSIFIED ───── Can read: U only         │
│                                              │
│  Rule: Subject clearance ≥ Object level      │
│  (Bell-LaPadula: No read up, no write down)  │
└──────────────────────────────────────────────┘
```

**Bell-LaPadula Model:**
- **Simple Security Rule (No Read Up):** A subject cannot read an object at a higher classification
- **Star Property (No Write Down):** A subject cannot write to an object at a lower classification

**Used in:** Military systems, SELinux, AppArmor

### 3.3 Role-Based Access Control (RBAC)

Access is granted based on **roles** assigned to users, not individual user permissions.

```
┌──────────┐     ┌──────────┐     ┌──────────────────┐
│  Users   │────▶│  Roles   │────▶│   Permissions    │
├──────────┤     ├──────────┤     ├──────────────────┤
│ alice    │──┐  │ admin    │────▶│ create_user      │
│ bob      │──┤  │ editor   │────▶│ edit_content     │
│ carol    │──┘  │ viewer   │────▶│ view_content     │
│ dave     │──┐  │ auditor  │────▶│ view_logs        │
│ eve      │──┘  └──────────┘     └──────────────────┘
│          │
│ alice ──── admin, editor
│ bob   ──── editor
│ carol ──── viewer
│ dave  ──── auditor
│ eve   ──── viewer
└──────────┘
```

**Key RBAC Concepts:**
| Concept | Description |
|---------|-------------|
| **Role Assignment** | User must have a role to exercise permission |
| **Role Authorization** | User's active role must be authorized |
| **Permission Authorization** | User can exercise permission only if authorized for active role |
| **Role Hierarchy** | Senior roles inherit permissions from junior roles |
| **Separation of Duties** | Mutually exclusive roles prevent conflict of interest |
| **Least Privilege** | Users only get permissions needed for their role |

**Code Example — RBAC in Python Flask:**

```python
# Vulnerable: No access control
@app.route('/admin/users')
def list_users():
    users = User.query.all()
    return jsonify([u.to_dict() for u in users])

# Secure: RBAC enforcement
from functools import wraps

def require_role(*roles):
    def decorator(f):
        @wraps(f)
        def wrapper(*args, **kwargs):
            if not current_user.is_authenticated:
                abort(401)
            if current_user.role not in roles:
                abort(403)
            return f(*args, **kwargs)
        return wrapper
    return decorator

@app.route('/admin/users')
@require_role('admin', 'super_admin')
def list_users():
    users = User.query.all()
    return jsonify([u.to_dict() for u in users])
```

### 3.4 Attribute-Based Access Control (ABAC)

Access decisions based on **attributes** of the subject, object, action, and environment.

```
┌────────────────────────────────────────────────────────────────┐
│                    ABAC POLICY ENGINE                          │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Subject Attributes    Object Attributes    Environment        │
│  ┌────────────────┐   ┌────────────────┐   ┌──────────────┐  │
│  │ role: doctor   │   │ type: medical  │   │ time: 9-5    │  │
│  │ dept: cardio   │   │ dept: cardio   │   │ IP: internal │  │
│  │ clearance: L3  │   │ sensitivity: H │   │ MFA: true    │  │
│  └────────┬───────┘   └───────┬────────┘   └──────┬───────┘  │
│           │                   │                    │           │
│           └───────────────────┼────────────────────┘           │
│                               ▼                                │
│                    ┌─────────────────────┐                     │
│                    │   Policy Decision   │                     │
│                    │   Point (PDP)       │                     │
│                    └─────────┬───────────┘                     │
│                              │                                 │
│              ┌───────────────┼───────────────┐                 │
│              ▼               ▼               ▼                 │
│          PERMIT           DENY          INDETERMINATE          │
└────────────────────────────────────────────────────────────────┘

Example Policy:
IF subject.role == "doctor"
AND subject.dept == object.dept
AND environment.time IN (09:00-17:00)
AND environment.network == "internal"
THEN PERMIT read(object)
```

### Model Comparison

| Feature | DAC | MAC | RBAC | ABAC |
|---------|-----|-----|------|------|
| **Control** | Owner | System | Admin | Policy |
| **Flexibility** | High | Low | Medium | Very High |
| **Scalability** | Low | Medium | High | Very High |
| **Complexity** | Low | High | Medium | Very High |
| **Use Case** | File systems | Military | Enterprise | Cloud/Healthcare |
| **Policy Granularity** | Per-object | Per-label | Per-role | Per-attribute |
| **User Override** | Yes | No | No | No |

---

## 4. Core Access Control Principles

### 4.1 Principle of Least Privilege

> Grant only the minimum permissions necessary to perform a task, and only for the duration needed.

```
BAD:  User → Full Admin Access → All Resources
GOOD: User → Specific Role → Only Required Resources

Example:
┌─────────────┐     ┌─────────────────────┐
│ Report User │────▶│ Can: view_reports   │
│             │     │ Cannot: edit_reports │
│             │     │ Cannot: delete_users │
│             │     │ Cannot: view_config  │
└─────────────┘     └─────────────────────┘
```

### 4.2 Separation of Duties

No single user should have enough privileges to misuse the system alone.

```
┌─────────────────────────────────────────────────┐
│           SEPARATION OF DUTIES                   │
├─────────────────────────────────────────────────┤
│                                                  │
│  Financial Transaction Flow:                     │
│                                                  │
│  Request ──▶ Approve ──▶ Execute ──▶ Audit      │
│     │           │           │          │         │
│  User A      User B     User C     User D       │
│  (Requester) (Manager) (Finance)  (Auditor)     │
│                                                  │
│  Rule: No single user can perform all steps     │
└─────────────────────────────────────────────────┘
```

### 4.3 Deny by Default

```
# WRONG: Allow by default
def check_access(user, resource):
    if user.role == 'blocked':
        return DENY
    return ALLOW  # Everything else is allowed!

# RIGHT: Deny by default
def check_access(user, resource):
    if user.role == 'admin' and resource.type in admin_resources:
        return ALLOW
    if user.role == 'editor' and resource.type in editor_resources:
        return ALLOW
    return DENY  # Everything else is denied
```

### 4.4 Complete Mediation

Every access to every resource must be checked — no caching of authorization decisions.

```
Request 1: /api/users/123 ──▶ [Check Auth] ──▶ ALLOW ──▶ Response
Request 2: /api/users/123 ──▶ [Check Auth] ──▶ ALLOW ──▶ Response  ← Check again!
Request 3: /api/users/456 ──▶ [Check Auth] ──▶ DENY  ──▶ 403

Never: "Already checked, skip auth" ← VULNERABLE
```

---

## 5. Policy Enforcement Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                 ACCESS CONTROL ARCHITECTURE                    │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Client Request                                                │
│       │                                                        │
│       ▼                                                        │
│  ┌─────────────────────────┐                                  │
│  │  Policy Enforcement     │  ← Intercepts every request      │
│  │  Point (PEP)            │    (middleware, filter, proxy)    │
│  └───────────┬─────────────┘                                  │
│              │                                                 │
│              ▼                                                 │
│  ┌─────────────────────────┐  ┌─────────────────────────┐    │
│  │  Policy Decision        │◀─│  Policy Information     │    │
│  │  Point (PDP)            │  │  Point (PIP)            │    │
│  │  (evaluates rules)      │  │  (user/role/attribute DB)│    │
│  └───────────┬─────────────┘  └─────────────────────────┘    │
│              │                                                 │
│              ▼                                                 │
│  ┌─────────────────────────┐                                  │
│  │  Policy Administration  │  ← Where admins define rules     │
│  │  Point (PAP)            │                                  │
│  └─────────────────────────┘                                  │
│                                                                │
│  XACML Components:                                            │
│  PEP = Middleware/API Gateway                                 │
│  PDP = Authorization Service (OPA, Casbin, Keycloak)         │
│  PIP = User DB, LDAP, Active Directory                       │
│  PAP = Admin Console                                          │
└────────────────────────────────────────────────────────────────┘
```

### Implementation Example — Express.js Middleware

```javascript
// Policy Enforcement Point (PEP) — Middleware
const authorize = (requiredPermission) => {
    return async (req, res, next) => {
        // 1. Extract identity (authentication already done)
        const userId = req.user?.id;
        if (!userId) return res.status(401).json({ error: 'Unauthenticated' });

        // 2. Query Policy Information Point (PIP)
        const userRoles = await getUserRoles(userId);
        const userPermissions = await getPermissionsForRoles(userRoles);

        // 3. Policy Decision Point (PDP)
        if (userPermissions.includes(requiredPermission)) {
            // 4. ALLOW — proceed
            next();
        } else {
            // 5. DENY — log and reject
            auditLog.warn(`Access denied: user=${userId} perm=${requiredPermission}`);
            res.status(403).json({ error: 'Insufficient permissions' });
        }
    };
};

// Usage
app.get('/api/users', authorize('users:read'), listUsers);
app.post('/api/users', authorize('users:create'), createUser);
app.delete('/api/users/:id', authorize('users:delete'), deleteUser);
```

---

## 6. Access Control in Web Applications

### Common Enforcement Points

| Layer | Mechanism | Example |
|-------|-----------|---------|
| **Reverse Proxy** | URL-based rules | Nginx `location`, Apache `<Directory>` |
| **API Gateway** | Route policies | Kong, AWS API Gateway |
| **Framework Middleware** | Decorators/filters | `@login_required`, `[Authorize]` |
| **Business Logic** | Code-level checks | `if (user.owns(resource))` |
| **Database** | Row-level security | PostgreSQL RLS, SQL views |
| **OS/File System** | File permissions | Linux chmod, Windows ACLs |

### Web Framework Examples

```python
# Django — class-based permission
from rest_framework.permissions import BasePermission

class IsOwnerOrAdmin(BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.user.is_staff:
            return True
        return obj.owner == request.user

class DocumentViewSet(viewsets.ModelViewSet):
    permission_classes = [IsAuthenticated, IsOwnerOrAdmin]
    
    def get_queryset(self):
        # Row-level filtering: users only see their own documents
        if self.request.user.is_staff:
            return Document.objects.all()
        return Document.objects.filter(owner=self.request.user)
```

```java
// Spring Security — method-level security
@RestController
@RequestMapping("/api/documents")
public class DocumentController {
    
    @PreAuthorize("hasRole('ADMIN') or @documentService.isOwner(#id, authentication.principal)")
    @GetMapping("/{id}")
    public Document getDocument(@PathVariable Long id) {
        return documentService.findById(id);
    }
    
    @PreAuthorize("hasAnyRole('ADMIN', 'EDITOR')")
    @PostMapping
    public Document createDocument(@RequestBody Document doc) {
        return documentService.save(doc);
    }
}
```

---

## Summary Table

| Concept | Description | Security Relevance |
|---------|-------------|-------------------|
| **DAC** | Owner controls access | Flexible but risky |
| **MAC** | System enforces labels | Strongest but rigid |
| **RBAC** | Role-based permissions | Most common in web apps |
| **ABAC** | Attribute-based policies | Most granular |
| **Least Privilege** | Minimum needed access | Limits blast radius |
| **Deny by Default** | Block unless explicitly allowed | Prevents access gaps |
| **Complete Mediation** | Check every request | No auth bypass via caching |
| **PEP/PDP/PIP** | Enforcement architecture | Centralized decisions |

---

## Revision Questions

1. Explain the difference between authentication and authorization. What HTTP status codes correspond to each failure?
2. Compare RBAC and ABAC. When would you choose one over the other in a web application?
3. What is the Bell-LaPadula model? How do its "No Read Up" and "No Write Down" rules prevent information leakage?
4. Describe the Principle of Least Privilege and give a practical web application example.
5. Draw the PEP/PDP/PIP/PAP architecture and explain each component's role.
6. Why is "deny by default" critical? Show a code example comparing allow-by-default vs deny-by-default implementations.

---

*Previous: None (First topic) | Next: [02-common-weaknesses.md](02-common-weaknesses.md)*

---

*[Back to README](../README.md)*
