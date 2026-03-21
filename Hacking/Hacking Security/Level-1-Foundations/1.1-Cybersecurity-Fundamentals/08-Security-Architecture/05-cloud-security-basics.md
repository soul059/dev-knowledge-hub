# Cloud Security Basics

## Unit 8 - Topic 5 | Security Architecture

---

## Overview

**Cloud security** encompasses the technologies, policies, and controls used to protect data, applications, and infrastructure in cloud environments. As organizations migrate to cloud platforms (AWS, Azure, GCP), understanding the **shared responsibility model** and cloud-specific threats is essential for building secure architectures.

---

## 1. Cloud Service Models

```
┌──────────────────────────────────────────────────────────────────┐
│                  CLOUD SERVICE MODELS                            │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────┐     ┌──────────┐     ┌──────────┐                │
│  │  IaaS    │     │  PaaS    │     │  SaaS    │                │
│  │          │     │          │     │          │                │
│  │ You      │     │ You      │     │ Provider │                │
│  │ manage:  │     │ manage:  │     │ manages: │                │
│  │ ┌──────┐ │     │ ┌──────┐ │     │ ┌──────┐ │                │
│  │ │ Apps │ │     │ │ Apps │ │     │ │ Apps │ │                │
│  │ │ Data │ │     │ │ Data │ │     │ │ Data │ │                │
│  │ │ OS   │ │     │ └──────┘ │     │ │ OS   │ │                │
│  │ │ Mid. │ │     │          │     │ │ Mid. │ │                │
│  │ └──────┘ │     │ Provider │     │ │ Net  │ │                │
│  │          │     │ manages: │     │ │ Stor │ │                │
│  │ Provider │     │ ┌──────┐ │     │ │ Infra│ │                │
│  │ manages: │     │ │ OS   │ │     │ └──────┘ │                │
│  │ ┌──────┐ │     │ │ Mid. │ │     │          │                │
│  │ │ Net  │ │     │ │ Net  │ │     │          │                │
│  │ │ Stor │ │     │ │ Stor │ │     │          │                │
│  │ │ Infra│ │     │ │ Infra│ │     │          │                │
│  │ └──────┘ │     │ └──────┘ │     │          │                │
│  └──────────┘     └──────────┘     └──────────┘                │
│                                                                  │
│  Examples:                                                       │
│  IaaS: AWS EC2, Azure VM, GCP Compute Engine                    │
│  PaaS: AWS Elastic Beanstalk, Azure App Service, Heroku         │
│  SaaS: Office 365, Salesforce, Google Workspace                 │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Shared Responsibility Model

| Component | IaaS | PaaS | SaaS |
|-----------|:----:|:----:|:----:|
| **Data & Access** | 🔵 You | 🔵 You | 🔵 You |
| **Applications** | 🔵 You | 🔵 You | ☁️ Provider |
| **OS / Runtime** | 🔵 You | ☁️ Provider | ☁️ Provider |
| **Network Controls** | 🔵/☁️ Shared | ☁️ Provider | ☁️ Provider |
| **Physical Infrastructure** | ☁️ Provider | ☁️ Provider | ☁️ Provider |

```
KEY TAKEAWAY:
┌──────────────────────────────────────────────────────────────┐
│  YOU are ALWAYS responsible for:                              │
│  • Your DATA                                                 │
│  • Your ACCESS CONTROLS (IAM)                                │
│  • Your CONFIGURATION                                        │
│  • Your COMPLIANCE                                           │
│                                                              │
│  The cloud provider secures the INFRASTRUCTURE,              │
│  but YOU secure what you PUT in the cloud.                    │
│                                                              │
│  "Security OF the cloud" = Provider                          │
│  "Security IN the cloud" = Customer                          │
└──────────────────────────────────────────────────────────────┘
```

---

## 3. Cloud Security Threats

| Threat | Description | Mitigation |
|--------|-------------|-----------|
| **Misconfiguration** | #1 cloud threat — open S3 buckets, permissive security groups | Automated config scanning (CSPM) |
| **Insufficient IAM** | Over-privileged accounts, no MFA | Least privilege, enforce MFA |
| **Data Breaches** | Exposed databases, unencrypted storage | Encryption at rest + in transit |
| **Insecure APIs** | Weak API authentication/authorization | API gateway, OAuth, rate limiting |
| **Account Hijacking** | Stolen credentials, phishing | MFA, credential monitoring |
| **Shadow IT** | Unauthorized cloud services | Cloud Access Security Broker (CASB) |
| **Insider Threats** | Malicious or negligent employees | Audit logging, DLP, least privilege |

---

## 4. Key Cloud Security Controls

| Control | Description |
|---------|-------------|
| **IAM** | Identity and Access Management — who can access what |
| **MFA** | Multi-Factor Authentication — required for all cloud accounts |
| **Encryption** | Encrypt data at rest (AES-256) and in transit (TLS) |
| **Security Groups / NACLs** | Virtual firewalls controlling inbound/outbound traffic |
| **CSPM** | Cloud Security Posture Management — continuous config monitoring |
| **CASB** | Cloud Access Security Broker — shadow IT visibility |
| **Logging & Monitoring** | CloudTrail (AWS), Activity Log (Azure) — audit everything |
| **VPC / VNet** | Virtual Private Cloud — network isolation |
| **WAF** | Web Application Firewall — protect cloud-hosted web apps |

---

## 5. Cloud Deployment Models

| Model | Description | Example |
|-------|-------------|---------|
| **Public Cloud** | Shared infrastructure, multi-tenant | AWS, Azure, GCP |
| **Private Cloud** | Dedicated infrastructure for one organization | VMware, OpenStack |
| **Hybrid Cloud** | Mix of public and private | On-prem + AWS |
| **Multi-Cloud** | Using multiple cloud providers | AWS + Azure + GCP |
| **Community Cloud** | Shared by organizations with common concerns | Government cloud (GovCloud) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Service Models** | IaaS (most control), PaaS (mid), SaaS (least control) |
| **Shared Responsibility** | Provider secures infrastructure; you secure your data/configs |
| **#1 Threat** | Misconfiguration (open buckets, permissive rules) |
| **Key Controls** | IAM, MFA, encryption, security groups, CSPM, logging |
| **Always Your Job** | Data security, access control, compliance |

---

## Quick Revision Questions

1. **What are the three cloud service models and how do they differ?**
2. **Explain the shared responsibility model.**
3. **What is the #1 security threat in cloud environments?**
4. **What is a CSPM and why is it important?**
5. **Why is IAM considered the most critical cloud security control?**
6. **What is the difference between public, private, and hybrid cloud?**

---

[← Previous: Segmentation Strategies](04-segmentation-strategies.md) | [Next: Identity Management →](06-identity-management.md)

---

*Unit 8 - Topic 5 of 6 | [Back to README](../README.md)*
