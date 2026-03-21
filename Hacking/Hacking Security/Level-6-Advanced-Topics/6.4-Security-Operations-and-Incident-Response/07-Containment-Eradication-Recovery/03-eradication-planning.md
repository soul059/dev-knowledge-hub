# Unit 7: Containment, Eradication, Recovery — Topic 3: Eradication Planning

## Overview

**Eradication** removes the threat completely from the environment — all malware, backdoors, compromised accounts, and attacker footholds must be eliminated. Eradication planning ensures the removal is thorough, coordinated, and doesn't create new vulnerabilities or tip off the attacker.

---

## 1. Eradication Planning Process

```
ERADICATION PLANNING:

  ┌────────────────────────────────────────────┐
  │         ERADICATION WORKFLOW               │
  │                                            │
  │  1. SCOPE FINALIZED                        │
  │     All compromised systems identified     │
  │     All persistence mechanisms mapped      │
  │     All compromised accounts listed        │
  │                                            │
  │  2. ERADICATION PLAN DEVELOPED             │
  │     Actions for each system                │
  │     Actions for each account               │
  │     Sequence and timing                    │
  │     Resources assigned                     │
  │     Rollback plan                          │
  │                                            │
  │  3. PLAN REVIEWED AND APPROVED             │
  │     IR team review                         │
  │     Management approval                   │
  │     Legal/compliance review               │
  │                                            │
  │  4. COORDINATED EXECUTION                  │
  │     All actions executed simultaneously    │
  │     Verification after each action         │
  │     Communication maintained               │
  │                                            │
  │  5. VERIFICATION                           │
  │     Confirm all threats removed            │
  │     Monitor for re-compromise              │
  └────────────────────────────────────────────┘

PLANNING INPUTS:
  → Complete scope documentation
  → All IOCs identified
  → All persistence mechanisms listed
  → Attacker TTP analysis
  → Timeline of compromise
  → Business requirements and constraints
  → Available resources and tools
```

---

## 2. Eradication Actions

```
ERADICATION BY CATEGORY:

MALWARE REMOVAL:
  → Delete/quarantine malicious files
  → Remove malicious registry keys
  → Remove malicious scheduled tasks
  → Remove malicious services
  → Remove web shells
  → Clean infected documents
  → Remove dropped tools (Mimikatz, PsExec, etc.)

  For each malware artifact:
  System      | Path/Location              | Action
  JSMITH-PC   | C:\Users\jsmith\AppData\   | Quarantine
              | Local\Temp\malware.exe      |
  FILESVR01   | C:\inetpub\wwwroot\        | Delete
              | shell.aspx                  |
  DC01        | HKLM\SOFTWARE\Microsoft\   | Remove key
              | Windows\CurrentVersion\Run  |

PERSISTENCE REMOVAL:
  → Scheduled tasks
  → Services
  → Registry run keys
  → Startup folder items
  → WMI event subscriptions
  → Group Policy modifications
  → DLL hijacking
  → Bootkit/rootkit components
  → Cron jobs (Linux)
  → SSH authorized_keys (Linux)
  → Systemd services (Linux)

  # Remove malicious scheduled task
  Unregister-ScheduledTask -TaskName "WindowsUpdate" `
    -Confirm:$false
  
  # Remove malicious service
  sc.exe delete "MaliciousService"
  
  # Remove run key
  Remove-ItemProperty -Path `
    "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Run" `
    -Name "WindowsDefenderUpdate"

ACCOUNT REMEDIATION:
  → Reset ALL compromised credentials
  → Reset Kerberos TGT (krbtgt) twice
  → Remove unauthorized accounts
  → Remove unauthorized group memberships
  → Revoke all OAuth tokens
  → Reset service account passwords
  → Rotate API keys and certificates

  # Reset krbtgt (do twice, 12+ hours apart)
  Reset-KrbtgtPassword -Server dc01.company.com
  
  # Find and remove rogue admin accounts
  Get-ADGroupMember "Domain Admins" | 
    Where-Object {$_.Name -notin $authorizedAdmins}
```

---

## 3. System Remediation Approaches

```
REMEDIATION OPTIONS:

APPROACH 1: CLEAN AND PATCH (Lower confidence)
  → Remove malware and persistence
  → Patch vulnerabilities
  → Harden configuration
  → Monitor closely

  Pros: Faster, less disruptive
  Cons: Risk of missing hidden persistence
  Use when: Low-severity, limited compromise

APPROACH 2: REIMAGE FROM GOLD IMAGE (Recommended)
  → Rebuild system from known-good image
  → Apply current patches
  → Restore data from clean backup
  → Reconfigure from documented settings

  Pros: High confidence in clean system
  Cons: Time consuming, data restoration needed
  Use when: Confirmed malware, any admin compromise

APPROACH 3: REPLACE HARDWARE (Highest confidence)
  → Deploy new hardware
  → Build from scratch
  → Migrate data after scanning

  Pros: Eliminates firmware-level threats
  Cons: Most expensive and time consuming
  Use when: Suspected firmware compromise, APT

DECISION MATRIX:
  Scenario             | Recommended Approach
  User workstation     | Reimage
  Server (non-domain)  | Reimage
  Domain controller    | Reimage (carefully)
  Web server           | Rebuild from code repo
  Database server      | Reimage + restore data
  Network device       | Factory reset + reconfig
  Cloud instance       | Destroy + redeploy (IaC)
  Firmware compromise  | Replace hardware
```

---

## 4. Coordinated Eradication

```
COORDINATED EXECUTION:

WHY COORDINATE:
  → Attacker may have multiple persistence
    mechanisms across systems
  → Partial eradication may alert attacker
  → Attacker could re-establish access
    from remaining footholds
  → All remediation must happen together

ERADICATION PLAN DOCUMENT:

  Incident: INC-2024-001
  Eradication Date: 2024-01-25
  Window: 02:00-06:00 UTC (maintenance window)
  
  TEAM ASSIGNMENTS:
  Team A (Endpoints):
  → Person 1: Reimage JSMITH-PC
  → Person 2: Reimage FILESVR01
  
  Team B (Identity):
  → Person 3: Reset all compromised passwords
  → Person 4: krbtgt reset, revoke tokens
  
  Team C (Network):
  → Person 5: Permanent firewall rules
  → Person 6: Update DNS, proxy policies
  
  Team D (Monitoring):
  → Person 7: Enhanced monitoring active
  → Person 8: Watch for re-compromise IOCs
  
  EXECUTION SEQUENCE:
  02:00 - Team D: Enhanced monitoring active
  02:05 - Team B: Password resets begin
  02:10 - Team C: Network rules implemented
  02:15 - Team A: Reimaging begins
  02:30 - Team B: krbtgt reset #1
  03:00 - Status check: All teams report
  04:00 - Team A: Reimaging complete
  04:30 - Systems brought back online
  05:00 - Verification testing
  06:00 - Eradication complete

ROLLBACK PLAN:
  → If eradication fails, revert to containment
  → Maintain containment rules until verified
  → Have backup plans for each action
  → Document rollback steps for each change
```

---

## 5. Verification After Eradication

```
POST-ERADICATION VERIFICATION:

  [ ] All malware artifacts removed
  [ ] All persistence mechanisms eliminated
  [ ] All compromised accounts remediated
  [ ] Vulnerability that allowed access patched
  [ ] IOC scan clean across environment
  [ ] No C2 traffic detected
  [ ] No suspicious processes running
  [ ] No unauthorized accounts present
  [ ] Clean antivirus/EDR scans
  [ ] Network traffic patterns normal
  [ ] Enhanced monitoring shows no anomalies

VERIFICATION TOOLS:
  → EDR full scan on all affected systems
  → IOC sweep across entire environment
  → Network monitoring for C2 patterns
  → AD audit for unauthorized changes
  → Vulnerability scan for unpatched systems
  → Configuration audit against baseline
  → File integrity monitoring comparison

MONITORING PERIOD:
  → Intensive monitoring: First 72 hours
  → Enhanced monitoring: 30 days
  → Standard monitoring: Ongoing
  → Re-check IOCs: Weekly for 90 days
  → Alert: Any related activity
```

---

## Summary Table

| Phase | Actions | Duration | Confidence |
|-------|---------|----------|-----------|
| Planning | Scope review, action list, team assign | 1-3 days | N/A |
| Execution | Coordinated removal/rebuild | 4-12 hours | Depends on approach |
| Verification | IOC scan, monitoring, audit | 72 hrs intensive | High if clean |
| Monitoring | Enhanced detection | 30-90 days | Ongoing validation |

---

## Revision Questions

1. Why must eradication be coordinated across all compromised systems?
2. When should you reimage vs. clean and patch a compromised system?
3. What account remediation steps are critical during eradication?
4. What should an eradication plan document include?
5. How do you verify that eradication was successful?

---

*Previous: [02-short-term-vs-long-term-containment.md](02-short-term-vs-long-term-containment.md) | Next: [04-system-recovery.md](04-system-recovery.md)*

---

*[Back to README](../README.md)*
