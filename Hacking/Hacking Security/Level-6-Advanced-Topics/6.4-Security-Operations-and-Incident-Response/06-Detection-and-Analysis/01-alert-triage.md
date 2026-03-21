# Unit 6: Detection and Analysis — Topic 1: Alert Triage

## Overview

**Alert triage** is the process of rapidly evaluating security alerts to determine their validity, severity, and required response. Effective triage is the first line of defense in the SOC — it separates genuine threats from noise and ensures critical incidents receive immediate attention.

---

## 1. Triage Process

```
TRIAGE WORKFLOW:

  ┌──────────┐
  │NEW ALERT │
  └────┬─────┘
       │
  ┌────▼──────────┐
  │ 1. READ ALERT │ Review title, severity, source
  └────┬──────────┘
       │
  ┌────▼──────────┐
  │ 2. CONTEXT    │ Check enrichment, asset info
  └────┬──────────┘
       │
  ┌────▼──────────┐
  │ 3. VALIDATE   │ Is it real? Check source data
  └────┬──────────┘
       │
  ┌────▼──────────────┐     ┌──────────┐
  │ 4. CLASSIFY       │────▶│ FALSE    │──▶ Close + tune
  │    TP / FP / BTP? │     │ POSITIVE │
  └────┬──────────────┘     └──────────┘
       │ True Positive
  ┌────▼──────────┐
  │ 5. PRIORITIZE │ Severity + asset criticality
  └────┬──────────┘
       │
  ┌────▼──────────────────────┐
  │ 6. ACTION                  │
  │  → Low: Handle at Tier 1   │
  │  → Medium: Investigate T1  │
  │  → High: Escalate to T2    │
  │  → Critical: IR team + Mgr │
  └────────────────────────────┘

TRIAGE DECISION:
  TP  = True Positive (real threat)
  FP  = False Positive (not a threat)
  BTP = Benign True Positive (real but expected/authorized)

TRIAGE TIME TARGETS:
  Severity | Triage Time | Action
  Critical | < 5 minutes | Immediate escalation
  High     | < 15 minutes| Prioritized investigation
  Medium   | < 30 minutes| Standard investigation
  Low      | < 1 hour    | Batch review acceptable
```

---

## 2. Triage Techniques

```
QUICK TRIAGE CHECKS:

CHECK 1: ALERT CONTEXT
  → What detection rule triggered?
  → What is the source system?
  → What severity was assigned?
  → Has this rule fired before?
  → What ATT&CK technique does it map to?

CHECK 2: ENTITY REPUTATION
  → Source IP: Internal or external?
  → If external: Known scanner? VPN? CDN? Malicious?
  → User: Normal user or service account?
  → Host: Production server or test system?
  → File hash: Known good/bad on VirusTotal?

CHECK 3: HISTORICAL CONTEXT
  → Has this same alert fired for this entity before?
  → Is there an existing incident for this activity?
  → Is this a known false positive pattern?
  → Is there an active change window?

CHECK 4: CORRELATION
  → Are there related alerts?
  → Does this fit a larger pattern?
  → Are other entities showing similar activity?
  → Does timing correlate with other events?

CHECK 5: BUSINESS CONTEXT
  → Is the affected system critical?
  → Is this during a maintenance window?
  → Is the user in a sensitive role?
  → Are there compliance implications?

QUICK LOOKUP TOOLS:
  → VirusTotal: Hash/IP/URL reputation
  → AbuseIPDB: IP abuse reports
  → GreyNoise: Known scanner identification
  → Shodan: Service/port information
  → WHOIS: Domain registration details
  → Internal CMDB: Asset classification
```

---

## 3. Common Alert Types

```
ALERT TRIAGE BY TYPE:

AUTHENTICATION ALERTS:
  Alert: Multiple Failed Logins
  Triage:
  → Check source IP reputation
  → Internal or external source?
  → One user or many users?
  → Followed by success?
  → VPN or RDP?
  → Known password spray pattern?

  TP Indicators: External IP, multiple users, success after
  FP Indicators: User forgot password, account lockout policy

MALWARE ALERTS:
  Alert: Malware Detected on Endpoint
  Triage:
  → Was it quarantined/blocked?
  → What was the detection (file, behavior)?
  → Check hash on VT
  → Was it executed?
  → What is the malware family?
  → Is the system critical?

  TP Indicators: Execution, network callback, lateral movement
  FP Indicators: Known installer, AV FP, test tool

NETWORK ALERTS:
  Alert: Connection to Known C2 IP
  Triage:
  → Verify the TI source and age
  → Check if IP is shared hosting/CDN
  → Was connection successful?
  → What process initiated?
  → Any data transferred?

  TP Indicators: Outbound connection, process is suspicious
  FP Indicators: CDN IP, old TI data, legitimate service

EMAIL ALERTS:
  Alert: Phishing Email Detected
  Triage:
  → Was it delivered or quarantined?
  → Did anyone click/open?
  → Check URL/attachment reputation
  → Is it targeted (spearphishing)?
  → Are there multiple recipients?

  TP Indicators: Credential harvesting, malware attachment
  FP Indicators: Marketing email, internal test
```

---

## 4. Triage Documentation

```
TRIAGE DOCUMENTATION:

TICKET TEMPLATE:
  Alert ID: ALR-2024-12345
  Detection Rule: Brute Force - Multiple Failed Logins
  Severity: HIGH
  Time: 2024-01-15 10:30 UTC
  
  Source: 203.0.113.50 (External, Russia)
  Target: admin@company.com on dc01
  Details: 47 failed logins in 5 minutes
  
  TRIAGE ANALYSIS:
  → Source IP: Not in any known feed, GreyNoise unknown
  → Target: Domain Admin account
  → Timing: Outside business hours
  → Correlation: No prior alerts for this IP
  → Follow-up: Successful login at 10:35
  
  VERDICT: TRUE POSITIVE
  → Brute force followed by successful logon
  → Domain admin account targeted
  → Escalating to Tier 2 as HIGH priority
  
  INITIAL ACTIONS:
  → Disabled admin account
  → Blocked source IP at firewall
  → Notified SOC Manager
  
  Assigned to: Tier 2 Analyst (Jane)
  Ticket: INC-2024-001

DOCUMENTATION BEST PRACTICES:
  → Be concise but complete
  → Include evidence of analysis
  → State your reasoning
  → Document actions taken
  → Note any uncertainty
  → Include timestamps
  → Reference relevant IOCs
```

---

## 5. Improving Triage Efficiency

```
TRIAGE OPTIMIZATION:

AUTOMATION:
  → Auto-enrich alerts with context
  → Auto-close known FP patterns
  → Auto-escalate critical patterns
  → SOAR playbooks for common alerts
  → Pre-computed risk scores

ANALYST TOOLS:
  → Custom dashboards per alert type
  → One-click lookups (VT, AbuseIPDB)
  → Entity summary views
  → Related alert grouping
  → Historical alert context

KNOWLEDGE BASE:
  → Document common FP patterns
  → Alert-specific triage guides
  → Known-good baselines
  → Lookup quick references
  → Decision trees for common scenarios

METRICS:
  Metric                  | Target
  Average triage time     | < 15 minutes
  FP identification rate  | > 90% accuracy
  Escalation accuracy     | > 95%
  Alerts triaged per shift| 50-75
  Backlog age             | < 24 hours

REDUCING ALERT FATIGUE:
  [ ] Tune rules with high FP rates
  [ ] Consolidate duplicate alerts
  [ ] Auto-close low-value alerts
  [ ] Prioritize by risk score
  [ ] Rotate alert types among analysts
  [ ] Provide adequate breaks
  [ ] Celebrate wins (real detections)
```

---

## Summary Table

| Triage Step | Action | Time Target |
|------------|--------|-------------|
| Read alert | Review title, severity, source | 1 minute |
| Check context | Enrichment, asset info | 2 minutes |
| Validate | Verify with source data | 5 minutes |
| Classify | TP / FP / BTP | 2 minutes |
| Prioritize | Severity + asset criticality | 1 minute |
| Act | Handle, investigate, or escalate | 5 minutes |

---

## Revision Questions

1. What are the key steps in the alert triage process?
2. How do you differentiate between true positive, false positive, and benign true positive?
3. What quick checks help validate an authentication alert?
4. What should triage documentation include?
5. How can SOAR improve triage efficiency?

---

*Previous: None (First topic in this unit) | Next: [02-initial-analysis.md](02-initial-analysis.md)*

---

*[Back to README](../README.md)*
