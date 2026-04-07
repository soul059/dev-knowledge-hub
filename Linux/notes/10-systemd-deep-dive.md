# Systemd Deep Dive

A comprehensive guide to systemd, the modern init system and service manager for Linux.

## Table of Contents

1. [Systemd Overview](#1-systemd-overview)
2. [Unit Files](#2-unit-files)
3. [Service Management](#3-service-management)
4. [Creating Custom Services](#4-creating-custom-services)
5. [Timers (Cron Alternative)](#5-timers-cron-alternative)
6. [Targets and Boot Process](#6-targets-and-boot-process)
7. [Journald Logging](#7-journald-logging)
8. [Resource Control with cgroups](#8-resource-control-with-cgroups)
9. [Socket Activation](#9-socket-activation)
10. [Troubleshooting](#10-troubleshooting)

---

## 1. Systemd Overview

### What is Systemd?

Systemd is a system and service manager that provides:
- Parallel service startup for faster boot
- On-demand service activation
- Process tracking using cgroups
- Snapshot and restore of system state
- Unified logging with journald

### Core Components

```
systemd          Main system manager
systemctl        Control systemd and services
journalctl       Query systemd journal
hostnamectl      Manage hostname
timedatectl      Manage time and date
localectl        Manage locale settings
loginctl         Manage user sessions
networkctl       Manage network (systemd-networkd)
```

### Basic Commands

```bash
# System state
systemctl status
systemctl --failed
systemctl list-units
systemctl list-unit-files

# Power management
systemctl reboot
systemctl poweroff
systemctl suspend
systemctl hibernate
```

---

## 2. Unit Files

### Unit Types

| Type | Extension | Purpose |
|------|-----------|---------|
| Service | .service | System services and daemons |
| Socket | .socket | IPC and network sockets |
| Target | .target | Group of units (runlevels) |
| Timer | .timer | Timer-based activation |
| Mount | .mount | File system mount points |
| Automount | .automount | Auto-mount points |
| Path | .path | File system path monitoring |
| Slice | .slice | Resource management |
| Scope | .scope | Externally created processes |

### Unit File Locations

```bash
/etc/systemd/system/          # Local configuration (highest priority)
/run/systemd/system/          # Runtime units
/usr/lib/systemd/system/      # Package-installed units
/lib/systemd/system/          # Package-installed units (Debian)

# User units
~/.config/systemd/user/       # User-specific units
/etc/systemd/user/            # System-wide user units
```

### Unit File Structure

```ini
[Unit]
Description=My Application Service
Documentation=https://example.com/docs
After=network.target postgresql.service
Requires=postgresql.service
Wants=redis.service

[Service]
Type=simple
User=appuser
Group=appgroup
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/bin/start
ExecStop=/opt/myapp/bin/stop
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Unit Dependencies

```ini
# Hard dependencies (unit fails if dependency fails)
Requires=dependency.service

# Soft dependencies (unit continues if dependency fails)
Wants=dependency.service

# Ordering (doesn't imply dependency)
After=dependency.service    # Start after
Before=dependency.service   # Start before

# Conflicts (units cannot run together)
Conflicts=other.service

# Bind lifecycle
BindsTo=dependency.service  # Stop if dependency stops
PartOf=dependency.service   # Stop/restart together
```

---

## 3. Service Management

### Service Control

```bash
# Start/stop/restart
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx        # Reload configuration
sudo systemctl reload-or-restart nginx

# Enable/disable at boot
sudo systemctl enable nginx
sudo systemctl disable nginx
sudo systemctl enable --now nginx  # Enable and start

# Check status
systemctl status nginx
systemctl is-active nginx
systemctl is-enabled nginx
systemctl is-failed nginx

# Mask/unmask (prevent starting)
sudo systemctl mask nginx
sudo systemctl unmask nginx
```

### Listing Services

```bash
# List all units
systemctl list-units

# List services only
systemctl list-units --type=service

# List all installed unit files
systemctl list-unit-files

# List failed units
systemctl list-units --failed

# List dependencies
systemctl list-dependencies nginx
systemctl list-dependencies --reverse nginx
```

### Service Properties

```bash
# Show all properties
systemctl show nginx

# Show specific property
systemctl show nginx -p MainPID
systemctl show nginx -p ExecStart

# Show environment
systemctl show nginx -p Environment
```

---

## 4. Creating Custom Services

### Simple Service

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=www-data
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

### Service Types

```ini
# Type=simple (default) - ExecStart is the main process
Type=simple
ExecStart=/usr/bin/myapp

# Type=forking - Traditional daemon that forks
Type=forking
PIDFile=/var/run/myapp.pid
ExecStart=/usr/bin/myapp -d

# Type=oneshot - Short-lived scripts
Type=oneshot
RemainAfterExit=yes
ExecStart=/usr/local/bin/setup-script.sh

# Type=notify - Signals readiness via sd_notify
Type=notify
ExecStart=/usr/bin/myapp --notify

# Type=dbus - Acquires D-Bus name
Type=dbus
BusName=org.example.MyApp
ExecStart=/usr/bin/myapp

# Type=idle - Delayed until all jobs finish
Type=idle
ExecStart=/usr/bin/myapp
```

### Environment Variables

```ini
[Service]
# Direct assignment
Environment="VAR1=value1" "VAR2=value2"

# From file
EnvironmentFile=/etc/myapp/env
EnvironmentFile=-/etc/myapp/optional.env  # - means optional

# Pass to process
PassEnvironment=HOME USER
```

### Pre/Post Execution

```ini
[Service]
ExecStartPre=/usr/bin/myapp-check
ExecStartPre=-/usr/bin/optional-check  # - means failure is OK
ExecStart=/usr/bin/myapp
ExecStartPost=/usr/bin/myapp-notify
ExecStop=/usr/bin/myapp-stop
ExecStopPost=/usr/bin/myapp-cleanup
ExecReload=/bin/kill -HUP $MAINPID
```

### Restart Policies

```ini
[Service]
# Restart conditions
Restart=no              # Never restart
Restart=always          # Always restart
Restart=on-success      # Restart if exit code 0
Restart=on-failure      # Restart if non-zero exit
Restart=on-abnormal     # Restart on signal/timeout/watchdog
Restart=on-abort        # Restart on uncaught signal
Restart=on-watchdog     # Restart on watchdog timeout

# Restart timing
RestartSec=5            # Wait 5 seconds before restart

# Limit restarts
StartLimitIntervalSec=60
StartLimitBurst=5       # Max 5 restarts in 60 seconds
```

### Hardened Service Example

```ini
# /etc/systemd/system/secure-app.service
[Unit]
Description=Secure Application
After=network.target

[Service]
Type=simple
User=appuser
Group=appgroup
ExecStart=/opt/app/bin/server

# Security hardening
NoNewPrivileges=yes
PrivateTmp=yes
PrivateDevices=yes
ProtectSystem=strict
ProtectHome=yes
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes
ReadWritePaths=/var/lib/app /var/log/app
ReadOnlyPaths=/etc/app

# Capabilities
CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE

# Network restrictions
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX

# System call filtering
SystemCallFilter=@system-service
SystemCallFilter=~@privileged @resources

# Memory protection
MemoryDenyWriteExecute=yes
LockPersonality=yes

[Install]
WantedBy=multi-user.target
```

### Reload After Changes

```bash
# Reload systemd configuration
sudo systemctl daemon-reload

# Then restart service
sudo systemctl restart myapp
```

---

## 5. Timers (Cron Alternative)

### Timer Basics

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily backup timer

[Timer]
OnCalendar=daily
Persistent=true
RandomizedDelaySec=1h

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Backup service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

### Timer Types

```ini
# Monotonic timers (relative to boot/activation)
OnBootSec=5min          # 5 minutes after boot
OnStartupSec=10min      # 5 minutes after systemd start
OnActiveSec=1h          # 1 hour after timer activation
OnUnitActiveSec=1d      # 1 day after service last ran
OnUnitInactiveSec=1h    # 1 hour after service stopped

# Calendar timers (absolute time)
OnCalendar=daily                    # Every day at midnight
OnCalendar=weekly                   # Monday at midnight
OnCalendar=monthly                  # First day of month
OnCalendar=*-*-* 04:00:00          # Every day at 4 AM
OnCalendar=Mon-Fri *-*-* 09:00:00  # Weekdays at 9 AM
OnCalendar=*:0/15                   # Every 15 minutes
OnCalendar=Sat *-*-1..7 18:00:00   # First Saturday of month
```

### Calendar Expressions

```bash
# Format: DayOfWeek Year-Month-Day Hour:Minute:Second

# Examples
hourly              → *-*-* *:00:00
daily               → *-*-* 00:00:00
weekly              → Mon *-*-* 00:00:00
monthly             → *-*-01 00:00:00
yearly              → *-01-01 00:00:00

# Custom
Mon,Fri *-*-* 10:00      # Monday and Friday at 10 AM
*-*-* 9,17:00            # 9 AM and 5 PM daily
*-*-* *:0/10             # Every 10 minutes
Sat,Sun *-*-* *:*:00     # Every minute on weekends

# Test calendar expression
systemd-analyze calendar "Mon-Fri *-*-* 09:00"
```

### Timer Options

```ini
[Timer]
# Accuracy (default 1 minute)
AccuracySec=1s

# Catch up on missed runs
Persistent=true

# Random delay to spread load
RandomizedDelaySec=5min

# Wake system from suspend
WakeSystem=true
```

### Managing Timers

```bash
# List timers
systemctl list-timers
systemctl list-timers --all

# Enable/start timer
sudo systemctl enable --now backup.timer

# Check timer status
systemctl status backup.timer

# Manually trigger associated service
sudo systemctl start backup.service
```

---

## 6. Targets and Boot Process

### Understanding Targets

Targets are groups of units, similar to runlevels:

| Target | Purpose | Old Runlevel |
|--------|---------|--------------|
| poweroff.target | System halt | 0 |
| rescue.target | Single user mode | 1 |
| multi-user.target | Multi-user, no GUI | 3 |
| graphical.target | Multi-user with GUI | 5 |
| reboot.target | System reboot | 6 |

### Managing Targets

```bash
# Check current target
systemctl get-default

# Change default target
sudo systemctl set-default multi-user.target
sudo systemctl set-default graphical.target

# Switch target immediately
sudo systemctl isolate multi-user.target
sudo systemctl isolate rescue.target

# Emergency mode
sudo systemctl isolate emergency.target
```

### Boot Process

```
1. BIOS/UEFI
    ↓
2. Bootloader (GRUB)
    ↓
3. Kernel initialization
    ↓
4. systemd (PID 1)
    ↓
5. default.target
    ↓
6. User session
```

### Analyzing Boot

```bash
# Boot time analysis
systemd-analyze

# Blame (slow units)
systemd-analyze blame

# Critical chain
systemd-analyze critical-chain

# Plot boot sequence
systemd-analyze plot > boot.svg

# Verify unit files
systemd-analyze verify myservice.service
```

---

## 7. Journald Logging

### Basic Journal Queries

```bash
# View all logs
journalctl

# Follow logs (like tail -f)
journalctl -f

# Last N lines
journalctl -n 50

# Reverse order (newest first)
journalctl -r

# No pager
journalctl --no-pager
```

### Filtering Logs

```bash
# By unit
journalctl -u nginx
journalctl -u nginx -u php-fpm

# By priority
journalctl -p err          # Errors and above
journalctl -p warning      # Warnings and above
# Priorities: emerg, alert, crit, err, warning, notice, info, debug

# By time
journalctl --since "2024-01-01"
journalctl --since "1 hour ago"
journalctl --since "yesterday" --until "today"
journalctl --since "09:00" --until "10:00"

# By boot
journalctl -b              # Current boot
journalctl -b -1           # Previous boot
journalctl --list-boots    # List all boots

# By PID/UID
journalctl _PID=1234
journalctl _UID=1000

# Kernel messages
journalctl -k

# By binary
journalctl /usr/bin/nginx
```

### Output Formats

```bash
# JSON output
journalctl -o json
journalctl -o json-pretty

# Short formats
journalctl -o short        # Default
journalctl -o short-iso    # ISO timestamps
journalctl -o verbose      # All fields

# Export
journalctl -o export > journal.export
```

### Journal Configuration

```bash
# /etc/systemd/journald.conf

[Journal]
Storage=persistent         # auto, volatile, persistent, none
Compress=yes
SystemMaxUse=500M          # Max disk usage
SystemKeepFree=1G          # Keep this much free
MaxRetentionSec=1month     # Max retention time
MaxFileSec=1week           # Max time per file
ForwardToSyslog=no
```

### Journal Maintenance

```bash
# Check disk usage
journalctl --disk-usage

# Rotate journals
sudo journalctl --rotate

# Clean old journals
sudo journalctl --vacuum-time=7d    # Keep 7 days
sudo journalctl --vacuum-size=500M  # Keep 500MB
sudo journalctl --vacuum-files=5    # Keep 5 files
```

---

## 8. Resource Control with cgroups

### Overview

Systemd uses cgroups (Control Groups) to manage resources:

```bash
# View cgroup hierarchy
systemd-cgls

# View resource usage
systemd-cgtop
```

### Resource Limits in Services

```ini
[Service]
# CPU limits
CPUQuota=50%               # Max 50% of one CPU
CPUWeight=100              # Relative weight (1-10000, default 100)
CPUShares=512              # Legacy, prefer CPUWeight

# Memory limits
MemoryMax=512M             # Hard limit
MemoryHigh=400M            # Throttle above this
MemoryLow=100M             # Protection from reclaim
MemorySwapMax=0            # Disable swap

# IO limits
IOWeight=100               # Relative weight (1-10000)
IODeviceWeight=/dev/sda 200
IOReadBandwidthMax=/dev/sda 10M
IOWriteBandwidthMax=/dev/sda 5M
IOReadIOPSMax=/dev/sda 1000
IOWriteIOPSMax=/dev/sda 500

# Process limits
TasksMax=50                # Max processes
LimitNOFILE=65536          # Max open files
LimitNPROC=4096            # Max processes per user
```

### Slices

```bash
# Default slices
system.slice    # System services
user.slice      # User sessions
machine.slice   # VMs and containers

# View slice hierarchy
systemctl status system.slice

# Create custom slice
# /etc/systemd/system/myapp.slice
[Unit]
Description=My Application Slice

[Slice]
MemoryMax=2G
CPUQuota=100%
```

```ini
# Assign service to slice
[Service]
Slice=myapp.slice
```

### Runtime Resource Control

```bash
# Set property at runtime
sudo systemctl set-property nginx.service MemoryMax=1G

# Make persistent
sudo systemctl set-property nginx.service MemoryMax=1G --runtime=false
```

---

## 9. Socket Activation

### Socket-Activated Service

```ini
# /etc/systemd/system/myapp.socket
[Unit]
Description=My App Socket

[Socket]
ListenStream=/run/myapp.sock
# Or TCP: ListenStream=8080

[Install]
WantedBy=sockets.target
```

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My App Service
Requires=myapp.socket

[Service]
Type=simple
ExecStart=/usr/bin/myapp
StandardInput=socket
```

### Socket Options

```ini
[Socket]
# Unix socket
ListenStream=/run/myapp.sock
SocketMode=0660
SocketUser=myapp
SocketGroup=myapp

# TCP socket
ListenStream=8080
BindIPv6Only=both

# UDP socket
ListenDatagram=5353

# Accept mode (for inetd-style)
Accept=yes
MaxConnections=64

# Backlog
Backlog=128
```

### Enable Socket

```bash
sudo systemctl enable --now myapp.socket

# Service starts on first connection
curl http://localhost:8080
```

---

## 10. Troubleshooting

### Common Issues

```bash
# Check failed units
systemctl --failed

# View service logs
journalctl -u myservice -xe

# Check unit file syntax
systemd-analyze verify /etc/systemd/system/myservice.service

# Debug service startup
SYSTEMD_LOG_LEVEL=debug systemctl start myservice
```

### Service Won't Start

```bash
# 1. Check status
systemctl status myservice

# 2. Check logs
journalctl -u myservice -n 100

# 3. Verify unit file
systemd-analyze verify myservice.service

# 4. Check dependencies
systemctl list-dependencies myservice

# 5. Try manual start
/path/to/ExecStart  # Run command manually
```

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| "Failed to start" | ExecStart failed | Check binary path, permissions |
| "Dependency failed" | Required unit failed | Check/fix dependencies |
| "Timeout" | Service didn't signal ready | Increase TimeoutStartSec or fix Type |
| "code=exited, status=217" | User doesn't exist | Create user or fix User= |
| "code=exited, status=203" | Exec format error | Check shebang, permissions |

### Reset Failed State

```bash
sudo systemctl reset-failed
sudo systemctl reset-failed myservice
```

### Debug Boot Problems

```bash
# Boot to rescue mode
# Add to kernel command line: systemd.unit=rescue.target

# Emergency mode (minimal)
# Add: systemd.unit=emergency.target

# Debug shell
# Add: systemd.debug-shell=1
# Access via Ctrl+Alt+F9
```

---

## Quick Reference

### Essential Commands

```bash
# Service control
systemctl start|stop|restart|reload SERVICE
systemctl enable|disable SERVICE
systemctl status SERVICE

# System control
systemctl poweroff|reboot|suspend

# Unit management
systemctl daemon-reload
systemctl list-units
systemctl list-unit-files

# Logs
journalctl -u SERVICE
journalctl -f
journalctl -b

# Analysis
systemd-analyze
systemd-analyze blame
systemd-analyze critical-chain
```

### Unit File Template

```ini
[Unit]
Description=
After=
Requires=
Wants=

[Service]
Type=simple
User=
Group=
WorkingDirectory=
ExecStart=
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

---

*Last Updated: April 2026*
