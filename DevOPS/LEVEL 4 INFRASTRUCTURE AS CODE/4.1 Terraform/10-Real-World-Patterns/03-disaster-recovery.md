# Chapter 3: Disaster Recovery

[← Previous: Multi-Region Deployment](02-multi-region-deployment.md) | [Back to README](../README.md) | [Next: Cost Optimization →](04-cost-optimization.md)

---

## Overview

Disaster Recovery (DR) ensures infrastructure can recover from failures. Terraform enables **reproducible**, **automated** recovery by defining infrastructure as code. This chapter covers DR strategies, RTO/RPO targets, and Terraform implementation patterns.

---

## 3.1 DR Strategies

```
┌──────────────────────────────────────────────────────────┐
│           DISASTER RECOVERY STRATEGIES                    │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Cost ←──────────────────────────────────→ Cost           │
│  Low                                       High           │
│                                                           │
│  RTO ←───────────────────────────────────→ RTO            │
│  Hours                                     Minutes        │
│                                                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
│  │ Backup & │  │ Pilot    │  │  Warm    │  │ Multi-   │ │
│  │ Restore  │  │ Light    │  │ Standby  │  │ Site     │ │
│  │          │  │          │  │          │  │ Active   │ │
│  ├──────────┤  ├──────────┤  ├──────────┤  ├──────────┤ │
│  │ RTO: hrs │  │ RTO: min │  │ RTO: min │  │ RTO: sec │ │
│  │ RPO: hrs │  │ RPO: min │  │ RPO: sec │  │ RPO: ~0  │ │
│  │ Cost: $  │  │ Cost: $$ │  │ Cost: $$$│  │Cost: $$$$│ │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘ │
│                                                           │
│  RTO = Recovery Time Objective (max downtime)             │
│  RPO = Recovery Point Objective (max data loss)           │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 Backup and Restore

```hcl
# ─── Strategy 1: Backup & Restore ────────────────
# Primary region only. Restore from backups in DR region.
# RTO: hours | RPO: last backup interval

# Automated backups
resource "aws_db_instance" "primary" {
  identifier                = "${var.project}-db"
  engine                    = "postgres"
  instance_class            = "db.r6g.xlarge"
  backup_retention_period   = 35     # Days
  backup_window             = "03:00-04:00"
  copy_tags_to_snapshot     = true
  delete_automated_backups  = false

  tags = { Name = "${var.project}-db-primary" }
}

# Cross-region backup copy
resource "aws_db_instance_automated_backups_replication" "dr" {
  provider                 = aws.dr_region
  source_db_instance_arn   = aws_db_instance.primary.arn
  retention_period         = 14
}

# S3 cross-region replication for data
resource "aws_s3_bucket_replication_configuration" "dr" {
  bucket = aws_s3_bucket.primary.id
  role   = aws_iam_role.replication.arn

  rule {
    id     = "dr-replication"
    status = "Enabled"

    destination {
      bucket        = aws_s3_bucket.dr.arn
      storage_class = "GLACIER_IR"   # Cost-effective DR tier
    }
  }
}

# DR runbook: Apply infrastructure in DR region from IaC
# terraform -chdir=environments/dr init
# terraform -chdir=environments/dr apply -var="restore_from_backup=true"
```

---

## 3.3 Pilot Light

```
┌──────────────────────────────────────────────────────────┐
│       PILOT LIGHT ARCHITECTURE                            │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Primary (us-east-1)          DR (eu-west-1)              │
│  ┌─────────────────┐         ┌─────────────────┐         │
│  │  ALB (active)   │         │  ALB (OFF)      │         │
│  │  ┌───────────┐  │         │  (no instances) │         │
│  │  │ ECS (3)   │  │         │                 │         │
│  │  └───────────┘  │         │                 │         │
│  │  ┌───────────┐  │  Repl.  │  ┌───────────┐ │         │
│  │  │ RDS       │──┼────────►├──│ RDS       │ │         │
│  │  │ Primary   │  │         │  │ Replica   │ │         │
│  │  └───────────┘  │         │  └───────────┘ │         │
│  └─────────────────┘         └─────────────────┘         │
│                                                           │
│  On failover:                                             │
│  1. Promote RDS replica to primary                        │
│  2. terraform apply (scales up compute)                   │
│  3. Update DNS                                            │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── Strategy 2: Pilot Light ────────────────────
# Core data services always running, compute scaled to zero
# RTO: 10-30 min | RPO: seconds (async replication)

variable "dr_active" {
  type        = bool
  description = "Set to true to activate DR region"
  default     = false
}

# DR region: RDS replica always running
resource "aws_db_instance" "dr_replica" {
  provider            = aws.dr_region
  identifier          = "${var.project}-db-dr"
  replicate_source_db = aws_db_instance.primary.arn
  instance_class      = "db.r6g.large"   # Smaller in standby
}

# DR region: Compute scaled down until needed
resource "aws_ecs_service" "dr_app" {
  provider        = aws.dr_region
  name            = "${var.project}-app-dr"
  cluster         = aws_ecs_cluster.dr.id
  task_definition = aws_ecs_task_definition.app.arn
  desired_count   = var.dr_active ? var.app_count : 0   # 0 until activated

  load_balancer {
    target_group_arn = aws_lb_target_group.dr.arn
    container_name   = "app"
    container_port   = 8080
  }
}

# Failover procedure:
# 1. Set dr_active = true
# 2. terraform apply
# 3. Promote RDS: aws rds promote-read-replica ...
# 4. Update Route 53 to DR region
```

---

## 3.4 Warm Standby

```hcl
# ─── Strategy 3: Warm Standby ───────────────────
# Scaled-down copy always running in DR region
# RTO: minutes | RPO: seconds

module "dr_region" {
  source = "./modules/regional-stack"
  providers = {
    aws = aws.dr_region
  }

  environment    = var.environment
  region         = "eu-west-1"
  is_primary     = false

  # Scaled down versions
  instance_type  = "t3.medium"      # vs m5.xlarge in primary
  min_instances  = 1                 # vs 3 in primary
  max_instances  = var.dr_active ? 10 : 2
  db_class       = "db.r6g.large"   # vs db.r6g.2xlarge
}

# Auto-scaling policy activates on failover
resource "aws_autoscaling_policy" "dr_scale_up" {
  provider               = aws.dr_region
  name                   = "dr-scale-up"
  autoscaling_group_name = module.dr_region.asg_name
  policy_type            = "TargetTrackingScaling"

  target_tracking_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ASGAverageCPUUtilization"
    }
    target_value = 60.0
  }
}
```

---

## 3.5 Terraform State DR

```
┌──────────────────────────────────────────────────────────┐
│       STATE FILE DISASTER RECOVERY                        │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  The state file IS your infrastructure's source of truth  │
│  Losing it means losing track of managed resources        │
│                                                           │
│  Protection measures:                                     │
│  ┌────────────────────────────────────────────┐          │
│  │ 1. S3 versioning (point-in-time recovery)  │          │
│  │ 2. Cross-region replication                 │          │
│  │ 3. DynamoDB backups (lock table)            │          │
│  │ 4. Regular state exports                    │          │
│  └────────────────────────────────────────────┘          │
│                                                           │
│  Recovery options:                                        │
│  • Restore from S3 version                               │
│  • Restore from cross-region replica                      │
│  • Rebuild state: terraform import                        │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── State bucket with DR protection ────────────
resource "aws_s3_bucket" "state" {
  bucket = "${var.company}-terraform-state"

  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket_versioning" "state" {
  bucket = aws_s3_bucket.state.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Cross-region replication for state
resource "aws_s3_bucket_replication_configuration" "state_dr" {
  bucket = aws_s3_bucket.state.id
  role   = aws_iam_role.state_replication.arn

  rule {
    id     = "state-dr"
    status = "Enabled"

    destination {
      bucket        = aws_s3_bucket.state_dr.arn
      storage_class = "STANDARD"
    }
  }
}

# DynamoDB backup for lock table
resource "aws_dynamodb_table" "locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  point_in_time_recovery {
    enabled = true
  }

  # Global table for cross-region DR
  replica {
    region_name = "eu-west-1"
  }
}
```

---

## 3.6 Failover Automation

```hcl
# ─── Automated failover with Route 53 ───────────
resource "aws_route53_health_check" "primary" {
  fqdn              = module.primary.alb_dns_name
  port               = 443
  type               = "HTTPS"
  resource_path      = "/health"
  failure_threshold  = 3
  request_interval   = 10

  tags = { Name = "primary-health-check" }
}

resource "aws_route53_record" "failover_primary" {
  zone_id        = data.aws_route53_zone.main.zone_id
  name           = "app.${var.domain_name}"
  type           = "A"
  set_identifier = "primary"

  failover_routing_policy {
    type = "PRIMARY"
  }

  health_check_id = aws_route53_health_check.primary.id

  alias {
    name                   = module.primary.alb_dns_name
    zone_id                = module.primary.alb_zone_id
    evaluate_target_health = true
  }
}

resource "aws_route53_record" "failover_secondary" {
  zone_id        = data.aws_route53_zone.main.zone_id
  name           = "app.${var.domain_name}"
  type           = "A"
  set_identifier = "secondary"

  failover_routing_policy {
    type = "SECONDARY"
  }

  alias {
    name                   = module.dr_region.alb_dns_name
    zone_id                = module.dr_region.alb_zone_id
    evaluate_target_health = true
  }
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| DR failover too slow | Compute needs provisioning | Use warm standby with pre-provisioned instances |
| Data loss on failover | Async replication lag | Use synchronous replication or accept RPO trade-off |
| State file lost | No backups/versioning | Enable S3 versioning + cross-region replication |
| DR apply fails | Stale modules/providers | Keep DR config in sync, test regularly |
| RDS promotion fails | Manual process error | Automate with Lambda + Step Functions |

---

## Summary Table

| Strategy | RTO | RPO | Monthly Cost | Automation |
|----------|-----|-----|-------------|------------|
| **Backup & Restore** | Hours | Hours | $ (storage only) | Manual apply |
| **Pilot Light** | 10-30 min | Seconds | $$ (DB replica) | Semi-auto |
| **Warm Standby** | Minutes | Seconds | $$$ (scaled-down infra) | Semi-auto |
| **Multi-Site Active** | Near-zero | Near-zero | $$$$ (full duplicate) | Automatic |

---

## Quick Revision Questions

1. **What is the difference between RTO and RPO?**
   > RTO (Recovery Time Objective) is the maximum acceptable downtime. RPO (Recovery Point Objective) is the maximum acceptable data loss measured in time.

2. **What is a pilot light DR strategy?**
   > Core data services (databases) run continuously in the DR region with replication, but compute resources are at zero. On failover, compute is scaled up via Terraform.

3. **How does Terraform enable DR?**
   > Infrastructure is defined as code, so the same configuration can be applied in a DR region. Combined with data replication, the entire stack can be rebuilt programmatically.

4. **Why is state file DR important?**
   > The state file tracks all managed resources. Losing it means Terraform loses awareness of existing infrastructure, requiring manual import of every resource.

5. **What AWS service provides automatic DNS failover?**
   > Route 53 with failover routing policy and health checks. When the primary health check fails, traffic automatically routes to the secondary endpoint.

---

[← Previous: Multi-Region Deployment](02-multi-region-deployment.md) | [Back to README](../README.md) | [Next: Cost Optimization →](04-cost-optimization.md)
