# Chapter 2: System Monitoring

[← Previous: Log Management](01-log-management.md) | [Back to README](../README.md) | [Next: Performance Tuning →](03-performance-tuning.md)

---

## Overview

System monitoring is the practice of continuously observing CPU, memory, disk, and network metrics to detect issues before they impact services. This chapter covers essential monitoring tools, from quick terminal commands to persistent monitoring setups used in DevOps production environments.

---

## 2.1 Quick System Overview

```bash
# Instant system summary
uptime                   # Load average & uptime
uname -a                 # Kernel and architecture
hostname                 # System hostname
hostnamectl              # Detailed host info
lscpu                    # CPU details
free -h                  # Memory summary
df -hT                   # Disk summary
ip -4 addr show          # Network interfaces
```

---

## 2.2 CPU Monitoring

```bash
# Load average
uptime
# 10:30:00 up 42 days, load average: 0.52, 0.58, 0.49
#                                     1min  5min  15min
# Rule of thumb: load avg ≤ number of CPU cores = healthy
# Check cores: nproc

# CPU usage with top
top                      # Interactive
top -bn1                 # Batch mode (1 iteration for scripts)
top -bn1 | head -20      # Quick snapshot

# CPU usage with mpstat
mpstat 2 5               # Every 2 sec, 5 times
mpstat -P ALL 1          # Per-CPU stats

# CPU usage with vmstat
vmstat 2 5               # System stats every 2 sec
```

```
┌──────────────────────────────────────────────────────────────┐
│         CPU METRICS EXPLAINED                                 │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  top output header:                                           │
│  %Cpu(s):  5.2 us,  2.1 sy,  0.0 ni, 91.5 id,               │
│            0.8 wa,  0.0 hi,  0.4 si,  0.0 st                 │
│                                                               │
│  us = user      — Application CPU time                        │
│  sy = system    — Kernel CPU time                             │
│  ni = nice      — Low-priority processes                      │
│  id = idle      — Idle percentage (higher = less busy)        │
│  wa = iowait    — Waiting for I/O (disk bottleneck)          │
│  hi = hardware  — Hardware interrupts                         │
│  si = software  — Software interrupts                         │
│  st = steal     — VM hypervisor stealing CPU (cloud VMs)     │
│                                                               │
│  Alerts:                                                      │
│  · wa > 20% → Disk I/O bottleneck                             │
│  · st > 10% → Noisy neighbor on shared VM                     │
│  · us + sy > 80% → CPU saturated                              │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.3 Memory Monitoring

```bash
# Memory summary
free -h
#               total    used    free    shared  buff/cache  available
# Mem:           16G     4.2G    1.8G    256M    10.0G       11.3G
# Swap:          4.0G    0B      4.0G

# Key insight: "available" is what matters, not "free"
# Linux uses free RAM for disk cache (buff/cache) — this is normal!

# Detailed memory info
cat /proc/meminfo
cat /proc/meminfo | grep -E "MemTotal|MemFree|MemAvailable|SwapTotal|SwapFree"

# Per-process memory
ps aux --sort=-%mem | head -10     # Top 10 by memory
smem -tk                            # More accurate (PSS)

# Swap usage
swapon --show
cat /proc/swaps
vmstat 1 5 | awk '{print $7, $8}'  # si (swap in), so (swap out)
```

```
┌──────────────────────────────────────────────────────────────┐
│         MEMORY USAGE — WHAT'S NORMAL?                        │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Total RAM: 16 GB                                            │
│  ┌──────────────────────────────────────────────────┐        │
│  │ Used (Apps)   │ Buff/Cache      │  Free          │        │
│  │ 4.2 GB        │ 10.0 GB         │  1.8 GB        │        │
│  └───────┬───────┴────────┬────────┴───────┬────────┘        │
│          │                │                │                  │
│          │    Available = Free + Reclaimable Cache            │
│          │    Available ≈ 11.3 GB                             │
│          │                                                    │
│  This is NORMAL and HEALTHY!                                  │
│  Linux uses free RAM as disk cache to speed up I/O.           │
│  The cache is released when applications need memory.         │
│                                                               │
│  Warning signs:                                               │
│  · Available < 10% of total                                   │
│  · Swap usage > 0 and growing                                 │
│  · OOM Killer messages in dmesg                               │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.4 Disk Monitoring

```bash
# Disk space
df -hT                    # All filesystems with types
df -h /                   # Specific mount
du -sh /var/log/*         # Directory sizes
du -sh /var/* | sort -rh | head -10  # Largest directories

# Inode usage (can run out even with space left)
df -i

# Disk I/O
iostat -x 2               # Extended stats every 2 sec
iotop -o                  # Per-process I/O (requires root)

# SMART disk health
sudo smartctl -H /dev/sda
```

---

## 2.5 Network Monitoring

```bash
# Connection overview
ss -tuln                   # Listening TCP/UDP ports
ss -tupn                   # Connected sockets with process
ss -s                      # Summary statistics

# Bandwidth monitoring
iftop                      # Per-connection bandwidth (requires root)
nethogs                    # Per-process bandwidth
nload                      # Per-interface bandwidth graph
sar -n DEV 1 5             # Network device stats

# Network errors
ip -s link                 # Interface statistics
cat /proc/net/dev          # Detailed network stats
netstat -i                 # Interface statistics
```

---

## 2.6 Top & Htop

### `top` Interactive Keys

```
┌──────────────────────────────────────────────────────────────┐
│         TOP INTERACTIVE SHORTCUTS                             │
├──────────┬───────────────────────────────────────────────────┤
│ Key      │ Action                                            │
├──────────┼───────────────────────────────────────────────────┤
│ 1        │ Toggle per-CPU display                            │
│ M        │ Sort by memory usage                              │
│ P        │ Sort by CPU usage                                 │
│ T        │ Sort by running time                              │
│ k        │ Kill a process (enter PID)                        │
│ r        │ Renice a process                                  │
│ c        │ Toggle full command line                          │
│ H        │ Toggle threads display                            │
│ V        │ Forest (tree) view                                │
│ f        │ Select display fields                             │
│ d        │ Change refresh interval                           │
│ q        │ Quit                                              │
└──────────┴───────────────────────────────────────────────────┘
```

### `htop` — Enhanced Interactive Monitor

```bash
# Install
sudo apt install htop      # Ubuntu
sudo dnf install htop      # RHEL

htop                       # Launch
# Features: mouse support, tree view, process filtering
# F2: Setup | F3: Search | F4: Filter | F5: Tree | F6: Sort
# F9: Kill | F10: Quit
```

---

## 2.7 `sar` — System Activity Reporter

```bash
# Install
sudo apt install sysstat
sudo systemctl enable --now sysstat

# CPU usage (historical)
sar -u 2 5                 # Every 2 sec, 5 times
sar -u                     # Today's history

# Memory
sar -r 2 5                 # Memory utilization
sar -S                     # Swap statistics

# Disk I/O
sar -d 2 5                 # Disk activity
sar -b                     # Overall I/O

# Network
sar -n DEV 2 5             # Network devices
sar -n SOCK                # Socket statistics

# Historical data (specific day)
sar -u -f /var/log/sysstat/sa15   # Day 15 of month

# Generate report
sar -A > system-report.txt        # All metrics
```

---

## 2.8 All-in-One Monitoring Tools

```bash
# glances — Cross-platform system monitor
sudo apt install glances
glances                    # Interactive
glances -w                 # Web server mode (port 61208)

# nmon — Performance monitor
sudo apt install nmon
nmon                       # Interactive
nmon -f -s 30 -c 120       # Record: every 30s for 1 hour

# dstat — Versatile resource statistics
sudo apt install dstat     # (or pcp-system-tools on newer systems)
dstat                      # CPU, disk, net, paging, system
dstat -cdngy               # Custom selection
dstat --top-cpu --top-mem  # Top consumers
```

---

## 2.9 Real-World Scenario: System Health Dashboard Script

```bash
#!/usr/bin/env bash
# sys-dashboard.sh — Quick system health dashboard

set -euo pipefail

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

status_color() {
    local val=$1 warn=$2 crit=$3
    if (( val >= crit )); then echo -e "${RED}${val}%${NC}"
    elif (( val >= warn )); then echo -e "${YELLOW}${val}%${NC}"
    else echo -e "${GREEN}${val}%${NC}"
    fi
}

clear
echo "=============================================="
echo "  SYSTEM HEALTH DASHBOARD — $(hostname)"
echo "  $(date '+%F %T %Z')"
echo "=============================================="

# Uptime
echo ""
echo "--- UPTIME ---"
uptime -p

# CPU
echo ""
echo "--- CPU ---"
CORES=$(nproc)
LOAD=$(awk '{print $1}' /proc/loadavg)
LOAD_PCT=$(echo "$LOAD $CORES" | awk '{printf "%.0f", ($1/$2)*100}')
echo "  Cores: $CORES | Load: $LOAD | Usage: $(status_color "$LOAD_PCT" 70 90)"

# Memory
echo ""
echo "--- MEMORY ---"
MEM_TOTAL=$(free -m | awk '/Mem:/ {print $2}')
MEM_USED=$(free -m | awk '/Mem:/ {print $3}')
MEM_PCT=$(( MEM_USED * 100 / MEM_TOTAL ))
echo "  Used: ${MEM_USED}M / ${MEM_TOTAL}M ($(status_color "$MEM_PCT" 80 95))"

SWAP_TOTAL=$(free -m | awk '/Swap:/ {print $2}')
SWAP_USED=$(free -m | awk '/Swap:/ {print $3}')
if (( SWAP_TOTAL > 0 )); then
    SWAP_PCT=$(( SWAP_USED * 100 / SWAP_TOTAL ))
    echo "  Swap:  ${SWAP_USED}M / ${SWAP_TOTAL}M ($(status_color "$SWAP_PCT" 50 80))"
fi

# Disk
echo ""
echo "--- DISK ---"
df -hT | grep -E '^/dev' | while read -r dev type size used avail pct mount; do
    USE_NUM=${pct%\%}
    echo "  $mount: $used/$size ($(status_color "$USE_NUM" 80 90)) [$type]"
done

# Top Processes
echo ""
echo "--- TOP 5 PROCESSES (CPU) ---"
ps aux --sort=-%cpu | awk 'NR>1 && NR<=6 {printf "  %-20s CPU: %5s  MEM: %5s\n", $11, $3"%", $4"%"}'

# Services
echo ""
echo "--- KEY SERVICES ---"
for svc in docker nginx mysql postgresql sshd; do
    if systemctl is-active --quiet "$svc" 2>/dev/null; then
        echo -e "  $svc: ${GREEN}running${NC}"
    elif systemctl list-unit-files | grep -q "$svc"; then
        echo -e "  $svc: ${RED}stopped${NC}"
    fi
done

echo ""
echo "=============================================="
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| High load average | `top` / `htop` | Identify top CPU consumer; check for runaway processes |
| High memory usage | `free -h`, `ps aux --sort=-%mem` | Check for memory leaks; increase swap temporarily |
| High I/O wait | `iostat -x 1`, `iotop` | Find I/O-heavy process; optimize DB queries, add SSD |
| OOM Killer triggered | `dmesg \| grep -i oom` | Increase RAM, tune `vm.overcommit` |
| CPU steal time | `top` shows `%st` | Contact cloud provider; upgrade instance |
| Network saturation | `iftop`, `nethogs` | Identify high-bandwidth process; add bandwidth |

---

## Summary Table

| Tool | Focus Area | Key Command |
|------|-----------|-------------|
| `top` / `htop` | Overall system | `htop` |
| `free` | Memory | `free -h` |
| `df` | Disk space | `df -hT` |
| `iostat` | Disk I/O | `iostat -x 2` |
| `ss` | Network connections | `ss -tuln` |
| `sar` | Historical metrics | `sar -u 2 5` |
| `vmstat` | VM statistics | `vmstat 2 5` |
| `mpstat` | Per-CPU stats | `mpstat -P ALL 1` |
| `iftop` | Network bandwidth | `sudo iftop` |
| `glances` | All-in-one | `glances` |

---

## Quick Revision Questions

1. **What does the load average represent and what is considered "high"?**
2. **Explain the difference between "free" and "available" memory in `free -h`.**
3. **How do you identify which process is causing high I/O wait?**
4. **What does CPU steal time (`%st`) indicate and when do you see it?**
5. **How do you get historical CPU/memory data using `sar`?**
6. **Write a command to find the top 5 memory-consuming processes.**

---

[← Previous: Log Management](01-log-management.md) | [Back to README](../README.md) | [Next: Performance Tuning →](03-performance-tuning.md)
