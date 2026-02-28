# Chapter 3.5 — Applications of KMP

> **Unit 3: KMP Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

KMP is not just a pattern matching algorithm — the LPS array unlocks solutions
to many string problems. This chapter covers the most important applications.

---

## 1. Pattern Search (Core Use)

```
  Find all occurrences of P in T.
  Time: O(n + m)

  This is the primary use case covered in Chapter 3.3.
```

---

## 2. String Period / Shortest Repeating Unit

```
  🔑 If the pattern length m is divisible by (m - LPS[m-1]),
     then (m - LPS[m-1]) is the length of the shortest period.

  P = "abcabcabc"   m = 9
  LPS = [0,0,0,1,2,3,4,5,6]
  LPS[8] = 6

  Period = m - LPS[m-1] = 9 - 6 = 3
  9 % 3 == 0 → "abc" is the repeating unit ✓

  P = "abcabcabc" = "abc" × 3

  ┌──────────────────────────────────────────┐
  │  Formula:                                │
  │  period = m - LPS[m - 1]                │
  │                                          │
  │  If m % period == 0:                     │
  │    P = P[0..period-1] repeated m/period  │
  │    times.                                │
  │  Else:                                   │
  │    P is not fully periodic, but period   │
  │    is the shortest "approximate" period. │
  └──────────────────────────────────────────┘
```

### Examples

```
  "abab"  → LPS[3]=2 → period=2 → 4%2=0 → "ab"×2  ✓
  "aaaa"  → LPS[3]=3 → period=1 → 4%1=0 → "a"×4   ✓
  "abcab" → LPS[4]=2 → period=3 → 5%3≠0 → not fully periodic
```

---

## 3. Counting Pattern Occurrences

```
  Count overlapping occurrences of P in T.

  Just run KMP and count every time j reaches m.

  Example: T = "aaaa", P = "aa"
  Overlapping count = 3 (at positions 0, 1, 2)
```

---

## 4. Check if String is a Rotation

```
  Is t a rotation of s?

  t is a rotation of s  ⟺  |s| == |t| AND t appears in s + s

  Use KMP to search for t in (s + s).

  s = "abcde", t = "cdeab"
  s + s = "abcdeabcde"
  KMP finds "cdeab" in "abcdeabcde" → YES ✓

  Time: O(n)
```

---

## 5. Longest Prefix That Is Also Suffix

```
  Direct application of the LPS array!

  s = "ABCAB"
  LPS = [0, 0, 0, 1, 2]

  Longest proper prefix = suffix: "AB" (length 2)

  All prefix-suffixes (using fallback chain):
  LPS[4] = 2 → "AB"
  LPS[1] = 0 → ""

  Chain: 2 → 0  →  prefix-suffixes of lengths: {2, 0}
```

---

## 6. Minimum Characters to Add for Palindrome

```
  Problem: What is the minimum number of characters to add at the
           END of s to make it a palindrome?

  Approach:
  1. Create: combined = s + "#" + reverse(s)
  2. Build LPS of combined
  3. Answer = n - LPS[last]

  Example: s = "aacecaaa"
  reverse(s) = "aaacecaa"
  combined = "aacecaaa#aaacecaa"

  LPS of combined → LPS[last] = 7

  Answer = 8 - 7 = 1  (add "a" at the end → "aacecaaaa"? No...)

  Actually, minimum chars to add at the FRONT:
  combined = reverse(s) + "#" + s
  LPS[last] = longest suffix of s that is a palindrome starting at index 0
  Add (n - LPS[last]) chars from reverse(s) at the front.

  For "aacecaaa":
  combined = "aaacecaa#aacecaaa"
  LPS[last] tells us the longest palindromic prefix.
```

---

## 7. String Matching with Multiple Texts

```
  If you have one pattern P and multiple texts T₁, T₂, ..., Tₖ:

  Build LPS once for P → O(m)
  Run KMP on each Tᵢ → O(nᵢ) each

  Total: O(m + Σnᵢ)

  Much better than Aho-Corasick when there's only one pattern.
```

---

## 8. KMP for Automaton Construction

```
  The LPS array defines a finite automaton that can be used for:
  - Streaming pattern matching (process one char at a time)
  - Counting patterns in DP on strings
  - String matching in matrix exponentiation problems

  State = j (number of chars matched)
  Transition:
    On char c:  while j > 0 and P[j] ≠ c: j = LPS[j-1]
                if P[j] == c: j++
                new state = j
```

---

## 9. Application Summary

| Application | How KMP Helps | Time |
|-------------|---------------|------|
| Pattern search | Core KMP | O(n + m) |
| String period | m - LPS[m-1] | O(m) |
| Rotation check | KMP on s+s | O(n) |
| Longest prefix=suffix | LPS[m-1] | O(m) |
| Count occurrences | Count j==m events | O(n + m) |
| Min chars for palindrome | LPS on composite string | O(n) |
| Streaming matching | KMP automaton | O(1) per char |

---

## 📝 Summary Table

| Concept | Key Point |
|---------|-----------|
| Period detection | period = m - LPS[m-1] |
| Rotation check | Search t in s+s |
| Prefix = suffix | Read directly from LPS |
| Palindrome prefix | Use KMP on reverse(s) + "#" + s |
| Automaton | LPS defines state transitions |

---

## ❓ Quick Revision Questions

1. **How do you find the shortest repeating unit of a string using LPS?**
   <details><summary>Answer</summary>period = m - LPS[m-1]. If m % period == 0, the repeating unit is P[0..period-1].</details>

2. **How do you check if string t is a rotation of string s?**
   <details><summary>Answer</summary>Check if |s| == |t| and use KMP to search for t in s + s.</details>

3. **What does LPS[m-1] represent for the entire pattern?**
   <details><summary>Answer</summary>The length of the longest proper prefix of P that is also a suffix of P.</details>

4. **How can KMP help find the shortest palindrome by adding characters?**
   <details><summary>Answer</summary>Build KMP on reverse(s) + "#" + s. The LPS value at the end gives the longest palindromic prefix, and n - LPS[end] characters need to be prepended.</details>

5. **What is the advantage of building a KMP automaton?**
   <details><summary>Answer</summary>It allows processing characters one at a time in a streaming fashion, useful for DP problems and real-time matching.</details>

---

| [⬅️ Previous: Time Complexity](04-time-complexity.md) | [Next: Step-by-Step Example ➡️](06-step-by-step-example.md) |
|:---|---:|
