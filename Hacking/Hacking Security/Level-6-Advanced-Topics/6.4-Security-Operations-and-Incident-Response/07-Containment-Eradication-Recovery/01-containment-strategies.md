# Unit 7: Containment, Eradication, Recovery — Topic 1: Containment Strategies

## Overview

**Containment** is the IR phase focused on stopping the attacker from causing further damage while preserving evidence. Effective containment limits the blast radius, prevents data loss, and buys time for eradication and recovery. The challenge is containing the threat without alerting the attacker or disrupting business operations unnecessarily.

---

## 1. Containment Principles

```
CONTAINMENT PHILOSOPHY:

  ┌─────────────────────────────────────────┐
  │          CONTAINMENT GOALS              │
  │                                         │
  │  1. Stop attacker advancement           │
  │  2. Prevent data exfiltration           │
  │  3. Preserve evidence                   │
  │  4. Minimize business disruption        │
  │  5. Maintain operational capability     │
  │  6. Enable eradication planning         │
  └─────────────────────────────────────────┘

  CONTAINMENT DECISION FACTORS:
  → Potential damage if not contained
  → Need to preserve evidence
  → Business impact of containment action
  → Available service alternatives
  → Duration of containment needed
  → Resources required for containment
  → Time of day / business hours
  → Attacker awareness risk

CONTAINMENT APPROACHES:
  Approach      | Description          | Risk
  Observe       | Monitor only         | Damage continues
  Restrict      | Limit access/traffic | Partial contain
  Isolate       | Full network/host    | Business impact
  Shutdown      | Power off / disable  | Max disruption
```

---

## 2. Network Containment

```
NETWORK CONTAINMENT STRATEGIES:

FIREWALL BLOCKING:
  → Block attacker source IPs
  → Block C2 destination IPs/domains
  → Block lateral movement ports
  → Implement geo-blocking
  → Enable stricter egress filtering

  # Palo Alto firewall - block C2 IP
  set security policies from-zone trust to-zone untrust
    source any destination 203.0.113.50
    action deny
  
  # Windows Firewall - block outbound
  New-NetFirewallRule -DisplayName "Block C2" `
    -Direction Outbound `
    -RemoteAddress 203.0.113.50 `
    -Action Block

DNS SINKHOLING:
  → Redirect C2 domains to internal server
  → Identifies all systems beaconing to C2
  → Non-disruptive containment method
  → Can collect connection metadata

  # DNS sinkhole entry
  evil-domain.com  IN A 10.0.0.1  (sinkhole IP)
  *.evil-domain.com IN A 10.0.0.1

VLAN ISOLATION:
  → Move compromised systems to quarantine VLAN
  → Allows continued monitoring
  → Maintains evidence collection
  → Can provide limited network access

  # Switch configuration (Cisco)
  interface GigabitEthernet0/1
    switchport access vlan 999  ! quarantine VLAN
    description QUARANTINE-INC2024001

NETWORK SEGMENTATION:
  → Implement emergency ACLs
  → Block inter-VLAN traffic for affected segments
  → Restrict management network access
  → Enable enhanced monitoring on segment boundaries
```

---

## 3. Endpoint Containment

```
ENDPOINT CONTAINMENT:

EDR ISOLATION:
  → Network isolate via EDR agent
  → Host remains powered on
  → Management channel maintained
  → Evidence preserved
  → Investigation continues

  CrowdStrike:
  → Host Management > Network Containment
  → Host isolated from network
  → Only talks to CrowdStrike cloud
  → Can run RTR commands remotely

  SentinelOne:
  → Sentinels > Select Host > Network Quarantine
  → Full isolation with agent communication
  → Remote shell available

  Microsoft Defender for Endpoint:
  → Device page > Isolate Device
  → Full or selective isolation
  → Remote investigation commands

HOST-LEVEL ACTIONS:
  → Disable compromised user accounts
  → Block malicious processes
  → Remove scheduled tasks
  → Disable services
  → Block IPs at host firewall
  → Quarantine malicious files

  # Disable user account (AD)
  Disable-ADAccount -Identity jsmith
  
  # Kill malicious process
  Stop-Process -Name "malware" -Force
  
  # Block at host firewall
  netsh advfirewall firewall add rule name="BlockC2" `
    dir=out action=block remoteip=203.0.113.50

ACCOUNT CONTAINMENT:
  → Disable compromised accounts
  → Reset passwords (affected + admin)
  → Revoke active sessions/tokens
  → Disable service accounts if safe
  → Enable conditional access policies
  → Force re-authentication

  # Azure AD - Revoke sessions
  Revoke-AzureADUserAllRefreshToken `
    -ObjectId "jsmith@company.com"
  
  # Reset AD password
  Set-ADAccountPassword -Identity jsmith `
    -Reset -NewPassword (ConvertTo-SecureString `
    "TempP@ss2024!" -AsPlainText -Force)
```

---

## 4. Containment Documentation

```
CONTAINMENT LOG:

  ┌────────────────────────────────────────────┐
  │ CONTAINMENT ACTIONS LOG                    │
  │ Incident: INC-2024-001                     │
  │                                            │
  │ Time (UTC)  | Action        | Target       │
  │ 11:00       | Account       | jsmith       │
  │             | disabled      | disabled in AD│
  │ 11:05       | Firewall rule | 203.0.113.50 │
  │             | created       | blocked       │
  │ 11:10       | EDR isolation | JSMITH-PC    │
  │             | applied       | isolated      │
  │ 11:15       | DNS sinkhole  | evil.com     │
  │             | created       | sinkholed     │
  │ 11:20       | EDR isolation | FILESVR01    │
  │             | applied       | isolated      │
  │ 11:25       | VLAN change   | Port Gi0/1   │
  │             | applied       | VLAN 999      │
  │ 11:30       | Password      | Domain admin │
  │             | reset         | all reset     │
  │                                            │
  │ IMPACT ASSESSMENT:                         │
  │ → JSMITH-PC: User issued replacement       │
  │ → FILESVR01: Users redirected to backup    │
  │ → Email: jsmith mailbox suspended          │
  │                                            │
  │ DECISIONS:                                 │
  │ → Decision: Isolate vs. observe            │
  │ → Chosen: Isolate (active data exfil)      │
  │ → Approved by: SOC Manager (J. Smith)      │
  └────────────────────────────────────────────┘

CONTAINMENT CHECKLIST:
  [ ] Compromised accounts disabled
  [ ] Compromised systems isolated
  [ ] C2 traffic blocked (firewall)
  [ ] C2 domains sinkholed (DNS)
  [ ] Credentials reset (affected accounts)
  [ ] Admin credentials reset
  [ ] Active sessions revoked
  [ ] Enhanced monitoring enabled
  [ ] Containment verified (no more C2)
  [ ] Business impact communicated
  [ ] Stakeholders notified
```

---

## 5. Containment Challenges

```
COMMON CHALLENGES:

  Challenge               | Mitigation
  Attacker notices contain| Stage containment, execute
  -ment and escalates     | all actions simultaneously
  Business impact too high| Use least disruptive method
                          | Escalate to management
  Scope unknown           | Contain what you know,
                          | continue scoping
  Cloud/hybrid environ    | Coordinate across on-prem
                          | and cloud teams
  Third-party systems     | Coordinate with vendors,
                          | have contacts ready
  After-hours incident    | Have on-call procedures,
                          | pre-authorized actions

COORDINATED CONTAINMENT:
  For sophisticated attacks, execute all
  containment simultaneously to prevent
  attacker reaction:

  STEP 1: Plan all containment actions
  STEP 2: Assign team members to each action
  STEP 3: Countdown to coordinated execution
  STEP 4: All actions executed within minutes
  STEP 5: Verify containment effectiveness
  STEP 6: Monitor for attacker response

  Timeline:
  11:00 - Planning complete, team briefed
  11:15 - Ready check, all positions confirmed
  11:16 - GO: All containment actions executed
  11:20 - All actions confirmed complete
  11:25 - Containment verification started
  11:30 - No C2 traffic, containment confirmed
```

---

## Summary Table

| Strategy | Method | Disruption | Evidence |
|----------|--------|-----------|----------|
| Firewall Block | Block IPs/ports | Low | Preserved |
| DNS Sinkhole | Redirect C2 domains | Low | Enhanced |
| EDR Isolation | Network quarantine | Medium | Preserved |
| VLAN Isolation | Move to quarantine VLAN | Medium | Preserved |
| Account Disable | Disable/reset accounts | Medium | Preserved |
| System Shutdown | Power off | High | At risk |

---

## Revision Questions

1. What factors influence the choice of containment strategy?
2. How does DNS sinkholing assist in containment?
3. What is coordinated containment and when is it used?
4. What account actions should be taken during containment?
5. How do you balance containment with business impact?

---

*Previous: None (First topic in this unit) | Next: [02-short-term-vs-long-term-containment.md](02-short-term-vs-long-term-containment.md)*

---

*[Back to README](../README.md)*
