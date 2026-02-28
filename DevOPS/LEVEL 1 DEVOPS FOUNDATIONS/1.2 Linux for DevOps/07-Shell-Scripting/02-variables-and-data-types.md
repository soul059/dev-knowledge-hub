# Chapter 2: Variables & Data Types

[← Previous: Bash Basics](01-bash-basics.md) | [Back to README](../README.md) | [Next: Conditionals →](03-conditionals.md)

---

## Overview

Variables are fundamental to Bash scripting, storing data that your scripts use and manipulate. Unlike most programming languages, Bash variables are untyped (everything is a string), but can be used in numerical and array contexts. Mastering variables is essential for writing dynamic, reusable DevOps automation scripts.

---

## 2.1 Variable Basics

```bash
# Assignment (NO spaces around =)
NAME="DevOps"
PORT=8080
LOG_DIR="/var/log/myapp"

# Access with $
echo $NAME
echo ${NAME}       # Preferred (explicit boundary)
echo "Hello, ${NAME}!"

# Common mistake
NAME = "value"     # ERROR: bash interprets NAME as a command
```

```
┌──────────────────────────────────────────────────────────────┐
│              VARIABLE TYPES IN BASH                           │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────┐   ┌──────────────────┐                 │
│  │ Local Variables   │   │ Environment Vars │                 │
│  │ (current shell)   │   │ (inherited by    │                 │
│  │                   │   │  child processes)│                 │
│  │ NAME="value"      │   │ export NAME="val"│                 │
│  └──────────────────┘   └──────────────────┘                 │
│                                                               │
│  ┌──────────────────┐   ┌──────────────────┐                 │
│  │ Special Variables │   │ Positional Params│                 │
│  │ $?, $$, $!, $#   │   │ $0, $1, $2 ...   │                 │
│  │ $@, $*           │   │ $@, $#           │                 │
│  └──────────────────┘   └──────────────────┘                 │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.2 Variable Naming Rules

```bash
# Valid names
my_var="ok"
MY_VAR="ok"
_private="ok"
var2="ok"
APP_VERSION="1.2.3"

# Invalid names
2var="bad"         # Cannot start with digit
my-var="bad"       # No hyphens
my var="bad"       # No spaces
my.var="bad"       # No dots

# Convention
local_var="lowercase for local"           # Local variables
GLOBAL_VAR="UPPERCASE for environment"    # Environment/constants
_internal="underscore for internal"       # Internal use
```

---

## 2.3 Quoting

```bash
NAME="World"

# Double quotes — Allows variable expansion
echo "Hello, $NAME"        # Hello, World

# Single quotes — Literal string (no expansion)
echo 'Hello, $NAME'        # Hello, $NAME

# Backticks / $() — Command substitution
echo "Date: $(date)"       # Date: Mon Jan 15 10:30:00 UTC 2024
echo "Date: `date`"        # Same, but $() is preferred (nestable)

# Escaping
echo "Price: \$10"         # Price: $10
echo "She said \"hi\""     # She said "hi"
```

```
┌──────────────────────────────────────────────────────────────┐
│            QUOTING COMPARISON                                 │
├────────────────┬─────────────────┬───────────────────────────┤
│ Quote Type     │ Expansion?      │ Example Output            │
├────────────────┼─────────────────┼───────────────────────────┤
│ "double"       │ Yes ($, `, \)   │ "Hello World"             │
│ 'single'       │ No (literal)    │ 'Hello $NAME'             │
│ $'ansi'        │ Escape seqs     │ $'tab\there'              │
│ none           │ Yes + splitting │ Hello World (2 words)     │
└────────────────┴─────────────────┴───────────────────────────┘
```

---

## 2.4 Environment Variables

```bash
# Set environment variable (inherited by child processes)
export APP_ENV="production"
export DB_HOST="db.example.com"
export DB_PORT=5432

# View all environment variables
env
printenv
export -p

# View specific
echo $HOME
echo $PATH
echo $USER
echo $SHELL
echo $PWD
printenv HOME

# Remove
unset APP_ENV

# One-time for a command (does not persist)
DB_HOST=localhost DB_PORT=5432 ./app.sh
```

```
┌──────────────────────────────────────────────────────────────┐
│       IMPORTANT BUILT-IN ENVIRONMENT VARIABLES               │
├────────────────┬─────────────────────────────────────────────┤
│ Variable       │ Description                                 │
├────────────────┼─────────────────────────────────────────────┤
│ HOME           │ User's home directory                       │
│ PATH           │ Command search directories                  │
│ USER / LOGNAME │ Current username                            │
│ SHELL          │ Current shell path                          │
│ PWD            │ Present working directory                   │
│ OLDPWD         │ Previous working directory                  │
│ HOSTNAME       │ System hostname                             │
│ LANG           │ System locale                               │
│ EDITOR         │ Default text editor                         │
│ TERM           │ Terminal type                               │
│ PS1            │ Primary prompt string                       │
│ RANDOM         │ Random number (0-32767)                     │
│ SECONDS        │ Seconds since shell started                 │
│ LINENO         │ Current line number in script               │
└────────────────┴─────────────────────────────────────────────┘
```

---

## 2.5 Special Variables

```bash
#!/bin/bash
# special-vars.sh arg1 arg2 arg3

echo "\$0 = $0"      # Script name (./special-vars.sh)
echo "\$1 = $1"      # First argument (arg1)
echo "\$2 = $2"      # Second argument (arg2)
echo "\$# = $#"      # Number of arguments (3)
echo "\$@ = $@"      # All arguments as separate words
echo "\$* = $*"      # All arguments as single string
echo "\$$ = $$"      # Current process PID
echo "\$! = $!"      # Last background process PID
echo "\$? = $?"      # Last command exit status
echo "\$- = $-"      # Current shell options

# Important: "$@" vs "$*"
# "$@" → "arg1" "arg2" "arg3"   (preserves individual args)
# "$*" → "arg1 arg2 arg3"       (one single string)
# Always use "$@" when passing arguments to other commands!
```

---

## 2.6 Readonly & Constants

```bash
# Readonly variables cannot be changed
readonly DB_NAME="production_db"
readonly PI=3.14159
DB_NAME="other"    # ERROR: DB_NAME: readonly variable

# declare -r also works
declare -r MAX_RETRIES=3

# Using uppercase convention for constants
readonly APP_NAME="MyApp"
readonly VERSION="2.1.0"
readonly CONFIG_DIR="/etc/myapp"
```

---

## 2.7 Arrays

```bash
# Indexed arrays
SERVERS=("web01" "web02" "web03" "db01")

# Access elements (0-indexed)
echo "${SERVERS[0]}"     # web01
echo "${SERVERS[2]}"     # web03

# All elements
echo "${SERVERS[@]}"     # web01 web02 web03 db01

# Array length
echo "${#SERVERS[@]}"    # 4

# Add element
SERVERS+=("cache01")

# Remove element
unset SERVERS[1]

# Loop over array
for server in "${SERVERS[@]}"; do
    echo "Checking $server..."
done

# Array slicing
echo "${SERVERS[@]:1:2}"   # 2 elements starting at index 1
```

```bash
# Associative arrays (bash 4+)
declare -A CONFIG
CONFIG[host]="db.example.com"
CONFIG[port]="5432"
CONFIG[user]="admin"
CONFIG[database]="myapp"

# Access
echo "${CONFIG[host]}"      # db.example.com

# All keys
echo "${!CONFIG[@]}"        # host port user database

# All values
echo "${CONFIG[@]}"

# Loop
for key in "${!CONFIG[@]}"; do
    echo "$key = ${CONFIG[$key]}"
done
```

---

## 2.8 String Operations

```bash
STR="Hello DevOps World"

# Length
echo "${#STR}"              # 18

# Substring
echo "${STR:6}"             # DevOps World
echo "${STR:6:6}"           # DevOps

# Replace
echo "${STR/DevOps/Cloud}"  # Hello Cloud World
echo "${STR//o/0}"          # Hell0 Dev0ps W0rld (all occurrences)

# Remove pattern (from beginning)
FILE="/var/log/app/error.log"
echo "${FILE#*/}"           # var/log/app/error.log  (shortest match)
echo "${FILE##*/}"          # error.log              (longest match)

# Remove pattern (from end)
echo "${FILE%/*}"           # /var/log/app           (shortest match)
echo "${FILE%%/*}"          #                        (longest match)

# Case conversion (bash 4+)
echo "${STR,,}"             # hello devops world     (lowercase)
echo "${STR^^}"             # HELLO DEVOPS WORLD     (uppercase)
echo "${STR^}"              # Hello devops world     (first char upper)
```

---

## 2.9 Arithmetic

```bash
# Using $(( ))
echo $((5 + 3))          # 8
echo $((10 / 3))         # 3 (integer division)
echo $((10 % 3))         # 1 (modulus)
echo $((2 ** 10))        # 1024 (exponent)

# Increment / Decrement
COUNT=0
((COUNT++))              # 1
((COUNT+=5))             # 6

# Using let
let "result = 5 * 3"
echo $result             # 15

# Using expr (legacy)
expr 5 + 3               # 8

# Floating point (use bc)
echo "scale=2; 10 / 3" | bc    # 3.33
echo "scale=4; 22 / 7" | bc    # 3.1428

# Comparison in arithmetic context
if (( COUNT > 5 )); then
    echo "Count is greater than 5"
fi
```

---

## 2.10 Default Values & Parameter Expansion

```bash
# Use default if variable is unset or empty
echo "${NAME:-default}"        # Use "default" if NAME unset
echo "${NAME:=default}"        # Set NAME to "default" if unset
echo "${NAME:+override}"       # Use "override" if NAME IS set
echo "${NAME:?Name is required}"  # Error if NAME unset

# DevOps use case
ENV="${1:-development}"        # First arg or "development"
PORT="${PORT:-8080}"           # Environment var or 8080
LOG_LEVEL="${LOG_LEVEL:-info}" # Environment var or "info"
WORKERS="${WORKERS:-$(nproc)}" # Number of CPU cores as default
```

---

## 2.11 Real-World Scenario: Config-Driven Deploy Script

```bash
#!/usr/bin/env bash
# deploy-config.sh — Uses variables and arrays for deployment

set -euo pipefail

# Constants
readonly SCRIPT_NAME="$(basename "$0")"
readonly DEPLOY_DIR="/opt/deployments"

# Configuration with defaults
APP_NAME="${APP_NAME:-myapp}"
ENV="${1:-staging}"
VERSION="${2:-latest}"
REPLICAS="${REPLICAS:-3}"

# Server arrays
declare -A ENVIRONMENTS
ENVIRONMENTS[dev]="dev01.example.com"
ENVIRONMENTS[staging]="staging01.example.com staging02.example.com"
ENVIRONMENTS[prod]="prod01.example.com prod02.example.com prod03.example.com"

# Validate environment
if [ -z "${ENVIRONMENTS[$ENV]+x}" ]; then
    echo "ERROR: Unknown environment: $ENV"
    echo "Valid: ${!ENVIRONMENTS[*]}"
    exit 1
fi

# Parse servers into array
read -ra SERVERS <<< "${ENVIRONMENTS[$ENV]}"

echo "=== Deployment Plan ==="
echo "App:         $APP_NAME"
echo "Version:     $VERSION"
echo "Environment: $ENV"
echo "Servers:     ${#SERVERS[@]} (${SERVERS[*]})"
echo "Replicas:    $REPLICAS"
echo "========================"

# Deploy to each server
for i in "${!SERVERS[@]}"; do
    server="${SERVERS[$i]}"
    echo ""
    echo "[$(( i + 1 ))/${#SERVERS[@]}] Deploying to $server..."
    echo "  ssh $server 'docker pull ${APP_NAME}:${VERSION}'"
    echo "  ssh $server 'docker service update --image ${APP_NAME}:${VERSION}'"
done

echo ""
echo "Deployment of ${APP_NAME}:${VERSION} to ${ENV} complete!"
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| `unbound variable` | Using `set -u` with unset var | Use `${VAR:-default}` or check with `${VAR+x}` |
| Variable has extra whitespace | `\r\n` line endings | `dos2unix` or `tr -d '\r'` |
| Array not working | Old bash version | Check `bash --version`; need bash 4+ for associative arrays |
| Variable lost after pipe | Pipe creates subshell | Use `while read ... done < <(command)` process substitution |
| Spaces break variable | Not quoting | Always use `"$VAR"` not `$VAR` |
| Arithmetic with decimals | Bash only does integers | Use `bc` or `awk` for floating point |

---

## Summary Table

| Feature | Syntax | Example |
|---------|--------|---------|
| Assign | `VAR=value` | `NAME="hello"` |
| Access | `${VAR}` | `echo "${NAME}"` |
| Export | `export VAR` | `export PATH="/usr/local/bin:$PATH"` |
| Default | `${VAR:-default}` | `PORT="${PORT:-8080}"` |
| Readonly | `readonly VAR` | `readonly VERSION="1.0"` |
| Array | `ARR=(a b c)` | `echo "${ARR[0]}"` |
| Assoc. array | `declare -A MAP` | `MAP[key]="value"` |
| String length | `${#VAR}` | `echo "${#NAME}"` |
| Substring | `${VAR:pos:len}` | `"${STR:0:5}"` |
| Arithmetic | `$(( expr ))` | `$(( 5 + 3 ))` |

---

## Quick Revision Questions

1. **What is the difference between `$@` and `$*` when double-quoted?**
2. **How do you set a default value for a variable that might be unset?**
3. **What is the difference between `export VAR=value` and `VAR=value`?**
4. **How do you extract the filename from the path `/var/log/app/error.log` using parameter expansion?**
5. **How do you perform floating-point arithmetic in Bash?**
6. **What happens if you pipe output into a `while read` loop and modify a variable inside it?**

---

[← Previous: Bash Basics](01-bash-basics.md) | [Back to README](../README.md) | [Next: Conditionals →](03-conditionals.md)
