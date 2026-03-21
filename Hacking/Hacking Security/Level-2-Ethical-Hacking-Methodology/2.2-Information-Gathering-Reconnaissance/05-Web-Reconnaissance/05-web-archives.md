# Web Archives

## Unit 5 - Topic 5 | Web Reconnaissance

---

## Overview

**Web archives** (particularly the Wayback Machine) preserve historical snapshots of websites over time. For reconnaissance, archives reveal old content, removed pages, previous configurations, and sensitive data that was once publicly accessible.

---

## 1. The Wayback Machine

```
WAYBACK MACHINE (web.archive.org):

HOW IT WORKS:
├── Automated crawlers archive websites periodically
├── Snapshots stored with timestamps
├── Billions of pages archived since 1996
├── Publicly searchable by URL
└── Cannot be removed by site owner easily

WHAT IT REVEALS:
├── Old versions of the website
├── Removed pages and content
├── Previous technology stack
├── Former employee information
├── Old contact details / email addresses
├── Development/staging references
├── Exposed credentials (if once public)
├── Deprecated API endpoints
└── Old JavaScript with hardcoded secrets
```

---

## 2. Using Web Archives for Recon

```bash
# === WAYBACK MACHINE (Web Interface) ===
# web.archive.org/web/*/target.com
# Browse snapshots from any date

# === WAYBACK MACHINE CDX API ===
# List ALL archived URLs for a domain:
curl "https://web.archive.org/cdx/search/cdx?url=target.com/*&output=text&fl=original&collapse=urlkey" \
  | sort -u > archived_urls.txt

# Filter for specific file types:
curl "https://web.archive.org/cdx/search/cdx?url=target.com/*&output=text&fl=original&filter=mimetype:application/pdf" \
  | sort -u > archived_pdfs.txt

# === WAYBACKURLS (Go Tool) ===
echo "target.com" | waybackurls > all_urls.txt
# Extracts all URLs from Wayback Machine
# May find thousands of historical URLs!

# === GAU (Get All URLs) ===
gau target.com > all_urls.txt
# Combines: Wayback Machine + Common Crawl + URLScan + AlienVault

# === WAYMORE ===
waymore -i target.com -mode U
# Even more comprehensive URL extraction
```

---

## 3. Intelligence from Archives

```bash
# === FIND SENSITIVE FILES IN ARCHIVED URLS ===
cat all_urls.txt | grep -iE "\.(sql|bak|old|zip|tar|conf|env|log|xml|json)$"
# May reveal: backup.sql, config.old, database.bak

# === FIND API ENDPOINTS ===
cat all_urls.txt | grep -i "api\|endpoint\|v1\|v2"
# Discover old/deprecated API versions

# === FIND ADMIN/LOGIN PAGES ===
cat all_urls.txt | grep -iE "admin|login|dashboard|panel|manage"

# === FIND PARAMETERS ===
cat all_urls.txt | grep "?" | sort -u
# URL parameters = potential injection points

# === COMPARE OLD VS NEW SITE ===
# Download old version:
wget "https://web.archive.org/web/20200101/https://target.com/" -O old_index.html
# Compare with current:
diff old_index.html <(curl -s https://target.com)
# Changes may reveal new or removed functionality
```

---

## 4. Other Web Archive Sources

| Source | URL | Speciality |
|--------|-----|-----------|
| **Wayback Machine** | web.archive.org | Most comprehensive |
| **Common Crawl** | commoncrawl.org | Petabytes of web data |
| **Google Cache** | `cache:target.com` | Recent cached version |
| **Bing Cache** | Click "Cached" in results | Microsoft's cache |
| **URLScan.io** | urlscan.io | Scanned page details |
| **AlienVault OTX** | otx.alienvault.com | Threat intelligence URLs |
| **VirusTotal** | virustotal.com | URL submission history |

---

## 5. Practical Scenarios

```
SCENARIO 1: FINDING REMOVED CREDENTIALS
─────────────────────────────────────────
Target accidentally pushed .env file to production
├── 2023-01-15: .env file was publicly accessible
├── 2023-01-16: Removed from site
├── BUT: Wayback Machine archived it!
└── web.archive.org/web/20230115/https://target.com/.env
    → Contains: DB_PASSWORD, API_KEYS, SECRET_KEY

SCENARIO 2: DISCOVERING OLD FUNCTIONALITY
─────────────────────────────────────────
Old admin panel was at /admin-v1/
├── Currently redirects to /admin/ (new version)
├── But old version may still be accessible
├── Or provides intel about authentication patterns
└── Technology changes show migration path

SCENARIO 3: EMPLOYEE ENUMERATION
─────────────────────────────────────────
Old "About Us" page listed team members
├── Current page only shows executives
├── Archived version shows all developers
├── Names → email pattern → credential stuffing
└── Names → LinkedIn → social engineering
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Wayback Machine** | Historical website snapshots |
| **CDX API** | List all archived URLs programmatically |
| **waybackurls** | Extract all historical URLs for domain |
| **gau** | Aggregate URLs from multiple sources |
| **Sensitive files** | Old backups, configs may be archived |
| **Intelligence** | Removed content, old versions, employees |

---

## Quick Revision Questions

1. **How does the Wayback Machine help reconnaissance?**
2. **What is the CDX API used for?**
3. **How do waybackurls and gau differ?**
4. **What sensitive data might be found in web archives?**
5. **Why compare archived versions with the current site?**

---

[← Previous: Robots.txt and Sitemap](04-robots-txt-and-sitemap.md) | [Next: Source Code Analysis →](06-source-code-analysis.md)

---

*Unit 5 - Topic 5 of 6 | [Back to README](../README.md)*
