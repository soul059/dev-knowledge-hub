# Source Code Analysis

## Unit 5 - Topic 6 | Web Reconnaissance

---

## Overview

**Source code analysis** during reconnaissance involves finding and analyzing publicly accessible source code from the target — through code repositories, exposed `.git` directories, JavaScript files, and other code leaks. This reveals credentials, API keys, internal logic, and application architecture.

---

## 1. Public Code Repositories

```bash
# === GITHUB SEARCH ===
# Search for target's organization:
# github.com/orgs/target-corp/repositories

# GitHub Dork queries (search.github.com):
# "target.com" password
# "target.com" api_key
# "target.com" secret
# org:target-corp password
# org:target-corp AWS_SECRET

# === GITHUB DORKING EXAMPLES ===
# Find secrets:
"target.com" filename:.env
"target.com" filename:config.php password
org:target-corp filename:id_rsa
org:target-corp extension:pem private
"target.com" filename:wp-config.php

# Find internal URLs:
org:target-corp "internal" "staging"
org:target-corp "10.0.0" OR "192.168"

# === GITLAB / BITBUCKET ===
# Also search on gitlab.com and bitbucket.org
# Some orgs accidentally use public repos on all platforms
```

---

## 2. Automated Secret Scanning

```bash
# === TRUFFLEHOG ===
# Scans git repos for secrets (high-entropy strings, patterns)
trufflehog git https://github.com/target-corp/web-app
# Finds: API keys, passwords, tokens across ALL commits

# === GITROB ===
# Scans org repos for sensitive files
gitrob analyze target-corp

# === GITLEAKS ===
gitleaks detect --source https://github.com/target-corp/repo
# Detects: AWS keys, private keys, passwords, tokens

# === GIT-SECRETS ===
git secrets --scan  # Scan local repo for secrets

# === COMMON SECRETS FOUND ===
# ├── AWS Access Key: AKIA...
# ├── AWS Secret Key: long base64 string
# ├── Database passwords: DB_PASS=...
# ├── API tokens: Authorization: Bearer ...
# ├── Private SSH keys: -----BEGIN RSA PRIVATE KEY-----
# ├── OAuth secrets: client_secret=...
# └── JWT signing keys: JWT_SECRET=...
```

---

## 3. Exposed .git Directories

```bash
# === CHECK FOR EXPOSED .GIT ===
curl -s https://target.com/.git/HEAD
# If returns "ref: refs/heads/main" → .git is exposed!

curl -s https://target.com/.git/config
# May reveal: remote URLs, branch names, author emails

# === DUMP ENTIRE REPOSITORY ===
# git-dumper — extracts full repo from exposed .git
git-dumper https://target.com/.git/ ./dumped_repo

# GitTools — another repo dumper
./gitdumper.sh https://target.com/.git/ dumped_repo
./extractor.sh dumped_repo extracted_repo

# === WHAT YOU GET ===
# ├── Complete source code
# ├── All commit history
# ├── Configuration files
# ├── Hardcoded credentials
# ├── Internal comments
# └── Author information (names, emails)

# === OTHER EXPOSED VCS ===
curl -s https://target.com/.svn/entries        # Subversion
curl -s https://target.com/.hg/requires        # Mercurial
curl -s https://target.com/.bzr/README         # Bazaar
curl -s https://target.com/CVS/Root            # CVS
```

---

## 4. JavaScript Source Code Analysis

```bash
# === EXTRACT JS FILES ===
curl -s https://target.com | grep -oP 'src="[^"]*\.js"'
# Download all JS files for analysis

# === BEAUTIFY MINIFIED JS ===
# js-beautify — readable format
js-beautify app.min.js > app.js

# === FIND SECRETS IN JS ===
grep -rn "api_key\|apiKey\|password\|secret\|token" app.js

# === FIND API ENDPOINTS ===
grep -oP '"/(api|v[0-9])/[^"]*"' app.js

# === FIND HIDDEN FUNCTIONALITY ===
grep -oP '"/(admin|debug|test|internal)/[^"]*"' app.js

# === JS SOURCE MAPS ===
curl -s https://target.com/js/app.js.map
# Source maps contain ORIGINAL unminified source code!
# Check for: *.js.map files for every .js file
```

---

## 5. Intelligence Summary

| Source | What It Reveals | Tools |
|--------|----------------|-------|
| **GitHub repos** | Source code, secrets, internal URLs | GitHub dorks, TruffleHog |
| **Exposed .git** | Full source + commit history | git-dumper, GitTools |
| **JavaScript** | API endpoints, hidden features | LinkFinder, grep |
| **JS source maps** | Original unminified source | curl, browser DevTools |
| **Package files** | Dependencies + versions | package.json, requirements.txt |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **GitHub dorking** | Search for secrets in public repos |
| **TruffleHog** | Automated secret scanning in git history |
| **Exposed .git** | Full source code dump from web server |
| **JavaScript analysis** | API keys, endpoints, hidden functionality |
| **Source maps** | .js.map files reveal original source |
| **Impact** | Credentials, architecture, vulnerabilities |

---

## Quick Revision Questions

1. **What GitHub dork queries find secrets?**
2. **How do you detect an exposed .git directory?**
3. **What tools scan git repos for hardcoded secrets?**
4. **Why are JavaScript source maps valuable?**
5. **What types of secrets are commonly found in source code?**

---

[← Previous: Web Archives](05-web-archives.md)

---

*Unit 5 - Topic 6 of 6 | [Back to README](../README.md) | [Next Unit: Email Reconnaissance →](../06-Email-Reconnaissance/01-email-harvesting.md)*
