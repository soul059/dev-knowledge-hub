# Chapter 3: Text Viewing

[← Previous: File Operations](02-file-operations.md) | [Back to README](../README.md) | [Next: Text Processing →](04-text-processing.md)

---

## Overview

DevOps engineers spend a huge amount of time reading files — logs, configurations, scripts, and output. Linux provides specialized commands for viewing text files in different ways: all at once, page by page, or just the beginning and end. Choosing the right tool saves time and helps you find information faster.

---

## 3.1 `cat` — Concatenate and Display

```bash
# Display entire file
cat /etc/hostname

# Display with line numbers
cat -n /etc/ssh/sshd_config

# Display with line numbers (skip blank lines)
cat -b /etc/ssh/sshd_config

# Show non-printing characters (tabs, line endings)
cat -A file.txt
# Shows: ^I (tab), $ (end of line)

# Show tabs as ^I
cat -T file.txt

# Concatenate multiple files
cat header.txt body.txt footer.txt > full-page.html

# Create file with cat (heredoc)
cat <<EOF > /etc/nginx/conf.d/myapp.conf
server {
    listen 80;
    server_name myapp.example.com;
    root /var/www/myapp;
}
EOF

# Number only non-empty lines
cat -b script.sh

# Suppress repeated empty lines
cat -s file-with-many-blanks.txt
```

```
┌────────────────────────────────────────────────────────────┐
│  cat BEST PRACTICES:                                       │
│                                                             │
│  ✓ Use for short files (< 50 lines)                       │
│  ✓ Use for piping content: cat file | grep pattern         │
│  ✓ Use for concatenation: cat part1 part2 > whole          │
│                                                             │
│  ✗ Don't use for large files (use less instead)            │
│  ✗ Avoid "useless use of cat": cat file | grep → grep file │
└────────────────────────────────────────────────────────────┘
```

---

## 3.2 `less` — The Best File Viewer

`less` is the most versatile file viewer. It loads files lazily (fast even for huge files), supports searching, and doesn't clutter your terminal.

```bash
# View a file
less /var/log/syslog

# View with line numbers
less -N /etc/ssh/sshd_config

# View and follow new content (like tail -f)
less +F /var/log/syslog
# Press Ctrl+C to stop following, then 'F' to resume

# Start at the end of file
less +G /var/log/syslog

# View multiple files
less file1.txt file2.txt
# :n  → next file
# :p  → previous file
```

### `less` Navigation

```
┌────────────────────────────────────────────────────────────┐
│              LESS NAVIGATION CHEAT SHEET                    │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  MOVEMENT:                                                  │
│  Space / Page Down   → Forward one screen                  │
│  b / Page Up         → Backward one screen                 │
│  j / ↓ / Enter       → Forward one line                    │
│  k / ↑               → Backward one line                   │
│  d                   → Forward half screen                  │
│  u                   → Backward half screen                 │
│  g                   → Go to beginning                      │
│  G                   → Go to end                            │
│  50g                 → Go to line 50                        │
│                                                             │
│  SEARCHING:                                                 │
│  /pattern            → Search forward                       │
│  ?pattern            → Search backward                      │
│  n                   → Next match                           │
│  N                   → Previous match                       │
│  /error|warn         → Search with regex (OR)               │
│  &pattern            → Show only matching lines             │
│                                                             │
│  OTHER:                                                     │
│  q                   → Quit                                 │
│  h                   → Help                                 │
│  v                   → Open in editor ($VISUAL or vi)       │
│  F                   → Follow mode (like tail -f)           │
│  -N                  → Toggle line numbers                  │
│  -S                  → Toggle line wrapping                 │
│  =                   → Show file info (line numbers, %)     │
│  !command            → Run shell command                    │
└────────────────────────────────────────────────────────────┘
```

---

## 3.3 `more` — Simple Pager

`more` is an older, simpler pager. It only moves forward through a file. Use `less` when available.

```bash
# View a file
more /var/log/syslog

# Start at line 100
more +100 /var/log/syslog

# Show n lines at a time
more -n 20 /var/log/syslog
```

| Feature | `less` | `more` |
|---------|--------|--------|
| Forward scrolling | ✓ | ✓ |
| Backward scrolling | ✓ | ✗ |
| Search | ✓ (forward & backward) | ✓ (forward only) |
| Large file handling | Excellent | Loads entirely |
| Follow mode | ✓ (`+F`) | ✗ |
| Line numbers | ✓ (`-N`) | ✗ |

---

## 3.4 `head` — View the Beginning

```bash
# Show first 10 lines (default)
head /var/log/syslog

# Show first N lines
head -n 20 /var/log/syslog
head -20 /var/log/syslog             # Short form

# Show first N bytes
head -c 100 /var/log/syslog

# Show everything EXCEPT the last N lines
head -n -5 /var/log/syslog

# View headers of multiple files
head /etc/hostname /etc/resolv.conf /etc/hosts

# DevOps usage: Check CSV headers
head -1 data.csv

# Check the shebang of a script
head -1 deploy.sh
# #!/bin/bash
```

---

## 3.5 `tail` — View the End (Critical for DevOps)

```bash
# Show last 10 lines (default)
tail /var/log/syslog

# Show last N lines
tail -n 50 /var/log/syslog
tail -50 /var/log/syslog             # Short form

# ═══════════════════════════════════════════
# tail -f — FOLLOW MODE (Most used for DevOps!)
# ═══════════════════════════════════════════

# Follow a log file in real-time
tail -f /var/log/syslog

# Follow with line count
tail -f -n 50 /var/log/nginx/access.log

# Follow multiple files
tail -f /var/log/nginx/access.log /var/log/nginx/error.log

# Follow and show filename headers
tail -f --verbose /var/log/nginx/*.log

# Follow with grep (filter in real-time)
tail -f /var/log/syslog | grep -i error
tail -f /var/log/nginx/access.log | grep "500\|502\|503"

# Follow even if file is rotated (recreated)
tail -F /var/log/syslog
# -F = --follow=name --retry
# Handles log rotation gracefully

# Show everything after line N
tail -n +100 /var/log/syslog         # From line 100 onwards

# Show last N bytes
tail -c 1024 /var/log/syslog
```

```
┌────────────────────────────────────────────────────────────┐
│  tail -f vs tail -F                                        │
│                                                             │
│  tail -f  → Follows the file descriptor                    │
│              If file is rotated (deleted + recreated),      │
│              tail still reads the old (deleted) file        │
│                                                             │
│  tail -F  → Follows the file NAME                          │
│              If file is rotated, tail detects the new       │
│              file and switches to it automatically          │
│                                                             │
│  Rule: Use tail -F for log files that get rotated          │
└────────────────────────────────────────────────────────────┘
```

---

## 3.6 Combining Commands for Log Analysis

```bash
# Show lines 50-70 of a file
head -70 file.txt | tail -20

# Alternative: sed
sed -n '50,70p' file.txt

# Real-time error monitoring with highlighting
tail -f /var/log/syslog | grep --color=auto -E "ERROR|WARN|CRITICAL"

# Monitor multiple log sources
tail -f /var/log/nginx/access.log | awk '{print $1, $7, $9}'
# Shows: IP, URL, Status Code

# Count errors in last 100 lines
tail -100 /var/log/syslog | grep -c "error"

# Get unique IPs from access log (most recent 1000 lines)
tail -1000 /var/log/nginx/access.log | awk '{print $1}' | sort -u | wc -l

# Watch for specific events
tail -f /var/log/auth.log | grep "Failed password"    # Brute force detection
tail -f /var/log/syslog | grep "OOM"                   # Out of memory events
```

---

## 3.7 Other Useful Viewing Commands

```bash
# strings — Extract printable strings from binary files
strings /usr/bin/ls | head -20
strings core-dump | grep "Error"

# xxd — Hex viewer
xxd file.bin | head -20

# tac — Reverse cat (last line first)
tac /var/log/syslog | head -20       # Latest entries first

# rev — Reverse characters in each line
echo "Hello" | rev                    # olleH

# nl — Number lines (more control than cat -n)
nl -ba -s '. ' config.yaml           # Number all lines with '. ' separator

# column — Format output in columns
cat /etc/passwd | column -t -s ':'
mount | column -t
```

---

## 3.8 Real-World DevOps Scenario

### Scenario: Monitoring a Production Deployment in Real-Time

```bash
# Terminal 1: Watch application logs
tail -F /var/log/myapp/application.log | grep --color -E "ERROR|WARN|INFO|^"

# Terminal 2: Watch web server access logs
tail -F /var/log/nginx/access.log | awk '{
    status=$9;
    if (status >= 500) print "\033[31m" $0 "\033[0m";   # Red for 5xx
    else if (status >= 400) print "\033[33m" $0 "\033[0m"; # Yellow for 4xx
    else print $0;
}'

# Terminal 3: Watch system resources
watch -n 1 'echo "=== Load ===" && cat /proc/loadavg && echo "=== Memory ===" && free -h | head -2 && echo "=== Disk ===" && df -h / | tail -1'

# Terminal 4: Watch for errors in system log
tail -F /var/log/syslog | grep -i "error\|fail\|critical\|oom"

# Quick health check after deployment
echo "=== Last 5 application log entries ==="
tail -5 /var/log/myapp/application.log

echo "=== Recent 5xx errors ==="
tail -1000 /var/log/nginx/access.log | awk '$9 >= 500' | tail -5

echo "=== Disk usage ==="
df -h / /var

echo "=== Memory ==="
free -h
```

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|---------|
| `tail -f` stops showing output | Log file was rotated | Use `tail -F` instead |
| `less` shows binary garbage | Binary file opened | Use `strings` or `xxd` |
| `cat` floods terminal with output | File is too large | Use `less`, `head`, or `tail` |
| Can't read file | Permission denied | Use `sudo` or check file permissions |
| `head`/`tail` wrong encoding | Non-UTF8 file | Use `file` to check encoding, `iconv` to convert |
| `less` search not finding pattern | Case sensitivity | Use `/pattern` with `-i` flag: `less -i file` |

---

## Summary Table

| Command | Purpose | Best Used For |
|---------|---------|---------------|
| `cat` | Display entire file | Short files, concatenation, piping |
| `less` | Interactive file viewer | Large files, searching, navigation |
| `more` | Simple forward-only pager | When `less` unavailable |
| `head` | View beginning of file | Check headers, first N lines |
| `tail` | View end of file | Last N lines |
| `tail -f` | Follow file in real-time | Live log monitoring |
| `tail -F` | Follow file (handles rotation) | Production log monitoring |
| `tac` | Reverse file display | View newest entries first |
| `strings` | Extract text from binaries | Binary file inspection |
| `nl` | Number lines | Formatted line numbering |

---

## Quick Revision Questions

1. **What is the difference between `tail -f` and `tail -F`?** When should you use each?
2. **How would you view lines 50-75 of a file?** Show two different methods.
3. **Why is `less` preferred over `cat` for large files?** Name 3 features that make `less` superior.
4. **Write a command to monitor a log file and only show lines containing "ERROR" or "WARN" in real-time.**
5. **What are the key navigation shortcuts in `less`?** How do you search for a pattern?
6. **How would you monitor multiple log files simultaneously?** Describe your approach.

---

[← Previous: File Operations](02-file-operations.md) | [Back to README](../README.md) | [Next: Text Processing →](04-text-processing.md)
