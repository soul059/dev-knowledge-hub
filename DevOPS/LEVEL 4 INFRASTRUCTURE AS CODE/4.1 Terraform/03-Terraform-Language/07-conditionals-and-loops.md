# Chapter 7: Conditionals and Loops

[← Previous: Dynamic Blocks](06-dynamic-blocks.md) | [Back to README](../README.md) | [Next: Unit 4 - State File Basics →](../04-State-Management/01-state-file-basics.md)

---

## Overview

Terraform provides several mechanisms for **conditionals** (making decisions) and **loops** (creating multiple resources). Mastering `count`, `for_each`, and `for` expressions is essential for writing flexible, DRY configurations.

---

## 7.1 Conditional Expressions

### Ternary Syntax

```hcl
# condition ? true_value : false_value

variable "environment" {
  type    = string
  default = "prod"
}

resource "aws_instance" "web" {
  instance_type = var.environment == "prod" ? "m5.large" : "t3.micro"
  
  # Conditional tags
  tags = {
    Name    = "web-${var.environment}"
    Backup  = var.environment == "prod" ? "daily" : "none"
  }
}
```

```
┌──────────────────────────────────────────────────┐
│           CONDITIONAL EXPRESSION FLOW            │
├──────────────────────────────────────────────────┤
│                                                   │
│   var.environment == "prod"                       │
│           │                                       │
│       ┌───┴───┐                                   │
│       │       │                                   │
│     TRUE    FALSE                                 │
│       │       │                                   │
│  "m5.large" "t3.micro"                            │
│       │       │                                   │
│       └───┬───┘                                   │
│           │                                       │
│    instance_type = <result>                       │
│                                                   │
└──────────────────────────────────────────────────┘
```

### Conditional Resource Creation

```hcl
variable "create_dns" {
  type    = bool
  default = true
}

# Create resource only if condition is true
resource "aws_route53_record" "web" {
  count = var.create_dns ? 1 : 0

  zone_id = var.zone_id
  name    = "web.example.com"
  type    = "A"
  records = [aws_instance.web.public_ip]
  ttl     = 300
}

# Reference conditional resource (may not exist)
output "dns_name" {
  value = var.create_dns ? aws_route53_record.web[0].name : "N/A"
}
```

### Chained Conditionals

```hcl
locals {
  size = (
    var.environment == "prod" ? "large" :
    var.environment == "staging" ? "medium" :
    "small"
  )
  
  instance_type = (
    local.size == "large"  ? "m5.2xlarge" :
    local.size == "medium" ? "m5.xlarge" :
    "t3.micro"
  )
}
```

---

## 7.2 Count

### Basic Count

```hcl
resource "aws_instance" "web" {
  count = 3

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  tags = {
    Name = "web-${count.index}"   # web-0, web-1, web-2
  }
}

# Access individual instances
output "first_ip" {
  value = aws_instance.web[0].public_ip
}

# Access all instances
output "all_ips" {
  value = aws_instance.web[*].public_ip
}
```

```
┌───────────────────────────────────────────────────┐
│                    COUNT BEHAVIOR                  │
├───────────────────────────────────────────────────┤
│                                                    │
│  count = 3                                         │
│                                                    │
│  Creates indexed resources:                        │
│  ┌─────────────────────────────────────────────┐   │
│  │  aws_instance.web[0]  →  Name: "web-0"     │   │
│  │  aws_instance.web[1]  →  Name: "web-1"     │   │
│  │  aws_instance.web[2]  →  Name: "web-2"     │   │
│  └─────────────────────────────────────────────┘   │
│                                                    │
│  ⚠ Removing web[1] shifts web[2] → web[1]         │
│    causing DESTROY + RECREATE                      │
│                                                    │
└───────────────────────────────────────────────────┘
```

### Count with Lists

```hcl
variable "server_names" {
  type    = list(string)
  default = ["web", "api", "worker"]
}

resource "aws_instance" "servers" {
  count = length(var.server_names)

  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"

  tags = {
    Name = var.server_names[count.index]
  }
}
```

### Count Limitations

```
┌──────────────────────────────────────────────────┐
│                COUNT LIMITATIONS                  │
├──────────────────────────────────────────────────┤
│                                                   │
│  ⚠  Index-based identification                   │
│     Removing item from middle shifts all indexes  │
│                                                   │
│  ⚠  No access to the item itself                 │
│     Only count.index is available                 │
│                                                   │
│  ⚠  count cannot use values from other resources │
│     that are not yet known (plan-time unknown)    │
│                                                   │
│  ✓  Good for: identical resources, on/off toggle │
│  ✗  Bad for: unique resources, maps, sets         │
│                                                   │
└──────────────────────────────────────────────────┘
```

---

## 7.3 For Each

### With Maps

```hcl
variable "instances" {
  type = map(object({
    instance_type = string
    ami           = string
  }))
  default = {
    web = {
      instance_type = "t3.micro"
      ami           = "ami-0c55b159cbfafe1f0"
    }
    api = {
      instance_type = "t3.small"
      ami           = "ami-0c55b159cbfafe1f0"
    }
    worker = {
      instance_type = "m5.large"
      ami           = "ami-0c55b159cbfafe1f0"
    }
  }
}

resource "aws_instance" "servers" {
  for_each = var.instances

  ami           = each.value.ami
  instance_type = each.value.instance_type

  tags = {
    Name = each.key   # "web", "api", "worker"
    Role = each.key
  }
}

# Access by key
output "web_ip" {
  value = aws_instance.servers["web"].public_ip
}

# All IPs
output "all_ips" {
  value = {for k, v in aws_instance.servers : k => v.public_ip}
}
```

```
┌──────────────────────────────────────────────────┐
│                FOR_EACH BEHAVIOR                  │
├──────────────────────────────────────────────────┤
│                                                   │
│  for_each = { web = {...}, api = {...} }          │
│                                                   │
│  Creates keyed resources:                         │
│  ┌─────────────────────────────────────────────┐  │
│  │  aws_instance.servers["web"]    → web       │  │
│  │  aws_instance.servers["api"]    → api       │  │
│  │  aws_instance.servers["worker"] → worker    │  │
│  └─────────────────────────────────────────────┘  │
│                                                   │
│  ✓ Removing "api" only destroys that instance    │
│    Other instances are NOT affected               │
│                                                   │
└──────────────────────────────────────────────────┘
```

### With Sets

```hcl
variable "subnet_ids" {
  type = set(string)
  default = toset(["subnet-abc", "subnet-def", "subnet-ghi"])
}

resource "aws_instance" "multi_az" {
  for_each = var.subnet_ids

  subnet_id     = each.value    # each.key == each.value for sets
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
}
```

### Converting Lists to Sets

```hcl
variable "user_names" {
  type    = list(string)
  default = ["alice", "bob", "charlie"]
}

# for_each requires map or set — convert list to set
resource "aws_iam_user" "users" {
  for_each = toset(var.user_names)
  name     = each.value
}
```

---

## 7.4 Count vs For Each

```
┌────────────────────────────────────────────────────────────────┐
│              COUNT vs FOR_EACH COMPARISON                      │
├────────────────────────┬───────────────────────────────────────┤
│       COUNT            │         FOR_EACH                      │
├────────────────────────┼───────────────────────────────────────┤
│ Indexed: [0], [1], [2] │ Keyed: ["web"], ["api"], ["worker"] │
│ count.index available  │ each.key + each.value available      │
│ Integer input only     │ Map or set input                     │
│ Order matters          │ Order doesn't matter                 │
│ Shift on delete = bad  │ Targeted delete = safe               │
│ Good for on/off toggle │ Good for unique resources            │
│ Good for identical res │ Good for varied resources            │
│ resource.name[0]       │ resource.name["key"]                 │
└────────────────────────┴───────────────────────────────────────┘
```

### When to Use Each

```hcl
# ✓ COUNT — Toggle creation on/off
resource "aws_cloudwatch_log_group" "app" {
  count = var.enable_logging ? 1 : 0
  name  = "/app/logs"
}

# ✓ COUNT — N identical copies
resource "aws_instance" "worker" {
  count         = var.worker_count
  instance_type = "t3.micro"
}

# ✓ FOR_EACH — Named unique resources
resource "aws_s3_bucket" "buckets" {
  for_each = toset(["logs", "assets", "backups"])
  bucket   = "${var.project}-${each.key}"
}

# ✓ FOR_EACH — Different configurations
resource "aws_instance" "servers" {
  for_each      = var.server_configs   # map of objects
  instance_type = each.value.type
  ami           = each.value.ami
}
```

---

## 7.5 For Expressions

### List Comprehension

```hcl
locals {
  names  = ["alice", "bob", "charlie"]
  
  # Transform list → list
  upper_names = [for name in local.names : upper(name)]
  # → ["ALICE", "BOB", "CHARLIE"]
  
  # Filter
  long_names = [for name in local.names : name if length(name) > 3]
  # → ["alice", "charlie"]
  
  # Transform with index
  numbered = [for i, name in local.names : "${i}: ${name}"]
  # → ["0: alice", "1: bob", "2: charlie"]
}
```

### Map Comprehension

```hcl
locals {
  users = {
    alice   = { role = "admin",  dept = "engineering" }
    bob     = { role = "dev",    dept = "engineering" }
    charlie = { role = "admin",  dept = "operations" }
  }

  # Map → Map transformation
  user_roles = {for name, info in local.users : name => info.role}
  # → {alice = "admin", bob = "dev", charlie = "admin"}

  # Filter to only admins
  admins = {for name, info in local.users : name => info if info.role == "admin"}
  # → {alice = {...}, charlie = {...}}
}
```

### Grouping with For

```hcl
locals {
  users = [
    { name = "alice",   dept = "eng" },
    { name = "bob",     dept = "eng" },
    { name = "charlie", dept = "ops" },
  ]

  # Group by department using ...
  by_dept = {for u in local.users : u.dept => u.name...}
  # → { eng = ["alice", "bob"], ops = ["charlie"] }
}
```

---

## 7.6 Complete Real-World Example

```hcl
# ─── Variables ─────────────────────────────────────
variable "environments" {
  type = map(object({
    instance_type  = string
    instance_count = number
    enable_cdn     = bool
    domain         = string
  }))
  default = {
    dev = {
      instance_type  = "t3.micro"
      instance_count = 1
      enable_cdn     = false
      domain         = "dev.example.com"
    }
    staging = {
      instance_type  = "t3.small"
      instance_count = 2
      enable_cdn     = false
      domain         = "staging.example.com"
    }
    prod = {
      instance_type  = "m5.large"
      instance_count = 3
      enable_cdn     = true
      domain         = "example.com"
    }
  }
}

# ─── Resources ─────────────────────────────────────

# One S3 bucket per environment
resource "aws_s3_bucket" "assets" {
  for_each = var.environments
  bucket   = "${each.key}-assets-${random_id.suffix.hex}"
}

# Multiple instances per environment
resource "aws_instance" "web" {
  for_each = {
    for pair in flatten([
      for env, config in var.environments : [
        for i in range(config.instance_count) : {
          key           = "${env}-${i}"
          env           = env
          instance_type = config.instance_type
          index         = i
        }
      ]
    ]) : pair.key => pair
  }

  ami           = data.aws_ami.ubuntu.id
  instance_type = each.value.instance_type

  tags = {
    Name        = "web-${each.key}"
    Environment = each.value.env
  }
}

# CDN only for environments with enable_cdn = true
resource "aws_cloudfront_distribution" "cdn" {
  for_each = {
    for env, config in var.environments : env => config
    if config.enable_cdn
  }

  origin {
    domain_name = aws_s3_bucket.assets[each.key].bucket_regional_domain_name
    origin_id   = each.key
  }

  # ... CDN configuration
}

# ─── Outputs ───────────────────────────────────────
output "web_instances" {
  value = {
    for key, instance in aws_instance.web :
    key => instance.public_ip
  }
}
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "count and for_each are mutually exclusive" | Can't use both on same resource | Choose one approach per resource |
| "for_each requires map or set" | Passed a list directly | Use `toset(list)` to convert |
| Index shift destroys resources | Used count with a list of unique items | Switch to `for_each` with `toset()` |
| "value depends on resource attributes that cannot be determined" | `for_each` value not known at plan time | Use data that's available before apply |
| Duplicate key in for expression | Multiple items map to same key | Use `...` for grouping or ensure unique keys |
| Cannot reference `count.index` and `each.key` | Mixed up meta-argument contexts | Use only the one matching your resource |

---

## Summary Table

| Feature | Syntax | Use Case |
|---------|--------|----------|
| **Conditional** | `condition ? true : false` | Toggle values |
| **count** | `count = N` | N identical copies, on/off toggle |
| **for_each (map)** | `for_each = map` | Unique keyed resources |
| **for_each (set)** | `for_each = toset(list)` | Unique resources from list |
| **for (list)** | `[for x in list : expr]` | Transform/filter lists |
| **for (map)** | `{for k, v in map : k => expr}` | Transform/filter maps |
| **for (group)** | `{for x in list : key => val...}` | Group items by key |
| **Conditional create** | `count = cond ? 1 : 0` | Optional resources |

---

## Quick Revision Questions

1. **What is the syntax for a conditional expression in Terraform?**
   > `condition ? true_value : false_value` — a ternary expression.

2. **Why is `for_each` preferred over `count` for unique resources?**
   > `for_each` uses string keys for identification, so removing one item doesn't shift or affect other resources. `count` uses numeric indexes and removing an item shifts all subsequent indexes.

3. **How do you convert a list to work with `for_each`?**
   > Use `toset(list)` — `for_each` accepts maps or sets, not lists.

4. **What does the `...` symbol do in a for expression?**
   > It enables grouping — values with the same key are collected into a list: `{for x in list : x.key => x.value...}`.

5. **How do you conditionally create a resource?**
   > Use `count = var.condition ? 1 : 0`. When `count = 0`, the resource is not created.

6. **Can you use `count` and `for_each` on the same resource?**
   > No — they are mutually exclusive. Choose one per resource.

---

[← Previous: Dynamic Blocks](06-dynamic-blocks.md) | [Back to README](../README.md) | [Next: Unit 4 - State File Basics →](../04-State-Management/01-state-file-basics.md)
