# Active Reconnaissance

## Unit 5 - Topic 2 | Reconnaissance Phase

---

## Overview

**Active reconnaissance** involves directly interacting with target systems to gather information. Unlike passive recon, active recon **sends packets** to the target and **can be detected** by firewalls, IDS/IPS, and monitoring systems. It requires **authorization** and is part of the in-scope testing activities.

---

## 1. Active Recon Techniques

| Technique | Description | Detectability |
|-----------|-------------|:---:|
| **DNS queries** | Direct queries to target DNS | Low |
| **Port scanning** | Discover open ports | Medium-High |
| **Service enumeration** | Identify running services | Medium |
| **Banner grabbing** | Capture service version info | Medium |
| **Web crawling** | Spider the website | Medium |
| **Directory bruteforce** | Find hidden directories | High |
| **Vulnerability scanning** | Automated vuln detection | Very High |

---

## 2. DNS Enumeration

```bash
# === DNS ZONE TRANSFER (often blocked) ===
dig axfr @ns1.target.com target.com
# If successful: reveals ALL DNS records!

# === DNS BRUTE FORCE ===
# Enumerate subdomains
dnsrecon -d target.com -t brt -D /usr/share/wordlists/subdomains.txt
fierce --domain target.com
dnsenum target.com

# === REVERSE DNS ===
dig -x 203.0.113.1                    # IP to hostname
nmap -sL 203.0.113.0/24              # Reverse DNS sweep

# === DNS TOOLS ===
dnsrecon -d target.com               # Comprehensive DNS recon
dnsenum target.com                   # DNS enumeration
amass enum -d target.com             # Advanced subdomain discovery
subfinder -d target.com              # Fast subdomain enumeration
```

---

## 3. Network Discovery

```bash
# === HOST DISCOVERY ===
nmap -sn 10.0.0.0/24                 # Ping sweep
nmap -sn -PE 10.0.0.0/24             # ICMP echo
nmap -sn -PA 10.0.0.0/24             # TCP ACK (bypasses ICMP block)
nmap -sn -PS22,80,443 10.0.0.0/24    # TCP SYN to specific ports

# === ARP DISCOVERY (internal only) ===
arp-scan --interface=eth0 --localnet
netdiscover -i eth0 -r 10.0.0.0/24

# === PORT SCANNING ===
nmap -sS 10.0.0.1                    # SYN scan (stealth)
nmap -sT 10.0.0.1                    # TCP connect scan
nmap -sU 10.0.0.1                    # UDP scan
nmap -p- 10.0.0.1                    # All 65535 ports
nmap -sV 10.0.0.1                    # Version detection
nmap -A 10.0.0.1                     # Aggressive (OS + version + scripts)

# === SERVICE ENUMERATION ===
nmap -sV -sC 10.0.0.1                # Version + default scripts
nmap --script vuln 10.0.0.1          # Vulnerability scripts
```

---

## 4. Web Enumeration

```bash
# === WEB TECHNOLOGY FINGERPRINTING ===
whatweb https://target.com            # Technology detection
wappalyzer                            # Browser extension
nikto -h https://target.com           # Web server scanner

# === DIRECTORY/FILE DISCOVERY ===
gobuster dir -u https://target.com -w /usr/share/wordlists/dirb/common.txt
ffuf -u https://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
dirb https://target.com
dirsearch -u https://target.com

# === WEB CRAWLING ===
gospider -s https://target.com -d 3   # Crawl 3 levels deep
hakrawler -url https://target.com     # URL extraction

# === ROBOTS.TXT and SITEMAP ===
curl https://target.com/robots.txt
curl https://target.com/sitemap.xml

# === SSL/TLS ANALYSIS ===
sslscan target.com
testssl.sh target.com
nmap --script ssl-enum-ciphers -p 443 target.com
```

---

## 5. Staying Under the Radar

```
EVASION TECHNIQUES:

TIMING:
├── Slow scans (nmap -T1 or -T2)
├── Randomize target order
├── Spread scans over time
└── Test during high-traffic periods

TECHNIQUE:
├── Fragment packets (nmap -f)
├── Use decoys (nmap -D)
├── Spoof source port (--source-port 53)
├── Use common user agents for web enum
└── Avoid aggressive scripts

⚠️ NOTE:
├── In a standard pen test, stealth is NOT usually required
├── Stealth is important for red team engagements
├── Always balance thoroughness vs stealth
└── More evasion = slower testing = higher cost
```

---

## Summary Table

| Technique | Tool | Detectability |
|-----------|------|:---:|
| **DNS enumeration** | dnsrecon, amass | Low |
| **Port scanning** | nmap | Medium-High |
| **Service detection** | nmap -sV | Medium |
| **Web directories** | gobuster, ffuf | High |
| **Vulnerability scan** | nikto, nmap NSE | Very High |
| **SSL analysis** | sslscan, testssl | Low |

---

## Quick Revision Questions

1. **What makes active recon different from passive recon?**
2. **What is a DNS zone transfer and why is it valuable?**
3. **How do you discover live hosts on a network?**
4. **Name 3 tools for web directory enumeration.**
5. **When is stealth important in active recon?**

---

[← Previous: Passive Reconnaissance](01-passive-reconnaissance.md) | [Next: OSINT Techniques →](03-osint-techniques.md)

---

*Unit 5 - Topic 2 of 6 | [Back to README](../README.md)*
