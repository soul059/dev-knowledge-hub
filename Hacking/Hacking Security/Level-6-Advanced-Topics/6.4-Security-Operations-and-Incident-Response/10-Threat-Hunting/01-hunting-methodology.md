# Unit 10: Threat Hunting — Topic 1: Hunting Methodology

## Overview

**Threat hunting** is the proactive, analyst-driven process of searching for threats that have evaded existing detection mechanisms. Unlike reactive alerting, hunting assumes the adversary is already present and uses hypotheses, intelligence, and data analysis to find them. Threat hunting bridges the gap between automated detection and advanced persistent threats.

---

## 1. Threat Hunting Fundamentals

```
THREAT HUNTING DEFINED:

  REACTIVE SECURITY          PROACTIVE HUNTING
  ┌──────────────┐          ┌──────────────┐
  │ Wait for     │          │ Assume breach│
  │ alert        │          │ Search for   │
  │ Respond to   │          │ evidence     │
  │ detection    │          │ Validate or  │
  │              │          │ disprove     │
  └──────────────┘          └──────────────┘
  
  Detection-based            Hypothesis-based
  Automated                  Human-driven
  Known threats              Unknown threats
  After the fact             Proactive
  Alert fatigue              Focused investigation

WHY HUNT:
  → Detect threats automated tools miss
  → Reduce dwell time (avg 200+ days)
  → Find fileless/living-off-the-land attacks
  → Discover misconfigurations
  → Validate detection coverage
  → Improve overall security posture
  → Develop new detection rules
  → Build institutional knowledge

HUNTING MATURITY MODEL (HMM):
  Level 0: Initial
  → No routine data collection
  → Primarily reactive

  Level 1: Minimal
  → Some data collection
  → IOC-based searching only

  Level 2: Procedural
  → Follow procedures from others
  → Regular data collection
  → Some automation

  Level 3: Innovative
  → Create own hypotheses
  → Develop custom analytics
  → Threat intelligence driven

  Level 4: Leading
  → Automate successful hunts
  → Contribute to community
  → Continuous improvement
```

---

## 2. Hunting Process

```
HUNTING METHODOLOGY:

  ┌──────────────────────────────────────────┐
  │          THREAT HUNTING LOOP             │
  │                                          │
  │  ┌───────────┐                           │
  │  │ HYPOTHESIS│ What are we looking for?  │
  │  └─────┬─────┘                           │
  │        │                                 │
  │  ┌─────▼─────┐                           │
  │  │INVESTIGATE│ Search and analyze data   │
  │  └─────┬─────┘                           │
  │        │                                 │
  │  ┌─────▼─────┐                           │
  │  │  DISCOVER │ Identify findings         │
  │  └─────┬─────┘                           │
  │        │                                 │
  │  ┌─────▼──────┐                          │
  │  │ INFORM     │ Share findings, create   │
  │  │            │ new detections           │
  │  └─────┬──────┘                          │
  │        │                                 │
  │        └──────▶ (New hypothesis)         │
  └──────────────────────────────────────────┘

DETAILED PROCESS:
  1. TRIGGER
     → Threat intelligence report
     → Industry incident
     → ATT&CK technique review
     → Anomaly in data
     → Vulnerability disclosure
     → Regulatory requirement

  2. HYPOTHESIS
     → Specific, testable statement
     → Based on threat intel or TTP
     → Focused on detectable behavior
     → Example: "An attacker may be using
       PowerShell to download payloads
       from external hosts"

  3. DATA COLLECTION
     → Identify needed data sources
     → Verify data availability
     → Define time window
     → Query and collect data

  4. ANALYSIS
     → Search for hypothesis indicators
     → Statistical analysis
     → Anomaly detection
     → Behavioral analysis
     → Timeline correlation

  5. FINDINGS
     → Threat found → Incident response
     → No threat → Document and improve
     → Detection gap → Create new rule
     → Misconfiguration → Remediate

  6. OUTPUT
     → New detection rules
     → Updated playbooks
     → Threat intelligence
     → Security recommendations
     → Hunt documentation
```

---

## 3. Hunting Approaches

```
THREE HUNTING APPROACHES:

INTEL-DRIVEN HUNTING:
  → Start with threat intelligence
  → IOCs from reports, feeds, ISACs
  → Search for known adversary TTPs
  → Focus on threats targeting your industry
  
  Example: "APT group X targets healthcare with
  technique Y. Do we see evidence of Y?"

  Steps:
  1. Review threat intelligence
  2. Extract relevant IOCs and TTPs
  3. Search for IOCs in environment
  4. Analyze matches for context
  5. Expand scope if indicators found

HYPOTHESIS-DRIVEN HUNTING:
  → Start with a theory about attacker behavior
  → Based on ATT&CK, experience, or research
  → Test the hypothesis against data
  → Independent of specific threat intel
  
  Example: "Attackers may be using scheduled
  tasks for persistence. Let's analyze all
  recently created scheduled tasks."

  Steps:
  1. Formulate hypothesis
  2. Identify data needed to test
  3. Collect and analyze data
  4. Evaluate results
  5. Refine hypothesis if needed

BASELINE/ANOMALY-DRIVEN HUNTING:
  → Establish normal behavior baselines
  → Search for deviations from normal
  → Data-driven, statistical approach
  → Effective for insider threats
  
  Example: "What processes are running that
  have never been seen before in our
  environment?"

  Steps:
  1. Establish baseline of normal
  2. Identify statistical outliers
  3. Investigate anomalies
  4. Determine if malicious or benign
  5. Update baseline
```

---

## 4. ATT&CK-Based Hunting

```
MITRE ATT&CK FOR HUNTING:

  ATT&CK provides structured TTPs to hunt for:

  INITIAL ACCESS:
  → T1566: Search for phishing indicators
  → T1190: Search for exploitation attempts
  → T1133: External remote services audit

  EXECUTION:
  → T1059: Script interpreter analysis
    Hunt: Unusual PowerShell, Python, cmd usage
  → T1053: Scheduled task analysis
    Hunt: New/modified scheduled tasks

  PERSISTENCE:
  → T1547: Boot/logon autostart
    Hunt: New registry run keys, startup items
  → T1543: Create/modify system service
    Hunt: New services created

  CREDENTIAL ACCESS:
  → T1003: OS credential dumping
    Hunt: LSASS access, mimikatz patterns
  → T1110: Brute force
    Hunt: Failed login patterns

  LATERAL MOVEMENT:
  → T1021: Remote services
    Hunt: Unusual RDP, SSH, WinRM
  → T1570: Lateral tool transfer
    Hunt: File copies between systems

  EXFILTRATION:
  → T1048: Exfiltration over alt protocol
    Hunt: DNS tunneling, ICMP anomalies
  → T1041: Exfiltration over C2
    Hunt: Large data transfers to C2

HUNT PLANNING WITH ATT&CK:
  Technique | Data Source    | Hunt Query
  T1059.001 | Sysmon (EID 1)| PowerShell with 
            |               | download commands
  T1003.001 | Sysmon (EID 10)| Process accessing
            |                | lsass.exe
  T1053.005 | Sysmon (EID 1)| schtasks.exe with
            |               | /create parameter
```

---

## 5. Hunt Documentation

```
HUNT DOCUMENTATION:

HUNT PLAN:
  ┌──────────────────────────────────────────┐
  │ HUNT PLAN                               │
  │                                          │
  │ Hunt ID: HUNT-2024-015                  │
  │ Hunter: Jane Doe                         │
  │ Date: 2024-01-20                        │
  │                                          │
  │ HYPOTHESIS:                              │
  │ Attackers may be using PowerShell to     │
  │ download and execute payloads from       │
  │ external hosts, bypassing application    │
  │ whitelisting.                            │
  │                                          │
  │ TRIGGER:                                 │
  │ Recent industry report of APT using      │
  │ PowerShell download cradles              │
  │                                          │
  │ ATT&CK: T1059.001 (PowerShell)          │
  │                                          │
  │ DATA SOURCES:                            │
  │ → Sysmon Event ID 1 (Process Create)    │
  │ → PowerShell Script Block Logging       │
  │ → Proxy logs (outbound connections)      │
  │                                          │
  │ SCOPE:                                   │
  │ → All Windows endpoints                  │
  │ → Last 30 days                          │
  │                                          │
  │ QUERIES: [See attached]                 │
  │ TIMELINE: 4 hours estimated             │
  └──────────────────────────────────────────┘

HUNT RESULTS:
  ┌──────────────────────────────────────────┐
  │ HUNT RESULTS                             │
  │                                          │
  │ FINDING: No active threats detected     │
  │                                          │
  │ OBSERVATIONS:                            │
  │ → 15 instances of PowerShell downloads  │
  │ → All attributed to IT automation       │
  │ → 3 instances used deprecated methods   │
  │                                          │
  │ RECOMMENDATIONS:                         │
  │ → Create detection rule for PowerShell  │
  │   download from non-approved sources    │
  │ → Update IT scripts to use approved     │
  │   download methods                       │
  │ → Enable constrained language mode      │
  │                                          │
  │ NEW DETECTION RULE CREATED: Yes         │
  │ Rule ID: DET-2024-089                   │
  └──────────────────────────────────────────┘
```

---

## Summary Table

| Approach | Trigger | Strength | Best For |
|----------|---------|----------|---------|
| Intel-driven | Threat reports, IOCs | Specific, actionable | Known adversaries |
| Hypothesis-driven | ATT&CK, experience | Broad coverage | Unknown threats |
| Baseline/Anomaly | Data analysis | Insider threat | Unusual behavior |

---

## Revision Questions

1. How does threat hunting differ from traditional detection?
2. What are the steps in the threat hunting process?
3. What are the three main hunting approaches?
4. How does MITRE ATT&CK support threat hunting?
5. What should hunt documentation include?

---

*Previous: None (First topic in this unit) | Next: [02-hypothesis-development.md](02-hypothesis-development.md)*

---

*[Back to README](../README.md)*
