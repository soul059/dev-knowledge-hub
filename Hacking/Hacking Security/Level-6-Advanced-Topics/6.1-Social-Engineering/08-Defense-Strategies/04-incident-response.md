# Unit 8: Defense Strategies — Topic 4: Incident Response for Social Engineering

## Overview

**Incident response (IR) for social engineering** requires specific procedures beyond standard technical IR because attacks target people and may span email, phone, physical, and digital channels simultaneously. Rapid response to social engineering incidents — especially phishing — can mean the difference between a contained event and a full-scale breach.

---

## 1. Social Engineering IR Framework

```
INCIDENT RESPONSE PHASES:

PHASE 1: PREPARATION
  ┌──────────────────────────────────┐
  │ → IR plan including SE scenarios │
  │ → Playbooks for phishing, BEC    │
  │ → Report button deployed         │
  │ → Team trained on SE response    │
  │ → Tools and access ready         │
  │ → Communication templates ready  │
  └──────────────────────────────────┘
              │
PHASE 2: DETECTION / IDENTIFICATION
  ┌──────────────────────────────────┐
  │ → Employee report received       │
  │ → Email gateway alert            │
  │ → SIEM correlation               │
  │ → Threat intel match             │
  │ → Unusual activity detected      │
  └──────────────────────────────────┘
              │
PHASE 3: CONTAINMENT
  ┌──────────────────────────────────┐
  │ → Block sender/domain            │
  │ → Remove email from all mailboxes│
  │ → Disable compromised accounts   │
  │ → Isolate affected endpoints     │
  │ → Block malicious URLs/IPs       │
  └──────────────────────────────────┘
              │
PHASE 4: ERADICATION
  ┌──────────────────────────────────┐
  │ → Remove malware if installed    │
  │ → Reset compromised credentials  │
  │ → Revoke compromised sessions    │
  │ → Patch exploited vulnerabilities│
  │ → Remove persistence mechanisms  │
  └──────────────────────────────────┘
              │
PHASE 5: RECOVERY
  ┌──────────────────────────────────┐
  │ → Restore affected accounts      │
  │ → Verify clean systems           │
  │ → Monitor for reinfection        │
  │ → Resume normal operations       │
  │ → Confirm no data exfiltration   │
  └──────────────────────────────────┘
              │
PHASE 6: LESSONS LEARNED
  ┌──────────────────────────────────┐
  │ → Document incident timeline     │
  │ → Identify root cause            │
  │ → Update detection rules         │
  │ → Improve training content       │
  │ → Update IR procedures           │
  │ → Brief stakeholders             │
  └──────────────────────────────────┘
```

---

## 2. Phishing Incident Playbook

```
PHISHING RESPONSE PLAYBOOK:

TRIGGER: Employee reports phishing email or 
         security tool detects phishing

IMMEDIATE (First 15 minutes):
  □ Acknowledge reporter
  □ Assess severity (targeted vs mass)
  □ Search mailboxes for same email
  □ Identify all recipients
  □ Determine who clicked/submitted
  □ Block sender and domain

SHORT-TERM (15-60 minutes):
  □ Remove email from all mailboxes
  □ Block malicious URL at proxy/firewall
  □ Reset passwords for affected users
  □ Invalidate active sessions
  □ Scan affected endpoints
  □ Alert affected employees

INVESTIGATION (1-24 hours):
  □ Analyze phishing email headers
  □ Analyze malicious URL/attachment
  □ Check for malware installation
  □ Review logs for lateral movement
  □ Check for data exfiltration
  □ Determine scope of compromise
  □ Collect IOCs for threat intel

FOLLOW-UP (24-72 hours):
  □ Complete incident report
  □ Update email filters
  □ Share IOCs with community
  □ Conduct lessons learned
  □ Provide targeted training
  □ Update playbook if needed
```

---

## 3. BEC/Wire Fraud Response

```
BEC RESPONSE PLAYBOOK:

TRIGGER: Suspected or confirmed BEC/wire fraud

IMMEDIATE (First 30 minutes):
  □ STOP any pending wire transfers
  □ Contact bank immediately
  □ Request wire recall/freeze
  □ Preserve all email evidence
  □ Do NOT reply to suspected BEC email
  □ Notify legal counsel

CRITICAL TIME: Wire transfer recall success
  → Within 24 hours: ~70% success
  → 24-48 hours: ~30% success  
  → 48+ hours: ~10% success
  → International wires: more difficult

INVESTIGATION:
  □ Determine if email account was compromised
  □ Check for email forwarding rules
  □ Examine inbox rules for auto-deletion
  □ Review recent email activity
  □ Determine if credentials were phished
  □ Check for OAuth app access grants

LAW ENFORCEMENT:
  □ File FBI IC3 complaint (ic3.gov)
  □ Contact local FBI field office
  □ File local law enforcement report
  □ Engage legal counsel
  □ Contact cyber insurance provider

RECOVERY:
  □ Secure all affected accounts
  □ Remove unauthorized inbox rules
  □ Revoke unauthorized OAuth apps
  □ Reset credentials + enable MFA
  □ Audit all recent financial transactions
  □ Update wire transfer procedures
```

---

## 4. Physical Social Engineering Response

```
PHYSICAL INCIDENT RESPONSE:

UNAUTHORIZED ACCESS DETECTED:
  □ Alert security team immediately
  □ Do NOT confront alone (safety first)
  □ Observe and note description
  □ Monitor on CCTV
  □ Contact building security/police if needed
  □ Lock down sensitive areas

POST-INCIDENT:
  □ Review CCTV footage
  □ Identify entry point and method
  □ Check for planted devices
    → Rogue WiFi access points
    → USB devices in machines
    → Keyloggers
    → Hidden cameras
  □ Audit what areas were accessed
  □ Review badge access logs
  □ Check for missing items/documents
  □ Sweep for electronic surveillance

REMEDIATION:
  □ Repair exploited access control
  □ Rekey locks if keys were copied
  □ Replace compromised badges
  □ Increase security at entry point
  □ Additional training for staff
  □ Update physical security procedures
```

---

## 5. Communication During Incidents

```
INTERNAL COMMUNICATION:

TO AFFECTED EMPLOYEES:
  Subject: Security Alert — Action Required

  "We have identified a phishing campaign 
  targeting our organization. If you received 
  an email about [subject], please:
  
  1. Do NOT click any links
  2. Do NOT open any attachments  
  3. Report it using the Report Phishing button
  4. Delete the email
  
  If you already clicked, please contact IT 
  security immediately at [phone/email].
  
  There will be no consequences for reporting."

TO MANAGEMENT:
  → Brief summary of incident
  → Number of employees affected
  → Current containment status
  → Business impact assessment
  → Expected resolution timeline
  → Next update scheduled

TO EXECUTIVE LEADERSHIP:
  → Business impact focus
  → Financial exposure if applicable
  → PR/media considerations
  → Regulatory reporting requirements
  → Resources needed for response
```

---

## Summary Table

| Incident Type | First Action | Critical Timeframe | Key Contact |
|--------------|-------------|-------------------|-------------|
| Phishing | Block sender + search mailboxes | 15 minutes | SOC/IR team |
| BEC/Wire fraud | Stop transfer + contact bank | 30 minutes | Bank + FBI |
| Credential compromise | Reset password + MFA | 1 hour | IT + SOC |
| Physical breach | Alert security | Immediate | Security + police |
| Malware via SE | Isolate endpoint | 15 minutes | SOC + IT |

---

## Revision Questions

1. How does social engineering IR differ from standard technical IR?
2. What are the immediate steps in a phishing incident response?
3. Why is time critical in BEC/wire fraud response?
4. What should be checked after a physical social engineering incident?
5. How should organizations communicate during a social engineering incident?

---

*Previous: [03-reporting-mechanisms.md](03-reporting-mechanisms.md) | Next: [05-technical-controls.md](05-technical-controls.md)*

---

*[Back to README](../README.md)*
