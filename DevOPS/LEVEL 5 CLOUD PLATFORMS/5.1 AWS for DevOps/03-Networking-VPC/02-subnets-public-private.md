# Chapter 3.2: Subnets (Public/Private)

## Overview

Subnets are **subdivisions of a VPC's IP range**, each residing in a single AZ. The distinction between public and private subnets is **how their route table handles internet traffic** — this is the most important networking concept in AWS.

---

## Public vs Private Subnets

```
┌──────────────── VPC: 10.0.0.0/16 ─────────────────────────────┐
│                                                                 │
│            ☁ Internet                                          │
│                │                                                │
│         ┌──────▼──────┐                                        │
│         │  Internet   │                                        │
│         │  Gateway    │                                        │
│         └──────┬──────┘                                        │
│                │                                                │
│  ┌─────────────▼────────────────┐                              │
│  │    PUBLIC SUBNET             │                              │
│  │    10.0.1.0/24               │                              │
│  │                              │                              │
│  │  Route Table:                │                              │
│  │  10.0.0.0/16 → local        │                              │
│  │  0.0.0.0/0   → igw-xxx  ◄── Key: routes to IGW = PUBLIC  │
│  │                              │                              │
│  │  Resources: ALB, NAT GW,    │                              │
│  │  Bastion Host, Public EC2   │                              │
│  └──────────────┬───────────────┘                              │
│                 │                                               │
│         ┌───────▼───────┐                                      │
│         │  NAT Gateway  │                                      │
│         └───────┬───────┘                                      │
│                 │                                               │
│  ┌──────────────▼───────────────┐                              │
│  │    PRIVATE SUBNET            │                              │
│  │    10.0.3.0/24               │                              │
│  │                              │                              │
│  │  Route Table:                │                              │
│  │  10.0.0.0/16 → local        │                              │
│  │  0.0.0.0/0   → nat-xxx  ◄── Key: routes to NAT = PRIVATE │
│  │                              │                              │
│  │  Resources: App servers,     │                              │
│  │  databases, backend services │                              │
│  └──────────────────────────────┘                              │
│                                                                 │
│  PUBLIC = Route table has 0.0.0.0/0 → Internet Gateway        │
│  PRIVATE = Route table has 0.0.0.0/0 → NAT Gateway (or none) │
└─────────────────────────────────────────────────────────────────┘
```

### Key Differences

| Feature | Public Subnet | Private Subnet |
|---------|--------------|----------------|
| Internet access (inbound) | ✓ Via IGW | ✗ Not directly |
| Internet access (outbound) | ✓ Via IGW | ✓ Via NAT Gateway |
| Public IP auto-assign | Usually enabled | Usually disabled |
| Route table default route | 0.0.0.0/0 → IGW | 0.0.0.0/0 → NAT GW |
| Use case | Load balancers, bastion | App servers, databases |

---

## Three-Tier Subnet Architecture

```
┌──────────────── Production VPC ────────────────────────────────┐
│                                                                 │
│  ┌──── AZ-1a ────────────────┐  ┌──── AZ-1b ────────────────┐ │
│  │                            │  │                            │ │
│  │  ┌ Public: 10.0.1.0/24 ─┐ │  │  ┌ Public: 10.0.2.0/24 ─┐ │ │
│  │  │  ALB, NAT Gateway    │ │  │  │  ALB                  │ │ │
│  │  └──────────────────────┘ │  │  └──────────────────────┘ │ │
│  │                            │  │                            │ │
│  │  ┌ Private: 10.0.3.0/24 ┐ │  │  ┌ Private: 10.0.4.0/24 ┐ │ │
│  │  │  EC2 App Servers     │ │  │  │  EC2 App Servers      │ │ │
│  │  │  ECS Tasks           │ │  │  │  ECS Tasks            │ │ │
│  │  └──────────────────────┘ │  │  └──────────────────────┘ │ │
│  │                            │  │                            │ │
│  │  ┌ Data: 10.0.5.0/24 ──┐ │  │  ┌ Data: 10.0.6.0/24 ──┐ │ │
│  │  │  RDS Primary         │ │  │  │  RDS Standby         │ │ │
│  │  │  ElastiCache         │ │  │  │  ElastiCache         │ │ │
│  │  └──────────────────────┘ │  │  └──────────────────────┘ │ │
│  └────────────────────────────┘  └────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## Creating Subnets

```bash
# Create public subnets
aws ec2 create-subnet \
  --vpc-id vpc-xxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=pub-1a}]'

aws ec2 create-subnet \
  --vpc-id vpc-xxx \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=pub-1b}]'

# Create private subnets
aws ec2 create-subnet \
  --vpc-id vpc-xxx \
  --cidr-block 10.0.3.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=priv-1a}]'

aws ec2 create-subnet \
  --vpc-id vpc-xxx \
  --cidr-block 10.0.4.0/24 \
  --availability-zone us-east-1b \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=priv-1b}]'

# Enable auto-assign public IP for public subnets
aws ec2 modify-subnet-attribute \
  --subnet-id subnet-pub1a \
  --map-public-ip-on-launch
```

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Public Subnet** | Route table points 0.0.0.0/0 to IGW. For internet-facing resources. |
| **Private Subnet** | Route table points 0.0.0.0/0 to NAT GW. For backend/internal resources. |
| **Data Subnet** | No internet route at all. For databases. Maximum isolation. |
| **Three-Tier** | Public (LB) → Private (App) → Data (DB). Standard production pattern. |
| **Subnet per AZ** | Each subnet lives in exactly one AZ. Create matching subnets in each AZ. |

---

## Quick Revision Questions

1. **What makes a subnet "public"?**
   > Its route table has a route to an Internet Gateway (0.0.0.0/0 → IGW).

2. **Can resources in a private subnet access the internet?**
   > Yes, outbound only, via a NAT Gateway in a public subnet.

3. **Why use a three-tier subnet architecture?**
   > Defense in depth: public tier for load balancers, private for app logic, data tier for databases with no internet access.

4. **Can a subnet span multiple Availability Zones?**
   > No. Each subnet is in exactly one AZ.

5. **How many usable IPs are in a /24 subnet in AWS?**
   > 251 (256 total minus 5 AWS-reserved IPs).

---

[← Previous: 3.1 VPC Concepts](01-vpc-concepts.md) | [Next: 3.3 Route Tables →](03-route-tables.md)

[← Back to README](../README.md)
