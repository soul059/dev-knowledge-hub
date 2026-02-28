# Chapter 8.1: Issues and Project Boards

[← Previous: Semantic Versioning](../07-Collaboration/06-semantic-versioning.md) | [Back to README](../README.md) | [Next: GitHub Actions Basics →](02-github-actions-basics.md)

---

## Overview

**Issues** are the primary way to track bugs, feature requests, and tasks in GitHub/GitLab. **Project boards** organize issues into visual workflows (like Kanban boards) for tracking progress. Together, they form a lightweight project management system built into your repository.

---

## GitHub Issues

### Creating Issues

```
┌──────────────────────────────────────────────────────────────┐
│  NEW ISSUE                                                  │
│                                                              │
│  Title: Login page crashes on mobile Safari                 │
│                                                              │
│  Description:                                               │
│  ┌────────────────────────────────────────────────────────┐  │
│  │ ## Bug Report                                          │  │
│  │                                                        │  │
│  │ **Describe the bug:**                                  │  │
│  │ App crashes when tapping login button on Safari iOS 17 │  │
│  │                                                        │  │
│  │ **Steps to reproduce:**                                │  │
│  │ 1. Open app on iPhone Safari                           │  │
│  │ 2. Enter credentials                                   │  │
│  │ 3. Tap "Login"                                         │  │
│  │ 4. Page becomes unresponsive                           │  │
│  │                                                        │  │
│  │ **Expected behavior:**                                 │  │
│  │ User should be redirected to dashboard                 │  │
│  │                                                        │  │
│  │ **Environment:** iOS 17, Safari, iPhone 15             │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  Assignees:    @alice                                       │
│  Labels:       bug, priority:high, mobile                   │
│  Milestone:    v1.2.0                                       │
│  Project:      Sprint 5                                     │
└──────────────────────────────────────────────────────────────┘
```

### Issue Templates

```markdown
<!-- .github/ISSUE_TEMPLATE/bug_report.md -->
---
name: Bug Report
about: Report a bug to help us improve
title: "[BUG] "
labels: bug
assignees: ''
---

## Describe the Bug
<!-- A clear description of the bug -->

## Steps to Reproduce
1. Go to '...'
2. Click on '...'
3. See error

## Expected Behavior
<!-- What should happen -->

## Screenshots
<!-- If applicable -->

## Environment
- OS: [e.g., Windows 11]
- Browser: [e.g., Chrome 120]
- Version: [e.g., v1.2.0]
```

---

## Labels

```
┌──────────────────────────────────────────────────────────────┐
│  COMMON LABEL CATEGORIES                                    │
│                                                              │
│  TYPE:                                                      │
│  ┌──────┐ ┌─────────┐ ┌──────────────┐ ┌──────┐           │
│  │ bug  │ │ feature │ │ enhancement  │ │ docs │           │
│  └──────┘ └─────────┘ └──────────────┘ └──────┘           │
│                                                              │
│  PRIORITY:                                                  │
│  ┌────────────────┐ ┌──────────────┐ ┌─────────────┐       │
│  │ priority:high  │ │ priority:med │ │ priority:low│       │
│  └────────────────┘ └──────────────┘ └─────────────┘       │
│                                                              │
│  STATUS:                                                    │
│  ┌─────────────┐ ┌─────────────┐ ┌──────────────────┐      │
│  │ in-progress │ │ needs-review│ │ waiting-feedback │      │
│  └─────────────┘ └─────────────┘ └──────────────────┘      │
│                                                              │
│  SCOPE:                                                     │
│  ┌──────────┐ ┌──────────┐ ┌─────┐ ┌───────────────┐      │
│  │ frontend │ │ backend  │ │ api │ │ infrastructure│      │
│  └──────────┘ └──────────┘ └─────┘ └───────────────┘      │
│                                                              │
│  SPECIAL:                                                   │
│  ┌──────────────────┐ ┌──────────────┐ ┌──────────┐        │
│  │ good-first-issue │ │ help-wanted  │ │ wontfix  │        │
│  └──────────────────┘ └──────────────┘ └──────────┘        │
└──────────────────────────────────────────────────────────────┘
```

---

## Milestones

```
┌──────────────────────────────────────────────────────────────┐
│  MILESTONE: v2.0.0                                          │
│  Due date: March 15, 2025                                   │
│                                                              │
│  Progress: ████████░░░░░░░░  8/15 issues closed (53%)       │
│                                                              │
│  Open Issues:                                               │
│  • #42 Add OAuth2 support           (feature, in-progress)  │
│  • #45 Redesign settings page       (enhancement)           │
│  • #48 Migrate to new API format    (breaking-change)       │
│  • #51 Update documentation         (docs)                  │
│  • ...                                                      │
│                                                              │
│  Milestones group related issues toward a release goal.     │
└──────────────────────────────────────────────────────────────┘
```

---

## GitHub Projects (Project Boards)

```
┌──────────────────────────────────────────────────────────────┐
│  PROJECT BOARD (Kanban View)                                │
│                                                              │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐        │
│  │  📋 Backlog   │ │  🔄 In Prog  │ │  ✅ Done      │        │
│  │              │ │              │ │              │        │
│  │ #42 OAuth    │ │ #38 Search   │ │ #35 Login    │        │
│  │ #45 Settings │ │ #40 API v2   │ │ #36 Signup   │        │
│  │ #48 Migrate  │ │              │ │ #37 Profile  │        │
│  │ #51 Docs     │ │              │ │ #39 Tests    │        │
│  │ #52 Tests    │ │              │ │              │        │
│  │              │ │              │ │              │        │
│  └──────────────┘ └──────────────┘ └──────────────┘        │
│                                                              │
│  GitHub Projects V2 supports:                               │
│  • Table view, Board view, Roadmap view                     │
│  • Custom fields (priority, size, sprint)                   │
│  • Automation (when PR merged → Done)                       │
│  • Cross-repo tracking                                      │
└──────────────────────────────────────────────────────────────┘
```

### GitHub Projects V2 Custom Fields

```
┌──────────────────────────────────────────────────────────────┐
│  TABLE VIEW WITH CUSTOM FIELDS                              │
│                                                              │
│  Title          Status     Priority  Size  Sprint  Assignee │
│  ─────────────  ─────────  ────────  ────  ──────  ──────── │
│  #42 OAuth      In Prog    High      L     S5      @alice   │
│  #45 Settings   Backlog    Medium    M     S6      @bob     │
│  #48 Migrate    Backlog    High      XL    S6      @alice   │
│  #51 Docs       Backlog    Low       S     S7      @carol   │
│                                                              │
│  Filter, sort, and group by any field!                      │
└──────────────────────────────────────────────────────────────┘
```

---

## GitLab Issues & Boards

```
┌──────────────────────────────────────────────────────────────┐
│  GITLAB EQUIVALENTS                                         │
│                                                              │
│  GitHub               GitLab                                │
│  ──────               ──────                                │
│  Issues               Issues                                │
│  Projects (boards)    Issue Boards                          │
│  Milestones           Milestones                            │
│  Labels               Labels (scoped: priority::high)       │
│  Projects V2          Epics (Premium)                       │
│  Discussions           Threads                              │
│                                                              │
│  GitLab extras:                                             │
│  • Issue weights (effort estimation)                        │
│  • Time tracking (/spend 2h, /estimate 4h)                  │
│  • Scoped labels (only one per scope)                       │
│  • Epics for grouping issues (Premium)                      │
│  • Roadmaps (Premium)                                       │
└──────────────────────────────────────────────────────────────┘
```

---

## Linking Issues to Code

```bash
# Reference an issue in a commit message
$ git commit -m "fix(auth): handle expired tokens

Fixes #42"

# Link in PR description
Closes #42, closes #45

# Reference without closing
See #42 for context
Related to #45
```

```
┌──────────────────────────────────────────────────────────────┐
│  ISSUE → CODE → MERGE FLOW                                 │
│                                                              │
│  Issue #42 ──► Branch fix/issue-42 ──► PR ──► Merge        │
│  (opened)      (code changes)         (review)  (auto-close)│
│                                                              │
│  Keywords that auto-close issues on merge:                  │
│  close, closes, closed, fix, fixes, fixed,                  │
│  resolve, resolves, resolved                                │
└──────────────────────────────────────────────────────────────┘
```

---

## Automation with GitHub Actions

```yaml
# Auto-label issues based on title/body
# Auto-move issues on project board
# Auto-assign issues to team members

# Example: Auto-add issues to project board
name: Add to Project
on:
  issues:
    types: [opened]
jobs:
  add-to-project:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/add-to-project@v0.5.0
        with:
          project-url: https://github.com/orgs/myorg/projects/1
          github-token: ${{ secrets.PROJECT_TOKEN }}
```

---

## Real-World Scenarios

### Scenario 1: Bug Tracking Workflow

```
1. User reports bug → Issue created with bug template
2. Triager adds labels: bug, priority:high, frontend
3. Issue added to Sprint 5 project board (Backlog)
4. Developer assigns themselves, moves to In Progress
5. Creates branch: fix/issue-42-login-crash
6. Opens PR: "fix(auth): handle Safari token storage. Fixes #42"
7. PR merged → Issue auto-closed → Board moves to Done
```

### Scenario 2: Feature Planning

```
1. PM creates milestone: v2.0.0 (due March 15)
2. Creates feature issues: #50-#60
3. All issues assigned to v2.0.0 milestone
4. Project board tracks sprint-by-sprint progress
5. Milestone page shows completion percentage
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Issue not auto-closing on merge | Use keywords `Closes #N` in PR description (not just commits) |
| Too many stale issues | Use stale bot to auto-close inactive issues |
| Labels are inconsistent | Standardize labels across repos; document in CONTRIBUTING.md |
| Board doesn't reflect reality | Set up automation rules; do regular board grooming |
| Can't find relevant issues | Use filters: `is:open label:bug assignee:@me milestone:v2.0` |

---

## Summary Table

| Feature | GitHub | GitLab |
|---------|--------|--------|
| Bug/task tracking | Issues | Issues |
| Templates | Issue Templates | Description Templates |
| Categorization | Labels | Labels (+ scoped labels) |
| Release grouping | Milestones | Milestones |
| Visual boards | Projects V2 | Issue Boards |
| Effort tracking | — | Issue Weights |
| Time tracking | — | /spend, /estimate |
| Epic grouping | — | Epics (Premium) |
| Auto-close | Closes #N | Closes #N |

---

## Quick Revision Questions

1. **What are GitHub Issues used for?**
   <details><summary>Answer</summary>Issues are used to track bugs, feature requests, tasks, and discussions within a repository. They support labels, milestones, assignees, and templates, and can be linked to PRs and commits for traceability from request to code change.</details>

2. **How do you auto-close an issue when a PR is merged?**
   <details><summary>Answer</summary>Use closing keywords in the PR description: `Closes #42`, `Fixes #42`, or `Resolves #42`. When the PR is merged into the default branch, the referenced issue is automatically closed.</details>

3. **What is the difference between Milestones and Projects?**
   <details><summary>Answer</summary>Milestones group issues toward a specific release goal with a due date and completion percentage. Projects (boards) provide visual workflow management (Kanban boards, tables) with custom fields, automation, and cross-repo tracking. They serve different but complementary purposes.</details>

4. **What extra features does GitLab offer for issue tracking?**
   <details><summary>Answer</summary>GitLab adds issue weights (effort estimation), built-in time tracking (`/spend`, `/estimate` commands), scoped labels (only one per scope, e.g., `priority::high`), and Epics for grouping related issues across projects (Premium tier).</details>

5. **What is a `good-first-issue` label?**
   <details><summary>Answer</summary>A label used to mark issues that are beginner-friendly and suitable for new contributors. GitHub highlights these in the "Contribute" section. Other similar labels include `help-wanted` for issues needing community help.</details>

---

[← Previous: Semantic Versioning](../07-Collaboration/06-semantic-versioning.md) | [Back to README](../README.md) | [Next: GitHub Actions Basics →](02-github-actions-basics.md)
