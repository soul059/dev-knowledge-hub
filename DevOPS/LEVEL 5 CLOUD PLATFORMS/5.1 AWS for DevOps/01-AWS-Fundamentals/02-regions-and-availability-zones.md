# Chapter 1.2: Regions and Availability Zones — Deep Dive

## Overview

Building on Chapter 1.1, this chapter dives deeper into the practical aspects of **Regions** and **Availability Zones** — how services behave across AZs, how to design for high availability, and how to manage multi-region architectures.

---

## Service Scope: Global vs Regional vs AZ-Level

Understanding which scope a service operates at is critical for architecture decisions.

```
┌───────────────────────────────────────────────────────────────┐
│                     SERVICE SCOPES                            │
│                                                               │
│  GLOBAL SERVICES              REGIONAL SERVICES               │
│  ─────────────────           ──────────────────               │
│  • IAM (Users, Roles)        • VPC                            │
│  • Route 53 (DNS)            • S3 (bucket in region)          │
│  • CloudFront (CDN)          • Lambda                         │
│  • WAF                       • API Gateway                    │
│  • AWS Organizations         • DynamoDB                       │
│                              • SNS / SQS                      │
│                                                               │
│  AZ-LEVEL SERVICES                                            │
│  ─────────────────                                            │
│  • EC2 instances (launched in specific AZ)                    │
│  • EBS volumes (tied to one AZ)                               │
│  • RDS primary instance (in one AZ, standby in another)       │
│  • Subnets (each subnet lives in one AZ)                      │
│  • ElastiCache nodes                                          │
└───────────────────────────────────────────────────────────────┘
```

### Service Scope Table

| Scope | Examples | Key Implication |
|-------|----------|-----------------|
| **Global** | IAM, Route 53, CloudFront | No region selection needed; data replicated globally |
| **Regional** | S3, Lambda, DynamoDB, VPC | Created per-region; must configure cross-region replication manually |
| **AZ-Level** | EC2, EBS, Subnet, RDS instance | Must deploy in right AZ; no automatic cross-AZ migration |

---

## Designing for High Availability

### Single-AZ Architecture (DON'T do this in production)

```
    ┌─────────────── us-east-1 ───────────────┐
    │                                          │
    │  ┌─────────── AZ: us-east-1a ────────┐  │
    │  │                                    │  │
    │  │  ┌──────┐    ┌──────┐   ┌───────┐ │  │
    │  │  │ EC2  │───►│ RDS  │   │  EBS  │ │  │
    │  │  │ App  │    │  DB  │   │  Vol  │ │  │
    │  │  └──────┘    └──────┘   └───────┘ │  │
    │  │                                    │  │
    │  │  ⚠ AZ failure = TOTAL DOWNTIME    │  │
    │  └────────────────────────────────────┘  │
    └──────────────────────────────────────────┘
```

### Multi-AZ Architecture (RECOMMENDED)

```
    ┌──────────────────── us-east-1 ─────────────────────────┐
    │                                                         │
    │              ┌────────────────────┐                     │
    │              │  Application Load  │                     │
    │              │    Balancer (ALB)  │                     │
    │              └────────┬───────────┘                     │
    │                 ┌─────┴─────┐                           │
    │                 ▼           ▼                           │
    │  ┌──── AZ: 1a ─────┐  ┌──── AZ: 1b ─────┐            │
    │  │  ┌──────────┐   │  │  ┌──────────┐   │            │
    │  │  │  EC2 (1) │   │  │  │  EC2 (2) │   │            │
    │  │  └────┬─────┘   │  │  └────┬─────┘   │            │
    │  │       │         │  │       │         │            │
    │  │  ┌────▼─────┐   │  │  ┌────▼─────┐   │            │
    │  │  │RDS Primary│◄─────►│RDS Standby│   │            │
    │  │  └──────────┘   │  │  └──────────┘   │            │
    │  │                 │  │   (auto-failover)│            │
    │  └─────────────────┘  └─────────────────┘            │
    │                                                         │
    │  ✓ AZ-1a failure → ALB routes to AZ-1b                │
    │  ✓ RDS auto-failover to standby                        │
    └─────────────────────────────────────────────────────────┘
```

### Multi-Region Architecture (Disaster Recovery)

```
    ┌─────── us-east-1 (Primary) ───────┐    ┌─────── eu-west-1 (DR) ──────────┐
    │                                    │    │                                  │
    │  ┌──────┐  ┌──────┐  ┌──────┐    │    │  ┌──────┐  ┌──────┐  ┌──────┐  │
    │  │ EC2  │  │ RDS  │  │  S3  │    │    │  │ EC2  │  │ RDS  │  │  S3  │  │
    │  │ (x3) │  │Primary│  │Bucket│───────────►│(x3)  │  │Replica│  │Bucket│  │
    │  └──────┘  └──────┘  └──────┘    │    │  └──────┘  └──────┘  └──────┘  │
    │                                    │    │                                  │
    └────────────────────────────────────┘    └──────────────────────────────────┘
                     │                                        ▲
                     │          Route 53                      │
                     │     (Failover Routing)                 │
                     └────────────────────────────────────────┘
```

---

## AZ IDs vs AZ Names

**Important concept:** AWS randomizes AZ names per account. Your `us-east-1a` may be a different physical AZ than another account's `us-east-1a`.

```
    Account A                          Account B
    ──────────                         ──────────
    us-east-1a → AZ ID: use1-az1      us-east-1a → AZ ID: use1-az4
    us-east-1b → AZ ID: use1-az2      us-east-1b → AZ ID: use1-az1
    us-east-1c → AZ ID: use1-az4      us-east-1c → AZ ID: use1-az2
```

**AZ IDs** (like `use1-az1`) are consistent across accounts. Use AZ IDs when coordinating resources across accounts.

```bash
# Get AZ ID mappings for your account
aws ec2 describe-availability-zones \
  --region us-east-1 \
  --query "AvailabilityZones[].{Name:ZoneName,ID:ZoneId,State:State}" \
  --output table
```

---

## Data Transfer Costs Between AZs and Regions

Understanding data transfer costs is essential for cost optimization.

```
┌────────────────────────────────────────────────────────────┐
│              DATA TRANSFER COST MODEL                      │
│                                                            │
│  Same AZ:                                                  │
│  ┌──────┐ ──── FREE ────► ┌──────┐                        │
│  │ EC2  │  (private IP)    │ EC2  │                        │
│  └──────┘                  └──────┘                        │
│                                                            │
│  Cross-AZ (same region):                                   │
│  ┌──────┐ ── $0.01/GB ──► ┌──────┐                        │
│  │ EC2  │  (each way)      │ EC2  │                        │
│  │ AZ-a │                  │ AZ-b │                        │
│  └──────┘                  └──────┘                        │
│                                                            │
│  Cross-Region:                                             │
│  ┌──────┐ ── $0.02/GB ──► ┌──────┐                        │
│  │ EC2  │  (varies)        │ EC2  │                        │
│  │us-e-1│                  │eu-w-1│                        │
│  └──────┘                  └──────┘                        │
│                                                            │
│  To Internet:                                              │
│  ┌──────┐ ─ $0.09/GB* ──► ☁ Internet                     │
│  │ EC2  │  (first 10TB)                                    │
│  └──────┘                                                  │
│  *First 100GB/month free                                   │
└────────────────────────────────────────────────────────────┘
```

---

## Practical Commands

### Working with Regions and AZs

```bash
# List all regions
aws ec2 describe-regions --output table

# List AZs in a specific region
aws ec2 describe-availability-zones --region us-east-1 --output table

# List only available AZs (not impaired)
aws ec2 describe-availability-zones \
  --region us-east-1 \
  --filters "Name=state,Values=available" \
  --query "AvailabilityZones[].ZoneName" --output text

# Check which AZ your EC2 instance is in
aws ec2 describe-instances \
  --instance-ids i-0123456789abcdef \
  --query "Reservations[].Instances[].Placement.AvailabilityZone"

# Launch instance in specific AZ
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --instance-type t3.micro \
  --placement AvailabilityZone=us-east-1a \
  --subnet-id subnet-0123456789abcdef
```

### Restrict Regions with IAM Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyNonApprovedRegions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "us-east-1",
            "eu-west-1"
          ]
        }
      }
    }
  ]
}
```

---

## Real-World Scenarios

### Scenario 1: E-Commerce Platform
- **Requirement:** Serve customers in US and EU with <100ms latency
- **Solution:** Deploy in `us-east-1` and `eu-west-1`. Use Route 53 latency-based routing. CloudFront for static assets globally.

### Scenario 2: Healthcare Application (HIPAA)
- **Requirement:** All data must remain in US
- **Solution:** Use only US regions. Apply IAM `aws:RequestedRegion` condition to prevent accidental deployment outside US. Sign BAA with AWS.

### Scenario 3: Financial Trading Platform
- **Requirement:** Ultra-low latency in New York
- **Solution:** Use `us-east-1` Local Zone in NYC (`us-east-1-nyc-1a`) for trading engines. Main application in `us-east-1` for cost efficiency.

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Can't find subnet in another AZ | Subnets are AZ-specific | Create new subnet in target AZ |
| EBS volume can't attach to EC2 | EBS and EC2 must be in same AZ | Create snapshot → restore in correct AZ |
| Cross-AZ costs unexpectedly high | Using public IPs between AZs | Use private IPs; review VPC endpoints |
| RDS failover causing app issues | DNS TTL too high in application | Use short TTL; use RDS Proxy |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Service Scope** | Global (IAM, Route53), Regional (VPC, S3, Lambda), AZ-Level (EC2, EBS, Subnet) |
| **Multi-AZ** | Deploy across 2+ AZs for HA. ALB + Auto Scaling handle failover automatically. |
| **Multi-Region** | Use for DR, compliance, latency. Route 53 for DNS failover. |
| **AZ IDs** | Consistent physical identifiers across accounts (unlike randomized AZ names). |
| **Data Transfer** | Same-AZ free (private IP), Cross-AZ ~$0.01/GB, Cross-Region ~$0.02/GB. |
| **Region Restrictions** | Use IAM condition `aws:RequestedRegion` to enforce compliance. |

---

## Quick Revision Questions

1. **Why does AWS randomize AZ names across accounts?**
   > To distribute load evenly across AZs. Without randomization, everyone would pick `us-east-1a` first, overloading that physical AZ.

2. **An EBS volume is in `us-east-1a`. Can you attach it to an EC2 instance in `us-east-1b`?**
   > No. EBS volumes are AZ-locked. You must create a snapshot and restore it in the target AZ.

3. **Is S3 a regional or AZ-level service?**
   > Regional. S3 automatically replicates data across at least 3 AZs within the region.

4. **How do you prevent users from launching resources in unauthorized regions?**
   > Use an IAM policy with a `Deny` effect and `aws:RequestedRegion` condition key, or use AWS Organizations SCP.

5. **What's the data transfer cost for traffic between two EC2 instances in the same AZ using private IPs?**
   > Free ($0.00/GB).

6. **Name two AWS services that are GLOBAL (not region-specific).**
   > IAM and Route 53 (also CloudFront, WAF, AWS Organizations).

---

[← Previous: 1.1 AWS Global Infrastructure](01-aws-global-infrastructure.md) | [Next: 1.3 AWS Console, CLI, SDK →](03-aws-console-cli-sdk.md)

[← Back to README](../README.md)
