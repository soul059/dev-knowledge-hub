# Chapter 4.6: Security Testing (SAST & DAST) in CI/CD

[← Previous: Performance Testing](05-performance-testing.md) | [Back to README](../README.md) | [Next: Test Automation Best Practices →](07-test-automation-best-practices.md)

---

## Overview

**Security testing** in CI/CD automates the detection of vulnerabilities before code reaches production. The two primary approaches are **SAST** (Static Application Security Testing) and **DAST** (Dynamic Application Security Testing), complemented by SCA, secrets scanning, and container scanning.

---

## Shift-Left Security

```
  TRADITIONAL SECURITY               SHIFT-LEFT SECURITY
  ════════════════════               ═══════════════════

  Code → Build → Test → Deploy       Code → Build → Test → Deploy
                              │       │      │       │
                          ┌───▼──┐   ┌▼──┐  ┌▼──┐  ┌▼──┐
                          │Sec   │   │SAS│  │SCA│  │DAS│
                          │Audit │   │T  │  │   │  │T  │
                          │(late)│   └───┘  └───┘  └───┘
                          └──────┘
  Cost to fix: $$$$$              Cost to fix: $
  Found late                      Found early
  Manual review                   Automated
```

---

## Security Testing Types

| Type | What | When | How |
|---|---|---|---|
| **SAST** | Scan source code for vulnerabilities | Build time | Pattern matching, data flow analysis |
| **DAST** | Scan running application for vulnerabilities | After deploy | HTTP requests, fuzzing |
| **SCA** | Scan dependencies for known CVEs | Build time | Compare deps against vulnerability DBs |
| **Secrets** | Detect leaked credentials in code | Pre-commit / CI | Regex + entropy analysis |
| **Container** | Scan container images for vulnerabilities | After build | Image layer analysis |
| **IaC** | Scan infrastructure code for misconfigurations | Build time | Policy checks on Terraform/CloudFormation |

---

## SAST — Static Application Security Testing

```
  ┌──────────────────────────────────────────────────┐
  │                  SAST FLOW                       │
  │                                                  │
  │  Source ──► Parse AST ──► Analyze ──► Report     │
  │  Code       (Abstract     Data flow   Findings   │
  │             Syntax Tree)  + patterns             │
  │                                                  │
  │  DETECTS:                                        │
  │  ✓ SQL Injection         ✓ XSS                   │
  │  ✓ Command Injection     ✓ Path Traversal        │
  │  ✓ Insecure Crypto       ✓ Buffer Overflow       │
  │  ✓ Hardcoded Secrets     ✓ Null Pointer Deref    │
  │                                                  │
  │  LIMITATIONS:                                    │
  │  ✕ False positives       ✕ No runtime context    │
  │  ✕ Can't test auth flows ✕ Language-specific     │
  └──────────────────────────────────────────────────┘
```

### SAST Tools

| Tool | Languages | Free/Paid | CI Integration |
|---|---|---|---|
| **Semgrep** | 30+ languages | Free (OSS) | GitHub, GitLab, CLI |
| **CodeQL** | C/C++, C#, Go, Java, JS, Python, Ruby | Free (GitHub) | GitHub Actions |
| **SonarQube** | 30+ languages | Free Community | Jenkins, GitHub, GitLab |
| **Snyk Code** | JS, Python, Java, Go, C# | Free tier | GitHub, CLI |
| **Bandit** | Python only | Free (OSS) | CLI |
| **ESLint Security** | JavaScript/TypeScript | Free (OSS) | CLI, npm |

### CodeQL (GitHub Actions)

```yaml
name: CodeQL Analysis
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * 1'    # Weekly Monday 6 AM

jobs:
  analyze:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    strategy:
      matrix:
        language: ['javascript', 'python']
    steps:
      - uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: ${{ matrix.language }}
          queries: +security-extended

      - name: Build (if compiled language)
        uses: github/codeql-action/autobuild@v3

      - name: Perform Analysis
        uses: github/codeql-action/analyze@v3
```

### Semgrep in CI

```yaml
# GitHub Actions
- name: Semgrep Scan
  uses: returntocorp/semgrep-action@v1
  with:
    config: >-
      p/owasp-top-ten
      p/security-audit
      p/secrets

# GitLab CI
semgrep:
  image: returntocorp/semgrep
  script:
    - semgrep ci --config auto
  variables:
    SEMGREP_RULES: "p/owasp-top-ten p/security-audit"
```

---

## DAST — Dynamic Application Security Testing

```
  ┌──────────────────────────────────────────────────┐
  │                  DAST FLOW                       │
  │                                                  │
  │  ┌──────────┐    HTTP     ┌──────────┐           │
  │  │  DAST    │───Requests─►│ Running  │           │
  │  │  Scanner │◄──Responses─│ App      │           │
  │  └────┬─────┘             └──────────┘           │
  │       │                                          │
  │       ▼                                          │
  │  ┌──────────┐                                    │
  │  │ Analyze  │   Tries: SQL injection, XSS,       │
  │  │ Responses│   auth bypass, CSRF, header        │
  │  │ & Report │   issues, SSL problems             │
  │  └──────────┘                                    │
  │                                                  │
  │  DETECTS:                        LIMITATIONS:    │
  │  ✓ Runtime vulnerabilities       ✕ Slow          │
  │  ✓ Auth/session issues           ✕ Needs running │
  │  ✓ Misconfigurations               app           │
  │  ✓ Server-level issues          ✕ Limited code   │
  └──────────────────────────────────────────────────┘
```

### DAST Tools

| Tool | Type | Free/Paid | Best For |
|---|---|---|---|
| **OWASP ZAP** | Full DAST | Free (OSS) | Comprehensive scanning |
| **Nuclei** | Template-based | Free (OSS) | Fast, customizable scans |
| **Burp Suite** | Full DAST | Paid (Pro) | Manual + automated |
| **Nikto** | Web scanner | Free (OSS) | Quick server checks |

### OWASP ZAP in CI

```yaml
# GitHub Actions — ZAP baseline scan
name: DAST Scan
on:
  pull_request:
    branches: [main]

jobs:
  zap-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Start application
      - run: docker-compose up -d
      - run: sleep 15   # Wait for app to start

      # Run ZAP baseline scan
      - name: ZAP Scan
        uses: zaproxy/action-baseline@v0.10.0
        with:
          target: 'http://localhost:3000'
          rules_file_name: 'zap-rules.tsv'
          cmd_options: '-a'

      # Results are automatically posted as PR comment
```

```
# zap-rules.tsv — Customize which rules fail the build
10021	IGNORE	(X-Content-Type-Options)
10038	WARN	(Content Security Policy)
40012	FAIL	(Cross Site Scripting - Reflected)
40014	FAIL	(Cross Site Scripting - Persistent)
90022	FAIL	(Application Error Disclosure)
```

---

## SCA — Software Composition Analysis

```yaml
# Dependabot — Automated dependency security updates
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: npm
    directory: /
    schedule:
      interval: daily
    open-pull-requests-limit: 10
    labels: ["security", "dependencies"]

# npm audit in CI
- name: Security audit
  run: |
    npm audit --audit-level=high
    # Fails if high/critical vulnerabilities found
```

### Trivy (Container Scanning)

```yaml
# Scan container image for vulnerabilities
- name: Trivy vulnerability scan
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:${{ github.sha }}'
    format: 'sarif'
    output: 'trivy-results.sarif'
    severity: 'CRITICAL,HIGH'
    exit-code: '1'               # Fail on critical/high

- name: Upload Trivy results
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: 'trivy-results.sarif'
```

---

## Secrets Detection

```yaml
# GitHub Actions — Gitleaks
- name: Gitleaks
  uses: gitleaks/gitleaks-action@v2
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

# Pre-commit hook (local)
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

```
  SECRETS DETECTION
  ═════════════════

  ✕ CAUGHT:
    AWS_SECRET_KEY = "AKIAIOSFODNN7EXAMPLE"     ← Hardcoded!
    password = "admin123"                        ← Credential!
    api_key: "sk-proj-abc123..."                 ← API key!

  ✓ USE INSTEAD:
    AWS_SECRET_KEY = os.environ["AWS_SECRET_KEY"]
    password = vault.get("db/password")
    api_key = secrets.get("openai_key")
```

---

## Complete Security Pipeline

```
  ┌─────────────────────────────────────────────────────┐
  │           SECURITY-ENABLED CI/CD PIPELINE           │
  │                                                     │
  │  Pre-commit ──► Build ──► Test ──► Deploy──►Monitor │
  │      │            │         │        │         │    │
  │  ┌───▼───┐   ┌───▼───┐ ┌──▼──┐ ┌──▼───┐ ┌───▼──┐ │
  │  │Secrets│   │ SAST  │ │ SCA │ │DAST  │ │RASP  │ │
  │  │Scan   │   │CodeQL │ │Audit│ │ZAP   │ │WAF   │ │
  │  │Gitleaks   │Semgrep│ │Trivy│ │Nuclei│ │Alerts│ │
  │  └───────┘   └───────┘ └─────┘ └──────┘ └──────┘ │
  │                                                     │
  │  Shift-Left: Find issues as early as possible       │
  └─────────────────────────────────────────────────────┘
```

---

## Troubleshooting

| Problem | Cause | Solution |
|---|---|---|
| Too many false positives (SAST) | Overly broad rules | Tune rules; use `// nosec` annotations sparingly with justification |
| DAST scan takes too long | Full spider + active scan | Use baseline scan for PRs; full scan nightly |
| npm audit blocks legitimate packages | Known vuln with no fix yet | Use `npm audit --omit=dev`; create `.npmrc` overrides |
| Secrets scanner flags test data | Example keys in test files | Add `.gitleaksignore` with specific allowances |
| Container scan finds OS vulns | Base image outdated | Use `distroless` / `alpine`; rebuild regularly |
| CodeQL analysis timeout | Large codebase | Split by language; increase timeout; exclude test files |

---

## Summary Table

| Aspect | Detail |
|---|---|
| **SAST** | Scans source code at build time; finds code-level vulns |
| **DAST** | Scans running app; finds runtime & config vulns |
| **SCA** | Scans dependencies for known CVEs |
| **Secrets** | Detects hardcoded credentials in code |
| **Container** | Scans Docker images for OS/library vulns |
| **Principle** | Shift-left — find issues earliest possible |

---

## Quick Revision Questions

1. **What is SAST?** *Static Application Security Testing — analyzes source code without running it to find vulnerabilities like SQL injection and XSS.*
2. **What is DAST?** *Dynamic Application Security Testing — tests a running application by sending malicious requests to find runtime vulnerabilities.*
3. **What is SCA?** *Software Composition Analysis — scans project dependencies against known vulnerability databases (CVEs).*
4. **What does "shift-left security" mean?** *Moving security testing earlier in the development lifecycle — into CI/CD rather than post-deployment manual audits.*
5. **Name two SAST tools and two DAST tools.** *SAST: CodeQL, Semgrep. DAST: OWASP ZAP, Nuclei.*
6. **Why is secrets detection important in CI?** *To prevent accidental exposure of API keys, passwords, and tokens in source code repositories.*

---

[← Previous: Performance Testing](05-performance-testing.md) | [Back to README](../README.md) | [Next: Test Automation Best Practices →](07-test-automation-best-practices.md)
