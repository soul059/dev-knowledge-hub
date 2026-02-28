# Chapter 1.6 — Palindromes

> **Unit 1: String Basics Review** | [Course Home](../README.md)

---

## 📋 Chapter Overview

A **palindrome** reads the same forwards and backwards. Palindrome problems
appear everywhere — from brute-force checks to Manacher's algorithm (Unit 8).
This chapter builds the fundamental intuition and covers the key problem types.

---

## 1. Definition

```
  🔑  A string s is a palindrome if s == reverse(s).

  "racecar"   →  "racecar"   ✓  Palindrome
  "level"     →  "level"     ✓  Palindrome
  "hello"     →  "olleh"     ✗  Not palindrome
  "a"         →  "a"         ✓  Palindrome (single char)
  ""          →  ""          ✓  Palindrome (empty string)
```

### Structure of a Palindrome

```
  Odd length:     r a c e c a r
                  ├───────┤
                  mirror ─┼── mirror
                          │
                       center

  Even length:    a b b a
                  ├───┤
                 mirror ── mirror
                     │
                  (no single center)

  Key property:  s[i] == s[n - 1 - i]  for all  0 ≤ i < n/2
```

---

## 2. Palindrome Check

### Method 1: Two Pointers (Optimal)

```
  IS_PALINDROME(s):
      left ← 0
      right ← |s| - 1
      while left < right:
          if s[left] ≠ s[right]:
              return false
          left ← left + 1
          right ← right - 1
      return true

  Trace: s = "racecar"
  ─────────────────────
      left=0, right=6:  r == r  ✓
      left=1, right=5:  a == a  ✓
      left=2, right=4:  c == c  ✓
      left=3, right=3:  left ≮ right → stop

  Result: true  ✓
```

**Time**: O(n/2) = O(n)  |  **Space**: O(1)

### Method 2: Compare with Reverse

```
  return s == reverse(s)

  Time:  O(n)
  Space: O(n)  (for the reversed copy)
```

---

## 3. Palindromic Substrings

### 3.1 Count All Palindromic Substrings

**Expand Around Center** approach:

```
  For each possible center position:
    - Odd length:   center at each index i  (n centers)
    - Even length:  center between i and i+1  (n-1 centers)
    Total: 2n - 1 centers

  EXPAND(s, left, right):
      count ← 0
      while left ≥ 0 AND right < |s| AND s[left] == s[right]:
          count += 1
          left -= 1
          right += 1
      return count

  COUNT_PALINDROMES(s):
      total ← 0
      for i = 0 to |s| - 1:
          total += EXPAND(s, i, i)      // odd-length
          total += EXPAND(s, i, i+1)    // even-length
      return total
```

### Trace: s = "aaba"

```
  Centers and expansions:

  Center i=0 (odd):  "a"           → 1 palindrome
  Center i=0 (even): "aa" → s[0]==s[1] ✓ → 1 palindrome
                      "—aa—" → out of bounds → stop

  Center i=1 (odd):  "a"           → 1 palindrome
                     "aab" → s[0]≠s[2] → stop
  Center i=1 (even): "ab" → s[1]≠s[2] → 0

  Center i=2 (odd):  "b"           → 1 palindrome
  Center i=2 (even): "ba" → s[2]≠s[3] → 0

  Center i=3 (odd):  "a"           → 1 palindrome

  Total = 1 + 1 + 1 + 0 + 1 + 0 + 1 = 5

  Palindromic substrings: "a", "a", "aa", "b", "a"
  (with "a" appearing at indices 0, 1, 3)
```

**Time**: O(n²) worst case  |  **Space**: O(1)

### 3.2 Longest Palindromic Substring

Same expand-around-center, but track the longest:

```
  LONGEST_PALINDROME(s):
      start ← 0,  maxLen ← 1
      for i = 0 to |s| - 1:
          // Odd expansion
          len1 ← expandLength(s, i, i)
          // Even expansion
          len2 ← expandLength(s, i, i+1)
          len ← max(len1, len2)
          if len > maxLen:
              maxLen ← len
              start ← i - (len - 1) / 2
      return s[start .. start + maxLen - 1]
```

**Time**: O(n²)  |  O(n) with Manacher's Algorithm (Unit 8)

---

## 4. Palindrome Patterns

### 4.1 Palindrome Number Check

```
  n = 12321

  Reverse the number:
    12321 → 12321  (same → palindrome)

  n = 12345
    12345 → 54321  (different → not palindrome)

  Trick: reverse only half the digits to avoid overflow.
```

### 4.2 Valid Palindrome (with non-alphanumeric chars)

```
  s = "A man, a plan, a canal: Panama"

  Step 1: Filter to alphanumeric only → "amanaplanacanalpanama"
  Step 2: Convert to lowercase
  Step 3: Two-pointer check → palindrome ✓

  Or: use two pointers that skip non-alphanumeric characters.
```

### 4.3 Palindrome Partitioning

```
  s = "aab"

  Find all ways to partition s into palindromic substrings:
    ["a", "a", "b"]
    ["aa", "b"]

  Approach: Backtracking
  At each position, try every prefix that is a palindrome,
  then recurse on the remainder.

  Optimization: precompute isPalin[i][j] using DP.
```

### 4.4 Minimum Insertions to Make Palindrome

```
  s = "abca"

  LPS (Longest Palindromic Subsequence) of "abca" = "aca" (length 3)

  Minimum insertions = |s| - LPS = 4 - 3 = 1

  Result: "abcba" (insert 'b' at position 3)

  🔑  min_insertions = n - LPS(s)
      where LPS(s) = LCS(s, reverse(s))
```

---

## 5. Palindromic Subsequences

### Longest Palindromic Subsequence (LPS)

```
  s = "bbbab"

  LPS = "bbbb" (length 4)

  Method: LPS(s) = LCS(s, reverse(s))

  s = "bbbab"
  r = "babbb"

  LCS("bbbab", "babbb") = "bbbb" → length 4
```

### DP for LPS

```
  dp[i][j] = length of LPS in s[i..j]

  Base: dp[i][i] = 1 for all i

  Recurrence:
    if s[i] == s[j]:
        dp[i][j] = dp[i+1][j-1] + 2
    else:
        dp[i][j] = max(dp[i+1][j], dp[i][j-1])

  Fill diagonally (increasing gap = j - i).
```

---

## 6. Visualization of Palindrome Structures

```
  Word: "abacaba"

  Mirror structure:
  ─────────────────
  Position: 0  1  2  3  4  5  6
  Char:     a  b  a  c  a  b  a
            │  │  │  │  │  │  │
            └──┼──┼──┼──┼──┼──┘   s[0] == s[6]
               └──┼──┼──┼──┘      s[1] == s[5]
                  └──┼──┘         s[2] == s[4]
                     │            center at s[3]

  Nested palindrome structure:
  ┌─────────────────────────────┐
  │ a ┌───────────────────┐ a  │
  │   │ b ┌───────────┐ b │    │
  │   │   │ a ┌───┐ a │   │    │
  │   │   │   │ c │   │   │    │
  │   │   │   └───┘   │   │    │
  │   │   └───────────┘   │    │
  │   └───────────────────┘    │
  └─────────────────────────────┘
```

---

## 7. Problem-Solving Strategies

| Problem | Key Technique | Time |
|---------|---------------|------|
| Is palindrome? | Two pointers | O(n) |
| Longest palindromic substring | Expand around center / Manacher | O(n²) / O(n) |
| Count palindromic substrings | Expand around center | O(n²) |
| Longest palindromic subsequence | DP or LCS(s, rev(s)) | O(n²) |
| Min insertions for palindrome | n - LPS(s) | O(n²) |
| Palindrome partitioning | Backtracking + DP precompute | O(2ⁿ) |
| Min cuts for palindrome partition | DP | O(n²) |

---

## 📝 Summary Table

| Concept | Key Point |
|---------|-----------|
| Definition | s == reverse(s) |
| Check | Two pointers from both ends — O(n) |
| Central property | s[i] == s[n-1-i] for all i |
| Expand around center | 2n-1 centers: n odd + n-1 even |
| LPS (subsequence) | LCS(s, reverse(s)) |
| Min insertions | n - LPS |
| Manacher's | O(n) for longest palindromic substring |

---

## ❓ Quick Revision Questions

1. **How many possible centers exist for palindromic expansion in a string of length n?**
   <details><summary>Answer</summary>2n - 1 centers: n for odd-length palindromes and n-1 for even-length palindromes.</details>

2. **What is the relationship between LPS and minimum insertions?**
   <details><summary>Answer</summary>Minimum insertions to make a string palindrome = n - LPS(s), where LPS is the longest palindromic subsequence.</details>

3. **How do you compute the Longest Palindromic Subsequence?**
   <details><summary>Answer</summary>LPS(s) = LCS(s, reverse(s)) — reduce to Longest Common Subsequence with the reversed string.</details>

4. **What is the time complexity of expand-around-center for finding the longest palindromic substring?**
   <details><summary>Answer</summary>O(n²) — for each of 2n-1 centers, expansion can take up to O(n).</details>

5. **How do you check a palindrome if the string contains spaces and punctuation?**
   <details><summary>Answer</summary>Use two pointers that skip non-alphanumeric characters and compare case-insensitively.</details>

---

| [⬅️ Previous: Anagrams](05-anagrams.md) | [Next Unit: Pattern Matching ➡️](../02-Pattern-Matching/01-brute-force-approach.md) |
|:---|---:|
