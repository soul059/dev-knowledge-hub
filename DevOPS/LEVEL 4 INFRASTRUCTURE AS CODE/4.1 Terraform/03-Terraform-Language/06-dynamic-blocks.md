# Chapter 6: Dynamic Blocks

[← Previous: Expressions and Functions](05-expressions-and-functions.md) | [Back to README](../README.md) | [Next: Conditionals and Loops →](07-conditionals-and-loops.md)

---

## Overview

**Dynamic blocks** allow you to generate repeated nested configuration blocks programmatically. They replace copy-pasting similar blocks and make configurations data-driven.

---

## 6.1 The Problem Dynamic Blocks Solve

```
WITHOUT Dynamic Blocks               WITH Dynamic Blocks
┌──────────────────────┐             ┌──────────────────────┐
│ resource "sg" "web" {│             │ resource "sg" "web" {│
│   ingress {          │             │   dynamic "ingress" {│
│     port = 80        │             │     for_each = var.  │
│   }                  │   ──────►   │       ingress_rules  │
│   ingress {          │             │     content {        │
│     port = 443       │             │       port = ingress │
│   }                  │             │         .value.port  │
│   ingress {          │             │     }                │
│     port = 8080      │             │   }                  │
│   }                  │             │ }                    │
│ }                    │             │                      │
│                      │             │ # Rules defined in   │
│ # 3 copies of same   │             │ # variables — DRY!   │
│ # block structure     │             │                      │
└──────────────────────┘             └──────────────────────┘
```

---

## 6.2 Dynamic Block Syntax

```hcl
resource "resource_type" "name" {
  # Static attributes
  name = "example"

  dynamic "<block_label>" {
    for_each = <collection>        # What to iterate over
    iterator = <name>              # Optional: custom iterator name
    content {
      # Block content using <block_label>.value or <name>.value
      attribute = <block_label>.value.<key>
    }
  }
}
```

```
┌─────────────────────────────────────────────────┐
│              DYNAMIC BLOCK ANATOMY              │
├─────────────────────────────────────────────────┤
│                                                  │
│  dynamic "ingress" {                             │
│      │                                           │
│      └── Block type to generate                  │
│                                                  │
│    for_each = var.rules                          │
│        │                                         │
│        └── Collection: list, map, or set         │
│                                                  │
│    iterator = rule  ◄── Optional custom name     │
│                         (default = block label)  │
│                                                  │
│    content {                                     │
│        │                                         │
│        └── Template for each generated block     │
│                                                  │
│      from_port = rule.value.port                 │
│          │          │     │                       │
│          │          │     └── Attribute from item │
│          │          └── Iterator name             │
│          └── Generated block attribute           │
│    }                                             │
│  }                                               │
│                                                  │
└─────────────────────────────────────────────────┘
```

---

## 6.3 Practical Examples

### Security Group Rules

```hcl
variable "ingress_rules" {
  type = list(object({
    port        = number
    protocol    = string
    cidr_blocks = list(string)
    description = string
  }))
  default = [
    {
      port        = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTP"
    },
    {
      port        = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTPS"
    },
    {
      port        = 22
      protocol    = "tcp"
      cidr_blocks = ["10.0.0.0/8"]
      description = "SSH - Internal only"
    }
  ]
}

resource "aws_security_group" "web" {
  name        = "web-sg"
  description = "Web server security group"
  vpc_id      = var.vpc_id

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
      description = ingress.value.description
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### Inline Policies with Dynamic Blocks

```hcl
variable "iam_policies" {
  type = map(object({
    effect    = string
    actions   = list(string)
    resources = list(string)
  }))
}

resource "aws_iam_role" "app" {
  name               = "app-role"
  assume_role_policy = data.aws_iam_policy_document.assume.json

  dynamic "inline_policy" {
    for_each = var.iam_policies
    content {
      name = inline_policy.key   # Map key as policy name
      policy = jsonencode({
        Version = "2012-10-17"
        Statement = [{
          Effect   = inline_policy.value.effect
          Action   = inline_policy.value.actions
          Resource = inline_policy.value.resources
        }]
      })
    }
  }
}
```

### Using Iterator

```hcl
resource "aws_security_group" "example" {
  name = "example"

  dynamic "ingress" {
    for_each = var.service_ports
    iterator = port            # Custom iterator name

    content {
      from_port   = port.value
      to_port     = port.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

### Nested Dynamic Blocks

```hcl
variable "load_balancer_config" {
  type = list(object({
    name     = string
    port     = number
    backends = list(object({
      address = string
      port    = number
    }))
  }))
}

resource "example_resource" "lb" {
  dynamic "frontend" {
    for_each = var.load_balancer_config
    content {
      name = frontend.value.name
      port = frontend.value.port

      dynamic "backend" {
        for_each = frontend.value.backends
        content {
          address = backend.value.address
          port    = backend.value.port
        }
      }
    }
  }
}
```

---

## 6.4 Maps vs Lists with Dynamic Blocks

```hcl
# Using a MAP — access both key and value
variable "tags_map" {
  default = {
    http  = 80
    https = 443
    ssh   = 22
  }
}

dynamic "ingress" {
  for_each = var.tags_map
  content {
    description = ingress.key       # "http", "https", "ssh"
    from_port   = ingress.value     # 80, 443, 22
    to_port     = ingress.value
    protocol    = "tcp"
  }
}

# Using a LIST — index is key, element is value
variable "ports_list" {
  default = [80, 443, 8080]
}

dynamic "ingress" {
  for_each = var.ports_list
  content {
    from_port = ingress.value      # 80, 443, 8080
    to_port   = ingress.value
    protocol  = "tcp"
  }
}
```

---

## 6.5 Conditional Dynamic Blocks

```hcl
variable "enable_logging" {
  type    = bool
  default = true
}

variable "log_bucket" {
  type    = string
  default = "my-log-bucket"
}

resource "aws_s3_bucket" "app" {
  bucket = "my-app-bucket"

  # Conditionally include logging block
  dynamic "logging" {
    for_each = var.enable_logging ? [1] : []
    content {
      target_bucket = var.log_bucket
      target_prefix = "logs/"
    }
  }
}
```

```
┌─────────────────────────────────────────────────┐
│         CONDITIONAL DYNAMIC BLOCK PATTERN       │
├─────────────────────────────────────────────────┤
│                                                  │
│  for_each = condition ? [1] : []                 │
│                │          │    │                  │
│                │          │    └─ Empty = skip    │
│                │          └── Single item = once  │
│                └── Boolean condition              │
│                                                  │
│  TRUE  → for_each = [1]  → block IS generated    │
│  FALSE → for_each = []   → block is SKIPPED      │
│                                                  │
└─────────────────────────────────────────────────┘
```

---

## 6.6 Best Practices

```
┌────────────────────────────────────────────────┐
│         DYNAMIC BLOCKS BEST PRACTICES          │
├────────────────────────────────────────────────┤
│                                                 │
│  ✓ Use for truly repetitive blocks             │
│  ✓ Keep content blocks simple                  │
│  ✓ Use iterator for clarity in nested blocks   │
│  ✓ Prefer maps over lists for named items      │
│  ✓ Use conditional pattern for optional blocks │
│                                                 │
│  ✗ Don't overuse — hurts readability           │
│  ✗ Don't nest more than 2 levels               │
│  ✗ Don't use for single blocks                 │
│  ✗ Don't make complex logic in content block   │
│                                                 │
└────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Unsupported block type" | Block label doesn't match resource schema | Check provider docs for valid nested block names |
| Iterator `.key` is index not name | Using a list instead of map | Switch to map or use `iterator` |
| Dynamic block generates nothing | `for_each` is empty | Verify collection has items; check conditionals |
| Nested dynamic errors | Incorrect iterator reference | Use custom `iterator` names to avoid shadowing |
| Unexpected plan changes | Using list (order-sensitive) | Convert to `toset()` for unordered collections |

---

## Summary Table

| Feature | Syntax | Purpose |
|---------|--------|---------|
| **Basic dynamic** | `dynamic "block" { for_each = list }` | Generate repeated nested blocks |
| **Iterator** | `iterator = name` | Custom iteration variable name |
| **Content** | `content { ... }` | Template for generated blocks |
| **Map iteration** | `block.key` / `block.value` | Access map key and value |
| **List iteration** | `block.key` (index) / `block.value` | Access index and element |
| **Conditional** | `for_each = cond ? [1] : []` | Include/exclude optional blocks |
| **Nested** | Dynamic inside dynamic | Multi-level repetition |

---

## Quick Revision Questions

1. **What is the purpose of a dynamic block?**
   > To programmatically generate repeated nested configuration blocks from a collection, eliminating duplication.

2. **What are the three sub-blocks/arguments inside a dynamic block?**
   > `for_each` (required — the collection to iterate), `content` (required — the block template), and `iterator` (optional — custom iteration variable name).

3. **How do you conditionally include or exclude a nested block?**
   > Use `for_each = condition ? [1] : []` — the block generates once when true, not at all when false.

4. **What is the difference between `.key` and `.value` when using a map with dynamic blocks?**
   > `.key` is the map key (string), `.value` is the map value. With lists, `.key` is the numeric index.

5. **Why should you use `iterator` when nesting dynamic blocks?**
   > To avoid name shadowing — nested dynamics might use the same label, so custom iterator names keep references unambiguous.

---

[← Previous: Expressions and Functions](05-expressions-and-functions.md) | [Back to README](../README.md) | [Next: Conditionals and Loops →](07-conditionals-and-loops.md)
