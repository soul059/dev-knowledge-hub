# Unit 6: Cloud Security Certifications — Topic 4: CCSP (Certified Cloud Security Professional)

## Overview

The **(ISC)² Certified Cloud Security Professional (CCSP)** is a vendor-neutral cloud security certification that validates comprehensive knowledge across all cloud security domains. It complements the CISSP and is ideal for professionals working in multi-cloud environments.

---

## 1. Exam Specifications

```
CCSP EXAM DETAILS:

  Organization:    (ISC)²
  Exam Format:     Linear (fixed-form)
  Questions:       150
  Duration:        4 hours
  Passing Score:   700/1000 (scaled)
  Cost:            $599 USD
  Prerequisites:   5 years IT experience
                   (3 in infosec, 1 in cloud)
                   OR CISSP can substitute all
  Validity:        3 years (30 CPE/year)
  Delivery:        Pearson VUE

EXPERIENCE OPTIONS:
  → Standard: 5 years cumulative
    - 3 years in information security
    - 1 year in cloud security (any domain)
  → With CISSP: No additional experience needed
  → Associate: Pass exam, earn experience later
    (same as CISSP Associate path)

EXAM DOMAINS:
  Domain                              | Weight
  1. Cloud Concepts, Architecture     | 17%
  2. Cloud Data Security              | 20%
  3. Cloud Platform & Infrastructure  | 17%
  4. Cloud Application Security       | 17%
  5. Cloud Security Operations        | 16%
  6. Legal, Risk & Compliance         | 13%

WEIGHT VISUALIZATION:
  D1 Architecture    [████████░░░░░░░░░░░░] 17%
  D2 Data Security   [██████████░░░░░░░░░░] 20%  ← Heaviest
  D3 Platform/Infra  [████████░░░░░░░░░░░░] 17%
  D4 Application Sec [████████░░░░░░░░░░░░] 17%
  D5 Operations      [████████░░░░░░░░░░░░] 16%
  D6 Legal/Risk      [██████░░░░░░░░░░░░░░] 13%
```

---

## 2. The Six Domains

```
DOMAIN DETAILS:

D1: CLOUD CONCEPTS, ARCHITECTURE (17%):
  → Cloud computing characteristics
    - On-demand, broad access, pooling
    - Elasticity, measured service
  → Service models: IaaS, PaaS, SaaS
  → Deployment models: Public, private, hybrid
  → Cloud reference architecture
  → Shared responsibility model
  → Security considerations per model
  → Business requirements analysis

D2: CLOUD DATA SECURITY (20%):
  → Data lifecycle: Create, store, use,
    share, archive, destroy
  → Data classification and labeling
  → Cloud data storage technologies
  → Data discovery and classification tools
  → Data rights management (IRM/DRM)
  → Data retention and deletion
  → Data privacy and sovereignty
  → Encryption and tokenization
  → Key management in cloud

D3: CLOUD PLATFORM & INFRASTRUCTURE (17%):
  → Physical and virtual infrastructure
  → Network security in cloud
  → Compute security
  → Management plane security
  → Business continuity / Disaster recovery
  → Virtualization security
  → Container security
  → Serverless security

D4: CLOUD APPLICATION SECURITY (17%):
  → Secure SDLC for cloud applications
  → Cloud-native application security
  → API security
  → Threat modeling
  → OWASP Cloud Top 10
  → DevSecOps in cloud
  → Application security testing
  → Identity management for apps

D5: CLOUD SECURITY OPERATIONS (16%):
  → Incident response in cloud
  → Log management and monitoring
  → Vulnerability management
  → Digital forensics in cloud
  → Communication with stakeholders
  → Continuous monitoring
  → Security operations center

D6: LEGAL, RISK & COMPLIANCE (13%):
  → Legal requirements (GDPR, HIPAA)
  → Privacy laws and regulations
  → Risk management in cloud
  → Audit methodologies
  → Cloud contracts and SLAs
  → Vendor management
  → Compliance frameworks
```

---

## 3. Key CCSP Concepts

```
CRITICAL CONCEPTS:

SHARED RESPONSIBILITY:
  Model | Customer | Provider
  IaaS  | OS, apps, data, network config | HW, virt, phys
  PaaS  | Apps, data | OS, runtime, HW
  SaaS  | Data, access mgmt | Everything else
  
  → Customer ALWAYS responsible for data
  → Provider ALWAYS responsible for physical
  → Middle varies by model

DATA LIFECYCLE IN CLOUD:
  Create → Store → Use → Share → Archive → Destroy
  
  Security at each phase:
  → Create: Classify, label, encrypt
  → Store: Encryption at rest, access controls
  → Use: DLP, monitoring, access logging
  → Share: DRM, secure transfer, authorization
  → Archive: Encryption, integrity, retention
  → Destroy: Crypto-shredding, secure delete

CLOUD FORENSICS CHALLENGES:
  → Multi-tenancy complicates evidence
  → Data may be in multiple jurisdictions
  → Provider cooperation required
  → Chain of custody in virtual environment
  → Volatile evidence (VMs can be destroyed)
  → SLA must include forensic provisions
  → Cloud-specific: snapshot, logging access

SOC REPORTS IN CLOUD:
  → SOC 1: Financial controls
  → SOC 2 Type I: Design effectiveness
  → SOC 2 Type II: Operating effectiveness
  → SOC 3: Public report (summary)
  → Most relevant: SOC 2 Type II
  → Request from cloud providers

CLOUD CONTRACTS:
  → SLA: Service Level Agreement
  → Right to audit
  → Data ownership clauses
  → Breach notification requirements
  → Data location/sovereignty
  → Exit strategy / data portability
  → Liability and indemnification
```

---

## 4. CCSP vs CISSP

```
CCSP vs CISSP COMPARISON:

  Aspect        | CCSP          | CISSP
  Focus         | Cloud security | General security
  Domains       | 6             | 8
  Questions     | 150           | 100-150 (CAT)
  Duration      | 4 hours       | 4 hours
  Format        | Linear        | CAT
  Cost          | $599          | $749
  Experience    | 5 years       | 5 years
  Cloud specific| Yes           | No
  Vendor neutral| Yes           | Yes

OVERLAP:
  → Risk management concepts
  → Access control models
  → Encryption and PKI
  → Incident response
  → Legal and compliance
  → Security governance

UNIQUE TO CCSP:
  → Cloud-specific architecture
  → Multi-tenancy security
  → Cloud data lifecycle
  → Cloud forensics challenges
  → Shared responsibility models
  → Cloud contract requirements
  → Data sovereignty in cloud

WHICH TO GET FIRST:
  → CISSP first if: Broad security career
  → CCSP first if: Cloud-focused career
  → Both: Maximum marketability
  → CISSP holders: CCSP is easier (overlap)
  → CISSP satisfies CCSP experience requirement

CAREER PATHS:
  CCSP positions:
  → Cloud Security Architect
  → Cloud Security Engineer
  → Cloud Compliance Officer
  → Cloud Security Consultant
  → Salary: $110,000-$160,000+
```

---

## 5. Study Resources

```
PREPARATION:

BOOKS:
  1. "(ISC)² CCSP CBK Reference"
     → Official (ISC)² textbook
     → Authoritative source
     → Reference-style
  
  2. "CCSP All-in-One Exam Guide"
     → Comprehensive coverage
     → Practice questions
     → Clear explanations
  
  3. "CCSP Study Guide" (Ben Malisow)
     → Sybex / Wiley
     → Online practice tests
     → Well-organized

COURSES:
  1. (ISC)² Official Training
     → Most comprehensive
     → Expensive ($2,000+)
     → Includes labs
  
  2. Pluralsight / A Cloud Guru
     → Online at own pace
     → Subscription-based
     → Practice exams
  
  3. LinkedIn Learning
     → Mike Chapple's course
     → Professional quality
     → Part of subscription

PRACTICE EXAMS:
  → (ISC)² Official Practice Tests
  → Boson ExSim
  → Sybex online test bank
  → Pocket Prep mobile app
  → CCCure (online)

STUDY TIMELINE (10-14 weeks):
  Week 1-2:  D1: Cloud concepts and architecture
  Week 3-5:  D2: Cloud data security (heaviest)
  Week 5-7:  D3: Platform and infrastructure
  Week 7-9:  D4: Application security
  Week 9-11: D5: Security operations
  Week 11-12: D6: Legal, risk, compliance
  Week 13-14: Review + practice exams

TIPS:
  → Similar mindset to CISSP (managerial)
  → Focus on cloud-SPECIFIC concepts
  → Know shared responsibility cold
  → Understand data lifecycle thoroughly
  → Know legal/privacy differences by region
  → If you have CISSP, leverage overlap
```

---

## Summary Table

| Domain | Weight | Key Concepts |
|--------|--------|-------------|
| D1: Architecture | 17% | Cloud models, reference architecture |
| D2: Data Security | 20% | Data lifecycle, encryption, DLP |
| D3: Platform/Infra | 17% | Network, compute, BC/DR |
| D4: App Security | 17% | SDLC, API, DevSecOps |
| D5: Operations | 16% | IR, monitoring, forensics |
| D6: Legal/Risk | 13% | Compliance, contracts, privacy |

---

## Revision Questions

1. What are the six CCSP domains and their weights?
2. How does the shared responsibility model differ across IaaS, PaaS, and SaaS?
3. What are the phases of the cloud data lifecycle?
4. What challenges are unique to digital forensics in cloud environments?
5. How does CCSP compare to CISSP and which should you pursue first?

---

*Previous: [03-google-cloud-security.md](03-google-cloud-security.md) | Next: [05-study-approaches.md](05-study-approaches.md)*

---

*[Back to README](../README.md)*
