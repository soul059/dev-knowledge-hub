# Unnecessary Service Removal

## Unit 2 - Topic 3 | Linux Hardening

---

## Overview

Removing **unnecessary services** is a core hardening step. Services that aren't needed for the system's purpose should be disabled or removed entirely. Common targets include legacy protocols, development tools, print services, Bluetooth, and Avahi — services that increase attack surface without providing business value.

---

## 1. Commonly Unnecessary Services

| Service | Package | Why Remove |
|---------|---------|-----------|
| **avahi-daemon** | avahi | mDNS — information disclosure, rarely needed on servers |
| **cups** | cups | Print service — not needed on most servers |
| **bluetooth** | bluez | Bluetooth — attack surface, not needed on servers |
| **rpcbind** | rpcbind | RPC services — legacy, often exploited |
| **nfs-server** | nfs-kernel-server | NFS — unless specifically needed |
| **telnet** | telnetd | Cleartext remote access — use SSH instead |
| **ftp** | vsftpd | Cleartext file transfer — use SFTP/SCP |
| **rsh/rlogin** | rsh-server | Legacy remote access — insecure |
| **tftp** | tftpd | Trivial FTP — no authentication |
| **postfix** | postfix | Mail server — unless needed |
| **apache2/nginx** | apache2/nginx | Web server — unless needed |
| **snapd** | snapd | Snap package manager — optional |

---

## 2. Removing Services

```bash
# STEP 1: Identify unnecessary services
systemctl list-units --type=service --state=running
ss -tlnp                            # What's listening?

# STEP 2: Stop the service
systemctl stop avahi-daemon
systemctl stop cups

# STEP 3: Disable from starting at boot
systemctl disable avahi-daemon
systemctl disable cups

# STEP 4: Mask (prevent manual start)
systemctl mask avahi-daemon
systemctl mask cups

# STEP 5: Remove the package entirely (best option)
# Debian/Ubuntu:
apt purge avahi-daemon cups bluetooth rpcbind telnetd
apt autoremove

# RHEL/CentOS:
yum remove avahi cups bluez rpcbind telnet-server
```

---

## 3. Legacy/Insecure Services to ALWAYS Remove

```
┌──────────────────────────────────────────────────────────────┐
│              SERVICES TO ALWAYS DISABLE/REMOVE                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  CLEARTEXT PROTOCOLS (replace with encrypted alternatives):  │
│  ❌ telnet    → ✅ SSH                                       │
│  ❌ FTP       → ✅ SFTP / SCP                                │
│  ❌ rsh/rlogin → ✅ SSH                                      │
│  ❌ TFTP      → ✅ SFTP                                      │
│  ❌ HTTP      → ✅ HTTPS (if web server needed)              │
│                                                              │
│  LEGACY SERVICES:                                            │
│  ❌ rpcbind (portmapper)                                     │
│  ❌ NIS (Network Information Service)                        │
│  ❌ finger                                                   │
│  ❌ talk/ntalk                                               │
│  ❌ chargen/daytime/discard/echo                             │
│                                                              │
│  INFORMATION DISCLOSURE:                                     │
│  ❌ avahi-daemon (mDNS — announces services)                │
│  ❌ SNMP (if not needed — exposes system info)              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Verification

```bash
# Verify no unnecessary services are running
systemctl list-units --type=service --state=running

# Verify no unnecessary ports are open
ss -tlnp
# Expected for a hardened web server:
# LISTEN  *:22    sshd
# LISTEN  *:443   nginx
# (nothing else!)

# Verify packages are removed
dpkg -l | grep -i telnet
dpkg -l | grep -i avahi
# Should return nothing

# Automated audit with Lynis
lynis audit system --tests-from-group "services"
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Remove first** | Uninstall packages, not just disable |
| **Mask** | Prevents even manual start of service |
| **Cleartext** | Always remove telnet, FTP, rsh |
| **Legacy** | Remove rpcbind, NIS, finger |
| **Verify** | `ss -tlnp` to confirm only needed ports open |
| **Audit** | Regular review of running services |

---

## Quick Revision Questions

1. **Why is removing packages better than just disabling services?**
2. **What does masking a service do?**
3. **Name 5 services commonly removed during hardening.**
4. **What should replace telnet and FTP?**
5. **How do you verify only necessary ports are listening?**

---

[← Previous: Service Management](02-service-management.md) | [Next: Kernel Hardening →](04-kernel-hardening.md)

---

*Unit 2 - Topic 3 of 7 | [Back to README](../README.md)*
