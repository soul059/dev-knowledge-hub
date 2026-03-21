# Unit 7: Containment, Eradication, Recovery — Topic 4: System Recovery

## Overview

**System recovery** restores affected systems and services to normal operation after eradication. Recovery must be carefully planned and executed to ensure systems are securely rebuilt, data is restored from clean sources, and business operations resume safely.

---

## 1. Recovery Planning

```
RECOVERY PROCESS:

  ┌────────────────────────────────────────────┐
  │           RECOVERY WORKFLOW                │
  │                                            │
  │  1. PRIORITIZE SYSTEMS                     │
  │     Business criticality ranking           │
  │     Dependencies mapped                    │
  │                                            │
  │  2. PREPARE RECOVERY                       │
  │     Clean media/images ready               │
  │     Backup verification                    │
  │     Network preparation                    │
  │                                            │
  │  3. REBUILD SYSTEMS                        │
  │     Reimage from gold image                │
  │     Apply patches                          │
  │     Harden configuration                   │
  │                                            │
  │  4. RESTORE DATA                           │
  │     Verify backup integrity                │
  │     Restore from clean backup              │
  │     Validate data completeness             │
  │                                            │
  │  5. TEST AND VALIDATE                      │
  │     Functionality testing                  │
  │     Security verification                  │
  │     Performance testing                    │
  │                                            │
  │  6. RETURN TO PRODUCTION                   │
  │     Staged reconnection                    │
  │     Enhanced monitoring                    │
  │     User notification                      │
  └────────────────────────────────────────────┘

RECOVERY PRIORITIZATION:
  Priority | Systems              | Target
  1        | Domain controllers   | First (AD needed)
  2        | DNS/DHCP             | Network services
  3        | Critical apps        | Business operations
  4        | File servers         | Data access
  5        | Email                | Communication
  6        | Web servers          | External services
  7        | User workstations    | End user productivity
  8        | Non-critical systems | When convenient
```

---

## 2. System Rebuild

```
SYSTEM REBUILD PROCESS:

WORKSTATION REBUILD:
  1. Boot from clean media (USB/PXE)
  2. Wipe disk (full format)
  3. Install OS from gold image
  4. Apply all current patches
  5. Install required software
  6. Apply security configuration baseline
  7. Install EDR agent
  8. Join domain (if clean)
  9. Restore user data from backup
  10. Verify with security scan

SERVER REBUILD:
  1. Build from documented configuration
  2. Install OS from gold image
  3. Apply all patches
  4. Harden per CIS benchmark
  5. Install monitoring agents
  6. Configure application
  7. Restore data from verified backup
  8. Test functionality
  9. Security scan
  10. Stage for production

DOMAIN CONTROLLER RECOVERY:
  ⚠ MOST CRITICAL — CAREFUL PLANNING REQUIRED
  
  Scenario 1: Some DCs clean
  → Rebuild compromised DCs from scratch
  → Replicate from clean DCs
  → Reset krbtgt twice (12+ hours apart)
  → Audit all AD objects
  
  Scenario 2: All DCs compromised
  → Rebuild AD from authoritative backup
  → OR: Build new forest and migrate
  → Reset ALL passwords
  → Audit all trusts
  → This is a major undertaking

CLOUD RECOVERY:
  → Destroy compromised instances
  → Redeploy from IaC (Terraform/CloudFormation)
  → Rotate all keys and credentials
  → Review and fix IAM policies
  → Verify cloud trail logging
  → Redeploy from clean CI/CD pipeline
```

---

## 3. Data Restoration

```
DATA RESTORATION:

BACKUP VERIFICATION:
  → Identify last known clean backup
  → Pre-compromise date required
  → Scan backup for malware/IOCs
  → Verify backup integrity (checksums)
  → Test restore on isolated system first

  BACKUP TIMELINE:
  Jan 5      Jan 8      Jan 10     Jan 15
  [Backup]   [Backup]   [BREACH]   [Detection]
     ↑          ↑
     Safe       May be     Compromised
                safe

  → Use Jan 5 backup (confirmed clean)
  → Jan 8 needs scanning before use

DATA RESTORATION STEPS:
  1. Identify required data
  2. Select clean backup point
  3. Verify backup integrity
  4. Restore to isolated environment
  5. Scan restored data for malware
  6. Validate data completeness
  7. Copy clean data to rebuilt system
  8. Verify application functionality
  9. Document any data loss

DATA LOSS ASSESSMENT:
  → Calculate data created between last clean
    backup and incident
  → Identify what cannot be recovered
  → Communicate data loss to stakeholders
  → Document for insurance/legal purposes

  Data Window    | Status
  Before Jan 5   | Recoverable from backup
  Jan 5 - Jan 8  | Recoverable (verify clean)
  Jan 8 - Jan 10 | Potentially recoverable
  Jan 10 - Jan 15| May be compromised, scan needed
```

---

## 4. Recovery Hardening

```
SECURITY HARDENING DURING RECOVERY:

HARDENING CHECKLIST:
  [ ] All patches applied (OS + apps)
  [ ] CIS benchmark applied
  [ ] Unnecessary services disabled
  [ ] Default accounts disabled
  [ ] Strong passwords enforced
  [ ] MFA enabled
  [ ] Firewall rules tightened
  [ ] Unnecessary ports closed
  [ ] EDR deployed and active
  [ ] Logging enhanced
  [ ] File integrity monitoring enabled
  [ ] Backup strategy validated

VULNERABILITY REMEDIATION:
  → Patch the vulnerability that allowed initial
    access
  → Patch all critical/high vulnerabilities
  → Remove unnecessary software
  → Update end-of-life systems
  → Apply security configurations

NETWORK HARDENING:
  → Implement network segmentation
  → Apply zero-trust principles
  → Restrict lateral movement paths
  → Enhanced monitoring at boundaries
  → VPN/RDP hardening
  → DNS security (DNSSEC, DNS filtering)

IDENTITY HARDENING:
  → Implement MFA for all admin accounts
  → Implement PAM for privileged access
  → Reduce admin account count
  → Implement tiered admin model
  → Enable conditional access policies
  → Regular access reviews
```

---

## 5. Recovery Communication

```
COMMUNICATION DURING RECOVERY:

INTERNAL COMMUNICATION:
  → IT team: Recovery schedule, assignments
  → Management: Timeline, business impact
  → Users: When systems will be available
  → Help desk: Expected issues and workarounds
  → Legal: Data loss, compliance impact

RECOVERY STATUS TEMPLATE:
  ┌──────────────────────────────────────────┐
  │ RECOVERY STATUS UPDATE                   │
  │ Date: 2024-01-26                        │
  │ Time: 14:00 UTC                         │
  │                                          │
  │ OVERALL STATUS: ON TRACK                 │
  │                                          │
  │ COMPLETED:                               │
  │ ✓ Domain controllers rebuilt             │
  │ ✓ DNS/DHCP restored                     │
  │ ✓ File servers rebuilt                   │
  │                                          │
  │ IN PROGRESS:                             │
  │ → Email server restoration (80%)         │
  │ → User workstation reimaging (60/100)    │
  │                                          │
  │ PENDING:                                 │
  │ ○ Web server rebuild (tomorrow)          │
  │ ○ Remaining workstations (40 left)       │
  │                                          │
  │ ISSUES:                                  │
  │ → 3 days of email data unrecoverable     │
  │ → Printer server delayed (driver issue)  │
  │                                          │
  │ NEXT UPDATE: 2024-01-27 10:00 UTC       │
  └──────────────────────────────────────────┘
```

---

## Summary Table

| Recovery Phase | Activities | Priority | Duration |
|---------------|-----------|----------|----------|
| Planning | Prioritize, prepare resources | Critical | 1-2 days |
| Rebuild | Reimage, patch, harden | Critical | 1-5 days |
| Data Restore | Verify backup, restore clean | High | 1-3 days |
| Testing | Functionality, security scan | High | 1-2 days |
| Return to Ops | Staged reconnection | Medium | 1-2 days |

---

## Revision Questions

1. How should systems be prioritized for recovery?
2. What special considerations apply to domain controller recovery?
3. How do you verify a backup is clean before restoring?
4. What hardening measures should be applied during recovery?
5. What should a recovery status communication include?

---

*Previous: [03-eradication-planning.md](03-eradication-planning.md) | Next: [05-validation-and-testing.md](05-validation-and-testing.md)*

---

*[Back to README](../README.md)*
