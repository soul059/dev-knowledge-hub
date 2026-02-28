# Chapter 10.2: Threat Detection

## Overview

Threat detection identifies malicious activities, unauthorized access, and anomalous behaviour across applications, infrastructure, and cloud environments. Modern threat detection combines signature-based rules, behavioural analytics, threat intelligence feeds, and machine learning to detect known and unknown threats. This chapter covers threat detection frameworks, tools, and implementation patterns.

---

## Threat Detection Layers

```
DEFENSE-IN-DEPTH THREAT DETECTION:

  ┌─────────────────────────────────────────────────────┐
  │ LAYER 1: NETWORK                                      │
  │ IDS/IPS • NetFlow Analysis • DNS Monitoring           │
  ├─────────────────────────────────────────────────────┤
  │ LAYER 2: HOST / OS                                     │
  │ HIDS • FIM • Auditd • Sysmon • EDR                   │
  ├─────────────────────────────────────────────────────┤
  │ LAYER 3: CONTAINER / RUNTIME                           │
  │ Falco • Sysdig • Tracee • Seccomp Violations          │
  ├─────────────────────────────────────────────────────┤
  │ LAYER 4: APPLICATION                                   │
  │ WAF • RASP • App Security Logs • API Anomalies        │
  ├─────────────────────────────────────────────────────┤
  │ LAYER 5: CLOUD                                         │
  │ GuardDuty • Defender • SCC • CloudTrail Analysis      │
  ├─────────────────────────────────────────────────────┤
  │ LAYER 6: IDENTITY                                      │
  │ Impossible Travel • Credential Stuffing • MFA Bypass  │
  └─────────────────────────────────────────────────────┘
```

---

## MITRE ATT&CK Framework

```
MITRE ATT&CK — ENTERPRISE TACTICS:

  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐
  │Recon  │▶│Resrce │▶│Init   │▶│Exectn │▶│Persist│
  │       │ │Devlop │ │Access │ │       │ │ence   │
  └───────┘ └───────┘ └───────┘ └───────┘ └───────┘
                                               │
  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐     │
  │Exfil  │◀│Collect│◀│Lateral│◀│Discov │◀────┘
  │tration│ │ion    │ │Move   │ │ery    │
  └───────┘ └───────┘ └───────┘ └───────┘

  DevSecOps-Relevant Techniques:
  ├── T1078: Valid Accounts (stolen credentials)
  ├── T1190: Exploit Public-Facing Application
  ├── T1525: Implant Container Image
  ├── T1552: Unsecured Credentials
  ├── T1610: Deploy Container
  ├── T1611: Escape to Host
  ├── T1613: Container and Resource Discovery
  └── T1053: Scheduled Task (cron jobs)
```

---

## Cloud Threat Detection Services

### AWS GuardDuty

```bash
# Enable GuardDuty
aws guardduty create-detector \
    --enable \
    --finding-publishing-frequency FIFTEEN_MINUTES \
    --data-sources '{
      "S3Logs": {"Enable": true},
      "Kubernetes": {"AuditLogs": {"Enable": true}},
      "MalwareProtection": {"ScanEc2InstanceWithFindings": {"EbsVolumes": true}}
    }'

# List findings
aws guardduty list-findings \
    --detector-id abc123 \
    --finding-criteria '{
      "Criterion": {
        "severity": {"Gte": 7}
      }
    }'

# GuardDuty finding types:
# • UnauthorizedAccess:EC2/MaliciousIPCaller
# • Recon:EC2/PortProbeUnprotectedPort
# • CryptoCurrency:EC2/BitcoinTool
# • Persistence:IAMUser/AnomalousBehavior
# • Impact:EC2/WinRMBruteForce
# • PrivilegeEscalation:IAMUser/AdministrativePermissions
# • Discovery:Kubernetes/SuccessfulAnonymousAccess
```

### Azure Defender / Microsoft Sentinel

```json
// Azure Sentinel Analytics Rule — Brute Force Detection
{
  "displayName": "SSH Brute Force Detection",
  "severity": "High",
  "query": "Syslog\n| where Facility == 'auth' and SyslogMessage contains 'Failed password'\n| summarize FailureCount=count(), \n    DistinctUsers=dcount(ExtractedUserName),\n    FirstSeen=min(TimeGenerated),\n    LastSeen=max(TimeGenerated)\n    by SrcIP=extract(@'\\b(\\d{1,3}\\.\\d{1,3}\\.\\d{1,3}\\.\\d{1,3})\\b', 1, SyslogMessage)\n| where FailureCount > 20 and DistinctUsers > 3",
  "queryFrequency": "PT5M",
  "triggerOperator": "GreaterThan",
  "triggerThreshold": 0,
  "tactics": ["CredentialAccess"],
  "techniques": ["T1110"]
}
```

---

## Falco Runtime Threat Detection

```yaml
# falco-custom-rules.yaml
- rule: Detect Reverse Shell
  desc: Detect reverse shell connections from containers
  condition: >
    spawned_process and
    container and
    ((proc.name in (bash, sh, zsh)) and
     (fd.type = ipv4 and fd.l4proto = tcp) and
     (fd.sip != "127.0.0.1"))
  output: >
    Reverse shell detected
    (user=%user.name command=%proc.cmdline
     container=%container.name image=%container.image.repository
     connection=%fd.name)
  priority: CRITICAL
  tags: [network, mitre_execution, T1059]

- rule: Detect Cryptocurrency Mining
  desc: Detect crypto mining processes
  condition: >
    spawned_process and
    (proc.name in (xmrig, minerd, cpuminer, minergate) or
     proc.cmdline contains "stratum+tcp://" or
     proc.cmdline contains "pool.minergate" or
     proc.cmdline contains "nanopool.org")
  output: >
    Crypto mining detected
    (user=%user.name command=%proc.cmdline
     container=%container.name image=%container.image.repository)
  priority: CRITICAL
  tags: [process, mitre_impact, T1496]

- rule: Sensitive File Access in Container
  desc: Detect access to sensitive host files from container
  condition: >
    open_read and container and
    (fd.name startswith /etc/shadow or
     fd.name startswith /etc/passwd or
     fd.name startswith /root/.ssh or
     fd.name startswith /etc/kubernetes/pki)
  output: >
    Sensitive file accessed in container
    (user=%user.name file=%fd.name
     container=%container.name image=%container.image.repository)
  priority: WARNING
  tags: [filesystem, mitre_credential_access, T1552]
```

---

## Sigma Rules (Generic Detection)

```yaml
# Sigma rule — vendor-agnostic detection format
title: Suspicious Kubectl Exec Command
id: a8b1c2d3-e4f5-6789-abcd-ef0123456789
status: experimental
description: Detects kubectl exec into pods, indicating potential compromise
author: Security Team
date: 2024/01/15
references:
  - https://attack.mitre.org/techniques/T1609/
logsource:
  product: kubernetes
  service: audit
detection:
  selection:
    verb: create
    resource: pods/exec
  filter:
    user.username:
      - system:serviceaccount:kube-system:*
      - system:admin
  condition: selection and not filter
falsepositives:
  - Legitimate debugging by authorized personnel
level: medium
tags:
  - attack.execution
  - attack.t1609
```

```bash
# Convert Sigma to platform-specific queries
# Install sigmac
pip install sigma-cli

# Convert to Elasticsearch query
sigma convert -t elasticsearch sigma-rules/kubectl-exec.yml

# Convert to Splunk query
sigma convert -t splunk sigma-rules/kubectl-exec.yml

# Convert to Azure Sentinel (KQL)
sigma convert -t microsoft365defender sigma-rules/kubectl-exec.yml
```

---

## Threat Intelligence Integration

```
THREAT INTELLIGENCE FEEDS:

  ┌────────────────────────────────────────────────────┐
  │                                                      │
  │  Open Source Feeds:                                  │
  │  ├── AlienVault OTX (IPs, domains, hashes)          │
  │  ├── Abuse.ch (malware, botnets)                    │
  │  ├── MISP (threat sharing platform)                 │
  │  ├── PhishTank (phishing URLs)                      │
  │  └── VirusTotal (file/URL reputation)               │
  │                                                      │
  │  Commercial Feeds:                                   │
  │  ├── CrowdStrike (adversary intelligence)           │
  │  ├── Recorded Future (predictive intel)             │
  │  ├── Mandiant (APT tracking)                        │
  │  └── Palo Alto Unit 42 (threat research)            │
  │                                                      │
  │  Integration Points:                                 │
  │  ├── SIEM enrichment (correlate IOCs with logs)     │
  │  ├── Firewall/WAF blocklists                        │
  │  ├── DNS sinkholing                                  │
  │  └── Automated IOC scanning                          │
  │                                                      │
  └────────────────────────────────────────────────────┘
```

```python
# Python — Query VirusTotal for IOC enrichment
import requests

VT_API_KEY = "your-api-key"

def check_ip_reputation(ip_address):
    """Check IP against VirusTotal threat intelligence."""
    url = f"https://www.virustotal.com/api/v3/ip_addresses/{ip_address}"
    headers = {"x-apikey": VT_API_KEY}
    
    response = requests.get(url, headers=headers)
    data = response.json()
    
    stats = data["data"]["attributes"]["last_analysis_stats"]
    malicious = stats.get("malicious", 0)
    suspicious = stats.get("suspicious", 0)
    
    if malicious > 5 or suspicious > 3:
        return {
            "ip": ip_address,
            "verdict": "MALICIOUS",
            "malicious_detections": malicious,
            "suspicious_detections": suspicious,
            "action": "BLOCK"
        }
    return {"ip": ip_address, "verdict": "CLEAN"}
```

---

## Detection Engineering Workflow

```
DETECTION ENGINEERING LIFECYCLE:

  ┌──────────┐    ┌──────────┐    ┌──────────┐
  │RESEARCH  │───▶│  WRITE   │───▶│  TEST    │
  │          │    │          │    │          │
  │• ATT&CK  │    │• Sigma   │    │• Atomic  │
  │• Threat  │    │  rules   │    │  Red Team│
  │  reports │    │• SIEM    │    │• Purple  │
  │• CVEs    │    │  queries │    │  Team    │
  │• Intel   │    │• Falco   │    │• Replay  │
  │  feeds   │    │  rules   │    │  attacks │
  └──────────┘    └──────────┘    └──────────┘
       ▲                               │
       │                               ▼
  ┌──────────┐    ┌──────────┐    ┌──────────┐
  │  TUNE    │◀───│  DEPLOY  │◀───│ VALIDATE │
  │          │    │          │    │          │
  │• Reduce  │    │• Push to │    │• Verify  │
  │  false   │    │  prod    │    │  detect  │
  │  positve │    │• Monitor │    │  rate    │
  │• Improve │    │• Alert   │    │• Check   │
  │  fidelity│    │  routing │    │  FP rate │
  └──────────┘    └──────────┘    └──────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Detection rule has too many false positives | Rule too broad; missing context | Add exclusion filters for known-good behaviour; use correlation with multiple data sources |
| GuardDuty findings overwhelming | Default findings include low-severity items | Filter by severity (≥7); suppress known-good patterns; archive handled findings |
| Sigma rules don't convert cleanly | Platform-specific field name differences | Maintain field mappings per backend; validate converted queries; use logsource abstraction |
| Runtime detection misses attacks | Attacker uses novel techniques | Layer multiple detection approaches; update rules from threat intel; run red team exercises |
| Threat intel feeds produce noise | Not all IOCs relevant to your environment | Filter feeds by sector/geography; score IOC confidence; age out old indicators |

---

## Summary Table

| Detection Layer | Tools | Detects |
|----------------|-------|---------|
| **Network** | Suricata, Zeek, VPC Flow Logs | Port scans, C2 traffic, lateral movement |
| **Host/OS** | OSSEC, Wazuh, Auditd, Sysmon | File changes, privilege escalation, rootkits |
| **Container** | Falco, Tracee, Sysdig | Container escapes, crypto mining, reverse shells |
| **Application** | WAF, RASP, app logs | SQLi, XSS, API abuse, bot traffic |
| **Cloud** | GuardDuty, Sentinel, SCC | IAM anomalies, resource abuse, misconfigs |
| **Identity** | Azure AD IPC, Okta ThreatInsight | Impossible travel, credential stuffing, MFA bypass |

---

## Quick Revision Questions

1. **What is the MITRE ATT&CK framework and how is it used in threat detection?**
   > MITRE ATT&CK is a knowledge base of adversary tactics, techniques, and procedures (TTPs) based on real-world observations. It organizes attack behaviours into 14 tactics (Reconnaissance through Impact). Security teams map detection rules to ATT&CK techniques to identify coverage gaps, prioritize detection development, and communicate threat scenarios using a common language.

2. **How does AWS GuardDuty detect threats?**
   > GuardDuty analyzes CloudTrail logs, VPC Flow Logs, DNS query logs, EKS audit logs, and S3 data events using threat intelligence feeds, machine learning for anomaly detection, and built-in rules. It identifies threats like unauthorized access, cryptocurrency mining, credential compromise, data exfiltration, and Kubernetes-specific attacks without requiring agents.

3. **What are Sigma rules?**
   > Sigma is a generic, vendor-agnostic signature format for SIEM detection rules, similar to how Snort rules work for IDS. Rules are written in YAML with structured logic (log source, detection criteria, severity). They can be converted to platform-specific query languages (Elasticsearch, Splunk, Sentinel KQL) using sigma-cli, enabling rule sharing across different SIEM platforms.

4. **What is the detection engineering lifecycle?**
   > Research (study ATT&CK, threat reports, CVEs), Write (create Sigma/Falco rules), Test (use Atomic Red Team to simulate attacks), Validate (verify detection rate and false positive rate), Deploy (push to production SIEM/detection platform), Tune (reduce false positives, improve fidelity). This cycle repeats continuously as new threats emerge.

5. **How should threat intelligence be integrated into security monitoring?**
   > Enrich SIEM logs with IOC (Indicators of Compromise) lookups — match IPs, domains, hashes against threat feeds. Use feeds to populate firewall/WAF blocklists. Score IOC confidence to reduce noise. Filter feeds by sector and geography relevance. Age out old indicators. Combine with behavioural detection for unknown threats.

---

[← Previous: 10.1 Security Monitoring](01-security-monitoring.md) | [Next: 10.3 Incident Response →](03-incident-response.md)

[Back to Table of Contents](../README.md)
