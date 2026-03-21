# Unit 6: Cloud Security Certifications — Topic 2: Azure Security Engineer

## Overview

The **Microsoft Certified: Azure Security Engineer Associate (AZ-500)** validates skills in managing security posture, implementing threat protection, and managing identity and access in Azure environments. This topic covers exam structure, key services, and preparation strategies.

---

## 1. Exam Specifications

```
AZ-500 EXAM DETAILS:

  Certification:   Azure Security Engineer Associate
  Exam Code:       AZ-500
  Questions:        40-60
  Question Types:   MCQ, case studies, labs, drag-drop
  Duration:         150 minutes
  Passing Score:    700/1000 (scaled)
  Cost:             $165 USD
  Prerequisites:    Recommended:
                    - AZ-104 (Azure Admin) knowledge
                    - Networking and virtualization exp
  Validity:         1 year (free renewal assessment)
  Delivery:         Pearson VUE

EXAM DOMAINS:
  Domain                              | Weight
  1. Manage Identity & Access         | 25-30%
  2. Secure Networking                | 20-25%
  3. Secure Compute, Storage, DBs     | 20-25%
  4. Manage Security Operations       | 25-30%

QUESTION TYPES:
  → Multiple choice (single/multi select)
  → Case studies (multi-question scenarios)
  → Active screen (simulated portal)
  → Drag and drop
  → Build list (ordering)
  → Some sections cannot go back
```

---

## 2. Key Azure Security Services

```
AZURE SECURITY SERVICES:

IDENTITY & ACCESS:
  → Microsoft Entra ID (Azure AD): Identity provider
  → Conditional Access: Risk-based access policies
  → PIM (Privileged Identity Management): JIT admin
  → MFA: Multi-factor authentication
  → Identity Protection: Risk detection
  → Access Reviews: Periodic access audit
  → Managed Identities: Service authentication
  → Key Vault: Secrets, keys, certificates

NETWORK SECURITY:
  → NSG (Network Security Groups): L4 filtering
  → Azure Firewall: L7 managed firewall
  → Azure DDoS Protection: DDoS mitigation
  → Application Gateway + WAF: Web protection
  → Private Link: Private connectivity
  → Service Endpoints: Secure service access
  → Front Door: Global load balancer + WAF
  → Bastion: Secure RDP/SSH access

SECURITY OPERATIONS:
  → Microsoft Defender for Cloud: CSPM + CWPP
  → Microsoft Sentinel: Cloud-native SIEM/SOAR
  → Microsoft Defender for Endpoint: EDR
  → Microsoft Defender for Identity: AD protection
  → Microsoft Defender for Office 365: Email security
  → Log Analytics: Central log collection

COMPUTE & DATA:
  → Disk Encryption: ADE, SSE
  → Storage encryption: At rest by default
  → SQL: TDE, Always Encrypted, Dynamic Masking
  → Azure Policy: Governance and compliance
  → Blueprints: Environment standards
  → Management Groups: Hierarchical governance
```

---

## 3. Core Exam Concepts

```
IDENTITY & ACCESS (25-30%):

MICROSOFT ENTRA ID:
  → Users, groups, applications
  → RBAC (Role-Based Access Control)
  → Built-in roles vs custom roles
  → Scope: Management group > Subscription >
    Resource group > Resource
  
CONDITIONAL ACCESS:
  → If [conditions] then [controls]
  → Conditions: User, location, device, risk
  → Controls: Allow, block, MFA, compliant device
  → Named locations: Trusted IPs
  → Risk-based: Sign-in risk, user risk

PIM (Privileged Identity Management):
  → Just-In-Time (JIT) access
  → Time-limited role activation
  → Approval workflows
  → Access reviews
  → Audit trail for all activations
  → Eligible vs Active assignments

MANAGED IDENTITIES:
  → System-assigned: Tied to resource lifecycle
  → User-assigned: Independent lifecycle
  → No credentials to manage
  → Used for service-to-service auth
  → Example: VM accessing Key Vault

SECURE NETWORKING (20-25%):

NSG vs AZURE FIREWALL:
  NSG:
  → Layer 4 (IP, port, protocol)
  → Subnet or NIC level
  → Free (with limitations)
  → Stateful
  
  Azure Firewall:
  → Layer 7 (FQDN filtering)
  → Centralized, managed
  → Threat intelligence
  → TLS inspection
  → Paid service
```

---

## 4. Security Operations

```
SECURITY OPERATIONS (25-30%):

MICROSOFT DEFENDER FOR CLOUD:
  → Cloud Security Posture Management (CSPM)
  → Secure Score: Security health metric
  → Recommendations: Actionable findings
  → Regulatory compliance dashboard
  → Cloud Workload Protection:
    - Servers
    - Containers
    - Databases
    - Storage
    - App Service
    - Key Vault
  → Just-In-Time VM access
  → Adaptive application controls
  → File integrity monitoring

MICROSOFT SENTINEL:
  → Cloud-native SIEM + SOAR
  → Data connectors: 100+ integrations
  → Analytics rules: Threat detection
  → Incidents: Alert correlation
  → Playbooks: Automated response (Logic Apps)
  → Workbooks: Visualization dashboards
  → Hunting: Proactive threat hunting
  → KQL (Kusto Query Language)
  
  KEY KQL CONCEPTS:
  SecurityEvent
  | where TimeGenerated > ago(24h)
  | where EventID == 4625
  | summarize count() by Account
  | order by count_ desc

DIAGNOSTIC SETTINGS:
  → Platform logs → Log Analytics workspace
  → Activity logs: Control plane operations
  → Resource logs: Data plane operations
  → Metrics: Performance data
  → Archive to Storage Account
  → Stream to Event Hubs
```

---

## 5. Study Resources and Strategy

```
PREPARATION:

OFFICIAL MICROSOFT:
  → Microsoft Learn: Free learning paths
    - AZ-500 learning path (comprehensive)
    - Hands-on labs in sandbox
    - Module assessments
  → Microsoft official practice assessment (free)
  → Azure documentation
  → Microsoft Security blog

COURSES:
  1. Microsoft Learn (FREE)
     → Complete AZ-500 path
     → Sandbox labs included
     → Best starting point
  
  2. John Savill (YouTube, FREE)
     → Excellent explanations
     → Diagram-heavy teaching
     → Study cram videos
  
  3. A Cloud Guru / Pluralsight
     → Structured course
     → Practice exams
     → Hands-on labs

  4. Scott Duffy (Udemy)
     → Concise and focused
     → Practice exams
     → Regular updates

PRACTICE:
  → Azure free account ($200 credit)
  → Set up: Entra ID, Key Vault, NSGs
  → Configure: Conditional Access, PIM
  → Deploy: Sentinel, Defender for Cloud
  → Practice: KQL queries
  → Use: Azure Policy and RBAC

STUDY TIMELINE (6-8 weeks):
  Week 1-2: Identity & Access (Entra ID, CA, PIM)
  Week 3-4: Networking (NSG, Firewall, WAF)
  Week 5:   Compute/Storage/DB security
  Week 6-7: Security Operations (Sentinel, Defender)
  Week 8:   Practice exams + review

CAREER IMPACT:
  → Azure market share growing rapidly
  → High demand for Azure security skills
  → Pairs with: SC-200 (SOC Analyst)
  → Salary range: $90,000-$140,000
  → Path to: Azure Solutions Architect
```

---

## Summary Table

| Domain | Weight | Key Services |
|--------|--------|-------------|
| Identity & Access | 25-30% | Entra ID, Conditional Access, PIM |
| Networking | 20-25% | NSG, Azure Firewall, WAF |
| Compute/Storage/DB | 20-25% | Disk Encryption, TDE, Policy |
| Security Operations | 25-30% | Sentinel, Defender for Cloud |

---

## Revision Questions

1. What are the four domains of the AZ-500 exam?
2. How does Conditional Access work in Microsoft Entra ID?
3. What is the difference between NSG and Azure Firewall?
4. What is Microsoft Sentinel and how does it differ from Defender for Cloud?
5. What is Privileged Identity Management (PIM)?

---

*Previous: [01-aws-security-specialty.md](01-aws-security-specialty.md) | Next: [03-google-cloud-security.md](03-google-cloud-security.md)*

---

*[Back to README](../README.md)*
