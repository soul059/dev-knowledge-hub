# Unit 10: Threat Hunting — Topic 2: Hypothesis Development

## Overview

**Hypothesis development** is the foundation of effective threat hunting. A well-crafted hypothesis transforms hunting from aimless searching into focused, testable investigation. Good hypotheses are specific, based on intelligence or knowledge, and lead to actionable results regardless of whether a threat is found.

---

## 1. Hypothesis Framework

```
HYPOTHESIS STRUCTURE:

  A GOOD HYPOTHESIS:
  → Is specific and testable
  → Describes a detectable behavior
  → Is based on threat intelligence or ATT&CK
  → Can be proven or disproven with available data
  → Leads to actionable outcomes

  HYPOTHESIS FORMULA:
  "An adversary [may be / could be] using
  [technique/tool/behavior] to [achieve objective]
  in our environment, which would be visible in
  [data source] as [observable indicator]."

EXAMPLES:

  GOOD HYPOTHESIS:
  "An adversary may be using PowerShell to download
  malicious payloads from external servers, which
  would be visible in Sysmon logs as powershell.exe
  processes with command lines containing
  'DownloadString', 'DownloadFile', or
  'Invoke-WebRequest' connecting to non-corporate
  URLs."

  BAD HYPOTHESIS:
  "There might be malware somewhere in the network."
  → Too vague, not testable, no specific indicator

  BAD HYPOTHESIS:
  "An attacker is using a zero-day exploit."
  → Cannot be tested without knowing the exploit
  → No specific observable defined

HYPOTHESIS QUALITY CRITERIA:
  ✓ Specific technique or behavior described
  ✓ Observable indicators defined
  ✓ Data sources identified
  ✓ Testable with available tools
  ✓ Based on real threat intelligence
  ✓ Relevant to your environment
  ✗ Too broad (any malware)
  ✗ Too specific (only one exact hash)
  ✗ Untestable (zero-day with no indicators)
  ✗ Not relevant (Linux hunt in Windows-only env)
```

---

## 2. Hypothesis Sources

```
WHERE HYPOTHESES COME FROM:

THREAT INTELLIGENCE:
  → Published threat reports (Mandiant, CrowdStrike)
  → Industry-specific threat briefs
  → ISAC/ISAO bulletins
  → Government advisories (CISA, FBI)
  → Vendor security advisories
  → Open-source intelligence

  Example: "Mandiant reports APT41 using
  scheduled tasks with encoded PowerShell
  commands for persistence. Hunt for this
  in our environment."

MITRE ATT&CK:
  → Review techniques relevant to environment
  → Focus on techniques lacking detection
  → Prioritize by adversary groups targeting
    your industry
  → Map detection coverage to ATT&CK

  Example: "We have no detection for T1547.001
  (Registry Run Keys). Hunt for new or modified
  run keys across all endpoints."

INCIDENT HISTORY:
  → Past incidents in your organization
  → Similar organizations' incidents
  → Recurring attack patterns
  → Previously missed indicators

  Example: "Our last incident used credential
  dumping. Hunt for other credential access
  techniques (T1003) that we might miss."

VULNERABILITY DISCLOSURES:
  → New CVEs affecting your technology
  → Actively exploited vulnerabilities
  → Proof-of-concept availability

  Example: "CVE-2024-XXXX affects our VPN.
  Hunt for exploitation indicators in VPN
  logs and associated endpoint activity."

ANOMALIES AND INTUITION:
  → Something that "doesn't look right"
  → Unusual patterns noticed during analysis
  → Analyst experience and gut feeling
  → Data exploration discoveries

  Example: "I noticed unusual DNS query
  patterns during routine monitoring. Develop
  hypothesis around DNS tunneling."
```

---

## 3. Hypothesis Prioritization

```
PRIORITIZATION FRAMEWORK:

  Score each hypothesis on:

  THREAT LIKELIHOOD (1-5):
  1 = Very unlikely
  2 = Unlikely
  3 = Possible
  4 = Likely
  5 = Very likely (active threat intel)

  POTENTIAL IMPACT (1-5):
  1 = Minimal
  2 = Low
  3 = Moderate
  4 = High
  5 = Critical (data loss, ransomware)

  DETECTION GAP (1-5):
  1 = Well-covered by existing detection
  2 = Mostly covered
  3 = Partial coverage
  4 = Poor coverage
  5 = No existing detection

  DATA AVAILABILITY (1-5):
  1 = No data available
  2 = Partial data
  3 = Adequate data
  4 = Good data coverage
  5 = Comprehensive data

PRIORITY SCORE = Likelihood × Impact × Gap × Data

HYPOTHESIS BACKLOG:
  ID   | Hypothesis              | L | I | G | D | Score
  H-01 | PS download cradles     | 4 | 4 | 4 | 5 | 320
  H-02 | Scheduled task persist  | 3 | 4 | 5 | 5 | 300
  H-03 | LSASS credential dump   | 3 | 5 | 3 | 4 | 180
  H-04 | DNS tunneling           | 2 | 3 | 5 | 4 | 120
  H-05 | SSH lateral movement    | 2 | 3 | 4 | 3 | 72
  
  → Hunt H-01 first (highest priority)
  → Then H-02, H-03, etc.
```

---

## 4. From Hypothesis to Hunt Plan

```
CONVERTING HYPOTHESIS TO HUNT PLAN:

HYPOTHESIS:
  "An adversary may be using WMI for lateral
  movement, creating remote processes on other
  systems using compromised credentials."

HUNT PLAN:
  1. ATT&CK MAPPING:
     → T1047: Windows Management Instrumentation
     → Related: T1021 (Remote Services)

  2. DATA SOURCES NEEDED:
     → Sysmon Event ID 1 (Process Creation)
     → Sysmon Event ID 20 (WMI Event)
     → Windows Event Log 4688 (Process Create)
     → Network: Port 135 (RPC), 5985 (WinRM)
     → PowerShell Script Block Logging

  3. SEARCH QUERIES:
     # WMI process creation
     index=sysmon EventCode=1
       (ParentImage="*wmiprvse.exe" OR
        Image="*wmic.exe")
     | table _time, ComputerName, User,
       ParentImage, Image, CommandLine

     # Remote WMI connections
     index=sysmon EventCode=3
       Image="*wmic.exe"
       DestinationPort=135
     | table _time, ComputerName, User,
       DestinationIp

  4. ANALYSIS APPROACH:
     → Identify all WMI process creation
     → Filter out known legitimate usage
     → Analyze remaining for suspicious patterns
     → Check user context and timing
     → Correlate with other events

  5. EXPECTED FINDINGS:
     → Legitimate: SCCM, monitoring tools
     → Suspicious: Interactive user WMI to
       multiple systems, unusual processes
       spawned by wmiprvse.exe

  6. RESPONSE IF FOUND:
     → Escalate to IR team
     → Contain affected systems
     → Expand scope investigation
```

---

## 5. Iterating on Hypotheses

```
HYPOTHESIS ITERATION:

INITIAL HYPOTHESIS:
  "Attacker using PowerShell for downloads"
  → Result: Found IT automation scripts only

REFINED HYPOTHESIS:
  "Attacker using encoded PowerShell commands"
  → Result: Found 3 encoded commands, all
    legitimate but poorly documented

FURTHER REFINED:
  "Attacker using PowerShell with obfuscation
  techniques (string concatenation, variable
  substitution, XOR encoding)"
  → Result: Found 1 suspicious instance →
    escalated to IR

ITERATION PATTERN:
  Broad → Narrow → Specific → Targeted

DOCUMENTING ITERATIONS:
  Iteration | Hypothesis        | Result     | Next
  1         | PS downloads      | Legit only | Narrow
  2         | Encoded PS        | Legit      | Add obfusc
  3         | Obfuscated PS     | 1 suspect  | Investigate
  4         | N/A               | IR case    | New hunt

WHEN TO STOP:
  → Hypothesis disproven with confidence
  → Sufficient data analyzed
  → Time allocation exhausted
  → Threat found (hand off to IR)
  → All reasonable refinements explored
  → Document findings regardless of outcome
```

---

## Summary Table

| Element | Description | Example |
|---------|------------|---------|
| Hypothesis | Testable theory | "Attacker may use WMI for lateral movement" |
| Source | What triggered it | Threat intel report on APT |
| Priority | Risk-based ranking | Likelihood × Impact × Gap × Data |
| Hunt Plan | How to test | Queries, data sources, analysis approach |
| Iteration | Refine and repeat | Broad → Narrow → Specific |

---

## Revision Questions

1. What makes a good threat hunting hypothesis?
2. What are the main sources for developing hypotheses?
3. How should hypotheses be prioritized?
4. How do you convert a hypothesis into an actionable hunt plan?
5. When should you stop iterating on a hypothesis?

---

*Previous: [01-hunting-methodology.md](01-hunting-methodology.md) | Next: [03-data-sources.md](03-data-sources.md)*

---

*[Back to README](../README.md)*
