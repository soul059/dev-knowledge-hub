# Chapter 8: Error Handling

[← Previous: Pipes & Filters](07-pipes-and-filters.md) | [Back to README](../README.md) | [Next: Best Practices →](09-best-practices.md)

---

## Overview

Robust error handling is what separates a quick-and-dirty script from production-ready automation. In DevOps, scripts run unattended in CI/CD pipelines, cron jobs, and deployments — a silent failure can mean data loss or downtime. This chapter covers exit codes, traps, defensive coding, and logging strategies.

---

## 8.1 The Strict Mode

```bash
#!/usr/bin/env bash
# Always start production scripts with this
set -euo pipefail

# -e       Exit immediately if any command fails
# -u       Treat unset variables as errors
# -o pipefail  Return the exit code of the FIRST failed command in a pipeline
```

```
┌──────────────────────────────────────────────────────────────┐
│              STRICT MODE EXPLAINED                            │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Without strict mode:                                         │
│  ┌─────────────────────────────────────────────────┐         │
│  │  rm -rf "$UNDEFINED_VAR"/*   ← runs as rm -rf /*│         │
│  │  failing_command              ← error ignored    │         │
│  │  false | true                 ← pipe "succeeds"  │         │
│  │  echo "Reached this line"     ← keeps going!     │         │
│  └─────────────────────────────────────────────────┘         │
│                                                               │
│  With set -euo pipefail:                                      │
│  ┌─────────────────────────────────────────────────┐         │
│  │  rm -rf "$UNDEFINED_VAR"/*   ← ERROR: unbound   │         │
│  │  failing_command              ← script EXITS     │         │
│  │  false | true                 ← pipe FAILS       │         │
│  │  echo "Never reached"        ← not executed      │         │
│  └─────────────────────────────────────────────────┘         │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 8.2 Exit Code Handling

```bash
# Check exit code explicitly
if ! docker pull myapp:latest; then
    echo "Failed to pull image"
    exit 1
fi

# Using $?
command_that_might_fail
EXIT_CODE=$?
if [ $EXIT_CODE -ne 0 ]; then
    echo "Command failed with exit code: $EXIT_CODE"
fi

# Allow specific commands to fail (with set -e)
set -e
some_optional_command || true       # Won't stop script
another_optional_command || :       # Same effect (: is no-op)

# Conditional execution
grep -q "pattern" file.txt && echo "Found" || echo "Not found"
```

---

## 8.3 Trap — Catching Signals & Errors

```bash
# Syntax: trap 'commands' SIGNAL_LIST

# Cleanup on exit (any exit — normal or error)
trap 'rm -rf "$TEMP_DIR"' EXIT

# Catch Ctrl+C
trap 'echo "Interrupted!"; exit 130' INT

# Catch termination signal
trap 'echo "Terminated!"; cleanup; exit 143' TERM

# Catch errors (when set -e triggers)
trap 'echo "ERROR at line $LINENO"' ERR

# Multiple signals
trap 'cleanup' EXIT INT TERM

# Remove a trap
trap - EXIT
```

```
┌──────────────────────────────────────────────────────────────┐
│             COMMON TRAP SIGNALS                               │
├──────────────┬────────────────────────────────────────────────┤
│ Signal       │ When Triggered                                 │
├──────────────┼────────────────────────────────────────────────┤
│ EXIT         │ Script exits (any reason)                      │
│ ERR          │ Command returns non-zero (with set -e)         │
│ INT          │ Ctrl+C (SIGINT)                                │
│ TERM         │ kill command (SIGTERM)                          │
│ HUP          │ Terminal closed (SIGHUP)                        │
│ DEBUG        │ Before each command (for tracing)              │
│ RETURN       │ After function/source returns                  │
└──────────────┴────────────────────────────────────────────────┘
```

---

## 8.4 Error Handling Patterns

### Pattern 1: Comprehensive Trap Handler

```bash
#!/usr/bin/env bash
set -euo pipefail

TEMP_DIR=""
LOCK_FILE="/tmp/deploy.lock"

cleanup() {
    local exit_code=$?
    echo ""
    if [ $exit_code -ne 0 ]; then
        echo "[FAILED] Script exited with code: $exit_code"
    fi
    # Remove temp files
    [ -n "$TEMP_DIR" ] && rm -rf "$TEMP_DIR"
    # Remove lock file
    [ -f "$LOCK_FILE" ] && rm -f "$LOCK_FILE"
    echo "[CLEANUP] Done"
    exit "$exit_code"
}

trap cleanup EXIT
trap 'echo "[INTERRUPTED] Caught SIGINT"; exit 130' INT
trap 'echo "[TERMINATED] Caught SIGTERM"; exit 143' TERM

TEMP_DIR=$(mktemp -d)
touch "$LOCK_FILE"

# Script work here...
```

### Pattern 2: Error Logging with Line Numbers

```bash
#!/usr/bin/env bash
set -euo pipefail

LOG_FILE="/var/log/deploy.log"

log() { echo "$(date '+%F %T') [$1] $2" | tee -a "$LOG_FILE"; }

on_error() {
    local line=$1
    local cmd=$2
    local code=$3
    log "ERROR" "Line $line: command '$cmd' exited with code $code"
    log "ERROR" "Stack trace:"
    local frame=0
    while caller $frame; do
        ((frame++))
    done 2>/dev/null | while read -r line func file; do
        log "ERROR" "  $file:$line in $func()"
    done
}

trap 'on_error $LINENO "$BASH_COMMAND" $?' ERR
```

### Pattern 3: Retry with Exponential Backoff

```bash
retry_with_backoff() {
    local max_retries="${1:-5}"
    local base_delay="${2:-1}"
    shift 2
    local cmd="$*"
    
    for ((attempt = 1; attempt <= max_retries; attempt++)); do
        if eval "$cmd"; then
            return 0
        fi
        
        if (( attempt < max_retries )); then
            local delay=$(( base_delay * (2 ** (attempt - 1)) ))
            # Add jitter (random 0-delay seconds)
            delay=$(( delay + RANDOM % delay ))
            echo "Attempt $attempt failed. Retrying in ${delay}s..."
            sleep "$delay"
        fi
    done
    
    echo "All $max_retries attempts failed for: $cmd"
    return 1
}

# Usage
retry_with_backoff 5 2 curl -sf http://api.example.com/health
retry_with_backoff 3 1 docker pull registry.example.com/app:latest
```

### Pattern 4: Lock File (Prevent Concurrent Runs)

```bash
LOCK_FILE="/tmp/$(basename "$0").lock"

acquire_lock() {
    if [ -f "$LOCK_FILE" ]; then
        local pid
        pid=$(cat "$LOCK_FILE")
        if kill -0 "$pid" 2>/dev/null; then
            echo "ERROR: Script already running (PID: $pid)"
            exit 1
        else
            echo "WARN: Stale lock file found, removing"
            rm -f "$LOCK_FILE"
        fi
    fi
    echo $$ > "$LOCK_FILE"
    trap 'rm -f "$LOCK_FILE"' EXIT
}

acquire_lock
# ... your script logic ...
```

---

## 8.5 Validating Input

```bash
# Check required arguments
if [ $# -lt 2 ]; then
    echo "Usage: $0 <environment> <version>"
    echo "  environment: dev | staging | prod"
    echo "  version:     semver (e.g., 1.2.3)"
    exit 1
fi

# Validate environment
ENV="$1"
case "$ENV" in
    dev|staging|prod) ;;
    *) echo "ERROR: Invalid environment: $ENV"; exit 1 ;;
esac

# Validate version format
VERSION="$2"
if [[ ! "$VERSION" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]]; then
    echo "ERROR: Invalid version format: $VERSION (expected X.Y.Z)"
    exit 1
fi

# Validate file exists
CONFIG="${3:-config.yml}"
if [[ ! -f "$CONFIG" ]]; then
    echo "ERROR: Config file not found: $CONFIG"
    exit 1
fi

# Validate command available
for cmd in docker kubectl jq; do
    if ! command -v "$cmd" &>/dev/null; then
        echo "ERROR: Required command not found: $cmd"
        exit 1
    fi
done
```

---

## 8.6 Timeout & Watchdog

```bash
# Timeout a command
timeout 30 curl -sf http://example.com
if [ $? -eq 124 ]; then
    echo "Command timed out"
fi

# Wait for condition with timeout
wait_for() {
    local description="$1"
    local timeout="${2:-60}"
    shift 2
    local cmd="$*"
    
    echo "Waiting for $description (timeout: ${timeout}s)..."
    local start=$SECONDS
    
    while ! eval "$cmd" 2>/dev/null; do
        if (( SECONDS - start >= timeout )); then
            echo "TIMEOUT: $description did not complete in ${timeout}s"
            return 1
        fi
        sleep 2
    done
    
    local elapsed=$(( SECONDS - start ))
    echo "$description ready (${elapsed}s)"
}

wait_for "database" 60 "pg_isready -h localhost"
wait_for "web server" 30 "curl -sf http://localhost:8080/health"
```

---

## 8.7 Real-World Scenario: Production Deploy with Full Error Handling

```bash
#!/usr/bin/env bash
#=============================================================
# deploy-production.sh — Production deployment with error handling
#=============================================================
set -euo pipefail

# ── Configuration ────────────────────────────────────
readonly APP="${1:?Usage: $0 <app> <version>}"
readonly VERSION="${2:?Usage: $0 <app> <version>}"
readonly DEPLOY_LOG="/var/log/deploys/${APP}-$(date '+%Y%m%d_%H%M%S').log"
readonly LOCK_FILE="/tmp/deploy-${APP}.lock"
readonly HEALTH_URL="http://localhost:8080/health"
readonly ROLLBACK_VERSION=$(docker inspect --format='{{.Config.Image}}' "$APP" 2>/dev/null || echo "none")

mkdir -p "$(dirname "$DEPLOY_LOG")"

# ── Logging ──────────────────────────────────────────
exec > >(tee -a "$DEPLOY_LOG") 2>&1

log()   { echo "$(date '+%T') [INFO]  $*"; }
warn()  { echo "$(date '+%T') [WARN]  $*"; }
error() { echo "$(date '+%T') [ERROR] $*" >&2; }

# ── Lock ─────────────────────────────────────────────
if [ -f "$LOCK_FILE" ]; then
    error "Deployment already in progress (lock: $LOCK_FILE)"
    exit 1
fi
echo $$ > "$LOCK_FILE"

# ── Cleanup & Rollback ──────────────────────────────
rollback() {
    error "DEPLOYMENT FAILED — Initiating rollback"
    if [[ "$ROLLBACK_VERSION" != "none" ]]; then
        log "Rolling back to: $ROLLBACK_VERSION"
        docker stop "$APP" 2>/dev/null || true
        docker rm "$APP" 2>/dev/null || true
        docker run -d --name "$APP" --restart unless-stopped "$ROLLBACK_VERSION"
        log "Rollback complete"
    else
        error "No previous version to rollback to!"
    fi
}

cleanup() {
    local exit_code=$?
    rm -f "$LOCK_FILE"
    if [ $exit_code -ne 0 ]; then
        rollback
    fi
    log "Log saved to: $DEPLOY_LOG"
}

trap cleanup EXIT
trap 'error "Interrupted by user"; exit 130' INT TERM

# ── Validation ───────────────────────────────────────
log "=== Deploying ${APP}:${VERSION} ==="

[[ "$VERSION" =~ ^[0-9]+\.[0-9]+\.[0-9]+$ ]] || {
    error "Invalid version: $VERSION"
    exit 1
}

for cmd in docker curl; do
    command -v "$cmd" &>/dev/null || { error "Missing: $cmd"; exit 1; }
done

# ── Deploy Steps ─────────────────────────────────────
log "Step 1/4: Pulling image..."
docker pull "registry.example.com/${APP}:${VERSION}"

log "Step 2/4: Stopping old container..."
docker stop "$APP" 2>/dev/null || true
docker rm "$APP" 2>/dev/null || true

log "Step 3/4: Starting new container..."
docker run -d \
    --name "$APP" \
    --restart unless-stopped \
    -e VERSION="$VERSION" \
    -p 8080:8080 \
    "registry.example.com/${APP}:${VERSION}"

log "Step 4/4: Health check..."
RETRIES=10
for ((i = 1; i <= RETRIES; i++)); do
    if curl -sf "$HEALTH_URL" &>/dev/null; then
        log "Health check passed (attempt $i/$RETRIES)"
        break
    fi
    if (( i == RETRIES )); then
        error "Health check failed after $RETRIES attempts"
        exit 1
    fi
    sleep 3
done

log "=== Deployment SUCCESS: ${APP}:${VERSION} ==="
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Script continues after error | `set -e` not enabled | Add `set -euo pipefail` |
| Trap not firing | Wrong signal name | Use `trap -l` to list valid signals |
| `set -e` exits unexpectedly | Command in `if` or `\|\|` | Those contexts disable `-e`; handle explicitly |
| Pipe error hidden | Only last command checked | Use `set -o pipefail` or `${PIPESTATUS[@]}` |
| Lock file left behind | No cleanup trap | Always `trap cleanup EXIT` |
| `unbound variable` error | Using `set -u` with optional var | Use `${VAR:-default}` for optional variables |

---

## Summary Table

| Technique | Syntax | Purpose |
|-----------|--------|---------|
| Strict mode | `set -euo pipefail` | Fail fast on errors |
| Exit on error | `set -e` | Stop on first failure |
| Unset check | `set -u` | Catch typos in variable names |
| Pipefail | `set -o pipefail` | Catch pipe failures |
| Trap EXIT | `trap 'cmd' EXIT` | Always cleanup |
| Trap ERR | `trap 'cmd' ERR` | Log error details |
| Allow failure | `cmd \|\| true` | Skip error with `set -e` |
| Retry | `for loop + sleep` | Transient failure recovery |
| Lock file | `echo $$ > lock` | Prevent concurrent runs |
| Timeout | `timeout N cmd` | Limit command duration |

---

## Quick Revision Questions

1. **What does each flag in `set -euo pipefail` do?**
2. **What is the difference between `trap 'cmd' EXIT` and `trap 'cmd' ERR`?**
3. **How do you allow a specific command to fail without triggering `set -e`?**
4. **Write a retry function with exponential backoff.**
5. **How do you prevent two instances of the same script from running simultaneously?**
6. **What is `${PIPESTATUS[@]}` and when would you use it?**

---

[← Previous: Pipes & Filters](07-pipes-and-filters.md) | [Back to README](../README.md) | [Next: Best Practices →](09-best-practices.md)
