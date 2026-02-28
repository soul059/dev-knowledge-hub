# Chapter 3: Internet and NAT Gateways

## Overview

**Internet Gateways (IGW)** and **NAT Gateways** are the two primary components that control internet access in a VPC. The IGW provides bidirectional internet access for public subnets, while the NAT Gateway enables outbound-only internet access for private subnets. Understanding when and how to use each is critical for secure cloud architecture.

---

## 3.1 Internet Gateway (IGW)

```
┌──────────────────────────────────────────────────────────────┐
│             INTERNET GATEWAY (IGW)                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Internet                                                    │
│     │                                                        │
│     ▼                                                        │
│  ┌──────────────┐                                            │
│  │  Internet    │  ← One per VPC                             │
│  │  Gateway     │  ← Highly available (managed by AWS)       │
│  │  (IGW)       │  ← No bandwidth constraints                │
│  └──────┬───────┘                                            │
│         │                                                    │
│  ┌──────▼──────────────────────────────────────────────┐     │
│  │  VPC: 10.0.0.0/16                                   │     │
│  │                                                      │     │
│  │  Public Subnet (Route: 0.0.0.0/0 → IGW)             │     │
│  │  ┌──────────────┐    ┌──────────────┐                │     │
│  │  │ EC2 Instance │    │ EC2 Instance │                │     │
│  │  │ Private IP:  │    │ Private IP:  │                │     │
│  │  │  10.0.1.10   │    │  10.0.1.11   │                │     │
│  │  │ Public IP:   │    │ EIP:         │                │     │
│  │  │  54.x.x.x    │    │  3.x.x.x     │                │     │
│  │  └──────────────┘    └──────────────┘                │     │
│  │                                                      │     │
│  │  What IGW does:                                      │     │
│  │  1. NAT translation (private IP ↔ public IP)         │     │
│  │  2. Routes internet traffic in/out                   │     │
│  │  3. No single point of failure                       │     │
│  └──────────────────────────────────────────────────────┘     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Requirements for Internet Access (Public Subnet)

1. VPC has an Internet Gateway attached
2. Subnet's route table has `0.0.0.0/0 → igw-xxx`
3. Instance has a public IP or Elastic IP
4. Security Group allows the traffic
5. NACL allows the traffic

---

## 3.2 NAT Gateway

```
┌──────────────────────────────────────────────────────────────┐
│             NAT GATEWAY                                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Internet                                                    │
│     ▲ (outbound only — no inbound initiation)                │
│     │                                                        │
│  ┌──┴───────────────┐                                        │
│  │  Internet Gateway │                                       │
│  └──┬───────────────┘                                        │
│     │                                                        │
│  ┌──▼────────────────────────────────────────────────┐       │
│  │  Public Subnet 10.0.1.0/24                        │       │
│  │  ┌──────────────────┐                              │       │
│  │  │  NAT Gateway     │ ← Has Elastic IP             │       │
│  │  │  EIP: 52.x.x.x  │ ← Lives in public subnet     │       │
│  │  │                  │ ← Managed by AWS (HA in AZ)   │       │
│  │  └────────┬─────────┘                              │       │
│  └───────────┼────────────────────────────────────────┘       │
│              │                                               │
│  ┌───────────▼────────────────────────────────────────┐       │
│  │  Private Subnet 10.0.11.0/24                      │       │
│  │  Route: 0.0.0.0/0 → nat-gateway-id                │       │
│  │                                                    │       │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐         │       │
│  │  │ App      │  │ App      │  │ Worker   │         │       │
│  │  │ Server   │  │ Server   │  │ Node     │         │       │
│  │  │ 10.0.11.5│  │ 10.0.11.6│  │ 10.0.11.7│         │       │
│  │  └──────────┘  └──────────┘  └──────────┘         │       │
│  │                                                    │       │
│  │  ✓ Can download packages (apt, yum, pip)           │       │
│  │  ✓ Can call external APIs                          │       │
│  │  ✗ Cannot be reached from internet                 │       │
│  └────────────────────────────────────────────────────┘       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.3 IGW vs NAT Gateway Comparison

| Feature | Internet Gateway | NAT Gateway |
|---------|-----------------|-------------|
| Direction | Bidirectional | Outbound only |
| Placement | VPC level | Public subnet |
| Public IP needed | Yes (on instance) | Yes (EIP on NAT GW) |
| Cost | Free | ~$0.045/hr + data processing |
| Availability | AWS-managed HA | HA within single AZ |
| Bandwidth | No limit | Up to 100 Gbps |
| Use case | Public-facing servers | Private subnet internet access |
| Protocol | All | TCP, UDP, ICMP |

---

## 3.4 High Availability NAT Design

```
┌──────────────────────────────────────────────────────────────┐
│          HA NAT GATEWAY DESIGN (1 per AZ)                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│        Internet                                              │
│           │                                                  │
│     ┌─────▼─────┐                                            │
│     │   IGW     │                                            │
│     └─────┬─────┘                                            │
│           │                                                  │
│  ┌────────┼──────────────────────────────────────────┐       │
│  │  VPC   │                                          │       │
│  │        │                                          │       │
│  │   AZ-1a           AZ-1b           AZ-1c           │       │
│  │  ┌─────▼─────┐  ┌───────────┐  ┌───────────┐     │       │
│  │  │ Public    │  │ Public    │  │ Public    │     │       │
│  │  │ Subnet    │  │ Subnet    │  │ Subnet    │     │       │
│  │  │ ┌───────┐ │  │ ┌───────┐ │  │ ┌───────┐ │     │       │
│  │  │ │NAT GW │ │  │ │NAT GW │ │  │ │NAT GW │ │     │       │
│  │  │ │EIP: A │ │  │ │EIP: B │ │  │ │EIP: C │ │     │       │
│  │  │ └───┬───┘ │  │ └───┬───┘ │  │ └───┬───┘ │     │       │
│  │  └─────┼─────┘  └─────┼─────┘  └─────┼─────┘     │       │
│  │        │              │              │            │       │
│  │  ┌─────▼─────┐  ┌─────▼─────┐  ┌─────▼─────┐     │       │
│  │  │ Private   │  │ Private   │  │ Private   │     │       │
│  │  │ Subnet    │  │ Subnet    │  │ Subnet    │     │       │
│  │  │ RT: NAT-A │  │ RT: NAT-B │  │ RT: NAT-C │     │       │
│  │  └───────────┘  └───────────┘  └───────────┘     │       │
│  │                                                   │       │
│  │  Each AZ has its own NAT GW → no cross-AZ issue  │       │
│  └───────────────────────────────────────────────────┘       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

> **Cost tip**: In dev/staging, you can use a single NAT Gateway to save costs. In production, always use one per AZ for high availability.

---

## 3.5 Terraform: NAT Gateway HA Setup

```hcl
variable "azs" {
  default = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

# Elastic IPs for NAT Gateways
resource "aws_eip" "nat" {
  count  = length(var.azs)
  domain = "vpc"
  tags   = { Name = "nat-eip-${var.azs[count.index]}" }
}

# NAT Gateways - one per AZ
resource "aws_nat_gateway" "main" {
  count         = length(var.azs)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = { Name = "nat-${var.azs[count.index]}" }

  depends_on = [aws_internet_gateway.main]
}

# Private route tables - each points to its AZ's NAT GW
resource "aws_route_table" "private" {
  count  = length(var.azs)
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }

  tags = { Name = "private-rt-${var.azs[count.index]}" }
}

# Cost-optimized: Single NAT (dev/staging only)
resource "aws_nat_gateway" "single" {
  count         = var.environment == "production" ? 0 : 1
  allocation_id = aws_eip.nat[0].id
  subnet_id     = aws_subnet.public[0].id
  tags          = { Name = "nat-single" }
}
```

---

## 3.6 NAT Gateway vs NAT Instance

| Feature | NAT Gateway | NAT Instance |
|---------|-------------|-------------|
| Management | AWS-managed | Self-managed EC2 |
| Availability | HA within AZ | Single instance |
| Bandwidth | Up to 100 Gbps | Depends on instance type |
| Cost | ~$0.045/hr + data | EC2 instance cost |
| Maintenance | None | Patching, monitoring |
| Security Groups | Not supported | Supported |
| Bastion host | No | Can double as bastion |
| **Recommendation** | **Production** | **Very low budget only** |

---

## 3.7 Egress-Only Internet Gateway (IPv6)

```
For IPv6 traffic, use an Egress-Only Internet Gateway:
- Equivalent of NAT Gateway but for IPv6
- Allows outbound IPv6 traffic
- Blocks inbound IPv6 connections
- No charge (free)
- One per VPC

aws ec2 create-egress-only-internet-gateway --vpc-id vpc-12345678
```

---

## 3.8 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Private instance can't reach internet | No NAT Gateway | Create NAT GW in public subnet |
| NAT Gateway creation fails | No IGW on VPC | Attach IGW first |
| "nat-xxx" route shows blackhole | NAT GW was deleted | Recreate NAT Gateway |
| High NAT Gateway bill | Cross-AZ data transfer | Use 1 NAT per AZ to keep traffic local |
| NAT GW bandwidth throttling | Exceeding 100 Gbps | Scale horizontally or use multiple NATs |
| Public instance unreachable | No public IP assigned | Assign EIP or enable auto-assign |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Internet Gateway | Bidirectional internet, 1 per VPC, free |
| NAT Gateway | Outbound-only, lives in public subnet, ~$0.045/hr |
| EIP | Static public IP, required for NAT Gateway |
| HA Design | One NAT Gateway per AZ in production |
| NAT Instance | Self-managed EC2, legacy approach |
| Egress-Only IGW | IPv6 outbound-only equivalent |
| Public subnet needs | IGW + route + public IP + SG + NACL |
| Cost optimization | Single NAT for dev, per-AZ NAT for prod |

---

## Quick Revision Questions

1. **What five things must be in place for an EC2 instance to reach the internet from a public subnet?**
2. **Why does a NAT Gateway need to be placed in a public subnet?**
3. **Compare the cost of NAT Gateway vs NAT Instance. When would each be appropriate?**
4. **Design a high-availability NAT architecture for 3 AZs. How many NAT Gateways do you need?**
5. **What is an Egress-Only Internet Gateway and when would you use it?**
6. **Your NAT Gateway bill is unexpectedly high. What's the most likely cause?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Subnets & Route Tables](02-subnets-route-tables.md) | [README](../README.md) | [Peering & Transit Gateways →](04-peering-transit-gateways.md) |
