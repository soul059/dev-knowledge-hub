# Chapter 2: Terraform Functions Deep Dive

[← Previous: Provisioners](01-provisioners.md) | [Back to README](../README.md) | [Next: Lifecycle Rules →](03-lifecycle-rules.md)

---

## Overview

Terraform provides a rich set of **built-in functions** for transforming and combining data. Functions are used in expressions throughout your configuration but **you cannot define custom functions** (though Terraform 1.8+ adds provider-defined functions).

---

## 2.1 Function Categories

```
┌──────────────────────────────────────────────────────────┐
│          TERRAFORM FUNCTION CATEGORIES                    │
├──────────────────────────────────────────────────────────┤
│                                                           │
│  String    │ format, join, split, replace, trim, upper,  │
│            │ lower, title, substr, regex, indent          │
│            │                                              │
│  Numeric   │ abs, ceil, floor, log, max, min, pow,       │
│            │ signum, parseint                             │
│            │                                              │
│  Collection│ concat, contains, distinct, flatten,         │
│            │ keys, values, length, lookup, merge,         │
│            │ range, reverse, sort, zipmap, element        │
│            │                                              │
│  Encoding  │ base64encode, base64decode, jsonencode,     │
│            │ jsondecode, yamlencode, yamldecode,           │
│            │ csvdecode, urlencode                         │
│            │                                              │
│  Filesystem│ file, fileexists, fileset, templatefile,    │
│            │ filebase64, abspath, dirname, basename,      │
│            │ pathexpand                                   │
│            │                                              │
│  Date/Time │ formatdate, timeadd, timecmp, timestamp,    │
│            │ plantimestamp                                │
│            │                                              │
│  Hash/Cryp│ md5, sha1, sha256, sha512, bcrypt,          │
│            │ base64sha256, uuid, uuidv5                   │
│            │                                              │
│  IP Network│ cidrhost, cidrnetmask, cidrsubnet,          │
│            │ cidrsubnets                                  │
│            │                                              │
│  Type Conv │ tostring, tonumber, tobool, tolist, toset,  │
│            │ tomap, try, can, sensitive, nonsensitive      │
│            │                                              │
└──────────────────────────────────────────────────────────┘
```

---

## 2.2 String Functions

```hcl
# ─── format: Printf-style formatting ─────────────
format("Hello, %s! You have %d items.", "Alice", 5)
# "Hello, Alice! You have 5 items."

format("%-20s %s", "Name:", "MyApp")
# "Name:                MyApp"

# ─── join / split ─────────────────────────────────
join(", ", ["a", "b", "c"])            # "a, b, c"
join("-", ["us", "east", "1"])         # "us-east-1"
split(",", "a,b,c")                    # ["a", "b", "c"]

# ─── replace ─────────────────────────────────────
replace("hello world", " ", "-")       # "hello-world"
replace("ami-12345", "/ami-/", "")     # "12345" (regex)

# ─── upper / lower / title ───────────────────────
upper("hello")                         # "HELLO"
lower("Hello World")                   # "hello world"
title("hello world")                   # "Hello World"

# ─── trim functions ──────────────────────────────
trim("  hello  ", " ")                 # "hello"
trimprefix("helloworld", "hello")      # "world"
trimsuffix("helloworld", "world")      # "hello"
trimspace("  hello  ")                 # "hello"

# ─── substr ──────────────────────────────────────
substr("hello world", 0, 5)           # "hello"
substr("hello world", 6, -1)          # "world"

# ─── regex / regexall ────────────────────────────
regex("[a-z]+", "53453453.345345aaabbb") # "aaabbb"
regexall("[a-z]+", "1234abcd5678efgh")   # ["abcd", "efgh"]

# ─── indent / chomp ──────────────────────────────
indent(4, "line1\nline2\nline3")
# "    line1\n    line2\n    line3"

chomp("hello\n\n")                     # "hello"
```

---

## 2.3 Collection Functions

```hcl
# ─── concat: Merge lists ─────────────────────────
concat(["a", "b"], ["c", "d"])         # ["a", "b", "c", "d"]

# ─── flatten: Flatten nested lists ────────────────
flatten([["a", "b"], ["c", ["d"]]])    # ["a", "b", "c", "d"]

# ─── merge: Combine maps ─────────────────────────
merge(
  { a = 1, b = 2 },
  { b = 3, c = 4 }
)
# { a = 1, b = 3, c = 4 }  ← Later values win

# ─── keys / values ───────────────────────────────
keys({ a = 1, b = 2, c = 3 })         # ["a", "b", "c"]
values({ a = 1, b = 2, c = 3 })       # [1, 2, 3]

# ─── lookup: Map with default ────────────────────
lookup({ dev = "t3.micro", prod = "m5.large" }, "dev", "t3.small")
# "t3.micro"
lookup({ dev = "t3.micro" }, "staging", "t3.small")
# "t3.small" (default)

# ─── contains ────────────────────────────────────
contains(["a", "b", "c"], "b")        # true
contains(["a", "b", "c"], "d")        # false

# ─── distinct: Remove duplicates ─────────────────
distinct(["a", "b", "a", "c", "b"])   # ["a", "b", "c"]

# ─── element: List indexing (wraps around) ────────
element(["a", "b", "c"], 0)           # "a"
element(["a", "b", "c"], 3)           # "a" (wraps!)

# ─── range: Generate number sequence ─────────────
range(3)                               # [0, 1, 2]
range(1, 5)                            # [1, 2, 3, 4]
range(0, 10, 2)                        # [0, 2, 4, 6, 8]

# ─── zipmap: Combine keys + values into map ──────
zipmap(["a", "b", "c"], [1, 2, 3])
# { a = 1, b = 2, c = 3 }

# ─── slice: List subset ──────────────────────────
slice(["a", "b", "c", "d", "e"], 1, 4)  # ["b", "c", "d"]

# ─── chunklist: Split list into sized chunks ──────
chunklist(["a", "b", "c", "d", "e"], 2)
# [["a", "b"], ["c", "d"], ["e"]]

# ─── coalesce / coalescelist ──────────────────────
coalesce("", "", "hello", "world")     # "hello"
coalescelist([], [], ["a", "b"])       # ["a", "b"]

# ─── setproduct: Cartesian product ───────────────
setproduct(["dev", "prod"], ["us-east-1", "eu-west-1"])
# [["dev","us-east-1"],["dev","eu-west-1"],
#  ["prod","us-east-1"],["prod","eu-west-1"]]
```

---

## 2.4 Encoding Functions

```hcl
# ─── JSON ─────────────────────────────────────────
jsonencode({
  name    = "myapp"
  ports   = [80, 443]
  enabled = true
})
# '{"enabled":true,"name":"myapp","ports":[80,443]}'

jsondecode("{\"name\": \"myapp\"}")
# { name = "myapp" }

# ─── YAML ─────────────────────────────────────────
yamlencode({
  apiVersion = "v1"
  kind       = "ConfigMap"
  metadata = {
    name = "myapp"
  }
})

yamldecode(file("config.yaml"))

# ─── CSV ──────────────────────────────────────────
csvdecode(file("users.csv"))
# Returns list of maps, one per CSV row

# ─── Base64 ───────────────────────────────────────
base64encode("Hello, World!")          # "SGVsbG8sIFdvcmxkIQ=="
base64decode("SGVsbG8sIFdvcmxkIQ==")   # "Hello, World!"

# ─── URL ──────────────────────────────────────────
urlencode("hello world/path")          # "hello+world%2Fpath"
```

---

## 2.5 Filesystem Functions

```hcl
# ─── file: Read file contents ────────────────────
file("${path.module}/scripts/init.sh")

# ─── fileexists: Check if file exists ─────────────
fileexists("${path.module}/optional.conf")  # true/false

# ─── templatefile: Template with variables ────────
# templates/user_data.tpl:
# #!/bin/bash
# echo "Hello, ${name}!"
# echo "Environment: ${env}"

templatefile("${path.module}/templates/user_data.tpl", {
  name = "MyApp"
  env  = "production"
})

# ─── fileset: Find files matching pattern ─────────
fileset(path.module, "templates/*.tpl")
# ["templates/a.tpl", "templates/b.tpl"]

fileset(path.module, "**/*.tf")
# All .tf files recursively

# ─── Path functions ──────────────────────────────
abspath(".")                           # "/home/user/project"
dirname("/path/to/file.txt")          # "/path/to"
basename("/path/to/file.txt")         # "file.txt"
pathexpand("~/config")                # "/home/user/config"
```

---

## 2.6 IP Network Functions

```hcl
# ─── cidrsubnet: Calculate subnet CIDR ────────────
cidrsubnet("10.0.0.0/16", 8, 0)    # "10.0.0.0/24"   (subnet 0)
cidrsubnet("10.0.0.0/16", 8, 1)    # "10.0.1.0/24"   (subnet 1)
cidrsubnet("10.0.0.0/16", 8, 255)  # "10.0.255.0/24" (subnet 255)
cidrsubnet("10.0.0.0/16", 4, 1)    # "10.0.16.0/20"  (larger subnets)

# ─── cidrsubnets: Multiple subnets at once ────────
cidrsubnets("10.0.0.0/16", 8, 8, 8, 4)
# ["10.0.0.0/24", "10.0.1.0/24", "10.0.2.0/24", "10.0.16.0/20"]

# ─── cidrhost: Get host IP from CIDR ─────────────
cidrhost("10.0.1.0/24", 5)         # "10.0.1.5"
cidrhost("10.0.1.0/24", -1)        # "10.0.1.254" (last usable)

# ─── cidrnetmask: Get netmask ────────────────────
cidrnetmask("10.0.0.0/16")         # "255.255.0.0"

# ─── Real-world usage ────────────────────────────
variable "vpc_cidr" {
  default = "10.0.0.0/16"
}

locals {
  subnet_cidrs = cidrsubnets(var.vpc_cidr, 8, 8, 8, 8, 8, 8)
  # [public-1, public-2, public-3, private-1, private-2, private-3]
}

resource "aws_subnet" "public" {
  count      = 3
  vpc_id     = aws_vpc.main.id
  cidr_block = local.subnet_cidrs[count.index]

  tags = { Name = "public-${count.index + 1}" }
}

resource "aws_subnet" "private" {
  count      = 3
  vpc_id     = aws_vpc.main.id
  cidr_block = local.subnet_cidrs[count.index + 3]

  tags = { Name = "private-${count.index + 1}" }
}
```

---

## 2.7 Type Conversion and Error Handling

```hcl
# ─── Type conversion ─────────────────────────────
tostring(42)                           # "42"
tonumber("42")                         # 42
tobool("true")                         # true
tolist(toset(["b", "a", "c"]))        # ["a", "b", "c"] (sorted)
toset(["a", "b", "a"])               # toset(["a", "b"])
tomap({ a = 1 })                       # { a = 1 }

# ─── try: Return first non-error value ────────────
try(var.config.settings.feature, "default")
# Returns feature value or "default" if path doesn't exist

try(
  jsondecode(var.json_input),
  { error = "invalid JSON" }
)

# ─── can: Test if expression would succeed ────────
can(tonumber("hello"))                 # false
can(tonumber("42"))                    # true

# Used in validation
variable "port" {
  type = string
  validation {
    condition     = can(tonumber(var.port))
    error_message = "Port must be a valid number."
  }
}

# ─── one: Extract single element ─────────────────
one([])                                # null
one(["hello"])                         # "hello"
# one(["a", "b"])                      # Error! More than one

# ─── sensitive / nonsensitive ─────────────────────
sensitive("my-secret-value")
nonsensitive(sensitive("known-value"))  # Remove sensitivity
```

---

## 2.8 Date and Time Functions

```hcl
# ─── timestamp: Current UTC time ─────────────────
timestamp()                            # "2024-01-15T10:30:00Z"

# ─── plantimestamp: Plan time (consistent) ────────
plantimestamp()                        # Same for entire plan

# ─── formatdate ──────────────────────────────────
formatdate("YYYY-MM-DD", timestamp())  # "2024-01-15"
formatdate("HH:mmaa", timestamp())     # "10:30am"
formatdate("DD MMM YYYY", timestamp()) # "15 Jan 2024"

# ─── timeadd ─────────────────────────────────────
timeadd(timestamp(), "24h")            # Tomorrow
timeadd(timestamp(), "720h")           # 30 days from now
timeadd("2024-01-01T00:00:00Z", "-1h") # Subtract 1 hour

# ─── timecmp ─────────────────────────────────────
timecmp("2024-01-01T00:00:00Z", "2024-06-01T00:00:00Z")  # -1
timecmp("2024-06-01T00:00:00Z", "2024-01-01T00:00:00Z")  # 1
timecmp("2024-01-01T00:00:00Z", "2024-01-01T00:00:00Z")  # 0
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| `function not found` | Typo or wrong function name | Check docs; use `terraform console` to test |
| Type error in function | Wrong argument type | Use type conversion: `tostring()`, `tonumber()` |
| `templatefile` error | Variable not passed | Ensure all template variables are in the vars map |
| `cidrsubnet` overlap | Wrong newbits/netnum | Use `cidrsubnets` for multiple non-overlapping subnets |
| `try` always returns default | Expression path is wrong | Debug with `terraform console`; check variable structure |

---

## Summary Table

| Category | Key Functions | Common Use |
|----------|--------------|------------|
| **String** | `format`, `join`, `split`, `replace` | Resource naming, string building |
| **Collection** | `merge`, `flatten`, `lookup`, `zipmap` | Combining configs, defaults |
| **Encoding** | `jsonencode`, `yamlencode`, `base64encode` | Policy docs, user data |
| **Filesystem** | `file`, `templatefile`, `fileset` | Loading scripts, templates |
| **IP Network** | `cidrsubnet`, `cidrsubnets`, `cidrhost` | Subnet calculation |
| **Type/Error** | `try`, `can`, `coalesce`, `one` | Safe defaults, validation |

---

## Quick Revision Questions

1. **How do you safely access a nested value that might not exist?**
   > Use `try(var.config.nested.value, "default")` — it returns the default if any part of the expression path fails.

2. **What is the difference between `cidrsubnet` and `cidrsubnets`?**
   > `cidrsubnet` calculates a single subnet CIDR. `cidrsubnets` calculates multiple non-overlapping subnets in one call, which is safer and easier.

3. **How do you test functions interactively?**
   > Run `terraform console` to get a REPL where you can test any function with live values from your configuration.

4. **What does `merge()` do when maps have overlapping keys?**
   > Later maps override earlier ones. `merge({a=1, b=2}, {b=3, c=4})` produces `{a=1, b=3, c=4}`.

5. **What is `templatefile` used for?**
   > Rendering a template file with variable substitution. It takes a file path and a map of variables, returning the rendered string.

---

[← Previous: Provisioners](01-provisioners.md) | [Back to README](../README.md) | [Next: Lifecycle Rules →](03-lifecycle-rules.md)
