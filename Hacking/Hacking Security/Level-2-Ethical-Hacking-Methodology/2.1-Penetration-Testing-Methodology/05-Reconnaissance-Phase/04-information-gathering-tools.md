# Information Gathering Tools

## Unit 5 - Topic 4 | Reconnaissance Phase

---

## Overview

This topic consolidates the **essential tools** used in the reconnaissance phase. From domain enumeration to infrastructure mapping, these tools form the pen tester's reconnaissance toolkit. Mastering them ensures thorough information gathering before exploitation.

---

## 1. Tool Categories

```
┌──────────────────────────────────────────────────────────────┐
│              RECON TOOL CATEGORIES                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  DOMAIN/DNS          │ WHOIS, dig, nslookup, dnsrecon       │
│  SUBDOMAIN           │ amass, subfinder, sublist3r           │
│  WEB TECH            │ whatweb, wappalyzer, builtwith        │
│  EMAIL               │ theHarvester, hunter.io               │
│  INFRASTRUCTURE      │ Shodan, Censys, nmap                  │
│  SOCIAL              │ Maltego, Sherlock, linkedin2username  │
│  CODE                │ Trufflehog, GitDorker, gitrob         │
│  AUTOMATION          │ SpiderFoot, Recon-ng, Osmedeus        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Essential Tools Reference

### theHarvester

```bash
# Multi-source OSINT gatherer
theHarvester -d target.com -b google           # Google
theHarvester -d target.com -b linkedin         # LinkedIn
theHarvester -d target.com -b all              # All sources
theHarvester -d target.com -b all -f report    # Save to file

# Sources: google, bing, linkedin, twitter, github, 
#          shodan, censys, dnsdumpster, crtsh, and more
```

### Amass

```bash
# Advanced subdomain enumeration
amass enum -d target.com                        # Basic enum
amass enum -d target.com -active                # Active + passive
amass enum -d target.com -brute -w wordlist.txt # Brute force
amass viz -d target.com                         # Visualization
amass db -names -d target.com                   # View stored results
```

### Recon-ng

```bash
recon-ng
> workspaces create target
> modules search
> modules load recon/domains-hosts/hackertarget
> options set SOURCE target.com
> run
> modules load recon/hosts-ports/shodan_ip
> run
> show hosts
> show ports
```

---

## 3. Quick Reference Table

| Task | Tool | Command |
|------|------|---------|
| WHOIS | whois | `whois target.com` |
| DNS records | dig | `dig target.com ANY` |
| Subdomains | subfinder | `subfinder -d target.com` |
| Subdomains | amass | `amass enum -d target.com` |
| SSL certs | crt.sh | `curl "https://crt.sh/?q=%.target.com"` |
| Web tech | whatweb | `whatweb https://target.com` |
| Emails | theHarvester | `theHarvester -d target.com -b all` |
| IP info | Shodan | `shodan host 1.2.3.4` |
| Username | Sherlock | `sherlock username` |
| Port scan | nmap | `nmap -sV target.com` |
| Web scan | nikto | `nikto -h https://target.com` |
| Directories | gobuster | `gobuster dir -u https://target.com -w list.txt` |
| Git secrets | trufflehog | `trufflehog github --org=target` |
| Metadata | exiftool | `exiftool document.pdf` |
| Automation | SpiderFoot | `spiderfoot -l 127.0.0.1:5001` |

---

## 4. Tool Selection Strategy

```
RECON WORKFLOW:

STEP 1: Passive (No detection risk)
├── WHOIS, DNS, crt.sh, Google dorks
├── theHarvester, Shodan, social media
└── Output: Domain map, email list, tech stack

STEP 2: Semi-passive (Low detection risk)
├── Web crawling, robots.txt, sitemap
├── Wayback Machine, Google cache
└── Output: Content map, historical data

STEP 3: Active (Detectable)
├── Port scanning (nmap)
├── Directory enumeration (gobuster)
├── Service fingerprinting (nmap -sV)
└── Output: Open ports, services, vulnerabilities

STEP 4: Consolidate
├── Merge all findings
├── Create target map
├── Prioritize targets
└── Plan exploitation
```

---

## 5. Recon Output Organization

```bash
# ORGANIZE YOUR FINDINGS:

mkdir -p recon/{domains,ips,emails,subdomains,screenshots,scans}

# Domain info
whois target.com > recon/domains/whois.txt

# Subdomains
subfinder -d target.com -o recon/subdomains/subfinder.txt
amass enum -d target.com -o recon/subdomains/amass.txt
cat recon/subdomains/*.txt | sort -u > recon/subdomains/all_subdomains.txt

# Emails
theHarvester -d target.com -b all -f recon/emails/harvester

# Port scans
nmap -sV -oA recon/scans/initial_scan target.com

# Screenshots of web apps
gowitness file -f recon/subdomains/all_subdomains.txt -P recon/screenshots/
```

---

## Summary Table

| Category | Primary Tool | Backup Tool |
|----------|:---:|:---:|
| **Subdomains** | Amass | Subfinder |
| **Emails** | theHarvester | Hunter.io |
| **Port scan** | Nmap | Masscan |
| **Web tech** | Whatweb | Wappalyzer |
| **Directories** | Gobuster | ffuf |
| **OSINT automation** | SpiderFoot | Recon-ng |

---

## Quick Revision Questions

1. **What is theHarvester used for?**
2. **How does amass differ from subfinder?**
3. **What is the recommended recon workflow order?**
4. **How should recon output be organized?**
5. **Name a tool for each: subdomains, emails, ports, directories.**

---

[← Previous: OSINT Techniques](03-osint-techniques.md) | [Next: Target Profiling →](05-target-profiling.md)

---

*Unit 5 - Topic 4 of 6 | [Back to README](../README.md)*
