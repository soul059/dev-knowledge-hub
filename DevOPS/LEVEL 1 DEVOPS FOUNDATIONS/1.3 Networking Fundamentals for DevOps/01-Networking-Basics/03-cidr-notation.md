# Chapter 3: CIDR Notation

## Overview

CIDR (Classless Inter-Domain Routing) replaced the old classful IP addressing system. It allows flexible allocation of IP addresses using a prefix length instead of rigid class boundaries. Every cloud platform, firewall rule, and routing table uses CIDR notation — this is non-negotiable knowledge for DevOps.

---

## 3.1 What Is CIDR?

CIDR notation combines an IP address with a **prefix length** that indicates how many bits are used for the network portion.

```
┌──────────────────────────────────────────────────────────────┐
│                    CIDR NOTATION                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Format:  <IP Address> / <Prefix Length>                     │
│                                                              │
│  Example: 10.0.0.0/16                                        │
│                                                              │
│  ┌──────────┐ ┌────────────┐                                 │
│  │ 10.0.0.0 │/│     16     │                                 │
│  └──────────┘ └────────────┘                                 │
│   IP Address   Number of bits                                │
│                for the network                               │
│                (remaining bits = hosts)                       │
│                                                              │
│  /16 means:                                                  │
│  ├── First 16 bits → Network portion                         │
│  ├── Remaining 16 bits → Host portion                        │
│  ├── Subnet mask: 255.255.0.0                                │
│  └── Total IPs: 2^16 = 65,536                                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.2 CIDR vs Classful Addressing

```
┌──────────────────────────────────────────────────────────────┐
│           CLASSFUL (Old) vs CIDR (Modern)                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  CLASSFUL (Wasteful):                                        │
│  ┌─────────────────────────────────────────┐                 │
│  │ Need 300 hosts?                         │                 │
│  │ Class C (/24) = 254 hosts → Too small!  │                 │
│  │ Class B (/16) = 65,534 hosts → Wastes   │                 │
│  │                  65,234 addresses!       │                 │
│  └─────────────────────────────────────────┘                 │
│                                                              │
│  CIDR (Efficient):                                           │
│  ┌─────────────────────────────────────────┐                 │
│  │ Need 300 hosts?                         │                 │
│  │ Use /23 = 510 usable hosts              │                 │
│  │ Much better fit! Only 210 wasted.       │                 │
│  └─────────────────────────────────────────┘                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.3 CIDR Prefix Cheat Sheet

```
┌──────┬──────────────────┬──────────────┬────────────┬──────────────────┐
│ CIDR │  Subnet Mask     │  Total IPs   │ Usable     │  Common Use      │
├──────┼──────────────────┼──────────────┼────────────┼──────────────────┤
│ /32  │ 255.255.255.255  │      1       │    1       │ Single host      │
│ /31  │ 255.255.255.254  │      2       │    2       │ Point-to-point   │
│ /30  │ 255.255.255.252  │      4       │    2       │ WAN link         │
│ /29  │ 255.255.255.248  │      8       │    6       │ Small segment    │
│ /28  │ 255.255.255.240  │     16       │   14       │ Small office     │
│ /27  │ 255.255.255.224  │     32       │   30       │ Department       │
│ /26  │ 255.255.255.192  │     64       │   62       │ Branch office    │
│ /25  │ 255.255.255.128  │    128       │  126       │ Medium network   │
│ /24  │ 255.255.255.0    │    256       │  254       │ Standard subnet  │
│ /23  │ 255.255.254.0    │    512       │  510       │ Two /24 blocks   │
│ /22  │ 255.255.252.0    │   1,024      │ 1,022      │ Large subnet     │
│ /21  │ 255.255.248.0    │   2,048      │ 2,046      │ Campus           │
│ /20  │ 255.255.240.0    │   4,096      │ 4,094      │ Large site       │
│ /16  │ 255.255.0.0      │  65,536      │ 65,534     │ VPC default      │
│ /12  │ 255.240.0.0      │ 1,048,576    │ 1,048,574  │ Large private    │
│ /8   │ 255.0.0.0        │ 16,777,216   │ 16,777,214 │ Class A network  │
│ /0   │ 0.0.0.0          │ ALL IPs      │   ALL      │ Default route    │
└──────┴──────────────────┴──────────────┴────────────┴──────────────────┘
```

**Quick formula**: Total IPs = 2^(32 - prefix)

---

## 3.4 CIDR in Cloud Environments

### AWS VPC CIDR Blocks

```
┌──────────────────────────────────────────────────────────────┐
│              AWS VPC CIDR PLANNING                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  VPC CIDR: 10.0.0.0/16  (65,536 IPs)                        │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Availability Zone 1 (us-east-1a)                   │     │
│  │  ┌────────────────┐  ┌────────────────┐             │     │
│  │  │  Public Subnet │  │ Private Subnet │             │     │
│  │  │ 10.0.1.0/24    │  │ 10.0.2.0/24   │             │     │
│  │  │ (251 usable*)  │  │ (251 usable*)  │             │     │
│  │  └────────────────┘  └────────────────┘             │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Availability Zone 2 (us-east-1b)                   │     │
│  │  ┌────────────────┐  ┌────────────────┐             │     │
│  │  │  Public Subnet │  │ Private Subnet │             │     │
│  │  │ 10.0.3.0/24    │  │ 10.0.4.0/24   │             │     │
│  │  │ (251 usable*)  │  │ (251 usable*)  │             │     │
│  │  └────────────────┘  └────────────────┘             │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                              │
│  * AWS reserves 5 IPs per subnet:                            │
│    .0 (network), .1 (VPC router), .2 (DNS),                  │
│    .3 (future), .255 (broadcast)                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Terraform Example

```hcl
# VPC with CIDR
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "production-vpc"
  }
}

# Public Subnet
resource "aws_subnet" "public_1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  availability_zone       = "us-east-1a"
  map_public_ip_on_launch = true

  tags = {
    Name = "public-subnet-1"
  }
}

# Private Subnet
resource "aws_subnet" "private_1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name = "private-subnet-1"
  }
}
```

---

## 3.5 CIDR in Security Rules

Security groups and firewall rules use CIDR extensively:

```bash
# Allow SSH from a specific IP
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.50/32      # /32 = single IP

# Allow HTTP from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0            # /0 = ALL IPs (the internet)

# Allow all traffic from within VPC
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol -1 \
  --cidr 10.0.0.0/16          # /16 = entire VPC
```

### Common CIDR Patterns in DevOps

```
┌──────────────────┬──────────────────────────────────────────┐
│  CIDR            │  Meaning in Security Rules               │
├──────────────────┼──────────────────────────────────────────┤
│  0.0.0.0/0       │  ALL IPv4 addresses (the internet)       │
│  ::/0            │  ALL IPv6 addresses (the internet)       │
│  10.0.0.0/16     │  All IPs in the 10.0.x.x VPC            │
│  10.0.1.0/24     │  All IPs in a specific subnet            │
│  203.0.113.50/32 │  A single specific IP address            │
│  192.168.0.0/16  │  All private 192.168.x.x addresses       │
│  172.16.0.0/12   │  All private 172.16-31.x.x addresses     │
│  10.0.0.0/8      │  All private 10.x.x.x addresses          │
└──────────────────┴──────────────────────────────────────────┘
```

---

## 3.6 CIDR Aggregation (Supernetting)

CIDR also enables **route aggregation** — combining multiple smaller networks into a single larger block to simplify routing.

```
┌──────────────────────────────────────────────────────────────┐
│              CIDR AGGREGATION EXAMPLE                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Without Aggregation (4 route entries):                      │
│  ├── 192.168.0.0/24                                          │
│  ├── 192.168.1.0/24                                          │
│  ├── 192.168.2.0/24                                          │
│  └── 192.168.3.0/24                                          │
│                                                              │
│  With Aggregation (1 route entry):                           │
│  └── 192.168.0.0/22    ← Covers all four /24 networks!      │
│                                                              │
│  Why? Binary view of third octet:                            │
│  192.168.  00000000  .0  → .0                                │
│  192.168.  00000001  .0  → .1                                │
│  192.168.  00000010  .0  → .2                                │
│  192.168.  00000011  .0  → .3                                │
│            ^^^^^^                                            │
│            First 6 bits same → /22 covers them all           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.7 Hands-On: Working with CIDR

```bash
# Using ipcalc (install: apt install ipcalc)
$ ipcalc 10.0.0.0/22
Address:   10.0.0.0             00001010.00000000.000000 00.00000000
Netmask:   255.255.252.0 = 22   11111111.11111111.111111 00.00000000
Wildcard:  0.0.3.255            00000000.00000000.000000 11.11111111
=>
Network:   10.0.0.0/22          00001010.00000000.000000 00.00000000
HostMin:   10.0.0.1             00001010.00000000.000000 00.00000001
HostMax:   10.0.3.254           00001010.00000000.000000 11.11111110
Broadcast: 10.0.3.255           00001010.00000000.000000 11.11111111
Hosts/Net: 1022

# Python CIDR tools
python3 -c "
import ipaddress

# Check if an IP is in a CIDR range
network = ipaddress.ip_network('10.0.0.0/22')
ip = ipaddress.ip_address('10.0.2.100')
print(f'{ip} in {network}? {ip in network}')  # True

# List all /24 subnets within a /22
for subnet in network.subnets(new_prefix=24):
    print(subnet)
"

# Check overlapping CIDRs
python3 -c "
import ipaddress
net1 = ipaddress.ip_network('10.0.0.0/22')
net2 = ipaddress.ip_network('10.0.2.0/24')
print(f'Overlaps? {net1.overlaps(net2)}')  # True
"
```

---

## 3.8 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "CIDR block conflicts with existing" | Overlapping ranges in VPC | Use non-overlapping CIDRs for peered VPCs |
| Cannot add route | Destination overlap | Check route table for existing broader CIDR |
| Security rule too permissive | Used 0.0.0.0/0 | Narrow down to specific CIDR blocks |
| Instance can't reach another | Different subnets, no route | Add CIDR-based route entry |
| VPC peering fails | Overlapping CIDRs | Plan CIDRs to be unique per VPC |

**[>] Best Practice**: Plan your CIDR allocation before building infrastructure. Use a spreadsheet or IPAM tool.

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| CIDR Notation | IP/prefix format (e.g., 10.0.0.0/16) replacing classful addressing |
| Prefix Length | Number of network bits (higher = smaller network) |
| /32 | Single host |
| /24 | 256 IPs (254 usable) — most common subnet size |
| /16 | 65,536 IPs — typical VPC size |
| /0 | All IPs — represents "the internet" in rules |
| Supernetting | Combining smaller CIDRs into a larger one |
| AWS reserves | 5 IPs per subnet (.0, .1, .2, .3, .255) |

---

## Quick Revision Questions

1. **What does 10.0.0.0/20 mean? How many total IP addresses does it contain?**
2. **You have 0.0.0.0/0 in a security group rule. What does this allow?**
3. **Why can't you peer two VPCs with CIDRs 10.0.0.0/16 and 10.0.5.0/24?**
4. **How many /24 subnets can you create inside a /16?**
5. **Convert the CIDR 172.16.0.0/12 to its subnet mask. What range of IPs does it cover?**
6. **In AWS, why does a /24 subnet give you only 251 usable IPs instead of 254?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← IP Addressing & Subnetting](02-ip-addressing-subnetting.md) | [README](../README.md) | [Public vs Private IPs →](04-public-vs-private-ips.md) |
