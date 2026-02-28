# Chapter 1: Diagnostic Tools

## Overview

Network diagnostic tools are a DevOps engineer's first line of defense when troubleshooting connectivity issues, latency problems, and service outages. Mastering these tools lets you quickly isolate whether a problem is DNS, routing, firewall, application, or infrastructure related.

---

## 1.1 Diagnostic Workflow

```
┌──────────────────────────────────────────────────────────────┐
│         NETWORK TROUBLESHOOTING WORKFLOW                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Step 1: Is it reachable?                                    │
│  ┌────────────────────────────┐                              │
│  │ ping / curl → up or down? │                               │
│  └────────────┬───────────────┘                              │
│               │                                              │
│  Step 2: Where does it break?                                │
│  ┌────────────▼───────────────┐                              │
│  │ traceroute → which hop?    │                              │
│  └────────────┬───────────────┘                              │
│               │                                              │
│  Step 3: Is the port open?                                   │
│  ┌────────────▼───────────────┐                              │
│  │ telnet / nc / nmap → port? │                              │
│  └────────────┬───────────────┘                              │
│               │                                              │
│  Step 4: Is DNS resolving?                                   │
│  ┌────────────▼───────────────┐                              │
│  │ dig / nslookup → correct?  │                              │
│  └────────────┬───────────────┘                              │
│               │                                              │
│  Step 5: What's on the wire?                                 │
│  ┌────────────▼───────────────┐                              │
│  │ tcpdump / ss → packets?    │                              │
│  └────────────┬───────────────┘                              │
│               │                                              │
│  Step 6: Check the application                               │
│  ┌────────────▼───────────────┐                              │
│  │ curl -v / logs → errors?   │                              │
│  └────────────────────────────┘                              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.2 ping — Test Basic Connectivity

```bash
# Basic ping
ping google.com

# Ping with count (stop after 5)
ping -c 5 google.com          # Linux
ping -n 5 google.com          # Windows

# Ping with timeout
ping -W 2 -c 3 10.0.1.50     # 2 second timeout

# Ping with specific packet size (test MTU)
ping -s 1472 -M do google.com  # Will fail if MTU < 1500

# Ping from specific interface
ping -I eth0 10.0.1.50

# Continuous ping with timestamp
ping google.com | while read line; do echo "$(date '+%H:%M:%S') $line"; done
```

### Reading Ping Output

```
PING google.com (142.250.80.46): 56 data bytes
64 bytes from 142.250.80.46: icmp_seq=0 ttl=118 time=12.3 ms
64 bytes from 142.250.80.46: icmp_seq=1 ttl=118 time=11.8 ms
64 bytes from 142.250.80.46: icmp_seq=2 ttl=118 time=14.1 ms

--- google.com ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max/stddev = 11.8/12.7/14.1/0.98 ms
```

| Field | Meaning |
|-------|---------|
| `ttl=118` | Time to live — hops remaining (started at 128 → 10 hops away) |
| `time=12.3 ms` | Round-trip latency |
| `icmp_seq` | Sequence number (gaps = packet loss) |
| `0% packet loss` | All packets returned |

> **Note:** Ping uses ICMP. Some hosts/firewalls block ICMP — ping failure doesn't always mean the host is down.

---

## 1.3 traceroute / tracert — Trace the Path

```bash
# Linux
traceroute google.com

# Windows
tracert google.com

# Use TCP instead of ICMP (bypasses ICMP blocks)
traceroute -T -p 443 google.com

# Use UDP (default on Linux)
traceroute -U google.com

# MTR — combines ping + traceroute (real-time)
mtr google.com

# MTR with report mode (runs 10 probes)
mtr -r -c 10 google.com
```

### Reading Traceroute Output

```
traceroute to google.com (142.250.80.46), 30 hops max
 1  gateway (10.0.0.1)          1.2 ms   0.9 ms   1.1 ms
 2  isp-router (203.0.113.1)    5.4 ms   5.2 ms   5.6 ms
 3  * * *                                                    ← Hop blocks ICMP
 4  core-router (198.51.100.1)  12.1 ms  11.9 ms  12.3 ms
 5  google-edge (142.250.0.1)   11.5 ms  11.4 ms  11.6 ms
 6  google.com (142.250.80.46)  12.0 ms  11.8 ms  12.1 ms
```

| Pattern | Meaning |
|---------|---------|
| `* * *` | Hop doesn't respond (firewall blocks ICMP) |
| Sudden latency jump | Possible congestion or geographic hop |
| Consistent `* * *` at end | Target unreachable |
| High latency at last hop only | Server overloaded (not network) |

---

## 1.4 telnet / nc (netcat) — Test Port Connectivity

```bash
# Test if a port is open (TCP)
telnet example.com 443
# "Connected to..." = port open
# "Connection refused" = port closed
# Timeout = filtered/blocked by firewall

# Netcat — more versatile
nc -zv example.com 443         # -z = scan, -v = verbose
nc -zv -w 3 example.com 443   # -w 3 = 3 second timeout

# Scan port range
nc -zv example.com 80-443

# Test UDP port
nc -zuv example.com 53

# Test from a container
kubectl run test --rm -it --image=busybox -- sh
/ # nc -zv my-service.default.svc.cluster.local 8080

# Quick HTTP test
echo -e "GET / HTTP/1.1\r\nHost: example.com\r\n\r\n" | nc example.com 80
```

---

## 1.5 dig / nslookup — DNS Diagnostics

```bash
# Basic DNS lookup
dig example.com

# Query specific record type
dig example.com A          # IPv4 address
dig example.com AAAA       # IPv6 address
dig example.com MX         # Mail servers
dig example.com TXT        # Text records
dig example.com NS         # Name servers
dig example.com CNAME      # Canonical name

# Short output (IP only)
dig +short example.com

# Query specific DNS server
dig @8.8.8.8 example.com           # Google DNS
dig @1.1.1.1 example.com           # Cloudflare
dig @169.254.169.253 example.com   # AWS VPC DNS

# Trace full resolution path
dig +trace example.com

# Reverse DNS lookup
dig -x 142.250.80.46

# Check TTL
dig example.com | grep -E "^example" | awk '{print $2}'

# nslookup (simpler, cross-platform)
nslookup example.com
nslookup example.com 8.8.8.8     # Use specific DNS server
nslookup -type=MX example.com
```

### Reading dig Output

```
;; ANSWER SECTION:
example.com.        300     IN      A       93.184.216.34
│                   │       │       │       │
│                   │       │       │       └─ IP address
│                   │       │       └─ Record type
│                   │       └─ Class (Internet)
│                   └─ TTL (seconds until cache expires)
└─ Domain name
```

---

## 1.6 ss / netstat — Socket Statistics

```bash
# Show all listening TCP ports
ss -tlnp
# -t = TCP, -l = listening, -n = numeric, -p = show process

# Show all established connections
ss -tnp state established

# Show all listening UDP ports
ss -ulnp

# Filter by port
ss -tlnp | grep :8080
ss -tlnp sport = :443

# Show connections to specific destination
ss -tnp dst 10.0.1.50

# Count connections per state
ss -s

# Legacy: netstat (still works on most systems)
netstat -tlnp    # TCP listeners
netstat -an      # All connections

# Show socket statistics summary
ss -s
# Output:
# TCP:   125 (estab 89, closed 12, orphaned 2, timewait 22)
```

### Connection States

| State | Meaning |
|-------|---------|
| LISTEN | Waiting for incoming connections |
| ESTABLISHED | Active connection |
| TIME_WAIT | Closed, waiting for lingering packets |
| CLOSE_WAIT | Remote closed, local hasn't closed yet (possible leak) |
| SYN_SENT | Connecting (if stuck → firewall/routing issue) |
| FIN_WAIT | Closing connection |

> **Warning:** Many `CLOSE_WAIT` connections = application bug (not closing sockets properly)

---

## 1.7 curl — HTTP Diagnostics

```bash
# Basic request
curl https://example.com

# Verbose mode (see TLS handshake, headers)
curl -vI https://example.com

# Show response headers only
curl -I https://example.com

# Show timing details
curl -o /dev/null -s -w "\
    DNS:        %{time_namelookup}s\n\
    Connect:    %{time_connect}s\n\
    TLS:        %{time_appconnect}s\n\
    FirstByte:  %{time_starttransfer}s\n\
    Total:      %{time_total}s\n\
    HTTP Code:  %{http_code}\n" \
  https://example.com

# Test with specific Host header (before DNS changes)
curl -H "Host: new-app.example.com" https://10.0.1.50/

# Test POST request
curl -X POST https://api.example.com/data \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'

# Follow redirects
curl -L http://example.com

# Test with specific TLS version
curl --tlsv1.3 https://example.com
curl --tlsv1.2 https://example.com

# Download and check HTTP/2
curl --http2 -I https://example.com

# Resolve to specific IP (bypass DNS)
curl --resolve example.com:443:10.0.1.50 https://example.com
```

---

## 1.8 nmap — Network Scanner

```bash
# Scan common ports on a host
nmap example.com

# Scan specific port
nmap -p 443 example.com

# Scan port range
nmap -p 1-1000 example.com

# Scan all ports
nmap -p- example.com

# Service version detection
nmap -sV example.com

# OS detection
nmap -O example.com

# Aggressive scan (version + scripts + OS + traceroute)
nmap -A example.com

# Scan entire subnet
nmap -sn 10.0.1.0/24          # Ping sweep (host discovery)
nmap 10.0.1.0/24              # Port scan whole subnet

# Check SSL/TLS ciphers
nmap --script ssl-enum-ciphers -p 443 example.com

# UDP scan (slower)
nmap -sU -p 53,123,161 example.com

# Scan from inside a Kubernetes pod
kubectl run nmap --rm -it --image=instrumentisto/nmap -- -sV target-service:8080
```

---

## 1.9 Quick Reference — Tool Selection

```
┌──────────────────────────────────────────────────────────────┐
│         WHICH TOOL TO USE?                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  "Is the host alive?"           → ping                       │
│  "Where does the path break?"   → traceroute / mtr           │
│  "Is the port open?"            → nc -zv / telnet            │
│  "Is DNS resolving correctly?"  → dig / nslookup             │
│  "What's listening on server?"  → ss -tlnp                   │
│  "What does the HTTP response   → curl -vI                   │
│   look like?"                                                │
│  "What ports are open?"         → nmap -sV                   │
│  "What's on the wire?"          → tcpdump (next chapter)     │
│  "Show real-time path quality?" → mtr                        │
│  "Test connectivity from pod?"  → kubectl exec + nc/curl     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.10 Troubleshooting Tips

| Problem | Tool | Command |
|---------|------|---------|
| Can't reach server | ping | `ping -c 3 target` |
| Latency spike | mtr | `mtr -r target` |
| Port appears filtered | nc | `nc -zv -w 3 target 443` |
| DNS not resolving | dig | `dig @8.8.8.8 domain.com` |
| Too many connections | ss | `ss -s` / `ss -tnp state established` |
| Slow HTTP response | curl | `curl -w "%{time_total}" -o /dev/null -s url` |
| Unknown open ports | nmap | `nmap -sV target` |
| CLOSE_WAIT accumulation | ss | `ss -tnp state close-wait` — fix app bug |

---

## Summary Table

| Tool | Purpose | Protocol | Key Flags |
|------|---------|----------|-----------|
| ping | Reachability, latency | ICMP | `-c` count, `-s` size |
| traceroute | Path discovery | ICMP/UDP/TCP | `-T` TCP, `-p` port |
| mtr | Real-time path quality | ICMP | `-r` report, `-c` count |
| telnet/nc | Port connectivity | TCP/UDP | `-z` scan, `-v` verbose |
| dig | DNS queries | DNS | `+short`, `+trace`, `@server` |
| ss | Socket stats | TCP/UDP | `-t` TCP, `-l` listen, `-p` process |
| curl | HTTP diagnostics | HTTP/S | `-v` verbose, `-I` headers, `-w` timing |
| nmap | Port scanning | TCP/UDP | `-sV` version, `-p-` all ports |

---

## Quick Revision Questions

1. **A service is unreachable. Walk through the diagnostic steps you'd take, naming the tool at each step.**
2. **What's the difference between `traceroute` and `mtr`?**
3. **How do you test if port 5432 is open on a remote host without PostgreSQL client?**
4. **Write a `curl` command that shows DNS lookup time, TLS handshake time, and total response time.**
5. **You see many `CLOSE_WAIT` connections with `ss`. What does this indicate and how do you fix it?**
6. **How would you test DNS resolution from inside a Kubernetes pod?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Common Vulnerabilities](../06-Network-Security/06-common-vulnerabilities.md) | [README](../README.md) | [Packet Capture & Analysis →](02-packet-capture.md) |
