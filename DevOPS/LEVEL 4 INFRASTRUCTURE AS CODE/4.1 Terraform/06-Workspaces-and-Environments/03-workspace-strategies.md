# Chapter 3: Workspace Strategies

[← Previous: Environment Management](02-environment-management.md) | [Back to README](../README.md) | [Next: Workspace Alternatives →](04-workspace-alternatives.md)

---

## Overview

Choosing the right workspace strategy depends on team size, security requirements, and infrastructure complexity. This chapter covers **when to use workspaces**, **safe usage patterns**, and **organizational strategies** for real-world projects.

---

## 3.1 Workspace Use Cases

```
┌──────────────────────────────────────────────────────────┐
│            WHEN TO USE WORKSPACES                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ✓ GOOD USE CASES                                        │
│  ├── Feature branch infrastructure                       │
│  │   └── Spin up temp env per feature branch             │
│  ├── Developer sandboxes                                 │
│  │   └── Each dev gets their own workspace               │
│  ├── Testing variations                                  │
│  │   └── Test same config with different params          │
│  └── Non-production environments                         │
│      └── dev/staging with same provider config           │
│                                                           │
│  ✗ BAD USE CASES                                         │
│  ├── Production vs non-production                        │
│  │   └── Different accounts needed                       │
│  ├── Multi-region deployments                            │
│  │   └── Different provider configs needed               │
│  ├── Different security boundaries                       │
│  │   └── No per-workspace access control                 │
│  └── Entirely different infrastructure                   │
│      └── Workspaces are for same config, different state │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 Feature Branch Workspaces

```hcl
# ─── Dynamically create workspace per branch ─────

# Naming convention: feature-<ticket>-<name>
# Example: feature-123-new-api

variable "branch_name" {
  description = "Git branch name for workspace naming"
  type        = string
  default     = ""
}

locals {
  # Use branch name as workspace or fall back to terraform.workspace
  env_name = var.branch_name != "" ? var.branch_name : terraform.workspace
}

resource "aws_instance" "test" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t3.micro"

  tags = {
    Name   = "test-${local.env_name}"
    Branch = local.env_name
    TTL    = "24h"          # Auto-cleanup marker
  }
}
```

```bash
# CI/CD Pipeline Usage
BRANCH=$(echo $GITHUB_REF | sed 's/refs\/heads\///' | sed 's/\//-/g')
terraform workspace new "$BRANCH" || terraform workspace select "$BRANCH"
terraform apply -auto-approve -var="branch_name=$BRANCH"

# Cleanup on branch deletion
terraform workspace select "$BRANCH"
terraform destroy -auto-approve
terraform workspace select default
terraform workspace delete "$BRANCH"
```

---

## 3.3 Safe Workspace Patterns

### Pattern: Workspace Validation

```hcl
# ─── Prevent accidental operations ────────────────

# Validate workspace name
variable "allowed_workspaces" {
  default = ["dev", "staging", "prod"]
}

resource "null_resource" "workspace_check" {
  count = contains(var.allowed_workspaces, terraform.workspace) ? 0 : 1

  provisioner "local-exec" {
    command = "echo 'ERROR: Invalid workspace: ${terraform.workspace}' && exit 1"
  }
}

# Better approach with custom validation (TF 1.2+)
check "workspace_valid" {
  assert {
    condition     = contains(["dev", "staging", "prod"], terraform.workspace)
    error_message = "Workspace '${terraform.workspace}' is not valid. Use: dev, staging, or prod."
  }
}
```

### Pattern: Workspace-Aware Naming

```hcl
locals {
  # Consistent naming across all resources
  name_prefix = "${var.project}-${terraform.workspace}"

  common_tags = {
    Project     = var.project
    Environment = terraform.workspace
    ManagedBy   = "terraform"
    Workspace   = terraform.workspace
  }
}

resource "aws_s3_bucket" "data" {
  bucket = "${local.name_prefix}-data-${data.aws_caller_identity.current.account_id}"
  tags   = local.common_tags
}

resource "aws_dynamodb_table" "app" {
  name = "${local.name_prefix}-sessions"
  tags = local.common_tags
  # ...
}
```

### Pattern: Workspace-Based Configuration Map

```hcl
locals {
  workspace_config = {
    dev = {
      instance_type   = "t3.micro"
      min_size        = 1
      max_size        = 2
      multi_az        = false
      backup_retention = 1
      enable_monitoring = false
      domain          = "dev.example.com"
    }
    staging = {
      instance_type   = "t3.small"
      min_size        = 2
      max_size        = 4
      multi_az        = false
      backup_retention = 7
      enable_monitoring = true
      domain          = "staging.example.com"
    }
    prod = {
      instance_type   = "m5.large"
      min_size        = 3
      max_size        = 10
      multi_az        = true
      backup_retention = 30
      enable_monitoring = true
      domain          = "example.com"
    }
  }

  # Fail fast if workspace isn't in the map
  config = local.workspace_config[terraform.workspace]
}
```

---

## 3.4 Workspace Naming Conventions

```
┌──────────────────────────────────────────────────────────┐
│          WORKSPACE NAMING CONVENTIONS                     │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Environment-Based:                                       │
│   dev, staging, prod                                      │
│                                                           │
│  Team-Based:                                              │
│   team-frontend, team-backend, team-data                  │
│                                                           │
│  Feature-Based:                                           │
│   feature-123-user-auth, feature-456-payments             │
│                                                           │
│  Region-Based:                                            │
│   us-east-1, eu-west-1, ap-southeast-1                    │
│                                                           │
│  Combined:                                                │
│   prod-us-east-1, dev-eu-west-1                           │
│                                                           │
│  RULES:                                                   │
│   • Lowercase only                                        │
│   • Hyphens, not underscores                              │
│   • No special characters                                 │
│   • Short but descriptive                                 │
│   • Consistent across projects                            │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.5 CI/CD with Workspaces

```yaml
# ─── GitHub Actions: Per-Environment Deployment ──

name: Terraform Deploy
on:
  push:
    branches: [main, develop]

jobs:
  deploy:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        include:
          - branch: develop
            workspace: dev
          - branch: main
            workspace: prod

    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Terraform Init
        run: terraform init

      - name: Select Workspace
        run: |
          terraform workspace select ${{ matrix.workspace }} || \
          terraform workspace new ${{ matrix.workspace }}

      - name: Terraform Plan
        run: terraform plan -out=plan.tfplan
        env:
          TF_WORKSPACE: ${{ matrix.workspace }}

      - name: Terraform Apply
        if: github.ref == 'refs/heads/${{ matrix.branch }}'
        run: terraform apply plan.tfplan
```

---

## 3.6 Terraform Cloud Workspaces (vs CLI)

```
┌──────────────────────────────────────────────────────────┐
│     CLI WORKSPACES vs TERRAFORM CLOUD WORKSPACES          │
├───────────────────────┬──────────────────────────────────┤
│ CLI Workspaces        │ TF Cloud Workspaces              │
├───────────────────────┼──────────────────────────────────┤
│ Same config, diff     │ Can have different configs        │
│ state                 │ per workspace                    │
│                       │                                  │
│ Shared backend        │ Separate backends possible       │
│                       │                                  │
│ No access control     │ Team/user permissions per        │
│                       │ workspace                        │
│                       │                                  │
│ No audit log          │ Full audit trail                 │
│                       │                                  │
│ Manual workspace      │ Auto-create from VCS             │
│ management            │                                  │
│                       │                                  │
│ Shared variables      │ Per-workspace variables          │
│                       │                                  │
│ Same provider config  │ Different provider config        │
│                       │ per workspace                    │
│                       │                                  │
│ Free                  │ Free tier + paid plans           │
│                       │                                  │
└───────────────────────┴──────────────────────────────────┘
```

```hcl
# ─── Terraform Cloud workspace configuration ─────
terraform {
  cloud {
    organization = "my-company"

    workspaces {
      # Single workspace
      name = "my-app-prod"

      # OR pattern-matched workspaces
      # tags = ["app:my-app", "env:prod"]
    }
  }
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Wrong workspace applied to | Forgot to switch | Add workspace validation checks |
| Feature workspace orphaned | Branch deleted without cleanup | Automate cleanup in CI/CD |
| Workspace name conflicts | No naming convention | Establish team naming standards |
| Can't use different providers | CLI workspace limitation | Use directory-per-env or TF Cloud |
| State grows too large | Everything in one workspace | Split into smaller, focused states |

---

## Summary Table

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Feature branch** | Temp workspace per git branch | Testing feature changes |
| **Environment map** | Config map keyed by workspace | Multi-env with same provider |
| **Validated** | Restrict allowed workspace names | Safety in production |
| **Named conventions** | Consistent workspace naming | Team standardization |
| **CI/CD integrated** | Automated workspace selection | Deployment pipelines |
| **TF Cloud** | Enterprise workspaces | Governance & access control |

---

## Quick Revision Questions

1. **When should you NOT use CLI workspaces?**
   > When environments need different AWS accounts, regions, or provider configurations, or when per-environment access control is required.

2. **What is a feature branch workspace?**
   > A temporary workspace created per git feature branch to test infrastructure changes in isolation, then destroyed when the branch is merged/deleted.

3. **How do you prevent applying to the wrong workspace?**
   > Add workspace validation checks in code (e.g., `check` blocks), use naming conventions, and always verify with `terraform workspace show` before applying.

4. **How do Terraform Cloud workspaces differ from CLI workspaces?**
   > TF Cloud workspaces support per-workspace variables, permissions, VCS integration, audit logs, and different provider configurations — CLI workspaces only provide separate state.

5. **What is the workspace configuration map pattern?**
   > A locals map keyed by workspace name containing environment-specific values, accessed via `local.workspace_config[terraform.workspace]`.

---

[← Previous: Environment Management](02-environment-management.md) | [Back to README](../README.md) | [Next: Workspace Alternatives →](04-workspace-alternatives.md)
