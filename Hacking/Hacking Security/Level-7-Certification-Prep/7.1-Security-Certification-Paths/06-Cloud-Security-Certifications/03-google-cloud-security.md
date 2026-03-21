# Unit 6: Cloud Security Certifications — Topic 3: Google Cloud Security

## Overview

The **Google Cloud Professional Cloud Security Engineer** certification validates the ability to design and implement security on Google Cloud Platform (GCP). While GCP has smaller market share than AWS/Azure, it's growing rapidly and offers unique security capabilities.

---

## 1. Exam Specifications

```
GCP SECURITY ENGINEER:

  Certification:   Professional Cloud Security Engineer
  Questions:        50-60
  Question Types:   MCQ + Multiple select
  Duration:         2 hours (120 minutes)
  Passing Score:    Not published (scaled)
  Cost:             $200 USD
  Prerequisites:    Recommended:
                    - 3+ years industry experience
                    - 1+ year GCP experience
                    - Associate Cloud Engineer helpful
  Validity:         2 years
  Delivery:         Kryterion test centers / online

EXAM DOMAINS:
  Domain                              | Approx Weight
  1. Configuring Access               | ~20%
  2. Managing Operations              | ~20%
  3. Configuring Network Security     | ~20%
  4. Ensuring Data Protection         | ~20%
  5. Managing Compliance              | ~20%

NOTE: Google doesn't publish exact weights.
All domains are roughly equally important.
```

---

## 2. Key GCP Security Services

```
GCP SECURITY SERVICES:

IDENTITY & ACCESS:
  → Cloud IAM: Identity and Access Management
  → Cloud Identity: User management
  → Identity-Aware Proxy (IAP): App-level access
  → BeyondCorp Enterprise: Zero trust
  → Workload Identity Federation: External identity
  → Service Accounts: Machine identity
  → Organization Policies: Constraints
  → Access Context Manager: Perimeter-based access

NETWORK SECURITY:
  → VPC: Virtual Private Cloud
  → VPC Firewall Rules: L4 filtering
  → Cloud Armor: DDoS + WAF
  → Cloud NAT: Outbound NAT
  → Private Google Access: Private service access
  → VPC Service Controls: Data exfiltration prevention
  → Cloud Interconnect: Private connectivity
  → Identity-Aware Proxy: HTTPS access control

DETECTION & MONITORING:
  → Security Command Center (SCC): Central dashboard
  → Chronicle (SIEM): Security analytics
  → Event Threat Detection: Automated detection
  → Cloud Audit Logs: Activity logging
  → Cloud Logging: Centralized logging
  → Cloud Monitoring: Metrics and alerting
  → Binary Authorization: Container verification

DATA PROTECTION:
  → Cloud KMS: Key Management Service
  → Cloud HSM: Hardware security modules
  → DLP API: Sensitive data discovery
  → Secret Manager: Secret storage
  → Certificate Authority Service: PKI
  → Customer-Managed Encryption Keys (CMEK)
  → Customer-Supplied Encryption Keys (CSEK)
```

---

## 3. Core GCP Security Concepts

```
IAM ON GCP:

RESOURCE HIERARCHY:
  Organization
  └── Folder
      └── Project
          └── Resource

IAM COMPONENTS:
  → Principal: Who (user, group, service account)
  → Role: What permissions
  → Resource: Where applied
  → Policy: Binding of principal + role + resource

ROLE TYPES:
  → Basic (primitive): Owner, Editor, Viewer
    → Avoid in production (too broad)
  → Predefined: Service-specific (recommended)
  → Custom: Organization-defined

BEST PRACTICES:
  → Use groups, not individual users
  → Apply least privilege
  → Use predefined roles over basic
  → Use service accounts for workloads
  → Implement Organization Policies
  → Regular IAM audit

VPC SERVICE CONTROLS:
  → Creates security perimeters
  → Prevents data exfiltration
  → Controls API access boundaries
  → Restricts: BigQuery, Cloud Storage, etc.
  → Service perimeter + access levels + policies
  → Critical for exam — unique to GCP

BEYONDCORP (ZERO TRUST):
  → No trusted network zones
  → Access based on identity + context
  → Device trust assessment
  → Identity-Aware Proxy for applications
  → Context-aware access policies
  → Google's internal security model
```

---

## 4. GCP-Specific Security Features

```
UNIQUE GCP SECURITY:

SECURITY COMMAND CENTER (SCC):
  → Standard vs Premium tier
  → Asset discovery
  → Vulnerability scanning
  → Threat detection
  → Compliance monitoring
  → Web Security Scanner
  → Event Threat Detection
  → Container Threat Detection

BINARY AUTHORIZATION:
  → Enforce deployment policies
  → Only deploy signed container images
  → Attestation workflow
  → Integrates with GKE
  → Supply chain security

SHIELDED VMS:
  → Secure Boot
  → Virtual TPM (vTPM)
  → Integrity Monitoring
  → Protection against rootkits/bootkits
  → Enable by default

CONFIDENTIAL COMPUTING:
  → Data encrypted in use (memory)
  → Confidential VMs
  → AMD SEV technology
  → Protects from hypervisor attacks
  → Unique differentiation

ENCRYPTION:
  → Default encryption at rest (AES-256)
  → Envelope encryption (DEK + KEK)
  → CMEK: Customer-managed keys (KMS)
  → CSEK: Customer-supplied keys
  → Cloud HSM: FIPS 140-2 Level 3
  → Client-side encryption

LOGGING:
  → Admin Activity: Always on, free
  → Data Access: Must enable, costs
  → System Events: Always on, free
  → Policy Denied: Always on, free
  → All logs → Cloud Logging → export to:
    → Cloud Storage, BigQuery, Pub/Sub
```

---

## 5. Study Resources

```
PREPARATION:

OFFICIAL GOOGLE:
  → Google Cloud Skills Boost (Qwiklabs)
    - Professional Cloud Security Engineer path
    - Hands-on labs in real GCP
    - Free tier + paid labs
  → Google Cloud documentation
  → Google Cloud Security Foundations Blueprint
  → Google Cloud blog (security posts)

COURSES:
  1. Google Cloud Skills Boost (Official)
     → Structured learning path
     → Real GCP environment labs
     → Best official resource
  
  2. A Cloud Guru / Pluralsight
     → Complete exam course
     → Practice exams
     → Supplementary labs
  
  3. Udemy (various instructors)
     → Wait for sales
     → Check reviews before buying
     → Practice exams available

KEY STUDY AREAS:
  → VPC Service Controls (unique and important)
  → Organization Policies
  → IAM best practices
  → Data Loss Prevention API
  → Security Command Center
  → BeyondCorp/Zero Trust model
  → Encryption key management hierarchy

STUDY TIMELINE (8-10 weeks):
  Week 1-2: GCP fundamentals + IAM
  Week 3-4: Network security + VPC
  Week 5-6: Data protection + encryption
  Week 7-8: Operations + monitoring
  Week 9:   Compliance + review
  Week 10:  Practice exams

CAREER IMPACT:
  → Growing demand (GCP market expanding)
  → Less competition (fewer cert holders)
  → Valued in data/AI-heavy organizations
  → Salary range: $90,000-$140,000
  → Pairs with: GCP Professional Cloud Architect
```

---

## Summary Table

| Domain | Key Services | Focus Areas |
|--------|-------------|-------------|
| Access | IAM, IAP, BeyondCorp | Roles, policies, zero trust |
| Operations | SCC, Chronicle, Logging | Detection, monitoring |
| Network | VPC, Cloud Armor, Service Controls | Perimeters, firewalls |
| Data Protection | KMS, HSM, DLP, CMEK/CSEK | Encryption hierarchy |
| Compliance | Org Policies, Audit Logs | Governance, auditing |

---

## Revision Questions

1. How does the GCP resource hierarchy affect IAM policy inheritance?
2. What are VPC Service Controls and why are they critical?
3. How does BeyondCorp implement zero trust on GCP?
4. What is the difference between CMEK and CSEK encryption?
5. What is Binary Authorization and how does it secure containers?

---

*Previous: [02-azure-security-engineer.md](02-azure-security-engineer.md) | Next: [04-ccsp.md](04-ccsp.md)*

---

*[Back to README](../README.md)*
