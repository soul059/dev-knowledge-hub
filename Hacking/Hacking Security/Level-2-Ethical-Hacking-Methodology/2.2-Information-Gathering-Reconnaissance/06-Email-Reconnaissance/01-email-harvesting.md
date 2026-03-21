# Email Harvesting

## Unit 6 - Topic 1 | Email Reconnaissance

---

## Overview

**Email harvesting** collects email addresses associated with the target organization. These emails enable social engineering attacks, credential stuffing, password spraying, and help identify the organization's email format for generating additional target addresses.

---

## 1. Email Format Discovery

```
COMMON EMAIL FORMAT PATTERNS:

FIRST.LAST@target.com       → john.smith@target.com
FIRST_LAST@target.com       → john_smith@target.com
FIRSTLAST@target.com        → johnsmith@target.com
FIRST@target.com            → john@target.com
FLAST@target.com            → jsmith@target.com
FIRSTL@target.com           → johns@target.com
F.LAST@target.com           → j.smith@target.com

DISCOVERY METHODS:
├── Find one known email → deduce pattern
├── Check email headers from received mail
├── LinkedIn employee profiles
├── hunter.io → shows detected format
└── phonebook.cz → email pattern detection
```

---

## 2. Email Harvesting Tools

```bash
# === THEHARVESTER ===
theHarvester -d target.com -b all
# Sources: Google, Bing, LinkedIn, Twitter, etc.
# Returns: emails, hosts, IPs, names

theHarvester -d target.com -b google -l 500
# -l 500: check 500 results

# === HUNTER.IO ===
# hunter.io — search by domain
# Shows: email format, confidence score, sources
# API: curl "https://api.hunter.io/v2/domain-search?domain=target.com&api_key=KEY"

# === PHONEBOOK.CZ ===
# phonebook.cz — massive email/domain/URL database
# Search by domain to find all associated emails

# === CLEARBIT CONNECT ===
# Gmail extension — find emails for a person at a company

# === LINKEDIN + CROSSLINKED ===
crosslinked -f '{first}.{last}@target.com' -t 'company:"Target Corporation"'
# Scrapes LinkedIn names → generates emails using format

# === OSINT TOOLS ===
# emailrep.io — reputation check for an email
# haveibeenpwned.com — check if email in data breaches
```

---

## 3. Search Engine Techniques

```bash
# === GOOGLE DORKS FOR EMAILS ===
# "@target.com"                       → Emails in quotes
# "@target.com" site:linkedin.com     → LinkedIn profiles
# "@target.com" filetype:pdf          → Emails in PDFs
# "@target.com" filetype:xlsx         → Emails in spreadsheets
# "@target.com" site:pastebin.com     → Leaked emails
# intext:"@target.com" -site:target.com → Emails on other sites

# === EXTRACT EMAILS FROM WEBSITE ===
# Crawl target website for emails:
curl -s https://target.com/contact | \
  grep -oP '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'

# Recursive crawl:
wget -r -l 3 https://target.com -O - 2>/dev/null | \
  grep -oP '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' | sort -u

# === METADATA FROM DOCUMENTS ===
# Download PDFs, DOCXs from target website
# Extract metadata with exiftool:
exiftool document.pdf
# Author: John Smith
# Creator: Microsoft Word 2019
# → john.smith@target.com (if format known)
```

---

## 4. Building Email Lists

```
EMAIL HARVESTING WORKFLOW:

STEP 1: Discover email format
├── Find 2-3 confirmed emails
├── Identify the pattern
└── first.last@target.com ← confirmed

STEP 2: Gather employee names
├── LinkedIn company page
├── About Us page
├── Press releases
├── Conference speakers
└── GitHub contributors

STEP 3: Generate email list
├── Apply format to all names
├── john.smith@target.com
├── jane.doe@target.com
├── bob.johnson@target.com
└── ... (50-500+ employees)

STEP 4: Verify emails
├── Email verification tools
├── SMTP VRFY command
├── Observe bounce vs accept
└── Mark confidence level

RESULT: Verified target email list
└── Ready for: phishing, password spray, etc.
```

---

## 5. Email Intelligence Table

| Source | Type | Example Output |
|--------|------|---------------|
| **theHarvester** | Automated | Emails from multiple search engines |
| **Hunter.io** | SaaS | Email format + verified emails |
| **CrossLinked** | LinkedIn scraper | Generated emails from names |
| **Google dorks** | Manual | Emails in docs and pages |
| **Metadata** | Document analysis | Author names → email format |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Email format** | Discover pattern (first.last@domain) |
| **theHarvester** | Multi-source email collection |
| **Hunter.io** | Format detection + verification |
| **CrossLinked** | LinkedIn names → email generation |
| **Google dorks** | "@target.com" filetype searches |
| **Verification** | Confirm emails before use |

---

## Quick Revision Questions

1. **Why is discovering the email format important?**
2. **What tools automate email harvesting?**
3. **How does CrossLinked generate email addresses?**
4. **What Google dorks find target email addresses?**
5. **How can document metadata reveal email addresses?**

---

[Next: Email Verification →](02-email-verification.md)

---

*Unit 6 - Topic 1 of 5 | [Back to README](../README.md)*
