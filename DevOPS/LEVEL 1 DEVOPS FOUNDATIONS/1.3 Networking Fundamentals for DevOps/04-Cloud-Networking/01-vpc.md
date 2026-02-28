# Chapter 1: Virtual Private Cloud (VPC)

## Overview

A **Virtual Private Cloud (VPC)** is an isolated virtual network within a cloud provider where you launch and manage resources. VPCs give you complete control over IP addressing, subnets, routing, and security — essentially your own private data center in the cloud. Understanding VPC design is foundational to every cloud deployment.

---

## 1.1 VPC Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                 AWS VPC ARCHITECTURE                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  AWS Region: us-east-1                                       │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              VPC: 10.0.0.0/16                          │  │
│  │              65,536 IP addresses                       │  │
│  │                                                        │  │
│  │  AZ: us-east-1a          AZ: us-east-1b                │  │
│  │  ┌──────────────────┐    ┌──────────────────┐          │  │
│  │  │ Public Subnet    │    │ Public Subnet    │          │  │
│  │  │ 10.0.1.0/24      │    │ 10.0.2.0/24      │          │  │
│  │  │ ┌──────────────┐ │    │ ┌──────────────┐ │          │  │
│  │  │ │  ALB / NAT   │ │    │ │  ALB / NAT   │ │          │  │
│  │  │ └──────────────┘ │    │ └──────────────┘ │          │  │
│  │  └──────────────────┘    └──────────────────┘          │  │
│  │                                                        │  │
│  │  ┌──────────────────┐    ┌──────────────────┐          │  │
│  │  │ Private Subnet   │    │ Private Subnet   │          │  │
│  │  │ 10.0.11.0/24     │    │ 10.0.12.0/24     │          │  │
│  │  │ ┌──────────────┐ │    │ ┌──────────────┐ │          │  │
│  │  │ │  App Servers │ │    │ │  App Servers │ │          │  │
│  │  │ └──────────────┘ │    │ └──────────────┘ │          │  │
│  │  └──────────────────┘    └──────────────────┘          │  │
│  │                                                        │  │
│  │  ┌──────────────────┐    ┌──────────────────┐          │  │
│  │  │ Data Subnet      │    │ Data Subnet      │          │  │
│  │  │ 10.0.21.0/24     │    │ 10.0.22.0/24     │          │  │
│  │  │ ┌──────────────┐ │    │ ┌──────────────┐ │          │  │
│  │  │ │  RDS / Redis │ │    │ │  RDS Standby │ │          │  │
│  │  │ └──────────────┘ │    │ └──────────────┘ │          │  │
│  │  └──────────────────┘    └──────────────────┘          │  │
│  │                                                        │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.2 VPC Components

| Component | Purpose | Key Detail |
|-----------|---------|------------|
| VPC | Isolated network | CIDR block (e.g., 10.0.0.0/16) |
| Subnet | Segment within VPC | Tied to a single AZ |
| Internet Gateway (IGW) | Internet access | 1 per VPC, highly available |
| NAT Gateway | Outbound internet for private subnets | 1 per AZ for HA |
| Route Table | Directs traffic | Associated with subnets |
| Security Group | Instance-level firewall | Stateful |
| NACL | Subnet-level firewall | Stateless |
| Elastic IP | Static public IP | Survives instance stop/start |
| ENI | Virtual network interface | Can be moved between instances |

---

## 1.3 CIDR Planning for VPCs

```
┌──────────────────────────────────────────────────────────────┐
│              VPC CIDR PLANNING                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Production VPC:  10.0.0.0/16   (65,536 IPs)               │
│  Staging VPC:     10.1.0.0/16   (65,536 IPs)               │
│  Development VPC: 10.2.0.0/16   (65,536 IPs)               │
│  On-Premise:      192.168.0.0/16                            │
│                                                              │
│  ⚠ IMPORTANT: VPC CIDRs must NOT overlap if peering!        │
│                                                              │
│  Subnet Sizing Guide:                                        │
│  ┌──────────┬──────────┬──────────────────────────┐          │
│  │ CIDR     │ Usable*  │ Use Case                 │          │
│  ├──────────┼──────────┼──────────────────────────┤          │
│  │ /28      │ 11       │ Minimal (NAT GW, IGW)    │          │
│  │ /26      │ 59       │ Small (bastion, tools)    │          │
│  │ /24      │ 251      │ Standard (app tier)       │          │
│  │ /22      │ 1,019    │ Large (K8s, auto-scale)   │          │
│  │ /20      │ 4,091    │ Very large (EKS pods)     │          │
│  └──────────┴──────────┴──────────────────────────┘          │
│  * AWS reserves 5 IPs per subnet                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.4 Terraform: Complete VPC

```hcl
# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags                 = { Name = "production-vpc" }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  tags   = { Name = "main-igw" }
}

# Public Subnets (2 AZs)
resource "aws_subnet" "public" {
  count                   = 2
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index + 1)
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  map_public_ip_on_launch = true
  tags                    = { Name = "public-${count.index + 1}" }
}

# Private Subnets (2 AZs)
resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(aws_vpc.main.cidr_block, 8, count.index + 11)
  availability_zone = data.aws_availability_zones.available.names[count.index]
  tags              = { Name = "private-${count.index + 1}" }
}

# NAT Gateway (in public subnet)
resource "aws_eip" "nat" {
  domain = "vpc"
}

resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public[0].id
  tags          = { Name = "main-nat" }
}

# Public Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }
  tags = { Name = "public-rt" }
}

# Private Route Table
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }
  tags = { Name = "private-rt" }
}

# Route Table Associations
resource "aws_route_table_association" "public" {
  count          = 2
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

resource "aws_route_table_association" "private" {
  count          = 2
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private.id
}
```

---

## 1.5 VPC Flow Logs

```bash
# Enable VPC Flow Logs (CLI)
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-12345678 \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /vpc/flow-logs \
  --deliver-logs-permission-arn arn:aws:iam::123456789012:role/flowlogsRole

# Sample flow log entry:
# account-id eni-id srcaddr  dstaddr  srcport dstport proto action
# 123456789  eni-x  10.0.1.5 10.0.2.8 49152   443     6     ACCEPT
# 123456789  eni-x  10.0.1.5 10.0.2.8 49152   22      6     REJECT
```

```
Flow Log Format:
┌──────────┬──────────┬─────────┬──────────┬────────┬────────┬────────┐
│ Account  │ ENI      │ Src IP  │ Dst IP   │ SrcPort│ DstPort│ Action │
├──────────┼──────────┼─────────┼──────────┼────────┼────────┼────────┤
│ 12345..  │ eni-abc  │ 10.0.1.5│ 10.0.2.8 │ 49152  │ 443    │ ACCEPT │
│ 12345..  │ eni-abc  │ 10.0.1.5│ 10.0.2.8 │ 49152  │ 22     │ REJECT │
└──────────┴──────────┴─────────┴──────────┴────────┴────────┴────────┘
```

---

## 1.6 Multi-Cloud VPC Comparison

| Feature | AWS VPC | Azure VNet | GCP VPC |
|---------|---------|------------|---------|
| Network scope | Regional | Regional | Global |
| Subnet scope | Single AZ | Single AZ | Regional |
| Default network | Default VPC | None | Default VPC |
| CIDR range | /16 to /28 | /8 to /29 | Auto or custom |
| Max VPCs/region | 5 (adjustable) | 50 | 15 (adjustable) |
| NAT solution | NAT Gateway | NAT Gateway | Cloud NAT |
| Flow logs | VPC Flow Logs | NSG Flow Logs | VPC Flow Logs |

---

## 1.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Instance has no internet | Missing IGW or route | Attach IGW, add 0.0.0.0/0 route |
| Private instance can't reach internet | No NAT Gateway | Create NAT GW in public subnet |
| Can't communicate between subnets | NACL blocking traffic | Check NACL rules |
| DNS not resolving | DNS support disabled | Enable `enableDnsSupport` on VPC |
| Can't peer two VPCs | Overlapping CIDRs | Use non-overlapping CIDR blocks |
| IP addresses exhausted | Subnet too small | Use larger CIDR or add secondary CIDR |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| VPC | Isolated virtual network in cloud |
| CIDR Block | Defines IP range (e.g., 10.0.0.0/16) |
| Subnet | Segment within VPC, tied to one AZ |
| IGW | Enables internet access for public subnets |
| NAT Gateway | Outbound-only internet for private subnets |
| Route Table | Directs traffic to correct destination |
| Flow Logs | Capture network traffic metadata for debugging |
| AWS reserves 5 IPs | First 4 + last IP in every subnet |

---

## Quick Revision Questions

1. **What is the relationship between a VPC, subnets, and availability zones?**
2. **Why should VPC CIDR blocks not overlap when planning for peering?**
3. **How many IPs does AWS reserve per subnet, and what are they used for?**
4. **Design a VPC with public and private subnets across 2 AZs. What components do you need?**
5. **What's the difference between an Internet Gateway and a NAT Gateway?**
6. **How do VPC Flow Logs help in troubleshooting network issues?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Proxy Servers](../03-Network-Architecture/06-proxy-servers.md) | [README](../README.md) | [Subnets & Route Tables →](02-subnets-route-tables.md) |
