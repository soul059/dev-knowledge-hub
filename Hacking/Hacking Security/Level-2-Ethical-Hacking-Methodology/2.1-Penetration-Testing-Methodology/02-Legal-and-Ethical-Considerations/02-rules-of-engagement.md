# Rules of Engagement

## Unit 2 - Topic 2 | Legal and Ethical Considerations

---

## Overview

**Rules of Engagement (RoE)** define the detailed operational guidelines for a penetration test. They specify **what**, **when**, **how**, and **how far** testing can go. RoE protect both the tester and the client by establishing clear boundaries and expectations.

---

## 1. RoE Components

```
┌──────────────────────────────────────────────────────────────┐
│              RULES OF ENGAGEMENT STRUCTURE                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. SCOPE DEFINITION                                         │
│     What systems/networks/apps are in scope                  │
│                                                              │
│  2. TESTING METHODS                                          │
│     Allowed and prohibited techniques                        │
│                                                              │
│  3. TESTING SCHEDULE                                         │
│     When testing can occur                                   │
│                                                              │
│  4. COMMUNICATION PLAN                                       │
│     How to report issues, escalation procedures              │
│                                                              │
│  5. HANDLING SENSITIVE DATA                                   │
│     What to do if PII/secrets are found                     │
│                                                              │
│  6. STOP CONDITIONS                                          │
│     When to immediately stop testing                         │
│                                                              │
│  7. EVIDENCE HANDLING                                        │
│     How to secure and dispose of evidence                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Key RoE Decisions

| Decision | Options |
|----------|---------|
| **Social engineering** | Allowed / Not allowed / Limited |
| **DoS testing** | Allowed / Not allowed / Simulated only |
| **Physical testing** | Allowed / Not allowed / Specific locations |
| **Data exfiltration** | Demonstrate / Simulate / Not allowed |
| **Exploitation depth** | Find only / Exploit / Full post-exploitation |
| **Account creation** | Allowed / Not allowed |
| **Credential use** | Provided creds only / Can crack / Can phish |
| **Working hours** | Business hours / After hours / Weekend |

---

## 3. Stop Conditions

```
IMMEDIATELY STOP TESTING IF:
├── 🛑 System becomes unresponsive or crashes
├── 🛑 Data integrity is at risk
├── 🛑 Active security incident detected (real attacker!)
├── 🛑 Client requests stop
├── 🛑 Testing causes unexpected business impact
├── 🛑 Legal issue arises
├── 🛑 Scope boundary is accidentally crossed
└── 🛑 Critical vulnerability found that poses immediate danger

UPON STOPPING:
1. Document what happened
2. Notify emergency contact immediately
3. Preserve evidence
4. Wait for authorization to continue
```

---

## 4. Sensitive Data Handling

```
IF YOU FIND SENSITIVE DATA DURING TESTING:

PII (Personally Identifiable Information):
├── Stop accessing data immediately
├── Note the location and type of data
├── Do NOT copy/store the actual data
├── Report finding with sanitized examples
└── "Found unencrypted SSNs in database X"

CREDENTIALS:
├── Note that credentials were accessible
├── Test 1-2 to prove access (with permission)
├── Do NOT dump entire credential database
├── Report with redacted examples
└── "admin:P****** grants access to..."

ILLEGAL CONTENT:
├── STOP immediately
├── Do NOT investigate further
├── Notify client immediately
├── May need to notify law enforcement
└── Document time, location, and what was observed
```

---

## 5. RoE Template Structure

```
RULES OF ENGAGEMENT DOCUMENT:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. PROJECT OVERVIEW
   - Project name, client, tester(s)
   - Test type (black/gray/white box)
   
2. SCOPE
   - In-scope targets (IPs, URLs, apps)
   - Out-of-scope targets
   - Third-party systems

3. AUTHORIZED ACTIVITIES
   - Network scanning ✅
   - Vulnerability scanning ✅
   - Exploitation ✅
   - Social engineering ❌
   - DoS testing ❌

4. SCHEDULE
   - Testing window: Mon-Fri, 10 PM - 4 AM
   - Start: 2024-01-15
   - End: 2024-02-15
   
5. COMMUNICATION
   - Primary contact: [Name, Phone, Email]
   - Emergency contact: [Name, Phone]
   - Status updates: Daily via encrypted email
   - Critical findings: Immediate phone call

6. DATA HANDLING
   - Encrypt all findings (AES-256)
   - Delete test data within 30 days
   - No client data on personal devices
   
7. STOP CONDITIONS
   - [As listed above]
   
8. SIGNATURES
   - Client authorized representative
   - Lead penetration tester
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **RoE** | Operational guidelines for the engagement |
| **Scope** | Defined targets, methods, schedule |
| **Stop conditions** | When to immediately halt testing |
| **Data handling** | Encrypt, minimize, delete after engagement |
| **Communication** | Regular updates + immediate critical alerts |
| **Signatures** | Both parties must sign |

---

## Quick Revision Questions

1. **What are Rules of Engagement?**
2. **Name 3 stop conditions for a pen test.**
3. **What should you do if you find PII during testing?**
4. **Why is a communication plan important?**
5. **What happens if you accidentally go out of scope?**

---

[← Previous: Authorization and Scope](01-authorization-and-scope.md) | [Next: Legal Frameworks →](03-legal-frameworks.md)

---

*Unit 2 - Topic 2 of 6 | [Back to README](../README.md)*
