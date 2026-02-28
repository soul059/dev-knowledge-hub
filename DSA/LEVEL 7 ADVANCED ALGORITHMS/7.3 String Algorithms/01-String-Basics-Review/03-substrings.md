# Chapter 1.3 — Substrings

> **Unit 1: String Basics Review** | [Course Home](../README.md)

---

## 📋 Chapter Overview

A **substring** is a contiguous block of characters within a string. This
concept appears in almost every string algorithm — pattern matching, hashing,
sliding window, and DP. Understanding how to enumerate, count, and reason
about substrings is essential.

---

## 1. Definition

```
  🔑  A substring of s[0..n-1] is s[i..j] where 0 ≤ i ≤ j ≤ n-1.
      It includes characters at positions i, i+1, ..., j.

  Example:  s = "algorithm"

            Substrings of length 3:
            s[0..2] = "alg"
            s[1..3] = "lgo"
            s[2..4] = "gor"
            s[3..5] = "ori"
            s[4..6] = "rit"
            s[5..7] = "ith"
            s[6..8] = "thm"
```

### Substring vs Subsequence

```
  s = "abcde"

  Substring:     "bcd"  ✓  (contiguous)
                 "ace"  ✗  (not contiguous → subsequence)

  Subsequence:   "ace"  ✓  (maintain relative order, skip chars)
                 "eca"  ✗  (wrong order)
```

---

## 2. Counting Substrings

### Total Substrings

```
  For a string of length n:

  Length 1: n substrings      (each character)
  Length 2: n-1 substrings
  Length 3: n-2 substrings
  ...
  Length n: 1 substring       (the whole string)

  Total = n + (n-1) + (n-2) + ... + 1
        = n(n+1) / 2

  Including the empty substring: n(n+1)/2 + 1
```

### Visualization for n = 4, s = "abcd"

```
  Length 1: a   b   c   d           → 4
  Length 2: ab  bc  cd              → 3
  Length 3: abc bcd                 → 2
  Length 4: abcd                    → 1
                                     ──
                              Total = 10 = 4×5/2
```

### 🔑 Formula

$$\text{Number of substrings} = \frac{n(n+1)}{2}$$

This is **O(n²)** — important for understanding why brute-force substring
algorithms are often quadratic.

---

## 3. Enumerating All Substrings

### Pseudocode

```
ENUMERATE_SUBSTRINGS(s, n):
    for i = 0 to n-1:           // start index
        for j = i to n-1:       // end index
            print s[i..j]
```

### Trace for s = "abc"

```
  i=0: j=0 → "a"
       j=1 → "ab"
       j=2 → "abc"
  i=1: j=1 → "b"
       j=2 → "bc"
  i=2: j=2 → "c"

  Total = 6 = 3×4/2  ✓
```

**Time**: O(n²) to enumerate, O(n³) if printing each substring  
**Space**: O(1) extra (just indices)

---

## 4. Distinct Substrings

The formula n(n+1)/2 counts with **repetition**. For distinct substrings,
duplicates must be removed.

### Example

```
  s = "aab"

  All substrings: "a", "a", "aa", "ab", "aab", "b"
  Distinct:       "a", "aa", "ab", "aab", "b"

  Total = 6,  Distinct = 5
```

### Methods to Count Distinct Substrings

| Method | Time | Space | Notes |
|--------|------|-------|-------|
| HashSet of all substrings | O(n³) | O(n³) | Brute force |
| Trie insertion | O(n²) | O(n²) | Each new node = new substring |
| Suffix Array + LCP | O(n log n) | O(n) | Advanced technique |

### Trie-Based Approach

```
  s = "aab"

  Insert all suffixes:
    "aab"  →  a → a → b
    "ab"   →  a → b
    "b"    →  b

  Trie:
         (root)
         /    \
        a      b ←── new node
       / \
      a   b  ←── new node
      |
      b  ←── new node

  Count of nodes (excluding root) = 5 = number of distinct substrings ✓
```

### 🔑 Suffix Array Formula

$$\text{Distinct substrings} = \frac{n(n+1)}{2} - \sum_{i} \text{LCP}[i]$$

Where LCP[i] is the Longest Common Prefix between adjacent suffixes in sorted
order. (Covered in Unit 10.)

---

## 5. Special Substring Problems

### 5.1 Longest Substring Without Repeating Characters

```
  s = "abcabcbb"

  Sliding Window Approach:
  ────────────────────────
  Use a set to track characters in current window [left, right].

  Step:  left  right  window     set            result
  ─────────────────────────────────────────────────────
    1     0     0     "a"       {a}              1
    2     0     1     "ab"      {a,b}            2
    3     0     2     "abc"     {a,b,c}          3
    4     1     3     "bca"     {b,c,a}          3   (removed 'a' at 0)
    5     2     4     "cab"     {c,a,b}          3
    6     3     5     "abc"     {a,b,c}          3
    7     4     6     "bcb"     → shrink left
    8     5     6     "cb"      {c,b}            3
    9     5     7     "cbb"     → shrink left
   10     7     7     "b"       {b}              3

  Answer: 3  ("abc")
```

**Time**: O(n) — each character enters and leaves the window at most once.

### 5.2 Smallest Substring Containing All Characters

```
  s = "ADOBECODEBANC",  target = "ABC"

  Sliding Window with frequency matching:
  ─ Expand right until all target chars are covered
  ─ Shrink left to minimize window
  ─ Track minimum window seen

  Answer: "BANC" (length 4)
```

### 5.3 Count Substrings with Exactly K Distinct Characters

```
  Trick: atMost(k) - atMost(k-1) = exactly(k)

  Use sliding window for atMost(k):
    Expand right; if distinct > k, shrink left.
    Count += (right - left + 1) at each step.
```

---

## 6. Prefix and Suffix (Special Substrings)

```
  s = "abcde"

  Prefixes:             Suffixes:
  ─────────             ─────────
  ""        (empty)     ""        (empty)
  "a"                   "e"
  "ab"                  "de"
  "abc"                 "cde"
  "abcd"                "bcde"
  "abcde"   (full)      "abcde"   (full)

  ┌───┬───┬───┬───┬───┐
  │ a │ b │ c │ d │ e │
  └───┴───┴───┴───┴───┘
   ◄─── prefix ───►
            ◄── suffix ──►
```

### Proper Prefix/Suffix

A **proper** prefix/suffix excludes the full string itself.

```
  s = "abab"

  Proper prefixes:  "", "a", "ab", "aba"
  Proper suffixes:  "", "b", "ab", "bab"

  Common:  "", "ab"
  Longest proper prefix that is also a suffix = "ab"   ← This is the LPS!
```

💡 This concept is the foundation of the **KMP algorithm** (Unit 3).

---

## 7. Substring Search Problem

Given text `T` and pattern `P`, find all occurrences of `P` in `T`.

```
  T = "aabaacaadaabaaba"
  P = "aaba"

  Occurrences at indices: 0, 9, 12

  T: a a b a a c a a d a a b a a b a
     ─────           ─────   ─────
     0               9       12
```

This is the **central problem** of Units 2–5.

---

## 8. Problem-Solving Strategies

| Problem Type | Approach |
|---|---|
| All substrings of length k | Sliding window |
| Longest substring with property X | Two pointers / sliding window |
| Count distinct substrings | Trie or suffix array |
| Find pattern in text | KMP / Rabin-Karp / Z-algorithm |
| Substring with frequency constraint | Sliding window + freq array |
| Shortest substring covering set | Sliding window (min-window) |

---

## 📝 Summary Table

| Concept | Key Point |
|---------|-----------|
| Substring | Contiguous block s[i..j] |
| Count | n(n+1)/2 total substrings |
| Distinct count | Use Trie or Suffix Array |
| Prefix | s[0..k] for some k |
| Suffix | s[k..n-1] for some k |
| LPS | Longest proper prefix = suffix |
| Sliding window | O(n) for fixed-window problems |
| Enumeration | O(n²) pairs, O(n³) with output |

---

## ❓ Quick Revision Questions

1. **How many substrings does a string of length 10 have?**
   <details><summary>Answer</summary>10 × 11 / 2 = 55 substrings.</details>

2. **What is the difference between a substring and a subsequence?**
   <details><summary>Answer</summary>A substring must be contiguous; a subsequence can skip characters but must maintain relative order.</details>

3. **How can you count distinct substrings efficiently?**
   <details><summary>Answer</summary>Insert all suffixes into a Trie — each node represents a distinct substring. Or use suffix array + LCP array.</details>

4. **What is a proper prefix?**
   <details><summary>Answer</summary>A prefix that is not the entire string itself (i.e., strictly shorter).</details>

5. **What sliding window trick gives "exactly k distinct" from "at most k"?**
   <details><summary>Answer</summary>exactly(k) = atMost(k) - atMost(k - 1).</details>

6. **Why is brute-force substring enumeration O(n³)?**
   <details><summary>Answer</summary>O(n²) pairs × O(n) to copy/print each substring.</details>

---

| [⬅️ Previous: String Operations](02-string-operations.md) | [Next: Subsequences ➡️](04-subsequences.md) |
|:---|---:|
