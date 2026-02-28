# Chapter 10.4: AWS WAF & Shield

## Overview

**AWS WAF (Web Application Firewall)** protects web applications from common exploits (SQL injection, XSS, bots) at the application layer (Layer 7). **AWS Shield** provides DDoS protection at the network/transport layers (Layer 3/4). Together, they defend AWS workloads against both application-layer attacks and volumetric DDoS attacks.

---

## WAF Architecture

```
┌──── WAF Request Flow ────────────────────────────────────┐
│                                                           │
│  Internet                                                 │
│     │                                                     │
│     ▼                                                     │
│  ┌──────────────────────────────────┐                    │
│  │  AWS WAF (Web ACL)               │                    │
│  │                                   │                    │
│  │  Rule 1: Rate-limit (2000/5min)  │ → Block if exceed │
│  │  Rule 2: SQL Injection           │ → Block            │
│  │  Rule 3: XSS Detection           │ → Block            │
│  │  Rule 4: Geo Restrict (Block CN)  │ → Block            │
│  │  Rule 5: AWS Managed Bot Control │ → Block bots       │
│  │  Default Action                   │ → Allow            │
│  └─────────────┬────────────────────┘                    │
│                │                                          │
│       Allowed requests pass through                      │
│                │                                          │
│     ┌──────────┴──────────┐                              │
│     ▼                     ▼                              │
│  ┌────────┐        ┌───────────┐        ┌──────────┐   │
│  │CloudFr.│        │    ALB    │        │ API GW   │   │
│  └────────┘        └───────────┘        └──────────┘   │
│                                                           │
│  WAF can be attached to:                                 │
│  • CloudFront distributions                              │
│  • Application Load Balancers                            │
│  • API Gateway REST APIs                                 │
│  • AppSync GraphQL APIs                                  │
│  • Cognito User Pools                                    │
│  ⚠ NOT Network Load Balancers                           │
└───────────────────────────────────────────────────────────┘
```

---

## Web ACL Configuration

```bash
# Create Web ACL
aws wafv2 create-web-acl \
  --name "production-web-acl" \
  --scope REGIONAL \
  --default-action '{"Allow":{}}' \
  --visibility-config '{
    "SampledRequestsEnabled": true,
    "CloudWatchMetricsEnabled": true,
    "MetricName": "productionWebACL"
  }' \
  --rules file://waf-rules.json

# For CloudFront, use --scope CLOUDFRONT and region us-east-1
```

### WAF Rules JSON

```json
[
  {
    "Name": "AWSManagedRulesCommonRuleSet",
    "Priority": 1,
    "OverrideAction": { "None": {} },
    "Statement": {
      "ManagedRuleGroupStatement": {
        "VendorName": "AWS",
        "Name": "AWSManagedRulesCommonRuleSet"
      }
    },
    "VisibilityConfig": {
      "SampledRequestsEnabled": true,
      "CloudWatchMetricsEnabled": true,
      "MetricName": "CommonRuleSet"
    }
  },
  {
    "Name": "RateLimitRule",
    "Priority": 2,
    "Action": { "Block": {} },
    "Statement": {
      "RateBasedStatement": {
        "Limit": 2000,
        "AggregateKeyType": "IP"
      }
    },
    "VisibilityConfig": {
      "SampledRequestsEnabled": true,
      "CloudWatchMetricsEnabled": true,
      "MetricName": "RateLimit"
    }
  },
  {
    "Name": "GeoBlockRule",
    "Priority": 3,
    "Action": { "Block": {} },
    "Statement": {
      "GeoMatchStatement": {
        "CountryCodes": ["CN", "RU"]
      }
    },
    "VisibilityConfig": {
      "SampledRequestsEnabled": true,
      "CloudWatchMetricsEnabled": true,
      "MetricName": "GeoBlock"
    }
  },
  {
    "Name": "SQLInjectionRule",
    "Priority": 4,
    "Action": { "Block": {} },
    "Statement": {
      "SqliMatchStatement": {
        "FieldToMatch": { "AllQueryArguments": {} },
        "TextTransformations": [
          { "Priority": 0, "Type": "URL_DECODE" },
          { "Priority": 1, "Type": "HTML_ENTITY_DECODE" }
        ]
      }
    },
    "VisibilityConfig": {
      "SampledRequestsEnabled": true,
      "CloudWatchMetricsEnabled": true,
      "MetricName": "SQLInjection"
    }
  }
]
```

---

## AWS Managed Rule Groups

```
┌──── Popular Managed Rule Groups ─────────────────────────┐
│                                                           │
│  Rule Group                    Protects Against          │
│  ────────────────────────────  ───────────────────────── │
│  AWSManagedRulesCommonRuleSet  OWASP Top 10 basics      │
│  AWSManagedRulesSQLiRuleSet    SQL injection             │
│  AWSManagedRulesKnownBadInputs Log4j, path traversal   │
│  AWSManagedRulesBotControlRuleSet Automated bots        │
│  AWSManagedRulesATPRuleSet     Account takeover (ATP)   │
│  AWSManagedRulesLinuxRuleSet   Linux-specific LFI       │
│  AWSManagedRulesAmazonIpReputationList Malicious IPs    │
│  AWSManagedRulesAnonymousIpList VPNs, proxies, Tor      │
│                                                           │
│  ⚠ WCU limit: 5000 per Web ACL (Web ACL Capacity Units)│
│    Each rule group consumes a certain number of WCUs     │
└───────────────────────────────────────────────────────────┘
```

---

## WAF Logging

```bash
# Enable WAF logging to S3 (via Kinesis Data Firehose)
aws wafv2 put-logging-configuration \
  --logging-configuration '{
    "ResourceArn": "arn:aws:wafv2:us-east-1:123456789012:regional/webacl/production-web-acl/abc123",
    "LogDestinationConfigs": [
      "arn:aws:firehose:us-east-1:123456789012:deliverystream/aws-waf-logs-prod"
    ],
    "RedactedFields": [
      { "SingleHeader": { "Name": "authorization" } }
    ]
  }'

# Log destinations:
# • S3 bucket (via Kinesis Data Firehose)   — prefix: aws-waf-logs-
# • CloudWatch Logs                          — prefix: aws-waf-logs-
# • S3 directly (WAFv2)                     — prefix: aws-waf-logs-
```

---

## AWS Shield

```
┌──── Shield Protection Tiers ─────────────────────────────┐
│                                                           │
│  ┌──────────────────────────────────────────────────┐    │
│  │  Shield Standard (FREE — always on)              │    │
│  │                                                   │    │
│  │  • Automatic protection for ALL AWS customers    │    │
│  │  • Layer 3/4 DDoS protection                     │    │
│  │  • SYN flood, UDP reflection, DNS amplification  │    │
│  │  • Protects: CloudFront, Route 53, ELB           │    │
│  │  • No cost, no setup required                    │    │
│  └──────────────────────────────────────────────────┘    │
│                                                           │
│  ┌──────────────────────────────────────────────────┐    │
│  │  Shield Advanced ($3,000/month + 1-year commit)  │    │
│  │                                                   │    │
│  │  • Layer 3/4/7 DDoS protection                   │    │
│  │  • Near real-time attack visibility              │    │
│  │  • 24/7 DDoS Response Team (SRT)                 │    │
│  │  • Cost protection (refund DDoS scaling costs)   │    │
│  │  • Automatic application layer mitigations       │    │
│  │  • Health-based detection                        │    │
│  │  • Protects: CloudFront, Route 53, ELB,          │    │
│  │              EC2, Global Accelerator              │    │
│  │  • WAF included at no additional cost            │    │
│  └──────────────────────────────────────────────────┘    │
│                                                           │
│  Shield Advanced + WAF = complete L3-L7 protection      │
└───────────────────────────────────────────────────────────┘
```

---

## Shield Advanced Setup

```bash
# Subscribe to Shield Advanced
aws shield create-subscription

# Create protection for ALB
aws shield create-protection \
  --name "prod-alb-protection" \
  --resource-arn "arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/prod-alb/abc123"

# Create protection for CloudFront
aws shield create-protection \
  --name "prod-cf-protection" \
  --resource-arn "arn:aws:cloudfront::123456789012:distribution/E1ABC2DEF3"

# Describe active attack
aws shield describe-attack --attack-id "abc-123"

# List protections
aws shield list-protections
```

---

## DDoS Resilient Architecture

```
┌──── DDoS Resilient Design ───────────────────────────────┐
│                                                           │
│  Users                                                    │
│    │                                                      │
│    ▼                                                      │
│  ┌──────────────┐                                        │
│  │  Route 53    │ ← Shield Standard (automatic)         │
│  └──────┬───────┘                                        │
│         │                                                 │
│         ▼                                                 │
│  ┌──────────────┐                                        │
│  │  CloudFront  │ ← Shield + WAF (edge filtering)       │
│  │  (Edge)      │   Absorbs volumetric attacks          │
│  └──────┬───────┘                                        │
│         │                                                 │
│         ▼                                                 │
│  ┌──────────────┐                                        │
│  │  WAF         │ ← Layer 7 rules (rate limit, SQLi)    │
│  │  (Web ACL)   │                                        │
│  └──────┬───────┘                                        │
│         │                                                 │
│         ▼                                                 │
│  ┌──────────────┐                                        │
│  │  ALB         │ ← Shield Standard (automatic)         │
│  │  (Regional)  │   Only receives clean traffic         │
│  └──────┬───────┘                                        │
│         │                                                 │
│         ▼                                                 │
│  ┌──────────────┐                                        │
│  │  Auto Scaling│ ← Scales to handle remaining load     │
│  │  Group       │                                        │
│  └──────────────┘                                        │
│                                                           │
│  Key: Push filtering to the edge (CloudFront + WAF)     │
│       so origin only processes legitimate traffic        │
└───────────────────────────────────────────────────────────┘
```

---

## AWS Firewall Manager

```
┌──── Firewall Manager ────────────────────────────────────┐
│                                                           │
│  Centrally manage security rules across accounts         │
│  (requires AWS Organizations)                            │
│                                                           │
│  Manages:                                                │
│  ├── WAF rules               across all ALBs / CF       │
│  ├── Shield Advanced         across all accounts        │
│  ├── Security Groups         across all VPCs            │
│  ├── Network Firewall        across all VPCs            │
│  └── Route 53 Resolver DNS   Firewall rules             │
│                                                           │
│  Benefits:                                               │
│  • Auto-apply rules to new resources                    │
│  • Compliance dashboard                                 │
│  • Central policy management from security account      │
│                                                           │
│  Pre-requisite: AWS Config must be enabled               │
└───────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Legitimate traffic blocked by WAF | Overly aggressive rule | Use Count mode first to test; check sampled requests |
| WCU limit exceeded | Too many complex rules | Optimize rules; remove unused; consider Firewall Manager |
| WAF not blocking attacks | Rules in Count (monitor) mode | Change action from Count to Block after testing |
| Shield Advanced cost concerns | $3,000/month minimum | Evaluate ROI vs business impact of downtime |
| Cannot attach WAF to NLB | WAF only supports ALB | Place CloudFront or ALB in front, or use Network Firewall |
| WAF logs missing | Logging not configured | Enable logging to S3 / CloudWatch / Firehose |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **WAF** | Layer 7 firewall. Attach to ALB, CloudFront, API GW. Rules for SQLi, XSS, bots, geo, rate-limit. |
| **Web ACL** | Container for WAF rules. Priority-ordered. 5000 WCU limit. |
| **Managed Rules** | AWS-curated rule sets (OWASP, bots, known bad inputs). Quick protection. |
| **Shield Standard** | Free. Automatic L3/L4 DDoS protection for all AWS customers. |
| **Shield Advanced** | $3,000/month. L3-L7 DDoS, SRT team, cost protection, WAF included. |
| **Firewall Manager** | Centrally manage WAF/Shield/SG across multi-account organizations. |

---

## Quick Revision Questions

1. **What layers does WAF protect vs Shield Standard?**
   > WAF protects Layer 7 (application — HTTP/HTTPS). Shield Standard protects Layer 3/4 (network/transport — SYN floods, UDP reflection).

2. **Can you attach WAF to a Network Load Balancer?**
   > No. WAF can only be attached to ALB, CloudFront, API Gateway, AppSync, and Cognito User Pools.

3. **What is a rate-based rule in WAF?**
   > A rule that counts requests from each source IP over a 5-minute window and blocks IPs exceeding the threshold (minimum: 100 requests).

4. **What does Shield Advanced provide over Standard?**
   > 24/7 DDoS Response Team, cost protection for scaling charges, Layer 7 protection, near real-time visibility, health-based detection, and WAF at no extra cost.

5. **How do you test WAF rules without blocking traffic?**
   > Set the rule action to **Count** mode. This logs matching requests without blocking them, letting you verify accuracy before switching to Block.

---

[← Previous: 10.3 Secrets Manager & KMS](03-secrets-manager-kms.md) | [Next: 10.5 Security Hub & GuardDuty →](05-security-hub-guardduty.md)

[← Back to README](../README.md)
