# Validation Techniques

## Unit 10 - Topic 2 | Remediation Verification

---

## Overview

**Validation techniques** go beyond simple retesting to comprehensively verify that a vulnerability has been properly remediated. This includes testing the fix itself, checking for bypass methods, ensuring no new vulnerabilities were introduced, and validating that the security control is robust.

---

## 1. Validation vs Simple Retesting

```
┌──────────────────────────────────────────────────────────────┐
│              RETESTING vs VALIDATION                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SIMPLE RETEST:                                              │
│  "Does the original exploit still work?"                     │
│  └── Run same exploit → check if it fails                    │
│  └── Quick but incomplete                                    │
│                                                              │
│  FULL VALIDATION:                                            │
│  "Is the vulnerability truly fixed?"                         │
│  ├── Run original exploit (should fail)                      │
│  ├── Try bypass techniques (should also fail)               │
│  ├── Test with different payloads                            │
│  ├── Check related attack vectors                            │
│  ├── Verify no regressions introduced                        │
│  └── Confirm control is sustainable                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. SQL Injection Validation Example

```bash
# ORIGINAL FINDING: SQL Injection on login page
# CLIENT FIX: "Added input validation"

# STEP 1: Try original payload
' OR 1=1--
# Result: Blocked ✅

# STEP 2: Try bypass techniques
' OR '1'='1'--               # Different syntax
' or 1=1#                    # Different comment style
admin'--                     # Truncation
' UNION SELECT 1,2,3--      # UNION injection
' AND 1=1--                  # Boolean-based
'||'1'='1                    # String concatenation
%27%20OR%201=1--             # URL encoding
' /*!OR*/ 1=1--              # MySQL inline comment

# STEP 3: Test with sqlmap
sqlmap -u "http://target.com/login?user=admin" \
  --tamper=space2comment,randomcase,between \
  --level=5 --risk=3 --batch

# STEP 4: Verify fix method
# ASK CLIENT: Did you use parameterized queries?
# If "input validation only" → still potentially vulnerable
# If "prepared statements" → much more robust

# VALIDATION RESULT:
# If ALL bypass attempts fail → ✅ Properly Remediated
# If any bypass works → ❌ Inadequate Fix
```

---

## 3. Common Fix Validation Checklist

| Finding Type | Validation Steps |
|-------------|-----------------|
| **SQL Injection** | Try 10+ bypass payloads, sqlmap with tamper scripts |
| **XSS** | Try encoding bypasses, different contexts (attribute, JS, URL) |
| **Auth bypass** | Try other bypass methods, check session management |
| **Default creds** | Verify password changed, check other default accounts |
| **Missing patch** | Verify version number, run vuln scan against specific CVE |
| **Misconfig** | Review configuration file, verify change survives restart |
| **Weak crypto** | Scan with sslyze/testssl, check all endpoints |

---

## 4. Automated Validation

```bash
# === VULNERABILITY SCANNER REVALIDATION ===
# Run same scanner with same profile
nessus_scan --policy "Original_Policy" --targets targets.txt

# === NMAP SCRIPT VALIDATION ===
# Specific vulnerability check
nmap --script smb-vuln-ms17-010 -p 445 10.0.0.10
nmap --script http-sql-injection -p 80 10.0.0.5
nmap --script ssl-enum-ciphers -p 443 10.0.0.5

# === SSL/TLS VALIDATION ===
sslyze --regular target.com
testssl.sh target.com

# === CONFIGURATION VALIDATION ===
# SSH hardening check
ssh-audit 10.0.0.5

# === COMPLIANCE SCANNING ===
# Run CIS benchmark scan after hardening
# OpenSCAP, Lynis
lynis audit system --tests-from-group ssh
```

---

## 5. Validation Documentation

```
VALIDATION RECORD:

FINDING: F-001 — SQL Injection on Login Page
ORIGINAL CVSS: 9.8 (Critical)

CLIENT REMEDIATION:
"Replaced all dynamic queries with PDO prepared
statements. Added input validation layer. Deployed
ModSecurity WAF rules."

VALIDATION TESTS PERFORMED:
┌─────┬────────────────────────────────────┬──────────┐
│  #  │ Test                               │ Result   │
├─────┼────────────────────────────────────┼──────────┤
│  1  │ Original payload: ' OR 1=1--       │ Blocked  │
│  2  │ URL-encoded payload               │ Blocked  │
│  3  │ Double-encoding bypass            │ Blocked  │
│  4  │ UNION-based injection             │ Blocked  │
│  5  │ Boolean blind injection           │ Blocked  │
│  6  │ Time-based blind injection        │ Blocked  │
│  7  │ sqlmap with tamper scripts        │ Blocked  │
│  8  │ WAF bypass techniques             │ Blocked  │
│  9  │ Different input fields            │ Blocked  │
│ 10  │ API endpoint testing              │ Blocked  │
└─────┴────────────────────────────────────┴──────────┘

CONCLUSION: ✅ PROPERLY REMEDIATED
The fix uses parameterized queries (root cause fix)
with WAF as defense-in-depth. All bypass attempts failed.
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Validation** | More thorough than simple retesting |
| **Bypass testing** | Try multiple techniques to circumvent fix |
| **Root cause** | Verify fix addresses the root cause |
| **Automation** | Use scanners for consistent validation |
| **Documentation** | Record every test and its result |
| **Regression** | Ensure fix doesn't break functionality |

---

## Quick Revision Questions

1. **How does validation differ from simple retesting?**
2. **Why test bypass techniques after a fix is applied?**
3. **What makes a SQL injection fix "robust"?**
4. **Name 3 automated validation tools.**
5. **Why verify the fix method used, not just the result?**

---

[← Previous: Retesting Procedures](01-retesting-procedures.md) | [Next: Closure Process →](03-closure-process.md)

---

*Unit 10 - Topic 2 of 4 | [Back to README](../README.md)*
