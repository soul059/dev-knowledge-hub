# Chapter 4: Public vs Private IPs

## Overview

Understanding the difference between public and private IP addresses is critical for cloud architecture, security design, and network connectivity. Private IPs are used within internal networks while public IPs are used for internet-facing communication. Every DevOps engineer must know when to use each, how they interact, and the security implications.

---

## 4.1 Private IP Address Ranges (RFC 1918)

These ranges are reserved for use within private networks. They are **not routable** on the public internet.

```
┌──────────────────────────────────────────────────────────────┐
│               PRIVATE IP ADDRESS RANGES                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Class A:   10.0.0.0     -  10.255.255.255                   │
│             10.0.0.0/8                                       │
│             16,777,216 addresses                             │
│             → Used for: Large enterprises, cloud VPCs        │
│                                                              │
│  Class B:   172.16.0.0   -  172.31.255.255                   │
│             172.16.0.0/12                                    │
│             1,048,576 addresses                              │
│             → Used for: Medium networks, Docker default      │
│                                                              │
│  Class C:   192.168.0.0  -  192.168.255.255                  │
│             192.168.0.0/16                                   │
│             65,536 addresses                                 │
│             → Used for: Home/office networks, labs           │
│                                                              │
│  Special:   169.254.0.0/16 (Link-local / APIPA)             │
│             → Auto-assigned when DHCP fails                  │
│                                                              │
│  Loopback:  127.0.0.0/8                                      │
│             → localhost (127.0.0.1)                           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.2 Public vs Private: Side-by-Side

```
┌──────────────────────────────────────────────────────────────┐
│           PUBLIC vs PRIVATE IP COMPARISON                     │
├──────────────────────┬───────────────────────────────────────┤
│  Feature             │  Public IP        │  Private IP       │
├──────────────────────┼───────────────────┼───────────────────┤
│  Routable on internet│  Yes              │  No               │
│  Uniqueness          │  Globally unique  │  Unique per LAN   │
│  Assigned by         │  ISP / Cloud      │  DHCP / Admin     │
│  Cost                │  Paid (limited)   │  Free (unlimited) │
│  Security            │  Exposed          │  Hidden behind NAT│
│  Example             │  54.231.50.10     │  10.0.1.50        │
│  Reusability         │  Not reusable     │  Reused everywhere│
│  Direct access       │  From anywhere    │  Only within LAN  │
└──────────────────────┴───────────────────┴───────────────────┘
```

---

## 4.3 How Public and Private IPs Work Together

```
┌──────────────────────────────────────────────────────────────────┐
│           HOW TRAFFIC FLOWS: PRIVATE → PUBLIC → INTERNET         │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Your Private Network                    The Internet            │
│  ┌───────────────────┐                                           │
│  │                   │                                           │
│  │ ┌──────────┐      │    ┌─────────┐    ┌────────────────┐     │
│  │ │ Server A │      │    │         │    │                │     │
│  │ │10.0.1.10 │──────┼───▶│  NAT /  │───▶│  google.com    │     │
│  │ └──────────┘      │    │ Router  │    │  142.250.80.46 │     │
│  │                   │    │         │    │                │     │
│  │ ┌──────────┐      │    │ Public  │    └────────────────┘     │
│  │ │ Server B │──────┼───▶│   IP:   │                           │
│  │ │10.0.1.11 │      │    │203.0.113│                           │
│  │ └──────────┘      │    │   .5    │                           │
│  │                   │    └─────────┘                           │
│  └───────────────────┘                                           │
│                                                                  │
│  Both servers share ONE public IP (203.0.113.5)                  │
│  via NAT (Network Address Translation)                           │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 4.4 Public and Private IPs in Cloud (AWS)

```
┌──────────────────────────────────────────────────────────────┐
│              AWS IP ADDRESSING                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────── VPC: 10.0.0.0/16 ───────────────────┐    │
│  │                                                      │    │
│  │  Public Subnet (10.0.1.0/24)                         │    │
│  │  ┌──────────────────────────────────┐                │    │
│  │  │  EC2 Instance (Web Server)       │                │    │
│  │  │  ├── Private IP: 10.0.1.15      │                │    │
│  │  │  ├── Public IP:  54.210.33.100  │ ← Internet     │    │
│  │  │  └── (or Elastic IP)            │   accessible   │    │
│  │  └──────────────────────────────────┘                │    │
│  │                                                      │    │
│  │  Private Subnet (10.0.2.0/24)                        │    │
│  │  ┌──────────────────────────────────┐                │    │
│  │  │  EC2 Instance (App Server)       │                │    │
│  │  │  ├── Private IP: 10.0.2.20      │ ← No direct   │    │
│  │  │  └── Public IP:  NONE           │   internet     │    │
│  │  └──────────────────────────────────┘   access       │    │
│  │                                                      │    │
│  │  Private Subnet (10.0.3.0/24)                        │    │
│  │  ┌──────────────────────────────────┐                │    │
│  │  │  RDS Instance (Database)         │                │    │
│  │  │  ├── Private IP: 10.0.3.50      │                │    │
│  │  │  └── Public IP:  NONE           │                │    │
│  │  └──────────────────────────────────┘                │    │
│  │                                                      │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Types of Public IPs in AWS

| Type | Persistence | Cost | Use Case |
|------|------------|------|----------|
| Dynamic Public IP | Changes on stop/start | Free (when in use) | Dev/test instances |
| Elastic IP (EIP) | Static, persists | Free if attached; charged if idle | Production servers, NAT gateways |

```bash
# Allocate an Elastic IP
aws ec2 allocate-address --domain vpc

# Associate with an instance
aws ec2 associate-address \
  --instance-id i-1234567890abcdef0 \
  --allocation-id eipalloc-12345678

# View instance IPs
aws ec2 describe-instances \
  --instance-id i-1234567890abcdef0 \
  --query 'Reservations[].Instances[].[PrivateIpAddress,PublicIpAddress]'
```

---

## 4.5 Special IP Addresses

```
┌──────────────────────────────────────────────────────────────┐
│                SPECIAL IP ADDRESSES                          │
├──────────────────┬───────────────────────────────────────────┤
│  Address         │  Purpose                                  │
├──────────────────┼───────────────────────────────────────────┤
│  0.0.0.0         │  "All interfaces" or default route        │
│  127.0.0.1       │  Loopback / localhost                     │
│  255.255.255.255 │  Broadcast to all hosts on local network  │
│  169.254.x.x     │  Link-local (APIPA) — DHCP failure        │
│  224.0.0.0/4     │  Multicast addresses                      │
│  100.64.0.0/10   │  Carrier-grade NAT (CGNAT)                │
│  198.51.100.0/24 │  Documentation/examples (RFC 5737)        │
│  203.0.113.0/24  │  Documentation/examples (RFC 5737)        │
│  192.0.2.0/24    │  Documentation/examples (RFC 5737)        │
└──────────────────┴───────────────────────────────────────────┘
```

---

## 4.6 Best Practices for DevOps

### Do's
- **Always use private IPs** for internal communication between services
- **Use Elastic IPs** for services that need stable public addresses
- **Place databases in private subnets** — never assign public IPs
- **Use NAT Gateways** for private instances that need outbound internet
- **Plan CIDR ranges** to avoid overlap across VPCs and on-premises networks

### Don'ts
- **Don't expose** services on public IPs unless absolutely necessary
- **Don't hardcode** public IPs — they can change; use DNS names
- **Don't open 0.0.0.0/0** on sensitive ports (SSH, DB ports)
- **Don't forget** to release unused Elastic IPs (they cost money)

---

## 4.7 Real-World Scenarios

### [~] Scenario 1: Microservices Communication

```
┌─────────────────────────────────────────────────────────┐
│  Service A calls Service B                              │
│                                                         │
│  WRONG (via public IP):                                 │
│  Service A → Internet → Public IP → Service B           │
│  ├── Slow (unnecessary internet hop)                    │
│  ├── Costs money (data transfer charges)                │
│  └── Security risk (traffic leaves VPC)                 │
│                                                         │
│  RIGHT (via private IP):                                │
│  Service A → Private Network → Private IP → Service B   │
│  ├── Fast (stays within VPC)                            │
│  ├── Free (no data transfer charges)                    │
│  └── Secure (never leaves VPC)                          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### [~] Scenario 2: Docker Container Networking

```bash
# Docker uses private IPs internally
docker network inspect bridge
# Default bridge: 172.17.0.0/16

# Containers communicate via private IPs
docker run -d --name web nginx       # Gets 172.17.0.2
docker run -d --name api node-app    # Gets 172.17.0.3

# Port mapping exposes to host's public/private IP
docker run -d -p 80:80 nginx
# Maps host:80 → container:80
# Accessible via host's public or private IP
```

### [~] Scenario 3: Kubernetes Pod Networking

```
Kubernetes Cluster:
┌──────────────────────────────────────────────────┐
│  Node 1 (10.0.1.10)         Node 2 (10.0.1.11)  │
│  ┌─────────────────┐        ┌─────────────────┐  │
│  │ Pod A           │        │ Pod C            │  │
│  │ 10.244.0.5      │───────▶│ 10.244.1.8      │  │
│  │ (cluster-only)  │        │ (cluster-only)   │  │
│  ├─────────────────┤        └─────────────────┘  │
│  │ Pod B           │                              │
│  │ 10.244.0.6      │        Pod IPs are private   │
│  └─────────────────┘        to the cluster        │
│                                                   │
│  Service (ClusterIP): 10.96.0.100  ← internal     │
│  Service (LoadBalancer): 54.x.x.x  ← public       │
└──────────────────────────────────────────────────┘
```

---

## 4.8 Hands-On Commands

```bash
# Find your private IP
hostname -I                     # Linux
ipconfig | findstr IPv4         # Windows

# Find your public IP
curl ifconfig.me
curl ipinfo.io/ip
curl checkip.amazonaws.com

# Check if an IP is private (Python)
python3 -c "
import ipaddress
ip = ipaddress.ip_address('10.0.1.50')
print(f'{ip} is private: {ip.is_private}')
ip2 = ipaddress.ip_address('54.210.33.100')
print(f'{ip2} is private: {ip2.is_private}')
"

# AWS: Describe instance IPs
aws ec2 describe-instances --query \
  'Reservations[*].Instances[*].[InstanceId,PrivateIpAddress,PublicIpAddress]' \
  --output table
```

---

## 4.9 Troubleshooting Tips

| Problem | Likely Cause | Solution |
|---------|-------------|----------|
| Can't access instance from internet | No public IP or EIP | Assign public IP or Elastic IP |
| Services slow between instances | Using public IPs internally | Switch to private IP communication |
| Instance loses public IP on restart | Using dynamic public IP | Attach an Elastic IP instead |
| 169.254.x.x IP assigned | DHCP server unreachable | Check network config, DHCP service |
| VPC peering: can't reach peer VPC | CIDR overlap or missing route | Ensure non-overlapping CIDRs, add routes |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Private IPs | 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 — not internet routable |
| Public IPs | Globally unique, assigned by ISP/cloud — internet accessible |
| Elastic IP | Static public IP in AWS — persists across stop/start |
| NAT | Translates private IPs to public for outbound internet |
| Best Practice | Internal services use private IPs; only expose what's needed |
| Docker | Uses 172.17.0.0/16 by default for container networking |
| Kubernetes | Pod IPs are private to the cluster; Services expose externally |

---

## Quick Revision Questions

1. **Name the three RFC 1918 private IP ranges and their CIDR notations.**
2. **Why should microservices within the same VPC communicate via private IPs instead of public?**
3. **What's the difference between a dynamic public IP and an Elastic IP in AWS?**
4. **If a server gets the IP 169.254.x.x, what has likely gone wrong?**
5. **In a 3-tier architecture (web/app/db), which tier(s) should have public IPs?**
6. **How do Docker containers get their IP addresses, and are they public or private?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← CIDR Notation](03-cidr-notation.md) | [README](../README.md) | [NAT and PAT →](05-nat-and-pat.md) |
