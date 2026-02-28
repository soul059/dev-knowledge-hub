# Chapter 9: Shell Scripting Best Practices

[← Previous: Error Handling](08-error-handling.md) | [Back to README](../README.md) | [Next: Log Management →](../08-System-Administration/01-log-management.md)

---

## Overview

Following best practices ensures your scripts are reliable, maintainable, and safe in production environments. This chapter compiles conventions, anti-patterns, security guidelines, and quality tools for writing professional-grade DevOps shell scripts.

---

## 9.1 Script Header Template

```bash
#!/usr/bin/env bash
#=============================================================
# Script:       deploy.sh
# Description:  Deploy application to target environment
# Author:       DevOps Team <devops@company.com>
# Created:      2024-01-15
# Modified:     2024-03-20
# Version:      2.1.0
# Usage:        ./deploy.sh --env prod --version 1.2.3
# Dependencies: docker, curl, jq
# Exit Codes:   0 = success, 1 = error, 2 = usage error
#=============================================================

set -euo pipefail
IFS=$'\n\t'
```

---

## 9.2 The Essential Checklist

```
┌──────────────────────────────────────────────────────────────┐
│         SHELL SCRIPT QUALITY CHECKLIST                        │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ☐ Shebang line (#!/usr/bin/env bash)                        │
│  ☐ set -euo pipefail at the top                              │
│  ☐ Header comment with description, usage, author            │
│  ☐ All variables quoted ("$var")                              │
│  ☐ Local variables in functions (local keyword)              │
│  ☐ Input validation (args, files, commands)                  │
│  ☐ Error handling (trap, exit codes)                         │
│  ☐ Cleanup on exit (trap cleanup EXIT)                       │
│  ☐ Meaningful exit codes (0, 1, 2)                           │
│  ☐ Logging with timestamps                                   │
│  ☐ Usage/help message (-h, --help)                           │
│  ☐ No hardcoded secrets                                      │
│  ☐ ShellCheck passes with no warnings                        │
│  ☐ Tested on target OS/shell                                  │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 9.3 Variable Best Practices

```bash
# DO: Always quote variables
echo "${name}"
rm -rf "${dir}/"
[ -f "${config}" ] && source "${config}"

# DON'T: Unquoted variables (word splitting, glob expansion)
# echo $name            ← Breaks on spaces
# rm -rf $dir/          ← DANGEROUS if empty
# [ -f $config ]        ← Error if empty

# DO: Use readonly for constants
readonly APP_NAME="myapp"
readonly CONFIG_DIR="/etc/myapp"
readonly MAX_RETRIES=5

# DO: Use meaningful names
log_file="/var/log/deploy.log"
retry_count=0
health_check_url="http://localhost:8080/health"

# DON'T: Single letters or cryptic names
# f="/var/log/deploy.log"
# x=0
# u="http://localhost:8080/health"

# DO: Use default values
port="${PORT:-8080}"
env="${ENV:-development}"
workers="${WORKERS:-$(nproc)}"

# DO: Use uppercase for environment/exported variables
export DB_HOST="localhost"
export LOG_LEVEL="info"

# DO: Use lowercase for local/script variables
local server_count=0
local deploy_dir="/opt/app"
```

---

## 9.4 Command Best Practices

```bash
# DO: Use $() instead of backticks
current_date=$(date '+%F')

# DON'T: Backticks (hard to read, can't nest)
# current_date=`date '+%F'`

# DO: Use [[ ]] instead of [ ] in bash
if [[ -f "$file" ]]; then echo "exists"; fi

# DO: Use (( )) for arithmetic
if (( count > 10 )); then echo "big"; fi

# DO: Check command existence
if command -v docker &>/dev/null; then
    echo "Docker available"
fi

# DON'T: Use which (non-portable)
# if which docker; then ...

# DO: Use arrays for lists
servers=("web01" "web02" "db01")
for s in "${servers[@]}"; do echo "$s"; done

# DON'T: Use space-separated strings as lists
# servers="web01 web02 db01"
# for s in $servers; do echo "$s"; done

# DO: Use process substitution to avoid subshell
while IFS= read -r line; do
    ((count++))
done < <(grep "error" log.txt)

# DON'T: Pipe into while (variable lost)
# grep "error" log.txt | while read -r line; do ((count++)); done
```

---

## 9.5 Argument Parsing

```bash
#!/usr/bin/env bash
set -euo pipefail

# Default values
ENV="staging"
VERSION=""
DRY_RUN=false
VERBOSE=false

usage() {
    cat << EOF
Usage: $(basename "$0") [OPTIONS]

Deploy application to target environment.

Options:
    -e, --env ENV        Target environment (dev|staging|prod) [default: staging]
    -v, --version VER    Application version (required)
    -d, --dry-run        Show what would be done without executing
    -V, --verbose        Enable verbose output
    -h, --help           Show this help message

Examples:
    $(basename "$0") -e prod -v 1.2.3
    $(basename "$0") --env staging --version 2.0.0 --dry-run
EOF
    exit "${1:-0}"
}

# Parse arguments
while [[ $# -gt 0 ]]; do
    case "$1" in
        -e|--env)
            ENV="$2"
            shift 2
            ;;
        -v|--version)
            VERSION="$2"
            shift 2
            ;;
        -d|--dry-run)
            DRY_RUN=true
            shift
            ;;
        -V|--verbose)
            VERBOSE=true
            shift
            ;;
        -h|--help)
            usage 0
            ;;
        -*)
            echo "Unknown option: $1" >&2
            usage 1
            ;;
        *)
            echo "Unexpected argument: $1" >&2
            usage 1
            ;;
    esac
done

# Validate required arguments
[[ -z "$VERSION" ]] && { echo "Error: --version is required" >&2; usage 1; }

# Validate allowed values
case "$ENV" in
    dev|staging|prod) ;;
    *) echo "Error: invalid --env: $ENV" >&2; usage 1 ;;
esac

echo "Deploying version $VERSION to $ENV (dry_run=$DRY_RUN)"
```

---

## 9.6 Security Best Practices

```bash
# NEVER hardcode secrets
# DON'T:
# DB_PASSWORD="s3cret123"

# DO: Use environment variables or secret managers
DB_PASSWORD="${DB_PASSWORD:?ERROR: DB_PASSWORD not set}"

# DO: Use restrictive permissions
chmod 700 script.sh        # Owner only
chmod 600 config.env       # Owner read/write only

# DO: Clean sensitive variables
unset DB_PASSWORD
unset API_KEY

# DO: Use mktemp for temporary files
TEMP_FILE=$(mktemp)
TEMP_DIR=$(mktemp -d)
trap 'rm -rf "$TEMP_FILE" "$TEMP_DIR"' EXIT

# DON'T: Use predictable temp files
# TEMP_FILE="/tmp/myscript.tmp"    ← Race condition / symlink attack

# DO: Validate paths to prevent directory traversal
sanitize_path() {
    local path="$1"
    local base_dir="$2"
    local real_path
    real_path=$(realpath -m "$path")
    if [[ "$real_path" != "$base_dir"* ]]; then
        echo "ERROR: Path outside allowed directory" >&2
        return 1
    fi
    echo "$real_path"
}

# DO: Be careful with eval
# DON'T: eval "$user_input"
# DO: Use arrays or direct execution instead
```

```
┌──────────────────────────────────────────────────────────────┐
│         SECURITY ANTI-PATTERNS                                │
├─────────────────────────┬────────────────────────────────────┤
│ Anti-Pattern            │ Secure Alternative                 │
├─────────────────────────┼────────────────────────────────────┤
│ Hardcoded passwords     │ Env vars, secret managers          │
│ eval "$user_input"      │ Parameterized commands             │
│ /tmp/fixed_name         │ mktemp                             │
│ chmod 777               │ chmod 750 or less                  │
│ curl | bash             │ Download, verify, then run         │
│ Storing secrets in git  │ .env (gitignored), vault           │
│ Running as root         │ Drop privileges, use sudo          │
│ Unvalidated input       │ Input validation + sanitization    │
└─────────────────────────┴────────────────────────────────────┘
```

---

## 9.7 ShellCheck — Static Analysis

```bash
# Install
sudo apt install shellcheck       # Ubuntu
sudo dnf install ShellCheck       # RHEL
brew install shellcheck           # macOS

# Lint a script
shellcheck script.sh

# Ignore specific rules
# shellcheck disable=SC2086
echo $unquoted_var

# Inline for whole file
# shellcheck disable=SC2034,SC2086

# In CI/CD pipeline
shellcheck --severity=error deploy.sh || exit 1
shellcheck --format=json *.sh > lint-results.json
```

```
┌──────────────────────────────────────────────────────────────┐
│        COMMON SHELLCHECK WARNINGS                             │
├──────────────┬───────────────────────────────────────────────┤
│ Code         │ Issue                                         │
├──────────────┼───────────────────────────────────────────────┤
│ SC2086       │ Double quote to prevent splitting             │
│ SC2046       │ Quote to prevent word splitting               │
│ SC2034       │ Variable appears unused                       │
│ SC2155       │ Declare and assign separately (local)         │
│ SC2164       │ Use cd ... || exit if cd might fail           │
│ SC2006       │ Use $() instead of backticks                  │
│ SC2015       │ A && B || C is not if-then-else               │
│ SC1091       │ Source file not found for checking             │
│ SC2181       │ Check exit code directly, not $?              │
│ SC2129       │ Group commands and redirect once              │
└──────────────┴───────────────────────────────────────────────┘
```

---

## 9.8 Testing Shell Scripts

```bash
# Using bats (Bash Automated Testing System)
# Install: npm install -g bats

# test/deploy.bats
#!/usr/bin/env bats

@test "validate_version accepts semver" {
    source ./lib/validate.sh
    run validate_version "1.2.3"
    [ "$status" -eq 0 ]
}

@test "validate_version rejects invalid" {
    source ./lib/validate.sh
    run validate_version "not-a-version"
    [ "$status" -eq 1 ]
}

@test "deploy requires version argument" {
    run ./deploy.sh
    [ "$status" -ne 0 ]
    [[ "$output" == *"version"* ]]
}

# Run tests
bats test/
```

---

## 9.9 Script Organization

```
┌──────────────────────────────────────────────────────────────┐
│         RECOMMENDED PROJECT STRUCTURE                         │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  scripts/                                                     │
│  ├── deploy.sh              # Main deployment script          │
│  ├── rollback.sh            # Rollback script                 │
│  ├── setup.sh               # Environment setup               │
│  ├── lib/                                                     │
│  │   ├── common.sh          # Shared functions                │
│  │   ├── logging.sh         # Logging utilities               │
│  │   ├── validate.sh        # Input validation                │
│  │   └── docker.sh          # Docker helpers                  │
│  ├── config/                                                  │
│  │   ├── dev.env                                              │
│  │   ├── staging.env                                          │
│  │   └── prod.env                                             │
│  └── test/                                                    │
│      ├── deploy.bats        # Deployment tests                │
│      └── validate.bats      # Validation tests                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 9.10 Performance Tips

```bash
# DO: Use built-in string operations instead of external commands
# Instead of: result=$(echo "$str" | sed 's/old/new/')
result="${str/old/new}"

# Instead of: length=$(echo "$str" | wc -c)
length=${#str}

# Instead of: upper=$(echo "$str" | tr 'a-z' 'A-Z')
upper="${str^^}"

# DO: Avoid useless cat
# Instead of: cat file | grep "pattern"
grep "pattern" file

# DO: Use printf instead of echo for reliable output
printf '%s\n' "$variable"   # Safe, no escape interpretation
printf '%d items found\n' "$count"

# DO: Read files into variables efficiently
content=$(<file.txt)        # Faster than $(cat file.txt)

# DO: Avoid subshells when possible
# Instead of: value=$(( count + 1 ))
(( value = count + 1 ))
```

---

## 9.11 Real-World Scenario: Production-Ready Script

```bash
#!/usr/bin/env bash
#=============================================================
# Script:       healthcheck.sh
# Description:  Multi-service health checker following best practices
# Version:      1.0.0
#=============================================================

set -euo pipefail
IFS=$'\n\t'

# ── Constants ────────────────────────────────────────
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly VERSION="1.0.0"
readonly DEFAULT_TIMEOUT=5
readonly DEFAULT_CONFIG="${SCRIPT_DIR}/config/services.conf"

# ── Colors ───────────────────────────────────────────
if [[ -t 1 ]]; then
    readonly RED='\033[0;31m'
    readonly GREEN='\033[0;32m'
    readonly YELLOW='\033[1;33m'
    readonly NC='\033[0m'
else
    readonly RED='' GREEN='' YELLOW='' NC=''
fi

# ── Logging ──────────────────────────────────────────
log_info()  { printf "${GREEN}[INFO]${NC}  %s %s\n" "$(date '+%T')" "$*"; }
log_warn()  { printf "${YELLOW}[WARN]${NC}  %s %s\n" "$(date '+%T')" "$*"; }
log_error() { printf "${RED}[ERROR]${NC} %s %s\n" "$(date '+%T')" "$*" >&2; }

# ── Usage ────────────────────────────────────────────
usage() {
    cat << EOF
${SCRIPT_NAME} v${VERSION} — Multi-service health checker

Usage: ${SCRIPT_NAME} [OPTIONS]

Options:
    -c, --config FILE    Config file [default: ${DEFAULT_CONFIG}]
    -t, --timeout SEC    Timeout per check [default: ${DEFAULT_TIMEOUT}]
    -q, --quiet          Only show failures
    -h, --help           Show this help message

Config format (one per line):
    name|url|expected_status
    api|http://localhost:8080/health|200
    db|tcp://localhost:5432|open
EOF
    exit "${1:-0}"
}

# ── Parse Arguments ──────────────────────────────────
CONFIG="${DEFAULT_CONFIG}"
TIMEOUT="${DEFAULT_TIMEOUT}"
QUIET=false

while [[ $# -gt 0 ]]; do
    case "$1" in
        -c|--config)  CONFIG="$2"; shift 2 ;;
        -t|--timeout) TIMEOUT="$2"; shift 2 ;;
        -q|--quiet)   QUIET=true; shift ;;
        -h|--help)    usage 0 ;;
        *)            log_error "Unknown option: $1"; usage 1 ;;
    esac
done

# ── Validate ─────────────────────────────────────────
[[ -f "$CONFIG" ]] || { log_error "Config not found: $CONFIG"; exit 1; }
[[ "$TIMEOUT" =~ ^[0-9]+$ ]] || { log_error "Invalid timeout: $TIMEOUT"; exit 1; }

# ── Health Check Functions ───────────────────────────
check_http() {
    local name="$1" url="$2" expected="$3"
    local status
    status=$(curl -sf -o /dev/null -w '%{http_code}' \
        --connect-timeout "$TIMEOUT" "$url" 2>/dev/null || echo "000")
    
    if [[ "$status" == "$expected" ]]; then
        [[ "$QUIET" == false ]] && printf "  %-20s ${GREEN}UP${NC}   (HTTP %s)\n" "$name" "$status"
        return 0
    else
        printf "  %-20s ${RED}DOWN${NC} (HTTP %s, expected %s)\n" "$name" "$status" "$expected"
        return 1
    fi
}

check_tcp() {
    local name="$1" host="$2" port="$3"
    if timeout "$TIMEOUT" bash -c "echo >/dev/tcp/$host/$port" 2>/dev/null; then
        [[ "$QUIET" == false ]] && printf "  %-20s ${GREEN}UP${NC}   (port %s open)\n" "$name" "$port"
        return 0
    else
        printf "  %-20s ${RED}DOWN${NC} (port %s closed)\n" "$name" "$port"
        return 1
    fi
}

# ── Main ─────────────────────────────────────────────
main() {
    local up=0 down=0
    
    log_info "Health check starting (config: $CONFIG)"
    echo ""
    
    while IFS='|' read -r name url expected; do
        [[ "$name" =~ ^#.*$ || -z "$name" ]] && continue   # Skip comments/empty
        
        if [[ "$url" =~ ^http ]]; then
            check_http "$name" "$url" "$expected" && ((up++)) || ((down++))
        elif [[ "$url" =~ ^tcp://(.+):([0-9]+)$ ]]; then
            check_tcp "$name" "${BASH_REMATCH[1]}" "${BASH_REMATCH[2]}" && ((up++)) || ((down++))
        else
            log_warn "Unknown protocol for $name: $url"
        fi
    done < "$CONFIG"
    
    local total=$((up + down))
    echo ""
    log_info "Results: ${up}/${total} services up, ${down}/${total} down"
    
    (( down > 0 )) && exit 1 || exit 0
}

main
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Script works locally, fails in CI | Different shell or PATH | Use `#!/usr/bin/env bash`, check `$PATH` |
| Windows line endings (`\r`) | Edited on Windows | Use `dos2unix` or `sed -i 's/\r$//'` |
| ShellCheck reports many issues | Script has legacy patterns | Fix incrementally, start with errors |
| Script works in zsh but not bash | Zsh-specific syntax | Test with `bash script.sh` explicitly |
| Performance is slow | Too many external commands | Use bash built-ins for string operations |
| Hard to debug | No logging | Add structured logging with log levels |

---

## Summary Table

| Best Practice | Rationale |
|---------------|-----------|
| `set -euo pipefail` | Fail fast, catch errors early |
| Quote all variables | Prevent word splitting & globbing |
| Use `local` in functions | Prevent variable scope leaks |
| Use `[[ ]]` over `[ ]` | Safer, more features in bash |
| Use `$()` over backticks | Readable, nestable |
| Use `mktemp` for temp files | Secure, no race conditions |
| Validate all inputs | Prevent unexpected behavior |
| Use ShellCheck | Catch bugs before runtime |
| Add `--help` / usage | Self-documenting scripts |
| Use structured logging | Debuggable in production |
| Trap EXIT for cleanup | Resources always freed |
| No hardcoded secrets | Security & portability |

---

## Quick Revision Questions

1. **What are the components of `set -euo pipefail` and why is each important?**
2. **Why should you use `[[ ]]` instead of `[ ]` in bash scripts?**
3. **How do you securely create temporary files in a shell script?**
4. **What is ShellCheck and how do you integrate it into a CI/CD pipeline?**
5. **Write a complete argument parser that handles `--env`, `--version`, and `--help`.**
6. **What are three security anti-patterns in shell scripting and their solutions?**

---

[← Previous: Error Handling](08-error-handling.md) | [Back to README](../README.md) | [Next: Log Management →](../08-System-Administration/01-log-management.md)
