# Comment and Metadata Analysis

## Unit 3: Web Reconnaissance — Topic 6

## 🎯 Overview

HTML comments, metadata, and embedded information in web pages often contain sensitive data left by developers — internal paths, API keys, TODO notes, debug information, and architectural details. Analyzing these artifacts is a low-effort, high-reward reconnaissance technique that can reveal critical information about the application.

---

## 1. HTML Comment Analysis

```
┌──────────────────────────────────────────────────────────────┐
│              COMMON HTML COMMENT FINDINGS                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  <!-- TODO: Fix SQL injection in search (JIRA-1234) -->     │
│  <!-- DEBUG: API endpoint = /internal/api/v3 -->             │
│  <!-- Admin panel: /super-secret-admin-panel -->             │
│  <!-- Database: mysql://root:p@ssword@db.internal:3306 -->  │
│  <!-- Commented out: Old auth bypass still works -->         │
│  <!-- Build: v2.3.1-beta, compiled 2024-01-15 -->           │
│  <!-- Developer: john.doe@company.com -->                    │
│  <!-- if (user.role == 'admin') show debug panel -->        │
│                                                              │
│  These comments are visible to ANYONE viewing page source!   │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Extract HTML comments from a page
curl -s https://target.com | grep -oP '<!--[\s\S]*?-->'

# Recursive comment extraction across multiple pages
for url in $(cat urls.txt); do
  echo "=== $url ==="
  curl -s "$url" | grep -oP '<!--[\s\S]*?-->' 2>/dev/null
done

# Extract from all spidered pages (Burp Suite)
# Target → Site map → right-click → Engagement tools → Find comments
```

---

## 2. Meta Tag Information

```html
<!-- Technology disclosure -->
<meta name="generator" content="WordPress 6.4.2" />
<meta name="generator" content="Drupal 10.1.5" />

<!-- Author information -->
<meta name="author" content="John Developer" />

<!-- Description may reveal functionality -->
<meta name="description" content="Internal employee portal - staging" />

<!-- Robots directives (what's being hidden?) -->
<meta name="robots" content="noindex, nofollow" />

<!-- Application-specific metadata -->
<meta name="csrf-token" content="abc123def456" />
<meta name="api-base" content="https://api.internal.target.com/v2" />
<meta name="environment" content="staging" />
```

```bash
# Extract all meta tags
curl -s https://target.com | grep -oE '<meta[^>]*>'

# Check for generator tags specifically
curl -s https://target.com | grep -i "generator"
```

---

## 3. JavaScript Comments and Debug Info

```bash
# Extract JavaScript comments
curl -s https://target.com/app.js | grep -oE '//.*$|/\*[\s\S]*?\*/'

# Look for sensitive patterns in JS
curl -s https://target.com/app.js | grep -iE \
  "(password|secret|key|token|api_key|admin|internal|debug|todo|fixme|hack|bypass)"

# Check for console.log statements (debug info)
curl -s https://target.com/app.js | grep -i "console\."

# Source map references
curl -s https://target.com/app.js | grep "sourceMappingURL"
```

### Common JS Comment Findings

```javascript
// TODO: Remove hardcoded API key before production
const API_KEY = "sk_live_abc123def456";

// HACK: Bypass authentication for testing
// if (process.env.NODE_ENV === 'development') skipAuth();

/* 
 * Internal API endpoints:
 * /api/internal/admin - admin operations
 * /api/internal/debug - debug information  
 * /api/internal/metrics - system metrics
 */

// FIXME: SQL query is vulnerable to injection
// db.query("SELECT * FROM users WHERE id = " + userId);
```

---

## 4. File Metadata (EXIF/Document)

```bash
# Extract metadata from images
exiftool image.jpg
# May reveal: GPS coordinates, camera info, software used,
# author name, creation date, editing software

# Extract metadata from PDFs
exiftool document.pdf
pdfinfo document.pdf
# May reveal: Author, creator application, company name,
# internal file paths, username

# Extract metadata from Office documents
exiftool spreadsheet.xlsx
# May reveal: Author, last modified by, company,
# internal paths, revision history

# Bulk metadata extraction from website
wget -r -A "*.pdf,*.doc,*.docx,*.xls,*.xlsx,*.jpg,*.png" \
  https://target.com/ -P downloads/
find downloads/ -type f -exec exiftool {} \; | \
  grep -iE "(author|creator|producer|company|email|gps)"
```

### FOCA — Metadata Extraction Tool

```
┌──────────────────────────────────────────────────────────────┐
│                     FOCA WORKFLOW                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Search Google for files: site:target.com filetype:pdf    │
│  2. Download all discovered documents                        │
│  3. Extract metadata from each file                          │
│  4. Analyze findings:                                        │
│     • Usernames (for credential attacks)                     │
│     • Software versions (for CVE lookup)                     │
│     • Internal paths (for directory traversal)               │
│     • Email addresses (for phishing/OSINT)                   │
│     • Network paths (for internal recon)                     │
│     • Printer names (for network mapping)                    │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Response Header Metadata

```bash
# Headers often leak information
curl -sI https://target.com

# Look for:
Server: Apache/2.4.52 (Ubuntu)          # Server version
X-Powered-By: PHP/8.1.2                 # Backend language
X-AspNet-Version: 4.0.30319             # .NET version
X-Debug-Token: abc123                    # Debug mode enabled
X-Request-Id: uuid-here                 # Request tracking
X-Runtime: 0.045632                      # Processing time
X-Generator: Drupal 10                   # CMS version
Via: 1.1 varnish (Varnish/6.6)          # Proxy/cache info
X-Varnish: 123456                       # Cache identifier
X-Amzn-Trace-Id: Root=1-abc-def        # AWS trace ID
```

---

## 6. Error Message Analysis

```bash
# Trigger errors to extract information
# SQL errors
curl "https://target.com/page?id=1'"
# "MySQL Error: You have an error in your SQL syntax"
# Reveals: Database type, potentially query structure

# Stack traces
curl "https://target.com/api/endpoint" -X POST -d "invalid"
# File paths: /var/www/app/controllers/api.py:42
# Framework: Django, Spring, Express, etc.
# Dependencies: Library versions in stack trace

# 404 with path disclosure
curl "https://target.com/nonexistent"
# "File not found: /var/www/html/nonexistent"
# Reveals: Document root path

# Debug pages
curl "https://target.com/__debug__/"
curl "https://target.com/elmah.axd"
curl "https://target.com/trace.axd"
```

---

## 7. Automated Metadata Collection

```bash
# Comprehensive one-liner
echo "=== COMMENTS ===" && \
curl -s https://target.com | grep -oP '<!--.*?-->' && \
echo "=== META TAGS ===" && \
curl -s https://target.com | grep -oE '<meta[^>]*>' && \
echo "=== HEADERS ===" && \
curl -sI https://target.com | grep -iE \
  "(server|x-powered|x-aspnet|x-debug|x-generator|via)" && \
echo "=== JS SECRETS ===" && \
curl -s https://target.com | grep -oE 'src="[^"]*\.js"' | \
  head -5 | while read -r src; do
    url=$(echo "$src" | grep -oE '"[^"]*"' | tr -d '"')
    curl -s "https://target.com$url" | \
      grep -iE "(api.?key|secret|password|token)" | head -3
  done
```

---

## 📊 Summary Table

| Source | Information Type | Tool |
|--------|-----------------|------|
| HTML comments | Internal notes, paths, creds | grep, Burp |
| Meta tags | CMS, author, config | grep, view source |
| JS comments | API keys, TODOs, debug | grep, JS analysis |
| File metadata | Usernames, paths, software | exiftool, FOCA |
| Response headers | Server info, versions | cURL, Nmap |
| Error messages | Stack traces, DB info | Manual testing |

---

## ❓ Revision Questions

1. What types of sensitive information are commonly found in HTML comments?
2. How can document metadata reveal internal usernames and paths?
3. Why should JavaScript files be searched for comments and debug statements?
4. What information can response headers inadvertently disclose?
5. How do error messages aid in technology fingerprinting?
6. What tools automate metadata extraction from downloaded files?

---

*Previous: [05-functionality-mapping.md](05-functionality-mapping.md)*

---

*[Back to README](../README.md)*
