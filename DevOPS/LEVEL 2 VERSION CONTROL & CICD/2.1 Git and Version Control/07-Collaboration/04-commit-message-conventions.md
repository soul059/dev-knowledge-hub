# Chapter 7.4: Commit Message Conventions

[← Previous: Resolving Conflicts in Teams](03-resolving-conflicts-in-teams.md) | [Back to README](../README.md) | [Next: Branch Protection Rules →](05-branch-protection-rules.md)

---

## Overview

Good commit messages are crucial for project maintainability. They serve as a **changelog**, help with **debugging** (`git bisect`, `git blame`), and make **code review** more effective. This chapter covers widely adopted commit message conventions.

---

## Anatomy of a Good Commit Message

```
┌──────────────────────────────────────────────────────────────┐
│  COMMIT MESSAGE STRUCTURE                                   │
│                                                              │
│  <type>(<scope>): <subject>        ← Header (max 72 chars) │
│                                     ← Blank line            │
│  <body>                            ← Body (optional)        │
│  Explain WHAT and WHY, not HOW.     ← Wrap at 72 chars      │
│                                     ← Blank line            │
│  <footer>                          ← Footer (optional)      │
│  Closes #42                        ← References, breaking   │
│  BREAKING CHANGE: ...               ← changes               │
│                                                              │
│  EXAMPLE:                                                   │
│  ────────────────────────────────────────────                │
│  feat(auth): add JWT refresh token support                  │
│                                                              │
│  Implement automatic token refresh when the access          │
│  token expires. Refresh tokens are stored in HTTP-only      │
│  cookies for security.                                      │
│                                                              │
│  Closes #42                                                 │
└──────────────────────────────────────────────────────────────┘
```

---

## Conventional Commits

The **Conventional Commits** specification is the most widely adopted standard.

### Commit Types

| Type | Description | Example |
|------|-------------|---------|
| `feat` | New feature | `feat: add search functionality` |
| `fix` | Bug fix | `fix: prevent crash on empty input` |
| `docs` | Documentation only | `docs: update API reference` |
| `style` | Formatting, no logic change | `style: fix indentation in utils` |
| `refactor` | Code change, no new feature/fix | `refactor: extract validation logic` |
| `perf` | Performance improvement | `perf: cache database queries` |
| `test` | Adding/updating tests | `test: add unit tests for auth` |
| `build` | Build system or dependencies | `build: upgrade webpack to v5` |
| `ci` | CI/CD configuration | `ci: add GitHub Actions workflow` |
| `chore` | Other changes (no src/test) | `chore: update .gitignore` |
| `revert` | Reverting a commit | `revert: feat: add search (abc123)` |

---

## The Seven Rules of Great Commit Messages

```
┌──────────────────────────────────────────────────────────────┐
│  THE 7 RULES                                                │
│                                                              │
│  1. Separate subject from body with a blank line            │
│  2. Limit subject line to 72 characters                     │
│  3. Do NOT end the subject with a period                    │
│  4. Use imperative mood in the subject                      │
│     ✅ "Add feature"  ❌ "Added feature"                    │
│     ✅ "Fix bug"      ❌ "Fixes bug"                        │
│     ✅ "Change color"  ❌ "Changed color"                    │
│                                                              │
│  5. Wrap body at 72 characters                              │
│  6. Use body to explain WHAT and WHY (not HOW)              │
│  7. Reference issues and PRs in the footer                  │
│                                                              │
│  TIP: Read it as "This commit will <subject>"               │
│  "This commit will Add user authentication" ✅              │
│  "This commit will Added user authentication" ❌            │
└──────────────────────────────────────────────────────────────┘
```

---

## Good vs Bad Commit Messages

```
┌──────────────────────────────────────────────────────────────┐
│  ❌ BAD MESSAGES              ✅ GOOD MESSAGES               │
│  ──────────────              ──────────────               │
│  "fix"                       "fix(auth): handle expired    │
│                                session token"               │
│                                                              │
│  "update code"               "refactor(api): extract       │
│                                validation middleware"        │
│                                                              │
│  "WIP"                       "feat(cart): add quantity      │
│                                update endpoint"              │
│                                                              │
│  "changes"                   "fix(ui): correct button       │
│                                alignment on mobile"          │
│                                                              │
│  "review feedback"           "refactor(auth): use bcrypt    │
│                                instead of md5 per review"   │
│                                                              │
│  "asdfasdf"                  "docs: add contributing        │
│                                guidelines"                   │
└──────────────────────────────────────────────────────────────┘
```

---

## Breaking Changes

```bash
# Method 1: Footer notation
feat(api): change user endpoint response format

The /api/users endpoint now returns a paginated response
with metadata instead of a plain array.

BREAKING CHANGE: /api/users now returns { data: [], meta: {} }
instead of a plain array. Clients must update their parsing logic.

Closes #108

# Method 2: Exclamation mark in type
feat(api)!: change user endpoint response format
```

---

## Scopes

```
┌──────────────────────────────────────────────────────────────┐
│  COMMON SCOPES (project-specific)                           │
│                                                              │
│  By module:   auth, api, ui, db, config                     │
│  By layer:    controller, service, model, view              │
│  By feature:  cart, search, profile, payment                │
│  By tool:     webpack, docker, eslint, ci                   │
│                                                              │
│  Examples:                                                  │
│  feat(auth): add OAuth2 support                             │
│  fix(api): return 404 for missing resources                 │
│  test(cart): add integration tests for checkout             │
│  ci(docker): optimize multi-stage build                     │
└──────────────────────────────────────────────────────────────┘
```

---

## Automating Commit Conventions

### Commitlint (Enforce rules)

```bash
# Install
$ npm install --save-dev @commitlint/cli @commitlint/config-conventional

# Configure: commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'header-max-length': [2, 'always', 72],
    'type-enum': [2, 'always', [
      'feat', 'fix', 'docs', 'style', 'refactor',
      'perf', 'test', 'build', 'ci', 'chore', 'revert'
    ]]
  }
};
```

### Husky (Git hooks)

```bash
# Install
$ npm install --save-dev husky
$ npx husky init
$ echo "npx commitlint --edit \$1" > .husky/commit-msg
```

### Commitizen (Interactive commit helper)

```bash
$ npm install -g commitizen
$ commitizen init cz-conventional-changelog --save-dev

# Instead of git commit, use:
$ git cz
# Interactive prompt:
# ? Select type: feat
# ? Scope: auth
# ? Subject: add password reset flow
# ? Body: ...
# ? Breaking changes: N
# ? Issues: #42
```

---

## Auto-Generating Changelogs

```bash
# With standard-version or release-please:
$ npx standard-version

# Generates CHANGELOG.md from commit messages:
# ## [1.2.0] - 2025-01-15
# ### Features
# * **auth:** add OAuth2 support (#42)
# * **cart:** add quantity update endpoint (#45)
# ### Bug Fixes
# * **ui:** correct button alignment on mobile (#43)
# ### BREAKING CHANGES
# * **api:** change user endpoint response format (#108)
```

```
┌──────────────────────────────────────────────────────────────┐
│  CONVENTIONAL COMMITS → AUTOMATIC VERSIONING                │
│                                                              │
│  fix:  → PATCH bump  (1.0.0 → 1.0.1)                       │
│  feat: → MINOR bump  (1.0.0 → 1.1.0)                       │
│  BREAKING CHANGE → MAJOR bump (1.0.0 → 2.0.0)              │
│                                                              │
│  Good commit messages enable fully automatic releases!      │
└──────────────────────────────────────────────────────────────┘
```

---

## Real-World Scenarios

### Scenario 1: Typical Feature Development

```bash
$ git commit -m "feat(search): add full-text search for products"
$ git commit -m "test(search): add unit tests for search indexing"
$ git commit -m "docs(search): add search API documentation"
```

### Scenario 2: Bug Fix with Context

```bash
$ git commit -m "fix(auth): prevent token replay attack

Previously, expired tokens could be replayed within a 5-minute
window due to clock skew tolerance. This reduces the tolerance
to 30 seconds and adds a token blacklist for revoked tokens.

Fixes #87"
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Team not following conventions | Set up commitlint + husky for enforcement |
| Subject line too long | Keep under 72 chars; move details to body |
| Don't know which type to use | If it adds behavior → `feat`; if it fixes → `fix`; else `refactor` or `chore` |
| "WIP" commits in history | Use interactive rebase to clean up before merging |
| Forgot to reference issue | Amend the commit: `git commit --amend` |

---

## Summary Table

| Element | Rule |
|---------|------|
| Subject line | Max 72 chars, imperative mood, no period |
| Type | feat, fix, docs, style, refactor, perf, test, build, ci, chore |
| Scope | Optional, in parentheses: `feat(auth):` |
| Body | Explain what and why, wrap at 72 chars |
| Footer | Issue references, breaking changes |
| Breaking change | `BREAKING CHANGE:` in footer or `!` after type |

---

## Quick Revision Questions

1. **What are the most common Conventional Commit types?**
   <details><summary>Answer</summary>`feat` (new feature), `fix` (bug fix), `docs` (documentation), `style` (formatting), `refactor` (restructuring without behavior change), `perf` (performance), `test` (tests), `build` (build system), `ci` (CI/CD), `chore` (other maintenance).</details>

2. **What mood should the subject line use?**
   <details><summary>Answer</summary>Imperative mood — as if giving a command. "Add feature" not "Added feature". Tip: read it as "This commit will [subject]" — "This commit will add feature" works; "This commit will added feature" doesn't.</details>

3. **How do you indicate a breaking change in Conventional Commits?**
   <details><summary>Answer</summary>Two ways: 1) Add `BREAKING CHANGE:` in the commit footer with a description. 2) Add `!` after the type/scope: `feat(api)!: change response format`. Breaking changes trigger a major version bump in semantic versioning.</details>

4. **How can commit messages enable automatic versioning?**
   <details><summary>Answer</summary>Tools like `standard-version` or `release-please` parse conventional commits: `fix:` triggers a PATCH bump, `feat:` triggers a MINOR bump, and `BREAKING CHANGE` triggers a MAJOR bump. They also auto-generate changelogs from commit messages.</details>

5. **What tools can enforce commit message conventions?**
   <details><summary>Answer</summary>Commitlint validates commit messages against rules. Husky runs commitlint as a Git hook (commit-msg). Commitizen provides an interactive prompt to build conventional commits. Together they prevent non-conforming messages from being committed.</details>

---

[← Previous: Resolving Conflicts in Teams](03-resolving-conflicts-in-teams.md) | [Back to README](../README.md) | [Next: Branch Protection Rules →](05-branch-protection-rules.md)
