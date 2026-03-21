# Unit 1: Security Operations Fundamentals — Topic 5: Tool Stack Overview

## Overview

The SOC **tool stack** comprises the integrated set of technologies used for monitoring, detection, investigation, response, and reporting. A well-designed tool stack reduces analyst workload, improves detection, and accelerates response. This topic covers the essential security tools, their integration, and how to build an effective SOC technology ecosystem.

---

## 1. Core SOC Technology Stack

```
SOC TOOL ECOSYSTEM:

  ┌──────────────────────────────────────────────────┐
  │                SOC TECHNOLOGY STACK                │
  │                                                    │
  │  LAYER 1: DATA COLLECTION                         │
  │  ┌───────────────────────────────────────────────┐│
  │  │ Log Collectors │ Agents │ Taps │ API Feeds    ││
  │  │ Syslog │ Beats │ Fluentd │ NXLog │ Cribl     ││
  │  └───────────────────────────────────────────────┘│
  │                                                    │
  │  LAYER 2: SIEM / DATA PLATFORM                    │
  │  ┌───────────────────────────────────────────────┐│
  │  │ Splunk │ Elastic │ Sentinel │ QRadar │ Wazuh  ││
  │  │ Normalization │ Indexing │ Storage │ Search    ││
  │  └───────────────────────────────────────────────┘│
  │                                                    │
  │  LAYER 3: DETECTION & ANALYTICS                   │
  │  ┌───────────────────────────────────────────────┐│
  │  │ Correlation Rules │ UEBA │ ML │ Threat Intel  ││
  │  │ Sigma │ YARA │ Snort/Suricata │ Custom Rules  ││
  │  └───────────────────────────────────────────────┘│
  │                                                    │
  │  LAYER 4: ENDPOINT & NETWORK                      │
  │  ┌───────────────────────────────────────────────┐│
  │  │ EDR: CrowdStrike │ SentinelOne │ Defender     ││
  │  │ NDR: Zeek │ Darktrace │ ExtraHop │ Corelight  ││
  │  │ FW/IPS: Palo Alto │ Fortinet │ Snort          ││
  │  └───────────────────────────────────────────────┘│
  │                                                    │
  │  LAYER 5: RESPONSE & ORCHESTRATION                │
  │  ┌───────────────────────────────────────────────┐│
  │  │ SOAR: Splunk SOAR │ XSOAR │ Shuffle │ Tines  ││
  │  │ Case Mgmt: TheHive │ ServiceNow │ Jira       ││
  │  └───────────────────────────────────────────────┘│
  │                                                    │
  │  LAYER 6: INTELLIGENCE & ENRICHMENT               │
  │  ┌───────────────────────────────────────────────┐│
  │  │ MISP │ OTX │ VirusTotal │ AbuseIPDB │ Shodan ││
  │  │ ThreatConnect │ Recorded Future │ TAXII       ││
  │  └───────────────────────────────────────────────┘│
  └──────────────────────────────────────────────────┘
```

---

## 2. SIEM Platforms

```
SIEM COMPARISON:

SPLUNK ENTERPRISE SECURITY:
  Type: Commercial (market leader)
  Strengths:
  → Powerful search language (SPL)
  → Massive scalability
  → Rich app ecosystem
  → Strong community
  → Advanced analytics
  Weaknesses:
  → Expensive (data volume licensing)
  → Complex administration
  → Heavy resource requirements

MICROSOFT SENTINEL:
  Type: Cloud-native (Azure)
  Strengths:
  → Cloud-native, serverless
  → Pay-per-use pricing
  → Native Microsoft integration
  → KQL query language
  → Built-in SOAR
  → Threat intelligence integration
  Weaknesses:
  → Azure-centric
  → Learning curve for KQL
  → Costs can grow quickly

ELASTIC SECURITY:
  Type: Open source + commercial
  Strengths:
  → Open source core
  → Free tier available
  → Powerful search (Lucene/EQL)
  → Built-in EDR (Elastic Agent)
  → Detection rules included
  → Flexible deployment
  Weaknesses:
  → Requires expertise to manage
  → Scaling can be complex
  → Less mature SIEM features

IBM QRADAR:
  Type: Commercial
  Strengths:
  → Strong correlation engine
  → Built-in UEBA
  → Flow analysis
  → Offense management
  Weaknesses:
  → Complex licensing
  → Slower development pace
  → Aging interface

WAZUH:
  Type: Open source
  Strengths:
  → Free and open source
  → Built-in FIM, vulnerability detection
  → Compliance monitoring
  → Agent-based
  Weaknesses:
  → Limited scalability
  → Smaller community
  → Fewer integrations
```

---

## 3. Endpoint and Network Tools

```
ENDPOINT DETECTION AND RESPONSE (EDR):

  Tool              | Type       | Key Feature
  CrowdStrike       | Commercial | Cloud-native, threat intel
  SentinelOne        | Commercial | AI-driven, autonomous
  Microsoft Defender | Commercial | Built-in Windows integration
  Carbon Black      | Commercial | VMware ecosystem
  Elastic Agent     | Open source| Free EDR capabilities
  Wazuh Agent       | Open source| FIM, vuln detection
  Velociraptor      | Open source| Hunting and forensics

  EDR CAPABILITIES:
  → Process monitoring
  → File activity tracking
  → Network connection logging
  → Behavior analysis
  → Threat detection
  → Automated response (isolation)
  → Forensic data collection
  → Live query/response

NETWORK DETECTION AND RESPONSE (NDR):

  Tool        | Type       | Key Feature
  Zeek (Bro)  | Open source| Protocol analysis
  Suricata    | Open source| IDS/IPS rules
  Darktrace   | Commercial | AI-driven anomaly
  ExtraHop    | Commercial | Wire data analytics
  Corelight   | Commercial | Enterprise Zeek

  NDR CAPABILITIES:
  → Network traffic analysis
  → Protocol decoding
  → Anomaly detection
  → Lateral movement detection
  → Data exfiltration detection
  → Encrypted traffic analysis
  → PCAP capture/analysis

FIREWALL / IPS:
  → Palo Alto Networks (NGFW leader)
  → Fortinet FortiGate
  → Check Point
  → Cisco Firepower
  → pfSense / OPNsense (open source)
```

---

## 4. SOAR and Case Management

```
SECURITY ORCHESTRATION, AUTOMATION, RESPONSE (SOAR):

  WHAT SOAR DOES:
  → Automates repetitive tasks
  → Orchestrates tool integration
  → Executes response playbooks
  → Reduces analyst workload
  → Standardizes processes
  → Tracks case management

  SOAR PLATFORMS:
  Platform         | Type       | Notes
  Splunk SOAR      | Commercial | Formerly Phantom
  Palo Alto XSOAR  | Commercial | Formerly Demisto
  Shuffle          | Open source| Web-based, simple
  Tines            | Commercial | No-code automation
  Swimlane         | Commercial | Low-code SOAR
  TheHive + Cortex | Open source| Case mgmt + enrichment

  COMMON SOAR USE CASES:
  → Phishing email triage
    • Extract URLs/attachments
    • Check reputation (VT, URLhaus)
    • Quarantine email
    • Block indicators
    • Create ticket
  
  → Malware alert response
    • Isolate endpoint (EDR)
    • Collect forensic data
    • Check file hash (VT)
    • Block hash on all endpoints
    • Create incident ticket
  
  → Account compromise
    • Disable account
    • Revoke sessions
    • Reset credentials
    • Check for lateral movement
    • Notify user and manager

  AUTOMATION ROI:
  Without SOAR: 45 min per phishing alert
  With SOAR: 5 min per phishing alert
  Volume: 50 phishing alerts/day
  Savings: 33 analyst-hours/day
```

---

## 5. Tool Stack Design

```
DESIGNING THE SOC TOOL STACK:

SELECTION CRITERIA:
  → Integration capabilities
  → Scalability
  → Total cost of ownership
  → Vendor support and community
  → Ease of use and learning curve
  → API availability
  → Deployment model (cloud/on-prem)
  → Compliance requirements

BUDGET TIERS:

BUDGET SOC (Open Source):
  → SIEM: Wazuh or Elastic (free)
  → EDR: Wazuh Agent + Velociraptor
  → NDR: Zeek + Suricata
  → SOAR: Shuffle or TheHive/Cortex
  → Threat Intel: MISP + OTX
  → Ticketing: TheHive
  → Cost: Staff only (~$300K-$500K/year)

MID-RANGE SOC:
  → SIEM: Elastic Security or Sentinel
  → EDR: Microsoft Defender or SentinelOne
  → NDR: Zeek/Corelight
  → SOAR: Shuffle or commercial trial
  → Threat Intel: Commercial feed + MISP
  → Ticketing: ServiceNow or Jira
  → Cost: $500K-$1.5M/year

ENTERPRISE SOC:
  → SIEM: Splunk ES or Sentinel
  → EDR: CrowdStrike or SentinelOne
  → NDR: Darktrace or ExtraHop
  → SOAR: Splunk SOAR or XSOAR
  → XDR: Vendor-integrated platform
  → Threat Intel: Recorded Future
  → Ticketing: ServiceNow SecOps
  → Cost: $2M-$5M+/year

TOOL INTEGRATION:
  ┌─────────┐     ┌──────┐     ┌──────┐
  │   EDR   │────▶│ SIEM │────▶│ SOAR │
  └─────────┘     └──┬───┘     └──┬───┘
  ┌─────────┐        │            │
  │   NDR   │────────┘            │
  └─────────┘                     │
  ┌─────────┐                     │
  │ Firewall│─────────────────────┘
  └─────────┘        │
  ┌─────────┐     ┌──▼───────┐
  │  Intel  │────▶│ Ticketing│
  └─────────┘     └──────────┘

  Integration methods:
  → API-based (REST/GraphQL)
  → Syslog forwarding
  → File-based (CSV, JSON)
  → Webhook triggers
  → STIX/TAXII for threat intel
  → CEF/LEEF log formats
```

---

## Summary Table

| Tool Category | Open Source | Commercial | Purpose |
|--------------|------------|------------|---------|
| SIEM | Wazuh, Elastic | Splunk, Sentinel, QRadar | Log analysis, correlation |
| EDR | Velociraptor, Wazuh | CrowdStrike, SentinelOne | Endpoint monitoring |
| NDR | Zeek, Suricata | Darktrace, ExtraHop | Network analysis |
| SOAR | Shuffle, TheHive | Splunk SOAR, XSOAR | Automation, orchestration |
| Threat Intel | MISP, OTX | Recorded Future | Intelligence management |
| Ticketing | TheHive | ServiceNow | Case management |

---

## Revision Questions

1. What are the layers of a SOC technology stack?
2. How do SIEM platforms differ in their approach and pricing?
3. What is the difference between EDR and NDR?
4. How does SOAR reduce analyst workload and what are common use cases?
5. How should organizations design their tool stack based on budget?

---

*Previous: [04-soc-metrics.md](04-soc-metrics.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*
