# Unit 10: Threat Hunting — Topic 4: Hunting Techniques

## Overview

**Hunting techniques** are the practical methods hunters use to search for threats in data. From simple IOC searching to advanced statistical analysis, these techniques form the hunter's analytical toolkit. Mastering multiple techniques enables hunters to detect threats that evade automated detection systems.

---

## 1. IOC-Based Hunting

```
IOC SEARCHING:

  Simplest form of hunting — search for
  known indicators of compromise.

IOC TYPES:
  Type              | Example
  IP Address        | 203.0.113.50
  Domain            | evil.example.com
  URL               | https://evil.com/payload
  File Hash (MD5)   | d41d8cd98f00b204e9800998
  File Hash (SHA256)| e3b0c44298fc1c149afb...
  File Name         | malware.exe
  Registry Key      | HKLM\...\Run\BadKey
  Email Address     | attacker@evil.com
  User Agent        | Mozilla/5.0 (Evil Bot)
  JA3 Hash          | a0e9f5d64349fb13191bc7...

IOC SEARCH QUERIES:
  # Search for C2 IP
  index=firewall dest_ip="203.0.113.50"
  | stats count by src_ip, dest_port, action

  # Search for malicious domain
  index=dns query="evil.example.com"
  | stats count by src_ip

  # Search for file hash across endpoints
  index=sysmon EventCode=1 
    Hashes="*SHA256=e3b0c44298fc1c*"
  | table _time, ComputerName, Image, CommandLine

  # Search for file hash in EDR
  # CrowdStrike: Event Search > SHA256
  # MDE Advanced Hunting:
  DeviceFileEvents
  | where SHA256 == "e3b0c44298fc1c..."

LIMITATIONS OF IOC HUNTING:
  → Only finds known threats
  → IOCs change frequently
  → High volume of IOCs to process
  → False positives from shared hosting
  → Doesn't find novel/unknown threats
  → Better automated than manual
```

---

## 2. TTP-Based Hunting

```
TTP (TACTICS, TECHNIQUES, PROCEDURES) HUNTING:

  More advanced — search for behaviors
  rather than specific indicators.

ADVANTAGES OVER IOC:
  → Detects unknown variants
  → More durable (TTPs change slowly)
  → Finds novel attacks using known methods
  → Identifies living-off-the-land attacks

HUNTING FOR COMMON TTPs:

PERSISTENCE — SCHEDULED TASKS (T1053.005):
  # Find new scheduled tasks
  index=sysmon EventCode=1 
    Image="*schtasks.exe"
    CommandLine="*/create*"
  | table _time, ComputerName, User, CommandLine
  | sort _time

  # Analyze: Are these expected?
  # Red flags: Created by unusual users,
  # pointing to temp/user directories,
  # running at odd intervals, encoded commands

CREDENTIAL ACCESS — LSASS ACCESS (T1003.001):
  # Detect processes accessing LSASS
  index=sysmon EventCode=10
    TargetImage="*lsass.exe"
    GrantedAccess IN ("0x1010","0x1410","0x1438",
    "0x143a","0x1fffff")
  | where NOT match(SourceImage, 
    "(?i)(csrss|svchost|winlogon|MsMpEng)")
  | table _time, ComputerName, SourceImage,
    GrantedAccess

  # Known legitimate: AV, LSASS itself, csrss
  # Suspicious: Unknown binaries, user-space tools

LATERAL MOVEMENT — REMOTE SERVICES (T1021):
  # Hunt for unusual RDP connections
  index=windows EventCode=4624 LogonType=10
  | stats count by TargetUserName, IpAddress,
    WorkstationName
  | where count < 3
  | sort count

  # Hunt for PsExec-style activity
  index=sysmon EventCode=1
    (ParentImage="*services.exe" AND
     (Image="*cmd.exe" OR Image="*powershell.exe"))
  | table _time, ComputerName, User, Image,
    CommandLine

EXECUTION — LIVING OFF THE LAND (T1218):
  # Hunt for LOLBin usage
  index=sysmon EventCode=1
    (Image="*mshta.exe" OR
     Image="*regsvr32.exe" OR
     Image="*rundll32.exe" OR
     Image="*certutil.exe" OR
     Image="*bitsadmin.exe")
  | table _time, ComputerName, User,
    ParentImage, Image, CommandLine
  | sort _time
```

---

## 3. Statistical and Anomaly Hunting

```
STATISTICAL HUNTING:

FREQUENCY ANALYSIS (Stacking):
  → Count occurrences of events
  → Look for rare/outlier events
  → "What is unusual compared to normal?"

  # Stack processes — find rare executables
  index=sysmon EventCode=1
  | stats count by Image
  | sort count
  | head 20
  # Rare processes = potentially suspicious

  # Stack network connections — find rare destinations
  index=firewall action=allowed
  | stats count by dest_ip
  | sort count
  | head 20

  # Stack user agents — find unusual browsers
  index=proxy
  | stats count by http_user_agent
  | sort count
  | head 20

LONG-TAIL ANALYSIS:
  Most data follows a pattern where a few
  items are very common and many items are
  rare. Threats often hide in the "long tail."

  Common (Top):   svchost.exe    (100,000 events)
                  chrome.exe     (50,000 events)
                  explorer.exe   (30,000 events)
  ...
  Long Tail:      unusual.exe    (1 event) ← HUNT HERE
                  payload.dll    (2 events) ← HUNT HERE

BEACONING DETECTION:
  → C2 communicates at regular intervals
  → Detect by analyzing connection timing

  # Detect beaconing
  index=proxy
  | bin _time span=1m
  | stats count by src_ip, dest_host, _time
  | streamstats window=60 
      stdev(count) as stdev,
      avg(count) as avg 
      by src_ip, dest_host
  | where stdev < 1 AND avg > 0
  | stats count dc(_time) as intervals 
      by src_ip, dest_host
  | where intervals > 50
  # Regular, low-variance connections = beaconing

CLUSTERING:
  → Group similar activities together
  → Identify outlier groups
  → Useful for user behavior analysis
```

---

## 4. Baseline Comparison

```
BASELINE HUNTING:

ESTABLISH NORMAL:
  → Build baseline of normal activity
  → Compare current activity to baseline
  → Investigate deviations

PROCESS BASELINE:
  # Build list of normal processes
  index=sysmon EventCode=1 earliest=-30d
  | stats count by Image
  | where count > 100
  | table Image
  → Save as "known_processes.csv"

  # Hunt: Find new processes not in baseline
  index=sysmon EventCode=1 earliest=-1d
  | stats count by Image
  | lookup known_processes.csv Image OUTPUT Image as known
  | where isnull(known)
  | sort -count
  → Investigate: New processes not seen before

SERVICE BASELINE:
  # Baseline of services
  index=sysmon EventCode=1 
    ParentImage="*services.exe"
    earliest=-30d
  | stats count by Image
  → Save baseline

  # Hunt: New services
  index=sysmon EventCode=1
    ParentImage="*services.exe"
    earliest=-1d
  | stats count by Image
  | lookup service_baseline.csv Image
  | where isnull(known)
  → New services = investigate

NETWORK BASELINE:
  # Baseline outbound destinations
  index=firewall action=allowed direction=outbound
    earliest=-30d
  | stats count by dest_ip
  → Save baseline

  # Hunt: New outbound destinations
  index=firewall action=allowed direction=outbound
    earliest=-1d
  | stats count by dest_ip
  | lookup network_baseline.csv dest_ip
  | where isnull(known)
  → New destinations = investigate

USER BEHAVIOR BASELINE:
  → Normal login hours per user
  → Normal systems accessed
  → Normal data volumes
  → Normal applications used
  → Deviations may indicate compromise
```

---

## 5. Advanced Hunting Techniques

```
ADVANCED TECHNIQUES:

PARENT-CHILD PROCESS ANALYSIS:
  → Map expected parent-child relationships
  → Flag unexpected relationships

  Expected:
  explorer.exe → chrome.exe ✓
  services.exe → svchost.exe ✓
  
  Suspicious:
  winword.exe → powershell.exe ✗
  outlook.exe → cmd.exe ✗
  svchost.exe → whoami.exe ✗

  # Hunt for suspicious parent-child
  index=sysmon EventCode=1
    (ParentImage="*winword.exe" OR
     ParentImage="*excel.exe" OR
     ParentImage="*outlook.exe")
    (Image="*cmd.exe" OR
     Image="*powershell.exe" OR
     Image="*wscript.exe" OR
     Image="*mshta.exe")
  | table _time, ComputerName, User,
    ParentImage, Image, CommandLine

GROUP ANALYSIS:
  → Compare user's behavior to peers
  → Same role should have similar patterns
  → Outlier in group = potential threat

TEMPORAL ANALYSIS:
  → Activity during unusual hours
  → Activity on weekends/holidays
  → Burst activity vs. steady state
  → Correlate with known events

  # Hunt: After-hours admin activity
  index=windows EventCode=4624 LogonType=10
  | eval hour=strftime(_time, "%H")
  | where hour < 6 OR hour > 22
  | stats count by TargetUserName, IpAddress
  | sort -count
```

---

## Summary Table

| Technique | Method | Strengths | Weaknesses |
|-----------|--------|----------|------------|
| IOC Search | Known indicators | Fast, specific | Only known threats |
| TTP Hunting | Behavior patterns | Finds unknown | More complex |
| Statistical | Frequency/outliers | Novel discovery | False positives |
| Baseline | Compare to normal | Insider threats | Baseline maintenance |
| Parent-Child | Process relationships | Living-off-land | Requires Sysmon |

---

## Revision Questions

1. What are the limitations of IOC-based hunting?
2. How does TTP-based hunting differ from IOC searching?
3. What is long-tail analysis and how does it find threats?
4. How is baseline comparison used to detect anomalies?
5. What parent-child process relationships are suspicious?

---

*Previous: [03-data-sources.md](03-data-sources.md) | Next: [05-tools-and-platforms.md](05-tools-and-platforms.md)*

---

*[Back to README](../README.md)*
