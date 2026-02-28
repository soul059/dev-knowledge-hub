# Chapter 2: Naming Conventions

[← Previous: Code Organization](01-code-organization.md) | [Back to README](../README.md) | [Next: Version Control →](03-version-control.md)

---

## Overview

Consistent naming conventions make Terraform code **readable**, **predictable**, and **maintainable**. This chapter covers naming rules for resources, variables, modules, files, and cloud resources.

---

## 2.1 Terraform Naming Rules

```
┌──────────────────────────────────────────────────────────┐
│           TERRAFORM NAMING CONVENTIONS                    │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────────────────────────────────────┐         │
│  │ General Rules                                │         │
│  ├─────────────────────────────────────────────┤         │
│  │ • Use snake_case for ALL identifiers         │         │
│  │ • Use lowercase only                         │         │
│  │ • Be descriptive but concise                 │         │
│  │ • Use nouns for resources, adjectives for     │         │
│  │   booleans, plural for lists                  │         │
│  │ • Don't repeat the resource type in name      │         │
│  └─────────────────────────────────────────────┘         │
│                                                           │
│  ✓ aws_instance.web_server                                │
│  ✗ aws_instance.webServer          (camelCase)            │
│  ✗ aws_instance.Web-Server         (kebab-case + caps)    │
│  ✗ aws_instance.aws_instance_web   (repeats type)         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 2.2 Resource Naming

```hcl
# ─── GOOD: Descriptive, snake_case names ─────────
resource "aws_vpc" "main" {}
resource "aws_subnet" "public" {}
resource "aws_subnet" "private" {}
resource "aws_security_group" "web" {}
resource "aws_security_group" "database" {}
resource "aws_instance" "bastion" {}
resource "aws_lb" "external" {}
resource "aws_db_instance" "primary" {}
resource "aws_s3_bucket" "logs" {}

# For single resources, use "main" or "this"
resource "aws_vpc" "main" {}     # One VPC, call it "main"
resource "aws_vpc" "this" {}     # Also acceptable

# ─── BAD: Avoid these patterns ───────────────────
resource "aws_vpc" "my_vpc" {}           # Redundant "vpc"
resource "aws_subnet" "subnet1" {}       # Numbered, meaningless
resource "aws_instance" "resource" {}    # Generic
resource "aws_security_group" "sg" {}    # Abbreviated
```

```
┌──────────────────────────────────────────────────────────┐
│           RESOURCE NAMING DECISION                        │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  How many of this resource type?                          │
│                                                           │
│  Only 1 ──→ Use "main" or "this"                         │
│               resource "aws_vpc" "main" {}                │
│                                                           │
│  2-3    ──→ Use role-based names                          │
│               resource "aws_subnet" "public" {}           │
│               resource "aws_subnet" "private" {}          │
│                                                           │
│  Many   ──→ Use for_each/count with descriptive base      │
│               resource "aws_subnet" "public" {            │
│                 for_each = var.public_subnets              │
│               }                                           │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 2.3 Variable Naming

```hcl
# ─── Standard naming patterns ────────────────────

# Simple values: noun describing the value
variable "environment" {}
variable "project_name" {}
variable "aws_region" {}
variable "instance_type" {}

# Booleans: use "enable_" or "is_" prefix
variable "enable_monitoring" {
  type    = bool
  default = true
}
variable "is_public" {
  type    = bool
  default = false
}
variable "enable_deletion_protection" {
  type    = bool
  default = true
}

# Lists: use plural nouns
variable "subnet_ids" {
  type = list(string)
}
variable "availability_zones" {
  type = list(string)
}
variable "allowed_cidr_blocks" {
  type = list(string)
}

# Maps: describe key-value relationship
variable "instance_types" {
  type        = map(string)
  description = "Map of environment to instance type"
}
variable "tags" {
  type    = map(string)
  default = {}
}

# Numbers: include unit when ambiguous
variable "max_instance_count" {
  type = number
}
variable "timeout_seconds" {
  type = number
}
variable "storage_gb" {
  type = number
}
variable "retention_days" {
  type = number
}
```

---

## 2.4 Output Naming

```hcl
# ─── Pattern: {resource_type}_{attribute} ────────

output "vpc_id" {
  value = aws_vpc.main.id
}

output "vpc_cidr_block" {
  value = aws_vpc.main.cidr_block
}

output "public_subnet_ids" {
  value = aws_subnet.public[*].id
}

output "database_endpoint" {
  value = aws_db_instance.primary.endpoint
}

output "load_balancer_dns_name" {
  value = aws_lb.external.dns_name
}

# ─── For modules: describe what you're exposing ──
output "cluster_endpoint" {
  description = "EKS cluster API endpoint"
  value       = aws_eks_cluster.main.endpoint
}

output "cluster_certificate_authority" {
  description = "Base64 encoded cluster CA certificate"
  value       = aws_eks_cluster.main.certificate_authority[0].data
  sensitive   = true
}
```

---

## 2.5 Module Naming

```
┌──────────────────────────────────────────────────────────┐
│           MODULE NAMING PATTERNS                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Module Source Names:                                     │
│  ┌────────────────────────────────────────────┐          │
│  │ terraform-{provider}-{purpose}              │          │
│  │                                             │          │
│  │ terraform-aws-vpc                           │          │
│  │ terraform-aws-eks-cluster                   │          │
│  │ terraform-aws-rds-postgres                  │          │
│  │ terraform-azurerm-virtual-network           │          │
│  └────────────────────────────────────────────┘          │
│                                                           │
│  Module Call Names (in code):                             │
│  ┌────────────────────────────────────────────┐          │
│  │ module "vpc" { ... }                        │          │
│  │ module "eks_cluster" { ... }                │          │
│  │ module "primary_database" { ... }           │          │
│  │ module "app_load_balancer" { ... }          │          │
│  └────────────────────────────────────────────┘          │
│                                                           │
│  ✗ module "Module1" { ... }        (PascalCase)          │
│  ✗ module "my-module" { ... }      (kebab-case)          │
│  ✗ module "mod" { ... }            (abbreviated)         │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 2.6 Cloud Resource Naming (Tags)

```hcl
# ─── Naming pattern for AWS resource tags ────────
# Pattern: {project}-{environment}-{component}-{resource}

locals {
  name_prefix = "${var.project}-${var.environment}"

  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "terraform"
    Team        = var.team
  }
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-vpc"
  })
}

resource "aws_subnet" "public" {
  count  = length(var.public_subnet_cidrs)
  vpc_id = aws_vpc.main.id

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-public-${count.index + 1}"
    Tier = "public"
  })
}

resource "aws_instance" "web" {
  count = var.web_instance_count

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-web-${count.index + 1}"
    Role = "web-server"
  })
}

# ─── S3 bucket naming (globally unique) ──────────
resource "aws_s3_bucket" "logs" {
  bucket = "${var.company}-${var.project}-${var.environment}-logs-${var.aws_region}"
  #        acme-webapp-prod-logs-us-east-1
}
```

```
┌──────────────────────────────────────────────────────────┐
│           CLOUD RESOURCE NAMING                           │
├────────────────┬─────────────────────────────────────────┤
│ Resource       │ Naming Pattern                          │
├────────────────┼─────────────────────────────────────────┤
│ VPC            │ {project}-{env}-vpc                     │
│ Subnet         │ {project}-{env}-{tier}-{az}             │
│ Security Group │ {project}-{env}-{role}-sg               │
│ EC2 Instance   │ {project}-{env}-{role}-{number}         │
│ RDS            │ {project}-{env}-{engine}                │
│ S3 Bucket      │ {company}-{project}-{env}-{purpose}     │
│ IAM Role       │ {project}-{env}-{service}-role          │
│ Lambda         │ {project}-{env}-{function-name}         │
│ CloudWatch     │ /{project}/{env}/{service}               │
└────────────────┴─────────────────────────────────────────┘
```

---

## 2.7 File and Directory Naming

```
┌──────────────────────────────────────────────────────────┐
│           FILE NAMING CONVENTIONS                         │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Terraform Files:                                         │
│  • Use snake_case: security_groups.tf                     │
│  • Name by concern: networking.tf, database.tf            │
│  • Standard names: main.tf, variables.tf, outputs.tf      │
│                                                           │
│  Variable Files:                                          │
│  • Per environment: dev.tfvars, prod.tfvars               │
│  • Auto-loaded: terraform.tfvars, *.auto.tfvars           │
│                                                           │
│  Directories:                                             │
│  • Use kebab-case: my-module/                             │
│  • Or snake_case: my_module/                              │
│  • Be consistent within project                           │
│                                                           │
│  Module Repos:                                            │
│  • terraform-{provider}-{name}                            │
│  • Example: terraform-aws-vpc                             │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Inconsistent naming across team | No convention documented | Create and enforce naming standards doc |
| Name collisions in S3 buckets | Not globally unique | Include account ID, region, or random suffix |
| Confusing resource references | Poor resource names | Use descriptive names based on role/purpose |
| Tag-based policies failing | Inconsistent tag casing | Use `default_tags` in provider for consistency |
| Hard to find resources in AWS console | No Name tag pattern | Always include Name tag with standard prefix |

---

## Summary Table

| Element | Convention | Example |
|---------|-----------|---------|
| **Resource names** | snake_case, role-based | `aws_instance.web_server` |
| **Variables** | snake_case, descriptive | `enable_monitoring`, `subnet_ids` |
| **Outputs** | snake_case, `{type}_{attr}` | `vpc_id`, `cluster_endpoint` |
| **Module calls** | snake_case, purpose | `module "primary_database"` |
| **Module repos** | `terraform-{provider}-{name}` | `terraform-aws-vpc` |
| **Cloud resources** | `{project}-{env}-{role}` | `myapp-prod-web-1` |
| **Files** | snake_case `.tf` | `security_groups.tf` |

---

## Quick Revision Questions

1. **What casing should Terraform identifiers use?**
   > Always `snake_case` — for resources, variables, outputs, locals, and module names.

2. **How should you name a resource when there's only one of its type?**
   > Use `"main"` or `"this"` (e.g., `aws_vpc.main`). Avoid repeating the resource type in the name.

3. **What prefix convention is used for boolean variables?**
   > Use `enable_` or `is_` prefix: `enable_monitoring`, `is_public`, `enable_deletion_protection`.

4. **What naming pattern is standard for module repositories?**
   > `terraform-{provider}-{name}` (e.g., `terraform-aws-vpc`). This is required for Terraform Registry publishing.

5. **How should cloud resources be named via tags?**
   > Use a prefix pattern like `{project}-{environment}-{role}` (e.g., `myapp-prod-web-1`) and apply consistently via `local.name_prefix` and `default_tags`.

---

[← Previous: Code Organization](01-code-organization.md) | [Back to README](../README.md) | [Next: Version Control →](03-version-control.md)
