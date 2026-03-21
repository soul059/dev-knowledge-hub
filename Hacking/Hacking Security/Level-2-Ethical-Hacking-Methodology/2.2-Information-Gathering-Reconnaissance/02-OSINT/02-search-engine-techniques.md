# Search Engine Techniques (Google Dorking)

## Unit 2 - Topic 2 | OSINT (Open Source Intelligence)

---

## Overview

**Google dorking** (also called Google hacking) uses advanced search operators to find sensitive information indexed by search engines. These techniques can reveal exposed files, login pages, configuration data, and vulnerabilities that were never meant to be public.

---

## 1. Essential Google Operators

| Operator | Purpose | Example |
|----------|---------|---------|
| `site:` | Restrict to domain | `site:target.com` |
| `inurl:` | Search in URL | `inurl:admin` |
| `intitle:` | Search in page title | `intitle:"index of"` |
| `intext:` | Search in page body | `intext:"password"` |
| `filetype:` | Find file types | `filetype:pdf` |
| `ext:` | File extension | `ext:sql` |
| `cache:` | Cached version | `cache:target.com` |
| `-` | Exclude terms | `site:target.com -www` |
| `""` | Exact phrase | `"error on line"` |
| `OR` | Either term | `admin OR login` |
| `*` | Wildcard | `"password is *"` |

---

## 2. Penetration Testing Dorks

```
# === SENSITIVE FILES ===
site:target.com filetype:pdf
site:target.com filetype:doc OR filetype:docx
site:target.com filetype:xls OR filetype:xlsx
site:target.com filetype:sql
site:target.com filetype:env
site:target.com filetype:log
site:target.com filetype:bak
site:target.com filetype:conf OR filetype:config

# === EXPOSED DIRECTORIES ===
site:target.com intitle:"index of"
site:target.com intitle:"index of" "backup"
site:target.com intitle:"index of" "password"

# === LOGIN & ADMIN PAGES ===
site:target.com inurl:login
site:target.com inurl:admin
site:target.com inurl:dashboard
site:target.com intitle:"admin panel"

# === SENSITIVE INFORMATION ===
site:target.com intext:"password"
site:target.com intext:"username" intext:"password" filetype:log
site:target.com "api_key" OR "api_secret"
site:target.com "BEGIN RSA PRIVATE KEY"

# === ERROR MESSAGES (reveal tech stack) ===
site:target.com "Fatal error" "on line"
site:target.com "Warning: mysql"
site:target.com "ORA-" (Oracle errors)
site:target.com "syntax error" "SQL"
site:target.com "stack trace" OR "traceback"

# === SUBDOMAINS ===
site:*.target.com -www
site:*.*.target.com
```

---

## 3. GitHub Dorking

```
# === SEARCH GITHUB FOR SECRETS ===
# Search: github.com/search

"target.com" password
"target.com" api_key
"target.com" secret
org:target-org password
org:target-org filename:.env
org:target-org filename:wp-config.php
"target.com" extension:sql
"target.com" "AWS_ACCESS_KEY"
"target.com" "PRIVATE KEY"

# === TOOLS FOR AUTOMATED GITHUB SEARCH ===
# trufflehog — scan repos for secrets
trufflehog github --org=target-org
# gitleaks — detect hardcoded secrets
gitleaks detect --source /path/to/repo
# gitrob — GitHub organization scanner
```

---

## 4. Beyond Google

| Search Engine | Advantage |
|--------------|-----------|
| **Bing** | Different index, may find unique results |
| **DuckDuckGo** | No tracking, different results |
| **Yandex** | Strong image search, Russian web |
| **Baidu** | Chinese web content |
| **Pastebin/Paste sites** | Leaked data, configs, credentials |
| **Wayback Machine** | Historical pages (removed content) |

---

## 5. Google Hacking Database (GHDB)

```
The Google Hacking Database (exploit-db.com/google-hacking-database)
contains thousands of pre-built dorks categorized by:

├── Files containing juicy info
├── Files containing passwords
├── Files containing usernames
├── Sensitive directories
├── Web server detection
├── Vulnerable files
├── Vulnerable servers
├── Error messages
├── Network or vulnerability data
├── Pages containing login portals
└── Various online devices
```

---

## Summary Table

| Technique | Operator | Use Case |
|-----------|----------|----------|
| **Domain scoping** | site: | Restrict to target |
| **File discovery** | filetype: | Find PDFs, SQLs, configs |
| **URL search** | inurl: | Find admin/login pages |
| **Content search** | intext: | Find passwords, keys |
| **Directory listing** | intitle:"index of" | Find exposed directories |
| **GitHub search** | org: filename: | Find leaked secrets |

---

## Quick Revision Questions

1. **What is Google dorking?**
2. **How do you find all PDFs on a target domain?**
3. **What does `intitle:"index of"` reveal?**
4. **Why search GitHub for target information?**
5. **What is the Google Hacking Database?**

---

[← Previous: OSINT Methodology](01-osint-methodology.md) | [Next: Social Media Intelligence →](03-social-media-intelligence.md)

---

*Unit 2 - Topic 2 of 6 | [Back to README](../README.md)*
