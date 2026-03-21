# Script Scanning (NSE)

## Unit 2 - Topic 3 | Service Enumeration

---

## Overview

The **Nmap Scripting Engine (NSE)** extends Nmap with hundreds of scripts for advanced enumeration, vulnerability detection, and exploitation. NSE transforms Nmap from a port scanner into a comprehensive security assessment tool.

---

## 1. NSE Script Categories

| Category | Purpose | Example |
|----------|---------|---------|
| **auth** | Authentication testing | `ssh-auth-methods` |
| **broadcast** | Network broadcast discovery | `broadcast-dhcp-discover` |
| **brute** | Brute-force attacks | `ssh-brute`, `http-brute` |
| **default** | Safe, useful scripts (-sC) | `http-title`, `ssl-cert` |
| **discovery** | Service/network discovery | `http-enum`, `dns-brute` |
| **dos** | Denial of service testing | `http-slowloris` |
| **exploit** | Active exploitation | `http-shellshock` |
| **external** | Query external services | `whois-ip` |
| **fuzzer** | Fuzzing for crashes | `http-form-fuzzer` |
| **intrusive** | Aggressive scripts | `smb-vuln-ms17-010` |
| **malware** | Malware detection | `http-malware-host` |
| **safe** | Won't crash services | `http-headers` |
| **version** | Enhanced version detection | `sshv1` |
| **vuln** | Vulnerability detection | `vuln` |

---

## 2. Running NSE Scripts

```bash
# === DEFAULT SCRIPTS (-sC) ===
nmap -sC target.com
# Runs all scripts in the "default" category
# Safe and informative

# === SPECIFIC SCRIPT ===
nmap --script http-title target.com
nmap --script ssl-cert -p 443 target.com

# === SCRIPT CATEGORY ===
nmap --script vuln target.com           # All vulnerability scripts
nmap --script "safe and discovery" target.com
nmap --script "default or safe" target.com
nmap --script "not intrusive" target.com

# === WILDCARD ===
nmap --script "http-*" target.com       # All HTTP scripts
nmap --script "smb-vuln*" target.com    # All SMB vuln scripts
nmap --script "ssh-*" target.com        # All SSH scripts

# === SCRIPT ARGUMENTS ===
nmap --script http-brute --script-args \
  http-brute.path=/admin,userdb=users.txt,passdb=pass.txt \
  target.com

# === LIST AVAILABLE SCRIPTS ===
ls /usr/share/nmap/scripts/ | head -20
nmap --script-help "http-*"            # Help for HTTP scripts
```

---

## 3. Essential NSE Scripts

```bash
# ══════════════════════════════════════
# ENUMERATION SCRIPTS
# ══════════════════════════════════════
nmap --script http-enum -p 80 target.com          # Web directory enumeration
nmap --script http-title -p 80 target.com         # Page titles
nmap --script http-headers -p 80 target.com       # HTTP headers
nmap --script smb-enum-shares -p 445 target.com   # SMB shares
nmap --script smb-enum-users -p 445 target.com    # SMB users
nmap --script dns-brute target.com                # DNS subdomain brute-force
nmap --script nbstat -p 137 target.com            # NetBIOS info

# ══════════════════════════════════════
# VULNERABILITY SCRIPTS
# ══════════════════════════════════════
nmap --script vuln target.com                      # All vuln scripts
nmap --script smb-vuln-ms17-010 -p 445 target.com # EternalBlue
nmap --script ssl-heartbleed -p 443 target.com     # Heartbleed
nmap --script http-shellshock -p 80 target.com     # Shellshock
nmap --script http-vuln-cve2017-5638 target.com    # Struts RCE

# ══════════════════════════════════════
# BRUTE FORCE SCRIPTS
# ══════════════════════════════════════
nmap --script ssh-brute -p 22 target.com           # SSH brute
nmap --script ftp-brute -p 21 target.com           # FTP brute
nmap --script http-brute -p 80 target.com          # HTTP auth brute
nmap --script mysql-brute -p 3306 target.com       # MySQL brute
nmap --script smb-brute -p 445 target.com          # SMB brute

# ══════════════════════════════════════
# INFORMATION GATHERING
# ══════════════════════════════════════
nmap --script ssl-cert -p 443 target.com           # SSL certificate
nmap --script ssl-enum-ciphers -p 443 target.com   # TLS versions/ciphers
nmap --script smb-os-discovery -p 445 target.com   # Windows OS version
nmap --script rdp-ntlm-info -p 3389 target.com    # RDP info + domain
nmap --script smtp-commands -p 25 target.com       # SMTP capabilities
```

---

## 4. Combining NSE with Scanning

```bash
# === FULL ASSESSMENT RECIPE ===
sudo nmap -sS -sV -sC -O -p- \
  --script "default,vuln,discovery" \
  -T4 -oA full_assessment \
  target.com

# === WEB APPLICATION SCAN ===
nmap -sV -p 80,443,8080 \
  --script "http-enum,http-title,http-headers,http-methods,http-vuln*" \
  target.com

# === SMB FULL ASSESSMENT ===
nmap -p 445 \
  --script "smb-enum*,smb-vuln*,smb-os-discovery" \
  target.com

# === SSL/TLS ASSESSMENT ===
nmap -p 443 \
  --script "ssl-cert,ssl-enum-ciphers,ssl-heartbleed,ssl-poodle,ssl-dh-params" \
  target.com
```

---

## 5. Writing Custom NSE Scripts

```lua
-- Simple custom NSE script example
-- Save as: /usr/share/nmap/scripts/http-custom-check.nse

local http = require "http"
local shortport = require "shortport"

description = [[
Checks for a specific file on web servers.
]]

portrule = shortport.http

action = function(host, port)
  local response = http.get(host, port, "/.env")
  if response.status == 200 then
    return "FOUND: .env file is publicly accessible!"
  end
end
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **-sC** | Run default scripts (safe + informative) |
| **--script vuln** | All vulnerability detection scripts |
| **--script "http-*"** | Wildcard script selection |
| **Categories** | default, vuln, safe, brute, discovery |
| **Script args** | `--script-args key=value` |
| **Custom scripts** | Written in Lua, stored in /scripts/ |

---

## Quick Revision Questions

1. **What does the `-sC` flag do?**
2. **How do you run all vulnerability scripts?**
3. **What script detects the EternalBlue vulnerability?**
4. **How do you pass arguments to NSE scripts?**
5. **What programming language are NSE scripts written in?**

---

[← Previous: Version Detection](02-version-detection.md) | [Next: Protocol-Specific Enumeration →](04-protocol-specific-enumeration.md)

---

*Unit 2 - Topic 3 of 6 | [Back to README](../README.md)*
