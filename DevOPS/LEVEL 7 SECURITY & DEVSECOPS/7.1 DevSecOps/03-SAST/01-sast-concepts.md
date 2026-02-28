# Chapter 3.1: SAST Concepts

## Overview

**Static Application Security Testing (SAST)** analyzes source code, bytecode, or binary code for security vulnerabilities **without executing the application**. It's the equivalent of a security-focused spell-checker for your code.

---

## How SAST Works

```
+====================================================================+
|                    SAST ANALYSIS PIPELINE                            |
+====================================================================+
|                                                                      |
|  SOURCE CODE                                                         |
|  +----------+     +-----------+     +----------+     +-----------+  |
|  |          |     |           |     |          |     |           |  |
|  |  Parse   |---->|  Build    |---->| Apply    |---->| Report    |  |
|  |  Code    |     |  AST/CFG  |     | Rules    |     | Findings  |  |
|  |          |     |           |     |          |     |           |  |
|  +----------+     +-----------+     +----------+     +-----------+  |
|                                                                      |
|  DETAIL:                                                             |
|  1. PARSE: Tokenize source into language-specific structures        |
|  2. BUILD: Create Abstract Syntax Tree & Control Flow Graph         |
|  3. APPLY: Match patterns against security rules database           |
|  4. REPORT: Generate findings with severity, location, fix advice   |
+====================================================================+
```

### Abstract Syntax Tree (AST) Analysis

```
CODE:
  result = db.execute("SELECT * FROM users WHERE id = " + user_id)

AST REPRESENTATION:
                    Assignment
                   /          \
              Variable      MethodCall
              "result"      /    |     \
                        Object Method  Argument
                        "db"  "execute" Concatenation
                                        /           \
                                   StringLiteral   Variable
                                   "SELECT *..."   "user_id"
                                   
  SAST RULE: "If MethodCall('execute') has Concatenation 
              with Variable as child → SQL Injection risk"
  
  FINDING: SQL Injection (CWE-89), Severity: HIGH
```

### Data Flow Analysis (Taint Tracking)

```
TAINT ANALYSIS:

  SOURCE (untrusted input):
  +------------------------------------------------------------+
  |  user_input = request.args.get('name')  ← TAINTED          |
  +------------------------------------------------------------+
              |
              | Taint propagates
              v
  +------------------------------------------------------------+
  |  processed = user_input.strip()  ← STILL TAINTED           |
  |  query = "SELECT * FROM users WHERE name = '" +             |
  |           processed + "'"  ← TAINTED                        |
  +------------------------------------------------------------+
              |
              | Taint reaches sink
              v
  SINK (dangerous function):
  +------------------------------------------------------------+
  |  cursor.execute(query)  ← SINK: SQL execution with taint!  |
  +------------------------------------------------------------+
              |
              v
  ALERT: Untrusted data flows from SOURCE to SINK
         without sanitization → SQL Injection!

  TAINT TRACKING CONCEPTS:
  +-------------------+---------------------------------------+
  | Source             | Where untrusted data enters           |
  |                    | (request params, cookies, headers)    |
  +-------------------+---------------------------------------+
  | Propagator         | Operations that pass taint            |
  |                    | (concatenation, assignment)           |
  +-------------------+---------------------------------------+
  | Sanitizer          | Functions that clean data             |
  |                    | (parameterized queries, encoding)     |
  +-------------------+---------------------------------------+
  | Sink               | Dangerous functions                   |
  |                    | (execute(), eval(), innerHTML)        |
  +-------------------+---------------------------------------+
```

---

## SAST vs Other Testing Methods

```
+================================================================+
|           COMPARISON: SAST vs DAST vs IAST vs SCA               |
+================================================================+
|                                                                  |
|  SAST (White-Box)          DAST (Black-Box)                     |
|  +----------------------+  +----------------------+             |
|  | Analyzes source code |  | Tests running app    |             |
|  | No execution needed  |  | Execution required   |             |
|  | Finds code-level     |  | Finds runtime        |             |
|  |   vulnerabilities    |  |   vulnerabilities    |             |
|  | Early in pipeline    |  | Late in pipeline     |             |
|  | Many false positives |  | Fewer false positives|             |
|  | Language-dependent   |  | Language-independent |             |
|  +----------------------+  +----------------------+             |
|                                                                  |
|  IAST (Grey-Box)           SCA (Composition)                    |
|  +----------------------+  +----------------------+             |
|  | Agents inside app    |  | Analyzes dependencies|             |
|  | Runtime + code       |  | Known CVEs in libs   |             |
|  |   context            |  | License compliance   |             |
|  | Fewer false positives|  | No custom code scan  |             |
|  | Requires test suite  |  | Fast and accurate    |             |
|  +----------------------+  +----------------------+             |
+================================================================+
```

| Feature | SAST | DAST | IAST | SCA |
|---------|------|------|------|-----|
| **What it scans** | Source code | Running app | Running app + code | Dependencies |
| **When in pipeline** | Build | Test/Staging | Test | Build |
| **Execution needed** | No | Yes | Yes | No |
| **False positive rate** | High | Medium | Low | Low |
| **Coverage** | All code paths | Only tested paths | Tested paths | All dependencies |
| **Speed** | Fast | Slow | Medium | Fast |
| **Languages** | Language-specific | Language-independent | Language-specific | Package manager |

---

## Types of Vulnerabilities SAST Finds

```
+================================================================+
|           SAST DETECTION CAPABILITIES                           |
+================================================================+
|                                                                  |
|  HIGH ACCURACY:                                                  |
|  +----------------------------------------------------------+  |
|  | - SQL Injection (string concat in queries)               |  |
|  | - XSS (unencoded output)                                 |  |
|  | - Hardcoded secrets (API keys, passwords)                |  |
|  | - Path traversal (unsanitized file paths)                |  |
|  | - Command injection (unsanitized shell commands)         |  |
|  | - Insecure cryptography (weak algorithms, ECB mode)      |  |
|  | - Buffer overflows (C/C++ specific)                      |  |
|  +----------------------------------------------------------+  |
|                                                                  |
|  MEDIUM ACCURACY:                                                |
|  +----------------------------------------------------------+  |
|  | - Broken authentication (missing checks)                 |  |
|  | - CSRF (missing tokens)                                  |  |
|  | - Insecure deserialization                               |  |
|  | - SSRF (user-controlled URLs)                            |  |
|  | - Race conditions                                        |  |
|  +----------------------------------------------------------+  |
|                                                                  |
|  LOW ACCURACY (Needs manual review):                             |
|  +----------------------------------------------------------+  |
|  | - Business logic flaws                                   |  |
|  | - Authorization bypass                                    |  |
|  | - Complex multi-step vulnerabilities                     |  |
|  | - Configuration issues                                   |  |
|  +----------------------------------------------------------+  |
+================================================================+
```

---

## SAST in the DevSecOps Pipeline

```
                    +-------------------+
                    |  Developer Commit |
                    +--------+----------+
                             |
              +--------------+--------------+
              |                             |
     +--------v--------+          +--------v--------+
     | IDE SAST Plugin  |          | Pre-commit Hook |
     | (SonarLint,      |          | (Semgrep)       |
     | Semgrep)          |          |                 |
     | IMMEDIATE         |          | BEFORE COMMIT   |
     +--------+----------+          +--------+--------+
              |                             |
              +--------------+--------------+
                             |
                    +--------v--------+
                    | CI/CD Pipeline   |
                    | SAST Scan        |
                    | (SonarQube,      |
                    |  Checkmarx)      |
                    | ON EVERY BUILD   |
                    +--------+--------+
                             |
                    +--------v--------+
                    | Quality Gate     |
                    | CRITICAL = BLOCK |
                    | HIGH = BLOCK     |
                    | MED = WARN       |
                    +--------+--------+
                             |
                   +---------+---------+
                   |                   |
              +----v----+        +----v----+
              |  PASS   |        |  FAIL   |
              | Continue|        | Block + |
              | Pipeline|        | Notify  |
              +---------+        +---------+
```

---

## Real-World Scenario: Finding a Critical Vulnerability with SAST

```
SCENARIO: E-commerce application

  SAST Scan detects:
  +------------------------------------------------------------+
  | FINDING: CWE-89 SQL Injection                               |
  | Severity: CRITICAL                                           |
  | File: /src/api/products.py, Line 42                         |
  |                                                              |
  | Code:                                                        |
  | def search_products(category):                               |
  |     query = "SELECT * FROM products WHERE category = '"     |
  |              + category + "' AND active = 1"                |
  |     results = db.execute(query)                             |
  |     return results                                           |
  |                                                              |
  | Taint flow:                                                  |
  | request.args['category'] → category → query → db.execute()  |
  |                                                              |
  | Fix: Use parameterized query                                 |
  | query = "SELECT * FROM products WHERE category = %s          |
  |          AND active = 1"                                     |
  | results = db.execute(query, (category,))                    |
  +------------------------------------------------------------+

  WITHOUT SAST:
  - Discovered in production after data breach
  - Cost: $500,000+ in incident response
  
  WITH SAST:
  - Discovered at build time, 2 minutes after commit
  - Cost: 30 minutes of developer time to fix
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| SAST scan takes too long | Scanning entire codebase every time | Use incremental analysis (only changed files) |
| Too many false positives | Rules too broad or not tuned | Tune rule sensitivity, create suppression file |
| Language not supported | Tool lacks parser for language | Use language-specific tool or Semgrep with custom rules |
| Can't find vuln in framework code | Tool doesn't understand the framework | Configure framework-specific rules and sources/sinks |
| Developers ignore findings | Too many low-severity alerts | Filter to HIGH/CRITICAL only in pipeline gates |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **SAST** | Static analysis of source code without execution |
| **AST Analysis** | Parsing code into tree structure for pattern matching |
| **Taint Tracking** | Following untrusted data from source to sink |
| **Source** | Where untrusted data enters (user input, external data) |
| **Sink** | Dangerous function where tainted data is used |
| **Sanitizer** | Function that cleans tainted data (encoding, parameterization) |
| **False Positive** | Finding reported that isn't actually a vulnerability |
| **Quality Gate** | Pipeline checkpoint that blocks on critical findings |
| **Incremental Scan** | Only scanning changed files for faster feedback |

---

## Quick Revision Questions

1. **What is SAST and how does it differ from DAST?**
   > SAST analyzes source code without running the application (white-box). DAST tests a running application by sending requests (black-box). SAST finds code-level issues early; DAST finds runtime issues later.

2. **What is taint tracking in SAST?**
   > Taint tracking follows untrusted data (from sources like user input) through the code (propagators like concatenation) to dangerous functions (sinks like SQL execute), flagging paths where no sanitization occurs.

3. **Why does SAST have a high false positive rate?**
   > SAST can't fully understand runtime context, complex data flows, or custom sanitization logic. It over-approximates to avoid missing real vulnerabilities, which leads to reporting safe code as vulnerable.

4. **At what stage of the CI/CD pipeline should SAST be run?**
   > At the build stage (after code commit, before deployment). Ideally also in the IDE (real-time) and pre-commit hooks (before commit) for even earlier feedback.

5. **What is an Abstract Syntax Tree (AST) and why is it used in SAST?**
   > An AST is a tree representation of code structure that captures the hierarchical relationships between code elements. SAST tools use it to apply pattern-matching rules that identify vulnerable code constructs.

6. **Name three types of vulnerabilities that SAST detects with high accuracy.**
   > SQL Injection (string concatenation in queries), hardcoded secrets (patterns matching API keys/passwords), and command injection (unsanitized input in shell commands).

---

[← Previous: 2.5 Developer Security Training](../02-Secure-Coding-Practices/05-developer-security-training.md) | [Next: 3.2 SAST Tools →](02-sast-tools.md)

[Back to Table of Contents](../README.md)
