# Chapter 5: Finding Files

[← Previous: Text Processing](04-text-processing.md) | [Back to README](../README.md) | [Next: User Management →](../03-Users-And-Permissions/01-user-management.md)

---

## Overview

Finding files efficiently on a Linux system is a critical skill. Whether you're hunting for a misconfigured file, locating binaries, or cleaning up old logs, Linux provides powerful tools: `find` for deep, flexible searches; `locate` for instant filename lookups; and `which`/`whereis`/`type` for locating commands.

---

## 5.1 `find` — The Swiss Army Knife

```
┌───────────────────────────────────────────────────────────┐
│                 find SYNTAX                                │
│                                                            │
│  find [path] [conditions] [actions]                       │
│                                                            │
│  Example:                                                  │
│  find /var/log -name "*.log" -mtime +30 -delete           │
│        │         │             │          │                 │
│        │         │             │          └── Action        │
│        │         │             └── Modified > 30 days ago  │
│        │         └── Filename pattern                      │
│        └── Starting path                                   │
└───────────────────────────────────────────────────────────┘
```

### Search by Name

```bash
# Find by exact name
find / -name "nginx.conf"

# Find by name (case-insensitive)
find / -iname "readme.md"

# Find by wildcard pattern
find /etc -name "*.conf"
find /var/log -name "*.log"

# Find by path pattern
find / -path "*/nginx/*.conf"

# Exclude a directory
find / -name "*.log" -not -path "*/proc/*"
```

### Search by Type

```bash
# File types: f=file, d=directory, l=symlink, s=socket, b=block, c=char

find /etc -type f                      # Regular files only
find /opt -type d                      # Directories only
find /usr/bin -type l                  # Symbolic links
find /var/run -type s                  # Sockets (e.g., docker.sock)

# Find empty files and directories
find /tmp -empty
find /var/log -type f -empty           # Empty files
find /home -type d -empty              # Empty directories
```

### Search by Size

```bash
# Size units: c=bytes, k=KB, M=MB, G=GB

find / -size +100M                     # Files larger than 100MB
find /var/log -size +1G                # Files larger than 1GB
find /tmp -size -1k                    # Files smaller than 1KB
find / -size +100M -size -1G           # Between 100MB and 1GB

# DevOps: Find large files consuming disk space
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh
```

### Search by Time

```
┌───────────────────────────────────────────────────────────┐
│  TIME-BASED SEARCH                                        │
│                                                            │
│  -mtime  = Modification time (content changed)            │
│  -atime  = Access time (file read)                        │
│  -ctime  = Change time (metadata/permissions changed)     │
│                                                            │
│  -mmin   = Minutes instead of days                        │
│                                                            │
│  +N = more than N days ago                                │
│  -N = less than N days ago                                │
│   N = exactly N days ago                                  │
└───────────────────────────────────────────────────────────┘
```

```bash
# Files modified in last 24 hours
find /var/log -mtime -1

# Files modified more than 30 days ago
find /var/log -mtime +30

# Files modified in last 60 minutes
find /var/log -mmin -60

# Files NOT accessed in 90 days
find /home -atime +90

# Files changed today
find /etc -mtime 0

# Newer than a reference file
find /opt/app -newer /opt/app/deploy.timestamp
```

### Search by Permissions and Ownership

```bash
# Find by permissions
find / -perm 777                       # Exactly 777
find / -perm -644                      # At least 644
find / -perm /u+s                      # SUID bit set
find / -perm /g+s                      # SGID bit set

# Find by owner
find /home -user devops
find /opt -user root -group www-data

# Find files not owned by any user
find / -nouser
find / -nogroup

# Security audit: world-writable files
find / -type f -perm -o+w 2>/dev/null

# Security audit: SUID/SGID files
find / -type f \( -perm -4000 -o -perm -2000 \) 2>/dev/null
```

### Actions

```bash
# Execute a command on each result
find /var/log -name "*.log" -exec ls -lh {} \;
find /tmp -name "*.tmp" -exec rm {} \;

# Execute with confirmation
find /tmp -name "*.old" -ok rm {} \;

# More efficient: use + instead of \; (batch execution)
find /var/log -name "*.gz" -exec ls -lh {} +

# Delete found files
find /tmp -name "*.tmp" -mtime +7 -delete

# Print with custom format
find /etc -name "*.conf" -printf "%p %s bytes %T+\n"

# Pipe to xargs (fast for large result sets)
find /var/log -name "*.gz" -mtime +30 | xargs rm -f
find /opt/app -name "*.py" -print0 | xargs -0 grep "TODO"
# -print0 and -0 handle filenames with spaces
```

### Complex `find` Examples for DevOps

```bash
# Find and fix permission issues
find /var/www -type d -exec chmod 755 {} +
find /var/www -type f -exec chmod 644 {} +

# Find config files modified in last hour
find /etc -name "*.conf" -mmin -60 -ls

# Find large log files and show sizes
find /var/log -name "*.log" -size +50M -printf "%s %p\n" | sort -rn | head -10

# Cleanup: Delete old archives
find /backup -name "*.tar.gz" -mtime +90 -delete

# Find broken symlinks
find /opt -type l ! -exec test -e {} \; -print

# Find files with specific content
find /etc -name "*.conf" -exec grep -l "password" {} +

# Find recently deployed files
find /opt/app/current -newer /opt/app/deploy.timestamp -type f
```

---

## 5.2 `locate` — Fast Filename Search

```
┌───────────────────────────────────────────────────────────┐
│  locate vs find                                           │
│                                                            │
│  locate:                          find:                   │
│  ├── Searches a database          ├── Searches filesystem │
│  ├── Near-instant results         ├── Slower (real-time)  │
│  ├── May be out of date           ├── Always current      │
│  └── No content/permission search └── Full flexibility    │
│                                                            │
│  Database updated by: updatedb (runs daily via cron)      │
└───────────────────────────────────────────────────────────┘
```

```bash
# Install locate (if not present)
sudo apt install mlocate           # Ubuntu/Debian
sudo dnf install mlocate           # RHEL/CentOS

# Update the database
sudo updatedb

# Search for files
locate nginx.conf
locate "*.yaml"

# Case-insensitive search
locate -i readme

# Count matches
locate -c "*.log"

# Show only existing files
locate -e nginx.conf

# Limit results  
locate -l 10 "*.conf"

# Use regex
locate -r "nginx.*\.conf$"
```

---

## 5.3 `which`, `whereis`, `type` — Locating Commands

```bash
# which — Find the path of a command (searches $PATH)
which nginx
# /usr/sbin/nginx

which python3
# /usr/bin/python3

# Show all matches in PATH
which -a python

# whereis — Find binary, source, and man pages
whereis nginx
# nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx /usr/share/man/man8/nginx.8.gz

whereis -b nginx                  # Binary only
whereis -m nginx                  # Man pages only

# type — Show how a command would be interpreted
type ls
# ls is aliased to 'ls --color=auto'

type cd
# cd is a shell builtin

type grep
# grep is /usr/bin/grep

type -a echo                      # All locations
# echo is a shell builtin
# echo is /usr/bin/echo

# command -v — POSIX-compatible way (best for scripts)
command -v docker
# /usr/bin/docker

# Check if command exists (for scripts)
if command -v docker &>/dev/null; then
    echo "Docker is installed"
else
    echo "Docker is NOT installed"
fi
```

---

## 5.4 Real-World DevOps Scenario

### Scenario: Server Disk Cleanup Automation

```bash
#!/bin/bash
# disk-cleanup.sh — Automated disk cleanup script

set -euo pipefail

echo "=== Disk Cleanup Script ==="
echo "Current usage:"
df -h / | tail -1

echo ""
echo "=== Finding large files (>100MB) ==="
find / -type f -size +100M \
    -not -path "/proc/*" \
    -not -path "/sys/*" \
    -not -path "/dev/*" \
    -exec ls -lh {} \; 2>/dev/null | sort -k5 -rh | head -20

echo ""
echo "=== Old log files (>30 days) ==="
find /var/log -name "*.log" -mtime +30 -printf "%T+ %s %p\n" | sort -r

echo ""
echo "=== Compressed logs (>7 days) ==="
COMPRESSED=$(find /var/log -name "*.gz" -mtime +7 | wc -l)
echo "Found $COMPRESSED compressed log files older than 7 days"

echo ""
echo "=== Temp files (>3 days) ==="
TEMP_FILES=$(find /tmp -type f -mtime +3 | wc -l)
echo "Found $TEMP_FILES temp files older than 3 days"

echo ""
echo "=== Package cache ==="
if command -v apt &>/dev/null; then
    du -sh /var/cache/apt/archives/
elif command -v dnf &>/dev/null; then
    du -sh /var/cache/dnf/
fi

# Cleanup (uncomment to execute)
# find /var/log -name "*.gz" -mtime +7 -delete
# find /tmp -type f -mtime +3 -delete
# sudo apt clean  # or sudo dnf clean all

echo ""
echo "=== After cleanup ==="
df -h / | tail -1
```

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|---------|
| `find` is slow | Searching from `/` | Narrow the search path |
| `Permission denied` errors | Searching restricted dirs | Redirect stderr: `2>/dev/null` |
| `locate` returns stale results | Database not updated | Run `sudo updatedb` |
| `locate` command not found | Not installed | `apt install mlocate` |
| `find -exec` too slow | `;` runs cmd per file | Use `+` for batch execution |
| Filenames with spaces break | Unquoted variables | Use `-print0 \| xargs -0` |
| `which` shows alias, not binary | Shell aliases | Use `which --skip-alias` or `command -v` |

---

## Summary Table

| Command | Purpose | Speed | Searches |
|---------|---------|-------|----------|
| `find` | Flexible file search | Slow (real-time FS scan) | Filesystem directly |
| `locate` | Fast filename search | Very fast (database) | Pre-built database |
| `which` | Find command in PATH | Instant | `$PATH` directories |
| `whereis` | Find binary, source, man | Instant | Standard locations |
| `type` | Show command type | Instant | Shell internals + PATH |
| `command -v` | POSIX command check | Instant | Best for scripts |

---

## Quick Revision Questions

1. **What's the difference between `find` and `locate`?** When would you use each?
2. **Write a `find` command to delete all `.log.gz` files in `/var/log` older than 30 days.**
3. **How do you search for files by size range?** Find files between 10MB and 100MB.
4. **What's the difference between `-exec {} \;` and `-exec {} +`?** Why does it matter?
5. **How would you find all SUID files on a system for a security audit?**
6. **How do you check if a command is installed in a shell script?** What's the most portable way?

---

[← Previous: Text Processing](04-text-processing.md) | [Back to README](../README.md) | [Next: User Management →](../03-Users-And-Permissions/01-user-management.md)
