# Unit 4: A04 - Insecure Design — Topic 5: Defense in Depth

## Overview

Defense in Depth is a security strategy that deploys **multiple layers of security controls** throughout an application and its infrastructure. The principle is simple: no single security mechanism is perfect, so layering multiple controls ensures that if one layer is breached, subsequent layers still protect the system. This military-derived concept is the cornerstone of enterprise security architecture.

---

## 1. Defense in Depth Model

```
┌─────────────────────────────────────────────────────────────────┐
│                   DEFENSE IN DEPTH LAYERS                       │
│                   (Outside → Inside)                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Layer 1: PHYSICAL SECURITY                                    │
│  ├── Data center access controls                               │
│  ├── Surveillance cameras                                      │
│  └── Hardware security modules (HSM)                           │
│                                                                 │
│  Layer 2: NETWORK SECURITY                                     │
│  ├── Firewalls (perimeter + internal)                          │
│  ├── IDS/IPS (Intrusion Detection/Prevention)                  │
│  ├── Network segmentation / VLANs                              │
│  ├── DDoS protection                                           │
│  └── VPN for remote access                                     │
│                                                                 │
│  Layer 3: HOST SECURITY                                        │
│  ├── OS hardening and patching                                 │
│  ├── Endpoint Detection and Response (EDR)                     │
│  ├── Host-based firewall                                       │
│  ├── Anti-malware                                              │
│  └── File integrity monitoring                                 │
│                                                                 │
│  Layer 4: APPLICATION SECURITY                                 │
│  ├── WAF (Web Application Firewall)                            │
│  ├── Input validation and output encoding                      │
│  ├── Authentication and authorization                          │
│  ├── Session management                                        │
│  ├── RASP (Runtime Application Self-Protection)                │
│  └── Security headers (CSP, HSTS, X-Frame-Options)            │
│                                                                 │
│  Layer 5: DATA SECURITY                                        │
│  ├── Encryption at rest (AES-256)                              │
│  ├── Encryption in transit (TLS 1.3)                           │
│  ├── Data masking and tokenization                             │
│  ├── Database access controls                                  │
│  ├── Backup encryption                                         │
│  └── Data Loss Prevention (DLP)                                │
│                                                                 │
│  Layer 6: MONITORING & RESPONSE                                │
│  ├── SIEM (Security Information & Event Management)            │
│  ├── Log aggregation and analysis                              │
│  ├── Alerting and incident response                            │
│  ├── Threat hunting                                            │
│  └── Forensics capability                                      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Web Application Defense in Depth

```
Request Flow Through Security Layers:

[Attacker] → [CDN/DDoS] → [WAF] → [Load Balancer] → [App Server] → [Database]
                 │            │           │                │             │
                 ▼            ▼           ▼                ▼             ▼
              GeoBlock    SQL/XSS     TLS Term         Input          ACLs
              Rate Limit  Rules       Health Check     Validation     Encryption
              Bot Detect  IP Block    Session          AuthN/AuthZ    Audit Log
                                      Sticky           CSP/HSTS       Least Priv

If WAF misses an attack → Input validation catches it
If input validation fails → Parameterized queries prevent SQLi
If query is compromised → Least privilege limits damage
If data is accessed → Encryption protects confidentiality
If breach occurs → Monitoring detects and alerts
```

### Practical Implementation

```python
# Layer 1: WAF rules (before application)
# ModSecurity, AWS WAF, Cloudflare WAF
# Blocks known attack patterns

# Layer 2: Rate limiting
from flask_limiter import Limiter
limiter = Limiter(app, default_limits=["100/minute"])

@app.route('/login', methods=['POST'])
@limiter.limit("5/minute")  # Brute force protection
def login():
    pass

# Layer 3: Input validation
from marshmallow import Schema, fields, validate

class LoginSchema(Schema):
    username = fields.Str(required=True, validate=validate.Length(min=3, max=50))
    password = fields.Str(required=True, validate=validate.Length(min=8, max=128))

# Layer 4: Authentication
@app.route('/api/data')
@require_authentication
@require_permission('read:data')
def get_data():
    pass

# Layer 5: Parameterized queries (SQLi prevention)
cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))

# Layer 6: Encryption at rest
# Database-level encryption (TDE)
# Application-level field encryption for PII

# Layer 7: Monitoring
import logging
security_logger = logging.getLogger('security')
security_logger.info(f"Data access: user={user_id}, resource={resource_id}")
```

---

## 3. Security Headers as a Defense Layer

```
HTTP/1.1 200 OK
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 0
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
Cache-Control: no-store, no-cache, must-revalidate

Each header is an independent defense:
- CSP → Prevents XSS even if vulnerable code exists
- HSTS → Prevents downgrade attacks even if HTTP is open
- X-Frame-Options → Prevents clickjacking even without JS protection
```

---

## 4. Defense in Depth for Common Attacks

### SQL Injection Defense Layers

```
Layer 1: WAF blocks known SQL patterns
Layer 2: Input validation rejects special characters
Layer 3: Parameterized queries prevent interpretation
Layer 4: Database user has read-only access (least privilege)
Layer 5: Database encryption protects data if dumped
Layer 6: Monitoring detects unusual query patterns

Each layer independently prevents or limits the attack.
Even if layers 1-2 fail, layer 3 (parameterized queries) stops SQLi.
Even if layer 3 fails, layer 4 limits what attacker can do.
```

### XSS Defense Layers

```
Layer 1: WAF blocks common XSS payloads
Layer 2: Input validation strips/rejects HTML/JS
Layer 3: Output encoding prevents browser interpretation
Layer 4: CSP restricts script execution sources
Layer 5: HttpOnly cookies prevent session theft via XSS
Layer 6: SRI (Subresource Integrity) prevents script tampering
```

### Authentication Attack Defense Layers

```
Layer 1: Rate limiting on login endpoint
Layer 2: CAPTCHA after failed attempts
Layer 3: Account lockout after N failures
Layer 4: MFA required for all users
Layer 5: Password strength requirements
Layer 6: Credential breach monitoring (HaveIBeenPwned)
Layer 7: Session timeout and management
Layer 8: Anomaly detection (unusual login location/time)
```

---

## 5. Zero Trust Architecture

```
Traditional: "Trust internal network, verify external"
Zero Trust:  "Never trust, always verify"

┌─────────────────────────────────────────────────────────────────┐
│                    ZERO TRUST PRINCIPLES                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. Verify explicitly                                          │
│     → Authenticate and authorize every request                 │
│     → Based on all available data points                       │
│                                                                 │
│  2. Use least privilege access                                 │
│     → Just-in-time (JIT) access                               │
│     → Just-enough-access (JEA)                                 │
│     → Time-limited access                                      │
│                                                                 │
│  3. Assume breach                                              │
│     → Segment access by network, user, device                  │
│     → Verify end-to-end encryption                             │
│     → Use analytics for threat detection                       │
│                                                                 │
│  Traditional:                                                  │
│  [Firewall] → Internal network = trusted                       │
│                                                                 │
│  Zero Trust:                                                   │
│  Every service authenticates every request to every service    │
│  [Service A] ←mTLS→ [Service B] ←mTLS→ [Service C]           │
│       ↕                  ↕                  ↕                  │
│  [Identity Provider / Policy Engine]                           │
└─────────────────────────────────────────────────────────────────┘
```

---

## 6. Monitoring as the Final Layer

```
Even with all preventive controls, assume some attacks will succeed.
Detection and response is the final layer.

Detection Capabilities:
┌──────────────────────┬──────────────────────────────────────────┐
│ What to Monitor      │ Alert Trigger                           │
├──────────────────────┼──────────────────────────────────────────┤
│ Authentication       │ >5 failed logins, login from new country│
│ Authorization        │ Access denied patterns, privilege use   │
│ Data access          │ Bulk exports, unusual queries           │
│ API usage            │ Rate spikes, new API keys used          │
│ File integrity       │ Config/binary changes                   │
│ Network traffic      │ New outbound connections, data exfil    │
│ Application errors   │ Spike in 500 errors, stack traces       │
│ User behavior        │ Off-hours access, impossible travel     │
└──────────────────────┴──────────────────────────────────────────┘
```

---

## Summary Table

| Layer | Controls | Attacks Mitigated |
|-------|----------|-------------------|
| Network | Firewall, IDS/IPS, segmentation | Network attacks, lateral movement |
| Host | Hardening, EDR, patching | Exploitation, malware |
| Application | WAF, validation, auth, CSP | Injection, XSS, broken auth |
| Data | Encryption, masking, ACLs | Data breach, unauthorized access |
| Monitoring | SIEM, alerting, IR | All (detection and response) |

---

## Revision Questions

1. Explain Defense in Depth using a real-world analogy (e.g., castle security).
2. List all six layers of Defense in Depth and give two controls for each layer.
3. How does Defense in Depth apply specifically to preventing SQL injection? Walk through each layer.
4. What is Zero Trust architecture and how does it differ from traditional perimeter security?
5. Why is monitoring considered the "last line of defense" and what should be monitored?
6. Design a Defense in Depth strategy for a healthcare web application handling patient data.

---

*Previous: [04-attack-surface-reduction.md](04-attack-surface-reduction.md) | Next: [06-security-requirements.md](06-security-requirements.md)*

---

*[Back to README](../README.md)*
