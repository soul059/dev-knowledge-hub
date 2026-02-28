# Chapter 6: Disk Monitoring & Troubleshooting

[← Previous: RAID](05-raid.md) | [Back to README](../README.md) | [Next: Bash Basics →](../07-Shell-Scripting/01-bash-basics.md)

---

## Overview

Proactive disk monitoring prevents outages caused by full disks, failing hardware, and I/O bottlenecks. This chapter covers `iostat`, `iotop`, SMART monitoring, and alerting strategies used in production DevOps environments.

---

## 6.1 Monitoring Disk Space

```bash
# Disk space usage
df -h                          # Human-readable
df -hT                         # Include filesystem type
df -h /data                    # Specific mount

# Inode usage
df -i                          # Can run out even with space available

# Watch disk usage
watch -n 5 'df -h | grep -E "/$|/data|/var"'

# Quick alert check
df -h | awk '$5+0 > 80 {print "WARNING:", $6, "is", $5, "full"}'
```

---

## 6.2 I/O Performance Monitoring

### `iostat` — Disk I/O Statistics

```bash
# Install
sudo apt install sysstat       # Ubuntu
sudo dnf install sysstat       # RHEL

# Basic usage
iostat                         # Summary since boot
iostat -x 2 5                  # Extended, every 2 sec, 5 times
iostat -xd 1                   # Disk extended, continuous

# Key columns
# Device   r/s   w/s  rMB/s  wMB/s  rrqm/s  wrqm/s  await  %util
# sda      50    120  1.2    5.6    0.5     12.3    2.5    15%
```

```
┌──────────────────────────────────────────────────────────────┐
│           IOSTAT KEY METRICS                                  │
├──────────────┬───────────────────────────────────────────────┤
│ Metric       │ Meaning & Thresholds                          │
├──────────────┼───────────────────────────────────────────────┤
│ r/s, w/s     │ Reads/writes per second                       │
│ rMB/s, wMB/s │ Read/write throughput in MB/s                 │
│ await        │ Avg I/O wait time (ms) — <10 ms good          │
│ svctm        │ Service time (deprecated but useful)          │
│ %util        │ % time disk is busy                           │
│              │   < 70%: OK                                   │
│              │   > 90%: Disk is bottleneck!                  │
│ rrqm/s       │ Read requests merged per second               │
│ wrqm/s       │ Write requests merged per second              │
│ avgqu-sz     │ Average queue length — high = overloaded      │
│ avgrq-sz     │ Average request size (sectors)                │
└──────────────┴───────────────────────────────────────────────┘
```

### `iotop` — I/O Top (Per-Process)

```bash
# Install
sudo apt install iotop        # Ubuntu
sudo dnf install iotop        # RHEL

# Run (requires root)
sudo iotop                    # Interactive
sudo iotop -o                 # Only processes doing I/O
sudo iotop -a                 # Accumulated I/O
sudo iotop -P                 # Show processes (not threads)
sudo iotop -b -n 5            # Batch mode, 5 iterations
```

### `pidstat` — Per-Process I/O

```bash
pidstat -d 1                   # Disk I/O per process, every 1 sec
pidstat -d -p $(pgrep mysql) 1 # Specific process
```

---

## 6.3 SMART Monitoring (Disk Health)

```bash
# Install smartmontools
sudo apt install smartmontools     # Ubuntu
sudo dnf install smartmontools     # RHEL

# Check SMART support
sudo smartctl -i /dev/sda

# Quick health check
sudo smartctl -H /dev/sda
# PASSED = healthy
# FAILED = replace disk IMMEDIATELY

# Full SMART data
sudo smartctl -a /dev/sda

# Key attributes to watch
sudo smartctl -A /dev/sda
```

```
┌──────────────────────────────────────────────────────────────┐
│         CRITICAL SMART ATTRIBUTES                             │
├──────────────────────┬───────────────────────────────────────┤
│ Attribute            │ Warning                               │
├──────────────────────┼───────────────────────────────────────┤
│ Reallocated_Sector_Ct│ > 0 = bad sectors remapped           │
│ Current_Pending_Sect │ > 0 = sectors waiting for remap       │
│ Offline_Uncorrectable│ > 0 = unrecoverable read errors       │
│ UDMA_CRC_Error_Count │ > 0 = cable/connection issue          │
│ Temperature_Celsius  │ > 50°C = cooling needed               │
│ Power_On_Hours       │ Lifespan indicator                    │
│ Wear_Leveling_Count  │ SSD wear indicator                    │
└──────────────────────┴───────────────────────────────────────┘
```

```bash
# Enable automatic SMART monitoring
sudo systemctl enable --now smartd

# Schedule tests
sudo smartctl -t short /dev/sda    # ~2 minutes
sudo smartctl -t long /dev/sda     # ~hours (thorough)
sudo smartctl -l selftest /dev/sda # View test results
```

---

## 6.4 Disk Performance Testing

```bash
# hdparm — quick read speed test
sudo hdparm -t /dev/sda          # Buffered read
sudo hdparm -T /dev/sda          # Cached read

# dd — write speed test
dd if=/dev/zero of=/tmp/test bs=1M count=1024 oflag=direct
# Read it back
dd if=/tmp/test of=/dev/null bs=1M
rm /tmp/test

# fio — professional I/O benchmarking
sudo apt install fio
# Random read test
fio --name=randread --rw=randread --bs=4k --size=1G --numjobs=4 --time_based --runtime=30
# Sequential write
fio --name=seqwrite --rw=write --bs=1M --size=1G --numjobs=1 --time_based --runtime=30
```

---

## 6.5 Real-World Scenario: Disk Monitoring & Alerting Script

```bash
#!/bin/bash
# disk-monitor.sh — Run via cron every 5 minutes
# */5 * * * * /opt/scripts/disk-monitor.sh

LOG="/var/log/disk-monitor.log"
THRESHOLD=85
ALERT_EMAIL="ops@company.com"

echo "$(date '+%F %T') === Disk Check ===" >> "$LOG"

# Check disk space
while read -r line; do
    USAGE=$(echo "$line" | awk '{print $5}' | tr -d '%')
    MOUNT=$(echo "$line" | awk '{print $6}')
    
    if [ "$USAGE" -gt "$THRESHOLD" ]; then
        MSG="ALERT: $MOUNT is ${USAGE}% full on $(hostname)"
        echo "$MSG" >> "$LOG"
        echo "$MSG" | mail -s "Disk Alert: $(hostname)" "$ALERT_EMAIL" 2>/dev/null
    fi
done < <(df -h | grep '^/dev' | grep -v 'tmpfs')

# Check inode usage
while read -r line; do
    IUSE=$(echo "$line" | awk '{print $5}' | tr -d '%')
    MOUNT=$(echo "$line" | awk '{print $6}')
    
    if [ -n "$IUSE" ] && [ "$IUSE" -gt "$THRESHOLD" ]; then
        echo "ALERT: Inodes ${IUSE}% on $MOUNT" >> "$LOG"
    fi
done < <(df -i | grep '^/dev')

# Check SMART health
for disk in /dev/sd?; do
    if sudo smartctl -H "$disk" 2>/dev/null | grep -q "FAILED"; then
        echo "CRITICAL: SMART failure on $disk" >> "$LOG"
    fi
done

# Check I/O wait
IOWAIT=$(iostat -c 1 2 | tail -1 | awk '{print $4}')
if (( $(echo "$IOWAIT > 20" | bc -l 2>/dev/null) )); then
    echo "WARNING: High I/O wait: ${IOWAIT}%" >> "$LOG"
fi
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Disk 100% full | `df -h` | Clean logs, tmp, Docker images |
| Inodes exhausted | `df -i` shows 100% | Find dirs with many small files: `find / -xdev -printf '%h\n' \| sort \| uniq -c \| sort -rn \| head` |
| High I/O wait | `iostat` shows high `%util` | Identify process with `iotop`; optimize queries, add caching |
| SMART failure | `smartctl -H` shows FAILED | Replace disk immediately; backup first |
| Slow disk performance | `hdparm -t` shows low speed | Check cable, replace disk, use SSD |
| Disk write errors in logs | `dmesg \| grep -i error` | Check SMART, replace failing disk |

---

## Summary Table

| Tool | Purpose | Key Command |
|------|---------|------------|
| `df` | Disk space usage | `df -hT` |
| `du` | Directory sizes | `du -sh /var/*` |
| `iostat` | Disk I/O stats | `iostat -x 2` |
| `iotop` | Per-process I/O | `sudo iotop -o` |
| `smartctl` | Disk health (SMART) | `smartctl -H /dev/sda` |
| `hdparm` | Quick speed test | `hdparm -t /dev/sda` |
| `fio` | Professional benchmarks | `fio --name=test ...` |
| `lsblk` | Block device overview | `lsblk -f` |

---

## Quick Revision Questions

1. **What is `%util` in `iostat` output and what value indicates a problem?**
2. **How do you find which process is causing heavy disk I/O?**
3. **What SMART attributes indicate a failing disk?**
4. **Disk shows "no space" but `df -h` shows 60% used. What else should you check?**
5. **What is I/O wait and how does it affect system performance?**
6. **Write a one-liner to alert if any partition exceeds 90% usage.**

---

[← Previous: RAID](05-raid.md) | [Back to README](../README.md) | [Next: Bash Basics →](../07-Shell-Scripting/01-bash-basics.md)
