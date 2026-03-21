# Unit 4: A04 - Insecure Design — Topic 6: Security Requirements

## Overview

Security requirements define **what the application must do to be secure**. They are functional and non-functional requirements that specify authentication, authorization, data protection, logging, and other security controls. Without explicit security requirements, developers build features without security controls — leading to insecure design. This topic covers how to elicit, document, and verify security requirements systematically.

---

## 1. Types of Security Requirements

```
┌─────────────────────────────────────────────────────────────────┐
│                SECURITY REQUIREMENTS TAXONOMY                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  FUNCTIONAL SECURITY REQUIREMENTS:                             │
│  ├── Authentication: "System SHALL support MFA"                │
│  ├── Authorization: "Users SHALL only access own data"         │
│  ├── Input Validation: "All inputs SHALL be validated"         │
│  ├── Session Management: "Sessions SHALL expire after 30min"   │
│  ├── Error Handling: "Errors SHALL NOT expose internal details"│
│  └── Audit Logging: "All auth events SHALL be logged"          │
│                                                                 │
│  NON-FUNCTIONAL SECURITY REQUIREMENTS:                         │
│  ├── Performance: "Auth check SHALL complete in <100ms"        │
│  ├── Availability: "Auth service SHALL have 99.9% uptime"     │
│  ├── Compliance: "System SHALL meet PCI DSS requirements"      │
│  ├── Encryption: "All data SHALL be encrypted with AES-256"    │
│  └── Recovery: "System SHALL recover from failure in <1hr"     │
│                                                                 │
│  ABUSE CASE REQUIREMENTS:                                      │
│  ├── "System SHALL resist brute force attacks"                 │
│  ├── "System SHALL prevent automated account creation"         │
│  ├── "System SHALL detect and block credential stuffing"       │
│  └── "System SHALL prevent unauthorized bulk data export"      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. OWASP ASVS (Application Security Verification Standard)

```
ASVS provides a comprehensive security requirements framework:

Level 1 — OPPORTUNISTIC (minimum for all applications)
  → Basic security controls
  → Can be verified with automated tools
  → ~130 requirements

Level 2 — STANDARD (for most applications)
  → Defense against most common attacks
  → Requires manual testing
  → ~270 requirements

Level 3 — ADVANCED (for critical applications)
  → Financial systems, healthcare, military
  → Deep security analysis required
  → ~310 requirements

ASVS Categories:
V1:  Architecture, Design, Threat Modeling
V2:  Authentication
V3:  Session Management
V4:  Access Control
V5:  Validation, Sanitization, Encoding
V6:  Stored Cryptography
V7:  Error Handling and Logging
V8:  Data Protection
V9:  Communication
V10: Malicious Code
V11: Business Logic
V12: Files and Resources
V13: API and Web Service
V14: Configuration
```

### ASVS Requirement Examples

| ID | Requirement | Level |
|----|-------------|-------|
| V2.1.1 | User-set passwords SHALL be at least 8 characters | L1 |
| V2.2.1 | Anti-automation controls SHALL be effective against credential stuffing | L1 |
| V2.8.1 | Time-based OTP SHALL have a lifetime of 120 seconds or less | L2 |
| V3.2.1 | Session tokens SHALL be generated using approved cryptographic algorithms | L1 |
| V3.3.1 | Logout and expiration SHALL invalidate the session token | L1 |
| V4.1.1 | Application SHALL enforce access control at a trusted service layer | L1 |
| V5.1.1 | Application SHALL have defenses against HTTP parameter pollution | L1 |
| V6.2.1 | All cryptographic modules SHALL fail securely | L1 |
| V7.1.1 | Application SHALL NOT log credentials or payment details | L1 |
| V8.3.1 | Sensitive data SHALL be sent in the HTTP body or headers | L1 |

---

## 3. Security Requirements per Feature

```
For EVERY feature, define these security requirements:

┌──────────────────────────────────────────────────────────────────┐
│        FEATURE: User Registration                               │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  AUTHENTICATION:                                                │
│  SR-001: Passwords MUST be >= 8 characters                      │
│  SR-002: Passwords MUST be hashed with bcrypt (cost >= 12)      │
│  SR-003: Email verification MUST be required                    │
│                                                                  │
│  INPUT VALIDATION:                                              │
│  SR-004: Email MUST match RFC 5322 format                       │
│  SR-005: Username MUST be 3-30 alphanumeric characters          │
│  SR-006: All fields MUST have maximum length limits             │
│                                                                  │
│  RATE LIMITING:                                                 │
│  SR-007: Registration MUST be limited to 3/hour per IP          │
│  SR-008: CAPTCHA MUST be required after first attempt           │
│                                                                  │
│  ABUSE PREVENTION:                                              │
│  SR-009: System MUST prevent duplicate email registration       │
│  SR-010: Response MUST NOT reveal if email already exists       │
│  SR-011: System MUST detect and block automated signups         │
│                                                                  │
│  LOGGING:                                                       │
│  SR-012: All registrations MUST be logged with IP and timestamp │
│  SR-013: Failed registrations MUST trigger alert if > threshold │
│                                                                  │
│  DATA PROTECTION:                                               │
│  SR-014: Registration data MUST be transmitted over TLS 1.2+    │
│  SR-015: PII MUST be encrypted at rest                          │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4. Abuse Cases and Misuse Cases

```
Traditional Use Case:
"As a user, I want to reset my password so I can regain account access"

Corresponding Abuse Cases:
┌─────────────────────────────────────────────────────────────────┐
│  ABUSE CASE 1: Account Takeover via Password Reset             │
│  Attacker resets another user's password using their email     │
│  Mitigation: Multi-step verification, secure token, expiry    │
├─────────────────────────────────────────────────────────────────┤
│  ABUSE CASE 2: User Enumeration                               │
│  Attacker discovers valid accounts via reset form responses    │
│  Mitigation: Generic "If account exists, email sent" message  │
├─────────────────────────────────────────────────────────────────┤
│  ABUSE CASE 3: Token Brute Force                              │
│  Attacker guesses reset tokens                                │
│  Mitigation: Long random tokens, short expiry, rate limit     │
├─────────────────────────────────────────────────────────────────┤
│  ABUSE CASE 4: Denial of Service                              │
│  Attacker floods reset requests to lock out users             │
│  Mitigation: Rate limit per IP, don't lock account on reset   │
├─────────────────────────────────────────────────────────────────┤
│  ABUSE CASE 5: Email Bomb                                     │
│  Attacker triggers thousands of reset emails to a victim      │
│  Mitigation: Rate limit per email address, cooldown period    │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Compliance-Driven Security Requirements

| Standard | Key Security Requirements |
|----------|--------------------------|
| **PCI DSS** | Encrypt cardholder data, strong access controls, logging, network segmentation, quarterly scans |
| **HIPAA** | Access controls, audit controls, encryption, integrity controls, transmission security |
| **GDPR** | Data minimization, consent management, right to erasure, breach notification (72hr), privacy by design |
| **SOC 2** | Security, availability, processing integrity, confidentiality, privacy controls |
| **NIST 800-53** | 1000+ controls across 20 families (AC, AU, CM, IA, IR, etc.) |

---

## 6. Verifying Security Requirements

```
┌─────────────────────────────────────────────────────────────────┐
│          VERIFICATION METHODS                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  AUTOMATED TESTING:                                            │
│  ├── Unit tests for security functions                         │
│  ├── Integration tests for auth flows                          │
│  ├── SAST scanning (Semgrep, SonarQube)                       │
│  ├── DAST scanning (ZAP, Burp)                                │
│  └── Dependency scanning (Snyk, Dependabot)                   │
│                                                                 │
│  MANUAL TESTING:                                               │
│  ├── Penetration testing                                       │
│  ├── Code review                                               │
│  ├── Architecture review                                       │
│  └── Threat model validation                                   │
│                                                                 │
│  CONTINUOUS MONITORING:                                        │
│  ├── Runtime security monitoring (RASP)                        │
│  ├── Log analysis and alerting                                 │
│  ├── Vulnerability scanning                                    │
│  └── Configuration drift detection                             │
└─────────────────────────────────────────────────────────────────┘
```

### Security Test Examples

```python
# Security requirement: SR-001 — Passwords >= 8 characters
def test_password_minimum_length():
    response = client.post('/register', json={
        'email': 'test@example.com',
        'password': 'short'  # 5 chars
    })
    assert response.status_code == 400
    assert 'password' in response.json['errors']

# Security requirement: SR-007 — Rate limit 3/hour per IP
def test_registration_rate_limit():
    for i in range(3):
        resp = client.post('/register', json={...})
        assert resp.status_code in [200, 201]
    
    resp = client.post('/register', json={...})
    assert resp.status_code == 429  # Too Many Requests

# Security requirement: SR-010 — Don't reveal if email exists
def test_no_email_enumeration():
    resp1 = client.post('/password-reset', json={'email': 'exists@test.com'})
    resp2 = client.post('/password-reset', json={'email': 'nonexist@test.com'})
    assert resp1.json['message'] == resp2.json['message']
    assert resp1.status_code == resp2.status_code
```

---

## 7. Security Requirements Traceability Matrix

```
┌──────────┬────────────────────────────┬───────────┬──────────┬──────────┐
│ Req ID   │ Requirement                │ Feature   │ Test     │ Status   │
├──────────┼────────────────────────────┼───────────┼──────────┼──────────┤
│ SR-001   │ Password >= 8 chars        │ Register  │ T-001    │ ✅ Pass  │
│ SR-002   │ bcrypt cost >= 12          │ Register  │ T-002    │ ✅ Pass  │
│ SR-003   │ Email verification         │ Register  │ T-003    │ ✅ Pass  │
│ SR-007   │ Rate limit 3/hr/IP         │ Register  │ T-007    │ ⚠️ Fail  │
│ SR-010   │ No email enumeration       │ PW Reset  │ T-010    │ ✅ Pass  │
│ SR-014   │ TLS 1.2+ required          │ All       │ T-014    │ ✅ Pass  │
│ SR-015   │ PII encrypted at rest      │ All       │ T-015    │ ❌ N/A   │
└──────────┴────────────────────────────┴───────────┴──────────┴──────────┘

Every requirement has:
- Unique ID for tracking
- Clear, testable statement
- Mapped to feature
- Mapped to test case
- Current verification status
```

---

## Summary Table

| Framework | Purpose | Use When |
|-----------|---------|----------|
| **OWASP ASVS** | Comprehensive app security checklist | Any web application |
| **Abuse Cases** | Identify attacker perspectives | Feature design |
| **PCI DSS** | Payment card security | Processing payments |
| **HIPAA** | Healthcare data protection | Handling PHI |
| **GDPR** | Privacy requirements | EU user data |
| **NIST 800-53** | Government security controls | Federal/regulated systems |

---

## Revision Questions

1. What is the difference between functional and non-functional security requirements? Give three examples of each.
2. Explain OWASP ASVS levels 1, 2, and 3. What type of application needs each level?
3. Write five security requirements for a "file upload" feature, covering authentication, validation, and abuse prevention.
4. What are abuse cases and how do they differ from traditional use cases? Create abuse cases for a "money transfer" feature.
5. How do you verify that security requirements are actually implemented? List automated and manual methods.
6. What is a Security Requirements Traceability Matrix and why is it important?

---

*Previous: [05-defense-in-depth.md](05-defense-in-depth.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
