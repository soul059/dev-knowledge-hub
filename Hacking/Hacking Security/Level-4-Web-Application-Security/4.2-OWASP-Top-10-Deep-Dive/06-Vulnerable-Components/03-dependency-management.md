# Unit 6: A06 - Vulnerable and Outdated Components — Topic 3: Dependency Management

## Overview

Dependency management encompasses the **practices, policies, and tools** for selecting, versioning, updating, and removing third-party components throughout the software lifecycle. Poor dependency management leads to outdated libraries with known vulnerabilities, dependency confusion attacks, and bloated applications with unnecessary attack surface. This topic covers secure dependency management strategies.

---

## 1. Dependency Lifecycle

```
┌─────────────────────────────────────────────────────────────────┐
│              DEPENDENCY LIFECYCLE                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. SELECTION → Choose the right component                     │
│     ├── Is it actively maintained?                             │
│     ├── How many contributors/stars?                           │
│     ├── Security history (past CVEs)?                          │
│     ├── License compatibility?                                 │
│     └── Can we do this without a dependency?                   │
│                                                                 │
│  2. PINNING → Lock to specific versions                        │
│     ├── Use lock files                                         │
│     ├── Avoid floating versions (^, ~, *)                      │
│     └── Pin transitive dependencies                            │
│                                                                 │
│  3. MONITORING → Watch for issues                              │
│     ├── New CVE announcements                                  │
│     ├── End-of-life notifications                              │
│     └── License changes                                        │
│                                                                 │
│  4. UPDATING → Keep current                                    │
│     ├── Regular update schedule                                │
│     ├── Automated PR tools (Dependabot, Renovate)              │
│     └── Test before deploying                                  │
│                                                                 │
│  5. REMOVAL → Remove when no longer needed                     │
│     ├── Audit unused dependencies                              │
│     ├── Replace deprecated packages                            │
│     └── Minimize dependency count                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Version Pinning and Lock Files

### Semantic Versioning

```
Version format: MAJOR.MINOR.PATCH (e.g., 2.4.1)

MAJOR → Breaking changes (not backward compatible)
MINOR → New features (backward compatible)
PATCH → Bug fixes (backward compatible)

Version ranges in package managers:
┌────────────┬────────────────────────────────────┬──────────────┐
│ Specifier  │ Meaning                            │ Risk Level   │
├────────────┼────────────────────────────────────┼──────────────┤
│ 2.4.1      │ Exact version                      │ ✅ Safe      │
│ ~2.4.1     │ >=2.4.1, <2.5.0 (patch updates)   │ ⚠️ Low risk  │
│ ^2.4.1     │ >=2.4.1, <3.0.0 (minor updates)   │ ⚠️ Med risk  │
│ >=2.4.1    │ Any version >= 2.4.1               │ ❌ High risk │
│ *          │ Any version                         │ ❌ Dangerous │
│ latest     │ Latest published version            │ ❌ Dangerous │
└────────────┴────────────────────────────────────┴──────────────┘

ALWAYS use lock files to pin exact versions.
```

### Lock Files by Ecosystem

| Ecosystem | Manifest | Lock File | Commit Lock File? |
|-----------|----------|-----------|-------------------|
| Node.js (npm) | package.json | package-lock.json | ✅ Yes |
| Node.js (yarn) | package.json | yarn.lock | ✅ Yes |
| Python (pip) | requirements.txt | - (use pip-compile) | ✅ Yes |
| Python (Poetry) | pyproject.toml | poetry.lock | ✅ Yes |
| Python (Pipenv) | Pipfile | Pipfile.lock | ✅ Yes |
| Ruby | Gemfile | Gemfile.lock | ✅ Yes |
| Java (Maven) | pom.xml | - (versions explicit) | N/A |
| Java (Gradle) | build.gradle | gradle.lockfile | ✅ Yes |
| Go | go.mod | go.sum | ✅ Yes |
| .NET | .csproj | packages.lock.json | ✅ Yes |

---

## 3. Automated Dependency Updates

### Dependabot (GitHub)

```yaml
# .github/dependabot.yml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    reviewers:
      - "security-team"
    labels:
      - "dependencies"
      - "security"
    open-pull-requests-limit: 10
    # Group minor and patch updates
    groups:
      production-dependencies:
        patterns:
          - "*"
        update-types:
          - "minor"
          - "patch"
    # Security updates always
    security-updates:
      enabled: true

  - package-ecosystem: "docker"
    directory: "/"
    schedule:
      interval: "weekly"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

### Renovate

```json
// renovate.json
{
    "$schema": "https://docs.renovatebot.com/renovate-schema.json",
    "extends": [
        "config:base",
        ":semanticCommits",
        "group:recommended"
    ],
    "schedule": ["before 6am on monday"],
    "vulnerabilityAlerts": {
        "enabled": true,
        "labels": ["security"]
    },
    "packageRules": [
        {
            "matchUpdateTypes": ["major"],
            "labels": ["breaking-change"],
            "reviewers": ["team:security"]
        },
        {
            "matchUpdateTypes": ["patch", "minor"],
            "automerge": true,
            "automergeType": "pr",
            "requiredStatusChecks": ["ci/tests"]
        }
    ]
}
```

---

## 4. Supply Chain Attack Prevention

```
┌─────────────────────────────────────────────────────────────────┐
│              SUPPLY CHAIN ATTACK VECTORS                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. TYPOSQUATTING                                              │
│     Attacker publishes "lod-ash" hoping you mistype "lodash"   │
│     Prevention: Verify exact package names, use lockfiles      │
│                                                                 │
│  2. DEPENDENCY CONFUSION                                       │
│     Attacker publishes public package matching internal name   │
│     Prevention: Scoped packages (@company/pkg), registry lock  │
│                                                                 │
│  3. ACCOUNT TAKEOVER                                           │
│     Attacker compromises maintainer's npm/PyPI account         │
│     Prevention: Package signing, reproducible builds           │
│                                                                 │
│  4. MALICIOUS COMMITS                                          │
│     Attacker contributes malicious code to open source project │
│     Prevention: Code review, pin to specific commits           │
│                                                                 │
│  5. BUILD SYSTEM COMPROMISE                                    │
│     Attacker compromises CI/CD to inject malicious code        │
│     Prevention: Signed releases, SBOM verification             │
└─────────────────────────────────────────────────────────────────┘
```

### Dependency Confusion Prevention

```bash
# Use scoped packages (npm)
npm install @mycompany/internal-lib  # Not just "internal-lib"

# Configure registry per scope (.npmrc)
@mycompany:registry=https://npm.mycompany.com/
registry=https://registry.npmjs.org/

# Python — configure internal index
# pip.conf
[global]
index-url = https://pypi.mycompany.com/simple/
extra-index-url = https://pypi.org/simple/
```

### Package Integrity Verification

```bash
# npm — verify package integrity via lock file
# package-lock.json contains SHA-512 hashes:
# "integrity": "sha512-abc123..."
npm ci  # Uses lock file + verifies integrity

# Python — hash verification
# requirements.txt with hashes
flask==2.3.3 --hash=sha256:abc123...

# Verify: pip install --require-hashes -r requirements.txt

# Subresource Integrity (SRI) for CDN resources
<script src="https://cdn.example.com/lib.js"
    integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/uxy9rx7HNQlGYl1kPzQho1wx4JwY8wC"
    crossorigin="anonymous"></script>
```

---

## 5. Dependency Audit and Cleanup

```bash
# Find unused dependencies (Node.js)
npx depcheck

# Output:
# Unused dependencies:
# * moment
# * underscore
# Missing dependencies: (none)

# Find unused imports (Python)
pip install vulture
vulture myproject/

# Remove unused dependencies
npm uninstall moment underscore
pip uninstall moment

# Audit dependency licenses
npx license-checker --summary
pip-licenses --format=table
```

---

## 6. Dependency Management Policy

```
ORGANIZATIONAL POLICY:

1. APPROVAL
   → New dependencies require security review
   → Evaluate: maintenance status, CVE history, license

2. VERSIONING
   → Lock files MUST be committed
   → Exact versions in production
   → No floating versions in production configs

3. UPDATES
   → Critical/High CVE patches: within 48 hours
   → Regular updates: weekly automated PRs
   → Major version updates: planned and tested

4. MONITORING
   → Continuous vulnerability scanning in CI/CD
   → Alerts for new CVEs in used components
   → Quarterly dependency audit

5. REMOVAL
   → Annual audit of all dependencies
   → Remove unused dependencies
   → Replace deprecated packages

6. SUPPLY CHAIN
   → Use scoped/namespaced packages for internal code
   → Verify package integrity (checksums)
   → Mirror critical dependencies internally
```

---

## Summary Table

| Practice | Purpose | Tools |
|----------|---------|-------|
| Lock files | Pin exact versions | npm, pip-compile, Poetry |
| Automated updates | Keep dependencies current | Dependabot, Renovate |
| Supply chain security | Prevent malicious packages | Scoped packages, SRI, signing |
| Dependency audit | Find unused/risky packages | depcheck, vulture, license-checker |
| Version pinning | Prevent unexpected changes | Exact versions in manifests |

---

## Revision Questions

1. Explain semantic versioning and the risk of using `^` vs exact version specifiers.
2. What are lock files, why must they be committed, and how do they improve security?
3. Configure Dependabot for a project with npm, Docker, and GitHub Actions dependencies.
4. Describe three supply chain attack vectors and prevention measures for each.
5. What is dependency confusion? How do scoped packages prevent it?
6. Design a dependency management policy for an organization with 50+ applications.

---

*Previous: [02-vulnerability-scanning.md](02-vulnerability-scanning.md) | Next: [04-software-composition-analysis.md](04-software-composition-analysis.md)*

---

*[Back to README](../README.md)*
