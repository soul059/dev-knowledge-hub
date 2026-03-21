# Unit 4: OSCP Preparation — Topic 2: Lab Strategy

## Overview

The PEN-200 lab environment is where you develop the practical skills needed for the OSCP exam. This topic covers effective strategies for maximizing your lab time, approaching machines systematically, and building the muscle memory needed for exam day.

---

## 1. Lab Environment Overview

```
PEN-200 LAB NETWORK:

  ┌─────────────────────────────────────┐
  │  STUDENT VPN → Public Network       │
  │      │                              │
  │  [Student Subnet]                   │
  │      │                              │
  │  [Public Network] (~25+ machines)   │
  │      │                              │
  │  [Dev Network] (pivot required)     │
  │      │                              │
  │  [IT Network] (deeper pivot)        │
  │      │                              │
  │  [Admin Network] (hardest)          │
  │      │                              │
  │  [AD Set] (domain environment)      │
  └─────────────────────────────────────┘

MACHINE CATEGORIES:
  → Easy: Similar to TryHackMe easy rooms
  → Medium: Standard OSCP-level
  → Hard: Above exam difficulty
  → AD machines: Domain-joined targets
  → Dependencies: Some require others first

LAB ACCESS OPTIONS:
  Plan      | Duration | Exam Attempts | Price
  Learn One | 90 days  | 1             | $1,749
  Learn     | 365 days | 2             | $2,499

RECOMMENDATION: 
  → 365-day plan (more time + retry)
  → Supplement with external platforms
  → Aim to complete 40-50 lab machines
```

---

## 2. Approaching Lab Machines

```
MACHINE ATTACK METHODOLOGY:

STEP 1: ENUMERATION (60-70% of time)
  # Full TCP scan
  nmap -sC -sV -p- -oA full_tcp target
  
  # UDP top ports
  nmap -sU --top-ports 20 -oA udp target
  
  # Web enumeration
  gobuster dir -u http://target -w /usr/share/
  wordlists/dirb/common.txt -o gobuster.txt
  
  # SMB enumeration
  smbclient -L //target -N
  enum4linux -a target
  
  # Always check: HTTP, SMB, FTP, SSH

STEP 2: VULNERABILITY IDENTIFICATION
  → Google: "service version exploit"
  → searchsploit service_name version
  → Check default credentials
  → Read web app source code
  → Check robots.txt, sitemap
  → Look for file upload functionality
  → Test for injection (SQLi, LFI, RFI)

STEP 3: EXPLOITATION
  → Try simplest exploit first
  → Modify public exploits as needed
  → Verify exploit target requirements
  → Set up listeners before exploiting
  → Capture initial shell

STEP 4: POST-EXPLOITATION
  → Stabilize shell
  → Enumerate local system
  → Find privilege escalation vector
  → Escalate to root/SYSTEM
  → Capture proof.txt

STEP 5: DOCUMENTATION
  → Screenshot every step
  → Record exact commands
  → Note failed attempts too
  → Write up methodology
```

---

## 3. Time Management in Labs

```
LAB TIME ALLOCATION:

90-DAY PLAN:
  Week 1-2:   Complete course exercises
  Week 3-4:   Easy lab machines (10-15)
  Week 5-8:   Medium machines (15-20)
  Week 9-10:  Hard machines + AD set
  Week 11-12: Review + practice exams
  Week 13:    Schedule and take exam

365-DAY PLAN:
  Month 1-2:   Course exercises
  Month 3-4:   Easy machines + practice
  Month 5-7:   Medium machines + pivoting
  Month 8-9:   Hard machines + AD
  Month 10-11: External practice (HTB/PG)
  Month 12:    Review and exam

PER-MACHINE TIME LIMITS:
  Easy:     2-4 hours
  Medium:   4-8 hours
  Hard:     8-16 hours
  
  RULES:
  → Set a timer for each machine
  → Stuck for 2+ hours? Take a break
  → Stuck for 4+ hours? Look at hints
  → Never spend more than a full day
  → Document what you tried
  → Return with fresh eyes later

WHEN TO USE HINTS:
  → After genuinely trying (2+ hours)
  → You've exhausted enumeration
  → You're learning, not just solving
  → Understand the hint, don't just copy
  → Hints: OffSec forums, community
```

---

## 4. Practice Platform Strategy

```
SUPPLEMENTARY PLATFORMS:

PROVING GROUNDS (OffSec):
  → Closest to OSCP difficulty
  → Play (free) + Practice ($19/month)
  → Community-rated difficulty
  → Walkthroughs available (Practice)
  → PRIORITY: Do these machines

  Recommended PG Practice:
  → Warm Up machines (easy)
  → Get To Work machines (medium)
  → Try Harder machines (hard)
  → Do 30-40 PG machines total

HACK THE BOX:
  → TJ Null's OSCP-like machine list
  → Active machines (no walkthroughs)
  → Retired machines (VIP for walkthroughs)
  → Very exam-relevant

  Recommended HTB Machines (TJ Null list):
  → Easy: Lame, Legacy, Blue, Devel
  → Medium: Poison, Nineveh, Cronos
  → Harder: Chatterbox, Conceal

TRYHACKME:
  → OSCP learning path
  → Guided rooms (good for fundamentals)
  → Buffer Overflow room
  → Active Directory rooms
  → Windows/Linux PrivEsc rooms

VULNHUB:
  → Downloadable VMs
  → Practice offline
  → DC series, Kioptrix, Mr-Robot

PRACTICE EXAM SIMULATION:
  → Pick 5 machines you haven't done
  → Set 24-hour timer
  → No hints or walkthroughs
  → Document everything
  → Write a report after
  → Repeat 2-3 times before real exam
```

---

## 5. Key Skills to Master

```
CRITICAL OSCP SKILLS:

ENUMERATION:
  [ ] Full Nmap scans (TCP + UDP)
  [ ] Web directory brute forcing
  [ ] SMB/NFS/FTP enumeration
  [ ] DNS enumeration and zone transfers
  [ ] Source code review
  [ ] Finding hidden parameters

WEB ATTACKS:
  [ ] SQL injection (manual)
  [ ] Local File Inclusion (LFI)
  [ ] Remote File Inclusion (RFI)
  [ ] Command injection
  [ ] File upload bypass
  [ ] Deserialization attacks

EXPLOITATION:
  [ ] Finding and modifying public exploits
  [ ] Reverse shell generation
  [ ] Shell stabilization
  [ ] Payload generation (msfvenom)
  [ ] Buffer overflow (basic)

PRIVILEGE ESCALATION:
  [ ] Linux: SUID, sudo, cron, kernel
  [ ] Windows: Services, tokens, UAC bypass
  [ ] Automated tools: linpeas, winpeas
  [ ] Manual enumeration techniques

ACTIVE DIRECTORY:
  [ ] AS-REP roasting
  [ ] Kerberoasting
  [ ] Pass the hash
  [ ] BloodHound analysis
  [ ] Lateral movement (psexec, wmiexec)
  [ ] DCSync

PIVOTING:
  [ ] SSH tunneling
  [ ] Chisel (SOCKS proxy)
  [ ] Ligolo-ng
  [ ] ProxyChains
  [ ] Port forwarding

MUST-KNOW TOOLS:
  → Nmap, Gobuster/Feroxbuster
  → Burp Suite Community
  → Netcat/Socat
  → Chisel/Ligolo
  → LinPEAS/WinPEAS
  → BloodHound
  → Impacket suite
  → CrackMapExec
  → John/Hashcat
  → msfvenom (payload gen)
```

---

## Summary Table

| Strategy Area | Recommendation |
|--------------|---------------|
| Lab Time | 365-day plan recommended |
| Machine Target | 40-50 lab + 30-40 PG/HTB |
| Time per Machine | 2-8 hours depending on difficulty |
| Practice Exams | 2-3 simulated 24h exams |
| Key Focus | Enumeration, PrivEsc, AD |
| External Platforms | Proving Grounds priority |

---

## Revision Questions

1. What percentage of time should be spent on enumeration vs exploitation?
2. When is it appropriate to use hints during lab practice?
3. What external platforms best simulate OSCP difficulty?
4. What are the critical AD attack techniques for the exam?
5. How should you structure a practice exam simulation?

---

*Previous: [01-exam-format.md](01-exam-format.md) | Next: [03-methodology-development.md](03-methodology-development.md)*

---

*[Back to README](../README.md)*
