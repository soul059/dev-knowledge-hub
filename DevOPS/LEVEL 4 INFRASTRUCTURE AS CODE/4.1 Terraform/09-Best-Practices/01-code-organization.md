# Chapter 1: Code Organization

[← Previous: Unit 8 - Checks and Validation](../08-Advanced-Features/06-checks-and-validation.md) | [Back to README](../README.md) | [Next: Naming Conventions →](02-naming-conventions.md)

---

## Overview

Well-organized Terraform code is easier to maintain, review, and scale. This chapter covers **project structures**, **file layouts**, and **module organization** patterns for teams of all sizes.

---

## 1.1 File Layout Conventions

```
┌──────────────────────────────────────────────────────────┐
│           STANDARD FILE LAYOUT                            │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  project/                                                 │
│  ├── main.tf           ← Primary resources                │
│  ├── variables.tf      ← Input variable declarations      │
│  ├── outputs.tf        ← Output value declarations        │
│  ├── providers.tf      ← Provider configuration           │
│  ├── versions.tf       ← Terraform/provider versions      │
│  ├── terraform.tf      ← Backend configuration            │
│  ├── data.tf           ← Data sources                     │
│  ├── locals.tf         ← Local values                     │
│  ├── terraform.tfvars  ← Default variable values          │
│  ├── dev.tfvars        ← Dev-specific values              │
│  ├── prod.tfvars       ← Prod-specific values             │
│  └── README.md         ← Documentation                    │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── versions.tf ─────────────────────────────────
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# ─── providers.tf ────────────────────────────────
provider "aws" {
  region = var.aws_region

  default_tags {
    tags = local.common_tags
  }
}

# ─── terraform.tf ────────────────────────────────
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "project/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

---

## 1.2 Small Project Structure

```
┌──────────────────────────────────────────────────────────┐
│  SINGLE DIRECTORY (< 10 resources)                        │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  simple-app/                                              │
│  ├── main.tf          # All resources                     │
│  ├── variables.tf     # All variables                     │
│  ├── outputs.tf       # All outputs                       │
│  ├── versions.tf      # Terraform block                   │
│  └── terraform.tfvars # Variable values                   │
│                                                           │
│  ✓ Quick to start                                         │
│  ✓ Everything visible at a glance                         │
│  ✗ Doesn't scale past ~10 resources                       │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── main.tf (small project) ────────────────────
# All resources in one file for simple projects

resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  tags                 = { Name = "${var.project}-vpc" }
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id
  cidr_block = var.public_subnet_cidr
  tags       = { Name = "${var.project}-public" }
}

resource "aws_instance" "app" {
  ami           = var.ami_id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.public.id
  tags          = { Name = "${var.project}-app" }
}
```

---

## 1.3 Medium Project Structure

```
┌──────────────────────────────────────────────────────────┐
│  RESOURCE-GROUPED FILES (10-50 resources)                 │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  medium-app/                                              │
│  ├── main.tf              # Core resources (VPC, etc.)    │
│  ├── compute.tf           # EC2, ASG, Launch Template     │
│  ├── database.tf          # RDS, ElastiCache              │
│  ├── networking.tf        # Subnets, Route Tables, NAT    │
│  ├── security.tf          # Security Groups, IAM          │
│  ├── storage.tf           # S3, EFS                       │
│  ├── loadbalancer.tf      # ALB, Target Groups            │
│  ├── dns.tf               # Route 53                      │
│  ├── monitoring.tf        # CloudWatch                    │
│  ├── variables.tf                                         │
│  ├── outputs.tf                                           │
│  ├── locals.tf                                            │
│  ├── data.tf                                              │
│  ├── versions.tf                                          │
│  ├── terraform.tf                                         │
│  ├── dev.tfvars                                           │
│  ├── staging.tfvars                                       │
│  ├── prod.tfvars                                          │
│  └── README.md                                            │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── networking.tf ───────────────────────────────
resource "aws_subnet" "public" {
  count             = length(var.public_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.public_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name = "${var.project}-public-${count.index + 1}"
    Tier = "public"
  }
}

resource "aws_subnet" "private" {
  count             = length(var.private_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  tags = {
    Name = "${var.project}-private-${count.index + 1}"
    Tier = "private"
  }
}

# ─── security.tf ─────────────────────────────────
resource "aws_security_group" "app" {
  name_prefix = "${var.project}-app-"
  vpc_id      = aws_vpc.main.id

  ingress {
    from_port       = var.app_port
    to_port         = var.app_port
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  lifecycle {
    create_before_destroy = true
  }
}
```

---

## 1.4 Large Project Structure

```
┌──────────────────────────────────────────────────────────┐
│  MODULE-BASED STRUCTURE (50+ resources)                   │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  large-platform/                                          │
│  ├── environments/                                        │
│  │   ├── dev/                                             │
│  │   │   ├── main.tf          # Module calls              │
│  │   │   ├── variables.tf                                 │
│  │   │   ├── outputs.tf                                   │
│  │   │   ├── terraform.tf     # Backend config            │
│  │   │   └── terraform.tfvars                             │
│  │   ├── staging/                                         │
│  │   │   └── ... (same structure)                         │
│  │   └── prod/                                            │
│  │       └── ... (same structure)                         │
│  │                                                        │
│  ├── modules/                                             │
│  │   ├── networking/                                      │
│  │   │   ├── main.tf                                      │
│  │   │   ├── variables.tf                                 │
│  │   │   ├── outputs.tf                                   │
│  │   │   └── README.md                                    │
│  │   ├── compute/                                         │
│  │   │   └── ...                                          │
│  │   ├── database/                                        │
│  │   │   └── ...                                          │
│  │   ├── security/                                        │
│  │   │   └── ...                                          │
│  │   └── monitoring/                                      │
│  │       └── ...                                          │
│  │                                                        │
│  └── README.md                                            │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── environments/prod/main.tf ───────────────────
module "networking" {
  source = "../../modules/networking"

  project            = var.project
  environment        = "prod"
  vpc_cidr           = "10.0.0.0/16"
  availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

module "compute" {
  source = "../../modules/compute"

  project        = var.project
  environment    = "prod"
  vpc_id         = module.networking.vpc_id
  subnet_ids     = module.networking.private_subnet_ids
  instance_type  = "m5.xlarge"
  min_instances  = 3
  max_instances  = 10
}

module "database" {
  source = "../../modules/database"

  project        = var.project
  environment    = "prod"
  vpc_id         = module.networking.vpc_id
  subnet_ids     = module.networking.database_subnet_ids
  instance_class = "db.r5.2xlarge"
  multi_az       = true
}
```

---

## 1.5 Monorepo vs Polyrepo

```
┌──────────────────────────────────────────────────────────┐
│                  MONOREPO vs POLYREPO                      │
├────────────────────────┬─────────────────────────────────┤
│      MONOREPO          │         POLYREPO                │
├────────────────────────┼─────────────────────────────────┤
│                        │                                 │
│  infra/                │  repo: vpc-infra/               │
│  ├── vpc/              │  ├── main.tf                    │
│  │   └── main.tf       │  └── ...                        │
│  ├── eks/              │                                 │
│  │   └── main.tf       │  repo: eks-infra/               │
│  ├── rds/              │  ├── main.tf                    │
│  │   └── main.tf       │  └── ...                        │
│  └── modules/          │                                 │
│      └── shared/       │  repo: terraform-modules/       │
│                        │  ├── networking/                │
│                        │  └── compute/                   │
│                        │                                 │
│ ✓ Easy cross-reference │ ✓ Independent releases          │
│ ✓ Atomic changes       │ ✓ Team ownership                │
│ ✓ Shared modules       │ ✓ Smaller blast radius          │
│ ✗ Large repo           │ ✗ Version management            │
│ ✗ Wide blast radius    │ ✗ Cross-repo coordination       │
│                        │                                 │
└────────────────────────┴─────────────────────────────────┘
```

---

## 1.6 State Isolation Strategies

```
┌──────────────────────────────────────────────────────────┐
│           STATE ISOLATION PATTERNS                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Option A: By Component                                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │ networking/  │  │ compute/    │  │ database/   │      │
│  │ state       │  │ state       │  │ state       │      │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘      │
│         │                │                │              │
│         └───── terraform_remote_state ────┘              │
│                                                           │
│  Option B: By Environment                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │
│  │ dev/        │  │ staging/    │  │ prod/       │      │
│  │ state       │  │ state       │  │ state       │      │
│  └─────────────┘  └─────────────┘  └─────────────┘      │
│                                                           │
│  Option C: By Component AND Environment (recommended)     │
│  ┌─────────────────────────────────────┐                  │
│  │ s3://state-bucket/                  │                  │
│  │   dev/networking/terraform.tfstate  │                  │
│  │   dev/compute/terraform.tfstate     │                  │
│  │   prod/networking/terraform.tfstate │                  │
│  │   prod/compute/terraform.tfstate    │                  │
│  └─────────────────────────────────────┘                  │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```hcl
# ─── Using data source for cross-state references ─
data "terraform_remote_state" "networking" {
  backend = "s3"

  config = {
    bucket = "company-terraform-state"
    key    = "${var.environment}/networking/terraform.tfstate"
    region = "us-east-1"
  }
}

resource "aws_instance" "app" {
  subnet_id = data.terraform_remote_state.networking.outputs.private_subnet_ids[0]
  # ...
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| File too large, hard to navigate | Too many resources in one file | Split by resource type or concern |
| Module tightly coupled to root | Hardcoded values in module | Parameterize with variables |
| State file conflicts in team | Single state for everything | Isolate state by component |
| Circular dependency between components | Poor layering | Use unidirectional data flow |
| Unable to find resource definitions | No naming convention | Adopt consistent file naming |

---

## Summary Table

| Project Size | Structure | Files Per Config | State Design |
|-------------|-----------|-----------------|--------------|
| **Small** (< 10 resources) | Single directory | 3-5 files | Single state |
| **Medium** (10-50 resources) | Resource-grouped files | 10-15 files | Per-environment |
| **Large** (50+ resources) | Modules + environments | 20+ files | Per-component + env |
| **Enterprise** | Polyrepo/Monorepo | Multiple repos | Fully isolated |

---

## Quick Revision Questions

1. **What are the standard Terraform file names?**
   > `main.tf` (resources), `variables.tf` (inputs), `outputs.tf` (outputs), `providers.tf` (provider config), `versions.tf` (version constraints), `terraform.tf` (backend).

2. **When should you split resources into separate files?**
   > When a single file exceeds ~200 lines or contains more than ~10 resources. Group by concern: networking, compute, security, storage, etc.

3. **What is the recommended state isolation strategy?**
   > Isolate by both component AND environment (e.g., `prod/networking/`, `dev/compute/`) for minimal blast radius.

4. **What's the difference between monorepo and polyrepo?**
   > Monorepo keeps all infrastructure in one repository (easy cross-references, atomic changes). Polyrepo uses separate repos per component or team (independent releases, smaller blast radius).

5. **How do you share data between isolated state files?**
   > Use `terraform_remote_state` data source or output to a shared store (SSM Parameter Store, Consul) and read via data sources.

---

[← Previous: Unit 8 - Checks and Validation](../08-Advanced-Features/06-checks-and-validation.md) | [Back to README](../README.md) | [Next: Naming Conventions →](02-naming-conventions.md)
