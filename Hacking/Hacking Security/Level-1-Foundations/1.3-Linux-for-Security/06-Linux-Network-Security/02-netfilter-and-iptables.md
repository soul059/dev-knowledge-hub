# Netfilter and iptables

## Unit 6 - Topic 2 | Linux Network Security

---

## Overview

**Netfilter** is the Linux kernel's packet filtering framework. **iptables** is the primary user-space tool to configure Netfilter rules. This topic covers advanced iptables usage beyond basic firewall rules — including **NAT**, **logging**, **rate limiting**, **connection tracking**, and **security-focused configurations**.

---

## 1. iptables Tables and Chains

```
┌──────────────────────────────────────────────────────────────┐
│              IPTABLES TABLES                                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  TABLE: filter (default — packet filtering)                  │
│  ├── INPUT    — Packets destined for this host              │
│  ├── FORWARD  — Packets being routed through                │
│  └── OUTPUT   — Packets originating from this host          │
│                                                              │
│  TABLE: nat (Network Address Translation)                    │
│  ├── PREROUTING  — Alter packets before routing (DNAT)      │
│  ├── OUTPUT      — Alter locally generated packets          │
│  └── POSTROUTING — Alter packets after routing (SNAT/MASQ)  │
│                                                              │
│  TABLE: mangle (Packet alteration)                           │
│  ├── All 5 chains                                           │
│  └── Modify ToS, TTL, mark packets                          │
│                                                              │
│  TABLE: raw (Connection tracking exemption)                  │
│  ├── PREROUTING                                             │
│  └── OUTPUT                                                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Advanced iptables Rules

```bash
# === CONNECTION TRACKING (Stateful) ===
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -m state --state NEW -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -m state --state INVALID -j DROP

# === RATE LIMITING ===
# Limit SSH connections (prevent brute force)
iptables -A INPUT -p tcp --dport 22 -m limit --limit 3/min --limit-burst 5 -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -j DROP

# Limit ICMP (prevent ping flood)
iptables -A INPUT -p icmp --icmp-type echo-request \
    -m limit --limit 1/s --limit-burst 4 -j ACCEPT
iptables -A INPUT -p icmp --icmp-type echo-request -j DROP

# === LOGGING ===
iptables -A INPUT -j LOG --log-prefix "IPTABLES-INPUT-DROP: " --log-level 4
iptables -A INPUT -j DROP

# Log specific traffic
iptables -A INPUT -p tcp --dport 22 -j LOG --log-prefix "SSH-ATTEMPT: "

# === PORT KNOCKING CONCEPT ===
# Sequential connection to ports 1000, 2000, 3000 → opens SSH
# Implemented with iptables recent module or knockd

# === GEO-BLOCKING (with xt_geoip) ===
# Block traffic from specific countries
iptables -A INPUT -m geoip --src-cc CN,RU -j DROP
```

---

## 3. NAT Configuration

```bash
# === MASQUERADE (Source NAT for dynamic IP) ===
# Enable IP forwarding first
echo 1 > /proc/sys/net/ipv4/ip_forward

# Internal network (192.168.1.0/24) → Internet via eth0
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o eth1 -m state --state ESTABLISHED,RELATED -j ACCEPT

# === DNAT (Port Forwarding) ===
# Forward port 8080 to internal web server
iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80
iptables -A FORWARD -p tcp -d 192.168.1.100 --dport 80 -j ACCEPT

# === REDIRECT (Local port redirect) ===
# Redirect port 80 to 8080 (transparent proxy)
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

---

## 4. Security Use Cases

| Use Case | iptables Command |
|----------|-----------------|
| Block specific IP | `iptables -A INPUT -s 10.0.0.99 -j DROP` |
| Allow only SSH from subnet | `iptables -A INPUT -p tcp --dport 22 -s 10.0.0.0/24 -j ACCEPT` |
| Block port scans | `iptables -A INPUT -p tcp --tcp-flags ALL NONE -j DROP` |
| Block XMAS scan | `iptables -A INPUT -p tcp --tcp-flags ALL ALL -j DROP` |
| Rate limit SSH | `iptables -A INPUT -p tcp --dport 22 -m limit --limit 3/min -j ACCEPT` |
| Log dropped packets | `iptables -A INPUT -j LOG --log-prefix "DROP: "` |

---

## 5. iptables vs nftables

| Feature | iptables | nftables |
|---------|:--------:|:--------:|
| **Status** | Legacy (still widely used) | Modern replacement |
| **Syntax** | Complex | Cleaner |
| **Performance** | Good | Better |
| **IPv4/IPv6** | Separate tools | Unified |
| **Atomicity** | Per-rule | Batch updates |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Netfilter** | Kernel-space packet filtering |
| **iptables** | User-space configuration tool |
| **Tables** | filter, nat, mangle, raw |
| **Stateful** | Track connection state (NEW, ESTABLISHED) |
| **Rate limiting** | Prevent brute force and DoS |
| **NAT** | SNAT/MASQUERADE for internet sharing |

---

## Quick Revision Questions

1. **What are the four iptables tables?**
2. **How do you rate-limit SSH connections?**
3. **What is the difference between SNAT and MASQUERADE?**
4. **How do you log dropped packets?**
5. **Write an iptables rule to block a specific IP address.**

---

[← Previous: Network Configuration Files](01-network-configuration-files.md) | [Next: TCP Wrappers →](03-tcp-wrappers.md)

---

*Unit 6 - Topic 2 of 5 | [Back to README](../README.md)*
