# Chapter 4: DNS Configuration

[← Previous: Network Commands](03-network-commands.md) | [Back to README](../README.md) | [Next: Firewall Configuration →](05-firewall-configuration.md)

---

## Overview

DNS (Domain Name System) translates human-readable domain names into IP addresses. For DevOps, proper DNS configuration is crucial for service discovery, load balancing, internal naming, and certificate validation.

---

## 4.1 DNS Resolution on Linux

```
┌──────────────────────────────────────────────────────────────┐
│            DNS RESOLUTION ORDER IN LINUX                      │
│                                                                │
│  Application calls getaddrinfo()                               │
│       │                                                        │
│       ▼                                                        │
│  /etc/nsswitch.conf    ← Defines lookup order                 │
│  hosts: files dns       (files first, then DNS)               │
│       │                                                        │
│       ├──→ /etc/hosts           ← Static mappings             │
│       │    127.0.0.1 localhost                                 │
│       │    192.168.1.10 app.local                              │
│       │                                                        │
│       └──→ /etc/resolv.conf    ← DNS servers                  │
│            nameserver 8.8.8.8                                  │
│            nameserver 8.8.4.4                                  │
│            search company.internal                             │
│                 │                                              │
│                 ▼                                              │
│            systemd-resolved     ← Local DNS cache/stub        │
│            (127.0.0.53)         (if enabled)                  │
│                 │                                              │
│                 ▼                                              │
│            External DNS Server                                 │
└──────────────────────────────────────────────────────────────┘
```

---

## 4.2 Key Configuration Files

### `/etc/hosts`

```bash
# Static hostname-to-IP mappings
# Checked BEFORE DNS (per nsswitch.conf)
127.0.0.1       localhost
127.0.1.1       myserver
::1             localhost ip6-localhost

# Custom entries (useful for development)
192.168.1.10    db.local
192.168.1.20    app.local
192.168.1.30    cache.local

# Block domains (redirect to nowhere)
0.0.0.0         ads.trackersite.com
```

### `/etc/resolv.conf`

```bash
# DNS resolver configuration
nameserver 8.8.8.8          # Primary DNS
nameserver 8.8.4.4          # Secondary DNS
nameserver 1.1.1.1          # Tertiary

search company.internal     # Default search domain
# "ping app" tries app.company.internal first

domain company.internal     # Default domain (one only)

options timeout:2           # Query timeout
options attempts:3          # Retry count
options ndots:2             # Dots before absolute lookup
```

### `/etc/nsswitch.conf`

```bash
# Name Service Switch — controls resolution order
hosts: files dns myhostname
#       │     │     │
#       │     │     └── systemd hostname
#       │     └──────── DNS servers (/etc/resolv.conf)
#       └────────────── /etc/hosts file
```

---

## 4.3 `systemd-resolved`

```bash
# Modern DNS management (Ubuntu 18.04+)
systemctl status systemd-resolved

# View current DNS settings
resolvectl status
resolvectl dns                    # Current DNS servers

# Set DNS for an interface
sudo resolvectl dns eth0 8.8.8.8 8.8.4.4
sudo resolvectl domain eth0 company.internal

# Flush DNS cache
sudo resolvectl flush-caches
resolvectl statistics             # Cache hit/miss stats

# Check resolution
resolvectl query google.com
```

```
┌──────────────────────────────────────────────────────────────┐
│        resolv.conf WITH systemd-resolved                      │
│                                                                │
│  /etc/resolv.conf → symlink to resolved's stub                │
│  nameserver 127.0.0.53    ← systemd-resolved stub listener   │
│                                                                │
│  Actual DNS configured via:                                    │
│  • netplan (nameservers section)                              │
│  • nmcli (ipv4.dns)                                           │
│  • resolvectl dns                                             │
│                                                                │
│  To bypass resolved and use direct DNS:                       │
│  sudo rm /etc/resolv.conf                                     │
│  sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'│
└──────────────────────────────────────────────────────────────┘
```

---

## 4.4 DNS Record Types

```
┌──────────────────────────────────────────────────────────────┐
│               DNS RECORD TYPES                                │
├────────┬─────────────────────────────────────────────────────┤
│ Type   │ Purpose & Example                                   │
├────────┼─────────────────────────────────────────────────────┤
│ A      │ IPv4 address                                        │
│        │ example.com → 93.184.216.34                         │
├────────┼─────────────────────────────────────────────────────┤
│ AAAA   │ IPv6 address                                        │
│        │ example.com → 2606:2800:220:1:...                   │
├────────┼─────────────────────────────────────────────────────┤
│ CNAME  │ Alias to another domain                             │
│        │ www.example.com → example.com                       │
├────────┼─────────────────────────────────────────────────────┤
│ MX     │ Mail server (with priority)                         │
│        │ example.com → 10 mail.example.com                   │
├────────┼─────────────────────────────────────────────────────┤
│ NS     │ Authoritative name server                           │
│        │ example.com → ns1.example.com                       │
├────────┼─────────────────────────────────────────────────────┤
│ TXT    │ Text record (SPF, DKIM, verification)               │
│        │ example.com → "v=spf1 include:_spf.google.com ~all" │
├────────┼─────────────────────────────────────────────────────┤
│ SRV    │ Service location (port + host)                      │
│        │ _http._tcp.example.com → 0 5 80 www.example.com    │
├────────┼─────────────────────────────────────────────────────┤
│ PTR    │ Reverse DNS (IP → hostname)                         │
│        │ 34.216.184.93.in-addr.arpa → example.com            │
├────────┼─────────────────────────────────────────────────────┤
│ SOA    │ Start of Authority (zone info)                      │
│        │ Primary NS, admin email, serial, refresh            │
└────────┴─────────────────────────────────────────────────────┘
```

---

## 4.5 DNS Lookup Tools

```bash
# dig — most detailed
dig example.com                    # Full query
dig +short example.com             # Just the answer
dig example.com MX                 # Specific record type
dig @8.8.8.8 example.com          # Query specific server
dig +trace example.com             # Full delegation trace
dig -x 93.184.216.34              # Reverse lookup

# nslookup — interactive/simple
nslookup example.com
nslookup -type=MX example.com
nslookup example.com 8.8.8.8

# host — simplest
host example.com
host -t MX example.com
host 93.184.216.34               # Reverse lookup

# getent — uses system resolver (respects /etc/hosts)
getent hosts example.com
getent ahosts example.com
```

---

## 4.6 Real-World Scenario: Internal DNS for Microservices

```bash
# /etc/hosts approach (simple, limited)
# On each server, add:
192.168.1.10    api-gateway.internal
192.168.1.11    user-service.internal
192.168.1.12    order-service.internal
192.168.1.13    payment-service.internal
192.168.1.20    postgres.internal
192.168.1.21    redis.internal

# Verify resolution
ping -c 1 api-gateway.internal

# Using search domain for shorter names
# /etc/resolv.conf
search internal
# Now "ping api-gateway" resolves to api-gateway.internal

# Docker Compose DNS (automatic)
# docker-compose.yml
# services:
#   api:
#     ...
#   db:
#     image: postgres
# Container "api" can reach "db" by name automatically
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Cannot resolve hostname | `dig domain` fails | Check `/etc/resolv.conf` nameserver |
| Resolves wrong IP | Stale cache or hosts entry | Flush cache; check `/etc/hosts` |
| Slow DNS resolution | `dig` shows high query time | Change to faster DNS (8.8.8.8, 1.1.1.1) |
| `resolv.conf` resets on reboot | Managed by DHCP/resolved | Edit via Netplan or nmcli instead |
| Works with IP but not hostname | DNS broken, network fine | Fix DNS configuration |
| Internal hostname not resolving | Missing search domain | Add `search domain.internal` in resolv.conf |

---

## Summary Table

| File/Tool | Purpose |
|-----------|---------|
| `/etc/hosts` | Static local hostname mappings |
| `/etc/resolv.conf` | DNS server configuration |
| `/etc/nsswitch.conf` | Resolution order (files, dns) |
| `dig` | Detailed DNS queries |
| `nslookup` | Simple DNS lookup |
| `host` | Quick DNS check |
| `resolvectl` | systemd-resolved management |

---

## Quick Revision Questions

1. **In what order does Linux resolve hostnames by default?**
2. **What is the purpose of the `search` directive in `/etc/resolv.conf`?**
3. **How do you query a specific DNS server using `dig`?**
4. **What is the difference between an A record and a CNAME record?**
5. **Why does `/etc/resolv.conf` show `nameserver 127.0.0.53` on modern Ubuntu?**
6. **How would you set up internal DNS names for a development environment without a DNS server?**

---

[← Previous: Network Commands](03-network-commands.md) | [Back to README](../README.md) | [Next: Firewall Configuration →](05-firewall-configuration.md)
