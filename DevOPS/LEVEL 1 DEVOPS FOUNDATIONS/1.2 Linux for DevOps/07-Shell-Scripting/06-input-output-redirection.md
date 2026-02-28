# Chapter 6: Input/Output Redirection

[← Previous: Functions](05-functions.md) | [Back to README](../README.md) | [Next: Pipes & Filters →](07-pipes-and-filters.md)

---

## Overview

I/O redirection is one of the most powerful features of the Linux shell, allowing you to control where commands read input and write output. Understanding file descriptors, redirection operators, and here documents is essential for DevOps scripting — from log management to automated configuration.

---

## 6.1 File Descriptors

```
┌──────────────────────────────────────────────────────────────┐
│            STANDARD FILE DESCRIPTORS                          │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────┐     FD 0 (stdin)     ┌───────────────────┐     │
│  │ Keyboard ├────────────────────►  │                   │     │
│  └──────────┘                       │                   │     │
│                                     │    PROCESS        │     │
│                    FD 1 (stdout)    │    (command)      │     │
│  ┌──────────┐  ◄────────────────── │                   │     │
│  │ Terminal  │                      │                   │     │
│  │ (screen)  │  FD 2 (stderr)      │                   │     │
│  │          │  ◄────────────────── │                   │     │
│  └──────────┘                       └───────────────────┘     │
│                                                               │
│  FD 0 = Standard Input  (stdin)  — where command reads       │
│  FD 1 = Standard Output (stdout) — normal output             │
│  FD 2 = Standard Error  (stderr) — error messages            │
│  FD 3+ = User-defined file descriptors                       │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.2 Output Redirection

```bash
# Redirect stdout to file (overwrite)
echo "Hello" > output.txt
ls /var/log > file_list.txt

# Redirect stdout to file (append)
echo "Line 1" >> log.txt
echo "Line 2" >> log.txt

# Redirect stderr to file
ls /nonexistent 2> errors.txt

# Redirect stderr (append)
command 2>> error.log

# Redirect both stdout and stderr to same file
command > all_output.txt 2>&1       # Traditional
command &> all_output.txt          # Bash shorthand
command &>> all_output.txt         # Append both

# Redirect stdout and stderr to DIFFERENT files
command > stdout.log 2> stderr.log

# Suppress all output
command > /dev/null 2>&1
command &> /dev/null                 # Shorthand
```

```
┌──────────────────────────────────────────────────────────────┐
│         REDIRECTION OPERATORS REFERENCE                       │
├──────────────────┬───────────────────────────────────────────┤
│ Operator         │ Description                               │
├──────────────────┼───────────────────────────────────────────┤
│ >  file          │ Redirect stdout, overwrite file           │
│ >> file          │ Redirect stdout, append to file           │
│ 2> file          │ Redirect stderr, overwrite                │
│ 2>> file         │ Redirect stderr, append                   │
│ &> file          │ Redirect both stdout+stderr, overwrite    │
│ &>> file         │ Redirect both stdout+stderr, append       │
│ > file 2>&1      │ Redirect stderr to same place as stdout   │
│ 2>&1             │ Duplicate stderr to stdout                │
│ 1>&2             │ Duplicate stdout to stderr                │
│ > /dev/null      │ Discard stdout                            │
│ 2> /dev/null     │ Discard stderr                            │
│ &> /dev/null     │ Discard everything                        │
│ < file           │ Redirect stdin from file                  │
│ << TAG           │ Here document (inline input)              │
│ <<< "string"     │ Here string                               │
└──────────────────┴───────────────────────────────────────────┘
```

---

## 6.3 Input Redirection

```bash
# Read from file instead of keyboard
wc -l < /etc/hosts
sort < unsorted.txt > sorted.txt

# Send mail from file
mail -s "Report" admin@example.com < report.txt

# Multiple redirections
sort < input.txt > sorted.txt 2> errors.txt
```

---

## 6.4 Here Documents (Heredoc)

```bash
# Basic heredoc
cat << EOF
Hello $USER,
Today is $(date).
Your home is $HOME.
EOF

# Heredoc to a file
cat > /etc/nginx/conf.d/app.conf << 'EOF'
server {
    listen 80;
    server_name app.example.com;
    
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
    }
}
EOF

# Note: Quoting 'EOF' prevents variable expansion
# Without quotes: variables are expanded
# With quotes: everything is literal
```

```bash
# Heredoc with indentation (<<-)
# Tab characters are stripped from the beginning of each line
create_service() {
    cat <<-EOF > /etc/systemd/system/myapp.service
	[Unit]
	Description=My Application
	After=network.target
	
	[Service]
	Type=simple
	User=app
	ExecStart=/opt/myapp/bin/server
	Restart=always
	
	[Install]
	WantedBy=multi-user.target
	EOF
}

# Heredoc piped to a command
cat << EOF | sudo tee /etc/hosts.d/custom.conf
192.168.1.10  web01
192.168.1.11  web02
192.168.1.20  db01
EOF

# Heredoc to SSH
ssh user@server << 'REMOTE'
cd /opt/app
git pull
docker-compose up -d
echo "Deployed at $(date)"
REMOTE
```

---

## 6.5 Here Strings

```bash
# Feed a string as stdin
wc -w <<< "Hello World Bash"
# 3

# Read into variables
read -r first last <<< "John Doe"
echo "First: $first, Last: $last"

# Process a variable as input
CSV_LINE="web01,192.168.1.10,nginx"
IFS=',' read -r name ip service <<< "$CSV_LINE"
echo "$name runs $service on $ip"

# Compare with echo pipe
echo "Hello" | wc -w      # Creates subshell
wc -w <<< "Hello"          # No subshell (more efficient)
```

---

## 6.6 Advanced File Descriptors

```bash
# Open custom file descriptor for writing
exec 3> output.log               # FD 3 → output.log
echo "Line 1" >&3
echo "Line 2" >&3
exec 3>&-                         # Close FD 3

# Open custom file descriptor for reading
exec 4< input.txt                 # FD 4 ← input.txt
read -r line <&4
echo "First line: $line"
exec 4<&-                         # Close FD 4

# Read and write
exec 5<> data.txt                 # FD 5 ↔ data.txt
read -r line <&5
echo "Modified" >&5
exec 5>&-
```

```bash
# Practical: Separate log channels
exec 3>/var/log/app-info.log
exec 4>/var/log/app-error.log

log_info()  { echo "$(date '+%T') [INFO]  $*" >&3; }
log_error() { echo "$(date '+%T') [ERROR] $*" >&4; }

log_info "Application started"
log_error "Database connection timeout"

exec 3>&-
exec 4>&-
```

---

## 6.7 The `tee` Command

```bash
# Write to stdout AND a file simultaneously
ls -la | tee listing.txt

# Append instead of overwrite
echo "new line" | tee -a log.txt

# Write to multiple files
echo "Hello" | tee file1.txt file2.txt file3.txt

# Common pattern: write as root
echo "setting=value" | sudo tee /etc/config.conf > /dev/null

# Log and display simultaneously
./deploy.sh 2>&1 | tee deploy-$(date '+%Y%m%d').log
```

```
┌──────────────────────────────────────────────────────────────┐
│              TEE FLOW                                         │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│                       ┌─── stdout ──► Terminal                │
│  Command ──► tee ─────┤                                       │
│                       └─── file ───► output.txt               │
│                                                               │
│  # Like a T-junction in plumbing                              │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.8 Process Substitution

```bash
# <(command) — Treat command output as a file
diff <(ls /dir1) <(ls /dir2)

# Compare sorted output
diff <(sort file1.txt) <(sort file2.txt)

# Avoid subshell problem with while loop
count=0
while IFS= read -r line; do
    ((count++))
done < <(grep "error" /var/log/syslog)
echo "Found $count errors"
# Variable persists because no pipe subshell!

# Multiple process substitutions
paste <(cut -d: -f1 /etc/passwd) <(cut -d: -f3 /etc/passwd)

# Compare running config vs expected
diff <(nginx -T 2>/dev/null) expected-nginx.conf
```

---

## 6.9 Real-World Scenario: Log-Managed Deploy Script

```bash
#!/usr/bin/env bash
# deploy-with-logging.sh — Demonstrates advanced I/O redirection

set -euo pipefail

APP="${1:?Usage: $0 <app_name>}"
LOG_DIR="/var/log/deploys"
mkdir -p "$LOG_DIR"

# Create timestamped log
TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
LOG_FILE="${LOG_DIR}/${APP}_${TIMESTAMP}.log"

# Redirect all output to both terminal and log file
exec > >(tee -a "$LOG_FILE") 2>&1

echo "=== Deployment Log ==="
echo "App:       $APP"
echo "Time:      $(date)"
echo "User:      $(whoami)"
echo "Log File:  $LOG_FILE"
echo "========================"

# Generate config using heredoc
cat > "/tmp/${APP}.env" << EOF
APP_NAME=$APP
DEPLOY_TIME=$TIMESTAMP
NODE_ENV=production
LOG_LEVEL=info
EOF

echo "Config written to /tmp/${APP}.env"

# Simulate deployment steps
for step in "Pull" "Build" "Test" "Deploy" "Verify"; do
    echo "[$(date '+%T')] ${step}..."
    sleep 1
done

echo ""
echo "=== Deployment Complete ==="
echo "Full log saved to: $LOG_FILE"
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| `> file` empties the file you're reading | Same file for input and output | Use `sponge` from moreutils or temp file |
| `sudo echo > /etc/file` fails | Redirect happens before sudo | Use `echo "text" \| sudo tee /etc/file` |
| Heredoc variables not expanding | Delimiter is quoted (`'EOF'`) | Remove quotes from delimiter |
| Heredoc variables expanding unexpectedly | Delimiter not quoted | Quote it: `<< 'EOF'` |
| Error output mixed with stdout | Not separating streams | Use `2>` or `2>/dev/null` |
| `Permission denied` on redirect | No write access | Check permissions or use `sudo tee` |

---

## Summary Table

| Technique | Syntax | Example |
|-----------|--------|---------|
| Stdout to file | `> file` | `ls > list.txt` |
| Append | `>> file` | `echo "log" >> app.log` |
| Stderr to file | `2> file` | `cmd 2> err.log` |
| Both to file | `&> file` | `cmd &> all.log` |
| Input from file | `< file` | `sort < data.txt` |
| Heredoc | `<< EOF ... EOF` | Config file generation |
| Here string | `<<< "string"` | `wc <<< "hello"` |
| Tee | `\| tee file` | `cmd \| tee log.txt` |
| Process sub | `<(cmd)` | `diff <(cmd1) <(cmd2)` |
| Discard | `> /dev/null` | `cmd > /dev/null 2>&1` |

---

## Quick Revision Questions

1. **What is the difference between `>` and `>>`?**
2. **How do you redirect stderr to one file and stdout to another?**
3. **Why does `sudo echo "text" > /etc/file` fail and how do you fix it?**
4. **What is the difference between `<< EOF` and `<< 'EOF'` in a heredoc?**
5. **How does process substitution `<(command)` differ from a pipe?**
6. **Write a command that logs output to a file AND displays it on screen.**

---

[← Previous: Functions](05-functions.md) | [Back to README](../README.md) | [Next: Pipes & Filters →](07-pipes-and-filters.md)
