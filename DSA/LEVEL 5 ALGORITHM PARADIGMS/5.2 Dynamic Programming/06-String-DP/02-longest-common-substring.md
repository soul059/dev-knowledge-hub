# Chapter 2: Longest Common Substring

## 📋 Overview
**Longest Common Substring** finds the longest contiguous sequence of characters that appears in both strings. Unlike subsequence, characters must be **adjacent**. This subtle difference completely changes the recurrence.

---

## 🧠 Subsequence vs Substring

```
s1 = "ABCXYZ"
s2 = "XYZABC"

Longest Common Subsequence: "ABC" or "XYZ" (length 3)
Longest Common Substring:   "ABC" or "XYZ" (length 3)

s1 = "ABCXDEF"
s2 = "XABYCD"

Longest Common Subsequence: "ABCD" (length 4, non-contiguous)
Longest Common Substring:   "AB" (length 2, must be contiguous)

Key Difference:
  Subsequence: A_C matches A__C        ✓ gaps allowed
  Substring:   ABC must match ABC      ✗ no gaps
```

---

## 🔨 Recurrence Comparison

```
LCS (Subsequence):
  if s1[i-1] == s2[j-1]:
      dp[i][j] = dp[i-1][j-1] + 1
  else:
      dp[i][j] = max(dp[i-1][j], dp[i][j-1])  ← carry forward

Longest Common Substring:
  if s1[i-1] == s2[j-1]:
      dp[i][j] = dp[i-1][j-1] + 1              ← extend match
  else:
      dp[i][j] = 0                              ← RESET to 0!

The "else" clause is the critical difference:
  Subsequence: max(skip one)  → matches persist
  Substring:   0              → matches break
```

---

## 🔬 Trace: s1 = "ABABC", s2 = "BABCA"

```
     ""  B  A  B  C  A
  "" [ 0  0  0  0  0  0 ]
  A  [ 0  0  1  0  0  1 ]
  B  [ 0  1  0  2  0  0 ]
  A  [ 0  0  2  0  0  1 ]
  B  [ 0  1  0  3  0  0 ]
  C  [ 0  0  0  0  4  0 ]

Track maximum: maxLen = 4, ending at dp[5][4]

Key cells:
  dp[2][1]: B==B → dp[1][0]+1 = 1
  dp[3][2]: A==A → dp[2][1]+1 = 2  (substring "BA")
  dp[4][3]: B==B → dp[3][2]+1 = 3  (substring "BAB")
  dp[5][4]: C==C → dp[4][3]+1 = 4  (substring "BABC")

Answer: 4 (substring "BABC" matches at positions s1[1..4], s2[0..3])

To find the actual substring:
  maxLen = 4, ending at i=5 in s1
  Substring = s1[5-4..5-1] = s1[1..4] = "BABC"
```

---

## 💻 Implementation

```
function longestCommonSubstring(s1, s2):
    m = len(s1), n = len(s2)
    maxLen = 0
    endIdx = 0  // ending index in s1
    
    prev = array of size n+1, filled with 0
    
    for i = 1 to m:
        curr = array of size n+1, filled with 0
        for j = 1 to n:
            if s1[i-1] == s2[j-1]:
                curr[j] = prev[j-1] + 1
                if curr[j] > maxLen:
                    maxLen = curr[j]
                    endIdx = i
            // else: curr[j] remains 0
        prev = curr
    
    substring = s1[endIdx-maxLen .. endIdx-1]
    return (maxLen, substring)

Time: O(m·n), Space: O(n)
```

---

## 🌐 Visualization of the Difference

```
dp table for LCS (Subsequence):     dp table for Substring:
Values persist and accumulate        Values reset to 0 on mismatch

Subsequence for "AB","BA":          Substring for "AB","BA":
     "" B  A                             "" B  A
  "" [ 0  0  0 ]                      "" [ 0  0  0 ]
  A  [ 0  0  1 ]                      A  [ 0  0  1 ]
  B  [ 0  1  1 ] ← carries 1         B  [ 0  1  0 ] ← resets!

Subsequence answer: 1               Substring answer: 1
(Both are 1 here, but for longer strings they diverge significantly)
```

---

## 💡 Binary Search + Rolling Hash Alternative

```
For very long strings, use binary search on length + rolling hash:

function longestCommonSubstring(s1, s2):
    lo, hi = 0, min(len(s1), len(s2))
    ans = 0
    
    while lo <= hi:
        mid = (lo + hi) / 2
        if hasCommonSubstringOfLength(s1, s2, mid):
            ans = mid
            lo = mid + 1
        else:
            hi = mid - 1
    
    return ans

function hasCommonSubstringOfLength(s1, s2, L):
    hashes = set()
    for i = 0 to len(s1)-L:
        hashes.add(rollingHash(s1[i..i+L-1]))
    for j = 0 to len(s2)-L:
        if rollingHash(s2[j..j+L-1]) in hashes:
            return true
    return false

Time: O((m+n) log(min(m,n))) average
Space: O(m)
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = length of common substring ending at s1[i-1], s2[j-1] |
| **Match** | dp[i][j] = dp[i-1][j-1] + 1 |
| **No Match** | dp[i][j] = 0 (reset!) |
| **Answer** | max(dp[i][j]) over all i, j |
| **Complexity** | O(m·n) time, O(n) space |
| **vs Subsequence** | Resets on mismatch instead of carrying forward |

---

## ❓ Quick Revision Questions

1. **How does the recurrence differ between substring and subsequence?**
2. **Why does the answer require tracking a global maximum?**
3. **How do you extract the actual common substring from the dp table?**
4. **What is the key difference in the "else" clause?**
5. **Walk through the dp table for s1="ABC", s2="BCA".**
6. **What is the rolling hash approach and when would you use it?**

---

[← Previous: Longest Common Subsequence](01-longest-common-subsequence.md) | [Next: Edit Distance →](03-edit-distance.md)

[← Back to README](../README.md)
