# Unit 2: SIEM Fundamentals — Topic 4: Correlation Rules

## Overview

**Correlation rules** are the intelligence engine of a SIEM, combining multiple events and conditions to detect complex threats that single events cannot reveal. Effective correlation separates actionable alerts from noise, reducing false positives while catching sophisticated attacks that span multiple systems and time periods.

---

## 1. Correlation Fundamentals

```
WHAT IS EVENT CORRELATION?

  Single Event: User logged in (Event ID 4624)
  → Not interesting by itself

  Correlated Events:
  → 50 failed logins (4625) from same IP in 5 minutes
  → Followed by 1 successful login (4624)
  → Followed by new service installed (7045)
  → FROM: Source not in known IP ranges
  = HIGH CONFIDENCE: Brute force + compromise

CORRELATION TYPES:

  1. RULE-BASED CORRELATION
     → If-then logic
     → Threshold-based
     → Time-window based
     → Most common type
     
  2. STATISTICAL CORRELATION
     → Baseline deviations
     → Anomaly detection
     → Standard deviation thresholds
     → Requires training period

  3. PATTERN-BASED CORRELATION
     → Sequence of events
     → Attack pattern matching
     → Kill chain correlation
     → MITRE ATT&CK mapping

  4. VULNERABILITY-BASED
     → Correlate attacks with known vulnerabilities
     → Asset vulnerability data + attack events
     → Prioritize based on exploitability

CORRELATION LOGIC:
  ┌──────────┐    ┌────────────┐    ┌──────────┐
  │ Event A  │───▶│ Correlation│───▶│  ALERT   │
  │ Event B  │───▶│   Engine   │    │          │
  │ Event C  │───▶│            │    │ Priority │
  │ Context  │───▶│ Rules +    │    │ Details  │
  │ Intel    │───▶│ Logic      │    │ Context  │
  └──────────┘    └────────────┘    └──────────┘
```

---

## 2. Rule Writing

```
CORRELATION RULE COMPONENTS:

  NAME: Descriptive rule name
  DESCRIPTION: What the rule detects
  SEVERITY: Critical/High/Medium/Low
  DATA SOURCES: Required log sources
  CONDITIONS: Logic for matching
  TIME WINDOW: Observation period
  THRESHOLD: Count/frequency trigger
  GROUPING: How to aggregate events
  SUPPRESSION: Avoid duplicate alerts
  ACTIONS: What to do when triggered
  ATT&CK MAPPING: Tactic/Technique IDs

SPLUNK (SPL) RULES:

  # Brute Force Detection
  index=windows sourcetype=WinEventLog:Security EventCode=4625
  | stats count as failed_attempts by src_ip, Account_Name
  | where failed_attempts > 10
  | join src_ip [search index=windows EventCode=4624
    | stats count by src_ip, Account_Name]

  # Impossible Travel
  index=auth action=success
  | iplocation src_ip
  | sort 0 user, _time
  | streamstats current=f last(Country) as prev_country,
    last(_time) as prev_time by user
  | eval time_diff=(_time-prev_time)/3600
  | where prev_country!=Country AND time_diff<2

KQL (MICROSOFT SENTINEL):

  // Brute Force Detection
  SecurityEvent
  | where EventID == 4625
  | summarize FailedAttempts=count() by SourceIP=IpAddress,
    TargetAccount=TargetUserName, bin(TimeGenerated, 5m)
  | where FailedAttempts > 10
  | join kind=inner (
      SecurityEvent
      | where EventID == 4624
      | project SourceIP=IpAddress, TargetAccount=TargetUserName
  ) on SourceIP, TargetAccount

SIGMA RULES (UNIVERSAL):

  title: Brute Force Attack
  status: stable
  description: Detects brute force login attempts
  logsource:
    product: windows
    service: security
  detection:
    selection:
      EventID: 4625
    condition: selection | count(TargetUserName) by IpAddress > 10
    timeframe: 5m
  level: high
  tags:
    - attack.credential_access
    - attack.t1110
```

---

## 3. Common Correlation Rules

```
ESSENTIAL CORRELATION RULES:

AUTHENTICATION:
  → Brute force (multiple failed + success)
  → Password spraying (1-2 fails per many accounts)
  → Impossible travel (geographic anomaly)
  → Off-hours authentication
  → Service account interactive logon
  → Multiple account lockouts
  → Dormant account reactivation

PRIVILEGE ESCALATION:
  → New admin account created
  → User added to privileged group
  → Privilege assigned outside change window
  → Service account used interactively
  → sudo abuse on Linux
  → Sensitive group membership change

LATERAL MOVEMENT:
  → Multiple host authentication in short time
  → Admin tool execution (PsExec, WMI, WinRM)
  → RDP to unusual destinations
  → Pass-the-hash indicators
  → SMB access to multiple hosts

MALWARE / EXECUTION:
  → Known malware hash detected
  → Process spawning from unusual parent
  → Encoded command execution (PowerShell -enc)
  → Suspicious script execution
  → Office application spawning cmd/PowerShell
  → DLL side-loading indicators

DATA EXFILTRATION:
  → Large data transfer to external
  → DNS tunneling (long DNS queries)
  → Upload to cloud storage
  → Email with large attachments to external
  → USB device activity + file copy

DEFENSE EVASION:
  → Audit log cleared (1102)
  → Security tool disabled
  → Firewall rule modified
  → EDR agent stopped
  → Sysmon service stopped
  → Timestomping detected

RULE PRIORITY MATRIX:
  Priority | Rule Type              | Target MTTD
  P1       | Known attack patterns  | < 15 min
  P2       | Behavioral anomalies   | < 1 hour
  P3       | Policy violations      | < 4 hours
  P4       | Informational          | Daily review
```

---

## 4. Rule Tuning

```
TUNING METHODOLOGY:

WHY TUNE?
  → Reduce false positives
  → Improve signal-to-noise ratio
  → Prevent alert fatigue
  → Ensure actionable alerts
  → Maintain analyst trust in alerts

TUNING PROCESS:
  1. Deploy rule in logging/test mode
  2. Review triggered alerts (1-2 weeks)
  3. Identify false positive patterns
  4. Adjust thresholds, exclusions
  5. Move to alerting mode
  6. Continue monitoring accuracy
  7. Regular quarterly review

TUNING TECHNIQUES:
  → Adjust thresholds (count, time window)
  → Add exclusions (known safe IPs, service accounts)
  → Refine data source filters
  → Add context conditions
  → Whitelist known behaviors
  → Reduce severity for noisy rules
  → Combine with additional conditions

EXAMPLE TUNING:
  Original Rule:
  → "Alert on any failed login"
  → Result: 10,000 alerts/day (unusable)

  Tuned Rule:
  → "Alert when > 10 failed logins from same IP
     in 5 minutes, exclude VPN concentrator IP,
     exclude locked-out accounts,
     followed by successful login"
  → Result: 5-10 alerts/day (actionable)

FALSE POSITIVE MANAGEMENT:
  Category           | Action
  Known safe activity| Add to whitelist
  Expected behavior  | Adjust threshold
  Incomplete context | Add enrichment
  Wrong data source  | Fix source filter
  Design flaw        | Rewrite rule

TUNING METRICS:
  → False positive rate per rule
  → Alert volume per rule
  → Analyst feedback per rule
  → Time to investigate per alert
  → Rules with 0 true positives (stale)
  → Rules never triggered (validate)
```

---

## 5. Advanced Correlation

```
ADVANCED TECHNIQUES:

CHAIN RULES:
  → Detect attack sequences
  → Event A within X minutes of Event B
  → Models kill chain progression
  
  Example: Phishing Attack Chain
  1. Email with attachment received
  2. Attachment opened (process creation)
  3. Outbound connection to new domain
  4. PowerShell encoded command
  5. Credential access attempt
  → Alert: "Possible phishing compromise"

RISK-BASED ALERTING:
  → Assign risk scores to events
  → Accumulate risk per entity (user/host)
  → Alert when threshold exceeded
  → Reduces alert volume dramatically
  
  Event                    | Risk Score
  Failed login             | +5
  Login from new country   | +20
  Admin tool execution     | +15
  Large data download      | +25
  Security tool disabled   | +30
  Alert threshold          | 100

  Example:
  User jsmith:
  → Failed login (+5)
  → Login new country (+20)
  → Admin tool (+15)
  → Data download (+25)
  → Total: 65 (below threshold)
  → Security tool disabled (+30)
  → Total: 95 → ALERT!

LOOKBACK CORRELATION:
  → Compare current events to historical baselines
  → "First time seen" rules
  → Unusual behavior for this user/system
  → Requires historical data analysis

THREAT INTEL CORRELATION:
  → Match events against IOC feeds
  → IP, domain, hash, URL matching
  → Automated enrichment
  → Prioritize based on intelligence confidence
```

---

## Summary Table

| Correlation Type | Complexity | False Positive Risk | Detection Capability |
|-----------------|-----------|-------------------|---------------------|
| Single event + threshold | Low | High | Basic attacks |
| Multi-event sequence | Medium | Medium | Attack chains |
| Statistical/baseline | High | Medium | Anomalies |
| Risk-based scoring | High | Low | Complex attacks |
| Threat intel matching | Low | Low | Known threats |

---

## Revision Questions

1. What are the different types of event correlation in SIEM?
2. How does a Sigma rule differ from vendor-specific detection rules?
3. What is the process for tuning correlation rules?
4. How does risk-based alerting reduce alert fatigue?
5. What correlation rules should every SOC implement first?

---

*Previous: [03-normalization-and-parsing.md](03-normalization-and-parsing.md) | Next: [05-alerting-and-dashboards.md](05-alerting-and-dashboards.md)*

---

*[Back to README](../README.md)*
