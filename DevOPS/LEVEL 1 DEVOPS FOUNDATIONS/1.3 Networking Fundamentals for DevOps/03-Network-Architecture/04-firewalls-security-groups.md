# Chapter 4: Firewalls and Security Groups

## Overview

Firewalls control network traffic flow based on predefined rules. In cloud environments, Security Groups and Network ACLs serve as virtual firewalls. Proper firewall configuration is the first line of defense in securing your infrastructure — misconfigured rules are one of the most common causes of both security breaches and connectivity issues.

---

## 4.1 Firewall Concepts

```
┌──────────────────────────────────────────────────────────────┐
│              FIREWALL: ALLOW/DENY TRAFFIC                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Internet                                                    │
│     │                                                        │
│     ▼                                                        │
│  ┌──────────────────────────────────────┐                    │
│  │          F I R E W A L L              │                    │
│  │                                      │                    │
│  │  Rule 1: Allow TCP 443 from 0.0.0.0/0│  ✓ HTTPS         │
│  │  Rule 2: Allow TCP 80 from 0.0.0.0/0 │  ✓ HTTP          │
│  │  Rule 3: Allow TCP 22 from 10.0.0./8 │  ✓ SSH (internal)│
│  │  Rule 4: Deny ALL from 0.0.0.0/0     │  ✗ Everything    │
│  │                                      │     else blocked  │
│  └──────────────────────────────────────┘                    │
│     │                                                        │
│     ▼ (only allowed traffic passes)                          │
│  ┌──────────────────┐                                        │
│  │  Web Server      │                                        │
│  │  10.0.1.50       │                                        │
│  └──────────────────┘                                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.2 AWS Security Groups vs NACLs

```
┌──────────────────────────────────────────────────────────────┐
│           SECURITY GROUPS vs NACLs                           │
├──────────────────────┬──────────────────┬────────────────────┤
│ Feature              │ Security Group   │ Network ACL        │
├──────────────────────┼──────────────────┼────────────────────┤
│ Level                │ Instance (ENI)   │ Subnet             │
│ State                │ Stateful         │ Stateless          │
│ Rules                │ Allow only       │ Allow AND Deny     │
│ Evaluation           │ All rules at once│ Rules in order     │
│ Default inbound      │ Deny all         │ Allow all          │
│ Default outbound     │ Allow all        │ Allow all          │
│ Return traffic       │ Auto-allowed     │ Must be explicit   │
│ Associated with      │ EC2, RDS, ELB    │ Subnet             │
│ Rule limit           │ 60 inbound +     │ 20 inbound +       │
│                      │ 60 outbound      │ 20 outbound        │
└──────────────────────┴──────────────────┴────────────────────┘
```

### Stateful vs Stateless

```
┌──────────────────────────────────────────────────────────────┐
│              STATEFUL vs STATELESS                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  STATEFUL (Security Groups):                                 │
│  ┌────────┐     ──▶ Inbound allowed     ┌────────┐          │
│  │ Client │                              │ Server │          │
│  │        │     ◀── Return auto-allowed  │        │          │
│  └────────┘         (no outbound rule    └────────┘          │
│                      needed!)                                │
│                                                              │
│  STATELESS (NACLs):                                          │
│  ┌────────┐     ──▶ Inbound rule needed  ┌────────┐          │
│  │ Client │                              │ Server │          │
│  │        │     ◀── Outbound rule ALSO   │        │          │
│  └────────┘         needed for return    └────────┘          │
│                     traffic!                                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.3 Security Group Examples

```bash
# Create a security group
aws ec2 create-security-group \
  --group-name web-server-sg \
  --description "Security group for web servers" \
  --vpc-id vpc-12345678

# Allow HTTP from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp --port 80 --cidr 0.0.0.0/0

# Allow HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp --port 443 --cidr 0.0.0.0/0

# Allow SSH from specific IP only
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345678 \
  --protocol tcp --port 22 --cidr 203.0.113.50/32

# Allow app port from another security group
aws ec2 authorize-security-group-ingress \
  --group-id sg-app-12345 \
  --protocol tcp --port 8080 \
  --source-group sg-alb-12345
```

### Terraform: Security Group

```hcl
resource "aws_security_group" "web" {
  name        = "web-server-sg"
  description = "Allow HTTP, HTTPS, and SSH"
  vpc_id      = aws_vpc.main.id

  # HTTP
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # HTTPS
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # SSH from bastion only
  ingress {
    from_port       = 22
    to_port         = 22
    protocol        = "tcp"
    security_groups = [aws_security_group.bastion.id]
  }

  # Allow all outbound
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = { Name = "web-server-sg" }
}
```

---

## 4.4 3-Tier Security Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              3-TIER SECURITY GROUP DESIGN                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Internet                                                    │
│     │                                                        │
│     ▼                                                        │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  ALB Security Group                                 │     │
│  │  Inbound: 80, 443 from 0.0.0.0/0                   │     │
│  │  Outbound: All to web-sg                            │     │
│  └─────────────────────┬───────────────────────────────┘     │
│                        │                                     │
│                        ▼                                     │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Web Server Security Group                          │     │
│  │  Inbound: 8080 from ALB-sg only                     │     │
│  │  Outbound: 8080 to app-sg                           │     │
│  └─────────────────────┬───────────────────────────────┘     │
│                        │                                     │
│                        ▼                                     │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  App Server Security Group                          │     │
│  │  Inbound: 8080 from web-sg only                     │     │
│  │  Outbound: 5432 to db-sg                            │     │
│  └─────────────────────┬───────────────────────────────┘     │
│                        │                                     │
│                        ▼                                     │
│  ┌─────────────────────────────────────────────────────┐     │
│  │  Database Security Group                            │     │
│  │  Inbound: 5432 from app-sg only                     │     │
│  │  Outbound: None (or limited)                        │     │
│  └─────────────────────────────────────────────────────┘     │
│                                                              │
│  Key: Each tier only accepts traffic from the tier above     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.5 Linux Firewall (iptables / nftables)

```bash
# View current rules
sudo iptables -L -n -v

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Drop everything else
sudo iptables -A INPUT -j DROP

# Modern alternative: UFW (Uncomplicated Firewall)
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
```

---

## 4.6 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Can't SSH to instance | Port 22 not in SG | Add SSH rule to security group |
| ALB health check failing | SG doesn't allow ALB traffic | Allow ALB SG as source |
| RDS connection refused | DB SG doesn't allow app SG | Add inbound rule for app SG |
| All traffic blocked | Default DENY, no rules | Add appropriate allow rules |
| NACL blocking return traffic | NACLs are stateless | Add outbound or ephemeral port rules |
| Can reach on one port, not another | Missing SG rule | Add the specific port rule |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Firewall | Controls traffic flow based on rules |
| Security Group | Instance-level, stateful, allow-only rules |
| NACL | Subnet-level, stateless, allow + deny rules |
| Stateful | Return traffic auto-allowed (Security Groups) |
| Stateless | Return traffic needs explicit rules (NACLs) |
| Best Practice | Reference SGs by ID, not CIDR where possible |
| 3-Tier Design | Each tier's SG only allows from the tier above |

---

## Quick Revision Questions

1. **What is the key difference between stateful and stateless firewalls?**
2. **Why can Security Groups only have "allow" rules while NACLs can have "deny" rules?**
3. **Design the security group rules for a 3-tier architecture (ALB → Web → App → DB).**
4. **In a NACL, you allow inbound port 80 but the web server is unreachable. What's missing?**
5. **What's the advantage of referencing another security group as a source instead of using CIDR?**
6. **How would you use `ufw` to secure a Linux server running a web application?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Load Balancing](03-load-balancing.md) | [README](../README.md) | [VPN Concepts →](05-vpn-concepts.md) |
