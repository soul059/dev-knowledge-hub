# Chapter 5.2 — Z-Array Construction (O(n) Algorithm)

> **Unit 5: Z Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

The efficient Z-array construction runs in O(n) by maintaining a window [L, R]
that tracks the rightmost Z-box. When a new position falls inside this window,
we reuse previously computed Z-values instead of recomputing from scratch.

---

## 1. Core Idea: Reusing Information

```
  If we know S[L..R] = S[0..R-L], and i is inside [L, R]:

  S:  ....[L.....i.........R]....
  ≡   [0.........i-L.....R-L]

  Position i in S corresponds to position (i - L) in the prefix.
  We already know Z[i - L]!

  Two cases:
  ┌──────────────────────────────────────────────────┐
  │ Case A: Z[i-L] < R - i + 1                      │
  │   The match at i-L ends BEFORE R.                │
  │   So Z[i] = Z[i-L]. Done! No new comparisons.   │
  │                                                   │
  │ Case B: Z[i-L] ≥ R - i + 1                      │
  │   The match might extend past R.                 │
  │   Start matching from R+1 onward.                │
  │   Update L, R.                                   │
  └──────────────────────────────────────────────────┘
```

---

## 2. Algorithm

```
function Z_efficient(S):
    n = len(S)
    Z = array of size n, initialized to 0
    L = 0, R = 0    // [L, R] = rightmost Z-box

    for i = 1 to n-1:
        if i < R:
            // i is inside [L, R]
            k = i - L
            if Z[k] < R - i:
                Z[i] = Z[k]            // Case A: fully inside
            else:
                // Case B: may extend past R
                Z[i] = R - i
                while i + Z[i] < n AND S[Z[i]] == S[i + Z[i]]:
                    Z[i] += 1
                L = i
                R = i + Z[i]
        else:
            // i is outside [L, R] — compute from scratch
            Z[i] = 0
            while i + Z[i] < n AND S[Z[i]] == S[i + Z[i]]:
                Z[i] += 1
            if Z[i] > 0:
                L = i
                R = i + Z[i]

    return Z
```

---

## 3. Detailed Trace

```
  S = "aabxaabxaaz"
       0123456789A       (A = position 10)

  Initialize: Z = [0,0,0,0,0,0,0,0,0,0,0], L=0, R=0

  ═══════════════════════════════════════════════════

  i=1: i ≥ R (1 ≥ 0) → compute from scratch
       Compare S[0] vs S[1]: a==a ✓  Z[1]=1
       Compare S[1] vs S[2]: a==b ✗
       Z[1] = 1, L=1, R=2

  ───────────────────────────────────────────────────
  i=2: i ≥ R (2 ≥ 2) → from scratch
       Compare S[0] vs S[2]: a==b ✗
       Z[2] = 0

  ───────────────────────────────────────────────────
  i=3: i ≥ R → from scratch
       Compare S[0] vs S[3]: a==x ✗
       Z[3] = 0

  ───────────────────────────────────────────────────
  i=4: i ≥ R → from scratch
       Compare S[0] vs S[4]:  a==a ✓  Z=1
       Compare S[1] vs S[5]:  a==a ✓  Z=2
       Compare S[2] vs S[6]:  b==b ✓  Z=3
       Compare S[3] vs S[7]:  x==x ✓  Z=4
       Compare S[4] vs S[8]:  a==a ✓  Z=5
       Compare S[5] vs S[9]:  a==a ✓  Z=6
       Compare S[6] vs S[10]: b==z ✗
       Z[4] = 6, L=4, R=10

  ───────────────────────────────────────────────────
  i=5: i < R (5 < 10) → inside [4, 10]
       k = 5 - 4 = 1, Z[1] = 1
       R - i = 10 - 5 = 5
       Z[1] = 1 < 5 → Case A!
       Z[5] = Z[1] = 1
       (No new comparisons!)

  ───────────────────────────────────────────────────
  i=6: i < R (6 < 10)
       k = 6 - 4 = 2, Z[2] = 0
       R - i = 4
       Z[2] = 0 < 4 → Case A!
       Z[6] = Z[2] = 0

  ───────────────────────────────────────────────────
  i=7: i < R (7 < 10)
       k = 7 - 4 = 3, Z[3] = 0
       R - i = 3
       Z[3] = 0 < 3 → Case A!
       Z[7] = Z[3] = 0

  ───────────────────────────────────────────────────
  i=8: i < R (8 < 10)
       k = 8 - 4 = 4, Z[4] = 6
       R - i = 10 - 8 = 2
       Z[4] = 6 ≥ 2 → Case B!
       Z[8] = 2 (start from R - i)
       Extend: Compare S[2] vs S[10]: b==z ✗
       Z[8] = 2, L=8, R=10

  ───────────────────────────────────────────────────
  i=9: i < R (9 < 10)
       k = 9 - 8 = 1, Z[1] = 1
       R - i = 10 - 9 = 1
       Z[1] = 1 ≥ 1 → Case B!
       Z[9] = 1
       Extend: Compare S[1] vs S[10]: a==z ✗
       Z[9] = 1, L=9, R=10

  ───────────────────────────────────────────────────
  i=10: i ≥ R (10 ≥ 10)
        Compare S[0] vs S[10]: a==z ✗
        Z[10] = 0

  ═══════════════════════════════════════════════════

  Final Z-array:
  ┌─────┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
  │  i  │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │ 9 │10 │
  ├─────┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
  │ S[i]│ a │ a │ b │ x │ a │ a │ b │ x │ a │ a │ z │
  ├─────┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
  │ Z[i]│ — │ 1 │ 0 │ 0 │ 6 │ 1 │ 0 │ 0 │ 2 │ 1 │ 0 │
  └─────┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘

  Total character comparisons: 6 (at i=1) + 0 + 0 + 6 (at i=4)
                              + 0 + 0 + 0 + 0 (at i=8) + 0 (at i=9)
                              + 0 = ~12 comparisons (vs 21 for naive)
```

---

## 4. Why It's O(n)

```
  Key Observation: R only increases.

  Every character comparison either:
  1. Is a match → R increases by 1
  2. Is a mismatch → loop ends

  R goes from 0 to at most n → total matches ≤ n
  Total mismatches ≤ n (at most 1 per position)

  Total comparisons ≤ 2n = O(n)

  ┌─────────────────────────────────────────────────┐
  │  Amortized analysis:                            │
  │  Each comparison either advances R (≤ n times)  │
  │  or ends the while loop (≤ n times).            │
  │  Total work: O(n)                               │
  └─────────────────────────────────────────────────┘
```

---

## 5. Implementation — Python

```python
def z_array(s):
    """Compute Z-array in O(n) time."""
    n = len(s)
    if n == 0:
        return []
    
    z = [0] * n
    z[0] = n  # convention: Z[0] = n
    l, r = 0, 0
    
    for i in range(1, n):
        if i < r:
            # Inside [l, r) window
            k = i - l
            if z[k] < r - i:
                z[i] = z[k]       # Case A
            else:
                z[i] = r - i      # Case B: start extending from r
                while i + z[i] < n and s[z[i]] == s[i + z[i]]:
                    z[i] += 1
                l, r = i, i + z[i]
        else:
            # Outside window — compute from scratch
            z[i] = 0
            while i + z[i] < n and s[z[i]] == s[i + z[i]]:
                z[i] += 1
            if z[i] > 0:
                l, r = i, i + z[i]
    
    return z

# Test
s = "aabxaabxaaz"
print(z_array(s))
# Output: [11, 1, 0, 0, 6, 1, 0, 0, 2, 1, 0]
```

---

## 6. Another Trace: "aaaaa"

```
  S = "aaaaa"
       01234

  i=1: i ≥ R → from scratch
       a==a ✓ a==a ✓ a==a ✓ a==a ✓ → Z[1]=4, L=1, R=5

  i=2: i < R, k=1, Z[1]=4, R-i=3
       Z[1]=4 ≥ 3 → Case B
       Z[2] = 3, extend from R=5: out of bounds
       Z[2] = 3, L=2, R=5

  i=3: i < R, k=1, Z[1]=4, R-i=2
       Z[1]=4 ≥ 2 → Case B
       Z[3] = 2, extend: out of bounds
       Z[3] = 2, L=3, R=5

  i=4: i < R, k=1, Z[1]=4, R-i=1
       Z[1]=4 ≥ 1 → Case B
       Z[4] = 1, extend: out of bounds
       Z[4] = 1

  Z = [5, 4, 3, 2, 1]

  Total comparisons: only 4 (at i=1) + 0+0+0 = 4
  Naive would take: 4+3+2+1 = 10
```

---

## 📝 Summary Table

| Step | Condition | Action |
|------|-----------|--------|
| i ≥ R | Outside window | Compute Z[i] from scratch |
| i < R, Z[k] < R-i | Fully inside | Z[i] = Z[k] (no comparisons) |
| i < R, Z[k] ≥ R-i | Hits boundary | Z[i] starts at R-i, extend |
| After any extension | Z[i] > 0 | Update L=i, R=i+Z[i] |

---

## ❓ Quick Revision Questions

1. **What are the two cases when i < R?**
   <details><summary>Answer</summary>Case A: Z[i-L] < R-i → Z[i] = Z[i-L] (fully inside, no new comparisons). Case B: Z[i-L] ≥ R-i → Z[i] starts at R-i and extends by character comparison.</details>

2. **Why does R never decrease?**
   <details><summary>Answer</summary>L and R are only updated when we find a Z-box that extends past the current R. We never set R to a smaller value.</details>

3. **How does the O(n) proof work?**
   <details><summary>Answer</summary>Each comparison either matches (R increases by 1, max n times total) or mismatches (loop exits, max n times total). So total comparisons ≤ 2n = O(n).</details>

4. **What happens when a position is inside the [L,R] window?**
   <details><summary>Answer</summary>Since S[L..R] = S[0..R-L], position i maps to position k = i-L in the prefix. We can copy Z[k] directly (Case A) or use it as a starting point (Case B).</details>

5. **What is the Z-array for "abcabc"?**
   <details><summary>Answer</summary>Z = [6, 0, 0, 3, 0, 0]. Only position 3 gives "abc" matching the prefix "abc".</details>

---

| [⬅️ Previous: Z-Array Concept](01-z-array-concept.md) | [Next: Pattern Matching with Z ➡️](03-pattern-matching-with-z.md) |
|:---|---:|
