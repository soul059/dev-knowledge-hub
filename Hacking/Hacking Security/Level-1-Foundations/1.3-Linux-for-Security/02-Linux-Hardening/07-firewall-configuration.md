# Firewall Configuration (iptables, nftables)

## Unit 2 - Topic 7 | Linux Hardening

---

## Overview

Linux's built-in firewall framework **Netfilter** (kernel-space) is managed through user-space tools: **iptables** (legacy but widely used) and **nftables** (modern replacement). Understanding Linux firewall configuration is essential for hardening servers and controlling network traffic.

---

## 1. Netfilter Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              NETFILTER / IPTABLES CHAIN FLOW                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  INCOMING PACKET                                             │
│       │                                                      │
│       ▼                                                      │
│  ┌─────────────┐                                            │
│  │ PREROUTING  │ (NAT, mangle)                              │
│  └──────┬──────┘                                            │
│         │                                                    │
│    ┌────┴────┐                                               │
│    │ Route?  │                                               │
│    └────┬────┘                                               │
│    For me │     Forward                                      │
│         │      │                                             │
│    ┌────┴────┐ ┌────┴─────┐                                 │
│    │  INPUT  │ │ FORWARD  │                                 │
│    └────┬────┘ └────┬─────┘                                 │
│         │           │                                        │
│    Local Process    │                                        │
│         │           │                                        │
│    ┌────┴────┐      │                                        │
│    │ OUTPUT  │      │                                        │
│    └────┬────┘      │                                        │
│         │           │                                        │
│    ┌────┴───────────┴──┐                                    │
│    │   POSTROUTING     │                                    │
│    └────────┬──────────┘                                    │
│             │                                                │
│       OUTGOING PACKET                                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. iptables Essentials

```bash
# === VIEW RULES ===
iptables -L -n -v                    # List all rules (numeric, verbose)
iptables -L -n --line-numbers        # With line numbers

# === DEFAULT POLICY (Default Deny!) ===
iptables -P INPUT DROP               # Drop all incoming by default
iptables -P FORWARD DROP             # Drop all forwarded
iptables -P OUTPUT ACCEPT            # Allow all outgoing

# === ALLOW ESTABLISHED CONNECTIONS ===
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# === ALLOW LOOPBACK ===
iptables -A INPUT -i lo -j ACCEPT

# === ALLOW SSH ===
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/24 -j ACCEPT

# === ALLOW WEB ===
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# === ALLOW ICMP (Ping) ===
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# === RATE LIMITING (Brute Force Prevention) ===
iptables -A INPUT -p tcp --dport 22 -m state --state NEW \
    -m recent --set --name SSH
iptables -A INPUT -p tcp --dport 22 -m state --state NEW \
    -m recent --update --seconds 60 --hitcount 4 --name SSH -j DROP

# === LOG DROPPED PACKETS ===
iptables -A INPUT -j LOG --log-prefix "IPTABLES-DROP: " --log-level 4
iptables -A INPUT -j DROP            # Final drop rule

# === DELETE RULES ===
iptables -D INPUT 3                  # Delete rule #3
iptables -F                          # Flush all rules

# === SAVE RULES ===
# Debian/Ubuntu:
apt install iptables-persistent
iptables-save > /etc/iptables/rules.v4
# RHEL/CentOS:
service iptables save
```

---

## 3. Complete Hardened iptables Ruleset

```bash
#!/bin/bash
# Hardened iptables firewall script

# Flush existing rules
iptables -F
iptables -X
iptables -t nat -F

# Default policies — DROP everything
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allow SSH from admin network only
iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/24 -j ACCEPT

# Allow HTTP/HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow ICMP (limited)
iptables -A INPUT -p icmp --icmp-type echo-request \
    -m limit --limit 1/s --limit-burst 4 -j ACCEPT

# Log and drop everything else
iptables -A INPUT -j LOG --log-prefix "DROPPED: "
iptables -A INPUT -j DROP
```

---

## 4. nftables — Modern Replacement

```bash
# nftables replaces iptables, ip6tables, arptables, ebtables
# Uses nft command and a cleaner syntax

# === VIEW RULES ===
nft list ruleset

# === CREATE TABLE AND CHAIN ===
nft add table inet filter
nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
nft add chain inet filter forward { type filter hook forward priority 0 \; policy drop \; }
nft add chain inet filter output { type filter hook output priority 0 \; policy accept \; }

# === ADD RULES ===
# Allow established
nft add rule inet filter input ct state established,related accept
# Allow loopback
nft add rule inet filter input iifname lo accept
# Allow SSH
nft add rule inet filter input tcp dport 22 accept
# Allow HTTP/HTTPS
nft add rule inet filter input tcp dport { 80, 443 } accept
# Log and drop
nft add rule inet filter input log prefix \"DROPPED: \" drop

# === SAVE RULES ===
nft list ruleset > /etc/nftables.conf
systemctl enable nftables
```

---

## 5. UFW — Simplified Firewall

```bash
# UFW (Uncomplicated Firewall) — frontend for iptables

# Enable UFW
ufw enable
ufw default deny incoming
ufw default allow outgoing

# Allow services
ufw allow ssh                        # Port 22
ufw allow 80/tcp                     # HTTP
ufw allow 443/tcp                    # HTTPS
ufw allow from 10.0.0.0/24 to any port 22  # SSH from specific network

# View status
ufw status verbose

# Delete rule
ufw delete allow 80/tcp
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Netfilter** | Kernel-space packet filtering framework |
| **iptables** | Legacy user-space tool (still widely used) |
| **nftables** | Modern replacement for iptables |
| **UFW** | Simplified frontend for iptables |
| **Default Deny** | DROP everything, then explicitly allow |
| **Persistence** | `iptables-save` / `nft list ruleset > file` |

---

## Quick Revision Questions

1. **What are the three default chains in iptables?**
2. **Why should the default policy be DROP?**
3. **How do you persist iptables rules across reboots?**
4. **What is the difference between iptables and nftables?**
5. **Write an iptables rule to allow SSH from 10.0.0.0/24 only.**

---

[← Previous: SSH Hardening](06-ssh-hardening.md)

---

*Unit 2 - Topic 7 of 7 | [Back to README](../README.md) | [Next Unit: Linux Logging and Auditing →](../03-Linux-Logging-and-Auditing/01-syslog-and-journald.md)*
