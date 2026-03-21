# Zone Transfers

## Unit 3 - Topic 3 | Domain and DNS Reconnaissance

---

## Overview

A **DNS zone transfer** (AXFR) is a mechanism for replicating DNS records between nameservers. When misconfigured, it allows anyone to download the **complete DNS zone file** — revealing every subdomain, IP address, and DNS record for the target domain. This is one of the most valuable recon findings.

---

## 1. How Zone Transfers Work

```
NORMAL ZONE TRANSFER (Intended):

Primary DNS ──── AXFR ────► Secondary DNS
(ns1.target.com)            (ns2.target.com)
     │                           │
     └── Full zone file ────────►│
         replicated              │

EXPLOITED ZONE TRANSFER:

Primary DNS ──── AXFR ────► ATTACKER
(ns1.target.com)            (pen tester)
     │                           │
     └── Full zone file ────────►│
         leaked to attacker!     │

⚠️ This happens when the DNS server doesn't
   restrict who can request zone transfers
```

---

## 2. Attempting Zone Transfers

```bash
# === DIG ===
dig @ns1.target.com target.com axfr
dig @ns2.target.com target.com axfr

# First, find nameservers:
dig target.com NS
# Then try AXFR against each nameserver

# === HOST ===
host -l target.com ns1.target.com

# === NSLOOKUP ===
nslookup
> server ns1.target.com
> set type=any
> ls -d target.com

# === DNSRECON ===
dnsrecon -d target.com -t axfr

# === DNSENUM ===
dnsenum target.com

# === FIERCE ===
fierce --domain target.com
```

---

## 3. Zone Transfer Output Example

```
# SUCCESSFUL ZONE TRANSFER OUTPUT:

; <<>> DiG AXFR target.com @ns1.target.com <<>>
target.com.          IN  SOA   ns1.target.com. admin.target.com. (
                               2024011501 ; serial
                               3600       ; refresh
                               900        ; retry
                               604800     ; expire
                               86400 )    ; minimum TTL

target.com.          IN  A     203.0.113.10
target.com.          IN  MX    10 mail.target.com.
target.com.          IN  NS    ns1.target.com.
target.com.          IN  NS    ns2.target.com.

; SUBDOMAINS REVEALED:
www.target.com.      IN  A     203.0.113.10
mail.target.com.     IN  A     203.0.113.20
vpn.target.com.      IN  A     203.0.113.30
admin.target.com.    IN  A     203.0.113.11     ← Admin panel!
dev.target.com.      IN  A     203.0.113.40     ← Dev server!
staging.target.com.  IN  A     203.0.113.41     ← Staging!
db.target.com.       IN  A     10.0.0.50        ← Internal DB!
intranet.target.com. IN  A     10.0.0.100       ← Intranet!
jenkins.target.com.  IN  A     203.0.113.50     ← CI/CD!
grafana.target.com.  IN  A     203.0.113.51     ← Monitoring!

; TOTAL RECORDS: 47 (complete zone dump)
```

---

## 4. Impact and Defense

| Aspect | Detail |
|--------|--------|
| **Severity** | Medium-High |
| **Impact** | Complete DNS infrastructure exposed |
| **What's revealed** | All subdomains, IPs, internal names |
| **Detection** | DNS server logs show AXFR requests |

```
DEFENSE — Restrict Zone Transfers:

BIND (named.conf):
  options {
      allow-transfer { 203.0.113.20; };   // Only to secondary DNS
  };

Windows DNS:
  DNS Manager → Zone Properties → Zone Transfers
  → "Only to servers listed on the Name Servers tab"

VERIFICATION:
  After fixing, test from external:
  dig @ns1.target.com target.com axfr
  Expected: "Transfer failed" or "REFUSED"
```

---

## 5. When Zone Transfer Fails

```bash
# If AXFR fails (most modern servers block it):
# Use alternative subdomain discovery methods:

# 1. Certificate Transparency
curl "https://crt.sh/?q=%25.target.com&output=json" | jq '.[].name_value'

# 2. Subdomain brute force
gobuster dns -d target.com -w subdomains.txt
dnsrecon -d target.com -t brt -D wordlist.txt

# 3. Passive DNS databases
subfinder -d target.com
amass enum -passive -d target.com

# 4. Search engines
# Google: site:*.target.com -www
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Zone transfer** | AXFR — replicate entire DNS zone |
| **Vulnerability** | Unrestricted AXFR leaks all records |
| **Test** | dig @ns target.com axfr |
| **Reveals** | All subdomains, IPs, internal names |
| **Defense** | Restrict AXFR to authorized servers only |
| **Alternative** | If blocked, use crt.sh, brute force, passive DNS |

---

## Quick Revision Questions

1. **What is a DNS zone transfer?**
2. **Why is an unrestricted zone transfer a security issue?**
3. **How do you attempt a zone transfer?**
4. **What alternatives exist if zone transfer is blocked?**
5. **How do you defend against unauthorized zone transfers?**

---

[← Previous: DNS Enumeration](02-dns-enumeration.md) | [Next: Subdomain Discovery →](04-subdomain-discovery.md)

---

*Unit 3 - Topic 3 of 6 | [Back to README](../README.md)*
