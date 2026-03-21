# Incident Response Scripts

## Unit 8 - Topic 5 | Bash Scripting for Security

---

## Overview

**Incident response (IR) scripts** automate the collection of forensic evidence during a security incident. Time is critical — IR scripts ensure nothing is missed, evidence is preserved, and volatile data is captured before it disappears. These scripts are part of every **IR toolkit**.

---

## 1. First Responder Script

```bash
#!/bin/bash
# first_responder.sh — Collect volatile data during incident
# ⚠️ RUN THIS FIRST — volatile data disappears on reboot!
set -uo pipefail

CASE_ID="${1:-CASE_$(date +%Y%m%d_%H%M%S)}"
EVIDENCE_DIR="/mnt/evidence/$CASE_ID"
mkdir -p "$EVIDENCE_DIR"

log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$EVIDENCE_DIR/collection.log"; }

log "=== INCIDENT RESPONSE: $CASE_ID ==="
log "Hostname: $(hostname)"
log "Investigator: $(whoami)"
log "Collection started"

# === VOLATILE DATA (collect first!) ===

# 1. System date/time
log "Collecting system time..."
date > "$EVIDENCE_DIR/01_system_time.txt"
uptime >> "$EVIDENCE_DIR/01_system_time.txt"

# 2. Logged-in users
log "Collecting logged-in users..."
who > "$EVIDENCE_DIR/02_logged_in_users.txt"
w >> "$EVIDENCE_DIR/02_logged_in_users.txt"

# 3. Running processes
log "Collecting process list..."
ps auxwwf > "$EVIDENCE_DIR/03_processes.txt"
ps -eo pid,ppid,user,args --sort=-pcpu > "$EVIDENCE_DIR/03_processes_sorted.txt"

# 4. Network connections
log "Collecting network connections..."
ss -anp > "$EVIDENCE_DIR/04_network_connections.txt"
ss -tlnp >> "$EVIDENCE_DIR/04_network_connections.txt"

# 5. Network configuration
log "Collecting network config..."
ip addr show > "$EVIDENCE_DIR/05_network_config.txt"
ip route show >> "$EVIDENCE_DIR/05_network_config.txt"
ip neigh show >> "$EVIDENCE_DIR/05_network_config.txt"
cat /etc/resolv.conf >> "$EVIDENCE_DIR/05_network_config.txt"

# 6. Open files
log "Collecting open files..."
lsof -nP > "$EVIDENCE_DIR/06_open_files.txt" 2>/dev/null

# 7. Loaded kernel modules
log "Collecting kernel modules..."
lsmod > "$EVIDENCE_DIR/07_kernel_modules.txt"

# 8. Environment variables
log "Collecting environment..."
env > "$EVIDENCE_DIR/08_environment.txt"
mount > "$EVIDENCE_DIR/08_mounts.txt"

# === NON-VOLATILE DATA ===

# 9. User accounts
log "Collecting user accounts..."
cat /etc/passwd > "$EVIDENCE_DIR/09_passwd.txt"
cat /etc/group > "$EVIDENCE_DIR/09_group.txt"
lastlog > "$EVIDENCE_DIR/09_lastlog.txt"
last -50 > "$EVIDENCE_DIR/09_last_logins.txt"

# 10. Scheduled tasks
log "Collecting scheduled tasks..."
for user in $(cut -d: -f1 /etc/passwd); do
    crontab -l -u "$user" 2>/dev/null && echo "=== $user ===" 
done > "$EVIDENCE_DIR/10_crontabs.txt"
ls -la /etc/cron.* >> "$EVIDENCE_DIR/10_crontabs.txt" 2>/dev/null
cat /etc/crontab >> "$EVIDENCE_DIR/10_crontabs.txt"

# 11. System logs
log "Collecting system logs..."
cp /var/log/auth.log "$EVIDENCE_DIR/11_auth.log" 2>/dev/null
cp /var/log/syslog "$EVIDENCE_DIR/11_syslog" 2>/dev/null
cp /var/log/kern.log "$EVIDENCE_DIR/11_kern.log" 2>/dev/null
journalctl --since "24 hours ago" > "$EVIDENCE_DIR/11_journal_24h.txt" 2>/dev/null

# 12. Hash evidence directory
log "Hashing all evidence files..."
find "$EVIDENCE_DIR" -type f -exec sha256sum {} \; > "$EVIDENCE_DIR/evidence_hashes.txt"

log "=== COLLECTION COMPLETE ==="
log "Evidence directory: $EVIDENCE_DIR"
log "Files collected: $(find "$EVIDENCE_DIR" -type f | wc -l)"
```

---

## 2. Malware Hunt Script

```bash
#!/bin/bash
# malware_hunt.sh — Hunt for indicators of compromise (IOCs)
set -uo pipefail

EVIDENCE_DIR="${1:-/mnt/evidence/malware_hunt}"
mkdir -p "$EVIDENCE_DIR"

echo "=== MALWARE HUNT ==="
echo "Date: $(date)"
echo ""

# 1. Suspicious processes
echo "--- SUSPICIOUS PROCESSES ---"
# Processes running from /tmp, /dev/shm, or /var/tmp
ps aux | awk '$11 ~ /\/(tmp|dev\/shm|var\/tmp)/' | tee "$EVIDENCE_DIR/suspicious_procs.txt"

# Processes with no associated binary
echo ""
echo "--- PROCESSES WITH DELETED BINARIES ---"
ls -la /proc/*/exe 2>/dev/null | grep "(deleted)" | tee -a "$EVIDENCE_DIR/suspicious_procs.txt"

# 2. Suspicious files
echo ""
echo "--- RECENTLY MODIFIED FILES IN SENSITIVE DIRS ---"
find /tmp /var/tmp /dev/shm -type f -mtime -7 2>/dev/null | tee "$EVIDENCE_DIR/suspicious_files.txt"

echo ""
echo "--- SETUID/SETGID FILES ---"
find / -perm -4000 -o -perm -2000 -type f 2>/dev/null | tee "$EVIDENCE_DIR/suid_files.txt"

echo ""
echo "--- WORLD-WRITABLE FILES IN SYSTEM DIRS ---"
find /usr /etc -perm -o+w -type f 2>/dev/null | tee "$EVIDENCE_DIR/world_writable.txt"

# 3. Suspicious network activity
echo ""
echo "--- CONNECTIONS TO UNUSUAL PORTS ---"
ss -tn state established | awk 'NR>1 {print $5}' | \
    grep -E ':(4444|5555|6666|1337|31337|8888|9999|1234)$' | \
    tee "$EVIDENCE_DIR/suspicious_connections.txt"

# 4. Persistence mechanisms
echo ""
echo "--- SUSPICIOUS CRON ENTRIES ---"
grep -r "wget\|curl\|nc\|ncat\|/dev/tcp\|base64\|eval" /etc/cron* /var/spool/cron 2>/dev/null | \
    tee "$EVIDENCE_DIR/suspicious_cron.txt"

echo ""
echo "--- SUSPICIOUS STARTUP ENTRIES ---"
find /etc/init.d /etc/rc*.d /etc/systemd/system -type f -mtime -30 2>/dev/null | \
    tee "$EVIDENCE_DIR/recent_startup_changes.txt"

# 5. Rootkit indicators
echo ""
echo "--- HIDDEN PROCESSES (comparing ps vs /proc) ---"
ps_count=$(ps aux | wc -l)
proc_count=$(ls -d /proc/[0-9]* 2>/dev/null | wc -l)
echo "ps reports: $ps_count processes"
echo "/proc shows: $proc_count processes"
[[ $((proc_count - ps_count)) -gt 5 ]] && echo "⚠️ POSSIBLE HIDDEN PROCESSES!"

echo ""
echo "=== HUNT COMPLETE ==="
```

---

## 3. IP Block Script

```bash
#!/bin/bash
# block_ip.sh — Emergency IP blocking during incident
set -euo pipefail

[[ "$(id -u)" -ne 0 ]] && { echo "Run as root!"; exit 1; }

ACTION="${1:-}"
IP="${2:-}"

usage() {
    echo "Usage: $0 <block|unblock|list> <IP>"
    echo "  $0 block 10.0.0.99"
    echo "  $0 unblock 10.0.0.99"
    echo "  $0 list"
    exit 1
}

[[ -z "$ACTION" ]] && usage

case "$ACTION" in
    block)
        [[ -z "$IP" ]] && usage
        iptables -I INPUT -s "$IP" -j DROP
        iptables -I OUTPUT -d "$IP" -j DROP
        echo "[$(date)] BLOCKED: $IP" | tee -a /var/log/ip_blocks.log
        echo "✅ $IP blocked (inbound + outbound)"
        
        # Kill existing connections from this IP
        ss -K dst "$IP" 2>/dev/null
        echo "Existing connections terminated"
        ;;
    unblock)
        [[ -z "$IP" ]] && usage
        iptables -D INPUT -s "$IP" -j DROP 2>/dev/null
        iptables -D OUTPUT -d "$IP" -j DROP 2>/dev/null
        echo "[$(date)] UNBLOCKED: $IP" | tee -a /var/log/ip_blocks.log
        echo "✅ $IP unblocked"
        ;;
    list)
        echo "--- CURRENTLY BLOCKED IPs ---"
        iptables -L INPUT -n | grep DROP
        ;;
    *)
        usage
        ;;
esac
```

---

## 4. Evidence Timeline Generator

```bash
#!/bin/bash
# timeline.sh — Generate filesystem activity timeline
set -euo pipefail

TARGET="${1:-/}"
HOURS="${2:-24}"
OUTPUT="timeline_$(date +%Y%m%d_%H%M%S).csv"

echo "=== FILESYSTEM TIMELINE ==="
echo "Target: $TARGET"
echo "Timeframe: Last $HOURS hours"
echo ""

echo "Timestamp,Type,File" > "$OUTPUT"

# Modified files
find "$TARGET" -type f -mmin -$((HOURS * 60)) -printf "%T+ MODIFIED %p\n" 2>/dev/null | \
    sort >> "$OUTPUT"

# Accessed files
find "$TARGET" -type f -amin -$((HOURS * 60)) -printf "%A+ ACCESSED %p\n" 2>/dev/null | \
    sort >> "$OUTPUT"

# Changed files (metadata)
find "$TARGET" -type f -cmin -$((HOURS * 60)) -printf "%C+ CHANGED %p\n" 2>/dev/null | \
    sort >> "$OUTPUT"

total=$(wc -l < "$OUTPUT")
echo "Timeline entries: $((total - 1))"
echo "Saved to: $OUTPUT"
```

---

## Summary Table

| Script | Purpose |
|--------|---------|
| **First responder** | Collect all volatile + non-volatile evidence |
| **Malware hunt** | Find IOCs, suspicious files/processes |
| **IP block** | Emergency IP blocking during incident |
| **Timeline generator** | Create filesystem activity timeline |

---

## Quick Revision Questions

1. **Why must volatile data be collected first during IR?**
2. **Name 5 types of volatile data on a Linux system.**
3. **How do you detect processes running from /tmp?**
4. **What persistence mechanisms should you check during IR?**
5. **How do you emergency-block an IP with iptables?**

---

[← Previous: Network Monitoring Scripts](04-network-monitoring-scripts.md)

---

*Unit 8 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: Container Security →](../09-Container-Security/01-docker-security-basics.md)*
