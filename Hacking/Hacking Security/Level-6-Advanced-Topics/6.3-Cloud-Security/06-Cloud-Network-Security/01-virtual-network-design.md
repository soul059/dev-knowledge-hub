# Unit 6: Cloud Network Security — Topic 1: Virtual Network Design

## Overview

**Virtual network design** in cloud environments establishes the foundation for secure communication between cloud resources. Proper VPC/VNet architecture ensures network segmentation, isolation, and controlled traffic flow. A well-designed cloud network minimizes attack surface, limits lateral movement, and enables defense in depth.

---

## 1. Virtual Network Concepts

```
CLOUD VIRTUAL NETWORKS:

  AWS VPC:
  → Regional resource
  → Private IP address space (CIDR)
  → Subnets are AZ-specific
  → Internet Gateway for public access
  → NAT Gateway for outbound-only
  → Route tables per subnet

  Azure VNet:
  → Regional resource
  → Address space (one or more CIDRs)
  → Subnets within VNet
  → Integrated DNS
  → VNet peering for connectivity
  → Service endpoints for PaaS

  GCP VPC:
  → Global resource (unique!)
  → Subnets are regional
  → Automatic internal routing
  → No Internet Gateway concept
  → Routes managed centrally
  → Shared VPC for multi-project

COMPARISON:
  Feature        | AWS VPC      | Azure VNet   | GCP VPC
  Scope          | Regional     | Regional     | Global
  Subnets        | AZ-specific  | VNet-specific| Regional
  Default        | Default VPC  | No default   | Default VPC
  Peering        | Regional/Cross| Regional/Global| Global
  Transit        | Transit GW   | Virtual WAN  | Cloud Router
```

---

## 2. Secure Network Architecture

```
HUB-AND-SPOKE TOPOLOGY:

  ┌──────────────────────────────┐
  │         HUB VPC/VNet          │
  │                              │
  │  ┌─────────┐  ┌──────────┐  │
  │  │Firewall │  │VPN/ExpRoute│ │
  │  │(Inspect)│  │(On-Prem)  │  │
  │  └─────────┘  └──────────┘  │
  │  ┌─────────┐  ┌──────────┐  │
  │  │Shared   │  │DNS/NTP   │  │
  │  │Services │  │           │  │
  │  └─────────┘  └──────────┘  │
  └──────┬─────────┬─────────┬──┘
         │         │         │
    ┌────▼───┐ ┌───▼────┐ ┌──▼─────┐
    │Spoke 1 │ │Spoke 2 │ │Spoke 3 │
    │Prod Web│ │Prod App│ │Dev/Test│
    └────────┘ └────────┘ └────────┘

BENEFITS:
  → Centralized security inspection
  → Shared services (DNS, NTP, monitoring)
  → Controlled inter-spoke traffic
  → On-premises connectivity through hub
  → Simplified management

MULTI-TIER ARCHITECTURE:
  ┌─────────────────────────────────────┐
  │               VPC/VNet              │
  │                                     │
  │  ┌──────────────────────────────┐   │
  │  │ PUBLIC SUBNET (DMZ)          │   │
  │  │ Load Balancer, WAF, Bastion  │   │
  │  └──────────┬───────────────────┘   │
  │             │                       │
  │  ┌──────────▼───────────────────┐   │
  │  │ PRIVATE SUBNET (APP TIER)    │   │
  │  │ Application servers          │   │
  │  └──────────┬───────────────────┘   │
  │             │                       │
  │  ┌──────────▼───────────────────┐   │
  │  │ PRIVATE SUBNET (DATA TIER)   │   │
  │  │ Databases, caches            │   │
  │  └──────────────────────────────┘   │
  │                                     │
  │  ┌──────────────────────────────┐   │
  │  │ MANAGEMENT SUBNET            │   │
  │  │ Bastion hosts, monitoring    │   │
  │  └──────────────────────────────┘   │
  └─────────────────────────────────────┘

IP ADDRESS PLANNING:
  VPC CIDR: 10.0.0.0/16 (65,536 IPs)
  
  Subnets:
  10.0.1.0/24   Public Subnet AZ-a    (251 usable)
  10.0.2.0/24   Public Subnet AZ-b    (251 usable)
  10.0.10.0/24  App Subnet AZ-a       (251 usable)
  10.0.11.0/24  App Subnet AZ-b       (251 usable)
  10.0.20.0/24  Data Subnet AZ-a      (251 usable)
  10.0.21.0/24  Data Subnet AZ-b      (251 usable)
  10.0.100.0/24 Management Subnet     (251 usable)
  
  Rules:
  → Non-overlapping CIDRs
  → Room for growth
  → Consistent across environments
  → RFC 1918 private ranges
  → Document all allocations
```

---

## 3. Connectivity Options

```
HYBRID CONNECTIVITY:

VPN (Virtual Private Network):
  → Encrypted tunnel over internet
  → IPsec/IKEv2
  → Lower cost, variable latency
  → Suitable for non-critical workloads
  → Redundant tunnels recommended

  AWS: Site-to-Site VPN
  Azure: VPN Gateway
  GCP: Cloud VPN

DEDICATED CONNECTION:
  → Private, dedicated fiber link
  → Consistent latency
  → Higher bandwidth (1-100 Gbps)
  → No internet traversal
  → Higher cost

  AWS: Direct Connect
  Azure: ExpressRoute
  GCP: Cloud Interconnect

SERVICE ENDPOINTS / PRIVATE LINK:
  → Private access to PaaS services
  → Traffic stays on cloud backbone
  → No public IP needed
  → Reduces data exfiltration risk

  AWS: VPC Endpoints (Interface/Gateway)
  Azure: Private Endpoints / Service Endpoints
  GCP: Private Google Access / Private Service Connect

  # AWS VPC Endpoint for S3
  aws ec2 create-vpc-endpoint \
    --vpc-id vpc-123 \
    --service-name com.amazonaws.us-east-1.s3 \
    --route-table-ids rtb-123

  # Azure Private Endpoint
  az network private-endpoint create \
    --name myEndpoint \
    --resource-group myRG \
    --vnet-name myVNet \
    --subnet mySubnet \
    --private-connection-resource-id /subscriptions/.../storageAccounts/myStorage \
    --connection-name myConnection \
    --group-id blob
```

---

## 4. Network Security Controls

```
SECURITY CONTROLS:

TRAFFIC FILTERING:
  AWS: Security Groups (stateful) + NACLs (stateless)
  Azure: NSGs (stateful) + Azure Firewall
  GCP: VPC Firewall Rules (stateful) + Cloud Armor

TRAFFIC INSPECTION:
  → Cloud-native firewalls (Layer 3-7)
  → Third-party NVAs (Network Virtual Appliances)
  → IDS/IPS capabilities
  → TLS inspection
  → Deep packet inspection

DNS SECURITY:
  → Use private DNS zones
  → DNS logging enabled
  → Block DNS exfiltration
  → DNSSEC where supported
  → Conditional forwarding for hybrid

FLOW LOGGING:
  AWS: VPC Flow Logs
  Azure: NSG Flow Logs
  GCP: VPC Flow Logs
  
  → Capture IP traffic information
  → Source/destination IP and port
  → Protocol and action (allow/deny)
  → Bytes transferred
  → Essential for security monitoring

BEST PRACTICES:
  [ ] Use hub-and-spoke architecture
  [ ] Separate environments (prod/dev/staging)
  [ ] No public IPs on compute (use NAT/LB)
  [ ] Private access for PaaS services
  [ ] Enable flow logs on all subnets
  [ ] Centralized egress through firewall
  [ ] DNS logging and monitoring
  [ ] Consistent IP addressing scheme
  [ ] Document all network flows
  [ ] Regular network architecture review
```

---

## 5. Assessment

```
NETWORK DESIGN ASSESSMENT:

REVIEW POINTS:
  → Network segmentation between tiers
  → Public vs private subnet usage
  → Firewall/filtering rules
  → Internet exposure (public IPs, LBs)
  → Hybrid connectivity security
  → DNS configuration
  → Flow log coverage
  → Encryption in transit

COMMON FINDINGS:
  Finding                          | Risk     | Fix
  No network segmentation          | Critical | Multi-tier subnets
  Public IPs on all instances      | High     | NAT Gateway/LB
  No flow logs                     | High     | Enable on all subnets
  Direct internet access from DBs  | Critical | Private subnets only
  No centralized firewall          | Medium   | Hub firewall
  Overlapping CIDRs                | Medium   | IP address planning
  No private access for PaaS       | Medium   | Endpoints/Private Link
  Default VPC in use               | Medium   | Custom VPC

TOOLS:
  → Cloud provider network diagrams
  → Tufin/AlgoSec for policy analysis
  → Nmap for port scanning
  → Prisma Cloud for network visibility
  → Cloud-native flow log analysis
```

---

## Summary Table

| Component | AWS | Azure | GCP |
|-----------|-----|-------|-----|
| Virtual Network | VPC (regional) | VNet (regional) | VPC (global) |
| Firewall | Security Groups + NACLs | NSG + Azure Firewall | Firewall Rules |
| Private Access | VPC Endpoints | Private Endpoints | Private Google Access |
| Dedicated Link | Direct Connect | ExpressRoute | Cloud Interconnect |
| Flow Logs | VPC Flow Logs | NSG Flow Logs | VPC Flow Logs |

---

## Revision Questions

1. How do virtual networks differ across AWS, Azure, and GCP?
2. What are the benefits of a hub-and-spoke network architecture?
3. Why should PaaS services use private endpoints instead of public access?
4. What should be included in a cloud IP address planning strategy?
5. What network design issues should a security assessment look for?

---

*Previous: None (First topic in this unit) | Next: [02-network-segmentation.md](02-network-segmentation.md)*

---

*[Back to README](../README.md)*
