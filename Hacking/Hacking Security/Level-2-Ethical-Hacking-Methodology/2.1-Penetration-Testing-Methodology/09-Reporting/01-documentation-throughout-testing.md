# Documentation Throughout Testing

## Unit 9 - Topic 1 | Reporting

---

## Overview

**Documentation throughout testing** is the practice of recording every action, finding, and observation *during* the pen test — not after. Good documentation is the foundation of a quality report. If you didn't document it, it didn't happen.

---

## 1. Why Continuous Documentation?

```
┌──────────────────────────────────────────────────────────────┐
│              DOCUMENTATION IMPORTANCE                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  REPRODUCIBILITY                                             │
│  └── Client must be able to reproduce findings              │
│                                                              │
│  EVIDENCE                                                    │
│  └── Proof that vulnerability exists                         │
│                                                              │
│  TIMELINE                                                    │
│  └── When each action was taken                              │
│                                                              │
│  ACCURACY                                                    │
│  └── Details are fresh during testing (not days later)       │
│                                                              │
│  LEGAL PROTECTION                                            │
│  └── Proves you stayed within scope                          │
│                                                              │
│  REPORT QUALITY                                              │
│  └── Complete notes → complete report                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. What to Document

```
DOCUMENTATION CHECKLIST FOR EVERY ACTION:

FOR EVERY SCAN/ATTACK:
├── Timestamp (date + time)
├── Source IP (your attack machine)
├── Target IP/hostname
├── Tool used + version
├── Full command executed
├── Output received (copy/paste or screenshot)
├── Result/interpretation
└── Next step planned

FOR EVERY FINDING:
├── Vulnerability name
├── Affected host(s)
├── Affected service/port
├── Steps to reproduce (numbered)
├── Evidence (screenshots, output)
├── Severity assessment
├── Business impact
└── Remediation recommendation

FOR EVERY CREDENTIAL FOUND:
├── Where found (file, database, memory)
├── What access it provides
├── Hash type (if applicable)
├── Whether it was cracked
└── Where it was reused
```

---

## 3. Documentation Tools

```bash
# === TERMINAL LOGGING ===
# script command (records everything)
script -a /root/pentest/logs/session_$(date +%Y%m%d_%H%M).log
# Captures all terminal input/output with timestamps

# === SCREENSHOTS ===
# Linux: Flameshot, scrot, import
flameshot gui                          # Interactive screenshot
scrot -s screenshot_%Y%m%d_%H%M.png   # Select area

# === NOTE-TAKING TOOLS ===
# CherryTree — hierarchical notes with rich formatting
# Obsidian — Markdown-based knowledge base
# KeepNote — simple, organized notes
# Joplin — encrypted markdown notes

# === AUTOMATED LOGGING ===
# Metasploit
msf6> spool /root/pentest/msf_log.txt  # Log all MSF output

# Burp Suite
# Project → save to file (auto-saves)

# Nmap
nmap -oA /root/pentest/scans/target_full 10.0.0.5
# Saves .nmap, .xml, .gnmap

# === EVIDENCE ORGANIZATION ===
pentest_project/
├── 01-scope/
│   ├── scope_document.pdf
│   └── authorization.pdf
├── 02-recon/
│   ├── osint_findings.md
│   └── screenshots/
├── 03-scanning/
│   ├── nmap_results/
│   └── vuln_scan_results/
├── 04-exploitation/
│   ├── exploitation_log.md
│   └── screenshots/
├── 05-post-exploitation/
│   ├── privesc_evidence/
│   ├── lateral_movement_log.md
│   └── data_access_proof/
├── 06-evidence/
│   └── all_screenshots/
└── 07-report/
    └── draft/
```

---

## 4. Activity Log Template

```
═══════════════════════════════════════════════
PENETRATION TEST ACTIVITY LOG
═══════════════════════════════════════════════
Project: [Client Name] — External Penetration Test
Tester: [Name]
Date: [Date]
═══════════════════════════════════════════════

09:00 | Started engagement
       Source: 203.0.113.50 (VPN)
       Targets: 10.0.0.0/24

09:05 | Host discovery scan
       Command: nmap -sn 10.0.0.0/24
       Result: 12 hosts discovered
       Evidence: scans/host_discovery.nmap

09:15 | Port scan — 10.0.0.5
       Command: nmap -sS -sV -p- 10.0.0.5
       Result: Ports 22, 80, 443, 3306 open
       Evidence: scans/10.0.0.5_full.nmap

09:30 | Web enumeration — 10.0.0.5:80
       Command: gobuster dir -u http://10.0.0.5 -w common.txt
       Result: /admin, /backup, /api found
       Evidence: screenshots/gobuster_10.0.0.5.png
       NOTE: /backup directory lists sensitive files!

09:45 | FINDING: Directory listing on /backup
       Severity: HIGH
       Evidence: screenshots/dir_listing_01.png
```

---

## 5. Best Practices

| Practice | Why |
|----------|-----|
| **Log continuously** | Don't rely on memory |
| **Use timestamps** | Establishes timeline |
| **Screenshot everything** | Visual proof of findings |
| **Save raw output** | Can re-analyze later |
| **Organize by phase** | Easy to find evidence |
| **Hash evidence files** | Proves integrity |
| **Backup regularly** | Don't lose evidence |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **When** | Document during testing, not after |
| **What** | Every command, output, finding, screenshot |
| **Tools** | script, CherryTree, Obsidian, Flameshot |
| **Organization** | Folder structure by phase |
| **Activity log** | Timestamped record of all actions |
| **Evidence** | Screenshots, output files, hashes |

---

## Quick Revision Questions

1. **Why should you document during testing, not after?**
2. **What should be recorded for every finding?**
3. **Name 3 documentation tools for pen testing.**
4. **How does the `script` command help with documentation?**
5. **Why hash evidence files?**

---

[Next: Report Structure →](02-report-structure.md)

---

*Unit 9 - Topic 1 of 7 | [Back to README](../README.md)*
