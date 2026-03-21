# Custom Scripting

## Unit 9 - Topic 2 | Reconnaissance Automation

---

## Overview

**Custom scripting** creates purpose-built reconnaissance tools tailored to specific engagement requirements. When existing tools don't cover a particular use case or need to be combined in a specific workflow, custom scripts fill the gap.

---

## 1. When to Write Custom Scripts

```
USE CUSTOM SCRIPTS WHEN:

├── Existing tools don't solve the exact problem
├── Need to combine multiple tool outputs
├── Target has unusual infrastructure
├── Need to handle rate limiting / OPSEC
├── Automating a repetitive manual process
├── Parsing and correlating data from multiple sources
└── Building engagement-specific workflows

COMMON CUSTOM SCRIPT USES:
├── Subdomain brute-forcing with custom wordlists
├── Parameter discovery on specific endpoints
├── Custom web crawling with authentication
├── Automated screenshot comparison
├── Data parsing and correlation
├── Alert on new findings (continuous recon)
└── Custom API interaction
```

---

## 2. Python Recon Scripts

```python
#!/usr/bin/env python3
"""
Subdomain Checker — Verify subdomains and collect info
"""
import dns.resolver
import requests
import sys
import json

def check_subdomain(subdomain):
    """Check if subdomain resolves and is alive"""
    result = {"subdomain": subdomain, "ips": [], "status": None, "title": None}
    
    # DNS Resolution
    try:
        answers = dns.resolver.resolve(subdomain, 'A')
        result["ips"] = [str(r) for r in answers]
    except Exception:
        return None  # Doesn't resolve
    
    # HTTP Check
    for scheme in ["https", "http"]:
        try:
            r = requests.get(f"{scheme}://{subdomain}", 
                           timeout=5, verify=False,
                           allow_redirects=True)
            result["status"] = r.status_code
            # Extract title
            if "<title>" in r.text.lower():
                start = r.text.lower().index("<title>") + 7
                end = r.text.lower().index("</title>")
                result["title"] = r.text[start:end].strip()
            result["url"] = f"{scheme}://{subdomain}"
            break
        except Exception:
            continue
    
    return result

# Main execution
domain = sys.argv[1]
wordlist = open(sys.argv[2]).read().splitlines()

results = []
for word in wordlist:
    sub = f"{word}.{domain}"
    result = check_subdomain(sub)
    if result:
        results.append(result)
        print(f"[+] {sub} → {result['ips']} ({result['status']})")

# Save results
with open(f"{domain}_subdomains.json", "w") as f:
    json.dump(results, f, indent=2)
```

---

## 3. Bash Recon Automation

```bash
#!/bin/bash
# === AUTOMATED RECON PIPELINE ===
# Usage: ./recon.sh target.com

TARGET=$1
OUTPUT="results/${TARGET}"
mkdir -p "${OUTPUT}"

echo "[*] Starting recon for ${TARGET}"

# Step 1: Subdomain Enumeration
echo "[+] Subdomain enumeration..."
subfinder -d ${TARGET} -silent > ${OUTPUT}/subs_subfinder.txt
amass enum -passive -d ${TARGET} -o ${OUTPUT}/subs_amass.txt 2>/dev/null
cat ${OUTPUT}/subs_*.txt | sort -u > ${OUTPUT}/subdomains.txt
echo "    Found: $(wc -l < ${OUTPUT}/subdomains.txt) subdomains"

# Step 2: Resolve Subdomains
echo "[+] Resolving subdomains..."
cat ${OUTPUT}/subdomains.txt | dnsx -silent > ${OUTPUT}/resolved.txt
echo "    Resolved: $(wc -l < ${OUTPUT}/resolved.txt)"

# Step 3: HTTP Probing
echo "[+] HTTP probing..."
cat ${OUTPUT}/resolved.txt | httpx -silent -title -status-code \
  -tech-detect > ${OUTPUT}/alive.txt
echo "    Alive: $(wc -l < ${OUTPUT}/alive.txt)"

# Step 4: Port Scanning
echo "[+] Port scanning..."
naabu -iL ${OUTPUT}/resolved.txt -top-ports 100 \
  -silent > ${OUTPUT}/ports.txt

# Step 5: URL Discovery
echo "[+] URL discovery..."
cat ${OUTPUT}/resolved.txt | waybackurls > ${OUTPUT}/urls.txt

# Step 6: Vulnerability Scanning
echo "[+] Nuclei scanning..."
nuclei -l ${OUTPUT}/alive.txt -severity critical,high \
  -o ${OUTPUT}/vulns.txt 2>/dev/null

echo "[*] Recon complete! Results in ${OUTPUT}/"
```

---

## 4. Common Script Patterns

| Pattern | Use Case | Language |
|---------|----------|---------|
| **HTTP request loop** | Check alive hosts | Python/Bash |
| **DNS resolver** | Subdomain validation | Python |
| **Web scraper** | Extract data from pages | Python |
| **API client** | Query Shodan, Censys, etc. | Python |
| **File parser** | Combine tool outputs | Python/Bash |
| **Diff checker** | Track changes between scans | Bash |
| **Alert sender** | Notify on new findings | Python |

---

## 5. Script Development Best Practices

```
BEST PRACTICES:

ERROR HANDLING:
├── Handle DNS timeout gracefully
├── Catch HTTP connection errors
├── Retry on temporary failures
├── Log errors for debugging
└── Don't crash on single failure

RATE LIMITING:
├── Add delays between requests (time.sleep)
├── Respect target capacity
├── Use exponential backoff on errors
├── Rotate user agents
└── Consider using proxies

OUTPUT:
├── JSON for structured data (machine-readable)
├── CSV for spreadsheet analysis
├── Text for quick review
├── HTML for reports
└── Always include timestamps

OPSEC:
├── Rotate source IPs (proxychains)
├── Randomize request patterns
├── Use realistic User-Agent strings
├── Avoid predictable timing
└── Log everything for accountability
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Custom scripts** | Tailored tools for specific needs |
| **Python** | Best for complex recon logic |
| **Bash** | Best for chaining existing tools |
| **Pipeline** | Subfinder → dnsx → httpx → nuclei |
| **Rate limiting** | Avoid detection and target overload |
| **Output** | JSON for structured, text for quick review |

---

## Quick Revision Questions

1. **When should you write custom recon scripts vs. using existing tools?**
2. **What makes Python suitable for recon scripting?**
3. **What is a typical bash recon pipeline?**
4. **Why is rate limiting important in custom scripts?**
5. **What output formats should recon scripts support?**

---

[← Previous: Recon Frameworks](01-recon-frameworks.md) | [Next: API Integration →](03-api-integration.md)

---

*Unit 9 - Topic 2 of 5 | [Back to README](../README.md)*
