# Kernel Hardening (sysctl)

## Unit 2 - Topic 4 | Linux Hardening

---

## Overview

The Linux kernel has numerous tunable parameters that affect security. **sysctl** is the tool used to view and modify these parameters at runtime. Kernel hardening through sysctl can prevent **IP spoofing**, **SYN floods**, **ICMP attacks**, **source routing**, and many other network and system-level attacks.

---

## 1. sysctl Basics

```bash
# View all kernel parameters
sysctl -a

# View specific parameter
sysctl net.ipv4.ip_forward

# Set parameter temporarily (lost on reboot)
sysctl -w net.ipv4.ip_forward=0

# Set parameter permanently
# Edit /etc/sysctl.conf or create /etc/sysctl.d/99-security.conf
echo "net.ipv4.ip_forward = 0" >> /etc/sysctl.d/99-security.conf

# Apply changes from config file
sysctl -p
sysctl --system                      # Load all sysctl.d files
```

---

## 2. Network Hardening Parameters

```bash
# /etc/sysctl.d/99-security.conf

# === IP FORWARDING ===
# Disable IP forwarding (not a router)
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0

# === ICMP HARDENING ===
# Ignore ICMP echo requests (ping) — optional, may break monitoring
net.ipv4.icmp_echo_ignore_all = 0
# Ignore broadcast pings (prevent Smurf attacks)
net.ipv4.icmp_echo_ignore_broadcasts = 1
# Ignore bogus ICMP error responses
net.ipv4.icmp_ignore_bogus_error_responses = 1

# === SOURCE ROUTING ===
# Disable source routing (prevent IP spoofing)
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# === REVERSE PATH FILTERING ===
# Enable — validates source address (anti-spoofing)
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# === REDIRECT PROTECTION ===
# Don't accept ICMP redirects (prevent MITM)
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
# Don't send ICMP redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# === SYN FLOOD PROTECTION ===
# Enable SYN cookies
net.ipv4.tcp_syncookies = 1
# Reduce SYN-ACK retries
net.ipv4.tcp_synack_retries = 2
# Increase SYN backlog
net.ipv4.tcp_max_syn_backlog = 2048

# === TCP HARDENING ===
# Log suspicious packets (martians)
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
```

---

## 3. System Hardening Parameters

```bash
# === MEMORY PROTECTION ===
# Restrict kernel pointer exposure
kernel.kptr_restrict = 2
# Restrict dmesg to root only
kernel.dmesg_restrict = 1
# Restrict access to kernel profiling
kernel.perf_event_paranoid = 3

# === PROCESS HARDENING ===
# Restrict ptrace (prevents debugging other processes)
kernel.yama.ptrace_scope = 2
# Disable core dumps for SUID programs
fs.suid_dumpable = 0

# === ADDRESS SPACE LAYOUT RANDOMIZATION ===
# Enable full ASLR (prevents memory exploitation)
kernel.randomize_va_space = 2
# 0 = disabled, 1 = partial, 2 = full

# === EXEC SHIELD / NX ===
# Most modern kernels have NX bit support by default

# === MAGIC SYSRQ ===
# Disable magic SysRq key (physical access exploit)
kernel.sysrq = 0
```

---

## 4. Parameter Reference Table

| Parameter | Value | Purpose |
|-----------|:-----:|---------|
| `net.ipv4.ip_forward` | 0 | Disable routing |
| `net.ipv4.tcp_syncookies` | 1 | SYN flood protection |
| `net.ipv4.conf.all.rp_filter` | 1 | Anti-IP spoofing |
| `net.ipv4.conf.all.accept_redirects` | 0 | Prevent MITM |
| `net.ipv4.conf.all.accept_source_route` | 0 | Prevent source routing |
| `net.ipv4.icmp_echo_ignore_broadcasts` | 1 | Prevent Smurf attack |
| `net.ipv4.conf.all.log_martians` | 1 | Log suspicious packets |
| `kernel.randomize_va_space` | 2 | Enable ASLR |
| `kernel.kptr_restrict` | 2 | Hide kernel addresses |
| `kernel.dmesg_restrict` | 1 | Restrict dmesg access |
| `kernel.yama.ptrace_scope` | 2 | Restrict process debugging |
| `fs.suid_dumpable` | 0 | No core dumps for SUID |

---

## 5. Verification

```bash
# Verify all security parameters
sysctl net.ipv4.ip_forward
sysctl net.ipv4.tcp_syncookies
sysctl kernel.randomize_va_space

# Check all security-related settings
sysctl -a | grep -E '(forward|syncookie|rp_filter|redirect|source_route|randomize)'

# Automated check with Lynis
lynis audit system --tests-from-group "kernel"
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **sysctl** | Tool to configure kernel parameters |
| **IP forwarding** | Disable on non-routers |
| **SYN cookies** | Enable for SYN flood protection |
| **rp_filter** | Enable for anti-spoofing |
| **ASLR** | Set to 2 for full randomization |
| **Persistence** | Use `/etc/sysctl.d/` for permanent config |

---

## Quick Revision Questions

1. **What is sysctl and what does it control?**
2. **How do you make sysctl changes persistent across reboots?**
3. **What does `rp_filter` do?**
4. **What is ASLR and why should it be set to 2?**
5. **How do SYN cookies prevent SYN flood attacks?**

---

[← Previous: Unnecessary Service Removal](03-unnecessary-service-removal.md) | [Next: PAM Configuration →](05-pam-configuration.md)

---

*Unit 2 - Topic 4 of 7 | [Back to README](../README.md)*
