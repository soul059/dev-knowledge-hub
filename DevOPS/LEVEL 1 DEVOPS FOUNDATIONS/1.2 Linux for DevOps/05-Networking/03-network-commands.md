# Chapter 3: Network Commands

[← Previous: IP Addressing & Subnetting](02-ip-addressing-subnetting.md) | [Back to README](../README.md) | [Next: DNS Configuration →](04-dns-configuration.md)

---

## Overview

This chapter covers the essential networking commands every DevOps engineer needs: `ping`, `traceroute`, `netstat`/`ss`, `curl`, `wget`, and `ip`. These are your primary tools for testing connectivity, diagnosing latency, and inspecting network state.

---

## 3.1 `ping` — Test Connectivity

```bash
# Basic ping
ping google.com
ping 8.8.8.8

# Limited count
ping -c 4 google.com         # Send 4 packets

# Set interval
ping -i 0.5 google.com       # Every 0.5 seconds

# Set timeout
ping -W 2 google.com         # Wait 2 seconds for response

# Set packet size
ping -s 1500 google.com      # Large packet (MTU test)

# Flood ping (root only — performance testing)
sudo ping -f -c 100 192.168.1.1
```

```
┌──────────────────────────────────────────────────────────────┐
│  $ ping -c 4 google.com                                      │
│  PING google.com (142.250.80.46) 56(84) bytes of data.       │
│  64 bytes from ...: icmp_seq=1 ttl=117 time=12.3 ms          │
│  64 bytes from ...: icmp_seq=2 ttl=117 time=11.8 ms          │
│  64 bytes from ...: icmp_seq=3 ttl=117 time=12.1 ms          │
│  64 bytes from ...: icmp_seq=4 ttl=117 time=11.9 ms          │
│                                                                │
│  --- google.com ping statistics ---                            │
│  4 packets transmitted, 4 received, 0% packet loss            │
│  rtt min/avg/max/mdev = 11.8/12.0/12.3/0.2 ms                │
│                                                                │
│  KEY FIELDS:                                                   │
│  ttl=117      → Time to Live (hops remaining)                 │
│  time=12.3 ms → Round-trip time                               │
│  0% loss      → All packets received                          │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.2 `traceroute` / `tracepath` — Trace Network Path

```bash
# traceroute
traceroute google.com
traceroute -n google.com         # No DNS resolution (faster)
traceroute -T google.com         # Use TCP instead of UDP
traceroute -p 443 google.com     # Specific port

# tracepath (no root needed)
tracepath google.com

# mtr — combines ping + traceroute (real-time)
mtr google.com
mtr -r -c 10 google.com         # Report mode, 10 cycles
```

```
┌──────────────────────────────────────────────────────────────┐
│  $ traceroute google.com                                      │
│                                                                │
│  traceroute to google.com (142.250.80.46), 30 hops max        │
│   1  gateway (192.168.1.1)      1.2 ms  0.9 ms  1.1 ms       │
│   2  isp-router (10.0.0.1)     5.3 ms  4.8 ms  5.1 ms       │
│   3  core-router (72.14.x.x)  10.2 ms  9.8 ms 10.0 ms       │
│   4  * * *                      (no response — filtered)      │
│   5  google-edge (142.250.x.x) 12.1 ms 11.9 ms 12.0 ms      │
│                                                                │
│  Each line = one hop (router)                                  │
│  * * * = router doesn't respond to probes (firewalled)         │
│  Three times shown = three probes per hop                      │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.3 `ss` — Socket Statistics (Modern `netstat`)

```bash
# Listening TCP ports
ss -tlnp
# -t = TCP    -l = listening    -n = numeric    -p = process

# Listening UDP ports
ss -ulnp

# All connections
ss -tanp                     # All TCP connections with process
ss -s                        # Summary statistics

# Filter by port
ss -tlnp | grep :80
ss -tlnp 'sport = :443'

# Filter by state
ss state established
ss state time-wait
ss state listening

# Filter by address
ss dst 192.168.1.100
ss src 10.0.0.5

# Count connections per state
ss -tan | awk '{print $1}' | sort | uniq -c | sort -rn
```

```
┌──────────────────────────────────────────────────────────────┐
│  $ ss -tlnp                                                   │
│  State   Recv-Q  Send-Q  Local Address:Port  Peer  Process    │
│  LISTEN  0       511     0.0.0.0:80         *      nginx      │
│  LISTEN  0       511     0.0.0.0:443        *      nginx      │
│  LISTEN  0       128     0.0.0.0:22         *      sshd       │
│  LISTEN  0       128     127.0.0.1:5432     *      postgres   │
│  LISTEN  0       511     :::8080            *      java       │
│                                                                │
│  BIND ADDRESSES:                                               │
│  0.0.0.0:80    → Listening on ALL interfaces                  │
│  127.0.0.1:5432 → Listening on localhost ONLY                 │
│  :::8080       → Listening on ALL interfaces (IPv6)           │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.4 `netstat` — Legacy Network Statistics

```bash
# Same options as ss (net-tools package)
netstat -tlnp                # Listening TCP with process
netstat -ulnp                # Listening UDP
netstat -an                  # All connections, numeric
netstat -r                   # Routing table (same as `route`)
netstat -i                   # Interface statistics
netstat -s                   # Protocol statistics

# ss vs netstat
# ss is faster (reads directly from kernel)
# netstat is more widely known but deprecated
# Both exist on most systems
```

---

## 3.5 `curl` — Transfer Data (HTTP Client)

```bash
# Basic GET request
curl http://example.com
curl -s http://example.com              # Silent mode
curl -o page.html http://example.com    # Save to file
curl -O http://example.com/file.tar.gz  # Save with original name

# Show headers
curl -I http://example.com              # HEAD request (headers only)
curl -i http://example.com              # Headers + body
curl -v http://example.com              # Verbose (debug)

# POST request
curl -X POST http://api.example.com/data \
  -H "Content-Type: application/json" \
  -d '{"name": "deploy", "env": "prod"}'

# Follow redirects
curl -L http://example.com

# Authentication
curl -u user:password http://api.example.com
curl -H "Authorization: Bearer TOKEN" http://api.example.com

# Download with progress
curl -# -O http://releases.example.com/v2.0.tar.gz

# Health check pattern
curl -sf http://localhost:8080/health && echo "UP" || echo "DOWN"

# Timeout
curl --connect-timeout 5 --max-time 30 http://example.com

# Test HTTPS certificate
curl -vI https://example.com 2>&1 | grep -E "subject|expire|issuer"
```

---

## 3.6 `wget` — Download Files

```bash
# Download a file
wget http://example.com/file.tar.gz

# Download silently
wget -q http://example.com/file.tar.gz

# Save with custom name
wget -O output.tar.gz http://example.com/file.tar.gz

# Resume interrupted download
wget -c http://example.com/large-file.iso

# Download entire directory
wget -r -np http://example.com/docs/

# Mirror a website
wget --mirror --convert-links --page-requisites http://example.com

# Limit speed
wget --limit-rate=1m http://example.com/file.iso

# Background download
wget -b http://example.com/large-file.iso
tail -f wget-log
```

```
┌──────────────────────────────────────────────────────────────┐
│              curl vs wget                                     │
├──────────────────┬─────────────────┬─────────────────────────┤
│ Feature          │ curl            │ wget                    │
├──────────────────┼─────────────────┼─────────────────────────┤
│ Protocols        │ Many (HTTP,FTP, │ HTTP, HTTPS, FTP        │
│                  │ SMTP, IMAP...)  │                         │
│ API testing      │ Excellent       │ Limited                 │
│ File download    │ Good            │ Excellent               │
│ Resume download  │ -C -            │ -c                      │
│ Recursive dl     │ No              │ Yes (-r)                │
│ Output to stdout │ Default         │ File (use -O -)         │
│ Scripting        │ Preferred       │ Good for downloads      │
│ POST/PUT/DELETE  │ Easy            │ Awkward                 │
└──────────────────┴─────────────────┴─────────────────────────┘
```

---

## 3.7 `ip` Command Suite

```bash
# View addresses
ip addr show                     # All
ip -4 addr show                  # IPv4 only
ip addr show dev eth0            # Specific interface

# View routes
ip route show                    # Routing table
ip route get 8.8.8.8             # How to reach an IP

# View neighbors (ARP table)
ip neigh show                    # ARP cache

# View links
ip link show                     # All interfaces
ip -s link show eth0             # With statistics

# Add/delete IP
sudo ip addr add 10.0.0.100/24 dev eth0
sudo ip addr del 10.0.0.100/24 dev eth0

# Add route
sudo ip route add 10.10.0.0/16 via 192.168.1.1
sudo ip route del 10.10.0.0/16
```

---

## 3.8 Real-World Scenario: Diagnosing a Failed Deployment

```bash
#!/bin/bash
# connectivity-check.sh — Debug why an app can't reach its database

DB_HOST="db.internal.company.com"
DB_PORT=5432
APP_PORT=8080

echo "=== Connectivity Diagnosis ==="

# 1. Can we resolve the hostname?
echo -e "\n[1] DNS Resolution"
dig +short "$DB_HOST" && echo "OK" || echo "FAIL: DNS resolution failed"

# 2. Can we reach the IP?
DB_IP=$(dig +short "$DB_HOST" | head -1)
echo -e "\n[2] ICMP Ping to $DB_IP"
ping -c 2 -W 2 "$DB_IP" > /dev/null 2>&1 && echo "OK" || echo "FAIL: Host unreachable"

# 3. Is the port open?
echo -e "\n[3] TCP connection to $DB_HOST:$DB_PORT"
timeout 5 bash -c "echo > /dev/tcp/$DB_IP/$DB_PORT" 2>/dev/null && echo "OK" || echo "FAIL: Port closed/filtered"

# 4. Is our app listening?
echo -e "\n[4] Local app on port $APP_PORT"
ss -tlnp | grep ":$APP_PORT" && echo "OK" || echo "FAIL: App not listening"

# 5. Check routing
echo -e "\n[5] Route to database"
ip route get "$DB_IP"

echo -e "\n=== Done ==="
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| `ping` works but `curl` fails | Port blocked or service down | Check `ss -tlnp`, firewall rules |
| High latency on `ping` | Network congestion or routing | Use `mtr` to find bottleneck hop |
| `curl: connection refused` | Nothing listening on port | Verify service is running and bound correctly |
| `wget` download interrupted | Unstable connection | Resume with `wget -c URL` |
| `ss` shows too many TIME_WAIT | Connection leak | Check application connection pooling |
| Can't resolve hostname | DNS issue | Test `dig`, check `/etc/resolv.conf` |

---

## Summary Table

| Command | Purpose | Common Usage |
|---------|---------|-------------|
| `ping` | Test reachability | `ping -c 4 host` |
| `traceroute` | Trace route to host | `traceroute -n host` |
| `mtr` | Real-time traceroute | `mtr host` |
| `ss` | Socket statistics | `ss -tlnp` |
| `curl` | HTTP client / API testing | `curl -sf URL` |
| `wget` | File download | `wget -c URL` |
| `ip addr` | View/set IP addresses | `ip addr show` |
| `ip route` | View/set routes | `ip route show` |

---

## Quick Revision Questions

1. **What is the difference between `ss` and `netstat`? Which is preferred?**
2. **How do you check which process is listening on port 443?**
3. **What does `curl -sf URL && echo UP` do? When would you use it?**
4. **How do you resume a partially downloaded file with `wget`?**
5. **What do the `* * *` entries in traceroute output mean?**
6. **Write a one-liner to count the number of established TCP connections to your server.**

---

[← Previous: IP Addressing & Subnetting](02-ip-addressing-subnetting.md) | [Back to README](../README.md) | [Next: DNS Configuration →](04-dns-configuration.md)
