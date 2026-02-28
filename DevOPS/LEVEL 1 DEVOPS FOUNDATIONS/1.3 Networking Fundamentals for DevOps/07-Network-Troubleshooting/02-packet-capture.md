# Chapter 2: Packet Capture & Analysis

## Overview

Packet capture is the deepest level of network troubleshooting — it lets you see exactly what's happening on the wire. When logs don't tell the full story, **tcpdump** and **Wireshark** reveal the actual packets being sent and received. This skill is invaluable for debugging TLS handshake failures, connection resets, DNS issues, and performance problems.

---

## 2.1 When to Use Packet Capture

```
┌──────────────────────────────────────────────────────────────┐
│         WHEN DO YOU NEED PACKET CAPTURE?                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Connection timeouts with no error logs                   │
│     → See if SYN packets are being sent/received             │
│                                                              │
│  2. TLS handshake failures                                   │
│     → See ClientHello, ServerHello, certificate exchange     │
│                                                              │
│  3. Intermittent connection resets (RST)                      │
│     → Identify which side sends RST and why                  │
│                                                              │
│  4. DNS resolution issues                                    │
│     → Verify queries are reaching DNS server                 │
│                                                              │
│  5. Performance problems (slow responses)                    │
│     → Measure actual network latency vs application time     │
│                                                              │
│  6. Firewall/NACL debugging                                  │
│     → Confirm packets arrive but responses are blocked       │
│                                                              │
│  7. "It works on my machine" scenarios                       │
│     → Compare packet captures from different environments    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.2 tcpdump — Command-Line Packet Capture

### Basic Usage

```bash
# Capture all traffic on default interface
sudo tcpdump

# Capture on specific interface
sudo tcpdump -i eth0

# Capture with readable output
sudo tcpdump -i eth0 -nn     # -nn = don't resolve hostnames or ports

# Capture specific number of packets
sudo tcpdump -c 100 -i eth0

# Save capture to file (for Wireshark analysis)
sudo tcpdump -i eth0 -w capture.pcap

# Read saved capture file
tcpdump -r capture.pcap

# Capture with timestamp
sudo tcpdump -i eth0 -tttt   # Full timestamp
```

### Filtering Expressions

```bash
# Filter by host
sudo tcpdump host 10.0.1.50
sudo tcpdump src host 10.0.1.50
sudo tcpdump dst host 10.0.1.50

# Filter by port
sudo tcpdump port 443
sudo tcpdump port 80 or port 443
sudo tcpdump portrange 8000-9000

# Filter by protocol
sudo tcpdump tcp
sudo tcpdump udp
sudo tcpdump icmp

# Combine filters (AND, OR, NOT)
sudo tcpdump 'host 10.0.1.50 and port 443'
sudo tcpdump 'src host 10.0.1.50 and dst port 5432'
sudo tcpdump 'not port 22'    # Exclude SSH noise

# Filter by network/subnet
sudo tcpdump net 10.0.1.0/24

# Filter TCP flags
sudo tcpdump 'tcp[tcpflags] & tcp-syn != 0'     # SYN packets
sudo tcpdump 'tcp[tcpflags] & tcp-rst != 0'     # RST packets (resets)
sudo tcpdump 'tcp[tcpflags] & tcp-fin != 0'     # FIN packets (close)
```

---

## 2.3 Real-World tcpdump Scenarios

### Scenario 1: Debug TCP Connection Issue

```bash
# Capture TCP handshake to specific server
sudo tcpdump -i eth0 -nn 'host 10.0.1.50 and port 5432' -c 20

# Healthy connection (3-way handshake):
# 10:00:01 IP 10.0.1.10.45678 > 10.0.1.50.5432: Flags [S]     ← SYN
# 10:00:01 IP 10.0.1.50.5432 > 10.0.1.10.45678: Flags [S.]    ← SYN-ACK
# 10:00:01 IP 10.0.1.10.45678 > 10.0.1.50.5432: Flags [.]     ← ACK

# Connection refused (port closed):
# 10:00:01 IP 10.0.1.10.45678 > 10.0.1.50.5432: Flags [S]     ← SYN
# 10:00:01 IP 10.0.1.50.5432 > 10.0.1.10.45678: Flags [R.]    ← RST (refused)

# Timeout (firewall blocking):
# 10:00:01 IP 10.0.1.10.45678 > 10.0.1.50.5432: Flags [S]     ← SYN
# 10:00:02 IP 10.0.1.10.45678 > 10.0.1.50.5432: Flags [S]     ← SYN (retry)
# 10:00:04 IP 10.0.1.10.45678 > 10.0.1.50.5432: Flags [S]     ← SYN (retry)
# (no response = packet dropped by firewall)
```

```
┌──────────────────────────────────────────────────────────────┐
│         TCP FLAGS REFERENCE                                 │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Flag   tcpdump   Meaning                                    │
│  ─────  ────────  ───────────────────────────────            │
│  SYN    [S]       Start connection                           │
│  SYN-ACK [S.]     Acknowledge connection                     │
│  ACK    [.]       Acknowledge data                           │
│  FIN    [F.]      Close connection (graceful)                │
│  RST    [R.]      Reset connection (abort)                   │
│  PSH    [P.]      Push data immediately                      │
│  URG    [U]       Urgent data                                │
│                                                              │
│  Common Patterns:                                            │
│  [S] → [S.] → [.] = Successful 3-way handshake              │
│  [S] → [R.] = Connection refused (port closed)              │
│  [S] → (nothing) = Firewall dropping packets                │
│  [R.] mid-connection = Connection reset (crash/error)        │
│  [F.] → [F.] → [.] = Graceful close                         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Scenario 2: Debug DNS Resolution

```bash
# Capture DNS traffic
sudo tcpdump -i eth0 -nn port 53

# Output:
# 10:00:01 IP 10.0.1.10.52341 > 10.0.0.2.53: A? api.example.com  ← Query
# 10:00:01 IP 10.0.0.2.53 > 10.0.1.10.52341: A 10.0.1.50        ← Response

# No DNS response? → Check:
# - DNS server (10.0.0.2) reachable?
# - Security group allows UDP 53?
# - VPC DNS resolution enabled?
```

### Scenario 3: Debug TLS Handshake

```bash
# Capture TLS handshake (see ClientHello)
sudo tcpdump -i eth0 -nn 'host 10.0.1.50 and port 443' -A -c 50

# Better: Save and analyze in Wireshark
sudo tcpdump -i eth0 -nn 'host 10.0.1.50 and port 443' \
  -w tls-debug.pcap -c 100
```

### Scenario 4: Find Connection Resets

```bash
# Find all RST packets (something is killing connections)
sudo tcpdump -i eth0 -nn 'tcp[tcpflags] & tcp-rst != 0'

# Filter RST from specific service
sudo tcpdump -i eth0 -nn 'dst port 8080 and tcp[tcpflags] & tcp-rst != 0'

# Common RST causes:
# - Application crashed
# - Connection pool exhausted
# - Firewall killing idle connections
# - Load balancer health check failing
```

---

## 2.4 Wireshark — GUI Packet Analysis

```
┌──────────────────────────────────────────────────────────────┐
│         WIRESHARK WORKFLOW                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Capture on server:                                       │
│     sudo tcpdump -i eth0 -w capture.pcap -c 5000             │
│                                                              │
│  2. Copy to local machine:                                   │
│     scp server:/tmp/capture.pcap .                           │
│                                                              │
│  3. Open in Wireshark:                                       │
│     wireshark capture.pcap                                   │
│                                                              │
│  4. Apply display filters:                                   │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ tcp.port == 443                                        │  │
│  │ dns                                                    │  │
│  │ http                                                   │  │
│  │ tls.handshake                                          │  │
│  │ tcp.flags.reset == 1                                   │  │
│  │ tcp.analysis.retransmission                            │  │
│  │ ip.addr == 10.0.1.50                                   │  │
│  │ http.response.code == 500                              │  │
│  │ frame.time_delta > 1                                   │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  5. Right-click a packet → Follow → TCP Stream              │
│     (See full request/response conversation)                 │
│                                                              │
│  6. Statistics → Conversations → Sort by bytes              │
│     (Find top talkers)                                       │
│                                                              │
│  7. Statistics → IO Graphs                                  │
│     (Visualize traffic patterns over time)                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Useful Wireshark Display Filters

| Filter | Purpose |
|--------|---------|
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | New connection attempts |
| `tcp.flags.reset == 1` | Connection resets |
| `tcp.analysis.retransmission` | Retransmitted packets (packet loss) |
| `tcp.analysis.zero_window` | Receiver buffer full (backpressure) |
| `dns.flags.rcode != 0` | DNS errors |
| `tls.handshake.type == 1` | TLS ClientHello |
| `tls.handshake.type == 2` | TLS ServerHello |
| `http.response.code >= 400` | HTTP errors |
| `frame.time_delta > 0.5` | Packets with >500ms gap (latency) |

---

## 2.5 Packet Capture in Cloud / Containers

### AWS VPC Traffic Mirroring

```hcl
# Mirror traffic from an ENI to a target for analysis
resource "aws_ec2_traffic_mirror_target" "collector" {
  network_interface_id = aws_instance.collector.primary_network_interface_id
  description          = "Packet capture collector"
}

resource "aws_ec2_traffic_mirror_filter" "all" {
  description = "Capture all traffic"
}

resource "aws_ec2_traffic_mirror_filter_rule" "inbound" {
  traffic_mirror_filter_id = aws_ec2_traffic_mirror_filter.all.id
  direction                = "ingress"
  rule_number              = 100
  rule_action              = "accept"
  protocol                 = 0  # All protocols
  destination_cidr_block   = "0.0.0.0/0"
  source_cidr_block        = "0.0.0.0/0"
}

resource "aws_ec2_traffic_mirror_session" "capture" {
  network_interface_id     = aws_instance.target.primary_network_interface_id
  traffic_mirror_target_id = aws_ec2_traffic_mirror_target.collector.id
  traffic_mirror_filter_id = aws_ec2_traffic_mirror_filter.all.id
  session_number           = 1
}
```

### Capture in Kubernetes Pods

```bash
# Capture inside a running pod
kubectl exec -it my-pod -- tcpdump -i eth0 -nn -c 100

# Ephemeral debug container (if pod has no tcpdump)
kubectl debug -it my-pod --image=nicolaka/netshoot -- tcpdump -i eth0 -nn -c 50

# Copy capture file from pod
kubectl exec my-pod -- tcpdump -i eth0 -w /tmp/cap.pcap -c 500
kubectl cp my-pod:/tmp/cap.pcap ./cap.pcap

# Use netshoot sidecar for network debugging
kubectl run netshoot --rm -it --image=nicolaka/netshoot -- bash
# Inside container: tcpdump, curl, dig, nmap all available
```

### Capture in Docker

```bash
# Find container's network interface
docker inspect <container_id> | jq '.[0].NetworkSettings.Networks'

# Capture traffic on Docker bridge
sudo tcpdump -i docker0 -nn port 8080

# Capture inside container
docker exec -it <container_id> tcpdump -i eth0 -nn -c 100

# Use tcpdump container on same network
docker run --rm -it --net container:<target_container> \
  nicolaka/netshoot tcpdump -i eth0 -nn -w /tmp/capture.pcap
```

---

## 2.6 Analyzing Common Patterns

```
┌──────────────────────────────────────────────────────────────┐
│       PACKET ANALYSIS PATTERNS                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Pattern: RETRANSMISSIONS                                    │
│  ┌──────────────────────────────────────────────────────┐    │
│  │ SYN → (1s) → SYN → (2s) → SYN → (4s) → SYN         │    │
│  │ Exponential backoff = packet loss or firewall drop    │    │
│  │ Fix: Check security groups, NACLs, route tables       │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  Pattern: ZERO WINDOW                                        │
│  ┌──────────────────────────────────────────────────────┐    │
│  │ Server sends: Window=0 (buffer full, stop sending)    │    │
│  │ Client waits → Window Update → resume                 │    │
│  │ Fix: App not reading from socket fast enough, scale   │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  Pattern: RST AFTER SYN-ACK                                  │
│  ┌──────────────────────────────────────────────────────┐    │
│  │ Client: SYN → Server: SYN-ACK → Client: RST          │    │
│  │ Client rejecting the connection (wrong cert? timeout?) │    │
│  │ Fix: Check client-side TLS config or connection pool  │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
│  Pattern: HALF-OPEN CONNECTIONS                              │
│  ┌──────────────────────────────────────────────────────┐    │
│  │ SYN → SYN-ACK → ... → RST (after long delay)         │    │
│  │ Connection pool not completing handshake               │    │
│  │ Fix: Check connection pool settings, keepalive        │    │
│  └──────────────────────────────────────────────────────┘    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2.7 Troubleshooting Tips

| Problem | tcpdump Filter | What to Look For |
|---------|---------------|-----------------|
| Connection timeout | `tcp[tcpflags] & tcp-syn != 0` | SYN sent but no SYN-ACK returned |
| Connection refused | `tcp[tcpflags] & tcp-rst != 0` | RST immediately after SYN |
| Slow responses | `host X and port Y` | Large time gap between request and response |
| Packet loss | N/A (use Wireshark) | `tcp.analysis.retransmission` filter |
| DNS failure | `port 53` | Query sent but no response, or NXDOMAIN |
| TLS failure | `port 443` | Handshake starts but RST before completion |

---

## Summary Table

| Tool | Use Case | Platform |
|------|----------|----------|
| tcpdump | CLI packet capture, quick debugging | Linux/Mac |
| Wireshark | GUI analysis, deep packet inspection | Desktop |
| tshark | CLI Wireshark (scriptable analysis) | Linux |
| VPC Traffic Mirroring | Cloud packet capture | AWS |
| kubectl debug | Capture in Kubernetes pods | K8s |
| netshoot | Network debug container | Docker/K8s |

| TCP Flag | Symbol | Meaning |
|----------|--------|---------|
| SYN | [S] | Start connection |
| SYN-ACK | [S.] | Accept connection |
| ACK | [.] | Acknowledge |
| RST | [R.] | Reset/abort |
| FIN | [F.] | Graceful close |
| PSH | [P.] | Push data |

---

## Quick Revision Questions

1. **Write a tcpdump command to capture only HTTPS traffic to a specific host and save it to a file.**
2. **What do repeated SYN packets without SYN-ACK indicate?**
3. **How do you capture packets inside a Kubernetes pod that doesn't have tcpdump installed?**
4. **What Wireshark display filter would you use to find all TCP retransmissions?**
5. **Explain the difference between a connection refused (RST) and a connection timeout (no response).**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Diagnostic Tools](01-diagnostic-tools.md) | [README](../README.md) | [Network Monitoring →](03-network-monitoring.md) |
