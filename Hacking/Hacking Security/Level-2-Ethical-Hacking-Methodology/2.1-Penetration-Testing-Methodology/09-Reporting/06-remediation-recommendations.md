# Remediation Recommendations

## Unit 9 - Topic 6 | Reporting

---

## Overview

**Remediation recommendations** provide actionable guidance for fixing each vulnerability found during the penetration test. Good recommendations are **specific**, **prioritized**, and **practical** — giving the technical team clear steps to improve security.

---

## 1. Effective Recommendations

```
┌──────────────────────────────────────────────────────────────┐
│              GOOD vs BAD RECOMMENDATIONS                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  ❌ BAD: "Fix the SQL injection."                            │
│  ❌ BAD: "Improve security."                                 │
│  ❌ BAD: "Apply patches."                                    │
│                                                              │
│  ✅ GOOD: "Replace dynamic SQL queries in login.php          │
│     (line 42) with parameterized queries using PDO           │
│     prepared statements. Example:                            │
│     $stmt = $pdo->prepare('SELECT * FROM users               │
│     WHERE username = ? AND password = ?');                    │
│     $stmt->execute([$username, $password]);"                 │
│                                                              │
│  ✅ GOOD: "Disable root SSH login by setting                 │
│     PermitRootLogin no in /etc/ssh/sshd_config              │
│     and restart the SSH service."                            │
│                                                              │
│  ✅ GOOD: "Apply Microsoft security patch KB5005565           │
│     to resolve MS17-010 (EternalBlue) on file-server-01     │
│     (10.0.0.10). Verify by running:                          │
│     nmap --script smb-vuln-ms17-010 -p 445 10.0.0.10"       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Recommendation Categories

| Category | Example |
|----------|---------|
| **Code fix** | Use parameterized queries, input validation |
| **Configuration** | Disable root login, remove default creds |
| **Patching** | Apply specific security updates |
| **Architecture** | Network segmentation, WAF deployment |
| **Process** | Password policy, access reviews |
| **Monitoring** | Enable logging, SIEM alerts |

---

## 3. Prioritized Remediation Plan

```
REMEDIATION ROADMAP:

══════════════════════════════════════════
IMMEDIATE (0-48 Hours) — Critical Findings
══════════════════════════════════════════
F-001: SQL Injection on Login Page
  → Implement parameterized queries
  → Deploy WAF rule as interim measure
  → Estimated effort: 4 hours

F-002: EternalBlue on File Server
  → Apply MS17-010 patch (KB5005565)
  → Disable SMBv1: Set-SmbServerConfiguration -EnableSMB1Protocol $false
  → Estimated effort: 2 hours + reboot window

══════════════════════════════════════════
SHORT-TERM (1 Week) — High Findings
══════════════════════════════════════════
F-003: Default Tomcat Manager Credentials
  → Change default password
  → Restrict /manager to internal IPs only
  → Estimated effort: 1 hour

F-004: Exposed Backup Directory
  → Remove /backup from web root
  → Add directory listing restriction
  → Estimated effort: 30 minutes

══════════════════════════════════════════
MEDIUM-TERM (30 Days) — Medium Findings
══════════════════════════════════════════
F-005: Weak SSH Configuration
  → Update sshd_config with hardened settings
  → Estimated effort: 1 hour

F-006: Missing Security Headers
  → Add HSTS, X-Frame-Options, CSP headers
  → Estimated effort: 2 hours

══════════════════════════════════════════
LONG-TERM (90 Days) — Strategic Improvements
══════════════════════════════════════════
• Implement network segmentation
• Deploy centralized log management (SIEM)
• Establish vulnerability management program
• Conduct security code review of all applications
```

---

## 4. Recommendation Template

```
═══════════════════════════════════════════════
REMEDIATION: F-001 — SQL Injection
═══════════════════════════════════════════════

IMMEDIATE ACTION (Interim):
Deploy WAF rule to block SQL injection patterns:
  SecRule ARGS "@detectSQLi" "id:1,deny,status:403"

PERMANENT FIX:
1. Replace all dynamic SQL with prepared statements:
   BEFORE:
   $query = "SELECT * FROM users WHERE id = " . $_GET['id'];

   AFTER:
   $stmt = $pdo->prepare("SELECT * FROM users WHERE id = ?");
   $stmt->execute([$_GET['id']]);

2. Implement input validation:
   - Whitelist allowed characters
   - Validate data types (integer, string, email)
   - Limit input length

3. Apply least-privilege database access:
   - Application user should NOT be db admin
   - Remove FILE, GRANT, CREATE privileges

VERIFICATION:
After remediation, verify fix with:
  sqlmap -u "http://target.com/page?id=1" --batch
  Expected: No injection found

EFFORT ESTIMATE: 4-8 hours (developer)
═══════════════════════════════════════════════
```

---

## 5. Strategic Recommendations

```
ORGANIZATION-WIDE IMPROVEMENTS:

1. VULNERABILITY MANAGEMENT PROGRAM
   → Regular scanning, patch management, tracking

2. SECURITY AWARENESS TRAINING
   → Annual training, phishing simulations

3. INCIDENT RESPONSE PLAN
   → Documented procedures, tabletop exercises

4. NETWORK SEGMENTATION
   → Separate DMZ, internal, management networks

5. ACCESS CONTROL REVIEW
   → Least privilege, regular access reviews

6. APPLICATION SECURITY
   → SAST/DAST in CI/CD pipeline, code reviews

7. MONITORING AND DETECTION
   → SIEM deployment, 24/7 SOC, alerting
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Specific** | Exact steps, config lines, code examples |
| **Prioritized** | Critical first, then high, medium, low |
| **Practical** | Include effort estimates |
| **Interim + permanent** | Quick fix now, proper fix later |
| **Verifiable** | Include validation steps |
| **Strategic** | Long-term program improvements |

---

## Quick Revision Questions

1. **What makes a remediation recommendation effective?**
2. **Why provide both interim and permanent fixes?**
3. **How should remediation be prioritized?**
4. **Why include verification steps?**
5. **What are strategic (organization-wide) recommendations?**

---

[← Previous: Risk Ratings](05-risk-ratings.md) | [Next: Evidence Presentation →](07-evidence-presentation.md)

---

*Unit 9 - Topic 6 of 7 | [Back to README](../README.md)*
