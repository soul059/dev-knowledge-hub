# Chapter 3.2: SAST Tools (SonarQube, Checkmarx, Semgrep)

## Overview

This chapter covers the three most popular SAST tools, their architecture, configuration, and practical usage in DevSecOps pipelines.

---

## Tool Comparison

```
+================================================================+
|                    SAST TOOL COMPARISON                          |
+================================================================+
|                                                                  |
|  SONARQUBE              CHECKMARX            SEMGREP            |
|  +------------------+   +------------------+ +------------------+|
|  | Open Source +     |   | Commercial       | | Open Source +    ||
|  | Commercial       |   | Enterprise       | | Commercial       ||
|  |                  |   |                  | |                  ||
|  | Code quality +   |   | Deep security    | | Pattern-based    ||
|  | security        |   | analysis         | | lightweight scan ||
|  |                  |   |                  | |                  ||
|  | 30+ languages   |   | 25+ languages    | | 30+ languages    ||
|  |                  |   |                  | |                  ||
|  | Self-hosted +    |   | Cloud + On-prem  | | CLI + Cloud      ||
|  | Cloud           |   |                  | |                  ||
|  +------------------+   +------------------+ +------------------+|
+================================================================+
```

| Feature | SonarQube | Checkmarx | Semgrep |
|---------|-----------|-----------|---------|
| **License** | Community (free) + Enterprise | Commercial | OSS (free) + Team/Enterprise |
| **Analysis Type** | AST + Pattern | Deep data flow + taint | Pattern matching (AST) |
| **Languages** | 30+ | 25+ | 30+ |
| **False Positives** | Medium | Low-Medium | Low |
| **Speed** | Medium | Slow (thorough) | Very Fast |
| **Custom Rules** | Yes (Java API) | Yes (CxQL) | Yes (YAML - easy!) |
| **CI/CD Integration** | Excellent | Excellent | Excellent |
| **Learning Curve** | Medium | High | Low |
| **Best For** | Code quality + security | Enterprise security | Developer-first scanning |

---

## SonarQube

### Architecture

```
SONARQUBE ARCHITECTURE:

  +-------------------+         +-------------------+
  |   Developer IDE   |         |   CI/CD Server    |
  |   (SonarLint)     |         |   (Jenkins/GH)    |
  +--------+----------+         +--------+----------+
           |                              |
           | Real-time                    | Scan on build
           | feedback                     |
           |                     +--------v----------+
           |                     |   SonarScanner    |
           |                     |   (CLI/Maven/     |
           |                     |    Gradle plugin) |
           |                     +--------+----------+
           |                              |
           |                              | Analysis results
           |                              |
           |                     +--------v----------+
           +-------------------->|   SonarQube       |
                                 |   Server          |
                                 |   +------------+  |
                                 |   | Web UI     |  |
                                 |   | Rules      |  |
                                 |   | Quality    |  |
                                 |   |   Gates    |  |
                                 |   | Dashboards |  |
                                 |   +------+-----+  |
                                 |          |         |
                                 |   +------v-----+  |
                                 |   | Database   |  |
                                 |   | (PostgreSQL)|  |
                                 |   +------------+  |
                                 +-------------------+
```

### Installation & Setup

```bash
# Docker-based SonarQube setup
docker run -d --name sonarqube \
  -p 9000:9000 \
  -v sonarqube_data:/opt/sonarqube/data \
  -v sonarqube_logs:/opt/sonarqube/logs \
  -v sonarqube_extensions:/opt/sonarqube/extensions \
  sonarqube:community

# Default credentials: admin/admin (change immediately!)
```

```properties
# sonar-project.properties
sonar.projectKey=my-project
sonar.projectName=My Project
sonar.sources=src
sonar.tests=tests
sonar.language=py
sonar.python.version=3.11
sonar.sourceEncoding=UTF-8

# Security-specific settings
sonar.security.hotspots.review.priority=HIGH,CRITICAL
```

```yaml
# GitHub Actions integration
- name: SonarQube Scan
  uses: sonarqube/sonarqube-scan-action@v2
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
    SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

- name: SonarQube Quality Gate
  uses: sonarqube/sonarqube-quality-gate-action@v1
  timeout-minutes: 5
  env:
    SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

---

## Checkmarx

### Architecture

```
CHECKMARX ONE ARCHITECTURE:

  +-------------------+
  |   Source Code      |
  |   Repository       |
  +--------+----------+
           |
  +--------v----------+
  |   Checkmarx        |
  |   Engine            |
  |   +-------------+  |
  |   | CxSAST      |  | ← Static Analysis
  |   +-------------+  |
  |   | CxSCA       |  | ← Software Composition
  |   +-------------+  |
  |   | CxIaC       |  | ← Infrastructure as Code
  |   +-------------+  |
  |   | CxContainer |  | ← Container Scanning
  |   +-------------+  |
  +--------+----------+
           |
  +--------v----------+
  |   Results          |
  |   Dashboard        |
  |   - Risk scoring   |
  |   - Remediation    |
  |   - Trend analysis |
  +-------------------+
```

### Configuration

```yaml
# Checkmarx CLI Scan
# Install: npm install -g @checkmarx/ast-cli

# Run scan
cx scan create \
  --project-name "my-project" \
  -s . \
  --scan-types sast \
  --branch main

# GitHub Actions integration
- name: Checkmarx SAST Scan
  uses: checkmarx/ast-github-action@main
  with:
    project_name: my-project
    cx_client_id: ${{ secrets.CX_CLIENT_ID }}
    cx_client_secret: ${{ secrets.CX_CLIENT_SECRET }}
    cx_tenant: ${{ secrets.CX_TENANT }}
    additional_params: --scan-types sast --sast-preset-name "Security"
```

---

## Semgrep

### Why Semgrep is Developer-Friendly

```
SEMGREP KEY FEATURES:

  1. SPEED: Scans 20,000+ files in seconds
  2. YAML RULES: Easy to write custom rules
  3. NO BUILD REQUIRED: Works on raw source code
  4. LOW FALSE POSITIVES: Pattern-based precision
  5. COMMUNITY RULES: 3,000+ pre-built rules
```

### Installation & Usage

```bash
# Install Semgrep
pip install semgrep
# or
brew install semgrep

# Run with community security rules
semgrep --config auto .

# Run specific rule sets
semgrep --config p/security-audit .
semgrep --config p/owasp-top-ten .
semgrep --config p/cwe-top-25 .

# Run with multiple rule sets
semgrep --config p/security-audit --config p/secrets .

# Output in SARIF format (for CI/CD integration)
semgrep --config auto --sarif --output results.sarif .

# Scan only changed files (fast incremental)
semgrep --config auto --diff-depth 2 .
```

### Writing Custom Semgrep Rules

```yaml
# custom-rules.yml
rules:
  # Rule 1: Detect SQL Injection
  - id: sql-injection-concat
    patterns:
      - pattern: |
          $QUERY = "..." + $INPUT + "..."
          $CURSOR.execute($QUERY)
    message: >
      SQL Injection: User input concatenated into query.
      Use parameterized queries instead.
    severity: ERROR
    languages: [python]
    metadata:
      cwe: CWE-89
      owasp: A03:2021

  # Rule 2: Detect Hardcoded Passwords
  - id: hardcoded-password
    patterns:
      - pattern: |
          $VAR = "..."
      - metavariable-regex:
          metavariable: $VAR
          regex: (password|passwd|pwd|secret|api_key)
    message: >
      Hardcoded secret detected in variable '$VAR'.
      Use environment variables or a secrets manager.
    severity: WARNING
    languages: [python, javascript, java]
    metadata:
      cwe: CWE-798

  # Rule 3: Detect Missing Authentication
  - id: flask-no-auth
    patterns:
      - pattern: |
          @app.route(...)
          def $FUNC(...):
              ...
      - pattern-not: |
          @app.route(...)
          @login_required
          def $FUNC(...):
              ...
    message: >
      Endpoint '$FUNC' has no @login_required decorator.
      Ensure authentication is required.
    severity: WARNING
    languages: [python]

  # Rule 4: Detect eval() with user input
  - id: dangerous-eval
    patterns:
      - pattern: eval($INPUT)
      - pattern-not: eval("...")  # Allow literal strings
    message: >
      eval() with dynamic input can lead to code injection.
      Use safer alternatives like ast.literal_eval().
    severity: ERROR
    languages: [python]
    metadata:
      cwe: CWE-95
```

```bash
# Run custom rules
semgrep --config custom-rules.yml .

# Test rules against a specific file
semgrep --config custom-rules.yml src/api/users.py
```

---

## Tool Selection Guide

```
CHOOSING THE RIGHT SAST TOOL:

  START HERE
      |
      v
  +---+---+
  | Budget? |
  +---+---+
      |
  +---+---+---+
  |           |
  Free    Enterprise
  |           |
  v           v
  +------+   +------+
  |Small |   |Large |
  |team? |   |org?  |
  +--+---+   +--+---+
     |          |
     v          v
  Semgrep    +------+
  OSS        |Need  |
             |deep  |
             |flow? |
             +--+---+
                |
           +----+----+
           |         |
          Yes       No
           |         |
           v         v
        Checkmarx  SonarQube
        CxSAST    Enterprise
```

---

## Real-World Scenario: Integrating Multiple SAST Tools

```
DEFENSE IN DEPTH: MULTI-TOOL APPROACH

  +-------------------+
  |   Code Commit     |
  +--------+----------+
           |
  +--------v----------+     Speed: ~5 seconds
  |   Semgrep          |     Catches: Pattern-based vulns
  |   (Fast, precise)  |     False Positives: Low
  +--------+----------+
           |
  +--------v----------+     Speed: ~2-5 minutes
  |   SonarQube        |     Catches: Code quality + security
  |   (Comprehensive)  |     False Positives: Medium
  +--------+----------+
           |
  +--------v----------+     Speed: ~10-30 minutes
  |   Checkmarx        |     Catches: Deep data flow vulns
  |   (Deep analysis)  |     False Positives: Low-Medium
  +--------+----------+
           |
  +--------v----------+
  |   Unified Report   |
  |   (Deduplicated)   |
  +-------------------+

  WHY MULTIPLE TOOLS:
  - Each tool has different strengths
  - One may catch what another misses
  - Use fast tools for immediate feedback
  - Use deep tools for thorough analysis
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| SonarQube won't start | Not enough memory | Set `sonar.web.javaOpts=-Xmx512m`, need min 2GB RAM |
| Semgrep rule not matching | Pattern syntax wrong | Use `semgrep --test` and Semgrep Playground to debug |
| Checkmarx scan too slow | Full project scan every time | Enable incremental scanning, exclude test files |
| Tool reports different results | Different rule versions | Pin tool and rule versions in CI configuration |
| Can't scan compiled code | Tool requires source access | Ensure source is available, not just build artifacts |

---

## Summary Table

| Tool | Type | Best For | Cost | Speed |
|------|------|----------|------|-------|
| **SonarQube** | AST + Pattern | Code quality + security | Free (Community) / Paid | Medium |
| **Checkmarx** | Deep data flow | Enterprise security programs | Paid | Slow |
| **Semgrep** | Pattern (AST) | Developer-first, custom rules | Free (OSS) / Paid | Very Fast |
| **Bandit** | AST (Python only) | Python-specific scanning | Free | Fast |
| **SpotBugs** | Bytecode (Java) | Java/JVM security | Free | Medium |
| **ESLint Security** | AST (JS) | JavaScript/TypeScript | Free | Fast |

---

## Quick Revision Questions

1. **What are the key differences between SonarQube, Checkmarx, and Semgrep?**
   > SonarQube focuses on code quality + security with broad language support. Checkmarx provides deep data flow analysis for enterprise security. Semgrep is fast, developer-friendly, with easy YAML-based custom rules and low false positives.

2. **Why might you use multiple SAST tools rather than just one?**
   > Different tools use different analysis techniques (pattern matching vs. data flow) and have different strengths. One tool may catch vulnerabilities another misses. Use fast tools (Semgrep) for immediate feedback and thorough tools (Checkmarx) for deep analysis.

3. **How do you write a custom Semgrep rule?**
   > Using YAML files with `pattern:` matching source code patterns, `metavariable-regex:` for variable constraints, `pattern-not:` for exclusions, `message:` for developer feedback, and metadata like CWE/OWASP references.

4. **What is a SonarQube Quality Gate?**
   > A set of conditions that code must meet to pass (e.g., no new critical vulnerabilities, code coverage above 80%). If conditions aren't met, the pipeline is blocked.

5. **How would you reduce SAST scan time in CI/CD?**
   > Use incremental scanning (only changed files), exclude test/generated code, use caching, run fast tools (Semgrep) first and deep tools (Checkmarx) in parallel or on scheduled runs.

6. **What is the advantage of Semgrep's pattern-based approach?**
   > Patterns match directly against code structure (AST), making them intuitive to write and read. They produce fewer false positives since they're precise, and they're extremely fast since they don't need to build full data flow graphs.

---

[← Previous: 3.1 SAST Concepts](01-sast-concepts.md) | [Next: 3.3 SAST CI/CD Integration →](03-sast-cicd-integration.md)

[Back to Table of Contents](../README.md)
