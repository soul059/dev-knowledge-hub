# Chapter 3.5: Fixing SAST Vulnerabilities

## Overview

Finding vulnerabilities is only valuable if they are fixed. This chapter covers systematic approaches to prioritizing, remediating, and verifying SAST findings, including common fix patterns for the most frequent vulnerability classes.

---

## Remediation Workflow

```
VULNERABILITY REMEDIATION LIFECYCLE:

  SAST Finding
      │
      ▼
  ┌──────────────┐      ┌──────────────┐
  │ 1. TRIAGE     │─FP──→│ Suppress     │
  │ True/False?   │      │ + Document   │
  └──────┬───────┘      └──────────────┘
         │ True Positive
         ▼
  ┌──────────────┐
  │ 2. PRIORITIZE │  Severity + Exploitability + Impact
  │ Risk score    │
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │ 3. ASSIGN     │  Owner based on CODEOWNERS / file blame
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │ 4. FIX        │  Apply secure coding pattern
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │ 5. VERIFY     │  Re-scan to confirm fix
  └──────┬───────┘
         ▼
  ┌──────────────┐
  │ 6. CLOSE      │  Document + update knowledge base
  └──────────────┘
```

---

## Prioritization Matrix

```
  PRIORITY MATRIX (Risk = Severity x Exploitability):

                    EXPLOITABILITY
                Low        Medium       High
           ┌──────────┬──────────┬──────────┐
  High     │ Medium   │ High     │ Critical │  Fix: 24h
  SEVERITY ├──────────┼──────────┼──────────┤
  Medium   │ Low      │ Medium   │ High     │  Fix: 1 week
           ├──────────┼──────────┼──────────┤
  Low      │ Info     │ Low      │ Medium   │  Fix: 1 month
           └──────────┴──────────┴──────────┘

  REMEDIATION SLAs:
  ┌──────────┬─────────────┬─────────────────────────┐
  │ Priority │ Fix Within  │ Example                  │
  ├──────────┼─────────────┼─────────────────────────┤
  │ Critical │ 24 hours    │ SQLi in login endpoint   │
  │ High     │ 7 days      │ XSS in user profile      │
  │ Medium   │ 30 days     │ Missing CSRF token       │
  │ Low      │ 90 days     │ Verbose error messages   │
  └──────────┴─────────────┴─────────────────────────┘
```

---

## Common Fix Patterns

### 1. SQL Injection Fix

```python
# VULNERABLE: String concatenation
def get_user(username):
    query = "SELECT * FROM users WHERE name = '" + username + "'"
    return db.execute(query)

# FIXED: Parameterized query
def get_user(username):
    query = "SELECT * FROM users WHERE name = %s"
    return db.execute(query, (username,))

# FIXED: Using ORM (SQLAlchemy)
def get_user(username):
    return User.query.filter_by(name=username).first()
```

### 2. Cross-Site Scripting (XSS) Fix

```javascript
// VULNERABLE: Direct DOM insertion
function showMessage(userInput) {
    document.getElementById('output').innerHTML = userInput;
}

// FIXED: Text content (no HTML parsing)
function showMessage(userInput) {
    document.getElementById('output').textContent = userInput;
}

// FIXED: DOMPurify for rich content
import DOMPurify from 'dompurify';
function showMessage(userInput) {
    document.getElementById('output').innerHTML = DOMPurify.sanitize(userInput);
}
```

### 3. Path Traversal Fix

```java
// VULNERABLE: Direct file access with user input
public File getFile(String filename) {
    return new File("/uploads/" + filename);
}

// FIXED: Validate and canonicalize path
public File getFile(String filename) {
    // Remove path separators
    String safeName = FilenameUtils.getName(filename);
    
    File baseDir = new File("/uploads/").getCanonicalFile();
    File targetFile = new File(baseDir, safeName).getCanonicalFile();
    
    // Verify file is within allowed directory
    if (!targetFile.toPath().startsWith(baseDir.toPath())) {
        throw new SecurityException("Path traversal attempt blocked");
    }
    return targetFile;
}
```

### 4. Hardcoded Secrets Fix

```python
# VULNERABLE: Hardcoded credentials
DATABASE_URL = "postgresql://admin:SuperSecret123@db.prod.internal:5432/app"
API_KEY = "sk-live-abc123def456"

# FIXED: Environment variables
import os
DATABASE_URL = os.environ["DATABASE_URL"]
API_KEY = os.environ["API_KEY"]

# FIXED: Secrets manager
from cloud_secrets import get_secret
DATABASE_URL = get_secret("prod/database/url")
API_KEY = get_secret("prod/api/key")
```

### 5. Insecure Deserialization Fix

```java
// VULNERABLE: Deserializing untrusted data
ObjectInputStream ois = new ObjectInputStream(request.getInputStream());
Object obj = ois.readObject();  // Arbitrary code execution risk

// FIXED: Use safe serialization format
import com.fasterxml.jackson.databind.ObjectMapper;
ObjectMapper mapper = new ObjectMapper();
// Disable default typing to prevent polymorphic deserialization attacks
mapper.deactivateDefaultTyping();
UserDTO user = mapper.readValue(request.getInputStream(), UserDTO.class);
```

### 6. Insecure Cryptography Fix

```python
# VULNERABLE: Weak hashing
import hashlib
password_hash = hashlib.md5(password.encode()).hexdigest()

# FIXED: Use bcrypt with salt
import bcrypt
password_hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))

# Verify
if bcrypt.checkpw(password.encode(), stored_hash):
    print("Login successful")
```

```python
# VULNERABLE: ECB mode encryption
from Crypto.Cipher import AES
cipher = AES.new(key, AES.MODE_ECB)  # ECB leaks patterns

# FIXED: GCM mode (authenticated encryption)
from Crypto.Cipher import AES
cipher = AES.new(key, AES.MODE_GCM)
ciphertext, tag = cipher.encrypt_and_digest(plaintext)
# Store: nonce + tag + ciphertext
```

---

## Verification Process

```
VERIFY THE FIX:

  ┌─────────────────────────────────────────────────────────┐
  │                                                          │
  │  Step 1: LOCAL SCAN                                      │
  │  $ semgrep --config auto path/to/fixed/file.py          │
  │  → Confirm finding no longer appears                     │
  │                                                          │
  │  Step 2: UNIT TEST                                       │
  │  Write a test that validates the secure behavior          │
  │  → e.g., test that SQL special chars are escaped          │
  │                                                          │
  │  Step 3: CI/CD SCAN                                      │
  │  Push fix → Pipeline runs SAST → Quality gate passes     │
  │                                                          │
  │  Step 4: PEER REVIEW                                     │
  │  Security-focused code review of the fix                  │
  │  → Ensure fix is complete, no regressions                 │
  │                                                          │
  │  Step 5: REGRESSION TEST                                 │
  │  Run full test suite to ensure fix didn't break anything  │
  │                                                          │
  └─────────────────────────────────────────────────────────┘
```

### Security Unit Test Example

```python
# test_sql_injection.py
import pytest
from app.db import get_user

class TestSQLInjectionPrevention:
    """Verify SQL injection fixes are effective."""
    
    def test_normal_input(self, db_session):
        """Normal input works correctly."""
        user = get_user("alice")
        assert user.name == "alice"
    
    def test_sql_injection_single_quote(self, db_session):
        """Single quote doesn't break query."""
        user = get_user("'; DROP TABLE users; --")
        assert user is None  # No user found, no error
    
    def test_sql_injection_union(self, db_session):
        """UNION injection doesn't extract data."""
        user = get_user("' UNION SELECT password FROM users --")
        assert user is None
    
    def test_sql_injection_boolean(self, db_session):
        """Boolean injection doesn't bypass auth."""
        user = get_user("' OR '1'='1")
        assert user is None  # Should not return all users
```

---

## Bulk Remediation Strategies

```
WHEN YOU HAVE 100+ FINDINGS OF THE SAME TYPE:

  Strategy 1: CENTRALIZED FIX
  ┌────────────────────────────────────────────────────┐
  │ Create a shared utility/helper                      │
  │ Replace all instances across codebase               │
  │                                                      │
  │ Example: Create sanitize_input() helper             │
  │ Then search-and-replace all raw input uses          │
  └────────────────────────────────────────────────────┘

  Strategy 2: CODEMOD (Automated Refactoring)
  ┌────────────────────────────────────────────────────┐
  │ Use tools like Semgrep autofix or jscodeshift       │
  │                                                      │
  │ # Semgrep rule with autofix                          │
  │ rules:                                               │
  │   - id: use-parameterized-query                      │
  │     pattern: db.execute("..." + $VAR + "...")        │
  │     fix: db.execute("... %s ...", ($VAR,))           │
  └────────────────────────────────────────────────────┘

  Strategy 3: FRAMEWORK-LEVEL FIX
  ┌────────────────────────────────────────────────────┐
  │ Enable framework security defaults                  │
  │                                                      │
  │ Django: CSRF middleware enabled by default           │
  │ Rails:  Content Security Policy header              │
  │ Spring: CSRF protection via SecurityFilterChain     │
  └────────────────────────────────────────────────────┘
```

### Semgrep Autofix Rule

```yaml
# .semgrep/autofix-rules.yml
rules:
  - id: replace-innerhtml-with-textcontent
    patterns:
      - pattern: $EL.innerHTML = $VALUE
    fix: $EL.textContent = $VALUE
    message: "Use textContent instead of innerHTML to prevent XSS"
    languages: [javascript, typescript]
    severity: ERROR

  - id: replace-md5-with-bcrypt
    pattern: hashlib.md5($PASSWORD.encode()).hexdigest()
    fix: bcrypt.hashpw($PASSWORD.encode(), bcrypt.gensalt(rounds=12))
    message: "Use bcrypt instead of MD5 for password hashing"
    languages: [python]
    severity: ERROR
```

```bash
# Apply autofix
semgrep --config .semgrep/autofix-rules.yml --autofix .

# Dry-run first
semgrep --config .semgrep/autofix-rules.yml --autofix --dryrun .
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Fix introduced new findings | Incomplete remediation | Re-scan after every fix; ensure the fix is holistic |
| Same finding reappears in other files | Pattern-level issue, not instance | Use bulk remediation or framework-level fix |
| Fix breaks existing functionality | Security fix too aggressive | Write regression tests before fixing; use feature flags |
| Developer doesn't know how to fix | Lack of secure coding knowledge | Provide fix examples in tool configuration; link to docs |
| Autofix produces incorrect code | Rule pattern too generic | Test autofix on sample code first; review diffs |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Remediation Workflow** | Triage → Prioritize → Assign → Fix → Verify → Close |
| **Prioritization Matrix** | Risk = Severity × Exploitability |
| **Parameterized Queries** | Primary fix for SQL injection |
| **Output Encoding** | Primary fix for XSS |
| **Path Canonicalization** | Primary fix for path traversal |
| **Autofix** | Automated code transformation using Semgrep rules |
| **Security Unit Tests** | Tests validating secure behavior against attack inputs |
| **Bulk Remediation** | Centralized helpers, codemods, framework defaults |

---

## Quick Revision Questions

1. **What are the six steps of the remediation workflow?**
   > Triage (true/false positive), Prioritize (severity × exploitability), Assign (to code owner), Fix (apply secure pattern), Verify (re-scan + test), Close (document + update knowledge base).

2. **What is the recommended SLA for critical vulnerabilities?**
   > 24 hours. Critical findings (high severity + high exploitability) such as SQL injection in authentication endpoints should be treated as emergency fixes.

3. **What is the primary fix for SQL injection?**
   > Use parameterized queries (prepared statements) or an ORM. Never concatenate user input into SQL strings.

4. **How can you fix 100+ instances of the same vulnerability efficiently?**
   > Three strategies: (1) Create a centralized helper function and replace all instances, (2) Use codemods/autofix tools like Semgrep autofix for automated refactoring, (3) Enable framework-level security defaults that protect all endpoints.

5. **Why should you write security unit tests for fixes?**
   > To verify the fix actually prevents the attack, ensure the fix doesn't regress in future changes, serve as documentation of the expected secure behavior, and provide confidence during code refactoring.

6. **What is Semgrep autofix and how does it work?**
   > Semgrep autofix allows you to define a `fix` field in custom rules that specifies a code transformation. When run with `--autofix`, Semgrep automatically rewrites matching code to the secure version. Always use `--dryrun` first to review changes.

---

[← Previous: 3.4 False Positive Management](04-false-positive-management.md) | [Next: 4.1 DAST Concepts →](../04-DAST/01-dast-concepts.md)

[Back to Table of Contents](../README.md)
