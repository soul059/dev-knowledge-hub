# Linux - Complete Learning Resource

A comprehensive collection of Linux documentation covering system administration, shell scripting, security, and productivity tools.

## 📚 Table of Contents

- [Overview](#overview)
- [Repository Structure](#repository-structure)
- [Learning Paths](#learning-paths)
- [Quick Reference](#quick-reference)

## 🎯 Overview

This Linux documentation provides in-depth coverage from basics to advanced topics, suitable for beginners, system administrators, security professionals, and DevOps engineers.

## 📂 Repository Structure

### Core Linux Notes
**[notes/](./notes/)**

Comprehensive 12-chapter curriculum covering Linux fundamentals to advanced topics:

| Chapter | Topic | Description |
|---------|-------|-------------|
| 01 | [Introduction to Linux](./notes/01-introduction-to-linux.md) | History, distributions, getting started |
| 02 | [File System Structure](./notes/02-file-system-structure.md) | FHS, directories, navigation |
| 03 | [Process Management](./notes/03-process-management.md) | Processes, signals, job control |
| 04 | [User & Group Management](./notes/04-user-group-management.md) | Users, groups, permissions |
| 05 | [Package Management](./notes/05-package-management.md) | APT, YUM/DNF, Pacman, Snap |
| 06 | [Networking](./notes/06-networking.md) | Configuration, troubleshooting, firewall |
| 07 | [System Administration](./notes/07-system-administration.md) | Systemd, logs, scheduling, backup |
| 08 | [Shell Scripting](./notes/08-shell-scripting.md) | Bash scripting fundamentals |
| 09 | [Security Hardening](./notes/09-linux-security-hardening.md) | Hardening, auditing, intrusion detection |
| 10 | [Systemd Deep Dive](./notes/10-systemd-deep-dive.md) | Advanced systemd configuration |
| 11 | [Containers Basics](./notes/11-containers-basics.md) | Docker, Podman, container fundamentals |
| 12 | [Performance Tuning](./notes/12-performance-tuning.md) | Optimization, benchmarking, profiling |

### Terminal Multiplexer
**[Tmux/](./Tmux/)**

Complete 10-chapter Tmux guide:
- Sessions, windows, and panes management
- Copy mode and scrollback
- Configuration and customization
- Plugins and scripting
- Cheatsheet and tips

### HackTheBox Practice
**[HTB/](./HTB/)**

Security-focused Linux practice notes:
- Linux Hardening (Firewall, Security, Logs)
- Shell operations
- Networking troubleshooting
- Regular expressions

### Arch Linux
**[arch/](./arch/)**

Arch-specific documentation:
- Hyprland installation and configuration
- AUR helper (yay)
- Network troubleshooting

### Quick References
| File | Description |
|------|-------------|
| [Terminal Commands.md](./Terminal%20Commands.md) | Essential command reference |
| [SSH.md](./SSH.md) | SSH configuration and usage |
| [regex.md](./regex.md) | Regular expressions guide |

## 🛤️ Learning Paths

### 🔰 Beginner Path
```
Introduction → File System → Terminal Commands → Process Management
    → User Management → Package Management → Basic Shell Scripting
```

### 🔧 System Administrator Path
```
All Beginner Topics → Networking → System Administration
    → Systemd Deep Dive → Security Hardening → Performance Tuning
```

### 🔒 Security Professional Path
```
Fundamentals → Security Hardening → HTB Practice
    → Shell Scripting → Networking → Tmux
```

### 🐳 DevOps/Container Path
```
Fundamentals → Shell Scripting → Containers Basics
    → Systemd → Performance Tuning → Networking
```

## 📝 Quick Reference

### Essential Commands
```bash
# File Operations
ls -la          # List with details
cd /path        # Change directory
cp -r src dst   # Copy recursively
mv old new      # Move/rename
rm -rf dir      # Remove directory

# Process Management
ps aux          # List processes
top / htop      # Process monitor
kill -9 PID     # Force kill
systemctl       # Service management

# User Management
useradd user    # Create user
passwd user     # Set password
sudo command    # Run as root

# Networking
ip addr         # Show IP addresses
ss -tulpn       # Show listening ports
ping host       # Test connectivity
curl url        # HTTP request

# Package Management (Debian/Ubuntu)
apt update      # Update package list
apt install pkg # Install package
apt remove pkg  # Remove package
```

### File Permissions
```
rwx = 7 (read + write + execute)
rw- = 6 (read + write)
r-x = 5 (read + execute)
r-- = 4 (read only)

chmod 755 file  # rwxr-xr-x
chmod 644 file  # rw-r--r--
chown user:group file
```

### Directory Structure
```
/           Root
├── bin     Essential binaries
├── boot    Boot loader files
├── dev     Device files
├── etc     Configuration files
├── home    User directories
├── lib     Shared libraries
├── opt     Optional software
├── proc    Process information
├── root    Root home directory
├── tmp     Temporary files
├── usr     User programs
└── var     Variable data (logs, mail)
```

## 🔧 Prerequisites

- Access to a Linux system (physical, VM, WSL, or cloud)
- Basic command-line familiarity
- Text editor knowledge (vim, nano, or any editor)

## 💡 Recommended Setup

| Purpose | Recommendation |
|---------|----------------|
| Beginners | Ubuntu Desktop in VirtualBox or WSL2 |
| Practice | Cloud VM (AWS, DigitalOcean, Linode) |
| Security | Kali Linux or Parrot OS |
| Minimal | Arch Linux or Debian minimal |

## 🎯 Skills Progression

```
Level 1: Navigation, file operations, basic commands
    ↓
Level 2: User management, permissions, package management
    ↓
Level 3: Shell scripting, process management, networking
    ↓
Level 4: System administration, systemd, services
    ↓
Level 5: Security hardening, performance tuning, containers
```

---

*Last Updated: April 2026*
