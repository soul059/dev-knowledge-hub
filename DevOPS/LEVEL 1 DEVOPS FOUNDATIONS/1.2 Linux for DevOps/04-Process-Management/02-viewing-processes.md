# Chapter 2: Viewing Processes

[← Previous: Process Lifecycle](01-process-lifecycle.md) | [Back to README](../README.md) | [Next: Process Control →](03-process-control.md)

---

## Overview

Monitoring processes is a primary task for any DevOps engineer. Commands like `ps`, `top`, and `htop` let you see what's running, how much CPU and memory each process uses, and which processes might be causing issues.

---

## 2.1 `ps` — Process Snapshot

```bash
# Basic usage
ps                           # Your processes only
ps aux                       # All processes (BSD syntax)
ps -ef                       # All processes (POSIX syntax)

# Detailed columns
ps aux
# USER  PID %CPU %MEM   VSZ    RSS TTY STAT START  TIME COMMAND
# root    1  0.0  0.1 169436 11788 ?   Ss   Feb20   0:05 /lib/systemd/systemd

# Specific process
ps aux | grep nginx
ps -C nginx -o pid,user,%cpu,%mem,comm    # By command name

# Custom output format
ps -eo pid,ppid,user,%cpu,%mem,vsz,rss,stat,comm --sort=-%cpu | head -15

# Process tree
ps auxf                      # Forest view
ps -ejH                      # Process tree
pstree                       # Visual tree
pstree -p                    # With PIDs
pstree -u devops             # Specific user's tree

# Thread view
ps -eLf                      # Show threads
ps -p $(pgrep nginx) -T      # Threads of specific process

# By user
ps -u devops
ps -U root -u root

# Memory-sorted (largest first)
ps aux --sort=-%mem | head -10

# CPU-sorted
ps aux --sort=-%cpu | head -10
```

---

## 2.2 `top` — Real-Time Process Monitor

```
┌──────────────────────────────────────────────────────────────────┐
│ top - 10:30:45 up 5 days, 3:20, 2 users, load avg: 1.2,0.8,0.5│
│ Tasks: 234 total, 2 running, 230 sleeping, 0 stopped, 2 zombie │
│ %Cpu(s): 12.5 us, 3.2 sy, 0.0 ni, 82.3 id, 1.5 wa, 0.0 si    │
│ MiB Mem:  16384.0 total,  4096.0 free, 8192.0 used, 4096.0 cache│
│ MiB Swap:  2048.0 total,  2048.0 free,    0.0 used              │
│                                                                   │
│   PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM  CMD  │
│  1234 www-data  20   0  512000  128000  12000 S  15.2   0.8  nginx│
│  5678 root      20   0  800000  256000  24000 S   8.5   1.6  java │
│  9012 devops    20   0  120000   40000   8000 R   5.1   0.2  python│
└──────────────────────────────────────────────────────────────────┘
```

### Top Interactive Commands

```
┌──────────────────────────────────────────────────────────────┐
│              TOP INTERACTIVE SHORTCUTS                        │
├──────────────────────────────────────────────────────────────┤
│  SORTING:                                                     │
│  P  = Sort by CPU%           M  = Sort by Memory%            │
│  T  = Sort by Time           N  = Sort by PID                │
│                                                               │
│  DISPLAY:                                                     │
│  1  = Toggle per-CPU view    c  = Toggle full command        │
│  H  = Toggle threads         V  = Forest/tree view           │
│  f  = Choose display fields  d  = Change refresh interval    │
│                                                               │
│  FILTERING:                                                   │
│  u  = Filter by user         o  = Add filter (e.g., %CPU>5) │
│  i  = Toggle idle processes  L  = Search for string          │
│                                                               │
│  ACTIONS:                                                     │
│  k  = Kill a process (enter PID)                             │
│  r  = Renice a process                                        │
│                                                               │
│  q  = Quit                   h  = Help                       │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Start top with options
top -u devops                # Filter by user
top -p 1234,5678             # Monitor specific PIDs
top -n 5                     # Exit after 5 refreshes
top -b -n 1 > top_snapshot.txt    # Batch mode (for scripts)
top -d 2                     # Refresh every 2 seconds
```

---

## 2.3 `htop` — Enhanced Process Monitor

```bash
# Install htop
sudo apt install htop     # Ubuntu
sudo dnf install htop     # RHEL

# Run htop
htop
htop -u devops            # Filter by user
htop -p 1234,5678         # Specific PIDs
```

```
┌──────────────────────────────────────────────────────────┐
│  htop vs top                                              │
├─────────────┬──────────────────┬─────────────────────────┤
│ Feature     │ top              │ htop                    │
├─────────────┼──────────────────┼─────────────────────────┤
│ Colors      │ No               │ Yes (color-coded)       │
│ Mouse       │ No               │ Yes (click to sort)     │
│ Scroll      │ Limited          │ Full scroll (h/v)       │
│ Tree view   │ V key            │ F5 (default)            │
│ Kill        │ k + PID          │ F9 + select signal      │
│ Search      │ L                │ F3 or /                 │
│ Filter      │ o                │ F4                      │
│ Setup       │ Not customizable │ F2 (full customization) │
│ Installed   │ Default          │ Needs installation      │
└─────────────┴──────────────────┴─────────────────────────┘
```

---

## 2.4 Other Monitoring Tools

```bash
# List open files by process
lsof -p 1234                 # Files opened by PID
lsof -u devops               # Files opened by user
lsof -i :80                  # What's using port 80
lsof -i :443                 # What's using port 443

# /proc filesystem for process details
cat /proc/1234/status        # Process status
cat /proc/1234/cmdline       # Command line
ls -la /proc/1234/fd/        # Open file descriptors
cat /proc/1234/environ       # Environment variables

# pidof — get PID by name
pidof nginx

# pgrep — find processes by pattern
pgrep nginx                  # PIDs
pgrep -a nginx               # PIDs with full command
pgrep -u devops              # By user
```

---

## 2.5 Real-World Scenario: Investigating High CPU

```bash
# 1. Quick overview
uptime
# 10:30:45 up 5 days,  load average: 4.50, 2.10, 1.05

# 2. Identify the culprit
ps aux --sort=-%cpu | head -5

# 3. Deep dive with top
top -p $(pgrep -d, java)     # Monitor Java processes

# 4. Check thread-level activity
ps -p $(pgrep java) -T -o pid,tid,%cpu,comm | sort -k3 -rn | head -10

# 5. Check open files/connections
lsof -p $(pgrep java) | wc -l
```

---

## Summary Table

| Command | Purpose | Best For |
|---------|---------|----------|
| `ps aux` | Process snapshot | Scripting, one-time checks |
| `top` | Real-time monitor | Basic monitoring (always available) |
| `htop` | Enhanced monitor | Interactive troubleshooting |
| `pstree` | Process hierarchy | Understanding parent-child relationships |
| `pgrep` | Find process by name | Quick PID lookup |
| `lsof` | Open files/ports | Network and file debugging |

---

## Quick Revision Questions

1. **What is the difference between `ps aux` and `ps -ef`?** Which columns differ?
2. **How do you sort processes by memory usage in `top`?** And by CPU?
3. **What do the CPU fields (`us`, `sy`, `wa`, `id`) in `top` mean?**
4. **How do you find which process is listening on port 80?**
5. **What advantages does `htop` have over `top`?**
6. **How would you create a non-interactive snapshot of current processes for logging?**

---

[← Previous: Process Lifecycle](01-process-lifecycle.md) | [Back to README](../README.md) | [Next: Process Control →](03-process-control.md)
