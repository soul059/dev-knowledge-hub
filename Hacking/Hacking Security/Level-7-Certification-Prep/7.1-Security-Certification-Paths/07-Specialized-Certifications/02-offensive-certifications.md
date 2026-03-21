# Unit 7: Specialized Certifications — Topic 2: Offensive Security Certifications (OSWE, OSEP)

## Overview

Beyond the OSCP, **Offensive Security** offers advanced certifications that validate expert-level offensive capabilities. The **OSWE** (Web Expert) and **OSEP** (Experienced Penetration Tester) are among the most challenging and respected practical certifications in the industry.

---

## 1. Offensive Security Certification Path

```
OFFSEC CERTIFICATION HIERARCHY:

  ENTRY:
  ┌─────────────────────────────┐
  │ OSCP (PEN-200)              │
  │ Penetration Testing         │
  │ Foundation for all others   │
  └──────────┬──────────────────┘
             │
  ADVANCED:  ├────────────────────────┐
  ┌──────────┴──────────┐  ┌─────────┴──────────┐
  │ OSWE (WEB-200)      │  │ OSEP (PEN-300)     │
  │ Web App Expert      │  │ Evasion & Breach   │
  │ White-box testing   │  │ Advanced pen test  │
  └──────────┬──────────┘  └─────────┬──────────┘
             │                       │
  EXPERT:    └────────┬──────────────┘
  ┌──────────────────┴───────────────┐
  │ OSED (EXP-301)                   │
  │ Exploit Development              │
  │ Windows user-mode exploitation   │
  └──────────────────┬───────────────┘
                     │
  ┌──────────────────┴───────────────┐
  │ OSEE (EXP-401)                   │
  │ Advanced Exploitation            │
  │ Most difficult OffSec cert       │
  └──────────────────────────────────┘

CERTIFICATION DETAILS:
  Cert | Course  | Exam Duration | Cost
  OSCP | PEN-200 | 24h           | $1,749
  OSWE | WEB-200 | 48h           | $1,749
  OSEP | PEN-300 | 48h           | $1,749
  OSED | EXP-301 | 48h           | $1,749
  OSEE | EXP-401 | 72h           | In-person only
```

---

## 2. OSWE (Web Expert)

```
OSWE DETAILS:

  Course:          WEB-200 (Advanced Web Attacks
                   and Exploitation)
  Exam Duration:   48 hours (hacking)
                   + 24 hours (report)
  Passing:         Must complete objectives
  Format:          White-box web app testing
  Cost:            $1,749 (90-day lab + 1 attempt)
  Prerequisite:    OSCP recommended

COURSE CONTENT:
  → Source code review (white-box)
  → Authentication bypass
  → Session management flaws
  → .NET deserialization attacks
  → Java deserialization
  → Server-Side Template Injection (SSTI)
  → SQL injection (advanced)
  → Type juggling
  → SSRF exploitation
  → XSS to RCE chains
  → Custom exploit development
  → Multi-step attack chains

EXAM FORMAT:
  → 2 web applications to exploit
  → White-box: Full source code provided
  → Must achieve specific objectives
  → Authentication bypass required
  → Remote Code Execution required
  → 48 hours for exploitation
  → 24 hours for report writing
  → Professional report required

KEY SKILLS NEEDED:
  → Strong programming ability
  → Code review skills (PHP, Java, C#, Python)
  → Deep web vulnerability knowledge
  → Debugging skills
  → Ability to chain vulnerabilities
  → Custom exploit writing
  → Understanding of frameworks
  → Database interaction knowledge

WHO SHOULD GET OSWE:
  → Web application security specialists
  → Penetration testers focusing on web
  → Application security engineers
  → Security consultants
  → Salary impact: $10,000-$20,000+ premium
```

---

## 3. OSEP (Experienced Pen Tester)

```
OSEP DETAILS:

  Course:          PEN-300 (Evasion Techniques
                   and Breaching Defenses)
  Exam Duration:   48 hours (hacking)
                   + 24 hours (report)
  Passing:         Must achieve specific objectives
  Format:          Multi-domain enterprise attack
  Cost:            $1,749 (90-day lab + 1 attempt)
  Prerequisite:    OSCP required

COURSE CONTENT:
  → Antivirus evasion techniques
  → Application whitelisting bypass
  → Process injection and hollowing
  → Custom C# and PowerShell tooling
  → Advanced Active Directory attacks
  → MSSQL exploitation
  → Linux lateral movement
  → Domain escalation techniques
  → Kerberos advanced attacks
  → NTLM relay attacks
  → Advanced phishing techniques
  → Code signing and obfuscation

EXAM FORMAT:
  → Enterprise network simulation
  → Multiple machines and domains
  → Must evade defenses (AV, EDR)
  → Chain multiple attack techniques
  → Reach specific objectives
  → 48 hours for exploitation
  → 24 hours for report
  → More realistic than OSCP

KEY SKILLS NEEDED:
  → OSCP-level skills as baseline
  → C# programming
  → PowerShell scripting
  → Windows API understanding
  → Active Directory expertise
  → AV/EDR evasion techniques
  → Custom tool development
  → Advanced privilege escalation

WHO SHOULD GET OSEP:
  → Senior penetration testers
  → Red team operators
  → Offensive security researchers
  → Those doing AD-heavy assessments
  → Salary impact: $15,000-$25,000+ premium
```

---

## 4. Other Offensive Certifications

```
ADDITIONAL OFFENSIVE CERTS:

OSED (EXP-301):
  → Windows User Mode Exploit Development
  → Reverse engineering
  → Custom shellcode
  → ROP chain development
  → DEP/ASLR bypass
  → Format string vulnerabilities
  → 48-hour practical exam
  → For exploit development specialists

OSEE (EXP-401):
  → Advanced Windows Exploitation
  → Most difficult OffSec cert
  → Kernel-mode exploitation
  → Advanced bypass techniques
  → In-person training only
  → 72-hour exam
  → Very few holders worldwide

CRTO (Certified Red Team Operator):
  → From Zero-Point Security
  → Cobalt Strike focused
  → Red team operations
  → Active Directory attacks
  → 48-hour practical exam
  → Cost: ~$500 (course + exam)
  → More affordable alternative

CRTP (Certified Red Team Professional):
  → From Pentester Academy
  → Active Directory attacks
  → PowerShell-based
  → 24-hour practical exam
  → Cost: ~$300-500
  → Good AD attack cert

PNPT (Practical Network Penetration Tester):
  → From TCM Security
  → Practical exam format
  → Includes OSINT, scanning, exploitation
  → 5-day exam period
  → Cost: ~$400 (training + exam)
  → Good OSCP alternative/prep

eCPPTv2 (eLearnSecurity Certified Professional):
  → From INE/eLearnSecurity
  → Practical pen testing exam
  → 14-day exam period
  → Report writing required
  → Cost: ~$400 (exam)
  → Good stepping stone to OSCP

COMPARISON:
  Cert  | Difficulty | Cost   | Format      | Focus
  OSCP  | Hard       | $1,749 | 24h hands-on| General
  OSWE  | Very Hard  | $1,749 | 48h hands-on| Web
  OSEP  | Very Hard  | $1,749 | 48h hands-on| Evasion/AD
  CRTO  | Medium-Hard| ~$500  | 48h hands-on| Red Team
  PNPT  | Medium     | ~$400  | 5d hands-on | General
```

---

## 5. Preparation Strategies

```
PREPARING FOR ADVANCED OFFENSIVE CERTS:

FOR OSWE:
  Pre-Study (before course):
  → Learn PHP, Java, C# basics
  → Practice code review
  → Understand OWASP Top 10 deeply
  → Practice on PortSwigger Web Security Academy
  → Build web apps to understand vulnerabilities
  → Study: deserialization, SSTI, type juggling

  During Course:
  → Complete ALL exercises
  → Extra miles are essential
  → Build custom tools
  → Practice debugging applications
  → Set up vulnerable apps locally

  Practice Platforms:
  → PortSwigger Academy (free)
  → HTB web challenges
  → PentesterLab Pro
  → DVWA, bWAPP, WebGoat

FOR OSEP:
  Pre-Study (before course):
  → Master Active Directory attacks
  → Learn C# programming
  → Study Windows internals
  → Practice PowerShell scripting
  → Understand AV detection methods
  → Study process injection techniques

  During Course:
  → Complete ALL exercises + extra miles
  → Build custom tools
  → Practice evasion techniques
  → Set up defended AD lab
  → Test against Windows Defender

  Practice Platforms:
  → Snap Labs
  → HTB Pro Labs (Offshore, RastaLabs)
  → HackTheBox Enterprise machines
  → Custom AD lab with Defender enabled

GENERAL TIPS:
  → Don't rush — these certs need preparation
  → The course exercises ARE the preparation
  → Build your own tools, don't rely on public ones
  → Document everything for the report
  → Time management is critical (48h exam)
  → Take breaks during the exam
  → The report matters — can pass/fail you
```

---

## Summary Table

| Certification | Focus | Exam Duration | Difficulty |
|--------------|-------|--------------|-----------|
| OSWE | Web app white-box | 48h + 24h report | Very Hard |
| OSEP | Evasion + AD | 48h + 24h report | Very Hard |
| OSED | Exploit development | 48h + 24h report | Expert |
| OSEE | Advanced exploitation | 72h | Extreme |
| CRTO | Red team ops | 48h | Hard |
| PNPT | Network pen testing | 5 days | Medium |

---

## Revision Questions

1. What is the OffSec certification hierarchy from entry to expert?
2. How does the OSWE exam differ from the OSCP?
3. What programming skills are needed for OSEP?
4. What are more affordable alternatives to OffSec certifications?
5. How should you prepare for white-box web application testing (OSWE)?

---

*Previous: [01-giac-certifications.md](01-giac-certifications.md) | Next: [03-blue-team-certifications.md](03-blue-team-certifications.md)*

---

*[Back to README](../README.md)*
