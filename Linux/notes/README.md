# Linux Notes - Table of Contents

This directory contains comprehensive notes about the Linux operating system, covering essential topics for both beginners and advanced users.

## Overview

Linux is an open-source, Unix-like operating system that powers everything from servers to smartphones. These notes provide detailed coverage of Linux fundamentals, system administration, and advanced topics.

## Notes Structure

### 1. [Introduction to Linux](01-introduction-to-linux.md)
- What is Linux?
- Key features and advantages
- Linux distributions
- Linux philosophy
- Getting started with Linux

### 2. [File System Structure](02-file-system-structure.md)
- Linux file system hierarchy (FHS)
- Important directories and their purposes
- File types and permissions
- Mount points and file systems
- Navigation and path concepts

### 3. [Process Management](03-process-management.md)
- Understanding processes
- Process states and lifecycle
- Process creation and control
- Job control and background processes
- Process monitoring and troubleshooting
- Signals and process communication

### 4. [User and Group Management](04-user-group-management.md)
- User types and accounts
- User information storage (/etc/passwd, /etc/shadow)
- Creating, modifying, and deleting users
- Group management
- Password policies and security
- sudo configuration

### 5. [Package Management](05-package-management.md)
- Package management systems
- APT (Debian/Ubuntu)
- YUM/DNF (Red Hat/Fedora)
- Pacman (Arch Linux)
- Universal packages (Snap, Flatpak)
- Dependency management

### 6. [Networking](06-networking.md)
- Network fundamentals
- Network interface configuration
- IP address management
- Routing and DNS
- Network troubleshooting tools
- Firewall configuration
- Wireless networking

### 7. [System Administration](07-system-administration.md)
- System information and monitoring
- Boot process and init systems
- Service management with systemd
- Log management and analysis
- Cron and task scheduling
- System backup and recovery
- Security administration

### 8. [Shell Scripting](08-shell-scripting.md)
- Introduction to shell scripting
- Variables and input/output
- Control structures (if/else, loops)
- Functions and arrays
- String manipulation
- Error handling
- Practical scripting examples

### 9. [Linux Security Hardening](09-linux-security-hardening.md)
- Security fundamentals and defense in depth
- User and access security
- SSH hardening
- Firewall configuration (UFW, iptables, nftables)
- File system security and ACLs
- Kernel hardening with sysctl
- Service hardening with systemd
- Audit and monitoring (auditd, logging)
- Intrusion detection (AIDE, rkhunter)
- Security automation

### 10. [Systemd Deep Dive](10-systemd-deep-dive.md)
- Unit files and types
- Service management and creation
- Timers (cron alternative)
- Targets and boot process
- Journald logging
- Resource control with cgroups
- Socket activation
- Troubleshooting systemd

### 11. [Containers Basics](11-containers-basics.md)
- Container fundamentals (namespaces, cgroups)
- Docker installation and setup
- Images, containers, and Dockerfile
- Docker networking and volumes
- Docker Compose
- Podman (rootless alternative)
- Container security best practices

### 12. [Performance Tuning](12-performance-tuning.md)
- Performance fundamentals (USE method)
- CPU, memory, disk, and network optimization
- Process performance and limits
- Kernel tuning with sysctl
- Application profiling (strace, perf, valgrind)
- Benchmarking tools
- Performance monitoring

## Learning Path

### Beginner Level
1. Start with **Introduction to Linux** to understand the basics
2. Learn **File System Structure** to navigate effectively
3. Understand **Process Management** fundamentals
4. Master basic **User and Group Management**

### Intermediate Level
1. Dive deeper into **Package Management**
2. Learn **Networking** concepts and configuration
3. Explore **System Administration** tasks
4. Begin with basic **Shell Scripting**

### Advanced Level
1. Master advanced **System Administration**
2. Develop proficiency in **Shell Scripting**
3. Implement security best practices
4. Learn automation and configuration management

## Prerequisites

- Basic computer literacy
- Familiarity with command-line interfaces (helpful but not required)
- Access to a Linux system (physical, virtual machine, or cloud instance)

## Recommended Setup

### For Beginners
- **VirtualBox** or **VMware** with Ubuntu Desktop
- **WSL2** (Windows Subsystem for Linux) on Windows
- **Live USB** with Ubuntu or Linux Mint

### For Practice
- Cloud instances (AWS EC2, Google Cloud, DigitalOcean)
- Local virtual machines
- Raspberry Pi for hardware experience

## Essential Commands by Topic

### File Operations
```bash
ls, cd, pwd, mkdir, rmdir, rm, cp, mv, find, locate, which, file
```

### Text Processing
```bash
cat, less, more, head, tail, grep, sed, awk, sort, uniq, wc
```

### Process Management
```bash
ps, top, htop, kill, killall, jobs, nohup, screen, tmux
```

### System Information
```bash
uname, whoami, id, uptime, df, du, free, lscpu, lsblk
```

### Network Commands
```bash
ping, traceroute, netstat, ss, wget, curl, ssh, scp, rsync
```

### Package Management
```bash
apt, yum, dnf, pacman, snap, flatpak, dpkg, rpm
```

## Practice Exercises

Each note file includes practical examples and exercises. Consider setting up a practice environment to:

1. **Install different Linux distributions**
2. **Practice command-line operations**
3. **Write and test shell scripts**
4. **Configure network services**
5. **Set up system monitoring**
6. **Implement backup strategies**

## Additional Resources

### Online Resources
- [Linux Documentation Project](https://tldp.org/)
- [Arch Linux Wiki](https://wiki.archlinux.org/)
- [Ubuntu Documentation](https://help.ubuntu.com/)
- [Red Hat Documentation](https://access.redhat.com/documentation/)

### Books
- "The Linux Command Line" by William Shotts
- "Linux Administration: A Beginner's Guide" by Wale Soyinka
- "UNIX and Linux System Administration Handbook" by Evi Nemeth

### Practice Platforms
- [OverTheWire](https://overthewire.org/wargames/) - Security challenges
- [HackerRank](https://www.hackerrank.com/domains/shell) - Shell scripting practice
- [Linux Journey](https://linuxjourney.com/) - Interactive learning

## Contributing

These notes are designed to be comprehensive but can always be improved. Areas for enhancement:

1. **More practical examples**
2. **Distribution-specific details**
3. **Troubleshooting scenarios**
4. **Performance optimization tips**
5. **Security hardening guides**

## Quick Reference

### File Permissions
```
rwx rwx rwx = 777 (read, write, execute for owner, group, others)
rw- r-- r-- = 644 (read/write owner, read group/others)
rwx r-x r-x = 755 (full owner, read/execute group/others)
```

### Special Directories
```
/       Root directory
/home   User home directories
/etc    Configuration files
/var    Variable data (logs, mail, etc.)
/tmp    Temporary files
/usr    User programs and data
/opt    Optional software
```

### Environment Variables
```
$HOME   User's home directory
$PATH   Executable search path
$USER   Current username
$PWD    Current working directory
$SHELL  Current shell
```

Remember: Linux mastery comes through practice. Use these notes as a reference, but make sure to apply the concepts in real scenarios to build practical experience.
