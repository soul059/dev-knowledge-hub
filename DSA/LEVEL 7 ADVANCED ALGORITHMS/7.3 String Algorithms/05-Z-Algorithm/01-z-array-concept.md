# Chapter 5.1 — Z-Array Concept

> **Unit 5: Z Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

The **Z-array** is the foundation of the Z Algorithm. For each position i in a
string, Z[i] tells the length of the longest substring starting at i that
matches a prefix of the string. This simple concept leads to a powerful O(n)
pattern matching algorithm.

---

## 1. Definition

```
  Given string S[0..n-1]:

  Z[i] = length of the longest substring starting at S[i]
          that is also a PREFIX of S

  Formally:
  Z[i] = max { k : S[0..k-1] == S[i..i+k-1] }

  Convention: Z[0] is undefined (or set to 0 or n).
```

---

## 2. Visual Example

```
  S = "aabxaabxaaz"
       0123456789 10

  Position 0: Z[0] = — (undefined, entire string is trivially a prefix)

  Position 1: S[1..] = "abxaabxaaz"
              S[0..] = "aabxaabxaaz"
              a ≠ a? → a == a ✓
              b ≠ a? → b ≠ a ✗
              Z[1] = 1

  Position 2: S[2..] = "bxaabxaaz"
              S[0..] = "aabxaabxaaz"
              b ≠ a ✗
              Z[2] = 0

  Position 3: S[3..] = "xaabxaaz"
              x ≠ a ✗
              Z[3] = 0

  Position 4: S[4..] = "aabxaaz"
              S[0..] = "aabxaab..."
              a==a ✓ a==a ✓ b==b ✓ x==x ✓ a==a ✓ a==a ✓ z≠b ✗
              Z[4] = 6

  Position 5: Z[5] = 1  (a==a, then b≠a)
  Position 6: Z[6] = 0  (b≠a)
  Position 7: Z[7] = 0  (x≠a)
  Position 8: Z[8] = 2  (aa prefix)
  Position 9: Z[9] = 1  (a prefix)
  Position 10: Z[10] = 0 (z≠a)

  ┌─────┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┐
  │  i  │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │ 6 │ 7 │ 8 │ 9 │10 │
  ├─────┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
  │ S[i]│ a │ a │ b │ x │ a │ a │ b │ x │ a │ a │ z │
  ├─────┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┼───┤
  │ Z[i]│ — │ 1 │ 0 │ 0 │ 6 │ 1 │ 0 │ 0 │ 2 │ 1 │ 0 │
  └─────┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┴───┘
```

---

## 3. Z-Box Concept

```
  If Z[i] > 0, the interval [i, i + Z[i] - 1] is called a "Z-box".
  This means S[i..i+Z[i]-1] matches S[0..Z[i]-1].

  For our example:
  S = a a b x a a b x a a z
      0 1 2 3 4 5 6 7 8 9 10

  Z-boxes:
    i=1: [1, 1]   — "a" matches prefix "a"
    i=4: [4, 9]   — "aabxaa" matches prefix "aabxaa"
    i=5: [5, 5]   — "a" matches prefix "a"
    i=8: [8, 9]   — "aa" matches prefix "aa"
    i=9: [9, 9]   — "a" matches prefix "a"

  The Z Algorithm tracks the rightmost Z-box [L, R].
  L = left boundary, R = right boundary of the Z-box
  that extends the farthest to the right.
```

---

## 4. The L, R Window

```
  Key invariant: [L, R] is the Z-box with the largest R seen so far.

  ┌──────────────────────────────────────────────────────┐
  │  The window [L, R] represents known information:    │
  │  S[L..R] = S[0..R-L]                               │
  │                                                      │
  │  When computing Z[i]:                                │
  │  • If i > R: no useful info → compute from scratch   │
  │  • If i ≤ R: we can REUSE previously computed Z vals │
  └──────────────────────────────────────────────────────┘

  This reuse is what makes the algorithm O(n)!
```

---

## 5. Why Z-Array Is Useful

```
  ┌────────────────────────────────────────────────────┐
  │  Application 1: PATTERN MATCHING                   │
  │  Concatenate: P + "$" + T                          │
  │  Compute Z-array of this combined string           │
  │  Any Z[i] == m means pattern found!                │
  │                                                    │
  │  Application 2: Number of distinct substrings      │
  │                                                    │
  │  Application 3: String compression                 │
  │  If Z[i] = n - i, then S has period i              │
  │                                                    │
  │  Application 4: Longest common prefix queries      │
  └────────────────────────────────────────────────────┘
```

---

## 6. Naive Z-Array Construction

```
  function Z_naive(S):
      n = len(S)
      Z = array of size n
      Z[0] = 0  (or n, or undefined)

      for i = 1 to n-1:
          Z[i] = 0
          while i + Z[i] < n AND S[Z[i]] == S[i + Z[i]]:
              Z[i] += 1

      return Z

  Time: O(n²) worst case
  Example worst case: S = "aaaaaaa"
    Z = [—, 6, 5, 4, 3, 2, 1]
    Total comparisons: 6+5+4+3+2+1 = 21 = O(n²)
```

---

## 7. Z-Array vs. Other Arrays

```
  ┌──────────────┬───────────────────────────────────────────┐
  │ Array        │ What it stores                            │
  ├──────────────┼───────────────────────────────────────────┤
  │ Z-array      │ Length of match with PREFIX at each pos   │
  │ LPS (KMP)    │ Length of longest proper prefix=suffix    │
  │ Suffix Array │ Sorted order of all suffixes              │
  └──────────────┴───────────────────────────────────────────┘

  Key difference:
  - Z[i]: How much does S[i..] match S[0..]?  (prefix matching)
  - LPS[i]: What is the longest prefix of P[0..i] that equals
             a suffix of P[0..i]?  (prefix-suffix matching)

  Both can be used for pattern matching, but:
  - Z-array is often simpler to understand
  - KMP's LPS enables online processing (character by character)
```

---

## 📝 Summary Table

| Property | Value |
|----------|-------|
| Z[i] definition | Length of longest prefix of S matching S[i..] |
| Z[0] | Undefined (or n) |
| Z-box | Interval [i, i+Z[i]-1] where Z[i] > 0 |
| L, R window | Rightmost Z-box seen so far |
| Naive construction | O(n²) |
| Efficient construction | O(n) — next chapter |
| Key insight | Reuse known information via [L, R] window |

---

## ❓ Quick Revision Questions

1. **What does Z[i] = 5 mean?**
   <details><summary>Answer</summary>The substring S[i..i+4] (5 characters) matches the prefix S[0..4]. That is, the first 5 characters of the suffix starting at position i are the same as the first 5 characters of the whole string.</details>

2. **What is a Z-box?**
   <details><summary>Answer</summary>When Z[i] > 0, the interval [i, i+Z[i]-1] is a Z-box, representing a substring that matches a prefix of the string.</details>

3. **Why is Z[0] typically undefined?**
   <details><summary>Answer</summary>The entire string trivially matches itself as a prefix. Setting Z[0] = n would be correct but unhelpful, so it's conventionally left undefined or set to 0.</details>

4. **What is the time complexity of naive Z-array construction?**
   <details><summary>Answer</summary>O(n²). For strings like "aaaa...a", every position requires comparisons proportional to the remaining length.</details>

5. **How is the Z-array used for pattern matching conceptually?**
   <details><summary>Answer</summary>Concatenate P + "$" + T to form string S. Compute Z-array of S. Any position i where Z[i] == len(P) indicates the pattern starts at position i - len(P) - 1 in the text.</details>

---

| [⬅️ Previous Unit: Rabin-Karp Implementation](../04-Rabin-Karp-Algorithm/06-implementation.md) | [Next: Z-Array Construction ➡️](02-z-array-construction.md) |
|:---|---:|
