# Chapter 1: Variables (Input Variables)

[← Previous: Terraform Workflow](../02-Terraform-Basics/05-terraform-workflow.md) | [Back to README](../README.md) | [Next: Outputs →](02-outputs.md)

---

## Overview

**Input variables** make Terraform configurations flexible and reusable. Instead of hardcoding values, you define variables that can be set differently per environment, making the same code work for dev, staging, and production.

---

## 1.1 Variable Declaration

```hcl
# variables.tf

# Basic string variable
variable "region" {
  description = "AWS region to deploy resources"
  type        = string
  default     = "us-east-1"
}

# Required variable (no default = must be provided)
variable "project_name" {
  description = "Name of the project"
  type        = string
}

# Number variable
variable "instance_count" {
  description = "Number of EC2 instances"
  type        = number
  default     = 1
}

# Boolean variable
variable "enable_monitoring" {
  description = "Enable CloudWatch monitoring"
  type        = bool
  default     = false
}

# List variable
variable "availability_zones" {
  description = "List of AZs"
  type        = list(string)
  default     = ["us-east-1a", "us-east-1b"]
}

# Map variable
variable "tags" {
  description = "Resource tags"
  type        = map(string)
  default = {
    Environment = "dev"
    ManagedBy   = "terraform"
  }
}
```

### Variable Block Anatomy

```
  ┌──────────────────────────────────────────────────────────┐
  │              VARIABLE BLOCK STRUCTURE                    │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │  variable "name" {                                       │
  │    description = "..."     ← Human-readable purpose     │
  │    type        = string    ← Data type constraint       │
  │    default     = "value"   ← Default (if not provided)  │
  │    sensitive   = true      ← Hide from output           │
  │    nullable    = false     ← Can it be null?            │
  │                                                           │
  │    validation {            ← Custom validation          │
  │      condition     = ...                                 │
  │      error_message = "..."                               │
  │    }                                                      │
  │  }                                                        │
  │                                                           │
  │  All attributes are OPTIONAL                             │
  │  If no default → variable is REQUIRED                    │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

---

## 1.2 Setting Variable Values

```
  ┌──────────────────────────────────────────────────────────┐
  │        VARIABLE VALUE PRECEDENCE (highest to lowest)    │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │  1. -var flag              (CLI)      HIGHEST            │
  │  2. -var-file flag         (CLI)                         │
  │  3. *.auto.tfvars          (auto-loaded)                 │
  │  4. terraform.tfvars       (auto-loaded)                 │
  │  5. TF_VAR_name            (environment variable)        │
  │  6. default in variable {} (config)   LOWEST             │
  │  7. Interactive prompt     (if no default, no value)     │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

### Method 1: CLI Flags

```bash
terraform apply -var="region=us-west-2" -var="instance_count=3"
```

### Method 2: Variable Definition Files (.tfvars)

```hcl
# terraform.tfvars (auto-loaded)
region         = "us-east-1"
project_name   = "web-app"
instance_count = 3

# production.tfvars (loaded with -var-file)
region         = "us-east-1"
project_name   = "web-app"
instance_count = 10
```

```bash
# Auto-loaded files (no flag needed):
#   terraform.tfvars
#   terraform.tfvars.json
#   *.auto.tfvars
#   *.auto.tfvars.json

# Manually specified:
terraform apply -var-file="production.tfvars"
```

### Method 3: Environment Variables

```bash
# Prefix with TF_VAR_
export TF_VAR_region="us-west-2"
export TF_VAR_instance_count=5
export TF_VAR_project_name="my-app"

terraform apply  # Variables picked up automatically
```

### Method 4: Interactive Prompt

```bash
$ terraform apply
var.project_name
  Name of the project

  Enter a value: my-app    # ← Manual input
```

---

## 1.3 Variable Validation

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.micro"

  validation {
    condition     = can(regex("^t3\\.", var.instance_type))
    error_message = "Instance type must be from the t3 family."
  }
}

variable "environment" {
  description = "Deployment environment"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

variable "cidr_block" {
  description = "VPC CIDR block"
  type        = string

  validation {
    condition     = can(cidrhost(var.cidr_block, 0))
    error_message = "Must be a valid CIDR block (e.g., 10.0.0.0/16)."
  }
}

variable "port" {
  description = "Application port"
  type        = number

  validation {
    condition     = var.port >= 1 && var.port <= 65535
    error_message = "Port must be between 1 and 65535."
  }
}
```

---

## 1.4 Sensitive Variables

```hcl
# Declare a sensitive variable
variable "db_password" {
  description = "Database password"
  type        = string
  sensitive   = true    # ← Hidden from plan/apply output
}

# Usage
resource "aws_db_instance" "main" {
  password = var.db_password    # Value hidden in output
}
```

```bash
# In plan output:
# + password = (sensitive value)

# Set via environment variable (most common)
export TF_VAR_db_password="SuperSecret123!"
terraform apply

# Or via .tfvars (keep out of Git!)
# secrets.tfvars
# db_password = "SuperSecret123!"
terraform apply -var-file="secrets.tfvars"
```

```
  ┌──────────────────────────────────────────────────────────┐
  │         SENSITIVE VARIABLE LIMITATIONS                   │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │  sensitive = true:                                       │
  │  ✓ Hides value in CLI output (plan/apply)               │
  │  ✓ Marks outputs using it as sensitive too               │
  │  ✗ Value is STILL in state file (in plaintext!)         │
  │  ✗ Does NOT encrypt anything                            │
  │                                                           │
  │  For true secret management:                             │
  │  → Use HashiCorp Vault                                   │
  │  → Use AWS Secrets Manager / Azure Key Vault             │
  │  → Encrypt state file at rest                            │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

---

## 1.5 Complex Variable Types

```hcl
# Object variable
variable "server_config" {
  type = object({
    instance_type = string
    disk_size     = number
    monitoring    = bool
    tags          = map(string)
  })
  default = {
    instance_type = "t3.micro"
    disk_size     = 20
    monitoring    = true
    tags          = { env = "dev" }
  }
}

# Usage
resource "aws_instance" "web" {
  instance_type = var.server_config.instance_type
  monitoring    = var.server_config.monitoring
  tags          = var.server_config.tags
}

# List of objects
variable "users" {
  type = list(object({
    name  = string
    role  = string
    email = string
  }))
  default = [
    { name = "alice", role = "admin", email = "alice@example.com" },
    { name = "bob",   role = "dev",   email = "bob@example.com" },
  ]
}
```

---

## 1.6 Real-World Pattern: Per-Environment Variables

```
  Project Structure:
  
  ├── main.tf
  ├── variables.tf
  ├── outputs.tf
  ├── envs/
  │   ├── dev.tfvars
  │   ├── staging.tfvars
  │   └── prod.tfvars
```

```hcl
# envs/dev.tfvars
environment    = "dev"
instance_type  = "t3.micro"
instance_count = 1
db_instance    = "db.t3.micro"

# envs/prod.tfvars
environment    = "prod"
instance_type  = "t3.large"
instance_count = 5
db_instance    = "db.r5.2xlarge"
```

```bash
# Deploy to dev
terraform apply -var-file="envs/dev.tfvars"

# Deploy to prod
terraform apply -var-file="envs/prod.tfvars"
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "No value for required variable" | Variable has no default and wasn't set | Add `-var`, `-var-file`, env var, or set a default |
| "Invalid value for variable" | Value doesn't match type or validation | Check type constraint and validation rules |
| Sensitive value visible in state | `sensitive` only hides CLI output | Encrypt state file; use Vault for real secrets |
| Variables not loading from .tfvars | File not named correctly | Must be `terraform.tfvars` or `*.auto.tfvars` for auto-load |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Input Variable** | Parameter that makes configuration flexible |
| **type** | Constrains the variable's data type |
| **default** | Fallback value if none provided |
| **description** | Human-readable explanation |
| **sensitive** | Hides value from plan/apply output |
| **validation** | Custom rules with condition + error_message |
| **terraform.tfvars** | Auto-loaded variable definition file |
| **-var-file** | Manually load a specific .tfvars file |
| **TF_VAR_** | Environment variable prefix for variables |
| **Precedence** | CLI flag > .tfvars > env var > default |

---

## Quick Revision Questions

1. **What makes a variable "required" in Terraform?**
   > A variable with no `default` value is required — Terraform will prompt for it or fail if not provided.

2. **In what order does Terraform resolve variable values?**
   > CLI `-var` > `-var-file` > `*.auto.tfvars` > `terraform.tfvars` > `TF_VAR_` env vars > `default` > interactive prompt.

3. **How do you pass a variable value via environment variable?**
   > Prefix the variable name with `TF_VAR_`, e.g., `export TF_VAR_region="us-west-2"`.

4. **What are the limitations of `sensitive = true`?**
   > It only hides the value in CLI output. The value is still stored in plaintext in the state file.

5. **How do you validate that a variable value meets specific criteria?**
   > Use a `validation` block inside the variable with a `condition` expression and `error_message`.

6. **What is the recommended way to manage variables across multiple environments?**
   > Create separate `.tfvars` files per environment (dev.tfvars, prod.tfvars) and use `-var-file` to select the right one.

---

[← Previous: Terraform Workflow](../02-Terraform-Basics/05-terraform-workflow.md) | [Back to README](../README.md) | [Next: Outputs →](02-outputs.md)
