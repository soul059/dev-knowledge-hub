# Chapter 1: DNS Hierarchy

## Overview

The **Domain Name System (DNS)** is organized as a hierarchical, distributed database. Understanding this hierarchy is critical for DevOps — it explains how domain names resolve, where to configure records, how caching works at each level, and why propagation takes time. DNS is one of the most common sources of outages in production systems.

---

## 1.1 The DNS Hierarchy Tree

```
┌──────────────────────────────────────────────────────────────┐
│              DNS HIERARCHY                                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│                     . (Root)                                 │
│              13 root server clusters                         │
│          (a.root-servers.net → m.root-servers.net)           │
│                    │                                         │
│         ┌──────────┼──────────┐                              │
│         │          │          │                               │
│        .com       .org       .io       ← TLD Servers        │
│         │          │          │           (Top-Level Domain)  │
│    ┌────┴────┐     │     ┌───┴───┐                           │
│    │         │     │     │       │                            │
│  google   example  │  myapp   startup                       │
│  .com     .com     │  .io     .io     ← Authoritative NS    │
│    │               │                    (Your domain)        │
│  ┌─┴──┐         ┌──┴──┐                                     │
│  │    │         │     │                                      │
│ www  mail     www    api              ← Individual Records  │
│               app    staging                                 │
│                                                              │
│  Resolution flows: Root → TLD → Authoritative → Record      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.2 DNS Components

| Component | Role | Example |
|-----------|------|---------|
| Root Servers | Know where TLD servers are | 13 clusters (a-m.root-servers.net) |
| TLD Servers | Know authoritative NS for domains | Verisign (.com), PIR (.org) |
| Authoritative NS | Hold actual DNS records | ns1.example.com (your NS) |
| Recursive Resolver | Walks the hierarchy for clients | 8.8.8.8 (Google), 1.1.1.1 (Cloudflare) |
| Stub Resolver | Client-side, sends queries | Your OS DNS client |
| Zone | Administrative boundary | example.com zone file |
| Zone File | Contains DNS records | A, AAAA, CNAME, MX, etc. |

---

## 1.3 Full DNS Resolution Process

```
┌──────────────────────────────────────────────────────────────┐
│        FULL DNS RESOLUTION: app.example.com                  │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  User's Browser                                              │
│  ┌──────────┐                                                │
│  │ 1. Check │ "Where is app.example.com?"                    │
│  │  browser │                                                │
│  │  cache   │──── Found? → Use cached IP                     │
│  └────┬─────┘                                                │
│       │ Not found                                            │
│       ▼                                                      │
│  ┌──────────┐                                                │
│  │ 2. Check │ OS DNS cache (/etc/hosts, systemd-resolved)    │
│  │  OS cache│──── Found? → Use cached IP                     │
│  └────┬─────┘                                                │
│       │ Not found                                            │
│       ▼                                                      │
│  ┌──────────┐  "Who handles .com?"   ┌───────────┐          │
│  │3.Recursive│ ────────────────────▶ │ Root      │          │
│  │  Resolver │ ◀──────────────────── │ Server .  │          │
│  │  (ISP or  │  "Ask a.gtld.net"     └───────────┘          │
│  │  8.8.8.8) │                                               │
│  │           │  "Who handles         ┌───────────┐          │
│  │           │   example.com?"       │ TLD Server│          │
│  │           │ ────────────────────▶ │ .com      │          │
│  │           │ ◀──────────────────── │           │          │
│  │           │  "Ask ns1.example.com"└───────────┘          │
│  │           │                                               │
│  │           │  "What is the IP of   ┌───────────┐          │
│  │           │   app.example.com?"   │Authoritat.│          │
│  │           │ ────────────────────▶ │ NS        │          │
│  │           │ ◀──────────────────── │ns1.exampl │          │
│  │           │  "93.184.216.34"      └───────────┘          │
│  │           │  TTL: 300 seconds                             │
│  └─────┬─────┘                                               │
│        │ Cache result for TTL duration                       │
│        ▼                                                     │
│  ┌──────────┐                                                │
│  │ Browser  │ Connects to 93.184.216.34                      │
│  └──────────┘                                                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.4 DNS Cache Layers

```
┌──────────────────────────────────────────────────────────────┐
│              DNS CACHING LAYERS                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Layer 1: Application / Browser Cache                        │
│  ├── Chrome: chrome://net-internals/#dns                     │
│  ├── TTL: Usually follows DNS TTL                            │
│  └── Clear: Browser restart or manual flush                  │
│                                                              │
│  Layer 2: OS DNS Cache                                       │
│  ├── Linux: systemd-resolved or nscd                         │
│  ├── Windows: DNS Client service                             │
│  ├── Clear: sudo systemd-resolve --flush-caches              │
│  └── Clear: ipconfig /flushdns (Windows)                     │
│                                                              │
│  Layer 3: Local DNS Resolver (Router/Office)                 │
│  ├── Home router, corporate DNS                              │
│  └── TTL-based caching                                       │
│                                                              │
│  Layer 4: ISP / Recursive Resolver                           │
│  ├── Google (8.8.8.8), Cloudflare (1.1.1.1)                 │
│  ├── Most impactful cache layer                              │
│  └── Respects TTL from authoritative server                  │
│                                                              │
│  ⚠ Each layer can cause "stale" responses after changes!     │
│  Lower TTL = faster propagation but more DNS queries         │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.5 DNS Zones and Delegation

```
┌──────────────────────────────────────────────────────────────┐
│              DNS ZONES AND DELEGATION                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  example.com Zone (managed in Route 53 / CloudFlare)         │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ SOA   ns1.example.com admin.example.com ...            │  │
│  │ NS    ns1.example.com                                  │  │
│  │ NS    ns2.example.com                                  │  │
│  │ A     example.com         93.184.216.34                │  │
│  │ A     www.example.com     93.184.216.34                │  │
│  │ A     api.example.com     93.184.216.35                │  │
│  │ CNAME blog.example.com    myblog.wordpress.com         │  │
│  │ MX    example.com         mail.example.com (pri: 10)   │  │
│  │                                                        │  │
│  │ # Delegation to subdomain zone:                        │  │
│  │ NS    staging.example.com  ns1.staging.example.com     │  │
│  └────────────────────────────────────────────────────────┘  │
│                        │                                     │
│                   Delegation                                 │
│                        │                                     │
│  staging.example.com Zone (separate hosted zone)             │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ SOA   ns1.staging.example.com ...                      │  │
│  │ A     app.staging.example.com    10.0.1.50             │  │
│  │ A     api.staging.example.com    10.0.1.51             │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 1.6 DNS Investigation Commands

```bash
# Trace full resolution path
dig +trace app.example.com

# Query specific DNS server
dig @8.8.8.8 app.example.com

# See all record types
dig example.com ANY

# Check authoritative nameservers
dig NS example.com

# Check SOA record (zone info)
dig SOA example.com

# Reverse DNS lookup
dig -x 93.184.216.34

# Check TTL remaining
dig +noall +answer app.example.com

# nslookup alternative
nslookup -type=NS example.com
nslookup app.example.com 8.8.8.8

# Check DNS propagation across resolvers
for dns in 8.8.8.8 1.1.1.1 208.67.222.222 9.9.9.9; do
  echo "=== $dns ==="
  dig @$dns app.example.com +short
done
```

---

## 1.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| DNS changes not visible | Cached old record | Wait for TTL or flush cache |
| Different IPs from different resolvers | Propagation in progress | Check with `dig @<resolver>` |
| NXDOMAIN (domain not found) | Wrong NS records or zone misconfigured | Verify NS delegation at registrar |
| SERVFAIL | Authoritative server unreachable | Check NS server health |
| Slow resolution | No local cache, high TTL misses | Use low-latency resolver (1.1.1.1) |
| Subdomain not resolving | Missing NS delegation or record | Check parent zone NS records |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| Root Servers | 13 clusters, know TLD locations |
| TLD Servers | .com, .org, .io — know authoritative NS |
| Authoritative NS | Hold actual records for your domain |
| Recursive Resolver | Walks hierarchy on behalf of client |
| TTL | How long resolvers cache a record |
| Zone | Administrative boundary for DNS records |
| Delegation | NS records pointing subdomain to another zone |
| dig +trace | Shows every step of DNS resolution |

---

## Quick Revision Questions

1. **Describe the complete DNS resolution path for `api.myapp.io` starting from the browser.**
2. **What is the role of a recursive resolver vs an authoritative nameserver?**
3. **Why are there exactly 13 root server addresses? (Hint: it's not 13 physical servers)**
4. **How does DNS delegation work for subdomains like `staging.example.com`?**
5. **If you change a DNS record with TTL 3600, what's the maximum time before all clients see the change?**
6. **Use `dig +trace` to explain each hop in a DNS query.**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Service Endpoints](../04-Cloud-Networking/06-service-endpoints.md) | [README](../README.md) | [DNS Record Types →](02-dns-record-types.md) |
