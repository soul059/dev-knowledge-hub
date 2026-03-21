# Rules of Engagement Document

## Unit 4 - Topic 2 | Pre-engagement

---

## Overview

The **Rules of Engagement (RoE) document** is the formal, written agreement that defines the operational boundaries of a penetration test. It is more detailed than the scope and specifies **exactly how testing will be conducted**, including methods, schedules, escalation procedures, and data handling.

---

## 1. RoE Document Structure

```
RULES OF ENGAGEMENT DOCUMENT:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

SECTION 1: ENGAGEMENT OVERVIEW
├── Project name and ID
├── Client organization
├── Testing firm
├── Test type and approach
└── Date of agreement

SECTION 2: SCOPE
├── In-scope targets (IPs, domains, apps)
├── Out-of-scope targets
├── Third-party systems and restrictions
└── Scope change procedure

SECTION 3: AUTHORIZED ACTIVITIES
├── Permitted testing methods
├── Prohibited testing methods
├── Exploitation depth limits
└── Data access boundaries

SECTION 4: SCHEDULE
├── Testing window (dates and times)
├── Blackout periods
├── Milestone dates
└── Report delivery date

SECTION 5: COMMUNICATION
├── Points of contact (primary/secondary)
├── Status update schedule
├── Critical finding notification
├── Escalation procedures
└── Encrypted communication channels

SECTION 6: DATA HANDLING
├── Evidence collection procedures
├── Data encryption requirements
├── Data retention policy
├── Data destruction timeline

SECTION 7: INCIDENT PROCEDURES
├── Stop conditions
├── Emergency contacts
├── Recovery procedures
└── Incident documentation

SECTION 8: SIGNATURES
├── Client authorized representative
└── Testing team lead

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 2. Authorized vs Prohibited Activities

| Category | ✅ Typically Authorized | ❌ Typically Prohibited |
|----------|----------------------|----------------------|
| **Scanning** | Port scan, vuln scan | DoS/DDoS testing |
| **Exploitation** | Known CVEs, misconfigs | Zero-day usage |
| **Social Engineering** | Phishing (if in scope) | Threats, blackmail |
| **Physical** | Tailgating (if agreed) | Lock picking (varies) |
| **Data** | View sample data | Mass data exfiltration |
| **Accounts** | Use provided creds | Create admin accounts |
| **Cleanup** | Remove test artifacts | Leave backdoors |

---

## 3. Critical Finding Protocol

```
CRITICAL FINDING ESCALATION:

Discovery of critical/exploitable vulnerability
        │
        ▼
Document the finding immediately
        │
        ▼
Contact client within 1 HOUR
├── Phone: Primary contact
├── If no answer: Secondary contact
└── If no answer: Email + SMS
        │
        ▼
Provide preliminary details
├── What was found
├── Where (system/application)
├── Severity estimate
└── Immediate risk assessment
        │
        ▼
Client decides on response
├── Continue testing
├── Pause and remediate
└── Accept risk and continue
        │
        ▼
Document the communication
```

---

## 4. Scope Change Procedure

```
WHEN SCOPE CHANGES ARE NEEDED:

Scenario: Tester discovers additional systems
         connected to in-scope targets

PROCEDURE:
1. STOP testing the new system
2. Document the discovery
3. Contact client POC
4. Request scope expansion
5. Get WRITTEN authorization
6. Update RoE document
7. Resume testing

⚠️ NEVER expand scope without written approval
⚠️ Even if the system is clearly vulnerable
⚠️ Document the request and response
```

---

## 5. Data Handling Requirements

| Requirement | Detail |
|-------------|--------|
| **Encryption** | AES-256 for stored data, TLS 1.2+ in transit |
| **Storage** | Encrypted drives, access-controlled systems |
| **Sharing** | Only with authorized team members |
| **Retention** | 30-90 days after report delivery (typical) |
| **Destruction** | Secure wipe (DoD 5220.22-M or equivalent) |
| **Incident** | Notify client within 24h if data is compromised |

---

## Summary Table

| Section | Key Point |
|---------|-----------|
| **Scope** | What can and cannot be tested |
| **Methods** | Allowed and prohibited techniques |
| **Schedule** | Testing windows and blackout periods |
| **Communication** | Contacts, updates, critical findings |
| **Data handling** | Encrypt, retain, destroy |
| **Signatures** | Both parties must sign |

---

## Quick Revision Questions

1. **What is the purpose of the RoE document?**
2. **What should you do if you discover a system not in scope?**
3. **How quickly should critical findings be reported?**
4. **What data handling requirements are typical?**
5. **Who should sign the RoE document?**

---

[← Previous: Scoping Discussions](01-scoping-discussions.md) | [Next: Authorization Forms →](03-authorization-forms.md)

---

*Unit 4 - Topic 2 of 6 | [Back to README](../README.md)*
