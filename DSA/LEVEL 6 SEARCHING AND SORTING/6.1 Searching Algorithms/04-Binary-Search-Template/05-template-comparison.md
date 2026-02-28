# Chapter 5: Template Comparison — Choosing the Right One

[← Previous: Common Mistakes](04-common-mistakes.md) | [Next: Practice Problems →](06-practice-problems.md)

---

## Overview

We've learned three templates. This chapter provides a **side-by-side comparison** and a **decision framework** to instantly pick the right one for any problem.

---

## The Three Templates at a Glance

```
   ╔════════════════════════════════════════════════════════════════════════╗
   ║  TEMPLATE 1: Standard (Exact Match)                                  ║
   ║  ─────────────────────────────────                                   ║
   ║  lo = 0, hi = n - 1                                                 ║
   ║  while (lo <= hi):                                                   ║
   ║      mid = lo + (hi - lo) / 2                                       ║
   ║      if arr[mid] == target: return mid                               ║
   ║      if arr[mid] < target: lo = mid + 1                             ║
   ║      else: hi = mid - 1                                              ║
   ║  return -1                                                           ║
   ╠════════════════════════════════════════════════════════════════════════╣
   ║  TEMPLATE 2: Boundary (Find First/Last True)                        ║
   ║  ───────────────────────────────────────────                         ║
   ║  lo = 0, hi = n - 1                                                 ║
   ║  result = -1                                                         ║
   ║  while (lo <= hi):                                                   ║
   ║      mid = lo + (hi - lo) / 2                                       ║
   ║      if condition(mid):                                              ║
   ║          result = mid      ← record candidate                       ║
   ║          hi = mid - 1      ← search left for FIRST                  ║
   ║          // or lo = mid+1  ← search right for LAST                  ║
   ║      else:                                                           ║
   ║          lo = mid + 1      ← for FIRST                              ║
   ║          // or hi = mid-1  ← for LAST                               ║
   ║  return result                                                       ║
   ╠════════════════════════════════════════════════════════════════════════╣
   ║  TEMPLATE 3: Shrinking Window                                        ║
   ║  ────────────────────────                                            ║
   ║  lo = 0, hi = n - 1                                                 ║
   ║  while (lo < hi):         ← strict inequality                       ║
   ║      mid = lo + (hi - lo) / 2    ← or right-biased                  ║
   ║      if condition(mid):                                              ║
   ║          hi = mid          ← keep mid in window                     ║
   ║      else:                                                           ║
   ║          lo = mid + 1      ← exclude mid                            ║
   ║  return lo                 ← lo == hi → answer                      ║
   ╚════════════════════════════════════════════════════════════════════════╝
```

---

## Comparison Table

| Feature | Template 1 | Template 2 | Template 3 |
|---------|-----------|-----------|-----------|
| **Goal** | Find exact match | Find first/last satisfying | Find boundary |
| **Loop** | `lo <= hi` | `lo <= hi` | `lo < hi` |
| **Returns** | Index or -1 | Candidate index or -1 | `lo` (convergence point) |
| **Space for answer** | Search range shrinks around target | Records candidates | Window shrinks to 1 |
| **When not found** | `lo > hi` (crossed) | `result` stays -1 | `lo == hi` (always) |
| **Risk of infinite loop** | None | None | Yes, if bias wrong |
| **Difficulty** | Easy | Medium | Medium |

---

## Decision Flowchart

```
   START: What do you need?
     │
     ├── "Does this specific value exist? Where?" 
     │   └── TEMPLATE 1 (Standard)
     │
     ├── "Find the first/last element satisfying X"
     │   └── TEMPLATE 2 (Boundary)
     │       ├── First → record + search left
     │       └── Last  → record + search right
     │
     ├── "Find the minimum/maximum value where condition changes"
     │   └── TEMPLATE 3 (Shrinking Window)
     │
     ├── "Search on answer space" (e.g., minimum capacity)
     │   └── TEMPLATE 3 (Shrinking Window)
     │       Use: lo < hi, condition(mid)
     │
     └── "Find peak/valley"
         └── TEMPLATE 3 (Shrinking Window)
             Compare mid with mid+1 for direction
```

---

## Problem → Template Mapping

| Problem | Template | Why |
|---------|----------|-----|
| Find target in sorted array | 1 | Exact match |
| First occurrence of `x` | 2 | Find first satisfying `arr[mid] == x` |
| Last occurrence of `x` | 2 | Find last satisfying `arr[mid] == x` |
| Search insert position | 3 | Find where `target` would go |
| Floor of `x` | 2 | Last element ≤ x |
| Ceiling of `x` | 2 | First element ≥ x |
| Peak element | 3 | Compare slope, shrink window |
| Min in rotated array | 3 | Shrink toward discontinuity |
| Search in rotated array | 1 (modified) | Determine sorted half, exact match |
| Square root of `n` | 3 | Largest `x` where `x*x ≤ n` |
| Koko eating bananas | 3 | Min speed where `canFinish(mid)` |
| Capacity to ship | 3 | Min capacity where `canShip(mid)` |
| Split array largest sum | 3 | Min sum where `canSplit(mid)` |
| First bad version | 3 | First where `isBad(mid)` is true |
| Count of smaller numbers | 2 | Lower bound of target |

---

## Converting Between Templates

### Template 1 → Template 2

```
   Template 1: if (arr[mid] == target) return mid;
   
   Template 2: if (arr[mid] == target) {
                   result = mid;       ← save it
                   hi = mid - 1;       ← keep looking left
               }
   
   Key change: Don't return immediately, save + continue searching
```

### Template 2 → Template 3

```
   Template 2:                         Template 3:
   result = -1                         (no result variable)
   while (lo <= hi):                   while (lo < hi):
       if condition(mid):                  if condition(mid):
           result = mid                        hi = mid
           hi = mid - 1                    else:
       else:                                   lo = mid + 1
           lo = mid + 1               return lo
   return result
   
   Key change: Remove result variable, use lo < hi, return lo
```

---

## Template 3 Bias Rules (Quick Reference)

```
   ┌────────────────────────────────────────────────────────────────┐
   │                                                                │
   │  If TRUE branch:  hi = mid    → left-biased mid   (default)   │
   │  If TRUE branch:  lo = mid    → right-biased mid  (add +1)    │
   │                                                                │
   │  Formula:                                                      │
   │  Left:  mid = lo + (hi - lo) / 2       ← rounds down          │
   │  Right: mid = lo + (hi - lo + 1) / 2   ← rounds up            │
   │                                                                │
   │  WHY: The branch that does NOT add/subtract 1                 │
   │       must use the bias that differs from it.                  │
   │       Otherwise mid could equal lo/hi → no progress.          │
   │                                                                │
   └────────────────────────────────────────────────────────────────┘
```

---

## Practice: Identify the Template

**Problem 1:** Given sorted array, find if `42` exists.  
<details><summary>Answer</summary>Template 1 — exact match</details>

**Problem 2:** Find the first day where `100` flowers have bloomed.  
<details><summary>Answer</summary>Template 3 (BS on answer) — find minimum day where condition is met</details>

**Problem 3:** Count how many times `7` appears in sorted array.  
<details><summary>Answer</summary>Template 2 — find first and last occurrence, subtract</details>

**Problem 4:** Find the peak of a mountain array.  
<details><summary>Answer</summary>Template 3 — shrinking window with slope comparison</details>

**Problem 5:** Find the largest element ≤ 15 in sorted array.  
<details><summary>Answer</summary>Template 2 — find last element satisfying `arr[mid] ≤ 15`</details>

---

## Quick Revision Questions

1. **Which template returns `-1` if not found? Which always returns a valid index?**
2. **When should you use Template 2 over Template 1?**
3. **For "binary search on answer" problems, which template is best?**
4. **What's the key difference between Template 2 and Template 3?**
5. **How do you convert Template 1 to find the first occurrence?**

---

[← Previous: Common Mistakes](04-common-mistakes.md) | [Next: Practice Problems →](06-practice-problems.md)
