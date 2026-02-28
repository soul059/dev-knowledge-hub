# Chapter 3: Module Inputs and Outputs

[← Previous: Module Structure](02-module-structure.md) | [Back to README](../README.md) | [Next: Public Registry Modules →](04-registry-modules.md)

---

## Overview

Module **inputs** (variables) and **outputs** form the module's public API — the interface through which the calling code communicates with the module. Well-designed inputs and outputs make modules flexible, reusable, and composable.

---

## 3.1 Module Input Flow

```
┌──────────────────────────────────────────────────────────┐
│              MODULE INPUT/OUTPUT FLOW                     │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  Root Module                    Child Module              │
│  ┌──────────────────┐          ┌──────────────────┐      │
│  │ module "vpc" {   │          │ variables.tf     │      │
│  │   source = "..."  │          │                  │      │
│  │                   │  ─────►  │ var.name         │      │
│  │   name = "prod"   │  INPUT   │ var.cidr         │      │
│  │   cidr = "10.0.." │          │ var.azs          │      │
│  │   azs  = [...]    │          │                  │      │
│  │ }                 │          │  main.tf         │      │
│  │                   │          │  (creates VPC,   │      │
│  │ module.vpc.vpc_id │  ◄─────  │   subnets, etc.) │      │
│  │ module.vpc.sub_ids│  OUTPUT  │                  │      │
│  │                   │          │ outputs.tf       │      │
│  └──────────────────┘          │ output "vpc_id"  │      │
│                                 │ output "sub_ids" │      │
│                                 └──────────────────┘      │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.2 Input Variable Best Practices

### Type Constraints

```hcl
# Always specify types
variable "name" {
  type = string                    # Single value
}

variable "instance_count" {
  type    = number
  default = 1                      # Optional with default
}

variable "enable_monitoring" {
  type    = bool
  default = true
}

variable "subnet_cidrs" {
  type = list(string)              # List of strings
}

variable "tags" {
  type    = map(string)            # Map of strings
  default = {}
}

# Complex object type
variable "database_config" {
  type = object({
    engine         = string
    instance_class = string
    storage_gb     = number
    multi_az       = bool
    backup_retention = optional(number, 7)  # Optional with default
  })
}
```

### Validation Rules

```hcl
variable "environment" {
  type = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "cidr_block" {
  type = string

  validation {
    condition     = can(cidrnetmask(var.cidr_block))
    error_message = "Must be a valid CIDR block (e.g., 10.0.0.0/16)."
  }
}

variable "instance_type" {
  type = string

  validation {
    condition     = can(regex("^t3\\.|^m5\\.|^c5\\.", var.instance_type))
    error_message = "Only t3, m5, and c5 instance families are allowed."
  }
}

variable "port" {
  type = number

  validation {
    condition     = var.port >= 1 && var.port <= 65535
    error_message = "Port must be between 1 and 65535."
  }
}
```

### Description and Documentation

```hcl
variable "vpc_cidr" {
  description = <<-EOT
    The CIDR block for the VPC. Must be a /16 or larger.
    Example: "10.0.0.0/16"
  EOT
  type = string

  validation {
    condition     = tonumber(split("/", var.vpc_cidr)[1]) <= 16
    error_message = "VPC CIDR must be /16 or larger."
  }
}
```

---

## 3.3 Output Best Practices

### Always Include Descriptions

```hcl
output "vpc_id" {
  description = "The ID of the created VPC"
  value       = aws_vpc.this.id
}

output "public_subnet_ids" {
  description = "List of public subnet IDs, ordered by availability zone"
  value       = aws_subnet.public[*].id
}
```

### Sensitive Outputs

```hcl
output "db_password" {
  description = "The database master password"
  value       = aws_db_instance.main.password
  sensitive   = true     # Hidden from CLI output
}
```

### Conditional Outputs

```hcl
output "igw_id" {
  description = "Internet Gateway ID (null if not created)"
  value       = var.create_igw ? aws_internet_gateway.this[0].id : null
}

output "nat_gateway_ips" {
  description = "NAT Gateway public IPs (empty if NAT disabled)"
  value       = var.enable_nat ? aws_eip.nat[*].public_ip : []
}
```

### Computed/Formatted Outputs

```hcl
output "vpc_summary" {
  description = "Summary of VPC configuration"
  value = {
    id          = aws_vpc.this.id
    cidr        = aws_vpc.this.cidr_block
    az_count    = length(var.azs)
    subnet_count = length(aws_subnet.public) + length(aws_subnet.private)
  }
}
```

---

## 3.4 Passing Data Between Modules

```hcl
# ─── Root Module ────────────────────────────────
module "networking" {
  source = "./modules/networking"

  name       = "prod"
  cidr_block = "10.0.0.0/16"
  azs        = ["us-east-1a", "us-east-1b"]
}

module "compute" {
  source = "./modules/compute"

  # Pass networking outputs as compute inputs
  vpc_id             = module.networking.vpc_id
  subnet_ids         = module.networking.public_subnet_ids
  security_group_ids = module.networking.security_group_ids

  instance_type = "t3.micro"
  instance_count = 2
}

module "database" {
  source = "./modules/database"

  vpc_id     = module.networking.vpc_id
  subnet_ids = module.networking.private_subnet_ids

  # Explicit dependency: wait for networking
  depends_on = [module.networking]
}

output "app_url" {
  value = "http://${module.compute.load_balancer_dns}"
}
```

```
┌──────────────────────────────────────────────────────────┐
│           DATA FLOW BETWEEN MODULES                       │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────┐    vpc_id      ┌─────────────┐          │
│  │             │──────────────►│             │          │
│  │ networking  │  subnet_ids   │  compute    │          │
│  │  module     │──────────────►│  module     │          │
│  │             │  sg_ids       │             │          │
│  │  VPC        │──────────────►│  EC2, ALB   │          │
│  │  Subnets    │               │             │          │
│  │  SGs        │               └─────────────┘          │
│  │             │                                         │
│  │             │  vpc_id       ┌─────────────┐          │
│  │             │──────────────►│             │          │
│  │             │  priv_sub_ids │  database   │          │
│  │             │──────────────►│  module     │          │
│  │             │               │             │          │
│  └─────────────┘               │  RDS        │          │
│                                 └─────────────┘          │
│                                                           │
└──────────────────────────────────────────────────────────┘
```

---

## 3.5 Common Variable Patterns

### Feature Toggles

```hcl
variable "create_igw" {
  description = "Whether to create an Internet Gateway"
  type        = bool
  default     = true
}

variable "enable_nat" {
  description = "Whether to create NAT Gateways"
  type        = bool
  default     = false
}

variable "enable_monitoring" {
  description = "Whether to create CloudWatch alarms"
  type        = bool
  default     = true
}
```

### Default Tags Pattern

```hcl
variable "tags" {
  description = "Additional tags for all resources"
  type        = map(string)
  default     = {}
}

locals {
  default_tags = {
    ManagedBy   = "terraform"
    Module      = "networking"
    Environment = var.environment
  }

  all_tags = merge(local.default_tags, var.tags)
}
```

### Configuration Object

```hcl
variable "app_config" {
  description = "Application configuration"
  type = object({
    name          = string
    environment   = string
    instance_type = optional(string, "t3.micro")
    min_size      = optional(number, 1)
    max_size      = optional(number, 3)
    enable_https  = optional(bool, true)
  })
}

# Usage
module "app" {
  source = "./modules/app"

  app_config = {
    name        = "myapp"
    environment = "prod"
    max_size    = 10
    # Other fields use defaults
  }
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Missing required argument" | Variable has no default | Pass value in module call or add a default |
| "An argument named X is not expected" | Variable not declared in module | Add it to `variables.tf` |
| Output not accessible | Output not declared | Add to `outputs.tf` |
| "Invalid value for variable" | Validation rule failed | Check validation conditions |
| "Unsupported attribute" | Accessing non-existent output | Check module's `outputs.tf` for correct name |
| Circular dependency between modules | Module A needs output from B and vice versa | Restructure modules or use data sources |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Input Variable** | `variable "name" { type = string }` |
| **Default Value** | `default = "value"` — makes variable optional |
| **Validation** | `validation { condition = ... }` — enforce constraints |
| **Sensitive** | `sensitive = true` — hide from output |
| **Output** | `output "name" { value = resource.attr }` |
| **Module Reference** | `module.<NAME>.<OUTPUT>` |
| **Feature Toggle** | Boolean variable to enable/disable features |
| **Optional Attrs** | `optional(type, default)` — TF 1.3+ |

---

## Quick Revision Questions

1. **What makes a variable required vs optional?**
   > A variable without a `default` value is required — callers must provide it. A variable with a `default` is optional.

2. **How do you enforce valid input values?**
   > Use `validation` blocks with a `condition` expression and `error_message`.

3. **How does data flow between modules?**
   > Module A declares `output` blocks. Module B receives those values as input variables: `var_name = module.A.output_name`.

4. **What does `sensitive = true` do on an output?**
   > It hides the output value from CLI display (`terraform output`, `plan`, `apply`), but it's still stored in state.

5. **What is the `optional()` type modifier?**
   > Introduced in Terraform 1.3, it makes object attributes optional with an optional default: `optional(string, "default")`.

---

[← Previous: Module Structure](02-module-structure.md) | [Back to README](../README.md) | [Next: Public Registry Modules →](04-registry-modules.md)
