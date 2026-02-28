# Chapter 1: TCP vs UDP

## Overview

TCP (Transmission Control Protocol) and UDP (User Datagram Protocol) are the two primary transport layer protocols. They determine HOW data is delivered across a network — reliably with ordering guarantees (TCP) or fast with minimal overhead (UDP). Every service you deploy, every port you open, and every load balancer you configure relies on one or both of these protocols.

---

## 1.1 TCP vs UDP at a Glance

```
┌──────────────────────────────────────────────────────────────┐
│                TCP vs UDP COMPARISON                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  TCP (Transmission Control Protocol)                         │
│  ┌──────────────────────────────────────┐                    │
│  │ ✓ Connection-oriented               │                    │
│  │ ✓ Reliable delivery (guaranteed)     │                    │
│  │ ✓ Ordered packets                   │                    │
│  │ ✓ Error checking & retransmission   │                    │
│  │ ✓ Flow control & congestion control │                    │
│  │ ✗ Slower (more overhead)            │                    │
│  │ Use: HTTP, SSH, FTP, SMTP, DB       │                    │
│  └──────────────────────────────────────┘                    │
│                                                              │
│  UDP (User Datagram Protocol)                                │
│  ┌──────────────────────────────────────┐                    │
│  │ ✓ Connectionless                    │                    │
│  │ ✓ Fast (minimal overhead)           │                    │
│  │ ✓ Low latency                       │                    │
│  │ ✗ No delivery guarantee             │                    │
│  │ ✗ No ordering                       │                    │
│  │ ✗ No retransmission                 │                    │
│  │ Use: DNS, DHCP, VoIP, streaming     │                    │
│  └──────────────────────────────────────┘                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.2 TCP Three-Way Handshake

TCP establishes a connection BEFORE sending data using a three-step process:

```
┌──────────────────────────────────────────────────────────────┐
│              TCP THREE-WAY HANDSHAKE                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│    Client                              Server                │
│    ──────                              ──────                │
│      │                                   │                   │
│      │──── SYN (seq=100) ──────────────▶│  Step 1: Client   │
│      │     "I want to connect"           │  initiates        │
│      │                                   │                   │
│      │◀─── SYN-ACK (seq=300,ack=101) ───│  Step 2: Server   │
│      │     "OK, I acknowledge"           │  acknowledges     │
│      │                                   │                   │
│      │──── ACK (ack=301) ──────────────▶│  Step 3: Client   │
│      │     "Connection established"      │  confirms         │
│      │                                   │                   │
│      │◀═══ DATA TRANSFER ═══════════════▶│  Connection is    │
│      │                                   │  now ESTABLISHED  │
│      │                                   │                   │
│      │──── FIN ────────────────────────▶│  Connection        │
│      │◀─── ACK ─────────────────────────│  teardown          │
│      │◀─── FIN ─────────────────────────│  (4-way)          │
│      │──── ACK ────────────────────────▶│                    │
│      │                                   │                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### UDP: No Handshake

```
┌──────────────────────────────────────────────────────────────┐
│              UDP — FIRE AND FORGET                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│    Client                              Server                │
│    ──────                              ──────                │
│      │                                   │                   │
│      │──── DATA ───────────────────────▶│  Just send it!    │
│      │──── DATA ───────────────────────▶│  No handshake     │
│      │──── DATA ───────────────────────▶│  No confirmation  │
│      │                                   │                   │
│      │     (some packets may be lost)    │                   │
│      │     (packets may arrive           │                   │
│      │      out of order)                │                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.3 Comparison Table

```
┌────────────────────────┬────────────────────┬────────────────────┐
│  Feature               │  TCP               │  UDP               │
├────────────────────────┼────────────────────┼────────────────────┤
│  Connection            │  Connection-based   │  Connectionless    │
│  Reliability           │  Guaranteed         │  Best-effort       │
│  Ordering              │  Yes (sequence #s)  │  No                │
│  Error checking        │  Yes + retransmit   │  Checksum only     │
│  Flow control          │  Yes (window)       │  No                │
│  Speed                 │  Slower             │  Faster            │
│  Header size           │  20-60 bytes        │  8 bytes           │
│  Overhead              │  Higher             │  Lower             │
│  Broadcasting          │  No                 │  Yes               │
│  Use cases             │  Web, email, file   │  DNS, VoIP, games  │
│  State                 │  Stateful           │  Stateless         │
└────────────────────────┴────────────────────┴────────────────────┘
```

---

## 1.4 Common Port Numbers

```
┌─────────────────────────────────────────────────────────────┐
│              WELL-KNOWN PORTS                                │
├──────────┬────────────┬──────────────────────────────────────┤
│  Port    │  Protocol  │  Service                             │
├──────────┼────────────┼──────────────────────────────────────┤
│  20, 21  │  TCP       │  FTP (data, control)                 │
│  22      │  TCP       │  SSH                                 │
│  23      │  TCP       │  Telnet                              │
│  25      │  TCP       │  SMTP (email sending)                │
│  53      │  TCP/UDP   │  DNS                                 │
│  67, 68  │  UDP       │  DHCP (server, client)               │
│  80      │  TCP       │  HTTP                                │
│  110     │  TCP       │  POP3 (email retrieval)              │
│  143     │  TCP       │  IMAP (email retrieval)              │
│  443     │  TCP       │  HTTPS                               │
│  465     │  TCP       │  SMTPS (SMTP over SSL)               │
│  587     │  TCP       │  SMTP (submission)                   │
│  993     │  TCP       │  IMAPS                               │
│  3306    │  TCP       │  MySQL                               │
│  3389    │  TCP       │  RDP (Remote Desktop)                │
│  5432    │  TCP       │  PostgreSQL                          │
│  5672    │  TCP       │  RabbitMQ (AMQP)                     │
│  6379    │  TCP       │  Redis                               │
│  8080    │  TCP       │  HTTP alternate                      │
│  8443    │  TCP       │  HTTPS alternate                     │
│  9090    │  TCP       │  Prometheus                          │
│  27017   │  TCP       │  MongoDB                             │
└──────────┴────────────┴──────────────────────────────────────┘

Port ranges:
├── 0-1023:    Well-known (system) ports
├── 1024-49151: Registered ports
└── 49152-65535: Dynamic/ephemeral ports
```

---

## 1.5 TCP in DevOps: Real-World Usage

### Security Group Rules (AWS)

```bash
# Allow inbound HTTP (TCP)
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Allow inbound DNS (UDP)
aws ec2 authorize-security-group-ingress \
  --group-id sg-12345 \
  --protocol udp \
  --port 53 \
  --cidr 10.0.0.0/16
```

### Docker Port Mapping

```bash
# TCP port mapping (default)
docker run -d -p 8080:80 nginx         # TCP

# UDP port mapping (explicit)
docker run -d -p 53:53/udp dns-server  # UDP

# Both TCP and UDP
docker run -d -p 53:53/tcp -p 53:53/udp dns-server
```

### Kubernetes Service Types

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
    - name: http
      protocol: TCP      # Default: TCP
      port: 80
      targetPort: 8080
    - name: dns
      protocol: UDP      # Explicit: UDP
      port: 53
      targetPort: 53
```

---

## 1.6 TCP Connection States

Understanding TCP states helps with debugging connection issues:

```
┌──────────────────────────────────────────────────────────────┐
│              TCP CONNECTION STATES                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  LISTEN        → Server waiting for connections              │
│  SYN_SENT      → Client sent SYN, waiting for SYN-ACK       │
│  SYN_RECEIVED  → Server got SYN, sent SYN-ACK               │
│  ESTABLISHED   → Connection is active (data flowing)         │
│  FIN_WAIT_1    → Client sent FIN, waiting for ACK            │
│  FIN_WAIT_2    → Client got ACK of FIN, waiting for FIN      │
│  TIME_WAIT     → Waiting before closing (2*MSL timeout)      │
│  CLOSE_WAIT    → Server received FIN, application closing    │
│  LAST_ACK      → Server sent FIN, waiting for final ACK      │
│  CLOSED         → Connection terminated                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```bash
# View TCP connection states
ss -s                           # Summary
ss -tuln                        # Listening ports
ss -tn state established        # Established connections
ss -tn state time-wait          # TIME_WAIT connections
netstat -an | grep ESTABLISHED  # Windows/macOS
```

### [?] Troubleshooting: Too many TIME_WAIT connections

```bash
# Check TIME_WAIT count
ss -s | grep TIME-WAIT

# If too many TIME_WAIT: your app is creating/closing connections too fast
# Solutions:
# 1. Enable connection pooling in your application
# 2. Use HTTP keep-alive
# 3. Tune kernel parameters (Linux):
sysctl -w net.ipv4.tcp_tw_reuse=1
sysctl -w net.ipv4.tcp_fin_timeout=30
```

---

## 1.7 When to Use TCP vs UDP

```
┌──────────────────────────────────────────────────────────────┐
│           DECISION GUIDE: TCP vs UDP                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Use TCP when:                                               │
│  ├── Data must arrive completely and in order                │
│  ├── Web applications (HTTP/HTTPS)                           │
│  ├── File transfers (FTP, SCP)                               │
│  ├── Email (SMTP, IMAP, POP3)                                │
│  ├── Database connections (MySQL, PostgreSQL)                 │
│  ├── SSH sessions                                            │
│  └── API calls (REST, gRPC)                                  │
│                                                              │
│  Use UDP when:                                               │
│  ├── Speed matters more than reliability                     │
│  ├── DNS lookups (small, fast queries)                        │
│  ├── DHCP (network bootstrapping)                            │
│  ├── Video/audio streaming                                   │
│  ├── Online gaming                                           │
│  ├── IoT sensor data                                         │
│  └── Network monitoring (SNMP)                               │
│                                                              │
│  Modern protocols using UDP:                                 │
│  ├── QUIC (HTTP/3) — reliable UDP!                           │
│  ├── WireGuard VPN                                           │
│  └── WebRTC                                                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.8 Troubleshooting Tips

| Problem | Cause | Command to Diagnose |
|---------|-------|-------------------|
| "Connection refused" | No service listening on port | `ss -tuln \| grep PORT` |
| "Connection timed out" | Firewall blocking, no route | `telnet HOST PORT` |
| Many TIME_WAIT connections | Connection churn | `ss -s` |
| Many CLOSE_WAIT | App not closing sockets | Check application code |
| Packet loss (UDP) | Network congestion, no retry | `mtr HOST` |
| Slow TCP connections | High latency, small window | `ss -ti` (show TCP info) |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| TCP | Connection-oriented, reliable, ordered, slower |
| UDP | Connectionless, fast, no guarantee, lightweight |
| Three-way handshake | SYN → SYN-ACK → ACK (TCP only) |
| Ports | 0-1023 well-known, 1024-49151 registered, 49152-65535 ephemeral |
| DevOps usage | Security groups, Docker port mapping, K8s services |
| TCP states | LISTEN, ESTABLISHED, TIME_WAIT, CLOSE_WAIT most relevant |
| QUIC/HTTP3 | Modern protocol: reliability built ON TOP of UDP |

---

## Quick Revision Questions

1. **Explain the TCP three-way handshake. What happens at each step?**
2. **Why does DNS primarily use UDP instead of TCP?**
3. **What TCP state indicates a server application isn't properly closing connections?**
4. **You're configuring a security group for a web server. What protocol and port do you open?**
5. **What is the difference between well-known ports and ephemeral ports?**
6. **Why is HTTP/3 (QUIC) built on UDP instead of TCP?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← NAT and PAT](../01-Networking-Basics/05-nat-and-pat.md) | [README](../README.md) | [HTTP/HTTPS →](02-http-https.md) |
