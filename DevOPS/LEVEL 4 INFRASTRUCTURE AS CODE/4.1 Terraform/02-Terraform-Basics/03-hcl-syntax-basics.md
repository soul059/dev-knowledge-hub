# Chapter 3: HCL Syntax Basics

[← Previous: Provider Configuration](02-provider-configuration.md) | [Back to README](../README.md) | [Next: Resources and Data Sources →](04-resources-and-data-sources.md)

---

## Overview

**HCL (HashiCorp Configuration Language)** is the domain-specific language used to write Terraform configurations. It's designed to be human-readable while remaining machine-parseable. This chapter covers the fundamental syntax rules and patterns.

---

## 3.1 HCL Structure

```
┌──────────────────────────────────────────────────────────────┐
│                    HCL BUILDING BLOCKS                      │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│   ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│   │  Blocks    │  │ Arguments  │  │ Expressions│            │
│   ├────────────┤  ├────────────┤  ├────────────┤            │
│   │ resource   │  │ key = value│  │ References │            │
│   │ variable   │  │ name = "x" │  │ Functions  │            │
│   │ output     │  │ count = 3  │  │ Operators  │            │
│   │ provider   │  │            │  │ Conditionals│           │
│   │ locals     │  │            │  │            │            │
│   │ data       │  │            │  │            │            │
│   │ module     │  │            │  │            │            │
│   └────────────┘  └────────────┘  └────────────┘            │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.2 Block Syntax

Blocks are the core structure of HCL. They have a type, optional labels, and a body.

```hcl
# Block syntax:
# block_type "label1" "label2" {
#   argument = value
# }

# Resource block (2 labels: type and name)
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}

# Variable block (1 label: name)
variable "region" {
  type    = string
  default = "us-east-1"
}

# Provider block (1 label: provider name)
provider "aws" {
  region = "us-east-1"
}

# Terraform block (0 labels)
terraform {
  required_version = ">= 1.0"
}
```

### Block Types Reference

```
  ┌─────────────────────────────────────────────────────────────┐
  │  Block Type  │  Labels           │  Purpose                │
  ├──────────────┼───────────────────┼─────────────────────────┤
  │  terraform   │  (none)           │  Settings, backends     │
  │  provider    │  provider_name    │  Provider config        │
  │  resource    │  type, name       │  Create infrastructure  │
  │  data        │  type, name       │  Read existing data     │
  │  variable    │  name             │  Input parameters       │
  │  output      │  name             │  Return values          │
  │  locals      │  (none)           │  Local computations     │
  │  module      │  name             │  Use a module           │
  └──────────────┴───────────────────┴─────────────────────────┘
```

---

## 3.3 Arguments and Values

### Primitive Types

```hcl
# String
name = "web-server"

# Number
count = 3

# Boolean
enable_monitoring = true

# Heredoc (multi-line string)
user_data = <<-EOF
  #!/bin/bash
  echo "Hello, World!"
  apt-get update
  apt-get install -y nginx
EOF
```

### Collection Types

```hcl
# List (ordered)
availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]

# Map (key-value pairs)
tags = {
  Name        = "web-server"
  Environment = "production"
  Team        = "platform"
}

# Set (unique, unordered)
# Used in certain resource arguments
security_groups = toset(["sg-123", "sg-456"])
```

### Complex Types

```hcl
# Object (structured type)
variable "server_config" {
  type = object({
    name          = string
    instance_type = string
    disk_size     = number
    is_public     = bool
    tags          = map(string)
  })

  default = {
    name          = "web"
    instance_type = "t3.micro"
    disk_size     = 20
    is_public     = true
    tags          = { env = "dev" }
  }
}

# Tuple (fixed-length, mixed types)
variable "mixed_data" {
  type    = tuple([string, number, bool])
  default = ["hello", 42, true]
}
```

---

## 3.4 Comments

```hcl
# Single-line comment (hash)

// Single-line comment (double-slash) — also valid

/*
  Multi-line comment
  Can span multiple lines
  Useful for detailed explanations
*/

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"  # AMI for Ubuntu 22.04
  instance_type = "t3.micro"               // Small instance
  
  /* 
    Temporarily disabled:
    monitoring = true
  */
}
```

---

## 3.5 References and Expressions

```hcl
# Referencing other resources
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "public" {
  vpc_id     = aws_vpc.main.id           # ← Resource reference
  cidr_block = "10.0.1.0/24"
}

# Referencing variables
variable "env" {
  default = "production"
}

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  
  tags = {
    Environment = var.env                  # ← Variable reference
  }
}

# Referencing data sources
data "aws_ami" "ubuntu" {
  most_recent = true
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-amd64-server-*"]
  }
}

resource "aws_instance" "web" {
  ami = data.aws_ami.ubuntu.id             # ← Data source reference
}

# Referencing local values
locals {
  common_tags = {
    Project   = "web-app"
    ManagedBy = "terraform"
  }
}

resource "aws_instance" "web" {
  tags = local.common_tags                 # ← Local value reference
}
```

### Reference Cheat Sheet

```
  ┌──────────────────────────────────────────────────────────┐
  │                REFERENCE PATTERNS                        │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │  Resource attribute:   aws_instance.web.id               │
  │  Variable:             var.region                        │
  │  Local value:          local.common_tags                 │
  │  Data source:          data.aws_ami.ubuntu.id            │
  │  Module output:        module.vpc.vpc_id                 │
  │  Terraform workspace:  terraform.workspace               │
  │  Output (other state): terraform_remote_state.x.outputs  │
  │  Self reference:       self.id (in provisioners only)    │
  │  Each instance:        each.key, each.value              │
  │  Count index:          count.index                       │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

---

## 3.6 String Interpolation

```hcl
# Embed expressions inside strings with ${}
variable "project" {
  default = "myapp"
}

variable "env" {
  default = "prod"
}

resource "aws_s3_bucket" "data" {
  bucket = "${var.project}-${var.env}-data"
  # Result: "myapp-prod-data"
}

resource "aws_instance" "web" {
  tags = {
    Name = "web-${count.index + 1}"
    # Result: "web-1", "web-2", "web-3", etc.
  }
}

# Directive form (for templates)
user_data = <<-EOF
  #!/bin/bash
  echo "Hostname: ${var.project}-server"
  echo "Region: ${var.region}"
EOF

# Escape literal ${}
example = "Use $${var.name} for literal text"
# Result: "Use ${var.name} for literal text"
```

---

## 3.7 Operators

```hcl
# Arithmetic
result = 5 + 3       # 8
result = 10 - 4      # 6
result = 3 * 2       # 6
result = 10 / 3      # 3.333...
result = 10 % 3      # 1 (modulo)
result = -5           # -5 (negation)

# Comparison
equal     = (5 == 5)    # true
not_equal = (5 != 3)    # true
less      = (3 < 5)     # true
greater   = (5 > 3)     # true
lte       = (5 <= 5)    # true
gte       = (5 >= 3)    # true

# Logical
and_result = (true && false)  # false
or_result  = (true || false)  # true
not_result = !true             # false

# Conditional (ternary)
instance_type = var.env == "prod" ? "t3.large" : "t3.micro"
```

---

## 3.8 Meta-arguments

Meta-arguments apply to all resource types:

```hcl
resource "aws_instance" "web" {
  # Regular arguments
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  # META-ARGUMENTS (special, work on any resource):

  # count — Create multiple instances
  count = 3

  # for_each — Create instances from a collection
  # for_each = toset(["web", "api", "worker"])

  # depends_on — Explicit dependency
  depends_on = [aws_vpc.main]

  # provider — Use a specific provider alias
  provider = aws.west

  # lifecycle — Control resource behavior
  lifecycle {
    create_before_destroy = true
    prevent_destroy       = true
    ignore_changes        = [tags]

    # Precondition (Terraform 1.2+)
    precondition {
      condition     = var.instance_type != "t3.nano"
      error_message = "Instance type too small for production!"
    }
  }
}
```

### Meta-Arguments Summary

```
  ┌──────────────────────────────────────────────────────────┐
  │              META-ARGUMENTS OVERVIEW                     │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │  count            Create N identical resources           │
  │  for_each         Create resources from a map/set        │
  │  depends_on       Override auto dependency detection     │
  │  provider         Select a specific provider instance    │
  │  lifecycle         Control create/update/destroy behavior│
  │                                                           │
  │  Lifecycle sub-options:                                  │
  │  ├── create_before_destroy  (avoid downtime)            │
  │  ├── prevent_destroy        (safety guard)              │
  │  ├── ignore_changes         (skip certain attributes)   │
  │  ├── replace_triggered_by   (force replacement)         │
  │  ├── precondition           (validation before apply)   │
  │  └── postcondition          (validation after apply)    │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

---

## 3.9 Terraform Formatting

```bash
# Auto-format all .tf files in current directory
terraform fmt

# Format recursively
terraform fmt -recursive

# Check if files are formatted (CI/CD)
terraform fmt -check

# Show diff of formatting changes
terraform fmt -diff
```

### Formatting Rules

```hcl
# BEFORE (messy)
resource "aws_instance" "web" {
ami="ami-0c55b159cbfafe1f0"
    instance_type  =    "t3.micro"
tags={Name="web"}
}

# AFTER terraform fmt (clean)
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  tags          = { Name = "web" }
}

# Key rules:
# • 2-space indentation
# • Aligned = signs within a block
# • Consistent spacing
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `Error: Invalid reference` | Typo in resource reference | Check `type.name.attribute` format |
| `Error: Unsupported argument` | Wrong argument for resource type | Check provider docs for valid arguments |
| `Error: Missing required argument` | Required field not set | Add the missing argument |
| `Error: Invalid value for variable` | Type mismatch | Ensure value matches declared type |
| Formatting inconsistent across team | Different editor settings | Run `terraform fmt` in CI/CD pipeline |

---

## Summary Table

| Concept | Syntax Example | Description |
|---------|---------------|-------------|
| **Block** | `resource "type" "name" {}` | Container for configuration |
| **Argument** | `key = value` | Setting within a block |
| **String** | `"hello"` | Text value |
| **Number** | `42` | Numeric value |
| **Boolean** | `true` / `false` | Logical value |
| **List** | `["a", "b", "c"]` | Ordered collection |
| **Map** | `{ key = "value" }` | Key-value pairs |
| **Reference** | `aws_instance.web.id` | Reference to another resource |
| **Interpolation** | `"${var.name}-app"` | Embed expressions in strings |
| **Comment** | `# comment` | Code annotation |
| **Heredoc** | `<<-EOF ... EOF` | Multi-line string |
| **Conditional** | `cond ? true_val : false_val` | Ternary expression |
| **Meta-argument** | `count`, `for_each`, `lifecycle` | Special resource arguments |

---

## Quick Revision Questions

1. **What are the three building blocks of HCL?**
   > Blocks (containers), Arguments (key-value settings), and Expressions (values, references, computations).

2. **How do you reference an attribute of another resource?**
   > Using the format `resource_type.resource_name.attribute`, e.g., `aws_vpc.main.id`.

3. **What is string interpolation and how is it used?**
   > Embedding expressions in strings using `${}`, e.g., `"${var.project}-${var.env}"` produces `"myapp-prod"`.

4. **What are meta-arguments? Give three examples.**
   > Special arguments available on all resources. Examples: `count` (create multiple), `for_each` (create from collection), `lifecycle` (control behavior), `depends_on` (explicit dependencies).

5. **What does `terraform fmt` do?**
   > Automatically reformats `.tf` files to follow HCL style conventions (2-space indent, aligned equals signs, consistent spacing).

6. **What is the difference between a list and a map in HCL?**
   > A list is an ordered sequence (`["a", "b"]`), while a map is a set of key-value pairs (`{ name = "web", env = "prod" }`).

---

[← Previous: Provider Configuration](02-provider-configuration.md) | [Back to README](../README.md) | [Next: Resources and Data Sources →](04-resources-and-data-sources.md)
