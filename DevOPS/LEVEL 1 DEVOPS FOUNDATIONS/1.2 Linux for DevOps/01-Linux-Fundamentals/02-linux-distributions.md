# Chapter 2: Linux Distributions

[← Previous: Linux Architecture](01-linux-architecture.md) | [Back to README](../README.md) | [Next: File System Hierarchy →](03-file-system-hierarchy.md)

---

## Overview

A Linux distribution (distro) bundles the Linux kernel with system utilities, package managers, desktop environments, and default configurations. Choosing the right distro is a critical decision for DevOps teams, as it affects stability, support, security, and tooling compatibility.

---

## 2.1 What Makes a Distribution?

```
┌─────────────────────────────────────────────────────┐
│              LINUX DISTRIBUTION                      │
│                                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │           Desktop Environment (optional)       │  │
│  │           (GNOME, KDE, XFCE, none)            │  │
│  ├────────────────────────────────────────────────┤  │
│  │           Package Manager                      │  │
│  │           (apt, yum, dnf, zypper, pacman)     │  │
│  ├────────────────────────────────────────────────┤  │
│  │           System Utilities & Services          │  │
│  │           (systemd, coreutils, OpenSSH)       │  │
│  ├────────────────────────────────────────────────┤  │
│  │           Init System                          │  │
│  │           (systemd, SysVinit, OpenRC)         │  │
│  ├────────────────────────────────────────────────┤  │
│  │           C Library                            │  │
│  │           (glibc, musl)                       │  │
│  ├────────────────────────────────────────────────┤  │
│  │           Linux Kernel                         │  │
│  │           (version varies by distro)          │  │
│  └────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────┘
```

---

## 2.2 Distribution Family Tree

```
                         ┌──────────┐
                         │  Linux   │
                         │  Kernel  │
                         └────┬─────┘
                              │
            ┌─────────────────┼─────────────────┐
            │                 │                 │
      ┌─────▼─────┐   ┌──────▼─────┐   ┌──────▼─────┐
      │  Debian    │   │  Red Hat   │   │   Others   │
      │  Family    │   │  Family    │   │            │
      └─────┬─────┘   └──────┬─────┘   └──────┬─────┘
            │                 │                 │
    ┌───────┼───────┐   ┌────┼─────┐     ┌─────┼──────┐
    │       │       │   │    │     │     │     │      │
┌───▼──┐ ┌──▼───┐   │ ┌▼───┐│┌────▼┐ ┌──▼──┐ │  ┌───▼───┐
│Ubuntu│ │Kali  │   │ │RHEL│││CentOS│ │Arch │ │  │Alpine │
└──┬───┘ └──────┘   │ └────┘│└─────┘ └─────┘ │  └───────┘
   │                │       │                  │
┌──▼───┐            │  ┌────▼─────┐      ┌────▼────┐
│Linux │            │  │  Fedora  │      │  SUSE   │
│Mint  │            │  └──────────┘      │  (SLES) │
└──────┘            │                    └─────────┘
                    │  ┌──────────┐
                    │  │  Amazon  │
                    └──│  Linux   │
                       └──────────┘
```

---

## 2.3 Key Distributions for DevOps

### Ubuntu (Debian-based)

```bash
# Identify Ubuntu version
lsb_release -a
cat /etc/os-release

# Package Manager: apt (Advanced Package Tool)
sudo apt update                  # Update package index
sudo apt upgrade                 # Upgrade installed packages
sudo apt install nginx           # Install a package
sudo apt remove nginx            # Remove a package
sudo apt autoremove              # Remove unused dependencies
dpkg -l                          # List installed packages
```

| Feature | Details |
|---------|---------|
| **Base** | Debian |
| **Package Format** | `.deb` |
| **Package Manager** | `apt` / `dpkg` |
| **Release Cycle** | Every 6 months; LTS every 2 years |
| **LTS Support** | 5 years (standard), 10 years (ESM) |
| **Default Init** | systemd |
| **Best For** | Cloud servers, development, containers |

### CentOS / CentOS Stream (RHEL-based)

```bash
# Identify CentOS version
cat /etc/centos-release
cat /etc/os-release

# Package Manager: yum / dnf
sudo yum update                  # Update packages (CentOS 7)
sudo dnf update                  # Update packages (CentOS 8+/Stream)
sudo yum install httpd           # Install a package
sudo yum remove httpd            # Remove a package
rpm -qa                          # List installed packages
```

| Feature | Details |
|---------|---------|
| **Base** | Red Hat Enterprise Linux |
| **Package Format** | `.rpm` |
| **Package Manager** | `yum` (CentOS 7) / `dnf` (CentOS 8+) |
| **CentOS Stream** | Rolling-release, upstream of RHEL |
| **Default Init** | systemd |
| **Best For** | Enterprise servers, traditional IT |

> **Note:** CentOS Linux 8 reached EOL Dec 2021. CentOS Stream is now the focus. Alternatives: AlmaLinux, Rocky Linux.

### Red Hat Enterprise Linux (RHEL)

```bash
# Identify RHEL version
cat /etc/redhat-release
subscription-manager status

# RHEL subscription management
sudo subscription-manager register
sudo subscription-manager attach --auto
sudo subscription-manager repos --enable=rhel-8-for-x86_64-appstream-rpms
```

| Feature | Details |
|---------|---------|
| **Base** | Fedora (upstream) |
| **Package Format** | `.rpm` |
| **Package Manager** | `dnf` (RHEL 8+) / `yum` (RHEL 7) |
| **Support** | 10 years per major version |
| **Cost** | Subscription-based (free for dev: RHEL Developer) |
| **Certifications** | Required for many enterprise deployments |
| **Best For** | Enterprise production, regulated industries |

### Amazon Linux

```bash
# Identify Amazon Linux version
cat /etc/os-release
cat /etc/system-release

# Uses yum / dnf
sudo yum install -y amazon-cloudwatch-agent
sudo amazon-linux-extras install docker
```

| Feature | Details |
|---------|---------|
| **Base** | RHEL/Fedora (AL2); Fedora (AL2023) |
| **Package Format** | `.rpm` |
| **Package Manager** | `yum` (AL2) / `dnf` (AL2023) |
| **Optimized For** | AWS EC2, ECS, Lambda |
| **AWS Integration** | Pre-configured AWS CLI, SSM agent |
| **Best For** | AWS workloads |

---

## 2.4 Choosing a Distribution — Decision Matrix

```
                    ┌─────────────────────────────────┐
                    │  What's your primary use case?   │
                    └───────────────┬─────────────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                   │
          ┌──────▼──────┐  ┌───────▼───────┐  ┌───────▼───────┐
          │  Cloud /    │  │  Enterprise   │  │  Containers   │
          │  General    │  │  / Regulated  │  │  / Minimal    │
          └──────┬──────┘  └───────┬───────┘  └───────┬───────┘
                 │                 │                   │
          ┌──────▼──────┐  ┌───────▼───────┐  ┌───────▼───────┐
          │   Ubuntu    │  │    RHEL       │  │   Alpine      │
          │   Server    │  │              │  │   Linux       │
          └─────────────┘  └───────────────┘  └───────────────┘
                 │                 │
          ┌──────▼──────┐  ┌───────▼───────┐
          │  On AWS?    │  │  Free RHEL    │
          │  → Amazon   │  │  clone?       │
          │    Linux    │  │  → AlmaLinux  │
          └─────────────┘  │  → Rocky      │
                           └───────────────┘
```

---

## 2.5 Comparison Table

| Feature | Ubuntu 22.04 | CentOS Stream 9 | RHEL 9 | Amazon Linux 2023 | Alpine |
|---------|-------------|-----------------|--------|-------------------|--------|
| **Package Mgr** | apt | dnf | dnf | dnf | apk |
| **Package Fmt** | .deb | .rpm | .rpm | .rpm | .apk |
| **Init System** | systemd | systemd | systemd | systemd | OpenRC |
| **C Library** | glibc | glibc | glibc | glibc | musl |
| **Base Image Size** | ~77 MB | ~200 MB | ~210 MB | ~150 MB | ~5 MB |
| **Support** | Community + Canonical | Community | Red Hat | AWS | Community |
| **SELinux** | AppArmor (default) | Yes | Yes | Yes | No |
| **Best Cloud** | All clouds | All clouds | All clouds | AWS only | Containers |

---

## 2.6 Checking Your Distribution

```bash
# Method 1: os-release (works on all modern distros)
cat /etc/os-release

# Method 2: lsb_release (Debian/Ubuntu)
lsb_release -a

# Method 3: Distribution-specific files
cat /etc/redhat-release      # RHEL/CentOS
cat /etc/debian_version      # Debian/Ubuntu
cat /etc/alpine-release      # Alpine

# Method 4: hostnamectl (systemd-based)
hostnamectl

# Method 5: uname (kernel info, not distro-specific)
uname -a
```

---

## 2.7 Real-World DevOps Scenario

### Scenario: Choosing a Distro for a Microservices Platform

Your team is deploying a microservices platform with these requirements:
- Container-based workloads on Kubernetes
- CI/CD pipelines
- Multi-cloud (AWS + GCP)
- Compliance requirements (SOC 2)

**Decision:**
```
┌────────────────────────────────────────────────────┐
│  PLATFORM DECISION                                  │
│                                                      │
│  Base Image for Containers: Alpine Linux             │
│  ├── Reason: ~5MB, minimal attack surface           │
│  ├── Reason: Fast build & pull times                │
│  └── Caution: musl libc — test compatibility        │
│                                                      │
│  Kubernetes Nodes: Ubuntu 22.04 LTS                  │
│  ├── Reason: Wide cloud support                     │
│  ├── Reason: 5-year LTS support                     │
│  └── Reason: Excellent K8s ecosystem support        │
│                                                      │
│  CI/CD Runners: Ubuntu 22.04                         │
│  ├── Reason: GitHub Actions native support          │
│  └── Reason: Broadest tool compatibility            │
└────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Issue | Solution |
|-------|---------|
| Package not found on RHEL | Enable the correct repo: `subscription-manager repos --list` |
| `apt update` fails | Check `/etc/apt/sources.list` for broken URLs |
| CentOS 8 EOL repos broken | Switch to CentOS Stream or Vault mirrors |
| Alpine: binary won't run | Likely a glibc binary on musl system — rebuild or use `gcompat` |
| Distro version mismatch in Docker | Always specify exact tag: `FROM ubuntu:22.04`, not `FROM ubuntu` |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Distribution** | Linux kernel + utilities + package manager + config |
| **Debian Family** | Ubuntu, Mint, Kali — uses `apt` / `.deb` |
| **Red Hat Family** | RHEL, CentOS, Fedora, Amazon Linux — uses `dnf`/`yum` / `.rpm` |
| **LTS** | Long Term Support — 5+ years of security updates |
| **Alpine** | Ultra-lightweight distro, ideal for containers (musl + apk) |
| **os-release** | Standard file for identifying the distribution |
| **Package Manager** | Tool for installing, updating, removing software |

---

## Quick Revision Questions

1. **What components make up a Linux distribution?** How is a distro different from just the kernel?
2. **Compare `apt` and `dnf`.** Which distro families use each?
3. **Why might you choose Alpine Linux for containers?** What's the trade-off?
4. **What happened to CentOS 8?** What are the recommended alternatives?
5. **How do you check which distribution and version you're running?** Give at least 3 methods.
6. **When would you choose RHEL over Ubuntu?** Consider enterprise requirements.

---

[← Previous: Linux Architecture](01-linux-architecture.md) | [Back to README](../README.md) | [Next: File System Hierarchy →](03-file-system-hierarchy.md)
