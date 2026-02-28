# Chapter 5: Expressions and Functions

[← Previous: Data Types](04-data-types.md) | [Back to README](../README.md) | [Next: Dynamic Blocks →](06-dynamic-blocks.md)

---

## Overview

Terraform provides a rich set of **built-in functions** and **expressions** for transforming data, performing calculations, and making decisions within your configuration.

---

## 5.1 Function Categories

```
┌──────────────────────────────────────────────────────────────┐
│               TERRAFORM FUNCTION CATEGORIES                  │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  STRING        NUMERIC       COLLECTION     DATE/TIME        │
│  ├ upper()     ├ max()       ├ length()     ├ timestamp()    │
│  ├ lower()     ├ min()       ├ merge()      ├ formatdate()   │
│  ├ format()    ├ ceil()      ├ concat()     └ timeadd()      │
│  ├ join()      ├ floor()     ├ flatten()                     │
│  ├ split()     ├ abs()       ├ keys()       ENCODING         │
│  ├ replace()   ├ pow()       ├ values()     ├ base64encode() │
│  ├ substr()    └ signum()    ├ lookup()     ├ base64decode() │
│  ├ trim()                    ├ element()    ├ jsonencode()   │
│  ├ regex()     TYPE          ├ contains()   ├ jsondecode()   │
│  └ chomp()     ├ tostring()  ├ sort()       ├ yamlencode()   │
│                ├ tonumber()  ├ distinct()   └ yamldecode()   │
│  FILESYSTEM    ├ tobool()    ├ zipmap()                      │
│  ├ file()      ├ tolist()    ├ range()      HASH/CRYPTO      │
│  ├ fileexists()├ toset()     └ slice()      ├ md5()          │
│  ├ templatefile()tomap()                    ├ sha256()       │
│  └ basename()                IP/NETWORK     └ uuid()         │
│                              ├ cidrsubnet()                   │
│                              ├ cidrhost()                     │
│                              └ cidrnetmask()                  │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.2 Essential Functions

### String Functions

```hcl
# format — Printf-style string formatting
format("Hello, %s! You have %d servers.", "Alice", 5)
# → "Hello, Alice! You have 5 servers."

# join — Combine list into string
join(", ", ["a", "b", "c"])          # → "a, b, c"
join("-", ["web", "prod", "us"])     # → "web-prod-us"

# split — String into list
split(",", "a,b,c")                  # → ["a", "b", "c"]

# upper / lower
upper("hello")                       # → "HELLO"
lower("HELLO")                       # → "hello"

# replace
replace("hello-world", "-", "_")     # → "hello_world"

# trimspace
trimspace("  hello  ")               # → "hello"

# substr(string, offset, length)
substr("hello world", 0, 5)          # → "hello"

# regex
regex("^([a-z]+)-([0-9]+)$", "web-123")  # → ["web", "123"]
```

### Numeric Functions

```hcl
max(5, 12, 9)       # → 12
min(5, 12, 9)       # → 5
ceil(4.2)            # → 5
floor(4.8)           # → 4
abs(-5)              # → 5
pow(2, 10)           # → 1024
```

### Collection Functions

```hcl
# length
length(["a", "b", "c"])             # → 3
length({a = 1, b = 2})              # → 2
length("hello")                      # → 5

# merge — Combine maps (later values win)
merge({a = 1, b = 2}, {b = 3, c = 4})
# → {a = 1, b = 3, c = 4}

# concat — Combine lists
concat(["a", "b"], ["c", "d"])       # → ["a", "b", "c", "d"]

# flatten — Remove nesting
flatten([["a", "b"], ["c"], ["d"]])  # → ["a", "b", "c", "d"]

# keys / values
keys({name = "web", env = "prod"})   # → ["env", "name"]
values({name = "web", env = "prod"}) # → ["prod", "web"]

# lookup — Map lookup with default
lookup({a = 1, b = 2}, "a", 0)       # → 1
lookup({a = 1, b = 2}, "c", 0)       # → 0 (default)

# contains
contains(["a", "b", "c"], "b")      # → true
contains(["a", "b", "c"], "d")      # → false

# element — Circular list indexing
element(["a", "b", "c"], 0)          # → "a"
element(["a", "b", "c"], 4)          # → "b" (wraps around)

# zipmap — Create map from keys and values lists
zipmap(["name", "env"], ["web", "prod"])
# → {name = "web", env = "prod"}

# range — Generate number sequence
range(3)                              # → [0, 1, 2]
range(1, 4)                           # → [1, 2, 3]
range(0, 10, 2)                       # → [0, 2, 4, 6, 8]

# distinct — Remove duplicates
distinct(["a", "b", "a", "c"])       # → ["a", "b", "c"]

# sort
sort(["banana", "apple", "cherry"])  # → ["apple", "banana", "cherry"]

# coalesce — Return first non-null/non-empty
coalesce("", "hello", "world")       # → "hello"
```

### IP/Network Functions

```hcl
# cidrsubnet — Calculate subnet CIDR
cidrsubnet("10.0.0.0/16", 8, 0)     # → "10.0.0.0/24"
cidrsubnet("10.0.0.0/16", 8, 1)     # → "10.0.1.0/24"
cidrsubnet("10.0.0.0/16", 8, 2)     # → "10.0.2.0/24"

# cidrhost — Get specific host IP
cidrhost("10.0.1.0/24", 5)          # → "10.0.1.5"
```

### Encoding Functions

```hcl
# JSON
jsonencode({name = "web", count = 3})
# → "{\"name\":\"web\",\"count\":3}"

jsondecode("{\"name\":\"web\"}")
# → {name = "web"}

# Base64
base64encode("hello")                # → "aGVsbG8="
base64decode("aGVsbG8=")             # → "hello"

# YAML
yamlencode({name = "web", ports = [80, 443]})
```

### Filesystem Functions

```hcl
# Read file contents
file("${path.module}/scripts/init.sh")

# Check if file exists
fileexists("${path.module}/config.json")

# Template file — RECOMMENDED for complex templates
templatefile("${path.module}/user_data.tftpl", {
  hostname = "web-server"
  packages = ["nginx", "curl"]
})
```

```
# user_data.tftpl
#!/bin/bash
hostname ${hostname}
%{ for pkg in packages ~}
apt-get install -y ${pkg}
%{ endfor ~}
```

---

## 5.3 For Expressions

```hcl
# Transform a list
locals {
  names     = ["alice", "bob", "charlie"]
  upper     = [for n in local.names : upper(n)]
  # → ["ALICE", "BOB", "CHARLIE"]
  
  # Filter with if
  long_names = [for n in local.names : upper(n) if length(n) > 3]
  # → ["ALICE", "CHARLIE"]
  
  # List → Map
  name_map = {for n in local.names : n => upper(n)}
  # → {alice = "ALICE", bob = "BOB", charlie = "CHARLIE"}
  
  # Transform a map
  tags = {env = "prod", team = "platform"}
  prefixed_tags = {for k, v in local.tags : "app_${k}" => v}
  # → {app_env = "prod", app_team = "platform"}
}
```

---

## 5.4 Splat Expressions

```hcl
# Shorthand for extracting attributes from lists
resource "aws_instance" "web" {
  count = 3
  # ...
}

# Splat expression
output "all_ips" {
  value = aws_instance.web[*].public_ip
  # Equivalent to: [for i in aws_instance.web : i.public_ip]
}
```

---

## 5.5 Testing Functions with Console

```bash
$ terraform console

> length(["a", "b", "c"])
3

> cidrsubnet("10.0.0.0/16", 8, 1)
"10.0.1.0/24"

> merge({a = 1}, {b = 2})
{
  "a" = 1
  "b" = 2
}

> format("server-%02d", 3)
"server-03"

> exit
```

---

## Summary Table

| Category | Key Functions | Common Use |
|----------|--------------|------------|
| **String** | `format`, `join`, `split`, `replace`, `upper` | Name construction, formatting |
| **Numeric** | `max`, `min`, `ceil`, `floor` | Size calculations |
| **Collection** | `length`, `merge`, `flatten`, `lookup`, `keys` | Tag merging, data transformation |
| **IP/Network** | `cidrsubnet`, `cidrhost` | Subnet calculations |
| **Encoding** | `jsonencode`, `base64encode`, `yamlencode` | Policy documents, user data |
| **Filesystem** | `file`, `templatefile` | Startup scripts, config files |
| **Type** | `tostring`, `tolist`, `toset`, `tomap` | Type conversion |

---

## Quick Revision Questions

1. **How do you combine two maps, with the second overriding conflicting keys?**
   > `merge(map1, map2)` — later maps take precedence on key conflicts.

2. **What function calculates subnet CIDRs from a VPC CIDR?**
   > `cidrsubnet(prefix, newbits, netnum)`, e.g., `cidrsubnet("10.0.0.0/16", 8, 1)` → `"10.0.1.0/24"`.

3. **How do you test functions interactively?**
   > Use `terraform console` — an interactive REPL for evaluating expressions.

4. **What is a splat expression?**
   > `resource[*].attribute` — shorthand for extracting an attribute from all instances of a resource.

5. **How do you render a template file with variables?**
   > `templatefile("path/to/template.tftpl", { var1 = "value1" })`.

---

[← Previous: Data Types](04-data-types.md) | [Back to README](../README.md) | [Next: Dynamic Blocks →](06-dynamic-blocks.md)
