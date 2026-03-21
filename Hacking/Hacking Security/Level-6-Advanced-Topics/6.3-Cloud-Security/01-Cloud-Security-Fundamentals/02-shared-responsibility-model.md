# Unit 1: Cloud Security Fundamentals — Topic 2: Shared Responsibility Model

## Overview

The **Shared Responsibility Model** defines the division of security obligations between the cloud service provider (CSP) and the customer. Understanding this model is critical because misunderstanding responsibilities is the #1 cause of cloud security incidents. Each major cloud provider publishes their version, but the core concept remains the same: the provider secures the cloud infrastructure, while the customer secures what they put in the cloud.

---

## 1. The Core Model

```
SHARED RESPONSIBILITY PRINCIPLE:

  "Security OF the cloud" = Provider's responsibility
  "Security IN the cloud" = Customer's responsibility

VISUAL MODEL:

  ┌─────────────────────────────────────────────┐
  │          CUSTOMER RESPONSIBILITY             │
  │                                              │
  │  ┌────────────────────────────────────────┐  │
  │  │ Customer Data                          │  │
  │  ├────────────────────────────────────────┤  │
  │  │ Platform, Apps, IAM                    │  │
  │  ├────────────────────────────────────────┤  │
  │  │ Operating System, Network, Firewall    │  │
  │  ├────────────────────────────────────────┤  │
  │  │ Client-side Encryption, Data Integrity │  │
  │  │ Network Traffic Protection             │  │
  │  └────────────────────────────────────────┘  │
  ├──────────────────────────────────────────────┤
  │          PROVIDER RESPONSIBILITY             │
  │                                              │
  │  ┌────────────────────────────────────────┐  │
  │  │ Compute, Storage, Database, Networking │  │
  │  ├────────────────────────────────────────┤  │
  │  │ Hardware / AWS Global Infrastructure   │  │
  │  ├────────────────────────────────────────┤  │
  │  │ Regions, Availability Zones, Edge      │  │
  │  ├────────────────────────────────────────┤  │
  │  │ Physical Security of Data Centers      │  │
  │  └────────────────────────────────────────┘  │
  └──────────────────────────────────────────────┘

KEY PRINCIPLE:
  → Moving to cloud does NOT eliminate security responsibility
  → It SHIFTS some responsibilities to the provider
  → Customer always responsible for their data and access
  → Misconfigured resources = CUSTOMER's fault
```

---

## 2. Responsibility by Service Model

```
IaaS SHARED RESPONSIBILITY:

  Provider Manages:              Customer Manages:
  ┌──────────────────┐          ┌──────────────────┐
  │ Physical security│          │ Data              │
  │ Hypervisor       │          │ Applications      │
  │ Physical network │          │ OS (patching)     │
  │ Physical storage │          │ Network config    │
  │ Power & cooling  │          │ Firewall rules    │
  │ Hardware maint.  │          │ IAM               │
  └──────────────────┘          │ Encryption        │
                                │ Logging/Monitoring│
                                └──────────────────┘
  
  Example: EC2 Instance
  → AWS: Physical server security, hypervisor isolation
  → You: OS patching, security groups, SSH keys, AV

PaaS SHARED RESPONSIBILITY:

  Provider Manages:              Customer Manages:
  ┌──────────────────┐          ┌──────────────────┐
  │ All IaaS items + │          │ Data              │
  │ Operating system  │          │ Application code  │
  │ Runtime           │          │ User access       │
  │ Middleware        │          │ App configuration │
  │ Patching         │          │ API security      │
  └──────────────────┘          │ Secrets mgmt      │
                                └──────────────────┘
  
  Example: Azure App Service
  → Azure: OS patching, runtime updates, infrastructure
  → You: App code security, authentication, data protection

SaaS SHARED RESPONSIBILITY:

  Provider Manages:              Customer Manages:
  ┌──────────────────┐          ┌──────────────────┐
  │ Everything       │          │ Data (what you    │
  │ except user      │          │   store/share)    │
  │ data and access  │          │ User accounts     │
  │                  │          │ Access controls   │
  │ App code         │          │ Configuration     │
  │ Infrastructure   │          │ Compliance        │
  │ Patching         │          │ MFA settings      │
  │ Availability     │          │ Sharing policies  │
  └──────────────────┘          └──────────────────┘
  
  Example: Office 365
  → Microsoft: Application security, infrastructure, updates
  → You: User accounts, MFA, data sharing, DLP policies
```

---

## 3. Provider-Specific Models

```
AWS SHARED RESPONSIBILITY:

AWS Responsibilities ("Security OF the Cloud"):
  → Physical security of data centers
  → Hardware and infrastructure
  → Network infrastructure
  → Hypervisor and virtualization
  → Managed services infrastructure (RDS, Lambda, S3)

Customer Responsibilities ("Security IN the Cloud"):
  → Customer data
  → Identity and access management
  → Application security
  → OS and network configuration (IaaS)
  → Encryption (client-side and server-side)
  → Network traffic protection
  → Firewall rules

AZURE SHARED RESPONSIBILITY:
  Similar to AWS model with same division
  Additional emphasis on:
  → Azure AD configuration
  → Conditional Access policies
  → Information protection labels
  → Azure Defender/Security Center configuration

GCP SHARED RESPONSIBILITY:
  Google calls it "Shared Fate" model
  → Google provides secure-by-default infrastructure
  → Customers receive security guidance and tools
  → Emphasis on security blueprints
  → Security Command Center integration
  → BeyondProd security model

COMMON MISCONCEPTION:
  ✗ "It's in the cloud, so the provider handles security"
  ✓ "The provider secures infrastructure; we secure our data,
     access, and configuration"
  
  REALITY:
  → Most cloud breaches are due to customer misconfiguration
  → Gartner: "Through 2025, 99% of cloud security failures 
     will be the customer's fault"
```

---

## 4. Security Gaps and Common Failures

```
WHERE BREACHES HAPPEN:

MISCONFIGURATION (Most Common):
  → Public S3 buckets
  → Open security groups (0.0.0.0/0 on RDP/SSH)
  → Excessive IAM permissions
  → Unencrypted databases
  → Public-facing management consoles
  → Default credentials on services
  → Missing MFA on privileged accounts

REAL-WORLD EXAMPLES:
  Capital One (2019):
  → Misconfigured WAF on AWS
  → SSRF to metadata service
  → Accessed IAM role credentials
  → Exfiltrated 100M+ records
  → Customer responsibility: WAF configuration, role permissions

  Twitch (2021):
  → Misconfigured server
  → Exposed entire source code
  → Internal tools and data leaked
  → Customer responsibility: Access controls, data classification

  Accenture (2017):
  → 4 public S3 buckets
  → Exposed client data, credentials
  → No authentication required
  → Customer responsibility: Bucket policies, access controls

SHARED RESPONSIBILITY GAPS:
  Area                  | Often Assumed | Actually
  Data encryption       | Provider      | Customer (mostly)
  OS patching (IaaS)    | Provider      | Customer
  IAM configuration     | Provider      | Customer
  Network rules         | Provider      | Customer
  Backup configuration  | Provider      | Customer
  Logging setup         | Provider      | Customer
  Compliance            | Provider      | Shared
```

---

## 5. Best Practices

```
IMPLEMENTING SHARED RESPONSIBILITY:

UNDERSTAND YOUR BOUNDARIES:
  1. Read provider's shared responsibility documentation
  2. Map responsibilities to your team/roles
  3. Document what you're responsible for
  4. Create security baselines for each service

FOR EACH SERVICE YOU USE:
  → Identify: What does the provider secure?
  → Identify: What do we need to secure?
  → Configure: Security settings and controls
  → Monitor: Logging, alerting, compliance checks
  → Review: Periodic security assessments

GOVERNANCE:
  → Cloud security policy documenting responsibilities
  → Training for teams on their obligations
  → Regular compliance audits
  → Automated configuration checks (CSPM)
  → Incident response plans for cloud

TOOLS FOR VERIFICATION:
  → AWS Security Hub + Trusted Advisor
  → Azure Security Center / Defender for Cloud
  → GCP Security Command Center
  → Third-party: ScoutSuite, Prowler, CloudSploit
  → Infrastructure as Code scanning: Checkov, tfsec

COMPLIANCE MAPPING:
  Standard    | Provider Responsibility    | Customer Responsibility
  SOC 2       | Infrastructure controls    | Application/data controls
  HIPAA       | BAA coverage               | PHI handling, encryption
  PCI DSS     | Infrastructure compliance  | Application, data, access
  GDPR        | Infrastructure security    | Data processing, consent
  ISO 27001   | Infrastructure cert        | Extend to your workloads
```

---

## Summary Table

| Service Model | Provider Secures | Customer Secures |
|--------------|-----------------|-----------------|
| IaaS | Physical, hypervisor, network | OS, apps, data, IAM |
| PaaS | IaaS + OS, runtime | App code, data, access |
| SaaS | Everything except data/access | Data, users, config |

---

## Revision Questions

1. What is the fundamental principle of the shared responsibility model?
2. How do security responsibilities differ across IaaS, PaaS, and SaaS?
3. What are the most common cloud security failures related to shared responsibility?
4. How does GCP's "Shared Fate" model differ from AWS's approach?
5. What tools help verify compliance with shared responsibility obligations?

---

*Previous: [01-cloud-service-models.md](01-cloud-service-models.md) | Next: [03-cloud-threat-landscape.md](03-cloud-threat-landscape.md)*

---

*[Back to README](../README.md)*
