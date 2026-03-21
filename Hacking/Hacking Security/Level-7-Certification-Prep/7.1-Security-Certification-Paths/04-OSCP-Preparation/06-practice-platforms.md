# Unit 4: OSCP Preparation — Topic 6: Practice Platforms (HTB, THM, PG)

## Overview

Supplementing PEN-200 labs with external practice platforms is essential for OSCP success. This topic covers the best platforms, recommended machines, and how to structure your practice for maximum exam readiness.

---

## 1. Proving Grounds (OffSec)

```
PROVING GROUNDS:

OVERVIEW:
  → Created by Offensive Security
  → Closest to OSCP exam difficulty
  → Two tiers: Play (free) + Practice (paid)
  → Community-rated difficulty levels
  → Direct relevance to exam

PG PLAY (FREE):
  → Limited machine selection
  → Good for getting started
  → Community walkthroughs available
  → No hints built-in
  → Good warm-up machines

PG PRACTICE ($19/month):
  → Full machine library (100+)
  → Official walkthroughs included
  → Difficulty ratings
  → OSCP-like machines flagged
  → HIGHLY recommended investment

RECOMMENDED PG PRACTICE ORDER:
  1. Warm Up (Easy):
     → Nibbles, ClamAV, Bratarina
     → Twiggy, Muddy, DC-1
  
  2. Get To Work (Medium):
     → Helpdesk, Heist, Meathead
     → Internal, Vault, Resourced
     → Hutch, Sorcerer, Astronaut
  
  3. Try Harder (Hard):
     → Craft, DVR4, Extplorer
     → Jacko, Medjed, Billyboss

DIFFICULTY MAPPING TO OSCP:
  PG Easy = OSCP Easy standalone
  PG Medium = OSCP Medium standalone
  PG Hard = Above OSCP difficulty
  PG AD = OSCP AD set practice
```

---

## 2. Hack The Box (HTB)

```
HACK THE BOX:

OVERVIEW:
  → Premier platform for pentesting practice
  → Active (no walkthroughs) + Retired (VIP)
  → OSCP-like machines available
  → Strong community
  → VIP: $14/month

TJ NULL'S OSCP LIST (HTB):
  Essential list of OSCP-preparation machines

  LINUX (Easy-Medium):
  → Lame, Legacy, Devel
  → Beep, Optimum, Bastard
  → Cronos, Nibbles, Sense
  → Solidstate, Valentine, Poison
  → Sunday, Tartarsauce, Irked
  → Friendzone, Networked, Jarvis

  WINDOWS (Easy-Medium):
  → Blue, Jerry, Netmon
  → Active, Forest, Sauna
  → Cascade, ServMon, Buff
  → Resolute, Monteverde
  → Remote, Fuse, Tabby

  AD MACHINES:
  → Active, Forest, Sauna
  → Cascade, Resolute, Monteverde
  → Support, Intelligence

PRACTICE APPROACH:
  1. Try machine without hints (4-hour limit)
  2. If stuck, check one hint at a time
  3. After completing, read full writeup
  4. Note new techniques learned
  5. Track your solve time
  6. Revisit machines you struggled with
```

---

## 3. TryHackMe (THM)

```
TRYHACKME:

OVERVIEW:
  → Guided learning platform
  → Structured rooms and paths
  → Best for building fundamentals
  → Free + Premium ($14/month)
  → Easier than HTB generally

OSCP-RELEVANT PATHS:
  1. "Offensive Pentesting" path
     → Includes: Vulnversity, Kenobi
     → Steel Mountain, Alfred
     → HackPark, GameZone
     → Skynet, Daily Bugle
     → Overpass, Relevant

  2. Key Individual Rooms:
     → Buffer Overflow Prep
     → Linux PrivEsc (TCM)
     → Windows PrivEsc (TCM)
     → Active Directory Basics
     → Attacking Kerberos
     → Wreath (network pivoting)
     → Holo (AD exploitation)

  3. Skill-Specific:
     → Nmap: "Nmap" room
     → Web: "OWASP Top 10" rooms
     → Shells: "What the Shell?"
     → SQL: "SQL Injection" room
     → PrivEsc: TCM rooms

USE THM FOR:
  → Building foundational skills
  → Learning new techniques
  → Guided practice
  → NOT final exam prep (too guided)
  → Transition to HTB/PG after basics
```

---

## 4. Other Practice Resources

```
ADDITIONAL PLATFORMS:

VULNHUB:
  → Free downloadable VMs
  → Run locally in VirtualBox/VMware
  → No internet required
  → Variable difficulty
  
  Recommended VMs:
  → Kioptrix series (1-5)
  → DC series (1-9)
  → Mr-Robot
  → SickOs 1.2
  → Stapler
  → PwnLab
  → HackLab Vulnix

HTB ACADEMY:
  → Structured learning modules
  → Tier system (easy to hard)
  → Covers OSCP topics systematically
  → Some free, some paid ($18/month)
  
  Key Modules:
  → Penetration Testing Process
  → Active Directory Enumeration
  → File Transfers
  → Pivoting and Tunneling
  → Windows Privilege Escalation
  → Linux Privilege Escalation
  → Web Attacks

PENTESTERLAB:
  → Web application focused
  → Step-by-step exercises
  → Pro subscription ($35/month)
  → Excellent for web vulns

IPPSEC VIDEOS (YouTube):
  → HTB machine walkthroughs
  → Learn methodology by watching
  → Search by technique
  → Excellent explanations
  → Watch AFTER attempting machine

CTF PLATFORMS (supplementary):
  → PicoCTF: Beginner-friendly
  → OverTheWire: Linux skills
  → CryptoHack: Crypto challenges
  → DVWA: Web app practice
```

---

## 5. Practice Plan and Progress Tracking

```
PRACTICE MACHINE TARGETS:

MINIMUM BEFORE EXAM:
  → PEN-200 Labs: 40+ machines
  → PG Practice: 30+ machines
  → HTB (TJ Null): 20+ machines
  → THM: Complete PrivEsc + AD rooms
  → Total: 100+ unique machines

TRACKING SPREADSHEET:
  Machine | Platform | OS    | Difficulty | Solved | Time
  Lame    | HTB      | Linux | Easy       | Yes    | 1.5h
  Active  | HTB      | Win   | Easy       | Yes    | 2h
  Nibbles | PG       | Linux | Warm Up    | Yes    | 45m
  ...

WEEKLY PRACTICE SCHEDULE:
  Mon: 1 PG machine + review
  Tue: 1 HTB machine + writeup review
  Wed: THM room (specific skill)
  Thu: 1 PG machine + review
  Fri: 1 HTB machine + writeup review
  Sat: 2-3 machines (practice sprint)
  Sun: Review notes, update cheat sheet

READINESS INDICATORS:
  [ ] Can solve easy machines in <2 hours
  [ ] Can solve medium machines in <4 hours
  [ ] Completed AD set practice (3+ times)
  [ ] PrivEsc vectors feel natural
  [ ] Enumeration is thorough and systematic
  [ ] Report writing is efficient
  [ ] Done 2+ simulated 24h exams
  [ ] Consistent methodology applied
  [ ] Tool usage is muscle memory
  [ ] Comfortable with pivoting

SIMULATION EXAM:
  → Pick 5 unseen machines (3 standalone + 2 AD)
  → Set 24-hour timer
  → No hints, no walkthroughs
  → Full documentation
  → Write complete report
  → Score yourself (20 pts per standalone,
    40 pts for AD set)
  → Did you pass? If yes: schedule real exam
  → If no: identify weak areas, practice more
```

---

## Summary Table

| Platform | Best For | Cost | OSCP Relevance |
|----------|---------|------|----------------|
| PG Practice | Exam-like machines | $19/mo | Highest |
| HTB | Realistic challenges | $14/mo (VIP) | Very High |
| TryHackMe | Fundamentals | $14/mo | Medium |
| VulnHub | Offline practice | Free | Medium |
| HTB Academy | Structured learning | $18/mo | High |

---

## Revision Questions

1. Why is Proving Grounds Practice considered the best OSCP prep platform?
2. What is TJ Null's OSCP list and where do you find it?
3. How many total machines should you complete before attempting OSCP?
4. What readiness indicators suggest you should schedule the exam?
5. How should you structure a simulated OSCP exam?

---

*Previous: [05-time-management.md](05-time-management.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
