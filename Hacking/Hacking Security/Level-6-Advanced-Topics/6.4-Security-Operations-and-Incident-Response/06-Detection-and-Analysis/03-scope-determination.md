# Unit 6: Detection and Analysis — Topic 3: Scope Determination

## Overview

**Scope determination** identifies the full extent of a security incident — all compromised systems, accounts, data, and attack pathways. Incomplete scoping leads to ineffective containment and recurring compromises. Accurate scope determination is one of the most critical and challenging aspects of incident response.

---

## 1. Scoping Methodology

```
SCOPING PROCESS:

  ┌────────────────────────────────────────────┐
  │           SCOPE DETERMINATION              │
  │                                            │
  │  Known Compromise                          │
  │  (Patient Zero)                            │
  │       │                                    │
  │  ┌────▼──────────────────────────────────┐ │
  │  │ LATERAL MOVEMENT ANALYSIS             │ │
  │  │ What systems did attacker access?     │ │
  │  └────┬──────────────────────────────────┘ │
  │       │                                    │
  │  ┌────▼──────────────────────────────────┐ │
  │  │ CREDENTIAL ANALYSIS                   │ │
  │  │ What credentials were compromised?    │ │
  │  └────┬──────────────────────────────────┘ │
  │       │                                    │
  │  ┌────▼──────────────────────────────────┐ │
  │  │ DATA ACCESS ANALYSIS                  │ │
  │  │ What data was accessed/exfiltrated?   │ │
  │  └────┬──────────────────────────────────┘ │
  │       │                                    │
  │  ┌────▼──────────────────────────────────┐ │
  │  │ PERSISTENCE ANALYSIS                  │ │
  │  │ Did attacker establish persistence?   │ │
  │  └────┬──────────────────────────────────┘ │
  │       │                                    │
  │  ┌────▼──────────────────────────────────┐ │
  │  │ INFRASTRUCTURE ANALYSIS               │ │
  │  │ What attacker infrastructure is used? │ │
  │  └──────────────────────────────────────┘ │
  └────────────────────────────────────────────┘

SCOPING QUESTIONS:
  → How did the attacker get in? (Initial access)
  → What systems were compromised?
  → What accounts were used/compromised?
  → What data was accessed or stolen?
  → Did the attacker establish persistence?
  → What tools did the attacker use?
  → How long has the attacker been present? (Dwell time)
  → What is the attacker's current status? (Active/contained)
```

---

## 2. Scoping Techniques

```
INVESTIGATION TECHNIQUES:

LATERAL MOVEMENT TRACKING:
  → Authentication log analysis
  → RDP connection logs
  → SMB access logs
  → WMI/WinRM activity
  → PsExec/service creation
  → SSH connections (Linux)

  # Track authentication from compromised host
  index=windows EventCode=4624 src_ip="10.0.1.50"
  | stats count by dest_host, LogonType, user
  | sort -count

  # Track admin tool usage
  index=sysmon EventCode=1 
  (Image="*psexec*" OR Image="*wmic*" OR 
   Image="*winrm*")
  | table _time, ComputerName, User, Image, CommandLine

CREDENTIAL COMPROMISE SCOPE:
  → Which credentials were on compromised systems?
  → Were any credentials cached/dumped?
  → What access do those credentials have?
  → Were credential dumping tools used?
  → Check for pass-the-hash indicators

  # LSASS access (credential dumping indicator)
  index=sysmon EventCode=10 TargetImage="*lsass.exe"
  | table _time, ComputerName, SourceImage, GrantedAccess

DATA ACCESS SCOPE:
  → File server access logs
  → Database query logs
  → Email access logs
  → Cloud storage access
  → DLP alerts
  → Data transfer volumes

PERSISTENCE SCOPE:
  Check for:
  → New scheduled tasks
  → New services
  → Registry run keys
  → Startup folder modifications
  → Web shells
  → Backdoor accounts
  → SSH authorized_keys
  → Cron jobs (Linux)

IOC-BASED SCOPING:
  → Identify all IOCs from known compromise
  → Search entire environment for those IOCs
  → C2 IPs/domains in firewall/proxy logs
  → Malware hashes across all endpoints
  → Attacker tools across all systems
```

---

## 3. Scope Tracking

```
SCOPE TRACKING DOCUMENT:

COMPROMISED SYSTEMS:
  System       | Evidence           | Status    | Priority
  JSMITH-PC    | Malware executed   | Isolated  | High
  FILESVR01    | Lateral movement   | Isolated  | Critical
  DC01         | Credential dump    | Monitoring| Critical
  WEBSVR02     | Web shell found    | Isolated  | High
  MAILSVR01    | Email accessed     | Monitoring| Medium

COMPROMISED ACCOUNTS:
  Account      | Type    | Evidence           | Status
  jsmith       | User    | Brute force + login| Disabled
  admin        | Admin   | PTH detected       | Reset
  svc_backup   | Service | Token stolen       | Reset
  domain\admin | Domain  | NTDS dump suspected| Reset

DATA ACCESSED:
  Data Type    | Location    | Volume    | Sensitivity
  Customer PII | FileShare01 | 2.3 GB    | High
  Financial    | SQLSVR01    | Unknown   | Critical
  Email        | Exchange    | 500 emails| Medium
  Source code  | GitLab      | None found| N/A

ATTACKER INFRASTRUCTURE:
  Type    | Value          | First Seen | Status
  C2 IP   | 203.0.113.50  | Jan 10     | Blocked
  C2 IP   | 198.51.100.25 | Jan 12     | Blocked
  Domain  | evil.example.com| Jan 10    | Sinkholed
  Tool    | Cobalt Strike  | Jan 11    | Hash blocked
```

---

## 4. Scope Expansion Triggers

```
WHEN TO EXPAND SCOPE:

  → New compromised system discovered
  → Additional credentials found compromised
  → New C2 infrastructure identified
  → Persistence mechanism found
  → Data exfiltration confirmed
  → Attacker activity in new network segment
  → New malware variant discovered
  → Insider involvement suspected

SCOPE MANAGEMENT:
  → Track scope changes over time
  → Document reason for each expansion
  → Re-prioritize based on new scope
  → Adjust resources as scope grows
  → Communicate scope changes to stakeholders
  → Update severity if warranted

SCOPE DIAGRAM:
  Day 1:     [JSMITH-PC]
  Day 2:     [JSMITH-PC] → [FILESVR01]
  Day 3:     [JSMITH-PC] → [FILESVR01] → [DC01]
                          → [WEBSVR02]
  Day 4:     [JSMITH-PC] → [FILESVR01] → [DC01]
                          → [WEBSVR02]
                          → [MAILSVR01]
  
  Scope grew from 1 system to 5 systems over 4 days
  Severity escalated from HIGH to CRITICAL on Day 3
```

---

## 5. Scoping Best Practices

```
BEST PRACTICES:

  [ ] Start with known compromise (patient zero)
  [ ] Work outward from initial point
  [ ] Track all credentials on compromised systems
  [ ] Search environment-wide for all IOCs
  [ ] Don't assume scope — verify with evidence
  [ ] Check for persistence on every compromised system
  [ ] Assess data access for every compromised account
  [ ] Update scope document in real-time
  [ ] Communicate scope changes to IR team
  [ ] Consider worst case when uncertain

COMMON MISTAKES:
  ✗ Declaring scope too early (before full investigation)
  ✗ Only looking at initially detected system
  ✗ Not tracking credential chains
  ✗ Missing persistence mechanisms
  ✗ Ignoring cloud/SaaS in scope
  ✗ Not searching for attacker across environment
  ✗ Underestimating dwell time
  ✗ Not checking for data exfiltration

SCOPE COMPLETION CRITERIA:
  → All IOCs searched across environment
  → All lateral movement paths traced
  → All compromised credentials identified
  → All persistence mechanisms found
  → Data access fully assessed
  → Attacker infrastructure fully mapped
  → No new scope-expanding findings for 48+ hours
```

---

## Summary Table

| Scoping Area | Key Investigation | Data Sources |
|-------------|------------------|-------------|
| Lateral Movement | Auth logs, RDP, SMB | SIEM, AD logs |
| Credentials | Credential dumps, PTH | Sysmon, EDR |
| Data Access | File/email/DB access | DLP, audit logs |
| Persistence | Tasks, services, shells | Sysmon, EDR |
| Infrastructure | C2 IPs, domains, tools | Firewall, proxy |

---

## Revision Questions

1. What are the key areas to investigate during scope determination?
2. How do you track lateral movement during an incident?
3. What triggers should cause scope expansion?
4. What should a scope tracking document include?
5. How do you know when scoping is complete?

---

*Previous: [02-initial-analysis.md](02-initial-analysis.md) | Next: [04-evidence-collection.md](04-evidence-collection.md)*

---

*[Back to README](../README.md)*
