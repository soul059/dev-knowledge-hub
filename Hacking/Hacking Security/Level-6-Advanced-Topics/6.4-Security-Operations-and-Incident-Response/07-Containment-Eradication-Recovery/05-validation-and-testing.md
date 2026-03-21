# Unit 7: Containment, Eradication, Recovery — Topic 5: Validation and Testing

## Overview

**Validation and testing** confirms that recovered systems are clean, functional, and secure before returning them to production. This phase catches incomplete eradication, configuration errors, and residual vulnerabilities, preventing re-compromise and ensuring business operations can resume safely.

---

## 1. Validation Framework

```
VALIDATION PROCESS:

  ┌────────────────────────────────────────────┐
  │      POST-RECOVERY VALIDATION              │
  │                                            │
  │  LAYER 1: SECURITY VALIDATION              │
  │  → No malware/IOCs present                 │
  │  → No persistence mechanisms               │
  │  → No unauthorized accounts                │
  │  → Patches applied                         │
  │  → Configuration hardened                  │
  │                                            │
  │  LAYER 2: FUNCTIONAL VALIDATION            │
  │  → Operating system functional             │
  │  → Applications working                   │
  │  → Data accessible and complete            │
  │  → Network connectivity correct            │
  │  → Authentication working                  │
  │                                            │
  │  LAYER 3: MONITORING VALIDATION            │
  │  → EDR agent reporting                     │
  │  → Logs flowing to SIEM                    │
  │  → Alerts generating correctly             │
  │  → Baseline established                    │
  └────────────────────────────────────────────┘
```

---

## 2. Security Validation

```
SECURITY TESTING CHECKLIST:

MALWARE SCANNING:
  [ ] Full AV/EDR scan — no detections
  [ ] Custom IOC scan — no matches
  [ ] YARA rule scan — no matches
  [ ] Rootkit scan — clean
  [ ] Memory analysis — no suspicious processes

  # Run YARA scan against recovered system
  yara -r incident_rules.yar C:\
  
  # PowerShell: Check for known IOCs
  Get-ChildItem -Path C:\ -Recurse -Include *.exe |
    Get-FileHash | Where-Object {
      $_.Hash -in $knownBadHashes
    }

PERSISTENCE CHECKS:
  [ ] Scheduled tasks — only known/approved
  [ ] Services — only known/approved
  [ ] Registry run keys — clean
  [ ] Startup folder — empty or approved only
  [ ] WMI subscriptions — none unexpected
  [ ] Browser extensions — approved only

  # Autoruns (Sysinternals) — comprehensive check
  autorunsc.exe -a * -c -h -s -v -vt
  
  # Check scheduled tasks
  Get-ScheduledTask | Where-Object {
    $_.State -eq 'Ready' -and 
    $_.TaskPath -notlike '\Microsoft\*'
  }

ACCOUNT VALIDATION:
  [ ] Local admin accounts — only approved
  [ ] Domain memberships — verified
  [ ] Service accounts — validated
  [ ] No unauthorized accounts exist
  [ ] Passwords meet policy

  # Check local administrators
  Get-LocalGroupMember -Group "Administrators"

VULNERABILITY SCAN:
  [ ] Nessus/Qualys scan — no critical/high
  [ ] All patches verified installed
  [ ] No unnecessary open ports
  [ ] No default credentials
  [ ] Configuration compliance check

NETWORK VALIDATION:
  [ ] Correct VLAN assignment
  [ ] Firewall rules applied
  [ ] No unauthorized outbound connections
  [ ] DNS resolving correctly
  [ ] Only required ports open
  
  # Check listening ports
  netstat -ano | findstr LISTENING
  
  # Check outbound connections
  netstat -ano | findstr ESTABLISHED
```

---

## 3. Functional Testing

```
FUNCTIONAL VALIDATION:

SYSTEM FUNCTIONALITY:
  [ ] OS boots normally
  [ ] System performance acceptable
  [ ] All required services running
  [ ] Disk space adequate
  [ ] System time synchronized (NTP)

APPLICATION TESTING:
  [ ] Application starts without errors
  [ ] Core features functional
  [ ] Data accessible and accurate
  [ ] Integrations working
  [ ] Authentication/authorization correct
  [ ] Performance baseline met

DATABASE VALIDATION:
  [ ] Database service running
  [ ] Data integrity verified
  [ ] Backup/restore tested
  [ ] Replication working (if applicable)
  [ ] Access controls correct
  [ ] Connection strings updated

USER ACCEPTANCE:
  [ ] Users can log in
  [ ] Users can access required resources
  [ ] Users can perform key tasks
  [ ] No unexpected errors
  [ ] Performance acceptable
```

---

## 4. Monitoring Validation

```
MONITORING VERIFICATION:

EDR VALIDATION:
  [ ] Agent installed and running
  [ ] Agent communicating with console
  [ ] Policy applied correctly
  [ ] Real-time protection enabled
  [ ] Last scan completed successfully

SIEM INTEGRATION:
  [ ] Logs flowing to SIEM
  [ ] Log sources verified:
      → Windows Event Logs
      → Sysmon events
      → Application logs
      → Security logs
  [ ] Parsing working correctly
  [ ] Alert rules applying
  [ ] Dashboard showing system

LOG VALIDATION:
  # Verify Sysmon is running
  Get-Service Sysmon64
  
  # Verify Windows Event Logging
  wevtutil el | Select-String "Security"
  
  # Check last log entries
  Get-WinEvent -LogName Security -MaxEvents 5

  # Verify log forwarding
  Get-WinEvent -LogName "Microsoft-Windows-EventCollector/Operational" `
    -MaxEvents 5

BASELINE ESTABLISHMENT:
  → Record normal process list
  → Record normal service list
  → Record normal network connections
  → Record normal scheduled tasks
  → Record normal user accounts
  → Use as reference for future detection
```

---

## 5. Validation Sign-Off

```
SIGN-OFF PROCESS:

VALIDATION REPORT:
  ┌────────────────────────────────────────────┐
  │ SYSTEM VALIDATION REPORT                   │
  │                                            │
  │ System: FILESVR01                          │
  │ Date: 2024-01-26                          │
  │ Validated by: Jane Doe                     │
  │                                            │
  │ SECURITY VALIDATION:        ✓ PASS         │
  │ → AV/EDR scan: Clean                      │
  │ → IOC scan: No matches                    │
  │ → Persistence check: Clean                │
  │ → Vulnerability scan: No critical/high     │
  │ → Account audit: Compliant                │
  │                                            │
  │ FUNCTIONAL VALIDATION:      ✓ PASS         │
  │ → Services running: All OK                │
  │ → Data restored: 100% verified            │
  │ → File shares accessible: Yes             │
  │ → Performance: Within baseline            │
  │                                            │
  │ MONITORING VALIDATION:      ✓ PASS         │
  │ → EDR active: Yes                         │
  │ → SIEM logging: Verified                  │
  │ → Alerts configured: Yes                  │
  │                                            │
  │ RECOMMENDATION: Approved for production    │
  │                                            │
  │ Sign-off: _________________ (IR Lead)     │
  │ Sign-off: _________________ (IT Manager)  │
  └────────────────────────────────────────────┘

APPROVAL REQUIRED FROM:
  → IR team lead (security clearance)
  → System owner (functional approval)
  → IT operations (operational readiness)
  → Management (if critical system)
```

---

## Summary Table

| Validation Layer | Tests | Pass Criteria |
|-----------------|-------|--------------|
| Security | AV/IOC scan, persistence, accounts | No findings |
| Functional | Apps, data, connectivity | All working |
| Monitoring | EDR, SIEM, logging | All reporting |
| Sign-off | Multi-party approval | All approvals |

---

## Revision Questions

1. What are the three layers of post-recovery validation?
2. What security checks should be performed on a recovered system?
3. How do you validate that monitoring is functioning correctly?
4. What does a validation sign-off process look like?
5. Why is baseline establishment important after recovery?

---

*Previous: [04-system-recovery.md](04-system-recovery.md) | Next: [06-return-to-operations.md](06-return-to-operations.md)*

---

*[Back to README](../README.md)*
