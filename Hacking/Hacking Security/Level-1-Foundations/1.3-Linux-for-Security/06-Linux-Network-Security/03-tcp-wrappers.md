# TCP Wrappers

## Unit 6 - Topic 3 | Linux Network Security

---

## Overview

**TCP Wrappers** is a host-based access control system that uses two files — `/etc/hosts.allow` and `/etc/hosts.deny` — to control which hosts can connect to network services. While considered legacy (many modern services don't support it), TCP wrappers are still used and tested in security certifications.

---

## 1. How TCP Wrappers Work

```
┌──────────────────────────────────────────────────────────────┐
│              TCP WRAPPERS FLOW                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client connects to service (e.g., sshd)                     │
│       │                                                      │
│       ▼                                                      │
│  Check /etc/hosts.allow                                      │
│       │                                                      │
│     MATCH → ALLOW connection ✅                               │
│     NO MATCH ▼                                               │
│  Check /etc/hosts.deny                                       │
│       │                                                      │
│     MATCH → DENY connection ❌                                │
│     NO MATCH → ALLOW connection ✅ (default allow)           │
│                                                              │
│  ⚠️ IMPORTANT: hosts.allow is checked FIRST                  │
│  ⚠️ If a match is found in hosts.allow, hosts.deny           │
│     is NOT checked (allow overrides deny)                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Configuration Syntax

```bash
# FORMAT: daemon_list : client_list [: action]

# /etc/hosts.allow — ALLOW these connections
sshd : 10.0.0.0/24                   # Allow SSH from internal network
sshd : 192.168.1.100                  # Allow SSH from specific IP
ALL : 127.0.0.1                       # Allow all from localhost
in.tftpd : 10.0.0.0/24              # Allow TFTP from internal

# /etc/hosts.deny — DENY everything else
ALL : ALL                             # Default deny all!

# WILDCARDS:
ALL      — All services or all hosts
LOCAL    — Hosts without a dot in hostname
KNOWN    — Hosts whose name/address are known
UNKNOWN  — Hosts whose name/address are unknown
PARANOID — Hosts where name doesn't match address
```

---

## 3. Best Practice Configuration

```bash
# /etc/hosts.allow
# Only allow SSH from admin network
sshd : 10.0.0.0/24 : ALLOW

# /etc/hosts.deny
# Deny everything else
ALL : ALL : DENY

# With logging
sshd : 10.0.0.0/24 : spawn /usr/bin/logger -t tcpwrappers "SSH allowed from %a"
ALL : ALL : spawn /usr/bin/logger -t tcpwrappers "Connection denied from %a to %d"
```

---

## 4. Checking TCP Wrapper Support

```bash
# Check if a service uses TCP wrappers
ldd /usr/sbin/sshd | grep libwrap
# If libwrap is linked → service supports TCP wrappers

# Services that typically support TCP wrappers:
# sshd, vsftpd, rpcbind, xinetd-managed services

# Services that typically do NOT support TCP wrappers:
# nginx, apache2, postfix (modern versions)
# Most systemd socket-activated services
```

---

## 5. TCP Wrappers vs Firewall

| Feature | TCP Wrappers | iptables/nftables |
|---------|:---:|:---:|
| **Layer** | Application | Network (kernel) |
| **Scope** | Only libwrap-linked services | All traffic |
| **Performance** | Per-connection overhead | Kernel-level (fast) |
| **Granularity** | Per-service | Per-port/protocol/IP |
| **Modern use** | Declining | Primary firewall |
| **Defense in depth** | ✅ Additional layer | ✅ Primary layer |

> **Best practice:** Use iptables/nftables as primary firewall AND TCP wrappers as an additional defense layer.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **TCP Wrappers** | Host-based access control via hosts.allow/deny |
| **Order** | hosts.allow checked first, then hosts.deny |
| **Default deny** | `ALL : ALL` in hosts.deny |
| **libwrap** | Service must be linked against libwrap |
| **Legacy** | Still useful but declining; use with firewall |

---

## Quick Revision Questions

1. **Which file is checked first: hosts.allow or hosts.deny?**
2. **What happens if a connection matches neither file?**
3. **How do you check if a service supports TCP wrappers?**
4. **Write a hosts.allow entry to allow SSH from 10.0.0.0/24.**
5. **How do TCP wrappers differ from iptables?**

---

[← Previous: Netfilter and iptables](02-netfilter-and-iptables.md) | [Next: Network Services Security →](04-network-services-security.md)

---

*Unit 6 - Topic 3 of 5 | [Back to README](../README.md)*
