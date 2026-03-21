# Retesting Procedures

## Unit 10 - Topic 1 | Remediation Verification

---

## Overview

**Retesting** is the process of verifying that vulnerabilities identified during the penetration test have been properly fixed. After the client implements remediation, the pen tester returns to confirm that each finding is resolved and no new vulnerabilities were introduced.

---

## 1. Retesting Process

```
┌──────────────────────────────────────────────────────────────┐
│              RETESTING WORKFLOW                               │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  STEP 1: RECEIVE REMEDIATION NOTIFICATION                    │
│  └── Client confirms fixes have been applied                 │
│                                                              │
│  STEP 2: REVIEW REMEDIATION ACTIONS                          │
│  └── Client describes what was changed                       │
│                                                              │
│  STEP 3: PLAN RETEST                                         │
│  └── Schedule window, confirm scope                          │
│                                                              │
│  STEP 4: EXECUTE RETEST                                      │
│  └── Attempt to reproduce each original finding             │
│                                                              │
│  STEP 5: VERIFY NEW CONTROLS                                 │
│  └── Test if fixes can be bypassed                           │
│                                                              │
│  STEP 6: CHECK FOR REGRESSIONS                               │
│  └── Verify fixes didn't introduce new issues               │
│                                                              │
│  STEP 7: DOCUMENT & REPORT                                   │
│  └── Update findings with retest results                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Retest Planning

```
RETEST PLANNING DOCUMENT:

PROJECT:    Acme Corporation — Retest
ORIGINAL:   Pen test conducted Jan 15-19, 2024
RETEST:     Scheduled Feb 12, 2024
TESTER:     [Same tester as original — preferred]
SCOPE:      Same targets as original engagement

FINDINGS TO RETEST:
┌──────┬───────────────────────────────┬──────────┬───────────┐
│ ID   │ Finding                       │ Severity │ Fix Status│
├──────┼───────────────────────────────┼──────────┼───────────┤
│F-001 │ SQL Injection — Login         │ Critical │ Fixed     │
│F-002 │ EternalBlue — File Server     │ Critical │ Fixed     │
│F-003 │ Default Tomcat Credentials    │ High     │ Fixed     │
│F-004 │ Exposed Backup Directory      │ High     │ Fixed     │
│F-005 │ Weak SSH Configuration        │ Medium   │ Pending   │
│F-006 │ Missing Security Headers      │ Medium   │ Fixed     │
└──────┴───────────────────────────────┴──────────┴───────────┘

ESTIMATED TIME: 4-8 hours (depending on finding count)
AUTHORIZATION: Original SOW covers 1 retest
```

---

## 3. Executing Retests

```bash
# RETEST EACH FINDING USING ORIGINAL STEPS:

# F-001: SQL Injection Retest
# Original: ' OR 1=1-- in username field
# Retest:
sqlmap -u "http://10.0.0.5/login?user=test" --batch
# Expected result: "all tested parameters do not appear to be injectable"
# STATUS: ✅ REMEDIATED

# F-002: EternalBlue Retest
nmap --script smb-vuln-ms17-010 -p 445 10.0.0.10
# Expected: "Host is NOT vulnerable"
# STATUS: ✅ REMEDIATED

# F-003: Default Tomcat Credentials
curl -u tomcat:tomcat http://10.0.0.5:8080/manager/html
# Expected: 401 Unauthorized
# STATUS: ✅ REMEDIATED

# F-005: Weak SSH Configuration
ssh-audit 10.0.0.5
# Expected: No deprecated algorithms listed
# STATUS: ❌ NOT REMEDIATED (still shows hmac-sha1)

# ALSO CHECK: Can the fix be bypassed?
# Try alternative injection techniques
# Try different default credential combinations
# Test edge cases and encoding bypasses
```

---

## 4. Retest Report Format

```
═══════════════════════════════════════════════
RETEST REPORT
═══════════════════════════════════════════════
Date: February 12, 2024
Original Test: January 15-19, 2024

RETEST SUMMARY:
Total Findings Retested: 6
Remediated: 5 (83%)
Not Remediated: 1 (17%)
New Findings: 0

DETAILED RESULTS:
┌──────┬──────────────────────┬──────────┬────────────┐
│ ID   │ Finding              │ Severity │ Status     │
├──────┼──────────────────────┼──────────┼────────────┤
│F-001 │ SQL Injection        │ Critical │ ✅ Fixed    │
│F-002 │ EternalBlue          │ Critical │ ✅ Fixed    │
│F-003 │ Default Credentials  │ High     │ ✅ Fixed    │
│F-004 │ Exposed Directory    │ High     │ ✅ Fixed    │
│F-005 │ Weak SSH Config      │ Medium   │ ❌ Open     │
│F-006 │ Missing Headers      │ Medium   │ ✅ Fixed    │
└──────┴──────────────────────┴──────────┴────────────┘

REMAINING RISK:
F-005 remains open. Deprecated SSH algorithms are still
accepted. Remediation guidance was re-provided to the
client. A follow-up retest is recommended.
═══════════════════════════════════════════════
```

---

## 5. Retest Considerations

| Consideration | Detail |
|---------------|--------|
| **Same tester** | Preferred for consistency and context |
| **Same tools** | Use identical methods to validate |
| **Bypass attempts** | Don't just confirm fix — try to bypass it |
| **Regression testing** | Check if fix broke other functionality |
| **New findings** | Note if remediation introduced new vulns |
| **Timeline** | Usually 2-4 weeks after remediation |
| **Authorization** | Verify retest is covered in original SOW |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Retesting** | Verify fixes actually work |
| **Same methods** | Use original exploit steps |
| **Bypass testing** | Try to circumvent the fix |
| **Regression** | Check for newly introduced issues |
| **Report update** | Document retest results per finding |
| **Follow-up** | Schedule additional retests if needed |

---

## Quick Revision Questions

1. **What is the purpose of retesting?**
2. **Who should conduct the retest?**
3. **Why try to bypass the fix, not just confirm it works?**
4. **What is regression testing in this context?**
5. **What should a retest report include?**

---

[Next: Validation Techniques →](02-validation-techniques.md)

---

*Unit 10 - Topic 1 of 4 | [Back to README](../README.md)*
