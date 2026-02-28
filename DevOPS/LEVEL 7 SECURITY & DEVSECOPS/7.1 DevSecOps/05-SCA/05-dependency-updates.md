# Chapter 5.5: Dependency Update Strategies

## Overview

Keeping dependencies up to date is essential for security, but uncontrolled updates can break applications. This chapter covers strategies for safe, efficient dependency management including automated updates, version pinning, and upgrade workflows.

---

## Update Strategy Framework

```
DEPENDENCY UPDATE APPROACHES:

  ┌────────────────────────────────────────────────────────────┐
  │                                                              │
  │  CONSERVATIVE                    AGGRESSIVE                  │
  │  ┌──────────────┐              ┌──────────────┐             │
  │  │ Pin exact     │              │ Auto-merge    │             │
  │  │ versions      │              │ all updates   │             │
  │  │ Manual update │              │ Always latest │             │
  │  │               │              │               │             │
  │  │ + Stable      │              │ + Most secure │             │
  │  │ - Slow patches│              │ - May break   │             │
  │  └──────────────┘              └──────────────┘             │
  │         │                            │                       │
  │         └──────────┬─────────────────┘                       │
  │                    ▼                                          │
  │          RECOMMENDED: BALANCED                                │
  │          ┌──────────────────────────────────────────┐        │
  │          │ Security patches: Auto-merge (if tests pass)│    │
  │          │ Minor updates:    Auto-PR, manual review    │     │
  │          │ Major updates:    Manual, planned migration │     │
  │          └──────────────────────────────────────────┘        │
  └────────────────────────────────────────────────────────────┘
```

---

## Semantic Versioning (SemVer)

```
SEMVER:  MAJOR.MINOR.PATCH
         4     .17   .21

  ┌──────────┬──────────────────────────┬──────────────────┐
  │ Part     │ When Changes              │ Risk Level       │
  ├──────────┼──────────────────────────┼──────────────────┤
  │ PATCH    │ Bug fixes, security      │ Low — safe to    │
  │ (4.17.X) │ patches, no API changes  │ auto-merge       │
  ├──────────┼──────────────────────────┼──────────────────┤
  │ MINOR    │ New features, backward   │ Medium — review   │
  │ (4.X.0)  │ compatible additions     │ changelog         │
  ├──────────┼──────────────────────────┼──────────────────┤
  │ MAJOR    │ Breaking changes,        │ High — test       │
  │ (X.0.0)  │ API incompatibilities    │ thoroughly        │
  └──────────┴──────────────────────────┴──────────────────┘

  VERSION RANGES IN PACKAGE MANAGERS:
  ┌──────────────────────────────────────────────────────────┐
  │ "lodash": "4.17.21"   → Exact (safest)                  │
  │ "lodash": "~4.17.21"  → Patch updates only (4.17.x)     │
  │ "lodash": "^4.17.21"  → Minor + patch (4.x.x)           │
  │ "lodash": "*"         → Any version (dangerous!)          │
  └──────────────────────────────────────────────────────────┘
```

---

## Automated Update Workflows

### Dependabot with Auto-merge (GitHub)

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "daily"
    groups:
      # Group patch updates for auto-merge
      patch-updates:
        update-types: ["patch"]
      # Group minor updates for review
      minor-updates:
        update-types: ["minor"]

# .github/workflows/dependabot-automerge.yml
name: Auto-merge Dependabot
on: pull_request
permissions:
  contents: write
  pull-requests: write

jobs:
  auto-merge:
    runs-on: ubuntu-latest
    if: github.actor == 'dependabot[bot]'
    steps:
      - uses: dependabot/fetch-metadata@v2
        id: metadata
      
      # Auto-merge security patches
      - name: Auto-merge security patches
        if: steps.metadata.outputs.update-type == 'version-update:semver-patch' 
            && steps.metadata.outputs.dependency-type == 'direct:production'
        run: gh pr merge --auto --squash "$PR_URL"
        env:
          PR_URL: ${{ github.event.pull_request.html_url }}
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Renovate Bot (Alternative to Dependabot)

```json
// renovate.json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": [
    "config:recommended",
    "security:openssf-scorecard"
  ],
  "packageRules": [
    {
      "description": "Auto-merge patch updates",
      "matchUpdateTypes": ["patch"],
      "matchCurrentVersion": "!/^0/",
      "automerge": true,
      "automergeType": "pr",
      "platformAutomerge": true
    },
    {
      "description": "Group minor updates weekly",
      "matchUpdateTypes": ["minor"],
      "groupName": "minor updates",
      "schedule": ["before 9am on Monday"]
    },
    {
      "description": "Manual review for major updates",
      "matchUpdateTypes": ["major"],
      "labels": ["breaking-change"],
      "reviewers": ["team:engineering-leads"]
    },
    {
      "description": "Security updates - immediate",
      "matchCategories": ["security"],
      "automerge": true,
      "schedule": ["at any time"],
      "prPriority": 10
    }
  ],
  "vulnerabilityAlerts": {
    "enabled": true,
    "labels": ["security"],
    "automerge": true
  }
}
```

---

## Dependency Override / Resolution

```json
// package.json - npm overrides (fix transitive vuln)
{
  "overrides": {
    "vulnerable-package": "2.0.1",
    "parent-package": {
      "vulnerable-transitive": "1.5.3"
    }
  }
}
```

```xml
<!-- pom.xml - Maven dependency management -->
<dependencyManagement>
    <dependencies>
        <!-- Force specific version of transitive dependency -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.21.1</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

```python
# pip - constraint file
# constraints.txt
requests>=2.31.0    # Force minimum version
urllib3>=2.0.7      # Patch CVE in transitive dep

# pip install -c constraints.txt -r requirements.txt
```

---

## Update Planning Process

```
DEPENDENCY UPDATE SPRINT WORKFLOW:

  Week 1: ASSESS
  ┌────────────────────────────────────────────────────┐
  │ 1. Run SCA scan → list all outdated dependencies   │
  │ 2. Check changelogs for breaking changes           │
  │ 3. Prioritize: Security fixes > Features > Cleanup │
  │ 4. Create update plan and branch                    │
  └────────────────────────────────────────────────────┘

  Week 2: UPDATE + TEST
  ┌────────────────────────────────────────────────────┐
  │ 1. Update one dependency at a time                  │
  │ 2. Run full test suite after each update            │
  │ 3. Check for deprecation warnings                   │
  │ 4. Update code if APIs changed                      │
  └────────────────────────────────────────────────────┘

  Week 3: VALIDATE + DEPLOY
  ┌────────────────────────────────────────────────────┐
  │ 1. Integration testing on staging                   │
  │ 2. Performance testing (regressions?)               │
  │ 3. Security scan (new vulns introduced?)            │
  │ 4. Deploy with monitoring                           │
  └────────────────────────────────────────────────────┘
```

---

## Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| Auto-merge breaks build | Patch update had breaking change | Always require passing tests before auto-merge; SemVer isn't always followed |
| Can't upgrade due to peer dep conflicts | Version constraints between packages | Use `--legacy-peer-deps` temporarily; check if newer versions resolve conflict |
| Transitive vulnerability can't be fixed | Direct dependency hasn't updated | Use overrides/resolutions; fork dependency; apply workaround |
| Too many Dependabot PRs | All dependencies outdated | Group updates; increase interval; handle backlog in a sprint |
| Renovate and Dependabot conflicting | Both enabled | Use only one; Renovate is more configurable |

---

## Summary Table

| Strategy | Description | Risk | Effort |
|----------|-------------|------|--------|
| **Pin Exact** | Lock to specific version | Lowest | High (manual updates) |
| **Auto-merge Patch** | Merge patches if tests pass | Low | Low |
| **Group Minor** | Batch minor updates weekly | Medium | Medium |
| **Manual Major** | Plan and test major upgrades | Controlled | High |
| **Override** | Force version for transitive deps | Medium | Low |
| **Constraint File** | Set minimum versions | Low | Low |

---

## Quick Revision Questions

1. **What is the recommended approach for managing dependency updates?**
   > A balanced approach: auto-merge security patches (if tests pass), auto-create PRs for minor updates (manual review), and manually plan major version upgrades. This balances security responsiveness with stability.

2. **What is semantic versioning and why does it matter for auto-updates?**
   > SemVer (MAJOR.MINOR.PATCH) indicates: PATCH = bug fixes (safe), MINOR = new features (backward compatible), MAJOR = breaking changes. Auto-merging is generally safe for patch updates but risky for major updates, since breaking changes require code modifications.

3. **How do you fix a vulnerability in a transitive dependency?**
   > Use package manager override mechanisms: npm `overrides`, Yarn `resolutions`, Maven `dependencyManagement`, or pip constraint files. These force specific versions of transitive dependencies without waiting for the direct dependency to update.

4. **What is the difference between Dependabot and Renovate?**
   > Both create automated update PRs. Dependabot is GitHub-native and simpler to configure. Renovate offers more features: auto-merge, grouping, scheduling, custom rules, and runs on any platform (GitHub, GitLab, Bitbucket). Renovate is preferred for complex monorepos.

5. **Why should you update one dependency at a time?**
   > If multiple dependencies are updated simultaneously and something breaks, it's difficult to identify which update caused the issue. Updating one at a time with testing after each change makes debugging easier and rollback cleaner.

---

[← Previous: 5.4 License Compliance](04-license-compliance.md) | [Next: 6.1 Image Security Scanning →](../06-Container-Security/01-image-security-scanning.md)

[Back to Table of Contents](../README.md)
