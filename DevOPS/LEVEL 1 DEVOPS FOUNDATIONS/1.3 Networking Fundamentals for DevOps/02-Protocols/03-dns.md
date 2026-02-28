# Chapter 3: DNS

## Overview

DNS (Domain Name System) translates human-readable domain names (like google.com) into IP addresses (like 142.250.80.46). It's the phonebook of the internet. DNS outages take down entire applications — understanding DNS is critical for DevOps engineers managing deployments, load balancing, and service discovery.

---

## 3.1 How DNS Works

```
┌──────────────────────────────────────────────────────────────┐
│                  DNS RESOLUTION FLOW                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  User types: www.example.com                                 │
│                                                              │
│  ┌────────┐  1. Query   ┌──────────────┐                    │
│  │Browser │────────────▶│ DNS Resolver  │                    │
│  │        │             │ (ISP/8.8.8.8) │                    │
│  └────────┘             └──────┬───────┘                    │
│       ▲                        │                             │
│       │ 8. IP: 93.184.216.34   │ 2. "Who handles .com?"     │
│       │                        ▼                             │
│       │                 ┌──────────────┐                     │
│       │                 │  Root DNS    │                     │
│       │                 │  Server (.)  │                     │
│       │                 └──────┬───────┘                    │
│       │                        │ 3. "Ask .com TLD server"    │
│       │                        ▼                             │
│       │                 ┌──────────────┐                     │
│       │                 │  TLD DNS     │                     │
│       │                 │ Server (.com)│                     │
│       │                 └──────┬───────┘                    │
│       │                        │ 4. "Ask ns1.example.com"    │
│       │                        ▼                             │
│       │                 ┌──────────────────┐                 │
│       │                 │ Authoritative DNS│                 │
│       │                 │ (ns1.example.com)│                 │
│       │                 └──────┬───────────┘                │
│       │                        │ 5. "IP is 93.184.216.34"    │
│       └────────────────────────┘                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.2 DNS Record Types

```
┌──────────┬────────────────────────────────────────────────────┐
│  Record  │  Purpose & Example                                │
├──────────┼────────────────────────────────────────────────────┤
│  A       │  Domain → IPv4 address                            │
│          │  example.com → 93.184.216.34                       │
│          │                                                    │
│  AAAA    │  Domain → IPv6 address                             │
│          │  example.com → 2606:2800:220:1:...                 │
│          │                                                    │
│  CNAME   │  Alias → Another domain name                      │
│          │  www.example.com → example.com                     │
│          │  (Cannot coexist with other records at zone apex)  │
│          │                                                    │
│  MX      │  Mail server for the domain                       │
│          │  example.com → 10 mail.example.com                 │
│          │                                                    │
│  TXT     │  Arbitrary text (SPF, DKIM, verification)         │
│          │  example.com → "v=spf1 include:_spf.google.com"   │
│          │                                                    │
│  NS      │  Nameserver for the domain                        │
│          │  example.com → ns1.example.com                     │
│          │                                                    │
│  SOA     │  Start of Authority (zone metadata)               │
│          │  Primary NS, admin email, serial, timers           │
│          │                                                    │
│  SRV     │  Service location (port + host)                   │
│          │  _sip._tcp.example.com → 10 5 5060 sip.example.com│
│          │                                                    │
│  PTR     │  IP → Domain (reverse DNS)                        │
│          │  34.216.184.93.in-addr.arpa → example.com          │
│          │                                                    │
│  CAA     │  Which CAs can issue certificates                 │
│          │  example.com → 0 issue "letsencrypt.org"           │
└──────────┴────────────────────────────────────────────────────┘
```

---

## 3.3 DNS Caching and TTL

```
┌──────────────────────────────────────────────────────────────┐
│                  DNS CACHING LAYERS                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Browser Cache        (Chrome: chrome://net-internals/#dns)│
│     TTL: seconds to minutes                                  │
│         │                                                    │
│         ▼                                                    │
│  2. OS Cache             (systemd-resolved, nscd)             │
│     TTL: follows record TTL                                  │
│         │                                                    │
│         ▼                                                    │
│  3. Recursive Resolver   (ISP DNS, 8.8.8.8, 1.1.1.1)         │
│     TTL: follows record TTL                                  │
│         │                                                    │
│         ▼                                                    │
│  4. Authoritative Server (Route 53, Cloudflare)               │
│     Source of truth — sets the TTL                            │
│                                                              │
│  TTL (Time To Live):                                         │
│  ├── Low TTL (60s)  → Faster changes, more DNS queries       │
│  ├── High TTL (3600s) → Fewer queries, slower updates        │
│  └── Deployment tip: Lower TTL before migration,             │
│       raise it after                                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.4 DNS Commands

```bash
# Basic DNS lookup
nslookup example.com
dig example.com

# Specific record types
dig example.com A          # IPv4 address
dig example.com AAAA       # IPv6 address
dig example.com MX         # Mail servers
dig example.com NS         # Nameservers
dig example.com TXT        # Text records
dig example.com CNAME      # Aliases
dig example.com SOA        # Start of Authority

# Query specific DNS server
dig @8.8.8.8 example.com
dig @1.1.1.1 example.com

# Trace full resolution path
dig +trace example.com

# Short answer only
dig +short example.com

# Reverse DNS lookup
dig -x 93.184.216.34

# Check TTL
dig example.com | grep -E "^example" | awk '{print $2}'

# Flush DNS cache
# Windows
ipconfig /flushdns

# macOS
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder

# Linux (systemd)
sudo systemd-resolve --flush-caches
```

---

## 3.5 DNS in DevOps

### Service Discovery

```
┌──────────────────────────────────────────────────────────────┐
│              DNS-BASED SERVICE DISCOVERY                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Kubernetes:                                                 │
│  ┌──────────────────────────────────────────────┐            │
│  │  Service Name: my-service                    │            │
│  │  Namespace: production                        │            │
│  │                                              │            │
│  │  DNS Name: my-service.production.svc.cluster.local │      │
│  │  Format:   <svc>.<namespace>.svc.cluster.local     │      │
│  │                                              │            │
│  │  Pod DNS:  10-244-1-5.production.pod.cluster.local │      │
│  └──────────────────────────────────────────────┘            │
│                                                              │
│  Docker Compose:                                             │
│  ┌──────────────────────────────────────────────┐            │
│  │  services:                                   │            │
│  │    web:                                      │            │
│  │      image: nginx                            │            │
│  │    api:                                      │            │
│  │      image: node-api                         │            │
│  │      environment:                            │            │
│  │        DB_HOST: db    ← DNS name!            │            │
│  │    db:                                       │            │
│  │      image: postgres                         │            │
│  └──────────────────────────────────────────────┘            │
│  Docker automatically resolves "db" to the container's IP    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Blue-Green Deployment with DNS

```
┌──────────────────────────────────────────────────────────────┐
│           BLUE-GREEN DEPLOYMENT VIA DNS                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  BEFORE: app.example.com → Blue (v1.0) at 10.0.1.10         │
│                                                              │
│  Step 1: Deploy Green (v2.0) at 10.0.2.20                    │
│  Step 2: Test Green environment                              │
│  Step 3: Update DNS record:                                  │
│          app.example.com → 10.0.2.20 (Green)                 │
│  Step 4: Wait for TTL to expire                              │
│  Step 5: Decommission Blue (v1.0)                            │
│                                                              │
│  ┌──────────┐            ┌──────────────┐                    │
│  │  DNS     │───BEFORE──▶│ Blue (v1.0)  │                    │
│  │ Record   │            │ 10.0.1.10    │                    │
│  │          │            └──────────────┘                    │
│  │app.      │                                                │
│  │example.  │            ┌──────────────┐                    │
│  │com       │───AFTER───▶│ Green (v2.0) │                    │
│  │          │            │ 10.0.2.20    │                    │
│  └──────────┘            └──────────────┘                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 3.6 /etc/hosts and DNS Override

```bash
# /etc/hosts takes priority over DNS
# Useful for testing, development, overrides

# Linux/macOS: /etc/hosts
# Windows: C:\Windows\System32\drivers\etc\hosts

# Format: IP_ADDRESS    HOSTNAME
127.0.0.1       localhost
10.0.1.50       api.internal.local
192.168.1.100   myapp.dev

# DevOps use cases:
# - Test a new server before DNS change
# - Override DNS for debugging
# - Local development with custom domains
```

---

## 3.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Could not resolve host" | DNS server unreachable | Check /etc/resolv.conf, try 8.8.8.8 |
| Old IP returned | DNS cache stale | Flush cache, wait for TTL |
| NXDOMAIN | Domain doesn't exist in DNS | Verify record exists, check for typos |
| SERVFAIL | DNS server error | Check authoritative NS, validate zone file |
| Slow DNS resolution | DNS server far/slow | Use faster DNS (1.1.1.1, 8.8.8.8) |
| Works with IP, fails with domain | DNS issue confirmed | `dig` the domain, check NS records |
| K8s service DNS not working | CoreDNS pod down | `kubectl get pods -n kube-system` |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| DNS | Translates domain names to IP addresses |
| A Record | Domain → IPv4 |
| CNAME | Alias → Another domain |
| MX | Mail server records |
| TTL | Cache duration; lower before migrations |
| dig/nslookup | Primary DNS debugging commands |
| /etc/hosts | Local DNS override (takes priority) |
| K8s DNS | `<svc>.<ns>.svc.cluster.local` |
| Docker DNS | Service names auto-resolve in compose |

---

## Quick Revision Questions

1. **Walk through the DNS resolution process when you type "www.google.com" in a browser.**
2. **What is the difference between an A record and a CNAME record?**
3. **Why should you lower TTL before a DNS migration?**
4. **How does Kubernetes DNS work? What's the format for a service DNS name?**
5. **You can ping an IP but `curl http://example.com` fails. How do you diagnose this?**
6. **What is the purpose of a PTR record?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← HTTP/HTTPS](02-http-https.md) | [README](../README.md) | [DHCP →](04-dhcp.md) |
