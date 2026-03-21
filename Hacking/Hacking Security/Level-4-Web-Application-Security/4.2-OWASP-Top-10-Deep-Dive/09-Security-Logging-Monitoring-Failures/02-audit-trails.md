# Unit 9: A09 - Security Logging and Monitoring Failures — Topic 2: Audit Trails

## Overview

An audit trail is a **chronological record of system activities** that provides documentary evidence of the sequence of activities affecting a specific operation, procedure, or event. Unlike general logging, audit trails are specifically designed for **accountability, non-repudiation, and forensic investigation** — proving who did what, when, and from where.

---

## 1. Audit Trail vs Application Logs

```
APPLICATION LOGS:                    AUDIT TRAILS:
├── Debugging info                   ├── Who performed the action
├── Performance metrics              ├── What action was performed
├── Error stack traces               ├── When it occurred (precise)
├── Request/response data            ├── Where (IP, location, device)
├── May be deleted/rotated           ├── What changed (before/after)
└── For developers                   ├── Immutable (cannot be deleted)
                                     ├── Legally admissible
                                     └── For compliance/forensics
```

---

## 2. Audit Trail Requirements

```json
// Complete audit trail entry
{
    "audit_id": "aud-7f8e9a0b",
    "timestamp": "2024-01-15T10:30:45.123456Z",
    "actor": {
        "user_id": "usr-456",
        "username": "jsmith",
        "role": "admin",
        "ip_address": "203.0.113.50",
        "user_agent": "Mozilla/5.0...",
        "session_id_hash": "sha256:abc123..."
    },
    "action": {
        "type": "UPDATE",
        "resource": "user_account",
        "resource_id": "usr-789",
        "description": "Changed user role"
    },
    "changes": {
        "role": {
            "before": "viewer",
            "after": "admin"
        }
    },
    "context": {
        "request_id": "req-xyz",
        "service": "user-service",
        "environment": "production"
    },
    "result": "SUCCESS"
}
```

---

## 3. Implementation

```python
class AuditLogger:
    def __init__(self, storage_backend):
        self.storage = storage_backend  # Append-only storage
    
    def log_action(self, actor, action, resource, resource_id, 
                   changes=None, result="SUCCESS"):
        entry = {
            "audit_id": str(uuid.uuid4()),
            "timestamp": datetime.utcnow().isoformat() + "Z",
            "actor": {
                "user_id": actor.id,
                "username": actor.username,
                "role": actor.role,
                "ip_address": request.remote_addr,
            },
            "action": {
                "type": action,
                "resource": resource,
                "resource_id": str(resource_id),
            },
            "changes": changes,
            "result": result,
            "checksum": None  # Computed below
        }
        
        # Integrity: Hash the entry for tamper detection
        entry["checksum"] = self._compute_checksum(entry)
        
        # Store in append-only log
        self.storage.append(entry)
        return entry["audit_id"]
    
    def _compute_checksum(self, entry):
        data = json.dumps(entry, sort_keys=True, default=str)
        return hashlib.sha256(data.encode()).hexdigest()

# Usage
audit = AuditLogger(storage_backend)

# Log a role change
audit.log_action(
    actor=current_user,
    action="UPDATE",
    resource="user_account",
    resource_id=target_user.id,
    changes={"role": {"before": "viewer", "after": "admin"}},
    result="SUCCESS"
)
```

---

## 4. Critical Actions to Audit

| Category | Actions | Priority |
|----------|---------|----------|
| **Authentication** | Login, logout, password change, MFA changes | Critical |
| **Authorization** | Role changes, permission grants/revokes | Critical |
| **Data** | Create, read, update, delete sensitive records | Critical |
| **Administration** | User creation/deletion, system config changes | Critical |
| **Financial** | Transactions, refunds, balance changes | Critical |
| **Compliance** | Data exports, consent changes, deletion requests | High |

---

## 5. Immutability and Tamper Protection

```
APPROACHES TO IMMUTABLE AUDIT LOGS:

1. APPEND-ONLY DATABASE
   → Write-only permissions for application
   → No UPDATE or DELETE permissions
   → Separate database with dedicated credentials

2. HASH CHAINING
   → Each entry includes hash of previous entry
   → Tampering with any entry breaks the chain
   → Similar to blockchain concept

3. EXTERNAL STORAGE
   → Send logs to external SIEM in real-time
   → Even if server compromised, logs are safe
   → AWS CloudWatch, Splunk, ELK

4. DIGITAL SIGNATURES
   → Sign each log entry
   → Detect modification after the fact
   → Requires key management

5. WRITE-ONCE STORAGE
   → AWS S3 Object Lock
   → Azure Immutable Blob Storage
   → WORM (Write Once Read Many) compliance
```

---

## Summary Table

| Requirement | Description | Implementation |
|-------------|-------------|----------------|
| Completeness | All security-relevant actions logged | Event classification matrix |
| Immutability | Cannot be modified or deleted | Append-only, hash chains, WORM |
| Accountability | Who did what, when, from where | Actor details in every entry |
| Before/After | What changed | Diff recording |
| Integrity | Tamper detection | Checksums, signing |

---

## Revision Questions

1. What is the difference between application logs and audit trails?
2. Design a complete audit trail entry schema with all required fields.
3. Implement an audit logger with hash chaining for tamper detection.
4. What actions must always be audited in a financial application?
5. How do you ensure audit log immutability even if the application server is compromised?

---

*Previous: [01-logging-requirements.md](01-logging-requirements.md) | Next: [03-monitoring-and-alerting.md](03-monitoring-and-alerting.md)*

---

*[Back to README](../README.md)*
