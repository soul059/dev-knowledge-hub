# Unit 9: A09 - Security Logging and Monitoring Failures — Topic 4: Incident Response Integration

## Overview

Logging and monitoring are only valuable when they **feed into an effective incident response (IR) process**. When alerts fire, the IR team must quickly triage, investigate, contain, and remediate security incidents using the logged data. This topic covers how to integrate logging infrastructure with IR workflows.

---

## 1. Incident Response Lifecycle

```
NIST SP 800-61 Incident Response Lifecycle:

1. PREPARATION
   ├── IR plan documented and tested
   ├── Logging infrastructure operational
   ├── Alert rules configured
   ├── Runbooks for common scenarios
   └── Team trained and on-call

2. DETECTION & ANALYSIS
   ├── Alert received from monitoring
   ├── Triage: Real incident or false positive?
   ├── Classify: Severity and type
   ├── Investigate: Analyze logs, determine scope
   └── Document: Timeline and findings

3. CONTAINMENT, ERADICATION, REMEDIATION
   ├── Short-term: Isolate affected systems
   ├── Evidence: Preserve logs and forensic data
   ├── Eradicate: Remove threat actor access
   ├── Remediate: Patch vulnerability, reset credentials
   └── Long-term: Prevent recurrence

4. POST-INCIDENT
   ├── Lessons learned meeting
   ├── Update detection rules
   ├── Improve logging coverage
   ├── Update IR playbooks
   └── Report to stakeholders/regulators
```

---

## 2. Log-Driven Investigation

```
INVESTIGATION WORKFLOW:

Alert: "Multiple failed logins followed by success from unusual IP"

Step 1: IDENTIFY the affected account
  Query: event_type=auth.* AND user_id=usr-456 AND time > -24h

Step 2: TIMELINE of events
  10:15:00 — Failed login from 203.0.113.50 (5 attempts)
  10:15:30 — Successful login from 203.0.113.50
  10:16:00 — Role changed from "viewer" to "admin"
  10:17:00 — Bulk data export initiated
  10:18:00 — New API key generated
  10:20:00 — Login from user's normal IP 192.168.1.100

Step 3: DETERMINE SCOPE
  - Which data was accessed/exported?
  - Were other accounts affected?
  - Is the attacker still active?

Step 4: CONTAIN
  - Invalidate all sessions for usr-456
  - Revoke the new API key
  - Revert role change
  - Block IP 203.0.113.50
```

---

## 3. Automated Response (SOAR)

```
SOAR: Security Orchestration, Automation, and Response

EXAMPLE PLAYBOOK — Account Compromise:

TRIGGER: Successful login after 5+ failures from same IP

AUTOMATED ACTIONS:
1. Enrich: Lookup IP reputation (VirusTotal, AbuseIPDB)
2. Check: Is IP in threat intelligence feeds?
3. Query: Other accounts targeted by this IP?
4. IF threat score > 0.7:
   a. Invalidate user sessions
   b. Require password reset
   c. Block IP at WAF
   d. Create incident ticket (Jira/ServiceNow)
   e. Notify user via email
   f. Alert SOC analyst

SOAR PLATFORMS:
├── Splunk SOAR (Phantom)
├── Palo Alto XSOAR
├── IBM Resilient
├── Tines (no-code)
└── Shuffle (open source)
```

---

## 4. Evidence Preservation

```
FOR FORENSIC INVESTIGATION:

DO:
✅ Preserve original logs (read-only copy)
✅ Document chain of custody
✅ Record timestamps of discovery and actions
✅ Take memory dumps if possible
✅ Capture network traffic (pcap)
✅ Screenshot active sessions before termination

DON'T:
❌ Modify or delete logs
❌ Reboot compromised systems (destroys memory)
❌ Run antivirus scans (overwrites evidence)
❌ Let the attacker know they're detected
❌ Investigate on the compromised system directly
```

---

## 5. IR Communication Template

```
INCIDENT NOTIFICATION:

Subject: [SEVERITY] Security Incident - [TYPE]
Date/Time: 2024-01-15 10:15 UTC
Status: Active / Contained / Resolved

SUMMARY:
Unauthorized access to user account usr-456 detected.
Attacker accessed admin panel and exported customer data.

TIMELINE:
- 10:15 — Brute force attack detected
- 10:15 — Account compromised  
- 10:17 — Data export initiated
- 10:25 — Alert triggered
- 10:30 — Account locked, sessions invalidated
- 10:45 — IP blocked, investigation ongoing

IMPACT:
- 1 account compromised
- ~5,000 customer records potentially exposed
- No financial data accessed

ACTIONS TAKEN:
- Account locked and sessions invalidated
- Attacker IP blocked
- Affected data identified
- Forensic evidence preserved

NEXT STEPS:
- Complete forensic investigation
- Assess notification requirements (GDPR: 72 hours)
- Password reset for affected user
- Review and enhance monitoring rules
```

---

## Summary Table

| IR Phase | Logging Role | Key Actions |
|----------|-------------|-------------|
| Preparation | Configure logging and alerting | Runbooks, alert rules |
| Detection | Alert triggers investigation | Log analysis, triage |
| Containment | Log evidence preservation | Isolate, block, preserve |
| Eradication | Root cause from logs | Remove access, patch |
| Recovery | Verify through monitoring | Restore, monitor closely |
| Post-Incident | Improve detection rules | Update logging, lessons learned |

---

## Revision Questions

1. Map the NIST IR lifecycle and explain the role of logging at each phase.
2. Given an alert for "successful login after brute force," write the investigation steps using log queries.
3. Design an automated SOAR playbook for account compromise detection and response.
4. What evidence should be preserved during an incident? What should NOT be done?
5. Write an incident communication template for a data breach notification.

---

*Previous: [03-monitoring-and-alerting.md](03-monitoring-and-alerting.md) | Next: [05-log-injection-prevention.md](05-log-injection-prevention.md)*

---

*[Back to README](../README.md)*
