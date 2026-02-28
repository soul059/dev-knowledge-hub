# Chapter 2: Environment Management

[← Previous: Workspace Basics](01-workspace-basics.md) | [Back to README](../README.md) | [Next: Workspace Strategies →](03-workspace-strategies.md)

---

## Overview

**Environment management** in Terraform involves structuring your code and state to safely manage multiple environments (dev, staging, prod). There are several approaches, each with trade-offs in complexity, isolation, and maintainability.

---

## 2.1 Environment Strategies Compared

```
┌───────────────────────────────────────────────────────────────┐
│           ENVIRONMENT MANAGEMENT STRATEGIES                    │
├───────────┬───────────────────────┬───────────────────────────┤
│ Strategy  │        How            │ Isolation Level           │
├───────────┼───────────────────────┼───────────────────────────┤
│           │                       │                           │
│ Workspaces│ Same code, switch     │ LOW  - shared backend,    │
│           │ workspace context     │        shared providers   │
│           │                       │                           │
│ Directories│ Copy/symlink code    │ HIGH - separate backends, │
│           │ per environment       │        separate configs   │
│           │                       │                           │
│ Var Files │ Same code, different  │ MED  - shared code,       │
│           │ .tfvars per env       │        separate state     │
│           │                       │                           │
│ Terragrunt│ DRY wrapper around    │ HIGH - separate state,    │
│           │ Terraform             │        DRY code           │
│           │                       │                           │
│ TF Cloud  │ Workspaces with       │ HIGH - full RBAC,         │
│           │ VCS integration       │        separate vars      │
│           │                       │                           │
└───────────┴───────────────────────┴───────────────────────────┘
```

---

## 2.2 Strategy 1: Workspaces

```hcl
# Same codebase, switch with workspace commands
# ─── main.tf ──────────────────────────────────────
locals {
  env_config = {
    dev = {
      instance_type = "t3.micro"
      replicas      = 1
    }
    staging = {
      instance_type = "t3.small"
      replicas      = 2
    }
    prod = {
      instance_type = "m5.large"
      replicas      = 3
    }
  }

  config = local.env_config[terraform.workspace]
}

resource "aws_instance" "app" {
  count         = local.config.replicas
  instance_type = local.config.instance_type
  # ...
}
```

```bash
terraform workspace new dev    && terraform apply
terraform workspace new staging && terraform apply
terraform workspace new prod   && terraform apply
```

**Pros:** Simple, single codebase, fast context switching.
**Cons:** Shared backend, no provider isolation, easy mistakes.

---

## 2.3 Strategy 2: Directory per Environment

```
project/
├── modules/                    ← Shared modules
│   ├── compute/
│   ├── networking/
│   └── database/
│
├── environments/
│   ├── dev/
│   │   ├── main.tf             ← Calls shared modules
│   │   ├── variables.tf
│   │   ├── terraform.tfvars    ← Dev-specific values
│   │   ├── backend.tf          ← Dev backend config
│   │   └── provider.tf         ← Dev provider (account/region)
│   │
│   ├── staging/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── terraform.tfvars
│   │   ├── backend.tf
│   │   └── provider.tf
│   │
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       ├── terraform.tfvars
│       ├── backend.tf
│       └── provider.tf
│
└── README.md
```

```hcl
# ─── environments/dev/main.tf ────────────────────
module "networking" {
  source = "../../modules/networking"

  environment = "dev"
  cidr_block  = var.cidr_block
}

module "compute" {
  source = "../../modules/compute"

  environment   = "dev"
  instance_type = var.instance_type
  vpc_id        = module.networking.vpc_id
  subnet_ids    = module.networking.private_subnet_ids
}

# ─── environments/dev/backend.tf ─────────────────
terraform {
  backend "s3" {
    bucket         = "company-tfstate-dev"
    key            = "app/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks-dev"
  }
}

# ─── environments/dev/provider.tf ────────────────
provider "aws" {
  region = "us-east-1"
  assume_role {
    role_arn = "arn:aws:iam::111111111111:role/TerraformDev"
  }
}

# ─── environments/dev/terraform.tfvars ───────────
cidr_block    = "10.0.0.0/16"
instance_type = "t3.micro"
```

```bash
# Deploy dev
cd environments/dev && terraform init && terraform apply

# Deploy prod (completely separate)
cd environments/prod && terraform init && terraform apply
```

**Pros:** Full isolation, separate backends/accounts, safe.
**Cons:** Some code duplication (main.tf in each env).

---

## 2.4 Strategy 3: Variable Files

```
project/
├── main.tf
├── variables.tf
├── outputs.tf
├── envs/
│   ├── dev.tfvars
│   ├── staging.tfvars
│   └── prod.tfvars
└── backends/
    ├── dev.hcl
    ├── staging.hcl
    └── prod.hcl
```

```hcl
# ─── main.tf (shared) ────────────────────────────
variable "environment" {}
variable "instance_type" {}
variable "replicas" {}

resource "aws_instance" "app" {
  count         = var.replicas
  instance_type = var.instance_type

  tags = {
    Environment = var.environment
  }
}

# ─── envs/dev.tfvars ─────────────────────────────
environment   = "dev"
instance_type = "t3.micro"
replicas      = 1

# ─── envs/prod.tfvars ────────────────────────────
environment   = "prod"
instance_type = "m5.large"
replicas      = 3

# ─── backends/dev.hcl ────────────────────────────
bucket         = "company-tfstate"
key            = "dev/app/terraform.tfstate"
region         = "us-east-1"
dynamodb_table = "terraform-locks"

# ─── backends/prod.hcl ───────────────────────────
bucket         = "company-tfstate"
key            = "prod/app/terraform.tfstate"
region         = "us-east-1"
dynamodb_table = "terraform-locks"
```

```bash
# Initialize with environment-specific backend
terraform init -backend-config=backends/dev.hcl

# Apply with environment-specific variables
terraform apply -var-file=envs/dev.tfvars

# Switch to prod (re-init required!)
terraform init -backend-config=backends/prod.hcl -reconfigure
terraform apply -var-file=envs/prod.tfvars
```

**Pros:** Single codebase, no duplication, flexible.
**Cons:** Must remember `-var-file` and `-backend-config`, risk of wrong combination.

---

## 2.5 Strategy 4: Terragrunt (DRY Wrapper)

```
live/
├── terragrunt.hcl              ← Root config (common settings)
├── dev/
│   ├── env.hcl                 ← Environment variables
│   ├── vpc/
│   │   └── terragrunt.hcl     ← Module config
│   └── app/
│       └── terragrunt.hcl
├── staging/
│   ├── env.hcl
│   ├── vpc/
│   │   └── terragrunt.hcl
│   └── app/
│       └── terragrunt.hcl
└── prod/
    ├── env.hcl
    ├── vpc/
    │   └── terragrunt.hcl
    └── app/
        └── terragrunt.hcl
```

```hcl
# ─── live/terragrunt.hcl (root) ─────────────────
remote_state {
  backend = "s3"
  config = {
    bucket         = "company-tfstate"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}

# ─── live/dev/env.hcl ───────────────────────────
locals {
  environment   = "dev"
  instance_type = "t3.micro"
}

# ─── live/dev/app/terragrunt.hcl ────────────────
include "root" {
  path = find_in_parent_folders()
}

locals {
  env = read_terragrunt_config(find_in_parent_folders("env.hcl"))
}

terraform {
  source = "../../../modules//app"
}

inputs = {
  environment   = local.env.locals.environment
  instance_type = local.env.locals.instance_type
}
```

```bash
# Deploy everything in dev
cd live/dev && terragrunt run-all apply

# Deploy only the app in prod
cd live/prod/app && terragrunt apply
```

**Pros:** DRY, automatic backend generation, dependency ordering.
**Cons:** Extra tool dependency, learning curve.

---

## 2.6 Decision Matrix

```
┌──────────────────────────────────────────────────────────┐
│         CHOOSING AN ENVIRONMENT STRATEGY                  │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Small team, simple project?                              │
│    └──► Workspaces or Var Files                           │
│                                                           │
│  Need separate AWS accounts per env?                      │
│    └──► Directory-per-Environment                         │
│                                                           │
│  Large org, many services, DRY code?                      │
│    └──► Terragrunt                                        │
│                                                           │
│  Enterprise with governance needs?                        │
│    └──► Terraform Cloud / Enterprise                      │
│                                                           │
│  Multiple teams, many environments?                       │
│    └──► Directory-per-Env or Terragrunt                   │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Applied to wrong environment | Wrong var-file/workspace | Add safety checks: validate `terraform.workspace` in code |
| Backend state conflicts | Shared state key across envs | Use unique keys per environment |
| Code drift between env dirs | Manual copy/paste | Use shared modules; keep env dirs minimal |
| Init fails when switching envs | Backend config changed | Use `terraform init -reconfigure` |
| Terragrunt cache issues | Stale `.terragrunt-cache` | Delete cache: `find . -name .terragrunt-cache -exec rm -rf {} +` |

---

## Summary Table

| Strategy | Isolation | DRY Code | Complexity | Best For |
|----------|-----------|----------|------------|----------|
| **Workspaces** | Low | High | Low | Small teams, dev/test |
| **Directories** | High | Medium | Medium | Multi-account setups |
| **Var Files** | Medium | High | Medium | Single-account, multi-env |
| **Terragrunt** | High | Very High | High | Large orgs, many services |
| **TF Cloud** | High | High | Medium | Enterprise governance |

---

## Quick Revision Questions

1. **What are the main strategies for environment management?**
   > Workspaces, directory-per-environment, variable files, Terragrunt, and Terraform Cloud workspaces.

2. **Which strategy provides the highest isolation?**
   > Directory-per-environment and Terragrunt, as they support separate backends, providers, and even AWS accounts per environment.

3. **What is the risk of using workspaces for production environments?**
   > Shared backend and provider configuration means all environments are in the same account. Accidentally selecting the wrong workspace can affect the wrong environment.

4. **How does the var-file approach work?**
   > You maintain one codebase and pass environment-specific values via `-var-file=envs/dev.tfvars` along with `-backend-config=backends/dev.hcl` for separate state.

5. **What is Terragrunt and why use it?**
   > Terragrunt is a thin wrapper around Terraform that keeps configurations DRY, generates backend config automatically, and manages dependencies between modules.

---

[← Previous: Workspace Basics](01-workspace-basics.md) | [Back to README](../README.md) | [Next: Workspace Strategies →](03-workspace-strategies.md)
