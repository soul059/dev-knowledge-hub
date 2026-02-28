# Chapter 8.3 — Manacher's Algorithm Steps

> **Unit 8: Manacher's Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

This chapter presents the **complete Manacher's algorithm** — a
linear-time algorithm for finding all palindromic substrings. We cover
the key insight, the P-array, and the mirror property.

---

## 1. The P-Array

```
  P[i] = palindrome radius centered at index i in the transformed string.

  Transformed T:  ^ # a # b # a # $
  Index:          0 1 2 3 4 5 6 7 8
  P[i]:           0 0 1 0 3 0 1 0 0

  P[4] = 3 means: palindrome of radius 3 centered at T[4]='b'
  → T[1..7] = "#a#b#a#" is a palindrome
  → Original: "aba" (length 3 = P[4])

  Goal: Compute P[i] for ALL positions in O(n) total.
```

---

## 2. Core Insight: Mirror Property

```
  If we know a large palindrome centered at C with right boundary R,
  then palindromes INSIDE it can be derived from their mirrors.

  ┌─────────────────────────────────────────────────────────┐
  │         L            C            R                     │
  │  ───────|────────────|────────────|───────              │
  │              mirror  ↑  current                        │
  │              i'      C      i                          │
  │                                                         │
  │  i' = 2C - i  (mirror of i around C)                   │
  │                                                         │
  │  If palindrome at i' is entirely inside [L..R],         │
  │  then P[i] = P[i'] — no expansion needed!              │
  └─────────────────────────────────────────────────────────┘

  Three cases for position i (where i < R):

  Case 1: P[mirror] < R - i
  ─────────────────────────
  Mirror palindrome fits inside. P[i] = P[mirror].
  No expansion.

      L ──── mirror ──── C ──── i ──── R
         [==pal==]           [==pal==]
         P[mirror] < R-i → P[i] = P[mirror]

  Case 2: P[mirror] > R - i
  ─────────────────────────
  Mirror palindrome extends BEYOND L.
  P[i] = R - i (at least). Can't be more (or R would be wrong).

      L ── mirror ──── C ──── i ──── R
    [===pal===]              [===pal...]  
    extends past L            P[i] = R - i

  Case 3: P[mirror] == R - i
  ──────────────────────────
  Mirror palindrome reaches exactly to L.
  P[i] ≥ R - i, but might extend further → EXPAND.

      L ── mirror ──── C ──── i ──── R
    [===pal===]              [===pal===]??
    reaches L exactly         expand beyond R
```

---

## 3. Algorithm Pseudocode

```
function Manacher(S):
    T = transform(S)          // "abc" → "^#a#b#c#$"
    n = len(T)
    P = array of size n, all 0

    C = 0     // center of rightmost palindrome
    R = 0     // right boundary of rightmost palindrome

    for i = 1 to n-2:         // skip sentinels ^ and $
        mirror = 2*C - i

        if i < R:
            P[i] = min(P[mirror], R - i)   // use mirror info

        // Attempt expansion
        while T[i + P[i] + 1] == T[i - P[i] - 1]:
            P[i] += 1

        // Update center and right boundary
        if i + P[i] > R:
            C = i
            R = i + P[i]

    return P
```

---

## 4. Complete Trace

```
  S = "abacaba"
  T = "^#a#b#a#c#a#b#a#$"
  Idx: 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17

  Initialize: C=0, R=0, P=[0,0,0,...,0]

  i=1 (T[1]='#'):
    i≥R → P[1]=0
    Expand: T[2]='a' vs T[0]='^' → no match
    P[1]=0. i+P[1]=1 > R=0 → C=1, R=1

  i=2 (T[2]='a'):
    i>R → P[2]=0
    Expand: T[3]='#' vs T[1]='#' → match! P[2]=1
    Expand: T[4]='b' vs T[0]='^' → no match
    P[2]=1. i+P[2]=3 > R=1 → C=2, R=3

  i=3 (T[3]='#'):
    i<R=3? NO (i==R) → P[3]=0
    Expand: T[4]='b' vs T[2]='a' → no match
    P[3]=0. i+P[3]=3 = R → no update

  i=4 (T[4]='b'):
    i>R → P[4]=0
    Expand: T[5]='#' vs T[3]='#' → match! P[4]=1
    Expand: T[6]='a' vs T[2]='a' → match! P[4]=2
    Expand: T[7]='#' vs T[1]='#' → match! P[4]=3
    Expand: T[8]='c' vs T[0]='^' → no match
    P[4]=3. i+P[4]=7 > R=3 → C=4, R=7

  i=5 (T[5]='#'):
    i<R=7, mirror=2*4-5=3, P[3]=0, R-i=2
    P[5] = min(0, 2) = 0
    Expand: T[6]='a' vs T[4]='b' → no match
    P[5]=0.

  i=6 (T[6]='a'):
    i<R=7, mirror=2*4-6=2, P[2]=1, R-i=1
    P[6] = min(1, 1) = 1    ← Case 3 (equal), may expand!
    Expand: T[8]='c' vs T[4]='b' → no match
    P[6]=1.

  i=7 (T[7]='#'):
    i=R=7, not less → P[7]=0
    Expand: T[8]='c' vs T[6]='a' → no match
    P[7]=0.

  i=8 (T[8]='c'):
    i>R → P[8]=0
    Expand: T[9]='#' vs T[7]='#' → match! P[8]=1
    Expand: T[10]='a' vs T[6]='a' → match! P[8]=2
    ...continuing...
    P[8]=7 (whole string "abacaba")
    C=8, R=15

  i=9..16: mirror values used from P[7]..P[1]
    Due to symmetry, many P[i] are set directly from mirrors.

  Final P:
  Idx: 0  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17
  T:   ^  #  a  #  b  #  a  #  c  #  a  #  b  #  a  #  $
  P:   0  0  1  0  3  0  1  0  7  0  1  0  3  0  1  0  0  0

  Max P[i] = 7 at i=8 → palindrome "abacaba" (length 7)
```

---

## 5. Finding the Answer

```python
def longest_palindrome_manacher(s: str) -> str:
    T = '^#' + '#'.join(s) + '#$'
    n = len(T)
    P = [0] * n
    C = R = 0

    for i in range(1, n - 1):
        mirror = 2 * C - i
        if i < R:
            P[i] = min(P[mirror], R - i)
        while T[i + P[i] + 1] == T[i - P[i] - 1]:
            P[i] += 1
        if i + P[i] > R:
            C, R = i, i + P[i]

    # Find maximum P[i]
    max_len = max(P)
    center = P.index(max_len)

    # Convert back to original string
    start = (center - max_len) // 2
    return s[start:start + max_len]
```

---

## 6. The Three Cases Visualized

```
  State: C=8, R=15 in our trace.

  ┌────────────────────────────────────────────────────────┐
  │  T: ^ # a # b # a # c # a # b # a # $                │
  │     0 1 2 3 4 5 6 7 8 9 ...                           │
  │              L=1       C=8         R=15                │
  │                                                        │
  │  For i=10 (T[10]='a'):                                │
  │    mirror = 2×8 - 10 = 6                              │
  │    P[mirror] = P[6] = 1                               │
  │    R - i = 15 - 10 = 5                                │
  │    P[6]=1 < 5 → Case 1: P[10] = 1 (copy mirror)      │
  │                                                        │
  │  For i=12 (T[12]='b'):                                │
  │    mirror = 2×8 - 12 = 4                              │
  │    P[mirror] = P[4] = 3                               │
  │    R - i = 15 - 12 = 3                                │
  │    P[4]=3 == 3 → Case 3: P[12] ≥ 3, try expand       │
  │    T[16]='$' vs T[8]='c' → no match                  │
  │    P[12] = 3                                           │
  └────────────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Component | Description |
|-----------|-------------|
| P-array | P[i] = palindrome radius at position i in T |
| C, R | Center and right boundary of rightmost palindrome |
| Mirror | mirror = 2C - i |
| Case 1 | P[mirror] < R-i → P[i] = P[mirror] |
| Case 2 | P[mirror] > R-i → P[i] = R-i |
| Case 3 | P[mirror] = R-i → P[i] ≥ R-i, expand beyond R |
| Combined | P[i] = min(P[mirror], R-i), then expand |
| Time | O(n) amortized |

---

## ❓ Quick Revision Questions

1. **What does P[i] represent?**
   <details><summary>Answer</summary>P[i] is the radius of the longest palindrome centered at index i in the transformed string. It also equals the length of the corresponding palindrome in the original string.</details>

2. **Why do we use min(P[mirror], R-i)?**
   <details><summary>Answer</summary>P[mirror] gives information from the mirror palindrome, but we can only trust it up to the boundary R. Beyond R, we have no information, so we cap at R-i and then try to expand.</details>

3. **When does expansion actually happen?**
   <details><summary>Answer</summary>Expansion happens in Case 3 (P[mirror] == R-i) when the mirror palindrome reaches exactly to the left boundary, meaning the palindrome at i might extend beyond R. It also happens when i ≥ R (no mirror info available).</details>

4. **What is the role of C and R?**
   <details><summary>Answer</summary>C is the center and R is the right boundary of the palindrome that extends furthest to the right seen so far. They define the "known" region where mirror information can be reused.</details>

5. **How do sentinels ^ and $ help the algorithm?**
   <details><summary>Answer</summary>They act as mismatching boundaries during expansion (since they don't match any other character), eliminating bounds-checking conditions in the inner while loop.</details>

---

| [⬅️ Previous: Transformed String](02-transformed-string.md) | [Next: Time Complexity ➡️](04-time-complexity.md) |
|:---|---:|
