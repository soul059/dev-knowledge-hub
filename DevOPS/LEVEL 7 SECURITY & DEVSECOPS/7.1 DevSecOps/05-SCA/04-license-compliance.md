# Chapter 5.4: License Compliance

## Overview

Open source licenses define how software can be used, modified, and distributed. Using a dependency with an incompatible license can create legal liability, force open-sourcing proprietary code, or block commercial release. This chapter covers license types, compliance scanning, and policy enforcement.

---

## License Categories

```
LICENSE SPECTRUM:

  PERMISSIVE ←──────────────────────────────────→ RESTRICTIVE

  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
  │ MIT      │  │ Apache   │  │ LGPL     │  │ GPL      │
  │ BSD      │  │ 2.0      │  │          │  │ AGPL     │
  │          │  │          │  │          │  │          │
  │ Do almost│  │ Patent   │  │ Must     │  │ Must     │
  │ anything │  │ grant +  │  │ share    │  │ share    │
  │ Keep     │  │ notice   │  │ changes  │  │ ALL code │
  │ notice   │  │          │  │ to lib   │  │ using it │
  └──────────┘  └──────────┘  └──────────┘  └──────────┘
     LOW RISK      LOW RISK    MEDIUM RISK    HIGH RISK
     (Commercial   (Commercial (Check usage)  (Viral —
      friendly)     friendly)                 avoid in
                                              commercial)
```

---

## License Types Deep Dive

```
  ┌──────────────┬───────────┬───────────┬───────────┬──────────┐
  │ Permission    │ MIT/BSD   │ Apache 2  │ LGPL 2.1  │ GPL 3    │
  ├──────────────┼───────────┼───────────┼───────────┼──────────┤
  │ Commercial   │ Yes       │ Yes       │ Yes       │ Caution  │
  │ use           │           │           │           │          │
  ├──────────────┼───────────┼───────────┼───────────┼──────────┤
  │ Modify        │ Yes       │ Yes       │ Yes       │ Yes      │
  ├──────────────┼───────────┼───────────┼───────────┼──────────┤
  │ Distribute   │ Yes       │ Yes       │ Yes       │ Yes      │
  ├──────────────┼───────────┼───────────┼───────────┼──────────┤
  │ Patent grant │ No        │ Yes       │ No        │ Yes      │
  ├──────────────┼───────────┼───────────┼───────────┼──────────┤
  │ Must include │ License   │ License + │ License   │ License +│
  │               │ notice    │ NOTICE    │ notice    │ source   │
  ├──────────────┼───────────┼───────────┼───────────┼──────────┤
  │ Copyleft     │ No        │ No        │ Weak      │ Strong   │
  │ (share-alike)│           │           │ (lib only)│ (all)    │
  ├──────────────┼───────────┼───────────┼───────────┼──────────┤
  │ SaaS loophole│ Yes       │ Yes       │ Yes       │ Yes      │
  │ (no sharing) │           │           │           │ (AGPL:No)│
  └──────────────┴───────────┴───────────┴───────────┴──────────┘

  AGPL (Affero GPL):
  ┌──────────────────────────────────────────────────────┐
  │ Like GPL but CLOSES the SaaS loophole                 │
  │ If you use AGPL code in a web service, you MUST      │
  │ share your entire source code — even if not           │
  │ distributing binaries                                  │
  │                                                        │
  │ → Most restrictive common license                      │
  │ → Many companies ban AGPL entirely                     │
  └──────────────────────────────────────────────────────┘
```

---

## License Policy Configuration

### Define Policy

```yaml
# license-policy.yml
policies:
  allowed:
    - MIT
    - Apache-2.0
    - BSD-2-Clause
    - BSD-3-Clause
    - ISC
    - CC0-1.0
    - Unlicense
    - 0BSD

  restricted:       # Requires legal review
    - LGPL-2.1
    - LGPL-3.0
    - MPL-2.0
    - EPL-1.0
    - EPL-2.0
    - CC-BY-4.0

  banned:           # Never allowed
    - GPL-2.0
    - GPL-3.0
    - AGPL-3.0
    - SSPL-1.0
    - EUPL-1.1
    - CC-BY-NC-4.0   # Non-commercial

  unknown:          # Flag for manual review
    action: warn
    message: "Unknown license requires legal review before use"
```

### Scanning with Trivy

```bash
# Scan licenses
trivy fs --scanners license .
trivy fs --scanners license --severity HIGH,CRITICAL .

# Custom license policy
trivy fs --scanners license --license-full \
    --ignored-licenses MIT,Apache-2.0,ISC .
```

### Scanning with license-checker (npm)

```bash
# Install
npm install -g license-checker

# Check all licenses
license-checker --json --out licenses.json

# Fail on restricted licenses
license-checker --failOn "GPL-2.0;GPL-3.0;AGPL-3.0"

# Only production dependencies
license-checker --production --csv --out licenses.csv

# Exclude allowed licenses (show only concerns)
license-checker --excludePackages "MIT;Apache-2.0;BSD-2-Clause;ISC"
```

### CI/CD License Check

```yaml
# GitHub Actions license scan
name: License Compliance
on: [pull_request]

jobs:
  license-check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      
      - run: npm ci
      
      - name: Check Licenses
        run: |
          npx license-checker --production \
            --failOn "GPL-2.0-only;GPL-3.0-only;AGPL-3.0-only" \
            --excludePrivatePackages
      
      - name: Generate License Report
        if: github.ref == 'refs/heads/main'
        run: |
          npx license-checker --production --csv \
            --out license-report.csv
      
      - uses: actions/upload-artifact@v4
        if: github.ref == 'refs/heads/main'
        with:
          name: license-report
          path: license-report.csv
```

---

## License Compliance Workflow

```
LICENSE COMPLIANCE PROCESS:

  New Dependency Added
      │
      ▼
  ┌──────────────────┐
  │ Automated scan    │
  │ identifies license│
  └────────┬─────────┘
           │
     ┌─────┼─────────┐
     ▼     ▼         ▼
  Allowed  Restricted  Banned
     │        │          │
     ▼        ▼          ▼
  Auto-    Legal      Block PR
  approve  Review     Alert dev
           │
     ┌─────┼─────┐
     ▼           ▼
  Approved    Rejected
  (with       (find
  conditions)  alternative)
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Package has no license declared | Maintainer didn't specify | Check repo manually; assume restrictive; contact maintainer |
| Dual-licensed package | Multiple licenses available | Choose the most permissive option; document choice |
| License changed in new version | Maintainer re-licensed | Pin to previous version; review new license implications |
| Transitive dep has GPL | Your direct dep uses GPL library | Check if linked dynamically (LGPL ok) or statically (GPL viral) |
| "Custom" or "SEE LICENSE" | Non-standard license | Flag for legal review; may be proprietary |

---

## Summary Table

| License | Type | Commercial OK | Copyleft | Risk Level |
|---------|------|--------------|----------|------------|
| **MIT** | Permissive | Yes | No | Low |
| **Apache-2.0** | Permissive | Yes | No | Low |
| **BSD** | Permissive | Yes | No | Low |
| **LGPL** | Weak Copyleft | Yes (careful) | Library only | Medium |
| **GPL** | Strong Copyleft | Careful | All linked code | High |
| **AGPL** | Network Copyleft | Ban in most orgs | All code, inc. SaaS | Critical |

---

## Quick Revision Questions

1. **What is copyleft and why does it matter for commercial software?**
   > Copyleft (share-alike) licenses require that derivative works be distributed under the same license. GPL requires sharing source code of anything linked with GPL code. For commercial software, this could force open-sourcing proprietary code, making GPL/AGPL generally incompatible with commercial licensing.

2. **What is the difference between GPL and AGPL?**
   > GPL requires sharing source code only when distributing binaries. AGPL closes the "SaaS loophole" — if you use AGPL code in a web service (even without distributing), you must share your entire source code. This makes AGPL the most restrictive common license.

3. **What licenses are generally safe for commercial use?**
   > MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, ISC, and CC0/Unlicense. These are permissive licenses that only require preserving copyright notices and license text.

4. **How do you handle a dependency with an unknown license?**
   > Flag it for manual review. Check the package repository for a LICENSE file, README mentions, or package.json license field. Contact the maintainer for clarification. Until confirmed, assume the most restrictive interpretation and avoid using it in production.

5. **Why should license scanning be automated in CI/CD?**
   > Developers frequently add dependencies without checking licenses. Automated scanning catches this at PR time before code enters the codebase. It prevents legal liability, ensures compliance with company policy, and creates an audit trail of all licenses used.

---

[← Previous: 5.3 Vulnerability Databases](03-vulnerability-databases.md) | [Next: 5.5 Dependency Updates →](05-dependency-updates.md)

[Back to Table of Contents](../README.md)
