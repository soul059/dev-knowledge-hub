# Unit 6: Detection and Analysis — Topic 2: Initial Analysis

## Overview

**Initial analysis** is the first deep-dive investigation after an alert is triaged as a potential true positive. This phase determines whether the alert represents an actual security incident, assesses the immediate impact, and gathers enough information to guide the response. Initial analysis bridges triage and full investigation.

---

## 1. Initial Analysis Process

```
INITIAL ANALYSIS WORKFLOW:

  Alert Triaged as Potential TP
       │
  ┌────▼──────────────────────┐
  │ 1. GATHER ALERT DETAILS   │
  │    Full alert context      │
  │    Source data/raw logs    │
  └────┬──────────────────────┘
       │
  ┌────▼──────────────────────┐
  │ 2. INVESTIGATE ENTITIES   │
  │    User, host, IP, file   │
  │    Recent activity        │
  └────┬──────────────────────┘
       │
  ┌────▼──────────────────────┐
  │ 3. CORRELATE EVENTS       │
  │    Related alerts          │
  │    Timeline of activity    │
  └────┬──────────────────────┘
       │
  ┌────▼──────────────────────┐
  │ 4. ASSESS IMPACT          │
  │    Systems affected        │
  │    Data at risk           │
  └────┬──────────────────────┘
       │
  ┌────▼──────────────────────┐
  │ 5. DETERMINE              │
  │    Incident? Severity?     │
  │    Declare or close        │
  └────────────────────────────┘

INITIAL ANALYSIS CHECKLIST:
  [ ] Raw log data reviewed
  [ ] Entity investigation complete
  [ ] Related events identified
  [ ] Timeline constructed (initial)
  [ ] Impact assessment (preliminary)
  [ ] Decision: Incident or not
  [ ] If incident: Severity declared
  [ ] If incident: IR process initiated
  [ ] Documentation updated
```

---

## 2. Investigation Techniques

```
INVESTIGATION BY DATA TYPE:

ENDPOINT INVESTIGATION:
  → Process tree analysis
  → Parent-child process relationships
  → Command line arguments
  → File system changes
  → Registry modifications
  → Network connections
  → Loaded modules/DLLs
  → Scheduled tasks and services

  Key Queries (Splunk SPL):
  # Process tree for host
  index=sysmon EventCode=1 ComputerName="AFFECTED-HOST"
  | table _time, User, ParentImage, Image, CommandLine
  | sort _time

  # Network connections
  index=sysmon EventCode=3 ComputerName="AFFECTED-HOST"
  | table _time, User, Image, DestinationIp, 
    DestinationPort

  # File creation
  index=sysmon EventCode=11 ComputerName="AFFECTED-HOST"
  | table _time, Image, TargetFilename

NETWORK INVESTIGATION:
  → Firewall logs for source/destination
  → Proxy logs for URLs accessed
  → DNS queries made
  → Data transfer volumes
  → Connection patterns (beaconing)
  → Geo-location of external IPs

  Key Queries:
  # All connections from host
  index=firewall src_ip="10.0.1.50"
  | stats count by dest_ip, dest_port, action
  | sort -count

  # DNS queries
  index=dns src_ip="10.0.1.50"
  | stats count by query
  | sort -count

IDENTITY INVESTIGATION:
  → Authentication logs (success/failure)
  → Access to resources (file shares, apps)
  → Account changes (password, group membership)
  → Email activity (sent, received, rules)
  → Geographic login locations
  → Device/browser fingerprints
```

---

## 3. Analysis Tools and Techniques

```
ANALYSIS TOOLKIT:

SIEM INVESTIGATION:
  → Pivot from alert to raw data
  → Search by entity (user, host, IP)
  → Time-bounded searches
  → Statistical analysis
  → Transaction/session grouping

EDR INVESTIGATION:
  → Process timeline
  → File activity
  → Network connections
  → Behavior indicators
  → Quarantine/isolation

  CrowdStrike:
  → Event Search → filter by host
  → Process Explorer → visualize tree
  → Detection details → technique mapped

  SentinelOne:
  → Deep Visibility → query events
  → Attack Story → visual timeline
  → Threat Intelligence → IOC matching

LOG ANALYSIS COMMANDS:
  # Search for user activity across sources
  index=* user="jsmith"
  | stats count by index, sourcetype, action
  | sort -count

  # Timeline of all activity for IP
  index=* (src_ip="203.0.113.50" OR dest_ip="203.0.113.50")
  | sort _time
  | table _time, index, sourcetype, src_ip, dest_ip, action

  # KQL (Sentinel)
  search "203.0.113.50"
  | where TimeGenerated > ago(24h)
  | project TimeGenerated, Type, SourceIP=column_ifexists("SourceIP",""),
    DestinationIP=column_ifexists("DestinationIP","")
  | sort by TimeGenerated asc

ENRICHMENT DURING ANALYSIS:
  → VirusTotal: Hash, IP, URL reputation
  → Shodan: Service identification
  → WHOIS: Domain ownership
  → GeoIP: Location of IPs
  → Internal CMDB: Asset details
  → HR system: User context
  → Previous incidents: Historical pattern
```

---

## 4. Impact Assessment

```
PRELIMINARY IMPACT ASSESSMENT:

QUESTIONS TO ANSWER:
  → What systems are affected?
  → What data is at risk?
  → How many users are impacted?
  → Is the threat active or contained?
  → What is the business impact?
  → Are critical systems involved?
  → Is there regulatory implication?

IMPACT CATEGORIES:
  Category      | Questions
  Confidentiality| Was data exposed or stolen?
  Integrity     | Was data modified?
  Availability  | Are systems/services down?

SEVERITY ASSESSMENT:
  Factor              | Low      | Medium     | High      | Critical
  Systems affected    | 1        | 2-10       | 10-50     | 50+ / critical
  Users affected      | 1        | 2-50       | 50-500    | 500+
  Data sensitivity    | Public   | Internal   | Confidential| Regulated PII
  Business impact     | None     | Minor      | Significant| Operations halt
  Active threat       | No       | Uncertain  | Likely    | Confirmed active
  Containment         | Easy     | Moderate   | Difficult | Unknown scope

INITIAL SEVERITY DECLARATION:
  Based on assessment, declare incident severity:
  → Critical: Active breach, critical systems, data loss
  → High: Confirmed compromise, significant risk
  → Medium: Suspicious activity, investigation needed
  → Low: Minor policy violation, contained
```

---

## 5. Analysis Documentation

```
ANALYSIS REPORT TEMPLATE:

  ┌──────────────────────────────────────────┐
  │ INITIAL ANALYSIS REPORT                  │
  │                                          │
  │ Alert: ALR-2024-12345                   │
  │ Analyst: Jane Doe                        │
  │ Date: 2024-01-15                        │
  │ Duration of Analysis: 45 minutes        │
  │                                          │
  │ SUMMARY:                                │
  │ Analysis confirms account compromise     │
  │ of user jsmith. Attacker gained access   │
  │ via brute force, accessed email and      │
  │ downloaded sensitive documents.          │
  │                                          │
  │ ENTITIES INVESTIGATED:                   │
  │ → User: jsmith@company.com              │
  │ → Source IP: 203.0.113.50 (Russia)      │
  │ → Host: JSMITH-PC (10.0.1.50)          │
  │ → Email: 15 documents accessed          │
  │                                          │
  │ FINDINGS:                               │
  │ 1. 47 failed logins from 203.0.113.50   │
  │ 2. Successful login at 10:35 UTC        │
  │ 3. Email mailbox accessed               │
  │ 4. 15 attachments downloaded            │
  │ 5. Forwarding rule created              │
  │ 6. No lateral movement detected (yet)   │
  │                                          │
  │ ASSESSMENT:                              │
  │ Severity: HIGH                           │
  │ Category: Account Compromise             │
  │ Active threat: Contained (acct disabled) │
  │ Data impact: Possible PII exposure       │
  │                                          │
  │ RECOMMENDATION:                          │
  │ Declare incident, proceed to full        │
  │ investigation and scope assessment.      │
  │ Engage legal for potential data breach.  │
  └──────────────────────────────────────────┘
```

---

## Summary Table

| Analysis Step | Activities | Time Target |
|--------------|-----------|-------------|
| Gather details | Raw logs, alert context | 10 min |
| Entity investigation | User, host, IP analysis | 15 min |
| Correlation | Related events, timeline | 15 min |
| Impact assessment | Systems, data, severity | 10 min |
| Decision | Declare or close | 5 min |

---

## Revision Questions

1. What steps make up the initial analysis process?
2. What endpoint data should be examined during initial analysis?
3. How do you assess the preliminary impact of a potential incident?
4. What tools assist with initial analysis investigations?
5. What should an initial analysis report include?

---

*Previous: [01-alert-triage.md](01-alert-triage.md) | Next: [03-scope-determination.md](03-scope-determination.md)*

---

*[Back to README](../README.md)*
