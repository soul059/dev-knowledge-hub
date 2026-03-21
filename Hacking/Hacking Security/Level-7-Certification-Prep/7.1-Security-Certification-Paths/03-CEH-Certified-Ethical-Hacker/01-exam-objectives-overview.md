# Unit 3: CEH (Certified Ethical Hacker) — Topic 1: Exam Objectives Overview

## Overview

The **EC-Council Certified Ethical Hacker (CEH v12)** is one of the most recognized offensive security certifications. This topic covers the exam structure, domains, question types, and preparation approach for successfully earning the CEH credential.

---

## 1. CEH Exam Specifications

```
CEH v12 EXAM DETAILS:

  Exam Code:       312-50v12
  Questions:       125
  Question Types:  Multiple choice
  Duration:        4 hours (240 minutes)
  Passing Score:   60-85% (varies by exam form)
  Cost:            $1,199 USD (exam only)
                   $2,199+ (with training)
  Testing:         Pearson VUE or ECC Exam Center
  Prerequisites:   2 years IT security experience
                   OR EC-Council official training
  Valid:           3 years (renew with ECE credits)

EXAM FORMAT:
  → 125 multiple choice questions
  → No performance-based questions
  → 4 hours — plenty of time (~1.9 min/question)
  → Scenario-based questions
  → Tool output interpretation
  → Attack methodology questions
  → Legal and ethical questions

CEH PRACTICAL:
  → Separate exam (optional but valuable)
  → 6-hour hands-on practical
  → 20 challenges in live environment
  → Real tools, real scenarios
  → Proves practical capability
  → Highly recommended for credibility
```

---

## 2. CEH Domains and Weights

```
CEH v12 DOMAINS:

  Domain                              | Weight | Questions
  1. Information Security & Ethical   | 6%     | ~8
     Hacking
  2. Reconnaissance Techniques        | 21%    | ~26
  3. System Hacking Phases &          | 17%    | ~21
     Attack Techniques
  4. Network & Perimeter Hacking      | 14%    | ~18
  5. Web Application Hacking          | 16%    | ~20
  6. Wireless Network Hacking         | 6%     | ~8
  7. Mobile Platform, IoT, OT Hacking | 8%     | ~10
  8. Cloud Computing                  | 6%     | ~8
  9. Cryptography                     | 6%     | ~7

WEIGHT VISUALIZATION:
  D1 [███░░░░░░░░░░░░░░░░░]  6%
  D2 [██████████░░░░░░░░░░] 21%  ← Heaviest
  D3 [████████░░░░░░░░░░░░] 17%
  D4 [███████░░░░░░░░░░░░░] 14%
  D5 [████████░░░░░░░░░░░░] 16%
  D6 [███░░░░░░░░░░░░░░░░░]  6%
  D7 [████░░░░░░░░░░░░░░░░]  8%
  D8 [███░░░░░░░░░░░░░░░░░]  6%
  D9 [███░░░░░░░░░░░░░░░░░]  6%

FOCUS AREAS (by weight):
  → Reconnaissance (21%) — Master this domain
  → System Hacking (17%) — Second priority
  → Web App Hacking (16%) — Third priority
  → Network Hacking (14%) — Fourth priority
  → These 4 domains = 68% of the exam
```

---

## 3. CEH Methodology

```
CEH HACKING METHODOLOGY:

  Phase 1: Reconnaissance
  → Passive: OSINT, social media, DNS
  → Active: Scanning, enumeration
  
  Phase 2: Scanning
  → Network scanning (Nmap)
  → Vulnerability scanning (Nessus)
  → Port scanning techniques
  
  Phase 3: Gaining Access
  → Exploitation techniques
  → Password cracking
  → Social engineering
  
  Phase 4: Maintaining Access
  → Backdoors and trojans
  → Rootkits
  → Persistence mechanisms
  
  Phase 5: Covering Tracks
  → Log clearing
  → Timestomping
  → Anti-forensics

ETHICAL HACKING PRINCIPLES:
  → Always get written authorization
  → Define scope clearly
  → Report all findings
  → Protect client data
  → Follow laws and regulations
  → Do no harm beyond scope
  → Maintain confidentiality

LEGAL FRAMEWORKS:
  → CFAA (Computer Fraud and Abuse Act)
  → GDPR implications
  → PCI-DSS requirements
  → HIPAA considerations
  → Local jurisdiction laws
  → Rules of engagement
```

---

## 4. Key Tools to Know

```
CEH TOOL CATEGORIES:

RECONNAISSANCE:
  → Maltego: Visual link analysis
  → theHarvester: Email/domain recon
  → Recon-ng: Web reconnaissance
  → Shodan: IoT/device search
  → Censys: Internet-wide scanning
  → Google Dorking: Advanced search

SCANNING:
  → Nmap: Port/service scanning
  → Hping3: Packet crafting
  → Netcat: Network utility
  → Nessus: Vulnerability scanning
  → OpenVAS: Open-source scanning

EXPLOITATION:
  → Metasploit: Exploitation framework
  → SQLmap: SQL injection automation
  → Burp Suite: Web app testing
  → Hydra: Password brute forcing
  → John the Ripper: Password cracking
  → Hashcat: GPU password cracking

WIRELESS:
  → Aircrack-ng: WiFi cracking suite
  → Kismet: Wireless detector
  → Wifite: Automated WiFi attacks

SNIFFING:
  → Wireshark: Packet analysis
  → tcpdump: CLI packet capture
  → Ettercap: MitM attacks
  → Bettercap: Advanced MitM

EXAM TIP: Know what each tool does,
which category it belongs to, and 
basic command syntax.
```

---

## 5. CEH vs Other Certifications

```
CEH COMPARISON:

  Aspect      | CEH         | Security+   | OSCP
  Focus       | Knowledge   | Concepts    | Practical
  Questions   | 125 MCQ     | 90 MCQ+PBQ  | Hands-on
  Duration    | 4 hours     | 90 min      | 24 hours
  Difficulty  | Medium      | Entry       | Hard
  Cost        | $1,199+     | $404        | $1,749
  Prerequisite| 2yr or train| None        | None
  Renewal     | 3 years/ECE | 3 years/CEU | No expiry
  Value       | HR/compliance| Entry-level | Practical skill

WHY GET CEH:
  → Required for many government positions
  → DoD 8570/8140 baseline certification
  → Recognized worldwide
  → Broad coverage of hacking topics
  → Good stepping stone to OSCP

CEH CRITICISM:
  → MCQ-only (no hands-on on main exam)
  → Expensive with required training
  → Memorization-heavy
  → CEH Practical addresses some concerns
  → Some employers prefer OSCP

CAREER POSITIONS:
  → Penetration Tester
  → Security Analyst
  → SOC Analyst
  → Vulnerability Analyst
  → Security Consultant
  → Salary range: $65,000-$110,000
```

---

## Summary Table

| Aspect | Details |
|--------|---------|
| Exam Code | 312-50v12 |
| Questions | 125 MCQ |
| Duration | 4 hours |
| Pass Score | 60-85% (varies) |
| Top Domain | Reconnaissance (21%) |
| Key Tools | Nmap, Metasploit, Burp Suite |
| Renewal | 3 years / 120 ECE credits |

---

## Revision Questions

1. How many questions and how much time do you have for the CEH exam?
2. Which CEH domain has the highest exam weight?
3. What are the five phases of the CEH hacking methodology?
4. What prerequisite options exist for taking the CEH exam?
5. How does CEH compare to OSCP in terms of exam format?

---

*Previous: None (First topic in this unit) | Next: [02-reconnaissance-domains.md](02-reconnaissance-domains.md)*

---

*[Back to README](../README.md)*
