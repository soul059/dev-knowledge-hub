# Minimal Installation

## Unit 2 - Topic 1 | Linux Hardening

---

## Overview

**Minimal installation** is the first principle of Linux hardening — install only what is absolutely necessary. Every installed package, service, and library expands the **attack surface**. A minimal system has fewer vulnerabilities, fewer patches to manage, and fewer avenues for exploitation.

---

## 1. Attack Surface Reduction

```
┌──────────────────────────────────────────────────────────────┐
│              ATTACK SURFACE COMPARISON                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  FULL DESKTOP INSTALL:                                       │
│  ┌────────────────────────────────────────┐                  │
│  │ OS + Desktop + Media + Games + Dev +   │ ~2000+ packages  │
│  │ Servers + Network Tools + Docs + ...   │ Large attack     │
│  └────────────────────────────────────────┘ surface          │
│                                                              │
│  MINIMAL SERVER INSTALL:                                     │
│  ┌──────────────────┐                                        │
│  │ OS + SSH + Tools │ ~200-400 packages                     │
│  └──────────────────┘ Small attack surface                   │
│                                                              │
│  CONTAINER / SCRATCH:                                        │
│  ┌──────┐                                                    │
│  │ App  │ ~10-50 packages                                   │
│  └──────┘ Minimal attack surface                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Minimal Install Strategies

| Strategy | Description |
|----------|-------------|
| **Minimal ISO** | Use server/minimal install media (not desktop) |
| **Deselect groups** | Uncheck GUI, development, multimedia during install |
| **Network install** | netinst image — download only selected packages |
| **Post-install audit** | Remove unnecessary packages after install |
| **Purpose-built** | Install only packages needed for the server's role |

---

## 3. Distro-Specific Minimal Install

```bash
# DEBIAN/UBUNTU — Minimal Install
# During installation, deselect all tasksel groups
# Only select "SSH Server" and "Standard system utilities"

# List installed packages
dpkg -l | wc -l                         # Count packages
dpkg --get-selections | grep -v deinstall

# Remove unnecessary packages
apt purge <package>
apt autoremove                           # Remove orphaned dependencies

# RHEL/CentOS/Rocky — Minimal Install
# Select "Minimal Install" during installation

# List installed packages
rpm -qa | wc -l
yum list installed

# Remove unnecessary packages
yum remove <package>
yum autoremove

# Check for GUI packages (should not exist on servers)
rpm -qa | grep -i xorg
rpm -qa | grep -i gnome
```

---

## 4. Post-Install Hardening Checklist

| Step | Command | Purpose |
|------|---------|---------|
| List packages | `dpkg -l` / `rpm -qa` | Inventory installed software |
| Remove GUI | `apt purge 'xorg*' 'gnome*'` | No desktop on servers |
| Remove games | `apt purge gnome-games` | Unnecessary software |
| Remove compilers | `apt purge gcc g++ make` | Prevent on-system compilation |
| Remove dev tools | `apt purge gdb strace` | Reduce debugging capability |
| Disable services | `systemctl disable <svc>` | Reduce running attack surface |
| Remove SUID | `chmod u-s /usr/bin/unnecessary` | Reduce privilege escalation |
| Remove docs | `apt purge man-db` | Reduce footprint (optional) |

---

## 5. CIS Benchmarks

```
CIS (Center for Internet Security) Benchmarks provide
step-by-step hardening guides for minimal installs:

• CIS Ubuntu Linux Benchmark
• CIS RHEL/CentOS Benchmark
• CIS Debian Benchmark

Key recommendations:
□ Ensure only required packages are installed
□ Ensure only required services are enabled
□ Ensure no development tools on production servers
□ Ensure no compilers unless required
□ Ensure no GUI on servers

Tools for automated CIS compliance:
• OpenSCAP — automated compliance scanning
• Lynis — Unix security auditing
• CIS-CAT — official CIS assessment tool
```

```bash
# Lynis — Security audit tool
lynis audit system

# OpenSCAP compliance check
oscap xccdf eval --profile cis /path/to/benchmark.xml
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Minimal install** | Only install necessary packages |
| **Attack surface** | More packages = more vulnerabilities |
| **No GUI on servers** | Remove Xorg, GNOME, KDE |
| **No compilers** | Prevent attackers from compiling exploits |
| **CIS Benchmarks** | Industry-standard hardening guides |
| **Audit tools** | Lynis, OpenSCAP, CIS-CAT |

---

## Quick Revision Questions

1. **Why is a minimal installation important for security?**
2. **What types of packages should be removed from production servers?**
3. **Why should compilers not be installed on production systems?**
4. **What are CIS Benchmarks?**
5. **Name 2 tools for automated security auditing.**

---

[Next: Service Management →](02-service-management.md)

---

*Unit 2 - Topic 1 of 7 | [Back to README](../README.md)*
