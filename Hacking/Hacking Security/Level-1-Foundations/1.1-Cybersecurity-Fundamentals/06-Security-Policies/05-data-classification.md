# Data Classification

## Unit 6 - Topic 5 | Security Policies

---

## Overview

**Data classification** is the process of organizing data into categories based on its sensitivity level, so that appropriate security controls can be applied. Not all data is equal — customer credit card numbers need more protection than a company's public press releases.

---

## 1. Classification Levels

### Corporate/Commercial Classification

```
┌──────────────────────────────────────────────────────────────────┐
│                DATA CLASSIFICATION LEVELS                        │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  🔴 RESTRICTED / TOP SECRET                                      │
│     • Trade secrets, M&A plans, encryption keys                  │
│     • Highest protection, very limited access                    │
│     • Breach = catastrophic business impact                      │
│                                                                  │
│  🟠 CONFIDENTIAL                                                 │
│     • Customer PII, financial data, HR records                   │
│     • Strong access controls, encryption required                │
│     • Breach = significant damage, regulatory fines              │
│                                                                  │
│  🟡 INTERNAL                                                     │
│     • Internal memos, policies, project plans                    │
│     • Available to all employees, not public                     │
│     • Breach = embarrassment, minor impact                       │
│                                                                  │
│  🟢 PUBLIC                                                       │
│     • Marketing materials, press releases, job postings          │
│     • No restrictions on distribution                            │
│     • Breach = no impact (already public)                        │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Government Classification

| Level | Description |
|-------|-------------|
| **Top Secret** | Exceptionally grave damage to national security |
| **Secret** | Serious damage to national security |
| **Confidential** | Damage to national security |
| **Unclassified** | No damage to national security |

---

## 2. Handling Requirements per Level

| Requirement | Public | Internal | Confidential | Restricted |
|-------------|:------:|:--------:|:------------:|:----------:|
| **Encryption at rest** | No | No | Yes | Yes |
| **Encryption in transit** | No | Recommended | Yes | Yes |
| **Access control** | None | Role-based | Need-to-know | Strict need-to-know |
| **Labeling** | Optional | Required | Required | Required |
| **Sharing externally** | Allowed | NDA required | Approval required | Prohibited |
| **Printing** | Allowed | Allowed | Controlled | Prohibited |
| **Disposal** | Recycle | Shred | Cross-cut shred | Pulverize/Incinerate |
| **Backup** | Standard | Standard | Encrypted | Encrypted + isolated |

---

## 3. Data Lifecycle

```
┌──────────────────────────────────────────────────────────────┐
│                  DATA LIFECYCLE                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  CREATE ──► Classify at creation, assign label               │
│     │                                                        │
│     ▼                                                        │
│  STORE ──► Apply appropriate encryption and access controls  │
│     │                                                        │
│     ▼                                                        │
│  USE ──► Follow handling procedures for the classification   │
│     │                                                        │
│     ▼                                                        │
│  SHARE ──► Only with authorized parties using secure methods │
│     │                                                        │
│     ▼                                                        │
│  ARCHIVE ──► Retain per retention policy with protection     │
│     │                                                        │
│     ▼                                                        │
│  DESTROY ──► Securely dispose based on classification level  │
│                                                              │
│  Security controls apply at EVERY stage.                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Data Roles

| Role | Responsibility |
|------|---------------|
| **Data Owner** | Business leader responsible for classification and access decisions |
| **Data Custodian** | IT team implementing and managing technical controls |
| **Data Steward** | Ensures data quality and compliance with policies |
| **Data User** | Anyone who accesses data — must follow handling rules |
| **Data Processor** | Third party processing data on behalf of the owner |

---

## 5. Secure Data Destruction

| Method | Description | For |
|--------|-------------|-----|
| **Clearing** | Overwriting data (not recoverable by standard tools) | Internal data |
| **Purging** | Making data unrecoverable even with forensic tools | Confidential data |
| **Shredding** | Physical destruction of media | All physical media |
| **Degaussing** | Magnetic field destroys data on magnetic media | Hard drives, tapes |
| **Crypto-shredding** | Delete encryption keys, making data unreadable | Cloud/encrypted data |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Levels** | Public → Internal → Confidential → Restricted |
| **Purpose** | Apply appropriate controls based on data sensitivity |
| **Lifecycle** | Create → Store → Use → Share → Archive → Destroy |
| **Data Owner** | Business leader who decides classification and access |
| **Destruction** | Method depends on classification level |
| **Label everything** | Data must be marked with its classification |

---

## Quick Revision Questions

1. **What are the four common data classification levels?**
2. **What handling differences exist between Confidential and Public data?**
3. **What is the difference between a Data Owner and a Data Custodian?**
4. **What are the six stages of the data lifecycle?**
5. **What is crypto-shredding and when is it used?**

---

[← Previous: Password Policy](04-password-policy.md) | [Next: Security Awareness Training →](06-security-awareness-training.md)

---

*Unit 6 - Topic 5 of 6 | [Back to README](../README.md)*
