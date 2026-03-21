# CIS Controls

## Unit 4 - Topic 3 | Security Frameworks

---

## Overview

The **CIS (Center for Internet Security) Controls** are a prioritized set of best practices designed to provide actionable ways to stop the most prevalent and dangerous cyberattacks. They are prescriptive, practical, and ranked by priority — making them one of the most implementable security frameworks.

---

## 1. What Makes CIS Controls Different

```
┌──────────────────────────────────────────────────────────────┐
│              CIS CONTROLS vs OTHER FRAMEWORKS                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  NIST CSF:      "What outcomes to achieve" (high-level)      │
│  ISO 27001:     "What management system to have" (process)   │
│  CIS Controls:  "What to DO FIRST, SECOND, THIRD" (tactical) │
│                                                              │
│  CIS is the most ACTIONABLE and PRIORITIZED framework.       │
│  Start with Control 1 and work your way up.                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. CIS Controls v8 (18 Controls)

| # | Control | Category |
|---|---------|----------|
| 1 | Inventory and Control of Enterprise Assets | **Basic** |
| 2 | Inventory and Control of Software Assets | **Basic** |
| 3 | Data Protection | **Basic** |
| 4 | Secure Configuration of Enterprise Assets and Software | **Basic** |
| 5 | Account Management | **Basic** |
| 6 | Access Control Management | **Basic** |
| 7 | Continuous Vulnerability Management | **Foundational** |
| 8 | Audit Log Management | **Foundational** |
| 9 | Email and Web Browser Protections | **Foundational** |
| 10 | Malware Defenses | **Foundational** |
| 11 | Data Recovery | **Foundational** |
| 12 | Network Infrastructure Management | **Foundational** |
| 13 | Network Monitoring and Defense | **Foundational** |
| 14 | Security Awareness and Skills Training | **Organizational** |
| 15 | Service Provider Management | **Organizational** |
| 16 | Application Software Security | **Organizational** |
| 17 | Incident Response Management | **Organizational** |
| 18 | Penetration Testing | **Organizational** |

---

## 3. Implementation Groups (IGs)

CIS Controls are organized into three Implementation Groups based on organization size and resources:

| IG | Description | Target Organization |
|----|-------------|-------------------|
| **IG1** | Essential Cyber Hygiene (56 safeguards) | Small orgs, limited IT staff |
| **IG2** | Intermediate (130 safeguards) | Mid-size orgs with IT teams |
| **IG3** | Advanced (153 safeguards) | Large orgs, dedicated security team |

```
IG1 (Essential)  ⊂  IG2 (Intermediate)  ⊂  IG3 (Advanced)

IG1: "Do THESE things at minimum — they stop most attacks."
IG2: "Add THESE for better coverage against targeted attacks."
IG3: "Full implementation for advanced threat defense."
```

---

## 4. Top Priority Controls (IG1 Essentials)

| Control | Why It's Essential |
|---------|-------------------|
| **Asset Inventory** | Can't protect what you don't know you have |
| **Software Inventory** | Unauthorized software = major attack vector |
| **Data Protection** | Know where sensitive data lives |
| **Secure Configuration** | Defaults are often insecure |
| **Account Management** | Disable unused accounts, enforce MFA |
| **Access Control** | Least privilege, role-based access |

---

## 5. Mapping to Other Frameworks

| CIS Control | NIST CSF Function | ISO 27001 Control |
|-------------|-------------------|-------------------|
| Asset Inventory | Identify | A.5.9 - Inventory |
| Secure Config | Protect | A.8.9 - Configuration management |
| Vulnerability Mgmt | Detect | A.8.8 - Technical vulnerability management |
| Audit Logs | Detect | A.8.15 - Logging |
| Incident Response | Respond | A.5.24-28 - Incident management |
| Data Recovery | Recover | A.8.13 - Backup |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **What** | 18 prioritized, actionable security controls |
| **Version** | CIS Controls v8 |
| **IGs** | IG1 (essential), IG2 (intermediate), IG3 (advanced) |
| **Key Advantage** | Prescriptive and prioritized — tells you WHAT to do first |
| **Start With** | IG1 (56 safeguards) — essential cyber hygiene |
| **Free?** | Yes — freely available from CIS |

---

## Quick Revision Questions

1. **How do CIS Controls differ from NIST CSF and ISO 27001?**
2. **How many controls are in CIS v8 and what are the first 6?**
3. **What are Implementation Groups and which should a small business use?**
4. **Why is "Inventory of Enterprise Assets" Control #1?**
5. **How many safeguards are in IG1 and why is it considered essential?**

---

[← Previous: ISO 27001/27002](02-iso-27001-27002.md) | [Next: MITRE ATT&CK →](04-mitre-attack.md)

---

*Unit 4 - Topic 3 of 6 | [Back to README](../README.md)*
