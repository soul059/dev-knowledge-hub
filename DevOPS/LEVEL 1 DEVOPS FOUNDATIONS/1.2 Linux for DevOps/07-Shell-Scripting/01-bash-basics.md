# Chapter 1: Bash Basics

[← Previous: Disk Monitoring](../06-Storage-Management/06-disk-monitoring.md) | [Back to README](../README.md) | [Next: Variables & Data Types →](02-variables-and-data-types.md)

---

## Overview

Bash (Bourne Again SHell) is the default shell on most Linux systems and the standard for DevOps automation scripts. This chapter covers script creation, execution, syntax fundamentals, and debugging techniques.

---

## 1.1 What is a Shell Script?

A shell script is a text file containing a sequence of commands that the shell executes line by line.

```
┌──────────────────────────────────────────────────────────────┐
│                    SCRIPT EXECUTION FLOW                      │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│   script.sh                                                   │
│   ┌──────────────┐                                            │
│   │ #!/bin/bash   │ ──► Kernel reads shebang, invokes bash    │
│   │ echo "Hello"  │ ──► Bash parses line, executes command    │
│   │ date          │ ──► Next command executed                  │
│   │ whoami        │ ──► Next command executed                  │
│   │ exit 0        │ ──► Script exits with status 0             │
│   └──────────────┘                                            │
│                                                               │
│   User ──► bash ──► fork/exec each command ──► output         │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.2 Creating & Running Your First Script

```bash
# Step 1: Create the script
cat > hello.sh << 'EOF'
#!/bin/bash
# My first script
echo "Hello, DevOps Engineer!"
echo "Today is: $(date '+%A, %B %d, %Y')"
echo "You are: $(whoami)"
echo "Working in: $(pwd)"
EOF

# Step 2: Make it executable
chmod +x hello.sh

# Step 3: Run it
./hello.sh
```

### Ways to Run a Script

```bash
# Method 1: Direct execution (requires +x and shebang)
./script.sh

# Method 2: Explicit interpreter
bash script.sh

# Method 3: Source (runs in CURRENT shell — affects environment)
source script.sh
. script.sh           # Same as source

# Method 4: Using env (portable)
/usr/bin/env bash script.sh
```

```
┌──────────────────────────────────────────────────────────────┐
│        EXECUTION vs SOURCING                                  │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│   ./script.sh (Execution)       source script.sh (Sourcing)  │
│   ┌─────────────┐               ┌─────────────┐              │
│   │ Parent Shell │               │ Current     │              │
│   │ (unchanged)  │               │ Shell       │              │
│   │  │           │               │  │          │              │
│   │  ▼           │               │  ▼          │              │
│   │ Child Shell  │               │ Commands    │              │
│   │ (new env)    │               │ run HERE    │              │
│   │  │ VAR=value │               │  │ VAR=value│              │
│   │  ▼           │               │  ▼          │              │
│   │ Child exits  │               │ VAR persists│              │
│   │ VAR is gone  │               │ in shell    │              │
│   └─────────────┘               └─────────────┘              │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.3 The Shebang Line

```bash
#!/bin/bash           # Use bash
#!/bin/sh             # Use POSIX sh (more portable, fewer features)
#!/usr/bin/env bash   # Portable — finds bash in $PATH
#!/usr/bin/env python3 # Can also run Python scripts this way
```

```bash
# Why shebang matters
file script.sh
# script.sh: Bash script, ASCII text executable
```

---

## 1.4 Comments

```bash
# This is a single-line comment

echo "code" # Inline comment

: '
This is a
multi-line comment
(actually a no-op command with a string argument)
'

# Best Practice: Header comment block
#!/bin/bash
#=============================================================
# Script Name:  deploy.sh
# Description:  Deploy application to production
# Author:       DevOps Team
# Date:         2024-01-15
# Version:      1.0
# Usage:        ./deploy.sh [environment] [version]
#=============================================================
```

---

## 1.5 Exit Codes

```bash
# Every command returns an exit code
# 0 = success, non-zero = failure

ls /tmp
echo $?   # 0 (success)

ls /nonexistent
echo $?   # 2 (failure)

# Set exit code in scripts
exit 0    # Success
exit 1    # General error
exit 2    # Misuse of shell command
exit 126  # Permission problem
exit 127  # Command not found
exit 128+N # Fatal error signal N
```

```
┌──────────────────────────────────────────────────────────────┐
│              COMMON EXIT CODES                                │
├──────────┬───────────────────────────────────────────────────┤
│ Code     │ Meaning                                           │
├──────────┼───────────────────────────────────────────────────┤
│ 0        │ Success                                           │
│ 1        │ General errors                                    │
│ 2        │ Misuse of shell builtins                          │
│ 126      │ Command found but not executable                  │
│ 127      │ Command not found                                 │
│ 128      │ Invalid exit argument                             │
│ 128+N    │ Process killed by signal N (e.g., 130 = Ctrl+C)  │
│ 255      │ Exit code out of range                            │
└──────────┴───────────────────────────────────────────────────┘
```

---

## 1.6 Command Chaining & Logic

```bash
# Sequential execution
cmd1 ; cmd2 ; cmd3

# AND — cmd2 runs only if cmd1 succeeds
cmd1 && cmd2

# OR — cmd2 runs only if cmd1 fails
cmd1 || cmd2

# Combined pattern (common in scripts)
mkdir /app && cd /app && echo "Ready" || echo "Failed"

# Guard pattern
[ -f config.yml ] && source config.yml || { echo "No config!"; exit 1; }

# Grouping with subshell
(cd /tmp && tar xzf archive.tar.gz)   # cd doesn't affect parent

# Grouping in current shell
{ echo "hello"; echo "world"; } > output.txt
```

---

## 1.7 Debugging Bash Scripts

```bash
# Method 1: Enable debug mode in script
#!/bin/bash -x          # Print each command before execution

# Method 2: Set within script
set -x     # Enable debug (trace)
set +x     # Disable debug

# Method 3: Debug specific section
set -x
problematic_command
another_command
set +x

# Method 4: Run with debug flag
bash -x script.sh

# Enable verbose + error checking
set -euxo pipefail
# -e  Exit on any error
# -u  Treat unset variables as errors
# -x  Print commands before executing
# -o pipefail  Pipe fails if any command in pipe fails
```

```bash
# Debugging toolkit
#!/bin/bash
set -euo pipefail

# Trap for debugging on error
trap 'echo "ERROR at line $LINENO: command \"$BASH_COMMAND\" failed with exit code $?"' ERR

# Debug logger
DEBUG=${DEBUG:-false}
debug_log() {
    [ "$DEBUG" = true ] && echo "[DEBUG $(date '+%T')] $*" >&2
}

debug_log "Script started"
debug_log "Working directory: $(pwd)"

# Usage: DEBUG=true ./script.sh
```

---

## 1.8 Script Template

```bash
#!/usr/bin/env bash
#=============================================================
# Script:      template.sh
# Description: Standard script template for DevOps
# Author:      Your Name
# Version:     1.0.0
#=============================================================

set -euo pipefail

# Constants
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"

# Colors (if terminal supports)
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'  # No Color

# Logging functions
log()   { echo -e "$(date '+%F %T') [INFO]  $*" | tee -a "$LOG_FILE"; }
warn()  { echo -e "$(date '+%F %T') ${YELLOW}[WARN]${NC}  $*" | tee -a "$LOG_FILE"; }
error() { echo -e "$(date '+%F %T') ${RED}[ERROR]${NC} $*" | tee -a "$LOG_FILE" >&2; }

# Cleanup on exit
cleanup() {
    log "Cleaning up..."
    # Remove temp files, etc.
}
trap cleanup EXIT

# Usage
usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS]

Options:
    -h, --help     Show this help message
    -v, --verbose  Enable verbose output
    -e, --env      Environment (dev|staging|prod)

Examples:
    $SCRIPT_NAME --env prod
    $SCRIPT_NAME -v -e staging
EOF
}

# Main
main() {
    log "Starting $SCRIPT_NAME"
    # Your code here
    log "Completed successfully"
}

main "$@"
```

---

## 1.9 Real-World Scenario: DevOps Starter Script

```bash
#!/usr/bin/env bash
# server-info.sh — Gather system info for a new server setup

set -euo pipefail

echo "============================================"
echo "  SERVER INFORMATION REPORT"
echo "  Generated: $(date)"
echo "============================================"

echo ""
echo "--- SYSTEM ---"
echo "Hostname:     $(hostname)"
echo "OS:           $(cat /etc/os-release | grep PRETTY_NAME | cut -d= -f2 | tr -d '"')"
echo "Kernel:       $(uname -r)"
echo "Arch:         $(uname -m)"
echo "Uptime:       $(uptime -p)"

echo ""
echo "--- CPU ---"
echo "Model:        $(grep 'model name' /proc/cpuinfo | head -1 | cut -d: -f2 | xargs)"
echo "Cores:        $(nproc)"
echo "Load Avg:     $(cat /proc/loadavg | awk '{print $1, $2, $3}')"

echo ""
echo "--- MEMORY ---"
free -h | grep -E 'Mem|Swap'

echo ""
echo "--- DISK ---"
df -hT | grep -v tmpfs | head -10

echo ""
echo "--- NETWORK ---"
ip -4 addr show | grep inet | awk '{print $NF, $2}'

echo ""
echo "--- DOCKER ---"
if command -v docker &>/dev/null; then
    echo "Docker:       $(docker --version)"
    echo "Containers:   $(docker ps -q | wc -l) running"
else
    echo "Docker:       Not installed"
fi

echo ""
echo "============================================"
echo "  Report complete"
echo "============================================"
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| `Permission denied` | Missing execute bit | `chmod +x script.sh` |
| `bad interpreter` | Wrong shebang path or Windows line endings | Use `dos2unix script.sh` or `sed -i 's/\r$//' script.sh` |
| `command not found` | Script not in PATH or missing `./` | Use `./script.sh` or add to PATH |
| Variables disappear after script | Running in subshell | Use `source script.sh` instead |
| Script stops on error | `set -e` is enabled | Handle errors explicitly or use `\|\| true` |
| Unexpected behavior | Spaces in variable expansion | Always quote: `"$variable"` |

---

## Summary Table

| Concept | Syntax | Example |
|---------|--------|---------|
| Shebang | `#!/bin/bash` | First line of script |
| Execute permission | `chmod +x` | `chmod +x deploy.sh` |
| Exit code | `$?` | `echo $?` |
| Debug mode | `set -x` | `bash -x script.sh` |
| Strict mode | `set -euo pipefail` | Top of production scripts |
| AND chain | `&&` | `mkdir dir && cd dir` |
| OR chain | `\|\|` | `cmd \|\| echo "Failed"` |
| Source | `source` or `.` | `. ~/.bashrc` |

---

## Quick Revision Questions

1. **What is the difference between running `./script.sh` and `source script.sh`?**
2. **What does the shebang `#!/usr/bin/env bash` do and why is it preferred?**
3. **What does `set -euo pipefail` do? Explain each flag.**
4. **How do you debug a specific section of a bash script?**
5. **What exit code does a script return if it's killed by SIGTERM (signal 15)?**
6. **Write a one-liner that creates a directory and enters it only if creation succeeds.**

---

[← Previous: Disk Monitoring](../06-Storage-Management/06-disk-monitoring.md) | [Back to README](../README.md) | [Next: Variables & Data Types →](02-variables-and-data-types.md)
