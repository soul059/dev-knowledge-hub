# Chapter 5: Route 53 and Cloud DNS Services

## Overview

**Amazon Route 53** is AWS's managed DNS service offering domain registration, DNS hosting, and health-checked routing. Understanding Route 53's routing policies, health checks, and integration with other AWS services is a core DevOps skill. This chapter also covers equivalent services in Azure (Azure DNS) and GCP (Cloud DNS).

---

## 5.1 Route 53 Overview

```
┌──────────────────────────────────────────────────────────────┐
│              ROUTE 53 CAPABILITIES                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. DOMAIN REGISTRATION                                      │
│     Buy and manage domain names (.com, .io, .dev, etc.)      │
│                                                              │
│  2. DNS HOSTING                                              │
│     ┌─────────────────────────────────────────────┐          │
│     │ Hosted Zones (Public and Private)            │          │
│     │ - A, AAAA, CNAME, MX, TXT, NS, SRV records  │          │
│     │ - ALIAS records (AWS-specific)               │          │
│     │ - 100% availability SLA                      │          │
│     └─────────────────────────────────────────────┘          │
│                                                              │
│  3. TRAFFIC ROUTING                                          │
│     ┌─────────────────────────────────────────────┐          │
│     │ Routing Policies:                            │          │
│     │ - Simple      - Weighted    - Latency        │          │
│     │ - Failover    - Geolocation - Multi-Value     │          │
│     │ - Geoproximity (Traffic Flow)                 │          │
│     └─────────────────────────────────────────────┘          │
│                                                              │
│  4. HEALTH CHECKS                                            │
│     ┌─────────────────────────────────────────────┐          │
│     │ Monitor endpoints and trigger failover       │          │
│     │ - HTTP/HTTPS/TCP checks                      │          │
│     │ - String matching                            │          │
│     │ - CloudWatch alarm-based                     │          │
│     └─────────────────────────────────────────────┘          │
│                                                              │
│  Pricing:                                                    │
│  - $0.50/month per hosted zone                               │
│  - $0.40 per million standard queries                        │
│  - ALIAS queries to AWS resources: FREE                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.2 Routing Policies

```
┌──────────────────────────────────────────────────────────────┐
│              ROUTE 53 ROUTING POLICIES                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. SIMPLE ROUTING                                           │
│     app.example.com → 1.2.3.4                                │
│     One record, one or multiple values (random if multiple)  │
│                                                              │
│  2. WEIGHTED ROUTING                                         │
│     app.example.com → 1.2.3.4 (weight: 90)  ← 90% traffic   │
│     app.example.com → 5.6.7.8 (weight: 10)  ← 10% traffic   │
│     Use case: Canary deployments, A/B testing                │
│                                                              │
│  3. LATENCY-BASED ROUTING                                    │
│     app.example.com → 1.2.3.4 (us-east-1)                   │
│     app.example.com → 9.8.7.6 (eu-west-1)                   │
│     Route to region with lowest latency for user             │
│                                                              │
│  4. FAILOVER ROUTING                                         │
│     app.example.com → 1.2.3.4 (PRIMARY + health check)      │
│     app.example.com → 5.6.7.8 (SECONDARY)                   │
│     Auto-switch to secondary when primary fails              │
│                                                              │
│  5. GEOLOCATION ROUTING                                      │
│     app.example.com → EU ALB (if user in Europe)             │
│     app.example.com → US ALB (if user in North America)      │
│     app.example.com → Default (everywhere else)              │
│     Use case: Compliance, content localization               │
│                                                              │
│  6. MULTI-VALUE ANSWER                                       │
│     app.example.com → up to 8 IPs (health-checked)          │
│     Returns only healthy IPs — client picks randomly         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.3 Terraform: Route 53 Examples

```hcl
# Public Hosted Zone
resource "aws_route53_zone" "main" {
  name = "example.com"
}

# Simple A Record
resource "aws_route53_record" "app" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"

  alias {
    name                   = aws_lb.app.dns_name
    zone_id                = aws_lb.app.zone_id
    evaluate_target_health = true
  }
}

# Weighted Routing (Canary)
resource "aws_route53_record" "app_stable" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"

  set_identifier = "stable"
  weighted_routing_policy { weight = 90 }

  alias {
    name                   = aws_lb.stable.dns_name
    zone_id                = aws_lb.stable.zone_id
    evaluate_target_health = true
  }
}

resource "aws_route53_record" "app_canary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"

  set_identifier = "canary"
  weighted_routing_policy { weight = 10 }

  alias {
    name                   = aws_lb.canary.dns_name
    zone_id                = aws_lb.canary.zone_id
    evaluate_target_health = true
  }
}

# Failover Routing with Health Check
resource "aws_route53_health_check" "primary" {
  fqdn              = "primary-alb.us-east-1.elb.amazonaws.com"
  type               = "HTTPS"
  port               = 443
  resource_path      = "/health"
  failure_threshold  = 3
  request_interval   = 10

  tags = { Name = "primary-health-check" }
}

resource "aws_route53_record" "primary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"

  set_identifier = "primary"
  failover_routing_policy { type = "PRIMARY" }
  health_check_id = aws_route53_health_check.primary.id

  alias {
    name                   = aws_lb.primary.dns_name
    zone_id                = aws_lb.primary.zone_id
    evaluate_target_health = true
  }
}

resource "aws_route53_record" "secondary" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"

  set_identifier = "secondary"
  failover_routing_policy { type = "SECONDARY" }

  alias {
    name                   = aws_lb.secondary.dns_name
    zone_id                = aws_lb.secondary.zone_id
    evaluate_target_health = true
  }
}

# Latency-Based Routing
resource "aws_route53_record" "app_us" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"

  set_identifier = "us-east-1"
  latency_routing_policy { region = "us-east-1" }

  alias {
    name                   = aws_lb.us.dns_name
    zone_id                = aws_lb.us.zone_id
    evaluate_target_health = true
  }
}

resource "aws_route53_record" "app_eu" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"

  set_identifier = "eu-west-1"
  latency_routing_policy { region = "eu-west-1" }

  alias {
    name                   = aws_lb.eu.dns_name
    zone_id                = aws_lb.eu.zone_id
    evaluate_target_health = true
  }
}
```

---

## 5.4 Multi-Cloud DNS Comparison

| Feature | Route 53 (AWS) | Azure DNS | Cloud DNS (GCP) |
|---------|---------------|-----------|-----------------|
| Hosted zone cost | $0.50/month | $0.50/month | $0.20/month |
| Query cost | $0.40/M | $0.40/M | $0.40/M |
| ALIAS records | Yes (AWS resources) | No (use A with alias) | No |
| Health checks | Built-in | Via Traffic Manager | Via Cloud Load Balancing |
| Failover | Routing policy | Traffic Manager | Cloud Load Balancing |
| Private zones | Yes | Yes | Yes |
| DNSSEC | Yes | Preview | Yes |
| 100% SLA | Yes | Yes | Yes |

---

## 5.5 Route 53 CLI Commands

```bash
# List hosted zones
aws route53 list-hosted-zones --query "HostedZones[*].{Name:Name,Id:Id}" --output table

# List records in a zone
aws route53 list-resource-record-sets \
  --hosted-zone-id Z123456 \
  --query "ResourceRecordSets[*].{Name:Name,Type:Type,TTL:TTL}" \
  --output table

# Create / update a record
aws route53 change-resource-record-sets \
  --hosted-zone-id Z123456 \
  --change-batch '{
    "Changes": [{
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "test.example.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{"Value": "10.0.1.50"}]
      }
    }]
  }'

# Check health check status
aws route53 get-health-check-status --health-check-id abc-123

# Test DNS resolution
aws route53 test-dns-answer \
  --hosted-zone-id Z123456 \
  --record-name app.example.com \
  --record-type A
```

---

## 5.6 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| ALIAS record not resolving | Target resource deleted | Verify ALB/CloudFront exists |
| Failover not triggering | Health check not failing | Check health check endpoint and threshold |
| Weighted routing not balanced | Cached responses | Use low TTL for weighted records |
| Zone delegation not working | NS records wrong at registrar | Verify NS at domain registrar matches Route 53 |
| Private zone not resolving | VPC not associated | Associate VPC with private hosted zone |
| High query costs | Low TTL causing excessive queries | Increase TTL where appropriate |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Hosted Zone | Container for DNS records ($0.50/month) |
| ALIAS | AWS-specific, works at apex, free queries for AWS resources |
| Simple Routing | Basic one-to-one or round-robin |
| Weighted | Traffic split by percentage (canary) |
| Latency | Route to lowest-latency region |
| Failover | Primary/secondary with health checks |
| Geolocation | Route by user's geographic location |
| Health Checks | Monitor endpoints, trigger failover |

---

## Quick Revision Questions

1. **Describe and compare all 6 Route 53 routing policies with use cases.**
2. **How would you set up canary deployment using Route 53 weighted routing?**
3. **What is the ALIAS record and why is it preferred over CNAME for AWS resources?**
4. **Design a multi-region failover architecture using Route 53.**
5. **How do Route 53 health checks work and what happens when they fail?**
6. **Compare Route 53 with Azure DNS and GCP Cloud DNS.**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Public vs Private DNS](04-public-vs-private-dns.md) | [README](../README.md) | [Unit 6: SSL/TLS Certificates →](../06-Network-Security/01-ssl-tls-certificates.md) |
