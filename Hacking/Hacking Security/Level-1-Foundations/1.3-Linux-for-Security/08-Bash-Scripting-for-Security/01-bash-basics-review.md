# Bash Basics Review

## Unit 8 - Topic 1 | Bash Scripting for Security

---

## Overview

**Bash** (Bourne Again Shell) is the default shell on most Linux systems and is essential for **security automation**, **system administration**, and **penetration testing**. This review covers the core scripting constructs needed before writing security-focused scripts.

---

## 1. Script Structure and Basics

```bash
#!/bin/bash
# Shebang line — tells system to use bash interpreter
# Always include this as the first line

# === VARIABLES ===
target="10.0.0.1"
port=22
wordlist="/usr/share/wordlists/rockyou.txt"

# Access variables with $
echo "Scanning $target on port $port"

# Command substitution
current_date=$(date +%Y-%m-%d)
hostname=$(hostname)
my_ip=$(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')

# === ARRAYS ===
ports=(21 22 80 443 8080 3306)
echo "First port: ${ports[0]}"
echo "All ports: ${ports[@]}"
echo "Number of ports: ${#ports[@]}"

# === READ USER INPUT ===
read -p "Enter target IP: " target
read -sp "Enter password: " password    # -s = silent (no echo)
```

---

## 2. Control Structures

```bash
# === IF/ELSE ===
if [ -f "/etc/shadow" ]; then
    echo "Shadow file exists"
elif [ -f "/etc/passwd" ]; then
    echo "Only passwd file found"
else
    echo "Neither file found"
fi

# Comparison operators
# -eq, -ne, -lt, -gt, -le, -ge    (numbers)
# ==, !=                           (strings)
# -z (empty), -n (not empty)       (strings)
# -f (file exists), -d (directory), -r (readable)

# === FOR LOOPS ===
# Loop through array
for port in "${ports[@]}"; do
    echo "Checking port $port"
done

# Loop through range
for i in $(seq 1 254); do
    ping -c 1 -W 1 10.0.0.$i &>/dev/null && echo "10.0.0.$i is UP"
done

# C-style for loop
for ((i=1; i<=100; i++)); do
    echo "Attempt $i"
done

# === WHILE LOOPS ===
while read -r line; do
    echo "Processing: $line"
done < input_file.txt

# Read from command output
ss -tlnp | while read -r line; do
    echo "$line"
done

# === CASE STATEMENT ===
case "$1" in
    scan)   echo "Starting scan..." ;;
    report) echo "Generating report..." ;;
    *)      echo "Usage: $0 {scan|report}" ;;
esac
```

---

## 3. Functions

```bash
# Define function
scan_port() {
    local target=$1
    local port=$2
    timeout 1 bash -c "echo >/dev/tcp/$target/$port" 2>/dev/null
    if [ $? -eq 0 ]; then
        echo "[OPEN] $target:$port"
        return 0
    else
        echo "[CLOSED] $target:$port"
        return 1
    fi
}

# Call function
scan_port "10.0.0.1" 80
scan_port "10.0.0.1" 443

# Function with return value
check_root() {
    if [ "$(id -u)" -eq 0 ]; then
        return 0
    else
        return 1
    fi
}

if check_root; then
    echo "Running as root"
else
    echo "Error: Run as root!" && exit 1
fi
```

---

## 4. Text Processing for Security

```bash
# === GREP ===
grep "Failed password" /var/log/auth.log
grep -c "Failed password" /var/log/auth.log          # Count
grep -i "error" /var/log/syslog                      # Case insensitive
grep -oP '\d+\.\d+\.\d+\.\d+' /var/log/auth.log     # Extract IPs

# === AWK ===
# Extract specific fields
awk '{print $1, $3}' /var/log/auth.log               # Fields 1 and 3
awk -F: '{print $1, $7}' /etc/passwd                  # User and shell
awk '/Failed/ {count++} END {print count}' /var/log/auth.log

# === SED ===
sed 's/http:/https:/g' urls.txt                       # Replace
sed -n '10,20p' logfile.txt                           # Lines 10-20
sed '/^#/d' config.txt                                # Remove comments

# === CUT ===
cut -d: -f1 /etc/passwd                               # Usernames
cut -d' ' -f1 /var/log/auth.log | sort | uniq -c     # Count per field

# === SORT + UNIQ ===
sort access.log | uniq -c | sort -rn | head -10       # Top 10 entries
```

---

## 5. Essential Patterns for Security Scripts

```bash
# === ERROR HANDLING ===
set -euo pipefail                     # Exit on error, undefined vars, pipe fails

# === LOGGING ===
LOG_FILE="/var/log/security_scan.log"
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}
log "Scan started"

# === ARGUMENT PARSING ===
while getopts "t:p:o:h" opt; do
    case $opt in
        t) TARGET="$OPTARG" ;;
        p) PORT="$OPTARG" ;;
        o) OUTPUT="$OPTARG" ;;
        h) echo "Usage: $0 -t target -p port -o output"; exit 0 ;;
        *) echo "Invalid option"; exit 1 ;;
    esac
done

# === TEMPORARY FILES ===
tmpfile=$(mktemp)
trap "rm -f $tmpfile" EXIT            # Cleanup on exit
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Shebang** | `#!/bin/bash` — always first line |
| **Variables** | `var=value`, access with `$var` |
| **Arrays** | `arr=(a b c)`, access `${arr[@]}` |
| **Functions** | Reusable code blocks with parameters |
| **Text processing** | grep, awk, sed, cut — essential for parsing |
| **Error handling** | `set -euo pipefail` — fail safely |

---

## Quick Revision Questions

1. **What does `#!/bin/bash` do?**
2. **How do you read user input silently (for passwords)?**
3. **What does `set -euo pipefail` do?**
4. **How do you extract all IP addresses from a log file?**
5. **Write a bash function that checks if the script is running as root.**

---

[Next: Security Automation Scripts →](02-security-automation-scripts.md)

---

*Unit 8 - Topic 1 of 5 | [Back to README](../README.md)*
