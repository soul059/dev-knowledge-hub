# Chapter 3.4: Internet Gateway & NAT Gateway

## Overview

An **Internet Gateway (IGW)** enables resources in public subnets to communicate with the internet. A **NAT Gateway** allows resources in private subnets to initiate outbound internet connections while remaining unreachable from the outside.

---

## Internet Gateway (IGW)

```
          ☁ Internet
              │
              ▼
    ┌─────────────────┐
    │ Internet Gateway │  ◄── 1 per VPC, horizontally scaled
    │     igw-xxx      │      by AWS, no bandwidth limit
    └────────┬────────┘
             │
    ┌────────▼────────┐
    │  PUBLIC SUBNET   │
    │  Route Table:    │
    │  0.0.0.0/0→IGW   │
    │                  │
    │  ┌────────────┐  │
    │  │ EC2 + EIP  │  │  ◄── Instance has Elastic IP (static)
    │  │ or         │  │      or auto-assigned public IP
    │  │ Public IP  │  │
    │  └────────────┘  │
    └──────────────────┘

    IGW performs 1:1 NAT between private IP ↔ public IP
```

### IGW Key Properties

- **One per VPC** — cannot attach multiple IGWs
- **Horizontally scaled** — no bandwidth constraints, no single point of failure
- **Free** — no hourly charges or data processing fees
- **Does two things:** (1) provides a route target, (2) performs NAT for instances with public IPs

### Setting Up IGW

```bash
# Create Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=prod-igw}]'

# Attach to VPC
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-xxx \
  --vpc-id vpc-xxx

# Add route to public route table
aws ec2 create-route \
  --route-table-id rtb-public \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-xxx
```

### Requirements for Internet Access (Public Subnet)

```
Checklist for EC2 Internet Access:
┌─────────────────────────────────────────────────┐
│  ☐ VPC has an Internet Gateway attached         │
│  ☐ Subnet's route table: 0.0.0.0/0 → IGW      │
│  ☐ Instance has public IP or Elastic IP         │
│  ☐ Security Group allows outbound traffic       │
│  ☐ Network ACL allows inbound + outbound        │
└─────────────────────────────────────────────────┘
  ALL five must be true!
```

---

## NAT Gateway

```
          ☁ Internet
              │
    ┌─────────▼───────┐
    │ Internet Gateway │
    └────────┬────────┘
             │
    ┌────────▼─────────────────────┐
    │  PUBLIC SUBNET               │
    │                              │
    │  ┌────────────────────────┐  │
    │  │  NAT Gateway           │  │  ◄── AWS managed, in public subnet
    │  │  nat-xxx               │  │      Elastic IP attached
    │  │  EIP: 52.1.2.3        │  │      Up to 100 Gbps
    │  └──────────┬─────────────┘  │
    └─────────────┼────────────────┘
                  │
    ┌─────────────▼────────────────┐
    │  PRIVATE SUBNET              │
    │  Route: 0.0.0.0/0 → nat-xxx │
    │                              │
    │  ┌──────────┐ ┌──────────┐  │
    │  │ App EC2  │ │ App EC2  │  │  ◄── Can reach internet (outbound)
    │  │ 10.0.3.5 │ │ 10.0.3.6│  │      Cannot be reached FROM internet
    │  └──────────┘ └──────────┘  │
    └──────────────────────────────┘

    Private → NAT GW → IGW → Internet  (outbound only)
```

### NAT Gateway Key Properties

| Property | Details |
|----------|---------|
| **Managed by** | AWS — no patching, auto-scaling |
| **Placement** | Must be in a public subnet |
| **Elastic IP** | Required — provides static outbound IP |
| **Bandwidth** | 5 Gbps, auto-scales up to 100 Gbps |
| **Availability** | Single AZ. Deploy one per AZ for HA. |
| **Cost** | ~$0.045/hr + $0.045/GB processed |
| **Protocols** | TCP, UDP, ICMP |

### Creating NAT Gateway

```bash
# Allocate Elastic IP for NAT Gateway
ALLOC_ID=$(aws ec2 allocate-address \
  --domain vpc \
  --query 'AllocationId' --output text)

# Create NAT Gateway in public subnet
aws ec2 create-nat-gateway \
  --subnet-id subnet-public-1a \
  --allocation-id $ALLOC_ID \
  --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=nat-1a}]'

# Wait for NAT Gateway to become available
aws ec2 wait nat-gateway-available \
  --nat-gateway-ids nat-xxx

# Add route in private subnet's route table
aws ec2 create-route \
  --route-table-id rtb-private \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id nat-xxx
```

---

## High Availability NAT Gateway

```
┌──────────────────── Production VPC ────────────────────────────┐
│                                                                 │
│  ┌─── AZ-1a ─────────────────┐  ┌─── AZ-1b ─────────────────┐ │
│  │                            │  │                            │ │
│  │  Public Subnet             │  │  Public Subnet             │ │
│  │  ┌──────────────────────┐  │  │  ┌──────────────────────┐  │ │
│  │  │ NAT-GW-1a (EIP-1)   │  │  │  │ NAT-GW-1b (EIP-2)   │  │ │
│  │  └──────────┬───────────┘  │  │  └──────────┬───────────┘  │ │
│  │             │              │  │             │              │ │
│  │  Private Subnet            │  │  Private Subnet            │ │
│  │  ┌──────────▼───────────┐  │  │  ┌──────────▼───────────┐  │ │
│  │  │ RT: 0.0.0.0/0       │  │  │  │ RT: 0.0.0.0/0       │  │ │
│  │  │    → NAT-GW-1a      │  │  │  │    → NAT-GW-1b      │  │ │
│  │  │ App Servers          │  │  │  │ App Servers          │  │ │
│  │  └──────────────────────┘  │  │  └──────────────────────┘  │ │
│  └────────────────────────────┘  └────────────────────────────┘ │
│                                                                 │
│  ⚠ Each AZ has its own NAT GW + route table                   │
│  ⚠ If AZ-1a fails, AZ-1b NAT GW still works                  │
│  ⚠ Cross-AZ data transfer avoided (saves cost)                │
└─────────────────────────────────────────────────────────────────┘
```

> **Cost Trade-off:** Each NAT Gateway costs ~$32/month. For dev environments, a single NAT GW is acceptable. For production, deploy one per AZ.

---

## NAT Gateway vs NAT Instance

| Feature | NAT Gateway | NAT Instance |
|---------|------------|--------------|
| Managed by | AWS | You (EC2) |
| Bandwidth | Up to 100 Gbps | Depends on instance type |
| Availability | HA within AZ | You configure HA |
| Maintenance | None | OS patching required |
| Security Groups | Not supported | Supported |
| Bastion host | Cannot use as | Can double as bastion |
| Cost | Higher (managed) | Lower (t3.micro) |
| Port forwarding | Not supported | Supported |
| **Recommendation** | **Production** | **Dev/learning only** |

---

## Egress-Only Internet Gateway (IPv6)

```
┌───────────────────────────────────────────────┐
│  For IPv6, use Egress-Only Internet Gateway   │
│  (equivalent of NAT GW for IPv6)              │
│                                               │
│  IPv6 addresses are globally unique, so:      │
│  - No NAT needed (no private/public concept)  │
│  - Use Egress-Only IGW to allow outbound but  │
│    block unsolicited inbound                  │
│                                               │
│  Route: ::/0 → eigw-xxx                       │
└───────────────────────────────────────────────┘
```

```bash
# Create egress-only IGW
aws ec2 create-egress-only-internet-gateway \
  --vpc-id vpc-xxx

# Add IPv6 route
aws ec2 create-route \
  --route-table-id rtb-private \
  --destination-ipv6-cidr-block ::/0 \
  --egress-only-internet-gateway-id eigw-xxx
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Private instance can't reach internet | NAT GW in wrong subnet or missing route | Ensure NAT GW is in PUBLIC subnet; route 0.0.0.0/0 → nat-xxx |
| NAT GW shows "failed" state | Elastic IP already in use or subnet issue | Release EIP, check subnet is public, retry |
| Intermittent connectivity | Single NAT GW, AZ outage | Deploy NAT GW in each AZ |
| High NAT costs | Large data transfer through NAT | Use VPC Endpoints for AWS services (S3, DynamoDB) |
| "ErrorPortAllocation" | >55,000 simultaneous connections to one destination | Spread across multiple IPs or destinations |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Internet Gateway** | Free, AWS-managed, one per VPC, enables public internet access |
| **NAT Gateway** | Managed NAT for private subnets, requires EIP, costs ~$32/mo/AZ |
| **HA NAT** | One NAT GW per AZ with separate route tables |
| **NAT Instance** | EC2-based NAT, cheaper but you manage it, dev/test only |
| **Egress-Only IGW** | NAT equivalent for IPv6, blocks inbound |
| **Checklist** | IGW attached + route + public IP + SG + NACL = internet access |

---

## Quick Revision Questions

1. **What is the cost model for an Internet Gateway?**
   > Free — no hourly or data processing charges.

2. **Why must a NAT Gateway be in a public subnet?**
   > It needs a route to the IGW to forward traffic to the internet. Only public subnets have that route.

3. **How do you achieve HA for NAT Gateways?**
   > Deploy one NAT Gateway per Availability Zone, each with its own route table pointing to it.

4. **What is the main cost-saving tip for NAT Gateway usage?**
   > Use VPC Endpoints for AWS services (S3, DynamoDB) so that traffic doesn't traverse the NAT Gateway.

5. **What is an Egress-Only Internet Gateway?**
   > An IPv6-only gateway that allows outbound internet traffic but blocks unsolicited inbound connections.

6. **Can you attach a Security Group to a NAT Gateway?**
   > No. Security Groups can only be attached to NAT Instances (EC2-based), not NAT Gateways.

---

[← Previous: 3.3 Route Tables](03-route-tables.md) | [Next: 3.5 Security Groups vs NACLs →](05-security-groups-vs-nacls.md)

[← Back to README](../README.md)
