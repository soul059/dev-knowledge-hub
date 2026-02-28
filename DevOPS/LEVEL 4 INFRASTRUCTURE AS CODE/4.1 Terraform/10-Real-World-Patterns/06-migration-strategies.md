# Chapter 6: Migration Strategies

[← Previous: Compliance and Governance](05-compliance-and-governance.md) | [Back to README](../README.md)

---

## Overview

Migrating existing infrastructure to Terraform (brownfield adoption) and migrating between cloud providers or architectures are common real-world challenges. This chapter covers **step-by-step migration strategies**, **import workflows**, and **coexistence patterns**.

---

## 6.1 Migration Types

```
┌──────────────────────────────────────────────────────────┐
│           MIGRATION SCENARIOS                             │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────────────────────────────────────┐         │
│  │ 1. Console/CLI → Terraform (Brownfield)     │         │
│  │    Manually created resources → IaC          │         │
│  └─────────────────────────────────────────────┘         │
│                                                           │
│  ┌─────────────────────────────────────────────┐         │
│  │ 2. CloudFormation → Terraform               │         │
│  │    AWS-native IaC → multi-cloud IaC          │         │
│  └─────────────────────────────────────────────┘         │
│                                                           │
│  ┌─────────────────────────────────────────────┐         │
│  │ 3. Terraform Version Migration              │         │
│  │    Old patterns → modern patterns            │         │
│  └─────────────────────────────────────────────┘         │
│                                                           │
│  ┌─────────────────────────────────────────────┐         │
│  │ 4. Architecture Migration                   │         │
│  │    Monolith → microservices, VM → containers │         │
│  └─────────────────────────────────────────────┘         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 6.2 Brownfield Migration (Console → Terraform)

```
┌──────────────────────────────────────────────────────────┐
│       BROWNFIELD MIGRATION PHASES                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Phase 1: Discovery                                       │
│  ├── Inventory all existing resources                     │
│  ├── Map dependencies                                     │
│  ├── Identify resource ownership                          │
│  └── Prioritize what to import                            │
│                                                           │
│  Phase 2: Import                                          │
│  ├── Write Terraform configuration                        │
│  ├── Import resources into state                          │
│  ├── Run plan to verify (no changes = success)            │
│  └── Fix drift between code and reality                   │
│                                                           │
│  Phase 3: Validate                                        │
│  ├── terraform plan shows "No changes"                    │
│  ├── Review all resource attributes                       │
│  └── Test in non-prod first                               │
│                                                           │
│  Phase 4: Operate                                         │
│  ├── All future changes via Terraform                     │
│  ├── Restrict console access                              │
│  └── Enable drift detection                               │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── Phase 1: Discovery with AWS CLI ────────────
# List all resources by type
# aws ec2 describe-vpcs --output table
# aws ec2 describe-instances --output table
# aws rds describe-db-instances --output table
# aws s3 ls

# Or use AWS Resource Explorer / Resource Groups

# ─── Phase 2: Import block method (TF 1.5+) ─────
import {
  to = aws_vpc.main
  id = "vpc-0abc123def456"
}

import {
  to = aws_subnet.public[0]
  id = "subnet-0aaa111"
}

import {
  to = aws_subnet.public[1]
  id = "subnet-0bbb222"
}

import {
  to = aws_instance.web
  id = "i-0abc123def456789"
}

import {
  to = aws_db_instance.main
  id = "production-database"
}

# ─── Generate config automatically ──────────────
# terraform plan -generate-config-out=generated.tf
# Review and refine the generated code
# Remove import blocks after successful import
```

---

## 6.3 CloudFormation to Terraform

```
┌──────────────────────────────────────────────────────────┐
│    CLOUDFORMATION → TERRAFORM MIGRATION                   │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Option A: Parallel (recommended)                         │
│  ┌─────────────────────────────────────────┐             │
│  │ 1. Write TF config matching CFN stack    │             │
│  │ 2. Import each resource into TF state    │             │
│  │ 3. Verify plan = "No changes"            │             │
│  │ 4. Remove resources from CFN stack       │             │
│  │    (retain, don't delete)                │             │
│  │ 5. Delete empty CFN stack                │             │
│  └─────────────────────────────────────────┘             │
│                                                           │
│  Option B: cf2tf / former2 (tools)                        │
│  ┌─────────────────────────────────────────┐             │
│  │ 1. Export running infra with former2     │             │
│  │ 2. Convert CFN template with cf2tf       │             │
│  │ 3. Review and fix generated Terraform    │             │
│  │ 4. Import resources                      │             │
│  └─────────────────────────────────────────┘             │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── CFN Stack: Remove resource without deleting ─
# In CloudFormation, set DeletionPolicy: Retain
# Then remove the resource from the template
# CFN update removes it from CFN state but keeps the resource

# ─── Terraform: Import the resource ─────────────
import {
  to = aws_instance.web
  id = "i-0abc123"   # Same instance CFN was managing
}

# After import, verify:
# terraform plan  →  "No changes"
# CFN no longer manages it, Terraform now does
```

---

## 6.4 Terraform Code Migration (Refactoring)

```hcl
# ─── Migrating from count to for_each ───────────
# OLD: count-based
resource "aws_subnet" "public" {
  count      = 3
  vpc_id     = aws_vpc.main.id
  cidr_block = var.subnet_cidrs[count.index]
}

# NEW: for_each-based
resource "aws_subnet" "public" {
  for_each   = toset(var.subnet_cidrs)
  vpc_id     = aws_vpc.main.id
  cidr_block = each.value
}

# State migration with moved blocks
moved {
  from = aws_subnet.public[0]
  to   = aws_subnet.public["10.0.1.0/24"]
}
moved {
  from = aws_subnet.public[1]
  to   = aws_subnet.public["10.0.2.0/24"]
}
moved {
  from = aws_subnet.public[2]
  to   = aws_subnet.public["10.0.3.0/24"]
}

# ─── Extracting into a module ───────────────────
# Before: resources in root module
resource "aws_vpc" "main" { ... }
resource "aws_subnet" "public" { ... }
resource "aws_subnet" "private" { ... }

# After: call a module
module "networking" {
  source   = "./modules/networking"
  vpc_cidr = var.vpc_cidr
}

# State migration
moved {
  from = aws_vpc.main
  to   = module.networking.aws_vpc.main
}
moved {
  from = aws_subnet.public
  to   = module.networking.aws_subnet.public
}
moved {
  from = aws_subnet.private
  to   = module.networking.aws_subnet.private
}
```

---

## 6.5 Architecture Migration Patterns

```
┌──────────────────────────────────────────────────────────┐
│       VM → CONTAINER MIGRATION                            │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Phase 1: Current State (VMs)                             │
│  ┌─────────────┐  ┌─────────────┐                        │
│  │ EC2 (app1)  │  │ EC2 (app2)  │                        │
│  └──────┬──────┘  └──────┬──────┘                        │
│         └────────┬───────┘                                │
│                  │                                        │
│         ┌────────┴────────┐                               │
│         │   RDS Database  │                               │
│         └─────────────────┘                               │
│                                                           │
│  Phase 2: Add Container Platform                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │ EC2 (app1)  │  │ ECS/EKS     │  │ EC2 (app2)  │      │
│  │ (legacy)    │  │ (new apps)  │  │ (migrating) │      │
│  └─────────────┘  └─────────────┘  └─────────────┘      │
│                                                           │
│  Phase 3: Fully Containerized                             │
│  ┌─────────────────────────────────┐                      │
│  │        ECS / EKS Cluster        │                      │
│  │  ┌───────┐ ┌───────┐ ┌───────┐ │                      │
│  │  │ app1  │ │ app2  │ │ app3  │ │                      │
│  │  └───────┘ └───────┘ └───────┘ │                      │
│  └─────────────────────────────────┘                      │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── Progressive migration with feature flags ───
variable "use_containers" {
  type        = bool
  description = "Enable container-based deployment"
  default     = false
}

# Keep VM resources until migration is complete
resource "aws_instance" "legacy_app" {
  count         = var.use_containers ? 0 : var.app_count
  ami           = var.ami_id
  instance_type = var.instance_type
}

# Container resources activated by flag
resource "aws_ecs_service" "app" {
  count           = var.use_containers ? 1 : 0
  name            = "${var.project}-app"
  cluster         = aws_ecs_cluster.main[0].id
  task_definition = aws_ecs_task_definition.app[0].arn
  desired_count   = var.app_count
}

# Weighted routing for gradual traffic shift
resource "aws_route53_record" "app" {
  zone_id = data.aws_route53_zone.main.zone_id
  name    = "app.${var.domain_name}"
  type    = "A"

  set_identifier = "legacy"

  weighted_routing_policy {
    weight = var.use_containers ? 0 : 100
  }

  alias {
    name    = aws_lb.legacy[0].dns_name
    zone_id = aws_lb.legacy[0].zone_id
    evaluate_target_health = true
  }
}
```

---

## 6.6 Migration Checklist

```
┌──────────────────────────────────────────────────────────┐
│       MIGRATION CHECKLIST                                 │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Pre-Migration:                                           │
│  □ Complete resource inventory                            │
│  □ Map all dependencies                                   │
│  □ Identify stateful vs stateless resources               │
│  □ Document current-state architecture                    │
│  □ Plan rollback strategy                                 │
│  □ Schedule maintenance window (if needed)                │
│                                                           │
│  During Migration:                                        │
│  □ Start with non-critical resources                      │
│  □ Import → plan → verify (no destroy)                    │
│  □ Test in non-prod first                                 │
│  □ Migrate one service/component at a time                │
│  □ Keep old and new running in parallel                   │
│  □ Validate each step before proceeding                   │
│                                                           │
│  Post-Migration:                                          │
│  □ Verify all resources in state                          │
│  □ terraform plan = "No changes"                          │
│  □ Enable drift detection                                 │
│  □ Restrict manual/console changes                        │
│  □ Document runbooks and procedures                       │
│  □ Train team on Terraform workflows                      │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Import shows plan changes | Config doesn't match actual resource | Adjust config to match, or accept the drift |
| CloudFormation won't release resource | DeletionPolicy not set to Retain | Set `DeletionPolicy: Retain` before removing from stack |
| Moved block causes recreate | Wrong `from`/`to` addresses | Double-check addresses with `terraform state list` |
| Migration breaks dependencies | Resources imported out of order | Import dependencies first (VPC before subnet) |
| State file too large after import | Imported everything at once | Split into multiple state files by component |

---

## Summary Table

| Migration Type | Approach | Risk | Duration |
|---------------|----------|------|----------|
| **Console → Terraform** | Import blocks + config generation | Low | Days-Weeks |
| **CFN → Terraform** | Parallel management, gradual transfer | Medium | Weeks |
| **Code refactoring** | Moved blocks, state manipulation | Low | Hours-Days |
| **VM → Containers** | Feature flags, weighted routing | Medium | Weeks-Months |
| **Cloud migration** | Parallel deployment, DNS cutover | High | Months |

---

## Quick Revision Questions

1. **What is a brownfield migration?**
   > Adopting Terraform for infrastructure that already exists (created manually or by other tools). Resources are imported into Terraform state rather than created new.

2. **How do you verify a successful import?**
   > Run `terraform plan` after importing — it should show "No changes." Any planned changes indicate drift between the Terraform config and the actual resource.

3. **How do you migrate from CloudFormation to Terraform?**
   > Write matching Terraform config, import each resource, verify no-change plan, then remove resources from CFN with `DeletionPolicy: Retain` so CFN releases them without destroying them.

4. **What are moved blocks used for?**
   > Refactoring Terraform code (renaming resources, moving into modules, switching from count to for_each) without destroying and recreating the actual infrastructure.

5. **What is the safest migration strategy?**
   > Incremental migration — one component at a time, starting with non-critical resources, testing in non-prod, and running old and new in parallel until verification is complete.

---

[← Previous: Compliance and Governance](05-compliance-and-governance.md) | [Back to README](../README.md)
