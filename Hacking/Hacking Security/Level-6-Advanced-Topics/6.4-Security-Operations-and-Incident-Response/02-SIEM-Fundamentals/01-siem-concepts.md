# Unit 2: SIEM Fundamentals — Topic 1: SIEM Concepts

## Overview

**Security Information and Event Management (SIEM)** is the cornerstone technology of modern security operations. SIEM systems aggregate log data from across the IT environment, normalize it into a common format, correlate events to identify threats, and provide dashboards and alerts for security analysts. Understanding SIEM architecture, capabilities, and deployment models is essential for effective security monitoring.

---

## 1. What is SIEM?

```
SIEM DEFINITION:

  Security Information and Event Management
  = SIM (Security Information Management)
    + SEM (Security Event Management)

  SIM: Log collection, storage, reporting, compliance
  SEM: Real-time monitoring, correlation, alerting

  ┌────────────────────────────────────────────────┐
  │                 SIEM PLATFORM                   │
  │                                                  │
  │  ┌─────────────┐    ┌──────────────┐            │
  │  │  COLLECT     │───▶│  NORMALIZE   │            │
  │  │  Logs from   │    │  Common      │            │
  │  │  all sources │    │  format      │            │
  │  └─────────────┘    └──────┬───────┘            │
  │                            │                     │
  │  ┌─────────────┐    ┌─────▼────────┐            │
  │  │  ALERT      │◀───│  CORRELATE   │            │
  │  │  Notify     │    │  Rules &     │            │
  │  │  analysts   │    │  patterns    │            │
  │  └─────────────┘    └──────────────┘            │
  │                                                  │
  │  ┌─────────────┐    ┌──────────────┐            │
  │  │  STORE      │    │  REPORT      │            │
  │  │  Index &    │    │  Dashboards  │            │
  │  │  retain     │    │  compliance  │            │
  │  └─────────────┘    └──────────────┘            │
  └────────────────────────────────────────────────┘

CORE SIEM FUNCTIONS:
  1. Log Collection: Aggregate logs from all sources
  2. Normalization: Convert to common format/schema
  3. Indexing: Store and index for fast searching
  4. Correlation: Match events against rules
  5. Alerting: Generate alerts for analysts
  6. Dashboards: Visual security monitoring
  7. Reporting: Compliance and operational reports
  8. Investigation: Search and query capabilities
  9. Retention: Long-term log storage
```

---

## 2. SIEM Architecture

```
SIEM ARCHITECTURE PATTERNS:

ON-PREMISES SIEM:
  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │ Endpoint │  │ Network  │  │ Server   │
  │ Logs     │  │ Logs     │  │ Logs     │
  └────┬─────┘  └────┬─────┘  └────┬─────┘
       │              │              │
  ┌────▼──────────────▼──────────────▼─────┐
  │         LOG COLLECTORS / FORWARDERS     │
  │  Syslog │ Beats │ Agents │ API │ WEF   │
  └────────────────────┬───────────────────┘
                       │
  ┌────────────────────▼───────────────────┐
  │            SIEM CORE                    │
  │  ┌──────────┐  ┌────────────────────┐  │
  │  │ Parser / │  │ Correlation Engine │  │
  │  │ Normalize│  │ Rule Evaluation    │  │
  │  └──────────┘  └────────────────────┘  │
  │  ┌──────────┐  ┌────────────────────┐  │
  │  │ Indexer  │  │ Alert Manager      │  │
  │  │ Storage  │  │ Dashboard Engine   │  │
  │  └──────────┘  └────────────────────┘  │
  └────────────────────────────────────────┘

CLOUD-NATIVE SIEM:
  → Serverless / managed infrastructure
  → Pay-per-use pricing
  → Elastic scalability
  → Built-in integrations
  → Example: Microsoft Sentinel, Chronicle

HYBRID SIEM:
  → On-prem collection, cloud analytics
  → Edge processing for compliance
  → Cloud storage for scale
  → Distributed deployment

KEY METRICS:
  → Events Per Second (EPS): Ingestion rate
  → Gigabytes Per Day (GB/day): Data volume
  → Search Response Time: Query performance
  → Retention Period: How long logs are kept
  → Uptime: System availability
```

---

## 3. Data Sources

```
SIEM DATA SOURCES:

ENDPOINT SOURCES:
  → Windows Event Logs (Security, System, Application)
  → Sysmon (enhanced Windows logging)
  → Linux audit logs (auditd)
  → macOS Unified Logging
  → EDR telemetry
  → Antivirus/Antimalware logs
  → Application logs
  → PowerShell script block logging
  → File Integrity Monitoring

NETWORK SOURCES:
  → Firewall logs (allow/deny)
  → IDS/IPS alerts (Snort, Suricata)
  → Proxy/Web Gateway logs
  → DNS query logs
  → DHCP logs
  → NetFlow / sFlow
  → VPN connection logs
  → Load balancer logs
  → WAF logs

IDENTITY SOURCES:
  → Active Directory logs
  → Azure AD sign-in / audit
  → RADIUS/TACACS logs
  → SSO/MFA logs
  → Privileged access management
  → Certificate authority logs

CLOUD SOURCES:
  → AWS CloudTrail
  → Azure Activity Log
  → GCP Audit Logs
  → Cloud provider security alerts
  → SaaS application logs (O365, GSuite)
  → Container/Kubernetes audit logs

APPLICATION SOURCES:
  → Web server logs (Apache, Nginx, IIS)
  → Database audit logs
  → Custom application logs
  → API gateway logs
  → Email security logs

PRIORITY ORDER:
  Priority | Source                    | Reason
  1        | Authentication logs       | Identity compromise
  2        | Firewall/Network          | Network attacks
  3        | Endpoint (EDR/Sysmon)     | Endpoint compromise
  4        | DNS logs                  | C2 detection
  5        | Email security            | Phishing detection
  6        | Cloud audit logs          | Cloud attacks
  7        | Application logs          | App-layer attacks
  8        | Vulnerability scanners    | Risk context
```

---

## 4. SIEM Deployment

```
SIEM DEPLOYMENT CONSIDERATIONS:

SIZING:
  EPS       | Organization Size | Infrastructure
  500-2000  | Small (<500 users)| Single server
  2000-10K  | Medium            | Clustered
  10K-50K   | Large             | Distributed
  50K+      | Enterprise        | Multi-tier

DEPLOYMENT STEPS:
  1. PLANNING
     → Define use cases and objectives
     → Identify critical log sources
     → Calculate expected EPS/GB
     → Size infrastructure
     → Plan retention requirements

  2. INFRASTRUCTURE
     → Deploy SIEM servers/cluster
     → Configure storage
     → Set up networking
     → Deploy log collectors
     → Configure high availability

  3. LOG ONBOARDING
     → Connect critical sources first
     → Validate log format/parsing
     → Verify field extraction
     → Test data flow
     → Document source configuration

  4. CONTENT CREATION
     → Create detection rules
     → Build dashboards
     → Configure alerts
     → Set up reports
     → Define notification channels

  5. OPERATIONS
     → Train analysts
     → Establish monitoring procedures
     → Define escalation paths
     → Set up maintenance schedule
     → Plan capacity monitoring

COMMON PITFALLS:
  → Collecting too much data (cost overrun)
  → Collecting too little data (gaps)
  → Not normalizing/parsing properly
  → Too many rules (alert fatigue)
  → Too few rules (missed threats)
  → Not tuning rules
  → Insufficient storage planning
  → Not testing detection rules
```

---

## 5. SIEM Use Cases

```
COMMON SIEM USE CASES:

AUTHENTICATION MONITORING:
  → Brute force detection
  → Impossible travel (geo-impossible logins)
  → Off-hours authentication
  → Service account abuse
  → Credential stuffing detection
  → MFA bypass attempts
  → Dormant account usage

THREAT DETECTION:
  → Malware execution
  → Command and control communication
  → Lateral movement
  → Privilege escalation
  → Data exfiltration
  → Ransomware behavior
  → Living-off-the-land attacks

NETWORK SECURITY:
  → Unauthorized network access
  → Port scanning detection
  → DNS tunneling
  → Beaconing behavior
  → Policy violations
  → Shadow IT discovery

COMPLIANCE:
  → Audit trail maintenance
  → Access review evidence
  → Change monitoring
  → Policy compliance checking
  → Regulatory reporting
  → Data retention compliance

INSIDER THREAT:
  → Abnormal data access
  → Unusual working hours
  → Mass file downloads
  → USB device usage
  → Email to personal accounts
  → Resignation + data access correlation
```

---

## Summary Table

| SIEM Component | Function | Key Consideration |
|---------------|----------|-------------------|
| Collection | Gather logs | Coverage, format, EPS |
| Normalization | Common format | Field mapping, parsing |
| Correlation | Pattern matching | Rules, accuracy |
| Alerting | Notify analysts | Tuning, false positives |
| Storage | Retain data | Capacity, retention |
| Search | Investigation | Speed, query language |
| Reporting | Compliance/ops | Templates, scheduling |

---

## Revision Questions

1. What are the core functions of a SIEM platform?
2. How does SIEM architecture differ between on-premises and cloud-native?
3. What are the most important data sources to connect first?
4. What factors determine SIEM sizing?
5. What are common SIEM deployment pitfalls?

---

*Previous: None (First topic in this unit) | Next: [02-log-collection.md](02-log-collection.md)*

---

*[Back to README](../README.md)*
