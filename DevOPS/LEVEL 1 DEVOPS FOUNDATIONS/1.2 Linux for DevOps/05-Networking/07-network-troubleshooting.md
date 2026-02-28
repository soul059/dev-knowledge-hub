# Chapter 7: Network Troubleshooting

[← Previous: SSH Configuration](06-ssh-configuration.md) | [Back to README](../README.md) | [Next: Partitioning & Disk Management →](../06-Storage-Management/01-partitioning-disk-management.md)

---

## Overview

Network troubleshooting is a critical DevOps skill. This chapter provides a systematic approach to diagnosing connectivity issues using a layered methodology — from physical/link layer checks all the way up to application-layer debugging.

---

## 7.1 Troubleshooting Methodology

```
┌──────────────────────────────────────────────────────────────┐
│         NETWORK TROUBLESHOOTING LAYERS                        │
│         (Bottom-Up Approach)                                  │
│                                                                │
│  Layer 7: Application                                         │
│  └─ curl, wget, application logs                              │
│       "Is the app responding correctly?"                      │
│                                                                │
│  Layer 4: Transport                                           │
│  └─ ss, netstat, telnet, nc                                   │
│       "Is the port open and accepting connections?"           │
│                                                                │
│  Layer 3: Network                                             │
│  └─ ping, traceroute, ip route                                │
│       "Can I reach the IP? Is routing correct?"               │
│                                                                │
│  Layer 2: Data Link                                           │
│  └─ ip link, ethtool, arp                                     │
│       "Is the interface up? Is there a link?"                 │
│                                                                │
│  Layer 1: Physical                                            │
│  └─ Cable, NIC, switch port                                   │
│       "Is the hardware connected?"                            │
│                                                                │
│  ALSO CHECK:                                                  │
│  • DNS resolution (dig, nslookup)                             │
│  • Firewall rules (ufw, iptables)                             │
│  • SELinux / AppArmor                                         │
└──────────────────────────────────────────────────────────────┘
```

---

## 7.2 Systematic Diagnosis Steps

```bash
# Step 1: Check interface status
ip link show
ip addr show
# Is the interface UP? Does it have an IP?

# Step 2: Check local connectivity
ping -c 2 127.0.0.1              # Loopback
ping -c 2 $(hostname -I | awk '{print $1}')   # Own IP

# Step 3: Check gateway
ip route show                     # Find default gateway
ping -c 2 192.168.1.1            # Ping gateway

# Step 4: Check external connectivity
ping -c 2 8.8.8.8               # External IP (bypasses DNS)

# Step 5: Check DNS
dig +short google.com            # DNS resolution
ping -c 2 google.com             # Name + network

# Step 6: Check specific port
ss -tlnp | grep :80              # Is service listening?
curl -v http://target:80         # Can we connect?

# Step 7: Check firewall
sudo iptables -L -n              # iptables rules
sudo ufw status                  # ufw (Ubuntu)
sudo firewall-cmd --list-all     # firewalld (RHEL)
```

---

## 7.3 Essential Diagnostic Tools

### `nc` (netcat) — Network Swiss Army Knife

```bash
# Test if a port is open
nc -zv hostname 80               # TCP port check
nc -zv hostname 22               # SSH port check
nc -zv hostname 3000-3100        # Port range scan
nc -zvu hostname 53              # UDP port check

# Listen on a port (for testing)
nc -l 8080                       # Listen on port 8080
# From another terminal:
echo "test" | nc localhost 8080

# Transfer a file
# Receiver: nc -l 9090 > received_file
# Sender:   nc hostname 9090 < file_to_send
```

### `tcpdump` — Packet Capture

```bash
# Capture all traffic on interface
sudo tcpdump -i eth0

# Filter by host
sudo tcpdump -i eth0 host 192.168.1.100

# Filter by port
sudo tcpdump -i eth0 port 80
sudo tcpdump -i eth0 port 443

# Filter by protocol
sudo tcpdump -i eth0 tcp
sudo tcpdump -i eth0 icmp

# Save to file (for Wireshark)
sudo tcpdump -i eth0 -w capture.pcap

# Readable output
sudo tcpdump -i eth0 -nn -A port 80    # ASCII
sudo tcpdump -i eth0 -nn -X port 80    # Hex + ASCII

# Common filters
sudo tcpdump -i eth0 'src 10.0.0.5 and dst port 443'
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'    # SYN packets
```

### `nmap` — Network Scanner

```bash
# Install
sudo apt install nmap       # Ubuntu
sudo dnf install nmap       # RHEL

# Scan open ports
nmap hostname
nmap -p 80,443,8080 hostname        # Specific ports
nmap -p 1-1024 hostname             # Port range
nmap -sV hostname                   # Service version detection
nmap -O hostname                    # OS detection

# Quick scan of a subnet
nmap -sn 192.168.1.0/24             # Ping scan (host discovery)
nmap -sT 192.168.1.0/24 -p 22      # Who has SSH open?
```

---

## 7.4 Common Scenarios & Solutions

```
┌──────────────────────────────────────────────────────────────┐
│        SCENARIO 1: "Website is down"                          │
│                                                                │
│  $ curl -I https://myapp.com                                  │
│  curl: (7) Failed to connect                                  │
│                                                                │
│  Debug path:                                                   │
│  1. DNS:     dig myapp.com          → IP resolves? ✓          │
│  2. Network: ping 93.184.216.34     → Host reachable? ✓       │
│  3. Port:    nc -zv 93.184 443      → Port open? ✗ BLOCKED   │
│  4. Server:  ssh server; ss -tlnp   → Nginx running? ✗       │
│  5. Fix:     systemctl start nginx                            │
├──────────────────────────────────────────────────────────────┤
│        SCENARIO 2: "Can't SSH to server"                      │
│                                                                │
│  $ ssh user@server                                            │
│  ssh: connect to host: Connection timed out                   │
│                                                                │
│  Debug path:                                                   │
│  1. Ping:    ping server            → Timeout ✗              │
│  2. Route:   traceroute server      → Dies at hop 3          │
│  3. Firewall: Check security group / ufw on server           │
│  4. Fix:     Open port 22 in firewall                        │
├──────────────────────────────────────────────────────────────┤
│        SCENARIO 3: "App can't connect to database"            │
│                                                                │
│  Debug path:                                                   │
│  1. DNS:     dig db.internal        → Resolves? ✓            │
│  2. Port:    nc -zv db.internal 5432 → Open? ✗               │
│  3. On DB:   ss -tlnp | grep 5432   → Bound to 127.0.0.1!  │
│  4. Fix:     Change listen_addresses = '*' in postgresql.conf│
│              + allow in pg_hba.conf                           │
│              + open firewall port 5432                        │
└──────────────────────────────────────────────────────────────┘
```

---

## 7.5 Real-World Scenario: Full Stack Connectivity Debug Script

```bash
#!/bin/bash
# network-debug.sh — Comprehensive connectivity diagnosis
# Usage: ./network-debug.sh <target_host> <target_port>

TARGET_HOST=${1:-"google.com"}
TARGET_PORT=${2:-"443"}

RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'

pass() { echo -e "${GREEN}[PASS]${NC} $1"; }
fail() { echo -e "${RED}[FAIL]${NC} $1"; }

echo "=== Network Diagnosis: ${TARGET_HOST}:${TARGET_PORT} ==="

# 1. Interface check
echo -e "\n--- Interface Status ---"
ip -br addr show | grep UP && pass "Interface UP" || fail "No active interface"

# 2. Gateway
echo -e "\n--- Default Gateway ---"
GW=$(ip route | grep default | awk '{print $3}')
if [ -n "$GW" ]; then
    ping -c 1 -W 2 "$GW" > /dev/null 2>&1 && pass "Gateway $GW reachable" || fail "Gateway $GW unreachable"
else
    fail "No default gateway"
fi

# 3. External connectivity
echo -e "\n--- External Connectivity ---"
ping -c 1 -W 2 8.8.8.8 > /dev/null 2>&1 && pass "Internet reachable" || fail "Internet unreachable"

# 4. DNS
echo -e "\n--- DNS Resolution ---"
RESOLVED_IP=$(dig +short "$TARGET_HOST" | head -1)
if [ -n "$RESOLVED_IP" ]; then
    pass "DNS: $TARGET_HOST → $RESOLVED_IP"
else
    fail "DNS: Cannot resolve $TARGET_HOST"
fi

# 5. ICMP
echo -e "\n--- ICMP Ping ---"
ping -c 2 -W 2 "$TARGET_HOST" > /dev/null 2>&1 && pass "Host responds to ping" || fail "No ping response (may be filtered)"

# 6. TCP Port
echo -e "\n--- TCP Port $TARGET_PORT ---"
nc -zv -w 3 "$TARGET_HOST" "$TARGET_PORT" 2>&1 && pass "Port $TARGET_PORT open" || fail "Port $TARGET_PORT closed/filtered"

# 7. Route
echo -e "\n--- Route to $TARGET_HOST ---"
ip route get "$RESOLVED_IP" 2>/dev/null | head -1

echo -e "\n=== Diagnosis Complete ==="
```

---

## Troubleshooting Tips

| Problem | Tool | Command |
|---------|------|---------|
| Is the interface up? | `ip` | `ip link show` |
| Does it have an IP? | `ip` | `ip addr show` |
| Can I reach gateway? | `ping` | `ping -c 2 GATEWAY` |
| Can I reach external IP? | `ping` | `ping -c 2 8.8.8.8` |
| Is DNS working? | `dig` | `dig +short domain` |
| Is the port open? | `nc` | `nc -zv host port` |
| What's the route? | `ip` | `ip route get IP` |
| Where does traffic stop? | `traceroute` | `traceroute host` |
| What's on the wire? | `tcpdump` | `tcpdump -i eth0 port 80` |
| Is firewall blocking? | `iptables` | `iptables -L -n` |

---

## Summary Table

| Tool | Layer | Purpose |
|------|-------|---------|
| `ip link` | L2 | Interface status |
| `ping` | L3 | ICMP reachability |
| `traceroute` | L3 | Path tracing |
| `mtr` | L3 | Continuous traceroute |
| `nc` | L4 | Port testing |
| `ss/netstat` | L4 | Socket state |
| `tcpdump` | L2-L7 | Packet capture |
| `nmap` | L3-L4 | Port scanning |
| `curl` | L7 | HTTP testing |
| `dig` | L7 | DNS queries |

---

## Quick Revision Questions

1. **Describe the bottom-up approach to network troubleshooting. What do you check at each layer?**
2. **`ping 8.8.8.8` works but `ping google.com` fails. What is the problem?**
3. **How do you check if a specific TCP port is open on a remote host?**
4. **What is `tcpdump` used for? Write a command to capture HTTP traffic.**
5. **A web server is running but external users can't access it. List all possible causes.**
6. **Write a quick diagnostic sequence (5 commands) to troubleshoot "cannot reach database".**

---

[← Previous: SSH Configuration](06-ssh-configuration.md) | [Back to README](../README.md) | [Next: Partitioning & Disk Management →](../06-Storage-Management/01-partitioning-disk-management.md)
