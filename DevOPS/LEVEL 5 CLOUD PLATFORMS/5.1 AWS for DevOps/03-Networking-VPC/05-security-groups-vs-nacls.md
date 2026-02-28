# Chapter 3.5: Security Groups vs NACLs

## Overview

AWS provides two layers of network security: **Security Groups** (instance-level, stateful) and **Network ACLs** (subnet-level, stateless). Understanding the difference is critical for the exam and real-world architecture.

---

## Two Layers of Defense

```
          ☁ Inbound Traffic
              │
              ▼
    ┌─────────────────────┐
    │   Network ACL       │  ◄── Layer 1: Subnet boundary (STATELESS)
    │   (NACL)            │      Evaluates BOTH inbound AND outbound
    │   Subnet-level      │      Rules have numbers (priority order)
    └─────────┬───────────┘
              │
              ▼
    ┌─────────────────────┐
    │   Security Group    │  ◄── Layer 2: Instance boundary (STATEFUL)
    │   (SG)              │      If inbound allowed, outbound auto-allowed
    │   ENI-level         │      All rules evaluated together
    └─────────┬───────────┘
              │
              ▼
         ┌─────────┐
         │   EC2   │
         │ Instance│
         └─────────┘
```

---

## Security Groups (Stateful)

```
┌──────────── Security Group: web-sg ──────────────────────┐
│                                                           │
│  INBOUND RULES:                                          │
│  ┌────────┬──────────┬──────────────────────────────┐    │
│  │ Type   │ Port     │ Source                        │    │
│  ├────────┼──────────┼──────────────────────────────┤    │
│  │ HTTP   │ 80       │ 0.0.0.0/0                    │    │
│  │ HTTPS  │ 443      │ 0.0.0.0/0                    │    │
│  │ SSH    │ 22       │ 10.0.0.0/16 (VPC only)       │    │
│  │ Custom │ 8080     │ sg-alb-xxx  ◄── SG reference │    │
│  └────────┴──────────┴──────────────────────────────┘    │
│                                                           │
│  OUTBOUND RULES:                                         │
│  ┌────────┬──────────┬──────────────────────────────┐    │
│  │ Type   │ Port     │ Destination                   │    │
│  ├────────┼──────────┼──────────────────────────────┤    │
│  │ All    │ All      │ 0.0.0.0/0  (default)         │    │
│  └────────┴──────────┴──────────────────────────────┘    │
│                                                           │
│  ⚠ STATEFUL: Return traffic automatically allowed        │
│  ⚠ ALLOW-only: No DENY rules possible                   │
│  ⚠ Default: All inbound DENIED, all outbound ALLOWED    │
└───────────────────────────────────────────────────────────┘
```

### Security Group Key Properties

| Property | Details |
|----------|---------|
| Level | Instance (ENI) level |
| State | **Stateful** — return traffic auto-allowed |
| Rules | **Allow only** — cannot create deny rules |
| Evaluation | All rules evaluated; if any rule allows, traffic passes |
| Default inbound | All denied |
| Default outbound | All allowed |
| References | Can reference other SGs as source/destination |
| Changes | Take effect immediately |

### SG CLI Commands

```bash
# Create Security Group
aws ec2 create-security-group \
  --group-name web-sg \
  --description "Web tier security group" \
  --vpc-id vpc-xxx

# Add inbound rules
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxx \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow traffic from another Security Group
aws ec2 authorize-security-group-ingress \
  --group-id sg-app \
  --protocol tcp \
  --port 8080 \
  --source-group sg-alb

# Remove a rule
aws ec2 revoke-security-group-ingress \
  --group-id sg-xxx \
  --protocol tcp \
  --port 22 \
  --cidr 0.0.0.0/0

# Describe SG rules
aws ec2 describe-security-group-rules \
  --filters "Name=group-id,Values=sg-xxx" \
  --query 'SecurityGroupRules[*].[IsEgress,IpProtocol,FromPort,ToPort,CidrIpv4]' \
  --output table
```

---

## Security Group Chaining Pattern

```
┌──────────────── Three-Tier SG Chain ─────────────────────┐
│                                                           │
│     Internet                                              │
│         │                                                 │
│   ┌─────▼──────┐   Source: 0.0.0.0/0                    │
│   │  ALB       │   SG: alb-sg                           │
│   │  Port: 443 │   Inbound: 443 from 0.0.0.0/0         │
│   └─────┬──────┘                                         │
│         │                                                 │
│   ┌─────▼──────┐   Source: sg-alb (SG reference!)       │
│   │  App EC2   │   SG: app-sg                           │
│   │  Port: 8080│   Inbound: 8080 from sg-alb ONLY      │
│   └─────┬──────┘                                         │
│         │                                                 │
│   ┌─────▼──────┐   Source: sg-app (SG reference!)       │
│   │  RDS       │   SG: db-sg                            │
│   │  Port: 3306│   Inbound: 3306 from sg-app ONLY      │
│   └────────────┘                                         │
│                                                           │
│  ⚠ Each tier only accepts traffic from the tier above   │
│  ⚠ SG references auto-update when instances change      │
└───────────────────────────────────────────────────────────┘
```

---

## Network ACLs (Stateless)

```
┌──────────── Network ACL: private-nacl ───────────────────┐
│                                                           │
│  INBOUND RULES:                                          │
│  ┌──────┬────────┬──────┬───────────────┬────────┐       │
│  │ Rule#│ Type   │ Port │ Source        │ Action │       │
│  ├──────┼────────┼──────┼───────────────┼────────┤       │
│  │ 100  │ HTTPS  │ 443  │ 0.0.0.0/0    │ ALLOW  │       │
│  │ 110  │ HTTP   │ 80   │ 0.0.0.0/0    │ ALLOW  │       │
│  │ 120  │ SSH    │ 22   │ 10.0.1.0/24  │ ALLOW  │       │
│  │ 130  │ Custom │ 1024-65535│ 0.0.0.0/0│ ALLOW  │◄─ Ephemeral│
│  │ 200  │ All    │ All  │ 192.168.1.0  │ DENY   │       │
│  │  *   │ All    │ All  │ 0.0.0.0/0    │ DENY   │◄─ Catch-all│
│  └──────┴────────┴──────┴───────────────┴────────┘       │
│                                                           │
│  OUTBOUND RULES:                                         │
│  ┌──────┬────────┬──────┬───────────────┬────────┐       │
│  │ Rule#│ Type   │ Port │ Destination   │ Action │       │
│  ├──────┼────────┼──────┼───────────────┼────────┤       │
│  │ 100  │ HTTPS  │ 443  │ 0.0.0.0/0    │ ALLOW  │       │
│  │ 110  │ Custom │ 1024-65535│ 0.0.0.0/0│ ALLOW  │◄─ Ephemeral│
│  │  *   │ All    │ All  │ 0.0.0.0/0    │ DENY   │       │
│  └──────┴────────┴──────┴───────────────┴────────┘       │
│                                                           │
│  ⚠ STATELESS: Must explicitly allow return traffic       │
│  ⚠ Rules processed in ORDER (lowest number first)        │
│  ⚠ First matching rule wins, rest ignored                │
└───────────────────────────────────────────────────────────┘
```

### NACL Key Properties

| Property | Details |
|----------|---------|
| Level | Subnet level |
| State | **Stateless** — must allow both directions |
| Rules | Allow AND Deny rules |
| Evaluation | Rules processed in number order; first match wins |
| Default NACL | Allows all inbound/outbound |
| Custom NACL | Denies all inbound/outbound by default |
| Ephemeral ports | Must allow 1024-65535 for return traffic |
| Association | One NACL per subnet (but one NACL → many subnets) |

### Ephemeral Ports (Critical Concept)

```
Client (:443) ──────────► Server (443)     Inbound: Port 443
Server (443)  ──────────► Client (:52738)  Outbound: Ephemeral port!

┌─────────────────────────────────────────────────────┐
│  Ephemeral Port Ranges:                             │
│  ─────────────────────                              │
│  Linux:    32768 – 65535                            │
│  Windows:  49152 – 65535                            │
│  ELB/NAT:  1024 – 65535                             │
│                                                     │
│  Best Practice: Allow 1024-65535 in NACLs           │
│  to cover all OS ephemeral port ranges              │
└─────────────────────────────────────────────────────┘
```

---

## Complete Comparison Table

| Feature | Security Group | Network ACL |
|---------|---------------|-------------|
| **Level** | Instance (ENI) | Subnet |
| **Stateful/less** | Stateful | Stateless |
| **Rules** | Allow only | Allow + Deny |
| **Rule evaluation** | All rules evaluated | Numbered order, first match |
| **Default (new)** | Deny all in, allow all out | Deny all (custom NACL) |
| **Return traffic** | Automatic | Must be explicitly allowed |
| **SG references** | ✓ Supported | ✗ Not supported |
| **Apply to** | Assigned to specific instances | All instances in subnet |
| **Ephemeral ports** | Not needed | Must allow (1024-65535) |
| **Block specific IP** | ✗ Cannot deny | ✓ Can deny specific IPs |
| **Use case** | Primary security control | Extra subnet-level defense, IP blocking |

---

## Real-World Pattern: Block Malicious IP

```bash
# Security Groups CANNOT deny — use NACL to block an IP

# Create custom NACL
aws ec2 create-network-acl \
  --vpc-id vpc-xxx \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=web-nacl}]'

# Block a malicious IP (low rule number = high priority)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxx \
  --rule-number 50 \
  --protocol -1 \
  --rule-action deny \
  --cidr-block 203.0.113.0/24 \
  --ingress

# Allow all other HTTP traffic
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxx \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=443,To=443 \
  --rule-action allow \
  --cidr-block 0.0.0.0/0 \
  --ingress

# Allow ephemeral ports (return traffic — NACL is stateless!)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-xxx \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=1024,To=65535 \
  --rule-action allow \
  --cidr-block 0.0.0.0/0 \
  --egress

# Associate with subnet
aws ec2 replace-network-acl-association \
  --association-id aclassoc-xxx \
  --network-acl-id acl-xxx
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Instance can't receive traffic | SG has no inbound rule for that port | Add inbound rule to SG |
| Instance can reach out but response fails | NACL missing ephemeral port outbound rule | Allow 1024-65535 outbound in NACL |
| NACL allow rule not working | Higher-priority deny rule exists | Check rule numbers — lower number = first evaluated |
| App tier can't reach DB | SG references wrong source group | Ensure DB SG allows from app SG-id |
| Want to block IP but SG can't deny | SGs are allow-only | Use NACL to add a deny rule for that IP |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Security Group** | Stateful, allow-only, instance-level. Primary security control. |
| **Network ACL** | Stateless, allow+deny, subnet-level. Extra defense layer. |
| **Stateful** | If inbound traffic allowed, outbound return is automatic (and vice versa). |
| **Stateless** | Both directions must have explicit rules. Must allow ephemeral ports. |
| **SG Chaining** | Reference other SGs as source instead of CIDRs. Auto-scales with instances. |
| **IP Blocking** | Only NACLs can deny — use low rule numbers for deny rules. |

---

## Quick Revision Questions

1. **What does "stateful" mean for Security Groups?**
   > If an inbound rule allows traffic, the response traffic is automatically allowed regardless of outbound rules (and vice versa).

2. **Can you create a deny rule in a Security Group?**
   > No. Security Groups only support allow rules. Use NACLs for deny rules.

3. **Why must you allow ephemeral ports in NACLs?**
   > Because NACLs are stateless — return traffic uses ephemeral ports (1024-65535), which must be explicitly allowed.

4. **How are NACL rules evaluated?**
   > In ascending rule-number order. The first matching rule is applied, and evaluation stops.

5. **What is Security Group chaining?**
   > Referencing another SG as the source in a rule (e.g., DB-SG allows port 3306 from App-SG), so only instances in the App SG can reach the DB.

6. **What is the default behavior of a custom NACL?**
   > Denies all inbound and outbound traffic until you add rules.

---

[← Previous: 3.4 Internet & NAT Gateways](04-internet-gateway-nat-gateway.md) | [Next: 3.6 VPC Peering & Transit Gateway →](06-vpc-peering-transit-gateway.md)

[← Back to README](../README.md)
