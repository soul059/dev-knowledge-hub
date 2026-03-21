# Unit 6: Cloud Network Security — Topic 2: Network Segmentation

## Overview

**Network segmentation** in cloud environments divides the network into isolated segments to limit lateral movement, contain breaches, and enforce security policies. Unlike traditional networks where segmentation relies on VLANs and physical firewalls, cloud segmentation uses virtual constructs like subnets, security groups, and service-level controls. Effective segmentation is a cornerstone of cloud security architecture.

---

## 1. Segmentation Strategies

```
CLOUD SEGMENTATION LEVELS:

LEVEL 1: ACCOUNT/PROJECT SEPARATION
  → Separate cloud accounts per environment
  → Strongest isolation boundary
  → Independent IAM, billing, networking
  
  AWS: Separate AWS accounts (Organizations)
  Azure: Separate subscriptions
  GCP: Separate projects
  
  Example:
  Account: prod-workloads (production)
  Account: dev-workloads (development)
  Account: security-tools (security monitoring)
  Account: shared-services (DNS, logging)

LEVEL 2: VPC/VNet SEPARATION
  → Separate virtual networks
  → Network-level isolation
  → Controlled peering for communication
  
  VPC: prod-web-vpc
  VPC: prod-data-vpc
  VPC: management-vpc

LEVEL 3: SUBNET SEGMENTATION
  → Subnets within VPC
  → Public vs private
  → Tier-based (web/app/data)
  → Security groups per subnet
  
  Public subnet: Load balancers
  App subnet: Application servers
  Data subnet: Databases
  Mgmt subnet: Bastion hosts

LEVEL 4: MICROSEGMENTATION
  → Per-workload security policies
  → Based on identity, not network
  → Security groups per application
  → Zero trust between services

LEVEL 5: API-LEVEL CONTROLS
  → VPC Service Controls (GCP)
  → Service endpoints (Azure)
  → VPC endpoints (AWS)
  → API-level access restrictions

SEGMENTATION HIERARCHY:
  ┌──────────────────────────────┐
  │ Account/Project (strongest)  │
  ├──────────────────────────────┤
  │ VPC/VNet                     │
  ├──────────────────────────────┤
  │ Subnet                       │
  ├──────────────────────────────┤
  │ Security Group / NSG         │
  ├──────────────────────────────┤
  │ Application-level (weakest)  │
  └──────────────────────────────┘
```

---

## 2. Implementation by Cloud Provider

```
AWS SEGMENTATION:

  Security Groups (SG):
  → Stateful (return traffic auto-allowed)
  → Applied to ENIs (elastic network interfaces)
  → Allow rules only (no deny)
  → Can reference other SGs
  → Up to 5 SGs per ENI
  
  Network ACLs (NACL):
  → Stateless (must allow both directions)
  → Applied at subnet level
  → Allow and deny rules
  → Processed in order (lowest number first)
  → Subnet boundary enforcement

  Example: Three-tier isolation
  SG: web-sg     → Allow 80/443 from 0.0.0.0/0
  SG: app-sg     → Allow 8080 from web-sg
  SG: db-sg      → Allow 3306 from app-sg
  SG: mgmt-sg    → Allow 22 from bastion-sg

AZURE SEGMENTATION:

  NSG:
  → Applied to subnet or NIC
  → Inbound and outbound rules
  → Priority-based processing
  → Service tags for simplified rules
  → ASGs for logical grouping
  
  Azure Firewall:
  → Centralized Layer 3-7 filtering
  → FQDN filtering
  → Threat intelligence
  → Force-tunnel through firewall

GCP SEGMENTATION:

  Firewall Rules:
  → VPC-level (global)
  → Target by tags or service accounts
  → Priority-based
  → Hierarchical policies (org level)
  
  VPC Service Controls:
  → API-level perimeter
  → Prevents data exfiltration
  → Independent of network rules
```

---

## 3. Microsegmentation

```
MICROSEGMENTATION:

CONCEPT:
  → Security at the workload level
  → Every workload has its own policy
  → Zero trust between services
  → Identity-based, not network-based
  → Limits lateral movement to near-zero

TRADITIONAL vs MICROSEGMENTATION:
  Traditional:                 Microsegmentation:
  ┌─────────────────┐         ┌──┐ ┌──┐ ┌──┐
  │ Flat Network    │         │A │ │B │ │C │
  │ A ↔ B ↔ C ↔ D  │         │  │ │  │ │  │
  │ (all can talk)  │         └┬─┘ └┬─┘ └┬─┘
  └─────────────────┘          │    │    │
                               ▼    ▼    ▼
                            Policies per workload
                            A→B: Allow port 8080
                            B→C: Allow port 5432
                            A→C: DENY
                            D→*: DENY

TOOLS:
  Cloud-Native:
  → AWS Security Groups (per-instance)
  → Azure ASGs (application grouping)
  → GCP Firewall Rules (per-SA target)
  
  Third-Party:
  → Illumio
  → Guardicore (Akamai)
  → VMware NSX
  → Cisco Tetration
  → Zscaler Workload Segmentation

KUBERNETES NETWORK POLICIES:
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: allow-frontend-to-api
    namespace: production
  spec:
    podSelector:
      matchLabels:
        app: api
    ingress:
    - from:
      - podSelector:
          matchLabels:
            app: frontend
      ports:
      - protocol: TCP
        port: 8080
    policyTypes:
    - Ingress

  → Default deny all
  → Explicit allow per communication path
  → Label-based selection
  → Namespace isolation
```

---

## 4. Zero Trust Network

```
ZERO TRUST IN CLOUD:

PRINCIPLES:
  → Never trust, always verify
  → Assume breach
  → Verify explicitly
  → Least privilege access
  → Continuous validation

COMPONENTS:
  ┌────────────────────────────────────┐
  │          ZERO TRUST                │
  │                                    │
  │  Identity: Strong auth, MFA       │
  │  Device: Compliance, health       │
  │  Network: Microsegmentation       │
  │  Application: Proxy-based access  │
  │  Data: Classification, encryption │
  │  Monitoring: Continuous logging   │
  └────────────────────────────────────┘

CLOUD IMPLEMENTATION:
  AWS:
  → AWS Verified Access (app-level proxy)
  → Security Groups (microsegmentation)
  → VPC endpoints (private access)
  → IAM conditions (context-based)
  
  Azure:
  → Conditional Access (identity-based)
  → Azure AD App Proxy
  → Private Link
  → NSG + ASG (microsegmentation)
  
  GCP:
  → BeyondCorp Enterprise
  → Identity-Aware Proxy (IAP)
  → VPC Service Controls
  → Context-Aware Access

NETWORK FLOW:
  Traditional: User → VPN → Full network access
  Zero Trust:  User → Identity Verification → 
               Device Check → Policy → 
               Only authorized app access
```

---

## 5. Assessment

```
SEGMENTATION ASSESSMENT:

QUESTIONS TO ASK:
  → Can workloads in different tiers communicate?
  → Can dev environment reach production?
  → Can all instances reach the internet?
  → Are database ports accessible from web tier?
  → Is east-west traffic monitored?
  → Are there overly permissive security groups?

TESTING METHODS:
  → Port scanning between segments
  → Traffic flow analysis via flow logs
  → Security group rule review
  → Attempt lateral movement in pentest
  → Verify deny rules are enforced

COMMON FINDINGS:
  Finding                          | Risk
  Flat network (no segmentation)   | Critical
  Security groups allow all ports  | High
  Dev can access prod databases    | High
  No east-west traffic monitoring  | Medium
  Default allow between subnets    | High
  No microsegmentation             | Medium

CHECKLIST:
  [ ] Environment isolation (prod/dev/staging)
  [ ] Tier-based segmentation (web/app/data)
  [ ] Security groups restrict port access
  [ ] No unnecessary cross-tier communication
  [ ] East-west traffic monitored
  [ ] Microsegmentation for sensitive workloads
  [ ] Zero trust principles applied
  [ ] Segmentation tested regularly
  [ ] Network policies documented
  [ ] Flow logs analyzed for violations
```

---

## Summary Table

| Level | Mechanism | Isolation Strength | Effort |
|-------|-----------|-------------------|--------|
| Account | Separate accounts | Strongest | High |
| VPC/VNet | Network isolation | Strong | Medium |
| Subnet | Subnet separation | Medium | Low |
| Security Group | Per-instance rules | Medium | Low |
| Microsegmentation | Per-workload | Strong | High |
| API Controls | Service perimeter | Strong | Medium |

---

## Revision Questions

1. What are the five levels of cloud network segmentation?
2. How do AWS Security Groups and NACLs complement each other?
3. What is microsegmentation and why is it important?
4. How does zero trust differ from traditional perimeter security?
5. What should a segmentation assessment test?

---

*Previous: [01-virtual-network-design.md](01-virtual-network-design.md) | Next: [03-firewall-rules.md](03-firewall-rules.md)*

---

*[Back to README](../README.md)*
