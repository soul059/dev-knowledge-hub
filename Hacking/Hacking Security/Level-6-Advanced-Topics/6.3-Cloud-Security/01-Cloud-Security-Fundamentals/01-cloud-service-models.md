# Unit 1: Cloud Security Fundamentals — Topic 1: Cloud Service Models (IaaS, PaaS, SaaS)

## Overview

Understanding cloud service models is foundational to cloud security. The three primary models — **Infrastructure as a Service (IaaS)**, **Platform as a Service (PaaS)**, and **Software as a Service (SaaS)** — define what the cloud provider manages versus what the customer is responsible for. Each model presents different security considerations, attack surfaces, and control capabilities.

---

## 1. Cloud Computing Basics

```
WHAT IS CLOUD COMPUTING?

NIST DEFINITION (5 Essential Characteristics):
  1. On-demand self-service
  2. Broad network access
  3. Resource pooling
  4. Rapid elasticity
  5. Measured service

DEPLOYMENT MODELS:
  ┌──────────────────────────────────────────────────┐
  │              DEPLOYMENT MODELS                   │
  ├──────────┬───────────┬───────────┬───────────────┤
  │  Public  │  Private  │  Hybrid   │  Multi-Cloud  │
  │          │           │           │               │
  │ AWS      │ On-prem   │ Mix of    │ Multiple      │
  │ Azure    │ OpenStack │ public &  │ public cloud  │
  │ GCP      │ VMware    │ private   │ providers     │
  │          │           │           │               │
  │ Shared   │ Dedicated │ Connected │ Best of breed │
  │ infra    │ infra     │ infra     │ avoid lock-in │
  └──────────┴───────────┴───────────┴───────────────┘

SERVICE MODEL STACK:
  ┌─────────────────────────────────────────────┐
  │                    SaaS                     │
  │  Gmail, Office 365, Salesforce, Slack       │
  │  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─    │
  │                    PaaS                     │
  │  AWS Elastic Beanstalk, Azure App Service   │
  │  Google App Engine, Heroku                  │
  │  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─    │
  │                    IaaS                     │
  │  AWS EC2, Azure VMs, GCP Compute Engine     │
  │  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─    │
  │             Physical Infrastructure         │
  │  Data Centers, Servers, Storage, Network    │
  └─────────────────────────────────────────────┘
```

---

## 2. Infrastructure as a Service (IaaS)

```
IaaS — INFRASTRUCTURE AS A SERVICE:

WHAT IT PROVIDES:
  → Virtual machines (compute)
  → Storage (block, object, file)
  → Networking (VPC, subnets, load balancers)
  → Operating system images
  → Raw infrastructure components

CUSTOMER MANAGES:
  → Operating system (patching, hardening)
  → Middleware and runtime
  → Applications
  → Data
  → User access
  → Firewall rules (cloud-level)
  → Encryption configuration

PROVIDER MANAGES:
  → Physical servers
  → Hypervisor
  → Physical network
  → Physical storage
  → Data center security
  → Power, cooling, connectivity

EXAMPLES:
  → AWS EC2 (virtual machines)
  → Azure Virtual Machines
  → GCP Compute Engine
  → DigitalOcean Droplets
  → AWS S3 (object storage)
  → Azure Blob Storage

SECURITY CONSIDERATIONS:
  → OS vulnerabilities (you patch)
  → Network configuration (security groups)
  → Access control (IAM, SSH keys)
  → Data encryption (your responsibility)
  → Logging and monitoring (you configure)
  → Application security (fully yours)
  → Largest attack surface of all models

COMMON MISCONFIGURATIONS:
  → Default/weak SSH keys
  → Open security groups (0.0.0.0/0)
  → Unpatched operating systems
  → Excessive IAM permissions
  → Unencrypted storage volumes
  → Public-facing management interfaces
  → Missing logging/monitoring
```

---

## 3. Platform as a Service (PaaS)

```
PaaS — PLATFORM AS A SERVICE:

WHAT IT PROVIDES:
  → Application hosting platform
  → Runtime environment (Node.js, Python, Java, .NET)
  → Managed databases
  → Development tools
  → Deployment automation
  → Auto-scaling

CUSTOMER MANAGES:
  → Application code
  → Data
  → User access
  → Application-level security
  → Business logic

PROVIDER MANAGES:
  → Operating system
  → Runtime environment
  → Middleware
  → Physical infrastructure
  → Patching
  → Scaling infrastructure

EXAMPLES:
  → AWS Elastic Beanstalk
  → Azure App Service
  → Google App Engine
  → Heroku
  → AWS RDS (managed database)
  → Azure SQL Database
  → Cloud Functions / Lambda (FaaS — subset of PaaS)

SECURITY CONSIDERATIONS:
  → Application vulnerabilities (your code)
  → Authentication and authorization
  → Input validation
  → Secrets management (connection strings, API keys)
  → Data security
  → API security
  → Dependency vulnerabilities
  → Less OS-level control

COMMON MISCONFIGURATIONS:
  → Hardcoded credentials in code
  → Insecure API endpoints
  → Missing authentication on services
  → Overly permissive CORS
  → Debug mode in production
  → Exposed management endpoints
  → Missing rate limiting
```

---

## 4. Software as a Service (SaaS)

```
SaaS — SOFTWARE AS A SERVICE:

WHAT IT PROVIDES:
  → Complete application
  → Accessed via browser or API
  → No installation required
  → Provider handles everything

CUSTOMER MANAGES:
  → User accounts and access
  → Data within the application
  → Configuration settings
  → Integration settings
  → Compliance of data usage

PROVIDER MANAGES:
  → Application code and updates
  → Infrastructure
  → Operating system
  → Middleware
  → Security of the application
  → Availability and performance

EXAMPLES:
  → Microsoft 365 (Office, Email)
  → Google Workspace
  → Salesforce
  → Slack
  → Zoom
  → Dropbox
  → ServiceNow

SECURITY CONSIDERATIONS:
  → Account security (passwords, MFA)
  → Data access controls
  → Data residency and sovereignty
  → Vendor security posture
  → API integrations (OAuth tokens)
  → Data export and portability
  → Shadow IT (unauthorized SaaS usage)
  → Compliance requirements

COMMON MISCONFIGURATIONS:
  → No MFA enabled
  → Overshared documents/files
  → Excessive API permissions (OAuth)
  → No data loss prevention
  → Unmonitored admin accounts
  → Missing audit logging
  → Unsanctioned third-party integrations
```

---

## 5. Service Model Comparison

```
RESPONSIBILITY COMPARISON:

Component              │ On-Prem │ IaaS │ PaaS │ SaaS
───────────────────────┼─────────┼──────┼──────┼─────
Data                   │  You    │ You  │ You  │ You
Applications           │  You    │ You  │ You  │ CSP
Runtime                │  You    │ You  │ CSP  │ CSP
Middleware             │  You    │ You  │ CSP  │ CSP
Operating System       │  You    │ You  │ CSP  │ CSP
Virtualization         │  You    │ CSP  │ CSP  │ CSP
Servers                │  You    │ CSP  │ CSP  │ CSP
Storage                │  You    │ CSP  │ CSP  │ CSP
Networking             │  You    │ CSP  │ CSP  │ CSP
Physical Security      │  You    │ CSP  │ CSP  │ CSP

(CSP = Cloud Service Provider; You = Customer)

ATTACK SURFACE:
  IaaS:  ██████████████████████  (Largest — most customer control)
  PaaS:  ██████████████          (Medium — code and data focus)
  SaaS:  ████████                (Smallest — mostly access control)

FLEXIBILITY vs MANAGEMENT:
  IaaS:  Maximum control, maximum responsibility
  PaaS:  Balanced control and management
  SaaS:  Minimal control, minimal management

SECURITY CONTROL:
  IaaS → You control: OS, network, access, encryption, patching
  PaaS → You control: App code, data, access
  SaaS → You control: User access, data sharing, configuration
```

---

## Summary Table

| Aspect | IaaS | PaaS | SaaS |
|--------|------|------|------|
| Control | Highest | Medium | Lowest |
| Management | Most | Moderate | Least |
| Attack Surface | Largest | Medium | Smallest |
| Patching | Customer | Shared | Provider |
| Example | EC2 | App Service | Office 365 |
| Security Focus | Infrastructure | Application | Access |

---

## Revision Questions

1. What are the three primary cloud service models?
2. In IaaS, what security responsibilities fall on the customer?
3. How does the attack surface differ between IaaS, PaaS, and SaaS?
4. What are common misconfigurations in each service model?
5. How do deployment models (public, private, hybrid) affect security?

---

*Previous: None (First topic in this unit) | Next: [02-shared-responsibility-model.md](02-shared-responsibility-model.md)*

---

*[Back to README](../README.md)*
