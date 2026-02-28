# Chapter 5: Functions

[← Previous: Loops](04-loops.md) | [Back to README](../README.md) | [Next: Input/Output Redirection →](06-input-output-redirection.md)

---

## Overview

Functions encapsulate reusable blocks of code, making scripts modular, readable, and maintainable. In DevOps, functions structure deployment scripts, health checks, logging utilities, and shared library code.

---

## 5.1 Defining Functions

```bash
# Method 1: Standard (preferred)
greet() {
    echo "Hello, $1!"
}

# Method 2: With 'function' keyword
function greet {
    echo "Hello, $1!"
}

# Method 3: One-liner
greet() { echo "Hello, $1!"; }

# Call the function
greet "DevOps Engineer"    # Hello, DevOps Engineer!
```

```
┌──────────────────────────────────────────────────────────────┐
│              FUNCTION ANATOMY                                 │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│   function_name() {                                           │
│       │                                                       │
│       │  # Arguments: $1, $2, ... $N                         │
│       │  # All args:  $@ (as array), $* (as string)          │
│       │  # Arg count: $#                                      │
│       │                                                       │
│       │  local var="value"     # Local variable               │
│       │                                                       │
│       │  # ... function body ...                              │
│       │                                                       │
│       │  return 0              # Exit status (0-255)          │
│       │  # OR                                                 │
│       │  echo "output"         # Return data via stdout       │
│   }                                                           │
│                                                               │
│   result=$(function_name "arg1" "arg2")  # Capture output    │
│   function_name "arg1"                    # Just call it      │
│   echo $?                                 # Check return code │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.2 Arguments & Parameters

```bash
deploy() {
    local app="$1"
    local version="$2"
    local env="${3:-staging}"     # Default value
    
    echo "Deploying $app v$version to $env"
    echo "Total args: $#"
}

deploy "myapp" "2.1.0" "production"
# Deploying myapp v2.1.0 to production
# Total args: 3

deploy "myapp" "1.0.0"
# Deploying myapp v1.0.0 to staging
# Total args: 2

# Passing all arguments through
wrapper() {
    echo "Wrapper called with: $@"
    actual_command "$@"    # Pass all args to another command
}
```

---

## 5.3 Return Values

```bash
# Return status code (0-255 only)
is_running() {
    local service="$1"
    systemctl is-active --quiet "$service"
    return $?     # 0 = running, non-zero = not running
}

if is_running "nginx"; then
    echo "Nginx is running"
fi

# Return data via echo + command substitution
get_ip() {
    hostname -I | awk '{print $1}'
}
MY_IP=$(get_ip)
echo "My IP: $MY_IP"

# Return multiple values
get_memory() {
    local total=$(free -m | awk '/Mem:/ {print $2}')
    local used=$(free -m | awk '/Mem:/ {print $3}')
    local free=$(free -m | awk '/Mem:/ {print $4}')
    echo "$total $used $free"
}
read -r TOTAL USED FREE <<< "$(get_memory)"
echo "Memory: ${USED}/${TOTAL} MB used, ${FREE} MB free"
```

---

## 5.4 Local Variables & Scope

```bash
# Without local — variable leaks to caller
bad_function() {
    RESULT="leaked"    # Global!
}
bad_function
echo "$RESULT"    # "leaked" — side effect

# With local — variable stays in function
good_function() {
    local result="contained"
    echo "$result"
}
good_function
echo "$result"    # Empty — properly scoped

# Best practice: Always use 'local'
process_file() {
    local file="$1"
    local line_count
    local word_count
    
    line_count=$(wc -l < "$file")
    word_count=$(wc -w < "$file")
    
    echo "$file: $line_count lines, $word_count words"
}
```

```
┌──────────────────────────────────────────────────────────────┐
│              VARIABLE SCOPE                                   │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Global Scope (script level)                                  │
│  ┌──────────────────────────────────────────────────┐        │
│  │  GLOBAL_VAR="I'm everywhere"                      │        │
│  │                                                    │        │
│  │  my_func() {                                       │        │
│  │  ┌──────────────────────────────────────────┐     │        │
│  │  │  Local Scope                              │     │        │
│  │  │  local LOCAL_VAR="I'm only here"          │     │        │
│  │  │  echo "$GLOBAL_VAR"  ← Can see global     │     │        │
│  │  │  GLOBAL_VAR="modified" ← Can modify it!   │     │        │
│  │  └──────────────────────────────────────────┘     │        │
│  │                                                    │        │
│  │  echo "$LOCAL_VAR"  ← Empty (out of scope)        │        │
│  └──────────────────────────────────────────────────┘        │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.5 Logging & Utility Functions

```bash
#!/usr/bin/env bash
# Common utility functions for DevOps scripts

# Colors
readonly RED='\033[0;31m'
readonly GREEN='\033[0;32m'
readonly YELLOW='\033[1;33m'
readonly BLUE='\033[0;34m'
readonly NC='\033[0m'

# Logging
log_info()  { echo -e "$(date '+%F %T') ${GREEN}[INFO]${NC}  $*"; }
log_warn()  { echo -e "$(date '+%F %T') ${YELLOW}[WARN]${NC}  $*"; }
log_error() { echo -e "$(date '+%F %T') ${RED}[ERROR]${NC} $*" >&2; }
log_debug() { [[ "${DEBUG:-false}" == "true" ]] && echo -e "$(date '+%F %T') ${BLUE}[DEBUG]${NC} $*"; }

# Die with error message
die() {
    log_error "$@"
    exit 1
}

# Check command exists
require_cmd() {
    command -v "$1" &>/dev/null || die "Required command not found: $1"
}

# Check root privileges
require_root() {
    [[ $EUID -eq 0 ]] || die "This script must be run as root"
}

# Confirm action
confirm() {
    local prompt="${1:-Are you sure?}"
    read -rp "$prompt [y/N] " response
    [[ "$response" =~ ^[Yy]$ ]]
}

# Usage
require_cmd docker
require_cmd kubectl
log_info "Starting deployment..."
confirm "Deploy to production?" || die "Deployment cancelled"
```

---

## 5.6 Function Libraries (Sourcing)

```bash
# lib/common.sh — Shared function library
#!/usr/bin/env bash

log_info()  { echo "[INFO]  $(date '+%T') $*"; }
log_error() { echo "[ERROR] $(date '+%T') $*" >&2; }

check_disk() {
    local threshold="${1:-80}"
    local usage
    usage=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
    (( usage < threshold ))
}

check_service() {
    systemctl is-active --quiet "$1" 2>/dev/null
}
```

```bash
# deploy.sh — Uses the library
#!/usr/bin/env bash
set -euo pipefail

# Source the library (relative to script location)
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
source "${SCRIPT_DIR}/lib/common.sh"

log_info "Starting deployment"

if check_disk 90; then
    log_info "Disk space OK"
else
    log_error "Disk space critical!"
    exit 1
fi

if check_service docker; then
    log_info "Docker is running"
fi
```

---

## 5.7 Advanced Function Patterns

### Trap & Cleanup Functions

```bash
cleanup() {
    local exit_code=$?
    log_info "Cleaning up..."
    rm -rf "$TEMP_DIR"
    [[ -f "$LOCK_FILE" ]] && rm -f "$LOCK_FILE"
    exit "$exit_code"
}
trap cleanup EXIT INT TERM

TEMP_DIR=$(mktemp -d)
LOCK_FILE="/tmp/deploy.lock"
```

### Argument Parsing in Functions

```bash
create_user() {
    local username=""
    local role="developer"
    local shell="/bin/bash"
    
    while [[ $# -gt 0 ]]; do
        case "$1" in
            --name)    username="$2"; shift 2 ;;
            --role)    role="$2"; shift 2 ;;
            --shell)   shell="$2"; shift 2 ;;
            *)         echo "Unknown option: $1"; return 1 ;;
        esac
    done
    
    [[ -z "$username" ]] && { echo "Error: --name required"; return 1; }
    
    echo "Creating user: $username (role: $role, shell: $shell)"
    sudo useradd -m -s "$shell" -G "$role" "$username"
}

create_user --name john --role admin --shell /bin/zsh
```

### Higher-Order Functions (Passing Functions)

```bash
# Execute a function on each server
run_on_servers() {
    local func="$1"
    shift
    local servers=("$@")
    
    for server in "${servers[@]}"; do
        echo "--- $server ---"
        "$func" "$server"
    done
}

check_uptime() { ssh "$1" "uptime"; }
check_disk()   { ssh "$1" "df -h /"; }

SERVERS=("web01" "web02" "db01")
run_on_servers check_uptime "${SERVERS[@]}"
run_on_servers check_disk "${SERVERS[@]}"
```

---

## 5.8 Real-World Scenario: Modular Deploy Script

```bash
#!/usr/bin/env bash
# deploy.sh — Modular deployment with functions

set -euo pipefail

APP_NAME="${1:?Usage: $0 <app> <version> <env>}"
VERSION="${2:?Usage: $0 <app> <version> <env>}"
ENV="${3:-staging}"

# ── Logging ──────────────────────────────────────────
log() { echo "[$(date '+%T')] $*"; }

# ── Pre-checks ──────────────────────────────────────
pre_check() {
    log "Running pre-deployment checks..."
    
    local checks_passed=true
    
    command -v docker &>/dev/null || { log "ERROR: docker not found"; checks_passed=false; }
    
    local disk_usage
    disk_usage=$(df / | tail -1 | awk '{print $5}' | tr -d '%')
    (( disk_usage < 85 )) || { log "ERROR: Disk usage at ${disk_usage}%"; checks_passed=false; }
    
    [[ "$checks_passed" == true ]] || { log "Pre-checks FAILED"; return 1; }
    log "Pre-checks passed"
}

# ── Pull Image ──────────────────────────────────────
pull_image() {
    local image="registry.example.com/${APP_NAME}:${VERSION}"
    log "Pulling image: $image"
    docker pull "$image"
}

# ── Stop Old ────────────────────────────────────────
stop_old() {
    log "Stopping old containers..."
    docker stop "${APP_NAME}" 2>/dev/null || true
    docker rm "${APP_NAME}" 2>/dev/null || true
}

# ── Start New ───────────────────────────────────────
start_new() {
    local image="registry.example.com/${APP_NAME}:${VERSION}"
    log "Starting new container..."
    docker run -d \
        --name "${APP_NAME}" \
        --restart unless-stopped \
        -e APP_ENV="${ENV}" \
        -p 8080:8080 \
        "$image"
}

# ── Health Check ────────────────────────────────────
health_check() {
    log "Waiting for health check..."
    local retries=10
    local delay=3
    
    for ((i = 1; i <= retries; i++)); do
        if curl -sf http://localhost:8080/health &>/dev/null; then
            log "Health check passed (attempt $i)"
            return 0
        fi
        sleep "$delay"
    done
    
    log "Health check FAILED after $retries attempts"
    return 1
}

# ── Rollback ────────────────────────────────────────
rollback() {
    log "Rolling back to previous version..."
    stop_old
    docker start "${APP_NAME}-backup" 2>/dev/null || log "No backup available!"
}

# ── Main ────────────────────────────────────────────
main() {
    log "=== Deploying ${APP_NAME}:${VERSION} to ${ENV} ==="
    
    pre_check
    pull_image
    
    # Backup current
    docker rename "${APP_NAME}" "${APP_NAME}-backup" 2>/dev/null || true
    
    stop_old
    start_new
    
    if health_check; then
        docker rm "${APP_NAME}-backup" 2>/dev/null || true
        log "=== Deployment SUCCESS ==="
    else
        rollback
        log "=== Deployment FAILED — Rolled back ==="
        exit 1
    fi
}

main
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Variable leaks from function | Missing `local` keyword | Always use `local` for function variables |
| Function not found | Defined after call | Define functions before calling them |
| Return value > 255 | `return` only accepts 0-255 | Use `echo` for data, `return` for status |
| Function modifies global state | Not using `local` | Audit all variables, add `local` |
| Arguments with spaces break | Not quoting `$@` | Always use `"$@"` to pass arguments |
| Source file not found | Relative path issues | Use `SCRIPT_DIR` pattern with `BASH_SOURCE` |

---

## Summary Table

| Concept | Syntax | Example |
|---------|--------|---------|
| Define | `name() { ... }` | `deploy() { echo "go"; }` |
| Call | `name args` | `deploy "myapp"` |
| Arguments | `$1, $2, $@, $#` | `local app="$1"` |
| Local var | `local var=value` | `local count=0` |
| Return status | `return N` | `return 0` |
| Return data | `echo "data"` | `result=$(func)` |
| Source library | `source file.sh` | `source lib/common.sh` |
| Check exists | `type -t funcname` | `type -t deploy` |

---

## Quick Revision Questions

1. **What is the difference between `return` and `echo` for returning values from a function?**
2. **Why is `local` important and what happens without it?**
3. **How do you source a function library relative to the script's location?**
4. **Write a function that takes named arguments (e.g., `--name value`).**
5. **What does `$FUNCNAME` contain and how is it useful for debugging?**
6. **How do you pass all arguments from one function to another without losing spaces in arguments?**

---

[← Previous: Loops](04-loops.md) | [Back to README](../README.md) | [Next: Input/Output Redirection →](06-input-output-redirection.md)
