# Unit 4: GCP Security — Topic 1: GCP Security Architecture

## Overview

**Google Cloud Platform (GCP)** provides a security architecture built on Google's infrastructure, which has been securing billions of users for decades. GCP's security model encompasses physical infrastructure, custom hardware, encrypted network communication, and a comprehensive set of security services. Understanding GCP's security architecture is essential for securing workloads and conducting security assessments on Google Cloud.

---

## 1. GCP Infrastructure

```
GCP GLOBAL INFRASTRUCTURE:

  ┌──────────────────────────────────────────────┐
  │              GCP GLOBAL NETWORK              │
  │                                              │
  │  ┌────────────────────────────────────────┐  │
  │  │    REGION (e.g., us-central1)          │  │
  │  │                                        │  │
  │  │  ┌──────┐  ┌──────┐  ┌──────┐         │  │
  │  │  │Zone-a│  │Zone-b│  │Zone-c│         │  │
  │  │  └──────┘  └──────┘  └──────┘         │  │
  │  └────────────────────────────────────────┘  │
  │                                              │
  │  39+ Regions | 118+ Zones | 187+ Edge Locs   │
  └──────────────────────────────────────────────┘

RESOURCE HIERARCHY:

  ┌──────────────────────────────┐
  │      Organization            │  ← Domain-level root
  │  ┌────────────────────────┐  │
  │  │      Folders           │  │  ← Grouping mechanism
  │  │  ┌──────────────────┐  │  │
  │  │  │    Projects      │  │  │  ← Resource containers
  │  │  │  ┌────────────┐  │  │  │
  │  │  │  │ Resources   │  │  │  │  ← VMs, Storage, etc.
  │  │  │  └────────────┘  │  │  │
  │  │  └──────────────────┘  │  │
  │  └────────────────────────┘  │
  └──────────────────────────────┘

  Organization → top-level node (linked to Google Workspace)
  Folders → group projects (e.g., dept, env)
  Projects → billing boundary, resource container
  Resources → actual compute, storage, networking

EXAMPLE:
  Organization: example.com
  ├── Folder: Production
  │   ├── Project: prod-web-app
  │   ├── Project: prod-database
  │   └── Project: prod-analytics
  ├── Folder: Development
  │   ├── Project: dev-web-app
  │   └── Project: dev-database
  └── Folder: Shared Services
      ├── Project: shared-networking
      └── Project: shared-monitoring

IAM INHERITANCE:
  → Policies set at organization → inherited by ALL folders/projects
  → Policies set at folder → inherited by child projects
  → Policies set at project → apply to that project's resources
  → More permissive policies ALWAYS win (additive only)
  → Cannot "deny" at lower level (use IAM deny policies for this)
```

---

## 2. GCP Security Model

```
GOOGLE'S SECURITY INFRASTRUCTURE:

PHYSICAL SECURITY:
  → Custom-designed data centers
  → Biometric access controls
  → 24/7 monitoring
  → Hardware destruction process
  → Titan security chip (hardware root of trust)

NETWORK SECURITY:
  → Private global fiber network
  → Encryption in transit (between data centers)
  → ALTS (Application Layer Transport Security)
  → BeyondCorp model (zero trust)
  → Google Front End (GFE) for TLS termination

STORAGE SECURITY:
  → Encryption at rest by default (AES-256)
  → Automatic key rotation
  → Customer-managed encryption keys (CMEK)
  → Customer-supplied encryption keys (CSEK)
  → Data deletion pipeline

COMPUTE SECURITY:
  → Shielded VMs (Secure Boot, vTPM, Integrity)
  → Confidential Computing (encrypted in use)
  → Container-Optimized OS
  → Binary Authorization

DEFENSE IN DEPTH:
  ┌────────────────────────────────┐
  │ Operations & Device Security   │
  ├────────────────────────────────┤
  │ Identity & Access              │
  │ (Cloud IAM, BeyondCorp)        │
  ├────────────────────────────────┤
  │ Network Security               │
  │ (VPC, Firewall, Cloud Armor)   │
  ├────────────────────────────────┤
  │ Compute & Workload Security    │
  │ (Shielded VMs, Binary Auth)    │
  ├────────────────────────────────┤
  │ Data Security                  │
  │ (Encryption, DLP, Key Mgmt)    │
  ├────────────────────────────────┤
  │ Monitoring & Logging           │
  │ (Cloud Audit, SCC, Chronicle)  │
  └────────────────────────────────┘
```

---

## 3. Core Security Services

```
GCP SECURITY SERVICES:

IDENTITY:
  Cloud IAM         → Role-based access control
  Identity Platform → Customer identity (CIAM)
  BeyondCorp        → Zero trust access
  Identity-Aware Proxy (IAP) → App-level access control
  Managed Service for Microsoft AD → AD in GCP

NETWORK:
  VPC               → Virtual private cloud
  Cloud Firewall    → Network traffic filtering
  Cloud Armor       → DDoS protection and WAF
  Cloud NAT         → Outbound NAT
  Private Google Access → Private PaaS access
  VPC Service Controls → API-level perimeter

DATA:
  Cloud KMS         → Key management service
  Secret Manager    → Secret storage
  DLP API           → Data loss prevention
  Certificate Authority Service → PKI

MONITORING:
  Security Command Center (SCC) → Security posture
  Cloud Audit Logs  → API activity logging
  Chronicle         → Security analytics (SIEM)
  Event Threat Detection → Automated threat detection
  Container Threat Detection → Runtime container security

COMPLIANCE:
  Assured Workloads → Compliance environments
  Access Transparency → Audit Google access
  VPC Service Controls → Data exfiltration protection

gcloud CLI COMMANDS:
  # Login
  gcloud auth login
  
  # Set project
  gcloud config set project PROJECT_ID
  
  # List projects
  gcloud projects list
  
  # List compute instances
  gcloud compute instances list
  
  # List firewall rules
  gcloud compute firewall-rules list
  
  # List IAM bindings
  gcloud projects get-iam-policy PROJECT_ID
  
  # List service accounts
  gcloud iam service-accounts list
  
  # List storage buckets
  gsutil ls
```

---

## 4. GCP vs AWS vs Azure

```
COMPARISON:

Feature           | GCP                    | AWS                  | Azure
Identity          | Cloud IAM              | IAM                  | Azure AD/RBAC
MFA               | 2-Step Verification    | MFA                  | MFA
SSO               | Cloud Identity         | SSO                  | Azure AD SSO
Firewall          | VPC Firewall           | Security Groups      | NSG
WAF               | Cloud Armor            | WAF                  | Azure WAF
DDoS              | Cloud Armor            | Shield               | DDoS Protection
SIEM              | Chronicle              | SecurityHub+GuardDuty| Sentinel
Posture Mgmt      | SCC                    | Security Hub         | Defender for Cloud
Key Mgmt          | Cloud KMS              | KMS                  | Key Vault
Secrets           | Secret Manager         | Secrets Manager      | Key Vault
Encryption        | Default (AES-256)      | Default (AES-256)    | Default (AES-256)
Container Sec     | Binary Authorization   | ECR Scanning         | Defender for Containers
Zero Trust        | BeyondCorp/IAP         | Verified Access      | Conditional Access
```

---

## 5. Security Best Practices

```
GCP SECURITY BEST PRACTICES:

ORGANIZATION:
  [ ] Enable Organization Policy constraints
  [ ] Use folders for environment separation
  [ ] Restrict project creation to admins
  [ ] Enable audit logging at organization level
  [ ] Enable Security Command Center (Premium)

IDENTITY:
  [ ] Enforce MFA for all users
  [ ] Use groups for IAM (not individual users)
  [ ] Minimize use of primitive roles (Owner/Editor)
  [ ] Use predefined roles or custom roles
  [ ] Regular IAM policy reviews
  [ ] Limit service account key usage

NETWORK:
  [ ] Use Shared VPC for centralized networking
  [ ] Implement VPC Service Controls
  [ ] Use Private Google Access
  [ ] Configure Cloud Firewall rules (least privilege)
  [ ] Enable VPC Flow Logs
  [ ] Use Cloud NAT (no public IPs on VMs)

DATA:
  [ ] Use CMEK where required
  [ ] Enable uniform bucket-level access
  [ ] Configure DLP for sensitive data
  [ ] Use Secret Manager (not env vars)
  [ ] Enable object versioning on critical buckets

MONITORING:
  [ ] Enable Cloud Audit Logs (all services)
  [ ] Export logs to centralized location
  [ ] Configure alerts for security events
  [ ] Enable Data Access logs for sensitive projects
  [ ] Regular security posture review in SCC
```

---

## Summary Table

| Service | Category | Purpose |
|---------|----------|---------|
| Cloud IAM | Identity | Access control |
| Cloud Armor | Network | DDoS and WAF |
| Security Command Center | Posture | Security assessment |
| Cloud KMS | Data | Key management |
| Chronicle | Monitoring | SIEM analytics |
| VPC Service Controls | Network | Data perimeter |

---

## Revision Questions

1. How does GCP's resource hierarchy differ from AWS and Azure?
2. What makes Google's infrastructure security unique (Titan chip, ALTS)?
3. What are the core GCP security services across each domain?
4. How does BeyondCorp implement zero trust in GCP?
5. What gcloud commands are essential for security assessment?

---

*Previous: None (First topic in this unit) | Next: [02-iam-fundamentals.md](02-iam-fundamentals.md)*

---

*[Back to README](../README.md)*
