# Chapter 11.4: Disaster Recovery on AWS

## Overview

Disaster Recovery (DR) ensures business continuity when primary systems fail due to natural disasters, hardware failures, or human errors. AWS provides multiple DR strategies with varying **Recovery Time Objectives (RTO)** and **Recovery Point Objectives (RPO)**. The strategy you choose depends on the acceptable downtime and data loss for your application.

---

## Key Metrics

```
┌──── RTO and RPO ─────────────────────────────────────────┐
│                                                           │
│  ◄──── RPO ────►             ◄──── RTO ────►             │
│  (data loss)                 (downtime)                  │
│                                                           │
│  ────────────────┬───────────┬───────────────────►       │
│  Last backup     │ DISASTER  │ Recovery          Time    │
│                  │  EVENT    │ Complete                   │
│                  │           │                            │
│  RPO = How much data can you afford to lose?            │
│  RTO = How long can you afford to be down?              │
│                                                           │
│  Example:                                                │
│  RPO = 1 hour → backups every hour minimum              │
│  RTO = 4 hours → must be operational within 4 hours     │
└───────────────────────────────────────────────────────────┘
```

---

## DR Strategies Comparison

```
┌──── DR Strategies (Cost vs Speed) ───────────────────────┐
│                                                           │
│  Cost ▲   RTO/RPO                                        │
│       │                                                   │
│   $$$$│                        ┌─────────────────────┐   │
│       │                        │  Multi-Site Active  │   │
│       │                        │  RTO: ~0  RPO: ~0   │   │
│       │                        └─────────────────────┘   │
│   $$$ │              ┌─────────────────────┐             │
│       │              │  Warm Standby       │             │
│       │              │  RTO: mins RPO: secs│             │
│       │              └─────────────────────┘             │
│    $$ │    ┌─────────────────────┐                       │
│       │    │  Pilot Light        │                       │
│       │    │  RTO: 10min RPO: min│                       │
│       │    └─────────────────────┘                       │
│     $ │  ┌─────────────────────┐                        │
│       │  │  Backup & Restore   │                        │
│       │  │  RTO: hrs RPO: hrs  │                        │
│       │  └─────────────────────┘                        │
│       └──────────────────────────────────────────►      │
│                                    Speed of Recovery     │
└──────────────────────────────────────────────────────────┘
```

---

## Strategy 1: Backup & Restore

```
┌──── Backup & Restore ────────────────────────────────────┐
│                                                           │
│  Primary Region (us-east-1)     DR Region (us-west-2)   │
│  ┌──────────────────────┐                                │
│  │  EC2 + EBS           │───── EBS Snapshots ──────►    │
│  │  RDS Database        │───── Automated Backups ──►    │
│  │  S3 Data             │───── Cross-Region Repl ──►    │
│  │  DynamoDB            │───── On-Demand Backup ───►    │
│  └──────────────────────┘                                │
│                                                           │
│  On Disaster:                     ┌──────────────────┐  │
│  1. Restore from backups ───────► │ Launch new infra │  │
│  2. Update DNS (Route 53)         │ from backups     │  │
│  3. Validate & go live            │ in DR region     │  │
│                                    └──────────────────┘  │
│                                                           │
│  RTO: Hours    RPO: Hours    Cost: $ (lowest)           │
│  Best for: Non-critical workloads, dev environments     │
└───────────────────────────────────────────────────────────┘
```

```bash
# Cross-region EBS snapshot copy
aws ec2 copy-snapshot \
  --source-region us-east-1 \
  --source-snapshot-id snap-abc123 \
  --destination-region us-west-2 \
  --description "DR copy"

# RDS cross-region read replica (can promote)
aws rds create-db-instance-read-replica \
  --db-instance-identifier dr-replica \
  --source-db-instance-identifier arn:aws:rds:us-east-1:123456789012:db:primary-db \
  --region us-west-2

# S3 cross-region replication
aws s3api put-bucket-replication \
  --bucket my-primary-bucket \
  --replication-configuration file://replication.json

# AWS Backup — create backup plan
aws backup create-backup-plan \
  --backup-plan '{
    "BackupPlanName": "DailyBackups",
    "Rules": [{
      "RuleName": "DailyRule",
      "TargetBackupVaultName": "Default",
      "ScheduleExpression": "cron(0 2 * * ? *)",
      "StartWindowMinutes": 60,
      "CompletionWindowMinutes": 180,
      "Lifecycle": {"DeleteAfterDays": 30},
      "CopyActions": [{
        "DestinationBackupVaultArn": "arn:aws:backup:us-west-2:123456789012:backup-vault:DR-Vault",
        "Lifecycle": {"DeleteAfterDays": 30}
      }]
    }]
  }'
```

---

## Strategy 2: Pilot Light

```
┌──── Pilot Light ─────────────────────────────────────────┐
│                                                           │
│  Primary (us-east-1)            DR (us-west-2)          │
│  ┌──────────────────────┐      ┌──────────────────┐     │
│  │  ALB                 │      │  (no ALB yet)     │     │
│  │  ┌────┐ ┌────┐      │      │                    │     │
│  │  │EC2 │ │EC2 │      │      │  (no EC2 yet)     │     │
│  │  └────┘ └────┘      │      │                    │     │
│  │  ┌────────────────┐  │      │  ┌──────────────┐ │     │
│  │  │  RDS Primary   │──┼──────┼─►│ RDS Replica  │ │     │
│  │  └────────────────┘  │      │  │ (always on)  │ │     │
│  └──────────────────────┘      │  └──────────────┘ │     │
│                                 └──────────────────┘     │
│  On Disaster:                                            │
│  1. Promote RDS replica to primary                      │
│  2. Launch EC2 from AMIs (pre-baked)                    │
│  3. Create ALB, attach instances                        │
│  4. Scale up to production capacity                     │
│  5. Update Route 53 DNS                                 │
│                                                           │
│  "Pilot light" = core DB always running, compute off    │
│  RTO: 10-30 min    RPO: Minutes    Cost: $$ (low)      │
│  Best for: Important workloads with moderate RTO        │
└───────────────────────────────────────────────────────────┘
```

---

## Strategy 3: Warm Standby

```
┌──── Warm Standby ────────────────────────────────────────┐
│                                                           │
│  Primary (us-east-1)            DR (us-west-2)          │
│  ┌──────────────────────┐      ┌──────────────────┐     │
│  │  ALB                 │      │  ALB              │     │
│  │  ┌────┐ ┌────┐ ┌────┐│      │  ┌────┐          │     │
│  │  │EC2 │ │EC2 │ │EC2 ││      │  │EC2 │ (min)    │     │
│  │  └────┘ └────┘ └────┘│      │  └────┘          │     │
│  │  ASG: min=3, max=6   │      │  ASG: min=1,max=6│     │
│  │  ┌────────────────┐  │      │  ┌──────────────┐ │     │
│  │  │  RDS Primary   │──┼──────┼─►│ RDS Replica  │ │     │
│  │  └────────────────┘  │      │  └──────────────┘ │     │
│  └──────────────────────┘      └──────────────────┘     │
│                                                           │
│  On Disaster:                                            │
│  1. Scale up ASG (min=3, desired=3)                     │
│  2. Promote RDS replica                                 │
│  3. Update Route 53 (failover routing)                  │
│                                                           │
│  "Warm" = reduced-scale copy always running             │
│  RTO: Minutes    RPO: Seconds    Cost: $$$ (moderate)   │
│  Best for: Business-critical workloads                  │
└───────────────────────────────────────────────────────────┘
```

---

## Strategy 4: Multi-Site Active-Active

```
┌──── Multi-Site Active-Active ────────────────────────────┐
│                                                           │
│  Region A (us-east-1)           Region B (us-west-2)    │
│  ┌──────────────────────┐      ┌──────────────────┐     │
│  │  ALB                 │      │  ALB              │     │
│  │  ┌────┐ ┌────┐ ┌────┐│      │  ┌────┐ ┌────┐ ┌────┐│ │
│  │  │EC2 │ │EC2 │ │EC2 ││      │  │EC2 │ │EC2 │ │EC2 ││ │
│  │  └────┘ └────┘ └────┘│      │  └────┘ └────┘ └────┘│ │
│  │  ASG: Full capacity  │      │  ASG: Full capacity│   │
│  │  ┌────────────────┐  │      │  ┌──────────────┐ │   │
│  │  │  DynamoDB       │◄─┼──────┼─►│  DynamoDB    │ │   │
│  │  │  Global Table   │  │      │  │  Global Tbl  │ │   │
│  │  └────────────────┘  │      │  └──────────────┘ │   │
│  └──────────────────────┘      └──────────────────┘    │
│                                                          │
│         ┌──────────────────────────┐                    │
│         │  Route 53                │                    │
│         │  Latency-based routing   │                    │
│         │  with health checks      │                    │
│         └──────────────────────────┘                    │
│                                                          │
│  On Disaster: Route 53 auto-fails over                  │
│  No manual intervention needed                          │
│                                                          │
│  RTO: ~0    RPO: ~0    Cost: $$$$ (highest)            │
│  Best for: Mission-critical, zero-downtime requirement  │
└──────────────────────────────────────────────────────────┘
```

---

## Route 53 Failover

```bash
# Create health check for primary
aws route53 create-health-check \
  --caller-reference "primary-hc-$(date +%s)" \
  --health-check-config '{
    "Type": "HTTPS",
    "FullyQualifiedDomainName": "primary.example.com",
    "Port": 443,
    "RequestInterval": 10,
    "FailureThreshold": 3
  }'

# Primary record (failover routing)
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234 \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "app.example.com",
        "Type": "A",
        "SetIdentifier": "primary",
        "Failover": "PRIMARY",
        "HealthCheckId": "hc-abc123",
        "AliasTarget": {
          "HostedZoneId": "Z35SXDOTRQ7X7K",
          "DNSName": "primary-alb.us-east-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }]
  }'

# Secondary record (failover routing)
aws route53 change-resource-record-sets \
  --hosted-zone-id Z1234 \
  --change-batch '{
    "Changes": [{
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "app.example.com",
        "Type": "A",
        "SetIdentifier": "secondary",
        "Failover": "SECONDARY",
        "AliasTarget": {
          "HostedZoneId": "Z1H1FL5HABSF5",
          "DNSName": "dr-alb.us-west-2.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }]
  }'
```

---

## DR Strategy Comparison

| Strategy | RTO | RPO | Cost | Complexity | Use Case |
|----------|-----|-----|------|------------|----------|
| Backup & Restore | Hours | Hours | $ | Low | Dev, non-critical |
| Pilot Light | 10-30 min | Minutes | $$ | Medium | Important apps |
| Warm Standby | Minutes | Seconds | $$$ | Medium-High | Business-critical |
| Multi-Site Active | Near zero | Near zero | $$$$ | High | Mission-critical |

---

## AWS Backup

```bash
# Create backup vault in DR region
aws backup create-backup-vault \
  --backup-vault-name DR-Vault \
  --region us-west-2

# Assign resources to backup plan
aws backup create-backup-selection \
  --backup-plan-id <plan-id> \
  --backup-selection '{
    "SelectionName": "ProductionResources",
    "IamRoleArn": "arn:aws:iam::123456789012:role/AWSBackupRole",
    "Resources": [
      "arn:aws:ec2:us-east-1:123456789012:volume/*",
      "arn:aws:rds:us-east-1:123456789012:db:*"
    ],
    "Conditions": {
      "StringEquals": [
        {"ConditionKey": "aws:ResourceTag/Environment", "ConditionValue": "production"}
      ]
    }
  }'
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| RDS failover takes too long | Single-AZ instance, no replica | Use Multi-AZ or pre-created cross-region replica |
| Route 53 failover not triggering | Health check misconfigured | Verify health check endpoint, threshold, and interval |
| DR region missing resources | Infrastructure not pre-provisioned | Use CloudFormation StackSets for multi-region IaC |
| Data inconsistency after failover | Async replication lag | Accept RPO trade-off or use synchronous replication ($$$$) |
| DR plan never tested | "We'll test it later" | Schedule quarterly DR drills; use AWS FIS for chaos testing |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| **RPO** | Maximum acceptable data loss (time). Drives backup frequency. |
| **RTO** | Maximum acceptable downtime. Drives DR strategy choice. |
| **Backup & Restore** | Cheapest. Hours to recover. Backups in DR region. |
| **Pilot Light** | Core DB running. Launch compute on failover. Minutes RTO. |
| **Warm Standby** | Scaled-down clone running. Scale up on failover. |
| **Multi-Site Active** | Both regions active. Route 53 auto-failover. Near-zero RTO/RPO. |
| **AWS Backup** | Centralized backup across services. Cross-region copy. |

---

## Quick Revision Questions

1. **What is the difference between RPO and RTO?**
   > RPO (Recovery Point Objective) is the maximum acceptable data loss measured in time. RTO (Recovery Time Objective) is the maximum acceptable downtime before the system must be operational again.

2. **Which DR strategy has the lowest cost but highest RTO?**
   > Backup & Restore. It has the lowest ongoing cost since no infrastructure runs in the DR region, but recovery takes hours because everything must be restored from backups.

3. **What is a Pilot Light DR strategy?**
   > The core database (RDS replica) is always running in the DR region, but compute resources (EC2, containers) are not provisioned. On failover, you launch compute from pre-baked AMIs and promote the database replica.

4. **How does Route 53 enable automatic failover?**
   > Using failover routing policy with health checks. Route 53 monitors the primary endpoint and automatically routes traffic to the secondary (DR) endpoint when the health check fails.

5. **How often should you test your DR plan?**
   > At minimum quarterly. Use AWS Fault Injection Simulator (FIS) for controlled chaos experiments and schedule full DR drills to validate runbooks, measure actual RTO/RPO, and train the team.

---

[← Previous: 11.3 Automation Strategies](03-automation-strategies.md) | [Next: 11.5 Multi-Account Strategy →](05-multi-account-strategy.md)

[← Back to README](../README.md)
