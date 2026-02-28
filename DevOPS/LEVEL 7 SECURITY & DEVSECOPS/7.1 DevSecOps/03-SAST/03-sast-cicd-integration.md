# Chapter 3.3: SAST Integration in CI/CD

## Overview

Integrating SAST into CI/CD pipelines enables automated, continuous security scanning on every code change. This chapter covers practical integration patterns, quality gate configuration, and optimization strategies.

---

## Integration Architecture

```
+====================================================================+
|              SAST CI/CD INTEGRATION PATTERNS                        |
+====================================================================+
|                                                                      |
|  PATTERN 1: INLINE (Blocking)                                       |
|  +--------------------------------------------------------------+  |
|  | Code → SAST → Quality Gate → Build → Deploy                  |  |
|  | Pipeline BLOCKS on critical findings                         |  |
|  +--------------------------------------------------------------+  |
|                                                                      |
|  PATTERN 2: PARALLEL (Non-blocking initial)                         |
|  +--------------------------------------------------------------+  |
|  | Code → Build ──────────────────────→ Deploy                  |  |
|  |     └→ SAST → Quality Gate → Report (async findings)        |  |
|  +--------------------------------------------------------------+  |
|                                                                      |
|  PATTERN 3: TIERED                                                  |
|  +--------------------------------------------------------------+  |
|  | PR:     Fast scan (Semgrep) → Block on critical              |  |
|  | Merge:  Full scan (SonarQube) → Quality gate                 |  |
|  | Nightly: Deep scan (Checkmarx) → Report + track              |  |
|  +--------------------------------------------------------------+  |
+====================================================================+
```

### Complete GitHub Actions Pipeline

```yaml
name: SAST Pipeline
on:
  pull_request:
    branches: [main, develop]
  push:
    branches: [main]

jobs:
  # Fast scan on every PR (< 1 min)
  semgrep-fast:
    name: Semgrep Quick Scan
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    steps:
      - uses: actions/checkout@v4
      - name: Semgrep Scan
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/owasp-top-ten
        env:
          SEMGREP_APP_TOKEN: ${{ secrets.SEMGREP_APP_TOKEN }}

  # Full scan on merge to main (5-10 min)
  sonarqube-full:
    name: SonarQube Full Analysis
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - name: SonarQube Scan
        uses: sonarqube/sonarqube-scan-action@v2
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      
      - name: Quality Gate Check
        uses: sonarqube/sonarqube-quality-gate-action@v1
        timeout-minutes: 5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
```

### Jenkins Pipeline

```groovy
// Jenkinsfile
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('SAST - Semgrep') {
            steps {
                sh '''
                    pip install semgrep
                    semgrep --config p/security-audit \
                            --config p/owasp-top-ten \
                            --sarif --output semgrep.sarif \
                            --error \
                            --severity ERROR \
                            .
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'semgrep.sarif'
                }
            }
        }
        
        stage('SAST - SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=${JOB_NAME} \
                          -Dsonar.sources=src \
                          -Dsonar.tests=tests
                    '''
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    
    post {
        failure {
            slackSend channel: '#security-alerts',
                      color: 'danger',
                      message: "SAST scan failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
    }
}
```

### GitLab CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - sast
  - quality-gate
  - build
  - deploy

sast:
  stage: sast
  image: returntocorp/semgrep
  script:
    - semgrep --config auto --sarif --output gl-sast-report.sarif .
  artifacts:
    reports:
      sast: gl-sast-report.sarif
    paths:
      - gl-sast-report.sarif

sonarqube:
  stage: sast
  image: sonarsource/sonar-scanner-cli
  variables:
    SONAR_HOST_URL: ${SONAR_HOST_URL}
    SONAR_TOKEN: ${SONAR_TOKEN}
  script:
    - sonar-scanner
        -Dsonar.projectKey=${CI_PROJECT_NAME}
        -Dsonar.sources=src
        -Dsonar.qualitygate.wait=true
  allow_failure: false
```

---

## Quality Gate Configuration

```
RECOMMENDED QUALITY GATE SETTINGS:

  +================================================================+
  |  SECURITY QUALITY GATE                                          |
  +================================================================+
  |                                                                  |
  |  BLOCKING CONDITIONS (Pipeline fails):                          |
  |  +----------------------------------------------------------+  |
  |  | New Critical Vulnerabilities > 0        → FAIL           |  |
  |  | New High Vulnerabilities > 0            → FAIL           |  |
  |  | Security Rating worse than B            → FAIL           |  |
  |  | New Security Hotspots Unreviewd > 0     → FAIL           |  |
  |  +----------------------------------------------------------+  |
  |                                                                  |
  |  WARNING CONDITIONS (Pipeline continues):                       |
  |  +----------------------------------------------------------+  |
  |  | New Medium Vulnerabilities > 5          → WARN           |  |
  |  | Security Rating worse than A            → WARN           |  |
  |  | Code Coverage < 80%                     → WARN           |  |
  |  +----------------------------------------------------------+  |
  |                                                                  |
  |  INFORMATIONAL (Tracked only):                                  |
  |  +----------------------------------------------------------+  |
  |  | New Low Vulnerabilities                  → INFO          |  |
  |  | Code Smells                              → INFO          |  |
  |  | Technical Debt                            → INFO         |  |
  |  +----------------------------------------------------------+  |
  +================================================================+
```

---

## Optimizing SAST Scan Performance

```
OPTIMIZATION STRATEGIES:

  Strategy 1: INCREMENTAL SCANNING
  +------------------------------------------------------------+
  | Only scan files changed in the PR/commit                    |
  | Speed improvement: 10-100x                                  |
  |                                                              |
  | # Semgrep - scan only changed files                         |
  | git diff --name-only HEAD~1 | xargs semgrep --config auto  |
  +------------------------------------------------------------+

  Strategy 2: CACHING
  +------------------------------------------------------------+
  | Cache scan results between builds                           |
  | Speed improvement: 2-5x                                     |
  |                                                              |
  | # GitHub Actions cache                                      |
  | - uses: actions/cache@v3                                    |
  |   with:                                                     |
  |     path: ~/.sonar/cache                                    |
  |     key: sonar-${{ hashFiles('pom.xml') }}                  |
  +------------------------------------------------------------+

  Strategy 3: EXCLUDE NON-ESSENTIAL FILES
  +------------------------------------------------------------+
  | Skip test files, generated code, vendor directories         |
  | Speed improvement: 2-10x                                    |
  |                                                              |
  | # sonar-project.properties                                  |
  | sonar.exclusions=**/test/**,**/vendor/**,**/node_modules/** |
  +------------------------------------------------------------+

  Strategy 4: TIERED SCANNING
  +------------------------------------------------------------+
  | Fast scan on PR, full scan on merge, deep scan nightly      |
  |                                                              |
  | PR:     Semgrep (5 sec)   → Immediate feedback              |
  | Merge:  SonarQube (3 min) → Quality gate                    |
  | Night:  Checkmarx (30 min)→ Comprehensive report            |
  +------------------------------------------------------------+
```

---

## PR Decoration (Inline Comments)

```
SAST findings posted directly on Pull Requests:

  +------------------------------------------------------------+
  |  PR #142: Add user search endpoint                          |
  |  ========================================================== |
  |                                                              |
  |  src/api/users.py                                           |
  |  Line 42-44:                                                 |
  |  ┌──────────────────────────────────────────────────────┐   |
  |  │  query = "SELECT * FROM users WHERE name = '"       │   |
  |  │           + search_term + "'"                        │   |
  |  │  results = db.execute(query)                         │   |
  |  └──────────────────────────────────────────────────────┘   |
  |                                                              |
  |  🔴 CRITICAL: SQL Injection (CWE-89)                        |
  |  Untrusted input 'search_term' concatenated into SQL query. |
  |  Use parameterized queries:                                  |
  |     query = "SELECT * FROM users WHERE name = %s"           |
  |     results = db.execute(query, (search_term,))             |
  |                                                              |
  |  [View Rule] [Suppress Finding] [Mark as Won't Fix]         |
  +------------------------------------------------------------+
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| SAST blocking all PRs | Too strict quality gates initially | Start with warnings, gradually tighten to blocking |
| SonarQube quality gate timeout | Server overloaded | Scale SonarQube (more RAM/CPU), use webhooks not polling |
| Scan results lost between runs | No artifact archiving | Store SARIF reports as pipeline artifacts |
| PR comments too noisy | Reporting all severities | Filter to HIGH/CRITICAL for PR comments; full report in dashboard |
| Branch analysis not working | Missing fetch-depth | Use `fetch-depth: 0` in checkout for proper branch comparison |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Inline Integration** | SAST runs as blocking step in pipeline |
| **Parallel Integration** | SAST runs alongside build (non-blocking) |
| **Tiered Scanning** | Different scan depths at different stages |
| **Quality Gate** | Conditions that must be met to proceed |
| **Incremental Scan** | Only scanning changed files |
| **PR Decoration** | SAST findings as inline PR comments |
| **SARIF Format** | Standard format for security scan results |

---

## Quick Revision Questions

1. **What are the three SAST pipeline integration patterns?**
   > Inline (blocking — SAST must pass before build), Parallel (non-blocking — runs alongside build), Tiered (different depths at PR, merge, and nightly stages).

2. **How do you prevent SAST from slowing down the development process?**
   > Incremental scanning (only changed files), caching, excluding test/vendor files, tiered scanning (fast on PR, deep on nightly), and running scans in parallel with other pipeline steps.

3. **What is SARIF and why is it used?**
   > Static Analysis Results Interchange Format — a standard JSON format for security scan results. It enables tool interoperability, allowing results from different scanners to be consumed by the same dashboards and CI/CD integrations.

4. **What should a security quality gate block on?**
   > Any new CRITICAL or HIGH severity vulnerabilities, security rating worse than B, and unreviewed security hotspots. Medium/Low should warn but not block.

5. **What is PR decoration and why is it effective?**
   > SAST findings posted as inline comments directly on the Pull Request, showing the vulnerable code with fix suggestions. It's effective because developers see findings in their normal workflow without switching tools.

---

[← Previous: 3.2 SAST Tools](02-sast-tools.md) | [Next: 3.4 False Positive Management →](04-false-positive-management.md)

[Back to Table of Contents](../README.md)
