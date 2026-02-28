# Chapter 4: Data Types

[← Previous: Local Values](03-local-values.md) | [Back to README](../README.md) | [Next: Expressions and Functions →](05-expressions-and-functions.md)

---

## Overview

Terraform has a **rich type system** that constrains variable values and ensures configuration correctness. Understanding data types helps you write safer, more predictable configurations.

---

## 4.1 Type Hierarchy

```
┌──────────────────────────────────────────────────────────────┐
│                TERRAFORM TYPE SYSTEM                         │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│   PRIMITIVE TYPES                                            │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐                 │
│   │  string  │  │  number  │  │   bool   │                 │
│   │ "hello"  │  │  42, 3.14│  │ true/false│                │
│   └──────────┘  └──────────┘  └──────────┘                 │
│                                                               │
│   COLLECTION TYPES                                           │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐                 │
│   │  list()  │  │   map()  │  │   set()  │                 │
│   │ ordered  │  │ key-value│  │ unique   │                 │
│   │ indexed  │  │ pairs    │  │ unordered│                 │
│   └──────────┘  └──────────┘  └──────────┘                 │
│                                                               │
│   STRUCTURAL TYPES                                           │
│   ┌──────────┐  ┌──────────┐                                │
│   │ object() │  │ tuple()  │                                │
│   │ named    │  │ fixed    │                                │
│   │ attrs    │  │ sequence │                                │
│   └──────────┘  └──────────┘                                │
│                                                               │
│   SPECIAL                                                    │
│   ┌──────────┐  ┌──────────┐                                │
│   │   any    │  │   null   │                                │
│   │ (no      │  │ (absence │                                │
│   │ constraint)│ │ of value)│                                │
│   └──────────┘  └──────────┘                                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.2 Primitive Types

```hcl
# STRING — text
variable "name" {
  type    = string
  default = "web-server"       # "hello", "", "10.0.0.0/16"
}

# NUMBER — integer or float
variable "count" {
  type    = number
  default = 3                  # 42, 3.14, -1, 0
}

# BOOL — true or false
variable "enabled" {
  type    = bool
  default = true               # true, false
}
```

### Type Conversion

```hcl
# Terraform auto-converts between compatible types
# "3" (string) → 3 (number) when number is expected
# true (bool) → "true" (string) when string is expected

# Explicit conversion functions
tostring(42)      # → "42"
tonumber("42")    # → 42
tobool("true")    # → true
```

---

## 4.3 Collection Types

### List — Ordered Sequence

```hcl
variable "azs" {
  type    = list(string)
  default = ["us-east-1a", "us-east-1b", "us-east-1c"]
}

# Access by index (0-based)
# var.azs[0] → "us-east-1a"
# var.azs[1] → "us-east-1b"

# List of numbers
variable "ports" {
  type    = list(number)
  default = [80, 443, 8080]
}
```

### Map — Key-Value Pairs

```hcl
variable "instance_types" {
  type = map(string)
  default = {
    dev     = "t3.micro"
    staging = "t3.small"
    prod    = "t3.large"
  }
}

# Access by key
# var.instance_types["dev"] → "t3.micro"
# var.instance_types["prod"] → "t3.large"

# Map of numbers
variable "port_mapping" {
  type = map(number)
  default = {
    http  = 80
    https = 443
    ssh   = 22
  }
}
```

### Set — Unique, Unordered

```hcl
variable "allowed_regions" {
  type    = set(string)
  default = ["us-east-1", "us-west-2", "eu-west-1"]
}

# Sets automatically deduplicate
# toset(["a", "b", "a"]) → ["a", "b"]

# Cannot access by index (unordered)
# Commonly used with for_each
```

---

## 4.4 Structural Types

### Object — Named Attributes

```hcl
variable "database" {
  type = object({
    engine         = string
    version        = string
    instance_class = string
    storage_gb     = number
    multi_az       = bool
    tags           = map(string)
  })
  default = {
    engine         = "postgres"
    version        = "15.4"
    instance_class = "db.t3.micro"
    storage_gb     = 20
    multi_az       = false
    tags           = { env = "dev" }
  }
}

# Access: var.database.engine → "postgres"
```

### Tuple — Fixed-Length Mixed Sequence

```hcl
variable "rule" {
  type    = tuple([string, number, string])
  default = ["tcp", 443, "0.0.0.0/0"]
}

# Access: var.rule[0] → "tcp"
#         var.rule[1] → 443
```

### Optional Attributes (Terraform 1.3+)

```hcl
variable "config" {
  type = object({
    name     = string
    size     = optional(string, "small")    # Optional with default
    enabled  = optional(bool, true)
  })
}

# Can provide partial object:
# config = { name = "web" }
# config.size defaults to "small"
```

---

## 4.5 The `any` Type

```hcl
# 'any' accepts any type — Terraform infers the actual type
variable "metadata" {
  type    = any
  default = "some-string"   # Could be string, number, list, etc.
}

# Common use: map(any) for flexible maps
variable "tags" {
  type    = map(any)
  default = {
    Name  = "web"           # string
    Count = 3               # number (auto-converted to string in tags)
  }
}
```

---

## 4.6 Type Comparison Table

| Type | Syntax | Example | Access |
|------|--------|---------|--------|
| **string** | `string` | `"hello"` | Direct |
| **number** | `number` | `42` | Direct |
| **bool** | `bool` | `true` | Direct |
| **list** | `list(string)` | `["a","b"]` | `var.x[0]` |
| **map** | `map(string)` | `{k="v"}` | `var.x["k"]` |
| **set** | `set(string)` | `["a","b"]` | `for_each` only |
| **object** | `object({...})` | `{name="x"}` | `var.x.name` |
| **tuple** | `tuple([...])` | `["a",1]` | `var.x[0]` |
| **any** | `any` | Any value | Depends on actual type |

---

## Quick Revision Questions

1. **What are the three primitive types in Terraform?**
   > `string`, `number`, and `bool`.

2. **What is the difference between a list and a set?**
   > A list is ordered and allows duplicates (accessed by index). A set is unordered and automatically deduplicates.

3. **How do you access a value in a map variable?**
   > Using key notation: `var.map_name["key"]`.

4. **What does `optional(string, "default")` do in an object type?**
   > It makes the attribute optional and assigns `"default"` if not provided.

5. **When would you use the `any` type?**
   > When the variable needs to accept different types, or when you want flexible maps like `map(any)`.

---

[← Previous: Local Values](03-local-values.md) | [Back to README](../README.md) | [Next: Expressions and Functions →](05-expressions-and-functions.md)
