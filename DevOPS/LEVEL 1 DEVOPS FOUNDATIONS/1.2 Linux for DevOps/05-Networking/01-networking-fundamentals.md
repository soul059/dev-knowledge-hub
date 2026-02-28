# Chapter 1: Networking Fundamentals

[← Previous: Cron Jobs & Scheduling](../04-Process-Management/06-cron-jobs-scheduling.md) | [Back to README](../README.md) | [Next: IP Addressing & Subnetting →](02-ip-addressing-subnetting.md)

---

## Overview

Networking is the backbone of DevOps. Understanding the OSI model, TCP/IP stack, protocols, and ports is fundamental for configuring servers, debugging connectivity issues, managing containers, and deploying cloud infrastructure.

---

## 1.1 OSI Model

```
┌──────────────────────────────────────────────────────────────┐
│                    OSI 7-LAYER MODEL                          │
├───────┬──────────────┬───────────────┬───────────────────────┤
│ Layer │ Name         │ Protocol/Tech │ DevOps Relevance      │
├───────┼──────────────┼───────────────┼───────────────────────┤
│   7   │ Application  │ HTTP, DNS,    │ API endpoints, web    │
│       │              │ SSH, SMTP     │ servers, microservices │
├───────┼──────────────┼───────────────┼───────────────────────┤
│   6   │ Presentation │ SSL/TLS,      │ Certificate mgmt,     │
│       │              │ JPEG, JSON    │ encryption             │
├───────┼──────────────┼───────────────┼───────────────────────┤
│   5   │ Session      │ NetBIOS,      │ Connection management │
│       │              │ RPC           │                        │
├───────┼──────────────┼───────────────┼───────────────────────┤
│   4   │ Transport    │ TCP, UDP      │ Port mapping, load    │
│       │              │               │ balancers, NAT        │
├───────┼──────────────┼───────────────┼───────────────────────┤
│   3   │ Network      │ IP, ICMP,     │ Routing, subnetting,  │
│       │              │ ARP           │ VPC, overlay networks  │
├───────┼──────────────┼───────────────┼───────────────────────┤
│   2   │ Data Link    │ Ethernet,     │ VLANs, MAC filtering, │
│       │              │ Wi-Fi, ARP    │ bridging               │
├───────┼──────────────┼───────────────┼───────────────────────┤
│   1   │ Physical     │ Cables, NICs, │ Data center, hardware │
│       │              │ Hubs          │                        │
└───────┴──────────────┴───────────────┴───────────────────────┘
```

---

## 1.2 TCP/IP Model (Practical Model)

```
┌──────────────────────────────────────────────────────────────┐
│           TCP/IP vs OSI MAPPING                               │
│                                                                │
│  OSI Layers          TCP/IP Layers       Examples              │
│  ┌────────────┐      ┌──────────────┐                         │
│  │ Application│      │              │   HTTP, SSH, DNS        │
│  │ Presentatn │ ───→ │ Application  │   FTP, SMTP, SNMP      │
│  │ Session    │      │              │                         │
│  ├────────────┤      ├──────────────┤                         │
│  │ Transport  │ ───→ │ Transport    │   TCP, UDP              │
│  ├────────────┤      ├──────────────┤                         │
│  │ Network    │ ───→ │ Internet     │   IP, ICMP, ARP        │
│  ├────────────┤      ├──────────────┤                         │
│  │ Data Link  │ ───→ │ Network      │   Ethernet, Wi-Fi      │
│  │ Physical   │      │ Access       │   Drivers, NICs        │
│  └────────────┘      └──────────────┘                         │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.3 TCP vs UDP

```
┌──────────────────────────────────────────────────────────────┐
│                  TCP vs UDP COMPARISON                        │
├────────────────────┬──────────────────┬──────────────────────┤
│ Feature            │ TCP              │ UDP                  │
├────────────────────┼──────────────────┼──────────────────────┤
│ Connection         │ Connection-based │ Connectionless       │
│ Reliability        │ Guaranteed       │ Best-effort          │
│ Ordering           │ Ordered          │ No ordering          │
│ Speed              │ Slower           │ Faster               │
│ Overhead           │ Higher (headers) │ Lower                │
│ Use cases          │ HTTP, SSH, FTP,  │ DNS, DHCP, streaming,│
│                    │ SMTP, databases  │ VoIP, gaming         │
│ Error checking     │ Yes + retransmit │ Checksum only        │
│ Flow control       │ Yes              │ No                   │
└────────────────────┴──────────────────┴──────────────────────┘

  TCP Three-Way Handshake:
  
  Client                    Server
    │                          │
    │──── SYN ────────────────→│
    │                          │
    │←── SYN-ACK ─────────────│
    │                          │
    │──── ACK ────────────────→│
    │                          │
    │     Connection Open      │
    │     (data exchange)      │
    │                          │
    │──── FIN ────────────────→│  (close)
    │←── ACK ─────────────────│
    │←── FIN ─────────────────│
    │──── ACK ────────────────→│
```

---

## 1.4 Common Ports

```
┌──────────────────────────────────────────────────────────────┐
│              ESSENTIAL PORTS FOR DEVOPS                       │
├───────┬───────────┬──────────────────────────────────────────┤
│ Port  │ Protocol  │ Service                                  │
├───────┼───────────┼──────────────────────────────────────────┤
│ 22    │ TCP       │ SSH (remote access)                      │
│ 53    │ TCP/UDP   │ DNS (name resolution)                    │
│ 80    │ TCP       │ HTTP (web traffic)                       │
│ 443   │ TCP       │ HTTPS (encrypted web)                    │
│ 25    │ TCP       │ SMTP (email sending)                     │
│ 587   │ TCP       │ SMTP (submission, TLS)                   │
│ 110   │ TCP       │ POP3 (email retrieval)                   │
│ 143   │ TCP       │ IMAP (email retrieval)                   │
│ 3306  │ TCP       │ MySQL / MariaDB                          │
│ 5432  │ TCP       │ PostgreSQL                               │
│ 6379  │ TCP       │ Redis                                    │
│ 27017 │ TCP       │ MongoDB                                  │
│ 8080  │ TCP       │ HTTP alt (Tomcat, Jenkins)               │
│ 8443  │ TCP       │ HTTPS alt                                │
│ 9090  │ TCP       │ Prometheus                               │
│ 3000  │ TCP       │ Grafana                                  │
│ 2375  │ TCP       │ Docker (unencrypted)                     │
│ 2376  │ TCP       │ Docker (TLS)                             │
│ 6443  │ TCP       │ Kubernetes API Server                    │
│ 10250 │ TCP       │ Kubelet                                  │
│ 2379  │ TCP       │ etcd (client)                            │
│ 9100  │ TCP       │ Node Exporter                            │
└───────┴───────────┴──────────────────────────────────────────┘

  Port Ranges:
  0 - 1023      Well-known (privileged, require root)
  1024 - 49151  Registered (applications)
  49152 - 65535 Dynamic / Ephemeral (temporary connections)
```

---

## 1.5 Network Interfaces in Linux

```bash
# View interfaces
ip link show
ip addr show                 # IP addresses
ifconfig                     # Legacy (net-tools)

# Interface types
# eth0 / ens33    → Physical Ethernet
# wlan0 / wlp2s0  → Wireless
# lo               → Loopback (127.0.0.1)
# docker0          → Docker bridge
# veth*            → Container virtual interfaces
# br-*             → Custom bridge networks
# virbr0           → KVM/libvirt bridge

# Bring interface up/down
sudo ip link set eth0 up
sudo ip link set eth0 down
```

```
┌──────────────────────────────────────────────────────────────┐
│             NETWORK INTERFACE NAMING (systemd)                │
│                                                                │
│  Predictable naming scheme:                                   │
│  en  = Ethernet           wl  = Wireless                     │
│                                                                │
│  Naming rules:                                                │
│  ens33  → Ethernet, slot 33 (hotplug)                        │
│  enp0s3 → Ethernet, PCI bus 0, slot 3                        │
│  eno1   → Ethernet, on-board port 1                          │
│  wlp2s0 → Wireless, PCI bus 2, slot 0                        │
│                                                                │
│  Legacy names: eth0, eth1, wlan0                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.6 DNS Basics

```
┌──────────────────────────────────────────────────────────────┐
│                DNS RESOLUTION FLOW                            │
│                                                                │
│  Application                                                   │
│    │                                                           │
│    ▼                                                           │
│  /etc/hosts        ← Check local hosts file first             │
│    │ (not found)                                               │
│    ▼                                                           │
│  /etc/resolv.conf  ← DNS server configured here               │
│    │                                                           │
│    ▼                                                           │
│  Local DNS Cache   ← systemd-resolved / nscd                  │
│    │ (cache miss)                                              │
│    ▼                                                           │
│  Recursive DNS     ← ISP / 8.8.8.8 / 1.1.1.1                │
│    │                                                           │
│    ▼                                                           │
│  Root → TLD → Authoritative DNS                               │
│  .    → .com → example.com                                    │
│                → A record: 93.184.216.34                       │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Key DNS files
cat /etc/hosts               # Local name resolution
cat /etc/resolv.conf         # DNS server config
cat /etc/nsswitch.conf       # Resolution order

# DNS lookup
nslookup google.com
dig google.com
dig +short google.com        # Just the IP
host google.com

# DNS record types
dig google.com A             # IPv4 address
dig google.com AAAA          # IPv6 address
dig google.com MX            # Mail server
dig google.com NS            # Name server
dig google.com CNAME         # Alias
dig google.com TXT           # Text records (SPF, DKIM)
```

---

## 1.7 Real-World Scenario: Verifying Network Stack on a New Server

```bash
#!/bin/bash
# network-audit.sh — Quick network check for a new server

echo "=== Network Audit ==="

echo -e "\n--- Interfaces ---"
ip -br addr show

echo -e "\n--- Default Gateway ---"
ip route | grep default

echo -e "\n--- DNS Servers ---"
cat /etc/resolv.conf | grep nameserver

echo -e "\n--- DNS Resolution Test ---"
dig +short google.com && echo "DNS: OK" || echo "DNS: FAIL"

echo -e "\n--- Internet Connectivity ---"
ping -c 2 8.8.8.8 > /dev/null 2>&1 && echo "Internet: OK" || echo "Internet: FAIL"

echo -e "\n--- Listening Ports ---"
ss -tlnp | head -15

echo -e "\n--- Active Connections ---"
ss -tnp | head -10

echo "=== Audit Complete ==="
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| No network connectivity | `ip addr show` — no IP | Check DHCP or static config |
| DNS not resolving | `ping 8.8.8.8` works, `ping google.com` fails | Fix `/etc/resolv.conf` |
| Port not reachable | `ss -tlnp` — not listening | Start service, check bind address |
| Connection refused | Service down or firewall | Check service status, firewall rules |
| Slow DNS | `dig` shows high query time | Change DNS server to 8.8.8.8 or 1.1.1.1 |
| Interface down | `ip link show` — state DOWN | `sudo ip link set dev ethX up` |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| OSI Model | 7 layers: Physical → Application |
| TCP/IP | 4 layers; practical model used in Linux |
| TCP | Reliable, ordered, connection-based |
| UDP | Fast, unreliable, connectionless |
| Common Ports | SSH=22, HTTP=80, HTTPS=443, DNS=53 |
| DNS | Resolves names → IPs; `/etc/resolv.conf` |
| Interfaces | `ip addr show`; predictable naming |

---

## Quick Revision Questions

1. **Name the 7 layers of the OSI model from bottom to top.**
2. **What is the difference between TCP and UDP? Give two use cases for each.**
3. **What port does SSH use by default? What about Kubernetes API Server?**
4. **Describe the DNS resolution order in Linux (which files are checked first).**
5. **What is the TCP three-way handshake? Draw the sequence.**
6. **What does `/etc/resolv.conf` contain and how is it used?**

---

[← Previous: Cron Jobs & Scheduling](../04-Process-Management/06-cron-jobs-scheduling.md) | [Back to README](../README.md) | [Next: IP Addressing & Subnetting →](02-ip-addressing-subnetting.md)
