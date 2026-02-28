# Chapter 4: Backup & Recovery

[← Previous: Performance Tuning](03-performance-tuning.md) | [Back to README](../README.md) | [Next: Security Hardening →](05-security-hardening.md)

---

## Overview

Backups are the last line of defense against data loss from hardware failure, ransomware, human error, and disasters. This chapter covers backup strategies, tools (`tar`, `rsync`, `borgbackup`), automated backup scripts, and recovery procedures essential for DevOps infrastructure.

---

## 4.1 Backup Strategy: The 3-2-1 Rule

```
┌──────────────────────────────────────────────────────────────┐
│              THE 3-2-1 BACKUP RULE                            │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  3 copies of your data                                        │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │ Original │  │  Copy 1  │  │  Copy 2  │                   │
│  │  (live)  │  │ (local)  │  │ (offsite)│                   │
│  └──────────┘  └──────────┘  └──────────┘                   │
│                                                               │
│  2 different storage types                                    │
│  ┌──────────┐  ┌──────────┐                                  │
│  │   SSD    │  │ External │                                  │
│  │  (disk)  │  │  HDD/NAS │                                  │
│  └──────────┘  └──────────┘                                  │
│                                                               │
│  1 copy offsite                                               │
│  ┌──────────────────────┐                                    │
│  │ Cloud (S3, GCS, B2)  │                                    │
│  │ or Remote datacenter  │                                    │
│  └──────────────────────┘                                    │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.2 Backup Types

```
┌──────────────────────────────────────────────────────────────┐
│              BACKUP TYPES COMPARISON                          │
├──────────────┬────────────────────────┬──────────────────────┤
│ Type         │ What It Backs Up       │ Pros/Cons            │
├──────────────┼────────────────────────┼──────────────────────┤
│ Full         │ Everything, every time │ Simple recovery      │
│              │                        │ Slow & large         │
├──────────────┼────────────────────────┼──────────────────────┤
│ Incremental  │ Changes since LAST     │ Fast & small         │
│              │ backup (any type)      │ Recovery needs chain │
├──────────────┼────────────────────────┼──────────────────────┤
│ Differential │ Changes since last     │ Medium size          │
│              │ FULL backup            │ Simpler recovery     │
├──────────────┼────────────────────────┼──────────────────────┤
│ Snapshot     │ Point-in-time copy     │ Instant, great       │
│              │ (LVM, ZFS, cloud)      │ for quick rollback   │
└──────────────┴────────────────────────┴──────────────────────┘
```

```
┌──────────────────────────────────────────────────────────────┐
│         BACKUP SCHEDULE EXAMPLE                               │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  Sunday:    FULL backup (complete copy)                       │
│  Monday:    Incremental (changes since Sunday)                │
│  Tuesday:   Incremental (changes since Monday)                │
│  Wednesday: Incremental (changes since Tuesday)               │
│  Thursday:  Incremental (changes since Wednesday)             │
│  Friday:    Incremental (changes since Thursday)              │
│  Saturday:  Incremental (changes since Friday)                │
│                                                               │
│  To restore Wednesday:                                        │
│  Sunday (full) + Mon (inc) + Tue (inc) + Wed (inc)           │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.3 Backup with `tar`

```bash
# Create compressed archive
tar -czf backup-$(date +%Y%m%d).tar.gz /etc /home /var/www

# Create with exclusions
tar -czf backup.tar.gz \
    --exclude='*.log' \
    --exclude='node_modules' \
    --exclude='.git' \
    /opt/app

# List contents without extracting
tar -tzf backup.tar.gz

# Extract
tar -xzf backup.tar.gz                  # To current directory
tar -xzf backup.tar.gz -C /restore/     # To specific directory

# Extract specific file
tar -xzf backup.tar.gz etc/nginx/nginx.conf

# Incremental backup with tar
# First (full):
tar -czf full.tar.gz --listed-incremental=snapshot.snar /data
# Subsequent (incremental):
tar -czf inc-$(date +%Y%m%d).tar.gz --listed-incremental=snapshot.snar /data
```

---

## 4.4 Backup with `rsync`

```bash
# Local sync
rsync -avh /source/ /backup/            # Archive, verbose, human-readable

# Remote sync (over SSH)
rsync -avhz /data/ user@backup-server:/backups/server01/

# Key flags
rsync -avhz \
    --delete \               # Remove files in dest not in source
    --exclude='*.log' \      # Exclude patterns
    --exclude='cache/' \
    --progress \             # Show progress
    --bwlimit=10000 \        # Limit bandwidth (KB/s)
    /source/ user@remote:/dest/

# Dry run (see what would happen)
rsync -avhn --delete /source/ /dest/

# Backup with hardlinks (incremental, space-efficient)
rsync -avh --delete \
    --link-dest=/backups/latest \
    /data/ /backups/$(date +%Y%m%d)/
ln -sfn /backups/$(date +%Y%m%d) /backups/latest
```

```
┌──────────────────────────────────────────────────────────────┐
│         RSYNC KEY FLAGS                                       │
├──────────────┬───────────────────────────────────────────────┤
│ Flag         │ Description                                   │
├──────────────┼───────────────────────────────────────────────┤
│ -a           │ Archive (preserves permissions, timestamps)   │
│ -v           │ Verbose                                       │
│ -h           │ Human-readable sizes                          │
│ -z           │ Compress during transfer                      │
│ --delete     │ Delete extraneous files from dest             │
│ --exclude    │ Exclude files matching pattern                │
│ --progress   │ Show transfer progress                        │
│ --dry-run    │ Show what would happen without doing it       │
│ --link-dest  │ Hardlink to files in reference directory      │
│ --bwlimit    │ Limit bandwidth in KB/s                       │
│ -e "ssh -p"  │ Specify SSH port                              │
│ --partial    │ Keep partially transferred files              │
└──────────────┴───────────────────────────────────────────────┘
```

---

## 4.5 BorgBackup — Deduplicating Backup

```bash
# Install
sudo apt install borgbackup       # Ubuntu
sudo dnf install borgbackup       # RHEL (EPEL)

# Initialize repository
borg init --encryption=repokey /backups/borg-repo
# Or remote:
borg init --encryption=repokey ssh://user@backup-server/backups/borg-repo

# Create backup
borg create /backups/borg-repo::server-{now:%Y%m%d-%H%M} \
    /etc /home /var/www \
    --exclude '*.log' \
    --exclude 'node_modules' \
    --stats --progress

# List archives
borg list /backups/borg-repo

# Restore
borg extract /backups/borg-repo::server-20240115-0200

# Restore specific path
borg extract /backups/borg-repo::server-20240115-0200 home/user/documents

# Prune old backups
borg prune /backups/borg-repo \
    --keep-daily=7 \
    --keep-weekly=4 \
    --keep-monthly=6

# Check repository integrity
borg check /backups/borg-repo

# Info about archive
borg info /backups/borg-repo::server-20240115-0200
```

---

## 4.6 LVM Snapshots for Backup

```bash
# Create snapshot
sudo lvcreate -L 5G -s -n data_snap /dev/vg0/data

# Mount snapshot (read-only)
sudo mkdir /mnt/snap
sudo mount -o ro /dev/vg0/data_snap /mnt/snap

# Backup from snapshot (consistent point-in-time)
tar -czf backup.tar.gz /mnt/snap/

# Cleanup
sudo umount /mnt/snap
sudo lvremove /dev/vg0/data_snap
```

---

## 4.7 Database Backup Patterns

```bash
# MySQL / MariaDB
mysqldump -u root -p --all-databases > all_dbs_$(date +%F).sql
mysqldump -u root -p --single-transaction mydb > mydb_$(date +%F).sql

# PostgreSQL
pg_dump -U postgres mydb > mydb_$(date +%F).sql
pg_dumpall -U postgres > all_dbs_$(date +%F).sql

# Compressed
mysqldump -u root -p mydb | gzip > mydb_$(date +%F).sql.gz

# Restore
mysql -u root -p mydb < mydb_backup.sql
psql -U postgres mydb < mydb_backup.sql
gunzip < mydb_backup.sql.gz | mysql -u root -p mydb
```

---

## 4.8 Cloud Backup (AWS S3 Example)

```bash
# Install AWS CLI
sudo apt install awscli

# Configure
aws configure

# Sync to S3
aws s3 sync /backups/ s3://my-backup-bucket/server01/ \
    --storage-class STANDARD_IA \
    --exclude "*.tmp"

# Upload single file
aws s3 cp backup.tar.gz s3://my-backup-bucket/daily/

# Download from S3
aws s3 cp s3://my-backup-bucket/daily/backup.tar.gz .

# Lifecycle policy (auto-move to Glacier after 30 days)
# Configure via AWS console or CLI JSON
```

---

## 4.9 Real-World Scenario: Automated Backup Script

```bash
#!/usr/bin/env bash
# backup.sh — Automated daily backup with retention
# Run via cron: 0 2 * * * /opt/scripts/backup.sh

set -euo pipefail

# Configuration
BACKUP_DIR="/backups"
SOURCE_DIRS=("/etc" "/home" "/var/www" "/opt/app")
EXCLUDES=("*.log" "node_modules" ".git" "__pycache__" "*.tmp")
RETAIN_DAYS=30
DATE=$(date '+%Y%m%d_%H%M%S')
HOSTNAME=$(hostname -s)
BACKUP_FILE="${BACKUP_DIR}/${HOSTNAME}_${DATE}.tar.gz"
LOG="/var/log/backup.log"
REMOTE_HOST="backup-server"
REMOTE_DIR="/backups/${HOSTNAME}"

log() { echo "$(date '+%F %T') $*" | tee -a "$LOG"; }

# Cleanup on failure
cleanup() {
    local exit_code=$?
    if [ $exit_code -ne 0 ]; then
        log "BACKUP FAILED (exit code: $exit_code)"
        rm -f "$BACKUP_FILE"
    fi
}
trap cleanup EXIT

log "=== Backup Started ==="

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Build exclude flags
EXCLUDE_FLAGS=""
for pattern in "${EXCLUDES[@]}"; do
    EXCLUDE_FLAGS+=" --exclude=${pattern}"
done

# Create backup
log "Creating archive: $BACKUP_FILE"
tar -czf "$BACKUP_FILE" $EXCLUDE_FLAGS "${SOURCE_DIRS[@]}" 2>/dev/null || true

# Verify backup
SIZE=$(du -sh "$BACKUP_FILE" | awk '{print $1}')
log "Backup size: $SIZE"
tar -tzf "$BACKUP_FILE" > /dev/null
log "Backup verification: OK"

# Database backup
if command -v mysqldump &>/dev/null; then
    DB_FILE="${BACKUP_DIR}/${HOSTNAME}_db_${DATE}.sql.gz"
    log "Backing up MySQL databases..."
    mysqldump -u root --all-databases 2>/dev/null | gzip > "$DB_FILE"
    log "Database backup: $(du -sh "$DB_FILE" | awk '{print $1}')"
fi

# Copy to remote server
if ssh -o ConnectTimeout=5 "$REMOTE_HOST" true 2>/dev/null; then
    log "Syncing to remote: $REMOTE_HOST"
    rsync -az "$BACKUP_FILE" "${REMOTE_HOST}:${REMOTE_DIR}/"
    log "Remote sync complete"
else
    log "WARNING: Remote server unreachable, skipping offsite copy"
fi

# Cleanup old backups
log "Removing backups older than ${RETAIN_DAYS} days..."
DELETED=$(find "$BACKUP_DIR" -name "${HOSTNAME}_*" -mtime +"$RETAIN_DAYS" -delete -print | wc -l)
log "Removed $DELETED old backup files"

log "=== Backup Complete ==="
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Backup too large | Includes unnecessary files | Add `--exclude` patterns for logs, caches, temp files |
| `rsync` connection drops | Network instability | Use `--partial --timeout=60` and run via cron for retry |
| Restore fails | Corrupted archive | Always verify: `tar -tzf backup.tar.gz > /dev/null` |
| Backup takes too long | Full backup every time | Switch to incremental (rsync `--link-dest` or borgbackup) |
| No space for backup | Target disk full | Set retention policy, compress, use cloud tier |
| Database inconsistency | Backup during writes | Use `--single-transaction` (MySQL) or pg_dump (consistent) |

---

## Summary Table

| Tool | Type | Best For |
|------|------|----------|
| `tar` | Archive + compress | Simple full backups |
| `rsync` | Incremental sync | Efficient sync to remote |
| `borgbackup` | Deduplicating | Large datasets, space efficiency |
| `mysqldump` | Database dump | MySQL/MariaDB backups |
| `pg_dump` | Database dump | PostgreSQL backups |
| LVM snapshot | Point-in-time | Consistent backup of active volumes |
| `aws s3 sync` | Cloud backup | Offsite backup to S3 |
| `cron` | Scheduling | Automating backup runs |

---

## Quick Revision Questions

1. **What is the 3-2-1 backup rule?**
2. **What is the difference between incremental and differential backups?**
3. **How does `rsync --link-dest` save disk space for incremental backups?**
4. **Write a command to backup `/data` excluding logs, compress it, and send to a remote server.**
5. **How do you create a consistent database backup while the database is running?**
6. **What should you always do after creating a backup to ensure it's valid?**

---

[← Previous: Performance Tuning](03-performance-tuning.md) | [Back to README](../README.md) | [Next: Security Hardening →](05-security-hardening.md)
