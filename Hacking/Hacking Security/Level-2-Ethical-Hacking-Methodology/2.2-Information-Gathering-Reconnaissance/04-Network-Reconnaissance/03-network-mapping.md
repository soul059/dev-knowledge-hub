# Network Mapping

## Unit 4 - Topic 3 | Network Reconnaissance

---

## Overview

**Network mapping** creates a visual or logical representation of the target's network infrastructure — including hosts, services, network segments, and relationships. It transforms raw scan data into an actionable intelligence picture.

---

## 1. Network Mapping Process

```
┌──────────────────────────────────────────────────────────────┐
│              NETWORK MAPPING WORKFLOW                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. IDENTIFY BOUNDARIES                                      │
│     └── IP ranges, domains, cloud assets                     │
│                                                              │
│  2. DISCOVER HOSTS                                           │
│     └── Ping sweep, ARP scan, DNS enum                       │
│                                                              │
│  3. IDENTIFY SERVICES                                        │
│     └── Port scan + version detection                        │
│                                                              │
│  4. MAP RELATIONSHIPS                                        │
│     └── Network segments, routing, dependencies              │
│                                                              │
│  5. VISUALIZE                                                │
│     └── Create network diagram                               │
│                                                              │
│  6. IDENTIFY TARGETS                                         │
│     └── High-value systems for exploitation                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Mapping Tools and Commands

```bash
# === HOST DISCOVERY ===
nmap -sn 203.0.113.0/24 -oG alive.txt     # Ping sweep
masscan 203.0.113.0/24 -p 80,443 --rate=1000

# === SERVICE MAPPING ===
nmap -sS -sV -O --top-ports 1000 -iL alive.txt -oA network_map

# === TRACEROUTE MAPPING ===
traceroute target.com                       # Path mapping
nmap --traceroute -sn 203.0.113.0/24       # Nmap traceroute

# === NETWORK TOPOLOGY ===
# Zenmap (Nmap GUI) — auto-generates topology maps
zenmap

# === VISUAL MAPPING TOOLS ===
# Draw.io / diagrams.net — manual network diagrams
# Maltego — automated relationship mapping
# Nmap + Zenmap — host/network visualization
```

---

## 3. Network Diagram Example

```
EXTERNAL NETWORK MAP:

INTERNET
    │
    ├── Cloudflare CDN (104.16.x.x)
    │   └── www.target.com (:80, :443)
    │
    ├── 203.0.113.0/24 (Main Data Center)
    │   ├── .10 — Web Server (Apache 2.4.49)
    │   ├── .11 — Admin Panel (nginx)
    │   ├── .20 — Mail Server (Postfix)
    │   ├── .30 — VPN Gateway (OpenVPN)
    │   ├── .40 — Dev Server (Node.js)
    │   └── .50 — Jenkins CI (:8080)
    │
    └── 198.51.100.0/24 (DR Site)
        ├── .10 — DR Web Server
        └── .20 — DR Database

INTERNAL (discovered via pivot):
    10.0.0.0/24
    ├── .1  — Gateway/Router
    ├── .10 — Domain Controller (AD)
    ├── .50 — Database Server (MySQL)
    └── .100— File Server (SMB)
```

---

## 4. High-Value Target Identification

| Target Type | Indicators | Priority |
|-------------|-----------|:--------:|
| **Domain Controller** | Port 88, 389, 636, 445 | ★★★★★ |
| **Database Server** | Port 3306, 1433, 5432 | ★★★★★ |
| **Web Application** | Port 80, 443, 8080 | ★★★★ |
| **Mail Server** | Port 25, 110, 143 | ★★★★ |
| **File Server** | Port 445, 139, 21 | ★★★★ |
| **VPN Gateway** | Port 443, 500, 1194 | ★★★ |
| **CI/CD Server** | Port 8080 (Jenkins) | ★★★ |
| **Monitoring** | Port 3000 (Grafana) | ★★★ |

---

## 5. Documentation Template

```
NETWORK MAP DOCUMENTATION:

SCOPE: 203.0.113.0/24
DATE: [date]
HOSTS DISCOVERED: 12 of 256 possible

HOST INVENTORY:
┌──────────────────┬─────────┬────────────┬──────────────┐
│ IP               │ Hostname│ OS         │ Key Services │
├──────────────────┼─────────┼────────────┼──────────────┤
│ 203.0.113.10     │ web01   │ Ubuntu 22  │ HTTP, SSH    │
│ 203.0.113.11     │ admin   │ Ubuntu 22  │ HTTPS, SSH   │
│ 203.0.113.20     │ mail    │ Ubuntu 20  │ SMTP, IMAP   │
│ 203.0.113.30     │ vpn     │ pfSense    │ OpenVPN      │
│ 203.0.113.40     │ dev     │ Ubuntu 22  │ HTTP:8080    │
│ 203.0.113.50     │ jenkins │ Ubuntu 20  │ HTTP:8080    │
└──────────────────┴─────────┴────────────┴──────────────┘

NETWORK SEGMENTS IDENTIFIED:
├── DMZ: 203.0.113.0/24 (internet-facing)
├── Internal: 10.0.0.0/24 (behind VPN)
└── DR: 198.51.100.0/24 (disaster recovery)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Network mapping** | Visualize target infrastructure |
| **Host discovery** | nmap -sn, masscan |
| **Service mapping** | nmap -sV, version detection |
| **Visualization** | Zenmap, Draw.io, Maltego |
| **High-value targets** | DC, DB, web apps, mail, files |
| **Documentation** | Host inventory with OS and services |

---

## Quick Revision Questions

1. **What is the purpose of network mapping?**
2. **What tools help visualize network topology?**
3. **What makes a system a "high-value target"?**
4. **How do you discover hosts on a network?**
5. **What should a network map document include?**

---

[← Previous: BGP and ASN Information](02-bgp-and-asn-information.md) | [Next: CDN Identification →](04-cdn-identification.md)

---

*Unit 4 - Topic 3 of 5 | [Back to README](../README.md)*
