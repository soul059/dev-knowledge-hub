# Service Fingerprinting

## Unit 2 - Topic 5 | Service Enumeration

---

## Overview

**Service fingerprinting** identifies services running on non-standard ports, services hiding behind different protocols, or services that don't provide clear banners. It uses behavioral analysis of how a service responds to various probes.

---

## 1. Why Service Fingerprinting?

```
SERVICES DON'T ALWAYS RUN ON STANDARD PORTS:

EXPECTED:
Port 22  → SSH
Port 80  → HTTP
Port 443 → HTTPS

REALITY (common in pentests):
Port 22    → HTTP (admin moved SSH to 2222)
Port 8443  → SSH (hidden SSH on HTTPS port)
Port 443   → VPN (OpenVPN over HTTPS port)
Port 53    → DNS tunnel (C2 channel)
Port 80    → Reverse shell listener

BANNER SUPPRESSION:
├── Admin disabled banner: nc -nv target 22 → (nothing)
├── Custom banner: "Welcome to server" (no version)
├── Firewall strips banners
└── Service fingerprinting still identifies it!
```

---

## 2. Fingerprinting Techniques

```bash
# === NMAP SERVICE FINGERPRINTING ===
nmap -sV target.com
# Uses protocol-specific probes regardless of port
# Port 8080 returns HTTP response → identified as HTTP

nmap -sV --version-all target.com
# Maximum intensity — sends all probes to all ports

nmap -sV --allports target.com
# Don't skip any ports (default skips port 9100)

# === AMAP (Application Mapper) ===
amap target.com 8443
# Sends application protocol probes
# Identifies: HTTP, SSH, SMTP, etc. on any port

# === MANUAL FINGERPRINTING ===
# Connect and observe behavior:
nc -nv target.com 8443
# If it responds to "GET / HTTP/1.0" → HTTP
# If it shows "SSH-2.0-" → SSH
# If it shows binary data → likely encrypted/binary protocol

# === PROTOCOL-SPECIFIC PROBES ===
# HTTP probe:
echo "GET / HTTP/1.0\r\n\r\n" | nc target.com 8080
# SSH probe:
echo "" | nc -w 3 target.com 2222
# SMTP probe:
echo "EHLO test" | nc target.com 2525
```

---

## 3. SSL/TLS Fingerprinting

```bash
# === SSL CERTIFICATE ANALYSIS ===
openssl s_client -connect target.com:443 < /dev/null 2>/dev/null | \
  openssl x509 -noout -text
# Reveals: organization, CN, SAN (subdomains), issuer, dates

# === JARM FINGERPRINT ===
# JARM creates a fingerprint of TLS server configuration
# Same software = same JARM hash
python3 jarm.py target.com
# JARM: 2ad2ad16d2ad2ad22c2ad2ad2ad2ad...
# Can identify: Cobalt Strike, Metasploit, specific web servers

# === JA3/JA3S FINGERPRINTING ===
# JA3: Client TLS fingerprint
# JA3S: Server TLS fingerprint
# Identifies applications by their TLS behavior
# Used for: malware detection, C2 identification

# === NMAP SSL SCRIPTS ===
nmap -p 443 --script ssl-cert target.com           # Certificate info
nmap -p 443 --script ssl-enum-ciphers target.com   # Cipher suites
```

---

## 4. Identifying Services on Non-Standard Ports

| Behavior | Likely Service |
|----------|---------------|
| Responds to `GET / HTTP/1.0` | Web server (HTTP) |
| Sends `SSH-2.0-` on connect | SSH |
| Sends `220 ` on connect | FTP or SMTP |
| Binary handshake | RDP, VNC, or binary protocol |
| TLS negotiation | HTTPS, SMTPS, or any TLS-wrapped service |
| Responds to `EHLO` | SMTP |
| Responds to `USER anonymous` | FTP |

```bash
# === QUICK TEST SCRIPT ===
for port in 2222 8080 8443 9090; do
    echo "[*] Testing port $port"
    
    # Try HTTP
    response=$(echo "GET / HTTP/1.0\r\nHost: target.com\r\n\r\n" | \
      nc -w 3 target.com $port 2>/dev/null | head -1)
    
    if echo "$response" | grep -q "HTTP"; then
        echo "  → HTTP service detected"
    elif echo "$response" | grep -q "SSH"; then
        echo "  → SSH service detected"
    elif echo "$response" | grep -q "220"; then
        echo "  → FTP/SMTP service detected"
    else
        echo "  → Unknown: $response"
    fi
done
```

---

## 5. Fingerprint Databases

| Database | Purpose |
|----------|---------|
| **nmap-service-probes** | Nmap's probe/response patterns |
| **JARM** | TLS server fingerprint database |
| **JA3** | TLS client fingerprint database |
| **Shodan** | Internet-wide fingerprint data |
| **Censys** | Certificate and service fingerprints |
| **HASSH** | SSH client/server fingerprints |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Why fingerprint** | Services on non-standard ports |
| **Nmap -sV** | Protocol probes identify services |
| **JARM** | TLS configuration fingerprint |
| **JA3/JA3S** | TLS handshake fingerprinting |
| **Manual** | Send protocol-specific probes |
| **Non-standard** | Always test ports for unexpected services |

---

## Quick Revision Questions

1. **Why is service fingerprinting needed beyond banner grabbing?**
2. **How does Nmap identify services on non-standard ports?**
3. **What is a JARM fingerprint?**
4. **How do you manually test what service runs on an unknown port?**
5. **What is the difference between JA3 and JA3S?**

---

[← Previous: Protocol-Specific Enumeration](04-protocol-specific-enumeration.md) | [Next: Vulnerability Correlation →](06-vulnerability-correlation.md)

---

*Unit 2 - Topic 5 of 6 | [Back to README](../README.md)*
