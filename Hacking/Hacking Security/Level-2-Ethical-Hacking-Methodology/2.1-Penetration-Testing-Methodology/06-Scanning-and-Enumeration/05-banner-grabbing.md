# Banner Grabbing

## Unit 6 - Topic 5 | Scanning and Enumeration

---

## Overview

**Banner grabbing** is the technique of connecting to a service and reading the **banner** it returns — a text string identifying the software name, version, and sometimes the operating system. This information helps identify **specific vulnerabilities** and **exploits** for the target service.

---

## 1. What Banners Reveal

```
EXAMPLE BANNERS:

SSH:    SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.1
HTTP:   Server: Apache/2.4.49 (Unix)
FTP:    220 (vsFTPd 3.0.5)
SMTP:   220 mail.target.com ESMTP Postfix (Ubuntu)
MySQL:  5.7.38-0ubuntu0.18.04.1
Redis:  redis_version:7.0.5

FROM THESE BANNERS:
├── Software name → Known vulnerabilities
├── Version number → Specific CVEs
├── OS information → Attack customization
└── Configuration → Potential misconfigurations
```

---

## 2. Banner Grabbing Techniques

```bash
# === NETCAT ===
nc -v 10.0.0.5 22                     # SSH banner
nc -v 10.0.0.5 80                     # HTTP (then type: GET / HTTP/1.0)
nc -v 10.0.0.5 25                     # SMTP banner
nc -v 10.0.0.5 21                     # FTP banner

# === TELNET ===
telnet 10.0.0.5 22                    # SSH banner
telnet 10.0.0.5 80                    # HTTP
# GET / HTTP/1.0
# Host: target.com
# (press Enter twice)

# === NMAP ===
nmap -sV 10.0.0.5                     # Version detection (best)
nmap -sV --version-intensity 9 10.0.0.5  # Aggressive version detection
nmap --script banner 10.0.0.5          # Banner grabbing script

# === CURL (HTTP) ===
curl -I https://target.com             # HTTP headers only
curl -v https://target.com 2>&1 | grep -i server

# === OPENSSL (HTTPS) ===
openssl s_client -connect target.com:443
# Shows: Certificate info, cipher, TLS version

# === WHATWEB ===
whatweb https://target.com             # Web technology detection
```

---

## 3. Active vs Passive Banner Grabbing

| Method | Description | Detection Risk |
|--------|-------------|:---:|
| **Active** | Connect directly to service | Detectable |
| **Passive** | Capture traffic (sniffing) | Undetectable |
| **Shodan/Censys** | Search cached banners | Undetectable |

```bash
# PASSIVE: Using Shodan (cached banners)
shodan host 203.0.113.10              # View stored banners
shodan search "hostname:target.com"   # Search by hostname

# PASSIVE: Using Censys
# censys.io — search for target IP

# PASSIVE: Network sniffing
sudo tcpdump -i eth0 -A port 80       # Capture HTTP headers
```

---

## 4. Hiding and Faking Banners

```bash
# DEFENSE: Servers can modify or hide banners

# Apache — hide version
ServerTokens Prod                      # Shows "Apache" only
ServerSignature Off                    # Remove from error pages

# Nginx — hide version
server_tokens off;                     # Remove version from headers

# SSH — custom banner
# /etc/ssh/sshd_config
Banner /etc/ssh/banner.txt            # Custom login banner

# ⚠️ PEN TESTER NOTE:
# Banner may be:
# • Accurate (most common)
# • Modified/hidden (security-conscious targets)
# • Fake/misleading (honeypots)
# Always validate with additional techniques!
```

---

## 5. Using Banners for Exploitation

```bash
# WORKFLOW: Banner → CVE → Exploit

# 1. Grab banner
nmap -sV -p 80 10.0.0.5
# Output: Apache/2.4.49

# 2. Search for CVEs
searchsploit apache 2.4.49
# Found: Apache 2.4.49 - Path Traversal (CVE-2021-41773)

# 3. Verify vulnerability
curl 'http://10.0.0.5/cgi-bin/.%2e/%2e%2e/%2e%2e/etc/passwd'

# 4. Exploit if vulnerable
# ... proceed with exploitation phase
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Banner** | Text identifying service name and version |
| **Active grabbing** | nc, telnet, nmap -sV, curl |
| **Passive grabbing** | Shodan, Censys, packet sniffing |
| **Purpose** | Map versions to known CVEs |
| **Caveat** | Banners can be modified or faked |
| **Next step** | searchsploit to find exploits |

---

## Quick Revision Questions

1. **What information does a service banner typically reveal?**
2. **Name 3 tools for banner grabbing.**
3. **How can you grab banners passively?**
4. **Why might a banner be inaccurate?**
5. **How do you use banner information to find exploits?**

---

[← Previous: Vulnerability Scanning](04-vulnerability-scanning.md) | [Next: OS Fingerprinting →](06-os-fingerprinting.md)

---

*Unit 6 - Topic 5 of 6 | [Back to README](../README.md)*
