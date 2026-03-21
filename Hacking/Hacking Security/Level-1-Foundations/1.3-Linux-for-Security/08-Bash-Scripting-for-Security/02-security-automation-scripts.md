# Security Automation Scripts

## Unit 8 - Topic 2 | Bash Scripting for Security

---

## Overview

**Security automation** replaces repetitive manual tasks with scripts — from **system hardening** and **vulnerability scanning** to **compliance checking** and **reporting**. Well-written automation scripts reduce human error and ensure consistent security baselines.

---

## 1. System Hardening Script

```bash
#!/bin/bash
# system_hardening.sh — Automated Linux hardening
set -euo pipefail

LOG="/var/log/hardening.log"
log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG"; }

# Check root
[[ "$(id -u)" -ne 0 ]] && { echo "Run as root!"; exit 1; }

log "=== SYSTEM HARDENING STARTED ==="

# 1. Update system
log "Updating packages..."
apt update -qq && apt upgrade -y -qq

# 2. Disable root SSH login
log "Hardening SSH..."
sed -i 's/^#*PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sed -i 's/^#*MaxAuthTries.*/MaxAuthTries 3/' /etc/ssh/sshd_config
systemctl restart sshd

# 3. Configure firewall
log "Configuring firewall..."
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp comment "SSH"
ufw --force enable

# 4. Set password policy
log "Setting password policies..."
sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS 90/' /etc/login.defs
sed -i 's/^PASS_MIN_DAYS.*/PASS_MIN_DAYS 7/' /etc/login.defs
sed -i 's/^PASS_MIN_LEN.*/PASS_MIN_LEN 12/' /etc/login.defs

# 5. Disable unnecessary services
log "Disabling unnecessary services..."
for svc in cups bluetooth avahi-daemon; do
    systemctl is-active "$svc" &>/dev/null && {
        systemctl stop "$svc"
        systemctl disable "$svc"
        log "Disabled: $svc"
    }
done

# 6. Secure shared memory
log "Securing shared memory..."
grep -q '/run/shm' /etc/fstab || \
    echo 'tmpfs /run/shm tmpfs defaults,noexec,nosuid 0 0' >> /etc/fstab

# 7. Set file permissions
log "Setting secure permissions..."
chmod 700 /root
chmod 600 /etc/shadow
chmod 644 /etc/passwd

log "=== HARDENING COMPLETE ==="
```

---

## 2. User Audit Script

```bash
#!/bin/bash
# user_audit.sh — Audit user accounts for security issues
set -euo pipefail

echo "=== USER SECURITY AUDIT ==="
echo "Date: $(date)"
echo ""

# 1. Users with UID 0 (root-level)
echo "--- Users with UID 0 (should only be root) ---"
awk -F: '$3 == 0 {print $1}' /etc/passwd

# 2. Users with empty passwords
echo ""
echo "--- Users with empty passwords ---"
awk -F: '($2 == "" || $2 == "!") {print $1}' /etc/shadow 2>/dev/null || echo "Cannot read shadow"

# 3. Users with login shells
echo ""
echo "--- Users with login shells ---"
grep -v '/nologin\|/false\|/sync\|/halt\|/shutdown' /etc/passwd | \
    awk -F: '{print $1 " → " $7}'

# 4. Recently created users (last 30 days)
echo ""
echo "--- Users created in last 30 days ---"
find /home -maxdepth 1 -type d -mtime -30 -exec basename {} \;

# 5. Sudo users
echo ""
echo "--- Users in sudo/wheel group ---"
getent group sudo 2>/dev/null || getent group wheel 2>/dev/null

# 6. Failed login attempts
echo ""
echo "--- Top 10 failed login sources ---"
grep "Failed password" /var/log/auth.log 2>/dev/null | \
    grep -oP '\d+\.\d+\.\d+\.\d+' | sort | uniq -c | sort -rn | head -10

echo ""
echo "=== AUDIT COMPLETE ==="
```

---

## 3. Port Scanner Script

```bash
#!/bin/bash
# port_scanner.sh — Simple TCP port scanner
set -uo pipefail

usage() {
    echo "Usage: $0 -t <target> [-p <ports>] [-o <output>]"
    echo "  -t  Target IP address"
    echo "  -p  Ports (comma-separated, default: common ports)"
    echo "  -o  Output file"
    exit 1
}

# Default ports
PORTS="21,22,23,25,53,80,110,143,443,445,993,995,3306,3389,5432,8080,8443"

while getopts "t:p:o:h" opt; do
    case $opt in
        t) TARGET="$OPTARG" ;;
        p) PORTS="$OPTARG" ;;
        o) OUTPUT="$OPTARG" ;;
        h) usage ;;
        *) usage ;;
    esac
done

[[ -z "${TARGET:-}" ]] && usage

echo "=== Port Scan: $TARGET ==="
echo "Date: $(date)"
echo ""

IFS=',' read -ra port_array <<< "$PORTS"
open_count=0

for port in "${port_array[@]}"; do
    timeout 1 bash -c "echo >/dev/tcp/$TARGET/$port" 2>/dev/null
    if [ $? -eq 0 ]; then
        result="[OPEN]   $TARGET:$port"
        echo "$result"
        ((open_count++))
    fi
done

echo ""
echo "Scan complete: $open_count open ports found"

# Save to file if specified
[[ -n "${OUTPUT:-}" ]] && echo "Results saved to $OUTPUT"
```

---

## 4. Service Monitoring Script

```bash
#!/bin/bash
# service_monitor.sh — Monitor critical services and alert
set -uo pipefail

SERVICES=("sshd" "nginx" "mysql" "fail2ban")
ALERT_EMAIL="admin@example.com"
LOG="/var/log/service_monitor.log"

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG"; }

for service in "${SERVICES[@]}"; do
    if systemctl is-active "$service" &>/dev/null; then
        log "[OK] $service is running"
    else
        log "[ALERT] $service is DOWN! Attempting restart..."
        systemctl start "$service" 2>/dev/null
        
        if systemctl is-active "$service" &>/dev/null; then
            log "[RECOVERED] $service restarted successfully"
        else
            log "[CRITICAL] $service failed to restart!"
            # Send alert (if mail is configured)
            echo "$service is down on $(hostname)" | \
                mail -s "SERVICE ALERT: $service DOWN" "$ALERT_EMAIL" 2>/dev/null
        fi
    fi
done
```

---

## 5. Cron Automation

```bash
# Schedule security scripts with cron
# Edit crontab: crontab -e

# Run hardening check daily at 2 AM
0 2 * * * /opt/security/system_audit.sh >> /var/log/audit.log 2>&1

# Run port scan weekly on Sundays
0 3 * * 0 /opt/security/port_scanner.sh -t 10.0.0.0/24 -o /var/log/portscan.log

# Monitor services every 5 minutes
*/5 * * * * /opt/security/service_monitor.sh

# Check for rootkits daily
0 4 * * * /usr/bin/rkhunter --check --skip-keypress >> /var/log/rkhunter.log

# Cron format: minute hour day month weekday command
```

---

## Summary Table

| Script Type | Purpose |
|-------------|---------|
| **Hardening** | Automate system security configuration |
| **User audit** | Find weak accounts, unauthorized users |
| **Port scanner** | Discover open ports on targets |
| **Service monitor** | Auto-restart failed services, alert |
| **Cron** | Schedule regular security checks |

---

## Quick Revision Questions

1. **Why is `set -euo pipefail` important in security scripts?**
2. **How do you schedule a script to run every 5 minutes?**
3. **What should a user audit script check for?**
4. **How do you scan a TCP port using only bash (no nmap)?**
5. **What is the purpose of a service monitoring script?**

---

[← Previous: Bash Basics Review](01-bash-basics-review.md) | [Next: Log Parsing Scripts →](03-log-parsing-scripts.md)

---

*Unit 8 - Topic 2 of 5 | [Back to README](../README.md)*
