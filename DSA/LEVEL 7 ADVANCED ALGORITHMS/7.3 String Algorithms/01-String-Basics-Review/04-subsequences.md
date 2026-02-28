# Chapter 1.4 — Subsequences

> **Unit 1: String Basics Review** | [Course Home](../README.md)

---

## 📋 Chapter Overview

A **subsequence** preserves the relative order of characters but allows gaps.
This concept underpins critical problems like Longest Common Subsequence (LCS),
and understanding it deeply prepares you for the DP chapter (Unit 9).

---

## 1. Definition

```
  🔑  A subsequence of s[0..n-1] is obtained by deleting zero or more
      characters without changing the relative order of the remaining ones.

  s = "abcde"

  Subsequences:                    NOT subsequences:
  ─────────────                    ─────────────────
  "ace"   (indices 0,2,4)   ✓     "eca"  (wrong order)  ✗
  "abd"   (indices 0,1,3)   ✓     "aec"  (wrong order)  ✗
  "abcde" (all indices)      ✓     "af"   ('f' not in s) ✗
  ""      (no indices)       ✓
```

### Visual Representation

```
  s = "a b c d e"
       │   │   │
       └───┼───┘
           │
     subsequence "ace"

  s = "a b c d e"
       │ │   │
       └─┼───┘
         │
     subsequence "abd"
```

---

## 2. Counting Subsequences

### Total Subsequences

Each character is either **included** or **excluded** → 2 choices per char.

$$\text{Total subsequences} = 2^n$$

(including the empty subsequence)

```
  s = "abc",  n = 3

  Choices:     a?    b?    c?    →  Subsequence
  ─────────────────────────────────────────────
               N     N     N        ""
               N     N     Y        "c"
               N     Y     N        "b"
               N     Y     Y        "bc"
               Y     N     N        "a"
               Y     N     Y        "ac"
               Y     Y     N        "ab"
               Y     Y     Y        "abc"
                                    ────
                               Total = 8 = 2³
```

### Distinct Subsequences

If characters repeat, some subsequences may be identical.

```
  s = "aab"

  All subsequences (2³ = 8):
  "", "b", "a", "ab", "a", "ab", "aa", "aab"

  Distinct: "", "a", "b", "aa", "ab", "aab"  →  6

  Formula: Use DP to count, handling duplicates.
```

### DP for Distinct Subsequences Count

```
  dp[i] = number of distinct subsequences using s[0..i-1]

  dp[0] = 1  (empty string)

  For each s[i]:
      dp[i+1] = 2 × dp[i]                    ← double (include/exclude s[i])
      If s[i] appeared before at position j:
          dp[i+1] -= dp[j]                    ← subtract overcounted

  Example: s = "aab"
  ─────────────────
  dp[0] = 1                         {""}
  dp[1] = 2×1 = 2                   {"", "a"}
  dp[2] = 2×2 = 4, minus dp[0]=1    {"", "a", "aa"} = 3
          because 'a' appeared at 0                   wait...

  Let's be precise:
  last['a'] = -1, last['b'] = -1

  i=0 (s[0]='a'): dp[1] = 2×1 = 2.       last['a'] was -1 → no subtract
                   last['a'] = 0
  i=1 (s[1]='a'): dp[2] = 2×2 = 4.       last['a'] was 0 → subtract dp[0]=1 → 3
                   last['a'] = 1
  i=2 (s[2]='b'): dp[3] = 2×3 = 6.       last['b'] was -1 → no subtract
                   last['b'] = 2

  Answer = 6  ✓  {"", "a", "b", "aa", "ab", "aab"}
```

---

## 3. Generating All Subsequences

### Recursive (Backtracking)

```
  GENERATE(s, i, current):
      if i == |s|:
          print current
          return

      // Exclude s[i]
      GENERATE(s, i+1, current)

      // Include s[i]
      GENERATE(s, i+1, current + s[i])

  Call: GENERATE("abc", 0, "")
```

### Recursion Tree for "abc"

```
                          ("", 0)
                        /         \
                   ("", 1)        ("a", 1)
                  /      \        /       \
             ("", 2)  ("b", 2) ("a", 2) ("ab", 2)
             /   \     /   \    /   \     /    \
          "" "c" "b" "bc" "a" "ac" "ab" "abc"
```

**Time**: O(2ⁿ)  |  **Space**: O(n) recursion stack

### Bitmask Method

```
  for mask = 0 to 2ⁿ - 1:
      subseq = ""
      for i = 0 to n-1:
          if mask has bit i set:
              subseq += s[i]
      print subseq

  Example: s = "abc", mask = 5 = 101₂
      bit 0 set → include 'a'
      bit 1 not set → skip 'b'
      bit 2 set → include 'c'
      → "ac"
```

---

## 4. Subsequence Check

**Problem**: Is `t` a subsequence of `s`?

### Two-Pointer Greedy

```
  IS_SUBSEQUENCE(s, t):
      i ← 0    // pointer for s
      j ← 0    // pointer for t
      while i < |s| AND j < |t|:
          if s[i] == t[j]:
              j ← j + 1
          i ← i + 1
      return j == |t|

  Trace: s = "ahbgdc", t = "abc"
  ──────────────────────────────
  i=0: s[0]='a' == t[0]='a' → j=1
  i=1: s[1]='h' ≠  t[1]='b'
  i=2: s[2]='b' == t[1]='b' → j=2
  i=3: s[3]='g' ≠  t[2]='c'
  i=4: s[4]='d' ≠  t[2]='c'
  i=5: s[5]='c' == t[2]='c' → j=3
  j == |t| = 3 → TRUE  ✓
```

**Time**: O(|s|)  |  **Space**: O(1)

---

## 5. Key Subsequence Problems

### 5.1 Longest Common Subsequence (LCS)

```
  s1 = "ABCBDAB"
  s2 = "BDCAB"

  LCS = "BCAB"  (length 4)

  DP Table (Preview — detailed in Unit 9):

        ""  B  D  C  A  B
    ""   0  0  0  0  0  0
     A   0  0  0  0  1  1
     B   0  1  1  1  1  2
     C   0  1  1  2  2  2
     B   0  1  1  2  2  3
     D   0  1  2  2  2  3
     A   0  1  2  2  3  3
     B   0  1  2  2  3  4
```

### 5.2 Longest Increasing Subsequence (LIS)

```
  arr = [10, 9, 2, 5, 3, 7, 101, 18]

  LIS = [2, 3, 7, 18]  or  [2, 5, 7, 18]  (length 4)

  O(n log n) solution uses binary search + patience sorting.
```

### 5.3 Count of Subsequences Equal to Target

```
  s = "rabbbit",  t = "rabbit"

  How many distinct subsequences of s equal t?
  Answer: 3

  This is a classic DP problem (LeetCode 115).
```

---

## 6. Subsequence vs Substring — Comparison

```
  ┌────────────────┬──────────────┬───────────────┐
  │    Property    │  Substring   │  Subsequence  │
  ├────────────────┼──────────────┼───────────────┤
  │ Contiguous     │     Yes      │      No       │
  │ Order kept     │     Yes      │      Yes      │
  │ Count          │   n(n+1)/2   │     2ⁿ        │
  │ Empty included │   Optional   │     Yes       │
  │ Check in       │   O(n×m)     │     O(n)      │
  │ Common problem │ Pattern match│     LCS       │
  └────────────────┴──────────────┴───────────────┘
```

---

## 7. Problem-Solving Strategies

| Goal | Approach | Time |
|------|----------|------|
| Check if t is subseq of s | Two-pointer greedy | O(n) |
| Generate all subsequences | Backtracking or bitmask | O(2ⁿ) |
| Count distinct subsequences | DP with last-occurrence tracking | O(n) |
| Longest common subsequence | 2D DP | O(n × m) |
| Shortest common supersequence | LCS-based DP | O(n × m) |
| Number of subseq matching t | 2D DP | O(n × m) |

---

## 📝 Summary Table

| Concept | Key Point |
|---------|-----------|
| Definition | Maintain order, allow gaps |
| Total count | 2ⁿ (include/exclude each char) |
| Distinct count | DP with duplicate handling |
| Subsequence check | Two-pointer, O(n) |
| Generation | Backtracking O(2ⁿ) or bitmask |
| vs Substring | Substring is contiguous, subseq is not |
| LCS | DP O(n×m) — central string DP problem |
| LIS | O(n log n) with binary search |

---

## ❓ Quick Revision Questions

1. **How many subsequences does a string of length 8 have?**
   <details><summary>Answer</summary>2⁸ = 256 (including the empty subsequence).</details>

2. **Is "aec" a subsequence of "abcde"?**
   <details><summary>Answer</summary>No — 'e' appears before 'c' in "aec" but after 'c' in "abcde". Wait, actually: a(0), e(4), c(2) — 'c' at index 2 < 'e' at index 4, so we'd need c before e. In "aec", 'e' comes before 'c', so we need e before c in s, but e is at 4 and c is at 2. So NO, "aec" is NOT a subsequence of "abcde".</details>

3. **What is the greedy approach to check subsequence membership?**
   <details><summary>Answer</summary>Use two pointers — scan s left to right, advance the t-pointer whenever characters match. If the t-pointer reaches the end, t is a subsequence.</details>

4. **How do you handle duplicate characters when counting distinct subsequences?**
   <details><summary>Answer</summary>Track the last occurrence of each character. When a character repeats, subtract dp[last_occurrence] to remove overcounted subsequences.</details>

5. **What is the relationship between LCS and the edit distance problem?**
   <details><summary>Answer</summary>Edit distance (only insertions and deletions) = |s1| + |s2| - 2 × LCS(s1, s2).</details>

---

| [⬅️ Previous: Substrings](03-substrings.md) | [Next: Anagrams ➡️](05-anagrams.md) |
|:---|---:|
