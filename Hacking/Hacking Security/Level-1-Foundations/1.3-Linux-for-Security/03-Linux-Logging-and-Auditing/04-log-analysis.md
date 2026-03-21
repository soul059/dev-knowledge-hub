# Log Analysis

## Unit 3 - Topic 4 | Linux Logging and Auditing

---

## Overview

**Log analysis** is the process of examining log data to detect security incidents, troubleshoot issues, and understand system behavior. Effective log analysis combines **command-line tools**, **pattern recognition**, and **automated alerting** to identify threats in real-time and during forensic investigations.

---

## 1. Essential CLI Tools for Log Analysis

```bash
# === VIEWING LOGS ===
cat /var/log/auth.log                 # Full file
less /var/log/auth.log                # Scrollable view
head -50 /var/log/auth.log            # First 50 lines
tail -50 /var/log/auth.log            # Last 50 lines
tail -f /var/log/auth.log             # Follow in real-time

# === FILTERING (grep) ===
grep "Failed password" /var/log/auth.log
grep -i "error" /var/log/syslog       # Case insensitive
grep -v "CRON" /var/log/syslog        # Exclude CRON entries
grep -c "Failed" /var/log/auth.log    # Count matches
grep -E "Failed|Invalid" /var/log/auth.log  # Multiple patterns

# === SORTING AND COUNTING ===
# Top 10 IPs with failed logins
grep "Failed password" /var/log/auth.log | \
    awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | head -10

# Failed login count by user
grep "Failed password" /var/log/auth.log | \
    awk '{print $(NF-5)}' | sort | uniq -c | sort -rn

# Logins per hour
grep "Accepted" /var/log/auth.log | \
    awk '{print $1, $2, substr($3,1,2)":00"}' | sort | uniq -c

# === AWK for structured parsing ===
# Extract specific fields
awk '/Failed password/ {print $1, $2, $3, $9, $11}' /var/log/auth.log

# === SED for transformation ===
# Remove timestamps for cleaner output
sed 's/^.*sshd/sshd/' /var/log/auth.log | grep "Failed"
```

---

## 2. Security Event Detection

```bash
# === BRUTE FORCE DETECTION ===
# More than 10 failed logins from same IP
grep "Failed password" /var/log/auth.log | \
    awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | \
    awk '$1 > 10 {print "ALERT: " $1 " failures from " $2}'

# === PRIVILEGE ESCALATION ===
# All sudo commands
grep "COMMAND=" /var/log/auth.log

# Failed sudo attempts
grep "NOT in sudoers" /var/log/auth.log

# su command usage
grep "su:" /var/log/auth.log

# === UNUSUAL ACTIVITY ===
# Logins at unusual hours (between midnight and 5 AM)
grep "Accepted" /var/log/auth.log | \
    awk -F: '{if ($1 ~ /0[0-4]/) print $0}'

# New user accounts created
grep "useradd" /var/log/auth.log

# Root login attempts
grep "root" /var/log/auth.log | grep "Accepted"

# === SERVICE ANOMALIES ===
# Service starts/stops
grep "systemd" /var/log/syslog | grep -E "Started|Stopped"

# Kernel errors
grep -i "error\|panic\|oops" /var/log/kern.log
```

---

## 3. One-Liner Security Checks

| Check | Command |
|-------|---------|
| Top failed IPs | `grep "Failed" /var/log/auth.log \| awk '{print $(NF-3)}' \| sort \| uniq -c \| sort -rn \| head` |
| Successful SSH | `grep "Accepted" /var/log/auth.log \| tail -20` |
| Sudo abuse | `grep "COMMAND=" /var/log/auth.log \| tail -20` |
| New users | `grep "useradd" /var/log/auth.log` |
| Boot errors | `journalctl -b -p err` |
| Cron activity | `grep CRON /var/log/syslog \| tail -20` |

---

## 4. Automated Log Monitoring

```bash
# === Simple monitoring script ===
#!/bin/bash
# Monitor auth.log for brute force (run via cron every 5 min)
THRESHOLD=10
LOG="/var/log/auth.log"
ALERT="/tmp/brute_force_alert"

grep "Failed password" "$LOG" | \
    awk '{print $(NF-3)}' | sort | uniq -c | sort -rn | \
    while read count ip; do
        if [ "$count" -gt "$THRESHOLD" ]; then
            echo "$(date): $count failed attempts from $ip" >> "$ALERT"
            # Optional: auto-ban with iptables
            # iptables -A INPUT -s "$ip" -j DROP
        fi
    done

# === logwatch — Automated log analysis ===
apt install logwatch
logwatch --detail High --range today --output stdout

# === logcheck — Alert on anomalies ===
apt install logcheck
# Automatically emails unusual log entries
```

---

## 5. Log Analysis Tools

| Tool | Purpose |
|------|---------|
| **grep/awk/sed** | CLI text processing |
| **logwatch** | Daily log summary reports |
| **logcheck** | Anomaly-based log alerting |
| **GoAccess** | Real-time web log analyzer |
| **Elastic Stack (ELK)** | Centralized log analysis platform |
| **Splunk** | Enterprise log analysis |
| **Graylog** | Open-source log management |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **grep** | Filter logs by pattern |
| **awk** | Parse and extract specific fields |
| **uniq -c \| sort -rn** | Count and rank occurrences |
| **tail -f** | Real-time log monitoring |
| **logwatch** | Automated daily reports |
| **Automation** | Scripts + cron for continuous monitoring |

---

## Quick Revision Questions

1. **How do you find the top 10 IPs with failed SSH logins?**
2. **What command follows a log file in real-time?**
3. **How would you detect unusual logins at 2 AM?**
4. **What is logwatch and what does it do?**
5. **Write a grep command to find all sudo commands in auth.log.**

---

[← Previous: Auditd Configuration](03-auditd-configuration.md) | [Next: Log Forwarding →](05-log-forwarding.md)

---

*Unit 3 - Topic 4 of 6 | [Back to README](../README.md)*
