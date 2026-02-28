# Chapter 4: Loops

[← Previous: Conditionals](03-conditionals.md) | [Back to README](../README.md) | [Next: Functions →](05-functions.md)

---

## Overview

Loops let you repeat actions — iterating over files, servers, log entries, or retrying operations until they succeed. Bash provides `for`, `while`, and `until` loops, each suited to different DevOps automation patterns.

---

## 4.1 The `for` Loop

### List-Style For Loop

```bash
# Iterate over a list
for fruit in apple banana cherry; do
    echo "I like $fruit"
done

# Iterate over array
SERVERS=("web01" "web02" "db01" "cache01")
for server in "${SERVERS[@]}"; do
    echo "Checking $server..."
    ping -c 1 -W 2 "$server" &>/dev/null && echo "  UP" || echo "  DOWN"
done

# Iterate over command output
for user in $(cut -d: -f1 /etc/passwd); do
    echo "User: $user"
done

# Iterate over files
for file in /var/log/*.log; do
    echo "$file: $(wc -l < "$file") lines"
done

# Iterate over arguments
for arg in "$@"; do
    echo "Argument: $arg"
done
```

### C-Style For Loop

```bash
# Traditional counting loop
for ((i = 0; i < 10; i++)); do
    echo "Iteration: $i"
done

# Step by 2
for ((i = 0; i <= 20; i += 2)); do
    echo "$i"
done

# Countdown
for ((i = 5; i > 0; i--)); do
    echo "T-minus $i..."
    sleep 1
done
echo "Launch!"
```

### Range-Based For Loop

```bash
# Using brace expansion
for i in {1..10}; do
    echo "Number: $i"
done

# With step
for i in {0..100..5}; do
    echo "$i"
done

# Using seq (allows variables)
END=10
for i in $(seq 1 "$END"); do
    echo "$i"
done
```

---

## 4.2 The `while` Loop

```bash
# Basic while loop
COUNT=1
while [[ $COUNT -le 5 ]]; do
    echo "Count: $COUNT"
    ((COUNT++))
done

# Infinite loop with break
while true; do
    read -p "Enter 'quit' to exit: " input
    [[ "$input" == "quit" ]] && break
    echo "You entered: $input"
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < /etc/hosts

# Read from command output (process substitution)
while IFS= read -r line; do
    echo "Process: $line"
done < <(ps aux | grep nginx)
```

```
┌──────────────────────────────────────────────────────────────┐
│              LOOP SELECTION GUIDE                              │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Need to iterate over a list?                                 │
│    └─► for item in list                                       │
│                                                               │
│  Need to count with a variable?                               │
│    └─► for ((i=0; i<N; i++))                                  │
│                                                               │
│  Need to loop while condition is true?                        │
│    └─► while [[ condition ]]                                  │
│                                                               │
│  Need to loop until condition becomes true?                   │
│    └─► until [[ condition ]]                                  │
│                                                               │
│  Need to read a file line by line?                            │
│    └─► while IFS= read -r line; do ... done < file           │
│                                                               │
│  Need infinite loop with exit condition?                      │
│    └─► while true; do ... [[ cond ]] && break; done          │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.3 The `until` Loop

```bash
# Runs UNTIL condition becomes true (opposite of while)
COUNT=1
until [[ $COUNT -gt 5 ]]; do
    echo "Count: $COUNT"
    ((COUNT++))
done

# Wait for service to be ready
until curl -sf http://localhost:8080/health &>/dev/null; do
    echo "Waiting for service to start..."
    sleep 2
done
echo "Service is ready!"
```

---

## 4.4 Loop Control: `break` and `continue`

```bash
# break — Exit the loop entirely
for i in {1..100}; do
    if (( i > 5 )); then
        break
    fi
    echo "$i"
done

# continue — Skip current iteration
for file in /var/log/*; do
    [[ ! -f "$file" ]] && continue   # Skip non-files
    [[ ! -r "$file" ]] && continue   # Skip unreadable
    echo "$file: $(wc -l < "$file") lines"
done

# break with nested loops (break N)
for i in {1..3}; do
    for j in {1..3}; do
        if (( i == 2 && j == 2 )); then
            break 2    # Break out of BOTH loops
        fi
        echo "$i,$j"
    done
done
```

---

## 4.5 Reading Files with Loops

```bash
# Read CSV file
while IFS=',' read -r name ip role; do
    echo "Server: $name ($ip) — Role: $role"
done < servers.csv

# Read with line numbers
LINE_NUM=0
while IFS= read -r line; do
    ((LINE_NUM++))
    echo "${LINE_NUM}: $line"
done < config.txt

# Process /etc/passwd
while IFS=: read -r user _ uid gid _ home shell; do
    if (( uid >= 1000 )) && [[ "$shell" != "/usr/sbin/nologin" ]]; then
        echo "Regular user: $user (UID: $uid) Home: $home"
    fi
done < /etc/passwd

# Read from a here-string
while IFS= read -r server; do
    echo "Pinging $server..."
done <<< "web01
web02
db01"
```

---

## 4.6 Parallel Execution

```bash
# Run commands in background within loop
for server in web0{1..5}; do
    ssh "$server" "uptime" &    # Run in background
done
wait    # Wait for all background jobs

# With job limiting (max N parallel)
MAX_JOBS=3
for server in "${SERVERS[@]}"; do
    while (( $(jobs -r | wc -l) >= MAX_JOBS )); do
        sleep 0.5
    done
    (
        echo "[$server] Starting..."
        ssh "$server" "sudo apt update && sudo apt upgrade -y"
        echo "[$server] Done"
    ) &
done
wait
echo "All servers updated"

# Using xargs for parallel
echo "web01 web02 web03" | xargs -n1 -P3 -I{} ssh {} "uptime"

# Using GNU parallel (if installed)
parallel ssh {} "uptime" ::: web01 web02 web03
```

---

## 4.7 Common Loop Patterns in DevOps

### Retry Pattern

```bash
retry() {
    local max_retries="${1:-3}"
    local delay="${2:-5}"
    shift 2
    local cmd="$*"
    
    for ((attempt = 1; attempt <= max_retries; attempt++)); do
        echo "Attempt $attempt/$max_retries: $cmd"
        if eval "$cmd"; then
            echo "Success on attempt $attempt"
            return 0
        fi
        if (( attempt < max_retries )); then
            echo "Failed. Retrying in ${delay}s..."
            sleep "$delay"
        fi
    done
    echo "All $max_retries attempts failed"
    return 1
}

retry 3 5 curl -sf http://api.example.com/health
```

### Progress Indicator

```bash
ITEMS=("deploy" "migrate" "test" "verify" "cleanup")
TOTAL=${#ITEMS[@]}
for i in "${!ITEMS[@]}"; do
    CURRENT=$((i + 1))
    echo "[${CURRENT}/${TOTAL}] ${ITEMS[$i]}..."
    sleep 1   # Simulating work
done
echo "All $TOTAL tasks complete!"
```

### Watchdog Pattern

```bash
#!/bin/bash
# Restart service if it crashes
SERVICE="myapp"
CHECK_INTERVAL=30

while true; do
    if ! systemctl is-active --quiet "$SERVICE"; then
        echo "$(date): $SERVICE is down! Restarting..."
        systemctl restart "$SERVICE"
        sleep 5
        if systemctl is-active --quiet "$SERVICE"; then
            echo "$(date): $SERVICE restarted successfully"
        else
            echo "$(date): $SERVICE failed to restart!"
        fi
    fi
    sleep "$CHECK_INTERVAL"
done
```

---

## 4.8 Real-World Scenario: Bulk Server Health Check

```bash
#!/usr/bin/env bash
# health-check.sh — Check health of multiple servers

set -euo pipefail

# Server list (could also read from a file)
SERVERS=(
    "web01:80:/health"
    "web02:80:/health"
    "api01:8080:/api/status"
    "db01:5432:"
)

TIMEOUT=5
UP=0
DOWN=0

echo "=== Server Health Check — $(date) ==="
echo ""

for entry in "${SERVERS[@]}"; do
    IFS=: read -r host port path <<< "$entry"
    
    printf "%-12s port %-5s " "$host" "$port"
    
    if [[ -n "$path" ]]; then
        # HTTP check
        if curl -sf --connect-timeout "$TIMEOUT" \
           "http://${host}:${port}${path}" &>/dev/null; then
            echo "[  UP  ] HTTP 200"
            ((UP++))
        else
            echo "[ DOWN ] HTTP check failed"
            ((DOWN++))
        fi
    else
        # TCP port check
        if timeout "$TIMEOUT" bash -c "echo >/dev/tcp/$host/$port" 2>/dev/null; then
            echo "[  UP  ] Port open"
            ((UP++))
        else
            echo "[ DOWN ] Port closed"
            ((DOWN++))
        fi
    fi
done

TOTAL=$((UP + DOWN))
echo ""
echo "Summary: $UP/$TOTAL up, $DOWN/$TOTAL down"
(( DOWN > 0 )) && exit 1 || exit 0
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Loop variable has extra whitespace | IFS not set properly | Use `IFS= read -r` |
| `for f in $(ls)` breaks on spaces | Word splitting | Use `for f in ./*` instead |
| Infinite loop | Missing increment or break | Add timeout: `timeout 60 ./script.sh` |
| Variable empty after while loop | Loop in a pipe subshell | Use `done < <(command)` instead of pipe |
| Loop not iterating | Empty glob pattern | Use `shopt -s nullglob` |
| Slow sequential operations | Serial execution | Use `&` and `wait` for parallel |

---

## Summary Table

| Loop Type | Syntax | Best For |
|-----------|--------|----------|
| for (list) | `for x in list; do ... done` | Known items |
| for (C-style) | `for ((i=0;i<N;i++)); do ... done` | Counting |
| for (range) | `for i in {1..10}; do ... done` | Sequences |
| while | `while [[ cond ]]; do ... done` | Condition-based |
| until | `until [[ cond ]]; do ... done` | Wait-until |
| read loop | `while read -r line; do ... done < file` | File processing |
| infinite | `while true; do ... done` | Daemons/watchers |

---

## Quick Revision Questions

1. **How do you read a CSV file line by line, splitting each column?**
2. **What is the difference between `while` and `until`?**
3. **Why is `for f in $(ls *.log)` problematic? What's the correct alternative?**
4. **How do you limit parallel background jobs in a loop?**
5. **Write a retry loop that tries a command 5 times with 10-second delays.**
6. **Why does a variable modified inside a piped `while` loop appear unchanged after the loop?**

---

[← Previous: Conditionals](03-conditionals.md) | [Back to README](../README.md) | [Next: Functions →](05-functions.md)
