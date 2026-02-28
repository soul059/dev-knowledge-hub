# Chapter 7.2: Code Reviews

[← Previous: Pull Requests](01-pull-requests.md) | [Back to README](../README.md) | [Next: Resolving Conflicts in Teams →](03-resolving-conflicts-in-teams.md)

---

## Overview

**Code review** is the systematic examination of source code by peers before it is merged. It catches bugs, improves code quality, shares knowledge across the team, and ensures adherence to standards. In Git-based workflows, code reviews happen through **Pull Requests / Merge Requests**.

---

## Why Code Reviews Matter

```
┌──────────────────────────────────────────────────────────────┐
│       BENEFITS OF CODE REVIEW                               │
│                                                              │
│  ┌─────────────────┐  ┌──────────────────┐                  │
│  │  QUALITY         │  │  KNOWLEDGE       │                  │
│  │  • Catch bugs    │  │  • Share context  │                  │
│  │  • Find edge     │  │  • Learn patterns │                  │
│  │    cases          │  │  • Mentor juniors │                  │
│  │  • Improve       │  │  • Reduce bus     │                  │
│  │    design         │  │    factor         │                  │
│  └─────────────────┘  └──────────────────┘                  │
│                                                              │
│  ┌─────────────────┐  ┌──────────────────┐                  │
│  │  CONSISTENCY     │  │  SECURITY        │                  │
│  │  • Style guide   │  │  • Spot vulns    │                  │
│  │  • Architecture  │  │  • Auth checks   │                  │
│  │  • Naming        │  │  • Input valid.  │                  │
│  │  • Best          │  │  • Injection     │                  │
│  │    practices      │  │    prevention    │                  │
│  └─────────────────┘  └──────────────────┘                  │
└──────────────────────────────────────────────────────────────┘
```

---

## Code Review Workflow

```
┌──────────────────────────────────────────────────────────────┐
│  CODE REVIEW PROCESS                                        │
│                                                              │
│  Author                       Reviewer                      │
│  ──────                       ────────                      │
│  1. Open PR ──────────────►   2. Read description            │
│                               3. Review diff line by line    │
│                               4. Leave comments / questions  │
│                               5. ┌─────────────────────┐    │
│                                  │ Decision:            │    │
│  ◄──────────────────────         │ • Approve ✅          │    │
│  6. Address feedback             │ • Request Changes 🔄 │    │
│  7. Push new commits             │ • Comment 💬         │    │
│  8. Re-request review            └─────────────────────┘    │
│  ──────────────────────►                                    │
│                               9. Re-review changes          │
│                              10. Approve ✅                  │
│ 11. Merge PR ✅                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## Types of Review Comments

```
┌──────────────────────────────────────────────────────────────┐
│  COMMENT TYPES                                              │
│                                                              │
│  🐛 Bug        — "This will throw a NullPointerException    │
│                   when user is null"                         │
│                                                              │
│  💡 Suggestion — "Consider using a Map here for O(1) lookup │
│                   instead of iterating the list"            │
│                                                              │
│  ❓ Question   — "What's the reason for this approach?      │
│                   Would X be simpler?"                       │
│                                                              │
│  ✏️  Nit       — "Typo in variable name: 'usre' → 'user'"  │
│                   (Minor, non-blocking)                      │
│                                                              │
│  🔒 Security   — "User input is not sanitized here —        │
│                   potential SQL injection"                   │
│                                                              │
│  📝 Praise     — "Nice refactoring! Much cleaner now."      │
└──────────────────────────────────────────────────────────────┘
```

---

## What to Look For

### Reviewer Checklist

| Area | What to Check |
|------|---------------|
| **Correctness** | Does the code do what it claims? Edge cases handled? |
| **Design** | Right abstractions? Good separation of concerns? |
| **Readability** | Clear variable/function names? Comments where needed? |
| **Testing** | Tests cover new code? Edge cases tested? |
| **Performance** | Efficient algorithms? N+1 queries? Memory leaks? |
| **Security** | Input validation? Auth checks? Sensitive data exposed? |
| **Style** | Follows project conventions? Consistent formatting? |
| **Documentation** | Updated README/docs if behavior changed? |

---

## Best Practices for Authors

```
┌──────────────────────────────────────────────────────────────┐
│  AUTHOR BEST PRACTICES                                      │
│                                                              │
│  BEFORE submitting PR:                                      │
│  ✓ Self-review your own diff first                          │
│  ✓ Run tests locally                                        │
│  ✓ Keep PRs small and focused (< 400 lines ideal)           │
│  ✓ Write a clear description                                │
│  ✓ Add context: why, not just what                          │
│                                                              │
│  DURING review:                                             │
│  ✓ Respond to ALL comments                                  │
│  ✓ Don't take feedback personally                           │
│  ✓ Explain reasoning, don't just defend                     │
│  ✓ Mark resolved comments as resolved                       │
│  ✓ Push fixes as new commits (easier to re-review)          │
│                                                              │
│  PR SIZE GUIDE:                                             │
│  < 200 lines — Easy to review      ✅                       │
│  200-400 lines — Manageable        ✅                       │
│  400-800 lines — Getting large     ⚠️                       │
│  > 800 lines — Too large, split!   ❌                       │
└──────────────────────────────────────────────────────────────┘
```

---

## Best Practices for Reviewers

```
┌──────────────────────────────────────────────────────────────┐
│  REVIEWER BEST PRACTICES                                    │
│                                                              │
│  ✓ Review promptly (within 24 hours)                        │
│  ✓ Be constructive, not critical                            │
│  ✓ Suggest alternatives, don't just say "this is wrong"     │
│  ✓ Distinguish blocking vs non-blocking comments            │
│  ✓ Approve when "good enough" — perfection isn't the goal   │
│  ✓ Praise good code, not just flag problems                 │
│  ✓ Use "nit:" prefix for minor, non-blocking suggestions    │
│  ✓ Ask questions rather than make demands                   │
│                                                              │
│  INSTEAD OF:             SAY:                               │
│  "This is wrong"    →   "Have you considered X?"            │
│  "Why did you..."   →   "I'm curious about the reason..." │
│  "You should..."    →   "What about doing X because..."    │
└──────────────────────────────────────────────────────────────┘
```

---

## GitHub Review Features

```
┌──────────────────────────────────────────────────────────────┐
│  GITHUB CODE REVIEW FEATURES                                │
│                                                              │
│  • Line Comments — Click on a line number in the diff       │
│  • Suggestions — Propose exact code changes inline          │
│    ```suggestion                                            │
│    const user = getUser(id);                                │
│    ```                                                      │
│    (Author can apply the suggestion with one click)         │
│                                                              │
│  • Multi-line Comments — Select range of lines              │
│  • Review Summary — Submit all comments as one review       │
│  • Required Reviews — Branch protection rules               │
│  • CODEOWNERS — Auto-assign reviewers by file path          │
│  • Review Requests — Tag specific people to review          │
└──────────────────────────────────────────────────────────────┘
```

### CODEOWNERS File

```bash
# .github/CODEOWNERS

# Default reviewers for everything
*                   @team-lead

# Frontend code
src/frontend/**     @frontend-team

# Backend API
src/api/**          @backend-team

# Database migrations
db/migrations/**    @dba-team @backend-team

# Documentation
docs/**             @tech-writer
```

---

## Real-World Scenarios

### Scenario 1: Effective Review Comment

```
❌ Bad:  "This is inefficient."
✅ Good: "This loop has O(n²) complexity because of the nested
         find(). Consider building a lookup map first for O(n):
         
         ```suggestion
         const userMap = new Map(users.map(u => [u.id, u]));
         orders.forEach(o => {
           const user = userMap.get(o.userId);
         });
         ```"
```

### Scenario 2: Stacked Reviews

```
PR #1: Database schema changes    → Review & merge first
PR #2: API layer (depends on #1)  → Review after #1 merged
PR #3: Frontend (depends on #2)   → Review after #2 merged

Each PR is small and focused — easier to review!
```

---

## Troubleshooting Tips

| Problem | Solution |
|---------|----------|
| Reviews take too long | Keep PRs small (< 400 lines); set team SLA (e.g., 24h) |
| Nitpicking slows things down | Use linters/formatters for style; focus reviews on logic |
| Disagreements on approach | Have a brief discussion, involve a third reviewer if stuck |
| Reviewers missing context | Write thorough PR descriptions with motivation |
| Same issues recurring | Add to linting rules or create a team guidelines doc |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| Code Review | Peer examination of code before merging |
| Approve | Reviewer approves the changes |
| Request Changes | Reviewer asks for modifications |
| CODEOWNERS | File that auto-assigns reviewers by path |
| Suggestion | Inline code change proposal in GitHub |
| Nit | Minor, non-blocking review comment |
| Self-review | Author reviews their own diff before requesting review |
| Review SLA | Team agreement on review response time |

---

## Quick Revision Questions

1. **What are the main benefits of code review?**
   <details><summary>Answer</summary>Code review catches bugs, improves code quality, shares knowledge across the team (reducing bus factor), ensures consistency with coding standards, identifies security vulnerabilities, and serves as documentation of design decisions.</details>

2. **What is the ideal size for a Pull Request?**
   <details><summary>Answer</summary>Under 400 lines of changed code is ideal. PRs under 200 lines are easiest to review thoroughly. PRs over 800 lines should be split into smaller, focused PRs. Smaller PRs get faster, higher-quality reviews.</details>

3. **What is a CODEOWNERS file?**
   <details><summary>Answer</summary>A file (`.github/CODEOWNERS` on GitHub) that maps file paths to team members or teams who are automatically assigned as reviewers when those files are modified in a PR. It ensures the right people review the right code.</details>

4. **How should a reviewer phrase feedback constructively?**
   <details><summary>Answer</summary>Ask questions instead of making demands ("Have you considered X?" instead of "You should do X"). Provide alternatives with reasoning. Use "nit:" for minor issues. Praise good code. Distinguish blocking from non-blocking comments.</details>

5. **What's the difference between Approve, Request Changes, and Comment?**
   <details><summary>Answer</summary>Approve: The code is good to merge. Request Changes: Changes are needed before merging (blocks merge if required reviews are configured). Comment: Feedback without explicit approval or rejection — useful for questions or suggestions that don't block merging.</details>

---

[← Previous: Pull Requests](01-pull-requests.md) | [Back to README](../README.md) | [Next: Resolving Conflicts in Teams →](03-resolving-conflicts-in-teams.md)
