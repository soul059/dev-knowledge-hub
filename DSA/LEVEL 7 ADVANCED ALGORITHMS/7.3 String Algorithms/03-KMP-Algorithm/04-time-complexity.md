# Chapter 3.4 — Time Complexity Analysis

> **Unit 3: KMP Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

This chapter rigorously proves the O(n + m) time complexity of KMP using
amortized analysis and the potential method.

---

## 1. Complexity Claims

| Phase | Time | Space |
|-------|------|-------|
| LPS Construction | O(m) | O(m) |
| Pattern Matching | O(n) | O(1) extra |
| **Total** | **O(n + m)** | **O(m)** |

---

## 2. LPS Construction — O(m) Proof

```
  Variables: i (scans 1 to m-1), len (current prefix-suffix length)

  Observation 1: i increases by 1 in TWO places:
    (a) after a match:     i++, len++
    (b) after mismatch with len==0: i++

  Observation 2: len decreases on mismatch with len > 0:
    len = LPS[len - 1]   (strictly decreases since LPS[k] < k)

  ┌───────────────────────────────────────────────────────┐
  │  Potential Function: Φ = len                         │
  │                                                      │
  │  • Match (case a): i++, len++                        │
  │    Actual cost = 1, Φ increases by 1                 │
  │    Amortized cost = 1 + 1 = 2                        │
  │                                                      │
  │  • Mismatch, len > 0: len decreases                  │
  │    Actual cost = 1, Φ decreases by ≥ 1               │
  │    Amortized cost = 1 - (decrease) ≤ 0               │
  │                                                      │
  │  • Mismatch, len = 0: i++                            │
  │    Actual cost = 1, Φ unchanged (len stays 0)        │
  │    Amortized cost = 1                                │
  │                                                      │
  │  Total amortized cost over all iterations:           │
  │  = Σ amortized_cost ≤ 2 × (# of i increments)      │
  │  = 2 × (m - 1) = O(m)                               │
  │                                                      │
  │  Since Φ_final ≥ 0 = Φ_initial:                     │
  │    Total actual cost ≤ Total amortized cost = O(m)   │
  └───────────────────────────────────────────────────────┘
```

### Alternative Argument

```
  len increases by 1 at most (m - 1) times (once per i increment on match).
  len can never go below 0.
  Therefore, len decreases at most (m - 1) times.
  Total operations (increases + decreases + i-advances) ≤ 3(m - 1) = O(m).
```

---

## 3. KMP Matching — O(n) Proof

```
  Variables: i (text pointer, 0 to n-1), j (pattern pointer)

  i increases by 1 when:
    (a) T[i] == P[j]:  both i and j advance
    (b) j == 0 and mismatch: i advances, j stays at 0

  j decreases when:
    (c) mismatch with j > 0:  j = LPS[j-1]

  ┌───────────────────────────────────────────────────────┐
  │  Potential Function: Φ = j                           │
  │                                                      │
  │  Case (a): i++, j++                                  │
  │    Amortized = 1 + 1 = 2                             │
  │                                                      │
  │  Case (b): i++, j unchanged (j=0)                   │
  │    Amortized = 1                                     │
  │                                                      │
  │  Case (c): j decreases, i unchanged                  │
  │    Amortized = 1 - (decrease) ≤ 0                    │
  │                                                      │
  │  i increments at most n times → total ≤ 2n = O(n)   │
  └───────────────────────────────────────────────────────┘
```

---

## 4. Comparison with Other Algorithms

```
  ┌─────────────────┬──────────────┬──────────────┬──────────────┐
  │ Algorithm       │  Best Case   │  Average     │  Worst Case  │
  ├─────────────────┼──────────────┼──────────────┼──────────────┤
  │ Brute Force     │  O(n)        │  O(n)        │  O(n × m)   │
  │ KMP             │  O(n + m)    │  O(n + m)    │  O(n + m)   │
  │ Rabin-Karp      │  O(n + m)    │  O(n + m)    │  O(n × m)   │
  │ Z-Algorithm     │  O(n + m)    │  O(n + m)    │  O(n + m)   │
  │ Boyer-Moore     │  O(n / m)    │  O(n)        │  O(n × m)   │
  └─────────────────┴──────────────┴──────────────┴──────────────┘

  💡 KMP guarantees O(n + m) in ALL cases — no worst-case degradation!
```

---

## 5. Number of Comparisons

```
  Total comparisons in KMP matching ≤ 2n - 1

  Proof:
  - Each of the n positions in T contributes:
    • At most 1 match comparison (when i advances)
    • Some mismatch comparisons (but these pay for j's decrease)
  - Total match comparisons ≤ n
  - Total mismatch comparisons ≤ n - 1 (j goes up ≤ n times, so down ≤ n-1)
  - Total ≤ 2n - 1

  In practice, KMP makes about 1.1n to 1.5n comparisons on average
  for typical English text.
```

---

## 6. Space Complexity

```
  ┌──────────────────────────────────────────┐
  │  LPS array:    O(m)                     │
  │  Text:         O(n) (input, not extra)  │
  │  Pattern:      O(m) (input, not extra)  │
  │  Variables:    O(1) (i, j, len)         │
  │                                          │
  │  Extra space:  O(m) for LPS array       │
  └──────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Aspect | Complexity | Proof Technique |
|--------|-----------|-----------------|
| LPS build | O(m) | Amortized: len increases ≤ m times |
| Matching | O(n) | Amortized: i increases ≤ n times |
| Total | O(n + m) | Sum of both phases |
| Space | O(m) | LPS array |
| Comparisons | ≤ 2n - 1 | Match + mismatch bound |
| Guarantee | All cases | No worst-case degradation |

---

## ❓ Quick Revision Questions

1. **Why is KMP guaranteed O(n + m) even in the worst case?**
   <details><summary>Answer</summary>The text pointer i never backtracks, and j's fallbacks are bounded by j's total increases. Since j increases at most n times, it decreases at most n times.</details>

2. **What is the maximum number of comparisons KMP makes?**
   <details><summary>Answer</summary>At most 2n - 1 comparisons during the matching phase.</details>

3. **How does KMP compare to Rabin-Karp in worst case?**
   <details><summary>Answer</summary>KMP is always O(n+m). Rabin-Karp can degrade to O(n×m) if hash collisions are frequent (spurious hits).</details>

4. **What potential function proves the matching is O(n)?**
   <details><summary>Answer</summary>Φ = j (the pattern pointer). Matches increase Φ at cost 2, mismatches decrease Φ at cost ≤ 0. Since i advances ≤ n times, total ≤ 2n.</details>

5. **Is O(n + m) optimal for single pattern matching?**
   <details><summary>Answer</summary>Yes — any algorithm must read the entire text O(n) and pattern O(m), so Ω(n + m) is a lower bound.</details>

---

| [⬅️ Previous: Pattern Matching with KMP](03-pattern-matching-with-kmp.md) | [Next: Applications ➡️](05-applications.md) |
|:---|---:|
