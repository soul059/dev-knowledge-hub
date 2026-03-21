# Directory Enumeration

## Unit 3: Web Reconnaissance — Topic 3

## 🎯 Overview

Directory enumeration (also called directory brute-forcing or content discovery) is the process of finding hidden files, directories, and endpoints that aren't linked from the application's visible pages. This technique often uncovers admin panels, backup files, configuration files, and debug endpoints that represent critical attack vectors.

---

## 1. Why Directory Enumeration Matters

```
┌──────────────────────────────────────────────────────────────┐
│              HIDDEN CONTENT CATEGORIES                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Admin Panels       │  /admin, /administrator, /panel        │
│  Backup Files       │  /backup.sql, /db.bak, /site.zip      │
│  Config Files       │  /.env, /web.config, /config.php       │
│  Debug/Dev Tools    │  /phpinfo.php, /debug, /test           │
│  API Endpoints      │  /api/v1/, /graphql, /swagger          │
│  Source Control     │  /.git/, /.svn/, /.hg/                 │
│  Log Files          │  /error.log, /access.log, /debug.log   │
│  Old Versions       │  /old/, /v1/, /legacy/                 │
│  Install Scripts    │  /install.php, /setup/, /wizard        │
│  Upload Dirs        │  /uploads/, /media/, /files/           │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Directory Enumeration Tools

### Gobuster

```bash
# Directory mode
gobuster dir -u https://target.com -w /usr/share/wordlists/dirb/common.txt \
  -t 50 -o gobuster-results.txt

# With extensions
gobuster dir -u https://target.com \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt \
  -x php,html,txt,bak,old,conf -t 50

# With authentication
gobuster dir -u https://target.com \
  -w wordlist.txt -c "session=abc123" -t 50

# Filter by status code
gobuster dir -u https://target.com -w wordlist.txt \
  -s "200,204,301,302,307,401,403" -b ""
```

### ffuf (Fuzz Faster U Fool)

```bash
# Basic directory fuzzing
ffuf -u https://target.com/FUZZ -w wordlist.txt

# Filter by response size (remove common 404 pages)
ffuf -u https://target.com/FUZZ -w wordlist.txt -fs 4242

# Filter by status code
ffuf -u https://target.com/FUZZ -w wordlist.txt -fc 404

# Match specific codes
ffuf -u https://target.com/FUZZ -w wordlist.txt -mc 200,301,302,403

# Recursive scanning
ffuf -u https://target.com/FUZZ -w wordlist.txt -recursion -recursion-depth 3

# With extensions
ffuf -u https://target.com/FUZZ -w wordlist.txt \
  -e .php,.html,.txt,.bak,.zip,.conf

# Rate limiting
ffuf -u https://target.com/FUZZ -w wordlist.txt -rate 100
```

### Feroxbuster

```bash
# Recursive content discovery
feroxbuster -u https://target.com -w wordlist.txt \
  -x php,html,txt -d 3 -t 50

# With authentication
feroxbuster -u https://target.com -w wordlist.txt \
  -H "Cookie: session=abc123"
```

---

## 3. Wordlist Selection

| Wordlist | Size | Best For |
|----------|------|---------|
| `/usr/share/wordlists/dirb/common.txt` | 4,614 | Quick initial scan |
| `/usr/share/wordlists/dirbuster/directory-list-2.3-small.txt` | 87,664 | Standard scan |
| `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` | 220,560 | Thorough scan |
| `/usr/share/seclists/Discovery/Web-Content/raft-large-files.txt` | 37,042 | File discovery |
| `/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt` | 62,284 | Directory discovery |
| Custom wordlist | Variable | Technology-specific |

```bash
# Generate custom wordlist from site content
cewl https://target.com -d 3 -m 5 -w custom-wordlist.txt

# Technology-specific wordlists (SecLists)
# /Discovery/Web-Content/CMS/wordpress.fuzz.txt
# /Discovery/Web-Content/CMS/drupal.txt
# /Discovery/Web-Content/apache.txt
# /Discovery/Web-Content/nginx.txt
# /Discovery/Web-Content/IIS.fuzz.txt
```

---

## 4. Advanced Enumeration Techniques

### Backup File Discovery

```bash
# Common backup patterns
ffuf -u https://target.com/FUZZ -w filenames.txt \
  -e .bak,.old,.backup,.swp,.save,.copy,.orig,.tmp,.zip,.tar.gz,.sql

# Specific backup file patterns
for file in index login config database settings; do
  for ext in .bak .old .backup .orig .save "~" .swp .php.bak; do
    code=$(curl -s -o /dev/null -w "%{http_code}" \
      "https://target.com/${file}${ext}")
    [ "$code" != "404" ] && echo "${file}${ext} → $code"
  done
done
```

### Git Repository Exposure

```bash
# Check for exposed .git directory
curl -s https://target.com/.git/HEAD
# ref: refs/heads/main  ← Git repo exposed!

# Extract repository with git-dumper
git-dumper https://target.com/.git/ ./dumped-repo

# Or manually
curl https://target.com/.git/config
curl https://target.com/.git/refs/heads/main
# Use hash to download objects
```

### Virtual Host Enumeration

```bash
# Discover virtual hosts on same IP
ffuf -u https://target.com -H "Host: FUZZ.target.com" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -fs 4242

# Gobuster vhost mode
gobuster vhost -u https://target.com \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

---

## 5. Handling WAF and Rate Limiting

```bash
# Slow down requests
ffuf -u https://target.com/FUZZ -w wordlist.txt -rate 10

# Randomize User-Agent
ffuf -u https://target.com/FUZZ -w wordlist.txt \
  -H "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64)"

# Add jitter/delay
gobuster dir -u https://target.com -w wordlist.txt --delay 500ms

# Use different HTTP method
ffuf -u https://target.com/FUZZ -w wordlist.txt -X POST

# Rotate headers
ffuf -u https://target.com/FUZZ -w wordlist.txt \
  -H "X-Forwarded-For: 10.0.0.FUZZ_NUM"
```

---

## 6. Results Analysis

```
┌──────────────────────────────────────────────────────────────┐
│              INTERPRETING RESULTS                             │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Status Code │  Meaning           │  Action                  │
│  ────────────┼────────────────────┼──────────────────────── │
│  200         │  Accessible        │  Investigate content     │
│  301/302     │  Redirect          │  Follow, check dest      │
│  401         │  Auth required     │  Try credentials         │
│  403         │  Forbidden         │  Try bypass techniques   │
│  500         │  Server error      │  May indicate vuln       │
│  405         │  Method blocked    │  Try other methods       │
│                                                              │
│  Also check:                                                 │
│  • Response size (filter false positives)                    │
│  • Response body content (custom 404 pages)                  │
│  • Redirect destination                                      │
└──────────────────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Tool | Speed | Features | Best For |
|------|-------|----------|---------|
| Gobuster | Fast | Simple, reliable | Quick scans |
| ffuf | Very fast | Flexible filtering, fuzzing | Advanced fuzzing |
| Feroxbuster | Fast | Recursive, auto-filter | Deep discovery |
| Dirb | Slow | Simple, built-in wordlists | Basic scanning |
| Dirsearch | Medium | Python, good defaults | Easy setup |

---

## ❓ Revision Questions

1. What types of hidden content can directory enumeration uncover?
2. How do you choose the right wordlist for a target?
3. What is the significance of a 403 response during enumeration?
4. How can you detect exposed Git repositories on a web server?
5. What techniques help avoid WAF detection during enumeration?
6. Why should enumeration be performed with different file extensions?

---

*Previous: [02-technology-fingerprinting.md](02-technology-fingerprinting.md) | Next: [04-parameter-discovery.md](04-parameter-discovery.md)*

---

*[Back to README](../README.md)*
