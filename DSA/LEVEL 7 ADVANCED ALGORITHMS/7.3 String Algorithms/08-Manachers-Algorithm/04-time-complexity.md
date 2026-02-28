# Chapter 8.4 — Time Complexity

> **Unit 8: Manacher's Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Manacher's algorithm runs in **O(n)** despite having a nested while loop.
This chapter provides a rigorous proof using amortized analysis.

---

## 1. The Complexity Concern

```
  The algorithm has this structure:

  for i = 1 to n-2:                     ← O(n) iterations
      P[i] = min(P[mirror], R - i)      ← O(1)

      while T[i+P[i]+1] == T[i-P[i]-1]: ← ??? iterations
          P[i] += 1

      if i + P[i] > R:                  ← O(1)
          C, R = i, i + P[i]

  The while loop could run many times for some i.
  Could this be O(n²) total?
  NO — and here's why.
```

---

## 2. Key Observation: R Never Decreases

```
  R = right boundary of the furthest-right palindrome.

  ┌─────────────────────────────────────────────────────┐
  │  R only gets updated in: if i + P[i] > R            │
  │                                                      │
  │  When it updates: R = i + P[i]  (new value > old)   │
  │  → R strictly increases when updated.                │
  │  → R never decreases.                               │
  │                                                      │
  │  R starts at 0.                                      │
  │  R can go up to at most n-1.                         │
  │  → R increases at most n-1 times total.              │
  └─────────────────────────────────────────────────────┘
```

---

## 3. Amortized Analysis

```
  Claim: Total number of expansion steps across ALL iterations = O(n).

  Proof:
  ─────
  Each expansion step (P[i] += 1) compares T[i+P[i]+1] with T[i-P[i]-1].
  If it succeeds, P[i] increases by 1.

  When does the while loop execute?
  1. When i ≥ R → expansion starts from P[i]=0
  2. When P[mirror] = R-i → expansion starts from P[i]=R-i

  In both cases, successful expansion pushes i + P[i] beyond R,
  which means R will be updated to i + P[i].

  ┌─────────────────────────────────────────────────────┐
  │  Each successful expansion step increases R by 1.   │
  │                                                      │
  │  R goes from 0 to at most n-1.                      │
  │                                                      │
  │  ∴ Total successful expansions ≤ n-1 = O(n)         │
  │                                                      │
  │  Each iteration also has at most 1 failed expansion │
  │  (the comparison that ends the while loop).          │
  │  → n-2 failed expansions total.                     │
  │                                                      │
  │  Total comparisons = O(n) + O(n) = O(n)             │
  └─────────────────────────────────────────────────────┘
```

---

## 4. Visual Proof

```
  R progression through the algorithm:

  i=1: R goes from 0 → 1  (1 expansion step)
  i=2: R goes from 1 → 3  (2 expansion steps)
  i=3: mirror used, no expansion. R stays 3.
  i=4: R goes from 3 → 7  (4 expansion steps)
  i=5: mirror used, no expansion. R stays 7.
  i=6: mirror used, 0 expansions. R stays 7.
  i=7: mirror used, 0 expansions. R stays 7.
  i=8: R goes from 7 → 15 (8 expansion steps)
  i=9..15: mirror used, minimal/no expansion.

  R increases: ████████████████████ (total ≤ n)
                         total expansion steps ≤ n

  ┌──── Timeline ────────────────────────────────────┐
  │  i:   1  2  3  4  5  6  7  8  9 10 11 12 13     │
  │  R:   1  3  3  7  7  7  7 15 15 15 15 15 15     │
  │  exp: 1  2  0  4  0  0  0  8  0  0  0  0  0     │
  │       ↑  ↑     ↑           ↑                     │
  │    expansion only when R advances                │
  └──────────────────────────────────────────────────┘
```

---

## 5. Formal Complexity Breakdown

```
  ┌────────────────────────────────┬──────────┐
  │ Operation                     │ Cost     │
  ├────────────────────────────────┼──────────┤
  │ Transform string              │ O(n)     │
  │ Initialize P array            │ O(n)     │
  │ Main loop iterations          │ O(n)     │
  │ Mirror copy (P[i] = ...)      │ O(1)×n   │
  │ Total expansion steps         │ O(n)     │
  │ C, R updates                  │ O(1)×n   │
  │ Find max P[i]                 │ O(n)     │
  │ Extract result                │ O(n)     │
  ├────────────────────────────────┼──────────┤
  │ TOTAL                         │ O(n)     │
  └────────────────────────────────┴──────────┘

  Space: O(n) for transformed string T and array P.
```

---

## 6. Comparison with Other Approaches

```
  ┌───────────────────────┬──────────┬─────────┐
  │ Algorithm             │ Time     │ Space   │
  ├───────────────────────┼──────────┼─────────┤
  │ Brute Force           │ O(n³)    │ O(1)    │
  │ DP Table              │ O(n²)    │ O(n²)   │
  │ Expand Around Center  │ O(n²)    │ O(1)    │
  │ Hashing + Binary Srch │ O(n lg n)│ O(n)    │
  │ Suffix Array + LCP    │ O(n lg n)│ O(n)    │
  │ Eertree (palindromic) │ O(n)     │ O(n)    │
  │ Manacher's Algorithm  │ O(n)     │ O(n)    │
  └───────────────────────┴──────────┴─────────┘

  Manacher's is optimal — Ω(n) is a lower bound
  (must read every character at least once).
```

---

## 7. Why Mirror Avoids Redundant Work

```
  Without mirroring (expand-around-center):

  S = "aaaaaa"  →  every center expands many times

  Center 0: expand 0 times → "a"
  Center 1: expand 1 time  → "aaa"      1 comparison
  Center 2: expand 2 times → "aaaaa"    2 comparisons
  Center 3: expand 3 times → "aaaaaa"   3 comparisons (hits end)
  ...
  Total: 0+1+2+3+... = O(n²)

  With Manacher's mirroring:

  Once we find the big palindrome, mirrors give us
  P values for free. Total expansions = O(n).

  ┌────────────────────────────────────────────────┐
  │  Mirroring eliminates RE-DISCOVERING known     │
  │  palindromes. Expansion only happens at the    │
  │  "frontier" (beyond R), and R advances at      │
  │  most n positions total.                       │
  └────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Property | Value |
|----------|-------|
| Time complexity | O(n) |
| Space complexity | O(n) |
| Proof technique | Amortized analysis via R's monotonic increase |
| Key insight | Expansion only advances R; R bounded by n |
| Total expansions | ≤ n (across all iterations combined) |
| Mirror copies | O(1) each, O(n) total |
| Optimality | Yes — Ω(n) lower bound (must read input) |

---

## ❓ Quick Revision Questions

1. **Why doesn't the while loop make the algorithm O(n²)?**
   <details><summary>Answer</summary>Each successful expansion step advances R by 1, and R can only go from 0 to n. So total successful expansions across ALL iterations is at most n. Adding n failed expansions (one per iteration at most), total work is O(n).</details>

2. **What is the potential function in the amortized analysis?**
   <details><summary>Answer</summary>The value of R serves as the potential. Each expansion step increases R by 1 (cost=1, potential increase=1). R is bounded by n, so total work is O(n). Non-expanding iterations have O(1) cost with no potential change.</details>

3. **How many total comparisons does Manacher's algorithm make?**
   <details><summary>Answer</summary>At most 2n comparisons total: at most n successful comparisons (each advancing R) and at most n failed comparisons (one per loop iteration that ends the while loop).</details>

4. **Is Manacher's algorithm optimal?**
   <details><summary>Answer</summary>Yes. Any algorithm must read every character at least once, giving Ω(n) lower bound. Manacher's achieves O(n), matching the lower bound.</details>

5. **What input causes the most expansion steps?**
   <details><summary>Answer</summary>A string like "aaa...a" (all same characters) causes early palindrome centers to expand, but once the whole-string palindrome is found, subsequent positions use mirrors. Total expansion is still O(n).</details>

---

| [⬅️ Previous: Algorithm Steps](03-algorithm-steps.md) | [Next: Applications ➡️](05-applications.md) |
|:---|---:|
