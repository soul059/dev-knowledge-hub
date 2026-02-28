# Chapter 1: Longest Palindromic Subsequence

## 📋 Overview
**Longest Palindromic Subsequence** (LeetCode #516) finds the longest subsequence of a string that reads the same forwards and backwards. The elegant insight: LPS(s) = LCS(s, reverse(s)), connecting palindrome problems to the well-understood LCS framework.

---

## 🧠 Core Concept

```
s = "bbbab"

Subsequences that are palindromes:
  "b"     length 1
  "bb"    length 2
  "bbb"   length 3
  "bbbb"  length 4 ← longest!

"bbbb" from "bbbab":  b b b _ b → indices 0,1,2,4

Key Insight: LPS(s) = LCS(s, reverse(s))
  s     = "bbbab"
  rev   = "babbb"
  LCS   = "bbbb" (length 4) ✓
```

---

## 🔨 Direct DP Approach (Interval DP)

```
State: dp[i][j] = length of LPS in s[i..j]

Recurrence:
  if s[i] == s[j]:
      dp[i][j] = dp[i+1][j-1] + 2    // both ends match
  else:
      dp[i][j] = max(dp[i+1][j], dp[i][j-1])  // try removing each end

Base cases:
  dp[i][i] = 1        // single character is palindrome
  dp[i][i-1] = 0      // empty range (for even-length cases)

Fill order: need dp[i+1][j-1] before dp[i][j]
  → Fill by increasing length (j - i)
  → Or fill i from bottom to top
```

---

## 🔬 Trace: s = "bbbab"

```
s = b b b a b
    0 1 2 3 4

Fill dp table (i = row, j = column):

Length 1 (base):
  dp[0][0]=1, dp[1][1]=1, dp[2][2]=1, dp[3][3]=1, dp[4][4]=1

Length 2:
  dp[0][1]: b==b → dp[1][0]+2 = 0+2 = 2
  dp[1][2]: b==b → dp[2][1]+2 = 0+2 = 2
  dp[2][3]: b≠a → max(dp[3][3], dp[2][2]) = max(1,1) = 1
  dp[3][4]: a≠b → max(dp[4][4], dp[3][3]) = max(1,1) = 1

Length 3:
  dp[0][2]: b==b → dp[1][1]+2 = 1+2 = 3
  dp[1][3]: b≠a → max(dp[2][3], dp[1][2]) = max(1,2) = 2
  dp[2][4]: b==b → dp[3][3]+2 = 1+2 = 3

Length 4:
  dp[0][3]: b≠a → max(dp[1][3], dp[0][2]) = max(2,3) = 3
  dp[1][4]: b==b → dp[2][3]+2 = 1+2 = 3

Length 5:
  dp[0][4]: b==b → dp[1][3]+2 = 2+2 = 4

Full dp table:
     j=0  j=1  j=2  j=3  j=4
i=0 [ 1    2    3    3    4  ]
i=1 [      1    2    2    3  ]
i=2 [           1    1    3  ]
i=3 [                1    1  ]
i=4 [                     1  ]

Answer: dp[0][4] = 4 ("bbbb")
```

---

## 💻 Implementation

### Interval DP — O(n²) time, O(n²) space
```
function longestPalindromeSubseq(s):
    n = len(s)
    dp = 2D array n × n, filled with 0
    
    for i = 0 to n-1:
        dp[i][i] = 1
    
    for length = 2 to n:
        for i = 0 to n-length:
            j = i + length - 1
            if s[i] == s[j]:
                dp[i][j] = dp[i+1][j-1] + 2
            else:
                dp[i][j] = max(dp[i+1][j], dp[i][j-1])
    
    return dp[0][n-1]
```

### Via LCS — O(n²) time, O(n) space
```
function longestPalindromeSubseq(s):
    return LCS(s, reverse(s))

function LCS(s1, s2):
    n = len(s1)
    prev = array of size n+1, filled with 0
    for i = 1 to n:
        curr = array of size n+1, filled with 0
        for j = 1 to n:
            if s1[i-1] == s2[j-1]:
                curr[j] = prev[j-1] + 1
            else:
                curr[j] = max(prev[j], curr[j-1])
        prev = curr
    return prev[n]
```

### Space-Optimized Direct DP — O(n) space
```
function longestPalindromeSubseq(s):
    n = len(s)
    dp = array of size n, filled with 1  // dp[j] for current i
    
    for i = n-2 down to 0:
        prev = 0  // dp[i+1][i] = 0 (empty range)
        for j = i+1 to n-1:
            temp = dp[j]
            if s[i] == s[j]:
                dp[j] = prev + 2
            else:
                dp[j] = max(dp[j], dp[j-1])
            prev = temp
    
    return dp[n-1]
```

---

## 🌐 Related: Minimum Insertions for Palindrome

```
Minimum insertions to make s a palindrome:
  = len(s) - LPS(s)

Why: the LPS is already palindromic.
     Add mirror of remaining characters.

Example: s = "abcaa"
  LPS = "aca" (length 3)
  Min insertions = 5 - 3 = 2
  Result: "aacbcaa" (insert 'c' and 'a')
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = LPS length in s[i..j] |
| **Match** | dp[i][j] = dp[i+1][j-1] + 2 |
| **No Match** | dp[i][j] = max(dp[i+1][j], dp[i][j-1]) |
| **LCS Insight** | LPS(s) = LCS(s, reverse(s)) |
| **Complexity** | O(n²) time, O(n) space |
| **Application** | Min insertions for palindrome = n - LPS |

---

## ❓ Quick Revision Questions

1. **How does LPS relate to LCS?**
2. **What is the interval DP recurrence for LPS?**
3. **What is the fill order for the dp table?**
4. **How do you compute minimum insertions to make a palindrome?**
5. **What is the space-optimized approach?**
6. **Walk through LPS for s = "abcba".**

---

[← Previous Unit: Shortest Common Supersequence](../06-String-DP/06-shortest-common-supersequence.md) | [Next: Longest Palindromic Substring →](02-longest-palindromic-substring.md)

[← Back to README](../README.md)
