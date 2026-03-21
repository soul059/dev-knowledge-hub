# Unit 8: A08 - Software and Data Integrity Failures — Topic 1: CI/CD Security

## Overview

CI/CD pipeline security is critical because the pipeline has **direct access to production environments**, source code, secrets, and deployment credentials. A compromised CI/CD pipeline allows attackers to inject malicious code that gets automatically deployed to production. This category was new in the OWASP Top 10 2021, reflecting the growing threat of supply chain attacks.

---

## 1. CI/CD Attack Surface

```
┌─────────────────────────────────────────────────────────────────┐
│                CI/CD PIPELINE ATTACK SURFACE                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  [Source Code] → [Build] → [Test] → [Deploy] → [Production]   │
│       ↑            ↑         ↑         ↑            ↑          │
│    Attack 1     Attack 2  Attack 3  Attack 4     Attack 5      │
│                                                                 │
│  1. SOURCE CODE:                                               │
│     ├── Compromised developer account                          │
│     ├── Malicious pull request                                 │
│     └── Dependency confusion/typosquatting                     │
│                                                                 │
│  2. BUILD:                                                     │
│     ├── Compromised build server                               │
│     ├── Malicious build scripts                                │
│     └── Injected build dependencies                            │
│                                                                 │
│  3. TEST:                                                      │
│     ├── Skipped security tests                                 │
│     ├── Test environment with production data                  │
│     └── Compromised test frameworks                            │
│                                                                 │
│  4. DEPLOY:                                                    │
│     ├── Stolen deployment credentials                          │
│     ├── Unauthorized deployment                                │
│     └── Configuration manipulation                             │
│                                                                 │
│  5. PRODUCTION:                                                │
│     ├── Backdoored artifacts                                   │
│     ├── Modified containers                                    │
│     └── Tampered configuration                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Notable CI/CD Attacks

| Attack | Year | Method | Impact |
|--------|------|--------|--------|
| **SolarWinds** | 2020 | Malicious code injected in build | 18,000+ orgs compromised |
| **Codecov** | 2021 | Bash uploader script modified | Credentials exfiltrated |
| **ua-parser-js** | 2021 | npm account hijacked | Cryptominer in package |
| **PHP Git** | 2021 | Git server compromised | Backdoor in PHP source |
| **CircleCI** | 2023 | Internal breach | Customer secrets exposed |

---

## 3. CI/CD Hardening

### Secret Management

```yaml
# ❌ INSECURE — Secrets in code/config
env:
  DB_PASSWORD: "SuperSecret123"
  API_KEY: "sk_live_abc123"

# ❌ INSECURE — Secrets in environment variables (visible in logs)
steps:
  - run: echo $DB_PASSWORD  # Accidentally logged

# ✅ SECURE — Use secret management
# GitHub Actions
steps:
  - name: Deploy
    env:
      DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
    run: deploy.sh
    # Secrets are masked in logs automatically

# Use dedicated secret managers:
# - HashiCorp Vault
# - AWS Secrets Manager
# - Azure Key Vault
# - Google Secret Manager
```

### Pipeline Security Controls

```yaml
# GitHub Actions — Secure pipeline
name: Secure Build and Deploy
on:
  push:
    branches: [main]

permissions:
  contents: read         # Minimum permissions
  packages: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # Pin actions to SHA (not tags)
      - uses: actions/setup-node@8f152de45cc393bb48ce5d89d36b731f54556e65  # v4.0.0

      - name: Install dependencies
        run: npm ci          # Uses lockfile (not npm install)

      - name: Security scan
        run: npm audit --audit-level=high

      - name: Run tests
        run: npm test

      - name: Build
        run: npm run build

      # Sign artifacts
      - name: Sign artifact
        run: cosign sign --key env://COSIGN_KEY artifact.tar.gz
        env:
          COSIGN_KEY: ${{ secrets.COSIGN_PRIVATE_KEY }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production    # Requires approval
    steps:
      - name: Deploy
        run: ./deploy.sh
```

---

## 4. Pipeline Security Checklist

```
SOURCE CONTROL:
□ Branch protection on main/production branches
□ Require pull request reviews (minimum 2 reviewers)
□ Require signed commits (GPG)
□ No force push allowed on protected branches
□ CODEOWNERS file for sensitive paths

BUILD:
□ Pin all dependencies to exact versions (lock files)
□ Pin CI actions/plugins to SHA hashes (not tags)
□ Use isolated build environments (containers)
□ Scan dependencies for vulnerabilities (SCA)
□ Static analysis (SAST) in pipeline

SECRETS:
□ No secrets in source code or config files
□ Use secret management service
□ Rotate secrets regularly
□ Audit secret access
□ Secrets masked in CI logs

DEPLOY:
□ Require approval for production deployments
□ Separate deployment credentials per environment
□ Sign deployment artifacts
□ Verify artifact signatures before deployment
□ Immutable infrastructure (no manual changes)

MONITORING:
□ Audit logs for all pipeline actions
□ Alert on unusual pipeline behavior
□ Monitor for unauthorized deployments
□ Track who deployed what and when
```

---

## Summary Table

| CI/CD Risk | Attack | Mitigation |
|-----------|--------|-----------|
| Compromised source | Malicious commits | Branch protection, code review, signed commits |
| Dependency tampering | Supply chain attack | Lock files, SCA scanning, pin SHAs |
| Secret exposure | Credential theft | Secret managers, rotation, masking |
| Unauthorized deploy | Backdoor deployment | Approval gates, environment protection |
| Build tampering | Modified artifacts | Artifact signing, reproducible builds |

---

## Revision Questions

1. Map the CI/CD pipeline attack surface — where can each phase be compromised?
2. How did the SolarWinds attack exploit the CI/CD pipeline? What controls would have prevented it?
3. Write a secure GitHub Actions workflow with pinned actions, SCA scanning, and deployment approval.
4. Why should CI action versions be pinned to SHA hashes instead of tags?
5. Design a secret management strategy for a CI/CD pipeline with development, staging, and production environments.
6. What is the principle of least privilege applied to CI/CD pipelines?

---

*Previous: None (First topic in this unit) | Next: [02-dependency-integrity.md](02-dependency-integrity.md)*

---

*[Back to README](../README.md)*
