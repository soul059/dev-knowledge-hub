# Chapter 5: Firewall Configuration

[← Previous: DNS Configuration](04-dns-configuration.md) | [Back to README](../README.md) | [Next: SSH Configuration →](06-ssh-configuration.md)

---

## Overview

Firewalls control incoming and outgoing network traffic based on rules. Linux provides `iptables` (low-level), `nftables` (modern replacement), `ufw` (Ubuntu), and `firewalld` (RHEL/CentOS). DevOps engineers must know how to open ports for services, restrict access, and debug connectivity issues caused by firewall rules.

---

## 5.1 Linux Firewall Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              LINUX FIREWALL STACK                              │
│                                                                │
│  High Level (user-friendly)                                    │
│  ┌──────────┐   ┌────────────┐                                │
│  │   ufw    │   │  firewalld │                                │
│  │ (Ubuntu) │   │ (RHEL/CentOS)│                              │
│  └────┬─────┘   └─────┬──────┘                                │
│       │               │                                        │
│  ┌────▼───────────────▼──────┐                                │
│  │ iptables / nftables       │  ← Kernel interface            │
│  └───────────┬───────────────┘                                │
│              │                                                 │
│  ┌───────────▼───────────────┐                                │
│  │   netfilter (kernel)      │  ← Actual packet filtering    │
│  └───────────────────────────┘                                │
│                                                                │
│  iptables chains:                                              │
│  ┌─────────┐ ┌─────────┐ ┌──────────┐                        │
│  │  INPUT  │ │ FORWARD │ │  OUTPUT  │                        │
│  │(incoming)│ │(routed) │ │(outgoing)│                        │
│  └─────────┘ └─────────┘ └──────────┘                        │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.2 `ufw` — Uncomplicated Firewall (Ubuntu/Debian)

```bash
# Enable / Disable
sudo ufw enable
sudo ufw disable
sudo ufw status                      # Short status
sudo ufw status verbose              # Detailed
sudo ufw status numbered             # With rule numbers

# Default policies
sudo ufw default deny incoming       # Block all incoming
sudo ufw default allow outgoing      # Allow all outgoing

# Allow ports
sudo ufw allow 22                    # SSH
sudo ufw allow 80                    # HTTP
sudo ufw allow 443                   # HTTPS
sudo ufw allow 8080/tcp              # Specific protocol

# Allow port range
sudo ufw allow 3000:3100/tcp

# Allow from specific IP
sudo ufw allow from 192.168.1.100
sudo ufw allow from 192.168.1.100 to any port 22
sudo ufw allow from 10.0.0.0/24 to any port 5432

# Deny
sudo ufw deny 3306                   # Block MySQL
sudo ufw deny from 203.0.113.50      # Block an IP

# Delete rules
sudo ufw delete allow 80
sudo ufw delete 3                    # Delete by number

# Reset (remove all rules)
sudo ufw reset

# Application profiles
sudo ufw app list
sudo ufw allow 'Nginx Full'         # Allow HTTP + HTTPS
sudo ufw allow 'OpenSSH'
```

### Common DevOps UFW Setup

```bash
#!/bin/bash
# ufw-setup.sh — Standard web server firewall

sudo ufw default deny incoming
sudo ufw default allow outgoing

# SSH (restrict to office IP)
sudo ufw allow from 203.0.113.0/24 to any port 22

# Web traffic
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Application port (internal only)
sudo ufw allow from 10.0.0.0/8 to any port 8080

# Monitoring (Prometheus scrape)
sudo ufw allow from 10.0.1.50 to any port 9100

sudo ufw enable
sudo ufw status verbose
```

---

## 5.3 `firewalld` — RHEL/CentOS/Fedora

```bash
# Start and enable
sudo systemctl enable --now firewalld
sudo firewall-cmd --state

# Zones
sudo firewall-cmd --get-zones
sudo firewall-cmd --get-default-zone
sudo firewall-cmd --get-active-zones

# List rules
sudo firewall-cmd --list-all
sudo firewall-cmd --zone=public --list-all

# Add ports (runtime)
sudo firewall-cmd --add-port=80/tcp
sudo firewall-cmd --add-port=443/tcp
sudo firewall-cmd --add-port=3000-3100/tcp

# Add ports (permanent)
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload                         # Apply permanent rules

# Add services (predefined)
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --permanent --add-service=ssh
sudo firewall-cmd --reload

# Allow from specific IP
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.0/24" port port="5432" protocol="tcp" accept'
sudo firewall-cmd --reload

# Remove
sudo firewall-cmd --permanent --remove-port=8080/tcp
sudo firewall-cmd --permanent --remove-service=http

# List services available
sudo firewall-cmd --get-services
```

```
┌──────────────────────────────────────────────────────────────┐
│           FIREWALLD ZONES                                     │
├──────────────┬───────────────────────────────────────────────┤
│ Zone         │ Default Behavior                              │
├──────────────┼───────────────────────────────────────────────┤
│ drop         │ Drop all incoming, no reply                   │
│ block        │ Reject all incoming with ICMP message         │
│ public       │ Default. Only selected services allowed       │
│ external     │ External networks with masquerading           │
│ dmz          │ For DMZ servers, limited access               │
│ work         │ Trusted work networks                         │
│ home         │ Home network, more trust                      │
│ internal     │ Internal network                              │
│ trusted      │ All traffic accepted                          │
└──────────────┴───────────────────────────────────────────────┘
```

---

## 5.4 `iptables` — Low-Level Firewall

```bash
# List rules
sudo iptables -L -n -v                # Verbose, numeric
sudo iptables -L -n --line-numbers     # With line numbers
sudo iptables -S                       # Show as commands

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Drop everything else
sudo iptables -A INPUT -j DROP

# Delete a rule by line number
sudo iptables -D INPUT 3

# Flush all rules (CAREFUL!)
sudo iptables -F

# Save rules (persist across reboot)
sudo iptables-save > /etc/iptables/rules.v4       # Debian/Ubuntu
sudo iptables-save > /etc/sysconfig/iptables       # RHEL/CentOS

# Restore
sudo iptables-restore < /etc/iptables/rules.v4
```

```
┌──────────────────────────────────────────────────────────────┐
│              IPTABLES CHAIN FLOW                              │
│                                                                │
│  Incoming Packet                                               │
│       │                                                        │
│       ▼                                                        │
│  ┌──────────┐    Is this      ┌───────────┐                  │
│  │PREROUTING│───→ for us? ───→│  INPUT    │───→ Local Process│
│  └──────────┘    Yes          └───────────┘                   │
│       │                                                        │
│       │ No (forwarding)                                        │
│       ▼                                                        │
│  ┌──────────┐                 ┌───────────┐                   │
│  │ FORWARD  │────────────────→│POSTROUTING│───→ Out           │
│  └──────────┘                 └───────────┘                   │
│                                    ▲                           │
│  Local Process ──→ ┌──────────┐   │                           │
│                    │  OUTPUT  │───┘                            │
│                    └──────────┘                                │
│                                                                │
│  Actions:  ACCEPT | DROP | REJECT | LOG                       │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.5 Real-World Scenario: Docker + Firewall

```bash
# IMPORTANT: Docker bypasses ufw/firewalld by default!
# Docker inserts its own iptables rules in the FORWARD chain

# Problem: You block port 3306 with ufw, but Docker
# container on port 3306 is still accessible externally!

# Solution 1: Bind Docker to localhost only
# docker run -p 127.0.0.1:3306:3306 mysql

# Solution 2: Configure Docker daemon
# /etc/docker/daemon.json
{
    "iptables": false
}
# Then manage FORWARD rules manually

# Solution 3: Use DOCKER-USER chain
sudo iptables -I DOCKER-USER -i eth0 -p tcp --dport 3306 -j DROP
sudo iptables -I DOCKER-USER -i eth0 -s 10.0.0.0/24 -p tcp --dport 3306 -j ACCEPT

# Verify
sudo iptables -L DOCKER-USER -n -v
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Port blocked but service running | `ss -tlnp` shows listening | Check firewall rules: `ufw status` |
| UFW enabled but Docker port exposed | Docker bypasses ufw | Use `127.0.0.1:PORT` binding or DOCKER-USER chain |
| `firewall-cmd` changes lost on reboot | Used runtime not permanent | Add `--permanent` flag then `--reload` |
| Locked out of SSH | Firewall blocked port 22 | Use console access; always allow SSH first |
| Can't connect after enabling firewall | Default deny with no rules | Add required rules BEFORE enabling |
| Rules conflict | Order matters in iptables | First matching rule wins; check with `--line-numbers` |

---

## Summary Table

| Tool | Distro | Key Commands |
|------|--------|-------------|
| `ufw` | Ubuntu/Debian | `ufw allow 80`, `ufw status` |
| `firewalld` | RHEL/CentOS | `firewall-cmd --add-port=80/tcp` |
| `iptables` | All (low-level) | `iptables -A INPUT -p tcp --dport 80 -j ACCEPT` |
| `nftables` | Modern replacement | Successor to iptables |

---

## Quick Revision Questions

1. **What is the relationship between ufw, iptables, and netfilter?**
2. **Why should you always add an SSH allow rule BEFORE enabling a firewall?**
3. **In `firewalld`, what is the difference between runtime and permanent rules?**
4. **How does Docker interact with Linux firewalls? Why is this a security concern?**
5. **What are the three main iptables chains and when is each used?**
6. **Write the UFW commands to set up a web server that only allows SSH from a specific subnet and HTTP/HTTPS from anywhere.**

---

[← Previous: DNS Configuration](04-dns-configuration.md) | [Back to README](../README.md) | [Next: SSH Configuration →](06-ssh-configuration.md)
