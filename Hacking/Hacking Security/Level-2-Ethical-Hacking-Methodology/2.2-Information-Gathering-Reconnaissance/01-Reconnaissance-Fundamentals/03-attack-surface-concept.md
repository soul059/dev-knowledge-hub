# Attack Surface Concept

## Unit 1 - Topic 3 | Reconnaissance Fundamentals

---

## Overview

The **attack surface** is the sum of all points where an attacker can try to enter or extract data from a system. Understanding and mapping the attack surface is a primary goal of reconnaissance — the larger the attack surface, the more potential entry points exist.

---

## 1. What Is an Attack Surface?

```
┌──────────────────────────────────────────────────────────────┐
│              ATTACK SURFACE COMPONENTS                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  DIGITAL ATTACK SURFACE:                                     │
│  ├── Web applications and APIs                               │
│  ├── Network services and open ports                         │
│  ├── Email systems                                           │
│  ├── DNS infrastructure                                      │
│  ├── Cloud services and storage                              │
│  ├── Mobile applications                                     │
│  ├── VPN and remote access                                   │
│  └── IoT devices                                             │
│                                                              │
│  HUMAN ATTACK SURFACE:                                       │
│  ├── Employees (phishing targets)                            │
│  ├── Contractors and vendors                                 │
│  ├── Help desk / support staff                               │
│  └── Social media presence                                   │
│                                                              │
│  PHYSICAL ATTACK SURFACE:                                    │
│  ├── Office locations                                        │
│  ├── Server rooms / data centers                             │
│  ├── Badge/access systems                                    │
│  ├── Dumpsters (dumpster diving)                             │
│  └── USB drops / rogue devices                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Attack Surface Mapping

```bash
# === EXTERNAL ATTACK SURFACE ===
# What's visible from the internet

# Subdomains
subfinder -d target.com -o subdomains.txt
amass enum -d target.com

# Open ports across all discovered IPs
nmap -sS --top-ports 1000 -iL ip_list.txt -oA external_scan

# Web applications
httpx -l subdomains.txt -title -status-code -tech-detect

# Cloud assets
# Check: AWS S3, Azure Blob, GCP buckets
aws s3 ls s3://target-backups --no-sign-request

# === INTERNAL ATTACK SURFACE ===
# What's visible once inside the network
# Internal services, file shares, databases
# Active Directory, internal web apps
# Printers, IoT devices, SCADA systems
```

---

## 3. Attack Surface Categories

| Category | Examples | Recon Method |
|----------|---------|:---:|
| **Web apps** | Login pages, APIs, forms | Spider, directory enum |
| **Network** | Open ports, services | Port scanning |
| **Email** | MX records, users | Email harvesting |
| **Cloud** | S3 buckets, Azure blobs | Cloud enum tools |
| **DNS** | Subdomains, zone data | DNS enumeration |
| **People** | Employees, roles | LinkedIn, social media |
| **Code** | GitHub repos, leaks | GitHub dorking |
| **Physical** | Offices, badges | Physical recon |

---

## 4. Attack Surface Reduction

```
DEFENSE: MINIMIZING THE ATTACK SURFACE

BEFORE REDUCTION:
┌─────────────────────────────────────────┐
│ ● Web app (3 public-facing)            │
│ ● 50 open ports across 20 servers      │
│ ● 15 subdomains (6 abandoned)          │
│ ● 5 cloud storage buckets (2 public)   │
│ ● 200 employee emails exposed          │
│ ● 3 legacy services still running      │
│ ● GitHub repo with hardcoded secrets   │
│ ATTACK SURFACE: ████████████████ LARGE │
└─────────────────────────────────────────┘

AFTER REDUCTION:
┌─────────────────────────────────────────┐
│ ● Web app (2, behind WAF)              │
│ ● 12 ports across 8 servers            │
│ ● 9 subdomains (abandoned removed)     │
│ ● 5 buckets (all private)              │
│ ● Emails protected from harvesting     │
│ ● Legacy services decommissioned       │
│ ● Secrets rotated, repo cleaned        │
│ ATTACK SURFACE: ████████ REDUCED       │
└─────────────────────────────────────────┘
```

---

## 5. Attack Surface Scoring

| Factor | Low Risk | Medium Risk | High Risk |
|--------|:--------:|:-----------:|:---------:|
| **Open ports** | < 10 | 10-50 | 50+ |
| **Subdomains** | < 10 | 10-50 | 50+ |
| **Public apps** | 1-2 | 3-10 | 10+ |
| **Cloud assets** | All private | Some public | Many public |
| **Exposed emails** | Few | Moderate | Many |
| **Code repos** | Private | Mixed | Public with secrets |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Attack surface** | All possible entry points for an attacker |
| **Digital** | Web apps, ports, cloud, APIs |
| **Human** | Employees, phishing, social engineering |
| **Physical** | Offices, badges, dumpsters |
| **Mapping** | Enumerate all components systematically |
| **Reduction** | Remove unnecessary exposure |

---

## Quick Revision Questions

1. **What is an attack surface?**
2. **Name the three categories of attack surface.**
3. **How do you map the external attack surface?**
4. **What is attack surface reduction?**
5. **Why do abandoned subdomains increase risk?**

---

[← Previous: Passive vs Active Reconnaissance](02-passive-vs-active-reconnaissance.md) | [Next: Information Categories →](04-information-categories.md)

---

*Unit 1 - Topic 3 of 5 | [Back to README](../README.md)*
