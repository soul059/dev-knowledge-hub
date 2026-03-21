# Unit 6: A06 - Vulnerable and Outdated Components — Topic 5: Update Strategies

## Overview

Having a **systematic update strategy** ensures that vulnerable components are patched promptly while minimizing the risk of breaking changes. This topic covers update policies, automated update tools, testing strategies, rollback procedures, and how to balance security urgency with stability requirements.

---

## 1. Update Policy Framework

```
┌─────────────────────────────────────────────────────────────────┐
│              UPDATE POLICY BY SEVERITY                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  CRITICAL (CVSS 9.0-10.0) — Active exploitation / wormable    │
│  ├── SLA: Patch within 24-48 hours                             │
│  ├── Process: Emergency change, skip standard release cycle    │
│  ├── Testing: Automated smoke tests + critical paths           │
│  └── Approval: Security team fast-track approval               │
│                                                                 │
│  HIGH (CVSS 7.0-8.9) — Significant risk                       │
│  ├── SLA: Patch within 7 days                                  │
│  ├── Process: Priority bug fix, expedited release              │
│  ├── Testing: Full regression test suite                       │
│  └── Approval: Standard review + security sign-off             │
│                                                                 │
│  MEDIUM (CVSS 4.0-6.9) — Moderate risk                        │
│  ├── SLA: Patch within 30 days                                 │
│  ├── Process: Include in next scheduled release                │
│  ├── Testing: Standard test cycle                              │
│  └── Approval: Standard review                                 │
│                                                                 │
│  LOW (CVSS 0.1-3.9) — Minimal risk                            │
│  ├── SLA: Patch in next quarterly update                       │
│  ├── Process: Bundle with other updates                        │
│  ├── Testing: Standard test cycle                              │
│  └── Approval: Batch approval                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Update Types

```
PATCH UPDATES (x.x.PATCH):
  Risk: Very low — bug fixes only
  Strategy: Auto-merge after CI passes
  Example: 4.17.20 → 4.17.21

MINOR UPDATES (x.MINOR.x):
  Risk: Low — new features, backward compatible
  Strategy: Auto-merge after CI + review
  Example: 4.17.21 → 4.18.0

MAJOR UPDATES (MAJOR.x.x):
  Risk: High — breaking changes possible
  Strategy: Manual review, migration plan, staged rollout
  Example: 4.18.0 → 5.0.0

SECURITY UPDATES:
  Risk: Varies by CVSS
  Strategy: Fast-track regardless of version bump
  Example: Any version with CVE fix
```

---

## 3. Automated Update Workflow

```
┌──────────────────────────────────────────────────────────────────┐
│           AUTOMATED UPDATE PIPELINE                              │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Dependabot/Renovate creates PR                              │
│       ↓                                                          │
│  2. CI pipeline runs automatically:                             │
│     ├── Unit tests                                              │
│     ├── Integration tests                                       │
│     ├── Security scan (SCA)                                     │
│     ├── Build verification                                      │
│     └── E2E tests                                               │
│       ↓                                                          │
│  3. IF all pass + patch/minor update:                           │
│     → Auto-merge (for trusted dependencies)                    │
│       ↓                                                          │
│  4. IF major update OR tests fail:                              │
│     → Assign to developer for review                           │
│       ↓                                                          │
│  5. Deploy to staging → verify                                  │
│       ↓                                                          │
│  6. Deploy to production (canary/blue-green)                    │
│       ↓                                                          │
│  7. Monitor for regressions                                     │
└──────────────────────────────────────────────────────────────────┘
```

### Renovate Auto-Merge Configuration

```json
{
    "packageRules": [
        {
            "description": "Auto-merge patch updates for production deps",
            "matchUpdateTypes": ["patch"],
            "matchDepTypes": ["dependencies"],
            "automerge": true,
            "automergeType": "pr",
            "requiredStatusChecks": ["ci/tests", "ci/security-scan"]
        },
        {
            "description": "Auto-merge minor updates for dev deps",
            "matchUpdateTypes": ["patch", "minor"],
            "matchDepTypes": ["devDependencies"],
            "automerge": true
        },
        {
            "description": "Manual review for major updates",
            "matchUpdateTypes": ["major"],
            "automerge": false,
            "reviewers": ["team:engineering"],
            "labels": ["breaking-change"]
        },
        {
            "description": "Fast-track security updates",
            "matchCategories": ["security"],
            "automerge": true,
            "automergeType": "pr",
            "priorityLevel": 1,
            "labels": ["security"]
        }
    ]
}
```

---

## 4. Testing Strategy for Updates

```
TIERED TESTING APPROACH:

Tier 1 — Automated (runs on every update PR):
  ├── Unit tests (fast, high coverage)
  ├── Lint and type checking
  ├── Build verification
  └── Security scanning

Tier 2 — Integration (runs on merge to main):
  ├── API integration tests
  ├── Database migration tests
  ├── Third-party service integration
  └── Performance benchmarks

Tier 3 — E2E (runs before production deploy):
  ├── Critical user flows
  ├── Browser compatibility
  ├── Load testing
  └── Security penetration tests

ROLLBACK CRITERIA:
  → Error rate increases > 1%
  → Response time increases > 20%
  → Any test failure in Tier 2/3
  → Any new security vulnerability
```

---

## 5. Handling Breaking Changes

```python
# Strategy for major version upgrades:

# 1. Create a feature branch
# git checkout -b upgrade/express-v5

# 2. Update the dependency
# npm install express@5

# 3. Run tests — identify failures
# npm test 2>&1 | tee test-output.txt

# 4. Read migration guide
# https://expressjs.com/en/guide/migrating-5.html

# 5. Fix breaking changes one by one
# Update code to use new API

# 6. Run full test suite
# npm run test:all

# 7. Deploy to staging first
# Test for 1-2 weeks in staging

# 8. Canary deploy to production
# Route 5% traffic → monitor → increase gradually
```

### Migration Checklist for Major Updates

```
□ Read the changelog and migration guide
□ Check for deprecated API usage in codebase
□ Create feature branch for upgrade
□ Update dependency version
□ Run tests and fix failures
□ Update any configuration changes
□ Check for peer dependency conflicts
□ Test in staging environment
□ Performance benchmark comparison
□ Security scan (no new vulnerabilities)
□ Canary deploy to production
□ Monitor error rates and performance
□ Full rollout if stable after 48 hours
□ Update documentation
```

---

## 6. Emergency Patching Procedure

```
CRITICAL VULNERABILITY RESPONSE:

Hour 0:   CVE disclosed / advisory published
          ├── Security team receives alert
          └── Triage: Is our application affected?

Hour 1-4: Assessment
          ├── Which services use the affected component?
          ├── Is the vulnerable function called?
          ├── Is there a public exploit?
          └── What's the fix version?

Hour 4-8: Patch Development
          ├── Update dependency to fixed version
          ├── Run automated test suite
          ├── If tests pass → fast-track deployment
          └── If no fix available → implement workaround

Hour 8-24: Deployment
          ├── Deploy to staging → quick verification
          ├── Deploy to production (all environments)
          ├── Verify patch in production
          └── If fix unavailable → deploy WAF rule / mitigation

Hour 24+: Follow-up
          ├── Incident report
          ├── Update SCA/SBOM
          ├── Verify no exploitation occurred
          └── Review detection time → improve monitoring
```

---

## Summary Table

| Update Type | Risk | SLA | Strategy |
|------------|------|-----|----------|
| Patch | Very Low | Next release | Auto-merge |
| Minor | Low | 1-2 weeks | Auto-merge + review |
| Major | High | Planned sprint | Manual review + staged rollout |
| Security Critical | Varies | 24-48 hours | Emergency patch |
| Security High | Varies | 7 days | Priority fix |

---

## Revision Questions

1. Design an update policy that defines SLAs for Critical, High, Medium, and Low vulnerabilities.
2. Configure Renovate to auto-merge patch updates, require review for major updates, and fast-track security updates.
3. What is a canary deployment and why is it important for dependency updates?
4. Create a migration checklist for upgrading a major framework version.
5. Describe the emergency patching procedure for a critical CVE in a production dependency.
6. How do you balance the need for rapid security patching with the risk of breaking changes?

---

*Previous: [04-software-composition-analysis.md](04-software-composition-analysis.md) | Next: [06-software-bill-of-materials.md](06-software-bill-of-materials.md)*

---

*[Back to README](../README.md)*
