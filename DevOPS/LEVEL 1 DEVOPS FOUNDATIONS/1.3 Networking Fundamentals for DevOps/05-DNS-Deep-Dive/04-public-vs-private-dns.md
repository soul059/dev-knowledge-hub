# Chapter 4: Public vs Private DNS

## Overview

**Public DNS** resolves domain names accessible from anywhere on the internet, while **Private DNS** resolves names only within a specific network (VPC, corporate network). In cloud environments, private DNS zones enable service discovery, simplify internal communication, and keep internal infrastructure names hidden from the public internet.

---

## 4.1 Public vs Private DNS

```
┌──────────────────────────────────────────────────────────────┐
│         PUBLIC vs PRIVATE DNS                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PUBLIC DNS                                                  │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ Zone: example.com (Route 53 Public Hosted Zone)        │  │
│  │                                                        │  │
│  │ app.example.com    → 54.23.45.67  (ALB public IP)      │  │
│  │ api.example.com    → 52.12.34.56  (ALB public IP)      │  │
│  │ www.example.com    → CNAME app.example.com             │  │
│  │                                                        │  │
│  │ ✓ Resolvable from anywhere on the internet             │  │
│  │ ✓ Managed by public DNS providers                      │  │
│  │ ✓ Anyone can query these records                       │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  PRIVATE DNS                                                 │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ Zone: internal.example.com (Route 53 Private Zone)     │  │
│  │ Associated with: VPC vpc-12345678                      │  │
│  │                                                        │  │
│  │ db.internal.example.com     → 10.0.21.50 (RDS)        │  │
│  │ cache.internal.example.com  → 10.0.21.51 (Redis)      │  │
│  │ api.internal.example.com    → 10.0.11.50 (internal)    │  │
│  │ queue.internal.example.com  → 10.0.11.60 (SQS proxy)  │  │
│  │                                                        │  │
│  │ ✓ Only resolvable within associated VPC(s)             │  │
│  │ ✗ Cannot be queried from the internet                  │  │
│  │ ✓ Keeps internal infrastructure names hidden           │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.2 Architecture with Both

```
┌──────────────────────────────────────────────────────────────┐
│      COMBINED PUBLIC + PRIVATE DNS ARCHITECTURE              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Internet Users                                              │
│       │                                                      │
│       │ DNS: app.example.com → 54.23.45.67                   │
│       │      (Public DNS resolution)                         │
│       ▼                                                      │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │  VPC: 10.0.0.0/16                                       │ │
│  │                                                         │ │
│  │  Public Subnet                                          │ │
│  │  ┌──────────────────┐                                   │ │
│  │  │ ALB (54.23.45.67)│ ← Public DNS points here          │ │
│  │  └────────┬─────────┘                                   │ │
│  │           │                                             │ │
│  │  Private Subnet                                         │ │
│  │  ┌────────▼─────────┐                                   │ │
│  │  │ App Server       │                                   │ │
│  │  │ 10.0.11.50       │                                   │ │
│  │  │                  │ → db.internal.example.com          │ │
│  │  │ Uses Private DNS │   (→ 10.0.21.50)                  │ │
│  │  └────────┬─────────┘                                   │ │
│  │           │                                             │ │
│  │  Data Subnet                                            │ │
│  │  ┌────────▼─────────┐                                   │ │
│  │  │ RDS Database     │ ← Private DNS name only           │ │
│  │  │ 10.0.21.50       │    (no public access)             │ │
│  │  └──────────────────┘                                   │ │
│  │                                                         │ │
│  │  Route 53 Resolver:                                     │ │
│  │  - *.internal.example.com → Private Hosted Zone          │ │
│  │  - Everything else → Public DNS                         │ │
│  │                                                         │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.3 AWS Route 53 Private Hosted Zone

```hcl
# Private Hosted Zone
resource "aws_route53_zone" "internal" {
  name = "internal.example.com"

  vpc {
    vpc_id = aws_vpc.main.id
  }

  tags = { Name = "internal-dns" }
}

# Associate additional VPCs
resource "aws_route53_zone_association" "staging" {
  zone_id = aws_route53_zone.internal.id
  vpc_id  = aws_vpc.staging.id
}

# Private DNS records
resource "aws_route53_record" "database" {
  zone_id = aws_route53_zone.internal.id
  name    = "db.internal.example.com"
  type    = "CNAME"
  ttl     = 300
  records = [aws_db_instance.main.address]
}

resource "aws_route53_record" "cache" {
  zone_id = aws_route53_zone.internal.id
  name    = "cache.internal.example.com"
  type    = "CNAME"
  ttl     = 300
  records = [aws_elasticache_cluster.redis.cache_nodes[0].address]
}

resource "aws_route53_record" "api" {
  zone_id = aws_route53_zone.internal.id
  name    = "api.internal.example.com"
  type    = "A"

  alias {
    name                   = aws_lb.internal.dns_name
    zone_id                = aws_lb.internal.zone_id
    evaluate_target_health = true
  }
}
```

---

## 4.4 Split-Horizon DNS

```
┌──────────────────────────────────────────────────────────────┐
│           SPLIT-HORIZON (SPLIT-VIEW) DNS                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Same domain name → different answers based on source        │
│                                                              │
│  External query for app.example.com:                         │
│  ┌──────────┐    Query     ┌────────────┐                    │
│  │ Internet │ ──────────▶  │ Public Zone│ → 54.23.45.67      │
│  │ User     │              │            │   (ALB public IP)  │
│  └──────────┘              └────────────┘                    │
│                                                              │
│  Internal query for app.example.com:                         │
│  ┌──────────┐    Query     ┌─────────────┐                   │
│  │ EC2 in   │ ──────────▶  │ Private Zone│ → 10.0.11.50      │
│  │ VPC      │              │             │   (internal ALB)  │
│  └──────────┘              └─────────────┘                   │
│                                                              │
│  When VPC has private zone for same domain:                  │
│  Private zone takes precedence for VPC resources!            │
│                                                              │
│  Benefits:                                                   │
│  - Internal traffic stays internal (no hairpin through IGW)   │
│  - Faster access (no public internet routing)                │
│  - Security: internal IPs not exposed publicly               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.5 Route 53 Resolver (Hybrid DNS)

```
┌──────────────────────────────────────────────────────────────┐
│         ROUTE 53 RESOLVER ENDPOINTS                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  On-Premise                          AWS VPC                 │
│  ┌──────────────────┐                ┌──────────────────┐    │
│  │                  │                │                  │    │
│  │ corp.internal    │    VPN/DX      │ internal.example │    │
│  │ ┌──────────────┐ │ ◄════════════▶ │ ┌──────────────┐ │    │
│  │ │ On-Prem DNS  │ │                │ │ Route 53     │ │    │
│  │ │ Server       │ │                │ │ Resolver     │ │    │
│  │ └──────────────┘ │                │ └──────────────┘ │    │
│  │                  │                │                  │    │
│  │ Inbound Endpoint │               │ Outbound Endpoint│   │
│  │ (for on-prem to  │               │ (for VPC to      │   │
│  │  query Route 53) │               │  query on-prem)  │   │
│  └──────────────────┘                └──────────────────┘    │
│                                                              │
│  Outbound: VPC queries for corp.internal → forwarded         │
│            to on-prem DNS via outbound endpoint               │
│                                                              │
│  Inbound: On-prem queries for internal.example →             │
│           forwarded to Route 53 via inbound endpoint          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```hcl
# Route 53 Resolver Outbound Endpoint
resource "aws_route53_resolver_endpoint" "outbound" {
  name      = "outbound-to-onprem"
  direction = "OUTBOUND"

  security_group_ids = [aws_security_group.dns.id]

  ip_address { subnet_id = aws_subnet.private[0].id }
  ip_address { subnet_id = aws_subnet.private[1].id }
}

# Forward on-prem domain queries
resource "aws_route53_resolver_rule" "corp_internal" {
  domain_name          = "corp.internal"
  rule_type            = "FORWARD"
  resolver_endpoint_id = aws_route53_resolver_endpoint.outbound.id

  target_ip {
    ip   = "192.168.1.10"    # On-prem DNS server
    port = 53
  }
}
```

---

## 4.6 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Private DNS not resolving | VPC not associated with zone | Associate VPC with private zone |
| Public record resolving inside VPC | No private zone override | Create matching private hosted zone |
| On-prem can't resolve AWS names | No inbound endpoint | Create Route 53 Resolver inbound |
| VPC can't resolve on-prem names | No outbound endpoint | Create Route 53 Resolver outbound |
| DNS resolution slow in VPC | enableDnsSupport disabled | Enable DNS support on VPC |
| Private zone name conflicts | Overlapping zone names | Use distinct internal domains |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Public DNS | Resolvable globally, external users |
| Private DNS | Resolvable only within associated VPC(s) |
| Private Hosted Zone | Route 53 zone linked to specific VPCs |
| Split-Horizon | Same domain returns different IPs internally vs externally |
| Resolver Inbound | On-prem queries forwarded to Route 53 |
| Resolver Outbound | VPC queries forwarded to on-prem DNS |
| enableDnsSupport | Must be enabled for VPC DNS resolution |
| enableDnsHostnames | Enables auto-generated DNS names for instances |

---

## Quick Revision Questions

1. **What's the difference between a public and private hosted zone in Route 53?**
2. **How does split-horizon DNS work and why is it useful?**
3. **Set up private DNS for an RDS database so apps can use `db.internal.example.com`.**
4. **When would you need Route 53 Resolver inbound vs outbound endpoints?**
5. **Your EC2 instance can't resolve `db.internal.example.com`. What do you check?**
6. **Why should internal traffic use private DNS instead of going through public DNS?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← DNS Resolution Process](03-dns-resolution-process.md) | [README](../README.md) | [Route 53 & Cloud DNS →](05-route53-cloud-dns.md) |
