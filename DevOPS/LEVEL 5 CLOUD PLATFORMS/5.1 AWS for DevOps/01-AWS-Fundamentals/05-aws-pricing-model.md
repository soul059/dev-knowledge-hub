# Chapter 1.5: AWS Pricing Model

## Overview

Understanding AWS pricing is essential for DevOps engineers. AWS uses a **pay-as-you-go model** where you pay only for the resources you consume, with no upfront costs or long-term commitments (unless you opt in). Mastering pricing helps you design cost-effective architectures and avoid surprise bills.

---

## Core Pricing Principles

```
┌─────────────────────────────────────────────────────────────────┐
│              AWS PRICING FUNDAMENTALS                           │
│                                                                 │
│  1. PAY-AS-YOU-GO ──► Use it? Pay for it. Stop? Stop paying.  │
│                                                                 │
│  2. PAY LESS WHEN   ──► Volume discounts on S3, data transfer │
│     YOU USE MORE                                                │
│                                                                 │
│  3. SAVE WHEN YOU   ──► Reserved Instances, Savings Plans,    │
│     COMMIT            Spot Instances                            │
│                                                                 │
│  4. FREE TIER ──────► 12-month + always-free offerings        │
│                                                                 │
│  Key facts:                                                     │
│  • No upfront costs (unless you choose Reserved)               │
│  • No termination fees                                          │
│  • Pricing varies by region                                     │
│  • Data IN is free; data OUT costs money                       │
└─────────────────────────────────────────────────────────────────┘
```

---

## AWS Free Tier

Three types of free offerings:

| Type | Duration | Examples |
|------|----------|---------|
| **12-Month Free** | First 12 months after signup | 750 hrs/mo EC2 t2.micro, 5GB S3, 750 hrs/mo RDS |
| **Always Free** | Never expires | 1M Lambda requests/mo, 25GB DynamoDB, 10 CloudWatch alarms |
| **Trials** | Service-specific trial period | 60 days of Inspector, 30 days of GuardDuty |

```
┌───────────────────────── FREE TIER HIGHLIGHTS ──────────────────┐
│                                                                  │
│  EC2:        750 hrs/mo of t2.micro or t3.micro (12-mo)        │
│  S3:         5 GB storage + 20K GET + 2K PUT (12-mo)           │
│  RDS:        750 hrs/mo of db.t2.micro (12-mo)                 │
│  Lambda:     1M requests + 400K GB-seconds/mo (always free)    │
│  DynamoDB:   25 GB + 25 RCU + 25 WCU (always free)            │
│  CloudWatch: 10 alarms + 1M API calls (always free)            │
│  SNS:        1M publishes (always free)                         │
│  SQS:        1M requests/mo (always free)                       │
│  CloudFront: 1TB data out + 10M requests/mo (12-mo)            │
│                                                                  │
│  ⚠ Set up billing alerts to avoid surprise charges!            │
└──────────────────────────────────────────────────────────────────┘
```

---

## Service Pricing Models

### EC2 Pricing Options

```
┌───────────────────────────────────────────────────────────────┐
│                    EC2 PRICING OPTIONS                         │
│                                                               │
│  ┌─────────────────┐  Cost: $$$$  Flexibility: ★★★★★        │
│  │  ON-DEMAND       │  Pay per second/hour                    │
│  │  No commitment   │  Best for: short-term, unpredictable   │
│  └─────────────────┘                                          │
│                                                               │
│  ┌─────────────────┐  Cost: $$ (up to 72% off)               │
│  │  RESERVED (RI)   │  1 or 3-year commitment                 │
│  │  Predictable use │  Best for: steady-state workloads       │
│  └─────────────────┘                                          │
│                                                               │
│  ┌─────────────────┐  Cost: $$ (up to 72% off)               │
│  │  SAVINGS PLANS   │  $/hr commitment (flexible)             │
│  │  More flexible   │  Best for: variable instance types      │
│  └─────────────────┘                                          │
│                                                               │
│  ┌─────────────────┐  Cost: $ (up to 90% off!)               │
│  │  SPOT INSTANCES  │  Bid on unused capacity                  │
│  │  Can be revoked! │  Best for: fault-tolerant, batch jobs   │
│  └─────────────────┘                                          │
│                                                               │
│  ┌─────────────────┐  Cost: $$$ (higher per hour)            │
│  │  DEDICATED HOST  │  Physical server for you alone          │
│  │  Compliance need │  Best for: licensing, regulatory        │
│  └─────────────────┘                                          │
└───────────────────────────────────────────────────────────────┘
```

### EC2 Pricing Comparison Table

| Option | Discount | Commitment | Risk | Best For |
|--------|----------|------------|------|----------|
| **On-Demand** | None (baseline) | None | None | Dev/test, short-term, unpredictable |
| **Reserved (RI)** | Up to 72% | 1 or 3 years | Locked in | Steady-state production workloads |
| **Savings Plans** | Up to 72% | $/hr commitment | Less flexible | Variable instance types/regions |
| **Spot** | Up to 90% | None | Can be interrupted | Batch processing, CI/CD, data analysis |
| **Dedicated Host** | On-demand pricing | Optional | None | BYOL, compliance, regulatory |

### Spot Instance Strategy

```
┌────────────────────────────────────────────────────────┐
│              SPOT INSTANCE LIFECYCLE                    │
│                                                        │
│  You set max price                                     │
│       │                                                │
│       ▼                                                │
│  ┌──────────────┐   Spot price                        │
│  │ Instance runs │ < your max ──► ✓ Keep running     │
│  └──────┬───────┘                                     │
│         │                                              │
│     Spot price                                         │
│   > your max ──► ⚠ 2-minute warning                  │
│                  ──► Instance terminated/stopped        │
│                                                        │
│  USE FOR:                     AVOID FOR:               │
│  • Batch processing           • Databases              │
│  • CI/CD build agents         • Stateful apps          │
│  • Data analysis              • Critical user-facing   │
│  • ML training                • Real-time processing   │
│  • Rendering                                           │
└────────────────────────────────────────────────────────┘
```

---

## S3 Pricing Components

```
┌────────────────────────────────────────────────────────┐
│                  S3 PRICING                            │
│                                                        │
│  Storage:     $/GB/month (varies by storage class)    │
│  Requests:    $/1000 requests (GET, PUT, etc.)        │
│  Data OUT:    $/GB (data transfer out to internet)    │
│  Data IN:     FREE                                     │
│  Lifecycle:   FREE (transition rules)                  │
│                                                        │
│  Storage Class      $/GB/mo    Retrieval     Use Case │
│  ───────────────────────────────────────────────────── │
│  Standard           $0.023     Instant       Frequent │
│  Standard-IA        $0.0125    $/GB fee      Infrequt │
│  One Zone-IA        $0.010     $/GB fee      Non-crit │
│  Glacier Instant    $0.004     Instant       Archive  │
│  Glacier Flexible   $0.0036    Min-hours     Archive  │
│  Glacier Deep       $0.00099   12-48 hours   Comply   │
│                                                        │
│  * Prices shown for us-east-1, subject to change      │
└────────────────────────────────────────────────────────┘
```

---

## Data Transfer Pricing

```
┌──────────────────────────────────────────────────────┐
│            DATA TRANSFER COSTS                       │
│                                                      │
│  ┌──────────┐                                        │
│  │          │◄────── Data IN: FREE (always)          │
│  │   AWS    │                                        │
│  │  Region  │──────► Data OUT: $0.09/GB*             │
│  │          │                                        │
│  │          │◄─────► Same AZ (private IP): FREE      │
│  │          │                                        │
│  │          │◄─────► Cross-AZ: $0.01/GB each way     │
│  │          │                                        │
│  │          │◄─────► Cross-Region: $0.02/GB          │
│  └──────────┘                                        │
│                                                      │
│  * First 100 GB/month FREE (data out to internet)   │
│  * $0.09/GB for next 10 TB, then decreasing tiers   │
│  * CloudFront can reduce data transfer costs!        │
└──────────────────────────────────────────────────────┘
```

---

## Cost Management Tools

### AWS-Native Tools

```
┌────────────────────────────────────────────────────────────┐
│              AWS COST MANAGEMENT TOOLS                      │
│                                                            │
│  ┌───────────────────┐  ┌───────────────────┐             │
│  │  AWS Cost Explorer │  │  AWS Budgets      │             │
│  │  ─────────────────│  │  ─────────────────│             │
│  │  • Visualize costs │  │  • Set spending   │             │
│  │  • Forecast future │  │    limits          │             │
│  │  • Filter by       │  │  • Email/SNS      │             │
│  │    service/tag     │  │    alerts          │             │
│  │  • RI utilization  │  │  • Auto-actions    │             │
│  └───────────────────┘  └───────────────────┘             │
│                                                            │
│  ┌───────────────────┐  ┌───────────────────┐             │
│  │  Cost & Usage Rpt │  │  Trusted Advisor  │             │
│  │  ─────────────────│  │  ─────────────────│             │
│  │  • Detailed CSV    │  │  • Cost optimiz.  │             │
│  │  • S3 delivery     │  │  • Idle resources │             │
│  │  • Athena queries  │  │  • Underutilized  │             │
│  │  • Hourly/daily    │  │  • RI coverage    │             │
│  └───────────────────┘  └───────────────────┘             │
└────────────────────────────────────────────────────────────┘
```

### Setting Up a Budget Alert

```bash
# Create a monthly budget with email alert
aws budgets create-budget \
  --account-id 123456789012 \
  --budget '{
    "BudgetName": "Monthly-100USD",
    "BudgetLimit": {
      "Amount": "100",
      "Unit": "USD"
    },
    "TimeUnit": "MONTHLY",
    "BudgetType": "COST"
  }' \
  --notifications-with-subscribers '[
    {
      "Notification": {
        "NotificationType": "ACTUAL",
        "ComparisonOperator": "GREATER_THAN",
        "Threshold": 80,
        "ThresholdType": "PERCENTAGE"
      },
      "Subscribers": [
        {
          "SubscriptionType": "EMAIL",
          "Address": "devops@company.com"
        }
      ]
    }
  ]'
```

### CloudWatch Billing Alarm

```bash
# Enable billing alerts first (only in us-east-1)
# Then create alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "BillingAlarm-50USD" \
  --alarm-description "Alert when bill exceeds $50" \
  --metric-name EstimatedCharges \
  --namespace AWS/Billing \
  --statistic Maximum \
  --period 21600 \
  --threshold 50 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=Currency,Value=USD \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:billing-alerts \
  --region us-east-1
```

---

## Cost Optimization Strategies

```
┌────────────────────────────────────────────────────────────────┐
│             COST OPTIMIZATION CHECKLIST                        │
│                                                                │
│  COMPUTE                                                       │
│  ☐ Right-size instances (use Cost Explorer recommendations)   │
│  ☐ Use Spot for fault-tolerant workloads (up to 90% off)     │
│  ☐ Reserved Instances for steady-state (up to 72% off)       │
│  ☐ Stop/terminate unused instances                            │
│  ☐ Use Lambda for bursty, short-duration tasks               │
│                                                                │
│  STORAGE                                                       │
│  ☐ S3 Lifecycle policies (move to cheaper classes)            │
│  ☐ Delete unused EBS volumes and snapshots                    │
│  ☐ Use S3 Intelligent-Tiering for unknown access patterns    │
│                                                                │
│  NETWORKING                                                    │
│  ☐ Use VPC Endpoints to avoid NAT Gateway charges            │
│  ☐ Use CloudFront to reduce data transfer costs              │
│  ☐ Use private IPs for same-AZ communication                 │
│                                                                │
│  DATABASE                                                      │
│  ☐ Use Reserved Instances for production RDS                  │
│  ☐ Stop dev/test RDS instances after hours                    │
│  ☐ Use Aurora Serverless for variable workloads              │
│                                                                │
│  MONITORING                                                    │
│  ☐ Set up AWS Budgets with alerts                             │
│  ☐ Review Cost Explorer monthly                               │
│  ☐ Use Resource tagging for cost allocation                   │
│  ☐ Enable Trusted Advisor cost checks                         │
└────────────────────────────────────────────────────────────────┘
```

---

## AWS Pricing Calculator

Use the **[AWS Pricing Calculator](https://calculator.aws/)** to estimate costs before deploying.

```bash
# Example: Estimate monthly cost for a web application
#
# ┌──────────────────────────────────────────────┐
# │  Resource           │  Estimated Cost/Month  │
# ├─────────────────────┼────────────────────────┤
# │  2x t3.medium EC2   │  ~$60                  │
# │  ALB                │  ~$22                  │
# │  100GB gp3 EBS (x2) │  ~$16                  │
# │  S3 (50GB + 1M req) │  ~$2                   │
# │  RDS db.t3.medium   │  ~$50                  │
# │  NAT Gateway        │  ~$32 + data           │
# │  CloudWatch         │  ~$5                   │
# │  Data Transfer      │  ~$10                  │
# ├─────────────────────┼────────────────────────┤
# │  TOTAL              │  ~$197/month           │
# └──────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Startup Minimizing Costs
- Use Free Tier for first 12 months
- t3.micro for dev/test environments
- Spot Instances for CI/CD build agents
- S3 Intelligent-Tiering for variable access patterns
- Lambda for API backend (pay per request)
- Set $50/month budget with email alerts

### Scenario 2: Enterprise Production Optimization
- Reserved Instances for baseline EC2/RDS (3-year, all upfront = max discount)
- Spot fleet for batch processing
- S3 Lifecycle policies: Standard → IA after 30 days → Glacier after 90 days
- VPC Endpoints for S3/DynamoDB (eliminate NAT costs)
- Consolidated billing across AWS Organizations

### Scenario 3: Cost Allocation by Team
- Use Resource Tags (`Team=Frontend`, `Env=Production`)
- Enable Cost Allocation Tags in Billing console
- Create AWS Budgets per team
- Use Cost Explorer filtered by tag

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|----------|
| Unexpected EC2 charges | Forgot to stop/terminate instances | Set up billing alarms; use auto-stop scripts |
| High data transfer bill | Cross-region or internet egress | Use CloudFront, VPC endpoints, private IPs |
| NAT Gateway costs high | All traffic routed through NAT | Use VPC endpoints for AWS services (S3, DynamoDB) |
| EBS charges for stopped EC2 | EBS volumes persist when EC2 stopped | Delete unneeded volumes; consider instance store |
| S3 costs growing fast | No lifecycle policy | Implement S3 Lifecycle rules to archive/delete old data |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Pay-As-You-Go** | No upfront. Pay per second/hour/GB. Stop using = stop paying. |
| **Free Tier** | 12-month free + always-free offerings. Set billing alerts! |
| **EC2 Pricing** | On-Demand > Reserved (72% off) > Spot (90% off). Match to workload. |
| **S3 Pricing** | Storage ($/GB/mo) + Requests ($/1K) + Data OUT ($/GB). Data IN free. |
| **Data Transfer** | Inbound free. Same-AZ free. Cross-AZ $0.01/GB. Internet $0.09/GB. |
| **Cost Tools** | Cost Explorer, Budgets, Trusted Advisor, Cost & Usage Report. |
| **Optimization** | Right-size, use Spot/RI, lifecycle policies, VPC endpoints, tagging. |

---

## Quick Revision Questions

1. **What are the three types of AWS Free Tier?**
   > 12-Month Free, Always Free, and Trials.

2. **Which EC2 pricing option gives the biggest discount and what's the catch?**
   > Spot Instances (up to 90% off), but they can be interrupted with 2-minute warning.

3. **Is data transfer INTO AWS free?**
   > Yes, inbound data transfer is always free. You pay for outbound.

4. **Name three AWS tools for monitoring and managing costs.**
   > AWS Cost Explorer, AWS Budgets, and AWS Trusted Advisor.

5. **How can you reduce NAT Gateway data transfer costs?**
   > Use VPC Gateway Endpoints for S3 and DynamoDB (free). This avoids routing traffic through the NAT Gateway.

6. **What's the difference between Reserved Instances and Savings Plans?**
   > RIs commit to a specific instance type/region. Savings Plans commit to a $/hr spend and apply more flexibly across instance types, regions, and even Fargate/Lambda.

---

[← Previous: 1.4 IAM Basics](04-iam-basics.md) | [Next: Unit 2 — EC2 Instances →](../02-Compute-Services/01-ec2-instances.md)

[← Back to README](../README.md)
