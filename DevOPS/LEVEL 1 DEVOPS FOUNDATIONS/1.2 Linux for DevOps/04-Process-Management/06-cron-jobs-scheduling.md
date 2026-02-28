# Chapter 6: Cron Jobs & Scheduling

[← Previous: Systemd Service Management](05-systemd-service-management.md) | [Back to README](../README.md) | [Next: Networking Fundamentals →](../05-Networking/01-networking-fundamentals.md)

---

## Overview

Task scheduling is essential for automating backups, log rotation, health checks, certificate renewal, and other recurring operations. Linux offers `cron` (traditional), `at` (one-time), and `systemd timers` (modern alternative).

---

## 6.1 Cron Basics

```
┌────────────────────────────────────────────────────────────────┐
│                     CRON EXPRESSION FORMAT                      │
│                                                                  │
│  ┌──────────── minute (0 - 59)                                  │
│  │ ┌────────── hour (0 - 23)                                    │
│  │ │ ┌──────── day of month (1 - 31)                            │
│  │ │ │ ┌────── month (1 - 12 or JAN-DEC)                       │
│  │ │ │ │ ┌──── day of week (0 - 7, 0 & 7 = Sun, or SUN-SAT)   │
│  │ │ │ │ │                                                       │
│  * * * * *  command_to_execute                                   │
│                                                                  │
│  SPECIAL CHARACTERS:                                             │
│  *   = every value           */5 = every 5th                    │
│  ,   = list (1,3,5)         -   = range (1-5)                  │
│  /   = step (*/10)                                               │
│                                                                  │
│  EXAMPLES:                                                       │
│  0 * * * *       = Every hour at :00                            │
│  */15 * * * *    = Every 15 minutes                             │
│  0 2 * * *       = Daily at 2:00 AM                             │
│  0 0 * * 0       = Every Sunday at midnight                     │
│  30 3 1 * *      = 1st of every month at 3:30 AM               │
│  0 9 * * 1-5     = Weekdays at 9:00 AM                         │
│  0 0 1,15 * *    = 1st and 15th of month at midnight           │
└────────────────────────────────────────────────────────────────┘
```

---

## 6.2 Crontab Management

```bash
# Edit your crontab
crontab -e

# List your crontab
crontab -l

# Edit crontab for another user (root)
sudo crontab -e -u devops

# List another user's crontab
sudo crontab -l -u devops

# Remove your crontab (DANGEROUS!)
crontab -r                      # Deletes entire crontab
crontab -ri                     # Interactive confirm

# System-wide cron files
# /etc/crontab              ← System crontab (includes USER field)
# /etc/cron.d/              ← Drop-in files
# /etc/cron.daily/          ← Daily scripts
# /etc/cron.hourly/         ← Hourly scripts
# /etc/cron.weekly/         ← Weekly scripts
# /etc/cron.monthly/        ← Monthly scripts
```

```
┌──────────────────────────────────────────────────────────────┐
│              CRON FILE LOCATIONS                              │
│                                                                │
│  User crontabs:                                               │
│  /var/spool/cron/crontabs/<user>      (Debian/Ubuntu)        │
│  /var/spool/cron/<user>               (RHEL/CentOS)          │
│                                                                │
│  System crontab:                                              │
│  /etc/crontab         ← Has extra USER field                  │
│  # m  h dom mon dow USER  command                             │
│  17 * * * *  root    cd / && run-parts --report /etc/cron.hourly│
│                                                                │
│  Drop-in directory:                                           │
│  /etc/cron.d/         ← Package cron jobs go here            │
│                                                                │
│  Periodic directories (managed by anacron):                   │
│  /etc/cron.hourly/                                            │
│  /etc/cron.daily/                                             │
│  /etc/cron.weekly/                                            │
│  /etc/cron.monthly/                                           │
│                                                                │
│  Access control:                                              │
│  /etc/cron.allow      ← If exists, only listed users can use │
│  /etc/cron.deny       ← Listed users cannot use cron         │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.3 Special Cron Strings

```bash
@reboot    /opt/scripts/on-boot.sh        # Run at startup
@hourly    /opt/scripts/health-check.sh    # Every hour (0 * * * *)
@daily     /opt/scripts/backup.sh          # Daily (0 0 * * *)
@weekly    /opt/scripts/report.sh          # Weekly (0 0 * * 0)
@monthly   /opt/scripts/audit.sh           # Monthly (0 0 1 * *)
@yearly    /opt/scripts/cert-renew.sh      # Yearly (0 0 1 1 *)
@annually  # Same as @yearly
```

---

## 6.4 Cron Best Practices

```bash
# 1. ALWAYS redirect output
0 2 * * * /opt/backup.sh > /var/log/backup.log 2>&1

# 2. Use absolute paths
0 3 * * * /usr/bin/python3 /opt/scripts/cleanup.py

# 3. Set PATH and environment
PATH=/usr/local/bin:/usr/bin:/bin
SHELL=/bin/bash
MAILTO=admin@company.com

# 4. Use lock files to prevent overlap
*/5 * * * * /usr/bin/flock -n /tmp/job.lock /opt/scripts/process.sh

# 5. Log with timestamps
0 * * * * /opt/scripts/check.sh 2>&1 | while read line; do echo "$(date '+\%F \%T') $line"; done >> /var/log/check.log

# 6. Comment your cron jobs
# Backup database every day at 2 AM
0 2 * * * /opt/scripts/db-backup.sh > /var/log/db-backup.log 2>&1
```

---

## 6.5 `at` — One-Time Scheduled Tasks

```bash
# Install
sudo apt install at        # Ubuntu
sudo dnf install at        # RHEL

# Enable the service
sudo systemctl enable --now atd

# Schedule a one-time job
at 2:00 AM
> /opt/scripts/maintenance.sh
> 
# (Ctrl+D to finish)

# Other time specs
at now + 30 minutes
at now + 2 hours
at midnight
at 10:00 PM tomorrow
at noon Dec 25

# List pending jobs
atq
at -l

# View job content
at -c 5                    # Show job #5

# Remove a job
atrm 5
at -d 5
```

---

## 6.6 Systemd Timers (Modern Alternative)

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Daily Backup Timer

[Timer]
OnCalendar=*-*-* 02:00:00
# Alternatives:
# OnCalendar=daily
# OnCalendar=Mon..Fri 09:00:00
# OnBootSec=5min
# OnUnitActiveSec=1h
Persistent=true
RandomizedDelaySec=300

[Install]
WantedBy=timers.target
```

```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Daily Backup Job

[Service]
Type=oneshot
User=backup
ExecStart=/opt/scripts/backup.sh
StandardOutput=journal
```

```bash
# Enable the timer
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer

# Check timers
systemctl list-timers --all
systemctl status backup.timer

# Run manually
sudo systemctl start backup.service
```

```
┌──────────────────────────────────────────────────────────────┐
│         CRON vs SYSTEMD TIMERS                                │
├────────────────────┬──────────────────┬──────────────────────┤
│ Feature            │ Cron             │ Systemd Timer        │
├────────────────────┼──────────────────┼──────────────────────┤
│ Logging            │ Manual redirect  │ journalctl built-in  │
│ Dependencies       │ None             │ After=, Requires=    │
│ Missed runs        │ Lost (anacron?)  │ Persistent=true      │
│ Random delay       │ Manual           │ RandomizedDelaySec   │
│ Resource limits    │ None             │ MemoryMax, CPUQuota  │
│ Status monitoring  │ Check manually   │ systemctl status     │
│ Debug              │ Difficult        │ journalctl -u timer  │
│ Setup complexity   │ Simple           │ 2 files needed       │
│ Availability       │ Universal        │ systemd only         │
└────────────────────┴──────────────────┴──────────────────────┘
```

---

## 6.7 Real-World DevOps Cron Jobs

```bash
# Database backup (daily at 2 AM)
0 2 * * * /usr/bin/pg_dump -U postgres mydb | gzip > /backup/db_$(date +\%F).sql.gz 2>> /var/log/db-backup.log

# Docker image cleanup (weekly)
0 3 * * 0 /usr/bin/docker system prune -af --filter "until=168h" >> /var/log/docker-cleanup.log 2>&1

# SSL certificate renewal (twice daily)
0 0,12 * * * /usr/bin/certbot renew --quiet --deploy-hook "systemctl reload nginx"

# Log rotation check
0 0 * * * /usr/sbin/logrotate /etc/logrotate.conf

# Health check every 5 minutes
*/5 * * * * /opt/scripts/health-check.sh || /opt/scripts/alert.sh

# Sync files to S3 (every hour)
0 * * * * /usr/local/bin/aws s3 sync /var/data s3://mybucket/data/ >> /var/log/s3sync.log 2>&1

# Disk space alert
*/30 * * * * /opt/scripts/disk-alert.sh
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Cron job doesn't run | Check `crontab -l`, syslog | Verify cron syntax, PATH, permissions |
| Command works manually but not in cron | Environment differs | Use absolute paths, set PATH in crontab |
| No output/errors from cron | Output discarded | Redirect: `>> /var/log/job.log 2>&1` |
| Job runs twice | Duplicate entries | Check `crontab -l` and `/etc/cron.d/` |
| Permission denied | Wrong user or permissions | Check script permissions (`chmod +x`) |
| Cron emails filling mailbox | MAILTO not set | Add `MAILTO=""` or redirect output |

---

## Summary Table

| Tool | Purpose | Best For |
|------|---------|----------|
| `crontab -e` | Edit recurring jobs | Simple scheduled tasks |
| `cron.d/` | System cron drop-ins | Package-managed jobs |
| `cron.daily/` | Daily scripts | Simple daily tasks |
| `at` | One-time future jobs | Maintenance windows |
| `systemd timer` | Modern scheduling | Complex dependencies, logging |
| `flock` | Prevent overlap | Long-running cron jobs |

---

## Quick Revision Questions

1. **What does `*/15 8-17 * * 1-5` mean in cron syntax?**
2. **Where are user crontabs stored on Ubuntu vs RHEL?**
3. **How do you prevent a cron job from running multiple overlapping instances?**
4. **What are two advantages of systemd timers over cron?**
5. **How do you schedule a one-time job to run at 3 AM tomorrow?**
6. **A cron job works when run manually but fails when run by cron. What is the most common cause?**

---

[← Previous: Systemd Service Management](05-systemd-service-management.md) | [Back to README](../README.md) | [Next: Networking Fundamentals →](../05-Networking/01-networking-fundamentals.md)
