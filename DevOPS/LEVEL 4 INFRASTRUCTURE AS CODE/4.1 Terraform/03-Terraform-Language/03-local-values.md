# Chapter 3: Local Values

[← Previous: Outputs](02-outputs.md) | [Back to README](../README.md) | [Next: Data Types →](04-data-types.md)

---

## Overview

**Local values** (locals) assign a name to an expression so it can be reused throughout your configuration. They simplify complex expressions, reduce repetition, and make code more readable — like variables in a programming language, but computed from other values.

---

## 3.1 Locals Declaration

```hcl
# locals.tf

locals {
  # Simple value
  project_name = "web-app"
  
  # Computed from variables
  name_prefix = "${var.project}-${var.environment}"
  
  # Complex computed value
  common_tags = {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "terraform"
    CreatedAt   = timestamp()
  }
  
  # Conditional logic
  instance_type = var.environment == "prod" ? "t3.large" : "t3.micro"
  
  # List computation
  az_names = data.aws_availability_zones.available.names
  az_count = length(local.az_names)
}
```

### Usage

```hcl
resource "aws_instance" "web" {
  instance_type = local.instance_type    # Reference with local.
  tags          = local.common_tags
}

resource "aws_s3_bucket" "data" {
  bucket = "${local.name_prefix}-data"
  tags   = local.common_tags              # Reuse everywhere!
}
```

---

## 3.2 Locals vs Variables vs Outputs

```
  ┌──────────────────────────────────────────────────────────┐
  │           LOCALS vs VARIABLES vs OUTPUTS                 │
  ├──────────────────────────────────────────────────────────┤
  │                                                           │
  │  VARIABLES (var.xxx)                                     │
  │  → Inputs FROM the user/caller                           │
  │  → Set externally (CLI, .tfvars, env vars)               │
  │                                                           │
  │  LOCALS (local.xxx)                                      │
  │  → Internal computations                                 │
  │  → Not settable externally                               │
  │  → Derived from variables, data, or expressions          │
  │                                                           │
  │  OUTPUTS (output "xxx")                                  │
  │  → Results exposed TO the user/caller                    │
  │  → Displayed after apply or read by other modules        │
  │                                                           │
  │  Flow:  var.input → local.computed → resource → output   │
  │                                                           │
  └──────────────────────────────────────────────────────────┘
```

---

## 3.3 Common Patterns

```hcl
locals {
  # Pattern 1: DRY tag management
  common_tags = merge(var.extra_tags, {
    Project     = var.project
    Environment = var.environment
    ManagedBy   = "terraform"
  })

  # Pattern 2: Name construction
  resource_prefix = "${var.company}-${var.project}-${var.environment}"

  # Pattern 3: CIDR calculations
  vpc_cidr    = "10.${var.env_number}.0.0/16"
  subnet_cidrs = [for i in range(3) : cidrsubnet(local.vpc_cidr, 8, i)]

  # Pattern 4: Flattening nested data
  all_rules = flatten([
    for sg_name, sg in var.security_groups : [
      for rule in sg.rules : {
        sg_name   = sg_name
        port      = rule.port
        protocol  = rule.protocol
        cidr      = rule.cidr
      }
    ]
  ])
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "A local value cannot use its own result" | Self-referencing local | Break into two separate locals |
| "Cycle detected" | Locals reference each other circularly | Restructure to eliminate cycles |
| Locals change every apply | Using `timestamp()` or similar | Move dynamic values to data sources or accept the change |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **locals** | Named expressions for internal reuse |
| **Syntax** | `locals { name = expression }` |
| **Reference** | `local.name` |
| **Purpose** | DRY code, simplify expressions, centralize logic |
| **Cannot be set externally** | Unlike variables, locals are computed internally |

---

## Quick Revision Questions

1. **What is the difference between a local value and an input variable?**
   > A variable is set by the user externally. A local is computed internally from expressions, variables, or data sources.

2. **How do you reference a local value?**
   > Using `local.name`, e.g., `local.common_tags`.

3. **Give a practical use case for locals.**
   > Centralizing tags: define `common_tags` once in locals and use `local.common_tags` across all resources.

4. **Can you set a local value from a `.tfvars` file?**
   > No. Locals are computed internally — they cannot be set externally.

5. **What happens if two locals reference each other?**
   > Terraform detects a cycle and throws a "Cycle detected" error.

---

[← Previous: Outputs](02-outputs.md) | [Back to README](../README.md) | [Next: Data Types →](04-data-types.md)
