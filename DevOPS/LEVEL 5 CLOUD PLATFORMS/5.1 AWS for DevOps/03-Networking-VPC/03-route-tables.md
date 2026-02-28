# Chapter 3.3: Route Tables

## Overview

Route tables contain a set of rules (**routes**) that determine where network traffic is directed. Every subnet must be associated with a route table. This is the mechanism that controls whether a subnet is public, private, or isolated.

---

## Route Table Architecture

```
┌──────────────────── VPC 10.0.0.0/16 ──────────────────────────┐
│                                                                 │
│  ┌─── Main Route Table (implicit default) ───────────────────┐ │
│  │  Dest: 10.0.0.0/16  → Target: local                      │ │
│  │  (Any subnet NOT explicitly associated uses this)         │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌─── Custom RT: Public-RT ──────────────────────────────────┐ │
│  │  Dest: 10.0.0.0/16  → Target: local                      │ │
│  │  Dest: 0.0.0.0/0    → Target: igw-xxx                    │ │
│  │                                                            │ │
│  │  Associated: pub-1a (10.0.1.0/24)                         │ │
│  │              pub-1b (10.0.2.0/24)                         │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌─── Custom RT: Private-RT ─────────────────────────────────┐ │
│  │  Dest: 10.0.0.0/16  → Target: local                      │ │
│  │  Dest: 0.0.0.0/0    → Target: nat-xxx                    │ │
│  │                                                            │ │
│  │  Associated: priv-1a (10.0.3.0/24)                        │ │
│  │              priv-1b (10.0.4.0/24)                        │ │
│  └───────────────────────────────────────────────────────────┘ │
│                                                                 │
│  ┌─── Custom RT: Data-RT ────────────────────────────────────┐ │
│  │  Dest: 10.0.0.0/16  → Target: local                      │ │
│  │  (No default route → fully isolated)                      │ │
│  │                                                            │ │
│  │  Associated: data-1a (10.0.5.0/24)                        │ │
│  │              data-1b (10.0.6.0/24)                        │ │
│  └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## Route Evaluation: Most Specific Wins

```
Route Table:
┌──────────────────────────────────────────────────┐
│  Destination        Target         Priority      │
│  ─────────────────  ────────────── ──────────    │
│  10.0.0.0/16        local          Most specific │
│  10.0.5.0/24        pcx-xxx        ◄ Wins for   │
│                                      10.0.5.x    │
│  172.16.0.0/16      tgw-xxx        Peered VPC   │
│  0.0.0.0/0          igw-xxx        Least specif. │
└──────────────────────────────────────────────────┘

Traffic to 10.0.5.10:
  Matches 10.0.0.0/16 (/16) AND 10.0.5.0/24 (/24)
  Winner: 10.0.5.0/24 → pcx-xxx  (longer prefix = more specific)
```

---

## Main vs Custom Route Tables

| Feature | Main Route Table | Custom Route Table |
|---------|-----------------|-------------------|
| Created by | AWS (one per VPC) | You |
| Default association | All subnets not explicitly associated | Only explicitly associated subnets |
| Deletable | No | Yes |
| Best practice | Leave minimal (local only) | Create specific ones per tier |

> **Best Practice:** Never add internet routes to the main route table. Create custom route tables for public subnets to avoid accidentally making new subnets public.

---

## CLI Commands

```bash
# Create a route table
aws ec2 create-route-table \
  --vpc-id vpc-xxx \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=Public-RT}]'

# Add route to internet gateway (makes associated subnets public)
aws ec2 create-route \
  --route-table-id rtb-xxx \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-xxx

# Add route to NAT gateway (for private subnets)
aws ec2 create-route \
  --route-table-id rtb-yyy \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id nat-xxx

# Associate subnet with route table
aws ec2 associate-route-table \
  --route-table-id rtb-xxx \
  --subnet-id subnet-xxx

# Add VPC peering route
aws ec2 create-route \
  --route-table-id rtb-xxx \
  --destination-cidr-block 172.16.0.0/16 \
  --vpc-peering-connection-id pcx-xxx

# Describe routes in a table
aws ec2 describe-route-tables \
  --route-table-ids rtb-xxx \
  --query 'RouteTables[0].Routes[*].[DestinationCidrBlock,GatewayId,NatGatewayId,State]' \
  --output table

# Replace existing route (e.g., failover NAT)
aws ec2 replace-route \
  --route-table-id rtb-xxx \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id nat-yyy
```

---

## Route Table for VPC Endpoints

```
Route Table with S3 Gateway Endpoint:
┌────────────────────────────────────────────────────────┐
│  Destination        Target                             │
│  ─────────────────  ──────────────────────             │
│  10.0.0.0/16        local                              │
│  pl-xxx (S3 IPs)    vpce-xxx  ◄── Auto-added by AWS   │
│  0.0.0.0/0          nat-xxx                            │
└────────────────────────────────────────────────────────┘

S3 traffic → matches pl-xxx → goes via VPC endpoint (free!)
Other traffic → matches 0.0.0.0/0 → goes via NAT (costs $)
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Instance can't reach internet | Subnet using main RT (no IGW route) | Associate subnet with a custom RT that has IGW route |
| Private subnet reaching internet | Main RT has 0.0.0.0/0 → IGW | Remove IGW route from main RT; use custom RT |
| Peered VPC traffic not flowing | Missing route to peer CIDR | Add route: peer CIDR → pcx-xxx in both VPCs' RTs |
| S3 traffic going through NAT | No VPC endpoint configured | Create S3 Gateway Endpoint and add to RT |
| Route table at 50 route limit | Too many routes | Use Transit Gateway or aggregate CIDRs |

---

## Real-World Scenario: Failover NAT Gateway

```bash
#!/bin/bash
# Monitor NAT Gateway health and failover to backup
PRIMARY_NAT="nat-primary"
BACKUP_NAT="nat-backup"
RT_ID="rtb-private"

# Check primary NAT status
STATUS=$(aws ec2 describe-nat-gateways \
  --nat-gateway-ids $PRIMARY_NAT \
  --query 'NatGateways[0].State' --output text)

if [ "$STATUS" != "available" ]; then
  echo "Primary NAT down, failing over..."
  aws ec2 replace-route \
    --route-table-id $RT_ID \
    --destination-cidr-block 0.0.0.0/0 \
    --nat-gateway-id $BACKUP_NAT
fi
```

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Route Table** | Set of rules determining where traffic goes. Every subnet needs one. |
| **Local Route** | Auto-added, non-deletable. Enables intra-VPC communication. |
| **Main RT** | Default for unassociated subnets. Keep minimal for safety. |
| **Most Specific Wins** | Longest prefix match determines which route is used. |
| **Custom RT** | Create per tier: public (→ IGW), private (→ NAT), data (local only). |
| **Gateway Endpoint** | Adds prefix list route to RT — S3/DynamoDB traffic stays in AWS. |

---

## Quick Revision Questions

1. **What happens if a subnet is not explicitly associated with any route table?**
   > It uses the VPC's main route table.

2. **How does AWS determine which route to use when multiple match?**
   > Longest prefix match — the most specific route (largest CIDR prefix number) wins.

3. **Can you delete the local route from a route table?**
   > No. The local route is automatically added and cannot be removed.

4. **Why should you not add an IGW route to the main route table?**
   > Because any new subnet will automatically become public, violating least-privilege.

5. **What is the `replace-route` command used for?**
   > To swap a route target (e.g., failover from one NAT GW to another) without deleting/recreating.

---

[← Previous: 3.2 Subnets](02-subnets-public-private.md) | [Next: 3.4 Internet & NAT Gateways →](04-internet-gateway-nat-gateway.md)

[← Back to README](../README.md)
