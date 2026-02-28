# Chapter 3: Left / Right Bias in Binary Search

[← Previous: Boundary Finding Template](02-boundary-finding-template.md) | [Next: Common Mistakes →](04-common-mistakes.md)

---

## Overview

When `lo + 1 == hi`, the mid-point calculation can either lean **left** or **right**. This "bias" determines which element `mid` points to, and choosing the wrong one can cause **infinite loops**.

---

## The Bias Problem

```
   When lo = 4, hi = 5:
   
   Left-biased mid  = 4 + (5 - 4) / 2 = 4    → mid = lo
   Right-biased mid = 4 + (5 - 4 + 1) / 2 = 5  → mid = hi
   
   ┌───────┬───────┐
   │  lo   │  hi   │
   │ idx 4 │ idx 5 │
   └───────┴───────┘
      ↑        ↑
   Left mid  Right mid
```

---

## Left-Biased Mid (Default)

```
   mid = lo + (hi - lo) / 2    // Floors toward lo

   Use with: lo = mid + 1  (move lo past mid)
   
   Safe because: lo always moves at least 1 step forward
```

### When Left Bias Causes Infinite Loop

```
   If you combine left-biased mid with lo = mid:
   
   lo=4, hi=5, mid=4
   lo = mid = 4    ← lo doesn't change!
   
   Next iteration: lo=4, hi=5, mid=4
   lo = mid = 4    ← INFINITE LOOP!
```

---

## Right-Biased Mid

```
   mid = lo + (hi - lo + 1) / 2    // Ceils toward hi

   Use with: hi = mid - 1  (move hi past mid)
   
   Safe because: hi always moves at least 1 step backward
```

### When to Use Right Bias

```
   When the update rule includes: lo = mid (without + 1)
   
   lo=4, hi=5
   Right-biased mid = 5
   lo = mid = 5    → lo changes! Loop continues correctly.
```

---

## The Rule

```
   ╔══════════════════════════════════════════════════════════╗
   ║                   BIAS SELECTION RULE                    ║
   ║                                                          ║
   ║  If lo update is: lo = mid + 1  →  LEFT bias  (default)  ║
   ║  If lo update is: lo = mid      →  RIGHT bias            ║
   ║                                                          ║
   ║  Left bias:   mid = lo + (hi - lo) / 2                   ║
   ║  Right bias:  mid = lo + (hi - lo + 1) / 2               ║
   ╚══════════════════════════════════════════════════════════╝
```

---

## Side-by-Side Comparison

### Find First TRUE (left bias + `lo = mid + 1`)

```python
def find_first_true(lo, hi, condition):
    while lo < hi:
        mid = lo + (hi - lo) // 2        # LEFT bias
        if condition(mid):
            hi = mid                       # Keep candidate
        else:
            lo = mid + 1                   # Skip mid (lo moves forward)
    return lo
```

### Find Last FALSE (right bias + `lo = mid`)

```python
def find_last_false(lo, hi, condition):
    while lo < hi:
        mid = lo + (hi - lo + 1) // 2    # RIGHT bias
        if condition(mid):
            hi = mid - 1                   # Skip mid
        else:
            lo = mid                       # Keep candidate (lo might stay)
    return lo
```

---

## Trace: Why Right Bias Prevents Infinite Loop

### Find Last FALSE: arr = `[F, F, T, T]` (indices 0-3)

**With LEFT bias (WRONG):**

| Step | lo | hi | mid (left) | condition | Action |
|------|----|----|-----------|-----------|--------|
| 1 | 0 | 3 | 1 | FALSE | lo = mid = 1 |
| 2 | 1 | 3 | 2 | TRUE | hi = mid - 1 = 1 |
| 3 | 1 | 1 | — | — | lo == hi → STOP. Answer: 1 ✓ (lucky!) |

But what if arr = `[F, F, F, T]`?

| Step | lo | hi | mid (left) | condition | Action |
|------|----|----|-----------|-----------|--------|
| 1 | 0 | 3 | 1 | FALSE | lo = mid = 1 |
| 2 | 1 | 3 | 2 | FALSE | lo = mid = 2 |
| 3 | 2 | 3 | 2 | FALSE | lo = mid = 2 ← **INFINITE LOOP!** |

**With RIGHT bias (CORRECT):**

| Step | lo | hi | mid (right) | condition | Action |
|------|----|----|------------|-----------|--------|
| 1 | 0 | 3 | 2 | FALSE | lo = mid = 2 |
| 2 | 2 | 3 | 3 | TRUE | hi = mid - 1 = 2 |
| 3 | 2 | 2 | — | — | lo == hi → STOP. Answer: 2 ✓ |

---

## Decision Flowchart

```
   ┌───────────────────────────────────────┐
   │  Does your code have lo = mid ?       │
   └──────────┬────────────────┬───────────┘
              │ Yes            │ No (lo = mid + 1)
              │                │
   ┌──────────▼──────────┐  ┌──▼───────────────────┐
   │  Use RIGHT bias     │  │  Use LEFT bias        │
   │  mid = lo +         │  │  mid = lo +           │
   │    (hi-lo+1)/2      │  │    (hi-lo)/2          │
   └─────────────────────┘  └───────────────────────┘
```

---

## Summary Table

| Aspect | Left Bias | Right Bias |
|--------|-----------|------------|
| **Formula** | `lo + (hi - lo) / 2` | `lo + (hi - lo + 1) / 2` |
| **mid leans toward** | `lo` | `hi` |
| **Use when** | `lo = mid + 1` | `lo = mid` |
| **Prevents** | — | Infinite loop with `lo = mid` |
| **Default?** | Yes (most common) | Use when needed |

---

## Quick Revision Questions

1. **What is "bias" in binary search mid-point calculation?**
2. **When does left bias cause an infinite loop?**
3. **How do you compute a right-biased mid?**
4. **State the rule for choosing left vs right bias in one sentence.**
5. **Why doesn't left bias cause infinite loops when `lo = mid + 1`?**
6. **Trace the result of using the wrong bias on `[F, F, F, T]` for "find last FALSE".**

---

[← Previous: Boundary Finding Template](02-boundary-finding-template.md) | [Next: Common Mistakes →](04-common-mistakes.md)
