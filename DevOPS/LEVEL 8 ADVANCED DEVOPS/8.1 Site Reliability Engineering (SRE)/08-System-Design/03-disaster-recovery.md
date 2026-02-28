# Chapter 8.3: Disaster Recovery

[← Previous: High Availability Patterns](02-high-availability-patterns.md) | [Back to README](../README.md) | [Next: Data Integrity →](04-data-integrity.md)

---

## Overview

Disaster Recovery (DR) is the set of policies, tools, and procedures to recover technology infrastructure and systems after a catastrophic event. While High Availability prevents minor outages, DR handles worst-case scenarios: region-wide failures, data center loss, data corruption, ransomware attacks, or natural disasters. Every SRE team must have a tested DR plan.

---

## 1. Key DR Metrics

```
  ┌──────────────────────────────────────────────────────────┐
  │  RPO vs RTO                                              │
  │                                                          │
  │  Time ──────────────────────────────────────────────▶    │
  │                                                          │
  │       Last backup    Disaster    Recovery complete        │
  │           │              │              │                 │
  │           ▼              ▼              ▼                 │
  │  ─────────●──────────────●──────────────●────────────    │
  │           │              │              │                 │
  │           │◄────RPO─────▶│◄────RTO─────▶│                │
  │           │              │              │                 │
  │                                                          │
  │  RPO (Recovery Point Objective):                         │
  │  └─ Maximum acceptable data loss (measured in time)     │
  │     "How much data can we afford to lose?"              │
  │     RPO = 1 hour → you might lose up to 1 hour of data │
  │                                                          │
  │  RTO (Recovery Time Objective):                          │
  │  └─ Maximum acceptable downtime (time to restore)       │
  │     "How long can we be down?"                          │
  │     RTO = 4 hours → must be operational within 4 hours  │
  │                                                          │
  │  ┌──────────────┬──────────┬──────────┬──────────────┐  │
  │  │ Tier         │ RPO      │ RTO      │ Cost         │  │
  │  ├──────────────┼──────────┼──────────┼──────────────┤  │
  │  │ Mission-crit │ ~0 (sync)│ < 15 min │ $$$$         │  │
  │  │ Business-crit│ < 1 hour │ < 4 hour │ $$$          │  │
  │  │ Important    │ < 24 hrs │ < 24 hrs │ $$           │  │
  │  │ Non-critical │ < 7 days │ < 72 hrs │ $            │  │
  │  └──────────────┴──────────┴──────────┴──────────────┘  │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. DR Strategies

```
  ┌──────────────────────────────────────────────────────────┐
  │  DR STRATEGIES (by cost and recovery speed)              │
  │                                                          │
  │  Cost ▲                                                  │
  │       │                                                  │
  │  $$$$ │                    ┌──────────────┐              │
  │       │                    │ MULTI-SITE   │              │
  │       │                    │ ACTIVE-ACTIVE│              │
  │  $$$  │           ┌───────┴──────────────┘              │
  │       │           │ HOT STANDBY                         │
  │       │           │ (always running, data sync)         │
  │  $$   │    ┌──────┴──────────────────────┐              │
  │       │    │ WARM STANDBY                │              │
  │       │    │ (infra provisioned, scaled  │              │
  │       │    │  down, periodic data sync)  │              │
  │  $    │ ┌──┴──────────────────────────┐  │              │
  │       │ │ PILOT LIGHT                 │  │              │
  │       │ │ (minimal infra, DB replica) │  │              │
  │       │ └──┬──────────────────────────┘  │              │
  │       │ ┌──┴──────────────────────────┐  │              │
  │       │ │ BACKUP & RESTORE            │  │              │
  │       │ │ (cheapest, slowest recovery)│  │              │
  │       └─┴─────────────────────────────┴──┴──────────▶   │
  │         Hours          Minutes           Seconds  RTO   │
  └──────────────────────────────────────────────────────────┘
```

| Strategy | RTO | RPO | Cost | Description |
|----------|-----|-----|------|-------------|
| **Backup & Restore** | Hours | Hours | $ | Regular backups to remote storage. Rebuild from scratch during DR. |
| **Pilot Light** | 30-60 min | Minutes | $$ | Core database replicated. Scale up compute on demand. |
| **Warm Standby** | 10-30 min | Seconds-Minutes | $$$ | Scaled-down copy of production running. Scale up during DR. |
| **Hot Standby** | 1-5 min | Seconds | $$$$ | Full copy of production, synchronized. Switch traffic. |
| **Multi-Site Active** | ~0 | ~0 | $$$$$ | Both sites serve traffic. No failover needed. |

---

## 3. Backup Strategies

```
  ┌──────────────────────────────────────────────────────────┐
  │  BACKUP TYPES                                            │
  │                                                          │
  │  FULL BACKUP:                                            │
  │  Day 1: [████████████████] 100GB  ← Complete copy       │
  │  Day 2: [████████████████] 100GB  ← Complete copy       │
  │  Day 3: [████████████████] 100GB  ← Complete copy       │
  │  Storage: 300GB   Restore: Fast (single backup)         │
  │                                                          │
  │  INCREMENTAL BACKUP:                                     │
  │  Day 1: [████████████████] 100GB  ← Full backup         │
  │  Day 2: [██]               5GB   ← Changes since Day 1 │
  │  Day 3: [███]              8GB   ← Changes since Day 2 │
  │  Storage: 113GB   Restore: Slow (chain required)        │
  │                                                          │
  │  DIFFERENTIAL BACKUP:                                    │
  │  Day 1: [████████████████] 100GB  ← Full backup         │
  │  Day 2: [██]               5GB   ← Changes since Day 1 │
  │  Day 3: [█████]           12GB   ← Changes since Day 1 │
  │  Storage: 117GB   Restore: Medium (full + latest diff)  │
  │                                                          │
  │  CONTINUOUS REPLICATION (CDP):                           │
  │  ████████████████████████████████▶  Real-time stream    │
  │  RPO: Near-zero   Cost: Highest   Most complex         │
  └──────────────────────────────────────────────────────────┘
```

---

## 4. DR Plan Template

```yaml
# Disaster Recovery Plan
disaster_recovery_plan:
  document_info:
    version: "3.2"
    last_updated: "2024-03-01"
    last_tested: "2024-02-15"
    next_test: "2024-05-15"
    owner: "SRE Team"
  
  classification:
    tier_1_mission_critical:
      services:
        - name: "Payment Processing"
          rpo: "0 seconds (synchronous replication)"
          rto: "5 minutes"
          strategy: "hot_standby"
          dr_region: "eu-west-1"
          
        - name: "User Authentication"
          rpo: "0 seconds"
          rto: "5 minutes"
          strategy: "hot_standby"
          dr_region: "eu-west-1"
    
    tier_2_business_critical:
      services:
        - name: "Product Catalog"
          rpo: "5 minutes"
          rto: "30 minutes"
          strategy: "warm_standby"
          dr_region: "eu-west-1"
          
        - name: "Order Management"
          rpo: "1 minute"
          rto: "15 minutes"
          strategy: "warm_standby"
          dr_region: "eu-west-1"
    
    tier_3_important:
      services:
        - name: "Analytics Dashboard"
          rpo: "1 hour"
          rto: "4 hours"
          strategy: "pilot_light"
          
        - name: "Email Notifications"
          rpo: "4 hours"
          rto: "8 hours"
          strategy: "backup_restore"

  communication_plan:
    internal:
      - channel: "slack:#incident-response"
        when: "immediately"
      - channel: "pagerduty:all-hands"
        when: "DR declared"
    external:
      - channel: "status_page"
        when: "within 5 minutes"
        template: "dr-notification-template"
      - channel: "customer_email"
        when: "within 30 minutes"
        template: "dr-customer-update"
  
  roles:
    dr_commander: "VP Engineering (primary) / CTO (backup)"
    technical_lead: "SRE Manager"
    communications: "Customer Success Lead"
    business_liaison: "Product VP"
```

---

## 5. DR Testing

```
  ┌──────────────────────────────────────────────────────────┐
  │  DR TESTING MATURITY LEVELS                              │
  │                                                          │
  │  Level 1: TABLETOP EXERCISE (quarterly)                  │
  │  ├─ Walk through DR plan on paper                       │
  │  ├─ Identify gaps in documentation                      │
  │  ├─ Verify contact lists are current                    │
  │  └─ Time: 2-4 hours   Risk: None                       │
  │                                                          │
  │  Level 2: COMPONENT TEST (monthly)                       │
  │  ├─ Test individual components (backup restore, etc.)   │
  │  ├─ Verify backup integrity (restore to test env)       │
  │  ├─ Test DNS failover in isolation                      │
  │  └─ Time: 4-8 hours   Risk: Minimal                    │
  │                                                          │
  │  Level 3: PARTIAL FAILOVER (quarterly)                   │
  │  ├─ Fail over one service to DR region                  │
  │  ├─ Verify data consistency post-failover               │
  │  ├─ Test with real traffic (small %)                    │
  │  └─ Time: 1 day   Risk: Moderate                       │
  │                                                          │
  │  Level 4: FULL DR DRILL (annually)                       │
  │  ├─ Simulate complete primary region failure            │
  │  ├─ All services fail over to DR region                 │
  │  ├─ Run production traffic from DR for 4+ hours         │
  │  ├─ Measure actual RTO and RPO                          │
  │  └─ Time: 1-2 days   Risk: Significant (plan carefully)│
  │                                                          │
  │  RULE: An untested DR plan is not a DR plan.            │
  │  Test regularly. Document results. Fix gaps.            │
  └──────────────────────────────────────────────────────────┘
```

---

## 6. Cloud-Native DR with Terraform

```hcl
# DR infrastructure provisioning (Pilot Light example)

# Primary region resources
provider "aws" {
  alias  = "primary"
  region = "us-east-1"
}

# DR region resources
provider "aws" {
  alias  = "dr"
  region = "eu-west-1"
}

# Cross-region RDS replication
resource "aws_db_instance" "primary" {
  provider               = aws.primary
  identifier             = "app-db-primary"
  engine                 = "postgres"
  engine_version         = "15.4"
  instance_class         = "db.r6g.xlarge"
  backup_retention_period = 7
  multi_az               = true  # HA within region
  
  tags = {
    DR_Role = "primary"
    DR_Tier = "mission-critical"
  }
}

resource "aws_db_instance" "dr_replica" {
  provider             = aws.dr
  identifier           = "app-db-dr-replica"
  replicate_source_db  = aws_db_instance.primary.arn
  instance_class       = "db.r6g.large"  # Smaller in DR (scale up during failover)
  
  tags = {
    DR_Role = "replica"
    DR_Tier = "mission-critical"
  }
}

# DR EKS cluster (minimal, scale up during DR)
resource "aws_eks_cluster" "dr" {
  provider = aws.dr
  name     = "app-cluster-dr"
  role_arn = aws_iam_role.eks_dr.arn
  version  = "1.28"
  
  vpc_config {
    subnet_ids = aws_subnet.dr_private[*].id
  }
}

# DR node group (minimal size, auto-scale during failover)
resource "aws_eks_node_group" "dr" {
  provider        = aws.dr
  cluster_name    = aws_eks_cluster.dr.name
  node_group_name = "dr-nodes"
  
  scaling_config {
    desired_size = 2    # Minimal (pilot light)
    min_size     = 2
    max_size     = 50   # Scale up during DR event
  }
  
  instance_types = ["m6i.xlarge"]
}

# S3 cross-region replication for backups
resource "aws_s3_bucket" "backups_dr" {
  provider = aws.dr
  bucket   = "app-backups-dr-eu-west-1"
}

resource "aws_s3_bucket_replication_configuration" "backup_replication" {
  provider = aws.primary
  bucket   = aws_s3_bucket.backups_primary.id
  role     = aws_iam_role.replication.arn

  rule {
    id     = "replicate-all"
    status = "Enabled"
    
    destination {
      bucket        = aws_s3_bucket.backups_dr.arn
      storage_class = "STANDARD_IA"
    }
  }
}
```

---

## 7. Real-World Scenario

### [SCENARIO] Region-Wide Cloud Outage Recovery

```
  INCIDENT: AWS US-East-1 experiences a major outage
  affecting EC2, RDS, and ELB services for 6+ hours.
  
  Company: SaaS platform, 200K active users, $50M ARR
  DR Strategy: Warm standby in EU-West-1
  
  ┌──────────────────────────────────────────────────────────┐
  │  TIMELINE                                                │
  │                                                          │
  │  09:00 — AWS status dashboard shows US-East-1 issues    │
  │  09:05 — Monitoring alerts: API error rate >50%         │
  │  09:08 — IC confirms: regional outage, not our code     │
  │  09:10 — DR COMMANDER DECISION: Activate DR plan        │
  │                                                          │
  │  DR ACTIVATION:                                          │
  │  09:12 — Scale up EU-West-1 EKS nodes (2 → 30)         │
  │  09:15 — Promote RDS read replica to primary            │
  │  09:18 — Deploy latest app version to DR cluster        │
  │  09:22 — Smoke tests pass in DR region                  │
  │  09:25 — Update Route53: DNS failover to EU-West-1     │
  │  09:28 — Status page: "Operating from backup region"    │
  │  09:30 — 80% of traffic now serving from EU-West-1     │
  │  09:35 — DNS propagation complete, 100% on DR           │
  │                                                          │
  │  RTO ACHIEVED: 25 minutes (target was 30 minutes) ✓     │
  │  RPO ACHIEVED: ~30 seconds of data (async repl lag) ✓   │
  │                                                          │
  │  FAILBACK (after AWS recovery):                         │
  │  16:00 — AWS US-East-1 recovered                        │
  │  16:30 — Begin data reconciliation (30s of lost writes) │
  │  17:00 — Set up reverse replication EU → US             │
  │  18:00 — Gradual traffic shift back to US-East-1       │
  │  20:00 — 100% back on US-East-1                        │
  │  20:30 — DR region scaled back down to warm standby     │
  │                                                          │
  │  LESSONS LEARNED:                                        │
  │  ├─ DR plan worked because of quarterly testing!        │
  │  ├─ 30s data loss acceptable (RPO met)                  │
  │  ├─ Some third-party integrations failed (US webhooks)  │
  │  └─ Failback took longer than expected (data sync)      │
  └──────────────────────────────────────────────────────────┘
```

---

## 8. Troubleshooting Tips

| Issue | Solution |
|-------|----------|
| "DR region is out of capacity when we need it" | Reserve capacity in DR region (RI or dedicated hosts). Test scaling regularly. Have multiple fallback regions. |
| "Data is inconsistent after failover" | Accept async replication lag. Implement conflict resolution. Log all writes during lag window for manual reconciliation. |
| "DNS failover is too slow (TTL)" | Use low TTL on DNS records (60s). Use health-checked routing (Route53). Consider anycast or client-side failover. |
| "DR plan is outdated (drift)" | Automate DR infra with Terraform. Run quarterly tests. Flag DR plan review in deploy checklist for architecture changes. |
| "Failback is harder than failover" | Plan failback as part of DR process. Test failback during drills. Implement bi-directional replication from the start. |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **RPO** | Maximum acceptable data loss (time-based) |
| **RTO** | Maximum acceptable downtime to recovery |
| **Backup & Restore** | Cheapest DR: regular backups, rebuild on demand (hours RTO) |
| **Pilot Light** | Core DB replicated, compute on-demand (30-60 min RTO) |
| **Warm Standby** | Scaled-down copy running (10-30 min RTO) |
| **Hot Standby** | Full replica, synchronized (1-5 min RTO) |
| **Multi-Site Active** | Both sites serve traffic, zero RTO |
| **DR Testing** | Untested DR plans fail. Test quarterly minimum. |

---

## Quick Revision Questions

1. **What is the difference between RPO and RTO?**
   > RPO (Recovery Point Objective) is the maximum acceptable data loss, measured as the time between the last backup/replication and the disaster. RTO (Recovery Time Objective) is the maximum acceptable downtime, measured from disaster to full recovery.

2. **What are the five DR strategies, from cheapest to most expensive?**
   > (1) Backup & Restore — rebuild from backups. (2) Pilot Light — core DB replicated, minimal infra. (3) Warm Standby — scaled-down production copy. (4) Hot Standby — full production replica. (5) Multi-Site Active-Active — both sites serve traffic simultaneously.

3. **Why is DR testing critical?**
   > An untested DR plan is effectively no plan at all. Testing reveals gaps in procedures, outdated documentation, capacity issues, and integration problems that only surface during actual failover. Teams that test quarterly recover significantly faster during real disasters.

4. **What is the difference between incremental and differential backups?**
   > Incremental backs up only changes since the last backup (any type). Differential backs up all changes since the last full backup. Incremental uses less storage but requires the full chain to restore. Differential uses more storage but only needs the full + latest differential.

5. **What makes failback often harder than failover?**
   > During failover, you switch to a known-good standby. During failback, the original region may have stale data, inconsistent state, or require data reconciliation from the DR region. Bi-directional replication and data conflict resolution make failback complex.

6. **How should services be classified for DR planning?**
   > By business criticality: Mission-critical (RPO ~0, RTO <15 min), Business-critical (RPO <1 hr, RTO <4 hr), Important (RPO <24 hr, RTO <24 hr), Non-critical (RPO <7 days, RTO <72 hr). Each tier gets an appropriate (and proportionally costly) DR strategy.

---

[← Previous: High Availability Patterns](02-high-availability-patterns.md) | [Back to README](../README.md) | [Next: Data Integrity →](04-data-integrity.md)
