# Chapter 1.3: Security in the CI/CD Pipeline

## Overview

A CI/CD pipeline automates the process of building, testing, and deploying software. In DevSecOps, security checks are embedded at every stage of this pipeline, creating security gates that prevent vulnerable code from reaching production.

---

## Pipeline Security Architecture

```
+========================================================================+
|                    SECURE CI/CD PIPELINE                                 |
+========================================================================+
|                                                                          |
|  DEVELOPER        SOURCE CONTROL       BUILD           TEST              |
|  +---------+      +------------+      +---------+     +----------+      |
|  |  Code   |      | Git Repo   |      | Compile |     | Unit     |      |
|  |  +       |----->| + Branch   |----->| Package |---->| Tests    |      |
|  | IDE Scan |      | Protection |      | + SAST  |     | + DAST   |      |
|  +---------+      +------------+      | + SCA   |     | + IAST   |      |
|       |                |              | + Secrets|     | + Fuzz   |      |
|       v                v              +---------+     +----------+      |
|  Pre-commit        Webhook               |                |              |
|  Hooks             Trigger                v                v              |
|                                     +----------+    +----------+        |
|                                     | Security |    | Security |        |
|                                     | Gate #1  |    | Gate #2  |        |
|                                     +----+-----+    +----+-----+        |
|                                          |               |              |
|  STAGING              PRODUCTION         |               |              |
|  +----------+        +----------+        |               |              |
|  | Deploy   |        | Deploy   |   PASS = Continue  PASS = Continue    |
|  | + Pen    |------->| + Monitor|   FAIL = Block     FAIL = Block       |
|  |   Test   |        | + WAF    |                                       |
|  | + Config |        | + SIEM   |                                       |
|  |   Audit  |        +----------+                                       |
|  +----------+             |                                              |
|       |                   v                                              |
|       v              +-----------+                                       |
|  +----------+        |  Runtime  |                                       |
|  | Security |        | Security  |                                       |
|  | Gate #3  |        | Monitoring|                                       |
|  +----------+        +-----------+                                       |
+========================================================================+
```

---

## Security Gates

Security gates are checkpoints in the pipeline that enforce minimum security standards. If a gate fails, the pipeline is blocked.

```
                    SECURITY GATE DECISION FLOW
                    
                    +-------------------+
                    |  Security Scan    |
                    |  Results          |
                    +---------+---------+
                              |
                    +---------v---------+
                    |  Severity Check   |
                    +---------+---------+
                              |
              +---------------+---------------+
              |                               |
     +--------v--------+            +--------v--------+
     |  CRITICAL/HIGH  |            |  MEDIUM/LOW     |
     |  found?         |            |  only?          |
     +--------+--------+            +--------+--------+
              |                               |
         +----v----+                    +-----v-----+
         |  BLOCK  |                    |   WARN    |
         | Pipeline|                    |  Continue |
         |  FAIL   |                    |  + Track  |
         +---------+                    +-----------+
```

### Configuring Quality Gates

```yaml
# Example: SonarQube Quality Gate Configuration
quality_gate:
  conditions:
    # Block on any new critical/high security issues
    - metric: new_security_hotspots_reviewed
      operator: LESS_THAN
      value: 100
    
    - metric: new_vulnerabilities
      operator: GREATER_THAN
      value: 0
      severity: CRITICAL
    
    - metric: new_vulnerabilities
      operator: GREATER_THAN
      value: 0  
      severity: HIGH
    
    # Allow medium/low but track
    - metric: security_rating
      operator: GREATER_THAN
      value: 2  # A or B rating required
```

---

## Complete Pipeline Implementation

### GitHub Actions Example

```yaml
# .github/workflows/devsecops-pipeline.yml
name: DevSecOps Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  IMAGE_NAME: myapp
  IMAGE_TAG: ${{ github.sha }}

jobs:
  # ============================================
  # STAGE 1: SECRET SCANNING
  # ============================================
  secret-scan:
    name: Secret Detection
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for scanning
      
      - name: Gitleaks Secret Scan
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
      - name: TruffleHog Secret Scan
        uses: trufflesecurity/trufflehog@main
        with:
          extra_args: --only-verified

  # ============================================
  # STAGE 2: STATIC ANALYSIS (SAST)
  # ============================================
  sast:
    name: SAST Scan
    runs-on: ubuntu-latest
    needs: secret-scan
    steps:
      - uses: actions/checkout@v4
      
      - name: Semgrep SAST
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/owasp-top-ten
            p/cwe-top-25
      
      - name: SonarQube Analysis
        uses: sonarqube/sonarqube-scan-action@v2
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
      
      - name: Check Quality Gate
        uses: sonarqube/sonarqube-quality-gate-action@v1
        timeout-minutes: 5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  # ============================================
  # STAGE 3: DEPENDENCY SCANNING (SCA)
  # ============================================
  sca:
    name: SCA Scan
    runs-on: ubuntu-latest
    needs: secret-scan
    steps:
      - uses: actions/checkout@v4
      
      - name: Snyk Dependency Check
        uses: snyk/actions/node@master
        continue-on-error: false
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
      
      - name: OWASP Dependency Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: 'MyApp'
          path: '.'
          format: 'HTML'
          args: --failOnCVSS 7

  # ============================================
  # STAGE 4: BUILD & CONTAINER SCAN
  # ============================================
  build-scan:
    name: Build & Container Scan
    runs-on: ubuntu-latest
    needs: [sast, sca]
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker Image
        run: docker build -t $IMAGE_NAME:$IMAGE_TAG .
      
      - name: Trivy Container Scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: '${{ env.IMAGE_NAME }}:${{ env.IMAGE_TAG }}'
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'
      
      - name: Hadolint Dockerfile Lint
        uses: hadolint/hadolint-action@v3
        with:
          dockerfile: Dockerfile
      
      - name: Push to Registry
        if: success()
        run: |
          docker tag $IMAGE_NAME:$IMAGE_TAG registry.example.com/$IMAGE_NAME:$IMAGE_TAG
          docker push registry.example.com/$IMAGE_NAME:$IMAGE_TAG

  # ============================================
  # STAGE 5: IaC SECURITY SCAN  
  # ============================================
  iac-scan:
    name: IaC Security Scan
    runs-on: ubuntu-latest
    needs: [sast, sca]
    steps:
      - uses: actions/checkout@v4
      
      - name: Checkov IaC Scan
        uses: bridgecrewio/checkov-action@master
        with:
          directory: terraform/
          framework: terraform
          soft_fail: false
      
      - name: tfsec Scan
        uses: aquasecurity/tfsec-action@v1
        with:
          soft_fail: false

  # ============================================
  # STAGE 6: DEPLOY TO STAGING
  # ============================================
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: [build-scan, iac-scan]
    environment: staging
    steps:
      - name: Deploy to Staging
        run: |
          kubectl set image deployment/myapp \
            myapp=registry.example.com/$IMAGE_NAME:$IMAGE_TAG \
            --namespace=staging

  # ============================================
  # STAGE 7: DAST SCAN
  # ============================================
  dast:
    name: DAST Scan
    runs-on: ubuntu-latest
    needs: deploy-staging
    steps:
      - name: OWASP ZAP Full Scan
        uses: zaproxy/action-full-scan@v0.9
        with:
          target: 'https://staging.example.com'
          rules_file_name: '.zap/rules.tsv'
          cmd_options: '-a'

  # ============================================
  # STAGE 8: DEPLOY TO PRODUCTION
  # ============================================
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: dast
    environment: production
    steps:
      - name: Deploy to Production
        run: |
          kubectl set image deployment/myapp \
            myapp=registry.example.com/$IMAGE_NAME:$IMAGE_TAG \
            --namespace=production
```

---

## Pipeline Security Flow Diagram

```
Developer Push
      |
      v
+-----+------+     +----------+     +---------+
| Secret Scan |---->|   SAST   |---->|   SCA   |
| (Gitleaks)  |     |(Semgrep) |     | (Snyk)  |
+-----+------+     +----+-----+     +----+----+
      |                  |               |
      |  GATE: No        |  GATE: No     |  GATE: No
      |  secrets found   |  critical     |  high/crit CVEs
      |                  |  findings     |
      +--------+---------+------+--------+
               |                |
               v                v
        +------+------+  +-----+-------+
        | Build &     |  | IaC Scan    |
        | Container   |  | (Checkov)   |
        | Scan(Trivy) |  |             |
        +------+------+  +-----+-------+
               |                |
               |  GATE: No      |  GATE: All
               |  critical      |  checks pass
               |  image vulns   |
               +--------+-------+
                        |
                        v
               +--------+--------+
               |  Deploy to      |
               |  STAGING        |
               +--------+--------+
                        |
                        v
               +--------+--------+
               |  DAST Scan      |
               |  (OWASP ZAP)    |
               +--------+--------+
                        |
                        |  GATE: No critical
                        |  runtime vulns
                        v
               +--------+--------+
               |  Deploy to      |
               |  PRODUCTION     |
               +-----------------+
```

---

## GitLab CI/CD Security Pipeline

```yaml
# .gitlab-ci.yml
stages:
  - secret-detection
  - sast
  - sca
  - build
  - container-scan
  - deploy-staging
  - dast
  - deploy-production

# Secret Detection
secret_detection:
  stage: secret-detection
  image: registry.gitlab.com/gitlab-org/security-products/analyzers/secrets
  script:
    - /analyzer run
  artifacts:
    reports:
      secret_detection: gl-secret-detection-report.json

# SAST
sast:
  stage: sast
  image: registry.gitlab.com/gitlab-org/security-products/analyzers/semgrep
  script:
    - /analyzer run
  artifacts:
    reports:
      sast: gl-sast-report.json

# Dependency Scanning
dependency_scanning:
  stage: sca
  image: registry.gitlab.com/gitlab-org/security-products/analyzers/gemnasium
  script:
    - /analyzer run
  artifacts:
    reports:
      dependency_scanning: gl-dependency-scanning-report.json

# Container Scanning
container_scanning:
  stage: container-scan
  image: registry.gitlab.com/gitlab-org/security-products/analyzers/container-scanning
  variables:
    CS_IMAGE: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  script:
    - gtcs scan
  artifacts:
    reports:
      container_scanning: gl-container-scanning-report.json

# DAST
dast:
  stage: dast
  image: registry.gitlab.com/gitlab-org/security-products/analyzers/dast
  variables:
    DAST_WEBSITE: https://staging.example.com
  script:
    - /analyze
  artifacts:
    reports:
      dast: gl-dast-report.json
```

---

## Pipeline Security Best Practices

### 1. Protect the Pipeline Itself

```
PIPELINE AS ATTACK SURFACE:

  Attacker Goals:
  +--------------------------------------------------+
  | 1. Inject malicious code into build               |
  | 2. Steal secrets from pipeline variables          |
  | 3. Modify pipeline to skip security checks        |
  | 4. Compromise build artifacts                     |
  | 5. Pivot to production infrastructure             |
  +--------------------------------------------------+

  Protections:
  +--------------------------------------------------+
  | - Branch protection rules (require reviews)       |
  | - Pipeline file in protected branch               |
  | - Signed commits                                  |
  | - Short-lived credentials only                   |
  | - Audit trail for pipeline changes               |
  | - Runner isolation (ephemeral runners)            |
  | - Limit who can modify CI/CD configuration       |
  +--------------------------------------------------+
```

### 2. Branch Protection Rules

```yaml
# GitHub Branch Protection Settings
branch_protection:
  branch: main
  rules:
    require_pull_request:
      required_approvals: 2
      dismiss_stale_reviews: true
      require_code_owner_reviews: true
    
    require_status_checks:
      strict: true
      checks:
        - "secret-scan"
        - "sast"
        - "sca"
        - "container-scan"
        - "iac-scan"
    
    require_signed_commits: true
    enforce_admins: true
    restrict_pushes: true
```

### 3. Pipeline as Code Security

```
DO:                                  DON'T:
+-------------------------------+    +-------------------------------+
| Store pipeline in version     |    | Use inline scripts with      |
| control                       |    | hardcoded secrets             |
|                               |    |                               |
| Use branch protection for     |    | Allow anyone to modify       |
| pipeline config files         |    | pipeline configuration       |
|                               |    |                               |
| Pin tool versions             |    | Use @latest or @master tags  |
| (sha256 hashes)              |    |                               |
|                               |    |                               |
| Use ephemeral runners         |    | Use persistent shared        |
|                               |    | runners with state           |
|                               |    |                               |
| Scan pipeline config for      |    | Trust all third-party        |
| security issues               |    | actions blindly              |
+-------------------------------+    +-------------------------------+
```

---

## Real-World Scenario: SolarWinds Attack

The SolarWinds supply chain attack (2020) exploited the CI/CD pipeline itself:

```
ATTACK FLOW:
+------------------+     +------------------+     +------------------+
| 1. Attacker      |---->| 2. Modified      |---->| 3. Malicious     |
|    compromised   |     |    build process  |     |    code injected |
|    build system   |     |    to inject     |     |    into signed   |
|                  |     |    SUNBURST       |     |    updates       |
+------------------+     +------------------+     +------------------+
                                                          |
                                                          v
+------------------+     +------------------+     +------------------+
| 6. Data          |<----| 5. 18,000+       |<----| 4. Legitimate    |
|    exfiltrated   |     |    organizations  |     |    auto-update   |
|    from victims  |     |    installed      |     |    distributed   |
|                  |     |    backdoor       |     |    malware       |
+------------------+     +------------------+     +------------------+

PREVENTION:
+------------------------------------------------------------+
| - Build environment integrity verification                  |
| - Reproducible builds                                       |
| - SLSA (Supply-chain Levels for Software Artifacts)        |
| - Code signing verification at every stage                 |
| - Build provenance attestation                             |
| - Isolated build environments (ephemeral)                  |
+------------------------------------------------------------+
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Pipeline too slow (>30 min) | Sequential security scans | Parallelize independent scans (SAST + SCA) |
| False positives blocking merges | Overly strict quality gates | Use severity thresholds, maintain suppression files |
| Secrets exposed in logs | Poor secret masking | Use secret management, mask variables in CI settings |
| Third-party actions compromised | Unpinned action versions | Pin actions to specific SHA hashes, not tags |
| DAST scan timeout | Target app not ready | Add health check wait before DAST stage |
| Inconsistent scan results | Different tool versions | Pin all tool versions, use lock files |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Security Gates** | Checkpoints that block pipeline on security failures |
| **SAST in CI** | Static analysis during build phase |
| **SCA in CI** | Dependency vulnerability scanning during build |
| **DAST in CI** | Dynamic scanning against running staging environment |
| **Container Scanning** | Image vulnerability detection before deployment |
| **IaC Scanning** | Infrastructure code security before provisioning |
| **Pipeline Protection** | Securing the CI/CD infrastructure itself |
| **Branch Protection** | Require reviews and passing security checks |
| **Supply Chain Security** | SLSA, signed builds, provenance attestation |
| **Parallel Scanning** | Run independent scans simultaneously for speed |

---

## Quick Revision Questions

1. **What is a security gate in a CI/CD pipeline?**
   > A checkpoint that evaluates security scan results and blocks the pipeline if findings exceed a defined threshold (e.g., any CRITICAL vulnerabilities), preventing vulnerable code from progressing to the next stage.

2. **Why should SAST and SCA be run in parallel rather than sequentially?**
   > They analyze different things (source code vs. dependencies) and have no dependencies on each other. Running them in parallel reduces total pipeline time without sacrificing security coverage.

3. **How should third-party CI/CD actions be secured?**
   > Pin actions to specific commit SHA hashes instead of version tags (which can be moved). Regularly audit and update pinned versions. Only use actions from trusted, verified publishers.

4. **What is SLSA and why is it important for pipeline security?**
   > Supply-chain Levels for Software Artifacts (SLSA) is a framework for ensuring the integrity of software artifacts throughout the supply chain. It defines levels of build provenance and verification to prevent attacks like SolarWinds.

5. **Name three ways to protect the CI/CD pipeline itself from attacks.**
   > (1) Branch protection rules requiring reviews, (2) Ephemeral/isolated build runners, (3) Signed commits and build provenance attestation, (4) Pipeline configuration in protected branches.

6. **When should DAST scanning happen in the pipeline and why?**
   > After deployment to a staging environment, because DAST requires a running application to test against. It sends actual HTTP requests and cannot be run against source code alone.

---

[← Previous: 1.2 Shift-Left Security](02-shift-left-security.md) | [Next: 1.4 Security Culture →](04-security-culture.md)

[Back to Table of Contents](../README.md)
