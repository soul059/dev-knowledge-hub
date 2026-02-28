# Chapter 10.3: Incident Response

## Overview

Incident response (IR) is the structured approach to handling security breaches, cyberattacks, and policy violations. In DevSecOps, incident response integrates with development workflows — using automation, infrastructure as code, and CI/CD capabilities to detect, contain, and remediate incidents faster. This chapter covers IR frameworks, playbooks, and automation.

---

## Incident Response Lifecycle (NIST SP 800-61)

```
NIST INCIDENT RESPONSE LIFECYCLE:

  ┌──────────────────┐
  │  1. PREPARATION  │◀─────────────────────────────┐
  │                  │                               │
  │ • IR plan        │     ┌──────────────────┐     │
  │ • Team roles     │     │  4. POST-         │     │
  │ • Tools ready    │     │  INCIDENT         │     │
  │ • Runbooks       │     │                    │     │
  │ • Training       │     │ • Lessons learned  │     │
  └────────┬─────────┘     │ • Process update   │     │
           │               │ • Detection tune   │     │
           ▼               │ • Report           │     │
  ┌──────────────────┐     └────────┬───────────┘     │
  │ 2. DETECTION &   │              │                  │
  │    ANALYSIS      │              │                  │
  │                  │              │                  │
  │ • Alert triage   │     ┌───────┴──────────┐       │
  │ • Severity class │     │ 3. CONTAINMENT,  │       │
  │ • Scope analysis │     │    ERADICATION,  │       │
  │ • Evidence       │────▶│    RECOVERY      │───────┘
  │   collection     │     │                  │
  └──────────────────┘     │ • Isolate        │
                           │ • Remove threat  │
                           │ • Restore        │
                           │ • Validate       │
                           └──────────────────┘
```

---

## Incident Severity Classification

```
SEVERITY LEVELS:

  ┌────────────────────────────────────────────────────────┐
  │ SEV 1 — CRITICAL                          Response: 15m│
  │ • Active data breach / exfiltration                    │
  │ • Production system compromised                        │
  │ • Ransomware / destructive malware                     │
  │ • Customer data exposed                                │
  ├────────────────────────────────────────────────────────┤
  │ SEV 2 — HIGH                              Response: 1h │
  │ • Unauthorized access detected                         │
  │ • Vulnerability actively exploited                     │
  │ • Security control bypassed                            │
  │ • Internal credentials compromised                     │
  ├────────────────────────────────────────────────────────┤
  │ SEV 3 — MEDIUM                            Response: 4h │
  │ • Suspicious activity (unconfirmed threat)             │
  │ • Policy violation detected                            │
  │ • Vulnerability discovered (not exploited)             │
  │ • Failed attack attempts                               │
  ├────────────────────────────────────────────────────────┤
  │ SEV 4 — LOW                              Response: 24h │
  │ • Informational security events                        │
  │ • Minor policy drift detected                          │
  │ • Routine vulnerability scan findings                  │
  │ • Non-critical misconfiguration                        │
  └────────────────────────────────────────────────────────┘
```

---

## Incident Response Playbooks

### Playbook: Compromised Credentials

```
PLAYBOOK: COMPROMISED CREDENTIALS

  TRIGGER: Secret detected in public repo / credential
           stuffing alert / unusual login pattern

  ┌──────────────────────────────────────────────────┐
  │ STEP 1: CONTAIN (< 15 minutes)                    │
  │                                                    │
  │ □ Immediately rotate/revoke the credential        │
  │ □ Disable associated user/service account          │
  │ □ Block suspicious source IPs                      │
  │ □ Enable enhanced logging on affected systems     │
  │ □ Notify Incident Commander                        │
  └──────────────────────────────────────────────────┘
            │
            ▼
  ┌──────────────────────────────────────────────────┐
  │ STEP 2: ASSESS (< 1 hour)                         │
  │                                                    │
  │ □ Determine what the credential had access to     │
  │ □ Review audit logs for unauthorized usage         │
  │ □ Check for lateral movement                       │
  │ □ Identify when credential was first exposed      │
  │ □ Determine blast radius                           │
  └──────────────────────────────────────────────────┘
            │
            ▼
  ┌──────────────────────────────────────────────────┐
  │ STEP 3: ERADICATE (< 4 hours)                     │
  │                                                    │
  │ □ Rotate ALL potentially affected credentials     │
  │ □ Remove credential from source (Git history)     │
  │ □ Revoke all sessions for the compromised user    │
  │ □ Patch the vulnerability that caused exposure    │
  │ □ Update secret detection rules                    │
  └──────────────────────────────────────────────────┘
            │
            ▼
  ┌──────────────────────────────────────────────────┐
  │ STEP 4: RECOVER & LEARN (< 24 hours)               │
  │                                                    │
  │ □ Verify all systems operating normally            │
  │ □ Confirm no persistent backdoors                  │
  │ □ Write post-mortem report                         │
  │ □ Update pre-commit hooks / CI scanning            │
  │ □ Schedule team retrospective                      │
  └──────────────────────────────────────────────────┘
```

### Playbook: Container Escape

```bash
#!/bin/bash
# container-escape-response.sh
# Automated response for container escape detection

CONTAINER_ID=$1
NODE_NAME=$2
NAMESPACE=$3

echo "[INCIDENT] Container escape detected: $CONTAINER_ID on $NODE_NAME"

# Step 1: Isolate — cordon the node
kubectl cordon $NODE_NAME
echo "[CONTAIN] Node $NODE_NAME cordoned — no new pods scheduled"

# Step 2: Network isolate — apply deny-all policy
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: emergency-isolate
  namespace: $NAMESPACE
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
EOF
echo "[CONTAIN] Network isolation applied to namespace $NAMESPACE"

# Step 3: Capture forensic data
kubectl logs $CONTAINER_ID -n $NAMESPACE > /tmp/forensics/container-logs.txt
kubectl describe pod $CONTAINER_ID -n $NAMESPACE > /tmp/forensics/pod-describe.txt
kubectl get events -n $NAMESPACE --sort-by='.lastTimestamp' > /tmp/forensics/events.txt
echo "[FORENSIC] Evidence captured"

# Step 4: Kill compromised pod
kubectl delete pod $CONTAINER_ID -n $NAMESPACE --grace-period=0 --force
echo "[ERADICATE] Compromised pod terminated"

# Step 5: Alert
curl -X POST https://hooks.slack.com/services/xxx/yyy/zzz \
    -H 'Content-type: application/json' \
    -d '{
      "text": "🚨 CONTAINER ESCAPE DETECTED\nNode: '"$NODE_NAME"'\nContainer: '"$CONTAINER_ID"'\nNamespace: '"$NAMESPACE"'\nStatus: Contained — awaiting investigation"
    }'
```

---

## SOAR (Security Orchestration, Automation and Response)

```
SOAR AUTOMATION PIPELINE:

  Alert ──▶ Triage ──▶ Enrich ──▶ Decide ──▶ Respond
    │         │          │          │           │
    │         │          │          │           │
    ▼         ▼          ▼          ▼           ▼
  SIEM    Deduplicate  Query:     Auto or    Execute:
  Falco   Correlate   • VirusTotal human    • Block IP
  Guard   Priority    • Shodan    decision  • Rotate cred
  Duty    Assign      • WHOIS    based on  • Isolate host
                      • CMDB     severity  • Create ticket
                      • Vault
```

```yaml
# Example SOAR playbook (Tines/Shuffle-like workflow)
name: Automated Credential Leak Response
trigger:
  source: github-secret-scanning
  event: secret_detected

steps:
  - name: classify_secret
    action: analyze
    input: "{{ trigger.secret_type }}"
    conditions:
      - if: secret_type == "aws_access_key"
        severity: critical
      - if: secret_type == "generic_password"
        severity: high

  - name: disable_credential
    action: aws_iam
    condition: severity == "critical"
    command: |
      aws iam update-access-key \
        --access-key-id {{ trigger.secret_value | extract_key_id }} \
        --status Inactive \
        --user-name {{ lookup.iam_user }}

  - name: create_ticket
    action: jira
    project: SEC
    issue_type: Incident
    summary: "Credential leak: {{ trigger.secret_type }}"
    priority: "{{ severity }}"
    description: |
      Secret detected in repository {{ trigger.repo }}
      Commit: {{ trigger.commit }}
      File: {{ trigger.file }}
      Action taken: Credential disabled

  - name: notify_team
    action: slack
    channel: "#security-incidents"
    message: |
      🚨 *Credential Leak Detected*
      Type: {{ trigger.secret_type }}
      Repo: {{ trigger.repo }}
      Status: Auto-contained
      Ticket: {{ steps.create_ticket.ticket_url }}
```

---

## Evidence Collection and Forensics

```bash
# Digital forensics evidence collection script
#!/bin/bash

INCIDENT_ID=$1
EVIDENCE_DIR="/secure/evidence/$INCIDENT_ID"
mkdir -p $EVIDENCE_DIR

echo "=== Collecting forensic evidence for incident $INCIDENT_ID ==="

# System state
date -u > $EVIDENCE_DIR/timestamp.txt
hostname >> $EVIDENCE_DIR/timestamp.txt
uname -a > $EVIDENCE_DIR/system-info.txt

# Running processes
ps auxww > $EVIDENCE_DIR/processes.txt

# Network connections
ss -tunap > $EVIDENCE_DIR/network-connections.txt
netstat -rn > $EVIDENCE_DIR/routing-table.txt

# User activity
who > $EVIDENCE_DIR/logged-in-users.txt
last -50 > $EVIDENCE_DIR/recent-logins.txt
cat /var/log/auth.log | tail -1000 > $EVIDENCE_DIR/auth-logs.txt

# File system
find / -mtime -1 -type f 2>/dev/null > $EVIDENCE_DIR/recently-modified.txt
find / -perm -4000 -type f 2>/dev/null > $EVIDENCE_DIR/suid-files.txt

# Container-specific
docker ps -a > $EVIDENCE_DIR/containers.txt 2>/dev/null
docker images > $EVIDENCE_DIR/images.txt 2>/dev/null

# Create integrity hash
find $EVIDENCE_DIR -type f -exec sha256sum {} \; > $EVIDENCE_DIR/evidence-hashes.txt

echo "=== Evidence collected at $EVIDENCE_DIR ==="
echo "=== DO NOT MODIFY FILES — chain of custody ==="
```

---

## Post-Incident Review Template

```markdown
# Post-Incident Review: [INCIDENT-ID]

## Summary
- **Date**: YYYY-MM-DD
- **Duration**: Detection → Resolution time
- **Severity**: SEV-1/2/3/4
- **Impact**: Number of affected users/systems/records

## Timeline
| Time (UTC) | Event |
|------------|-------|
| HH:MM | Initial alert triggered |
| HH:MM | Incident declared, IC assigned |
| HH:MM | Containment actions taken |
| HH:MM | Root cause identified |
| HH:MM | Eradication complete |
| HH:MM | Systems restored |
| HH:MM | Incident closed |

## Root Cause Analysis
[5 Whys or fishbone diagram]

## What Went Well
- [Item 1]
- [Item 2]

## What Needs Improvement
- [Item 1]
- [Item 2]

## Action Items
| # | Action | Owner | Due Date | Status |
|---|--------|-------|----------|--------|
| 1 | Update detection rules | Security | YYYY-MM-DD | Open |
| 2 | Fix root cause | Engineering | YYYY-MM-DD | Open |
| 3 | Update runbook | SRE | YYYY-MM-DD | Open |
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Incident takes too long to detect | Missing detection rules; blind spots | Map detection coverage to MITRE ATT&CK; run red team exercises; add missing telemetry |
| Response is slow and uncoordinated | No documented playbooks; unclear roles | Create playbooks for top 10 scenarios; assign IC role; practice with tabletop exercises |
| Evidence lost or contaminated | Responders modify affected systems | Train on forensic best practices; use read-only access; automate evidence collection scripts |
| Same incident type keeps recurring | Root cause not properly addressed | Enforce action items from post-mortems; track remediation; blameless culture |
| SOAR automation triggers false actions | Over-aggressive automation without human gate | Add human approval for destructive actions; use progressive automation (alert → quarantine → block) |

---

## Summary Table

| IR Phase | Activities | DevSecOps Tools |
|----------|-----------|----------------|
| **Preparation** | Plans, runbooks, training, tools | GitOps-managed playbooks, IaC for forensic infra |
| **Detection** | Monitoring, alerting, triage | SIEM, Falco, GuardDuty, Prometheus alerts |
| **Containment** | Isolate, block, disable | K8s network policies, SG updates, cred revocation |
| **Eradication** | Remove threat, patch | CI/CD hotfix pipeline, IaC rebuild |
| **Recovery** | Restore, verify, monitor | IaC redeploy, enhanced monitoring |
| **Post-Incident** | Review, improve, train | Blameless post-mortem, detection rule updates |

---

## Quick Revision Questions

1. **What are the four phases of NIST incident response?**
   > Preparation (plans, tools, training), Detection and Analysis (identify and classify incidents), Containment/Eradication/Recovery (stop the threat, remove it, restore operations), and Post-Incident Activity (lessons learned, process improvement, reporting). These phases form a cycle where lessons from each incident improve preparation.

2. **What should happen in the first 15 minutes of a SEV-1 incident?**
   > Immediately contain the threat — revoke compromised credentials, isolate affected systems (network isolation, cordon K8s nodes), block attacker IPs, enable enhanced logging, and notify the Incident Commander. Focus is on stopping the bleed, not root cause analysis. Document every action taken with timestamps.

3. **What is SOAR and how does it help incident response?**
   > Security Orchestration, Automation, and Response (SOAR) automates repetitive IR tasks — alert triage, IOC enrichment (VirusTotal, WHOIS lookups), credential revocation, firewall rule updates, ticket creation, and team notification. It reduces response time from hours to minutes, ensures consistent response, and frees analysts to focus on complex decisions.

4. **Why are blameless post-mortems important?**
   > Blameless post-mortems focus on systemic failures rather than individual mistakes. This encourages transparency — people share what actually happened without fear of punishment. This leads to better root cause analysis, more effective preventive measures, and a culture where security improvements are driven by learning rather than blame.

5. **How should forensic evidence be preserved during an incident?**
   > Capture system state before making changes (processes, network connections, logs, file modifications). Use read-only access where possible. Store evidence with integrity hashes (SHA-256) in a secure, access-controlled location. Maintain chain of custody documentation. Automate evidence collection with pre-built scripts to reduce human error and speed.

---

[← Previous: 10.2 Threat Detection](02-threat-detection.md) | [Next: 10.4 Penetration Testing →](04-penetration-testing.md)

[Back to Table of Contents](../README.md)
