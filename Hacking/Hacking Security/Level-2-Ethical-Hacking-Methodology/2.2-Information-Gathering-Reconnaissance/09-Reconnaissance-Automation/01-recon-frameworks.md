# Recon Frameworks

## Unit 9 - Topic 1 | Reconnaissance Automation

---

## Overview

**Recon frameworks** are comprehensive tools that automate multiple reconnaissance tasks into a single workflow — subdomain enumeration, port scanning, technology detection, vulnerability scanning, and more. They orchestrate individual tools and aggregate results for efficient information gathering.

---

## 1. Popular Recon Frameworks

| Framework | Language | Speciality |
|-----------|----------|-----------|
| **Recon-ng** | Python | Modular OSINT framework |
| **SpiderFoot** | Python | Automated OSINT with 200+ modules |
| **theHarvester** | Python | Email, subdomain, IP harvesting |
| **Amass** | Go | Subdomain enumeration + network mapping |
| **Reconftw** | Bash | Full automated recon pipeline |
| **LazyRecon** | Bash | Automated subdomain + scanning |
| **FinalRecon** | Python | All-in-one web recon |
| **Osmedeus** | Go | Automated offensive recon |

---

## 2. Recon-ng

```bash
# === RECON-NG OVERVIEW ===
# Modular framework (similar to Metasploit)
# Built-in database for storing results
# 100+ modules for different recon tasks

# Start Recon-ng:
recon-ng

# === CREATE WORKSPACE ===
workspaces create target_recon
workspaces select target_recon

# === INSTALL MODULES ===
marketplace search
marketplace install recon/domains-hosts/hackertarget
marketplace install recon/hosts-hosts/resolve
marketplace install recon/domains-contacts/whois_pocs

# === ADD TARGET DOMAIN ===
db insert domains
# domain (TEXT): target.com

# === RUN MODULES ===
modules load recon/domains-hosts/hackertarget
run
# Discovers subdomains using HackerTarget API

modules load recon/hosts-hosts/resolve
run
# Resolves all discovered hosts to IPs

# === VIEW RESULTS ===
show hosts
show contacts
show domains

# === REPORTING ===
modules load reporting/html
options set FILENAME /tmp/recon_report.html
run
```

---

## 3. Amass

```bash
# === AMASS — Advanced Subdomain Enumeration ===

# Passive enumeration (OSINT only):
amass enum -passive -d target.com -o amass_passive.txt

# Active enumeration (DNS queries):
amass enum -active -d target.com -o amass_active.txt

# Full enumeration:
amass enum -d target.com -o amass_full.txt

# Intel module — find associated organizations:
amass intel -org "Target Corporation"
# Returns: ASNs, domains, IP ranges

# Visualization:
amass viz -d3 -d target.com
# Creates interactive D3.js visualization

# Database queries:
amass db -names -d target.com
# List all discovered names from database

# Track changes over time:
amass track -d target.com
# Shows new/removed subdomains since last scan
```

---

## 4. Reconftw (Full Pipeline)

```bash
# === RECONFTW — Full Automated Recon ===
# Combines 30+ tools into one pipeline

# Full recon:
./reconftw.sh -d target.com -r

# The pipeline runs:
# 1. Subdomain enumeration (Amass, Subfinder, etc.)
# 2. Resolution and probing (httpx, dnsx)
# 3. Web screenshot (Aquatone, gowitness)
# 4. Port scanning (Nmap, Naabu)
# 5. Technology detection (Wappalyzer)
# 6. URL extraction (gau, waybackurls)
# 7. JavaScript analysis (LinkFinder)
# 8. Vulnerability scanning (Nuclei)
# 9. Parameter fuzzing (Arjun)
# 10. Report generation

# Output structure:
# results/target.com/
# ├── subdomains/
# ├── webs/
# ├── hosts/
# ├── screenshots/
# ├── nuclei/
# ├── vulns/
# └── report/
```

---

## 5. SpiderFoot

```bash
# === SPIDERFOOT — Automated OSINT ===
# 200+ modules for comprehensive intelligence gathering

# Start web UI:
python3 sf.py -l 127.0.0.1:5001

# CLI mode:
python3 sfcli.py -s target.com -m all

# Modules include:
# ├── Domain → subdomains, DNS, WHOIS
# ├── Email → harvesting, breach check
# ├── Network → IP ranges, ports, ASN
# ├── Social → LinkedIn, Twitter, GitHub
# ├── Dark web → tor .onion sites, paste sites
# ├── Malware → VirusTotal, threat intelligence
# └── Cloud → S3 buckets, Azure blobs

# Automated correlation:
# SpiderFoot connects findings:
# Domain → IP → ASN → Other domains → Related IPs
```

---

## 6. Framework Comparison

| Feature | Recon-ng | Amass | Reconftw | SpiderFoot |
|---------|:-------:|:-----:|:--------:|:----------:|
| Subdomain enum | ★★★ | ★★★★★ | ★★★★★ | ★★★★ |
| Port scanning | ★★ | ★★ | ★★★★★ | ★★★ |
| OSINT | ★★★★ | ★★★ | ★★★ | ★★★★★ |
| Vuln scanning | ★ | ★ | ★★★★★ | ★★★ |
| Ease of use | ★★★ | ★★★★ | ★★★★★ | ★★★★★ |
| Customization | ★★★★★ | ★★★ | ★★★ | ★★★★ |
| Reporting | ★★★★ | ★★★ | ★★★★ | ★★★★★ |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Recon-ng** | Modular framework with database |
| **Amass** | Best for subdomain enumeration |
| **Reconftw** | Full automated pipeline (30+ tools) |
| **SpiderFoot** | 200+ module OSINT automation |
| **Automation** | Consistent, thorough, repeatable |
| **Customization** | Tailor pipeline to engagement needs |

---

## Quick Revision Questions

1. **What is the advantage of using a recon framework over individual tools?**
2. **How does Recon-ng store and manage results?**
3. **What makes Amass the best tool for subdomain enumeration?**
4. **What does Reconftw's automated pipeline include?**
5. **How does SpiderFoot correlate findings across modules?**

---

[Next: Custom Scripting →](02-custom-scripting.md)

---

*Unit 9 - Topic 1 of 5 | [Back to README](../README.md)*
