# Unit 3: Threat Detection — Topic 3: Behavior-Based Detection

## Overview

**Behavior-based detection** identifies threats by recognizing malicious behaviors and techniques rather than specific indicators. This approach is more resilient to evasion because attackers can change their tools and indicators but must still perform the same fundamental actions. Behavior-based detection focuses on the "how" of attacks rather than the "what."

---

## 1. Behavioral Detection Fundamentals

```
IOC vs BEHAVIOR DETECTION:

  IOC-Based:
  → "Is this file hash known malicious?"
  → "Is this IP on a blocklist?"
  → Detects KNOWN threats
  → Easily evaded by changing indicators

  Behavior-Based:
  → "Is this process behavior suspicious?"
  → "Is this sequence of actions abnormal?"
  → Detects UNKNOWN threats
  → Harder to evade (technique must still work)

EXAMPLE:
  IOC Approach:
  → Alert if file hash = abc123def456...
  → Attacker changes one byte → new hash → evaded

  Behavioral Approach:
  → Alert if Word spawns PowerShell which makes 
    outbound connection
  → Attacker must still execute this chain → detected
  → Regardless of specific file or domain used

BEHAVIORAL DETECTION CATEGORIES:

  1. PROCESS BEHAVIOR
     → Unusual parent-child relationships
     → Suspicious command line arguments
     → Process injection indicators
     → Living-off-the-land binaries (LOLBins)

  2. NETWORK BEHAVIOR
     → Beaconing patterns (regular intervals)
     → DNS tunneling (long queries)
     → Unusual port usage
     → Data exfiltration patterns

  3. USER BEHAVIOR
     → Off-hours activity
     → Unusual resource access
     → Privilege escalation patterns
     → Account anomalies

  4. SYSTEM BEHAVIOR
     → Configuration changes
     → Service modifications
     → Scheduled task creation
     → Registry modifications
```

---

## 2. Process-Based Detection

```
SUSPICIOUS PROCESS BEHAVIORS:

PARENT-CHILD RELATIONSHIPS:
  Normal:
  explorer.exe → chrome.exe (user launched browser)
  services.exe → svchost.exe (system service)
  
  Suspicious:
  winword.exe → cmd.exe → powershell.exe (macro execution)
  outlook.exe → powershell.exe (email-based attack)
  svchost.exe → cmd.exe (service exploitation)
  w3wp.exe → cmd.exe (web shell)

LOLBIN ABUSE:
  Binary       | Legitimate Use      | Malicious Use
  certutil.exe | Certificate mgmt    | Download files
  mshta.exe    | HTML applications   | Execute scripts
  regsvr32.exe | Register DLLs       | Proxy execution
  rundll32.exe | Run DLL functions   | Execute payloads
  wmic.exe     | System management   | Remote execution
  bitsadmin.exe| Background transfer | Download payloads
  msiexec.exe  | Install packages    | Remote payload exec

DETECTION RULES:

  # Sigma: Office spawning shell
  title: Microsoft Office Process Spawning Shell
  logsource:
    category: process_creation
    product: windows
  detection:
    selection_parent:
      ParentImage|endswith:
        - '\winword.exe'
        - '\excel.exe'
        - '\powerpnt.exe'
        - '\outlook.exe'
    selection_child:
      Image|endswith:
        - '\cmd.exe'
        - '\powershell.exe'
        - '\wscript.exe'
        - '\mshta.exe'
    condition: selection_parent and selection_child
  level: high
  tags:
    - attack.execution
    - attack.t1204.002

  # Sigma: Encoded PowerShell
  title: Encoded PowerShell Command
  detection:
    selection:
      CommandLine|contains:
        - '-enc '
        - '-EncodedCommand'
        - 'FromBase64String'
    condition: selection
  level: high
```

---

## 3. Network Behavior Detection

```
NETWORK BEHAVIORAL INDICATORS:

BEACONING DETECTION:
  → Regular interval communication
  → C2 channels often beacon every X seconds/minutes
  → Statistical analysis of connection timing
  
  Detection Logic:
  → Calculate time between connections to same dest
  → Look for low jitter (consistent intervals)
  → Flag destinations with regular patterns
  
  # Splunk: Beaconing detection
  index=proxy
  | stats count, avg(interval) as avg_interval,
    stdev(interval) as std_interval by src_ip, dest_ip
  | eval jitter = std_interval / avg_interval
  | where jitter < 0.1 AND count > 50
  | sort jitter

DNS TUNNELING:
  → Unusually long DNS queries
  → High volume of DNS requests to single domain
  → Encoded data in subdomain labels
  → Unusual record types (TXT, NULL)
  
  Detection:
  → Query length > 50 characters
  → Entropy of subdomain > threshold
  → Volume of unique subdomains per domain
  → TXT record queries to unusual domains

  # Sigma: Long DNS query
  title: Potential DNS Tunneling
  detection:
    selection:
      query_length|gt: 50
      query_type: 'TXT'
    condition: selection
  level: medium

DATA EXFILTRATION PATTERNS:
  → Large outbound data transfers
  → Uploads to cloud storage (Dropbox, GDrive)
  → FTP/SCP to external systems
  → Email with large attachments to personal domains
  → Unusual protocols for data transfer

LATERAL MOVEMENT PATTERNS:
  → Single source → multiple internal destinations
  → Authentication to many hosts in short time
  → SMB/WMI/WinRM connections to multiple hosts
  → PsExec-like service creation patterns
```

---

## 4. User Behavior Analytics (UBA/UEBA)

```
USER AND ENTITY BEHAVIOR ANALYTICS:

CONCEPT:
  → Baseline normal behavior per user/entity
  → Detect deviations from baseline
  → Risk scoring based on anomalies
  → Combines multiple weak signals

BASELINE FACTORS:
  Factor              | Normal Baseline
  Working hours       | 9 AM - 6 PM weekdays
  Login locations     | Office, home, VPN
  Systems accessed    | Usual set of servers
  Data volume         | Average download/upload
  Applications used   | Standard app set
  Peer group behavior | Similar role patterns

ANOMALY EXAMPLES:
  Anomaly                      | Risk Score
  Login at 3 AM                | +15
  Login from new country       | +25
  Accessing unusual system     | +10
  Downloading 10x normal data  | +30
  Using admin tools (unusual)  | +20
  First time using PowerShell  | +15
  Accessing sensitive shares   | +20
  
  Threshold: Alert when score > 80

UEBA PLATFORMS:
  → Microsoft Sentinel UEBA
  → Splunk UBA
  → Exabeam
  → Securonix
  → Gurucul
  → Elastic ML (anomaly detection)

DETECTION EXAMPLES:
  → User downloads 500MB after submitting resignation
  → Admin account used from previously unseen device
  → Service account with interactive login
  → Multiple failed auth → success → privilege change
  → User accesses file share for first time in 6 months
```

---

## 5. Building Behavioral Detections

```
METHODOLOGY:

STEP 1: UNDERSTAND THE TECHNIQUE
  → Study MITRE ATT&CK description
  → Identify the fundamental behavior
  → What MUST the attacker do?
  → What data shows this behavior?

STEP 2: IDENTIFY DATA SOURCES
  → What logs capture this behavior?
  → Process creation (Sysmon Event 1)
  → Network connections (Sysmon Event 3)
  → Registry changes (Sysmon Event 13)
  → File creation (Sysmon Event 11)

STEP 3: DEFINE NORMAL vs ABNORMAL
  → What does legitimate use look like?
  → What makes malicious use different?
  → Context that distinguishes the two
  → Edge cases to consider

STEP 4: WRITE THE RULE
  → Focus on technique, not tool
  → Use process relationships
  → Include contextual conditions
  → Layer conditions to reduce FP

STEP 5: TEST AND TUNE
  → Execute the attack technique
  → Verify detection fires
  → Run against production data
  → Identify and exclude false positives
  → Document tuning decisions

BEST PRACTICES:
  [ ] Detect techniques, not tools
  [ ] Use process ancestry chains
  [ ] Combine multiple weak signals
  [ ] Leverage UEBA for baseline comparison
  [ ] Test with real attack simulations
  [ ] Accept some false positives for better coverage
  [ ] Layer with IOC detection for defense in depth
  [ ] Document the attack behavior being detected
```

---

## Summary Table

| Detection Type | Focus | Evasion Resistance | Complexity |
|---------------|-------|-------------------|------------|
| Process behavior | Parent-child, command line | High | Medium |
| Network behavior | Beaconing, tunneling | High | Medium-High |
| UEBA | User anomalies | Very High | High |
| LOLBin abuse | Living-off-the-land | Medium-High | Medium |
| File behavior | File operations | Medium | Medium |

---

## Revision Questions

1. How does behavior-based detection differ from IOC-based detection?
2. What suspicious parent-child process relationships indicate attacks?
3. How is beaconing behavior detected in network traffic?
4. What is UEBA and how does it establish behavioral baselines?
5. What methodology should be followed for building behavioral detections?

---

*Previous: [02-indicator-based-detection.md](02-indicator-based-detection.md) | Next: [04-anomaly-detection.md](04-anomaly-detection.md)*

---

*[Back to README](../README.md)*
