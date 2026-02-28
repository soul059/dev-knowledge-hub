# Chapter 4: Cost Optimization

[← Previous: Disaster Recovery](03-disaster-recovery.md) | [Back to README](../README.md) | [Next: Compliance and Governance →](05-compliance-and-governance.md)

---

## Overview

Terraform enables **cost-aware infrastructure** through right-sizing, scheduling, reserved capacity, and automated cleanup. This chapter covers patterns for reducing cloud costs while maintaining performance and reliability.

---

## 4.1 Cost Optimization Framework

```
┌──────────────────────────────────────────────────────────┐
│           COST OPTIMIZATION PILLARS                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐ ┌──────────┐ │
│  │ Right-    │ │ Scheduling│ │ Reserved  │ │ Cleanup  │ │
│  │ Sizing    │ │           │ │ Capacity  │ │          │ │
│  ├───────────┤ ├───────────┤ ├───────────┤ ├──────────┤ │
│  │ Match     │ │ Stop dev  │ │ Commit to │ │ Remove   │ │
│  │ resources │ │ resources │ │ long-term │ │ unused   │ │
│  │ to actual │ │ after     │ │ usage for │ │ resources│ │
│  │ workload  │ │ hours     │ │ discounts │ │          │ │
│  └───────────┘ └───────────┘ └───────────┘ └──────────┘ │
│                                                           │
│  ┌───────────┐ ┌───────────┐ ┌───────────┐              │
│  │ Storage   │ │ Spot/     │ │ Tagging   │              │
│  │ Tiering   │ │ Preempt   │ │ & Billing │              │
│  ├───────────┤ ├───────────┤ ├───────────┤              │
│  │ Move old  │ │ Use spot  │ │ Track     │              │
│  │ data to   │ │ for fault │ │ costs by  │              │
│  │ cheaper   │ │ tolerant  │ │ team and  │              │
│  │ tiers     │ │ workloads │ │ project   │              │
│  └───────────┘ └───────────┘ └───────────┘              │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.2 Right-Sizing with Variables

```hcl
# ─── Environment-based sizing ───────────────────
variable "environment" {
  type = string
}

locals {
  instance_sizes = {
    dev     = "t3.small"
    staging = "t3.medium"
    prod    = "m5.large"
  }

  db_sizes = {
    dev     = "db.t3.medium"
    staging = "db.r6g.large"
    prod    = "db.r6g.2xlarge"
  }

  instance_counts = {
    dev     = 1
    staging = 2
    prod    = 3
  }

  storage_sizes = {
    dev     = 20
    staging = 50
    prod    = 200
  }
}

resource "aws_instance" "app" {
  count         = local.instance_counts[var.environment]
  instance_type = local.instance_sizes[var.environment]
  ami           = var.ami_id

  root_block_device {
    volume_size = local.storage_sizes[var.environment]
    volume_type = var.environment == "prod" ? "gp3" : "gp2"
  }
}

resource "aws_db_instance" "main" {
  instance_class    = local.db_sizes[var.environment]
  allocated_storage = local.storage_sizes[var.environment]
  multi_az          = var.environment == "prod" ? true : false
  
  # Skip backups in dev
  backup_retention_period = var.environment == "dev" ? 0 : 7
}
```

---

## 4.3 Spot Instances

```hcl
# ─── EC2 Spot Instances (up to 90% savings) ─────
resource "aws_launch_template" "spot" {
  name_prefix   = "${var.project}-spot-"
  instance_type = "m5.large"
  image_id      = var.ami_id

  instance_market_options {
    market_type = "spot"
    spot_options {
      max_price                      = "0.05"    # Max bid
      spot_instance_type             = "one-time"
      instance_interruption_behavior = "terminate"
    }
  }
}

# ─── Mixed instances ASG (spot + on-demand) ─────
resource "aws_autoscaling_group" "mixed" {
  name                = "${var.project}-mixed-asg"
  vpc_zone_identifier = var.subnet_ids
  min_size            = 2
  max_size            = 10
  desired_capacity    = 4

  mixed_instances_policy {
    instances_distribution {
      on_demand_base_capacity                  = 2    # Always 2 on-demand
      on_demand_percentage_above_base_capacity = 25   # 25% on-demand above base
      spot_allocation_strategy                 = "capacity-optimized"
    }

    launch_template {
      launch_template_specification {
        launch_template_id = aws_launch_template.app.id
        version            = "$Latest"
      }

      # Multiple instance types for better spot availability
      override {
        instance_type = "m5.large"
      }
      override {
        instance_type = "m5a.large"
      }
      override {
        instance_type = "m4.large"
      }
    }
  }
}
```

---

## 4.4 Scheduling (Start/Stop Dev Resources)

```hcl
# ─── Auto-stop dev instances after hours ─────────
resource "aws_scheduler_schedule" "stop_dev" {
  name       = "stop-dev-instances"
  group_name = "dev-scheduling"

  flexible_time_window {
    mode = "OFF"
  }

  # Stop at 7 PM weekdays
  schedule_expression          = "cron(0 19 ? * MON-FRI *)"
  schedule_expression_timezone = "America/New_York"

  target {
    arn      = "arn:aws:scheduler:::aws-sdk:ec2:stopInstances"
    role_arn = aws_iam_role.scheduler.arn

    input = jsonencode({
      InstanceIds = aws_instance.dev[*].id
    })
  }
}

resource "aws_scheduler_schedule" "start_dev" {
  name       = "start-dev-instances"
  group_name = "dev-scheduling"

  flexible_time_window {
    mode = "OFF"
  }

  # Start at 8 AM weekdays
  schedule_expression          = "cron(0 8 ? * MON-FRI *)"
  schedule_expression_timezone = "America/New_York"

  target {
    arn      = "arn:aws:scheduler:::aws-sdk:ec2:startInstances"
    role_arn = aws_iam_role.scheduler.arn

    input = jsonencode({
      InstanceIds = aws_instance.dev[*].id
    })
  }
}

# ─── RDS auto-stop for dev ──────────────────────
resource "aws_rds_cluster" "dev" {
  count              = var.environment == "dev" ? 1 : 0
  cluster_identifier = "${var.project}-dev"
  engine             = "aurora-postgresql"

  # Auto-pause after 5 min idle (Aurora Serverless)
  scaling_configuration {
    auto_pause               = true
    seconds_until_auto_pause = 300
    min_capacity             = 2
    max_capacity             = 8
  }
}
```

---

## 4.5 Storage Optimization

```hcl
# ─── S3 Lifecycle Policies ──────────────────────
resource "aws_s3_bucket_lifecycle_configuration" "data" {
  bucket = aws_s3_bucket.data.id

  rule {
    id     = "archive-old-data"
    status = "Enabled"

    transition {
      days          = 30
      storage_class = "STANDARD_IA"   # ~40% cheaper
    }

    transition {
      days          = 90
      storage_class = "GLACIER_IR"    # ~68% cheaper
    }

    transition {
      days          = 365
      storage_class = "DEEP_ARCHIVE"  # ~95% cheaper
    }

    expiration {
      days = 2555  # Delete after 7 years
    }
  }

  rule {
    id     = "cleanup-incomplete-uploads"
    status = "Enabled"

    abort_incomplete_multipart_upload {
      days_after_initiation = 7
    }
  }
}

# ─── EBS Volume Optimization ────────────────────
resource "aws_ebs_volume" "data" {
  availability_zone = var.az
  size              = 100
  type              = "gp3"       # gp3 is cheaper than gp2 for same perf
  iops              = 3000        # Free IOPS included with gp3
  throughput        = 125         # Free throughput included
  encrypted         = true

  tags = { Name = "${var.project}-data" }
}
```

---

## 4.6 Cost Estimation and Tracking

```hcl
# ─── Tagging for cost allocation ────────────────
locals {
  cost_tags = {
    Project     = var.project
    Environment = var.environment
    Team        = var.team
    CostCenter  = var.cost_center
    ManagedBy   = "terraform"
  }
}

provider "aws" {
  region = var.aws_region

  default_tags {
    tags = local.cost_tags
  }
}

# ─── AWS Budget alerts ─────────────────────────
resource "aws_budgets_budget" "monthly" {
  name         = "${var.project}-${var.environment}-monthly"
  budget_type  = "COST"
  limit_amount = var.monthly_budget
  limit_unit   = "USD"
  time_unit    = "MONTHLY"

  cost_filter {
    name   = "TagKeyValue"
    values = ["user:Project$${var.project}"]
  }

  notification {
    comparison_operator       = "GREATER_THAN"
    threshold                 = 80
    threshold_type            = "PERCENTAGE"
    notification_type         = "ACTUAL"
    subscriber_email_addresses = var.alert_emails
  }

  notification {
    comparison_operator       = "GREATER_THAN"
    threshold                 = 100
    threshold_type            = "PERCENTAGE"
    notification_type         = "FORECASTED"
    subscriber_email_addresses = var.alert_emails
  }
}
```

```bash
# ─── Infracost: Pre-apply cost estimation ────────
# Shows cost impact of Terraform changes

# Install
brew install infracost

# Generate cost breakdown
infracost breakdown --path .

# Compare costs between branches
infracost diff --path . --compare-to infracost-base.json

# CI/CD integration (post cost as PR comment)
infracost comment github \
  --path /tmp/infracost.json \
  --repo myorg/myrepo \
  --pull-request $PR_NUMBER \
  --github-token $GITHUB_TOKEN
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Unexpected high costs | Oversized dev resources | Use environment-based sizing maps |
| Spot interruptions | Capacity reclaimed | Use mixed instances, multiple instance types |
| Forgotten dev resources | No auto-shutdown | Implement scheduler + budget alerts |
| EBS costs accumulating | Unattached volumes | Tag and automate cleanup of unused volumes |
| Data transfer costs | Cross-AZ/region traffic | Keep resources in same AZ when possible |

---

## Summary Table

| Strategy | Savings | Effort | Risk |
|----------|---------|--------|------|
| **Right-sizing** | 20-40% | Low | None |
| **Dev scheduling** | 60-70% (dev) | Low | Slight inconvenience |
| **Spot instances** | 60-90% | Medium | Interruption risk |
| **Storage tiering** | 40-95% | Low | Access latency |
| **Reserved/Savings Plans** | 30-60% | Low | Commitment lock-in |
| **Resource cleanup** | Varies | Medium | Accidental deletion |

---

## Quick Revision Questions

1. **How do you implement environment-based resource sizing in Terraform?**
   > Use a locals map keyed by environment name (dev/staging/prod) to look up instance types, counts, and storage sizes. Reference with `local.instance_sizes[var.environment]`.

2. **What is a mixed instances ASG?**
   > An Auto Scaling Group that combines on-demand and spot instances, with a configurable ratio. Ensures baseline capacity with on-demand while saving money with spot.

3. **How does Infracost help with cost management?**
   > It analyzes Terraform plans and estimates the monthly cost impact of changes. It can post cost comparisons as PR comments before infrastructure is deployed.

4. **What is the cheapest S3 storage class?**
   > S3 Glacier Deep Archive (~$0.00099/GB/month), but retrieval takes 12-48 hours. For more balanced access, Glacier Instant Retrieval offers ~68% savings with millisecond access.

5. **How can you automatically stop dev resources outside business hours?**
   > Use AWS EventBridge Scheduler (or Lambda + CloudWatch Events) to stop/start instances on a cron schedule, typically stopping at 7 PM and starting at 8 AM on weekdays.

---

[← Previous: Disaster Recovery](03-disaster-recovery.md) | [Back to README](../README.md) | [Next: Compliance and Governance →](05-compliance-and-governance.md)
