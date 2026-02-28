# Chapter 2.4: Code Review for Security

## Overview

Security-focused code review is one of the most effective methods for finding vulnerabilities that automated tools miss. It combines human intelligence with contextual understanding to identify logic flaws, business logic vulnerabilities, and subtle security issues.

---

## Security Code Review Process

```
+====================================================================+
|              SECURITY CODE REVIEW WORKFLOW                          |
+====================================================================+
|                                                                      |
|  1. AUTOMATED SCAN (First Pass)                                     |
|  +--------------------------------------------------------------+  |
|  |  SAST  -->  SCA  -->  Secret Scan  -->  Lint                 |  |
|  |  (Automated results feed into manual review)                 |  |
|  +--------------------------------------------------------------+  |
|                            |                                         |
|                            v                                         |
|  2. MANUAL REVIEW (Second Pass)                                     |
|  +--------------------------------------------------------------+  |
|  |  Business logic  |  Auth flows  |  Data handling  |  Config  |  |
|  +--------------------------------------------------------------+  |
|                            |                                         |
|                            v                                         |
|  3. FEEDBACK & FIX                                                  |
|  +--------------------------------------------------------------+  |
|  |  Comments on PR  -->  Developer fixes  -->  Re-review        |  |
|  +--------------------------------------------------------------+  |
|                            |                                         |
|                            v                                         |
|  4. APPROVE & MERGE                                                 |
|  +--------------------------------------------------------------+  |
|  |  All security issues resolved  -->  Merge to main            |  |
|  +--------------------------------------------------------------+  |
+====================================================================+
```

---

## What to Look for in Security Reviews

### Security Review Checklist

```
+================================================================+
|  SECURITY CODE REVIEW CHECKLIST                                 |
+================================================================+
|                                                                  |
|  AUTHENTICATION & AUTHORIZATION                                  |
|  [ ] Are all endpoints protected with authentication?           |
|  [ ] Is authorization checked at object level (not just role)?  |
|  [ ] Are passwords hashed with bcrypt/Argon2?                   |
|  [ ] Is MFA implemented for sensitive operations?               |
|  [ ] Are sessions properly managed (timeout, rotation)?        |
|                                                                  |
|  INPUT HANDLING                                                  |
|  [ ] Is all user input validated server-side?                   |
|  [ ] Are parameterized queries used for databases?              |
|  [ ] Is output encoded for the correct context?                 |
|  [ ] Are file uploads validated (type, size, content)?          |
|  [ ] Is path traversal prevented?                               |
|                                                                  |
|  DATA PROTECTION                                                 |
|  [ ] Is sensitive data encrypted at rest?                       |
|  [ ] Is TLS enforced for data in transit?                       |
|  [ ] Are secrets stored in vault (not in code)?                 |
|  [ ] Is PII properly handled and masked in logs?               |
|  [ ] Are proper HTTP security headers set?                       |
|                                                                  |
|  ERROR HANDLING                                                  |
|  [ ] Are errors handled gracefully (no stack traces to user)?   |
|  [ ] Is sensitive data excluded from error messages?            |
|  [ ] Are all exceptions caught and logged?                      |
|                                                                  |
|  BUSINESS LOGIC                                                  |
|  [ ] Can the flow be manipulated (skip steps, replay)?          |
|  [ ] Are race conditions possible?                               |
|  [ ] Is rate limiting implemented?                              |
|  [ ] Are financial calculations accurate and tamper-proof?      |
|                                                                  |
|  DEPENDENCIES                                                    |
|  [ ] Are all dependencies necessary?                            |
|  [ ] Are dependency versions pinned?                            |
|  [ ] Are there known vulnerabilities in dependencies?           |
+================================================================+
```

---

## Effective PR Review Comments

### Good vs. Bad Security Review Comments

```
BAD COMMENT:
+------------------------------------------------------------+
| "This is insecure. Fix it."                                 |
|                                                              |
| Problems:                                                    |
| - No explanation of the vulnerability                       |
| - No suggested fix                                          |
| - Doesn't help developer learn                              |
+------------------------------------------------------------+

GOOD COMMENT:
+------------------------------------------------------------+
| "SECURITY: SQL Injection vulnerability (CWE-89)            |
|                                                              |
| This query uses string concatenation with user input,       |
| which allows SQL injection attacks.                         |
|                                                              |
| Current code:                                                |
|   query = f"SELECT * FROM users WHERE id = {user_id}"      |
|                                                              |
| Suggested fix:                                               |
|   query = "SELECT * FROM users WHERE id = %s"               |
|   cursor.execute(query, (user_id,))                         |
|                                                              |
| Reference: https://cheatsheetseries.owasp.org/              |
| cheatsheets/Query_Parameterization_Cheat_Sheet.html"        |
+------------------------------------------------------------+
```

### Comment Severity Labels

```
Use labels to prioritize findings:

  [CRITICAL-SEC]  Must fix before merge. Exploitable vulnerability.
  [HIGH-SEC]      Must fix before merge. Significant security risk.
  [MEDIUM-SEC]    Should fix. Creates defense weakness.
  [LOW-SEC]       Nice to fix. Minor security improvement.
  [SEC-QUESTION]  Need clarification on security implications.
  [SEC-STYLE]     Coding style that could lead to future security issues.
```

---

## Common Patterns to Catch

### Pattern 1: Hardcoded Secrets

```python
# RED FLAG: Hardcoded credentials
API_KEY = "sk_live_abc123def456"  # [CRITICAL-SEC]
DB_PASSWORD = "super_secret_123"  # [CRITICAL-SEC]
JWT_SECRET = "mysecretkey"        # [CRITICAL-SEC]

# CORRECT:
API_KEY = os.environ.get('API_KEY')
DB_PASSWORD = vault.get_secret('db_password')
JWT_SECRET = os.environ.get('JWT_SECRET')
```

### Pattern 2: Missing Authorization

```python
# RED FLAG: No authorization check
@app.route('/api/users/<user_id>/data')
def get_user_data(user_id):
    return db.get_user_data(user_id)  # Any user can access any data!

# CORRECT: 
@app.route('/api/users/<user_id>/data')
@login_required
def get_user_data(user_id):
    if current_user.id != int(user_id) and not current_user.is_admin:
        abort(403)
    return db.get_user_data(user_id)
```

### Pattern 3: Unsafe Deserialization

```python
# RED FLAG: Pickle with untrusted data
import pickle

@app.route('/api/import', methods=['POST'])
def import_data():
    data = pickle.loads(request.data)  # Remote Code Execution!
    
# CORRECT:
import json

@app.route('/api/import', methods=['POST'])
def import_data():
    data = json.loads(request.data)  # Safe - JSON only
    validate_schema(data)  # Validate structure
```

### Pattern 4: Race Conditions

```python
# RED FLAG: Time-of-check to time-of-use (TOCTOU)
@app.route('/api/withdraw', methods=['POST'])
def withdraw():
    amount = request.json['amount']
    balance = get_balance(current_user.id)
    
    if balance >= amount:  # Check
        # Another request could change balance HERE!
        deduct_balance(current_user.id, amount)  # Use
    
# CORRECT: Use database-level locking
@app.route('/api/withdraw', methods=['POST'])
def withdraw():
    amount = request.json['amount']
    with db.session.begin():
        user = User.query.with_for_update().get(current_user.id)
        if user.balance >= amount:
            user.balance -= amount
            db.session.commit()
```

---

## Automated Review Tools Integration

```yaml
# GitHub Actions - Automated Security Review
name: Security Review

on: [pull_request]

jobs:
  security-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      # Auto-comment security findings on PR
      - name: Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: p/security-audit
          generateSarif: true
      
      # Check for secrets
      - name: Gitleaks
        uses: gitleaks/gitleaks-action@v2
      
      # Dependency check
      - name: Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  # Require security review from CODEOWNERS
  # .github/CODEOWNERS:
  # /src/auth/**    @security-team
  # /src/payment/** @security-team
  # /src/crypto/**  @security-team
```

### CODEOWNERS for Security-Sensitive Files

```
# .github/CODEOWNERS

# Security team must review these paths
/src/auth/           @org/security-team
/src/crypto/         @org/security-team
/src/payment/        @org/security-team
/config/security/    @org/security-team
Dockerfile           @org/security-team
docker-compose*.yml  @org/security-team
*.tf                 @org/security-team @org/platform-team
.github/workflows/   @org/security-team @org/platform-team
```

---

## Security Review Workflow

```
PR Created
    |
    v
+---+---+---+---+---+---+---+---+
| Automated Checks (Parallel)    |
| +-------+  +----+  +--------+ |
| |Semgrep|  |Snyk|  |Gitleaks| |
| +---+---+  +-+--+  +---+----+ |
+-----+--------+----------+-----+
      |        |          |
      v        v          v
+----------------------------+
| Results posted as PR       |
| comments                   |
+----------------------------+
      |
      v
+----------------------------+     +---------------------------+
| Developer fixes            |---->| Security Champion reviews |
| automated findings         |     | (manual review)           |
+----------------------------+     +---+----+------------------+
                                       |    |
                              +--------+    +--------+
                              |                      |
                         Approved               Changes
                              |                Requested
                              v                      |
                    +-------------------+            |
                    |  Merge to main    |            |
                    +-------------------+    +-------v-------+
                                             | Developer     |
                                             | addresses     |
                                             | feedback      |
                                             +-------+-------+
                                                     |
                                                     +---> Re-review
```

---

## Real-World Scenario: Uber's Authorization Bug

```
THE BUG:
+------------------------------------------------------------+
| Uber's API lacked object-level authorization checks.        |
| Any authenticated user could access other users' trip       |
| history by changing the user ID in the API request.         |
|                                                              |
| GET /api/trips?user_id=12345  (own trips)                   |
| GET /api/trips?user_id=67890  (someone else's trips!)       |
|                                                              |
| This could have been caught by a security code reviewer     |
| checking for IDOR (Insecure Direct Object Reference).       |
+------------------------------------------------------------+

CODE REVIEW WOULD CHECK:
+------------------------------------------------------------+
| Q: "Is the user_id in the request compared against the      |
|     authenticated user's ID?"                                |
| A: No! --> IDOR vulnerability found                         |
+------------------------------------------------------------+
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Reviews take too long | Reviewing too much code at once | Keep PRs small (< 400 lines), review in < 1 hour |
| Reviewers miss security issues | Not trained in security | Security code review training + checklists |
| Developers see reviews as criticism | Poor review culture | Frame as learning, use constructive language |
| Critical paths not reviewed | No CODEOWNERS | Set up CODEOWNERS for security-sensitive files |
| Automated tools too noisy | Default rules too broad | Tune rules, focus on high/critical severity |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Security Code Review** | Manual review focused on finding security vulnerabilities |
| **Automated + Manual** | Use SAST for breadth, manual review for depth |
| **Review Checklist** | Structured list covering auth, input, data, errors, logic |
| **Severity Labels** | CRITICAL/HIGH/MEDIUM/LOW tags for prioritization |
| **CODEOWNERS** | Require security team review for sensitive files |
| **IDOR Check** | Verify object-level authorization in every endpoint |
| **Race Conditions** | Check for TOCTOU bugs in financial/state operations |
| **Small PRs** | Keep under 400 lines for effective reviews |

---

## Quick Revision Questions

1. **Why is manual code review important even with SAST tools?**
   > SAST tools can't understand business logic, detect authorization flaws requiring context, or identify design-level vulnerabilities. Manual review catches logic bugs, IDOR, race conditions, and subtle issues that pattern matching misses.

2. **What is IDOR and how do you check for it during code review?**
   > Insecure Direct Object Reference — when a user can access another user's data by changing an ID parameter. Check by verifying that every endpoint compares the requested resource's owner against the authenticated user.

3. **What makes a good security review comment?**
   > It should identify the vulnerability type (CWE), explain why it's dangerous, show the vulnerable code, provide a suggested fix with code, and link to a reference. This teaches the developer and speeds up remediation.

4. **What is a TOCTOU race condition and how is it prevented?**
   > Time-of-Check to Time-of-Use: checking a condition and then acting on it without atomicity. Between check and use, another process can change the state. Prevent with database locks (`SELECT ... FOR UPDATE`), atomic operations, or transactions.

5. **Why should CODEOWNERS include security-sensitive file paths?**
   > It ensures that changes to auth, crypto, payment, infrastructure, and CI/CD files require explicit approval from the security team, preventing accidental or malicious security weakening.

6. **What is the recommended maximum PR size for effective security review?**
   > Under 400 lines of code. Larger PRs lead to reviewer fatigue and missed vulnerabilities. Studies show review effectiveness drops significantly above this threshold.

---

[← Previous: 2.3 Secure Coding Guidelines](03-secure-coding-guidelines.md) | [Next: 2.5 Developer Security Training →](05-developer-security-training.md)

[Back to Table of Contents](../README.md)
