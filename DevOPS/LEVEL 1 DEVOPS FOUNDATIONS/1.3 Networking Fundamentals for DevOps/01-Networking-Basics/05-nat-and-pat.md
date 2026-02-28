# Chapter 5: NAT and PAT

## Overview

Network Address Translation (NAT) and Port Address Translation (PAT) allow devices with private IP addresses to communicate with the public internet. NAT is a cornerstone of cloud networking — every time your private EC2 instance downloads a package from the internet, NAT is involved. Understanding NAT is essential for designing secure, cost-effective cloud architectures.

---

## 5.1 What Is NAT?

NAT translates private IP addresses to public IP addresses (and vice versa) as traffic crosses a network boundary.

```
┌──────────────────────────────────────────────────────────────┐
│                    NAT CONCEPT                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  BEFORE NAT:                                                 │
│  Private IPs cannot reach the internet directly              │
│                                                              │
│  ┌──────────┐          ╳╳╳╳╳╳╳╳╳          ┌──────────┐      │
│  │ 10.0.1.5 │────────▶ BLOCKED! ────────▶ │ Internet │      │
│  └──────────┘          ╳╳╳╳╳╳╳╳╳          └──────────┘      │
│                                                              │
│  WITH NAT:                                                   │
│  NAT device translates private → public                      │
│                                                              │
│  ┌──────────┐     ┌─────────┐          ┌──────────────┐      │
│  │ 10.0.1.5 │────▶│   NAT   │────────▶ │   Internet   │     │
│  └──────────┘     │ Gateway │          │ (sees public │     │
│                   │203.0.113│          │  IP only)    │     │
│  Source:10.0.1.5  │   .10   │          └──────────────┘     │
│         becomes   └─────────┘                                │
│       203.0.113.10                                           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.2 Types of NAT

```
┌──────────────────────────────────────────────────────────────┐
│                    TYPES OF NAT                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. STATIC NAT (1:1 Mapping)                                 │
│     ┌────────────┐          ┌────────────┐                   │
│     │ 10.0.1.5   │ ◀══════▶ │ 54.10.20.1 │                  │
│     └────────────┘          └────────────┘                   │
│     One private IP ↔ One public IP (permanent)               │
│     Use: Servers needing consistent public access            │
│                                                              │
│  2. DYNAMIC NAT (Pool-based)                                 │
│     ┌────────────┐          ┌────────────┐                   │
│     │ 10.0.1.5   │ ───────▶ │ Pool:      │                  │
│     │ 10.0.1.6   │ ───────▶ │ 54.10.20.1 │                  │
│     │ 10.0.1.7   │ ───────▶ │ 54.10.20.2 │                  │
│     └────────────┘          │ 54.10.20.3 │                   │
│                             └────────────┘                   │
│     Many private IPs → Pool of public IPs (temporary)        │
│     Use: When you have multiple public IPs available         │
│                                                              │
│  3. PAT / NAT Overload (Many:1)                              │
│     ┌────────────┐          ┌────────────┐                   │
│     │ 10.0.1.5   │:50001 ──▶│            │                   │
│     │ 10.0.1.6   │:50002 ──▶│ 54.10.20.1 │                  │
│     │ 10.0.1.7   │:50003 ──▶│ (one IP!)  │                  │
│     └────────────┘          └────────────┘                   │
│     Many private IPs share ONE public IP using port numbers  │
│     Use: Most common! Home routers, AWS NAT Gateway          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.3 PAT (Port Address Translation) — Deep Dive

PAT is the most common form of NAT. It's what your home router and cloud NAT gateways use.

```
┌──────────────────────────────────────────────────────────────┐
│                PAT IN ACTION                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Internal Network                 NAT Device        Internet │
│                                                              │
│  10.0.1.5:45000 ──┐                                         │
│                    │         ┌──────────┐                    │
│  10.0.1.6:45001 ──┼────────▶│203.0.113 │────▶ google.com   │
│                    │         │   .10    │       :443         │
│  10.0.1.7:45002 ──┘         └──────────┘                    │
│                                                              │
│                                                              │
│  NAT Translation Table:                                      │
│  ┌───────────────────┬─────────────────────┬────────────────┐│
│  │ Internal          │ External            │ Destination    ││
│  ├───────────────────┼─────────────────────┼────────────────┤│
│  │ 10.0.1.5:45000    │ 203.0.113.10:50001  │ google:443    ││
│  │ 10.0.1.6:45001    │ 203.0.113.10:50002  │ google:443    ││
│  │ 10.0.1.7:45002    │ 203.0.113.10:50003  │ google:443    ││
│  └───────────────────┴─────────────────────┴────────────────┘│
│                                                              │
│  Response from google.com:443 → 203.0.113.10:50001          │
│  NAT looks up table → forwards to 10.0.1.5:45000            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.4 NAT in AWS

### NAT Gateway Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                 AWS NAT GATEWAY ARCHITECTURE                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌────────────────── VPC: 10.0.0.0/16 ──────────────────────┐   │
│  │                                                           │   │
│  │  Public Subnet (10.0.1.0/24)                              │   │
│  │  ┌──────────────────────────────────────────────────┐     │   │
│  │  │                                                  │     │   │
│  │  │  ┌─────────────┐      ┌──────────────┐          │     │   │
│  │  │  │ NAT Gateway │      │ Internet     │          │     │   │
│  │  │  │ EIP:        │─────▶│ Gateway (IGW)│──▶Internet│    │   │
│  │  │  │ 54.x.x.x   │      │              │          │     │   │
│  │  │  └──────▲──────┘      └──────────────┘          │     │   │
│  │  │         │                                        │     │   │
│  │  └─────────┼────────────────────────────────────────┘     │   │
│  │            │                                              │   │
│  │  Private Subnet (10.0.2.0/24)                             │   │
│  │  ┌─────────┼────────────────────────────────────────┐     │   │
│  │  │         │                                        │     │   │
│  │  │  ┌──────┴──────┐   ┌──────────────┐              │     │   │
│  │  │  │ Route Table │   │ App Server   │              │     │   │
│  │  │  │ 0.0.0.0/0   │   │ 10.0.2.20   │              │     │   │
│  │  │  │ → NAT GW    │   │ (no public   │              │     │   │
│  │  │  └─────────────┘   │  IP)         │              │     │   │
│  │  │                    └──────────────┘              │     │   │
│  │  └──────────────────────────────────────────────────┘     │   │
│  │                                                           │   │
│  └───────────────────────────────────────────────────────────┘   │
│                                                                  │
│  Traffic flow: App Server → NAT GW → IGW → Internet             │
│  Return: Internet → IGW → NAT GW → App Server                   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

### Setting Up NAT Gateway in AWS

```bash
# Step 1: Allocate an Elastic IP
EIP_ALLOC=$(aws ec2 allocate-address --domain vpc \
  --query 'AllocationId' --output text)

# Step 2: Create NAT Gateway in public subnet
NAT_GW=$(aws ec2 create-nat-gateway \
  --subnet-id subnet-public-12345 \
  --allocation-id $EIP_ALLOC \
  --query 'NatGateway.NatGatewayId' --output text)

# Step 3: Add route in private subnet's route table
aws ec2 create-route \
  --route-table-id rtb-private-12345 \
  --destination-cidr-block 0.0.0.0/0 \
  --nat-gateway-id $NAT_GW
```

### Terraform Example

```hcl
# Elastic IP for NAT Gateway
resource "aws_eip" "nat" {
  domain = "vpc"
  tags = { Name = "nat-eip" }
}

# NAT Gateway (must be in public subnet)
resource "aws_nat_gateway" "main" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.public.id
  tags = { Name = "main-nat-gw" }
}

# Route in private subnet route table
resource "aws_route" "private_nat" {
  route_table_id         = aws_route_table.private.id
  destination_cidr_block = "0.0.0.0/0"
  nat_gateway_id         = aws_nat_gateway.main.id
}
```

---

## 5.5 NAT Gateway vs NAT Instance

```
┌──────────────────────────────────────────────────────────────┐
│            NAT GATEWAY vs NAT INSTANCE (AWS)                 │
├──────────────────┬──────────────────┬────────────────────────┤
│  Feature         │ NAT Gateway      │ NAT Instance           │
├──────────────────┼──────────────────┼────────────────────────┤
│  Managed by      │ AWS (fully)      │ You (EC2 instance)     │
│  Availability    │ Highly available │ Single point of failure│
│  Bandwidth       │ Up to 100 Gbps   │ Depends on instance    │
│  Maintenance     │ None required    │ Patching, monitoring   │
│  Cost            │ ~$0.045/hr + data│ Instance cost          │
│  Security Groups │ Not supported    │ Supported              │
│  Bastion host    │ Cannot be used   │ Can double as bastion  │
│  Port forwarding │ Not supported    │ Supported              │
│  Recommended     │ YES (production) │ Only for cost savings  │
└──────────────────┴──────────────────┴────────────────────────┘
```

---

## 5.6 NAT in Docker and Kubernetes

### Docker NAT

```
┌──────────────────────────────────────────────────────────────┐
│                DOCKER NAT (iptables)                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Host Machine (192.168.1.100)                                │
│  ┌───────────────────────────────────────────┐               │
│  │ Docker Bridge (docker0): 172.17.0.1       │               │
│  │                                           │               │
│  │ ┌──────────────┐   ┌──────────────┐       │               │
│  │ │ Container A  │   │ Container B  │       │               │
│  │ │ 172.17.0.2   │   │ 172.17.0.3   │       │               │
│  │ │ Port 80      │   │ Port 3000    │       │               │
│  │ └──────┬───────┘   └──────┬───────┘       │               │
│  │        │                  │               │               │
│  │        ▼ NAT              ▼ NAT           │               │
│  │   host:8080          host:3000            │               │
│  └────────┬──────────────────┬───────────────┘               │
│           │                  │                               │
│           ▼                  ▼                               │
│     192.168.1.100:8080  192.168.1.100:3000                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘

# Docker port mapping = NAT
docker run -d -p 8080:80 nginx
# External: 192.168.1.100:8080 → Container: 172.17.0.2:80
```

### Kubernetes NAT

```bash
# Kubernetes uses iptables/IPVS for NAT
# ClusterIP services NAT traffic to pod IPs

# View NAT rules
sudo iptables -t nat -L -n -v | grep KUBE
```

---

## 5.7 NAT Traversal Problems

NAT can cause issues with certain protocols and applications:

```
┌──────────────────────────────────────────────────────────────┐
│              NAT TRAVERSAL CHALLENGES                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Problem: Inbound connections to NAT'd devices               │
│                                                              │
│  ┌────────┐     ┌─────────┐     ┌─────────┐                 │
│  │External│────▶│  NAT    │──╳──│ Internal│                  │
│  │ Client │     │ Gateway │     │ Server  │                  │
│  └────────┘     └─────────┘     └─────────┘                  │
│                                                              │
│  NAT doesn't know WHERE to forward incoming connections!     │
│                                                              │
│  Solutions:                                                  │
│  ├── Port forwarding (static NAT mapping)                    │
│  ├── UPnP (automatic port mapping)                           │
│  ├── STUN/TURN servers (WebRTC, VoIP)                        │
│  └── VPN tunnel (bypass NAT entirely)                        │
│                                                              │
│  Affected Applications:                                      │
│  ├── VoIP / SIP                                              │
│  ├── Online gaming                                           │
│  ├── P2P file sharing                                        │
│  └── Video conferencing                                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.8 Cost Optimization

**[>] Best Practices for NAT Gateway Costs**

NAT Gateway charges: ~$0.045/hr + $0.045/GB processed

```
┌──────────────────────────────────────────────────────────────┐
│              NAT GATEWAY COST OPTIMIZATION                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Use VPC Endpoints for AWS services                       │
│     ├── S3 Gateway Endpoint (free!)                          │
│     ├── DynamoDB Gateway Endpoint (free!)                    │
│     └── Avoids routing through NAT GW                        │
│                                                              │
│  2. One NAT GW per AZ (for HA) — not one per subnet         │
│                                                              │
│  3. Use S3 VPC endpoint for ECR image pulls                  │
│                                                              │
│  4. Consider NAT Instance for dev/test                       │
│     (t3.micro ~$0.0104/hr vs $0.045/hr)                      │
│                                                              │
│  5. Monitor NAT GW metrics:                                  │
│     aws cloudwatch get-metric-statistics \                   │
│       --namespace AWS/NATGateway \                           │
│       --metric-name BytesOutToDestination \                  │
│       --dimensions Name=NatGatewayId,Value=nat-xxxxx         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.9 Troubleshooting Tips

| Problem | Likely Cause | Solution |
|---------|-------------|----------|
| Private instance can't reach internet | No NAT GW or wrong route | Create NAT GW, add 0.0.0.0/0 route |
| NAT GW in wrong subnet | NAT GW must be in PUBLIC subnet | Move to public subnet with IGW route |
| Intermittent connectivity via NAT | Port exhaustion | Check ErrorPortAllocation metric |
| High NAT GW data transfer costs | All traffic through NAT | Use VPC endpoints for AWS services |
| NAT GW shows "Failed" state | No EIP or subnet issue | Verify EIP allocation and subnet |
| Docker container can't reach internet | iptables NAT rules missing | Check Docker daemon, restart Docker |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| NAT | Translates private IPs to public IPs for internet access |
| Static NAT | 1:1 permanent mapping (private ↔ public) |
| Dynamic NAT | Many:Pool temporary mapping |
| PAT | Many:1 mapping using port numbers (most common) |
| NAT Gateway (AWS) | Managed, HA, recommended for production |
| NAT Instance (AWS) | Self-managed EC2, cheaper for dev/test |
| Placement | NAT Gateway MUST be in a public subnet |
| Routing | Private subnet route: 0.0.0.0/0 → NAT Gateway |
| Cost savings | Use VPC Endpoints to bypass NAT for AWS services |

---

## Quick Revision Questions

1. **What is the difference between NAT and PAT?**
2. **Why must a NAT Gateway be placed in a public subnet?**
3. **Draw the traffic flow: private EC2 instance → internet (include all components).**
4. **How does Docker use NAT when you run `docker run -p 8080:80 nginx`?**
5. **You notice high NAT Gateway costs. What are three ways to reduce them?**
6. **What is the NAT Translation Table, and how does PAT use it to handle multiple connections from different hosts?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Public vs Private IPs](04-public-vs-private-ips.md) | [README](../README.md) | [TCP vs UDP →](../02-Protocols/01-tcp-vs-udp.md) |
