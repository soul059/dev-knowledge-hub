# Chapter 1: Navigation Commands

[← Previous: Package Managers](../01-Linux-Fundamentals/05-package-managers.md) | [Back to README](../README.md) | [Next: File Operations →](02-file-operations.md)

---

## Overview

Navigating the Linux filesystem from the command line is the most fundamental skill for any DevOps engineer. Mastering `cd`, `pwd`, `ls`, and related commands allows you to move through directories efficiently, inspect file structures, and understand how applications and services are organized.

---

## 1.1 The Shell Prompt

```
┌────────────────────────────────────────────────────────┐
│                  SHELL PROMPT                           │
│                                                         │
│  user@hostname:~/projects$                              │
│   │      │       │        │                             │
│   │      │       │        └── $ = regular user          │
│   │      │       │            # = root user             │
│   │      │       └── Current directory (~ = home)       │
│   │      └── Hostname of the machine                    │
│   └── Current username                                  │
│                                                         │
│  Examples:                                              │
│  devops@web-server-01:/var/log$                         │
│  root@db-server-02:/etc#                                │
└────────────────────────────────────────────────────────┘
```

---

## 1.2 `pwd` — Print Working Directory

```bash
# Show current directory
pwd
# Output: /home/devops

# Logical path (follows symlinks — default)
pwd -L

# Physical path (resolves symlinks)
pwd -P
# If /var/log is a symlink → shows real path

# In scripts, store the current directory
CURRENT_DIR=$(pwd)
echo "We are in: $CURRENT_DIR"
```

---

## 1.3 `cd` — Change Directory

```
┌────────────────────────────────────────────────────────────┐
│              PATH TYPES IN LINUX                            │
│                                                             │
│  ABSOLUTE PATH           RELATIVE PATH                     │
│  (starts from /)         (starts from current dir)         │
│                                                             │
│  /home/devops/scripts    ./scripts                          │
│  /var/log/nginx          ../../var/log                      │
│  /etc/ssh/sshd_config    ../etc/ssh/sshd_config            │
│                                                             │
│  Special Symbols:                                           │
│  /     Root directory                                       │
│  ~     Home directory (/home/username)                      │
│  .     Current directory                                    │
│  ..    Parent directory                                     │
│  -     Previous directory                                   │
└────────────────────────────────────────────────────────────┘
```

```bash
# Go to home directory (all equivalent)
cd
cd ~
cd $HOME
cd /home/devops

# Navigate with absolute path
cd /var/log/nginx

# Navigate with relative path
cd ../apache2           # Go up one level, then into apache2

# Go to parent directory
cd ..

# Go to grandparent
cd ../..

# Go to previous directory (toggle between two)
cd /var/log
cd /etc/ssh
cd -                    # Back to /var/log
cd -                    # Back to /etc/ssh

# Go to root
cd /

# Navigate using environment variables
cd $HOME
cd $OLDPWD              # Same as cd -

# Use with pushd/popd (directory stack)
pushd /var/log          # Push current dir, go to /var/log
pushd /etc/ssh          # Push /var/log, go to /etc/ssh
popd                    # Go back to /var/log
popd                    # Go back to original dir
dirs -v                 # View directory stack
```

---

## 1.4 `ls` — List Directory Contents

### Basic Usage

```bash
# List files in current directory
ls

# List all files (including hidden)
ls -a

# Long format (detailed info)
ls -l

# Combine flags
ls -la                  # Long format + hidden files
ls -lah                 # + human-readable sizes

# List specific directory
ls /var/log
ls -la /etc/ssh/
```

### Understanding `ls -l` Output

```
┌────────────────────────────────────────────────────────────────────┐
│  $ ls -la                                                          │
│                                                                     │
│  drwxr-xr-x  5 devops devops  4096 Feb 20 10:30 .                 │
│  drwxr-xr-x  3 root   root    4096 Feb 15 08:00 ..                │
│  -rw-r--r--  1 devops devops   220 Feb 15 08:00 .bashrc           │
│  -rwxr-xr-x  1 devops devops  1024 Feb 20 10:15 deploy.sh        │
│  lrwxrwxrwx  1 devops devops    15 Feb 20 10:30 logs -> /var/log  │
│                                                                     │
│  ├─────────┤  │ ├─────┤├─────┤├────┤├───────────┤ ├────────────┤  │
│  │          │  │   │      │     │        │           │              │
│  Type &     │  │  Owner  Group Size   Modified     Name            │
│  Permissions│  │                       Date/Time                    │
│             │  │                                                    │
│  Hard link count                                                    │
│                                                                     │
│  FILE TYPE INDICATORS:                                              │
│  -  = Regular file                                                  │
│  d  = Directory                                                     │
│  l  = Symbolic link                                                 │
│  c  = Character device                                              │
│  b  = Block device                                                  │
│  s  = Socket                                                        │
│  p  = Named pipe                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### Useful `ls` Variations

```bash
# Sort by modification time (newest first)
ls -lt

# Sort by modification time (oldest first)
ls -ltr

# Sort by size (largest first)
ls -lS

# Recursive listing
ls -R

# Show only directories
ls -d */

# Show file sizes in human-readable format
ls -lh
# -rw-r--r-- 1 user user 2.4M Feb 20 10:30 backup.tar.gz

# Show inode numbers
ls -li

# One file per line
ls -1

# Show files with classification (/ for dirs, * for executables)
ls -F

# Colorized output (usually default)
ls --color=auto

# List with comma separator
ls -m

# Show only hidden files
ls -d .*
```

### `ls` for DevOps Tasks

```bash
# Check recent log files
ls -lt /var/log/ | head -10

# Find largest files in a directory
ls -lS /var/log/ | head -10

# Check permissions on SSH keys
ls -la ~/.ssh/
# Should show:
# drwx------  2 user user 4096 ... .ssh
# -rw-------  1 user user 3243 ... id_rsa        (600)
# -rw-r--r--  1 user user  741 ... id_rsa.pub    (644)
# -rw-r--r--  1 user user  222 ... known_hosts   (644)

# List systemd service files
ls -la /etc/systemd/system/

# Check Docker socket permissions
ls -la /var/run/docker.sock

# Monitor a directory for new files
watch -n 2 'ls -lt /var/log/nginx/ | head -5'
```

---

## 1.5 Related Navigation Commands

```bash
# tree — Visual directory structure
tree                     # Current directory
tree -L 2                # Limit depth to 2 levels
tree -d                  # Directories only
tree -I "node_modules"   # Ignore pattern
tree /etc/nginx/

# Example tree output:
# /etc/nginx/
# ├── conf.d/
# │   └── default.conf
# ├── mime.types
# ├── nginx.conf
# └── sites-enabled/
#     └── default -> ../sites-available/default

# Install tree if not available
# sudo apt install tree     # Ubuntu
# sudo dnf install tree     # RHEL

# basename — Extract filename from path
basename /var/log/syslog
# Output: syslog

# dirname — Extract directory from path
dirname /var/log/syslog
# Output: /var/log

# realpath — Resolve the full absolute path
realpath ./scripts/../deploy.sh
# Output: /home/devops/deploy.sh

# readlink — Read the target of a symlink
readlink /usr/bin/python3
# Output: python3.10
readlink -f /usr/bin/python3   # Full path resolution
```

---

## 1.6 Navigation Shortcuts & Efficiency Tips

```
┌────────────────────────────────────────────────────────┐
│          NAVIGATION EFFICIENCY TIPS                     │
├────────────────────────────────────────────────────────┤
│                                                         │
│  Tab Completion:                                        │
│    cd /va[TAB]     → cd /var/                          │
│    cd /var/lo[TAB] → cd /var/log/                      │
│    ls /etc/ss[TAB] → ls /etc/ssh/                      │
│                                                         │
│  Brace Expansion:                                       │
│    ls /var/{log,cache,lib}                              │
│    mkdir -p project/{src,bin,docs}                      │
│                                                         │
│  Wildcards (Globbing):                                  │
│    ls *.log          → All .log files                   │
│    ls file?.txt      → file1.txt, file2.txt             │
│    ls [abc]*.conf    → a.conf, b.conf, c.conf           │
│    ls [!a]*.conf     → NOT starting with 'a'            │
│                                                         │
│  CDPATH (auto-search directories):                      │
│    export CDPATH=.:~:/var/log                           │
│    cd nginx   → finds /var/log/nginx                    │
│                                                         │
│  Aliases:                                                │
│    alias ll='ls -alh'                                    │
│    alias la='ls -A'                                      │
│    alias ..='cd ..'                                      │
│    alias ...='cd ../..'                                   │
└────────────────────────────────────────────────────────┘
```

---

## 1.7 Real-World DevOps Scenario

### Scenario: Investigating a Deployment Issue

```bash
# You SSH into a production server to debug a failed deployment

# 1. Check where you are
pwd
# /home/deploy

# 2. Check the deployment directory
cd /opt/myapp/releases
ls -lt | head -5
# drwxr-xr-x 8 deploy deploy 4096 Feb 20 14:30 20260220-143000
# drwxr-xr-x 8 deploy deploy 4096 Feb 19 10:15 20260219-101500
# drwxr-xr-x 8 deploy deploy 4096 Feb 18 09:00 20260218-090000

# 3. Check which release is current (symlink)
ls -la /opt/myapp/current
# lrwxrwxrwx 1 deploy deploy 45 Feb 20 14:30 current -> /opt/myapp/releases/20260220-143000

# 4. Check the latest release
cd /opt/myapp/current
ls -la
tree -L 1

# 5. Check logs
cd /var/log/myapp/
ls -lt | head -5
tail -50 error.log

# 6. Go back to the previous release for comparison
cd /opt/myapp/releases/20260219-101500
diff <(ls -la) <(ls -la /opt/myapp/current/)
```

---

## Troubleshooting Tips

| Issue | Cause | Solution |
|-------|-------|---------|
| `No such file or directory` | Typo or file doesn't exist | Use `Tab` completion; check with `ls` |
| `Permission denied` | No execute (x) on directory | `chmod +x dir` or use `sudo` |
| Can't `cd` into a directory | Missing execute permission | `ls -la` to check, `chmod 755 dir` |
| Symlink leads nowhere | Broken symlink | `readlink -f link`, recreate if needed |
| `ls` shows garbled names | Special characters in filenames | Use `ls -b` or quote filenames |
| Tab completion not working | Bash completion not installed | `sudo apt install bash-completion` |

---

## Summary Table

| Command | Purpose | Common Flags |
|---------|---------|-------------|
| `pwd` | Print current directory | `-P` (physical) |
| `cd` | Change directory | `~`, `..`, `-` |
| `ls` | List files/dirs | `-l`, `-a`, `-h`, `-t`, `-R`, `-S` |
| `tree` | Visual directory tree | `-L`, `-d`, `-I` |
| `pushd/popd` | Directory stack navigation | — |
| `basename` | Extract filename | — |
| `dirname` | Extract directory path | — |
| `realpath` | Resolve full path | — |
| `readlink` | Read symlink target | `-f` (full) |

---

## Quick Revision Questions

1. **What is the difference between an absolute path and a relative path?** Give examples of each.
2. **What do `.`, `..`, `~`, and `-` represent when used with `cd`?**
3. **Explain each field in the output of `ls -l`.** What does `drwxr-xr-x` mean?
4. **How would you list the 5 most recently modified files in `/var/log`?** Write the command.
5. **What is `pushd` and `popd`?** When are they useful compared to `cd`?
6. **How does Tab completion improve efficiency?** What other shortcuts help with navigation?

---

[← Previous: Package Managers](../01-Linux-Fundamentals/05-package-managers.md) | [Back to README](../README.md) | [Next: File Operations →](02-file-operations.md)
