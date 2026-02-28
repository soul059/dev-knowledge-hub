# Chapter 2: Routing Basics

## Overview

Routing is the process of determining the path that network packets take from source to destination across interconnected networks. Routers make forwarding decisions based on routing tables. In cloud environments, route tables in VPCs serve the same purpose. Understanding routing is essential for designing multi-subnet architectures, VPN connections, and debugging connectivity issues.

---

## 2.1 How Routing Works

```
┌──────────────────────────────────────────────────────────────┐
│                  ROUTING DECISION FLOW                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Packet arrives at router: "Where should I go?"              │
│                                                              │
│  ┌──────────────┐                                            │
│  │   Incoming   │                                            │
│  │   Packet     │                                            │
│  │  Dest: 10.0. │                                            │
│  │    2.50      │                                            │
│  └──────┬───────┘                                            │
│         │                                                    │
│         ▼                                                    │
│  ┌──────────────────────────────────────────┐                │
│  │        ROUTING TABLE LOOKUP              │                │
│  │                                          │                │
│  │  Destination      Next Hop      Interface│                │
│  │  ─────────────────────────────────────── │                │
│  │  10.0.1.0/24      local         eth0     │                │
│  │  10.0.2.0/24      10.0.1.1      eth1     │ ← Match!      │
│  │  10.0.3.0/24      10.0.1.2      eth2     │                │
│  │  0.0.0.0/0        10.0.1.254    eth0     │ ← Default     │
│  │                                          │                │
│  └──────────────────────────────────────────┘                │
│         │                                                    │
│         ▼                                                    │
│  Route to 10.0.2.50 → Forward via eth1 to 10.0.1.1          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.2 Routing Table

```
┌──────────────────────────────────────────────────────────────┐
│              ROUTING TABLE COMPONENTS                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Each entry contains:                                        │
│  ┌──────────────────┬──────────────────────────────────────┐ │
│  │ Destination      │ Network/host to reach                │ │
│  │ Subnet Mask/CIDR │ How many IPs in destination          │ │
│  │ Next Hop/Gateway │ Where to forward the packet          │ │
│  │ Interface        │ Which network interface to use       │ │
│  │ Metric           │ Preference (lower = better)          │ │
│  └──────────────────┴──────────────────────────────────────┘ │
│                                                              │
│  Special entries:                                            │
│  ├── Default Route: 0.0.0.0/0 → "catch-all" for unknown     │
│  │   destinations (usually points to internet gateway)       │
│  └── Local Route: For directly connected networks            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### View Routing Tables

```bash
# Linux
ip route show
# Output:
# default via 10.0.1.1 dev eth0
# 10.0.1.0/24 dev eth0 proto kernel scope link src 10.0.1.50
# 10.0.2.0/24 via 10.0.1.1 dev eth0

# Windows
route print

# macOS
netstat -rn

# AWS: Describe route table
aws ec2 describe-route-tables --route-table-id rtb-12345678
```

---

## 2.3 Static vs Dynamic Routing

```
┌──────────────────────────────────────────────────────────────┐
│           STATIC vs DYNAMIC ROUTING                          │
├──────────────────────┬──────────────────┬────────────────────┤
│ Feature              │ Static           │ Dynamic            │
├──────────────────────┼──────────────────┼────────────────────┤
│ Configuration        │ Manual           │ Automatic          │
│ Scalability          │ Low              │ High               │
│ Adapts to failures   │ No               │ Yes                │
│ CPU/Memory overhead  │ None             │ Some               │
│ Security             │ More controlled  │ Needs auth         │
│ Use cases            │ Small networks,  │ Large networks,    │
│                      │ cloud route      │ ISPs, data centers │
│                      │ tables           │                    │
│ Examples             │ ip route add     │ OSPF, BGP, EIGRP   │
│ Cloud equivalent     │ VPC route tables │ Transit gateway    │
│                      │                  │ route propagation  │
└──────────────────────┴──────────────────┴────────────────────┘
```

### Common Dynamic Routing Protocols

```
┌──────────────────────────────────────────────────────────────┐
│  Protocol │ Type      │ Scope      │ DevOps Relevance        │
├───────────┼───────────┼────────────┼─────────────────────────┤
│  OSPF     │ Link-state│ Internal   │ On-prem data centers    │
│  BGP      │ Path-vect.│ Inter-AS   │ VPN, Direct Connect,    │
│           │           │            │ multi-cloud             │
│  EIGRP    │ Hybrid    │ Internal   │ Cisco environments      │
│  RIP      │ Distance  │ Internal   │ Legacy, avoid           │
└───────────┴───────────┴────────────┴─────────────────────────┘

BGP is the most important for DevOps:
├── AWS Direct Connect uses BGP
├── VPN connections use BGP
├── Transit Gateway route propagation uses BGP
└── The entire internet runs on BGP
```

---

## 2.4 Routing in Cloud (AWS)

```
┌──────────────────────────────────────────────────────────────┐
│              AWS VPC ROUTE TABLE EXAMPLE                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PUBLIC SUBNET Route Table:                                  │
│  ┌──────────────────┬──────────────────┬──────────────┐      │
│  │ Destination      │ Target           │ Purpose      │      │
│  ├──────────────────┼──────────────────┼──────────────┤      │
│  │ 10.0.0.0/16      │ local            │ VPC internal │      │
│  │ 0.0.0.0/0        │ igw-12345678     │ Internet     │      │
│  └──────────────────┴──────────────────┴──────────────┘      │
│                                                              │
│  PRIVATE SUBNET Route Table:                                 │
│  ┌──────────────────┬──────────────────┬──────────────┐      │
│  │ Destination      │ Target           │ Purpose      │      │
│  ├──────────────────┼──────────────────┼──────────────┤      │
│  │ 10.0.0.0/16      │ local            │ VPC internal │      │
│  │ 0.0.0.0/0        │ nat-12345678     │ NAT Gateway  │      │
│  │ 172.16.0.0/16    │ pcx-12345678     │ VPC Peering  │      │
│  │ 192.168.0.0/16   │ vgw-12345678     │ VPN/DC conn  │      │
│  └──────────────────┴──────────────────┴──────────────┘      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Terraform: Route Tables

```hcl
# Public route table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = { Name = "public-rt" }
}

# Private route table
resource "aws_route_table" "private" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.main.id
  }

  tags = { Name = "private-rt" }
}

# Associate subnet with route table
resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

---

## 2.5 Longest Prefix Match

When multiple routes match a destination, the **most specific route wins**:

```
Route Table:
  10.0.0.0/16   → Gateway A
  10.0.1.0/24   → Gateway B
  10.0.1.128/25 → Gateway C

Packet to 10.0.1.200:
  Matches 10.0.0.0/16   ✓ (but /16 is least specific)
  Matches 10.0.1.0/24   ✓ (more specific)
  Matches 10.0.1.128/25 ✓ (MOST specific → WINS!)
  
  → Routes to Gateway C
```

---

## 2.6 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "No route to host" | Missing route entry | Add appropriate route |
| Asymmetric routing | Different paths in/out | Ensure consistent route tables |
| Black hole route | Route target doesn't exist | Check target (deleted NAT GW?) |
| Private subnet can't reach internet | Missing 0.0.0.0/0 → NAT GW | Add default route to NAT Gateway |
| VPC peering traffic not flowing | Missing route for peer CIDR | Add peering route in both VPCs |

```bash
# Trace the routing path
traceroute 10.0.2.50        # Linux
tracert 10.0.2.50           # Windows

# Check if a route exists
ip route get 10.0.2.50      # Linux — shows which route is used

# Add a static route (Linux)
sudo ip route add 10.0.2.0/24 via 10.0.1.1 dev eth0

# Delete a route
sudo ip route del 10.0.2.0/24
```

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Routing | Determines packet path across networks |
| Route Table | Destination → Next Hop → Interface mapping |
| Default Route | 0.0.0.0/0 — catch-all for unknown destinations |
| Longest Prefix Match | Most specific route wins (/25 > /24 > /16) |
| Static | Manual configuration, cloud route tables |
| Dynamic | Automatic (OSPF, BGP), adapts to changes |
| BGP | Used for VPN, Direct Connect, internet routing |
| AWS | Route tables associated with subnets |

---

## Quick Revision Questions

1. **What is the default route, and why is it important?**
2. **Explain the "longest prefix match" rule with an example.**
3. **What's the difference between a public subnet route table and a private subnet route table in AWS?**
4. **When would you use BGP in a DevOps context?**
5. **You added a VPC peering connection but instances still can't communicate. What did you forget?**
6. **How do you check which route a packet will take on a Linux system?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← VLANs](01-vlans.md) | [README](../README.md) | [Load Balancing →](03-load-balancing.md) |
