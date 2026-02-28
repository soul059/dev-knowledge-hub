# Chapter 3: Conditionals

[← Previous: Variables & Data Types](02-variables-and-data-types.md) | [Back to README](../README.md) | [Next: Loops →](04-loops.md)

---

## Overview

Conditional logic allows scripts to make decisions based on test results, variable values, and command outcomes. Bash provides `if`, `case`, and test operators for building robust decision-making in DevOps automation scripts.

---

## 3.1 The `if` Statement

```bash
# Basic syntax
if [ condition ]; then
    # commands
fi

# if-else
if [ condition ]; then
    echo "True"
else
    echo "False"
fi

# if-elif-else
if [ "$ENV" = "prod" ]; then
    echo "Production deployment"
elif [ "$ENV" = "staging" ]; then
    echo "Staging deployment"
elif [ "$ENV" = "dev" ]; then
    echo "Development deployment"
else
    echo "Unknown environment: $ENV"
    exit 1
fi
```

```
┌──────────────────────────────────────────────────────────────┐
│              CONDITIONAL FLOW                                 │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│            ┌─────────┐                                        │
│            │  START   │                                        │
│            └────┬─────┘                                       │
│                 ▼                                              │
│         ┌──────────────┐   No    ┌──────────────┐            │
│         │ condition 1? ├────────►│ condition 2? │            │
│         └──────┬───────┘         └──────┬───────┘            │
│           Yes  │                   Yes  │   No               │
│                ▼                        ▼    │               │
│         ┌──────────┐           ┌──────────┐  ▼              │
│         │  then     │           │  elif    │ ┌─────────┐    │
│         │  block    │           │  block   │ │ else    │    │
│         └──────┬────┘           └────┬─────┘ │ block   │    │
│                │                     │       └────┬────┘    │
│                └─────────┬───────────┘            │         │
│                          ▼                        │         │
│                    ┌─────────┐◄───────────────────┘         │
│                    │   END   │                               │
│                    └─────────┘                               │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.2 Test Operators: `[ ]` vs `[[ ]]` vs `(( ))`

```bash
# [ ] — POSIX test (portable, more caveats)
if [ "$a" = "$b" ]; then echo "equal"; fi

# [[ ]] — Bash extended test (preferred in bash scripts)
if [[ "$a" == "$b" ]]; then echo "equal"; fi

# (( )) — Arithmetic evaluation
if (( count > 10 )); then echo "big"; fi
```

```
┌──────────────────────────────────────────────────────────────┐
│         [ ] vs [[ ]] COMPARISON                               │
├──────────────┬────────────────────┬──────────────────────────┤
│ Feature      │ [ ] (test)         │ [[ ]] (bash)             │
├──────────────┼────────────────────┼──────────────────────────┤
│ Portability  │ POSIX — all shells │ Bash/Zsh only            │
│ Word split   │ Yes (must quote)   │ No                       │
│ Glob expand  │ Yes                │ No (except ==)           │
│ Pattern match│ No                 │ Yes (== with pattern)    │
│ Regex match  │ No                 │ Yes (=~ operator)        │
│ &&, ||       │ -a, -o (or chain)  │ Yes (native)             │
│ < > compare  │ Alphabetical       │ Alphabetical             │
│ Empty var    │ Error if unquoted  │ Safe                     │
│ Recommended  │ For /bin/sh scripts│ For bash scripts         │
└──────────────┴────────────────────┴──────────────────────────┘
```

---

## 3.3 String Comparisons

```bash
STR="hello"

# Equality
[[ "$STR" == "hello" ]]          # True
[[ "$STR" != "world" ]]          # True

# Empty / Not empty
[[ -z "$STR" ]]                  # True if empty
[[ -n "$STR" ]]                  # True if not empty

# Pattern matching (glob)
[[ "$STR" == h* ]]               # True (starts with h)
[[ "$STR" == *llo ]]             # True (ends with llo)

# Regex matching
[[ "$STR" =~ ^[a-z]+$ ]]        # True (all lowercase letters)
[[ "error: disk full" =~ error ]]  # True (contains 'error')

# Extract regex match
if [[ "v2.14.3" =~ ^v([0-9]+)\.([0-9]+)\.([0-9]+)$ ]]; then
    echo "Major: ${BASH_REMATCH[1]}"  # 2
    echo "Minor: ${BASH_REMATCH[2]}"  # 14
    echo "Patch: ${BASH_REMATCH[3]}"  # 3
fi
```

---

## 3.4 Numeric Comparisons

```bash
A=10
B=20

# Inside [ ] or [[ ]]
[[ $A -eq $B ]]    # Equal
[[ $A -ne $B ]]    # Not equal
[[ $A -lt $B ]]    # Less than
[[ $A -le $B ]]    # Less than or equal
[[ $A -gt $B ]]    # Greater than
[[ $A -ge $B ]]    # Greater than or equal

# Inside (( )) — More readable for math
(( A == B ))       # Equal
(( A != B ))       # Not equal
(( A < B ))        # Less than
(( A <= B ))       # Less than or equal
(( A > B ))        # Greater than
(( A >= B ))       # Greater than or equal
```

```
┌──────────────────────────────────────────────────────────────┐
│        NUMERIC COMPARISON OPERATORS                           │
├────────────────┬──────────────────┬──────────────────────────┤
│ Test [ ] / [[ ]]│  Arithmetic (( )) │ Meaning                │
├────────────────┼──────────────────┼──────────────────────────┤
│ -eq            │ ==               │ Equal                    │
│ -ne            │ !=               │ Not equal                │
│ -lt            │ <                │ Less than                │
│ -le            │ <=               │ Less than or equal       │
│ -gt            │ >                │ Greater than             │
│ -ge            │ >=               │ Greater than or equal    │
└────────────────┴──────────────────┴──────────────────────────┘
```

---

## 3.5 File Test Operators

```bash
FILE="/etc/nginx/nginx.conf"
DIR="/var/log"

# Existence
[[ -e "$FILE" ]]     # Exists (file or directory)
[[ -f "$FILE" ]]     # Is a regular file
[[ -d "$DIR" ]]      # Is a directory
[[ -L "$FILE" ]]     # Is a symlink

# Permissions
[[ -r "$FILE" ]]     # Is readable
[[ -w "$FILE" ]]     # Is writable
[[ -x "$FILE" ]]     # Is executable

# Size
[[ -s "$FILE" ]]     # File exists and is not empty

# Comparison
[[ "$FILE1" -nt "$FILE2" ]]   # FILE1 is newer
[[ "$FILE1" -ot "$FILE2" ]]   # FILE1 is older
[[ "$FILE1" -ef "$FILE2" ]]   # Same inode (hard link)
```

```
┌──────────────────────────────────────────────────────────────┐
│          FILE TEST OPERATORS CHEAT SHEET                      │
├──────────┬───────────────────────────────────────────────────┤
│ Operator │ True If...                                        │
├──────────┼───────────────────────────────────────────────────┤
│ -e file  │ File exists                                       │
│ -f file  │ Regular file                                      │
│ -d file  │ Directory                                         │
│ -L file  │ Symbolic link                                     │
│ -b file  │ Block device                                      │
│ -c file  │ Character device                                  │
│ -p file  │ Named pipe (FIFO)                                 │
│ -S file  │ Socket                                            │
│ -r file  │ Readable by current user                          │
│ -w file  │ Writable by current user                          │
│ -x file  │ Executable by current user                        │
│ -s file  │ File size > 0                                     │
│ -t fd    │ File descriptor is a terminal                     │
│ f1 -nt f2│ f1 is newer than f2                               │
│ f1 -ot f2│ f1 is older than f2                               │
└──────────┴───────────────────────────────────────────────────┘
```

---

## 3.6 Logical Operators

```bash
# AND
if [[ -f "$FILE" && -r "$FILE" ]]; then
    echo "File exists and is readable"
fi

# OR
if [[ "$ENV" == "dev" || "$ENV" == "test" ]]; then
    echo "Non-production environment"
fi

# NOT
if [[ ! -d "$DIR" ]]; then
    echo "Directory does not exist"
fi

# Complex conditions
if [[ -f "$CONFIG" ]] && [[ -s "$CONFIG" ]] && [[ -r "$CONFIG" ]]; then
    source "$CONFIG"
fi

# Nested
if [[ "$ENV" == "prod" ]]; then
    if [[ "$APPROVED" == "yes" ]]; then
        echo "Deploying to production"
    else
        echo "Production deployment requires approval"
        exit 1
    fi
fi
```

---

## 3.7 The `case` Statement

```bash
# Syntax
case "$variable" in
    pattern1)
        commands
        ;;           # Break (required)
    pattern2|pattern3)
        commands
        ;;
    *)               # Default (like else)
        commands
        ;;
esac

# Example: Service management script
case "$1" in
    start)
        echo "Starting service..."
        systemctl start myapp
        ;;
    stop)
        echo "Stopping service..."
        systemctl stop myapp
        ;;
    restart)
        echo "Restarting service..."
        systemctl restart myapp
        ;;
    status)
        systemctl status myapp
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}"
        exit 1
        ;;
esac
```

```bash
# Case with patterns
read -p "Continue? (y/n): " ANSWER
case "$ANSWER" in
    [Yy]|[Yy][Ee][Ss])
        echo "Proceeding..."
        ;;
    [Nn]|[Nn][Oo])
        echo "Cancelled."
        exit 0
        ;;
    *)
        echo "Invalid input"
        exit 1
        ;;
esac

# Case with file extensions
case "$FILE" in
    *.tar.gz|*.tgz)
        tar xzf "$FILE"
        ;;
    *.tar.bz2)
        tar xjf "$FILE"
        ;;
    *.zip)
        unzip "$FILE"
        ;;
    *.deb)
        sudo dpkg -i "$FILE"
        ;;
    *)
        echo "Unknown file format: $FILE"
        ;;
esac
```

---

## 3.8 Ternary-Style Expressions

```bash
# Bash doesn't have ternary (?:) but you can use:

# Method 1: AND-OR
[[ "$ENV" == "prod" ]] && PORT=443 || PORT=8080

# Method 2: Inline if
PORT=$( [[ "$ENV" == "prod" ]] && echo 443 || echo 8080 )

# Method 3: Arithmetic ternary
(( result = (a > b) ? a : b ))
```

---

## 3.9 Real-World Scenario: Pre-Deployment Checker

```bash
#!/usr/bin/env bash
# pre-deploy-check.sh — Validates system before deployment

set -euo pipefail

echo "=== Pre-Deployment Checks ==="
PASS=0
FAIL=0

check() {
    local desc="$1"
    local result="$2"
    if [[ "$result" == "pass" ]]; then
        echo "  ✓ $desc"
        ((PASS++))
    else
        echo "  ✗ $desc"
        ((FAIL++))
    fi
}

# Check 1: Disk space > 20% free
DISK_FREE=$(df / | tail -1 | awk '{print 100 - $5}')
if (( DISK_FREE > 20 )); then
    check "Disk space (${DISK_FREE}% free)" "pass"
else
    check "Disk space (${DISK_FREE}% free — need >20%)" "fail"
fi

# Check 2: Memory available > 512MB
MEM_AVAIL=$(free -m | awk '/Mem:/ {print $7}')
if (( MEM_AVAIL > 512 )); then
    check "Memory (${MEM_AVAIL}MB available)" "pass"
else
    check "Memory (${MEM_AVAIL}MB available — need >512MB)" "fail"
fi

# Check 3: Docker running
if systemctl is-active --quiet docker 2>/dev/null; then
    check "Docker service running" "pass"
else
    check "Docker service not running" "fail"
fi

# Check 4: Required files exist
for file in docker-compose.yml .env Dockerfile; do
    if [[ -f "$file" ]]; then
        check "File exists: $file" "pass"
    else
        check "File missing: $file" "fail"
    fi
done

# Check 5: Port 80 available
if ! ss -tuln | grep -q ':80 '; then
    check "Port 80 available" "pass"
else
    check "Port 80 in use" "fail"
fi

echo ""
echo "Results: $PASS passed, $FAIL failed"

if (( FAIL > 0 )); then
    echo "DEPLOYMENT BLOCKED — Fix failures above"
    exit 1
else
    echo "ALL CHECKS PASSED — Ready to deploy"
    exit 0
fi
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| `[: too many arguments` | Unquoted variable with spaces | Always quote: `[ "$VAR" = "value" ]` |
| `[: =: unary operator expected` | Variable is empty | Quote variable or use `[[ ]]` |
| `==` not working in `[ ]` | Not POSIX compliant | Use `=` in `[ ]` or switch to `[[ ]]` |
| Pattern not matching in `[ ]` | Globbing not supported | Use `[[ ]]` for pattern matching or `case` |
| Numeric compare with `-gt` fails | Variable contains non-numeric | Validate input: `[[ "$VAR" =~ ^[0-9]+$ ]]` |
| `;;` missing in case | Syntax error | Every case branch needs `;;` before next `)` |

---

## Summary Table

| Construct | Syntax | Use Case |
|-----------|--------|----------|
| `if` | `if [[ cond ]]; then ... fi` | General decisions |
| `elif` | `elif [[ cond ]]; then ...` | Multiple conditions |
| `case` | `case "$var" in pat) ... ;; esac` | Multi-value matching |
| `[[ ]]` | `[[ -f file \|\| $a == $b ]]` | Extended test (bash) |
| `(( ))` | `(( count > 5 ))` | Arithmetic comparison |
| String test | `-z`, `-n`, `==`, `!=` | String checks |
| File test | `-f`, `-d`, `-r`, `-w`, `-x` | File system checks |
| Numeric test | `-eq`, `-lt`, `-gt`, `-ge` | Integer comparison |
| Regex | `[[ $s =~ pattern ]]` | Pattern matching |

---

## Quick Revision Questions

1. **What is the difference between `[ ]` and `[[ ]]`?**
2. **How do you check if a file exists AND is readable in one condition?**
3. **Write a `case` statement that handles `.tar.gz`, `.zip`, and `.deb` files differently.**
4. **How do you extract captured groups from a regex match in Bash?**
5. **What happens if you use `>` instead of `-gt` for numeric comparison in `[[ ]]`?**
6. **How would you check if a variable is either empty or unset?**

---

[← Previous: Variables & Data Types](02-variables-and-data-types.md) | [Back to README](../README.md) | [Next: Loops →](04-loops.md)
