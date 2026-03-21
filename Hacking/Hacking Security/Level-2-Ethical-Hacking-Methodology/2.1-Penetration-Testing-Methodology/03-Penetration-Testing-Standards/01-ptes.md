# PTES (Penetration Testing Execution Standard)

## Unit 3 - Topic 1 | Penetration Testing Standards

---

## Overview

**PTES** is the most comprehensive open standard for penetration testing. It defines a structured methodology with **7 phases** covering the entire engagement lifecycle — from pre-engagement through reporting. PTES provides both high-level guidance and detailed technical procedures.

---

## 1. PTES Seven Phases

```
┌──────────────────────────────────────────────────────────────┐
│              PTES METHODOLOGY — 7 PHASES                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────┐                                        │
│  │ 1. Pre-engagement│                                       │
│  │    Interactions   │                                       │
│  └────────┬─────────┘                                       │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │ 2. Intelligence  │                                       │
│  │    Gathering      │                                       │
│  └────────┬─────────┘                                       │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │ 3. Threat        │                                       │
│  │    Modeling       │                                       │
│  └────────┬─────────┘                                       │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │ 4. Vulnerability │                                       │
│  │    Analysis       │                                       │
│  └────────┬─────────┘                                       │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │ 5. Exploitation  │                                       │
│  └────────┬─────────┘                                       │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │ 6. Post-         │                                       │
│  │    Exploitation   │                                       │
│  └────────┬─────────┘                                       │
│           ▼                                                  │
│  ┌─────────────────┐                                        │
│  │ 7. Reporting     │                                       │
│  └─────────────────┘                                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Phase Details

| Phase | Activities | Output |
|-------|-----------|--------|
| **1. Pre-engagement** | Scope, RoE, authorization, contracts | Signed agreements |
| **2. Intelligence Gathering** | OSINT, DNS, WHOIS, social media | Target profile |
| **3. Threat Modeling** | Identify assets, threats, attack vectors | Threat model |
| **4. Vulnerability Analysis** | Scanning, manual testing, validation | Vulnerability list |
| **5. Exploitation** | Exploit vulnerabilities, gain access | Proof of access |
| **6. Post-Exploitation** | Pivot, escalate, assess business impact | Impact assessment |
| **7. Reporting** | Document findings, recommendations | Final report |

---

## 3. Phase 1: Pre-engagement Interactions

```
PRE-ENGAGEMENT CHECKLIST (PTES):
├── Scope Definition
│   ├── IP ranges and domains
│   ├── Applications and services
│   └── Out-of-scope systems
├── Rules of Engagement
│   ├── Testing hours
│   ├── Allowed methods
│   └── Stop conditions
├── Authorization
│   ├── Written permission
│   └── Third-party notifications
├── Emergency Contacts
├── Communication Channels
├── Timeline and Milestones
└── Payment Terms
```

---

## 4. Phase 2-4: Information to Exploitation

```
INTELLIGENCE GATHERING:
├── Passive: WHOIS, DNS, Google dorks, social media
├── Active: Port scanning, service enumeration
└── Output: Target map, technology stack

THREAT MODELING:
├── Identify high-value assets
├── Map attack vectors
├── Prioritize targets by impact
└── Output: Attack plan

VULNERABILITY ANALYSIS:
├── Automated scanning (Nessus, OpenVAS)
├── Manual testing and validation
├── False positive elimination
└── Output: Verified vulnerability list
```

---

## 5. Phase 5-7: Exploitation to Reporting

```
EXPLOITATION:
├── Select exploits based on vulnerabilities
├── Test in controlled manner
├── Document each exploitation attempt
├── Minimize impact on target systems
└── Output: Proof of exploitation

POST-EXPLOITATION:
├── Privilege escalation
├── Lateral movement
├── Data access assessment
├── Business impact analysis
└── Output: Impact evidence

REPORTING:
├── Executive Summary (non-technical)
├── Technical Details (for IT team)
├── Risk Ratings (CVSS scores)
├── Remediation Recommendations
├── Evidence (screenshots, logs)
└── Output: Final report
```

---

## Summary Table

| Phase | Focus |
|-------|-------|
| **Pre-engagement** | Scope, legal, contracts |
| **Intelligence** | Research the target |
| **Threat Modeling** | Plan the attack |
| **Vuln Analysis** | Find weaknesses |
| **Exploitation** | Prove the vulnerabilities |
| **Post-Exploitation** | Assess real impact |
| **Reporting** | Document everything |

---

## Quick Revision Questions

1. **How many phases does PTES define?**
2. **What happens in the threat modeling phase?**
3. **Why is pre-engagement the most important phase legally?**
4. **What is the difference between vulnerability analysis and exploitation?**
5. **What should the final report include?**

---

[Next: OWASP Testing Guide →](02-owasp-testing-guide.md)

---

*Unit 3 - Topic 1 of 5 | [Back to README](../README.md)*
