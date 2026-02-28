# Chapter 7: Pipes & Filters

[← Previous: Input/Output Redirection](06-input-output-redirection.md) | [Back to README](../README.md) | [Next: Error Handling →](08-error-handling.md)

---

## Overview

Pipes connect commands together, sending one command's output as input to the next. This "Unix philosophy" of combining small, single-purpose tools into powerful pipelines is central to DevOps workflows — parsing logs, transforming data, and building monitoring one-liners.

---

## 7.1 The Pipe Operator `|`

```bash
# Basic pipe: output of cmd1 → input of cmd2
ls -la | less
cat /var/log/syslog | grep "error"

# Chaining multiple pipes
cat /var/log/auth.log | grep "Failed" | awk '{print $11}' | sort | uniq -c | sort -rn | head
```

```
┌──────────────────────────────────────────────────────────────┐
│                  PIPE DATA FLOW                               │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  cmd1  ──stdout──►  cmd2  ──stdout──►  cmd3  ──stdout──►     │
│                                                               │
│  ┌──────┐   pipe   ┌──────┐   pipe   ┌──────┐               │
│  │ cat  ├─────────►│ grep ├─────────►│ sort │               │
│  │ log  │  stdout  │filter│  stdout  │order │               │
│  └──────┘  →stdin  └──────┘  →stdin  └──────┘               │
│                                                               │
│  Each command runs in its own process/subshell                 │
│  Data flows left to right, one line at a time                 │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 7.2 Essential Filter Commands

### `grep` — Pattern Matching

```bash
# Basic grep
grep "error" /var/log/syslog
grep -i "error" log.txt            # Case insensitive
grep -v "debug" log.txt            # Invert (exclude)
grep -c "error" log.txt            # Count matches
grep -n "error" log.txt            # Line numbers
grep -r "TODO" /app/src/           # Recursive
grep -l "password" /etc/*          # Files with match only
grep -E "error|warning|critical" log.txt   # Extended regex

# Grep with context
grep -B 3 "error" log.txt         # 3 lines Before
grep -A 5 "error" log.txt         # 5 lines After
grep -C 2 "error" log.txt         # 2 lines Context (both)
```

### `sort` — Sorting

```bash
sort file.txt                      # Alphabetical
sort -n file.txt                   # Numeric
sort -r file.txt                   # Reverse
sort -k 2 file.txt                 # By 2nd field
sort -t: -k3 -n /etc/passwd        # By UID (field 3, : delimiter)
sort -u file.txt                   # Sort + unique
sort -h sizes.txt                  # Human-readable (1K, 2M, 3G)
```

### `uniq` — Deduplicate

```bash
sort file.txt | uniq               # Remove duplicates (must sort first!)
sort file.txt | uniq -c            # Count occurrences
sort file.txt | uniq -d            # Show only duplicates
sort file.txt | uniq -u            # Show only unique lines
```

### `cut` — Extract Columns

```bash
cut -d: -f1 /etc/passwd            # Field 1, delimiter :
cut -d, -f1,3 data.csv             # Fields 1 and 3
cut -c1-10 file.txt                # Characters 1-10
echo "hello:world" | cut -d: -f2   # world
```

### `tr` — Translate/Delete Characters

```bash
echo "Hello" | tr 'a-z' 'A-Z'     # HELLO (uppercase)
echo "Hello" | tr 'A-Z' 'a-z'     # hello (lowercase)
echo "Hello   World" | tr -s ' '  # Hello World (squeeze spaces)
echo "Hello123" | tr -d '0-9'     # Hello (delete digits)
cat file.txt | tr '\n' ','         # Join lines with comma
```

### `awk` — Field Processing

```bash
# Print specific fields
awk '{print $1, $3}' file.txt
awk -F: '{print $1, $3}' /etc/passwd    # Custom delimiter

# Filter + print
awk '$3 > 1000 {print $1}' /etc/passwd  # UID > 1000
ps aux | awk '$3 > 5 {print $11, $3"%"}'  # High CPU processes

# Sum a column
awk '{sum += $5} END {print "Total:", sum}' data.txt

# Format output
df -h | awk 'NR>1 {printf "%-20s %s\n", $6, $5}'
```

### `sed` — Stream Editor

```bash
# Substitution
sed 's/old/new/' file.txt           # First occurrence per line
sed 's/old/new/g' file.txt          # All occurrences
sed -i 's/old/new/g' file.txt       # In-place edit

# Delete lines
sed '/^#/d' file.txt                # Delete comment lines
sed '/^$/d' file.txt                # Delete empty lines
sed '1,5d' file.txt                 # Delete lines 1-5

# Print specific lines
sed -n '10,20p' file.txt            # Lines 10-20
sed -n '/START/,/END/p' file.txt    # Between patterns
```

---

## 7.3 Common Pipeline Patterns

```bash
# Top 10 largest files
du -sh /var/log/* 2>/dev/null | sort -rh | head -10

# Most frequent IPs in access log
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -20

# Count HTTP status codes
awk '{print $9}' access.log | sort | uniq -c | sort -rn

# Find failed SSH login attempts
grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -rn

# Disk usage by directory (sorted)
du -sh /home/* | sort -rh

# Extract unique email addresses from file
grep -oE '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' file.txt | sort -u

# Monitor log in real-time, filtered
tail -f /var/log/syslog | grep --line-buffered "error"

# Process list filtered and formatted
ps aux | awk 'NR==1 || $3>1' | sort -k3 -rn | head -20
```

---

## 7.4 Named Pipes (FIFOs)

```bash
# Create named pipe
mkfifo /tmp/mypipe

# Writer (terminal 1)
echo "Hello from writer" > /tmp/mypipe

# Reader (terminal 2) — blocks until data arrives
cat < /tmp/mypipe

# Cleanup
rm /tmp/mypipe
```

```
┌──────────────────────────────────────────────────────────────┐
│           NAMED PIPE (FIFO) FLOW                              │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Process A                                   Process B        │
│  ┌────────┐    ┌──────────────┐    ┌────────┐                │
│  │ Writer ├───►│ /tmp/mypipe  ├───►│ Reader │                │
│  │        │    │ (FIFO file)  │    │        │                │
│  └────────┘    └──────────────┘    └────────┘                │
│                                                               │
│  · Persists in filesystem (unlike |)                          │
│  · Both sides must be open for data to flow                   │
│  · Blocks until both reader and writer connect                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 7.5 `xargs` — Build Commands from Input

```bash
# Basic: pass lines as arguments
echo "file1.txt file2.txt file3.txt" | xargs rm

# One argument at a time
find /tmp -name "*.log" | xargs -I{} mv {} /var/log/archive/

# Parallel execution
find . -name "*.jpg" | xargs -P4 -I{} convert {} -resize 50% thumb_{}

# With confirmation
echo "file1 file2 file3" | xargs -p rm

# Handle filenames with spaces
find . -name "*.log" -print0 | xargs -0 rm

# Limit arguments per command
echo {1..100} | xargs -n 10 echo
```

---

## 7.6 `pipefail` and Error Handling

```bash
# Without pipefail: exit code = LAST command
false | true
echo $?    # 0 (from true, even though false failed!)

# With pipefail: exit code = LAST failed command
set -o pipefail
false | true
echo $?    # 1 (from false)

# Check pipe status array
ls /nonexistent 2>/dev/null | sort
echo "${PIPESTATUS[@]}"    # 2 0 (ls failed, sort succeeded)

# Best practice in scripts
set -euo pipefail
```

---

## 7.7 Real-World Scenario: Log Analysis Pipeline

```bash
#!/usr/bin/env bash
# log-analyzer.sh — Analyze Nginx access logs using pipe chains

set -euo pipefail

LOG="${1:-/var/log/nginx/access.log}"

[[ -f "$LOG" ]] || { echo "Log file not found: $LOG"; exit 1; }

echo "=============================================="
echo "  NGINX ACCESS LOG ANALYSIS"
echo "  File: $LOG"
echo "  Lines: $(wc -l < "$LOG")"
echo "=============================================="

echo ""
echo "--- Top 10 IP Addresses ---"
awk '{print $1}' "$LOG" | sort | uniq -c | sort -rn | head -10 | \
    awk '{printf "  %6d  %s\n", $1, $2}'

echo ""
echo "--- HTTP Status Code Distribution ---"
awk '{print $9}' "$LOG" | grep -E '^[0-9]+$' | sort | uniq -c | sort -rn | \
    awk '{printf "  %6d  HTTP %s\n", $1, $2}'

echo ""
echo "--- Top 10 Requested URLs ---"
awk '{print $7}' "$LOG" | sort | uniq -c | sort -rn | head -10 | \
    awk '{printf "  %6d  %s\n", $1, $2}'

echo ""
echo "--- Requests per Hour ---"
awk '{print $4}' "$LOG" | cut -d: -f2 | sort | uniq -c | \
    awk '{printf "  %6d  %02d:00\n", $1, $2}'

echo ""
echo "--- Top 10 User Agents ---"
awk -F'"' '{print $6}' "$LOG" | sort | uniq -c | sort -rn | head -10 | \
    awk '{$1=$1; printf "  %6d  %s\n", $1, substr($0, index($0,$2))}'

echo ""
echo "--- Error Rate ---"
TOTAL=$(wc -l < "$LOG")
ERRORS=$(awk '$9 >= 400' "$LOG" | wc -l)
if (( TOTAL > 0 )); then
    RATE=$(echo "scale=2; $ERRORS * 100 / $TOTAL" | bc)
    echo "  Total: $TOTAL | Errors (4xx/5xx): $ERRORS | Rate: ${RATE}%"
else
    echo "  No log entries"
fi

echo ""
echo "=============================================="
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Pipe hides errors | Only last command's exit code checked | Use `set -o pipefail` |
| Variable lost in pipe loop | Pipe creates subshell | Use `< <(cmd)` process substitution |
| `grep` buffering in `tail -f` pipe | Line buffering disabled | Use `grep --line-buffered` |
| `xargs: argument line too long` | Too many args | Use `-n` to limit batch size |
| Filenames with spaces break pipe | Word splitting | Use `find -print0 \| xargs -0` |
| Wrong field in `awk` | Delimiter mismatch | Check separator with `-F` |

---

## Summary Table

| Tool | Purpose | Key Usage |
|------|---------|-----------|
| `\|` | Connect stdout → stdin | `cmd1 \| cmd2` |
| `grep` | Filter lines by pattern | `grep -i "error" log.txt` |
| `sort` | Sort lines | `sort -rn -k2` |
| `uniq` | Remove duplicates | `sort \| uniq -c` |
| `cut` | Extract fields | `cut -d: -f1` |
| `tr` | Translate characters | `tr 'a-z' 'A-Z'` |
| `awk` | Field processing | `awk '{print $1}'` |
| `sed` | Stream editing | `sed 's/old/new/g'` |
| `xargs` | Build commands from stdin | `find \| xargs rm` |
| `tee` | Split output | `cmd \| tee file` |

---

## Quick Revision Questions

1. **What is the difference between `|` and `>`?**
2. **Why must you `sort` before using `uniq`?**
3. **Write a pipeline that finds the top 5 largest directories in `/var`.**
4. **What does `set -o pipefail` do and why is it important?**
5. **How do you handle filenames with spaces when piping `find` to `xargs`?**
6. **What is the difference between a regular pipe `|` and a named pipe (FIFO)?**

---

[← Previous: Input/Output Redirection](06-input-output-redirection.md) | [Back to README](../README.md) | [Next: Error Handling →](08-error-handling.md)
