# Chapter 5.5: Error Handling

```
╔══════════════════════════════════════════════════════════╗
║            ERROR HANDLING                                ║
║   Defensive coding in interview context                  ║
╚══════════════════════════════════════════════════════════╝
```

## Overview

While production-level error handling isn't expected in interviews, demonstrating **awareness of what can go wrong** shows engineering maturity. This chapter covers when and how to handle errors in interview code without over-engineering.

---

## Error Handling in Interviews vs Production

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  PRODUCTION CODE:                                        │
│  ├── Try-catch blocks everywhere                        │
│  ├── Custom exception classes                           │
│  ├── Logging, monitoring, alerting                      │
│  ├── Input validation at every boundary                 │
│  ├── Graceful degradation                               │
│  └── Recovery strategies                                │
│                                                          │
│  INTERVIEW CODE:                                         │
│  ├── Guard clauses for invalid input                    │
│  ├── Clear return values for "not found"                │
│  ├── Avoid crashing on edge cases                       │
│  ├── Mention what you'd add in production               │
│  └── Focus on algorithm correctness                     │
│                                                          │
│  KEY DIFFERENCE:                                         │
│  In interviews, handle the LIKELY errors in code.       │
│  MENTION additional handling you'd add in production.    │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## What to Handle in Interview Code

```
MUST HANDLE (in code):
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  1. NULL / NONE INPUTS                                  │
│     if root is None: return 0                           │
│     if not nums: return []                              │
│                                                          │
│  2. EMPTY COLLECTIONS                                   │
│     if len(arr) == 0: return -1                         │
│     if not graph: return []                             │
│                                                          │
│  3. BOUNDARY CONDITIONS                                 │
│     if index < 0 or index >= len(arr): return           │
│     if row < 0 or row >= rows: return                   │
│                                                          │
│  4. IMPOSSIBLE SCENARIOS                                │
│     if k > len(arr): return []  # can't find k-th      │
│     if target < 0: return -1   # impossible             │
│                                                          │
└──────────────────────────────────────────────────────────┘

MENTION VERBALLY (don't code):
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  "In production, I'd also add:"                         │
│  ├── Type checking / input validation                   │
│  ├── Try-catch for potential exceptions                 │
│  ├── Logging for debugging                              │
│  ├── Rate limiting for API calls                        │
│  └── Timeout handling for long operations               │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Return Values for Error States

```
CONVENTIONS BY RETURN TYPE:

  SEARCHING (not found):
  ├── Return -1  (index-based: "Find position of X")
  ├── Return None/null (object-based: "Find the node")
  └── Return empty list [] ("Find all matching elements")

  BOOLEAN:
  ├── Return False ("Is the tree valid?")
  └── Return True only when condition is met

  NUMERIC:
  ├── Return 0 ("Count the pairs" → none found)
  ├── Return float('inf') ("Find minimum cost" → impossible)
  └── Return -1 ("Find maximum" → no valid answer)

  COLLECTIONS:
  ├── Return [] ("Find all paths" → no paths exist)
  └── Return {} ("Build frequency map" → empty input)

EXAMPLE:
┌──────────────────────────────────────────────────────────┐
│  def binary_search(nums, target):                        │
│      if not nums:                                        │
│          return -1  ← Clear "not found" signal          │
│                                                          │
│      lo, hi = 0, len(nums) - 1                          │
│      while lo <= hi:                                     │
│          mid = lo + (hi - lo) // 2                      │
│          if nums[mid] == target:                        │
│              return mid                                  │
│          elif nums[mid] < target:                       │
│              lo = mid + 1                                │
│          else:                                           │
│              hi = mid - 1                                │
│                                                          │
│      return -1  ← Not found after full search           │
└──────────────────────────────────────────────────────────┘
```

---

## Defensive Coding Patterns

```
PATTERN 1: Null-safe Tree Traversal
┌──────────────────────────────────────┐
│  def traverse(node):                 │
│      if node is None:  ← Guard      │
│          return                       │
│      traverse(node.left)             │
│      process(node.val)               │
│      traverse(node.right)            │
└──────────────────────────────────────┘

PATTERN 2: Safe Dictionary Access
┌──────────────────────────────────────┐
│  # Risky:                            │
│  count = frequency[key]  ← KeyError │
│                                      │
│  # Safe options:                     │
│  count = frequency.get(key, 0)      │
│  # OR                                │
│  from collections import defaultdict│
│  frequency = defaultdict(int)       │
└──────────────────────────────────────┘

PATTERN 3: Bounds Checking
┌──────────────────────────────────────┐
│  def is_valid(r, c, grid):           │
│      return (0 <= r < len(grid) and │
│              0 <= c < len(grid[0]))  │
│                                      │
│  # Use before grid access:          │
│  if is_valid(nr, nc, grid):          │
│      process(grid[nr][nc])           │
└──────────────────────────────────────┘
```

---

## What Interviewers Want to See

```
┌──────────────────────────────────────────────────────────┐
│  SHOWS ENGINEERING MATURITY:                             │
│                                                          │
│  ✓ Handles null/empty inputs without crashing           │
│  ✓ Uses clear sentinel values (-1, None, [])            │
│  ✓ Checks bounds before array access                    │
│  ✓ Mentions production considerations verbally          │
│  ✓ Doesn't over-engineer (no try-catch for everything) │
│                                                          │
│  RED FLAGS:                                              │
│                                                          │
│  ✗ Code crashes on empty input                          │
│  ✗ ArrayIndexOutOfBounds exceptions                     │
│  ✗ NullPointerException on tree traversal               │
│  ✗ Never considers what could go wrong                  │
│  ✗ Spends 5 minutes on elaborate error handling         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Situation | Handle in Code | Mention Verbally |
|-----------|---------------|-----------------|
| Null/empty input | Yes — guard clause | — |
| Out of bounds | Yes — bounds check | — |
| Not found result | Yes — return -1/None | — |
| Type validation | No | "In production, I'd validate types" |
| Exception handling | Rarely | "I'd add try-catch in production" |
| Logging | No | "I'd log errors in production" |

---

## Quick Revision Questions

1. **Should you write try-catch blocks in interview code?**
   - Generally no. Use guard clauses and null checks instead. Mention try-catch as something you'd add in production

2. **What return value should you use when an element isn't found?**
   - Convention: -1 for index searches, None for object searches, [] for collection searches, False for boolean checks

3. **What's the difference between interview and production error handling?**
   - Interviews: Handle likely errors (null, empty, bounds). Production: Full validation, logging, custom exceptions, recovery strategies

4. **How do you safely access a dictionary key?**
   - Use `dict.get(key, default)` or `collections.defaultdict` instead of direct `dict[key]` access which can raise KeyError

5. **What's the biggest error-handling red flag in interviews?**
   - Code that crashes on empty input or null pointers — shows you didn't think about edge cases

6. **How much time should you spend on error handling?**
   - 30 seconds to add guard clauses. Don't spend 5 minutes on elaborate error handling — focus on the algorithm

---

[← Previous: Edge Case Handling](04-edge-case-handling.md) | [Next: Code Readability →](06-code-readability.md)

---
[← Back to README](../README.md)
