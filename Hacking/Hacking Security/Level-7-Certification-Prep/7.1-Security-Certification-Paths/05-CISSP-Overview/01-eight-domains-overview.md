# Unit 5: CISSP Overview — Topic 1: Eight Domains Overview

## Overview

The **(ISC)² Certified Information Systems Security Professional (CISSP)** is the gold standard management-level security certification. Unlike technical certifications, CISSP focuses on security governance, risk management, and organizational security strategy. This topic provides a comprehensive overview of all eight CISSP domains.

---

## 1. CISSP Exam Specifications

```
CISSP EXAM DETAILS:

  Organization:    (ISC)²
  Exam Format:     CAT (Computerized Adaptive Testing)
  Questions:       100-150 (adaptive)
  Duration:        4 hours
  Passing Score:   700 out of 1000
  Language:        English (CAT), others (linear)
  Cost:            $749 USD
  Prerequisites:   5 years professional experience
                   (4 with relevant degree)
  Validity:        3 years (40 CPE/year, 120 total)
  
CAT FORMAT:
  → Starts at medium difficulty
  → Correct → harder questions
  → Incorrect → easier questions
  → Minimum: 100 questions
  → Maximum: 150 questions
  → Pass/fail determined by ability estimate
  → Finishes early if pass/fail is clear
  → NOT a fixed percentage scoring
  
EXPERIENCE REQUIREMENTS:
  → 5 years in 2+ of 8 domains
  → OR 4 years + 4-year degree
  → OR 4 years + approved credential
  → Can sit exam first, earn "Associate of (ISC)²"
  → Endorsement by (ISC)² member required
  → Experience verified by (ISC)²
```

---

## 2. The Eight Domains

```
CISSP EIGHT DOMAINS:

  Domain                          | Weight
  1. Security & Risk Management   | 15%
  2. Asset Security               | 10%
  3. Security Architecture        | 13%
  4. Communication & Network Sec  | 13%
  5. Identity & Access Mgmt (IAM) | 13%
  6. Security Assessment/Testing  | 12%
  7. Security Operations          | 13%
  8. Software Development Security| 11%

WEIGHT VISUALIZATION:
  D1 [████████░░░░░░░░░░░░] 15%  ← Heaviest
  D2 [█████░░░░░░░░░░░░░░░] 10%
  D3 [██████░░░░░░░░░░░░░░] 13%
  D4 [██████░░░░░░░░░░░░░░] 13%
  D5 [██████░░░░░░░░░░░░░░] 13%
  D6 [██████░░░░░░░░░░░░░░] 12%
  D7 [██████░░░░░░░░░░░░░░] 13%
  D8 [█████░░░░░░░░░░░░░░░] 11%

KEY INSIGHT: Domains are relatively
evenly weighted — no domain can be skipped.
Domain 1 (Risk Management) is slightly
heavier and sets the foundation.
```

---

## 3. Domain Details (1-4)

```
DOMAIN 1: SECURITY & RISK MANAGEMENT (15%):
  → CIA triad (Confidentiality, Integrity, Availability)
  → Security governance principles
  → Compliance and regulatory requirements
  → Legal and regulatory issues
  → Professional ethics (ISC² Code of Ethics)
  → Security policies, standards, procedures
  → Business continuity (BCP)
  → Personnel security policies
  → Risk management concepts
  → Threat modeling (STRIDE, PASTA, DREAD)
  → Supply chain risk management
  → Security awareness and training

DOMAIN 2: ASSET SECURITY (10%):
  → Information and asset classification
  → Data ownership (owner, custodian, steward)
  → Privacy protection
  → Asset retention and destruction
  → Data security controls
  → Data handling requirements
  → Data remanence and sanitization
  → DLP (Data Loss Prevention)

DOMAIN 3: SECURITY ARCHITECTURE (13%):
  → Secure design principles
  → Security models (Bell-LaPadula, Biba, Clark-Wilson)
  → Security evaluation models (Common Criteria)
  → System security architecture
  → Vulnerability of architectures
  → Cryptography concepts
  → Physical security
  → Site and facility design

DOMAIN 4: COMMUNICATION & NETWORK (13%):
  → OSI and TCP/IP models
  → Network protocols and services
  → Network attacks (DoS, MitM, spoofing)
  → Network security devices
  → Secure communication channels
  → VPN, IPSec, TLS/SSL
  → Wireless security
  → Network access control
```

---

## 4. Domain Details (5-8)

```
DOMAIN 5: IDENTITY & ACCESS MANAGEMENT (13%):
  → Physical and logical access control
  → Authentication methods (MFA)
  → Authorization mechanisms
  → Identity management lifecycle
  → Access control models (DAC, MAC, RBAC, ABAC)
  → Federated identity management
  → Single Sign-On (SSO)
  → Session management
  → Registration and proofing
  → Credential management

DOMAIN 6: SECURITY ASSESSMENT & TESTING (12%):
  → Assessment strategies
  → Security testing methods
  → Vulnerability assessments
  → Penetration testing
  → Log reviews and audit trails
  → Synthetic transactions
  → Code review and testing
  → Security audits (internal/external)
  → SOC reports (Type I, II, III)
  → Key performance indicators (KPIs)

DOMAIN 7: SECURITY OPERATIONS (13%):
  → Investigations (administrative, criminal)
  → Digital forensics
  → Incident management
  → Logging and monitoring
  → Provisioning of resources
  → Configuration management
  → Change management
  → Patch and vulnerability management
  → Disaster recovery (DR)
  → Business continuity (BC)
  → Physical security operations

DOMAIN 8: SOFTWARE DEVELOPMENT SECURITY (11%):
  → SDLC (Software Development Lifecycle)
  → Secure coding practices
  → Software security effectiveness
  → Acquired software security
  → DevOps/DevSecOps
  → Database security
  → Application vulnerabilities
  → Web application security
  → AI/ML security considerations
  → Code repositories and versioning
```

---

## 5. Cross-Domain Concepts

```
CONCEPTS THAT SPAN MULTIPLE DOMAINS:

RISK MANAGEMENT (appears everywhere):
  → Risk assessment methodologies
  → Quantitative vs qualitative
  → Risk treatment options
  → Risk frameworks (NIST, ISO)
  → Risk acceptance criteria

GOVERNANCE (D1, D2, D5, D7):
  → Policies, standards, procedures
  → Compliance requirements
  → Due diligence and due care
  → Organizational roles

CRYPTOGRAPHY (D1, D3, D4, D5):
  → Symmetric/asymmetric algorithms
  → Hashing and digital signatures
  → PKI and certificates
  → Key management lifecycle

INCIDENT RESPONSE (D1, D6, D7):
  → IR phases
  → Evidence handling
  → Chain of custody
  → Forensic procedures

ACCESS CONTROL (D2, D5, D7):
  → Least privilege
  → Separation of duties
  → Need to know
  → Defense in depth

EXAM PHILOSOPHY:
  → Think like a MANAGER, not a technician
  → Choose the answer that protects the BUSINESS
  → Risk-based decision making
  → Cost-effective security
  → Compliance-driven approach
  → Safety of human life ALWAYS first
```

---

## Summary Table

| Domain | Weight | Focus Area |
|--------|--------|-----------|
| D1: Security & Risk Mgmt | 15% | Governance, risk, compliance |
| D2: Asset Security | 10% | Data classification, privacy |
| D3: Security Architecture | 13% | Design, models, crypto |
| D4: Network Security | 13% | Protocols, devices, VPN |
| D5: IAM | 13% | Auth, access control models |
| D6: Assessment & Testing | 12% | Auditing, pen testing |
| D7: Security Operations | 13% | IR, forensics, DR/BC |
| D8: Software Dev Security | 11% | SDLC, secure coding |

---

## Revision Questions

1. How many domains does the CISSP exam cover and what are their weights?
2. What experience is required to earn the CISSP certification?
3. How does the CAT (Computerized Adaptive Testing) format work?
4. What security models are covered in Domain 3?
5. What is the primary mindset difference between CISSP and technical certifications?

---

*Previous: None (First topic in this unit) | Next: [02-experience-requirements.md](02-experience-requirements.md)*

---

*[Back to README](../README.md)*
