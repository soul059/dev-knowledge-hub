# Chapter 6: Shortest Common Supersequence

## 📋 Overview
**Shortest Common Supersequence** (LeetCode #1092) finds the shortest string that has both `s1` and `s2` as subsequences. It elegantly builds on the **LCS** algorithm — the SCS length equals `m + n - LCS(s1, s2)`, and reconstruction requires backtracking through the LCS table.

---

## 🧠 Core Concept

```
s1 = "abac"
s2 = "cab"

SCS must contain:
  - s1 as subsequence: a_b_a_c → chars a,b,a,c in order
  - s2 as subsequence: c_a_b   → chars c,a,b in order

One valid SCS: "cabac" (length 5)
  s1 in "cabac": _a_ac → "a_ac"... actually "abac":
    c[a]b[a][c] → a,a,c... missing b
  
  Let me find properly:
  LCS("abac", "cab") = "ab" (length 2)
  SCS length = 4 + 3 - 2 = 5
  
  SCS = "cabac"
  Verify: s1 = [c]a[b]ac → a,b,a,c... 
  c-a-b-a-c: s1="abac" → positions 1,2,3,4 ✓ (a,b,a,c)
             s2="cab"  → positions 0,1,2 ✓ (c,a,b)

Formula: SCS_length = len(s1) + len(s2) - LCS_length
```

---

## 🔍 Why SCS = m + n - LCS?

```
Key insight: LCS characters are shared — include them ONCE

s1 = "AGGTAB"
s2 = "GXTXAYB"
LCS = "GTAB" (length 4)

Building SCS:
  Walk both strings, merge using LCS as anchor:

  s1: A G G T A B
  s2: G X T X A Y B
  LCS:  G   T   A   B  (shared characters, included once)

  SCS: A G X G T X A Y B  (length 9)
       = 6 + 7 - 4 = 9 ✓

  Non-LCS from s1: A, G  (2 chars)
  Non-LCS from s2: X, X, Y  (3 chars)
  LCS chars: G, T, A, B  (4 chars)
  Total: 2 + 3 + 4 = 9 ✓
```

---

## 🔨 Solution: LCS + Reconstruction

```
function shortestCommonSupersequence(s1, s2):
    m = len(s1), n = len(s2)
    
    // Step 1: Build LCS dp table
    dp = 2D array (m+1) × (n+1), filled with 0
    for i = 1 to m:
        for j = 1 to n:
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    
    // Step 2: Backtrack to build SCS
    result = ""
    i = m, j = n
    
    while i > 0 and j > 0:
        if s1[i-1] == s2[j-1]:
            result = s1[i-1] + result    // LCS char: include once
            i -= 1; j -= 1
        else if dp[i-1][j] > dp[i][j-1]:
            result = s1[i-1] + result    // s1 char not in LCS
            i -= 1
        else:
            result = s2[j-1] + result    // s2 char not in LCS
            j -= 1
    
    // Remaining characters
    while i > 0:
        result = s1[i-1] + result
        i -= 1
    while j > 0:
        result = s2[j-1] + result
        j -= 1
    
    return result
```

---

## 🔬 Trace: s1 = "abac", s2 = "cab"

```
Step 1: LCS dp table
      ""  c  a  b
  ""  [ 0  0  0  0 ]
  a   [ 0  0  1  1 ]
  b   [ 0  0  1  2 ]
  a   [ 0  0  1  2 ]
  c   [ 0  1  1  2 ]

LCS length = 2 ("ab")

Step 2: Backtrack from (4,3)
  i=4,j=3: s1[3]='c' ≠ s2[2]='b'
    dp[3][3]=2 vs dp[4][2]=1 → dp[3][3] > dp[4][2]
    Add s1[3]='c', i=3
    result = "c"

  i=3,j=3: s1[2]='a' ≠ s2[2]='b'
    dp[2][3]=2 vs dp[3][2]=1 → dp[2][3] > dp[3][2]
    Add s1[2]='a', i=2... wait, dp[2][3]=2, dp[3][2]=1
    
    Actually: dp[i-1][j] = dp[2][3] = 2, dp[i][j-1] = dp[3][2] = 1
    dp[2][3] > dp[3][2], so take s1[i-1]='a', i=2
    result = "ac"

  i=2,j=3: s1[1]='b' == s2[2]='b' → LCS match!
    Add 'b' once, i=1, j=2
    result = "bac"

  i=1,j=2: s1[0]='a' == s2[1]='a' → LCS match!
    Add 'a' once, i=0, j=1
    result = "abac"

  i=0, j=1: remaining s2
    Add s2[0]='c', j=0
    result = "cabac"

Answer: "cabac" (length 5 = 4+3-2)

Verify:
  s1="abac" in "cabac": c[a][b][a][c] → ✓
  s2="cab" in "cabac":  [c][a][b]ac   → ✓
```

---

## 🌐 Alternative: Direct DP for SCS Length

```
Instead of using LCS as intermediate:

dp[i][j] = length of SCS for s1[0..i-1] and s2[0..j-1]

if s1[i-1] == s2[j-1]:
    dp[i][j] = dp[i-1][j-1] + 1
else:
    dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1])

Base: dp[i][0] = i, dp[0][j] = j

This is essentially Edit Distance with only inserts!
```

---

## 💡 Relationship Map

```
┌───────────────────────────────────────────────────┐
│         String DP Relationships                   │
│                                                   │
│  LCS length = dp[m][n]                            │
│                                                   │
│  SCS length = m + n - LCS                         │
│                                                   │
│  Edit Distance (ins+del only) = m + n - 2×LCS    │
│                                                   │
│  Diff output = SCS with markings of s1-only       │
│                and s2-only characters              │
│                                                   │
│  All three build on the SAME dp table!            │
└───────────────────────────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Approach** | Build LCS table → backtrack to construct SCS |
| **Formula** | SCS_length = m + n - LCS_length |
| **Reconstruction** | LCS chars once + non-LCS chars from both strings |
| **Complexity** | O(m·n) time, O(m·n) space (need full table) |
| **Relationship** | SCS and LCS are complementary views |
| **Key Insight** | Shared chars (LCS) included once, rest included |

---

## ❓ Quick Revision Questions

1. **What is the formula relating SCS length, s1 length, s2 length, and LCS length?**
2. **Why do LCS characters appear only once in the SCS?**
3. **How do you reconstruct the SCS from the LCS dp table?**
4. **What happens during backtracking when characters match vs don't match?**
5. **Walk through SCS construction for s1="abc", s2="bcd".**
6. **How is SCS related to the unix `diff` command?**

---

[← Previous: Interleaving String](05-interleaving-string.md) | [Next Unit: Palindrome DP →](../07-Palindrome-DP/01-longest-palindromic-subsequence.md)

[← Back to README](../README.md)
