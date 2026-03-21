# Unit 3: Azure Security — Topic 6: Azure Sentinel

## Overview

**Microsoft Sentinel** is Azure's cloud-native **SIEM (Security Information and Event Management)** and **SOAR (Security Orchestration, Automation, and Response)** solution. It collects data from across your organization, detects threats using analytics and AI, investigates incidents, and automates responses with playbooks. Sentinel is central to modern Azure security operations.

---

## 1. Sentinel Architecture

```
SENTINEL COMPONENTS:

  ┌──────────────────────────────────────────┐
  │          MICROSOFT SENTINEL              │
  │                                          │
  │  DATA CONNECTORS:                        │
  │  ┌────────┐ ┌────────┐ ┌────────┐       │
  │  │Azure AD│ │Office  │ │Defender│       │
  │  │Logs    │ │365 Logs│ │Alerts  │       │
  │  └───┬────┘ └───┬────┘ └───┬────┘       │
  │      │          │          │             │
  │  ┌───▼──────────▼──────────▼───┐         │
  │  │    LOG ANALYTICS WORKSPACE  │         │
  │  │    (Data Storage - KQL)     │         │
  │  └───┬─────────────────────────┘         │
  │      │                                   │
  │  ┌───▼──────────────────┐                │
  │  │  ANALYTICS RULES     │                │
  │  │  (Detection Logic)   │                │
  │  └───┬──────────────────┘                │
  │      │                                   │
  │  ┌───▼──────────────────┐                │
  │  │  INCIDENTS           │                │
  │  │  (Grouped Alerts)    │                │
  │  └───┬──────────────────┘                │
  │      │                                   │
  │  ┌───▼──────────────────┐                │
  │  │  PLAYBOOKS           │                │
  │  │  (Automated Response)│                │
  │  └─────────────────────┘                │
  └──────────────────────────────────────────┘

KEY COMPONENTS:
  → Data Connectors: Ingest log data
  → Log Analytics: Store and query data (KQL)
  → Analytics Rules: Detect threats
  → Incidents: Grouped alerts for investigation
  → Workbooks: Dashboards and visualization
  → Playbooks: Automated response (Logic Apps)
  → Hunting: Proactive threat search
  → Notebooks: Jupyter for advanced analysis
```

---

## 2. Data Connectors

```
DATA SOURCES:

MICROSOFT SERVICES:
  → Azure AD sign-in and audit logs
  → Microsoft 365 (Exchange, SharePoint, Teams)
  → Defender for Cloud alerts
  → Defender for Endpoint
  → Defender for Office 365
  → Azure Activity Log
  → Azure Firewall
  → NSG Flow Logs

THIRD-PARTY:
  → AWS CloudTrail
  → GCP Audit Logs
  → Palo Alto Networks
  → Check Point
  → Fortinet
  → F5
  → CrowdStrike
  → Carbon Black

CUSTOM:
  → Common Event Format (CEF)
  → Syslog
  → REST API
  → Azure Functions
  → Log Analytics agent
  → Azure Monitor Agent (AMA)

ENABLING CONNECTORS:
  # Azure Activity connector
  Content Hub → Azure Activity → Install
  Data connectors → Azure Activity → Connect
  
  # Azure AD connector
  Data connectors → Azure Active Directory
  → Select: Sign-in logs, Audit logs
  → Connect

DATA TABLES (KQL):
  SigninLogs              → Azure AD sign-ins
  AuditLogs               → Azure AD changes
  SecurityAlert            → Security alerts
  SecurityIncident         → Sentinel incidents
  AzureActivity            → Azure operations
  OfficeActivity           → M365 activity
  CommonSecurityLog        → CEF data
  Syslog                   → Syslog data
  ThreatIntelligenceIndicator → IOCs
```

---

## 3. Analytics Rules and Detection

```
ANALYTICS RULES:

RULE TYPES:
  → Microsoft Security: Import alerts from Microsoft services
  → Scheduled: Custom KQL queries on schedule
  → Fusion: ML-based multi-stage attack detection
  → NRT (Near Real-Time): Low-latency detection
  → Anomaly: Built-in anomaly detection

SCHEDULED RULE EXAMPLE:
  Name: "Multiple failed sign-ins followed by success"
  Severity: Medium
  Tactics: InitialAccess
  
  KQL Query:
  let threshold = 5;
  let timeframe = 1h;
  SigninLogs
  | where TimeGenerated > ago(timeframe)
  | where ResultType != "0"  // Failed
  | summarize FailedCount = count(),
              FailedIPs = make_set(IPAddress)
              by UserPrincipalName
  | where FailedCount >= threshold
  | join kind=inner (
      SigninLogs
      | where TimeGenerated > ago(timeframe)
      | where ResultType == "0"  // Success
      | project UserPrincipalName, SuccessTime = TimeGenerated,
                SuccessIP = IPAddress
  ) on UserPrincipalName
  | project UserPrincipalName, FailedCount, FailedIPs,
            SuccessTime, SuccessIP

COMMON DETECTION RULES:
  → Brute force attacks (multiple failed logins)
  → Impossible travel (logins from distant locations)
  → Suspicious PowerShell activity
  → New admin role assignment
  → Mass file download/deletion
  → Malware communication (known C2 IPs)
  → Lateral movement patterns
  → Data exfiltration indicators

KQL ESSENTIALS:
  // Filter
  SecurityAlert | where Severity == "High"
  
  // Aggregate
  SigninLogs | summarize count() by UserPrincipalName
  
  // Time range
  AzureActivity | where TimeGenerated > ago(24h)
  
  // Join
  Table1 | join kind=inner Table2 on CommonField
  
  // Project
  SigninLogs | project UserPrincipalName, IPAddress, TimeGenerated
```

---

## 4. Incident Management

```
INCIDENT WORKFLOW:

  New Incident
      │
  ┌───▼───┐
  │ Triage │ → Severity, assignment, initial review
  └───┬───┘
      │
  ┌───▼──────────┐
  │ Investigation │ → Entities, timeline, evidence
  └───┬──────────┘
      │
  ┌───▼──────────┐
  │ Response     │ → Contain, remediate, recover
  └───┬──────────┘
      │
  ┌───▼──────────┐
  │ Closure      │ → Document, lessons learned
  └──────────────┘

INVESTIGATION:
  → Entity mapping (users, IPs, hosts)
  → Investigation graph (visual relationships)
  → Timeline view
  → Bookmarks (mark important evidence)
  → Comments and collaboration

AUTOMATION WITH PLAYBOOKS:
  → Logic Apps-based automation
  → Triggered by incidents or alerts
  
  Example Playbooks:
  → Block user in Azure AD
  → Block IP in NSG/Firewall
  → Create ServiceNow ticket
  → Send Teams notification
  → Enrich alert with threat intel
  → Isolate compromised VM
  → Revoke user sessions
```

---

## 5. Threat Hunting

```
PROACTIVE HUNTING:

HUNTING QUERIES:
  → Pre-built queries in Sentinel
  → Custom KQL queries
  → MITRE ATT&CK mapped
  → Community-contributed

EXAMPLE HUNTING QUERIES:

// Find anomalous login locations
SigninLogs
| where TimeGenerated > ago(7d)
| where ResultType == "0"
| summarize Locations = make_set(Location),
            LocationCount = dcount(Location)
            by UserPrincipalName
| where LocationCount > 3
| sort by LocationCount desc

// Detect PowerShell encoded commands
SecurityEvent
| where EventID == 4688
| where CommandLine contains "-enc" or 
        CommandLine contains "-EncodedCommand"
| project TimeGenerated, Computer, Account, CommandLine

// Large data transfer detection
AzureNetworkAnalytics_CL
| where TimeGenerated > ago(24h)
| where BytesSentToDestination_d > 100000000  // 100MB
| summarize TotalBytes = sum(BytesSentToDestination_d) 
            by SourceIP_s, DestinationIP_s
| sort by TotalBytes desc

LIVESTREAM:
  → Real-time query execution
  → Monitor for specific patterns
  → Interactive hunting session
  → Results as they happen

NOTEBOOKS:
  → Jupyter notebooks in Sentinel
  → Advanced analytics
  → Python/KQL integration
  → Guided investigation workflows
  → Machine learning models
  → Community notebooks available
```

---

## Summary Table

| Component | Purpose | Key Technology |
|-----------|---------|---------------|
| Data Connectors | Log ingestion | API, agents, CEF |
| Analytics Rules | Threat detection | KQL queries |
| Incidents | Alert management | Grouping, assignment |
| Playbooks | Automated response | Logic Apps |
| Workbooks | Visualization | Dashboards |
| Hunting | Proactive search | KQL, notebooks |

---

## Revision Questions

1. What are the main components of Microsoft Sentinel?
2. How do analytics rules detect threats using KQL?
3. What is the difference between scheduled and fusion rules?
4. How do playbooks automate incident response?
5. What is threat hunting and how is it performed in Sentinel?

---

*Previous: [05-azure-security-center.md](05-azure-security-center.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
