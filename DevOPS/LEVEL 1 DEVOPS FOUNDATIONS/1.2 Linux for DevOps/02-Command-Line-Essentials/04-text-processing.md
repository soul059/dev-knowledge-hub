# Chapter 4: Text Processing

[← Previous: Text Viewing](03-text-viewing.md) | [Back to README](../README.md) | [Next: Finding Files →](05-finding-files.md)

---

## Overview

Text processing is where Linux command-line skills truly shine for DevOps. Commands like `grep`, `sed`, `awk`, `cut`, and `sort` let you search, transform, extract, and analyze text data from logs, configs, and command output. These are the workhorses of automation scripts and one-liner problem solving.

---

## 4.1 `grep` — Search Text with Patterns

```
┌───────────────────────────────────────────────────────────┐
│                    grep WORKFLOW                           │
│                                                            │
│  Input ──→ grep "pattern" ──→ Matching Lines              │
│                                                            │
│  Sources:                                                  │
│  • Files:     grep "error" /var/log/syslog                │
│  • Stdin:     cat log.txt | grep "error"                  │
│  • Commands:  ps aux | grep nginx                         │
│  • Recursive: grep -r "TODO" /opt/app/                   │
└───────────────────────────────────────────────────────────┘
```

```bash
# Basic search
grep "error" /var/log/syslog

# Case-insensitive
grep -i "error" /var/log/syslog

# Recursive search in directories
grep -r "password" /etc/
grep -rn "password" /etc/           # With line numbers

# Invert match (lines NOT containing pattern)
grep -v "debug" /var/log/syslog

# Count matches
grep -c "error" /var/log/syslog

# Show only matching part
grep -o "error[a-zA-Z]*" /var/log/syslog

# Show N lines before/after match (context)
grep -B 3 "error" logfile           # 3 lines Before
grep -A 5 "error" logfile           # 5 lines After
grep -C 2 "error" logfile           # 2 lines Context (before + after)

# Multiple patterns
grep -e "error" -e "warn" logfile
grep "error\|warn" logfile          # Same with OR
grep -E "error|warn|critical" logfile   # Extended regex

# Fixed string (no regex interpretation)
grep -F "192.168.1.1" access.log

# Show filenames only
grep -rl "password" /etc/
grep -rL "password" /etc/           # Files NOT matching

# Regex patterns
grep -E "^root" /etc/passwd         # Lines starting with "root"
grep -E "bash$" /etc/passwd         # Lines ending with "bash"
grep -E "[0-9]{3}\.[0-9]{3}" log   # IP-like patterns
grep -P "\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}" log   # Perl regex for IPs

# Exclude directories/files
grep -r "TODO" . --exclude-dir={.git,node_modules}
grep -r "TODO" . --include="*.py"

# DevOps one-liners
grep "Failed password" /var/log/auth.log | tail -20          # Failed SSH attempts
grep -c "200" /var/log/nginx/access.log                       # Count 200 responses
grep -E "50[0-9]" /var/log/nginx/access.log | wc -l          # Count 5xx errors
ps aux | grep "[n]ginx"                                       # Find nginx processes
env | grep -i proxy                                           # Check proxy settings
```

---

## 4.2 `sed` — Stream Editor

```
┌───────────────────────────────────────────────────────────┐
│                    sed WORKFLOW                            │
│                                                            │
│  Input ──→ sed 'command' ──→ Transformed Output           │
│                                                            │
│  Common commands:                                          │
│  s/old/new/     Substitute                                │
│  d              Delete line                                │
│  p              Print line                                 │
│  i\             Insert before                              │
│  a\             Append after                               │
└───────────────────────────────────────────────────────────┘
```

```bash
# ═══════════════════════════════════════════
# SUBSTITUTION (most common use)
# ═══════════════════════════════════════════

# Replace first occurrence per line
sed 's/old/new/' file.txt

# Replace ALL occurrences per line
sed 's/old/new/g' file.txt

# Replace case-insensitively
sed 's/error/WARNING/gi' file.txt

# Replace in-place (modify the file directly)
sed -i 's/old/new/g' file.txt

# Replace in-place with backup
sed -i.bak 's/old/new/g' file.txt

# Replace on specific line numbers
sed '5s/old/new/' file.txt            # Line 5 only
sed '10,20s/old/new/g' file.txt       # Lines 10-20

# ═══════════════════════════════════════════
# DELETE LINES
# ═══════════════════════════════════════════

# Delete specific line
sed '5d' file.txt                      # Delete line 5
sed '1d' file.txt                      # Delete first line (header)
sed '$d' file.txt                      # Delete last line

# Delete range
sed '10,20d' file.txt                  # Delete lines 10-20

# Delete matching lines
sed '/^#/d' file.txt                   # Delete comment lines
sed '/^$/d' file.txt                   # Delete empty lines
sed '/debug/d' logfile                 # Delete debug lines

# ═══════════════════════════════════════════
# PRINT / EXTRACT
# ═══════════════════════════════════════════

# Print specific lines (-n suppresses default output)
sed -n '5p' file.txt                   # Print line 5
sed -n '10,20p' file.txt              # Print lines 10-20
sed -n '/error/p' file.txt            # Print matching lines

# ═══════════════════════════════════════════
# INSERT / APPEND
# ═══════════════════════════════════════════

# Insert before line 3
sed '3i\New line here' file.txt

# Append after line 3
sed '3a\New line here' file.txt

# ═══════════════════════════════════════════
# DEVOPS EXAMPLES
# ═══════════════════════════════════════════

# Update config file value
sed -i 's/^port = .*/port = 8080/' config.ini

# Comment out a line
sed -i 's/^PasswordAuthentication yes/# PasswordAuthentication yes/' /etc/ssh/sshd_config

# Uncomment a line
sed -i 's/^#\s*PasswordAuthentication no/PasswordAuthentication no/' /etc/ssh/sshd_config

# Replace IP address in config
sed -i 's/192\.168\.1\.1/10.0.0.1/g' /etc/nginx/nginx.conf

# Remove trailing whitespace
sed -i 's/[[:space:]]*$//' script.sh

# Add text after a matching line
sed -i '/\[mysqld\]/a max_connections = 500' /etc/mysql/my.cnf

# Multiple operations
sed -e 's/foo/bar/g' -e '/^#/d' -e '/^$/d' file.txt
```

---

## 4.3 `awk` — Pattern Scanning and Processing

```
┌───────────────────────────────────────────────────────────┐
│                    awk WORKFLOW                            │
│                                                            │
│  Input ──→ awk 'pattern {action}' ──→ Processed Output   │
│                                                            │
│  Built-in variables:                                       │
│  $0  = Entire line          NR  = Line number             │
│  $1  = First field          NF  = Number of fields        │
│  $2  = Second field         FS  = Field Separator         │
│  $NF = Last field           OFS = Output Field Sep        │
└───────────────────────────────────────────────────────────┘
```

```bash
# ═══════════════════════════════════════════
# FIELD EXTRACTION
# ═══════════════════════════════════════════

# Print specific fields (space-separated by default)
awk '{print $1}' file.txt               # First field
awk '{print $1, $3}' file.txt           # First and third
awk '{print $NF}' file.txt              # Last field

# Custom field separator
awk -F: '{print $1, $7}' /etc/passwd    # Username and shell
awk -F, '{print $2}' data.csv           # Second CSV column

# ═══════════════════════════════════════════
# PATTERN MATCHING
# ═══════════════════════════════════════════

# Print lines matching a pattern
awk '/error/' logfile
awk '/error/ {print $0}' logfile         # Same thing, explicit

# Print field from matching lines
awk '/nginx/ {print $2, $11}' /var/log/nginx/access.log

# Conditional filtering
awk '$9 >= 500' /var/log/nginx/access.log        # 5xx errors
awk '$9 == 404 {print $7}' access.log            # URLs returning 404
awk 'length($0) > 100' file.txt                   # Lines > 100 chars

# ═══════════════════════════════════════════
# CALCULATIONS
# ═══════════════════════════════════════════

# Sum a column
awk '{sum += $5} END {print "Total:", sum}' data.txt

# Average
awk '{sum += $5; count++} END {print "Avg:", sum/count}' data.txt

# Min and Max
awk 'NR==1 || $5 > max {max=$5} END {print "Max:", max}' data.txt

# ═══════════════════════════════════════════
# FORMATTING
# ═══════════════════════════════════════════

# Printf for formatted output
awk '{printf "%-20s %10s\n", $1, $5}' data.txt

# Add header
awk 'BEGIN {print "User\tShell"} {print $1"\t"$7}' FS=: /etc/passwd

# ═══════════════════════════════════════════
# DEVOPS EXAMPLES
# ═══════════════════════════════════════════

# Parse nginx access log
# Log format: IP - - [date] "METHOD URL PROTO" STATUS SIZE
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
# Top 10 IPs by request count

# Request count per status code
awk '{count[$9]++} END {for (code in count) print code, count[code]}' access.log

# Average response time (assuming $NF is response time)
awk '{sum += $NF; count++} END {printf "Avg: %.2fms\n", sum/count}' access.log

# Disk usage - sort and format
df -h | awk 'NR>1 {printf "%-30s %s used\n", $6, $5}'

# Memory usage percentage
free | awk 'NR==2 {printf "Memory: %.1f%% used\n", $3/$2*100}'

# Find processes using most CPU
ps aux | awk 'NR>1 {printf "%-20s %s%%\n", $11, $3}' | sort -t'%' -k2 -rn | head -5

# Parse /etc/passwd for system info
awk -F: '$3 >= 1000 && $3 < 65534 {print $1, "(UID:"$3")"}' /etc/passwd
```

---

## 4.4 `cut` — Extract Columns/Fields

```bash
# Cut by field (delimiter-based)
cut -d: -f1 /etc/passwd              # Usernames
cut -d: -f1,7 /etc/passwd            # Username and shell
cut -d: -f1-3 /etc/passwd            # Fields 1 through 3
cut -d, -f2 data.csv                 # Second CSV column

# Cut by character position
cut -c1-10 file.txt                  # First 10 characters
cut -c1-8 /etc/passwd                # First 8 chars of each line

# Cut by bytes
cut -b1-5 file.txt

# Combine with other commands
cat /etc/passwd | cut -d: -f1 | sort
ps aux | awk '{print $1}' | cut -c1-8 | sort -u
```

---

## 4.5 `sort` and `uniq` — Ordering and Deduplication

```bash
# ═══════════════════════════════════════════
# sort
# ═══════════════════════════════════════════

# Alphabetical sort
sort names.txt

# Reverse sort
sort -r names.txt

# Numeric sort
sort -n numbers.txt

# Sort by specific field
sort -t: -k3 -n /etc/passwd          # Sort by UID (field 3, : delimiter)

# Sort human-readable sizes
du -sh /* 2>/dev/null | sort -rh

# Unique sort
sort -u file.txt                     # Remove duplicates while sorting

# Case-insensitive sort
sort -f file.txt

# ═══════════════════════════════════════════
# uniq (requires sorted input!)
# ═══════════════════════════════════════════

# Remove consecutive duplicate lines
sort file.txt | uniq

# Count occurrences
sort file.txt | uniq -c

# Show only duplicates
sort file.txt | uniq -d

# Show only unique lines (no duplicates)
sort file.txt | uniq -u

# ═══════════════════════════════════════════
# COMBINED DEVOPS PATTERNS
# ═══════════════════════════════════════════

# Top 10 requesting IPs from access log
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10

# Most requested URLs
awk '{print $7}' access.log | sort | uniq -c | sort -rn | head -10

# Unique users who logged in
last | awk '{print $1}' | sort -u

# Count unique error types
grep -i error /var/log/syslog | awk '{print $5}' | sort | uniq -c | sort -rn
```

---

## 4.6 Other Text Processing Tools

```bash
# tr — Translate/delete characters
echo "Hello World" | tr 'a-z' 'A-Z'           # HELLO WORLD
echo "Hello World" | tr -d ' '                 # HelloWorld
echo "Hello   World" | tr -s ' '               # Hello World (squeeze)
cat file.txt | tr '\t' ','                      # Tabs to commas

# paste — Merge lines side by side
paste file1.txt file2.txt                       # Tab-separated columns
paste -d, file1.txt file2.txt                   # Comma-separated

# diff — Compare files
diff file1.txt file2.txt
diff -u file1.txt file2.txt                     # Unified format (like git diff)
diff -r dir1/ dir2/                             # Compare directories

# wc — Word/line/character count
wc -l file.txt                                  # Line count
wc -w file.txt                                  # Word count
cat /etc/passwd | wc -l                         # Number of users

# tee — Write to file AND stdout
command | tee output.log                        # Write and display
command | tee -a output.log                     # Append mode
command 2>&1 | tee debug.log                    # Capture stdout+stderr

# xargs — Build and execute commands from stdin
find /tmp -name "*.log" | xargs rm -f
cat urls.txt | xargs -I {} curl -s {}
echo "file1 file2 file3" | xargs -n 1 cp -v --target-directory=/backup/
```

---

## 4.7 Real-World DevOps Scenario

### Scenario: Analyzing Web Server Access Logs

```bash
#!/bin/bash
# log-analysis.sh — Quick access log analysis

LOG="/var/log/nginx/access.log"

echo "═══════════════════════════════════════"
echo "  NGINX ACCESS LOG ANALYSIS"
echo "  $(date)"
echo "═══════════════════════════════════════"

echo -e "\n📊 Total requests:"
wc -l < "$LOG"

echo -e "\n📊 Requests by HTTP status:"
awk '{print $9}' "$LOG" | sort | uniq -c | sort -rn

echo -e "\n📊 Top 10 IPs:"
awk '{print $1}' "$LOG" | sort | uniq -c | sort -rn | head -10

echo -e "\n📊 Top 10 URLs:"
awk '{print $7}' "$LOG" | sort | uniq -c | sort -rn | head -10

echo -e "\n📊 Requests per hour (today):"
grep "$(date +%d/%b/%Y)" "$LOG" | \
    awk '{print $4}' | cut -d: -f2 | sort | uniq -c

echo -e "\n📊 5xx Errors (last 20):"
awk '$9 >= 500 {print $1, $4, $7, $9}' "$LOG" | tail -20

echo -e "\n📊 Top User Agents:"
awk -F'"' '{print $6}' "$LOG" | sort | uniq -c | sort -rn | head -5

echo -e "\n📊 Bandwidth usage (MB):"
awk '{sum += $10} END {printf "%.2f MB\n", sum/1024/1024}' "$LOG"
```

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|---------|
| `grep` no output | Pattern not found; case mismatch | Use `-i` for case-insensitive |
| `sed -i` error on macOS | GNU vs BSD sed | Use `sed -i '' 's/...'` on macOS |
| `awk` wrong field | Different delimiter | Set `-F` to correct delimiter |
| `sort \| uniq` not working | Input not sorted | Always `sort` before `uniq` |
| `cut -d` wrong output | Multi-char delimiter | Use `awk` with `-F` instead |
| Regex not matching | Need extended regex | Use `grep -E` or `awk` |

---

## Summary Table

| Command | Purpose | Key Flags |
|---------|---------|-----------|
| `grep` | Search for patterns | `-i`, `-r`, `-v`, `-c`, `-E`, `-A/B/C` |
| `sed` | Stream editing / substitution | `s///g`, `-i`, `-n`, `d`, `p` |
| `awk` | Field processing / calculations | `-F`, `$1-$NF`, `NR`, `NF` |
| `cut` | Extract columns/fields | `-d`, `-f`, `-c` |
| `sort` | Sort lines | `-n`, `-r`, `-k`, `-t`, `-h`, `-u` |
| `uniq` | Remove/count duplicates | `-c`, `-d`, `-u` |
| `tr` | Translate/delete characters | `-d`, `-s` |
| `tee` | Split output to file + stdout | `-a` (append) |
| `xargs` | Build commands from stdin | `-I {}`, `-n` |
| `diff` | Compare files | `-u`, `-r` |

---

## Quick Revision Questions

1. **What is the difference between `grep`, `grep -E`, and `grep -F`?** When would you use each?
2. **Write a `sed` command to replace all occurrences of an IP address in a config file in-place.** Include a backup.
3. **How does `awk` split input into fields?** How do you change the delimiter?
4. **Why must input be sorted before piping to `uniq`?** What's the alternative?
5. **Write a one-liner to find the top 10 IP addresses by request count from an nginx access log.**
6. **How would you use `sed` to uncomment a specific line in a config file?** Show the command.

---

[← Previous: Text Viewing](03-text-viewing.md) | [Back to README](../README.md) | [Next: Finding Files →](05-finding-files.md)
