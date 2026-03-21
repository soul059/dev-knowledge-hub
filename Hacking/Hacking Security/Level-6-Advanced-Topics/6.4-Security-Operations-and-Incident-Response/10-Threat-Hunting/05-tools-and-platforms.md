# Unit 10: Threat Hunting — Topic 5: Tools and Platforms

## Overview

**Threat hunting tools and platforms** provide the infrastructure for data collection, searching, analysis, and visualization needed for effective hunting. From open-source SIEM solutions to enterprise hunting platforms, understanding the tool landscape enables hunters to choose the right tools for their environment and budget.

---

## 1. SIEM-Based Hunting

```
SIEM PLATFORMS FOR HUNTING:

SPLUNK:
  → Industry-leading search and analytics
  → SPL (Search Processing Language)
  → Extensive app ecosystem
  → Real-time and historical search
  → Custom dashboards and reports

  Hunting Queries:
  # Process execution analysis
  index=sysmon EventCode=1
  | stats count by Image
  | sort count
  | head 20

  # Network beaconing detection
  index=proxy
  | bucket _time span=5m
  | stats count by src_ip, dest, _time
  | eventstats stdev(count) as sd by src_ip, dest
  | where sd < 1

ELASTIC STACK (ELK):
  → Elasticsearch: Search and analytics
  → Logstash: Log ingestion
  → Kibana: Visualization
  → Open source core
  → KQL (Kibana Query Language)
  → Elastic Security: SIEM features

  Hunting in Kibana:
  # Process hunting
  process.name: "powershell.exe" AND
  process.command_line: *download*

  # Timeline feature for investigation
  # Elastic Security > Timelines

MICROSOFT SENTINEL:
  → Cloud-native SIEM
  → KQL (Kusto Query Language)
  → Built-in hunting workbooks
  → Notebooks (Jupyter)
  → Threat intelligence integration
  → Entity behavior analytics

  Hunting Queries (KQL):
  // Suspicious PowerShell
  DeviceProcessEvents
  | where FileName =~ "powershell.exe"
  | where ProcessCommandLine has_any 
      ("downloadstring","invoke-expression",
       "encodedcommand")
  | project Timestamp, DeviceName, AccountName,
    ProcessCommandLine
  | sort by Timestamp desc

  // Failed logins followed by success
  SecurityEvent
  | where EventID == 4625
  | summarize FailCount=count() by 
      TargetAccount, IpAddress, bin(TimeGenerated, 1h)
  | where FailCount > 5
  | join kind=inner (
      SecurityEvent | where EventID == 4624
  ) on TargetAccount, IpAddress
```

---

## 2. EDR-Based Hunting

```
EDR PLATFORMS FOR HUNTING:

CROWDSTRIKE FALCON:
  → Event Search: Query endpoint telemetry
  → Falcon OverWatch: Managed hunting
  → Custom IOA (Indicators of Attack)
  → Process tree visualization
  → Real-time response shell

  Hunting in CrowdStrike:
  → Investigate > Event Search
  → Filter by event type, time, host
  → Event types: ProcessRollup2, 
    DnsRequest, NetworkConnect
  → Custom hunting dashboards

MICROSOFT DEFENDER FOR ENDPOINT:
  → Advanced Hunting (KQL)
  → Rich endpoint telemetry
  → Cross-product hunting (M365)
  → Threat analytics
  → Custom detection rules

  Advanced Hunting:
  // Hunt for LSASS access
  DeviceProcessEvents
  | where Timestamp > ago(7d)
  | where FileName in~ ("procdump.exe",
      "mimikatz.exe","taskmgr.exe")
  | where ProcessCommandLine has "lsass"

  // Hunt for encoded PowerShell
  DeviceProcessEvents
  | where FileName =~ "powershell.exe"
  | where ProcessCommandLine has "-enc"
    or ProcessCommandLine has "-encodedcommand"
  | project Timestamp, DeviceName, AccountName,
    ProcessCommandLine

SENTINELONE:
  → Deep Visibility: Endpoint query
  → Storyline: Attack chain visualization
  → Star (Storyline Active Response)
  → Custom rules and hunting queries

CARBON BLACK:
  → Process search and analysis
  → Binary search
  → Watchlists for ongoing monitoring
  → Live Response for investigation
```

---

## 3. Specialized Hunting Tools

```
DEDICATED HUNTING TOOLS:

VELOCIRAPTOR:
  → Open-source DFIR and hunting
  → VQL (Velociraptor Query Language)
  → Remote artifact collection
  → Agentless or agent-based
  → Scales to large environments

  # Hunt for persistence
  SELECT * FROM Artifact.Windows.Persistence.
    PermanentWMIEvents()

  # Hunt for suspicious processes
  SELECT Pid, Name, Exe, CommandLine
  FROM pslist()
  WHERE Name =~ "powershell"
    AND CommandLine =~ "download"

OSQUERY:
  → SQL-based endpoint visibility
  → Cross-platform (Win, Mac, Linux)
  → Real-time and scheduled queries
  → Fleet management (Kolide, Fleetdm)

  -- Listening ports
  SELECT * FROM listening_ports
  WHERE address != '127.0.0.1'
    AND port NOT IN (80, 443, 22);

  -- Scheduled tasks
  SELECT * FROM scheduled_tasks
  WHERE enabled = 1
    AND last_run_time > '2024-01-01';

  -- Startup items
  SELECT * FROM startup_items
  WHERE path NOT LIKE 'C:\Windows\%'
    AND path NOT LIKE 'C:\Program Files%';

JUPYTER NOTEBOOKS:
  → Interactive analysis environment
  → Python/R for custom analysis
  → Visualization libraries
  → Reproducible hunting workflows
  → Integration with SIEM APIs

  # Example: Beaconing analysis in Python
  import pandas as pd
  import numpy as np
  
  df = pd.read_csv('proxy_logs.csv')
  grouped = df.groupby(['src_ip','dest_host'])
  
  for name, group in grouped:
      intervals = group['timestamp'].diff()
      if intervals.std() < 5:  # Low variance
          print(f"Beaconing: {name}")

YARA:
  → Pattern matching for malware
  → File and memory scanning
  → Custom rule creation
  → Integration with many tools

  rule suspicious_powershell {
    strings:
      $a = "DownloadString" ascii nocase
      $b = "IEX" ascii nocase
      $c = "Invoke-Expression" ascii nocase
    condition:
      any of them
  }
```

---

## 4. Threat Intelligence Platforms

```
TI PLATFORMS FOR HUNTING:

MISP:
  → Open-source TI sharing platform
  → IOC management and sharing
  → Community feeds
  → SIEM integration
  → Correlation engine

OPENCTI:
  → Open-source cyber threat intelligence
  → STIX/TAXII support
  → Knowledge graph
  → Integration with hunting tools

MITRE ATT&CK NAVIGATOR:
  → Visualize detection coverage
  → Map hunting activities
  → Identify coverage gaps
  → Plan hunt priorities

  Usage:
  1. Map existing detections
  2. Identify uncovered techniques
  3. Prioritize hunts for gaps
  4. Track hunting coverage over time

COMMERCIAL TI:
  → Recorded Future
  → Mandiant Advantage
  → CrowdStrike Falcon Intelligence
  → Anomali ThreatStream
  → IBM X-Force Exchange

TI INTEGRATION FOR HUNTING:
  → Enrich hunt findings with TI
  → Correlate IOCs during hunts
  → Prioritize hunts by threat landscape
  → Context for discovered indicators
  → Attribution support
```

---

## 5. Building a Hunting Toolkit

```
RECOMMENDED TOOLKIT BY BUDGET:

FREE / OPEN-SOURCE:
  Component       | Tool
  SIEM            | Elastic Stack (ELK)
  Endpoint Query  | Velociraptor + osquery
  Network Monitor | Security Onion (Zeek + Suricata)
  PCAP Analysis   | Wireshark
  Threat Intel    | MISP + MITRE ATT&CK
  Malware Analysis| YARA + ClamAV
  Forensics       | Autopsy + Volatility
  Notebooks       | Jupyter
  Visualization   | Kibana + Grafana

ENTERPRISE:
  Component       | Tool
  SIEM            | Splunk / Sentinel
  EDR             | CrowdStrike / MDE
  Network         | Arkime / Vectra
  TI              | Recorded Future / Mandiant
  SOAR            | Cortex XSOAR / Splunk SOAR
  Forensics       | EnCase / FTK
  Hunting Platform| CrowdStrike Falcon

TOOL SELECTION CRITERIA:
  → Data coverage (what telemetry?)
  → Query capability (search language?)
  → Scale (how much data?)
  → Integration (works with existing?)
  → Cost (license + infrastructure)
  → Learning curve (team capability?)
  → Community support
  → API availability
  → Automation capabilities
```

---

## Summary Table

| Tool Category | Free Option | Enterprise Option | Primary Use |
|--------------|-------------|-------------------|-------------|
| SIEM | ELK Stack | Splunk, Sentinel | Log search/analytics |
| EDR | Velociraptor | CrowdStrike, MDE | Endpoint hunting |
| Network | Security Onion | Vectra, Arkime | Network hunting |
| TI | MISP, ATT&CK | Recorded Future | Intelligence context |
| Analysis | Jupyter, Python | Splunk ML | Statistical analysis |
| Scanning | YARA, osquery | Carbon Black | IOC/pattern matching |

---

## Revision Questions

1. How do SIEM platforms support threat hunting?
2. What advantages do EDR platforms provide for hunting over SIEM?
3. How does Velociraptor differ from traditional EDR for hunting?
4. What role do Jupyter notebooks play in threat hunting?
5. How would you build a hunting toolkit on a limited budget?

---

*Previous: [04-hunting-techniques.md](04-hunting-techniques.md) | Next: [06-hunting-metrics.md](06-hunting-metrics.md)*

---

*[Back to README](../README.md)*
