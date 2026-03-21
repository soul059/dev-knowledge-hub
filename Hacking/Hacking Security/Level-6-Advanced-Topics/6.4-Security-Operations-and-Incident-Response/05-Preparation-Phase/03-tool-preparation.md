# Unit 5: Preparation Phase — Topic 3: Tool Preparation

## Overview

**Tool preparation** ensures that all technical capabilities needed for incident response are deployed, configured, tested, and accessible when an incident occurs. Having the right tools ready and validated before an incident is critical — there is no time to set up tools during an active breach.

---

## 1. IR Tool Categories

```
IR TOOL STACK:

  ┌─────────────────────────────────────────────────┐
  │             IR TOOL CATEGORIES                   │
  │                                                   │
  │  DETECTION & MONITORING                          │
  │  → SIEM (Splunk, Sentinel, Elastic)             │
  │  → EDR (CrowdStrike, SentinelOne, Defender)     │
  │  → NDR (Zeek, Suricata)                         │
  │  → Email Security                                │
  │                                                   │
  │  INVESTIGATION & ANALYSIS                        │
  │  → Log analysis (SIEM queries)                   │
  │  → Forensic imaging (FTK Imager, dd)            │
  │  → Memory analysis (Volatility, WinPmem)        │
  │  → Network capture (Wireshark, tcpdump)          │
  │  → Malware analysis (sandbox, YARA)              │
  │  → Timeline tools (Plaso, KAPE)                  │
  │                                                   │
  │  CONTAINMENT & REMEDIATION                       │
  │  → EDR isolation capability                      │
  │  → Firewall management                           │
  │  → DNS sinkholing                                │
  │  → Account management (AD, Azure AD)             │
  │  → Patch management                              │
  │                                                   │
  │  ORCHESTRATION & CASE MANAGEMENT                 │
  │  → SOAR (Splunk SOAR, XSOAR, Shuffle)           │
  │  → Case management (TheHive, ServiceNow)         │
  │  → Threat intel (MISP, VirusTotal)               │
  │  → Communication (Slack, Teams, Signal)           │
  └─────────────────────────────────────────────────┘
```

---

## 2. Forensic Tools

```
FORENSIC TOOL PREPARATION:

DISK FORENSICS:
  Tool            | Purpose            | License
  FTK Imager      | Disk imaging       | Free
  Autopsy         | Disk analysis      | Open source
  Sleuth Kit      | CLI forensics      | Open source
  KAPE            | Artifact collection| Free
  Arsenal Image   | Disk mounting      | Free
  X-Ways          | Advanced forensics | Commercial
  EnCase          | Enterprise forens  | Commercial

MEMORY FORENSICS:
  Tool            | Purpose            | License
  Volatility 3    | Memory analysis    | Open source
  WinPmem         | Memory acquisition | Open source
  DumpIt          | Memory dump (Win)  | Free
  LiME            | Memory dump (Linux)| Open source
  Rekall          | Memory analysis    | Open source

NETWORK FORENSICS:
  Tool            | Purpose            | License
  Wireshark       | Packet analysis    | Open source
  tcpdump         | Packet capture     | Open source
  NetworkMiner    | Network forensics  | Open source
  Zeek            | Network analysis   | Open source
  Arkime          | Full PCAP + search | Open source

MALWARE ANALYSIS:
  Tool            | Purpose            | License
  ANY.RUN         | Interactive sandbox| Free tier
  Joe Sandbox     | Automated sandbox  | Free tier
  YARA            | Pattern matching   | Open source
  PEStudio        | PE analysis        | Free
  Process Monitor | Runtime analysis   | Free
  Ghidra          | Reverse engineering| Free (NSA)

PREPARATION STEPS:
  [ ] All tools installed on forensic workstation
  [ ] Tools tested and validated
  [ ] Licenses current
  [ ] Analysts trained on each tool
  [ ] Documentation/cheat sheets created
  [ ] Forensic OS USB drives prepared (SIFT, REMnux)
```

---

## 3. Remote Collection Tools

```
REMOTE EVIDENCE COLLECTION:

VELOCIRAPTOR:
  → Agent-based remote forensics
  → Real-time querying of endpoints
  → Artifact collection at scale
  → Hunting across fleet
  → VQL (Velociraptor Query Language)
  
  # Collect browser history
  SELECT * FROM Artifact.Windows.Application.Chrome.History()
  
  # Collect process listing
  SELECT * FROM pslist()
  
  # Collect prefetch files
  SELECT * FROM Artifact.Windows.Forensics.Prefetch()

KAPE (KROLL):
  → Targeted artifact collection
  → Pre-built collection profiles (targets)
  → Fast evidence acquisition
  → Modular processing
  
  # Collect common artifacts
  kape.exe --tsource C: --tdest D:\Evidence
    --target KapeTriage

OSQUERY:
  → SQL-based endpoint querying
  → Cross-platform (Windows, macOS, Linux)
  → Fleet-wide investigation
  
  -- List running processes
  SELECT pid, name, path, cmdline FROM processes;
  
  -- Check startup items
  SELECT name, path, source FROM startup_items;
  
  -- List listening ports
  SELECT pid, port, address FROM listening_ports;

GRR (Google Rapid Response):
  → Remote live forensics
  → Agent-based
  → Scalable to large environments
  → Flow-based investigation

TOOL READINESS CHECKLIST:
  [ ] Remote collection agents deployed
  [ ] Access credentials secured and available
  [ ] Collection scripts/profiles tested
  [ ] Network connectivity verified
  [ ] Storage for evidence pre-provisioned
  [ ] Analysts trained on remote collection
```

---

## 4. Communication and Coordination Tools

```
COMMUNICATION TOOLS:

SECURE CHANNELS:
  → Signal (encrypted messaging)
  → Dedicated Slack/Teams workspace
  → War room conference bridge
  → Encrypted email (S/MIME, PGP)
  → Secure file sharing

CASE MANAGEMENT:
  TheHive:
  → Open source IR case management
  → Integrates with Cortex (enrichment)
  → Task assignment and tracking
  → Observable management
  → Alert import from SIEM
  → Template-based case creation

  ServiceNow SecOps:
  → Enterprise IT integration
  → Workflow automation
  → SLA tracking
  → Executive dashboards

TICKETING:
  → Track all incidents and tasks
  → Link related incidents
  → Document evidence and findings
  → Measure response metrics
  → Generate reports

COORDINATION CHECKLIST:
  [ ] War room procedures documented
  [ ] Conference bridge numbers available
  [ ] Secure messaging configured
  [ ] Case management system operational
  [ ] Evidence storage accessible
  [ ] Status reporting templates ready
  [ ] Escalation contacts verified
```

---

## 5. Tool Testing and Validation

```
TOOL VALIDATION PROCESS:

TESTING SCHEDULE:
  Frequency  | Activity
  Weekly     | Verify tool accessibility
  Monthly    | Run sample collection/analysis
  Quarterly  | Full end-to-end test
  Annually   | Comprehensive validation exercise

VALIDATION CHECKLIST:
  DETECTION:
  [ ] SIEM receiving logs from all sources
  [ ] EDR agents reporting on all endpoints
  [ ] Alert rules firing correctly
  [ ] Dashboard data is current

  INVESTIGATION:
  [ ] Forensic workstation functional
  [ ] Remote collection tools operational
  [ ] Analysis tools installed and updated
  [ ] Query access to all log sources

  CONTAINMENT:
  [ ] Can isolate endpoint via EDR
  [ ] Can block IP/domain at firewall
  [ ] Can disable user accounts
  [ ] Can quarantine email

  COMMUNICATION:
  [ ] Secure channels operational
  [ ] Contact lists current
  [ ] War room accessible
  [ ] Case management functional

COMMON FAILURES:
  → Expired licenses
  → Outdated agent versions
  → Lost credentials/access
  → Changed network architecture
  → New systems not covered
  → Staff turnover (no one trained)

ACCESS MANAGEMENT:
  → IR team members have required access
  → Emergency/break-glass accounts documented
  → Access tested periodically
  → MFA configured for IR tools
  → Separate admin accounts for IR
```

---

## Summary Table

| Tool Category | Key Tools | Preparation Priority |
|--------------|----------|---------------------|
| Detection | SIEM, EDR, NDR | Critical |
| Forensics | FTK Imager, Volatility, KAPE | High |
| Remote Collection | Velociraptor, osquery | High |
| Containment | EDR isolation, firewall | Critical |
| Case Management | TheHive, ServiceNow | Medium |
| Communication | Signal, Teams, bridge | High |

---

## Revision Questions

1. What are the main categories of IR tools?
2. What forensic tools should be pre-installed on the forensic workstation?
3. How do remote collection tools like Velociraptor aid IR?
4. Why is regular tool validation important for IR readiness?
5. What are common tool-related failures during real incidents?

---

*Previous: [02-playbook-creation.md](02-playbook-creation.md) | Next: [04-training-and-exercises.md](04-training-and-exercises.md)*

---

*[Back to README](../README.md)*
