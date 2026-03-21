# Service Management

## Unit 2 - Topic 2 | Linux Hardening

---

## Overview

**Service management** in Linux controls which processes start automatically and run in the background. From a security perspective, every running service is a potential **entry point** for attackers. Modern Linux uses **systemd** as the init system and service manager.

---

## 1. systemd Basics

```bash
# SERVICE LIFECYCLE COMMANDS

# View service status
systemctl status sshd
systemctl status nginx

# Start / Stop / Restart
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx              # Reload config without restart

# Enable / Disable (boot persistence)
systemctl enable nginx              # Start at boot
systemctl disable nginx             # Don't start at boot
systemctl enable --now nginx        # Enable AND start immediately

# Mask (prevent ANY start — even manual)
systemctl mask bluetooth            # Completely prevent starting
systemctl unmask bluetooth          # Allow starting again

# List all services
systemctl list-units --type=service              # Active services
systemctl list-units --type=service --all         # All services
systemctl list-unit-files --type=service          # Enabled/disabled status
```

---

## 2. Identifying Running Services

```bash
# View all listening network services (CRITICAL for security)
ss -tlnp                            # TCP listeners with process info
ss -ulnp                            # UDP listeners
ss -tlnp | grep LISTEN              # Only listening ports

# Example output:
# State  Local Address:Port  Process
# LISTEN 0.0.0.0:22          sshd
# LISTEN 0.0.0.0:80          nginx
# LISTEN 127.0.0.1:3306      mysqld

# Alternative: netstat
netstat -tlnp                       # If net-tools installed

# View all running services
systemctl list-units --type=service --state=running

# View services enabled at boot
systemctl list-unit-files --type=service --state=enabled
```

---

## 3. Service Security Assessment

| Question | How to Check | Action |
|----------|-------------|--------|
| Is this service needed? | Business requirement | Disable if not needed |
| Does it listen on network? | `ss -tlnp` | Restrict to localhost if possible |
| Is it up to date? | `apt list --installed` | Patch vulnerabilities |
| Does it run as root? | `ps aux \| grep svc` | Use dedicated service user |
| Is it enabled at boot? | `systemctl is-enabled svc` | Disable if not needed at boot |
| What files does it access? | `lsof -p PID` | Restrict with AppArmor/SELinux |

---

## 4. systemd Security Features

```bash
# systemd provides built-in security hardening for services

# View security assessment of a service
systemd-analyze security nginx

# Example output:
# Overall exposure: 9.2 UNSAFE  ← High number = poorly hardened
# PrivateNetwork=        no
# ProtectSystem=         no
# ProtectHome=           no

# HARDENING A SERVICE UNIT FILE (/etc/systemd/system/myservice.service)
[Service]
# Run as non-root user
User=www-data
Group=www-data

# Filesystem restrictions
ProtectSystem=strict              # Mount / as read-only
ProtectHome=yes                   # Hide /home, /root, /run/user
ReadWritePaths=/var/www            # Only allow writes here
PrivateTmp=yes                    # Isolated /tmp

# Network restrictions
PrivateNetwork=no                 # (yes = no network access)
RestrictAddressFamilies=AF_INET AF_INET6  # IPv4/IPv6 only

# Privilege restrictions
NoNewPrivileges=yes               # Can't gain new privileges
ProtectKernelTunables=yes         # Can't modify /proc/sys
ProtectKernelModules=yes          # Can't load kernel modules
ProtectControlGroups=yes          # Can't modify cgroups
CapabilityBoundingSet=CAP_NET_BIND_SERVICE  # Only needed capabilities

# After editing, reload:
systemctl daemon-reload
systemctl restart myservice
```

---

## 5. Service Hardening Checklist

| Step | Description |
|------|-------------|
| 1 | List all enabled services |
| 2 | Identify which are needed for the server's purpose |
| 3 | Disable/mask unnecessary services |
| 4 | For required services: bind to localhost if possible |
| 5 | Run services as non-root (dedicated user) |
| 6 | Apply systemd sandboxing directives |
| 7 | Use `systemd-analyze security` to assess |
| 8 | Monitor services with logging |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **systemctl** | Primary tool for service management |
| **ss -tlnp** | Show listening network services |
| **mask** | Completely prevent a service from starting |
| **systemd security** | Built-in sandboxing (ProtectSystem, PrivateTmp, etc.) |
| **Non-root** | Run services as dedicated users |
| **Audit** | `systemd-analyze security <service>` |

---

## Quick Revision Questions

1. **What is the difference between `disable` and `mask`?**
2. **How do you list all listening network services?**
3. **What does `ProtectSystem=strict` do in a systemd unit?**
4. **Why should services run as non-root users?**
5. **How do you assess the security of a systemd service?**

---

[← Previous: Minimal Installation](01-minimal-installation.md) | [Next: Unnecessary Service Removal →](03-unnecessary-service-removal.md)

---

*Unit 2 - Topic 2 of 7 | [Back to README](../README.md)*
