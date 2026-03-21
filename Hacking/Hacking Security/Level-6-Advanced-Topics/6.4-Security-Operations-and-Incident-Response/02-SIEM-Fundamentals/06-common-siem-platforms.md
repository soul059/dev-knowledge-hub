# Unit 2: SIEM Fundamentals — Topic 6: Common SIEM Platforms

## Overview

Selecting the right **SIEM platform** is one of the most impactful decisions for security operations. Each platform has distinct strengths in query language, scalability, pricing model, and ecosystem integration. This topic provides a detailed comparison of major SIEM platforms to help guide selection and understanding.

---

## 1. Splunk Enterprise Security

```
SPLUNK:

  TYPE: Commercial (market leader)
  DEPLOYMENT: On-prem, cloud (Splunk Cloud), hybrid
  QUERY LANGUAGE: SPL (Search Processing Language)
  PRICING: Volume-based (GB/day ingested)

  ARCHITECTURE:
  ┌──────────┐    ┌──────────┐    ┌──────────┐
  │ Universal │───▶│ Indexers │───▶│ Search   │
  │ Forwarder│    │ (Store)  │    │ Heads    │
  └──────────┘    └──────────┘    └──────────┘
                       ↑              │
                  ┌────┴────┐    ┌────▼────┐
                  │ Heavy   │    │ Splunk  │
                  │Forwarder│    │ Web UI  │
                  └─────────┘    └─────────┘

  SPL EXAMPLES:
  # Basic search
  index=main sourcetype=syslog | stats count by host
  
  # Failed logins
  index=windows EventCode=4625
  | stats count by src_ip, user
  | where count > 10
  | sort -count
  
  # Process creation with parent
  index=sysmon EventCode=1
  | table _time, ComputerName, User, 
    ParentImage, Image, CommandLine

  STRENGTHS:
  ✓ Powerful, flexible SPL query language
  ✓ Massive scalability
  ✓ Splunkbase app ecosystem (2000+ apps)
  ✓ Enterprise Security (ES) premium app
  ✓ Strong community and documentation
  ✓ SOAR integration (Phantom/SOAR)
  ✓ Machine learning toolkit

  WEAKNESSES:
  ✗ Expensive (license costs)
  ✗ Complex administration
  ✗ Resource-intensive infrastructure
  ✗ Steep learning curve for advanced SPL
  ✗ License management overhead
```

---

## 2. Microsoft Sentinel

```
MICROSOFT SENTINEL:

  TYPE: Cloud-native SIEM (Azure)
  DEPLOYMENT: Cloud only (Azure)
  QUERY LANGUAGE: KQL (Kusto Query Language)
  PRICING: Pay-per-GB ingested + retention

  ARCHITECTURE:
  ┌────────────┐    ┌──────────────┐    ┌──────────┐
  │ Data       │───▶│ Log Analytics│───▶│ Sentinel │
  │ Connectors │    │ Workspace    │    │ Portal   │
  └────────────┘    └──────────────┘    └──────────┘
       │                                      │
  ┌────┴────┐                           ┌─────▼─────┐
  │ 200+    │                           │ Workbooks │
  │ Built-in│                           │ Notebooks │
  │ Sources │                           │ Playbooks │
  └─────────┘                           └───────────┘

  KQL EXAMPLES:
  // Failed logins by IP
  SecurityEvent
  | where EventID == 4625
  | summarize FailCount=count() by IpAddress
  | where FailCount > 10
  | sort by FailCount desc
  
  // Process creation timeline
  DeviceProcessEvents
  | where Timestamp > ago(1h)
  | project Timestamp, DeviceName, AccountName,
    FileName, ProcessCommandLine
  | sort by Timestamp desc
  
  // Threat hunting
  let suspicious_ips = 
    ThreatIntelligenceIndicator
    | where NetworkIP != ""
    | distinct NetworkIP;
  CommonSecurityLog
  | where DestinationIP in (suspicious_ips)

  STRENGTHS:
  ✓ Cloud-native, serverless (no infrastructure)
  ✓ Native Microsoft ecosystem integration
  ✓ Built-in SOAR (Logic Apps)
  ✓ Free data ingestion (Microsoft sources)
  ✓ Growing connector ecosystem
  ✓ Jupyter Notebooks for hunting
  ✓ Cost-effective for Microsoft shops
  ✓ UEBA built-in

  WEAKNESSES:
  ✗ Azure-only deployment
  ✗ KQL learning curve
  ✗ Costs can scale unpredictably
  ✗ Less mature than Splunk
  ✗ Limited on-prem integration
```

---

## 3. Elastic Security

```
ELASTIC SECURITY:

  TYPE: Open source core + commercial features
  DEPLOYMENT: Self-managed, Elastic Cloud, ECK (K8s)
  QUERY LANGUAGE: Lucene, KQL, EQL, ES|QL
  PRICING: Free (basic) / Subscription (advanced)

  ARCHITECTURE:
  ┌──────────┐    ┌──────────────┐    ┌──────────┐
  │ Beats /  │───▶│ Elasticsearch│───▶│ Kibana   │
  │ Elastic  │    │ (Index/Store)│    │ (UI)     │
  │ Agent    │    └──────────────┘    └──────────┘
  └──────────┘         ↑
                  ┌────┴────┐
                  │ Logstash│ (optional pipeline)
                  └─────────┘

  QUERY EXAMPLES:
  # KQL in Kibana
  event.action: "logon-failed" and source.ip: 203.0.113.*
  
  # EQL (Event Query Language)
  sequence by source.ip with maxspan=5m
    [authentication where event.outcome == "failure"]
      with runs=10
    [authentication where event.outcome == "success"]
  
  # ES|QL
  FROM logs-* 
  | WHERE event.action == "logon-failed" 
  | STATS count = COUNT(*) BY source.ip 
  | WHERE count > 10

  STRENGTHS:
  ✓ Open source core (free)
  ✓ Powerful full-text search
  ✓ EQL for event sequences
  ✓ Built-in detection rules
  ✓ Elastic Agent (EDR + collection)
  ✓ ML anomaly detection
  ✓ Active community
  ✓ Flexible deployment options

  WEAKNESSES:
  ✗ Requires expertise to manage
  ✗ Cluster management complexity
  ✗ Advanced features require license
  ✗ Resource-intensive (memory/disk)
  ✗ Less turnkey than commercial options
```

---

## 4. Other SIEM Platforms

```
IBM QRADAR:
  Type: Commercial
  Query: AQL (Ariel Query Language)
  Strengths: Offense management, flow analysis, UEBA
  Weaknesses: Aging UI, complex licensing
  Best for: Large enterprises, existing IBM shops

WAZUH:
  Type: Open source
  Stack: Wazuh Manager + Elasticsearch + Kibana
  Strengths: Free, FIM, compliance, agent-based
  Weaknesses: Limited scalability, smaller ecosystem
  Best for: Budget-constrained, SMBs

GOOGLE CHRONICLE/SECOPS:
  Type: Cloud-native (Google Cloud)
  Query: YARA-L
  Strengths: Massive scale, 12-month retention, Google TI
  Weaknesses: Google ecosystem focus, newer platform
  Best for: Google Cloud shops, high-volume environments

LOGRHYTHM:
  Type: Commercial
  Strengths: Built-in SOAR, compliance modules
  Weaknesses: Market share declining
  Best for: Mid-market organizations

SECURONIX:
  Type: Cloud-native
  Strengths: Advanced UEBA, cloud-native
  Weaknesses: Newer platform
  Best for: Organizations focused on insider threats

SUMO LOGIC:
  Type: Cloud-native
  Strengths: Cloud-native, ML analytics
  Weaknesses: Limited security focus
  Best for: DevOps + security convergence
```

---

## 5. Platform Selection

```
SELECTION CRITERIA:

  Factor             | Weight | Question
  Budget             | High   | TCO over 3-5 years?
  Data volume        | High   | Expected GB/day?
  Cloud strategy     | High   | Cloud-first or on-prem?
  Existing ecosystem | Medium | Microsoft/Google/AWS shop?
  Team expertise     | Medium | SPL, KQL, or other skills?
  Integration needs  | Medium | What tools must integrate?
  Compliance         | Medium | Data residency requirements?
  Scalability        | Medium | Growth projections?
  Community/support  | Low    | Documentation, forums?
  Vendor stability   | Low    | Long-term viability?

COMPARISON MATRIX:
  Feature      | Splunk | Sentinel | Elastic | QRadar | Wazuh
  Cost         | $$$$   | $$$      | $ (OSS) | $$$$   | Free
  Scalability  | High   | Very High| High    | High   | Medium
  Cloud-native | Partial| Yes      | Partial | No     | No
  EDR included | No     | Defender | Yes     | No     | Yes
  SOAR built-in| Add-on | Yes      | No      | Add-on | No
  Free tier    | No     | Partial  | Yes     | No     | Yes
  Query lang   | SPL    | KQL      | Multiple| AQL    | Custom
  Learning     | Steep  | Medium   | Medium  | Steep  | Easy

DEPLOYMENT RECOMMENDATIONS:
  Scenario                    | Recommended
  Enterprise, budget OK       | Splunk ES
  Microsoft ecosystem         | Sentinel
  Budget-constrained          | Elastic or Wazuh
  Google Cloud                | Chronicle
  Need EDR + SIEM             | Elastic Security
  Compliance-focused          | LogRhythm or Splunk
  High-volume, cloud-native   | Sentinel or Chronicle
```

---

## Summary Table

| Platform | Type | Query Language | Best For |
|----------|------|---------------|----------|
| Splunk | Commercial | SPL | Enterprise, flexibility |
| Sentinel | Cloud-native | KQL | Microsoft shops |
| Elastic | Open source+ | Multiple | Budget, flexibility |
| QRadar | Commercial | AQL | IBM ecosystem |
| Wazuh | Open source | Custom | Free, compliance |
| Chronicle | Cloud-native | YARA-L | Google, scale |

---

## Revision Questions

1. How do Splunk SPL and Microsoft KQL compare for security queries?
2. What makes Elastic Security unique as an open-source SIEM option?
3. What factors are most important in SIEM platform selection?
4. When would you recommend Sentinel over Splunk?
5. What are the total cost considerations beyond licensing?

---

*Previous: [05-alerting-and-dashboards.md](05-alerting-and-dashboards.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
