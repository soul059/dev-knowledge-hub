# Data Correlation

## Unit 9 - Topic 5 | Reconnaissance Automation

---

## Overview

**Data correlation** connects, cross-references, and analyzes reconnaissance data from multiple sources to extract patterns, relationships, and actionable intelligence. Raw data from individual tools becomes exponentially more valuable when correlated.

---

## 1. Why Correlate Data?

```
RAW DATA vs CORRELATED INTELLIGENCE:

RAW DATA (Individual Tools):
├── Subfinder: 150 subdomains found
├── Nmap: 45 open ports across 20 IPs
├── theHarvester: 30 email addresses
├── Shodan: 15 internet-facing devices
├── WhatWeb: 10 technology fingerprints
└── Nuclei: 8 vulnerabilities detected

CORRELATED INTELLIGENCE:
├── dev.target.com (203.0.113.40)
│   ├── Runs: Apache 2.4.49 (vulnerable!)
│   ├── Has: CVE-2021-41773 (path traversal)
│   ├── Admin: john.smith@target.com (from WHOIS)
│   ├── Same server hosts: staging.target.com
│   ├── No WAF detected (unlike production)
│   └── PRIORITY: ★★★★★ HIGH VALUE TARGET
│
└── This correlation transforms 150+ data points
    into ONE actionable finding with context
```

---

## 2. Correlation Techniques

```
DATA CORRELATION METHODS:

IP-BASED CORRELATION:
├── Same IP hosts multiple domains (shared hosting)
├── IP → ASN → other IPs in same range
├── IP history → previous domains hosted
└── IP → geolocation → physical location

DOMAIN-BASED CORRELATION:
├── Subdomains → reveal internal structure
├── DNS records → point to same infrastructure
├── WHOIS → same registrant = related domains
├── SSL certificates → SAN field lists all domains
└── Shared nameservers → same organization

PERSON-BASED CORRELATION:
├── Email → LinkedIn → role → access level
├── Email → HIBP → breached credentials
├── Name → social media → personal intel
├── GitHub username → code → secrets
└── Person → department → relevant targets

TECHNOLOGY-BASED CORRELATION:
├── Same CMS version → same vulnerabilities
├── Shared libraries → shared weaknesses
├── Default configs → same misconfigurations
└── Common framework → known attack vectors
```

---

## 3. Correlation Tools

```bash
# === MALTEGO (Visual Correlation) ===
# Transform-based intelligence platform
# Visualizes relationships between entities
# Entities: Person, Domain, IP, Email, Company
# Transforms: DNS lookup, WHOIS, email search, etc.

# Workflow:
# 1. Add target domain as entity
# 2. Run transforms (DNS, WHOIS, subdomains)
# 3. Expand discovered entities
# 4. Visual graph shows all relationships
# 5. Identify clusters and high-value nodes

# === SPIDERFOOT (Automated Correlation) ===
# Automatically correlates across 200+ modules
# Links: domain → IP → ASN → other domains → emails

# === CUSTOM CORRELATION (Python) ===
# Combine tool outputs into structured database
```

```python
#!/usr/bin/env python3
"""Data Correlation Engine"""
import json
from collections import defaultdict

class ReconCorrelator:
    def __init__(self):
        self.ip_to_domains = defaultdict(set)
        self.domain_to_ips = defaultdict(set)
        self.ip_to_ports = defaultdict(set)
        self.ip_to_vulns = defaultdict(list)
        self.emails = defaultdict(dict)
    
    def load_subdomains(self, filepath):
        """Load subdomain:IP mappings"""
        with open(filepath) as f:
            for line in f:
                parts = line.strip().split(",")
                if len(parts) == 2:
                    domain, ip = parts
                    self.ip_to_domains[ip].add(domain)
                    self.domain_to_ips[domain].add(ip)
    
    def load_nmap(self, filepath):
        """Load port scan results"""
        # Parse nmap grepable output
        with open(filepath) as f:
            for line in f:
                if "Ports:" in line:
                    ip = line.split()[1]
                    ports = line.split("Ports: ")[1]
                    for port_info in ports.split(","):
                        port = port_info.strip().split("/")[0]
                        self.ip_to_ports[ip].add(port)
    
    def correlate(self):
        """Find high-value targets"""
        targets = []
        for ip, domains in self.ip_to_domains.items():
            ports = self.ip_to_ports.get(ip, set())
            vulns = self.ip_to_vulns.get(ip, [])
            
            priority = len(ports) + len(vulns) * 3
            if any("dev" in d or "staging" in d for d in domains):
                priority += 5  # Dev/staging = higher value
            
            targets.append({
                "ip": ip,
                "domains": list(domains),
                "ports": list(ports),
                "vulns": vulns,
                "priority": priority
            })
        
        return sorted(targets, key=lambda x: x["priority"], reverse=True)
```

---

## 4. Correlation Report Format

```
CORRELATED RECONNAISSANCE REPORT
═══════════════════════════════════

TARGET: Target Corporation

HIGH-PRIORITY TARGETS:
━━━━━━━━━━━━━━━━━━━━━

1. dev.target.com (203.0.113.40) ★★★★★
   ├── Also hosts: staging.target.com, test.target.com
   ├── Ports: 22, 80, 443, 3306, 8080
   ├── Vulns: CVE-2021-41773 (Apache), CVE-2021-44228 (Log4j)
   ├── No WAF (Cloudflare not configured for this subdomain)
   ├── Admin: john.smith@target.com (found in WHOIS)
   │   └── john.smith@target.com appeared in 3 breaches
   └── RECOMMENDATION: Primary exploitation target

2. mail.target.com (203.0.113.20) ★★★★
   ├── Exchange Server 2019 (CU11)
   ├── Ports: 25, 443, 587, 993
   ├── ProxyShell potential (needs verification)
   ├── SPF: ~all (softfail — spoofing possible)
   └── RECOMMENDATION: Test ProxyShell, email spoofing

ATTACK PATH IDENTIFIED:
━━━━━━━━━━━━━━━━━━━━━━
dev.target.com (Apache RCE)
→ Access development server
→ Potential DB credentials in config
→ Pivot to internal network (3306 MySQL)
→ Potential domain credentials
→ Active Directory compromise
```

---

## 5. Data Storage and Management

| Storage Method | Use Case | Tools |
|---------------|----------|-------|
| **JSON files** | Individual tool output | jq, Python |
| **SQLite/PostgreSQL** | Structured correlation | SQL queries |
| **Elasticsearch** | Large-scale search | Kibana dashboards |
| **Neo4j** | Relationship graphing | Graph queries |
| **Recon-ng DB** | Built-in recon database | Recon-ng CLI |
| **Notion/Wiki** | Report documentation | Web interface |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Correlation** | Connect data points for intelligence |
| **IP-based** | Same IP = related services |
| **Domain-based** | WHOIS, DNS, SSL reveal relationships |
| **Maltego** | Visual relationship mapping |
| **Priority scoring** | Rank targets by ports, vulns, exposure |
| **Attack paths** | Chain findings into exploitation routes |

---

## Quick Revision Questions

1. **Why is data correlation more valuable than raw data?**
2. **What are the four main correlation methods?**
3. **How does Maltego help with data correlation?**
4. **What makes a target "high priority" in correlation?**
5. **How do you identify attack paths from correlated data?**

---

[← Previous: Continuous Reconnaissance](04-continuous-reconnaissance.md)

---

*Unit 9 - Topic 5 of 5 | [Back to README](../README.md)*

---

*🎉 Congratulations! You've completed Subject 2.2: Information Gathering and Reconnaissance!*
