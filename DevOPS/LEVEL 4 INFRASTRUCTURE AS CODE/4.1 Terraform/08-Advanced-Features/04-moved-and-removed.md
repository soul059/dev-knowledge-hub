# Chapter 4: Moved and Removed Blocks

[← Previous: Lifecycle Rules](03-lifecycle-rules.md) | [Back to README](../README.md) | [Next: Terraform Import →](05-terraform-import.md)

---

## Overview

**Moved** and **removed** blocks (Terraform 1.1+ and 1.7+) enable safe code refactoring without destroying and recreating infrastructure. They tell Terraform that a resource has been renamed, moved into a module, or intentionally removed from management.

---

## 4.1 The Refactoring Problem

```
┌──────────────────────────────────────────────────────────┐
│       WITHOUT moved BLOCKS                                │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Before: resource "aws_instance" "web" { ... }           │
│  After:  resource "aws_instance" "app" { ... }           │
│                                                           │
│  Terraform sees:                                          │
│  - Destroy aws_instance.web                              │
│  + Create aws_instance.app                               │
│                                                           │
│  Result: DOWNTIME! Resource destroyed and recreated.     │
│                                                           │
│  WITH moved BLOCKS                                        │
│  ──────────────────                                       │
│  moved {                                                  │
│    from = aws_instance.web                               │
│    to   = aws_instance.app                               │
│  }                                                        │
│                                                           │
│  Terraform sees:                                          │
│  ~ aws_instance.web → aws_instance.app (state updated)  │
│                                                           │
│  Result: NO DOWNTIME! Only state is updated.             │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.2 moved Block Syntax

```hcl
# ─── Basic rename ────────────────────────────────
moved {
  from = aws_instance.web
  to   = aws_instance.app
}

resource "aws_instance" "app" {
  ami           = var.ami_id
  instance_type = var.instance_type
}
```

---

## 4.3 Common Move Scenarios

### Rename a Resource

```hcl
# Before:
# resource "aws_s3_bucket" "data" { ... }

# After:
moved {
  from = aws_s3_bucket.data
  to   = aws_s3_bucket.application_data
}

resource "aws_s3_bucket" "application_data" {
  bucket = "my-app-data"
}
```

### Move Into a Module

```hcl
# Before: resource in root module
# resource "aws_vpc" "main" { ... }

# After: resource moved into a module
moved {
  from = aws_vpc.main
  to   = module.networking.aws_vpc.main
}

module "networking" {
  source = "./modules/networking"
}
```

### Move Between Modules

```hcl
# Resource moved from one module to another
moved {
  from = module.old_module.aws_instance.app
  to   = module.new_module.aws_instance.app
}
```

### Move Out of a Module

```hcl
# Move resource from module back to root
moved {
  from = module.compute.aws_instance.app
  to   = aws_instance.app
}

resource "aws_instance" "app" {
  ami           = var.ami_id
  instance_type = var.instance_type
}
```

### Rename a Module

```hcl
# Before: module "server" { source = "./modules/compute" }
# After:  module "compute" { source = "./modules/compute" }

moved {
  from = module.server
  to   = module.compute
}

module "compute" {
  source = "./modules/compute"
}
```

### Count to for_each Migration

```hcl
# Before: count-based
# resource "aws_instance" "app" {
#   count = 3
# }

# After: for_each-based
moved {
  from = aws_instance.app[0]
  to   = aws_instance.app["web-1"]
}

moved {
  from = aws_instance.app[1]
  to   = aws_instance.app["web-2"]
}

moved {
  from = aws_instance.app[2]
  to   = aws_instance.app["web-3"]
}

resource "aws_instance" "app" {
  for_each = toset(["web-1", "web-2", "web-3"])

  ami           = var.ami_id
  instance_type = var.instance_type
  tags = { Name = each.key }
}
```

---

## 4.4 removed Block (Terraform 1.7+)

```hcl
# ─── Remove from Terraform without destroying ────

# Before: resource managed by Terraform
# resource "aws_instance" "legacy" { ... }

# After: stop managing but keep the real infrastructure
removed {
  from = aws_instance.legacy

  lifecycle {
    destroy = false
  }
}

# The resource is removed from state but NOT destroyed
# in the real cloud environment
```

### With Destruction

```hcl
# Remove from Terraform AND destroy the resource
removed {
  from = aws_instance.temporary

  lifecycle {
    destroy = true    # Actually destroy the resource
  }
}
```

### Remove a Module

```hcl
# Stop managing an entire module (keep real resources)
removed {
  from = module.legacy_app

  lifecycle {
    destroy = false
  }
}
```

```
┌──────────────────────────────────────────────────────────┐
│         removed vs terraform state rm                     │
├───────────────────────┬──────────────────────────────────┤
│ removed block         │ terraform state rm               │
├───────────────────────┼──────────────────────────────────┤
│ Declarative           │ Imperative (CLI command)         │
│ In version control    │ Not tracked in code              │
│ Reviewed in PR        │ No PR review                     │
│ Repeatable            │ One-time operation               │
│ Can destroy or keep   │ Only removes from state          │
│ Team-friendly         │ Solo operation                   │
│ Terraform 1.7+        │ Any version                      │
└───────────────────────┴──────────────────────────────────┘
```

---

## 4.5 Moved Block Best Practices

```
┌──────────────────────────────────────────────────────────┐
│          MOVED BLOCK BEST PRACTICES                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  1. Keep moved blocks temporarily                        │
│     └── Remove after all states have been updated        │
│                                                           │
│  2. Test with terraform plan first                       │
│     └── Should show "moved" not "destroy + create"       │
│                                                           │
│  3. Move one step at a time                              │
│     └── Don't chain moves (A → B → C)                   │
│                                                           │
│  4. Coordinate with team                                 │
│     └── Everyone must apply the moved block              │
│                                                           │
│  5. Use in PRs for code review                           │
│     └── Reviewers can verify moves are correct           │
│                                                           │
│  6. Document the reason for the move                     │
│     └── Add comments explaining why                      │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.6 moved vs state mv

```hcl
# ─── Declarative (moved block) - PREFERRED ────────
# In your .tf files:
moved {
  from = aws_instance.old_name
  to   = aws_instance.new_name
}
# Run: terraform apply
# Tracked in version control, reviewable, repeatable

# ─── Imperative (CLI) ────────────────────────────
# terraform state mv aws_instance.old_name aws_instance.new_name
# One-time, not tracked, not reviewable
```

---

## 4.7 Complete Refactoring Example

```hcl
# ─── Original flat structure ─────────────────────
# resource "aws_vpc" "main" { ... }
# resource "aws_subnet" "public" { ... }
# resource "aws_instance" "web" { ... }
# resource "aws_db_instance" "db" { ... }

# ─── Refactored into modules ─────────────────────

# Declare all moves
moved {
  from = aws_vpc.main
  to   = module.networking.aws_vpc.main
}

moved {
  from = aws_subnet.public
  to   = module.networking.aws_subnet.public
}

moved {
  from = aws_instance.web
  to   = module.compute.aws_instance.web
}

moved {
  from = aws_db_instance.db
  to   = module.database.aws_db_instance.main
}

# New module structure
module "networking" {
  source = "./modules/networking"
}

module "compute" {
  source     = "./modules/compute"
  vpc_id     = module.networking.vpc_id
  subnet_ids = module.networking.subnet_ids
}

module "database" {
  source     = "./modules/database"
  vpc_id     = module.networking.vpc_id
  subnet_ids = module.networking.db_subnet_ids
}
```

```bash
# Verify the plan shows moves, not destroy+create
terraform plan
# Output should show:
# ~ aws_vpc.main has moved to module.networking.aws_vpc.main
# ~ aws_subnet.public has moved to module.networking.aws_subnet.public
# ~ aws_instance.web has moved to module.compute.aws_instance.web
# ~ aws_db_instance.db has moved to module.database.aws_db_instance.main

terraform apply
# State updated — no infrastructure changes!
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Plan shows destroy+create instead of move | Wrong `from` address | Check exact state address with `terraform state list` |
| `moved` block errors | Resource doesn't exist in state | Verify source address matches state |
| Chained moves fail | A → B → C not supported | Move directly: A → C |
| `removed` not recognized | Terraform < 1.7 | Upgrade Terraform or use `terraform state rm` |
| Team member gets errors | They haven't applied the move yet | Coordinate: everyone applies before removing moved blocks |

---

## Summary Table

| Feature | Purpose | Min Version |
|---------|---------|-------------|
| `moved` | Rename/relocate resources in state | TF 1.1 |
| `removed` (destroy=false) | Stop managing without destroying | TF 1.7 |
| `removed` (destroy=true) | Declarative resource removal | TF 1.7 |
| `state mv` (CLI) | Imperative state move | Any |
| `state rm` (CLI) | Imperative state removal | Any |

---

## Quick Revision Questions

1. **What does a `moved` block do?**
   > It tells Terraform that a resource has been renamed or relocated in the code, so it updates the state instead of destroying and recreating the resource.

2. **What happens without a `moved` block when renaming a resource?**
   > Terraform sees the old name as deleted and the new name as new — it destroys the real resource and creates a new one, causing downtime and data loss.

3. **What is the `removed` block used for?**
   > It declaratively removes a resource from Terraform management. With `destroy = false`, the real infrastructure is kept. With `destroy = true`, it's destroyed.

4. **How is `moved` better than `terraform state mv`?**
   > `moved` is declarative, tracked in version control, reviewed in PRs, and automatically applied by all team members. `state mv` is a one-time imperative operation.

5. **How do you migrate from count to for_each?**
   > Use individual `moved` blocks mapping each index to its for_each key: `from = resource[0]` → `to = resource["name"]`.

---

[← Previous: Lifecycle Rules](03-lifecycle-rules.md) | [Back to README](../README.md) | [Next: Terraform Import →](05-terraform-import.md)
