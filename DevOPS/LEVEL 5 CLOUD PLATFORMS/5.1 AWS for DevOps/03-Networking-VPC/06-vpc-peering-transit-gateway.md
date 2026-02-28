# Chapter 3.6: VPC Peering & Transit Gateway

## Overview

**VPC Peering** connects two VPCs directly for private communication. **Transit Gateway** acts as a central hub connecting multiple VPCs, on-premises networks, and VPNs. Choosing between them depends on scale.

---

## VPC Peering

```
┌─────────────────┐          ┌─────────────────┐
│  VPC-A          │          │  VPC-B          │
│  10.0.0.0/16    │          │  172.16.0.0/16  │
│                 │  pcx-xxx │                 │
│  Route Table:   │◄────────►│  Route Table:   │
│  172.16.0.0/16  │  Peering │  10.0.0.0/16    │
│   → pcx-xxx    │  Conn.   │   → pcx-xxx    │
│                 │          │                 │
│  EC2: 10.0.1.5  │          │  EC2: 172.16.1.8│
└─────────────────┘          └─────────────────┘

    ✓ Private IP communication
    ✓ No bandwidth bottleneck (uses AWS backbone)
    ✓ Cross-account and cross-region supported
    ✗ Non-transitive (A↔B, B↔C does NOT mean A↔C)
    ✗ No overlapping CIDRs
```

### VPC Peering Key Rules

| Rule | Details |
|------|---------|
| **Non-transitive** | A↔B and B↔C does NOT allow A↔C. Must create direct peering. |
| **No overlapping CIDRs** | Both VPCs must have non-overlapping IP ranges |
| **Cross-region** | Supported (inter-region peering) |
| **Cross-account** | Supported (requester-accepter model) |
| **Max per VPC** | 125 peering connections |
| **DNS resolution** | Can enable private DNS resolution across peered VPCs |

### The Non-Transitive Problem

```
          A ◄──► B ◄──► C

    A can talk to B    ✓
    B can talk to C    ✓
    A can talk to C    ✗  ◄── NOT transitive!

    For A↔C, you need a DIRECT peering connection:

          A ◄──► B ◄──► C
          │               │
          └──────────────┘
               Direct

    With N VPCs, full mesh = N(N-1)/2 connections:
     5 VPCs  →  10 connections
    10 VPCs  →  45 connections
    50 VPCs  → 1,225 connections  ◄── Unmanageable!

    Solution: Transit Gateway
```

### VPC Peering CLI

```bash
# Create peering connection (from VPC-A to VPC-B)
aws ec2 create-vpc-peering-connection \
  --vpc-id vpc-aaa \
  --peer-vpc-id vpc-bbb \
  --peer-region us-west-2 \
  --peer-owner-id 123456789012

# Accept peering (from VPC-B's account/region)
aws ec2 accept-vpc-peering-connection \
  --vpc-peering-connection-id pcx-xxx

# Add routes in BOTH VPCs
# In VPC-A route table:
aws ec2 create-route \
  --route-table-id rtb-vpc-a \
  --destination-cidr-block 172.16.0.0/16 \
  --vpc-peering-connection-id pcx-xxx

# In VPC-B route table:
aws ec2 create-route \
  --route-table-id rtb-vpc-b \
  --destination-cidr-block 10.0.0.0/16 \
  --vpc-peering-connection-id pcx-xxx

# Enable DNS resolution for peering
aws ec2 modify-vpc-peering-connection-options \
  --vpc-peering-connection-id pcx-xxx \
  --requester-peering-connection-options AllowDnsResolutionFromRemoteVpc=true \
  --accepter-peering-connection-options AllowDnsResolutionFromRemoteVpc=true
```

---

## Transit Gateway (TGW)

```
┌───────────────────────────────────────────────────────────────┐
│                   Transit Gateway (Hub)                       │
│                       tgw-xxx                                │
│                                                               │
│         ┌──────────┬──────────┬──────────┐                   │
│         │          │          │          │                    │
│    ┌────▼───┐ ┌────▼───┐ ┌───▼────┐ ┌───▼──────┐           │
│    │ VPC-A  │ │ VPC-B  │ │ VPC-C  │ │ VPC-D    │           │
│    │10.0/16 │ │10.1/16 │ │10.2/16 │ │10.3/16   │           │
│    └────────┘ └────────┘ └────────┘ └──────────┘           │
│                                         │                    │
│         ┌──────────┐          ┌─────────▼──────────┐        │
│         │ VPN      │          │ Direct Connect     │        │
│         │ On-Prem  │          │ Data Center        │        │
│         └──────────┘          └────────────────────┘        │
│                                                               │
│  ✓ Hub-and-spoke: all VPCs connect to TGW only              │
│  ✓ Transitive routing out of the box                         │
│  ✓ Scales to thousands of VPCs                               │
│  ✓ Cross-region peering (TGW↔TGW)                           │
│  ✓ Centralized routing policies via route tables             │
└───────────────────────────────────────────────────────────────┘
```

### Transit Gateway Key Features

| Feature | Details |
|---------|---------|
| **Transitive** | All attached VPCs can communicate (if route tables allow) |
| **Scale** | Up to 5,000 attachments per TGW |
| **Bandwidth** | Up to 50 Gbps per AZ per VPC attachment |
| **Cross-region** | TGW peering supported |
| **Route tables** | Separate TGW route tables for segmentation |
| **Attachments** | VPCs, VPNs, Direct Connect, peered TGWs |
| **Cost** | ~$0.05/hr per attachment + $0.02/GB data |

### TGW Route Table Segmentation

```
┌──── TGW Route Table: Production ──────────────┐
│                                                 │
│  10.0.0.0/16 → VPC-Prod-A                     │
│  10.1.0.0/16 → VPC-Prod-B                     │
│  10.2.0.0/16 → VPC-Shared-Services            │
│                                                 │
│  Associated: VPC-Prod-A, VPC-Prod-B            │
└─────────────────────────────────────────────────┘

┌──── TGW Route Table: Development ─────────────┐
│                                                 │
│  10.3.0.0/16 → VPC-Dev-A                      │
│  10.4.0.0/16 → VPC-Dev-B                      │
│  10.2.0.0/16 → VPC-Shared-Services            │
│                                                 │
│  Associated: VPC-Dev-A, VPC-Dev-B              │
└─────────────────────────────────────────────────┘

  Prod VPCs can reach each other + Shared Services
  Dev VPCs can reach each other + Shared Services
  Prod CANNOT reach Dev (different TGW route tables)
```

### TGW CLI

```bash
# Create Transit Gateway
aws ec2 create-transit-gateway \
  --description "Central Hub TGW" \
  --options AutoAcceptSharedAttachments=enable,DefaultRouteTableAssociation=enable

# Attach VPC to Transit Gateway
aws ec2 create-transit-gateway-vpc-attachment \
  --transit-gateway-id tgw-xxx \
  --vpc-id vpc-aaa \
  --subnet-ids subnet-1a subnet-1b

# Add route in VPC route table pointing to TGW
aws ec2 create-route \
  --route-table-id rtb-vpc-a \
  --destination-cidr-block 10.1.0.0/16 \
  --transit-gateway-id tgw-xxx

# Create TGW route table for segmentation
aws ec2 create-transit-gateway-route-table \
  --transit-gateway-id tgw-xxx \
  --tag-specifications 'ResourceType=transit-gateway-route-table,Tags=[{Key=Name,Value=prod-rt}]'

# Associate attachment with specific route table
aws ec2 associate-transit-gateway-route-table \
  --transit-gateway-route-table-id tgw-rtb-xxx \
  --transit-gateway-attachment-id tgw-attach-xxx
```

---

## VPC Peering vs Transit Gateway

| Feature | VPC Peering | Transit Gateway |
|---------|------------|-----------------|
| **Topology** | Point-to-point | Hub-and-spoke |
| **Transitivity** | Non-transitive | Transitive |
| **Scale** | 125 per VPC, full mesh complex | 5,000 attachments |
| **Bandwidth** | No limit (AWS backbone) | 50 Gbps per AZ |
| **Cost** | Free (data transfer only) | Hourly + data processing |
| **Cross-region** | Supported | TGW peering supported |
| **VPN/DX** | Not supported | Supported |
| **Route tables** | VPC route tables only | Dedicated TGW route tables |
| **Best for** | 2-3 VPCs | 4+ VPCs, hybrid architectures |

### Decision Flow

```
How many VPCs need to communicate?
│
├─ 2-3 VPCs → VPC Peering (simpler, free)
│
├─ 4+ VPCs → Transit Gateway (hub-spoke, scalable)
│
└─ Need VPN/Direct Connect? → Transit Gateway (mandatory)
```

---

## Additional Connectivity Options

| Service | Use Case |
|---------|----------|
| **VPC Endpoints** | Private access to AWS services (S3, DynamoDB, etc.) |
| **AWS PrivateLink** | Expose service to other VPCs privately (no peering needed) |
| **Site-to-Site VPN** | Encrypted tunnel to on-premises over internet |
| **Direct Connect** | Dedicated private line to AWS (1/10/100 Gbps) |
| **CloudFront** | CDN for edge caching and DDoS protection |

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Peered VPCs can't communicate | Missing routes in one/both VPCs | Add routes in both VPC route tables |
| Overlapping CIDR error | Both VPCs use same IP range | Re-design CIDR plan; can't peer overlapping VPCs |
| TGW attachment stuck "pending" | TGW not set to auto-accept | Accept attachment or enable auto-accept |
| Prod/Dev traffic leaking | Single TGW route table | Create separate TGW route tables for segmentation |
| Cross-region peering slow | Data transfer latency | Expected behavior; consider region placement |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **VPC Peering** | Point-to-point, non-transitive, free, up to 125 per VPC |
| **Transit Gateway** | Hub-and-spoke, transitive, scalable to 5,000 attachments |
| **Non-Transitive** | A↔B + B↔C ≠ A↔C. Must peer directly or use TGW. |
| **TGW Segmentation** | Use separate TGW route tables to isolate prod/dev |
| **PrivateLink** | Share services across VPCs without peering |
| **Hybrid** | TGW supports VPN + Direct Connect attachments |

---

## Quick Revision Questions

1. **What does "non-transitive" mean for VPC peering?**
   > If VPC-A peers with VPC-B and VPC-B peers with VPC-C, VPC-A cannot reach VPC-C through VPC-B. A direct peering is required.

2. **When should you use Transit Gateway instead of VPC Peering?**
   > When connecting 4+ VPCs, or when you need VPN/Direct Connect integration, or when you want centralized routing and segmentation.

3. **How do you isolate production from dev traffic on a Transit Gateway?**
   > Create separate TGW route tables for prod and dev, and associate the respective VPC attachments with each.

4. **Can you peer VPCs with overlapping CIDRs?**
   > No. VPC peering requires non-overlapping IP ranges.

5. **What is AWS PrivateLink used for?**
   > To expose a service in one VPC to consumers in other VPCs privately, without requiring peering, public IPs, or route table changes.

---

[← Previous: 3.5 Security Groups vs NACLs](05-security-groups-vs-nacls.md)

[← Back to README](../README.md)
