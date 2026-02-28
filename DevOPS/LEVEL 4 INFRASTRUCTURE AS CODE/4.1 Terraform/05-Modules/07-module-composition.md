# Chapter 7: Module Composition

[← Previous: Module Testing](06-module-testing.md) | [Back to README](../README.md) | [Next: Unit 6 - Workspace Basics →](../06-Workspaces-and-Environments/01-workspace-basics.md)

---

## Overview

**Module composition** is the practice of combining multiple focused modules to build complete infrastructure stacks. Rather than one monolithic module, compose small, single-purpose modules that connect through outputs and inputs.

---

## 7.1 Composition vs Inheritance

```
┌──────────────────────────────────────────────────────────┐
│        MONOLITHIC vs COMPOSED ARCHITECTURE                │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Monolithic Module              Composed Modules          │
│  ┌────────────────┐            ┌──────────┐              │
│  │ module "infra" │            │ module   │              │
│  │  ├── VPC       │            │ "network"│──┐           │
│  │  ├── Subnets   │            └──────────┘  │           │
│  │  ├── EC2       │                          │ outputs   │
│  │  ├── RDS       │            ┌──────────┐  │           │
│  │  ├── ALB       │            │ module   │◄─┘           │
│  │  ├── SGs       │   ──────►  │ "compute"│──┐           │
│  │  ├── IAM       │            └──────────┘  │           │
│  │  ├── DNS       │                          │ outputs   │
│  │  └── CDN       │            ┌──────────┐  │           │
│  │                │            │ module   │◄─┘           │
│  │ 2000+ lines    │            │ "data"   │              │
│  │ Hard to test   │            └──────────┘              │
│  │ Hard to reuse  │                                      │
│  └────────────────┘            Focused, testable,         │
│                                 reusable, maintainable     │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 7.2 Composition Pattern

```hcl
# ─── Root module: main.tf ──────────────────────────

locals {
  project     = "myapp"
  environment = "prod"
  region      = "us-east-1"
}

# Layer 1: Networking
module "networking" {
  source = "./modules/networking"

  name        = local.project
  environment = local.environment
  cidr_block  = "10.0.0.0/16"
  azs         = ["${local.region}a", "${local.region}b", "${local.region}c"]
}

# Layer 2: Security
module "security" {
  source = "./modules/security"

  name        = local.project
  environment = local.environment
  vpc_id      = module.networking.vpc_id
}

# Layer 3: Compute
module "compute" {
  source = "./modules/compute"

  name               = local.project
  environment        = local.environment
  vpc_id             = module.networking.vpc_id
  subnet_ids         = module.networking.private_subnet_ids
  security_group_ids = [module.security.app_sg_id]
  instance_type      = "m5.large"
  min_size           = 3
  max_size           = 10
}

# Layer 4: Database
module "database" {
  source = "./modules/database"

  name               = local.project
  environment        = local.environment
  vpc_id             = module.networking.vpc_id
  subnet_ids         = module.networking.database_subnet_ids
  security_group_ids = [module.security.db_sg_id]
  instance_class     = "db.r6g.large"
}

# Layer 5: Load Balancer
module "load_balancer" {
  source = "./modules/load-balancer"

  name             = local.project
  environment      = local.environment
  vpc_id           = module.networking.vpc_id
  subnet_ids       = module.networking.public_subnet_ids
  security_group_ids = [module.security.alb_sg_id]
  target_group_arn = module.compute.target_group_arn
  certificate_arn  = module.security.certificate_arn
}

# Layer 6: Monitoring
module "monitoring" {
  source = "./modules/monitoring"

  name           = local.project
  environment    = local.environment
  alb_arn        = module.load_balancer.alb_arn
  asg_name       = module.compute.asg_name
  db_instance_id = module.database.instance_id
  alarm_email    = "ops@example.com"
}
```

```
┌──────────────────────────────────────────────────────────┐
│              DEPENDENCY GRAPH                             │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  networking ──┬──► security ──┬──► compute ──┐            │
│               │               │              │            │
│               │               ├──► database  ├──► monitor │
│               │               │              │    ing     │
│               └───────────────┴──► load     ─┘            │
│                                    balancer               │
│                                                           │
│  Terraform automatically resolves this dependency graph   │
│  Resources are created in the correct order               │
│  Parallel creation where dependencies allow               │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 7.3 Data Flow Patterns

### Hub-and-Spoke

```hcl
# Shared module provides outputs consumed by multiple modules
module "shared" {
  source = "./modules/shared"
  # VPC, common tags, etc.
}

module "app_a" {
  source = "./modules/app"
  vpc_id = module.shared.vpc_id
  # ...
}

module "app_b" {
  source = "./modules/app"
  vpc_id = module.shared.vpc_id
  # ...
}
```

### Pipeline (Sequential)

```hcl
# Each module feeds the next
module "step1" { source = "./modules/networking" }

module "step2" {
  source = "./modules/compute"
  vpc_id = module.step1.vpc_id
}

module "step3" {
  source = "./modules/app"
  instance_ids = module.step2.instance_ids
}
```

### Cross-Project Composition (Remote State)

```hcl
# PROJECT A: Provides networking
# (separate terraform project / state)

# PROJECT B: Consumes networking
data "terraform_remote_state" "networking" {
  backend = "s3"
  config = {
    bucket = "tf-state"
    key    = "networking/terraform.tfstate"
    region = "us-east-1"
  }
}

module "compute" {
  source     = "./modules/compute"
  vpc_id     = data.terraform_remote_state.networking.outputs.vpc_id
  subnet_ids = data.terraform_remote_state.networking.outputs.subnet_ids
}
```

---

## 7.4 Module Dependency Management

### Implicit Dependencies

```hcl
# Terraform detects dependencies through references
module "compute" {
  vpc_id = module.networking.vpc_id
  #           └── Creates implicit dependency
}
```

### Explicit Dependencies

```hcl
# When there's no data reference but ordering matters
module "app" {
  source = "./modules/app"
  # ...

  depends_on = [module.networking]
}
```

### Providers Passed to Modules

```hcl
# Pass providers explicitly when needed
provider "aws" {
  alias  = "us_east"
  region = "us-east-1"
}

provider "aws" {
  alias  = "eu_west"
  region = "eu-west-1"
}

module "us_infra" {
  source = "./modules/regional"
  providers = {
    aws = aws.us_east
  }
}

module "eu_infra" {
  source = "./modules/regional"
  providers = {
    aws = aws.eu_west
  }
}
```

---

## 7.5 Scaling with for_each

```hcl
# Deploy the same module stack to multiple environments
variable "environments" {
  default = {
    dev = {
      instance_type = "t3.micro"
      min_size      = 1
    }
    staging = {
      instance_type = "t3.small"
      min_size      = 2
    }
    prod = {
      instance_type = "m5.large"
      min_size      = 3
    }
  }
}

module "app" {
  source   = "./modules/web-app"
  for_each = var.environments

  name          = "myapp"
  environment   = each.key
  instance_type = each.value.instance_type
  min_size      = each.value.min_size
  vpc_id        = module.networking.vpc_id
  subnet_ids    = module.networking.subnet_ids
}

# Access: module.app["prod"].url
output "prod_url" {
  value = module.app["prod"].url
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Circular dependency | Module A → B → A | Restructure; one-way data flow |
| Slow plan/apply | Too many resources in one state | Split into separate root modules |
| Implicit dependency not detected | No reference between modules | Add `depends_on` |
| Module for_each errors | Unknown values at plan time | Use only known values in for_each |
| Provider not available in module | Wrong provider configuration | Pass providers explicitly |

---

## Summary Table

| Pattern | Description | Use Case |
|---------|-------------|----------|
| **Composition** | Combine focused modules | Standard infrastructure stacks |
| **Hub-and-Spoke** | Shared module → multiple consumers | Shared networking for multiple apps |
| **Pipeline** | Sequential module chain | Layered infrastructure |
| **Remote State** | Cross-project references | Multi-team / multi-project |
| **for_each** | Same module, multiple instances | Multi-environment deployments |
| **Provider Passing** | Explicit provider to module | Multi-region / multi-account |

---

## Quick Revision Questions

1. **What is module composition?**
   > Combining multiple focused, single-purpose modules to build complete infrastructure stacks, connected through outputs and inputs.

2. **Why is composition preferred over monolithic modules?**
   > Composed modules are easier to test, reuse, maintain, and understand. Each module has clear boundaries and responsibilities.

3. **How does Terraform resolve module dependencies?**
   > Automatically through implicit dependencies (references between modules). Use `depends_on` when there's no reference but ordering matters.

4. **How do you deploy the same module to multiple environments?**
   > Use `for_each` on the module block with a map of environment configurations.

5. **How do modules in different projects share data?**
   > Via `data "terraform_remote_state"` to read outputs from another project's state file.

---

[← Previous: Module Testing](06-module-testing.md) | [Back to README](../README.md) | [Next: Unit 6 - Workspace Basics →](../06-Workspaces-and-Environments/01-workspace-basics.md)
