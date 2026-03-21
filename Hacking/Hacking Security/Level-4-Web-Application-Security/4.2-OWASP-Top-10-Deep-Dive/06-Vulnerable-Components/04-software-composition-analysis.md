# Unit 6: A06 - Vulnerable and Outdated Components — Topic 4: Software Composition Analysis

## Overview

Software Composition Analysis (SCA) is a **category of security tools** that automatically identifies open-source components in a codebase, maps them to known vulnerabilities, checks license compliance, and assesses code quality risks. SCA is essential for modern applications where 70-90% of code comes from open-source dependencies.

---

## 1. What SCA Does

```
┌─────────────────────────────────────────────────────────────────┐
│                SCA ANALYSIS PIPELINE                            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  INPUT: Source code, binaries, container images                │
│           │                                                    │
│           ▼                                                    │
│  ┌─────────────────────┐                                       │
│  │ Component Discovery │  Identify all open-source components  │
│  │ - Manifest files    │  including transitive dependencies    │
│  │ - Binary analysis   │                                       │
│  │ - Code fingerprint  │                                       │
│  └────────┬────────────┘                                       │
│           ▼                                                    │
│  ┌─────────────────────┐                                       │
│  │ Vulnerability Match │  Map components to CVE databases      │
│  │ - NVD / CVE         │  Match version → known vulnerabilities│
│  │ - GitHub Advisory   │                                       │
│  │ - Vendor databases  │                                       │
│  └────────┬────────────┘                                       │
│           ▼                                                    │
│  ┌─────────────────────┐                                       │
│  │ License Analysis    │  Check license compatibility          │
│  │ - MIT, Apache, GPL  │  Flag copyleft and restrictive licenses│
│  │ - LGPL, AGPL, BSD   │                                       │
│  └────────┬────────────┘                                       │
│           ▼                                                    │
│  ┌─────────────────────┐                                       │
│  │ Risk Assessment     │  Maintenance status, popularity,      │
│  │ - Maintainer count  │  age, known malware                   │
│  │ - Last update       │                                       │
│  └────────┬────────────┘                                       │
│           ▼                                                    │
│  OUTPUT: Risk report, remediation advice, SBOM                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. SCA Tools Comparison

| Tool | Type | Languages | Vuln DB | License Check | SBOM | Price |
|------|------|-----------|---------|---------------|------|-------|
| **Snyk** | SaaS/CLI | 10+ | Snyk DB + NVD | ✅ | ✅ | Freemium |
| **OWASP Dep-Check** | OSS/CLI | Java, .NET, Node, Python | NVD | ❌ | ❌ | Free |
| **Trivy** | OSS/CLI | All + containers | Multiple | ✅ | ✅ | Free |
| **Grype** | OSS/CLI | All + containers | Anchore DB | ❌ | Pairs with Syft | Free |
| **Mend (WhiteSource)** | SaaS | 200+ | Proprietary | ✅ | ✅ | Commercial |
| **Black Duck** | SaaS | 80+ | Proprietary | ✅ | ✅ | Commercial |
| **Sonatype Nexus IQ** | SaaS | Java, JS, Python | OSS Index | ✅ | ✅ | Commercial |
| **FOSSA** | SaaS | 20+ | NVD | ✅ | ✅ | Commercial |
| **GitHub Dep Graph** | Native | All GitHub | GitHub Advisory | ❌ | ✅ | Free |

---

## 3. Running SCA Scans

### Snyk (Most Popular)

```bash
# Authenticate
snyk auth

# Test for vulnerabilities
snyk test
# Output:
# Testing /path/to/project...
# ✗ High severity vulnerability found in lodash
#   Description: Prototype Pollution
#   Info: https://snyk.io/vuln/SNYK-JS-LODASH-1234
#   From: express > body-parser > qs > lodash
#   Fix: Upgrade lodash to >=4.17.21

# Monitor continuously (sends alerts)
snyk monitor

# Test and fail on high/critical
snyk test --severity-threshold=high

# Generate SBOM
snyk sbom --format=cyclonedx1.4+json

# Container image scan
snyk container test myapp:latest

# Infrastructure as Code scan
snyk iac test terraform/
```

### Trivy (Open Source, Comprehensive)

```bash
# Scan project directory
trivy fs .

# Scan with severity filter
trivy fs --severity HIGH,CRITICAL .

# Scan and generate SBOM
trivy fs --format cyclonedx --output sbom.json .

# Scan container image
trivy image --severity CRITICAL myapp:latest

# Scan in CI with exit code
trivy fs --exit-code 1 --severity HIGH,CRITICAL .

# License scanning
trivy fs --scanners license .
trivy fs --scanners license --severity HIGH .  # Flag GPL etc.
```

---

## 4. SCA in CI/CD Pipeline

```yaml
# GitHub Actions — Multi-tool SCA
name: SCA Security Scan
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  sca-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install dependencies
        run: npm ci

      # Tool 1: npm audit
      - name: npm audit
        run: npm audit --audit-level=high

      # Tool 2: Trivy
      - name: Trivy scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          severity: 'HIGH,CRITICAL'
          exit-code: '1'
          format: 'sarif'
          output: 'trivy-results.sarif'

      # Upload to GitHub Security tab
      - name: Upload Trivy SARIF
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: 'trivy-results.sarif'

      # Tool 3: Snyk
      - name: Snyk scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
```

---

## 5. Reachability Analysis

```
Standard SCA: "You have lodash 4.17.20 with CVE-XXXX"
→ But does your code actually USE the vulnerable function?

Reachability Analysis: "lodash.template() is vulnerable AND 
your code calls lodash.template() in src/render.js:42"
→ Confirms the vulnerability is actually exploitable

This dramatically reduces false positives.

Tools with Reachability:
- Snyk (call graph analysis)
- Sonatype Nexus IQ
- Mend (WhiteSource)

Without reachability: 100 findings → 80 false positives
With reachability: 100 findings → 20 true positives
```

---

## 6. License Compliance

```
License Risk Categories:
┌────────────────┬────────────────────────────────────────────────┐
│ Risk Level     │ Licenses                                       │
├────────────────┼────────────────────────────────────────────────┤
│ ✅ Permissive  │ MIT, Apache-2.0, BSD-2/3-Clause, ISC          │
│ (Low risk)     │ Can use in commercial/proprietary software     │
├────────────────┼────────────────────────────────────────────────┤
│ ⚠️ Weak Copyleft│ LGPL-2.1, LGPL-3.0, MPL-2.0                 │
│ (Medium risk)  │ Can use if not modifying the library itself    │
├────────────────┼────────────────────────────────────────────────┤
│ ❌ Strong Copy │ GPL-2.0, GPL-3.0, AGPL-3.0                    │
│ (High risk)    │ May require open-sourcing your entire app      │
│                │ AGPL: even SaaS usage triggers copyleft        │
├────────────────┼────────────────────────────────────────────────┤
│ 🔴 No License  │ No license specified                          │
│ (Highest risk) │ Default copyright — cannot legally use         │
└────────────────┴────────────────────────────────────────────────┘
```

---

## Summary Table

| SCA Capability | Purpose | Key Tools |
|----------------|---------|-----------|
| Component discovery | Find all dependencies | Trivy, Syft, Snyk |
| Vulnerability matching | Map to CVEs | Snyk, OWASP DC, Trivy |
| License compliance | Check legal risk | FOSSA, Snyk, Black Duck |
| Reachability analysis | Confirm exploitability | Snyk, Sonatype |
| SBOM generation | Inventory export | Trivy, Syft, Snyk |
| CI/CD integration | Automated scanning | All tools |

---

## Revision Questions

1. What is Software Composition Analysis (SCA) and what are its four main functions?
2. Compare three SCA tools — features, languages supported, and pricing.
3. Set up a CI/CD pipeline with multiple SCA tools for a Node.js application.
4. What is reachability analysis and why does it reduce false positives?
5. Explain the difference between permissive, weak copyleft, and strong copyleft licenses.
6. Generate an SBOM using Trivy and explain the fields in the output.

---

*Previous: [03-dependency-management.md](03-dependency-management.md) | Next: [05-update-strategies.md](05-update-strategies.md)*

---

*[Back to README](../README.md)*
