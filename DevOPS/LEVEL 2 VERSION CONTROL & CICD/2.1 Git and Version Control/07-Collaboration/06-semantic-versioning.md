# Chapter 7.6: Semantic Versioning

[← Previous: Branch Protection Rules](05-branch-protection-rules.md) | [Back to README](../README.md) | [Next: Issues and Project Boards →](../08-GitHub-GitLab-Features/01-issues-and-project-boards.md)

---

## Overview

**Semantic Versioning (SemVer)** is a versioning scheme that communicates the nature of changes through version numbers. It uses the format **MAJOR.MINOR.PATCH** and is the standard for libraries, APIs, and software releases.

---

## The SemVer Format

```
┌──────────────────────────────────────────────────────────────┐
│       SEMANTIC VERSIONING FORMAT                            │
│                                                              │
│              v 2 . 4 . 1                                    │
│                │   │   │                                    │
│                │   │   └── PATCH: Backward-compatible fixes │
│                │   │       (bug fixes, patches)             │
│                │   │                                        │
│                │   └────── MINOR: Backward-compatible adds  │
│                │           (new features, no breaking)      │
│                │                                            │
│                └────────── MAJOR: Breaking changes          │
│                            (incompatible API changes)       │
│                                                              │
│  RULE: Start at 0.1.0 (development), go to 1.0.0 (stable) │
└──────────────────────────────────────────────────────────────┘
```

---

## Version Bump Rules

```
┌──────────────────────────────────────────────────────────────┐
│  WHEN TO BUMP EACH NUMBER                                   │
│                                                              │
│  Current: 1.4.2                                             │
│                                                              │
│  Bug fix (backward compatible):                             │
│    1.4.2  →  1.4.3   (PATCH +1)                             │
│                                                              │
│  New feature (backward compatible):                         │
│    1.4.2  →  1.5.0   (MINOR +1, PATCH reset to 0)          │
│                                                              │
│  Breaking change:                                           │
│    1.4.2  →  2.0.0   (MAJOR +1, MINOR & PATCH reset to 0)  │
│                                                              │
│  ⚠️  Higher-level bump resets lower numbers!                │
└──────────────────────────────────────────────────────────────┘
```

### Examples

| Change | Type | Version |
|--------|------|---------|
| Fix login crash | PATCH | 1.0.0 → 1.0.1 |
| Add search feature | MINOR | 1.0.1 → 1.1.0 |
| Fix search edge case | PATCH | 1.1.0 → 1.1.1 |
| Redesign API endpoints | MAJOR | 1.1.1 → 2.0.0 |
| Add pagination to new API | MINOR | 2.0.0 → 2.1.0 |

---

## Pre-Release and Build Metadata

```
┌──────────────────────────────────────────────────────────────┐
│  EXTENDED VERSION FORMAT                                    │
│                                                              │
│  MAJOR.MINOR.PATCH-prerelease+build                         │
│                                                              │
│  Pre-release identifiers (lower precedence):                │
│  ├── 1.0.0-alpha        (earliest, unstable)                │
│  ├── 1.0.0-alpha.1      (iteration)                         │
│  ├── 1.0.0-beta         (feature-complete, testing)         │
│  ├── 1.0.0-beta.2       (iteration)                         │
│  ├── 1.0.0-rc.1         (release candidate)                 │
│  └── 1.0.0              (stable release)                    │
│                                                              │
│  Build metadata (ignored in precedence):                    │
│  ├── 1.0.0+build.123                                        │
│  ├── 1.0.0+20250115                                         │
│  └── 1.0.0-beta.1+build.456                                 │
│                                                              │
│  Precedence (lowest → highest):                             │
│  1.0.0-alpha < 1.0.0-alpha.1 < 1.0.0-beta < 1.0.0-rc.1    │
│  < 1.0.0                                                    │
└──────────────────────────────────────────────────────────────┘
```

---

## SemVer in Git Workflows

```
┌──────────────────────────────────────────────────────────────┐
│  VERSION LIFECYCLE WITH GIT TAGS                            │
│                                                              │
│  Development:                                               │
│  ●──●──●──●──●──●──●──●──●──●──●──●                        │
│     ▲        ▲     ▲        ▲     ▲                        │
│   v0.1.0   v0.2.0 v1.0.0  v1.0.1 v1.1.0                   │
│   (dev)    (dev)  (stable)(patch)(feature)                  │
│                                                              │
│  Tagging releases:                                          │
│  $ git tag -a v1.0.0 -m "Initial stable release"           │
│  $ git push origin v1.0.0                                   │
│                                                              │
│  Pre-release tags:                                          │
│  $ git tag -a v2.0.0-rc.1 -m "Version 2.0 release cand."  │
└──────────────────────────────────────────────────────────────┘
```

---

## Version 0.x.x (Initial Development)

```
┌──────────────────────────────────────────────────────────────┐
│  INITIAL DEVELOPMENT (0.x.x)                                │
│                                                              │
│  • Version 0.x.x means anything can change at any time      │
│  • The public API is NOT considered stable                   │
│  • Breaking changes can happen in MINOR bumps               │
│  • Start at 0.1.0                                           │
│                                                              │
│  0.1.0 — First development release                          │
│  0.2.0 — Added features (may have breaking changes)         │
│  0.9.0 — Approaching stability                              │
│  1.0.0 — First stable release (public API defined)          │
│                                                              │
│  RULE: Version 1.0.0 defines the public API.                │
│  After 1.0.0, SemVer rules are strictly followed.           │
└──────────────────────────────────────────────────────────────┘
```

---

## Version Ranges in Package Managers

```
┌──────────────────────────────────────────────────────────────┐
│  HOW SEMVER IS USED IN DEPENDENCIES                         │
│                                                              │
│  npm / package.json:                                        │
│  "dependencies": {                                          │
│    "express": "^4.18.2",    // ^: minor + patch updates OK  │
│                              //    4.18.2 ≤ v < 5.0.0       │
│    "lodash": "~4.17.21",   // ~: patch updates only         │
│                              //    4.17.21 ≤ v < 4.18.0     │
│    "react": "18.2.0",      // Exact version only            │
│    "webpack": ">=5.0.0"    // Any version 5+                │
│  }                                                          │
│                                                              │
│  ^  (caret) — Accept MINOR + PATCH updates                  │
│  ~  (tilde) — Accept PATCH updates only                     │
│  =  (exact) — Only this exact version                       │
│  >= (range) — This version or higher                        │
└──────────────────────────────────────────────────────────────┘
```

---

## Automating Version Bumps

### With Conventional Commits

```bash
# standard-version reads commit messages to determine bump:
$ npx standard-version
# fix: → PATCH, feat: → MINOR, BREAKING CHANGE → MAJOR

# Or specify manually:
$ npx standard-version --release-as minor
$ npx standard-version --release-as 2.0.0

# With npm version (built-in):
$ npm version patch    # 1.0.0 → 1.0.1
$ npm version minor    # 1.0.1 → 1.1.0
$ npm version major    # 1.1.0 → 2.0.0
# Automatically creates a git tag!
```

### GitHub Release-Please

```yaml
# .github/workflows/release.yml
name: Release Please
on:
  push:
    branches: [main]
jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: google-github-actions/release-please-action@v4
        with:
          release-type: node
# Automatically creates release PRs based on conventional commits
```

---

## Real-World Scenarios

### Scenario 1: Library Maintainer Versioning

```
v1.0.0  — Initial stable release
v1.0.1  — Fix: handle null input (patch)
v1.1.0  — Add: new utility function (minor)
v1.1.1  — Fix: memory leak in utility (patch)
v2.0.0  — Rewrite: new API, old functions removed (major)
v2.0.1  — Fix: typo in error messages (patch)
```

### Scenario 2: Deciding the Bump Type

```
Question: "I renamed a public function."
Answer:   MAJOR — it's a breaking change for consumers.

Question: "I added a new optional parameter to a function."
Answer:   MINOR — backward compatible (existing calls still work).

Question: "I fixed a typo in an error message."
Answer:   PATCH — no behavior change, just a fix.
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Don't know whether it's MAJOR, MINOR, or PATCH | Ask: "Will existing users' code break?" Yes → MAJOR; adds feature → MINOR; bug fix → PATCH |
| Version 0.x.x confusion | 0.x.x = development; anything may change. 1.0.0+ = stable SemVer |
| Pre-release ordering | alpha < beta < rc < release (1.0.0-alpha < 1.0.0) |
| Forgot to bump version | Use automated tools (standard-version, release-please) |
| Multiple breaking changes in one release | Still one MAJOR bump — combine in one release |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| MAJOR | Breaking changes (incompatible API changes) |
| MINOR | New features (backward compatible) |
| PATCH | Bug fixes (backward compatible) |
| Pre-release | alpha → beta → rc (unstable, lower precedence) |
| Build metadata | +build.123 (ignored in version precedence) |
| v0.x.x | Initial development, anything can change |
| v1.0.0 | First stable API, SemVer rules apply |
| ^ (caret) | Accept minor + patch updates |
| ~ (tilde) | Accept patch updates only |

---

## Quick Revision Questions

1. **What does each number in SemVer represent?**
   <details><summary>Answer</summary>MAJOR.MINOR.PATCH — MAJOR is incremented for breaking (incompatible) changes, MINOR for backward-compatible new features, and PATCH for backward-compatible bug fixes. When a higher-level number is bumped, lower numbers reset to 0.</details>

2. **When should you bump to version 2.0.0 from 1.x.x?**
   <details><summary>Answer</summary>When you make any incompatible/breaking change to the public API. Examples: removing a function, changing a function's signature in an incompatible way, renaming public methods, or changing return types.</details>

3. **What is the difference between `^4.18.2` and `~4.18.2` in package.json?**
   <details><summary>Answer</summary>`^4.18.2` (caret) accepts minor and patch updates: any version ≥4.18.2 and <5.0.0. `~4.18.2` (tilde) accepts only patch updates: any version ≥4.18.2 and <4.19.0. Caret is more permissive.</details>

4. **What does version 0.x.x signify?**
   <details><summary>Answer</summary>Initial development phase. The public API is not considered stable, and breaking changes can happen at any time (even in minor bumps). Version 1.0.0 marks the first stable release where SemVer rules are strictly followed.</details>

5. **How can Conventional Commits automate versioning?**
   <details><summary>Answer</summary>Tools like standard-version or release-please parse commit types: `fix:` commits trigger a PATCH bump, `feat:` commits trigger a MINOR bump, and commits with `BREAKING CHANGE` trigger a MAJOR bump. This removes manual guesswork from versioning.</details>

---

[← Previous: Branch Protection Rules](05-branch-protection-rules.md) | [Back to README](../README.md) | [Next: Issues and Project Boards →](../08-GitHub-GitLab-Features/01-issues-and-project-boards.md)
