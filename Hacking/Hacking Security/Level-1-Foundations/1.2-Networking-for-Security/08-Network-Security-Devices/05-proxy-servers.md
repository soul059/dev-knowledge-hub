# Proxy Servers

## Unit 8 - Topic 5 | Network Security Devices

---

## Overview

A **proxy server** acts as an intermediary between clients and servers, forwarding requests on behalf of the client. Proxies provide **anonymity**, **caching**, **content filtering**, **access control**, and **traffic inspection** — making them a key component of enterprise network security.

---

## 1. Proxy Types

```
┌──────────────────────────────────────────────────────────────┐
│              PROXY TYPES                                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  FORWARD PROXY (Client-side):                                │
│  Client → [Forward Proxy] → Internet                         │
│  • Client knows about proxy, configures it                   │
│  • Hides client identity from server                         │
│  • Used for: content filtering, caching, access control      │
│                                                              │
│  REVERSE PROXY (Server-side):                                │
│  Client → [Reverse Proxy] → Backend Servers                  │
│  • Client doesn't know about proxy                           │
│  • Hides server identity from client                         │
│  • Used for: load balancing, SSL, WAF, caching               │
│                                                              │
│  TRANSPARENT PROXY:                                          │
│  Client → [Transparent Proxy] → Internet                     │
│  • No client configuration needed                            │
│  • Intercepts traffic at network level                       │
│  • Used for: mandatory filtering, monitoring                  │
│                                                              │
│  SOCKS PROXY:                                                │
│  • Operates at Layer 5 (Session layer)                       │
│  • Protocol-agnostic (any TCP/UDP traffic)                   │
│  • Used for: bypassing restrictions, Tor                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Security Functions

| Function | Description |
|----------|-------------|
| **Content Filtering** | Block access to malicious/inappropriate websites |
| **URL Categorization** | Classify websites by category (social, malware, etc.) |
| **SSL/TLS Inspection** | Decrypt, inspect, re-encrypt HTTPS traffic |
| **Malware Scanning** | Scan downloads for malicious content |
| **Data Loss Prevention** | Prevent sensitive data from leaving the network |
| **Authentication** | Require user login before internet access |
| **Logging & Monitoring** | Record all web activity for audit |
| **Bandwidth Control** | Limit bandwidth for specific sites/users |

---

## 3. SSL/TLS Inspection (MITM Proxy)

```
┌──────────────────────────────────────────────────────────────┐
│              SSL INSPECTION                                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Without SSL Inspection:                                     │
│  Client ════════encrypted════════► Server                    │
│  (Proxy can't see content)                                   │
│                                                              │
│  With SSL Inspection:                                        │
│  Client ═══TLS═══► Proxy ═══TLS═══► Server                  │
│          (Proxy CA)       (Server cert)                      │
│                                                              │
│  1. Proxy intercepts TLS connection                          │
│  2. Proxy presents its own certificate to client             │
│  3. Proxy establishes separate TLS to destination            │
│  4. Proxy decrypts, inspects, re-encrypts traffic            │
│                                                              │
│  Requires: Proxy CA certificate trusted by all clients       │
│  Privacy concern: All user traffic is visible to proxy       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Proxy Solutions

| Solution | Type | Notes |
|----------|------|-------|
| **Squid** | Open source | Forward proxy, caching |
| **Nginx** | Open source | Reverse proxy, web server |
| **HAProxy** | Open source | Reverse proxy, load balancer |
| **Zscaler** | Cloud | Cloud-based secure web gateway |
| **Blue Coat (Symantec)** | Commercial | Enterprise web proxy |
| **mitmproxy** | Open source | SSL inspection, pentesting tool |
| **BurpSuite** | Commercial | Web app pentesting proxy |

---

## 5. Attacker Use of Proxies

| Use | Purpose |
|-----|---------|
| **Anonymization** | Hide attacker's real IP address |
| **Proxy chains** | Route through multiple proxies for anonymity |
| **Tor network** | Onion routing through volunteer proxies |
| **SOCKS proxy** | Tunnel any traffic through compromised host |
| **Reverse proxy** | C2 infrastructure — hide backend C2 server |

```bash
# Proxychains (attacker tool)
proxychains nmap -sT 10.0.0.1     # Scan through proxy chain
proxychains curl http://target.com # Browse through proxies

# SSH SOCKS proxy
ssh -D 9050 user@pivot-host        # Create SOCKS proxy
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Forward Proxy** | Client-side, hides client from server |
| **Reverse Proxy** | Server-side, hides server from client |
| **Transparent** | No client config, intercepts at network level |
| **SSL Inspection** | Decrypt HTTPS to inspect content |
| **Security Role** | Filtering, DLP, malware scanning, logging |
| **Attacker Use** | Anonymization, proxy chains, Tor |

---

## Quick Revision Questions

1. **What is the difference between a forward and reverse proxy?**
2. **How does SSL/TLS inspection work?**
3. **What is a transparent proxy?**
4. **How do attackers use proxies?**
5. **Name 3 proxy solutions and their primary use.**

---

[← Previous: Load Balancers](04-load-balancers.md) | [Next: VPN Gateways →](06-vpn-gateways.md)

---

*Unit 8 - Topic 5 of 6 | [Back to README](../README.md)*
