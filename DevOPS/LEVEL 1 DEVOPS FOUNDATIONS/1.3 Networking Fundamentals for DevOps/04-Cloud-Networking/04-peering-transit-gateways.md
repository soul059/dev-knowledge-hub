# Chapter 4: VPC Peering and Transit Gateways

## Overview

As cloud environments scale, you'll need to connect multiple VPCs together. **VPC Peering** creates a direct, private connection between two VPCs. **Transit Gateway** acts as a central hub connecting many VPCs and on-premise networks. Choosing the right connectivity model is critical for scalable, maintainable cloud architectures.

---

## 4.1 VPC Peering

```
┌──────────────────────────────────────────────────────────────┐
│              VPC PEERING                                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌───────────────────┐        ┌───────────────────┐          │
│  │  VPC A             │  Peer  │  VPC B             │          │
│  │  10.0.0.0/16       │◄══════▶│  10.1.0.0/16       │          │
│  │                    │ pcx-xx │                    │          │
│  │  Route Table:      │        │  Route Table:      │          │
│  │  10.1.0.0/16→pcx-xx│        │  10.0.0.0/16→pcx-xx│          │
│  │                    │        │                    │          │
│  │  ┌──────┐ ┌──────┐│        │┌──────┐ ┌──────┐  │          │
│  │  │App   │ │App   ││        ││DB    │ │DB    │  │          │
│  │  │Server│ │Server││        ││RDS   │ │Redis │  │          │
│  │  └──────┘ └──────┘│        │└──────┘ └──────┘  │          │
│  └───────────────────┘        └───────────────────┘          │
│                                                              │
│  Key Points:                                                 │
│  ✓ Direct private connection (no internet)                   │
│  ✓ Cross-account and cross-region supported                  │
│  ✗ CIDRs must NOT overlap                                    │
│  ✗ NOT transitive (A↔B, B↔C does NOT mean A↔C)              │
│  ✗ Max ~125 peering connections per VPC                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Non-Transitive Limitation

```
┌──────────────────────────────────────────────────────────────┐
│         PEERING IS NOT TRANSITIVE                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│     VPC A ◄════════▶ VPC B ◄════════▶ VPC C                 │
│     10.0.0/16   pcx-1   10.1.0/16  pcx-2  10.2.0/16        │
│                                                              │
│     VPC A ──── ✗ ────▶ VPC C                                 │
│     (Cannot route through VPC B!)                            │
│                                                              │
│     To connect A↔C, you need a THIRD peering: pcx-3          │
│     Or use a Transit Gateway instead!                        │
│                                                              │
│     Full mesh with 4 VPCs = 6 peering connections            │
│     Full mesh with 10 VPCs = 45 peering connections ← messy  │
│                                                              │
│     Formula: n(n-1)/2 connections needed                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.2 Terraform: VPC Peering

```hcl
# Create peering connection
resource "aws_vpc_peering_connection" "app_to_data" {
  vpc_id        = aws_vpc.app.id          # Requester VPC
  peer_vpc_id   = aws_vpc.data.id         # Accepter VPC
  peer_region   = "us-east-1"             # For cross-region
  auto_accept   = false                    # Must be accepted
  tags          = { Name = "app-to-data-peering" }
}

# Accept peering (same account, same region)
resource "aws_vpc_peering_connection_accepter" "data" {
  vpc_peering_connection_id = aws_vpc_peering_connection.app_to_data.id
  auto_accept               = true
  tags                      = { Name = "app-to-data-peering" }
}

# Route from App VPC to Data VPC
resource "aws_route" "app_to_data" {
  route_table_id            = aws_route_table.app_private.id
  destination_cidr_block    = aws_vpc.data.cidr_block  # 10.1.0.0/16
  vpc_peering_connection_id = aws_vpc_peering_connection.app_to_data.id
}

# Route from Data VPC to App VPC
resource "aws_route" "data_to_app" {
  route_table_id            = aws_route_table.data_private.id
  destination_cidr_block    = aws_vpc.app.cidr_block   # 10.0.0.0/16
  vpc_peering_connection_id = aws_vpc_peering_connection.app_to_data.id
}

# Security Group: Allow traffic from peered VPC
resource "aws_security_group_rule" "allow_from_app" {
  type              = "ingress"
  from_port         = 5432
  to_port           = 5432
  protocol          = "tcp"
  cidr_blocks       = [aws_vpc.app.cidr_block]
  security_group_id = aws_security_group.database.id
}
```

---

## 4.3 Transit Gateway

```
┌──────────────────────────────────────────────────────────────┐
│              TRANSIT GATEWAY (TGW)                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│           ┌───────────┐     ┌───────────┐                    │
│           │ VPC A     │     │ VPC B     │                    │
│           │ 10.0.0/16 │     │ 10.1.0/16 │                    │
│           └─────┬─────┘     └─────┬─────┘                    │
│                 │                 │                           │
│                 ▼                 ▼                           │
│           ┌─────────────────────────────┐                    │
│           │                             │                    │
│           │     TRANSIT GATEWAY         │                    │
│           │     (Central Hub)           │                    │
│           │                             │                    │
│           │  Route Table:               │                    │
│           │  10.0.0/16 → VPC A attach   │                    │
│           │  10.1.0/16 → VPC B attach   │                    │
│           │  10.2.0/16 → VPC C attach   │                    │
│           │  192.168/16 → VPN attach    │                    │
│           │                             │                    │
│           └─────┬───────────┬───────────┘                    │
│                 │           │                                │
│                 ▼           ▼                                │
│           ┌───────────┐  ┌──────────────┐                   │
│           │ VPC C     │  │ VPN / Direct │                   │
│           │ 10.2.0/16 │  │ Connect      │                   │
│           └───────────┘  │ 192.168.0/16 │                   │
│                          └──────────────┘                   │
│                                                              │
│  Benefits:                                                   │
│  ✓ Hub-and-spoke model                                       │
│  ✓ Transitive routing (A↔B↔C all connected)                  │
│  ✓ Scales to 5,000+ VPCs                                     │
│  ✓ Centralized routing control                               │
│  ✓ Supports VPN and Direct Connect                           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.4 Peering vs Transit Gateway

| Feature | VPC Peering | Transit Gateway |
|---------|------------|-----------------|
| Topology | Point-to-point | Hub-and-spoke |
| Transitive routing | No | Yes |
| Max connections | ~125 per VPC | 5,000+ attachments |
| Cost | Free (data transfer only) | $0.05/hr + data |
| Cross-region | Yes | Yes (inter-region peering) |
| Cross-account | Yes | Yes (RAM sharing) |
| Bandwidth | No limit | Up to 50 Gbps per VPC |
| Complexity (5+ VPCs) | High (full mesh) | Low (hub-and-spoke) |
| VPN integration | No | Yes |
| Best for | 2-3 VPC connections | Multi-VPC, hybrid cloud |

---

## 4.5 Transit Gateway with Terraform

```hcl
# Transit Gateway
resource "aws_ec2_transit_gateway" "main" {
  description                     = "Main Transit Gateway"
  default_route_table_association = "enable"
  default_route_table_propagation = "enable"
  dns_support                     = "enable"
  tags                            = { Name = "main-tgw" }
}

# Attach VPCs
resource "aws_ec2_transit_gateway_vpc_attachment" "app" {
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = aws_vpc.app.id
  subnet_ids         = aws_subnet.app_private[*].id
  tags               = { Name = "app-vpc-attachment" }
}

resource "aws_ec2_transit_gateway_vpc_attachment" "data" {
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = aws_vpc.data.id
  subnet_ids         = aws_subnet.data_private[*].id
  tags               = { Name = "data-vpc-attachment" }
}

resource "aws_ec2_transit_gateway_vpc_attachment" "shared" {
  transit_gateway_id = aws_ec2_transit_gateway.main.id
  vpc_id             = aws_vpc.shared.id
  subnet_ids         = aws_subnet.shared_private[*].id
  tags               = { Name = "shared-vpc-attachment" }
}

# VPC route tables point to TGW for cross-VPC traffic
resource "aws_route" "app_to_tgw" {
  route_table_id         = aws_route_table.app_private.id
  destination_cidr_block = "10.0.0.0/8"     # All internal
  transit_gateway_id     = aws_ec2_transit_gateway.main.id
}

resource "aws_route" "data_to_tgw" {
  route_table_id         = aws_route_table.data_private.id
  destination_cidr_block = "10.0.0.0/8"
  transit_gateway_id     = aws_ec2_transit_gateway.main.id
}
```

---

## 4.6 Real-World Architecture: Multi-Account Hub-Spoke

```
┌──────────────────────────────────────────────────────────────┐
│     ENTERPRISE TRANSIT GATEWAY ARCHITECTURE                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Shared Services Account                                     │
│  ┌────────────────────┐                                      │
│  │ Shared VPC         │                                      │
│  │ - DNS              │                                      │
│  │ - Active Directory │──────┐                               │
│  │ - Monitoring       │      │                               │
│  └────────────────────┘      │                               │
│                              ▼                               │
│                    ┌─────────────────┐                        │
│  Prod Account      │                 │    Dev Account         │
│  ┌──────────────┐  │  Transit        │  ┌──────────────┐     │
│  │ Prod VPC     │──▶  Gateway        ◀──│ Dev VPC      │     │
│  │ 10.0.0.0/16  │  │  (Central Hub)  │  │ 10.2.0.0/16  │     │
│  └──────────────┘  │                 │  └──────────────┘     │
│                    │                 │                        │
│  Staging Account   │                 │    On-Premise          │
│  ┌──────────────┐  │                 │  ┌──────────────┐     │
│  │ Staging VPC  │──▶                 ◀──│ Data Center  │     │
│  │ 10.1.0.0/16  │  │                 │  │ 192.168.0/16 │     │
│  └──────────────┘  └─────────────────┘  │ (via VPN)    │     │
│                                         └──────────────┘     │
│                                                              │
│  TGW Route Tables can isolate:                               │
│  - Prod can reach Shared but NOT Dev                         │
│  - Dev can reach Shared but NOT Prod                         │
│  - All can reach On-Premise                                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Peered VPCs can't communicate | Missing routes in route tables | Add routes to pcx-xxx in both VPCs |
| Overlapping CIDR error on peering | VPC CIDRs overlap | Use non-overlapping CIDR ranges |
| TGW attachment pending | Not auto-accepted | Accept attachment in console/CLI |
| Cross-account peering not working | Not accepted by peer | Accept in peer account |
| Peered VPC DNS not resolving | DNS resolution disabled | Enable DNS resolution on peering |
| Can't access service in peered VPC | Security group blocks | Allow peer VPC CIDR in SG |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| VPC Peering | Point-to-point, non-transitive, free |
| Transit Gateway | Hub-and-spoke, transitive, ~$0.05/hr |
| Non-transitive | A↔B + B↔C does NOT mean A↔C (peering) |
| Full mesh formula | n(n-1)/2 connections needed with peering |
| TGW route tables | Can isolate traffic between attachments |
| Cross-account | Both peering and TGW support cross-account |
| CIDR overlap | Must not overlap for any peering/TGW |
| Best practice | Use TGW when connecting 3+ VPCs |

---

## Quick Revision Questions

1. **What does "non-transitive" mean in VPC peering and why is it a limitation?**
2. **Calculate how many peering connections you need for 8 VPCs in full mesh.**
3. **When would you choose Transit Gateway over VPC Peering?**
4. **How do TGW route tables enable network isolation between environments?**
5. **What are the steps to set up cross-account VPC peering?**
6. **Two peered VPCs have routes configured but instances can't communicate. What else should you check?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Internet & NAT Gateways](03-internet-nat-gateways.md) | [README](../README.md) | [NACLs vs Security Groups →](05-nacls-vs-security-groups.md) |
