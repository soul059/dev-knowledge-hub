# Unit 10: Threat Hunting — Topic 3: Data Sources

## Overview

**Data sources** are the foundation of threat hunting — without the right data, even the best hypothesis cannot be tested. Understanding what data is available, where it comes from, and what it reveals is essential for effective hunting. This topic covers the key data sources, their forensic value, and how to ensure adequate data collection for hunting operations.

---

## 1. Data Source Categories

```
DATA SOURCE LANDSCAPE:

  ┌────────────────────────────────────────────┐
  │           DATA SOURCES                     │
  │                                            │
  │  ENDPOINT DATA                             │
  │  ├── Process execution                     │
  │  ├── File system activity                  │
  │  ├── Registry changes                      │
  │  ├── Network connections                   │
  │  ├── Authentication events                 │
  │  └── Service/driver installation           │
  │                                            │
  │  NETWORK DATA                              │
  │  ├── Firewall logs                         │
  │  ├── Proxy/web filter logs                 │
  │  ├── DNS query logs                        │
  │  ├── IDS/IPS alerts                        │
  │  ├── NetFlow/PCAP                          │
  │  └── Email gateway logs                    │
  │                                            │
  │  IDENTITY DATA                             │
  │  ├── Active Directory logs                 │
  │  ├── Authentication logs                   │
  │  ├── Privilege usage                       │
  │  ├── Account changes                       │
  │  └── Group membership changes              │
  │                                            │
  │  CLOUD DATA                                │
  │  ├── Cloud audit logs                      │
  │  ├── SaaS application logs                 │
  │  ├── Cloud infrastructure logs             │
  │  ├── Identity provider logs                │
  │  └── Container/serverless logs             │
  │                                            │
  │  THREAT INTELLIGENCE                       │
  │  ├── IOC feeds                             │
  │  ├── TTP databases (ATT&CK)               │
  │  ├── Industry reports                      │
  │  └── Dark web monitoring                   │
  └────────────────────────────────────────────┘
```

---

## 2. Endpoint Data Sources

```
ENDPOINT DATA — KEY SOURCES:

SYSMON (System Monitor):
  → Most valuable endpoint data source for hunting
  → Detailed process, network, file activity
  → Configurable via XML configuration
  → Free (Microsoft Sysinternals)

  Key Event IDs:
  EID | Event              | Hunting Use
  1   | Process Create     | What executed, by whom
  2   | File creation time | Timestomping detection
  3   | Network Connection | Process network activity
  5   | Process Terminate  | Process lifecycle
  6   | Driver Loaded      | Rootkit detection
  7   | Image Loaded       | DLL loading analysis
  8   | CreateRemoteThread | Code injection
  10  | Process Access     | Credential dumping
  11  | File Create        | Malware drops
  12  | Registry Create/Del| Persistence
  13  | Registry Value Set | Persistence
  15  | File Create Stream | ADS detection
  17  | Pipe Created       | Lateral movement
  18  | Pipe Connected     | Lateral movement
  22  | DNS Query          | DNS activity
  23  | File Delete        | Anti-forensics
  25  | Process Tampering  | Process hollowing

WINDOWS EVENT LOGS:
  Log              | Key Events           | Use
  Security         | 4624, 4625, 4648     | Authentication
                   | 4672, 4720, 4732     | Privilege, accounts
  System           | 7045, 7036           | Service install
  PowerShell       | 4103, 4104           | Script execution
  Task Scheduler   | 106, 140, 200        | Task management
  WMI              | 5857, 5858, 5861     | WMI activity

EDR TELEMETRY:
  → CrowdStrike: Event Search
  → SentinelOne: Deep Visibility
  → Microsoft MDE: Advanced Hunting
  → Carbon Black: Process Search

  # MDE Advanced Hunting (KQL)
  DeviceProcessEvents
  | where Timestamp > ago(7d)
  | where ProcessCommandLine has_any 
      ("mimikatz","sekurlsa","lsadump")
  | project Timestamp, DeviceName, 
      AccountName, ProcessCommandLine
```

---

## 3. Network Data Sources

```
NETWORK DATA — KEY SOURCES:

FIREWALL LOGS:
  → Source/destination IP and port
  → Action (allow/deny)
  → Bytes transferred
  → Session duration
  → NAT translations
  → Application identification

  Hunting Use:
  → Identify C2 communication
  → Detect data exfiltration
  → Find lateral movement
  → Discover unauthorized services

PROXY/WEB FILTER LOGS:
  → Full URL accessed
  → User agent string
  → HTTP method and status
  → Bytes uploaded/downloaded
  → Category (if categorized)
  → User identity (if authenticated)

  Hunting Use:
  → Detect web-based C2
  → Identify suspicious downloads
  → Find data uploads
  → Discover shadow IT

DNS LOGS:
  → Query name and type
  → Response IP
  → Client IP
  → Query timestamp
  → Query frequency

  Hunting Use:
  → Detect DNS tunneling
  → Find C2 domain lookups
  → Discover DGA patterns
  → Identify fast-flux domains

  # Hunt: DNS tunneling
  index=dns
  | eval query_length=len(query)
  | where query_length > 50
  | stats count by src_ip, query
  | where count > 10
  | sort -count

IDS/IPS:
  → Signature-based alerts
  → Protocol anomalies
  → Known exploit patterns
  → Contextual enrichment

NETFLOW:
  → Connection metadata
  → Bytes per flow
  → Duration
  → Port usage
  → Good for long-term trending
```

---

## 4. Identity and Cloud Data

```
IDENTITY DATA SOURCES:

ACTIVE DIRECTORY:
  → Authentication events (success/fail)
  → Kerberos ticket activity
  → Account modifications
  → Group membership changes
  → GPO modifications
  → Replication events

  # Hunt: Kerberoasting
  index=windows EventCode=4769
    TicketEncryptionType=0x17
    ServiceName!="krbtgt"
    ServiceName!="$*"
  | stats count by ServiceName, TargetUserName
  | where count > 5

  # Hunt: DCSync
  index=windows EventCode=4662
    Properties="*1131f6ad*"
  | table _time, SubjectUserName, ObjectName

AZURE AD / ENTRA ID:
  → Sign-in logs
  → Audit logs
  → Risk events
  → Conditional access logs
  → Application consent events

CLOUD DATA SOURCES:
  AWS:
  → CloudTrail (API calls)
  → VPC Flow Logs (network)
  → GuardDuty (threat detection)
  → S3 access logs

  Azure:
  → Activity Log (management)
  → Diagnostic Logs (resource)
  → NSG Flow Logs (network)
  → Sentinel (SIEM)

  GCP:
  → Cloud Audit Logs
  → VPC Flow Logs
  → Cloud Armor Logs

  # Hunt: Unusual API calls (AWS)
  index=cloudtrail
    eventName IN ("CreateUser", "AttachUserPolicy",
    "CreateAccessKey", "AssumeRole")
  | stats count by userIdentity.arn, eventName,
    sourceIPAddress
  | where count > 5
```

---

## 5. Data Source Assessment

```
DATA SOURCE ASSESSMENT:

COVERAGE MATRIX:
  ATT&CK Technique  | Data Source      | Coverage
  T1059 Scripting    | Sysmon EID 1     | ✓ Good
  T1003 Cred Dump    | Sysmon EID 10    | ✓ Good
  T1021 Remote Svc   | Firewall + Auth  | ◐ Partial
  T1048 DNS Tunnel   | DNS logs         | ◐ Partial
  T1053 Sched Tasks  | Sysmon EID 1     | ✓ Good
  T1547 Run Keys     | Sysmon EID 12/13 | ✓ Good
  T1190 Exploit      | IDS + WAF        | ◐ Partial
  T1566 Phishing     | Email gateway    | ✓ Good

GAPS ASSESSMENT:
  □ Do we have Sysmon deployed on all endpoints?
  □ Are Windows Event Logs forwarded to SIEM?
  □ Do we have DNS query logging?
  □ Are proxy logs capturing all web traffic?
  □ Do we have PowerShell script block logging?
  □ Are cloud audit logs collected?
  □ Do we have email gateway logging?
  □ Is NetFlow or PCAP available?
  □ Are authentication logs comprehensive?
  □ Do we have adequate retention?

IMPROVING DATA COLLECTION:
  Priority | Action                    | Impact
  1        | Deploy Sysmon everywhere  | Critical
  2        | Enable PS script logging  | High
  3        | Enable DNS query logging  | High
  4        | Forward all logs to SIEM  | Critical
  5        | Enable cloud audit logs   | High
  6        | Increase log retention    | Medium
  7        | Deploy network monitoring | Medium

LOG RETENTION FOR HUNTING:
  Source          | Minimum  | Recommended
  Sysmon          | 30 days  | 90 days
  Windows Events  | 30 days  | 90 days
  Firewall        | 30 days  | 90 days
  Proxy           | 30 days  | 90 days
  DNS             | 30 days  | 90 days
  Authentication  | 90 days  | 365 days
  Cloud audit     | 90 days  | 365 days
  NetFlow         | 30 days  | 90 days
```

---

## Summary Table

| Source Category | Key Sources | Primary Use | Gap Risk |
|----------------|------------|-------------|----------|
| Endpoint | Sysmon, EDR, Event Logs | Process, file, registry | High if missing |
| Network | Firewall, proxy, DNS | Communication patterns | Medium |
| Identity | AD, Azure AD, IAM | Authentication, access | High if missing |
| Cloud | CloudTrail, Activity Logs | API activity | High in cloud env |
| Threat Intel | IOC feeds, ATT&CK | Context and prioritization | Medium |

---

## Revision Questions

1. Why is Sysmon considered the most valuable endpoint data source for hunting?
2. What network data sources help detect C2 communication?
3. How do identity data sources support hunting for lateral movement?
4. How do you assess data source coverage gaps?
5. What log retention periods support effective threat hunting?

---

*Previous: [02-hypothesis-development.md](02-hypothesis-development.md) | Next: [04-hunting-techniques.md](04-hunting-techniques.md)*

---

*[Back to README](../README.md)*
