# Chapter 2: Module Structure

[← Previous: Module Basics](01-module-basics.md) | [Back to README](../README.md) | [Next: Module Inputs and Outputs →](03-module-inputs-outputs.md)

---

## Overview

A well-structured module is easy to understand, use, and maintain. This chapter covers the standard file layout, naming conventions, and organizational patterns for Terraform modules.

---

## 2.1 Standard Module Structure

```
┌──────────────────────────────────────────────────────────┐
│              STANDARD MODULE STRUCTURE                     │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  modules/vpc/                                             │
│  ├── main.tf              ← Primary resources             │
│  ├── variables.tf         ← Input variable declarations   │
│  ├── outputs.tf           ← Output value declarations     │
│  ├── versions.tf          ← Provider & TF version reqs    │
│  ├── locals.tf            ← Local values (optional)       │
│  ├── data.tf              ← Data sources (optional)       │
│  ├── README.md            ← Module documentation          │
│  ├── LICENSE              ← License file (if published)   │
│  ├── examples/            ← Usage examples                │
│  │   ├── simple/                                          │
│  │   │   └── main.tf                                      │
│  │   └── complete/                                        │
│  │       └── main.tf                                      │
│  └── tests/               ← Test files (TF 1.6+)         │
│      └── main.tftest.hcl                                  │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 2.2 File Responsibilities

### main.tf

```hcl
# Primary resource definitions
resource "aws_vpc" "this" {
  cidr_block           = var.cidr_block
  enable_dns_support   = true
  enable_dns_hostnames = var.enable_dns_hostnames

  tags = merge(var.tags, {
    Name = var.name
  })
}

resource "aws_internet_gateway" "this" {
  count  = var.create_igw ? 1 : 0
  vpc_id = aws_vpc.this.id

  tags = merge(var.tags, {
    Name = "${var.name}-igw"
  })
}

resource "aws_subnet" "public" {
  count             = length(var.public_subnets)
  vpc_id            = aws_vpc.this.id
  cidr_block        = var.public_subnets[count.index]
  availability_zone = var.azs[count.index]

  tags = merge(var.tags, {
    Name = "${var.name}-public-${var.azs[count.index]}"
    Tier = "Public"
  })
}
```

### variables.tf

```hcl
variable "name" {
  description = "Name prefix for all resources"
  type        = string
}

variable "cidr_block" {
  description = "CIDR block for the VPC"
  type        = string

  validation {
    condition     = can(cidrnetmask(var.cidr_block))
    error_message = "Must be a valid CIDR block."
  }
}

variable "public_subnets" {
  description = "List of public subnet CIDR blocks"
  type        = list(string)
  default     = []
}

variable "azs" {
  description = "Availability zones"
  type        = list(string)
}

variable "enable_dns_hostnames" {
  description = "Enable DNS hostnames in the VPC"
  type        = bool
  default     = true
}

variable "create_igw" {
  description = "Whether to create an Internet Gateway"
  type        = bool
  default     = true
}

variable "tags" {
  description = "Tags to apply to all resources"
  type        = map(string)
  default     = {}
}
```

### outputs.tf

```hcl
output "vpc_id" {
  description = "The ID of the VPC"
  value       = aws_vpc.this.id
}

output "vpc_cidr" {
  description = "The CIDR block of the VPC"
  value       = aws_vpc.this.cidr_block
}

output "public_subnet_ids" {
  description = "List of public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "igw_id" {
  description = "Internet Gateway ID"
  value       = var.create_igw ? aws_internet_gateway.this[0].id : null
}
```

### versions.tf

```hcl
terraform {
  required_version = ">= 1.3"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 5.0"
    }
  }
}
```

### locals.tf

```hcl
locals {
  common_tags = merge(var.tags, {
    ManagedBy = "terraform"
    Module    = "vpc"
  })

  # Calculated values
  max_subnets = pow(2, var.subnet_newbits)
}
```

---

## 2.3 Project Layout Patterns

### Flat Structure (Small Projects)

```
project/
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
└── providers.tf
```

### Module-Based Structure (Medium Projects)

```
project/
├── main.tf                 ← Module calls
├── variables.tf            ← Root variables
├── outputs.tf              ← Root outputs
├── providers.tf            ← Provider config
├── terraform.tfvars        ← Variable values
│
└── modules/
    ├── networking/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    ├── compute/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── database/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### Environment-Based Structure (Large Projects)

```
infrastructure/
├── modules/                    ← Shared modules
│   ├── vpc/
│   ├── ecs-cluster/
│   ├── rds/
│   └── monitoring/
│
├── environments/
│   ├── dev/
│   │   ├── main.tf            ← Calls shared modules
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── backend.tf         ← Dev state config
│   │   └── terraform.tfvars
│   ├── staging/
│   │   ├── main.tf
│   │   ├── backend.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       ├── backend.tf
│       └── terraform.tfvars
│
└── global/                     ← Shared (IAM, DNS)
    ├── iam/
    └── dns/
```

---

## 2.4 Naming Conventions

```
┌──────────────────────────────────────────────────────────┐
│              NAMING CONVENTIONS                           │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Module Names:                                            │
│  ├── terraform-<PROVIDER>-<NAME>  (published modules)     │
│  │   Examples: terraform-aws-vpc, terraform-azure-network│
│  └── <descriptive-name>          (local modules)          │
│      Examples: networking, compute, database              │
│                                                           │
│  Resource Names (inside modules):                         │
│  ├── "this" → When module has one primary resource        │
│  │   resource "aws_vpc" "this" { }                       │
│  ├── Descriptive name when multiple resources             │
│  │   resource "aws_subnet" "public" { }                  │
│  │   resource "aws_subnet" "private" { }                 │
│  └── Avoid redundancy:                                    │
│      ✗ resource "aws_vpc" "vpc" { }    ← redundant      │
│      ✓ resource "aws_vpc" "this" { }   ← clean          │
│                                                           │
│  Variables:                                               │
│  ├── snake_case: instance_type, subnet_ids               │
│  ├── Boolean: enable_*, create_*, is_*                   │
│  └── Lists: *_ids, *_names, *_arns                       │
│                                                           │
│  Files:                                                   │
│  ├── Lower case with underscores or hyphens              │
│  └── Standard names: main.tf, variables.tf, outputs.tf   │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 2.5 Module Documentation

### README.md Template

```markdown
# Terraform VPC Module

Creates a VPC with public and private subnets across multiple AZs.

## Usage

​```hcl
module "vpc" {
  source = "./modules/vpc"

  name         = "production"
  cidr_block   = "10.0.0.0/16"
  azs          = ["us-east-1a", "us-east-1b"]
  public_subnets = ["10.0.1.0/24", "10.0.2.0/24"]

  tags = {
    Environment = "prod"
  }
}
​```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|----------|
| name | Name prefix | string | - | yes |
| cidr_block | VPC CIDR | string | - | yes |
| azs | Availability zones | list(string) | - | yes |
| public_subnets | Public subnet CIDRs | list(string) | [] | no |
| tags | Resource tags | map(string) | {} | no |

## Outputs

| Name | Description |
|------|-------------|
| vpc_id | VPC ID |
| public_subnet_ids | List of public subnet IDs |
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Module not found" | Wrong source path | Verify relative path from root module |
| Too many files — confusing | Over-splitting | Keep standard 3-5 files unless complexity warrants more |
| Circular dependencies | Modules reference each other | Restructure so data flows one direction |
| Module too large | Doing too much | Split into focused modules (single responsibility) |
| Module too small | One resource per module | Combine related resources that change together |

---

## Summary Table

| File | Purpose |
|------|---------|
| `main.tf` | Primary resource definitions |
| `variables.tf` | Input variable declarations |
| `outputs.tf` | Output value declarations |
| `versions.tf` | Provider and Terraform version constraints |
| `locals.tf` | Computed local values |
| `data.tf` | Data source queries |
| `README.md` | Documentation and usage examples |
| `examples/` | Example configurations |

---

## Quick Revision Questions

1. **What are the three essential files in a module?**
   > `main.tf` (resources), `variables.tf` (inputs), and `outputs.tf` (outputs).

2. **Why name a sole resource `"this"` instead of repeating the type name?**
   > To avoid redundancy. `aws_vpc.this` is cleaner than `aws_vpc.vpc`, and it's a convention for modules with one primary resource.

3. **When should you use an environment-based project layout?**
   > For large projects with multiple deployment environments (dev, staging, prod) that share common modules but have different configurations.

4. **What naming convention is used for published modules?**
   > `terraform-<PROVIDER>-<NAME>`, e.g., `terraform-aws-vpc`.

5. **What should a module README include?**
   > Description, usage example, inputs table (name, type, default, required), and outputs table.

---

[← Previous: Module Basics](01-module-basics.md) | [Back to README](../README.md) | [Next: Module Inputs and Outputs →](03-module-inputs-outputs.md)
