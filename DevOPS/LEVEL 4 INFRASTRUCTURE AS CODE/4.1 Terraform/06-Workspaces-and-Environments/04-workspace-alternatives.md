# Chapter 4: Workspace Alternatives

[← Previous: Workspace Strategies](03-workspace-strategies.md) | [Back to README](../README.md) | [Next: Unit 7 - Provider Basics →](../07-Providers/01-provider-basics.md)

---

## Overview

CLI workspaces aren't always the best fit. This chapter explores **alternative approaches** to managing multiple environments, including directory structures, Terragrunt, Terraform Cloud, and hybrid strategies.

---

## 4.1 Directory-per-Environment (Deep Dive)

```
┌──────────────────────────────────────────────────────────┐
│      DIRECTORY-PER-ENVIRONMENT ARCHITECTURE               │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  infrastructure/                                          │
│  ├── modules/           ← Shared, version-controlled     │
│  │   ├── networking/                                      │
│  │   ├── compute/                                         │
│  │   ├── database/                                        │
│  │   └── monitoring/                                      │
│  │                                                        │
│  ├── environments/      ← Per-env root modules            │
│  │   ├── dev/           Account: 111111111111             │
│  │   │   ├── main.tf                                      │
│  │   │   ├── backend.tf                                   │
│  │   │   └── dev.auto.tfvars                              │
│  │   ├── staging/       Account: 222222222222             │
│  │   │   ├── main.tf                                      │
│  │   │   ├── backend.tf                                   │
│  │   │   └── staging.auto.tfvars                          │
│  │   └── prod/          Account: 333333333333             │
│  │       ├── main.tf                                      │
│  │       ├── backend.tf                                   │
│  │       └── prod.auto.tfvars                             │
│  │                                                        │
│  └── global/            ← Shared resources (IAM, DNS)     │
│      ├── iam/                                             │
│      └── dns/                                             │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

### Minimizing Duplication with Symlinks

```bash
# Shared files via symlinks (Unix/Mac)
cd environments/dev
ln -s ../../shared/main.tf main.tf
ln -s ../../shared/variables.tf variables.tf
ln -s ../../shared/outputs.tf outputs.tf

# Each env only has unique files:
# backend.tf, provider.tf, <env>.auto.tfvars
```

### Template Approach (Generating Environments)

```hcl
# ─── environments/dev/main.tf (thin wrapper) ─────
module "stack" {
  source = "../../modules/full-stack"

  environment   = "dev"
  instance_type = "t3.micro"
  min_size      = 1
  max_size      = 2
  domain        = "dev.example.com"
  enable_waf    = false
}

# ─── environments/prod/main.tf (same structure) ──
module "stack" {
  source = "../../modules/full-stack"

  environment   = "prod"
  instance_type = "m5.large"
  min_size      = 3
  max_size      = 10
  domain        = "example.com"
  enable_waf    = true
}
```

### Separate Backend per Account

```hcl
# ─── environments/dev/backend.tf ─────────────────
terraform {
  backend "s3" {
    bucket         = "dev-terraform-state"
    key            = "app/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "dev-terraform-locks"
    role_arn       = "arn:aws:iam::111111111111:role/TerraformBackend"
  }
}

# ─── environments/prod/backend.tf ────────────────
terraform {
  backend "s3" {
    bucket         = "prod-terraform-state"
    key            = "app/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "prod-terraform-locks"
    role_arn       = "arn:aws:iam::333333333333:role/TerraformBackend"
  }
}
```

---

## 4.2 Terragrunt for DRY Multi-Environment

```
┌──────────────────────────────────────────────────────────┐
│          TERRAGRUNT ARCHITECTURE                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  modules/                    ← Pure Terraform modules    │
│  ├── vpc/                                                │
│  ├── ecs/                                                │
│  └── rds/                                                │
│                                                           │
│  live/                       ← Terragrunt configs        │
│  ├── terragrunt.hcl          ← Root (global settings)    │
│  ├── _env/                                               │
│  │   └── common.hcl          ← Common to all envs       │
│  ├── dev/                                                │
│  │   ├── env.hcl             ← Dev settings              │
│  │   ├── vpc/                                            │
│  │   │   └── terragrunt.hcl                              │
│  │   ├── ecs/                                            │
│  │   │   └── terragrunt.hcl                              │
│  │   └── rds/                                            │
│  │       └── terragrunt.hcl                              │
│  └── prod/                                               │
│      ├── env.hcl                                         │
│      ├── vpc/                                            │
│      │   └── terragrunt.hcl                              │
│      ├── ecs/                                            │
│      │   └── terragrunt.hcl                              │
│      └── rds/                                            │
│          └── terragrunt.hcl                              │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── live/terragrunt.hcl (root) ─────────────────
remote_state {
  backend = "s3"
  generate = {
    path      = "backend.tf"
    if_exists = "overwrite_terragrunt"
  }
  config = {
    bucket         = "company-terraform-state-${get_aws_account_id()}"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

generate "provider" {
  path      = "provider.tf"
  if_exists = "overwrite_terragrunt"
  contents  = <<EOF
provider "aws" {
  region = "${local.region}"
}
EOF
}

locals {
  region = "us-east-1"
}

# ─── live/dev/env.hcl ───────────────────────────
locals {
  environment   = "dev"
  account_id    = "111111111111"
  instance_type = "t3.micro"
}

# ─── live/dev/vpc/terragrunt.hcl ────────────────
include "root" {
  path = find_in_parent_folders()
}

locals {
  env = read_terragrunt_config(find_in_parent_folders("env.hcl"))
}

terraform {
  source = "../../../modules//vpc"
}

inputs = {
  environment = local.env.locals.environment
  cidr_block  = "10.0.0.0/16"
}

# ─── live/dev/ecs/terragrunt.hcl (with dependency) ─
include "root" {
  path = find_in_parent_folders()
}

locals {
  env = read_terragrunt_config(find_in_parent_folders("env.hcl"))
}

terraform {
  source = "../../../modules//ecs"
}

dependency "vpc" {
  config_path = "../vpc"
}

inputs = {
  environment   = local.env.locals.environment
  vpc_id        = dependency.vpc.outputs.vpc_id
  subnet_ids    = dependency.vpc.outputs.private_subnet_ids
  instance_type = local.env.locals.instance_type
}
```

```bash
# Deploy all resources in dev (respects dependency order)
cd live/dev
terragrunt run-all plan
terragrunt run-all apply

# Deploy specific component
cd live/dev/ecs
terragrunt apply
```

---

## 4.3 Terraform Cloud Workspaces

```hcl
# ─── Terraform Cloud configuration ───────────────
terraform {
  cloud {
    organization = "my-company"

    workspaces {
      # Option 1: Single named workspace
      name = "myapp-prod"

      # Option 2: Tag-based (multiple workspaces)
      # tags = ["app:myapp"]
    }
  }
}
```

### Key Features

```
┌──────────────────────────────────────────────────────────┐
│       TERRAFORM CLOUD WORKSPACE FEATURES                  │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Per-Workspace Variables                                  │
│  ├── Environment variables (AWS_ACCESS_KEY_ID, etc.)     │
│  ├── Terraform variables (instance_type, etc.)           │
│  └── Sensitive variables (encrypted at rest)             │
│                                                           │
│  VCS Integration                                          │
│  ├── Auto-plan on PR                                     │
│  ├── Auto-apply on merge to main                         │
│  └── Different branches → different workspaces           │
│                                                           │
│  Access Control                                           │
│  ├── Organization-level teams                            │
│  ├── Workspace-level permissions                         │
│  │   ├── Read (view state, plans)                        │
│  │   ├── Plan (trigger plans)                            │
│  │   ├── Write (approve applies)                         │
│  │   └── Admin (configure workspace)                     │
│  └── SSO integration                                     │
│                                                           │
│  Policy as Code (Sentinel / OPA)                          │
│  ├── Enforce tagging standards                           │
│  ├── Restrict instance types                             │
│  ├── Require encryption                                  │
│  └── Cost estimation guardrails                          │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.4 Hybrid Approaches

### Workspace + Directory Hybrid

```
infrastructure/
├── networking/              ← Its own state/workspace
│   ├── main.tf
│   └── terraform.tfvars
│
├── platform/                ← Its own state/workspace
│   ├── main.tf
│   └── terraform.tfvars
│
└── applications/
    ├── app-a/               ← Uses CLI workspaces for envs
    │   ├── main.tf          ← workspace: dev, staging, prod
    │   └── config.tf
    │
    └── app-b/               ← Uses CLI workspaces for envs
        ├── main.tf          ← workspace: dev, staging, prod
        └── config.tf
```

```
┌──────────────────────────────────────────────────────────┐
│           STATE SEPARATION STRATEGY                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│           networking (single state)                       │
│                  │                                        │
│                  ▼                                        │
│           platform (single state)                         │
│                  │                                        │
│        ┌─────────┴─────────┐                             │
│        ▼                   ▼                              │
│     app-a                app-b                            │
│  ┌────┼────┐          ┌────┼────┐                        │
│  │    │    │          │    │    │                         │
│  dev stg prod       dev stg prod                         │
│  (ws) (ws) (ws)     (ws) (ws) (ws)                      │
│                                                           │
│  Shared infra: directory separation                       │
│  App environments: workspace separation                   │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 4.5 Comparison Summary

```
┌──────────────────────────────────────────────────────────────────┐
│                    COMPLETE COMPARISON                             │
├──────────┬───────────┬──────────┬───────────┬────────────────────┤
│ Feature  │ Workspace │ Dir/Env  │ Terragrunt│ TF Cloud           │
├──────────┼───────────┼──────────┼───────────┼────────────────────┤
│ Isolation│ Low       │ High     │ High      │ High               │
│ DRY Code │ High      │ Medium   │ Very High │ High               │
│ RBAC     │ None      │ Via IAM  │ Via IAM   │ Built-in           │
│ Audit    │ None      │ Via Cloud│ Via Cloud │ Built-in           │
│ Multi-   │ No        │ Yes      │ Yes       │ Yes                │
│ account  │           │          │           │                    │
│ Cost     │ Free      │ Free     │ Free      │ Free tier + paid   │
│ Learning │ Low       │ Low      │ Medium    │ Medium             │
│ CI/CD    │ OK        │ Good     │ Good      │ Built-in           │
│ Scale    │ Small     │ Medium   │ Large     │ Enterprise         │
└──────────┴───────────┴──────────┴───────────┴────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Code duplication across env dirs | Not using shared modules | Create thin env wrappers calling shared modules |
| Terragrunt cache conflicts | Stale downloaded modules | `terragrunt run-all init --terragrunt-no-auto-init` |
| TF Cloud rate limits | Too many concurrent runs | Queue runs, use run triggers |
| Symlinks not working on Windows | OS limitation | Use module references instead of symlinks |
| Cross-state references broken | State key/bucket changed | Update `terraform_remote_state` data sources |

---

## Summary Table

| Alternative | Best For | Key Advantage |
|-------------|----------|---------------|
| **Directory-per-env** | Multi-account, security boundaries | Full isolation |
| **Terragrunt** | Large orgs, many services | DRY + dependency management |
| **TF Cloud** | Enterprise governance | RBAC, audit, policy-as-code |
| **Hybrid** | Complex orgs | Right tool for each layer |
| **Var files** | Simple multi-env | Minimal setup |

---

## Quick Revision Questions

1. **What is the directory-per-environment approach?**
   > Each environment has its own directory with separate backend and provider configurations, calling shared modules for infrastructure definitions.

2. **How does Terragrunt achieve DRY code?**
   > Through `include` blocks and `find_in_parent_folders()` to inherit common config, plus `dependency` blocks for cross-module references — eliminating repeated backend, provider, and variable configurations.

3. **What is a key advantage of Terraform Cloud over CLI workspaces?**
   > Per-workspace RBAC (role-based access control), allowing different teams to have different permissions on different environments.

4. **When would you use a hybrid approach?**
   > When shared infrastructure (networking, platform) needs directory separation (high isolation) but applications need workspace separation for quick environment switching.

5. **Why might symlinks not work for code sharing between environments?**
   > Symlinks don't work well on Windows and can cause issues with some CI/CD systems. Module references are a more portable alternative.

6. **What does `terragrunt run-all apply` do?**
   > It applies all Terraform configurations in subdirectories, automatically resolving dependencies between them based on `dependency` blocks.

---

[← Previous: Workspace Strategies](03-workspace-strategies.md) | [Back to README](../README.md) | [Next: Unit 7 - Provider Basics →](../07-Providers/01-provider-basics.md)
