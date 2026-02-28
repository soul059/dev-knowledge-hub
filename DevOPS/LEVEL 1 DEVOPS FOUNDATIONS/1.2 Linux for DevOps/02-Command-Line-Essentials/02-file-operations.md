# Chapter 2: File Operations

[← Previous: Navigation Commands](01-navigation-commands.md) | [Back to README](../README.md) | [Next: Text Viewing →](03-text-viewing.md)

---

## Overview

File operations — creating, copying, moving, renaming, and deleting files and directories — are core tasks you'll perform constantly as a DevOps engineer. Whether you're managing configuration files, deploying applications, or organizing backups, mastery of these commands is non-negotiable.

---

## 2.1 Creating Files and Directories

```bash
# ═══════════════════════════════════════════
# CREATING FILES
# ═══════════════════════════════════════════

# Create an empty file (or update timestamp)
touch newfile.txt
touch file1.txt file2.txt file3.txt       # Multiple files
touch /var/log/myapp/app.log              # Full path

# Create file with content
echo "Hello World" > hello.txt             # Write (overwrite)
echo "Another line" >> hello.txt           # Append
printf "Line 1\nLine 2\n" > file.txt       # With formatting

# Create file with heredoc
cat <<EOF > config.yaml
server:
  host: 0.0.0.0
  port: 8080
  environment: production
EOF

# ═══════════════════════════════════════════
# CREATING DIRECTORIES
# ═══════════════════════════════════════════

# Create a directory
mkdir mydir

# Create nested directories (parents included)
mkdir -p /opt/myapp/releases/v1.0/config

# Create multiple directories
mkdir -p {src,bin,docs,tests,config}

# Create with specific permissions
mkdir -m 750 private_dir

# Create project structure in one command
mkdir -p project/{src/{main,test},docs,config,scripts}
```

---

## 2.2 Copying Files and Directories

```
┌────────────────────────────────────────────────────────────┐
│                    cp (copy)                                │
│                                                             │
│  Source ─────────copy──────────→ Destination                │
│                                                             │
│  Original file remains unchanged                           │
└────────────────────────────────────────────────────────────┘
```

```bash
# Copy a file
cp source.txt destination.txt
cp config.yaml config.yaml.bak          # Create backup

# Copy to a directory
cp file.txt /tmp/
cp file.txt /tmp/renamed.txt

# Copy multiple files to a directory
cp file1.txt file2.txt file3.txt /backup/

# Copy directory (recursive)
cp -r /etc/nginx/ /backup/nginx-backup/

# Copy with preservation of attributes
cp -p file.txt backup.txt               # Preserve mode, ownership, timestamps
cp -a source_dir/ dest_dir/             # Archive mode (recursive + preserve all)

# Copy interactively (ask before overwrite)
cp -i important.conf /etc/

# Copy with verbose output
cp -v file.txt /backup/
# 'file.txt' -> '/backup/file.txt'

# Force copy (overwrite without asking)
cp -f source.txt destination.txt

# Copy only if source is newer
cp -u source.txt destination.txt

# Copy with backup of destination
cp --backup=numbered config.yaml /etc/myapp/config.yaml
# Creates config.yaml.~1~, config.yaml.~2~, etc.

# Common DevOps combinations
cp -av /etc/nginx/ /backup/nginx-$(date +%Y%m%d)/     # Dated backup
cp -rp /opt/app/v1/ /opt/app/v2/                       # Clone release
```

---

## 2.3 Moving and Renaming

```
┌────────────────────────────────────────────────────────────┐
│                    mv (move/rename)                         │
│                                                             │
│  Move:   Source ──────move──────→ New Location              │
│          (source removed)                                   │
│                                                             │
│  Rename: old_name ───rename────→ new_name                  │
│          (same location, new name)                          │
└────────────────────────────────────────────────────────────┘
```

```bash
# Rename a file
mv oldname.txt newname.txt

# Move file to a directory
mv file.txt /tmp/
mv file.txt /tmp/renamed.txt

# Move multiple files
mv file1.txt file2.txt file3.txt /backup/

# Move a directory
mv /opt/app/old-release/ /archive/

# Interactive (ask before overwrite)
mv -i source.txt /etc/config.txt

# Force (don't ask)
mv -f source.txt destination.txt

# Don't overwrite existing files
mv -n source.txt destination.txt

# Verbose
mv -v source.txt /backup/
# 'source.txt' -> '/backup/source.txt'

# Backup before overwriting
mv --backup=numbered config.txt /etc/config.txt

# Common DevOps patterns
mv /opt/app/current /opt/app/previous                  # Rotate releases
mv /var/log/app.log /var/log/app.log.$(date +%Y%m%d)   # Rotate logs
```

---

## 2.4 Deleting Files and Directories

```
┌────────────────────────────────────────────────────────────┐
│  ⚠️  WARNING: rm is PERMANENT — there is NO recycle bin    │
│                                                             │
│  ALWAYS double-check paths before running rm               │
│  NEVER run: rm -rf /  or  rm -rf /*                        │
│  Use --preserve-root (default on modern systems)           │
└────────────────────────────────────────────────────────────┘
```

```bash
# Delete a file
rm file.txt

# Delete multiple files
rm file1.txt file2.txt file3.txt

# Delete with confirmation
rm -i important.txt
# rm: remove regular file 'important.txt'? y

# Delete without error if file doesn't exist
rm -f nonexistent.txt

# Delete a directory (must be empty)
rmdir empty_dir/

# Delete directory and all contents (DANGEROUS)
rm -r directory/
rm -rf directory/              # Force, no confirmation

# Delete with verbose output
rm -rv /tmp/old-build/

# Safe deletion patterns for DevOps
# Always verify the path first:
echo "Will delete: /var/log/myapp/old-logs/"
ls /var/log/myapp/old-logs/
rm -rf /var/log/myapp/old-logs/

# Delete files matching a pattern
rm -f /tmp/*.log
rm -f /var/cache/apt/archives/*.deb

# Delete files older than 30 days
find /var/log/myapp/ -name "*.log" -mtime +30 -delete

# SAFE ALTERNATIVE: move to trash before deleting
mkdir -p /tmp/trash
mv risky-file.txt /tmp/trash/
```

---

## 2.5 Links: Hard Links and Symbolic Links

```
┌─────────────────────────────────────────────────────────────────┐
│              HARD LINK vs SYMBOLIC (SOFT) LINK                  │
│                                                                  │
│  HARD LINK:                      SYMBOLIC LINK:                 │
│  ┌────────┐   ┌──────────┐      ┌────────┐   ┌──────────┐     │
│  │ name1  │──→│  inode   │      │ link   │──→│ target   │     │
│  └────────┘   │  (data)  │      │(pointer)│   │ (file)   │     │
│  ┌────────┐   │          │      └────────┘   └──────────┘     │
│  │ name2  │──→│          │                                     │
│  └────────┘   └──────────┘      If target deleted → link       │
│  Both point to same data        is BROKEN                       │
│  Delete one → data remains      Works across filesystems        │
│  Same filesystem only           Can link to directories         │
│  Cannot link directories                                        │
└─────────────────────────────────────────────────────────────────┘
```

```bash
# Create a hard link
ln original.txt hardlink.txt
ls -li original.txt hardlink.txt    # Same inode number

# Create a symbolic (soft) link
ln -s /var/log/nginx/access.log ~/nginx-access.log
ln -s /opt/app/releases/v2.0 /opt/app/current

# View link target
readlink /opt/app/current
readlink -f /opt/app/current        # Full path

# Find broken symlinks
find /opt/app -type l ! -exec test -e {} \; -print

# Common DevOps symlink patterns
# Symlink for current release (zero-downtime deployments)
ln -sfn /opt/app/releases/v2.1 /opt/app/current
# -s = symbolic, -f = force, -n = don't follow existing symlink

# Symlink for config
ln -s /opt/app/shared/config.yaml /opt/app/current/config.yaml
```

---

## 2.6 File Information Commands

```bash
# File type and info
file document.pdf
# document.pdf: PDF document, version 1.4

file script.sh
# script.sh: Bash script, ASCII text executable

file /dev/sda
# /dev/sda: block special

# File statistics  
stat myfile.txt
# Output:
#   File: myfile.txt
#   Size: 1024         Blocks: 8          IO Block: 4096   regular file
#   Device: 801h/2049d  Inode: 1234567     Links: 1
#   Access: (0644/-rw-r--r--)  Uid: (1000/devops)   Gid: (1000/devops)
#   Access: 2026-02-20 10:30:00.000000000 +0000
#   Modify: 2026-02-20 10:15:00.000000000 +0000
#   Change: 2026-02-20 10:15:00.000000000 +0000

# Count lines, words, characters
wc myfile.txt
# 42 150 1024 myfile.txt

wc -l myfile.txt        # Line count only
wc -w myfile.txt        # Word count only
wc -c myfile.txt        # Byte count only

# Disk usage of files/directories
du -sh /var/log/         # Summary, human-readable
du -h --max-depth=1 /var/    # One level deep
```

---

## 2.7 Real-World DevOps Scenario

### Scenario: Setting Up a Blue-Green Deployment Structure

```bash
#!/bin/bash
# setup-deployment.sh — Create deployment directory structure

APP_NAME="mywebapp"
BASE_DIR="/opt/${APP_NAME}"

# Create directory structure
mkdir -p ${BASE_DIR}/{releases,shared/{config,logs,tmp,pids}}

# Structure:
# /opt/mywebapp/
# ├── current -> releases/20260220-143000  (symlink)
# ├── releases/
# │   ├── 20260220-143000/
# │   ├── 20260219-101500/
# │   └── 20260218-090000/
# └── shared/
#     ├── config/
#     ├── logs/
#     ├── tmp/
#     └── pids/

# Deploy new release
RELEASE_DIR="${BASE_DIR}/releases/$(date +%Y%m%d-%H%M%S)"
mkdir -p ${RELEASE_DIR}

# Copy application files
cp -a /tmp/build/* ${RELEASE_DIR}/

# Link shared files
ln -s ${BASE_DIR}/shared/config/database.yml ${RELEASE_DIR}/config/database.yml
ln -s ${BASE_DIR}/shared/logs ${RELEASE_DIR}/logs

# Switch current to new release (atomic operation)
ln -sfn ${RELEASE_DIR} ${BASE_DIR}/current

# Keep only last 5 releases
cd ${BASE_DIR}/releases
ls -dt */ | tail -n +6 | xargs rm -rf

echo "Deployed to ${RELEASE_DIR}"
echo "Current: $(readlink ${BASE_DIR}/current)"
```

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|---------|
| `cp: cannot create regular file: Permission denied` | No write permission on destination | Use `sudo` or check permissions |
| `rm: cannot remove: Is a directory` | Trying to `rm` a directory without `-r` | Use `rm -r directory/` |
| `mv: cannot move across filesystems` | Different partitions | Use `cp` + `rm` instead |
| `ln: hard link not allowed for directory` | Hard links can't point to dirs | Use `ln -s` (symlink) |
| `Too many levels of symbolic links` | Circular symlinks | `readlink -f` to trace, fix the chain |
| `No space left on device` | Disk full | `df -h`, clean old files |

---

## Summary Table

| Command | Purpose | Key Flags |
|---------|---------|-----------|
| `touch` | Create file / update timestamp | — |
| `mkdir` | Create directory | `-p` (parents), `-m` (mode) |
| `cp` | Copy files/dirs | `-r`, `-a`, `-p`, `-v`, `-i` |
| `mv` | Move / rename | `-i`, `-n`, `-v`, `--backup` |
| `rm` | Delete files/dirs | `-r`, `-f`, `-i`, `-v` |
| `rmdir` | Delete empty directory | — |
| `ln` | Create link | `-s` (symbolic), `-f` (force) |
| `file` | Identify file type | — |
| `stat` | Detailed file info | — |
| `wc` | Count lines/words/bytes | `-l`, `-w`, `-c` |

---

## Quick Revision Questions

1. **What is the difference between `cp -r` and `cp -a`?** When should you use each?
2. **How do you create a nested directory structure in one command?** Give an example.
3. **Explain the difference between hard links and symbolic links.** When would you use each?
4. **Why is `rm -rf /` dangerous?** What safeguards exist?
5. **How would you implement an atomic deployment using symlinks?** Describe the process.
6. **What does `ln -sfn` do and why is `-n` important for deployment symlinks?**

---

[← Previous: Navigation Commands](01-navigation-commands.md) | [Back to README](../README.md) | [Next: Text Viewing →](03-text-viewing.md)
