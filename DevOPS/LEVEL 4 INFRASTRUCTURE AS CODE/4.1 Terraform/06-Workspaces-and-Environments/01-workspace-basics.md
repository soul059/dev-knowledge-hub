# Chapter 1: Workspace Basics

[← Previous: Module Composition](../05-Modules/07-module-composition.md) | [Back to README](../README.md) | [Next: Environment Management →](02-environment-management.md)

---

## Overview

**Terraform workspaces** allow you to manage multiple distinct sets of infrastructure state within a single configuration directory. Each workspace has its own state file, enabling the same code to target different environments or variations.

---

## 1.1 What Are Workspaces?

```
┌──────────────────────────────────────────────────────────┐
│                  TERRAFORM WORKSPACES                     │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Same Configuration ──┬──► Workspace: default             │
│  (main.tf, etc.)      │        └── terraform.tfstate      │
│                       │                                   │
│                       ├──► Workspace: dev                 │
│                       │        └── terraform.tfstate      │
│                       │                                   │
│                       ├──► Workspace: staging              │
│                       │        └── terraform.tfstate      │
│                       │                                   │
│                       └──► Workspace: prod                │
│                                └── terraform.tfstate      │
│                                                           │
│  Each workspace = isolated state                          │
│  Same code, different infrastructure instances            │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.2 Default Workspace

Every Terraform configuration starts with a workspace called `default`. You cannot delete it.

```bash
# Check current workspace
terraform workspace show
# Output: default

# List all workspaces (* marks current)
terraform workspace list
# Output:
# * default
```

---

## 1.3 Workspace Commands

```bash
# ─── Create a new workspace ───────────────────────
terraform workspace new dev
# Created and switched to workspace "dev"!

# ─── List workspaces ──────────────────────────────
terraform workspace list
#   default
# * dev

# ─── Switch workspace ─────────────────────────────
terraform workspace select staging
# Switched to workspace "staging"

# ─── Delete a workspace ───────────────────────────
terraform workspace select default
terraform workspace delete dev
# Deleted workspace "dev"!

# ─── Show current workspace ───────────────────────
terraform workspace show
# default
```

> **Note:** You must switch away from a workspace before deleting it. You cannot delete `default`.

---

## 1.4 How State Is Stored

### Local Backend

```
project/
├── main.tf
├── variables.tf
├── terraform.tfstate              ← default workspace
└── terraform.tfstate.d/
    ├── dev/
    │   └── terraform.tfstate      ← dev workspace
    ├── staging/
    │   └── terraform.tfstate      ← staging workspace
    └── prod/
        └── terraform.tfstate      ← prod workspace
```

### Remote Backend (S3)

```
s3://my-tf-state/
├── env:/
│   ├── dev/
│   │   └── project/terraform.tfstate
│   ├── staging/
│   │   └── project/terraform.tfstate
│   └── prod/
│       └── project/terraform.tfstate
└── project/terraform.tfstate         ← default workspace
```

```hcl
# Backend configuration (same for all workspaces)
terraform {
  backend "s3" {
    bucket = "my-tf-state"
    key    = "project/terraform.tfstate"
    region = "us-east-1"
  }
}
```

---

## 1.5 Using terraform.workspace

The `terraform.workspace` variable returns the name of the current workspace. Use it to vary behavior:

```hcl
# ─── Vary resource names by workspace ─────────────
resource "aws_instance" "app" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = terraform.workspace == "prod" ? "m5.large" : "t3.micro"

  tags = {
    Name        = "app-${terraform.workspace}"
    Environment = terraform.workspace
  }
}

# ─── Vary resource count by workspace ─────────────
resource "aws_instance" "app" {
  count = terraform.workspace == "prod" ? 3 : 1

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = lookup(var.instance_types, terraform.workspace, "t3.micro")

  tags = {
    Name = "app-${terraform.workspace}-${count.index}"
  }
}

# ─── Workspace-specific variables map ─────────────
variable "instance_types" {
  default = {
    dev     = "t3.micro"
    staging = "t3.small"
    prod    = "m5.large"
  }
}

variable "instance_counts" {
  default = {
    dev     = 1
    staging = 2
    prod    = 3
  }
}

locals {
  instance_type  = lookup(var.instance_types, terraform.workspace, "t3.micro")
  instance_count = lookup(var.instance_counts, terraform.workspace, 1)
}
```

---

## 1.6 Complete Workspace Example

```hcl
# ─── variables.tf ─────────────────────────────────
variable "config" {
  default = {
    dev = {
      instance_type = "t3.micro"
      min_size      = 1
      max_size      = 2
      multi_az      = false
    }
    staging = {
      instance_type = "t3.small"
      min_size      = 2
      max_size      = 4
      multi_az      = false
    }
    prod = {
      instance_type = "m5.large"
      min_size      = 3
      max_size      = 10
      multi_az      = true
    }
  }
}

locals {
  env    = terraform.workspace
  config = var.config[local.env]
}

# ─── main.tf ──────────────────────────────────────
resource "aws_db_instance" "main" {
  identifier     = "myapp-${local.env}"
  instance_class = local.env == "prod" ? "db.r6g.large" : "db.t3.micro"
  multi_az       = local.config.multi_az

  tags = {
    Name        = "myapp-db-${local.env}"
    Environment = local.env
    ManagedBy   = "terraform"
  }
}

resource "aws_autoscaling_group" "app" {
  name     = "myapp-${local.env}"
  min_size = local.config.min_size
  max_size = local.config.max_size

  tag {
    key                 = "Environment"
    value               = local.env
    propagate_at_launch = true
  }
}
```

```bash
# Deploy dev
terraform workspace select dev
terraform apply

# Deploy staging
terraform workspace select staging
terraform apply

# Deploy prod
terraform workspace select prod
terraform apply
```

---

## 1.7 Workspace Limitations

```
┌──────────────────────────────────────────────────────────┐
│             WORKSPACE LIMITATIONS                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ⚠ Same backend configuration for all workspaces         │
│    └── Cannot use different accounts/regions per ws      │
│                                                           │
│  ⚠ Same provider configuration for all workspaces        │
│    └── Must use terraform.workspace in provider config   │
│                                                           │
│  ⚠ No workspace-level access control (OSS)               │
│    └── Anyone with access can switch workspaces          │
│                                                           │
│  ⚠ Easy to forget which workspace you're in              │
│    └── Risk of applying to wrong environment             │
│                                                           │
│  ⚠ State stored in same backend                          │
│    └── Blast radius if backend is compromised            │
│                                                           │
│  ✓ Terraform Cloud workspaces are more feature-rich      │
│    └── Separate variables, permissions, VCS per ws       │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Resources created in wrong env | Wrong workspace selected | Always run `terraform workspace show` before apply |
| Cannot delete workspace | Currently selected or has state | Switch away; use `-force` if state exists |
| `terraform.workspace` returns "default" | Never created/selected workspace | Create workspace with `terraform workspace new` |
| Workspace state not found remotely | Backend doesn't support workspaces | Check backend documentation; not all support it |
| Different configs needed per env | Workspace limitations | Consider directory-based approach instead |

---

## Summary Table

| Item | Detail |
|------|--------|
| **Default workspace** | `default` — always exists, cannot be deleted |
| **Create** | `terraform workspace new <name>` |
| **Switch** | `terraform workspace select <name>` |
| **List** | `terraform workspace list` |
| **Delete** | `terraform workspace delete <name>` |
| **Current** | `terraform workspace show` or `terraform.workspace` |
| **State storage** | Separate state file per workspace |
| **Best for** | Simple environment separation with same config |

---

## Quick Revision Questions

1. **What is the default Terraform workspace?**
   > `default` — it's created automatically and cannot be deleted.

2. **How do you reference the current workspace in configuration?**
   > Use the `terraform.workspace` variable, which returns the workspace name as a string.

3. **Where does Terraform store workspace state locally?**
   > In `terraform.tfstate.d/<workspace_name>/terraform.tfstate`. The `default` workspace uses `terraform.tfstate` in the root.

4. **What is a key limitation of CLI workspaces?**
   > All workspaces share the same backend and provider configuration, so you cannot use different accounts or regions per workspace without workarounds.

5. **Can you delete the default workspace?**
   > No. The default workspace always exists and cannot be deleted.

---

[← Previous: Module Composition](../05-Modules/07-module-composition.md) | [Back to README](../README.md) | [Next: Environment Management →](02-environment-management.md)
