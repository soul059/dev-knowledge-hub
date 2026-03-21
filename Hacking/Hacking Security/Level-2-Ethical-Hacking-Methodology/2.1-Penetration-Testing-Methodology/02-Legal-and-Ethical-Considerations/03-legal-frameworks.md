# Legal Frameworks

## Unit 2 - Topic 3 | Legal and Ethical Considerations

---

## Overview

Penetration testers must understand the **legal frameworks** that govern computer security testing. Laws vary significantly by country and jurisdiction. Unauthorized access — even with good intentions — can result in **criminal prosecution, fines, and imprisonment**.

---

## 1. Major Computer Crime Laws

| Country | Law | Key Provisions |
|---------|-----|---------------|
| **USA** | Computer Fraud and Abuse Act (CFAA) | Unauthorized access to protected computers; up to 10 years |
| **USA** | Electronic Communications Privacy Act | Wiretapping and electronic surveillance |
| **UK** | Computer Misuse Act 1990 | Unauthorized access; up to 10 years |
| **EU** | Directive 2013/40/EU | Attacks against information systems |
| **EU** | GDPR | Data protection and privacy |
| **India** | IT Act 2000 (Section 66) | Unauthorized access; up to 3 years |
| **Australia** | Criminal Code Act (Part 10.7) | Unauthorized access to computer data |
| **Canada** | Criminal Code (Section 342.1) | Unauthorized use of computer |

---

## 2. US Computer Fraud and Abuse Act (CFAA)

```
┌──────────────────────────────────────────────────────────────┐
│              CFAA — KEY PROVISIONS                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  OFFENSE                    │ PENALTY                        │
│  ──────────────────────────┼───────────────────────         │
│  Unauthorized access to     │ Up to 1 year (misdemeanor)    │
│  obtain information         │ Up to 5 years (felony)        │
│                             │                                │
│  Unauthorized access to     │ Up to 5 years                  │
│  government computers       │                                │
│                             │                                │
│  Accessing to defraud       │ Up to 5 years                  │
│  and obtain value           │                                │
│                             │                                │
│  Intentional damage         │ Up to 10 years                 │
│  (malware, DoS)             │ Up to 20 years (repeat)       │
│                             │                                │
│  Trafficking in passwords   │ Up to 1 year                   │
│                             │ Up to 10 years (repeat)       │
│                                                              │
│  ⚠️ "Exceeding authorized access" is broadly interpreted    │
│  ⚠️ Even employees can violate CFAA by exceeding access     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Data Protection Laws

| Law | Scope | Relevance to Pen Testing |
|-----|-------|-------------------------|
| **GDPR** (EU) | EU citizens' data | Tester may access personal data |
| **HIPAA** (US) | Healthcare data | Medical records may be exposed |
| **PCI-DSS** | Payment card data | Requires pen testing annually |
| **SOX** | Financial data | Financial system testing |
| **CCPA** (US) | California residents | Consumer data protection |
| **PIPEDA** (Canada) | Personal information | Canadian data privacy |

```
GDPR IMPLICATIONS FOR PEN TESTERS:
├── If you access EU citizen data during testing:
│   ├── Data processing agreement may be needed
│   ├── Minimize data collection
│   ├── Encrypt all captured data
│   ├── Delete after engagement
│   └── Report data exposure in findings
├── Pen test report itself may contain personal data
│   └── Must be handled according to GDPR
└── Client must have lawful basis for the pen test
```

---

## 4. Industry Compliance Requirements

| Standard | Pen Test Requirement |
|----------|---------------------|
| **PCI-DSS** | Annual pen test + after significant changes (Req 11.3) |
| **HIPAA** | Risk assessment (pen test recommended) |
| **SOC 2** | Pen test as part of security controls |
| **ISO 27001** | Pen test within security testing program |
| **NIST CSF** | Pen test as part of identify/protect functions |
| **FedRAMP** | Annual pen test for cloud providers |

---

## 5. Ethical Guidelines

```
PENETRATION TESTER CODE OF ETHICS:

1. NEVER test without explicit written authorization
2. NEVER exceed the defined scope
3. NEVER access data beyond what's needed to prove a finding
4. ALWAYS protect client confidentiality
5. ALWAYS report ALL findings honestly
6. ALWAYS handle sensitive data responsibly
7. NEVER use findings for personal gain
8. ALWAYS disclose conflicts of interest
9. NEVER plant backdoors for personal access
10. ALWAYS clean up after testing
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **CFAA** | US law — unauthorized access up to 10+ years |
| **Computer Misuse Act** | UK law — unauthorized access up to 10 years |
| **GDPR** | EU data protection — applies to data found during tests |
| **PCI-DSS** | Requires annual penetration testing |
| **Authorization** | Only legal defense against computer crime charges |
| **Ethics** | Never exceed scope, protect data, report honestly |

---

## Quick Revision Questions

1. **What is the CFAA and what does it prohibit?**
2. **How does GDPR affect penetration testing?**
3. **Which compliance standard requires annual pen tests?**
4. **What is the UK Computer Misuse Act?**
5. **Name 3 ethical obligations of a penetration tester.**

---

[← Previous: Rules of Engagement](02-rules-of-engagement.md) | [Next: Contractual Requirements →](04-contractual-requirements.md)

---

*Unit 2 - Topic 3 of 6 | [Back to README](../README.md)*
