# Chapter 3.1: VPC Concepts

## Overview

A **Virtual Private Cloud (VPC)** is your isolated network in AWS. It's the foundation of all networking — every resource (EC2, RDS, Lambda in VPC mode) lives inside a VPC. Understanding VPC is critical for DevOps engineers building secure, scalable architectures.

---

## VPC Architecture

```
┌────────────────────────── Region: us-east-1 ─────────────────────┐
│                                                                    │
│  ┌────────────────── VPC: 10.0.0.0/16 ──────────────────────┐   │
│  │  65,536 IP addresses                                      │   │
│  │                                                            │   │
│  │  ┌──── AZ: us-east-1a ────┐  ┌──── AZ: us-east-1b ────┐ │   │
│  │  │                        │  │                         │ │   │
│  │  │ ┌─ Public Subnet ───┐ │  │ ┌─ Public Subnet ───┐  │ │   │
│  │  │ │  10.0.1.0/24      │ │  │ │  10.0.2.0/24      │  │ │   │
│  │  │ │  (256 IPs)        │ │  │ │  (256 IPs)        │  │ │   │
│  │  │ │  ALB, NAT GW,     │ │  │ │  ALB, Bastion     │  │ │   │
│  │  │ │  Bastion           │ │  │ │                   │  │ │   │
│  │  │ └───────────────────┘ │  │ └───────────────────┘  │ │   │
│  │  │                        │  │                         │ │   │
│  │  │ ┌─ Private Subnet ──┐ │  │ ┌─ Private Subnet ──┐  │ │   │
│  │  │ │  10.0.3.0/24      │ │  │ │  10.0.4.0/24      │  │ │   │
│  │  │ │  (256 IPs)        │ │  │ │  (256 IPs)        │  │ │   │
│  │  │ │  EC2 App Servers   │ │  │ │  EC2 App Servers  │  │ │   │
│  │  │ └───────────────────┘ │  │ └───────────────────┘  │ │   │
│  │  │                        │  │                         │ │   │
│  │  │ ┌─ Data Subnet ─────┐ │  │ ┌─ Data Subnet ─────┐  │ │   │
│  │  │ │  10.0.5.0/24      │ │  │ │  10.0.6.0/24      │  │ │   │
│  │  │ │  RDS, ElastiCache  │ │  │ │  RDS Standby      │  │ │   │
│  │  │ └───────────────────┘ │  │ └───────────────────┘  │ │   │
│  │  └────────────────────────┘  └─────────────────────────┘ │   │
│  │                                                            │   │
│  │  ┌─ Internet GW ─┐  ┌─ NAT GW ─┐  ┌─ VPC Endpoint ─┐   │   │
│  │  │  (to Internet) │  │ (outbound)│  │  (to AWS svc)  │   │   │
│  │  └────────────────┘  └──────────┘  └────────────────┘   │   │
│  └────────────────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────────────────┘
```

---

## CIDR Notation Quick Reference

```
┌──────────────────────────────────────────────────────────┐
│              CIDR NOTATION                               │
│                                                          │
│  CIDR          Subnet Mask       IPs    Usable IPs      │
│  ──────────── ─────────────────  ─────  ──────────      │
│  /16          255.255.0.0        65,536  65,531         │
│  /17          255.255.128.0      32,768  32,763         │
│  /18          255.255.192.0      16,384  16,379         │
│  /19          255.255.224.0       8,192   8,187         │
│  /20          255.255.240.0       4,096   4,091         │
│  /21          255.255.248.0       2,048   2,043         │
│  /22          255.255.252.0       1,024   1,019         │
│  /23          255.255.254.0         512     507         │
│  /24          255.255.255.0         256     251         │
│  /25          255.255.255.128       128     123         │
│  /26          255.255.255.192        64      59         │
│  /27          255.255.255.224        32      27         │
│  /28          255.255.255.240        16      11         │
│                                                          │
│  AWS reserves 5 IPs per subnet:                         │
│  .0 = Network    .1 = VPC Router   .2 = DNS             │
│  .3 = Future     .255 = Broadcast                       │
│                                                          │
│  VPC CIDR: /16 (largest) to /28 (smallest)              │
│  Recommended: /16 for production VPCs                   │
└──────────────────────────────────────────────────────────┘
```

---

## Creating a VPC

```bash
# Create VPC
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=prod-vpc}]'

# Enable DNS resolution and hostnames
aws ec2 modify-vpc-attribute --vpc-id vpc-xxx --enable-dns-support '{"Value": true}'
aws ec2 modify-vpc-attribute --vpc-id vpc-xxx --enable-dns-hostnames '{"Value": true}'
```

---

## VPC Components Overview

```
┌──────────────────────────────────────────────────────────────┐
│                VPC COMPONENT MAP                             │
│                                                              │
│  Internet ◄──► Internet Gateway ◄──► Public Subnet          │
│                                       │                      │
│                                       Route Table            │
│                                       ├── 0.0.0.0/0 → IGW   │
│                                       └── 10.0.0.0/16 → local│
│                                                              │
│                NAT Gateway (in public) ◄──► Private Subnet   │
│                                              │               │
│                                              Route Table     │
│                                              ├── 0.0.0.0/0 → NAT│
│                                              └── 10.0.0.0/16 → local│
│                                                              │
│  Security Groups ──► Instance level (stateful)              │
│  NACLs ──────────► Subnet level (stateless)                 │
│  VPC Flow Logs ──► Traffic monitoring                       │
│  VPC Endpoints ──► Private access to AWS services           │
└──────────────────────────────────────────────────────────────┘
```

---

## Default VPC vs Custom VPC

| Feature | Default VPC | Custom VPC |
|---------|-------------|------------|
| Created automatically | ✓ (one per region) | ✗ (you create) |
| CIDR | 172.31.0.0/16 | You choose |
| Subnets | One per AZ (public) | You design |
| Internet Gateway | ✓ (attached) | You create |
| Public IPs | Auto-assigned | You control |
| Use case | Quick testing | Production workloads |

---

## VPC Design Best Practices

```
┌──────────────────────────────────────────────────────────────┐
│            VPC DESIGN BEST PRACTICES                        │
│                                                              │
│  1. Use /16 CIDR for production VPCs                        │
│  2. Plan CIDR ranges to avoid overlap (for peering/VPN)     │
│  3. Use at least 2 AZs (prefer 3)                          │
│  4. Three subnet tiers: Public, Private, Data               │
│  5. Use private subnets for app and data tiers              │
│  6. NAT Gateway in each AZ for HA                          │
│  7. Use VPC Endpoints to avoid NAT costs for AWS services   │
│  8. Enable VPC Flow Logs for monitoring                     │
│  9. Use consistent CIDR scheme across environments          │
│                                                              │
│  Example CIDR scheme:                                       │
│  Dev:  10.1.0.0/16                                          │
│  Stg:  10.2.0.0/16                                          │
│  Prod: 10.3.0.0/16                                          │
│  Shared: 10.0.0.0/16                                        │
└──────────────────────────────────────────────────────────────┘
```

---

## VPC Flow Logs

Monitor network traffic at the VPC, subnet, or ENI level.

```bash
# Enable VPC Flow Logs to CloudWatch
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /vpc/flow-logs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/FlowLogsRole

# Flow Log Record Format:
# version account-id interface-id srcaddr dstaddr srcport dstport protocol packets bytes start end action log-status
# 2 123456789012 eni-xxx 10.0.1.5 10.0.2.10 49152 443 6 25 20000 1579711200 1579711260 ACCEPT OK
```

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **VPC** | Isolated virtual network in a region. CIDR range /16 to /28. |
| **CIDR** | IP range notation. /16 = 65,536 IPs. AWS reserves 5 per subnet. |
| **Subnets** | Subdivision of VPC. One subnet = one AZ. Public or Private. |
| **Default VPC** | Auto-created, 172.31.0.0/16, one public subnet per AZ. |
| **Design** | 3-tier (public/private/data), multi-AZ, consistent CIDR scheme. |
| **Flow Logs** | Network traffic logs at VPC/subnet/ENI level. CloudWatch or S3. |

---

## Quick Revision Questions

1. **What is a VPC and is it global or regional?**
   > A VPC is an isolated virtual network. It is regional — spans all AZs in a region but cannot span regions.

2. **How many IPs does AWS reserve in each subnet?**
   > 5 IPs: .0 (network), .1 (router), .2 (DNS), .3 (future), .255 (broadcast).

3. **Why should VPC CIDR ranges not overlap across environments?**
   > Non-overlapping CIDRs are required for VPC Peering and Transit Gateway connections.

4. **What is the recommended VPC CIDR block size for production?**
   > /16 (65,536 IPs) — provides room for growth and multiple subnet tiers.

5. **What do VPC Flow Logs capture?**
   > Metadata about IP traffic (source/dest IP, ports, protocol, action). NOT packet contents.

6. **What's the difference between the default VPC and a custom VPC?**
   > Default VPC is auto-created with public subnets and internet access. Custom VPCs are designed by you with full control over CIDR, subnets, and routing.

---

[← Previous: 2.6 Lambda](../02-Compute-Services/06-lambda-serverless.md) | [Next: 3.2 Subnets →](02-subnets-public-private.md)

[← Back to README](../README.md)
