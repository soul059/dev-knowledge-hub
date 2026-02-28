# Chapter 5.1: Clean Code Principles

```
╔══════════════════════════════════════════════════════════╗
║          CLEAN CODE PRINCIPLES                           ║
║   Writing interview code that impresses                  ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

Clean code isn't just for production — it's a **signal of engineering quality** in interviews. Interviewers evaluate your code style, organization, and clarity. Writing clean code during interviews shows you'll write maintainable code on the job.

---

## The 5 Pillars of Clean Interview Code

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  1. READABLE — Can someone understand it in 30 seconds? │
│  2. ORGANIZED — Logical flow, clear structure           │
│  3. CONCISE — No unnecessary code, but not cryptic      │
│  4. CORRECT — Handles edge cases, no bugs               │
│  5. IDIOMATIC — Uses language features appropriately    │
│                                                          │
│       ┌──────────┐                                      │
│       │ READABLE │                                      │
│       └─────┬────┘                                      │
│    ┌────────┴────────┐                                  │
│    │               │                                    │
│  ┌─┴──────┐  ┌─────┴────┐                              │
│  │ORGANIZED│  │ CONCISE  │                              │
│  └─┬──────┘  └─────┬────┘                              │
│    │               │                                    │
│  ┌─┴──────┐  ┌─────┴────┐                              │
│  │CORRECT │  │IDIOMATIC │                              │
│  └────────┘  └──────────┘                              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Clean Code Rules for Interviews

```
RULE 1: One Function, One Purpose
┌────────────────────────────────────┐
│  BAD:                              │
│  def solve(arr):                   │
│      # 50 lines doing everything  │
│                                    │
│  GOOD:                             │
│  def solve(arr):                   │
│      arr.sort()                    │
│      return find_pairs(arr)        │
│                                    │
│  def find_pairs(sorted_arr):       │
│      # focused logic               │
└────────────────────────────────────┘

RULE 2: Early Returns for Edge Cases
┌────────────────────────────────────┐
│  BAD:                              │
│  def solve(arr):                   │
│      if arr:                       │
│          # 40 lines nested inside │
│                                    │
│  GOOD:                             │
│  def solve(arr):                   │
│      if not arr:                   │
│          return []                  │
│      # main logic at base indent  │
└────────────────────────────────────┘

RULE 3: Meaningful Variable Names
┌────────────────────────────────────┐
│  BAD:  x, y, z, temp, flag, a, b │
│  GOOD: left, right, count,       │
│        max_sum, is_valid, node    │
└────────────────────────────────────┘

RULE 4: Consistent Style
┌────────────────────────────────────┐
│  Pick one style and stick with it: │
│  ├── camelCase OR snake_case      │
│  ├── Consistent indentation       │
│  ├── Consistent spacing           │
│  └── Consistent bracket style     │
└────────────────────────────────────┘

RULE 5: No Magic Numbers
┌────────────────────────────────────┐
│  BAD:  if count > 26:             │
│  GOOD: ALPHABET_SIZE = 26         │
│        if count > ALPHABET_SIZE:  │
│  (In interviews, a quick comment  │
│   is usually sufficient)          │
└────────────────────────────────────┘
```

---

## Before vs After: Real Examples

```
BEFORE (messy, hard to follow):
┌──────────────────────────────────────────────────────────┐
│  def f(a, t):                                           │
│      d = {}                                              │
│      for i in range(len(a)):                            │
│          x = t - a[i]                                    │
│          if x in d:                                      │
│              return [d[x], i]                            │
│          d[a[i]] = i                                     │
│      return []                                           │
└──────────────────────────────────────────────────────────┘

AFTER (clean, professional):
┌──────────────────────────────────────────────────────────┐
│  def two_sum(nums, target):                              │
│      seen = {}  # value -> index                        │
│                                                          │
│      for i, num in enumerate(nums):                     │
│          complement = target - num                       │
│          if complement in seen:                          │
│              return [seen[complement], i]                │
│          seen[num] = i                                   │
│                                                          │
│      return []  # no pair found                         │
└──────────────────────────────────────────────────────────┘
```

---

## Code Structure Template

```
def solve(input_params):
    # 1. Edge cases (early returns)
    if not input_params:
        return default_value
    
    # 2. Initialize data structures
    result = []
    seen = set()
    
    # 3. Main logic
    for element in input_params:
        # Process each element
        pass
    
    # 4. Return result
    return result
```

---

## Summary Table

| Principle | Bad Example | Good Example |
|-----------|------------|--------------|
| Naming | `d = {}` | `seen = {}` |
| Structure | Everything in one block | Edge cases → Init → Logic → Return |
| Nesting | Deep if/else nesting | Early returns, flat code |
| Comments | No comments or obvious ones | Brief comment explaining "why" |
| Functions | One 60-line function | Helper functions for sub-tasks |

---

## Quick Revision Questions

1. **What are the 5 pillars of clean interview code?**
   - Readable, Organized, Concise, Correct, Idiomatic

2. **Why does code quality matter in interviews?**
   - It signals engineering maturity and predicts how you'll write code on the job. Clean code is easier for interviewers to evaluate

3. **What's the "early return" pattern and why use it?**
   - Handle edge cases at the top with immediate returns, keeping the main logic at base indentation level — reduces nesting and improves readability

4. **How detailed should comments be in interview code?**
   - Brief and focused on "why" not "what." A short comment like `# value -> index mapping` is perfect. Don't comment every line

5. **Should you use helper functions in interview code?**
   - Yes, for non-trivial sub-operations. It shows code organization skills. But don't over-engineer — use helpers only when they genuinely improve clarity

6. **What's the standard code structure template?**
   - Edge cases → Initialize data structures → Main logic → Return result

---

[← Previous: Unit 4 — Common Complexity Patterns](../04-Time-Complexity-Discussion/05-common-complexity-patterns.md) | [Next: Variable Naming →](02-variable-naming.md)

---
[← Back to README](../README.md)
