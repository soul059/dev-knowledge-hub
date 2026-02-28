# Chapter 3: Performance Tuning

[← Previous: System Monitoring](02-system-monitoring.md) | [Back to README](../README.md) | [Next: Backup & Recovery →](04-backup-and-recovery.md)

---

## Overview

Performance tuning involves adjusting kernel parameters, resource limits, and system configurations to optimize Linux servers for specific workloads. In DevOps, this means tuning for high-concurrency web servers, databases, container hosts, and CI build machines.

---

## 3.1 Kernel Parameters (sysctl)

```bash
# View all parameters
sysctl -a

# View specific parameter
sysctl net.core.somaxconn
sysctl vm.swappiness

# Set temporarily (lost on reboot)
sudo sysctl -w net.core.somaxconn=65535
sudo sysctl -w vm.swappiness=10

# Set permanently
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.d/99-tuning.conf
sudo sysctl -p /etc/sysctl.d/99-tuning.conf

# Apply all config files
sudo sysctl --system
```

---

## 3.2 Essential Tuning Parameters

```
┌──────────────────────────────────────────────────────────────┐
│        COMMONLY TUNED KERNEL PARAMETERS                       │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  MEMORY                                                       │
│  ├── vm.swappiness = 10                                       │
│  │   How aggressively kernel swaps (0-100, default 60)       │
│  │   Lower = prefer keeping data in RAM                       │
│  │                                                            │
│  ├── vm.dirty_ratio = 40                                      │
│  │   % of RAM for dirty pages before sync forced              │
│  │                                                            │
│  └── vm.overcommit_memory = 0                                 │
│      0=heuristic, 1=always allow, 2=never overcommit         │
│                                                               │
│  NETWORKING                                                   │
│  ├── net.core.somaxconn = 65535                               │
│  │   Max socket connection backlog                            │
│  │                                                            │
│  ├── net.core.netdev_max_backlog = 65535                      │
│  │   Max packets queued on INPUT                              │
│  │                                                            │
│  ├── net.ipv4.tcp_max_syn_backlog = 65535                     │
│  │   Max SYN queue size                                       │
│  │                                                            │
│  ├── net.ipv4.ip_local_port_range = 1024 65535                │
│  │   Ephemeral port range for outbound connections            │
│  │                                                            │
│  └── net.ipv4.tcp_tw_reuse = 1                                │
│      Reuse TIME-WAIT sockets for new connections             │
│                                                               │
│  FILE SYSTEM                                                  │
│  ├── fs.file-max = 2097152                                    │
│  │   System-wide maximum open files                           │
│  │                                                            │
│  └── fs.inotify.max_user_watches = 524288                     │
│      For file watchers (IDEs, build tools)                    │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Production Web/App Server Sysctl Profile

```bash
# /etc/sysctl.d/99-web-server.conf

# Memory
vm.swappiness = 10
vm.dirty_ratio = 40
vm.dirty_background_ratio = 10

# Networking — High concurrency
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_keepalive_intvl = 15

# File descriptors
fs.file-max = 2097152
fs.inotify.max_user_watches = 524288

# Security
net.ipv4.conf.all.rp_filter = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
```

---

## 3.3 Resource Limits (ulimit)

```bash
# View current limits
ulimit -a

# Key limits
ulimit -n          # Max open files (default: 1024 — too low!)
ulimit -u          # Max user processes
ulimit -v          # Max virtual memory
ulimit -l          # Max locked memory
ulimit -c          # Core file size

# Set temporarily (current session)
ulimit -n 65535    # Increase open files

# Set permanently: /etc/security/limits.conf
# Or /etc/security/limits.d/99-custom.conf
```

```bash
# /etc/security/limits.d/99-custom.conf

# Format: <domain> <type> <item> <value>

# For all users
*        soft    nofile    65535
*        hard    nofile    65535
*        soft    nproc     65535
*        hard    nproc     65535

# For specific user
nginx    soft    nofile    100000
nginx    hard    nofile    100000

# For systemd services, also set in unit file:
# [Service]
# LimitNOFILE=65535
# LimitNPROC=65535
```

```
┌──────────────────────────────────────────────────────────────┐
│         ULIMIT KEY VALUES                                     │
├──────────────┬───────────┬───────────────────────────────────┤
│ Flag         │ Default   │ Production Recommendation         │
├──────────────┼───────────┼───────────────────────────────────┤
│ -n (nofile)  │ 1024      │ 65535+                            │
│ -u (nproc)   │ varies    │ 65535+                            │
│ -l (memlock) │ 64 KB     │ unlimited (for Docker, DBs)       │
│ -c (core)    │ 0         │ unlimited (for debugging)         │
│ -s (stack)   │ 8192 KB   │ Usually fine as-is                │
└──────────────┴───────────┴───────────────────────────────────┘
```

---

## 3.4 I/O Scheduler Tuning

```bash
# Check current scheduler
cat /sys/block/sda/queue/scheduler
# [mq-deadline] kyber bfq none

# Change scheduler (temporary)
echo "none" | sudo tee /sys/block/sda/queue/scheduler    # For SSDs/NVMe
echo "mq-deadline" | sudo tee /sys/block/sda/queue/scheduler  # For HDDs

# Permanent: Add to GRUB
# GRUB_CMDLINE_LINUX="elevator=mq-deadline"
```

```
┌──────────────────────────────────────────────────────────────┐
│         I/O SCHEDULERS                                        │
├──────────────────┬───────────────────────────────────────────┤
│ Scheduler        │ Best For                                  │
├──────────────────┼───────────────────────────────────────────┤
│ none (noop)      │ SSDs, NVMe, VMs (no reordering needed)   │
│ mq-deadline      │ HDDs, databases (latency-focused)         │
│ bfq              │ Desktop, mixed workloads (fairness)       │
│ kyber            │ Fast SSDs (low latency)                    │
└──────────────────┴───────────────────────────────────────────┘
```

---

## 3.5 Process Priority & CPU Affinity

```bash
# Nice values: -20 (highest priority) to 19 (lowest)
nice -n 10 ./background-task.sh       # Lower priority
nice -n -5 ./important-task.sh        # Higher priority (needs root)

# Change priority of running process
renice -n 5 -p 12345
renice -n -10 -p $(pgrep mysql)       # Boost MySQL

# CPU affinity — pin process to specific cores
taskset -c 0,1 ./app            # Run on cores 0 and 1
taskset -p -c 2,3 12345         # Move PID to cores 2-3

# Check current affinity
taskset -p 12345

# cgroups — Resource limits for groups of processes
# Used internally by Docker, Kubernetes, systemd
# Manual example:
sudo cgcreate -g cpu,memory:/myapp
sudo cgset -r cpu.cfs_quota_us=50000 myapp      # 50% of 1 core
sudo cgset -r memory.limit_in_bytes=512M myapp   # 512MB RAM limit
sudo cgexec -g cpu,memory:/myapp ./app
```

---

## 3.6 Filesystem Tuning

```bash
# Mount options for performance
# /etc/fstab entry:
# /dev/sda1  /data  ext4  defaults,noatime,nodiratime  0 2

# noatime    — Don't update access times (big performance win)
# nodiratime — Don't update directory access times
# barrier=0  — Disable write barriers (risky but fast)
# data=writeback — Faster ext4 writes (risk on crash)

# Check current mount options
mount | grep sda1

# Tune ext4 filesystem
sudo tune2fs -o journal_data_writeback /dev/sda1

# Tune read-ahead (for sequential reads)
sudo blockdev --getra /dev/sda          # Current value
sudo blockdev --setra 4096 /dev/sda     # Set to 4096 sectors
```

---

## 3.7 Real-World Scenario: Server Tuning Script

```bash
#!/usr/bin/env bash
# tune-server.sh — Apply production tuning to a Linux server

set -euo pipefail

echo "=== Server Performance Tuning ==="
echo "Hostname: $(hostname)"
echo "Date: $(date)"
echo ""

# Check if root
[[ $EUID -eq 0 ]] || { echo "ERROR: Run as root"; exit 1; }

# 1. Sysctl tuning
echo "[1/4] Applying sysctl parameters..."
cat > /etc/sysctl.d/99-performance.conf << 'EOF'
# Memory
vm.swappiness = 10
vm.dirty_ratio = 40
vm.dirty_background_ratio = 10

# Network
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 15

# File system
fs.file-max = 2097152
fs.inotify.max_user_watches = 524288
EOF
sysctl --system > /dev/null 2>&1
echo "  Done"

# 2. Ulimits
echo "[2/4] Configuring resource limits..."
cat > /etc/security/limits.d/99-performance.conf << 'EOF'
*    soft    nofile    65535
*    hard    nofile    65535
*    soft    nproc     65535
*    hard    nproc     65535
EOF
echo "  Done (requires re-login)"

# 3. I/O scheduler for SSDs
echo "[3/4] Tuning I/O scheduler..."
for disk in /sys/block/sd* /sys/block/nvme*; do
    [ -d "$disk" ] || continue
    ROTATIONAL=$(cat "${disk}/queue/rotational" 2>/dev/null || echo "1")
    DEVICE=$(basename "$disk")
    if [ "$ROTATIONAL" -eq 0 ]; then
        echo "none" > "${disk}/queue/scheduler" 2>/dev/null || true
        echo "  $DEVICE: SSD → scheduler=none"
    else
        echo "mq-deadline" > "${disk}/queue/scheduler" 2>/dev/null || true
        echo "  $DEVICE: HDD → scheduler=mq-deadline"
    fi
done

# 4. Disable transparent hugepages (for databases)
echo "[4/4] Disabling transparent hugepages..."
if [ -f /sys/kernel/mm/transparent_hugepage/enabled ]; then
    echo never > /sys/kernel/mm/transparent_hugepage/enabled
    echo never > /sys/kernel/mm/transparent_hugepage/defrag
    echo "  Done"
else
    echo "  Not available"
fi

echo ""
echo "=== Tuning Complete ==="
echo "Note: Some changes require a re-login or reboot to take full effect."
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| "Too many open files" | `ulimit -n` shows 1024 | Increase in limits.conf and systemd unit |
| High swap usage on server with RAM | `vm.swappiness` too high | Set `vm.swappiness=10` |
| Connection refused under load | `somaxconn` too low | Increase `net.core.somaxconn` |
| Slow SSD performance | Wrong I/O scheduler | Set scheduler to `none` for SSDs |
| Database slow on VM | Transparent hugepages | Disable THP: `echo never > .../enabled` |
| Firewall connection tracking full | `nf_conntrack_max` too low | Increase `nf_conntrack_max` |

---

## Summary Table

| Area | Parameter / Tool | Typical Production Value |
|------|-----------------|-------------------------|
| Swap behavior | `vm.swappiness` | 10 |
| Open files | `ulimit -n` / limits.conf | 65535+ |
| TCP backlog | `net.core.somaxconn` | 65535 |
| Ephemeral ports | `ip_local_port_range` | 1024 65535 |
| File descriptors | `fs.file-max` | 2097152 |
| I/O scheduler | `/sys/block/*/queue/scheduler` | none (SSD), mq-deadline (HDD) |
| CPU priority | `nice`, `renice` | -5 to -10 for critical apps |
| CPU affinity | `taskset` | Pin to specific cores |
| Mount options | `noatime` | Add to /etc/fstab |

---

## Quick Revision Questions

1. **What is `vm.swappiness` and what value should you set for a production server?**
2. **How do you permanently increase the open file limit to 65535 for all users?**
3. **What I/O scheduler should be used for SSDs vs HDDs?**
4. **What does `net.core.somaxconn` control and why is the default too low?**
5. **Why should transparent hugepages be disabled for database servers?**
6. **How do you pin a process to specific CPU cores?**

---

[← Previous: System Monitoring](02-system-monitoring.md) | [Back to README](../README.md) | [Next: Backup & Recovery →](04-backup-and-recovery.md)
