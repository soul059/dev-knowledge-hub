# Linux Performance Tuning

A comprehensive guide to monitoring, analyzing, and optimizing Linux system performance.

## Table of Contents

1. [Performance Fundamentals](#1-performance-fundamentals)
2. [CPU Performance](#2-cpu-performance)
3. [Memory Performance](#3-memory-performance)
4. [Disk I/O Performance](#4-disk-io-performance)
5. [Network Performance](#5-network-performance)
6. [Process Performance](#6-process-performance)
7. [Kernel Tuning](#7-kernel-tuning)
8. [Application Profiling](#8-application-profiling)
9. [Benchmarking](#9-benchmarking)
10. [Performance Monitoring Tools](#10-performance-monitoring-tools)

---

## 1. Performance Fundamentals

### The USE Method

For every resource, check:
- **U**tilization: How busy is the resource?
- **S**aturation: How much work is queued?
- **E**rrors: Are there any errors?

```
┌──────────────────────────────────────────────────────────┐
│                  System Resources                         │
├───────────┬──────────────┬─────────────┬─────────────────┤
│    CPU    │    Memory    │    Disk     │    Network      │
├───────────┼──────────────┼─────────────┼─────────────────┤
│Utilization│ Utilization  │ Utilization │ Utilization     │
│Saturation │ Saturation   │ Saturation  │ Saturation      │
│  Errors   │   Errors     │   Errors    │   Errors        │
└───────────┴──────────────┴─────────────┴─────────────────┘
```

### Quick System Overview

```bash
# Instant system snapshot
uptime                    # Load averages
free -h                   # Memory usage
df -h                     # Disk space
vmstat 1 5                # Virtual memory stats
iostat -x 1 5             # I/O statistics
sar -u 1 5                # CPU utilization

# All-in-one tools
htop                      # Interactive process viewer
glances                   # System monitoring
nmon                      # Performance monitor
```

### Load Average

```bash
# Check load average
uptime
# Output: load average: 1.50, 2.00, 2.50
#         (1 min) (5 min) (15 min)

# Rule of thumb:
# - Load < CPU cores: System is underutilized
# - Load = CPU cores: System is fully utilized
# - Load > CPU cores: System is overloaded

# Check CPU cores
nproc
grep -c processor /proc/cpuinfo
```

---

## 2. CPU Performance

### CPU Monitoring

```bash
# Real-time CPU usage
top
htop

# CPU statistics
mpstat 1 5                # Per-CPU stats
mpstat -P ALL 1           # All CPUs

# Process CPU usage
pidstat 1                 # Per-process CPU
pidstat -u 1 5            # CPU only

# CPU info
lscpu
cat /proc/cpuinfo
```

### CPU Bottleneck Signs

```bash
# High CPU usage indicators
vmstat 1
# Look for:
# - High 'us' (user CPU) or 'sy' (system CPU)
# - High 'r' (run queue) compared to CPU count
# - Low 'id' (idle)

# Check for CPU-bound processes
ps aux --sort=-%cpu | head
top -o %CPU
```

### CPU Optimization

```bash
# Set CPU affinity (pin process to CPUs)
taskset -c 0,1 ./myapp              # Use CPU 0 and 1
taskset -cp 0-3 PID                 # Assign to CPUs 0-3

# Change scheduling priority
nice -n 10 ./myapp                  # Lower priority (10)
nice -n -10 ./myapp                 # Higher priority (-10)
renice -n 5 -p PID                  # Change running process

# Real-time scheduling (use carefully)
chrt -f 99 ./myapp                  # FIFO, priority 99
chrt -r 50 ./myapp                  # Round-robin, priority 50
```

### CPU Governor

```bash
# Check current governor
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor

# Available governors
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors

# Set governor
echo "performance" | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# Governors:
# performance  - Max frequency
# powersave    - Min frequency
# ondemand     - Dynamic (default)
# conservative - Gradual changes
# schedutil    - Scheduler-driven
```

---

## 3. Memory Performance

### Memory Monitoring

```bash
# Memory usage
free -h
cat /proc/meminfo

# Detailed memory stats
vmstat 1
# Columns: si/so (swap in/out), bi/bo (block in/out)

# Per-process memory
ps aux --sort=-%mem | head
top -o %MEM

# Memory by process
pmap PID
smem -tk
```

### Memory Metrics

```bash
# Key metrics from /proc/meminfo
MemTotal:        Total physical RAM
MemFree:         Completely unused RAM
MemAvailable:    Available for new processes
Buffers:         Block device cache
Cached:          Page cache
SwapTotal:       Total swap space
SwapFree:        Available swap
Dirty:           Waiting to be written to disk
```

### Swap Management

```bash
# Check swap usage
swapon --show
free -h

# Create swap file
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Add to fstab
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Adjust swappiness (0-100)
cat /proc/sys/vm/swappiness
sudo sysctl vm.swappiness=10
echo "vm.swappiness=10" | sudo tee -a /etc/sysctl.conf
```

### Memory Optimization

```bash
# Clear page cache (not recommended for production)
sync; echo 1 > /proc/sys/vm/drop_caches  # Page cache
sync; echo 2 > /proc/sys/vm/drop_caches  # Dentries/inodes
sync; echo 3 > /proc/sys/vm/drop_caches  # Both

# Memory overcommit settings
cat /proc/sys/vm/overcommit_memory
# 0 = Heuristic (default)
# 1 = Always overcommit
# 2 = Don't overcommit

# OOM killer tuning (per process)
echo -17 > /proc/PID/oom_adj           # Protect from OOM
echo 1000 > /proc/PID/oom_score_adj    # Priority for OOM
```

### Huge Pages

```bash
# Check huge pages
grep Huge /proc/meminfo

# Enable transparent huge pages
cat /sys/kernel/mm/transparent_hugepage/enabled
echo "always" | sudo tee /sys/kernel/mm/transparent_hugepage/enabled

# Allocate static huge pages
echo 128 | sudo tee /proc/sys/vm/nr_hugepages

# Mount hugetlbfs
sudo mkdir /mnt/hugepages
sudo mount -t hugetlbfs none /mnt/hugepages
```

---

## 4. Disk I/O Performance

### I/O Monitoring

```bash
# I/O statistics
iostat -x 1
# Key columns:
# %util    - Device utilization
# await    - Average I/O time (ms)
# r/s, w/s - Reads/writes per second
# rMB/s, wMB/s - Throughput

# Per-process I/O
iotop
pidstat -d 1

# Disk latency
ioping /dev/sda
```

### I/O Bottleneck Signs

```bash
# High I/O wait
vmstat 1
# Look for high 'wa' (I/O wait)

# Disk saturation
iostat -x 1
# %util > 90% indicates saturation

# Check for I/O bound processes
iotop -o  # Only show active I/O
```

### I/O Scheduler

```bash
# Check current scheduler
cat /sys/block/sda/queue/scheduler

# Available schedulers
# [mq-deadline] kyber bfq none

# Change scheduler
echo "mq-deadline" | sudo tee /sys/block/sda/queue/scheduler

# Scheduler recommendations:
# SSD: none or mq-deadline
# HDD: bfq or mq-deadline
# Database: mq-deadline
```

### I/O Optimization

```bash
# Read-ahead buffer
blockdev --getra /dev/sda
sudo blockdev --setra 4096 /dev/sda  # 2MB read-ahead

# Deadline scheduler tuning
echo 500 > /sys/block/sda/queue/iosched/read_expire
echo 5000 > /sys/block/sda/queue/iosched/write_expire

# NVMe optimization
# Usually no tuning needed

# Filesystem mount options
# Add to /etc/fstab:
# noatime    - Don't update access times
# nodiratime - Don't update directory access times
# discard    - Enable TRIM for SSDs
```

### File System Tuning

```bash
# ext4 tuning
sudo tune2fs -o journal_data_writeback /dev/sda1
sudo tune2fs -O ^has_journal /dev/sda1  # Disable journal (risky)

# Mount options for performance
mount -o noatime,nodiratime,discard /dev/sda1 /mnt

# XFS tuning
mkfs.xfs -f -d agcount=4 /dev/sda1

# Check filesystem
df -Th
```

---

## 5. Network Performance

### Network Monitoring

```bash
# Interface statistics
ip -s link show eth0
cat /proc/net/dev

# Real-time bandwidth
iftop
nethogs
nload

# Connection statistics
ss -s
netstat -s

# TCP statistics
ss -ti
cat /proc/net/tcp
```

### Network Bottleneck Signs

```bash
# Check for dropped packets
ip -s link show eth0 | grep -E "dropped|errors"

# Check socket buffers
ss -m
netstat -an | grep -c ESTABLISHED

# TCP retransmissions
netstat -s | grep retransmit
```

### TCP Tuning

```bash
# /etc/sysctl.d/99-network-tuning.conf

# Increase socket buffer sizes
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.core.rmem_default = 1048576
net.core.wmem_default = 1048576

# TCP buffer sizes (min, default, max)
net.ipv4.tcp_rmem = 4096 1048576 16777216
net.ipv4.tcp_wmem = 4096 1048576 16777216

# Connection backlog
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 65535

# TCP optimizations
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_no_metrics_save = 1
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_keepalive_intvl = 15

# Enable TCP window scaling
net.ipv4.tcp_window_scaling = 1

# Increase ephemeral port range
net.ipv4.ip_local_port_range = 1024 65535

# TIME_WAIT optimization
net.ipv4.tcp_tw_reuse = 1

# Apply changes
sudo sysctl -p /etc/sysctl.d/99-network-tuning.conf
```

### Network Interface Tuning

```bash
# Check interface settings
ethtool eth0

# Enable ring buffer increase
ethtool -g eth0
sudo ethtool -G eth0 rx 4096 tx 4096

# Enable interrupt coalescing
sudo ethtool -C eth0 rx-usecs 100

# Enable receive-side scaling (RSS)
ethtool -l eth0
sudo ethtool -L eth0 combined 8

# Offloading features
ethtool -k eth0
sudo ethtool -K eth0 gso on gro on tso on
```

---

## 6. Process Performance

### Process Monitoring

```bash
# Process list
ps aux
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu

# Real-time monitoring
top
htop

# Process tree
pstree -p

# Process statistics
pidstat -u -d -r 1
```

### Process Limits

```bash
# Check limits
ulimit -a

# Set limits (current session)
ulimit -n 65536      # Open files
ulimit -u 65536      # Max processes

# Permanent limits (/etc/security/limits.conf)
* soft nofile 65536
* hard nofile 65536
* soft nproc 65536
* hard nproc 65536
root soft nofile 65536
root hard nofile 65536

# Systemd service limits
# In service file:
[Service]
LimitNOFILE=65536
LimitNPROC=65536
```

### Process Priority

```bash
# View priorities
ps -eo pid,ni,pri,comm

# Set nice value (priority)
nice -n 10 ./myapp          # Start with nice 10
renice -n 5 -p PID          # Change running process

# I/O priority
ionice -c 2 -n 0 ./myapp    # Best effort, high priority
ionice -c 3 ./myapp         # Idle only

# Classes: 1=realtime, 2=best-effort, 3=idle
```

---

## 7. Kernel Tuning

### Sysctl Configuration

```bash
# /etc/sysctl.d/99-performance.conf

# Memory
vm.swappiness = 10
vm.dirty_ratio = 15
vm.dirty_background_ratio = 5
vm.vfs_cache_pressure = 50

# Network
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_fastopen = 3

# File system
fs.file-max = 2097152
fs.inotify.max_user_watches = 524288

# Kernel
kernel.pid_max = 65536
kernel.sched_migration_cost_ns = 5000000

# Apply
sudo sysctl -p /etc/sysctl.d/99-performance.conf
```

### Virtual Memory Tuning

```bash
# Dirty page settings
vm.dirty_ratio = 15              # % of RAM for dirty pages
vm.dirty_background_ratio = 5    # Start flushing at this %
vm.dirty_writeback_centisecs = 100   # Flush interval
vm.dirty_expire_centisecs = 200      # Max age before flush

# For databases (reduce dirty ratios)
vm.dirty_ratio = 10
vm.dirty_background_ratio = 3

# For write-heavy workloads (increase ratios)
vm.dirty_ratio = 40
vm.dirty_background_ratio = 10
```

### Kernel Boot Parameters

```bash
# /etc/default/grub
GRUB_CMDLINE_LINUX="mitigations=off transparent_hugepage=madvise"

# Update GRUB
sudo update-grub
# Or
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

## 8. Application Profiling

### strace (System Calls)

```bash
# Trace system calls
strace ./myapp
strace -p PID

# Count system calls
strace -c ./myapp

# Trace specific calls
strace -e open,read,write ./myapp

# Trace with timing
strace -T ./myapp
strace -tt ./myapp
```

### ltrace (Library Calls)

```bash
# Trace library calls
ltrace ./myapp
ltrace -p PID

# Count library calls
ltrace -c ./myapp
```

### perf (Performance Counters)

```bash
# Install perf
sudo apt install linux-tools-common linux-tools-$(uname -r)

# CPU profiling
sudo perf record -g ./myapp
sudo perf report

# Real-time top
sudo perf top

# Statistics
sudo perf stat ./myapp

# Specific events
sudo perf record -e cache-misses ./myapp
```

### Flame Graphs

```bash
# Record
sudo perf record -F 99 -g ./myapp

# Generate flame graph
git clone https://github.com/brendangregg/FlameGraph
sudo perf script | ./FlameGraph/stackcollapse-perf.pl | ./FlameGraph/flamegraph.pl > flame.svg
```

### Memory Profiling

```bash
# Valgrind (memory leaks)
valgrind --leak-check=full ./myapp

# Memory usage over time
valgrind --tool=massif ./myapp
ms_print massif.out.*

# Heap profiler
valgrind --tool=massif --heap=yes ./myapp
```

---

## 9. Benchmarking

### CPU Benchmarks

```bash
# sysbench
sysbench cpu --cpu-max-prime=20000 run
sysbench cpu --threads=4 --cpu-max-prime=20000 run

# stress-ng
stress-ng --cpu 4 --timeout 60s --metrics

# Geekbench (download separately)
./geekbench5
```

### Memory Benchmarks

```bash
# sysbench memory
sysbench memory run
sysbench memory --memory-block-size=1M --memory-total-size=10G run

# stress-ng
stress-ng --vm 2 --vm-bytes 1G --timeout 60s --metrics

# memtest86+ (from boot)
```

### Disk Benchmarks

```bash
# fio
fio --name=randread --ioengine=libaio --iodepth=32 \
    --rw=randread --bs=4k --direct=1 --size=1G \
    --numjobs=4 --runtime=60 --group_reporting

# dd (simple)
dd if=/dev/zero of=testfile bs=1M count=1024 conv=fdatasync
dd if=testfile of=/dev/null bs=1M

# hdparm
sudo hdparm -Tt /dev/sda

# ioping (latency)
ioping -c 10 .
```

### Network Benchmarks

```bash
# iperf3
# Server
iperf3 -s

# Client
iperf3 -c server_ip
iperf3 -c server_ip -P 10  # 10 parallel streams
iperf3 -c server_ip -u     # UDP

# netperf
netperf -H server_ip
```

### Web Server Benchmarks

```bash
# Apache Bench
ab -n 10000 -c 100 http://localhost/

# wrk
wrk -t12 -c400 -d30s http://localhost/

# hey
hey -n 10000 -c 100 http://localhost/
```

---

## 10. Performance Monitoring Tools

### Real-Time Monitoring

| Tool | Purpose |
|------|---------|
| top/htop | Process monitoring |
| vmstat | Virtual memory stats |
| iostat | I/O statistics |
| sar | System activity reporter |
| dstat | Versatile resource stats |
| glances | All-in-one monitoring |
| nmon | Performance monitor |

### Logging and Historical Data

```bash
# sar (System Activity Reporter)
# Collect data
sar -o /var/log/sa/sa$(date +%d) 60 &

# View historical data
sar -u -f /var/log/sa/sa01    # CPU
sar -r -f /var/log/sa/sa01    # Memory
sar -b -f /var/log/sa/sa01    # I/O
sar -n DEV -f /var/log/sa/sa01  # Network
```

### Monitoring Stack

```bash
# Prometheus + Node Exporter + Grafana
# Node Exporter exposes system metrics
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvf node_exporter-*.tar.gz
./node_exporter

# Metrics at http://localhost:9100/metrics
```

### Essential Commands Summary

```bash
# Quick diagnostics
uptime                    # Load averages
free -h                   # Memory
df -h                     # Disk space
vmstat 1                  # System stats
iostat -x 1               # I/O stats
ss -s                     # Network connections

# Deep dive
htop                      # Processes
iotop                     # I/O by process
nethogs                   # Network by process
perf top                  # CPU profiling
strace -p PID             # System calls
```

---

## Performance Tuning Checklist

```
☐ CPU
  ☐ Check load average vs CPU count
  ☐ Identify CPU-bound processes
  ☐ Set appropriate CPU governor
  ☐ Consider CPU affinity for critical processes

☐ Memory
  ☐ Monitor memory usage and swap
  ☐ Tune swappiness
  ☐ Configure huge pages if beneficial
  ☐ Set appropriate OOM killer scores

☐ Disk I/O
  ☐ Choose appropriate I/O scheduler
  ☐ Tune read-ahead buffer
  ☐ Use noatime mount option
  ☐ Consider SSD-specific optimizations

☐ Network
  ☐ Tune TCP buffer sizes
  ☐ Increase connection backlog
  ☐ Enable TCP fastopen
  ☐ Configure network interface offloading

☐ Processes
  ☐ Set appropriate ulimits
  ☐ Use nice/ionice for prioritization
  ☐ Configure systemd resource limits

☐ Monitoring
  ☐ Set up continuous monitoring
  ☐ Configure alerting thresholds
  ☐ Establish performance baselines
```

---

*Last Updated: April 2026*
