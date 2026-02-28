# Chapter 5: Package Managers

[← Previous: Boot Process](04-boot-process.md) | [Back to README](../README.md) | [Next: Navigation Commands →](../02-Command-Line-Essentials/01-navigation-commands.md)

---

## Overview

Package managers are essential tools for installing, updating, and removing software on Linux systems. As a DevOps engineer, you'll use them daily for provisioning servers, managing dependencies, and automating infrastructure. Understanding the differences between `apt`, `yum`, and `dnf` is critical when working across multiple distributions.

---

## 5.1 How Package Management Works

```
┌──────────────────────────────────────────────────────────────┐
│                PACKAGE MANAGEMENT WORKFLOW                    │
│                                                               │
│  ┌────────────┐    ┌─────────────┐    ┌─────────────────┐   │
│  │ Repository │    │   Package   │    │   Local System   │   │
│  │  (Remote)  │───→│   Manager   │───→│   (Installed)    │   │
│  │            │    │             │    │                   │   │
│  │ *.deb/.rpm │    │ apt/dnf/yum │    │ /usr, /etc,     │   │
│  │ metadata   │    │             │    │ /var, /opt       │   │
│  └────────────┘    └──────┬──────┘    └─────────────────┘   │
│                           │                                   │
│                    ┌──────▼──────┐                            │
│                    │ Dependency  │                            │
│                    │ Resolution  │                            │
│                    │ & Conflict  │                            │
│                    │ Checking    │                            │
│                    └─────────────┘                            │
└──────────────────────────────────────────────────────────────┘
```

### Package Format Comparison

```
┌───────────────────────────────────────────────────────────┐
│              PACKAGE FORMATS                               │
├────────────────┬──────────────────────────────────────────┤
│   .deb         │            .rpm                          │
│  (Debian)      │         (Red Hat)                        │
├────────────────┼──────────────────────────────────────────┤
│ dpkg (low)     │ rpm (low-level)                         │
│ apt (high)     │ yum / dnf (high-level)                  │
│                │                                          │
│ Ubuntu         │ RHEL, CentOS                            │
│ Debian         │ Fedora                                   │
│ Linux Mint     │ Amazon Linux                             │
└────────────────┴──────────────────────────────────────────┘
```

---

## 5.2 APT (Advanced Package Tool) — Debian/Ubuntu

### Architecture

```
┌───────────────────────────────────────────────────────┐
│                   APT ECOSYSTEM                        │
│                                                        │
│  ┌──────────┐                                         │
│  │   apt    │  ← High-level (user-friendly)           │
│  └────┬─────┘                                         │
│       │                                                │
│  ┌────▼─────┐                                         │
│  │ apt-get  │  ← Traditional CLI tool                 │
│  │ apt-cache│  ← Query package info                   │
│  └────┬─────┘                                         │
│       │                                                │
│  ┌────▼─────┐                                         │
│  │  dpkg    │  ← Low-level package installer          │
│  └──────────┘                                         │
│                                                        │
│  Config: /etc/apt/sources.list                        │
│          /etc/apt/sources.list.d/                      │
│  Cache:  /var/cache/apt/archives/                     │
│  State:  /var/lib/apt/lists/                          │
└───────────────────────────────────────────────────────┘
```

### Essential APT Commands

```bash
# ═══════════════════════════════════════════
# UPDATING & UPGRADING
# ═══════════════════════════════════════════

# Update package index (ALWAYS do this first)
sudo apt update

# Upgrade all packages
sudo apt upgrade              # Safe upgrade (won't remove packages)
sudo apt full-upgrade         # May remove packages if needed
sudo apt dist-upgrade         # Same as full-upgrade

# ═══════════════════════════════════════════
# INSTALLING & REMOVING
# ═══════════════════════════════════════════

# Install a package
sudo apt install nginx
sudo apt install nginx=1.18.0-6ubuntu14    # Specific version
sudo apt install -y nginx                   # Auto-confirm

# Install multiple packages
sudo apt install nginx curl vim git

# Install from .deb file
sudo dpkg -i package.deb
sudo apt install -f              # Fix broken dependencies

# Remove a package
sudo apt remove nginx            # Remove, keep config
sudo apt purge nginx             # Remove + config files
sudo apt autoremove              # Remove unused dependencies

# ═══════════════════════════════════════════
# SEARCHING & INFO
# ═══════════════════════════════════════════

# Search for packages
apt search nginx
apt search "web server"

# Show package details
apt show nginx

# List installed packages
dpkg -l                          # All installed
dpkg -l | grep nginx             # Filter
apt list --installed             # Alternative

# Which package owns a file
dpkg -S /usr/sbin/nginx

# List files in a package
dpkg -L nginx

# ═══════════════════════════════════════════
# MAINTENANCE
# ═══════════════════════════════════════════

# Clean package cache
sudo apt clean                   # Remove all cached packages
sudo apt autoclean               # Remove obsolete cached packages

# Check for broken packages
sudo dpkg --configure -a
sudo apt install -f

# Hold a package (prevent upgrades)
sudo apt-mark hold nginx
sudo apt-mark unhold nginx
apt-mark showhold
```

### Repository Management

```bash
# View current repositories
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/

# Example sources.list entry:
# deb http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse

# Add a PPA (Personal Package Archive)
sudo add-apt-repository ppa:ondrej/nginx
sudo apt update

# Add a third-party repository (e.g., Docker)
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list

sudo apt update
sudo apt install docker-ce
```

---

## 5.3 YUM & DNF — RHEL/CentOS/Fedora

### Architecture

```
┌───────────────────────────────────────────────────────┐
│                YUM / DNF ECOSYSTEM                     │
│                                                        │
│  ┌──────────┐                                         │
│  │   dnf    │  ← Modern replacement for yum           │
│  │   (yum)  │  ← yum still works as alias            │
│  └────┬─────┘                                         │
│       │                                                │
│  ┌────▼─────┐                                         │
│  │   rpm    │  ← Low-level package manager            │
│  └──────────┘                                         │
│                                                        │
│  Config: /etc/yum.repos.d/                            │
│          /etc/dnf/dnf.conf                            │
│  Cache:  /var/cache/yum/ or /var/cache/dnf/           │
│  RPM DB: /var/lib/rpm/                                │
└───────────────────────────────────────────────────────┘
```

### Essential YUM/DNF Commands

```bash
# ═══════════════════════════════════════════
# UPDATING & UPGRADING
# ═══════════════════════════════════════════

# Update package cache and upgrade
sudo dnf check-update           # Check for updates
sudo dnf update                 # Update all packages
sudo dnf upgrade                # Same as update

# ═══════════════════════════════════════════
# INSTALLING & REMOVING
# ═══════════════════════════════════════════

# Install a package
sudo dnf install httpd
sudo dnf install httpd-2.4.51   # Specific version
sudo dnf install -y httpd       # Auto-confirm

# Install from local RPM
sudo dnf install ./package.rpm

# Install a group of packages
sudo dnf group install "Development Tools"
sudo dnf group list              # List available groups

# Remove a package
sudo dnf remove httpd
sudo dnf autoremove              # Remove unused dependencies

# ═══════════════════════════════════════════
# SEARCHING & INFO
# ═══════════════════════════════════════════

# Search for packages
dnf search nginx
dnf search all "web server"

# Show package details
dnf info httpd

# List installed packages
rpm -qa                          # All installed
rpm -qa | grep httpd
dnf list installed

# Which package owns a file
rpm -qf /usr/sbin/httpd

# List files in a package
rpm -ql httpd

# ═══════════════════════════════════════════
# MAINTENANCE
# ═══════════════════════════════════════════

# Clean cache
sudo dnf clean all
sudo dnf clean packages
sudo dnf clean metadata

# Check for broken dependencies
sudo rpm -Va                     # Verify all packages

# History and rollback
dnf history                      # View transaction history
sudo dnf history undo 15         # Undo transaction #15
sudo dnf history rollback 12     # Rollback to transaction #12

# Version lock
sudo dnf install python3-dnf-plugin-versionlock
sudo dnf versionlock add httpd
sudo dnf versionlock list
sudo dnf versionlock delete httpd
```

### Repository Management

```bash
# View configured repositories
dnf repolist
dnf repolist all                 # Include disabled repos

# View repo files
ls /etc/yum.repos.d/

# Example repo file: /etc/yum.repos.d/docker-ce.repo
# [docker-ce-stable]
# name=Docker CE Stable - $basearch
# baseurl=https://download.docker.com/linux/centos/$releasever/$basearch/stable
# enabled=1
# gpgcheck=1
# gpgkey=https://download.docker.com/linux/centos/gpg

# Enable/disable a repo
sudo dnf config-manager --set-enabled docker-ce-stable
sudo dnf config-manager --set-disabled docker-ce-stable

# Add a repo
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Enable EPEL (Extra Packages for Enterprise Linux)
sudo dnf install epel-release
```

---

## 5.4 APT vs DNF — Side-by-Side Comparison

| Task | APT (Debian/Ubuntu) | DNF/YUM (RHEL/CentOS) |
|------|---------------------|----------------------|
| Update index | `apt update` | `dnf check-update` |
| Upgrade packages | `apt upgrade` | `dnf upgrade` |
| Install | `apt install nginx` | `dnf install nginx` |
| Remove | `apt remove nginx` | `dnf remove nginx` |
| Purge (+ config) | `apt purge nginx` | `dnf remove nginx` (no purge) |
| Search | `apt search nginx` | `dnf search nginx` |
| Package info | `apt show nginx` | `dnf info nginx` |
| List installed | `dpkg -l` | `rpm -qa` |
| File owner | `dpkg -S /path/file` | `rpm -qf /path/file` |
| List pkg files | `dpkg -L nginx` | `rpm -ql nginx` |
| Clean cache | `apt clean` | `dnf clean all` |
| Auto-remove | `apt autoremove` | `dnf autoremove` |
| Hold package | `apt-mark hold nginx` | `dnf versionlock add nginx` |
| Add repo | `add-apt-repository` | `dnf config-manager --add-repo` |
| Install local | `dpkg -i pkg.deb` | `dnf install ./pkg.rpm` |
| Fix broken | `apt install -f` | `dnf distro-sync` |

---

## 5.5 Best Practices for DevOps

```
┌──────────────────────────────────────────────────────────────┐
│           PACKAGE MANAGEMENT BEST PRACTICES                  │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  1. ALWAYS update index before installing                    │
│     sudo apt update && sudo apt install nginx                │
│     sudo dnf makecache && sudo dnf install httpd             │
│                                                               │
│  2. Pin versions in production                               │
│     sudo apt install nginx=1.18.0-6ubuntu14                  │
│     sudo dnf install httpd-2.4.51                            │
│                                                               │
│  3. Use -y flag in automation scripts                        │
│     sudo apt install -y nginx                                │
│     sudo dnf install -y httpd                                │
│                                                               │
│  4. Clean caches in Docker images                            │
│     RUN apt-get update && apt-get install -y nginx \         │
│         && rm -rf /var/lib/apt/lists/*                       │
│                                                               │
│  5. Use non-interactive mode in scripts                      │
│     DEBIAN_FRONTEND=noninteractive apt-get install -y pkg    │
│                                                               │
│  6. Verify GPG keys for external repos                       │
│                                                               │
│  7. Test updates in staging before production                │
│                                                               │
│  8. Use version locking for critical packages                │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### Dockerfile Best Practice

```dockerfile
# GOOD: Single layer, cache cleaned
FROM ubuntu:22.04
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        nginx \
        curl \
        vim && \
    rm -rf /var/lib/apt/lists/*

# BAD: Multiple layers, cache not cleaned
FROM ubuntu:22.04
RUN apt-get update
RUN apt-get install -y nginx
RUN apt-get install -y curl
```

---

## 5.6 Real-World DevOps Scenario

### Scenario: Automating Server Provisioning with a Shell Script

```bash
#!/bin/bash
# provision.sh — Idempotent server provisioning script

set -euo pipefail

# Detect distribution
if [ -f /etc/debian_version ]; then
    PKG_MANAGER="apt"
    echo "Detected Debian-based distribution"
elif [ -f /etc/redhat-release ]; then
    PKG_MANAGER="dnf"
    echo "Detected Red Hat-based distribution"
else
    echo "Unsupported distribution"
    exit 1
fi

# Install packages based on detected distro
install_packages() {
    case $PKG_MANAGER in
        apt)
            export DEBIAN_FRONTEND=noninteractive
            sudo apt-get update -qq
            sudo apt-get install -y -qq \
                nginx \
                curl \
                wget \
                vim \
                git \
                htop \
                net-tools \
                unzip
            ;;
        dnf)
            sudo dnf install -y -q \
                nginx \
                curl \
                wget \
                vim \
                git \
                htop \
                net-tools \
                unzip
            ;;
    esac
}

echo "Installing packages..."
install_packages
echo "Server provisioned successfully!"
```

---

## Troubleshooting Tips

| Issue | Distro | Solution |
|-------|--------|---------|
| `E: Unable to locate package` | Ubuntu | Run `apt update` first; check sources.list |
| `No package available` | RHEL | Enable EPEL or correct repo |
| Dependency conflict | Both | `apt install -f` or `dnf distro-sync` |
| GPG key error | Both | Import the key manually |
| Lock file exists | Ubuntu | `sudo rm /var/lib/dpkg/lock-frontend` (if no apt running) |
| Broken packages | Ubuntu | `sudo dpkg --configure -a && apt install -f` |
| RPM DB corrupted | RHEL | `sudo rpm --rebuilddb` |
| Disk full on update | Both | Clean cache: `apt clean` / `dnf clean all` |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **apt** | High-level package manager for Debian/Ubuntu |
| **dnf** | Modern package manager for RHEL/CentOS/Fedora |
| **yum** | Legacy package manager (now aliased to dnf) |
| **dpkg** | Low-level .deb package tool |
| **rpm** | Low-level .rpm package tool |
| **Repository** | Remote server hosting packages and metadata |
| **GPG Key** | Cryptographic key to verify package authenticity |
| **Version Pinning** | Locking a package to a specific version |
| **EPEL** | Extra Packages for Enterprise Linux |
| **PPA** | Personal Package Archive (Ubuntu) |

---

## Quick Revision Questions

1. **What is the difference between `apt update` and `apt upgrade`?** When should each be used?
2. **How do you install a specific version of a package?** Show examples for both apt and dnf.
3. **What is the difference between `apt remove` and `apt purge`?** When would you use each?
4. **How do you add a third-party repository?** Describe the steps for Ubuntu.
5. **Why should you clean package caches in Dockerfiles?** Show the correct pattern.
6. **How do you prevent a critical package from being accidentally upgraded?** Show both apt and dnf methods.

---

[← Previous: Boot Process](04-boot-process.md) | [Back to README](../README.md) | [Next: Navigation Commands →](../02-Command-Line-Essentials/01-navigation-commands.md)
