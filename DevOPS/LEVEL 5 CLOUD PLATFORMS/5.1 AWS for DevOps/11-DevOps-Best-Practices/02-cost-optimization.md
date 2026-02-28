# Chapter 11.2: Cost Optimization on AWS

## Overview

Cost optimization is about running systems at the lowest price point while delivering business value. AWS offers a **pay-as-you-go** model, but without disciplined management, costs can spiral. This chapter covers tools, strategies, and patterns to control and reduce AWS spending in a DevOps environment.

---

## Cost Management Architecture

```
┌──── Cost Management Stack ───────────────────────────────┐
│                                                           │
│  ┌─────────────────────────────────────────────────┐     │
│  │                AWS Billing Console               │     │
│  │  ┌───────────────────────────────────────────┐   │     │
│  │  │  Cost Explorer   │   Budgets   │   CUR*   │   │     │
│  │  └───────────────────────────────────────────┘   │     │
│  │                                                   │     │
│  │  ┌───────────────────────────────────────────┐   │     │
│  │  │  Savings Plans  │  Reserved Instances      │   │     │
│  │  └───────────────────────────────────────────┘   │     │
│  └─────────────────────────────────────────────────┘     │
│                                                           │
│  *CUR = Cost and Usage Reports                           │
│                                                           │
│  Optimization Tools:                                     │
│  ┌──────────────────┐  ┌────────────────────────┐       │
│  │ Compute Optimizer │  │ Trusted Advisor        │       │
│  │ (right-sizing)    │  │ (cost recommendations) │       │
│  └──────────────────┘  └────────────────────────┘       │
│  ┌──────────────────┐  ┌────────────────────────┐       │
│  │ S3 Storage Lens  │  │ Cost Anomaly Detection │       │
│  │ (storage insight) │  │ (spending alerts)      │       │
│  └──────────────────┘  └────────────────────────┘       │
└───────────────────────────────────────────────────────────┘
```

---

## Tagging Strategy for Cost Allocation

```
┌──── Tag-Based Cost Allocation ───────────────────────────┐
│                                                           │
│  Required Tags:                                          │
│  ├── Environment    → dev / staging / production        │
│  ├── Project        → project-name                      │
│  ├── Team           → team-name                         │
│  ├── CostCenter     → CC-12345                          │
│  └── Owner          → user@company.com                  │
│                                                           │
│  Steps:                                                  │
│  1. Define tag policy in AWS Organizations               │
│  2. Activate cost allocation tags in Billing             │
│  3. Enforce tags via SCPs or AWS Config rules            │
│  4. View costs by tag in Cost Explorer                   │
│                                                           │
│  ⚠ Tags take 24 hours to appear in billing reports     │
└───────────────────────────────────────────────────────────┘
```

```bash
# Enforce tags with SCP (deny untagged resources)
# See 10-Security/02-organizations-scps.md for SCP example

# Tag existing resources
aws ec2 create-tags \
  --resources i-1234567890abcdef0 \
  --tags Key=Environment,Value=production Key=Project,Value=api-service

# Activate cost allocation tag
aws ce update-cost-allocation-tags-status \
  --cost-allocation-tags-status '[{
    "TagKey": "Environment",
    "Status": "Active"
  }]'
```

---

## EC2 Pricing Models

```
┌──── EC2 Pricing Comparison ──────────────────────────────┐
│                                                           │
│  Model            Savings   Commitment   Best For        │
│  ──────────────── ──────── ──────────── ──────────────── │
│  On-Demand        0%        None         Short-term,     │
│                                          unpredictable   │
│                                                           │
│  Savings Plans    ≤72%      1 or 3 year  Steady baseline │
│  (Compute)                  $/hr commit  across instance │
│                                          types/regions   │
│                                                           │
│  Savings Plans    ≤72%      1 or 3 year  Specific        │
│  (EC2 Instance)             $/hr commit  family+region   │
│                                                           │
│  Reserved (RI)    ≤72%      1 or 3 year  Specific        │
│                             All/Partial/ instance type   │
│                             No Upfront   +AZ+tenancy     │
│                                                           │
│  Spot Instances   ≤90%      None (can    Fault-tolerant  │
│                             interrupt)   batch, CI/CD,   │
│                                          containers      │
│                                                           │
│  Recommendation: Savings Plans > RIs for flexibility    │
└───────────────────────────────────────────────────────────┘
```

```bash
# View Savings Plans recommendations
aws ce get-savings-plans-purchase-recommendation \
  --savings-plans-type COMPUTE_SP \
  --term-in-years ONE_YEAR \
  --payment-option NO_UPFRONT \
  --lookback-period-in-days SIXTY_DAYS

# View RI recommendations
aws ce get-reservation-purchase-recommendation \
  --service "Amazon Elastic Compute Cloud - Compute"
```

---

## AWS Budget Setup

```bash
# Create a monthly budget with alerts
aws budgets create-budget \
  --account-id 123456789012 \
  --budget '{
    "BudgetName": "Monthly-Total",
    "BudgetLimit": {
      "Amount": "5000",
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
      "Subscribers": [{
        "SubscriptionType": "EMAIL",
        "Address": "alerts@company.com"
      }]
    },
    {
      "Notification": {
        "NotificationType": "FORECASTED",
        "ComparisonOperator": "GREATER_THAN",
        "Threshold": 100,
        "ThresholdType": "PERCENTAGE"
      },
      "Subscribers": [{
        "SubscriptionType": "EMAIL",
        "Address": "alerts@company.com"
      }]
    }
  ]'
```

---

## Cost Optimization Strategies

```
┌──── Strategy Matrix ──────────────────────────────────────┐
│                                                            │
│  COMPUTE                                                   │
│  ├── Right-size: Use Compute Optimizer recommendations    │
│  ├── Graviton: 20-40% better price-performance           │
│  ├── Spot for CI/CD: CodeBuild, Jenkins agents, tests    │
│  ├── Scheduled scaling: Scale down non-prod at night     │
│  └── Lambda: Pay per invocation (idle = $0)              │
│                                                            │
│  STORAGE                                                   │
│  ├── S3 Intelligent-Tiering: Auto-moves infrequent data  │
│  ├── Lifecycle rules: Archive to Glacier after 90 days   │
│  ├── gp3 over gp2: 20% cheaper, better baseline IOPS    │
│  ├── Delete old snapshots: Use DLM lifecycle policies    │
│  └── S3 Storage Lens: Identify optimization opportunities │
│                                                            │
│  DATABASE                                                  │
│  ├── Aurora Serverless v2: Scale to zero (near)          │
│  ├── DynamoDB on-demand: Unpredictable workloads         │
│  ├── RDS Reserved Instances: Predictable databases       │
│  └── Stop non-prod RDS: Automatic stop after 7 days     │
│                                                            │
│  NETWORK                                                   │
│  ├── VPC endpoints: Avoid NAT Gateway data processing    │
│  ├── CloudFront: Cache at edge, reduce origin traffic    │
│  ├── Same-AZ traffic: Keep compute + DB in same AZ      │
│  └── Data transfer: Compress data; minimize cross-region │
│                                                            │
│  OPERATIONAL                                               │
│  ├── Tag everything: Accountability per team/project     │
│  ├── Budget alarms: Alert before overspending            │
│  ├── Cost anomaly detection: ML-based alerts             │
│  └── Regular reviews: Monthly FinOps reviews             │
└────────────────────────────────────────────────────────────┘
```

---

## Non-Prod Cost Reduction

```bash
# Schedule Auto Scaling to scale down at night
aws autoscaling put-scheduled-action \
  --auto-scaling-group-name dev-asg \
  --scheduled-action-name ScaleDownNight \
  --recurrence "0 19 * * MON-FRI" \
  --desired-capacity 0 \
  --min-size 0

aws autoscaling put-scheduled-action \
  --auto-scaling-group-name dev-asg \
  --scheduled-action-name ScaleUpMorning \
  --recurrence "0 7 * * MON-FRI" \
  --desired-capacity 2 \
  --min-size 1

# Stop non-production RDS instance
aws rds stop-db-instance \
  --db-instance-identifier dev-database
# ⚠ Auto-restarts after 7 days — automate re-stop with Lambda

# Spot Instances for CI/CD (CodeBuild)
# In buildspec or project config:
# Set compute type to BUILD_GENERAL1_SMALL
# Enable Spot in project environment settings
```

---

## Cost Explorer Queries

```bash
# Get monthly costs by service (last 3 months)
aws ce get-cost-and-usage \
  --time-period Start=2024-01-01,End=2024-04-01 \
  --granularity MONTHLY \
  --metrics "BlendedCost" \
  --group-by Type=DIMENSION,Key=SERVICE

# Get daily costs for EC2
aws ce get-cost-and-usage \
  --time-period Start=2024-03-01,End=2024-04-01 \
  --granularity DAILY \
  --metrics "UnblendedCost" \
  --filter '{
    "Dimensions": {
      "Key": "SERVICE",
      "Values": ["Amazon Elastic Compute Cloud - Compute"]
    }
  }'

# Get costs by tag
aws ce get-cost-and-usage \
  --time-period Start=2024-03-01,End=2024-04-01 \
  --granularity MONTHLY \
  --metrics "BlendedCost" \
  --group-by Type=TAG,Key=Environment

# Get cost forecast
aws ce get-cost-forecast \
  --time-period Start=2024-04-01,End=2024-05-01 \
  --metric BLENDED_COST \
  --granularity MONTHLY
```

---

## Common Cost Waste Sources

```
┌──── Top Cost Waste Areas ────────────────────────────────┐
│                                                           │
│  Waste Type           How to Find          Fix           │
│  ──────────────────── ──────────────────── ────────────  │
│  Idle EC2 instances   CPU < 5% average     Right-size    │
│  Unattached EBS       Volumes: available   Delete or     │
│                       state                snapshot+del  │
│  Old EBS snapshots    No lifecycle policy  DLM policies  │
│  Unused Elastic IPs   Not attached to EC2  Release them  │
│  Over-provisioned RDS CPU < 10%            Downsize      │
│  NAT Gateway data     High data charges    VPC endpoints │
│  Cross-region xfer    Data transfer costs  Same-region   │
│  Idle load balancers  No healthy targets   Delete        │
│  Unused keys/secrets  No recent access     Audit + del   │
│  Lambda over-memory   Duration × memory    Right-size    │
│                                                           │
│  Tool: Trusted Advisor (cost optimization checks)       │
│  Tool: Compute Optimizer (EC2, EBS, Lambda, ECS)        │
└───────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Surprise high bill | Untagged or unmonitored resources | Set up Budgets + Cost Anomaly Detection immediately |
| Can't track costs by team | No cost allocation tags | Enforce tagging via SCPs + AWS Config |
| NAT Gateway costs high | Large data transfer through NAT | Use VPC endpoints for S3/DynamoDB; review architecture |
| Data transfer between AZs | Cross-AZ traffic charges | Co-locate related services in same AZ when possible |
| Savings Plans not applying | Wrong plan type or account scope | Verify plan covers the compute type and region |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **Cost Explorer** | Visualize spend by service, tag, account. Forecast future costs. |
| **Budgets** | Set spending thresholds. Alert at 80% actual, 100% forecast. |
| **Savings Plans** | Compute SP (flexible) or EC2 Instance SP (specific). Up to 72% off. |
| **Spot Instances** | Up to 90% savings. Use for CI/CD, batch, fault-tolerant workloads. |
| **Right-sizing** | Compute Optimizer recommendations. Review before committing. |
| **Tagging** | Foundation of cost allocation. Enforce with SCPs + Config rules. |

---

## Quick Revision Questions

1. **What is the difference between Compute Savings Plans and EC2 Instance Savings Plans?**
   > Compute Savings Plans apply across EC2, Fargate, and Lambda regardless of instance family, size, or Region. EC2 Instance Savings Plans are locked to a specific instance family and Region but offer slightly deeper discounts.

2. **How do you prevent surprise AWS bills?**
   > Set up AWS Budgets with actual and forecasted alerts, enable Cost Anomaly Detection, enforce tagging for visibility, and review Cost Explorer weekly/monthly.

3. **When should you use Spot Instances?**
   > For fault-tolerant, flexible workloads: CI/CD build agents, batch processing, data analysis, containerized microservices with multiple replicas, and test environments.

4. **What is the most common hidden cost on AWS?**
   > Data transfer costs — especially NAT Gateway data processing ($0.045/GB), cross-AZ traffic ($0.01/GB each way), and cross-region replication.

5. **How does Compute Optimizer help?**
   > It analyzes CloudWatch metrics (CPU, memory, network) for EC2, EBS, Lambda, and ECS. It recommends right-sized instance types, volume types, and memory settings to reduce costs or improve performance.

---

[← Previous: 11.1 Well-Architected Framework](01-well-architected-framework.md) | [Next: 11.3 Automation Strategies →](03-automation-strategies.md)

[← Back to README](../README.md)
