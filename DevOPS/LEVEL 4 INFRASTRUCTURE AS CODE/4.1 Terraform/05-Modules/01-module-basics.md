# Chapter 1: Module Basics

[← Previous: Unit 4 - Drift Detection](../04-State-Management/07-drift-detection.md) | [Back to README](../README.md) | [Next: Module Structure →](02-module-structure.md)

---

## Overview

**Modules** are the primary way to organize, reuse, and share Terraform configuration. A module is simply a directory of `.tf` files that encapsulates a set of related resources, much like a function in programming.

---

## 1.1 What is a Module?

```
┌──────────────────────────────────────────────────────────┐
│                TERRAFORM MODULES                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Module = A reusable package of Terraform configuration   │
│                                                           │
│  Think of it like:                                        │
│  ├── A function in programming                            │
│  ├── A class in OOP                                       │
│  ├── A package in a library                               │
│  └── A Docker image for infrastructure                    │
│                                                           │
│  Every Terraform project is already a module:             │
│  ├── Root Module → Your main working directory            │
│  └── Child Modules → Modules called by the root           │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

```
┌──────────────────────────────────────────────────────────┐
│              MODULE HIERARCHY                             │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Root Module (your project)                               │
│  ├── main.tf                                              │
│  ├── variables.tf                                         │
│  ├── outputs.tf                                           │
│  │                                                        │
│  ├── module "networking" ────► modules/networking/         │
│  │   └── Child Module          ├── main.tf                │
│  │                             ├── variables.tf           │
│  │                             └── outputs.tf             │
│  │                                                        │
│  ├── module "compute" ──────► modules/compute/            │
│  │   └── Child Module          ├── main.tf                │
│  │                             ├── variables.tf           │
│  │                             └── outputs.tf             │
│  │                                                        │
│  └── module "database" ─────► modules/database/           │
│      └── Child Module          ├── main.tf                │
│                                ├── variables.tf           │
│                                └── outputs.tf             │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.2 Why Use Modules?

```
┌──────────────────────────────────────────────────────────┐
│              MODULE BENEFITS                              │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  1. REUSABILITY                                           │
│     └── Write once, use across projects/environments      │
│                                                           │
│  2. ENCAPSULATION                                         │
│     └── Hide complexity behind a simple interface         │
│                                                           │
│  3. CONSISTENCY                                           │
│     └── Enforce standards across the organization         │
│                                                           │
│  4. ORGANIZATION                                          │
│     └── Break large configs into manageable pieces        │
│                                                           │
│  5. TESTABILITY                                           │
│     └── Test modules independently                        │
│                                                           │
│  6. VERSIONING                                            │
│     └── Pin module versions; controlled upgrades          │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.3 Calling a Module

```hcl
# Root module — main.tf
module "web_server" {
  source = "./modules/ec2-instance"   # Module source path

  # Input variables (arguments)
  instance_type = "t3.micro"
  ami_id        = "ami-0c55b159cbfafe1f0"
  environment   = "prod"
  
  # Pass in dependencies
  subnet_id          = module.networking.public_subnet_id
  security_group_ids = [module.networking.web_sg_id]
}

# Access module outputs
output "web_server_ip" {
  value = module.web_server.public_ip
}
```

```
┌──────────────────────────────────────────────────────────┐
│              MODULE CALL ANATOMY                          │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  module "web_server" {                                    │
│      │                                                    │
│      └── Module name (local identifier)                   │
│                                                           │
│    source = "./modules/ec2-instance"                      │
│        │                                                  │
│        └── Where to find the module code                  │
│                                                           │
│    instance_type = "t3.micro"                             │
│        │                                                  │
│        └── Input variables passed to module               │
│  }                                                        │
│                                                           │
│  module.web_server.public_ip                              │
│      │       │         │                                  │
│      │       │         └── Output name from module        │
│      │       └── Module local name                        │
│      └── Keyword to reference a module                    │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 1.4 Module Sources

```hcl
# Local path
module "vpc" {
  source = "./modules/vpc"
}

# Terraform Registry (public)
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"
}

# GitHub
module "vpc" {
  source = "github.com/myorg/terraform-vpc-module"
}

# GitHub with tag
module "vpc" {
  source = "github.com/myorg/terraform-vpc-module?ref=v1.2.0"
}

# Bitbucket
module "vpc" {
  source = "bitbucket.org/myorg/terraform-vpc-module"
}

# S3 Bucket
module "vpc" {
  source = "s3::https://s3-eu-west-1.amazonaws.com/bucket/vpc.zip"
}

# GCS Bucket
module "vpc" {
  source = "gcs::https://www.googleapis.com/storage/v1/bucket/vpc.zip"
}

# Generic Git
module "vpc" {
  source = "git::https://example.com/vpc-module.git?ref=v1.0.0"
}

# Git SSH
module "vpc" {
  source = "git::ssh://git@github.com/myorg/vpc-module.git"
}
```

```
┌───────────────────────────────────────────────────┐
│            MODULE SOURCE TYPES                     │
├───────────────────────────────────────────────────┤
│                                                    │
│  Local Path  → "./modules/name"                    │
│               Fast, simple, same repo              │
│                                                    │
│  Registry    → "namespace/name/provider"           │
│               Official/community modules           │
│               Versioned, documented                │
│                                                    │
│  GitHub      → "github.com/org/repo"               │
│               Private or public repos              │
│               Pin with ?ref=tag                    │
│                                                    │
│  S3/GCS      → "s3::/gcs:: URL"                   │
│               Cloud-hosted modules                 │
│                                                    │
│  Git         → "git::url"                          │
│               Any Git repository                   │
│                                                    │
└───────────────────────────────────────────────────┘
```

---

## 1.5 Module Versioning

```hcl
# Registry modules support version constraints
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"       # Exact version
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"      # >= 5.0, < 6.0
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = ">= 4.0, < 6.0"  # Range
}

# Git sources use ref for version control
module "vpc" {
  source = "git::https://github.com/org/module.git?ref=v2.1.0"
}
```

---

## 1.6 Module Workflow

```bash
# After adding a new module source:
$ terraform init
# Downloads/installs the module to .terraform/modules/

$ terraform plan
# Plans including module resources

$ terraform apply
# Creates module resources

# Update module version:
# 1. Change version in config
# 2. Run terraform init -upgrade
$ terraform init -upgrade
```

---

## 1.7 Your First Module

```hcl
# ─── modules/ec2-instance/variables.tf ─────────────
variable "instance_type" {
  type    = string
  default = "t3.micro"
}

variable "ami_id" {
  type = string
}

variable "name" {
  type = string
}

variable "environment" {
  type = string
}

# ─── modules/ec2-instance/main.tf ──────────────────
resource "aws_instance" "this" {
  ami           = var.ami_id
  instance_type = var.instance_type

  tags = {
    Name        = var.name
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}

# ─── modules/ec2-instance/outputs.tf ───────────────
output "instance_id" {
  value       = aws_instance.this.id
  description = "The ID of the EC2 instance"
}

output "public_ip" {
  value       = aws_instance.this.public_ip
  description = "The public IP address"
}

output "private_ip" {
  value       = aws_instance.this.private_ip
  description = "The private IP address"
}
```

```hcl
# ─── Root module — main.tf ─────────────────────────
module "web" {
  source = "./modules/ec2-instance"

  name          = "web-server"
  ami_id        = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  environment   = "prod"
}

module "api" {
  source = "./modules/ec2-instance"

  name          = "api-server"
  ami_id        = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.small"
  environment   = "prod"
}

output "web_ip" {
  value = module.web.public_ip
}

output "api_ip" {
  value = module.api.public_ip
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Module not installed" | Didn't run `terraform init` | Run `terraform init` |
| "Module version not found" | Wrong version or registry | Check registry for available versions |
| "Unsupported argument" | Variable not declared in module | Add the variable to the module or remove the argument |
| "Missing required argument" | Required variable not passed | Add the missing variable to the module call |
| Module output not accessible | Output not declared in module | Add `output` block to the child module |
| "Module source has changed" | Source path modified | Run `terraform init -upgrade` |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Module** | Reusable package of Terraform configuration |
| **Root Module** | The main working directory |
| **Child Module** | Module called from the root |
| **Source** | Where module code lives (local, registry, git) |
| **Version** | Pin module versions with `version = "~> X.Y"` |
| **Inputs** | Variables passed to the module |
| **Outputs** | Values exposed by the module |
| **Init** | `terraform init` downloads/installs modules |

---

## Quick Revision Questions

1. **What is a Terraform module?**
   > A module is a reusable directory of `.tf` files that encapsulates related resources. Every Terraform project is a root module.

2. **What is the difference between a root module and a child module?**
   > The root module is the main working directory. Child modules are called from the root (or other modules) using `module` blocks.

3. **Name three module source types.**
   > Local path (`./modules/name`), Terraform Registry (`namespace/name/provider`), and GitHub (`github.com/org/repo`).

4. **How do you access a module's outputs?**
   > Using `module.<NAME>.<OUTPUT_NAME>`, e.g., `module.web.public_ip`.

5. **What command installs modules?**
   > `terraform init` downloads and installs modules. Use `terraform init -upgrade` to update to newer versions.

---

[← Previous: Unit 4 - Drift Detection](../04-State-Management/07-drift-detection.md) | [Back to README](../README.md) | [Next: Module Structure →](02-module-structure.md)
