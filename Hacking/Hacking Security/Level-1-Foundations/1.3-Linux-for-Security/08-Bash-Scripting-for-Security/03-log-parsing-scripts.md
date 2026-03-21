# Log Parsing Scripts

## Unit 8 - Topic 3 | Bash Scripting for Security

---

## Overview

**Log parsing** is one of the most practical security skills. Bash scripts can process massive log files to extract **failed logins**, **suspicious IPs**, **attack patterns**, and **anomalies** faster than manual review. These scripts are the foundation of lightweight security monitoring.

---

## 1. Authentication Log Parser

```bash
#!/bin/bash
# auth_log_parser.sh — Parse authentication logs for security events
set -euo pipefail

LOG_FILE="${1:-/var/log/auth.log}"
REPORT="auth_report_$(date +%Y%m%d).txt"

[[ ! -f "$LOG_FILE" ]] && { echo "Log file not found: $LOG_FILE"; exit 1; }

{
echo "==========================================="
echo "  AUTHENTICATION LOG SECURITY REPORT"
echo "  Date: $(date)"
echo "  Log: $LOG_FILE"
echo "==========================================="

# 1. Failed login attempts
echo ""
echo "--- FAILED LOGIN ATTEMPTS ---"
failed_count=$(grep -c "Failed password" "$LOG_FILE" 2>/dev/null || echo 0)
echo "Total failed attempts: $failed_count"

# 2. Top 10 attacking IPs
echo ""
echo "--- TOP 10 ATTACKING IPs ---"
echo "Count | IP Address"
grep "Failed password" "$LOG_FILE" 2>/dev/null | \
    grep -oP '\d+\.\d+\.\d+\.\d+' | \
    sort | uniq -c | sort -rn | head -10

# 3. Top targeted usernames
echo ""
echo "--- TOP 10 TARGETED USERNAMES ---"
grep "Failed password" "$LOG_FILE" 2>/dev/null | \
    grep -oP 'for (\S+)' | \
    sort | uniq -c | sort -rn | head -10

# 4. Successful logins
echo ""
echo "--- SUCCESSFUL LOGINS ---"
grep "Accepted" "$LOG_FILE" 2>/dev/null | \
    awk '{for(i=1;i<=NF;i++) if($i=="from") print $(i+1)}' | \
    sort | uniq -c | sort -rn

# 5. Sudo usage
echo ""
echo "--- SUDO COMMANDS EXECUTED ---"
grep "sudo:" "$LOG_FILE" 2>/dev/null | \
    grep "COMMAND" | tail -20

# 6. Account changes
echo ""
echo "--- ACCOUNT MODIFICATIONS ---"
grep -E "useradd|userdel|usermod|groupadd|passwd" "$LOG_FILE" 2>/dev/null | tail -10

echo ""
echo "==========================================="
echo "  Report generated: $REPORT"
echo "==========================================="
} | tee "$REPORT"
```

---

## 2. Web Access Log Analyzer

```bash
#!/bin/bash
# web_log_analyzer.sh — Analyze Apache/Nginx access logs
set -euo pipefail

LOG_FILE="${1:-/var/log/nginx/access.log}"

[[ ! -f "$LOG_FILE" ]] && { echo "Log file not found"; exit 1; }

echo "=== WEB ACCESS LOG ANALYSIS ==="
echo "File: $LOG_FILE"
echo "Lines: $(wc -l < "$LOG_FILE")"
echo ""

# 1. Top 10 IPs by requests
echo "--- TOP 10 IPs (by request count) ---"
awk '{print $1}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

# 2. Top 10 requested URLs
echo ""
echo "--- TOP 10 REQUESTED URLs ---"
awk '{print $7}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

# 3. HTTP status code distribution
echo ""
echo "--- STATUS CODE DISTRIBUTION ---"
awk '{print $9}' "$LOG_FILE" | sort | uniq -c | sort -rn

# 4. Potential SQL injection attempts
echo ""
echo "--- POTENTIAL SQL INJECTION ATTEMPTS ---"
grep -iE "(union|select|insert|drop|delete|update|--|;|'|\")" "$LOG_FILE" | \
    awk '{print $1, $7}' | head -20

# 5. Potential directory traversal
echo ""
echo "--- POTENTIAL DIRECTORY TRAVERSAL ---"
grep -E "\.\.\/" "$LOG_FILE" | awk '{print $1, $7}' | head -20

# 6. Potential scanner activity (many 404s from one IP)
echo ""
echo "--- IPs WITH MOST 404 ERRORS (possible scanners) ---"
awk '$9 == 404 {print $1}' "$LOG_FILE" | sort | uniq -c | sort -rn | head -10

# 7. Suspicious user agents
echo ""
echo "--- SUSPICIOUS USER AGENTS ---"
grep -iE "(nikto|sqlmap|dirbuster|gobuster|nmap|masscan|burp)" "$LOG_FILE" | \
    awk -F'"' '{print $6}' | sort -u | head -10
```

---

## 3. Firewall Log Parser

```bash
#!/bin/bash
# firewall_log_parser.sh — Parse iptables/ufw logs
set -euo pipefail

LOG_FILE="${1:-/var/log/kern.log}"

echo "=== FIREWALL LOG ANALYSIS ==="

# 1. Total blocked connections
echo ""
echo "--- BLOCKED CONNECTIONS ---"
blocked=$(grep -c "BLOCK\|DROP\|DENY" "$LOG_FILE" 2>/dev/null || echo 0)
echo "Total blocked: $blocked"

# 2. Top blocked source IPs
echo ""
echo "--- TOP 10 BLOCKED SOURCE IPs ---"
grep -E "BLOCK|DROP|DENY" "$LOG_FILE" 2>/dev/null | \
    grep -oP 'SRC=\K[\d.]+' | sort | uniq -c | sort -rn | head -10

# 3. Top blocked destination ports
echo ""
echo "--- TOP 10 BLOCKED DESTINATION PORTS ---"
grep -E "BLOCK|DROP|DENY" "$LOG_FILE" 2>/dev/null | \
    grep -oP 'DPT=\K\d+' | sort | uniq -c | sort -rn | head -10

# 4. Blocked protocols
echo ""
echo "--- BLOCKED BY PROTOCOL ---"
grep -E "BLOCK|DROP|DENY" "$LOG_FILE" 2>/dev/null | \
    grep -oP 'PROTO=\K\w+' | sort | uniq -c | sort -rn
```

---

## 4. Real-Time Log Monitor

```bash
#!/bin/bash
# realtime_monitor.sh — Real-time security event monitoring
set -uo pipefail

THRESHOLD=5                           # Failed attempts threshold
ALERT_LOG="/var/log/security_alerts.log"

log_alert() {
    echo "[ALERT $(date '+%H:%M:%S')] $1" | tee -a "$ALERT_LOG"
}

echo "=== REAL-TIME SECURITY MONITOR ==="
echo "Monitoring /var/log/auth.log..."
echo "Alert threshold: $THRESHOLD failed attempts"
echo "Press Ctrl+C to stop"
echo ""

# Monitor auth log in real-time
tail -f /var/log/auth.log 2>/dev/null | while read -r line; do
    # Detect brute force
    if echo "$line" | grep -q "Failed password"; then
        ip=$(echo "$line" | grep -oP '\d+\.\d+\.\d+\.\d+')
        user=$(echo "$line" | grep -oP 'for \K\S+')
        count=$(grep "Failed password.*$ip" /var/log/auth.log | wc -l)
        
        if [ "$count" -ge "$THRESHOLD" ]; then
            log_alert "BRUTE FORCE: $ip ($count attempts, user: $user)"
        fi
    fi
    
    # Detect successful root login
    if echo "$line" | grep -q "Accepted.*root"; then
        log_alert "ROOT LOGIN detected!"
    fi
    
    # Detect new user creation
    if echo "$line" | grep -q "useradd"; then
        log_alert "NEW USER created: $line"
    fi
done
```

---

## 5. One-Liner Log Analysis

```bash
# Quick security checks you can run anytime

# Failed SSH logins today
grep "$(date +%b' '%d)" /var/log/auth.log | grep "Failed" | wc -l

# Unique IPs that failed login
grep "Failed" /var/log/auth.log | grep -oP '\d+\.\d+\.\d+\.\d+' | sort -u

# Who is logged in right now
who | awk '{print $1, $5}'

# Last 10 logins
last -10

# Processes listening on network
ss -tlnp | awk 'NR>1 {print $4, $6}'

# Files modified in last hour
find /etc -mmin -60 -type f 2>/dev/null

# Large files created recently (possible data staging)
find /tmp /var/tmp -size +10M -mtime -1 2>/dev/null
```

---

## Summary Table

| Script | Purpose |
|--------|---------|
| **Auth log parser** | Failed logins, attacking IPs, sudo usage |
| **Web log analyzer** | SQL injection, scanners, status codes |
| **Firewall parser** | Blocked IPs, ports, protocols |
| **Real-time monitor** | Live brute force detection |
| **One-liners** | Quick daily security checks |

---

## Quick Revision Questions

1. **How do you extract all IP addresses from auth.log?**
2. **What grep pattern detects SQL injection in web logs?**
3. **How do you monitor a log file in real-time?**
4. **What does `uniq -c | sort -rn` do?**
5. **How would you detect a scanner in web access logs?**

---

[← Previous: Security Automation Scripts](02-security-automation-scripts.md) | [Next: Network Monitoring Scripts →](04-network-monitoring-scripts.md)

---

*Unit 8 - Topic 3 of 5 | [Back to README](../README.md)*
