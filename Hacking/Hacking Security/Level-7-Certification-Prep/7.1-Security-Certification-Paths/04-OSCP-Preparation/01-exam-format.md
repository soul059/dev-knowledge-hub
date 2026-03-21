# Unit 4: OSCP Preparation — Topic 1: Exam Format and Requirements

## Overview

The **Offensive Security Certified Professional (OSCP)** is widely regarded as the gold standard for practical penetration testing certifications. Unlike MCQ-based exams, the OSCP requires you to compromise real machines in a proctored 24-hour exam. This topic covers the exam format, prerequisites, and what to expect.

---

## 1. OSCP Exam Specifications

```
OSCP EXAM DETAILS:

  Certification:   Offensive Security Certified Professional
  Course:          PEN-200 (Penetration Testing with Kali Linux)
  Exam Duration:   23 hours 45 minutes (hacking)
                   + 24 hours (report writing)
  Passing Score:   70 out of 100 points
  Format:          Hands-on practical exam
  Proctoring:      Live webcam proctoring
  Cost:            $1,749 (90-day lab access + 1 exam attempt)
                   $2,499 (365-day lab access + 2 attempts)
  Prerequisites:   None (but experience strongly recommended)
  Validity:        Does not expire

EXAM STRUCTURE (2023+):
  → 3 standalone machines (20 points each = 60)
  → 1 Active Directory set (3 machines = 40 points)
  → Total possible: 100 points
  → Pass: 70 points
  → Bonus points: Up to 10 from lab exercises
    (30 lab exercises + 80% Topic exercises)

POINT BREAKDOWN:
  Component           | Points | Notes
  Standalone Box 1    | 20     | Local + proof.txt
  Standalone Box 2    | 20     | Local + proof.txt
  Standalone Box 3    | 20     | Local + proof.txt
  AD Set (3 machines) | 40     | All-or-nothing
  Bonus Points        | 10     | Lab exercises
  TOTAL               | 110    | Need 70 to pass

CRITICAL RULES:
  → No automated exploitation tools (SQLmap, etc.)
  → No commercial tools (Burp Pro, Cobalt Strike)
  → Metasploit: Limited to ONE machine only
  → Must document everything for report
  → Proof files: proof.txt and local.txt
  → Screenshot requirements for each machine
```

---

## 2. PEN-200 Course Content

```
PEN-200 COURSE MODULES:

  Module                              | Topics
  1. Penetration Testing with Kali    | Methodology overview
  2. Report Writing                   | Documentation
  3. Information Gathering            | Passive/active recon
  4. Vulnerability Scanning           | Nmap, Nessus
  5. Web App Attacks                  | SQL injection, XSS, LFI
  6. Client-Side Attacks              | Macros, HTA, phishing
  7. Finding Public Exploits          | ExploitDB, searchsploit
  8. Fixing Exploits                  | Modifying exploit code
  9. Antivirus Evasion                | Encoding, obfuscation
  10. Password Attacks                | Brute force, cracking
  11. Port Redirection & Tunneling    | SSH, chisel, proxychains
  12. Linux Privilege Escalation      | SUID, sudo, kernel
  13. Windows Privilege Escalation    | Services, tokens
  14. Active Directory Attacks        | Kerberoasting, AS-REP
  15. Assembly Language Intro         | Buffer overflow basics
  16. Buffer Overflows (Windows)      | Stack-based BOF

LEARNING FORMAT:
  → PDF courseware (850+ pages)
  → Video demonstrations
  → Lab exercises (hands-on)
  → Challenge labs (OSCP-like)
  → Discord community access
  → Student forum

LAB ENVIRONMENT:
  → ~75 machines in practice labs
  → Multiple networks to pivot through
  → Varying difficulty levels
  → Realistic enterprise environment
  → Active Directory domain
```

---

## 3. Technical Prerequisites

```
RECOMMENDED KNOWLEDGE:

NETWORKING:
  → TCP/IP fundamentals
  → Subnetting and routing
  → Common protocols (HTTP, SSH, SMB, DNS)
  → Wireshark/tcpdump proficiency
  → Port scanning concepts

LINUX:
  → Command line proficiency
  → File permissions (chmod, chown)
  → Service management
  → Scripting (Bash basics)
  → Package management
  → Text processing (grep, awk, sed)

WINDOWS:
  → PowerShell basics
  → Command prompt usage
  → Active Directory concepts
  → Windows services
  → Registry understanding
  → File system navigation

PROGRAMMING:
  → Python (scripting, exploit modification)
  → Bash scripting
  → Basic understanding of C
  → Web languages (HTML, JavaScript, PHP)
  → SQL query syntax
  → PowerShell scripting

SECURITY TOOLS:
  → Nmap (fluent)
  → Burp Suite Community
  → Metasploit Framework
  → Gobuster/Feroxbuster
  → Netcat/Socat
  → John the Ripper/Hashcat
  → Chisel/SSH tunneling

SELF-ASSESSMENT: If you can do these, 
you're ready to start PEN-200:
  [ ] Solve easy HTB/THM machines
  [ ] Write basic Python scripts
  [ ] Comfortable on Linux CLI
  [ ] Understand networking basics
  [ ] Know common web vulnerabilities
```

---

## 4. Exam Environment

```
EXAM DAY LOGISTICS:

BEFORE EXAM:
  → Test VPN connection
  → Verify webcam and microphone
  → Government-issued ID ready
  → Clear desk (exam rules)
  → Stable internet connection
  → Second monitor allowed
  → Energy food/drinks prepared
  → Bathroom breaks allowed (notify proctor)

EXAM VPN:
  → Connect via OpenVPN
  → Receive exam machines IP addresses
  → Access exam control panel
  → Submit proof screenshots
  → 23 hours 45 minutes from start

WHAT'S PROVIDED:
  → Kali Linux VM (preconfigured)
  → VPN connectivity pack
  → Exam control panel (web)
  → Machine IP addresses
  → Exam guide document

WHAT'S ALLOWED:
  → Kali Linux tools (most)
  → Custom scripts you wrote
  → Nmap, Gobuster, Burp Community
  → Metasploit (ONE machine only)
  → Personal notes
  → Internet access for research
  → CherryTree, Obsidian, etc.

WHAT'S PROHIBITED:
  → SQLmap, automated exploitation
  → Commercial tools
  → Spoofing (IP, ARP, DNS)
  → DoS/DDoS attacks
  → Modifying exam network
  → External help from others
  → AI tools for exploitation
```

---

## 5. Report Requirements

```
EXAM REPORT:

REPORT TIMELINE:
  → 24 hours after exam ends
  → Must be uploaded via portal
  → Professional-quality document
  → PDF format (no other formats)

REPORT STRUCTURE:
  1. Cover page
  2. Executive summary
  3. Methodology overview
  4. For each machine:
     a. Service enumeration
     b. Vulnerability identified
     c. Exploitation steps (reproducible)
     d. Post-exploitation
     e. Screenshots with proof
     f. Remediation recommendations
  5. Appendices

SCREENSHOT REQUIREMENTS:
  → IP address visible (ifconfig/ipconfig)
  → proof.txt contents displayed
  → Hostname shown
  → User context (whoami)
  → Commands used visible
  → One screenshot per proof

REPORT TIPS:
  → Start report template BEFORE exam
  → Screenshot EVERYTHING during exam
  → Document exact commands used
  → Include full tool output
  → Make steps reproducible
  → Use consistent formatting
  → Write as you go (during exam)
  → Proofread before submitting
  → Use provided report template

COMMON REPORT FAILURES:
  → Missing screenshots
  → Steps not reproducible
  → Incomplete documentation
  → Poor formatting
  → Submitted late
  → Wrong file format
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Duration | 24h exam + 24h report |
| Points | 100 total, 70 to pass |
| Machines | 3 standalone + 1 AD set |
| Cost | $1,749 (90-day) / $2,499 (365-day) |
| Metasploit | ONE machine only |
| Validity | Never expires |
| Report | PDF, professional quality |

---

## Revision Questions

1. How many points are needed to pass the OSCP exam?
2. What restrictions exist on using Metasploit during the exam?
3. What are the components of the OSCP exam point structure?
4. What must be included in the post-exam report?
5. What tools are prohibited during the OSCP exam?

---

*Previous: None (First topic in this unit) | Next: [02-lab-strategy.md](02-lab-strategy.md)*

---

*[Back to README](../README.md)*
