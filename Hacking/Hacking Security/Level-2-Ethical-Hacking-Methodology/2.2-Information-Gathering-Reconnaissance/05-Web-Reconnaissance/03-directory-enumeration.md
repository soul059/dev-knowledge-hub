# Directory Enumeration

## Unit 5 - Topic 3 | Web Reconnaissance

---

## Overview

**Directory enumeration** (also called directory brute-forcing or content discovery) systematically discovers hidden files, directories, and endpoints on a web server that aren't linked from the main site. This reveals admin panels, backup files, configuration files, and other sensitive resources.

---

## 1. Why Directory Enumeration?

```
HIDDEN CONTENT THAT ENUMERATION REVEALS:

ADMIN INTERFACES:
├── /admin/         → Administration panel
├── /wp-admin/      → WordPress admin
├── /administrator/ → Joomla admin
├── /cpanel/        → cPanel hosting control
└── /phpmyadmin/    → Database management

SENSITIVE FILES:
├── /backup.zip     → Full site backup
├── /.env           → Environment variables (passwords!)
├── /config.php     → Configuration with DB credentials
├── /web.config     → IIS configuration
├── /.git/          → Source code repository
└── /.htaccess      → Apache configuration

DEVELOPMENT ARTIFACTS:
├── /test/          → Test pages
├── /debug/         → Debug endpoints
├── /api/docs/      → API documentation
├── /swagger/       → Swagger API docs
└── /status/        → Server status page
```

---

## 2. Primary Tools

```bash
# === GOBUSTER ===
gobuster dir -u https://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -t 50 \
  -o gobuster_results.txt

# With extensions:
gobuster dir -u https://target.com \
  -w /usr/share/wordlists/dirb/common.txt \
  -x php,html,txt,bak,old,zip \
  -t 50

# === FEROXBUSTER (Recursive) ===
feroxbuster -u https://target.com \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  --depth 3 \
  -t 50

# === FFUF (Fast Web Fuzzer) ===
ffuf -u https://target.com/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -mc 200,301,302,403 \
  -o results.json

# File extension fuzzing:
ffuf -u https://target.com/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt \
  -e .php,.bak,.old,.zip,.sql,.conf

# === DIRB ===
dirb https://target.com /usr/share/dirb/wordlists/common.txt

# === DIRSEARCH ===
dirsearch -u https://target.com -e php,html,js -t 50
```

---

## 3. Wordlist Selection

| Wordlist | Size | Use Case |
|----------|:----:|----------|
| `/usr/share/dirb/common.txt` | ~4,600 | Quick initial scan |
| `raft-small-directories.txt` | ~20,000 | Standard scan |
| `raft-medium-directories.txt` | ~30,000 | Thorough scan |
| `raft-large-directories.txt` | ~60,000 | Comprehensive scan |
| `directory-list-2.3-medium.txt` | ~220,000 | Exhaustive scan |
| `big.txt` | ~20,000 | General purpose |
| Custom wordlist | Varies | Target-specific terms |

```bash
# === CUSTOM WORDLIST GENERATION ===
# CeWL — crawl website for custom words
cewl https://target.com -d 3 -m 5 -w custom_wordlist.txt

# Combine with standard lists:
cat custom_wordlist.txt /usr/share/seclists/Discovery/Web-Content/common.txt \
  | sort -u > combined.txt
```

---

## 4. Advanced Techniques

```bash
# === FILTER BY RESPONSE SIZE ===
# Filter out 404 pages by response size
ffuf -u https://target.com/FUZZ \
  -w wordlist.txt \
  -fs 4242    # Filter responses of 4242 bytes (the 404 page)

# === RECURSIVE SCANNING ===
# Scan found directories recursively
feroxbuster -u https://target.com --depth 4

# === VHOST/SUBDOMAIN FUZZING ===
ffuf -u https://target.com \
  -H "Host: FUZZ.target.com" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -fs 0

# === API ENDPOINT DISCOVERY ===
ffuf -u https://target.com/api/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt

# === BACKUP FILE PATTERNS ===
# Check for backup variations of known files:
# index.php.bak, index.php.old, index.php~
# .index.php.swp (vim swap files)
```

---

## 5. Interpreting Results

| Status Code | Meaning | Action |
|:-----------:|---------|--------|
| **200** | Accessible | Investigate content |
| **301/302** | Redirect | Follow → check destination |
| **401** | Auth required | Try default credentials |
| **403** | Forbidden | May still exist — try bypass |
| **500** | Server error | Investigate for injection |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Purpose** | Find hidden files and directories |
| **Gobuster** | Fast directory brute-forcing |
| **Feroxbuster** | Recursive content discovery |
| **ffuf** | Flexible fuzzing tool |
| **Wordlists** | Start small, go bigger as needed |
| **Custom wordlists** | CeWL for target-specific terms |

---

## Quick Revision Questions

1. **What is directory enumeration and why is it important?**
2. **Compare Gobuster, Feroxbuster, and ffuf.**
3. **How do you choose the right wordlist?**
4. **What does a 403 response during enumeration indicate?**
5. **How does CeWL generate custom wordlists?**

---

[← Previous: Technology Fingerprinting](02-technology-fingerprinting.md) | [Next: Robots.txt and Sitemap →](04-robots-txt-and-sitemap.md)

---

*Unit 5 - Topic 3 of 6 | [Back to README](../README.md)*
