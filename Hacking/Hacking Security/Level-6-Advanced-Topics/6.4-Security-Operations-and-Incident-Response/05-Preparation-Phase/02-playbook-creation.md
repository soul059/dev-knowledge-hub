# Unit 5: Preparation Phase — Topic 2: Playbook Creation

## Overview

**Playbooks** (also called runbooks) provide step-by-step procedures for handling specific incident types. Well-crafted playbooks ensure consistent, efficient responses regardless of which analyst is on shift. They bridge the gap between high-level IR plans and tactical execution.

---

## 1. Playbook Fundamentals

```
PLAYBOOK STRUCTURE:

  ┌──────────────────────────────────────────┐
  │  PLAYBOOK: PB-MAL-001                   │
  │  Title: Malware Infection Response       │
  │                                          │
  │  METADATA                                │
  │  → Version: 2.3                         │
  │  → Last Updated: 2024-06-01             │
  │  → Author: IR Team                      │
  │  → Severity: High                        │
  │  → ATT&CK: T1204, T1059                │
  │  → Estimated Time: 2-4 hours            │
  │                                          │
  │  TRIGGER                                 │
  │  → EDR malware detection alert           │
  │  → AV quarantine notification            │
  │  → User-reported suspicious activity     │
  │                                          │
  │  STEPS (see detailed sections)           │
  │  1. Initial Triage                       │
  │  2. Containment                          │
  │  3. Investigation                        │
  │  4. Eradication                          │
  │  5. Recovery                             │
  │  6. Documentation                        │
  │                                          │
  │  ESCALATION CRITERIA                     │
  │  → Multiple systems infected             │
  │  → Ransomware detected                   │
  │  → Data exfiltration confirmed           │
  │  → Critical system affected              │
  └──────────────────────────────────────────┘

PLAYBOOK TYPES:
  Type         | Description
  Detection    | How to investigate an alert type
  Response     | How to respond to incident type
  Hunting      | How to hunt for specific threat
  Operational  | Routine SOC procedures
```

---

## 2. Essential Playbooks

```
MUST-HAVE PLAYBOOKS:

PB-PHI-001: PHISHING RESPONSE
  Step 1: Gather email details (sender, subject, URLs, attachments)
  Step 2: Check if email is in quarantine or delivered
  Step 3: Search for all recipients of the message
  Step 4: Identify who clicked links or opened attachments
  Step 5: Check URL reputation (VirusTotal, URLhaus)
  Step 6: Check attachment hash (VirusTotal, sandbox)
  Step 7: Block sender/domain at email gateway
  Step 8: Remove email from all mailboxes
  Step 9: Block malicious URLs at proxy/firewall
  Step 10: If credentials entered: Reset passwords, check for misuse
  Step 11: If malware: Execute PB-MAL-001
  Step 12: Document findings and close ticket

PB-MAL-001: MALWARE RESPONSE
  Step 1: Verify alert (true positive?)
  Step 2: Identify affected system(s)
  Step 3: Isolate system from network (EDR isolation)
  Step 4: Capture memory if needed
  Step 5: Identify malware (hash, family, capabilities)
  Step 6: Determine infection vector
  Step 7: Search for IOCs across environment
  Step 8: Block IOCs (hash, C2 IP/domain)
  Step 9: Clean or reimage affected system
  Step 10: Reset credentials used on affected system
  Step 11: Verify no lateral movement
  Step 12: Return system to production
  Step 13: Enhanced monitoring for 72 hours

PB-ACC-001: ACCOUNT COMPROMISE
  Step 1: Disable compromised account
  Step 2: Revoke all active sessions and tokens
  Step 3: Review sign-in logs for unauthorized access
  Step 4: Check for mailbox rules (forwarding, delegation)
  Step 5: Check for OAuth app consents
  Step 6: Review actions taken by the account
  Step 7: Assess data access (what was viewed/downloaded)
  Step 8: Remove any persistence
  Step 9: Reset password and re-enroll MFA
  Step 10: Re-enable account
  Step 11: Notify user and manager
  Step 12: Enhanced monitoring

PB-RAN-001: RANSOMWARE
  Step 1: ISOLATE affected systems immediately
  Step 2: Identify ransomware variant (ransom note, extension)
  Step 3: Determine scope (how many systems affected)
  Step 4: Notify IR manager → escalate to CRITICAL
  Step 5: Engage external IR if needed
  Step 6: Notify legal counsel
  Step 7: Assess backup integrity and availability
  Step 8: Collect samples for analysis
  Step 9: Check for decryptor availability
  Step 10: Begin recovery from clean backups
  Step 11: Rebuild from known-good images
  Step 12: Harden to prevent recurrence
```

---

## 3. Playbook Design Best Practices

```
DESIGN PRINCIPLES:

CLEAR AND ACTIONABLE:
  ✓ Each step is a specific action
  ✓ Include exact commands/tools
  ✓ Decision points clearly marked
  ✓ Expected outcomes stated
  ✓ Links to relevant tools/resources

  Good Step:
  "Run VirusTotal lookup on file hash using
  the SOAR integration. If score > 5/70,
  proceed to Step 5 (Containment)."

  Bad Step:
  "Check if the file is malicious."

DECISION TREES:
  ┌──────────┐
  │ Alert    │
  │ Received │
  └────┬─────┘
       │
  ┌────▼─────────┐
  │ Is it a true │
  │ positive?    │
  └───┬──────┬───┘
     Yes     No
      │       │
  ┌───▼──┐ ┌──▼───────┐
  │Invest│ │Close as  │
  │igate │ │FP, tune  │
  └───┬──┘ │rule      │
      │    └──────────┘
  ┌───▼──────────┐
  │ Is system    │
  │ critical?    │
  └───┬──────┬───┘
     Yes     No
      │       │
  ┌───▼──┐ ┌──▼──────┐
  │Esc to│ │Standard │
  │IR Mgr│ │response │
  └──────┘ └─────────┘

INCLUDE IN EACH PLAYBOOK:
  [ ] Unique identifier (PB-XXX-NNN)
  [ ] Clear trigger conditions
  [ ] Step-by-step procedures
  [ ] Decision points with criteria
  [ ] Escalation conditions
  [ ] Evidence collection steps
  [ ] Communication requirements
  [ ] Tools and access needed
  [ ] Expected timeline
  [ ] Close/documentation requirements
  [ ] Links to related playbooks
  [ ] ATT&CK technique mapping
```

---

## 4. SOAR Playbook Automation

```
AUTOMATING PLAYBOOKS WITH SOAR:

AUTOMATION CANDIDATES:
  → Phishing triage (URL/hash lookups)
  → IOC blocking (firewall/proxy rules)
  → Account disabling
  → Endpoint isolation
  → Evidence collection
  → Notification/escalation
  → Ticket creation and updates
  → Enrichment lookups

AUTOMATION LEVELS:
  Level 0: Fully manual (no automation)
  Level 1: Automated enrichment only
  Level 2: Semi-automated (human decisions)
  Level 3: Fully automated (human oversight)

EXAMPLE: AUTOMATED PHISHING TRIAGE
  Trigger: Email reported as phishing
  
  Auto Steps:
  1. Extract URLs, attachments, headers
  2. Check URL reputation (VT, URLhaus)
  3. Sandbox attachment (if present)
  4. Check sender reputation
  5. Score risk (low/medium/high)
  
  Decision Point (Human):
  → Low risk: Auto-close, notify reporter
  → Medium risk: Assign to Tier 1
  → High risk: Assign to Tier 2 + alert
  
  Auto Followup:
  6. If confirmed: Block sender
  7. Search and remove from mailboxes
  8. Block malicious URLs
  9. Create ticket with full analysis
  10. Notify reporter of outcome

ROI CALCULATION:
  Manual phishing triage: 30 minutes
  Automated triage: 3 minutes
  Daily phishing reports: 40
  Time saved: 18 hours/day
  Annual savings: ~$200K+ (analyst time)
```

---

## 5. Playbook Maintenance

```
PLAYBOOK LIFECYCLE:

  Create → Review → Test → Deploy → Monitor → Update

REVIEW TRIGGERS:
  → Quarterly scheduled review
  → After using playbook in real incident
  → New tools or processes implemented
  → Organizational changes
  → New threat types identified
  → Analyst feedback received
  → After tabletop exercise

METRICS TO TRACK:
  → Playbook usage frequency
  → Average completion time
  → Steps most often skipped
  → Escalation rate from playbook
  → Analyst satisfaction rating
  → False positive rate for trigger
  → Time to first containment action

PLAYBOOK INVENTORY:
  ID        | Title              | Last Review | Status
  PB-PHI-001| Phishing Response  | 2024-03-01 | Current
  PB-MAL-001| Malware Response   | 2024-02-15 | Current
  PB-ACC-001| Account Compromise | 2024-04-01 | Current
  PB-RAN-001| Ransomware         | 2024-01-10 | Review due
  PB-DOS-001| DDoS Response      | 2023-12-01 | Outdated
  PB-INS-001| Insider Threat     | 2024-03-20 | Current
  PB-WEB-001| Web App Attack     | 2024-02-28 | Current
```

---

## Summary Table

| Playbook | Trigger | Severity | Automation Potential |
|----------|---------|----------|---------------------|
| Phishing | Email report/alert | Medium-High | High |
| Malware | EDR/AV alert | High | Medium |
| Account Compromise | Auth alert | High | Medium |
| Ransomware | EDR/user report | Critical | Low |
| DDoS | Network alert | High | Medium |
| Insider Threat | UEBA/HR report | Medium-High | Low |

---

## Revision Questions

1. What are the essential components of an IR playbook?
2. Which playbooks should every organization have?
3. How do decision trees improve playbook usability?
4. What playbook steps are best candidates for SOAR automation?
5. How should playbooks be maintained over time?

---

*Previous: [01-ir-plan-development.md](01-ir-plan-development.md) | Next: [03-tool-preparation.md](03-tool-preparation.md)*

---

*[Back to README](../README.md)*
