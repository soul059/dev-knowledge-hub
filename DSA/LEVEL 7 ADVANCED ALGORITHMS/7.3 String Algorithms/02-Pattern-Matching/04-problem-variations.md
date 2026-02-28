# Chapter 2.4 — Problem Variations

> **Unit 2: Pattern Matching** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Pattern matching isn't just "find P in T". There are many variations you'll
encounter in contests and interviews. This chapter catalogues the major
variations with the right algorithm for each.

---

## 1. Exact Single Pattern Matching

```
  Given:  T = "abcabcabc",  P = "cab"
  Find:   All indices where P occurs exactly.
  Answer: [2, 5]

  Algorithms: Brute Force O(nm), KMP O(n+m), Z O(n+m)
```

---

## 2. Exact Multi-Pattern Matching

```
  Given:  T = "abcdefgh"
          P = {"abc", "cde", "fgh", "xyz"}
  Find:   Which patterns occur and where.

  Answer: "abc" at 0, "cde" at 2, "fgh" at 5

  ┌──────────────────────────────────────────┐
  │  Naive: Run KMP for each pattern         │
  │         O(n × k + Σ|Pᵢ|)  for k patterns│
  │                                          │
  │  Better: Aho-Corasick                    │
  │         O(n + Σ|Pᵢ| + z)                │
  │         where z = total matches          │
  │                                          │
  │  Also: Rabin-Karp with multiple hashes   │
  └──────────────────────────────────────────┘
```

---

## 3. Approximate Pattern Matching

```
  Find occurrences of P in T allowing up to k mismatches/edits.

  Example (k=1 mismatch):
  T = "abcaXcabc"
  P = "abc"

  Exact matches: positions 0, 6
  With 1 mismatch: also position 3 ("aXc" differs in 1 char)

  Approaches:
  ┌─────────────────────────────────────────┐
  │ • DP: O(n × m) edit distance per window │
  │ • Bitap algorithm: bit-parallel search  │
  │ • Suffix automaton + BFS               │
  └─────────────────────────────────────────┘
```

---

## 4. Wildcard Pattern Matching

```
  '?' matches any single character
  '*' matches any sequence (including empty)

  P = "a*b?c"
  T = "axyzbc"  →  a(xyz)b(any)c = "axyzbc"  ✓

  ┌──────────────────────────────────────────┐
  │  Approach: DP with states (i, j)         │
  │  dp[i][j] = can P[0..i-1] match T[0..j-1]?│
  │                                          │
  │  Time: O(n × m)                          │
  │  (Detailed in Unit 9: String DP)         │
  └──────────────────────────────────────────┘
```

---

## 5. Regular Expression Matching

```
  '.' matches any single character
  '*' means 0 or more of the preceding element

  P = "a.*b"    matches "aXYZb", "ab", "aXb"
  P = "c*a"     matches "a", "ca", "cca", "ccca"

  Approach: DP or NFA simulation
  Time: O(n × m)
```

---

## 6. Pattern Counting

```
  Count how many times P occurs in T (possibly overlapping).

  T = "aaaa",  P = "aa"

  Non-overlapping: "aa|aa" → 2
  Overlapping:     positions 0,1,2 → 3

  ┌──────────────────────────────────────────┐
  │  KMP naturally finds overlapping matches │
  │  (just continue after each match)        │
  │                                          │
  │  Non-overlapping: after match at i,      │
  │  skip to i + m                           │
  └──────────────────────────────────────────┘
```

---

## 7. First / Last Occurrence

```
  Find only the first occurrence → stop after first KMP match
  Find only the last occurrence  → run full KMP, keep last match

  Or: binary search + hashing for specific problems.
```

---

## 8. Cyclic / Circular Pattern Matching

```
  Does P occur in any rotation of T?

  T = "abcde"
  P = "deabc"  →  P is a rotation of T? No, but P can appear in T+T.

  T + T = "abcdeabcde"
  Search for P in T+T → if found and |P| ≤ |T|, it's a cyclic match.

  Algorithm: KMP on T+T with pattern P.
```

---

## 9. 2D Pattern Matching

```
  Text T is a matrix, Pattern P is a smaller matrix.

  T:           P:
  a b c d      b c
  e b c f      b c
  g b c h

  Find P in T → row 0-1, col 1-2

  Approach:
  1. For each row, find all positions where each row of P matches.
  2. Then look for consecutive rows with matching column positions.

  Or: Use Aho-Corasick row-by-row.
```

---

## 10. Longest Repeated Substring

```
  Find the longest substring that appears at least twice.

  s = "banana"

  Substrings appearing 2+ times: "a"(×3), "an"(×2), "ana"(×2), "n"(×2), "na"(×2)
  Longest repeated: "ana" (length 3)

  Approaches:
  ┌──────────────────────────────────────┐
  │ Binary search + hashing: O(n log n) │
  │ Suffix array + LCP:     O(n log n)  │
  │ Suffix tree:            O(n)        │
  └──────────────────────────────────────┘
```

---

## 11. Shortest Unique Substring

```
  Find the shortest substring of s that doesn't appear elsewhere in s.

  s = "aababaa"

  Check length 1: "a" appears many times, "b" appears many times
  Check length 2: "aa", "ab", "ba" — all appear multiple times
  Check length 3: "aab" appears 2×, "aba" appears 2×, "bab" once! ✓

  Approach: Suffix tree or binary search + hashing.
```

---

## 12. Pattern Matching with Don't-Cares

```
  Pattern can contain positions that match any character.

  P = "ab#cd"  (# matches anything)
  T = "abxcdyz"

  Match at index 0: T[0..4] = "abxcd" matches "ab#cd" ✓

  Approach:
  - Break pattern at #s into sub-patterns
  - Match each sub-pattern at expected offset
  - Or use FFT-based convolution (advanced)
```

---

## 13. Problem Variation Decision Tree

```
                        Pattern Matching
                              │
              ┌───────────────┼───────────────┐
              │               │               │
         Single Pattern  Multi Pattern   Approximate
              │               │               │
        ┌─────┼─────┐    Aho-Corasick    DP / Bitap
        │     │     │
      Exact  Count  First/Last
        │     │     │
       KMP   KMP   KMP/Z
       Z-alg  (continue  (stop early/
       R-K    after     track last)
              match)
```

---

## 14. Complexity Overview

| Variation | Best Algorithm | Time |
|-----------|--------------|------|
| Exact single | KMP / Z | O(n + m) |
| Exact multi | Aho-Corasick | O(n + Σmᵢ + z) |
| Approximate (k edits) | DP / Bitap | O(n × m) |
| Wildcard (*,?) | DP | O(n × m) |
| Regex (.,*) | NFA / DP | O(n × m) |
| Overlapping count | KMP | O(n + m) |
| Cyclic match | KMP on T+T | O(n + m) |
| 2D matching | Aho-Corasick rows | O(n×m) text |
| Longest repeated | Suffix Array / Hash | O(n log n) |

---

## 📝 Summary Table

| Variation | Key Idea |
|-----------|----------|
| Single pattern | KMP / Z-Algorithm |
| Multiple patterns | Aho-Corasick (trie automaton) |
| Approximate match | DP with edit distance |
| Wildcard / Regex | DP state transitions |
| Cyclic match | Search P in T + T |
| Overlapping count | Continue KMP after match |
| 2D matching | Row-by-row + column alignment |
| Longest repeated | Binary search + hashing or suffix structures |

---

## ❓ Quick Revision Questions

1. **How do you find a pattern in a circular/rotated string?**
   <details><summary>Answer</summary>Search for P in T + T using KMP. If P appears and |P| ≤ |T|, it's a cyclic match.</details>

2. **What algorithm handles multiple patterns simultaneously?**
   <details><summary>Answer</summary>Aho-Corasick — builds an automaton from all patterns and processes the text in a single pass.</details>

3. **How does overlapping pattern counting differ from non-overlapping?**
   <details><summary>Answer</summary>Overlapping: after a match at i, continue from i+1. Non-overlapping: after match at i, skip to i+m.</details>

4. **What is the approach for pattern matching with wildcards (* and ?)?**
   <details><summary>Answer</summary>Dynamic programming where dp[i][j] represents whether P[0..i-1] matches T[0..j-1], with special transitions for * and ?.</details>

5. **How do you find the longest repeated substring efficiently?**
   <details><summary>Answer</summary>Binary search on the answer length + rolling hash to check if any substring of that length appears twice. O(n log n) total.</details>

---

| [⬅️ Previous: Pattern Matching Applications](03-pattern-matching-applications.md) | [Next Unit: KMP Algorithm ➡️](../03-KMP-Algorithm/01-failure-function-lps-array.md) |
|:---|---:|
