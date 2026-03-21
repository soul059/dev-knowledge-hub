# Unit 3: Threat Detection — Topic 5: MITRE ATT&CK for Detection

## Overview

The **MITRE ATT&CK** framework is the industry-standard knowledge base of adversary tactics, techniques, and procedures (TTPs) used by security teams to organize detection coverage, map threats, and prioritize security investments. ATT&CK transforms detection from ad-hoc rule creation into a structured, comprehensive approach.

---

## 1. ATT&CK Framework Structure

```
MITRE ATT&CK STRUCTURE:

  TACTICS (14 Categories - The "Why"):
  ┌─────────────────────────────────────────────────────┐
  │ Recon │ Resource │ Initial │ Execution │ Persistence│
  │       │ Dev      │ Access  │           │            │
  ├───────┼──────────┼─────────┼───────────┼────────────┤
  │ Priv  │ Defense  │ Cred    │ Discovery │ Lateral    │
  │ Esc   │ Evasion  │ Access  │           │ Movement   │
  ├───────┼──────────┼─────────┼───────────┼────────────┤
  │ Collec│ C2       │ Exfil   │ Impact    │            │
  │ tion  │          │ tration │           │            │
  └───────┴──────────┴─────────┴───────────┴────────────┘

  TECHNIQUES (The "How"):
  → Specific methods adversaries use
  → ~200 techniques, ~400 sub-techniques
  → Each has unique ID (e.g., T1059)
  → Includes: Description, procedures, detections,
    mitigations, data sources

  PROCEDURES (The "What Exactly"):
  → Specific implementations by threat actors
  → Real-world examples
  → Linked to threat groups (e.g., APT29)

  EXAMPLE:
  Tactic: Execution
  └── Technique: T1059 - Command and Scripting Interpreter
      ├── T1059.001 - PowerShell
      ├── T1059.003 - Windows Command Shell
      ├── T1059.005 - Visual Basic
      └── T1059.006 - Python

ATT&CK MATRICES:
  → Enterprise (Windows, macOS, Linux, Cloud)
  → Mobile (Android, iOS)
  → ICS (Industrial Control Systems)
```

---

## 2. ATT&CK for Detection Mapping

```
MAPPING DETECTIONS TO ATT&CK:

DETECTION COVERAGE MAP:
  → Map each detection rule to ATT&CK technique(s)
  → Visualize coverage across the matrix
  → Identify gaps in detection
  → Prioritize new detection development

  Example Detection Mapping:
  Detection Rule              | Technique
  Encoded PowerShell          | T1059.001
  Brute force login           | T1110.001
  New service installed       | T1543.003
  Scheduled task created      | T1053.005
  LSASS memory access         | T1003.001
  Pass-the-hash               | T1550.002
  DNS tunneling               | T1071.004
  Large data transfer         | T1048

USING ATT&CK NAVIGATOR:
  → Web-based visualization tool
  → Color-code techniques by coverage
  → Layer multiple data sets
  → Export/import configurations
  → URL: attack.mitre.org/matrices/enterprise/

  Color Scheme Example:
  Red    = No detection
  Yellow = Partial detection
  Green  = Good detection
  Blue   = Multiple detections

DATA SOURCES (ATT&CK v10+):
  → Each technique lists required data sources
  → Guides log collection priorities
  → Examples:
    T1059.001 (PowerShell):
    → Process: Process Creation
    → Command: Command Execution  
    → Script: Script Execution
    → Module: Module Load
```

---

## 3. Building ATT&CK-Based Detection

```
PRIORITIZATION FRAMEWORK:

STEP 1: IDENTIFY RELEVANT THREATS
  → What threat actors target your industry?
  → What techniques do they use?
  → ATT&CK Groups: https://attack.mitre.org/groups/
  
  Example (Financial Sector):
  → FIN7: T1566.001, T1059.001, T1071.001
  → APT38: T1190, T1059.006, T1048
  → Carbanak: T1566.002, T1053.005

STEP 2: MAP CURRENT COVERAGE
  → List all existing detection rules
  → Map each to ATT&CK technique
  → Identify gaps
  → Use ATT&CK Navigator to visualize

STEP 3: PRIORITIZE GAPS
  Priority based on:
  → Threat actor relevance
  → Technique prevalence
  → Data source availability
  → Implementation difficulty
  → Business impact if undetected

STEP 4: DEVELOP DETECTIONS
  For each prioritized technique:
  → Study technique description
  → Identify data sources needed
  → Write detection logic
  → Test with attack simulation
  → Deploy and monitor

TOP 10 TECHNIQUES TO DETECT:
  Rank | Technique            | ID
  1    | PowerShell           | T1059.001
  2    | Windows Command Shell| T1059.003
  3    | Scheduled Task       | T1053.005
  4    | Credential Dumping   | T1003
  5    | Remote Services      | T1021
  6    | Phishing Attachment  | T1566.001
  7    | Valid Accounts       | T1078
  8    | Registry Run Keys    | T1547.001
  9    | Ingress Tool Transfer| T1105
  10   | Proxy/Tunneling      | T1090
```

---

## 4. ATT&CK Integration Tools

```
TOOLS FOR ATT&CK-BASED DETECTION:

ATT&CK NAVIGATOR:
  → Visualize technique coverage
  → Layer comparison
  → Gap analysis
  → https://mitre-attack.github.io/attack-navigator/

DeTT&CT:
  → Detection & Threat Coverage Tool
  → Map data sources and detections
  → Compare against threat groups
  → Visibility and detection scoring
  → YAML-based configuration

  # DeTT&CT usage
  python dettect.py editor
  python dettect.py d -ft techniques.yaml
  python dettect.py ds -ft data_sources.yaml

SIGMA + ATT&CK:
  → Sigma rules include ATT&CK tags
  → Map Sigma rules to techniques
  → Convert to SIEM-specific format
  
  # Sigma rule with ATT&CK mapping
  title: Suspicious PowerShell Download
  tags:
    - attack.execution
    - attack.t1059.001
    - attack.command_and_control
    - attack.t1105
  detection:
    selection:
      CommandLine|contains:
        - 'Invoke-WebRequest'
        - 'wget'
        - 'curl'
        - 'DownloadString'
    condition: selection

ATOMIC RED TEAM:
  → Test cases mapped to ATT&CK
  → Validate detections per technique
  → Automated execution
  
  # Test T1003.001 (Credential Dumping)
  Invoke-AtomicTest T1003.001
  
  # List all tests for a technique
  Invoke-AtomicTest T1059.001 -ShowDetailsBrief

CALDERA:
  → MITRE's adversary emulation platform
  → Chain techniques into operations
  → Automated attack simulation
  → Tests detection across kill chain
```

---

## 5. Measuring ATT&CK Coverage

```
COVERAGE METRICS:

QUANTITATIVE METRICS:
  Metric                      | Calculation
  Technique coverage %        | Detected / Total × 100
  Sub-technique coverage %    | Sub-detected / Total × 100
  Tactic coverage %          | Tactics with 1+ detection / 14
  Detection per technique    | Avg rules per technique
  Validated detections %     | Tested / Total rules × 100

COVERAGE SCORING:
  Score | Description
  0     | No detection capability
  1     | Partial detection (some variants)
  2     | Good detection (most variants)
  3     | Comprehensive detection
  4     | Validated + automated response

EXAMPLE COVERAGE REPORT:
  Tactic             | Techniques | Detected | Score
  Initial Access     | 9          | 6        | 67%
  Execution          | 12         | 10       | 83%
  Persistence        | 19         | 11       | 58%
  Privilege Esc      | 13         | 8        | 62%
  Defense Evasion    | 42         | 15       | 36%
  Credential Access  | 17         | 12       | 71%
  Discovery          | 31         | 10       | 32%
  Lateral Movement   | 9          | 7        | 78%
  Collection         | 17         | 5        | 29%
  C2                 | 16         | 9        | 56%
  Exfiltration       | 9          | 4        | 44%
  Impact             | 13         | 8        | 62%
  OVERALL            | 207        | 105      | 51%

IMPROVEMENT PLAN:
  Priority | Tactic          | Current | Target
  1        | Defense Evasion  | 36%     | 60%
  2        | Collection       | 29%     | 55%
  3        | Discovery        | 32%     | 50%
  4        | Exfiltration     | 44%     | 65%
```

---

## Summary Table

| ATT&CK Component | Description | Use in Detection |
|------------------|-------------|-----------------|
| Tactics | Attack goals (14) | Organize detection strategy |
| Techniques | Attack methods (~200) | Map detection rules |
| Sub-techniques | Specific variants (~400) | Granular detection |
| Data Sources | Required log data | Guide collection |
| Groups | Threat actors | Prioritize by relevance |
| Navigator | Visualization tool | Coverage mapping |

---

## Revision Questions

1. How is the MITRE ATT&CK framework structured?
2. How should organizations prioritize which techniques to detect?
3. What tools help map detection coverage to ATT&CK?
4. How is ATT&CK coverage measured and scored?
5. How does Sigma integrate with the ATT&CK framework?

---

*Previous: [04-anomaly-detection.md](04-anomaly-detection.md) | Next: [06-detection-as-code.md](06-detection-as-code.md)*

---

*[Back to README](../README.md)*
