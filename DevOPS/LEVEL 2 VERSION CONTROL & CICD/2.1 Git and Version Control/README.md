# 📘 Git and Version Control — Complete Study Notes

> **Comprehensive study guide covering Git fundamentals, branching strategies, workflows, collaboration, and platform features.**

---

## Course Overview

This course provides an in-depth exploration of **Git**, the most widely used distributed version control system in modern software development. From foundational concepts to advanced techniques, these notes are designed to give you a strong conceptual understanding, hands-on command fluency, and real-world workflow expertise.

```
┌─────────────────────────────────────────────────────────────┐
│                  GIT & VERSION CONTROL                      │
│                    LEARNING ROADMAP                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   Unit 1          Unit 2          Unit 3          Unit 4    │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐  │
│  │ VCS     │───▶│  Git    │───▶│ Branch  │───▶│ Remote  │  │
│  │Concepts │    │ Basics  │    │& Merge  │    │  Repos  │  │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘  │
│       │                                            │        │
│       ▼                                            ▼        │
│   Unit 5          Unit 6          Unit 7          Unit 8    │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐  │
│  │  Git    │───▶│Advanced │───▶│ Collab- │───▶│ GitHub/ │  │
│  │Workflows│    │  Git    │    │oration  │    │ GitLab  │  │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- Basic command-line / terminal knowledge
- A text editor (VS Code recommended)
- Git installed on your system ([Download Git](https://git-scm.com/downloads))

---

## Complete Table of Contents

### Unit 1: Version Control Concepts
| # | Chapter | File |
|---|---------|------|
| 1.1 | What is Version Control? | [01-what-is-version-control.md](01-Version-Control-Concepts/01-what-is-version-control.md) |
| 1.2 | Centralized vs Distributed VCS | [02-centralized-vs-distributed.md](01-Version-Control-Concepts/02-centralized-vs-distributed.md) |
| 1.3 | Git History and Architecture | [03-git-history-and-architecture.md](01-Version-Control-Concepts/03-git-history-and-architecture.md) |
| 1.4 | Git Objects (blob, tree, commit, tag) | [04-git-objects.md](01-Version-Control-Concepts/04-git-objects.md) |

### Unit 2: Git Basics
| # | Chapter | File |
|---|---------|------|
| 2.1 | Repository Initialization | [01-repository-initialization.md](02-Git-Basics/01-repository-initialization.md) |
| 2.2 | Staging and Committing | [02-staging-and-committing.md](02-Git-Basics/02-staging-and-committing.md) |
| 2.3 | Viewing History (log, show, diff) | [03-viewing-history.md](02-Git-Basics/03-viewing-history.md) |
| 2.4 | .gitignore Configuration | [04-gitignore-configuration.md](02-Git-Basics/04-gitignore-configuration.md) |
| 2.5 | Git Configuration | [05-git-configuration.md](02-Git-Basics/05-git-configuration.md) |

### Unit 3: Branching and Merging
| # | Chapter | File |
|---|---------|------|
| 3.1 | Branch Concepts | [01-branch-concepts.md](03-Branching-and-Merging/01-branch-concepts.md) |
| 3.2 | Creating and Switching Branches | [02-creating-and-switching-branches.md](03-Branching-and-Merging/02-creating-and-switching-branches.md) |
| 3.3 | Merging Strategies (Fast-Forward, 3-Way) | [03-merging-strategies.md](03-Branching-and-Merging/03-merging-strategies.md) |
| 3.4 | Merge Conflicts Resolution | [04-merge-conflicts-resolution.md](03-Branching-and-Merging/04-merge-conflicts-resolution.md) |
| 3.5 | Rebasing vs Merging | [05-rebasing-vs-merging.md](03-Branching-and-Merging/05-rebasing-vs-merging.md) |
| 3.6 | Cherry-Picking | [06-cherry-picking.md](03-Branching-and-Merging/06-cherry-picking.md) |

### Unit 4: Remote Repositories
| # | Chapter | File |
|---|---------|------|
| 4.1 | Remote Concepts | [01-remote-concepts.md](04-Remote-Repositories/01-remote-concepts.md) |
| 4.2 | Cloning Repositories | [02-cloning-repositories.md](04-Remote-Repositories/02-cloning-repositories.md) |
| 4.3 | Pushing and Pulling | [03-pushing-and-pulling.md](04-Remote-Repositories/03-pushing-and-pulling.md) |
| 4.4 | Fetch vs Pull | [04-fetch-vs-pull.md](04-Remote-Repositories/04-fetch-vs-pull.md) |
| 4.5 | Tracking Branches | [05-tracking-branches.md](04-Remote-Repositories/05-tracking-branches.md) |
| 4.6 | Multiple Remotes | [06-multiple-remotes.md](04-Remote-Repositories/06-multiple-remotes.md) |

### Unit 5: Git Workflows
| # | Chapter | File |
|---|---------|------|
| 5.1 | Feature Branch Workflow | [01-feature-branch-workflow.md](05-Git-Workflows/01-feature-branch-workflow.md) |
| 5.2 | Gitflow Workflow | [02-gitflow-workflow.md](05-Git-Workflows/02-gitflow-workflow.md) |
| 5.3 | GitHub Flow | [03-github-flow.md](05-Git-Workflows/03-github-flow.md) |
| 5.4 | Trunk-Based Development | [04-trunk-based-development.md](05-Git-Workflows/04-trunk-based-development.md) |
| 5.5 | Choosing the Right Workflow | [05-choosing-the-right-workflow.md](05-Git-Workflows/05-choosing-the-right-workflow.md) |

### Unit 6: Advanced Git
| # | Chapter | File |
|---|---------|------|
| 6.1 | Interactive Rebase | [01-interactive-rebase.md](06-Advanced-Git/01-interactive-rebase.md) |
| 6.2 | Stashing Changes | [02-stashing-changes.md](06-Advanced-Git/02-stashing-changes.md) |
| 6.3 | Git Bisect | [03-git-bisect.md](06-Advanced-Git/03-git-bisect.md) |
| 6.4 | Git Hooks | [04-git-hooks.md](06-Advanced-Git/04-git-hooks.md) |
| 6.5 | Submodules and Subtrees | [05-submodules-and-subtrees.md](06-Advanced-Git/05-submodules-and-subtrees.md) |
| 6.6 | Git LFS (Large File Storage) | [06-git-lfs.md](06-Advanced-Git/06-git-lfs.md) |

### Unit 7: Collaboration
| # | Chapter | File |
|---|---------|------|
| 7.1 | Pull Requests / Merge Requests | [01-pull-requests.md](07-Collaboration/01-pull-requests.md) |
| 7.2 | Code Review Best Practices | [02-code-review-best-practices.md](07-Collaboration/02-code-review-best-practices.md) |
| 7.3 | Branch Protection Rules | [03-branch-protection-rules.md](07-Collaboration/03-branch-protection-rules.md) |
| 7.4 | CODEOWNERS | [04-codeowners.md](07-Collaboration/04-codeowners.md) |
| 7.5 | Commit Message Conventions | [05-commit-message-conventions.md](07-Collaboration/05-commit-message-conventions.md) |
| 7.6 | Semantic Versioning | [06-semantic-versioning.md](07-Collaboration/06-semantic-versioning.md) |

### Unit 8: GitHub/GitLab Features
| # | Chapter | File |
|---|---------|------|
| 8.1 | Issues and Project Boards | [01-issues-and-project-boards.md](08-GitHub-GitLab-Features/01-issues-and-project-boards.md) |
| 8.2 | Actions / CI Pipelines | [02-actions-ci-pipelines.md](08-GitHub-GitLab-Features/02-actions-ci-pipelines.md) |
| 8.3 | Wikis and Documentation | [03-wikis-and-documentation.md](08-GitHub-GitLab-Features/03-wikis-and-documentation.md) |
| 8.4 | Releases and Tags | [04-releases-and-tags.md](08-GitHub-GitLab-Features/04-releases-and-tags.md) |
| 8.5 | Security Features | [05-security-features.md](08-GitHub-GitLab-Features/05-security-features.md) |
| 8.6 | Integration with Other Tools | [06-integration-with-other-tools.md](08-GitHub-GitLab-Features/06-integration-with-other-tools.md) |

---

## How to Use These Notes

1. **Sequential Study** — Follow the units in order for a structured learning path
2. **Quick Reference** — Jump to any specific chapter using the TOC links
3. **Hands-On Practice** — Try every command example in a test repository
4. **Revision** — Use the summary tables and quick revision questions at the end of each chapter

---

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────────┐
│                  ESSENTIAL GIT COMMANDS                      │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SETUP              │  BRANCHING          │  REMOTE         │
│  git init           │  git branch         │  git clone      │
│  git config         │  git checkout       │  git remote     │
│  git clone          │  git switch         │  git fetch      │
│                     │  git merge          │  git pull       │
│  BASIC              │  git rebase         │  git push       │
│  git add            │                     │                 │
│  git commit         │  INSPECT            │  UNDO           │
│  git status         │  git log            │  git reset      │
│  git diff           │  git show           │  git revert     │
│  git rm             │  git diff           │  git checkout   │
│                     │  git blame          │  git stash      │
└─────────────────────────────────────────────────────────────┘
```

---

## License

These notes are for educational purposes. Feel free to share and adapt for personal learning.

---

> **Start your journey →** [Unit 1: What is Version Control?](01-Version-Control-Concepts/01-what-is-version-control.md)
