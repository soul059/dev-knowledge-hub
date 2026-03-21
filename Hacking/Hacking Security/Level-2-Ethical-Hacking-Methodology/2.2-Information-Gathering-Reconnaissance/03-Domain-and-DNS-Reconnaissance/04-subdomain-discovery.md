# Subdomain Discovery

## Unit 3 - Topic 4 | Domain and DNS Reconnaissance

---

## Overview

**Subdomain discovery** is one of the most critical recon activities. Subdomains often host development servers, admin panels, APIs, and legacy applications with weaker security than the main website. Finding these hidden subdomains dramatically expands the attack surface.

---

## 1. Discovery Methods

| Method | Type | Technique |
|--------|:---:|-----------|
| **Certificate Transparency** | Passive | Query CT logs for issued certificates |
| **DNS brute force** | Active | Try wordlist of common subdomain names |
| **Search engines** | Passive | Google: site:*.target.com |
| **Passive DNS** | Passive | Query historical DNS databases |
| **Web archives** | Passive | Find archived subdomains |
| **ASN/IP scanning** | Active | Reverse DNS on IP ranges |
| **VHost enumeration** | Active | Probe different Host headers |

---

## 2. Subdomain Discovery Tools

```bash
# === PASSIVE TOOLS ===

# Subfinder (fastest passive)
subfinder -d target.com -o subs.txt -silent

# Amass (most comprehensive)
amass enum -passive -d target.com -o amass.txt

# Assetfinder
assetfinder --subs-only target.com

# Certificate Transparency
curl -s "https://crt.sh/?q=%25.target.com&output=json" | \
  jq -r '.[].name_value' | sort -u

# theHarvester
theHarvester -d target.com -b all

# === ACTIVE TOOLS ===

# Gobuster DNS
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -t 50

# Wfuzz
wfuzz -c -w wordlist.txt -u "http://target.com" -H "Host: FUZZ.target.com" --hc 404

# Fierce
fierce --domain target.com

# DNSRecon brute force
dnsrecon -d target.com -t brt -D wordlist.txt

# Puredns (DNS resolution + brute force)
puredns bruteforce wordlist.txt target.com -r resolvers.txt
```

---

## 3. Combining Multiple Sources

```bash
# COMPREHENSIVE SUBDOMAIN ENUMERATION:

# Step 1: Collect from multiple passive sources
subfinder -d target.com -silent >> all_subs.txt
amass enum -passive -d target.com >> all_subs.txt
assetfinder --subs-only target.com >> all_subs.txt

# Step 2: Sort and deduplicate
sort -u all_subs.txt -o all_subs.txt
echo "Found $(wc -l < all_subs.txt) unique subdomains"

# Step 3: Resolve and validate
cat all_subs.txt | httpx -silent -status-code -title -tech-detect -o live_subs.txt

# Step 4: Screenshot live subdomains
gowitness file -f live_subs.txt
# or
aquatone < live_subs.txt

# Step 5: Categorize findings
# ├── Web applications (HTTP 200)
# ├── Login pages (redirects to /login)
# ├── API endpoints (/api/, /v1/)
# ├── Development/staging
# └── Admin panels
```

---

## 4. High-Value Subdomain Patterns

| Pattern | Why It's Valuable |
|---------|------------------|
| `dev.`, `staging.`, `test.` | Often less secured, debug enabled |
| `admin.`, `panel.`, `manage.` | Administrative interfaces |
| `api.`, `api-v2.`, `graphql.` | API endpoints |
| `vpn.`, `remote.`, `gateway.` | Remote access points |
| `jenkins.`, `gitlab.`, `jira.` | DevOps tools |
| `mail.`, `webmail.`, `owa.` | Email access |
| `db.`, `mysql.`, `mongo.` | Database access |
| `backup.`, `old.`, `legacy.` | Forgotten/unmaintained systems |
| `internal.`, `intranet.` | Internal applications |
| `cdn.`, `static.`, `assets.` | Content delivery |

---

## 5. Subdomain Takeover

```
SUBDOMAIN TAKEOVER:

When a subdomain's DNS points to a service that's been
decommissioned, an attacker can claim that service and
serve content under the target's subdomain.

EXAMPLE:
1. blog.target.com → CNAME → target.herokuapp.com
2. Heroku app was deleted
3. Attacker creates target.herokuapp.com on their account
4. blog.target.com now serves attacker's content!

DETECTION:
├── CNAME pointing to external service
├── Service returns 404 or "not found" page
├── Service-specific error messages
└── Tools: subjack, nuclei takeover templates

VULNERABLE SERVICES:
├── GitHub Pages, Heroku, AWS S3
├── Azure, Shopify, Fastly
├── Tumblr, WordPress.com
└── Many SaaS providers
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Why subdomains** | Expand attack surface, find weak points |
| **Passive** | crt.sh, subfinder, amass, assetfinder |
| **Active** | gobuster dns, fierce, brute force |
| **Combine sources** | Multiple tools for best coverage |
| **Validate** | httpx to find live subdomains |
| **Takeover** | Dangling CNAMEs = subdomain takeover risk |

---

## Quick Revision Questions

1. **Why is subdomain discovery important?**
2. **Name 3 passive subdomain discovery methods.**
3. **What is subdomain takeover?**
4. **Which subdomain patterns indicate high-value targets?**
5. **How do you validate which subdomains are live?**

---

[← Previous: Zone Transfers](03-zone-transfers.md) | [Next: DNS Record Analysis →](05-dns-record-analysis.md)

---

*Unit 3 - Topic 4 of 6 | [Back to README](../README.md)*
