# Unit 6: Detection and Analysis — Topic 5: Timeline Development

## Overview

**Timeline development** reconstructs the chronological sequence of events during a security incident. A well-constructed timeline is essential for understanding the attack narrative, determining scope, identifying root cause, and supporting legal proceedings.

---

## 1. Timeline Construction

```
TIMELINE METHODOLOGY:

  ┌──────────────────────────────────────┐
  │       TIMELINE CONSTRUCTION          │
  │                                      │
  │  1. Identify data sources            │
  │  2. Collect timestamps               │
  │  3. Normalize to UTC                 │
  │  4. Merge and sort chronologically   │
  │  5. Identify key events              │
  │  6. Fill gaps with additional data   │
  │  7. Validate and verify              │
  │  8. Document narrative               │
  └──────────────────────────────────────┘

DATA SOURCES FOR TIMELINE:
  Source               | Events
  Windows Event Logs   | Logins, process, services
  Sysmon               | Process, network, file
  Firewall             | Connections, blocks
  Proxy/DNS            | Web activity, DNS queries
  Email                | Phishing delivery
  EDR                  | Endpoint activity
  Active Directory     | Account changes
  Application logs     | App-specific events
  Cloud audit logs     | Cloud activity
  File system metadata | File creation/modification

TIMESTAMP NORMALIZATION:
  → Convert ALL timestamps to UTC
  → Account for timezone differences
  → Handle clock skew between systems
  → Use NTP-synchronized time where possible
  → Note if system clocks are unreliable
```

---

## 2. Timeline Tools

```
AUTOMATED TIMELINE TOOLS:

PLASO (log2timeline):
  → Super timeline creation
  → Parses 100+ artifact types
  → Outputs to CSV, Elasticsearch, etc.
  → Open source (Python)
  
  # Create timeline from disk image
  log2timeline.py timeline.plaso disk_image.E01
  
  # Output to CSV
  psort.py -o l2tcsv timeline.plaso -w timeline.csv
  
  # Filter by date range
  psort.py -o l2tcsv timeline.plaso 
    "date > '2024-01-10' AND date < '2024-01-20'"
    -w filtered_timeline.csv

TIMELINE EXPLORER:
  → GUI tool by Eric Zimmerman
  → Opens large CSV timelines
  → Filtering and searching
  → Color coding
  → Column customization

KAPE + TIMELINE:
  → Collect artifacts with KAPE
  → Process with log2timeline
  → Analyze in Timeline Explorer
  
  kape.exe --tsource C: --tdest D:\Evidence
    --target KapeTriage --msource D:\Evidence
    --mdest D:\Processed --module MFTECmd

SIEM-BASED TIMELINE:
  # Splunk: Build timeline for entity
  index=* (host="AFFECTED-PC" OR src_ip="10.0.1.50"
    OR user="jsmith")
  | eval source_type=sourcetype
  | table _time, source_type, host, user, action, details
  | sort _time

  # KQL: Timeline query
  search "jsmith" or "10.0.1.50"
  | where TimeGenerated between(
      datetime(2024-01-10)..datetime(2024-01-20))
  | sort by TimeGenerated asc
  | project TimeGenerated, Type, Activity=
      column_ifexists("Activity","")
```

---

## 3. Timeline Analysis

```
EXAMPLE INCIDENT TIMELINE:

  Date/Time (UTC)      | Source    | Event
  ─────────────────────┼──────────┼──────────────────────
  Jan 10 09:15         | Email    | Phishing email received
  Jan 10 09:22         | Email    | User opened attachment
  Jan 10 09:22         | Sysmon   | WINWORD.EXE spawned
                       |          | PowerShell.exe
  Jan 10 09:23         | Sysmon   | PowerShell downloaded
                       |          | payload from evil.com
  Jan 10 09:23         | Firewall | Outbound to 203.0.113.50
  Jan 10 09:24         | Sysmon   | Malware.exe created in
                       |          | AppData\Local\Temp
  Jan 10 09:24         | Sysmon   | Malware.exe executed
  Jan 10 09:25         | Sysmon   | Registry run key created
                       |          | (persistence)
  Jan 10 09:30         | EDR      | Cobalt Strike beacon
                       |          | detected
  Jan 10 09:30-14:00   | Firewall | Regular beaconing to
                       |          | 203.0.113.50 every 60s
  Jan 11 02:15         | AD       | Credential dump detected
                       |          | (Mimikatz indicators)
  Jan 11 02:20         | AD       | Lateral movement to
                       |          | FILESVR01 using PTH
  Jan 11 02:30-06:00   | FileLog  | File access on FILESVR01
                       |          | (2.3 GB accessed)
  Jan 12 03:00         | Proxy    | Data upload to
                       |          | mega.nz (1.8 GB)
  Jan 13 10:30         | SIEM     | Alert triggered on
                       |          | beaconing pattern
  Jan 13 10:45         | SOC      | Investigation started

KEY TIMELINE ANALYSIS:
  → Initial access: Phishing email (Jan 10, 09:15)
  → Execution: Macro → PowerShell (Jan 10, 09:22)
  → Persistence: Registry key (Jan 10, 09:25)
  → C2: Cobalt Strike (Jan 10, 09:30)
  → Credential access: Mimikatz (Jan 11, 02:15)
  → Lateral movement: PTH to file server (Jan 11, 02:20)
  → Data staging: File access (Jan 11, 02:30-06:00)
  → Exfiltration: Upload to mega.nz (Jan 12, 03:00)
  → Detection: Beaconing alert (Jan 13, 10:30)
  → Dwell time: ~3 days
```

---

## 4. Timeline Visualization

```
VISUALIZATION METHODS:

LINEAR TIMELINE:
  Jan 10     Jan 11     Jan 12     Jan 13
  │          │          │          │
  ●─────────●──────────●──────────●
  │          │          │          │
  Phishing   Cred dump  Exfil      Detection
  Execution  Lateral    
  Persistence Movement

ATTACK PHASE MAPPING:
  Phase              | Events               | Duration
  Initial Access     | Phishing email       | Minutes
  Execution          | Macro + PowerShell   | Minutes
  Persistence        | Registry run key     | Minutes
  C2                 | Cobalt Strike beacon | Ongoing
  Credential Access  | Mimikatz             | Hours
  Lateral Movement   | PTH to file server   | Hours
  Collection         | File access          | 3.5 hours
  Exfiltration       | Upload to mega.nz    | Hours
  Detection          | SOC alert            | Day 3

VISUALIZATION TOOLS:
  → Timeline Explorer (Eric Zimmerman)
  → Elastic timeline visualization
  → Splunk investigation timeline
  → Microsoft Sentinel entity timeline
  → Draw.io for custom diagrams
  → Excel/Google Sheets for simple timelines
```

---

## 5. Timeline Best Practices

```
BEST PRACTICES:

  [ ] Use UTC for all timestamps
  [ ] Include data source for each event
  [ ] Distinguish confirmed vs suspected events
  [ ] Update timeline as new evidence emerges
  [ ] Highlight key events (access, persistence, exfil)
  [ ] Map events to ATT&CK techniques
  [ ] Note gaps in timeline (missing data)
  [ ] Cross-reference events across sources
  [ ] Include both attacker and defender actions
  [ ] Keep timeline accessible to entire IR team

COMMON CHALLENGES:
  → Clock skew between systems
  → Missing log data
  → Large volume of events (noise)
  → Multiple time zones
  → Incomplete logging
  → Anti-forensic timestamp manipulation

TIMELINE IN IR REPORT:
  → Include summarized timeline in executive summary
  → Include detailed timeline in technical section
  → Use visual timeline for presentations
  → Reference evidence for each event
  → Note confidence level for each entry
```

---

## Summary Table

| Phase | Tool | Output | Use |
|-------|------|--------|-----|
| Collection | KAPE, FTK Imager | Artifacts | Raw evidence |
| Processing | Plaso/log2timeline | Super timeline | Merged events |
| Analysis | Timeline Explorer | Filtered view | Investigation |
| Visualization | SIEM, diagrams | Visual timeline | Reporting |

---

## Revision Questions

1. What is the methodology for constructing an incident timeline?
2. What tools automate timeline creation from forensic artifacts?
3. How should timestamps be normalized for timeline accuracy?
4. What does an effective incident timeline look like?
5. How does timeline analysis support root cause determination?

---

*Previous: [04-evidence-collection.md](04-evidence-collection.md) | Next: [06-root-cause-analysis.md](06-root-cause-analysis.md)*

---

*[Back to README](../README.md)*
