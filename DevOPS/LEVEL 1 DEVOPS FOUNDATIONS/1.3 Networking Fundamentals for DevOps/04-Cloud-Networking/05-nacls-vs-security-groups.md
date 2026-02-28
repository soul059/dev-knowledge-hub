# Chapter 5: NACLs vs Security Groups

## Overview

AWS provides two layers of network security: **Network Access Control Lists (NACLs)** at the subnet level and **Security Groups** at the instance level. Understanding how both work together — and their critical differences — is essential for building defense-in-depth and troubleshooting connectivity issues.

---

## 5.1 Defense in Depth

```
┌──────────────────────────────────────────────────────────────┐
│         TWO LAYERS OF VIRTUAL FIREWALL                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Internet Traffic                                            │
│        │                                                     │
│        ▼                                                     │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Layer 1: NACL (Subnet Boundary)                        │ │
│  │  ┌───────────────────────────────────────────────────┐  │ │
│  │  │ Rule# │ Type  │ Protocol │ Port │ Source  │ Allow │  │ │
│  │  │ 100   │ HTTP  │ TCP      │ 80   │ 0.0.0.0│ ALLOW │  │ │
│  │  │ 110   │ HTTPS │ TCP      │ 443  │ 0.0.0.0│ ALLOW │  │ │
│  │  │ 120   │ SSH   │ TCP      │ 22   │ 10.0.0 │ ALLOW │  │ │
│  │  │ *     │ ALL   │ ALL      │ ALL  │ 0.0.0.0│ DENY  │  │ │
│  │  └───────────────────────────────────────────────────┘  │ │
│  └─────────────────────────────────────────────────────────┘ │
│        │  (traffic that passes NACL)                         │
│        ▼                                                     │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  Layer 2: Security Group (Instance Boundary)            │ │
│  │  ┌───────────────────────────────────────────────────┐  │ │
│  │  │ Type  │ Protocol │ Port │ Source         │ Allow │  │ │
│  │  │ HTTP  │ TCP      │ 80   │ 0.0.0.0/0     │ ALLOW │  │ │
│  │  │ HTTPS │ TCP      │ 443  │ 0.0.0.0/0     │ ALLOW │  │ │
│  │  │ SSH   │ TCP      │ 22   │ sg-bastion     │ ALLOW │  │ │
│  │  │ (everything else implicitly denied)       │       │  │ │
│  │  └───────────────────────────────────────────────────┘  │ │
│  └─────────────────────────────────────────────────────────┘ │
│        │                                                     │
│        ▼                                                     │
│  ┌──────────────┐                                            │
│  │  EC2 Instance │ ← Traffic must pass BOTH layers           │
│  └──────────────┘                                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.2 Complete Comparison

| Feature | Security Group | NACL |
|---------|---------------|------|
| **Scope** | Instance (ENI) level | Subnet level |
| **Statefulness** | Stateful | Stateless |
| **Rule type** | Allow only | Allow AND Deny |
| **Rule evaluation** | All rules evaluated | Rules evaluated in order (lowest # first) |
| **Default inbound** | Deny all | Allow all (default NACL) |
| **Default outbound** | Allow all | Allow all (default NACL) |
| **Return traffic** | Automatically allowed | Must explicitly allow |
| **Rule numbers** | No numbering | Numbered (processed in order) |
| **Can reference SGs** | Yes (as source/dest) | No (CIDR only) |
| **Association** | Multiple SGs per instance | One NACL per subnet |
| **Multiple instances** | Yes (apply to many) | Applied to all in subnet |
| **Ephemeral ports** | Not needed | Must allow (1024-65535) |

---

## 5.3 Stateful vs Stateless in Practice

```
┌──────────────────────────────────────────────────────────────┐
│          STATEFUL vs STATELESS: PRACTICAL EXAMPLE            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client (203.0.113.5) → Server (10.0.1.50:443)              │
│                                                              │
│  SECURITY GROUP (Stateful):                                  │
│  ┌────────────────────────────────────────────────────┐      │
│  │ Inbound Rule:  Allow TCP 443 from 0.0.0.0/0  ✓    │      │
│  │ Outbound Rule: (NOT NEEDED for return traffic) ✓   │      │
│  │                                                    │      │
│  │ Request:  203.0.113.5:52000 → 10.0.1.50:443  ✓   │      │
│  │ Response: 10.0.1.50:443 → 203.0.113.5:52000  ✓   │      │
│  │           ↑ Auto-allowed because stateful          │      │
│  └────────────────────────────────────────────────────┘      │
│                                                              │
│  NACL (Stateless):                                           │
│  ┌────────────────────────────────────────────────────┐      │
│  │ Inbound Rule:   Allow TCP 443 from 0.0.0.0/0  ✓   │      │
│  │ Outbound Rule:  Allow TCP 1024-65535 to          │      │
│  │                 0.0.0.0/0                     ✓   │      │
│  │                 ↑ REQUIRED! Return traffic uses    │      │
│  │                   ephemeral ports (1024-65535)     │      │
│  │                                                    │      │
│  │ Request:  203.0.113.5:52000 → 10.0.1.50:443  ✓   │      │
│  │ Response: 10.0.1.50:443 → 203.0.113.5:52000  ✓   │      │
│  │           ↑ Only works if outbound rule exists!    │      │
│  └────────────────────────────────────────────────────┘      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.4 NACL Rule Evaluation Order

```
┌──────────────────────────────────────────────────────────────┐
│           NACL RULE EVALUATION                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Rules are evaluated in order (lowest # first)               │
│  FIRST MATCH wins — remaining rules skipped                  │
│                                                              │
│  Example:                                                    │
│  ┌──────┬──────────┬──────┬──────────────┬────────┐          │
│  │ Rule │ Protocol │ Port │ Source       │ Action │          │
│  ├──────┼──────────┼──────┼──────────────┼────────┤          │
│  │ 100  │ TCP      │ 443  │ 0.0.0.0/0   │ ALLOW  │ ← Check │
│  │ 110  │ TCP      │ 80   │ 0.0.0.0/0   │ ALLOW  │          │
│  │ 120  │ TCP      │ 22   │ 10.0.0.0/8  │ ALLOW  │          │
│  │ 200  │ TCP      │ 22   │ 0.0.0.0/0   │ DENY   │ ← Block │
│  │ *    │ ALL      │ ALL  │ 0.0.0.0/0   │ DENY   │ ← Final │
│  └──────┴──────────┴──────┴──────────────┴────────┘          │
│                                                              │
│  Scenario: SSH from 10.0.1.5                                 │
│  → Rule 120 matches (10.0.0.0/8) → ALLOW ✓                  │
│                                                              │
│  Scenario: SSH from 203.0.113.5                              │
│  → Rule 120 doesn't match                                    │
│  → Rule 200 matches (0.0.0.0/0) → DENY ✗                    │
│                                                              │
│  Best Practice: Leave gaps (100, 110, 120...)                │
│  so you can insert rules later (e.g., 115)                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.5 Terraform: NACL Configuration

```hcl
# Custom NACL for public subnet
resource "aws_network_acl" "public" {
  vpc_id     = aws_vpc.main.id
  subnet_ids = aws_subnet.public[*].id
  tags       = { Name = "public-nacl" }

  # Inbound: Allow HTTP
  ingress {
    rule_no    = 100
    protocol   = "tcp"
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 80
    to_port    = 80
  }

  # Inbound: Allow HTTPS
  ingress {
    rule_no    = 110
    protocol   = "tcp"
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 443
    to_port    = 443
  }

  # Inbound: Allow SSH from VPC only
  ingress {
    rule_no    = 120
    protocol   = "tcp"
    action     = "allow"
    cidr_block = "10.0.0.0/16"
    from_port  = 22
    to_port    = 22
  }

  # Inbound: Allow return traffic (ephemeral ports)
  ingress {
    rule_no    = 140
    protocol   = "tcp"
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 1024
    to_port    = 65535
  }

  # Outbound: Allow all
  egress {
    rule_no    = 100
    protocol   = "-1"
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 0
  }
}

# Custom NACL for private/data subnet
resource "aws_network_acl" "data" {
  vpc_id     = aws_vpc.main.id
  subnet_ids = aws_subnet.data[*].id
  tags       = { Name = "data-nacl" }

  # Inbound: Allow PostgreSQL from app subnet only
  ingress {
    rule_no    = 100
    protocol   = "tcp"
    action     = "allow"
    cidr_block = "10.0.11.0/24"   # App subnet
    from_port  = 5432
    to_port    = 5432
  }

  # Inbound: Allow ephemeral return traffic
  ingress {
    rule_no    = 200
    protocol   = "tcp"
    action     = "allow"
    cidr_block = "10.0.0.0/16"
    from_port  = 1024
    to_port    = 65535
  }

  # Outbound: Allow ephemeral return traffic
  egress {
    rule_no    = 100
    protocol   = "tcp"
    action     = "allow"
    cidr_block = "10.0.11.0/24"
    from_port  = 1024
    to_port    = 65535
  }
}
```

---

## 5.6 When to Use What

```
┌──────────────────────────────────────────────────────────────┐
│          WHEN TO USE SECURITY GROUPS vs NACLs                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SECURITY GROUPS (Primary defense — use always):             │
│  ✓ Per-instance access control                               │
│  ✓ Allow app server SG to access database SG                 │
│  ✓ Reference security groups by ID (dynamic)                 │
│  ✓ Microservice-level isolation                              │
│                                                              │
│  NACLs (Secondary defense — use for):                        │
│  ✓ Blocking specific IP ranges (DENY rules)                  │
│  ✓ Subnet-wide blanket rules                                 │
│  ✓ Compliance requirements (extra layer)                     │
│  ✓ Blocking known malicious IPs/CIDRs                        │
│                                                              │
│  Rule of thumb:                                              │
│  - Security Groups: 90% of your firewall rules               │
│  - NACLs: 10% — broad DENY rules, compliance                │
│  - Many teams leave default NACLs (allow all)                │
│    and rely entirely on Security Groups                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| NACL allows inbound but no response | Missing outbound ephemeral port rule | Add outbound 1024-65535 |
| NACL rule not taking effect | Higher-priority rule overriding | Check rule numbers (lower # wins) |
| Want to block one IP | Security Groups can't DENY | Use NACL with explicit DENY rule |
| All traffic blocked after custom NACL | Default rule `*` DENY, no ALLOW rules | Add appropriate ALLOW rules |
| SG reference not working across VPCs | SG references are VPC-scoped | Use CIDR blocks for cross-VPC |
| Intermittent connection issues | Ephemeral port range too narrow | Allow full 1024-65535 range |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Security Groups | Instance-level, stateful, allow-only, evaluate all rules |
| NACLs | Subnet-level, stateless, allow + deny, evaluate in order |
| Ephemeral ports | 1024-65535, needed for NACL outbound/inbound return |
| Rule numbering | NACLs use numbered rules, gaps of 10-100 recommended |
| Default NACL | Allows all traffic (don't confuse with custom NACL) |
| Custom NACL | Denies all traffic by default until rules added |
| Defense in depth | Use both layers for maximum security |
| Best practice | SGs for most rules, NACLs for broad DENY |

---

## Quick Revision Questions

1. **Why do NACLs need ephemeral port rules but Security Groups don't?**
2. **A custom NACL is created but all traffic is blocked. Why?**
3. **Write NACL rules to allow HTTPS inbound and its return traffic outbound.**
4. **Can you block a specific IP address using a Security Group? If not, what do you use?**
5. **Explain how NACL rule evaluation order works. Why leave gaps between rule numbers?**
6. **In a 3-tier architecture, where would you use NACLs vs Security Groups?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Peering & Transit Gateways](04-peering-transit-gateways.md) | [README](../README.md) | [Service Endpoints →](06-service-endpoints.md) |
