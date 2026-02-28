# Chapter 5.2: SCA Tools

## Overview

Software Composition Analysis (SCA) tools scan your dependencies to identify known vulnerabilities, license issues, and outdated packages. This chapter covers the leading SCA tools — Snyk, Dependabot, OWASP Dependency-Check, and Trivy — with practical setup and CI/CD integration.

---

## Tool Comparison

```
SCA TOOL LANDSCAPE:

  ┌──────────────┬──────────────┬──────────────┬──────────────┬──────────┐
  │ Feature       │ Snyk         │ Dependabot   │ OWASP DC     │ Trivy    │
  ├──────────────┼──────────────┼──────────────┼──────────────┼──────────┤
  │ Cost          │ Free tier +  │ Free (GitHub)│ Free (OSS)   │ Free     │
  │               │ paid plans   │              │              │ (OSS)    │
  │ Ecosystems   │ 10+          │ 10+          │ Java/.NET    │ 10+      │
  │ Auto-fix PRs │ Yes          │ Yes          │ No           │ No       │
  │ License scan │ Yes          │ No           │ No           │ Yes      │
  │ Container    │ Yes          │ No           │ No           │ Yes      │
  │ SBOM         │ Yes          │ No           │ No           │ Yes      │
  │ CI/CD        │ Excellent    │ GitHub only  │ Good         │ Excellent│
  │ DB Source    │ Snyk DB      │ GitHub Adv.  │ NVD          │ Multiple │
  └──────────────┴──────────────┴──────────────┴──────────────┴──────────┘
```

---

## Snyk

### Setup and Usage

```bash
# Install Snyk CLI
npm install -g snyk

# Authenticate
snyk auth

# Test project for vulnerabilities
snyk test

# Test with severity filter
snyk test --severity-threshold=high

# Monitor project (continuous monitoring)
snyk monitor

# Test a specific manifest
snyk test --file=requirements.txt
snyk test --file=pom.xml
snyk test --file=go.mod

# Test container image
snyk container test nginx:latest

# Test IaC files
snyk iac test terraform/
```

### Snyk CI/CD Integration

```yaml
# GitHub Actions
name: SCA Scan
on: [push, pull_request]

jobs:
  snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Snyk Test
        uses: snyk/actions/node@master  # or /python, /maven, /gradle
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
      
      - name: Snyk Monitor
        if: github.ref == 'refs/heads/main'
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          command: monitor
```

### Snyk Policy File

```yaml
# .snyk
version: v1.25.0
ignore:
  SNYK-JS-LODASH-1018905:
    - '*':
        reason: 'Not exploitable in our usage - we never pass user input to lodash.template'
        expires: 2024-12-31T00:00:00.000Z
        created: 2024-06-15T00:00:00.000Z
  
  SNYK-PYTHON-REQUESTS-5595532:
    - '*':
        reason: 'Fixed in next sprint - tracked in JIRA-456'
        expires: 2024-07-15T00:00:00.000Z

patch: {}
```

---

## GitHub Dependabot

### Configuration

```yaml
# .github/dependabot.yml
version: 2
updates:
  # npm dependencies
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
      timezone: "Europe/London"
    open-pull-requests-limit: 10
    reviewers:
      - "security-team"
    labels:
      - "dependencies"
      - "security"
    # Group minor/patch updates together
    groups:
      production-dependencies:
        patterns:
          - "*"
        exclude-patterns:
          - "@types/*"
        update-types:
          - "minor"
          - "patch"
    # Ignore specific packages
    ignore:
      - dependency-name: "webpack"
        versions: ["5.x"]
        update-types: ["version-update:semver-major"]

  # Python dependencies
  - package-ecosystem: "pip"
    directory: "/"
    schedule:
      interval: "daily"
    open-pull-requests-limit: 5

  # Docker images
  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"

  # GitHub Actions
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

### Dependabot Security Alerts Auto-merge

```yaml
# .github/workflows/dependabot-automerge.yml
name: Dependabot Auto-merge
on: pull_request

permissions:
  contents: write
  pull-requests: write

jobs:
  automerge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - name: Dependabot metadata
        id: metadata
        uses: dependabot/fetch-metadata@v2
        with:
          github-token: "${{ secrets.GITHUB_TOKEN }}"
      
      - name: Auto-merge patch updates
        if: steps.metadata.outputs.update-type == 'version-update:semver-patch'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## Trivy (Aqua Security)

```bash
# Install Trivy
# macOS
brew install trivy

# Linux
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh

# Windows
choco install trivy

# Scan project dependencies
trivy fs .
trivy fs --severity HIGH,CRITICAL .
trivy fs --format json --output results.json .

# Scan specific lockfile
trivy fs --scanners vuln package-lock.json

# Scan container image
trivy image nginx:latest
trivy image --severity CRITICAL myapp:latest

# Generate SBOM
trivy fs --format cyclonedx --output sbom.json .
trivy image --format spdx-json --output sbom.json myapp:latest

# Scan SBOM for vulnerabilities
trivy sbom sbom.json
```

### Trivy CI/CD Integration

```yaml
# GitHub Actions with Trivy
name: SCA with Trivy
on: [push, pull_request]

jobs:
  trivy-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Trivy Vulnerability Scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'
          format: 'sarif'
          output: 'trivy-results.sarif'
          exit-code: '1'  # Fail on findings
      
      - name: Upload to GitHub Security
        if: always()
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'
```

---

## OWASP Dependency-Check

```bash
# Install
# Download from: https://owasp.org/www-project-dependency-check/

# Run scan
dependency-check.sh \
    --project "MyApp" \
    --scan ./src \
    --format HTML \
    --format JSON \
    --out ./reports \
    --failOnCVSS 7  # Fail if any CVE >= 7.0

# Maven plugin
mvn org.owasp:dependency-check-maven:check \
    -DfailBuildOnCVSS=7
```

```xml
<!-- pom.xml - Maven integration -->
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>9.0.9</version>
    <configuration>
        <failBuildOnCVSS>7</failBuildOnCVSS>
        <formats>
            <format>HTML</format>
            <format>JSON</format>
        </formats>
        <suppressionFiles>
            <suppressionFile>dependency-check-suppression.xml</suppressionFile>
        </suppressionFiles>
    </configuration>
</plugin>
```

---

## Multi-Tool Strategy

```
RECOMMENDED SCA STACK:

  ┌────────────────────────────────────────────────────────────┐
  │                                                              │
  │  Tier 1: EVERY PR (Fast, blocking)                          │
  │  ┌──────────────────────────────────────────────────────┐  │
  │  │ Trivy fs . --severity CRITICAL,HIGH --exit-code 1   │  │
  │  │ Time: 10-30 seconds                                  │  │
  │  └──────────────────────────────────────────────────────┘  │
  │                                                              │
  │  Tier 2: CONTINUOUS MONITORING                               │
  │  ┌──────────────────────────────────────────────────────┐  │
  │  │ Snyk Monitor (tracks project continuously)           │  │
  │  │ Dependabot (auto-creates update PRs)                 │  │
  │  │ GitHub Security Alerts                                │  │
  │  └──────────────────────────────────────────────────────┘  │
  │                                                              │
  │  Tier 3: DEEP ANALYSIS (Weekly)                              │
  │  ┌──────────────────────────────────────────────────────┐  │
  │  │ OWASP Dependency-Check (comprehensive NVD scan)      │  │
  │  │ License compliance report                             │  │
  │  │ SBOM generation and archival                          │  │
  │  └──────────────────────────────────────────────────────┘  │
  └────────────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| NVD database download slow/fails | OWASP DC first run downloads full NVD | Use `--nvdApiKey` for faster downloads; cache DB in CI |
| Snyk test fails on monorepo | Multiple manifests | Use `--all-projects` flag to scan all sub-projects |
| Dependabot PRs pile up | Too many outdated deps | Group updates; set lower PR limits; prioritize security updates |
| False positive CVE | CVE applies to different usage context | Suppress with policy file (.snyk); document reason and expiry |
| Trivy misses some vulns | DB not updated | Run `trivy --download-db-only` first; check DB age |

---

## Summary Table

| Tool | Best For | Cost | Auto-Fix |
|------|----------|------|----------|
| **Snyk** | Full-featured SCA + monitoring | Freemium | Yes (PRs) |
| **Dependabot** | GitHub-native dependency updates | Free | Yes (PRs) |
| **Trivy** | Fast CI/CD scanning + containers | Free (OSS) | No |
| **OWASP DC** | Java/.NET deep analysis (NVD) | Free (OSS) | No |

---

## Quick Revision Questions

1. **What is the difference between Snyk and Dependabot?**
   > Snyk is a full-featured SCA platform supporting multiple ecosystems, license scanning, container scanning, and continuous monitoring. Dependabot is GitHub-native, focused on creating PRs to update dependencies. Snyk provides richer vulnerability data and fix advice; Dependabot is simpler and free for GitHub users.

2. **Why use Trivy over other SCA tools?**
   > Trivy is free, open-source, extremely fast (10-30 second scans), supports multiple ecosystems (npm, pip, Maven, Go, etc.), scans containers/IaC too, generates SBOMs, and integrates easily into any CI/CD. It's ideal as a fast first-line scanner.

3. **What is the purpose of Snyk's .snyk policy file?**
   > It allows teams to ignore specific vulnerabilities (with documented reason and expiry date) and apply patches. This handles false positives and accepted risks without disabling the entire scanner, maintaining visibility while reducing noise.

4. **How do you prevent Dependabot PR overload?**
   > Set `open-pull-requests-limit` to a manageable number, group minor/patch updates together, configure weekly schedules instead of daily, auto-merge patch updates that pass tests, and ignore major version bumps for stable packages using the `ignore` setting.

5. **What is a recommended multi-tool SCA strategy?**
   > Tier 1: Trivy for fast blocking scans on every PR (seconds). Tier 2: Snyk Monitor + Dependabot for continuous monitoring and auto-update PRs. Tier 3: OWASP Dependency-Check weekly for comprehensive NVD analysis and SBOM generation.

---

[← Previous: 5.1 Third-Party Risks](01-third-party-risks.md) | [Next: 5.3 Vulnerability Databases →](03-vulnerability-databases.md)

[Back to Table of Contents](../README.md)
