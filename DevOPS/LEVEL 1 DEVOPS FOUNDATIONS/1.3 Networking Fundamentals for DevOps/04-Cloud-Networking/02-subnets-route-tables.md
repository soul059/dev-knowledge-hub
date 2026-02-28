# Chapter 2: Subnets and Route Tables

## Overview

**Subnets** divide a VPC into smaller network segments, each tied to a specific Availability Zone. **Route Tables** determine where network traffic is directed. Together, they form the backbone of VPC traffic flow — controlling which resources can reach the internet, communicate with each other, or connect to on-premise networks.

---

## 2.1 Subnet Types

```
┌──────────────────────────────────────────────────────────────┐
│              PUBLIC vs PRIVATE SUBNETS                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PUBLIC SUBNET                                               │
│  ┌────────────────────────────────────┐                      │
│  │ Route Table:                       │                      │
│  │   10.0.0.0/16 → local             │                      │
│  │   0.0.0.0/0   → igw-xxxxx  ← IGW │                      │
│  │                                    │                      │
│  │ ┌──────────┐  ┌──────────┐        │                      │
│  │ │ EC2      │  │ NAT      │        │                      │
│  │ │ (Public  │  │ Gateway  │        │                      │
│  │ │  IP)     │  │          │        │                      │
│  │ └──────────┘  └──────────┘        │                      │
│  └────────────────────────────────────┘                      │
│  ↕ Has route to Internet Gateway                             │
│                                                              │
│  PRIVATE SUBNET                                              │
│  ┌────────────────────────────────────┐                      │
│  │ Route Table:                       │                      │
│  │   10.0.0.0/16 → local             │                      │
│  │   0.0.0.0/0   → nat-xxxxx ← NAT  │                      │
│  │                                    │                      │
│  │ ┌──────────┐  ┌──────────┐        │                      │
│  │ │ App      │  │ Database │        │                      │
│  │ │ Server   │  │          │        │                      │
│  │ │ (No pub  │  │ (No pub  │        │                      │
│  │ │  IP)     │  │  IP)     │        │                      │
│  │ └──────────┘  └──────────┘        │                      │
│  └────────────────────────────────────┘                      │
│  ↕ Route to NAT (outbound only) or no internet route        │
│                                                              │
│  ISOLATED SUBNET (no internet at all)                        │
│  ┌────────────────────────────────────┐                      │
│  │ Route Table:                       │                      │
│  │   10.0.0.0/16 → local  (only!)    │                      │
│  │                                    │                      │
│  │ ┌──────────┐                       │                      │
│  │ │ Sensitive│  No internet access   │                      │
│  │ │ Database │  whatsoever           │                      │
│  │ └──────────┘                       │                      │
│  └────────────────────────────────────┘                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.2 Route Table Anatomy

```
┌──────────────────────────────────────────────────────────────┐
│              ROUTE TABLE STRUCTURE                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Public Route Table:                                         │
│  ┌────────────────────┬──────────────┬────────────────────┐  │
│  │ Destination        │ Target       │ Status             │  │
│  ├────────────────────┼──────────────┼────────────────────┤  │
│  │ 10.0.0.0/16        │ local        │ Active (auto)      │  │
│  │ 0.0.0.0/0          │ igw-abc123   │ Active (internet)  │  │
│  │ 192.168.0.0/16     │ vgw-xyz789   │ Active (VPN)       │  │
│  └────────────────────┴──────────────┴────────────────────┘  │
│                                                              │
│  Private Route Table:                                        │
│  ┌────────────────────┬──────────────┬────────────────────┐  │
│  │ Destination        │ Target       │ Status             │  │
│  ├────────────────────┼──────────────┼────────────────────┤  │
│  │ 10.0.0.0/16        │ local        │ Active (auto)      │  │
│  │ 0.0.0.0/0          │ nat-def456   │ Active (NAT GW)    │  │
│  │ 192.168.0.0/16     │ vgw-xyz789   │ Active (VPN)       │  │
│  │ 10.1.0.0/16        │ pcx-peer123  │ Active (peering)   │  │
│  └────────────────────┴──────────────┴────────────────────┘  │
│                                                              │
│  Routing Priority: Most specific route wins                  │
│  - 10.0.1.5 matches 10.0.0.0/16 (local) ← more specific    │
│  - 8.8.8.8 matches 0.0.0.0/0 (default) ← least specific    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.3 Route Targets Explained

| Target | What It Is | Traffic Direction |
|--------|-----------|-------------------|
| `local` | Within VPC | VPC internal (auto-created, can't delete) |
| `igw-xxx` | Internet Gateway | Public internet (bidirectional) |
| `nat-xxx` | NAT Gateway | Outbound internet only |
| `vgw-xxx` | Virtual Private Gateway | VPN to on-premise |
| `pcx-xxx` | Peering Connection | To another VPC |
| `tgw-xxx` | Transit Gateway | To TGW hub |
| `vpce-xxx` | VPC Endpoint | To AWS service (private) |
| `eni-xxx` | Network Interface | To specific instance (e.g., firewall appliance) |

---

## 2.4 Subnet Design Patterns

```
┌──────────────────────────────────────────────────────────────┐
│     PRODUCTION VPC SUBNET LAYOUT (10.0.0.0/16)              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Tier          AZ-a              AZ-b              AZ-c     │
│  ┌──────────┬──────────────┬──────────────┬──────────────┐  │
│  │ Public   │ 10.0.1.0/24  │ 10.0.2.0/24  │ 10.0.3.0/24  │  │
│  │ (ALB,    │ 251 IPs      │ 251 IPs      │ 251 IPs      │  │
│  │  NAT)    │              │              │              │  │
│  ├──────────┼──────────────┼──────────────┼──────────────┤  │
│  │ App      │ 10.0.11.0/24 │ 10.0.12.0/24 │ 10.0.13.0/24 │  │
│  │ (EC2,    │ 251 IPs      │ 251 IPs      │ 251 IPs      │  │
│  │  ECS)    │              │              │              │  │
│  ├──────────┼──────────────┼──────────────┼──────────────┤  │
│  │ Data     │ 10.0.21.0/24 │ 10.0.22.0/24 │ 10.0.23.0/24 │  │
│  │ (RDS,    │ 251 IPs      │ 251 IPs      │ 251 IPs      │  │
│  │  Redis)  │              │              │              │  │
│  ├──────────┼──────────────┼──────────────┼──────────────┤  │
│  │ EKS Pods │ 10.0.32.0/20 │ 10.0.48.0/20 │ 10.0.64.0/20 │  │
│  │ (Large   │ 4,091 IPs    │ 4,091 IPs    │ 4,091 IPs    │  │
│  │  CIDR)   │              │              │              │  │
│  └──────────┴──────────────┴──────────────┴──────────────┘  │
│                                                              │
│  Naming: {env}-{tier}-{az}                                   │
│  Example: prod-app-1a, prod-data-1b                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.5 Terraform: Subnets and Route Tables

```hcl
# Variables
variable "azs" {
  default = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

# Public Subnets
resource "aws_subnet" "public" {
  count                   = length(var.azs)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet("10.0.0.0/16", 8, count.index + 1)
  availability_zone       = var.azs[count.index]
  map_public_ip_on_launch = true

  tags = {
    Name = "public-${var.azs[count.index]}"
    Tier = "public"
  }
}

# Private App Subnets
resource "aws_subnet" "app" {
  count             = length(var.azs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet("10.0.0.0/16", 8, count.index + 11)
  availability_zone = var.azs[count.index]

  tags = {
    Name = "app-${var.azs[count.index]}"
    Tier = "app"
  }
}

# Private Data Subnets
resource "aws_subnet" "data" {
  count             = length(var.azs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet("10.0.0.0/16", 8, count.index + 21)
  availability_zone = var.azs[count.index]

  tags = {
    Name = "data-${var.azs[count.index]}"
    Tier = "data"
  }
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

resource "aws_route_table_association" "public" {
  count          = length(var.azs)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

# NAT Gateways (one per AZ for HA)
resource "aws_eip" "nat" {
  count  = length(var.azs)
  domain = "vpc"
}

resource "aws_nat_gateway" "main" {
  count         = length(var.azs)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  tags          = { Name = "nat-${var.azs[count.index]}" }
}

# Private Route Tables (one per AZ, each pointing to its own NAT)
resource "aws_route_table" "private" {
  count  = length(var.azs)
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main[count.index].id
  }

  tags = { Name = "private-rt-${var.azs[count.index]}" }
}

resource "aws_route_table_association" "app" {
  count          = length(var.azs)
  subnet_id      = aws_subnet.app[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}

resource "aws_route_table_association" "data" {
  count          = length(var.azs)
  subnet_id      = aws_subnet.data[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}
```

---

## 2.6 AWS CLI: Route Table Operations

```bash
# List route tables
aws ec2 describe-route-tables \
  --filters "Name=vpc-id,Values=vpc-12345678" \
  --query "RouteTables[*].{ID:RouteTableId,Routes:Routes}" \
  --output table

# Create a route
aws ec2 create-route \
  --route-table-id rtb-12345678 \
  --destination-cidr-block 192.168.0.0/16 \
  --gateway-id vgw-87654321

# Associate subnet with route table
aws ec2 associate-route-table \
  --subnet-id subnet-12345678 \
  --route-table-id rtb-12345678

# Check which route table a subnet uses
aws ec2 describe-route-tables \
  --filters "Name=association.subnet-id,Values=subnet-12345678"
```

---

## 2.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Instance can't reach internet | No route to IGW in route table | Add 0.0.0.0/0 → igw-xxx route |
| Private subnet has no internet | No NAT Gateway or wrong route | Create NAT GW, add 0.0.0.0/0 → nat-xxx |
| VPN traffic not flowing | Missing route to VGW | Add on-prem CIDR → vgw-xxx |
| Peered VPC unreachable | No route to peering connection | Add peer CIDR → pcx-xxx in both VPCs |
| "Blackhole" route status | Target resource was deleted | Remove stale route, recreate target |
| Subnet has wrong route table | Implicit main route table used | Explicitly associate correct route table |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Public Subnet | Has route to IGW (0.0.0.0/0 → igw) |
| Private Subnet | Route to NAT Gateway for outbound only |
| Isolated Subnet | Only local route — no internet at all |
| Route Table | Maps destination CIDRs to targets |
| Most Specific Wins | /24 route chosen over /16 over /0 |
| Local Route | Auto-created, non-deletable, VPC-internal |
| Main Route Table | Default RT if subnet has no explicit association |
| 1 NAT per AZ | Best practice for high availability |

---

## Quick Revision Questions

1. **What makes a subnet "public" vs "private" in AWS?**
2. **Why should you create a separate NAT Gateway per AZ?**
3. **Explain the "most specific route wins" rule with an example.**
4. **What is the `local` route in a route table and can it be modified?**
5. **Design a subnet layout for a production VPC with 3 tiers across 3 AZs.**
6. **A route shows status "blackhole." What does this mean and how do you fix it?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← VPC](01-vpc.md) | [README](../README.md) | [Internet & NAT Gateways →](03-internet-nat-gateways.md) |
