# Network Services Security

## Unit 6 - Topic 4 | Linux Network Security

---

## Overview

Every network service running on Linux is a potential **attack vector**. Securing services involves **minimizing exposure**, **hardening configurations**, **applying patches**, and **monitoring activity**. This topic covers security hardening for common Linux network services.

---

## 1. Service Hardening Principles

```
┌──────────────────────────────────────────────────────────────┐
│              SERVICE HARDENING CHECKLIST                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Run as non-root user (dedicated service account)        │
│  2. Bind to specific interface (not 0.0.0.0)                │
│  3. Restrict with firewall rules                             │
│  4. Enable logging and monitoring                            │
│  5. Keep software updated (patch vulnerabilities)            │
│  6. Use TLS/encryption for data in transit                   │
│  7. Disable unnecessary features/modules                     │
│  8. Apply MAC policy (SELinux/AppArmor profile)             │
│  9. Set resource limits (cgroups, ulimits)                   │
│  10. Chroot or containerize when possible                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Common Service Hardening

### Web Server (Apache/Nginx)

```bash
# Hide version information
# Apache: /etc/apache2/apache2.conf
ServerTokens Prod
ServerSignature Off

# Nginx: /etc/nginx/nginx.conf
server_tokens off;

# Disable directory listing
# Apache:
Options -Indexes

# Nginx:
autoindex off;

# Security headers
add_header X-Frame-Options "SAMEORIGIN";
add_header X-Content-Type-Options "nosniff";
add_header X-XSS-Protection "1; mode=block";
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";

# Disable unnecessary modules
# Apache:
a2dismod status info cgi
# Nginx: compile without unneeded modules
```

### DNS Server (BIND)

```bash
# /etc/bind/named.conf.options
options {
    recursion no;                   # Disable recursion (authoritative only)
    allow-transfer { none; };       # Disable zone transfers
    version "not disclosed";        # Hide version
    allow-query { 10.0.0.0/24; };  # Restrict queries
};
```

### Database (MySQL/PostgreSQL)

```bash
# MySQL hardening
mysql_secure_installation           # Run post-install security script
# • Set root password
# • Remove anonymous users
# • Disable remote root login
# • Remove test database

# Bind to localhost only (unless remote access needed)
# /etc/mysql/mysql.conf.d/mysqld.cnf
bind-address = 127.0.0.1

# PostgreSQL — /etc/postgresql/*/main/pg_hba.conf
# Only allow local connections
local all all                       peer
host  all all 127.0.0.1/32         scram-sha-256
```

---

## 3. Binding to Specific Interfaces

```bash
# PROBLEM: Service listens on 0.0.0.0 (all interfaces)
ss -tlnp
# LISTEN  0.0.0.0:3306  mysqld    ← Exposed to all networks!

# SOLUTION: Bind to localhost
# MySQL:  bind-address = 127.0.0.1
# Redis:  bind 127.0.0.1
# MongoDB: bindIp: 127.0.0.1

# After change:
ss -tlnp
# LISTEN  127.0.0.1:3306  mysqld  ← Only accessible locally ✅
```

---

## 4. Service Isolation Techniques

| Technique | Description |
|-----------|-------------|
| **chroot** | Restrict service to a directory subtree |
| **Container** | Docker/Podman for full isolation |
| **systemd sandboxing** | ProtectSystem, PrivateTmp, etc. |
| **SELinux/AppArmor** | MAC confinement |
| **Network namespace** | Isolated network stack |
| **Dedicated user** | Non-root service account |

```bash
# chroot example (BIND DNS)
# BIND can run in a chroot jail
named -t /var/named/chroot

# systemd sandboxing example
# /etc/systemd/system/myservice.service
[Service]
User=myservice
ProtectSystem=strict
ProtectHome=yes
PrivateTmp=yes
NoNewPrivileges=yes
ReadWritePaths=/var/lib/myservice
```

---

## 5. Service Monitoring

```bash
# Monitor service status
systemctl status nginx

# Monitor logs
journalctl -u nginx -f

# Monitor connections
ss -tlnp | grep nginx
watch -n 5 'ss -tlnp'               # Refresh every 5 seconds

# Monitor resource usage
systemd-cgtop                        # cgroup-based resource monitoring
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Non-root** | Run services as dedicated users |
| **Bind localhost** | Don't expose services to all interfaces |
| **Hide versions** | Prevent information disclosure |
| **TLS** | Encrypt all service communications |
| **Patch** | Keep services updated |
| **Isolate** | chroot, containers, systemd sandboxing |

---

## Quick Revision Questions

1. **Why should services bind to 127.0.0.1 when possible?**
2. **How do you hide the Apache version?**
3. **What does `mysql_secure_installation` do?**
4. **Name 3 service isolation techniques.**
5. **What is chroot and how does it improve security?**

---

[← Previous: TCP Wrappers](03-tcp-wrappers.md) | [Next: Network Namespaces →](05-network-namespaces.md)

---

*Unit 6 - Topic 4 of 5 | [Back to README](../README.md)*
