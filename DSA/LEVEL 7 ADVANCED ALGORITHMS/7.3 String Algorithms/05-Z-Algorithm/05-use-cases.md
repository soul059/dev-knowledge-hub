# Chapter 5.5 — Z Algorithm Use Cases & Applications

> **Unit 5: Z Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Beyond simple pattern matching, the Z-array enables elegant solutions for
string periodicity, rotation checks, counting distinct substrings, and more.

---

## 1. String Period Detection

```
  The PERIOD of string S is the smallest p such that
  S[i] == S[i + p] for all valid i.

  Using Z-array:
  If Z[p] == n - p  (suffix starting at p matches prefix of length n-p)
  AND n % p == 0    (length divides evenly)
  Then p is the period.

  Example: S = "abcabcabc"  n = 9
  Z = [9, 0, 0, 6, 0, 0, 3, 0, 0]

  Z[3] = 6 = 9 - 3 ✓, 9 % 3 = 0 ✓ → period = 3
  Z[6] = 3 = 9 - 6 ✓, 9 % 6 ≠ 0 ✗ → not a period

  Period is "abc", repeated 3 times.

  function FindPeriod(S):
      Z = Z_array(S)
      n = len(S)
      for p = 1 to n-1:
          if Z[p] == n - p and n % p == 0:
              return p
      return n   // string is its own period
```

---

## 2. String Rotation Check

```
  Is B a rotation of A?
  
  B is a rotation of A iff B appears in A + A.
  
  Example: A = "abcde", B = "cdeab"
  A + A = "abcdeabcde"
  Does "cdeab" appear? Use Z Algorithm to check!

  S = B + "$" + A + A
    = "cdeab$abcdeabcde"
  
  Compute Z-array. If any Z[i] == len(B), then YES.
  
  Time: O(n)
  Space: O(n)
```

```python
def is_rotation(a, b):
    """Check if b is a rotation of a."""
    if len(a) != len(b):
        return False
    combined = b + "$" + a + a
    z = z_array(combined)
    m = len(b)
    for i in range(m + 1, len(combined)):
        if z[i] == m:
            return True
    return False
```

---

## 3. Longest Common Prefix of Suffix with Prefix

```
  Z[i] directly gives:
  "How long is the common prefix between S and S[i..]?"

  This is useful for:
  1. String comparison queries
  2. Lexicographic comparisons of substrings
  3. Suffix array construction helpers

  Example: S = "abcabcxyz"
  Z = [9, 0, 0, 3, 0, 0, 0, 0, 0]

  LCP(S, S[3..]) = Z[3] = 3   → "abc"
  LCP(S, S[1..]) = Z[1] = 0   → no common prefix
  LCP(S, S[6..]) = Z[6] = 0   → no common prefix
```

---

## 4. Count Distinct Substrings (Iterative Z Approach)

```
  Add characters one by one to build string.
  At each step, determine how many NEW substrings are added.

  When adding character c to existing string S (forming S + c):
  - New string T = reverse(S + c)
  - Compute Z-array of T
  - max_z = max(Z[1], Z[2], ..., Z[len(T)-1])
  - New distinct substrings added = len(S) + 1 - max_z

  Total distinct substrings = sum of all additions.

  This gives O(n²) total — better approaches exist with suffix arrays.
```

---

## 5. Pattern Matching with Wildcards

```
  Pattern with '?' matching any single character.
  
  Approach: Split pattern at wildcards, match each segment with Z.

  P = "ab?cd"  split into: ["ab", "cd"]
  Positions in P:           [0,     3]

  For each segment:
    Use Z to find where it matches in T.
    Verify positional alignment.

  Example:
  P = "a?c"
  T = "abcaxc"

  Segment "a" at P[0]: found at T[0], T[3]
  Segment "c" at P[2]: found at T[2], T[5]

  Check alignment:
    T[0]+2 matches T[2]? "a" at 0, "c" at 2 → T[0..2]="abc" matches ✓
    T[3]+2 matches T[5]? "a" at 3, "c" at 5 → T[3..5]="axc" matches ✓
```

---

## 6. Counting Occurrences of Each Prefix

```
  How many times does each prefix of S appear as a substring?

  Let f[len] = number of times S[0..len-1] appears in S.

  Using Z-array:
  1. Initialize f[i] = 0  for all i
  2. For i = 1 to n-1:
       if Z[i] > 0: f[Z[i]] += 1
  3. For i = n-1 down to 1:
       f[i-1] += f[i]    // prefix of length i-1 appears wherever prefix of length i does
  4. f[i] += 1  for all i (count the prefix itself)

  Example: S = "aaaa"
  Z = [4, 3, 2, 1]
  
  f[3]++ (from Z[1])
  f[2]++ (from Z[2])
  f[1]++ (from Z[3])
  
  After propagation: f = [4, 3, 2, 1]
  Prefix "a" appears 4 times
  Prefix "aa" appears 3 times
  Prefix "aaa" appears 2 times
  Prefix "aaaa" appears 1 time
```

---

## 7. Minimum Rotations for Periodicity Check

```
  Given S, find the minimum number of characters to append
  to make S periodic.

  Using Z-array:
  Find the smallest p such that Z[p] ≥ n - p.
  Characters to append: p - (n % p) if n % p ≠ 0, else 0.

  Example: S = "abcab"  n = 5
  Z = [5, 0, 0, 2, 0]

  p = 3: Z[3] = 2 ≥ 5-3 = 2 ✓
  n % p = 5 % 3 = 2 ≠ 0
  Append: 3 - 2 = 1 character → "abcab" + "c" = "abcabc" 
  Period "abc" verified! ✓
```

---

## 8. Summary of Applications

```
  ┌─────────────────────────────────┬─────────────────────────┐
  │ Application                    │ How Z-array helps        │
  ├─────────────────────────────────┼─────────────────────────┤
  │ Pattern matching               │ Z[i]==m in P$T           │
  │ Period detection               │ Z[p]==n-p, n%p==0        │
  │ Rotation check                 │ Z on B$AA               │
  │ LCP of suffix with prefix      │ Z[i] directly            │
  │ Prefix occurrence count        │ Accumulate Z[i] values   │
  │ Make string periodic           │ Smallest valid p          │
  │ Distinct substrings            │ Incremental Z approach    │
  │ Palindrome-related             │ Z on S + "$" + rev(S)    │
  └─────────────────────────────────┴─────────────────────────┘
```

---

## 📝 Summary Table

| Use Case | Time | Key Technique |
|----------|------|---------------|
| Pattern matching | O(n + m) | Z on P$T |
| Period finding | O(n) | Z[p] == n-p check |
| Rotation check | O(n) | Z on B$AA |
| Prefix counting | O(n) | Z-array + propagation |
| Shortest periodic completion | O(n) | Smallest valid period |

---

## ❓ Quick Revision Questions

1. **How do you check if string B is a rotation of string A using Z?**
   <details><summary>Answer</summary>Compute Z-array of B + "$" + A + A. If any Z[i] == len(B) for i > len(B), then B is a rotation of A.</details>

2. **How does Z[p] == n - p relate to periodicity?**
   <details><summary>Answer</summary>It means S[p..n-1] = S[0..n-p-1], i.e., the string shifted by p positions matches the beginning. If additionally n % p == 0, the string has period p.</details>

3. **What is LCP(S, S[i..]) in terms of the Z-array?**
   <details><summary>Answer</summary>It's exactly Z[i] — the length of the longest common prefix between the whole string and the suffix starting at position i.</details>

4. **How many times does a prefix of length k appear in the string?**
   <details><summary>Answer</summary>Count positions where Z[i] ≥ k (each such position contributes one occurrence), plus 1 for the prefix itself. The propagation method makes this efficient.</details>

5. **What is the time complexity to find the period of a string using Z?**
   <details><summary>Answer</summary>O(n): O(n) to compute the Z-array plus O(n) to scan for the smallest valid p.</details>

---

| [⬅️ Previous: Comparison with KMP](04-comparison-with-kmp.md) | [Next Unit: String Hashing ➡️](../06-String-Hashing/01-polynomial-hash.md) |
|:---|---:|
