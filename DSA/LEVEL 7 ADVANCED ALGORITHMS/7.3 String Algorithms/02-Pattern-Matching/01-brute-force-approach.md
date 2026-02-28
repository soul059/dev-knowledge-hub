# Chapter 2.1 — Brute Force Approach

> **Unit 2: Pattern Matching** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Pattern matching — finding occurrences of a pattern `P` inside a text `T` — is
the central problem of string algorithms. We begin with the naive brute-force
approach to understand its mechanics and limitations, setting the stage for
optimized algorithms.

---

## 1. Problem Statement

```
  Given:   Text    T  of length  n
           Pattern P  of length  m     (m ≤ n)

  Find:    All positions i where P occurs in T.
           i.e., T[i..i+m-1] == P[0..m-1]

  Example:
  ─────────
  T = "AABAACAADAABAABA"
  P = "AABA"

  Output: Positions 0, 9, 12

  T:  A A B A A C A A D A A B A A B A
      ─────           ─────   ─────
       0               9       12
```

---

## 2. Brute Force Algorithm

### Idea

Try every possible starting position `i` in `T`, and check character by
character if `P` matches.

```
  NAIVE_SEARCH(T, P):
      n ← |T|
      m ← |P|
      for i = 0 to n - m:
          j ← 0
          while j < m AND T[i + j] == P[j]:
              j ← j + 1
          if j == m:
              report match at position i
```

### Visual Walkthrough

```
  T = "AABCAAB"
  P = "AAB"

  i=0: Compare T[0..2] with P[0..2]
       A A B
       A A B  ← match!  ✓  Report index 0
       ─────

  i=1: Compare T[1..3] with P[0..2]
       A B C
       A A B  ← mismatch at j=1 (B ≠ A)  ✗
         ▲

  i=2: Compare T[2..4] with P[0..2]
       B C A
       A A B  ← mismatch at j=0 (B ≠ A)  ✗
       ▲

  i=3: Compare T[3..5] with P[0..2]
       C A A
       A A B  ← mismatch at j=0 (C ≠ A)  ✗
       ▲

  i=4: Compare T[4..6] with P[0..2]
       A A B
       A A B  ← match!  ✓  Report index 4
       ─────

  Result: [0, 4]
```

---

## 3. Detailed Step-by-Step Trace

```
  T = "ABABDABACDABABCABAB"   (n = 18)
  P = "ABABCABAB"             (m = 9)

  i=0:  T: A B A B D A B A C D ...
        P: A B A B C A B A B
                   ▲
           j goes: 0,1,2,3 → mismatch at j=4  (D ≠ C)

  i=1:  T: . B A B D ...
        P:   A B A B C ...
              ▲
           mismatch at j=0  (B ≠ A)

  i=2:  T: . . A B D ...
        P:     A B A B C ...
                   ▲
           j goes: 0,1 → mismatch at j=2  (D ≠ A)

  ... (many failed attempts) ...

  i=9:  T: . . . . . . . . . D A B A B C A B A B
        P:                      A B A B C A B A B
           mismatch at j=0  (D ≠ A)

  i=10: T: . . . . . . . . . . A B A B C A B A B  ← remaining = 9 = m ⬇
        P:                        A B A B C A B A B
           j goes: 0,1,2,3,4,5,6,7,8 → all match!  ✓

  Result: [10]   (actually let me recount)
  
  Wait — let me recount properly. Pattern found when all m chars match.
```

---

## 4. Complexity Analysis

### Time Complexity

```
  Outer loop: runs (n - m + 1) times
  Inner loop: runs up to m times per position

  Worst case:  O((n - m + 1) × m)  =  O(n × m)

  ┌─────────────────────────────────────────────────┐
  │  Worst-Case Example:                            │
  │                                                 │
  │  T = "AAAAAAAAAAAA"   (n = 12)                  │
  │  P = "AAAAB"          (m = 5)                   │
  │                                                 │
  │  At every position i, we compare 4 A's before   │
  │  discovering the mismatch at 'B'. So inner      │
  │  loop runs m-1 = 4 times for each of the        │
  │  n-m+1 = 8 positions.                           │
  │                                                 │
  │  Total comparisons ≈ 8 × 4 = 32 = O(n × m)     │
  └─────────────────────────────────────────────────┘
```

### Best Case

```
  When first character of P mismatches at every position:

  T = "ABCDEFGHIJ"
  P = "XYZ"

  Each position: 1 comparison (T[i] ≠ P[0])
  Total: n - m + 1 comparisons = O(n)
```

### Space Complexity

```
  O(1) extra — only indices i and j
```

### Summary

| Case | Comparisons | Example |
|------|-------------|---------|
| Best | O(n) | First char always mismatches |
| Average | O(n) | Random text and pattern |
| Worst | O(n × m) | T = "AAA...A", P = "AA...AB" |

---

## 5. Why Brute Force Fails

The critical inefficiency: **when a mismatch occurs, we forget everything
we learned from previous comparisons.**

```
  T: A B A B A B A B C
  P: A B A B C
             ▲  mismatch at j=4

  What we KNOW:  T[0..3] = "ABAB" = P[0..3]
  What we DO:    Start over at i=1, comparing from scratch!

  At i=1:  B ≠ A  → fail (1 comparison wasted)
  At i=2:  A B A B C — we already knew T[2..3] = "AB" = P[0..1]!
           But brute force re-verifies this.

  ┌──────────────────────────────────────────────┐
  │  💡 KEY INSIGHT                              │
  │                                              │
  │  We can skip ahead using the structure of P  │
  │  itself. This is exactly what KMP does.      │
  │                                              │
  │  "The partial match tells us where to        │
  │   resume without re-examining characters."   │
  └──────────────────────────────────────────────┘
```

---

## 6. Pseudocode (Clean Version)

```
  BRUTE_FORCE_SEARCH(text, pattern):
  ───────────────────────────────────
      Input:  text T[0..n-1], pattern P[0..m-1]
      Output: list of match positions

      matches ← []
      for i ← 0 to n - m:
          match ← true
          for j ← 0 to m - 1:
              if T[i + j] ≠ P[j]:
                  match ← false
                  break
          if match:
              matches.append(i)
      return matches
```

---

## 7. Implementation Considerations

### Sentinel Optimization

```
  Place pattern at the end of text as a sentinel:
  T' = T + P

  This guarantees the inner loop always finds a match eventually,
  eliminating the j < m check.
  
  (Minor constant-factor improvement, still O(n×m) worst case)
```

### Early Termination with Bad-Character Heuristic

```
  If T[i+j] mismatches P[j], and T[i+j] doesn't appear in P at all:
    → skip ahead by j+1 positions!

  This is the basis of the Boyer-Moore algorithm.
```

---

## 📝 Summary Table

| Aspect | Detail |
|--------|--------|
| Algorithm | Try every starting position, check char by char |
| Time (worst) | O(n × m) |
| Time (best) | O(n) |
| Space | O(1) |
| Advantage | Simple to implement and understand |
| Disadvantage | Redundant comparisons on structured patterns |
| When adequate | Short patterns, random text, one-time search |
| Key weakness | Forgets partial match information |

---

## ❓ Quick Revision Questions

1. **What is the worst-case time complexity of brute-force pattern matching?**
   <details><summary>Answer</summary>O(n × m), where n is text length and m is pattern length.</details>

2. **Give a concrete worst-case input for the brute-force algorithm.**
   <details><summary>Answer</summary>T = "AAAAAAA...A" (all same chars), P = "AAA...AB" (same chars with different last). Each window matches m-1 chars before failing.</details>

3. **How many starting positions does the brute-force algorithm try?**
   <details><summary>Answer</summary>n - m + 1 positions (from index 0 to index n - m).</details>

4. **What information does brute force "forget" after a mismatch?**
   <details><summary>Answer</summary>The partial match — characters of the pattern that already matched correctly at the current position.</details>

5. **When is brute force actually acceptable in practice?**
   <details><summary>Answer</summary>When the pattern is very short (say m ≤ 5), when searching random text, or when simplicity matters more than speed.</details>

---

| [⬅️ Previous Unit: Palindromes](../01-String-Basics-Review/06-palindromes.md) | [Next: Optimization Need ➡️](02-optimization-need.md) |
|:---|---:|
