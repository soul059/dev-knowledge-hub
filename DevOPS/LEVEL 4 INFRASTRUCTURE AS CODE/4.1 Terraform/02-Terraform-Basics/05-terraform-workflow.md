# Chapter 5: Terraform Workflow (init, plan, apply, destroy)

[← Previous: Resources and Data Sources](04-resources-and-data-sources.md) | [Back to README](../README.md) | [Next: Variables →](../03-Terraform-Language/01-variables.md)

---

## Overview

Terraform follows a **predictable, four-stage workflow**: `init → plan → apply → destroy`. Understanding each stage — what it does, what it outputs, and when to use it — is fundamental to using Terraform safely.

---

## 5.1 The Core Workflow

```
┌────────────────────────────────────────────────────────────────────┐
│                    TERRAFORM CORE WORKFLOW                        │
├────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────────┐  │
│   │  WRITE   │──▶│   INIT   │──▶│   PLAN   │──▶│    APPLY     │  │
│   │  .tf     │   │          │   │          │   │              │  │
│   │  files   │   │ Download │   │ Preview  │   │ Execute      │  │
│   │          │   │ plugins  │   │ changes  │   │ changes      │  │
│   └──────────┘   └──────────┘   └──────────┘   └──────────────┘  │
│                                                       │            │
│                                                       ▼            │
│                                                 ┌──────────────┐  │
│                                                 │   DESTROY    │  │
│                                                 │ (cleanup)    │  │
│                                                 └──────────────┘  │
│                                                                     │
│   Additional commands:                                             │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────┐     │
│   │ validate │  │   fmt    │  │  output  │  │   refresh    │     │
│   │          │  │          │  │          │  │              │     │
│   └──────────┘  └──────────┘  └──────────┘  └──────────────┘     │
│                                                                     │
└────────────────────────────────────────────────────────────────────┘
```

---

## 5.2 terraform init

**Purpose:** Initialize the working directory — download providers, modules, and set up the backend.

```bash
# Basic init
terraform init

# Upgrade providers to latest allowed versions
terraform init -upgrade

# Reconfigure backend
terraform init -reconfigure

# Migrate state to a new backend
terraform init -migrate-state
```

### What init Does

```
  $ terraform init
  
  ┌─────────────────────────────────────────────────────┐
  │  Step 1: Initialize Backend                         │
  │  ├── Create/verify state storage                    │
  │  └── Configure state locking                        │
  │                                                      │
  │  Step 2: Download Provider Plugins                  │
  │  ├── Parse required_providers block                 │
  │  ├── Download matching versions                     │
  │  └── Store in .terraform/providers/                 │
  │                                                      │
  │  Step 3: Download Modules                           │
  │  ├── Parse module blocks                            │
  │  └── Store in .terraform/modules/                   │
  │                                                      │
  │  Step 4: Create Lock File                           │
  │  └── .terraform.lock.hcl                            │
  │                                                      │
  │  Output: "Terraform has been successfully           │
  │           initialized!"                              │
  └─────────────────────────────────────────────────────┘
```

### When to Run init

```
  Run terraform init when:
  ✓ Starting a new project
  ✓ Adding a new provider
  ✓ Adding a new module
  ✓ Changing backend configuration
  ✓ Cloning an existing project (first time)
  ✓ After upgrading Terraform version
  
  You DON'T need init for:
  ✗ Changing resource arguments
  ✗ Adding new resources (same provider)
  ✗ Modifying variables
```

---

## 5.3 terraform validate

**Purpose:** Check configuration for syntax errors and internal consistency.

```bash
# Validate configuration
terraform validate

# Output on success:
# Success! The configuration is valid.
```

```
  ┌──────────────────────────────────────────────────────┐
  │             WHAT VALIDATE CHECKS                     │
  ├──────────────────────────────────────────────────────┤
  │                                                       │
  │  ✓ HCL syntax is correct                            │
  │  ✓ Required arguments are present                   │
  │  ✓ Attribute types match declarations               │
  │  ✓ References are valid (resource.name.attr)        │
  │  ✓ No circular dependencies                         │
  │                                                       │
  │  ✗ Does NOT check cloud credentials                 │
  │  ✗ Does NOT check if AMIs/resources exist           │
  │  ✗ Does NOT communicate with any API                │
  │                                                       │
  └──────────────────────────────────────────────────────┘
```

---

## 5.4 terraform plan

**Purpose:** Preview changes without executing them. The most important safety step.

```bash
# Generate and display execution plan
terraform plan

# Save plan to a file (for apply later)
terraform plan -out=tfplan

# Plan for destruction
terraform plan -destroy

# Target specific resources
terraform plan -target=aws_instance.web

# Pass variables
terraform plan -var="instance_type=t3.large"
terraform plan -var-file="production.tfvars"
```

### Reading Plan Output

```
  $ terraform plan
  
  ┌─────────────────────────────────────────────────────────────┐
  │  Symbols in plan output:                                   │
  │                                                              │
  │  +  CREATE   — New resource will be created                │
  │  -  DESTROY  — Existing resource will be destroyed         │
  │  ~  UPDATE   — Resource will be modified in-place          │
  │  -/+ REPLACE — Resource will be destroyed and recreated    │
  │  <=  READ    — Data source will be read                    │
  │                                                              │
  └─────────────────────────────────────────────────────────────┘
  
  Example output:
  
  # aws_instance.web will be created
  + resource "aws_instance" "web" {
      + ami                    = "ami-0c55b159cbfafe1f0"
      + instance_type          = "t3.micro"
      + id                     = (known after apply)
      + public_ip              = (known after apply)
      + tags                   = {
          + "Name" = "web-server"
        }
    }
  
  # aws_security_group.web will be updated in-place
  ~ resource "aws_security_group" "web" {
        id   = "sg-12345678"
      ~ name = "old-name" -> "new-name"
    }
  
  Plan: 1 to add, 1 to change, 0 to destroy.
```

### Saved Plans

```bash
# Save the plan
terraform plan -out=tfplan

# Apply the EXACT saved plan (no confirmation prompt)
terraform apply tfplan

# This guarantees what was reviewed IS what gets applied
# Critical for CI/CD pipelines
```

```
  ┌─────────────────────────────────────────────────────┐
  │            SAVED PLAN WORKFLOW                      │
  ├─────────────────────────────────────────────────────┤
  │                                                      │
  │  Developer           CI/CD Pipeline                 │
  │  ┌──────────┐       ┌──────────────────┐           │
  │  │ terraform│─plan──▶│   Review plan   │           │
  │  │ plan     │─save──▶│   (human/auto)  │           │
  │  │ -out=tfplan│      │                  │           │
  │  └──────────┘       └───────┬──────────┘           │
  │                              │ approved             │
  │                              ▼                      │
  │                     ┌──────────────────┐           │
  │                     │ terraform apply  │           │
  │                     │ tfplan           │           │
  │                     └──────────────────┘           │
  │                                                      │
  └─────────────────────────────────────────────────────┘
```

---

## 5.5 terraform apply

**Purpose:** Execute the planned changes to create, update, or destroy infrastructure.

```bash
# Apply with confirmation prompt
terraform apply

# Apply a saved plan (no prompt)
terraform apply tfplan

# Auto-approve (CI/CD — use with caution)
terraform apply -auto-approve

# Apply specific resources only
terraform apply -target=aws_instance.web

# Apply with variables
terraform apply -var="env=production"
terraform apply -var-file="prod.tfvars"

# Apply with parallelism control
terraform apply -parallelism=5    # Default is 10
```

### Apply Process

```
  $ terraform apply
  
  ┌─────────────────────────────────────────────────────────────┐
  │  1. REFRESH — Query cloud APIs for current state           │
  │     "Is aws_instance.web still running? What's its IP?"    │
  │                                                              │
  │  2. PLAN — Compare desired vs current state                │
  │     "Config says 3 servers. State shows 2. Need 1 more."  │
  │                                                              │
  │  3. PROMPT — Show plan and ask for confirmation            │
  │     "Plan: 1 to add, 0 to change, 0 to destroy."          │
  │     "Do you want to perform these actions?"                │
  │     "Enter a value: yes"                                   │
  │                                                              │
  │  4. EXECUTE — Call cloud APIs to make changes              │
  │     "aws_instance.web: Creating..."                        │
  │     "aws_instance.web: Creation complete after 45s"        │
  │                                                              │
  │  5. UPDATE STATE — Record new resource IDs                 │
  │     State now includes new instance ID: i-0abc123...       │
  │                                                              │
  │  Apply complete! Resources: 1 added, 0 changed, 0 destroyed│
  └─────────────────────────────────────────────────────────────┘
```

---

## 5.6 terraform destroy

**Purpose:** Remove all resources managed by the current configuration.

```bash
# Destroy with confirmation prompt
terraform destroy

# Auto-approve destruction (dangerous!)
terraform destroy -auto-approve

# Destroy specific resources
terraform destroy -target=aws_instance.web

# Preview what will be destroyed
terraform plan -destroy
```

```
  ┌─────────────────────────────────────────────────────────────┐
  │                  DESTROY PROCESS                            │
  ├─────────────────────────────────────────────────────────────┤
  │                                                              │
  │  terraform destroy                                          │
  │       │                                                      │
  │       ▼                                                      │
  │  Read state file → Find all managed resources              │
  │       │                                                      │
  │       ▼                                                      │
  │  Calculate destroy order (reverse of creation)              │
  │       │                                                      │
  │       ▼                                                      │
  │  ┌────────┐   ┌────────┐   ┌────────┐                     │
  │  │  EC2   │──▶│ Subnet │──▶│  VPC   │  Destroy order      │
  │  │(last)  │   │(middle)│   │(first) │  (reverse deps)     │
  │  └────────┘   └────────┘   └────────┘                     │
  │                                                              │
  │  Clear state file                                           │
  │                                                              │
  │  Destroy complete! Resources: 3 destroyed.                  │
  │                                                              │
  └─────────────────────────────────────────────────────────────┘
```

---

## 5.7 Additional Commands

```bash
# FORMAT — Auto-format .tf files
terraform fmt
terraform fmt -recursive
terraform fmt -check          # CI: fail if not formatted

# OUTPUT — Display output values
terraform output
terraform output bucket_name  # Specific output
terraform output -json        # JSON format

# SHOW — Display state or plan
terraform show               # Show current state
terraform show tfplan        # Show saved plan

# REFRESH — Sync state with real infrastructure
terraform refresh            # Deprecated — use apply -refresh-only
terraform apply -refresh-only

# GRAPH — Generate dependency graph (DOT format)
terraform graph | dot -Tpng > graph.png

# CONSOLE — Interactive expression tester
terraform console
> var.region
"us-east-1"
> length(var.subnets)
3
> exit

# TAINT / UNTAINT — Mark resource for recreation (deprecated)
# Use -replace instead:
terraform apply -replace="aws_instance.web"
```

---

## 5.8 Complete Workflow Example

```bash
# 1. Create project
mkdir my-infra && cd my-infra

# 2. Write configuration
cat > main.tf << 'EOF'
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "app" {
  bucket = "my-unique-app-bucket-12345"
  tags   = { Environment = "dev" }
}

output "bucket_arn" {
  value = aws_s3_bucket.app.arn
}
EOF

# 3. Initialize
terraform init

# 4. Format
terraform fmt

# 5. Validate
terraform validate

# 6. Plan
terraform plan -out=tfplan

# 7. Apply
terraform apply tfplan

# 8. Check outputs
terraform output

# 9. Make a change (update tags)
# Edit main.tf...

# 10. Plan change
terraform plan

# 11. Apply change
terraform apply

# 12. Cleanup
terraform destroy
```

---

## 5.9 Team Workflow

```
  ┌─────────────────────────────────────────────────────────────┐
  │                TEAM WORKFLOW WITH GIT                       │
  ├─────────────────────────────────────────────────────────────┤
  │                                                              │
  │  Developer:                                                 │
  │  ┌──────────┐                                               │
  │  │ 1. git   │  Create feature branch                       │
  │  │ checkout │  git checkout -b add-rds                     │
  │  └────┬─────┘                                               │
  │       ▼                                                      │
  │  ┌──────────┐                                               │
  │  │ 2. Write │  Edit .tf files                               │
  │  │ code     │  terraform fmt && terraform validate          │
  │  └────┬─────┘                                               │
  │       ▼                                                      │
  │  ┌──────────┐                                               │
  │  │ 3. Plan  │  terraform plan (local preview)               │
  │  └────┬─────┘                                               │
  │       ▼                                                      │
  │  ┌──────────┐                                               │
  │  │ 4. Push  │  git push → Create Pull Request              │
  │  └────┬─────┘                                               │
  │       ▼                                                      │
  │  CI/CD Pipeline:                                            │
  │  ┌──────────┐                                               │
  │  │ 5. Auto  │  terraform fmt -check                        │
  │  │ checks   │  terraform validate                          │
  │  │          │  terraform plan (comment on PR)               │
  │  └────┬─────┘                                               │
  │       ▼                                                      │
  │  ┌──────────┐                                               │
  │  │ 6. Review│  Team reviews code + plan output             │
  │  └────┬─────┘                                               │
  │       ▼                                                      │
  │  ┌──────────┐                                               │
  │  │ 7. Merge │  Merge PR → terraform apply (automated)      │
  │  │ + Apply  │                                               │
  │  └──────────┘                                               │
  │                                                              │
  └─────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `terraform plan` shows unexpected changes | State out of sync | Run `terraform apply -refresh-only` |
| Apply fails mid-way (partial apply) | API error or quota | Fix the issue, re-run `terraform apply` (idempotent) |
| "Error acquiring state lock" | Another apply running | Wait, or force-unlock: `terraform force-unlock LOCK_ID` |
| Plan shows destroy + recreate | Immutable attribute changed (e.g., AMI) | Expected behavior; use `create_before_destroy` for zero downtime |
| `-target` causes weird state | Partial applies skip dependencies | Avoid `-target` except for debugging; always do full apply |

---

## Summary Table

| Command | Purpose | Safe? |
|---------|---------|-------|
| `terraform init` | Download providers, modules, set up backend | Yes — read-only |
| `terraform validate` | Check syntax and internal consistency | Yes — no API calls |
| `terraform fmt` | Auto-format `.tf` files | Yes — only edits local files |
| `terraform plan` | Preview changes (diff) | Yes — read-only |
| `terraform apply` | Execute changes | **Modifies infrastructure** |
| `terraform destroy` | Delete all managed resources | **Destructive** |
| `terraform output` | Show output values | Yes — reads state only |
| `terraform show` | Display state or saved plan | Yes — read-only |
| `terraform console` | Interactive expression tester | Yes — no changes |
| `terraform graph` | Generate dependency graph | Yes — read-only |

---

## Quick Revision Questions

1. **What are the four core Terraform commands in order?**
   > `init` → `plan` → `apply` → `destroy`.

2. **What does `terraform plan -out=tfplan` do and why is it important?**
   > It saves the execution plan to a file. Applying the saved plan guarantees the exact reviewed changes are applied — critical for auditing and CI/CD.

3. **What do the symbols +, -, ~, and -/+ mean in plan output?**
   > `+` = create, `-` = destroy, `~` = update in-place, `-/+` = destroy and recreate.

4. **When should you run `terraform init`?**
   > When starting a new project, adding providers/modules, changing backend config, or cloning a project for the first time.

5. **What is the risk of using `-auto-approve` and when is it acceptable?**
   > It skips the confirmation prompt, which can cause accidental changes. Acceptable only in CI/CD pipelines where the plan was already reviewed and approved.

6. **What happens if `terraform apply` fails halfway through?**
   > Terraform updates the state for successfully created resources. Re-running `apply` will only attempt the remaining changes — it's idempotent.

---

[← Previous: Resources and Data Sources](04-resources-and-data-sources.md) | [Back to README](../README.md) | [Next: Variables →](../03-Terraform-Language/01-variables.md)
