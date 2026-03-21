# Unit 9: A09 - Security Logging and Monitoring Failures — Topic 1: Logging Requirements

## Overview

Security logging captures **security-relevant events** that enable detection of attacks, investigation of incidents, and compliance with regulations. Without adequate logging, breaches go undetected for months (average: 277 days according to IBM). This topic covers what to log, log formats, and the essential requirements for security-effective logging.

---

## 1. What Must Be Logged

```
┌─────────────────────────────────────────────────────────────────┐
│              SECURITY EVENT LOGGING MATRIX                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  AUTHENTICATION EVENTS:                                        │
│  ├── Successful logins (who, when, from where)                 │
│  ├── Failed logins (who attempted, from where)                 │
│  ├── Logout events                                             │
│  ├── Password changes and resets                               │
│  ├── MFA setup, verification, and failures                     │
│  ├── Account lockouts and unlocks                              │
│  └── Session creation and destruction                          │
│                                                                 │
│  AUTHORIZATION EVENTS:                                         │
│  ├── Access denied (403) events                                │
│  ├── Privilege escalation attempts                             │
│  ├── Role changes                                              │
│  ├── Permission modifications                                  │
│  └── Admin actions                                             │
│                                                                 │
│  INPUT VALIDATION EVENTS:                                      │
│  ├── Input validation failures                                 │
│  ├── Suspected injection attempts                              │
│  ├── File upload violations                                    │
│  └── Rate limit triggers                                       │
│                                                                 │
│  DATA ACCESS EVENTS:                                           │
│  ├── Sensitive data access (PII, financial)                    │
│  ├── Bulk data exports or downloads                            │
│  ├── Data modifications on critical records                    │
│  └── Database admin operations                                 │
│                                                                 │
│  SYSTEM EVENTS:                                                │
│  ├── Application startup and shutdown                          │
│  ├── Configuration changes                                     │
│  ├── Error conditions and exceptions                           │
│  ├── Backup and restore operations                             │
│  └── Certificate/key operations                                │
│                                                                 │
│  NEVER LOG:                                                    │
│  ├── Passwords (plain or hashed)                               │
│  ├── Credit card numbers                                       │
│  ├── Social Security Numbers                                   │
│  ├── API keys / tokens / secrets                               │
│  ├── Session IDs                                               │
│  └── Encryption keys                                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Log Entry Structure

```json
{
    "timestamp": "2024-01-15T10:30:45.123Z",
    "level": "WARN",
    "event_type": "authentication.login_failed",
    "source": "auth-service",
    "request_id": "req-abc123",
    "user_id": "user-456",
    "username": "jsmith",
    "ip_address": "203.0.113.50",
    "user_agent": "Mozilla/5.0...",
    "geo_location": "US/New York",
    "endpoint": "/api/v1/login",
    "method": "POST",
    "status_code": 401,
    "message": "Login failed - invalid credentials",
    "details": {
        "attempt_count": 3,
        "account_locked": false
    },
    "correlation_id": "corr-xyz789"
}
```

### Structured Logging Implementation

```python
import logging
import json
from datetime import datetime

class SecurityLogger:
    def __init__(self, service_name):
        self.logger = logging.getLogger('security')
        self.service = service_name
    
    def log_event(self, event_type, level, message, **kwargs):
        entry = {
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'level': level,
            'event_type': event_type,
            'source': self.service,
            'message': message,
            **kwargs
        }
        # Remove None values
        entry = {k: v for k, v in entry.items() if v is not None}
        
        log_method = getattr(self.logger, level.lower(), self.logger.info)
        log_method(json.dumps(entry))
    
    def log_auth_success(self, user_id, ip, user_agent):
        self.log_event('authentication.login_success', 'INFO',
            'User authenticated successfully',
            user_id=user_id, ip_address=ip, user_agent=user_agent)
    
    def log_auth_failure(self, username, ip, reason):
        self.log_event('authentication.login_failed', 'WARN',
            f'Login failed: {reason}',
            username=username, ip_address=ip)
    
    def log_access_denied(self, user_id, resource, ip):
        self.log_event('authorization.access_denied', 'WARN',
            f'Access denied to {resource}',
            user_id=user_id, ip_address=ip, resource=resource)
    
    def log_data_access(self, user_id, resource_type, resource_id, action):
        self.log_event('data.access', 'INFO',
            f'{action} on {resource_type}',
            user_id=user_id, resource_type=resource_type,
            resource_id=resource_id, action=action)

security_log = SecurityLogger('api-server')
```

---

## 3. Log Levels for Security

| Level | Use For | Example |
|-------|---------|---------|
| **CRITICAL** | System compromise, data breach | Deserialization RCE detected |
| **ERROR** | Security control failure | Auth service unavailable |
| **WARNING** | Suspicious activity | 5+ failed logins, SQL injection attempt |
| **INFO** | Normal security events | Successful login, password change |
| **DEBUG** | Detailed security diagnostics | Token validation steps (dev only) |

---

## 4. Log Storage Requirements

```
RETENTION:
├── Security logs: Minimum 1 year (compliance may require more)
├── Audit logs: 3-7 years (PCI DSS, SOX, HIPAA)
├── Application logs: 30-90 days
└── Debug logs: 7-14 days

PROTECTION:
├── Logs stored separately from application servers
├── Write-once / append-only storage
├── Encrypted at rest and in transit
├── Access restricted to security/ops team
├── Log integrity verification (hashing/signing)
└── Tamper detection alerting

AVAILABILITY:
├── Real-time streaming for alerting
├── Searchable within seconds (SIEM)
├── Archived for long-term compliance
└── Backup and disaster recovery
```

---

## 5. Common Logging Mistakes

```
❌ MISTAKE 1: Not logging security events at all
❌ MISTAKE 2: Logging sensitive data (passwords, tokens, PII)
❌ MISTAKE 3: Logging to local files only (lost on server compromise)
❌ MISTAKE 4: No timestamp or inconsistent timestamp format
❌ MISTAKE 5: No correlation IDs (can't trace across services)
❌ MISTAKE 6: Verbose but useless logs (disk full, no signal)
❌ MISTAKE 7: Logs not monitored (logged but never reviewed)
❌ MISTAKE 8: No log rotation (disk fills up, app crashes)
❌ MISTAKE 9: Logs accessible to application users
❌ MISTAKE 10: No integrity protection (attacker deletes logs)
```

---

## Summary Table

| Requirement | Description | Standard |
|-------------|-------------|----------|
| Completeness | Log all security events | OWASP Logging Cheat Sheet |
| Format | Structured JSON with required fields | RFC 5424, CEF, ECS |
| Protection | Encrypted, tamper-proof, restricted access | PCI DSS 10.5 |
| Retention | 1-7 years depending on compliance | PCI DSS, HIPAA, SOX |
| Monitoring | Real-time alerting on critical events | NIST 800-92 |

---

## Revision Questions

1. What security events must be logged? What must NEVER be logged?
2. Design a structured log entry format for security events with all required fields.
3. Implement a SecurityLogger class that handles auth, authorization, and data access events.
4. What are the log storage requirements for PCI DSS compliance?
5. List ten common logging mistakes and how to avoid each.
6. How do you protect logs from tampering by an attacker who has compromised the application server?

---

*Previous: None (First topic in this unit) | Next: [02-audit-trails.md](02-audit-trails.md)*

---

*[Back to README](../README.md)*
