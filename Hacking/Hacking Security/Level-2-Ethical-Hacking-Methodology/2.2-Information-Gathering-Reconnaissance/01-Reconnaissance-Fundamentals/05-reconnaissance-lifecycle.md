# Reconnaissance Lifecycle

## Unit 1 - Topic 5 | Reconnaissance Fundamentals

---

## Overview

The **reconnaissance lifecycle** is a structured, iterative process for gathering information about a target. Rather than random data collection, professionals follow a systematic approach that builds on previous findings and continuously refines the target picture.

---

## 1. Lifecycle Phases

```
┌──────────────────────────────────────────────────────────────┐
│              RECONNAISSANCE LIFECYCLE                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  PHASE 1: DEFINE OBJECTIVES                                  │
│  └── What do we need to know? What's in scope?              │
│                    │                                         │
│                    ▼                                         │
│  PHASE 2: PASSIVE COLLECTION                                 │
│  └── OSINT, public sources, no target interaction           │
│                    │                                         │
│                    ▼                                         │
│  PHASE 3: SEMI-PASSIVE COLLECTION                            │
│  └── Website browsing, DNS queries, certificate checks      │
│                    │                                         │
│                    ▼                                         │
│  PHASE 4: ACTIVE COLLECTION                                  │
│  └── Scanning, enumeration, probing                         │
│                    │                                         │
│                    ▼                                         │
│  PHASE 5: ANALYSIS & CORRELATION                             │
│  └── Connect data points, identify patterns                 │
│                    │                                         │
│                    ▼                                         │
│  PHASE 6: DOCUMENTATION                                      │
│  └── Organize findings, create target profile               │
│           │                                                  │
│           └── ITERATE (new findings trigger new collection) │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Phase Details

| Phase | Activities | Output |
|-------|-----------|--------|
| **Define objectives** | Review scope, identify requirements | Collection plan |
| **Passive collection** | OSINT, WHOIS, Google dorking, Shodan | Raw data sets |
| **Semi-passive** | Browse website, DNS queries, cert checks | Application data |
| **Active collection** | Port scans, enumeration, vulnerability scans | Technical data |
| **Analysis** | Correlate findings, map relationships | Target profile |
| **Documentation** | Organize and report findings | Recon report |

---

## 3. Iterative Nature

```
RECON IS ITERATIVE — Each discovery leads to new questions:

ITERATION 1:
  WHOIS lookup → discover nameserver ns1.hosting.com
  → Who else uses this hosting provider?

ITERATION 2:
  DNS enum → find subdomain dev.target.com
  → What's running on dev.target.com?

ITERATION 3:
  dev.target.com has exposed .git directory
  → Download source code → find hardcoded API keys

ITERATION 4:
  API keys → access internal API
  → Enumerate internal endpoints

EACH FINDING OPENS NEW AVENUES OF INVESTIGATION
```

---

## 4. Practical Workflow Example

```bash
# PHASE 1: Define Objectives
# Target: target.com
# Goal: Map external attack surface

# PHASE 2: Passive Collection
whois target.com                       # Domain info
subfinder -d target.com -o subs.txt    # Subdomains
theHarvester -d target.com -b all      # Emails, names
shodan search "hostname:target.com"    # Cached services

# PHASE 3: Semi-Passive
curl -s https://target.com             # Browse website
dig target.com ANY                     # DNS records
curl https://crt.sh/?q=target.com      # Certificate transparency

# PHASE 4: Active Collection
nmap -sS -sV -p- target.com            # Port scan
gobuster dir -u https://target.com -w common.txt  # Dir enum
nikto -h https://target.com            # Web vuln scan

# PHASE 5: Analysis
# Correlate all findings
# Build target profile
# Identify attack vectors

# PHASE 6: Documentation
# Write recon report with all findings organized by category
```

---

## 5. Lifecycle Timing

| Phase | Duration | Notes |
|-------|:--------:|-------|
| **Objectives** | 1-2 hours | At engagement start |
| **Passive** | 1-3 days | Bulk of time spent here |
| **Semi-passive** | 4-8 hours | Low-risk data gathering |
| **Active** | 4-16 hours | With authorization only |
| **Analysis** | 4-8 hours | Connect the dots |
| **Documentation** | 2-4 hours | Ongoing throughout |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Lifecycle** | Structured, iterative recon process |
| **Phases** | Objectives → Passive → Active → Analysis → Docs |
| **Iterative** | Each finding opens new investigation paths |
| **Passive first** | Always start with no-contact methods |
| **Documentation** | Record everything continuously |
| **Time** | Recon takes days, not hours |

---

## Quick Revision Questions

1. **What are the phases of the reconnaissance lifecycle?**
2. **Why is recon iterative?**
3. **Why start with passive before active?**
4. **How long should reconnaissance take?**
5. **What is the output of the analysis phase?**

---

[← Previous: Information Categories](04-information-categories.md)

---

*Unit 1 - Topic 5 of 5 | [Back to README](../README.md) | [Next Unit: OSINT →](../02-OSINT/01-osint-methodology.md)*
