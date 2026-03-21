# API Integration

## Unit 9 - Topic 3 | Reconnaissance Automation

---

## Overview

**API integration** connects reconnaissance tools to external intelligence services — Shodan, Censys, VirusTotal, SecurityTrails, and others — enabling automated large-scale data collection that would be impossible through manual browsing.

---

## 1. Key Reconnaissance APIs

| API | Data Provided | Free Tier |
|-----|--------------|:---------:|
| **Shodan** | Devices, ports, banners, vulns | 100 queries/month |
| **Censys** | Hosts, certificates, services | 250 queries/month |
| **VirusTotal** | Malware, URLs, domains | 500 queries/day |
| **SecurityTrails** | DNS, subdomains, WHOIS | 50 queries/month |
| **Hunter.io** | Emails, email format | 25 searches/month |
| **HIBP** | Breach data by email | Rate limited |
| **Whoisxml** | WHOIS, DNS, subdomains | 500 queries/month |
| **AlienVault OTX** | Threat intel, indicators | Unlimited (free) |
| **GreyNoise** | Internet noise vs targeted | 100 queries/day |

---

## 2. Shodan API Integration

```python
#!/usr/bin/env python3
"""Shodan API — Search for target infrastructure"""
import shodan

SHODAN_API_KEY = "YOUR_API_KEY"
api = shodan.Shodan(SHODAN_API_KEY)

# Search by organization
results = api.search('org:"Target Corporation"')
print(f"Total results: {results['total']}")

for result in results['matches']:
    print(f"""
    IP: {result['ip_str']}
    Port: {result['port']}
    Org: {result.get('org', 'N/A')}
    OS: {result.get('os', 'N/A')}
    Banner: {result['data'][:100]}
    """)

# Search by domain
dns_info = api.dns.domain_info('target.com')
for subdomain in dns_info.get('subdomains', []):
    print(f"  {subdomain}.target.com")

# Host lookup
host = api.host('203.0.113.10')
print(f"OS: {host.get('os', 'Unknown')}")
print(f"Ports: {[s['port'] for s in host['data']]}")
for vuln in host.get('vulns', []):
    print(f"  Vuln: {vuln}")
```

```bash
# === SHODAN CLI ===
shodan init YOUR_API_KEY
shodan search "org:Target Corporation"
shodan host 203.0.113.10
shodan domain target.com
shodan stats --facets port "org:Target Corporation"
```

---

## 3. Censys API Integration

```python
#!/usr/bin/env python3
"""Censys API — Certificate and host search"""
from censys.search import CensysHosts, CensysCerts

# Host search
h = CensysHosts()
query = h.search("services.http.response.headers.server: Apache "
                 "AND autonomous_system.name: 'Target Corporation'")
for host in query():
    print(f"IP: {host['ip']} - Services: {host.get('services', [])}")

# Certificate search (find subdomains)
c = CensysCerts()
query = c.search("parsed.names: target.com")
for cert in query():
    for name in cert.get('parsed', {}).get('names', []):
        print(f"  Domain: {name}")
```

---

## 4. Multi-API Aggregation Script

```python
#!/usr/bin/env python3
"""
Multi-API Recon Aggregator
Queries multiple APIs and correlates results
"""
import json
import requests
import time

class ReconAggregator:
    def __init__(self, config):
        self.config = config
        self.results = {
            "subdomains": set(),
            "emails": set(),
            "ips": set(),
            "technologies": [],
            "vulnerabilities": []
        }
    
    def query_securitytrails(self, domain):
        """Get subdomains from SecurityTrails"""
        url = f"https://api.securitytrails.com/v1/domain/{domain}/subdomains"
        headers = {"APIKEY": self.config["securitytrails_key"]}
        r = requests.get(url, headers=headers)
        if r.status_code == 200:
            for sub in r.json().get("subdomains", []):
                self.results["subdomains"].add(f"{sub}.{domain}")
        time.sleep(1)  # Rate limiting
    
    def query_hunter(self, domain):
        """Get emails from Hunter.io"""
        url = f"https://api.hunter.io/v2/domain-search"
        params = {"domain": domain, "api_key": self.config["hunter_key"]}
        r = requests.get(url, params=params)
        if r.status_code == 200:
            for email in r.json()["data"]["emails"]:
                self.results["emails"].add(email["value"])
        time.sleep(1)
    
    def run_all(self, domain):
        """Run all API queries"""
        print(f"[*] Running multi-API recon for {domain}")
        self.query_securitytrails(domain)
        self.query_hunter(domain)
        # Add more API queries here
        return self.results

# Usage
config = {
    "securitytrails_key": "YOUR_KEY",
    "hunter_key": "YOUR_KEY"
}
recon = ReconAggregator(config)
results = recon.run_all("target.com")
```

---

## 5. API Key Management

```
API KEY SECURITY:

DO:
├── Store keys in environment variables
│   export SHODAN_API_KEY="xxxxx"
├── Use .env files (NOT in git!)
├── Rotate keys regularly
├── Use separate keys per project
└── Monitor API usage for abuse

DON'T:
├── Hardcode keys in scripts
├── Commit keys to git repos
├── Share keys with others
├── Use personal keys for team projects
└── Exceed rate limits (get banned)

ENVIRONMENT VARIABLES:
├── SHODAN_API_KEY
├── CENSYS_API_ID / CENSYS_API_SECRET
├── VT_API_KEY (VirusTotal)
├── HUNTER_API_KEY
└── SECURITYTRAILS_API_KEY

# Python: os.environ.get("SHODAN_API_KEY")
# Bash: ${SHODAN_API_KEY}
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Shodan** | Device search — ports, banners, vulns |
| **Censys** | Certificate search for subdomains |
| **Hunter.io** | Email harvesting by domain |
| **Aggregation** | Combine multiple APIs for completeness |
| **Rate limiting** | Respect API limits, add delays |
| **Key security** | Environment variables, never in code |

---

## Quick Revision Questions

1. **What data does the Shodan API provide?**
2. **How can Censys certificate search find subdomains?**
3. **Why aggregate data from multiple APIs?**
4. **How should API keys be stored securely?**
5. **What rate limiting strategies should scripts implement?**

---

[← Previous: Custom Scripting](02-custom-scripting.md) | [Next: Continuous Reconnaissance →](04-continuous-reconnaissance.md)

---

*Unit 9 - Topic 3 of 5 | [Back to README](../README.md)*
