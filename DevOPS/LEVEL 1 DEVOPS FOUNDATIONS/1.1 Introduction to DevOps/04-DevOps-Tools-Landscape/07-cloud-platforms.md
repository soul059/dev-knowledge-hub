# Chapter 4.7 — Cloud Platforms

[← Previous: Monitoring Tools](06-monitoring-tools.md) | [Next: Unit 5 — DORA Metrics →](../05-DevOps-Metrics/01-dora-metrics.md)

---

## Overview

Cloud platforms provide on-demand infrastructure, services, and tools over the internet. Understanding the **Big Three** — AWS, Azure, and Google Cloud — is essential for DevOps engineers, as most modern infrastructure runs in the cloud.

---

## Cloud Service Models

```
┌──────────────────────────────────────────────────────────┐
│  CLOUD SERVICE MODELS                                    │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │  On-                                              │   │
│  │  Premises   IaaS        PaaS        SaaS          │   │
│  │                                                    │   │
│  │  ┌──────┐  ┌──────┐   ┌──────┐   ┌──────┐        │   │
│  │  │ App  │  │ App  │   │ App  │   │██████│        │   │
│  │  ├──────┤  ├──────┤   ├──────┤   ├──────┤        │   │
│  │  │ Data │  │ Data │   │ Data │   │██████│        │   │
│  │  ├──────┤  ├──────┤   ├──────┤   ├──────┤        │   │
│  │  │Runtme│  │Runtme│   │██████│   │██████│        │   │
│  │  ├──────┤  ├──────┤   ├──────┤   ├──────┤        │   │
│  │  │  OS  │  │  OS  │   │██████│   │██████│        │   │
│  │  ├──────┤  ├──────┤   ├──────┤   ├──────┤        │   │
│  │  │Virtua│  │██████│   │██████│   │██████│        │   │
│  │  ├──────┤  ├──────┤   ├──────┤   ├──────┤        │   │
│  │  │Server│  │██████│   │██████│   │██████│        │   │
│  │  ├──────┤  ├──────┤   ├──────┤   ├──────┤        │   │
│  │  │Netwrk│  │██████│   │██████│   │██████│        │   │
│  │  └──────┘  └──────┘   └──────┘   └──────┘        │   │
│  │                                                    │   │
│  │  You manage  ███ = Cloud provider manages          │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  IaaS: EC2, Azure VMs, GCE                               │
│  PaaS: AWS Elastic Beanstalk, Azure App Service, GCP GAE│
│  SaaS: Gmail, Slack, Salesforce                          │
└──────────────────────────────────────────────────────────┘
```

---

## The Big Three Cloud Providers

```
┌──────────────────────────────────────────────────────────┐
│  CLOUD MARKET SHARE (approx. 2024)                       │
│                                                          │
│  AWS     ████████████████████████████  ~31%              │
│  Azure   ██████████████████████        ~25%              │
│  GCP     ████████████                  ~11%              │
│  Others  ██████████████                ~33%              │
│                                                          │
│  AWS: First mover, broadest services (200+)              │
│  Azure: Enterprise + Microsoft ecosystem                 │
│  GCP: Data/ML/Kubernetes leadership                      │
└──────────────────────────────────────────────────────────┘
```

---

## Service Mapping Across Clouds

| Category | AWS | Azure | GCP |
|----------|-----|-------|-----|
| **Compute (VMs)** | EC2 | Virtual Machines | Compute Engine |
| **Containers** | ECS / EKS | AKS | GKE |
| **Serverless** | Lambda | Functions | Cloud Functions |
| **Object Storage** | S3 | Blob Storage | Cloud Storage |
| **Relational DB** | RDS | Azure SQL | Cloud SQL |
| **NoSQL DB** | DynamoDB | Cosmos DB | Firestore |
| **DNS** | Route 53 | Azure DNS | Cloud DNS |
| **CDN** | CloudFront | Azure CDN | Cloud CDN |
| **IAM** | IAM | Entra ID (AD) | Cloud IAM |
| **Monitoring** | CloudWatch | Azure Monitor | Cloud Monitoring |
| **IaC** | CloudFormation | ARM / Bicep | Deployment Manager |
| **CI/CD** | CodePipeline | Azure DevOps | Cloud Build |
| **Secrets** | Secrets Manager | Key Vault | Secret Manager |
| **Message Queue** | SQS / SNS | Service Bus | Pub/Sub |

---

## AWS Core Services for DevOps

```yaml
# Key AWS Services a DevOps Engineer Should Know

Compute:
  - EC2: Virtual servers
  - ECS: Container service (AWS-native)
  - EKS: Managed Kubernetes
  - Lambda: Serverless functions
  - Fargate: Serverless containers

Networking:
  - VPC: Virtual private network
  - ALB/NLB: Load balancers
  - Route 53: DNS service
  - CloudFront: CDN

Storage:
  - S3: Object storage (limitless)
  - EBS: Block storage (for EC2)
  - EFS: Shared file system

Database:
  - RDS: Managed SQL (MySQL, PostgreSQL)
  - DynamoDB: Managed NoSQL
  - ElastiCache: Managed Redis/Memcached

DevOps Tools:
  - CodePipeline: CI/CD orchestration
  - CodeBuild: Build service
  - CodeDeploy: Deployment automation
  - ECR: Container registry
  - CloudWatch: Monitoring + logs
  - Systems Manager: Server management
  - Secrets Manager: Secrets storage

Security:
  - IAM: Identity & access management
  - KMS: Key management
  - WAF: Web application firewall
  - GuardDuty: Threat detection
```

---

## Cloud-Native DevOps Architecture

```
┌──────────────────────────────────────────────────────────┐
│  TYPICAL CLOUD-NATIVE ARCHITECTURE (AWS)                 │
│                                                          │
│                    ┌──────────┐                          │
│  Users ──────────►│CloudFront│  (CDN)                   │
│                    └─────┬────┘                          │
│                    ┌─────▼────┐                          │
│                    │   ALB    │  (Load Balancer)         │
│                    └─────┬────┘                          │
│               ┌──────────┼──────────┐                   │
│          ┌────▼───┐ ┌────▼───┐ ┌────▼───┐              │
│          │  EKS   │ │  EKS   │ │  EKS   │              │
│          │ Pod 1  │ │ Pod 2  │ │ Pod 3  │              │
│          └────┬───┘ └────┬───┘ └────┬───┘              │
│               │          │          │                    │
│         ┌─────▼──────────▼──────────▼──────┐            │
│         │         VPC (Private Subnets)     │            │
│         │  ┌─────────┐  ┌──────────────┐   │            │
│         │  │   RDS   │  │ ElastiCache  │   │            │
│         │  │(Postgres)│  │  (Redis)     │   │            │
│         │  └─────────┘  └──────────────┘   │            │
│         └──────────────────────────────────┘            │
│                          │                              │
│         ┌────────────────▼──────────────────┐           │
│         │   S3 (static assets, backups)     │           │
│         │   CloudWatch (logs + metrics)      │           │
│         │   Secrets Manager (credentials)    │           │
│         └────────────────────────────────────┘           │
└──────────────────────────────────────────────────────────┘
```

---

## Multi-Cloud vs Single Cloud

```
┌──────────────────────────────────────────────────────────┐
│  STRATEGY TRADEOFFS                                      │
│                                                          │
│  SINGLE CLOUD                    MULTI-CLOUD             │
│  ════════════                    ═══════════             │
│  ✅ Simpler architecture         ✅ No vendor lock-in    │
│  ✅ Deeper integration           ✅ Leverage best of each│
│  ✅ Fewer skills needed          ✅ Regulatory compliance│
│  ✅ Volume discounts             ✅ Disaster recovery    │
│  ⚠️ Vendor lock-in              ⚠️ Higher complexity    │
│  ⚠️ Single point of failure     ⚠️ More expertise needed│
│                                  ⚠️ Inconsistent tooling│
│                                                          │
│  RECOMMENDATION:                                         │
│  Start single-cloud. Go multi-cloud only when there's    │
│  a clear business requirement (compliance, DR, etc.)     │
└──────────────────────────────────────────────────────────┘
```

---

## Real-World Scenario: Choosing a Cloud Provider

```
STARTUP SCENARIO:
├── Team of 10 engineers
├── Building a SaaS product
├── Need: containers, database, CI/CD, monitoring
├── Budget: Minimize costs
├── Team experience: Some AWS, some GCP

DECISION MATRIX:
                    AWS        Azure      GCP
Kubernetes (EKS)    $$$$       $$$        $$ (GKE free CP)
Database (managed)  $$$        $$$        $$
Startup credits     $100K      $150K      $200K*
Free tier           12 months  12 months  Always free tier
CI/CD               CodePipeline Azure DO Cloud Build
Container Registry  ECR        ACR        GCR / Artifact

DECISION: GCP
├── GKE has free control plane (saves ~$73/month)
├── $200K startup credits program
├── Strong Kubernetes tooling (Google invented K8s)
├── Competitive pricing for compute
└── Cloud Build has generous free tier

* Credit amounts vary by program and eligibility
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Unexpected high bill | Forgotten running resources | Enable billing alerts; use cost explorer; tag all resources |
| Permission denied | IAM policy too restrictive | Check IAM policies; use least privilege debugging |
| Region outage | Single-region deployment | Deploy across multiple AZs/regions; automate failover |
| Slow performance | Wrong instance type or region | Right-size instances; use closer regions; cache with CDN |
| Data transfer costs | Cross-region/cross-AZ traffic | Keep services in same AZ; use VPC endpoints |
| Security breach | Exposed credentials | Rotate credentials; use IAM roles instead of keys; enable MFA |

---

## Summary Table

| Cloud Concept | Description |
|--------------|-------------|
| **IaaS** | Infrastructure as a Service — VMs, networking, storage |
| **PaaS** | Platform as a Service — managed runtime for apps |
| **SaaS** | Software as a Service — fully managed applications |
| **AWS** | Market leader with broadest service catalog (200+) |
| **Azure** | Best for Microsoft/enterprise ecosystems |
| **GCP** | Strong in Kubernetes, data, and ML |
| **Region / AZ** | Geographic locations and isolated data centers |
| **Multi-Cloud** | Using multiple cloud providers (adds complexity) |

---

## Quick Revision Questions

1. **What is the difference between IaaS, PaaS, and SaaS?**
   <details><summary>Answer</summary>IaaS (Infrastructure as a Service): cloud provides VMs, storage, networking — you manage OS and above. Example: EC2, Azure VMs. PaaS (Platform as a Service): cloud also manages OS and runtime — you manage only your app and data. Example: Heroku, Azure App Service. SaaS (Software as a Service): cloud manages everything — you just use the application. Example: Gmail, Slack.</details>

2. **Name five AWS services a DevOps engineer should know and their purposes.**
   <details><summary>Answer</summary>1) EC2 — virtual servers for compute. 2) EKS — managed Kubernetes for container orchestration. 3) S3 — object storage for artifacts, backups, static files. 4) CloudWatch — monitoring, logs, and alerting. 5) IAM — identity and access management for security. Others: ECR (container registry), RDS (managed database), Lambda (serverless).</details>

3. **What is a Region and an Availability Zone (AZ)?**
   <details><summary>Answer</summary>A Region is a geographic area (e.g., us-east-1 = Northern Virginia) containing multiple data centers. An Availability Zone (AZ) is an isolated data center within a region with independent power, cooling, and networking. Best practice: deploy across multiple AZs for high availability (survive a data center failure) and across regions for disaster recovery.</details>

4. **When would you choose multi-cloud over single-cloud?**
   <details><summary>Answer</summary>Choose multi-cloud when: 1) Regulatory requirements mandate data in specific regions/providers. 2) Avoid vendor lock-in for strategic reasons. 3) Need disaster recovery across providers. 4) Want to leverage best-of-breed services (e.g., GCP for ML, AWS for general infra). Drawbacks: higher complexity, more expertise needed, inconsistent tooling.</details>

5. **How do you prevent unexpected cloud bills?**
   <details><summary>Answer</summary>1) Set billing alerts and budgets. 2) Tag all resources for cost allocation. 3) Use cost explorer/advisor tools. 4) Right-size instances (don't over-provision). 5) Use reserved instances or savings plans for steady workloads. 6) Shut down non-production resources outside business hours. 7) Review and delete unused resources regularly.</details>

6. **What is the equivalent of AWS S3 in Azure and GCP?**
   <details><summary>Answer</summary>AWS S3 = Azure Blob Storage = Google Cloud Storage. All three provide object storage for files, backups, static assets, and data lakes. They support versioning, lifecycle policies, encryption, access controls, and cross-region replication. When using Terraform, you can abstract these behind a module to simplify multi-cloud storage management.</details>

---

[← Previous: Monitoring Tools](06-monitoring-tools.md) | [Next: Unit 5 — DORA Metrics →](../05-DevOps-Metrics/01-dora-metrics.md)
