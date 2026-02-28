# Chapter 9.2 — Longest Common Substring

> **Unit 9: String DP** | [Course Home](../README.md)

---

## 📋 Chapter Overview

The **Longest Common Substring** problem finds the longest contiguous
sequence that appears in both strings. Unlike LCS (subsequence), the
characters must be **adjacent**.

---

## 1. Subsequence vs Substring

```
  X = "ABCDEF"
  Y = "ZBCDF"

  Longest Common Subsequence:  "BCDF"  (length 4, not contiguous)
  Longest Common Substring:    "BCD"   (length 3, contiguous)

  ┌───────────────────────────────────────────────┐
  │  Substring:    consecutive characters          │
  │  Subsequence:  characters in order, may skip   │
  │                                                 │
  │  Every substring IS a subsequence.             │
  │  Not every subsequence is a substring.         │
  └───────────────────────────────────────────────┘
```

---

## 2. DP Recurrence

```
  dp[i][j] = length of longest common substring ENDING at
             X[i] and Y[j].

  If X[i] == Y[j]:
      dp[i][j] = dp[i-1][j-1] + 1   ← extend the match
  Else:
      dp[i][j] = 0                    ← match broken!

  Answer: max of all dp[i][j]

  Key difference from LCS:
  ┌────────────────────────────────────────────────┐
  │  LCS: dp[i][j] = max(dp[i-1][j], dp[i][j-1]) │
  │  LCSubstring: dp[i][j] = 0 when no match      │
  │                                                 │
  │  No carrying forward — if match breaks,         │
  │  counter resets to 0!                           │
  └────────────────────────────────────────────────┘
```

---

## 3. DP Table Example

```
  X = "ABAB"
  Y = "BABA"

         ""  B  A  B  A
    ""  [ 0  0  0  0  0 ]
     A  [ 0  0  1  0  1 ]
     B  [ 0  1  0  2  0 ]
     A  [ 0  0  2  0  3 ]
     B  [ 0  1  0  3  0 ]

  Maximum value = 3 at dp[4][3]
  → Longest common substring has length 3

  Backtrack from dp[4][3]:
  dp[4][3]=3: X[4]='B', go to dp[3][2]
  dp[3][2]=2: X[3]='A', go to dp[2][1]
  dp[2][1]=1: X[2]='B', go to dp[1][0]=0 → stop

  LCS substring: "BAB" (reading backwards: B,A,B)
  Verify: "BAB" appears in both "ABAB" and "BABA" ✓
```

---

## 4. Implementation

```python
def longest_common_substring(X: str, Y: str) -> tuple:
    """Returns (length, substring)."""
    m, n = len(X), len(Y)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    max_len = 0
    end_idx = 0  # ending index in X
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if X[i-1] == Y[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
                if dp[i][j] > max_len:
                    max_len = dp[i][j]
                    end_idx = i
            else:
                dp[i][j] = 0
    
    substring = X[end_idx - max_len:end_idx]
    return max_len, substring

# Example:
length, sub = longest_common_substring("ABAB", "BABA")
print(f"Length: {length}, Substring: {sub}")  # Length: 3, Substring: BAB
```

---

## 5. Space Optimization

```python
def lc_substring_optimized(X: str, Y: str) -> int:
    """O(min(m,n)) space — length only."""
    if len(X) < len(Y):
        X, Y = Y, X
    m, n = len(X), len(Y)
    
    prev = [0] * (n + 1)
    max_len = 0
    
    for i in range(1, m + 1):
        curr = [0] * (n + 1)
        for j in range(1, n + 1):
            if X[i-1] == Y[j-1]:
                curr[j] = prev[j-1] + 1
                max_len = max(max_len, curr[j])
            # else: curr[j] remains 0
        prev = curr
    
    return max_len
```

```
  Space saved:
  ┌───────────┬────────────┐
  │ Full DP   │ O(m × n)   │
  │ Two-row   │ O(n)       │
  │ One-row*  │ O(n)       │
  └───────────┴────────────┘
  * One-row is trickier because dp[i][j] depends on
    dp[i-1][j-1] (diagonal), need to save previous value.
```

---

## 6. Alternative: Suffix-Based Approach

```
  Approach 1: Binary Search + Hashing O(n log n)
  ──────────────────────────────────────────────
  Binary search on length L.
  For each L, hash all substrings of length L in X,
  then check if any substring of length L in Y matches.

  Approach 2: Suffix Array O(n log n)
  ────────────────────────────────────
  Concatenate X + "$" + Y, build suffix array + LCP array.
  Find max LCP where adjacent suffixes come from different strings.

  Approach 3: Generalized Suffix Tree O(n + m)
  ─────────────────────────────────────────────
  Build suffix tree for both strings.
  Find deepest internal node with leaves from both strings.

  ┌──────────────────────────────────────────────┐
  │  DP:           O(mn)   simple, practical     │
  │  Hash + BS:    O(n lg n) for equal-length     │
  │  Suffix Array: O(n+m)  optimal for large n   │
  └──────────────────────────────────────────────┘
```

---

## 7. Variations

```
  1. K Common Substrings: Find k longest common substrings
     → Collect all dp values > 0, sort descending

  2. Common Substring of 3+ Strings:
     → Generalized suffix tree, or pairwise DP

  3. Longest Repeated Substring (single string):
     → Same DP with X = Y, but exclude i == j case
     → Better: suffix array with max LCP

  4. Print ALL Common Substrings of length ≥ k:
     → Collect from DP where dp[i][j] ≥ k
```

---

## 📝 Summary Table

| Aspect | LCS (Subsequence) | LC Substring |
|--------|-------------------|--------------|
| Contiguity | Not required | Required |
| Recurrence (match) | dp[i-1][j-1] + 1 | dp[i-1][j-1] + 1 |
| Recurrence (no match) | max(dp[i-1][j], dp[i][j-1]) | 0 |
| Answer location | dp[m][n] | max of all dp[i][j] |
| Time | O(mn) | O(mn) |
| Space | O(mn) or O(n) | O(mn) or O(n) |
| Backtracking | From dp[m][n] upward | From max cell diagonally |

---

## ❓ Quick Revision Questions

1. **What resets dp[i][j] to 0 in longest common substring?**
   <details><summary>Answer</summary>When X[i] ≠ Y[j], dp[i][j] = 0 because the substring must be contiguous. Any mismatch breaks the chain, unlike LCS where we can skip characters.</details>

2. **Where is the answer in the DP table?**
   <details><summary>Answer</summary>The answer is the maximum value anywhere in the table (not just dp[m][n]). We need to scan the entire table because the common substring can end at any position.</details>

3. **How to recover the actual substring?**
   <details><summary>Answer</summary>Track the position (i,j) of the maximum dp value. The substring is X[i-maxLen+1 .. i] (or equivalently Y[j-maxLen+1 .. j]). Follow the diagonal back maxLen steps.</details>

4. **What is the time complexity of the suffix array approach?**
   <details><summary>Answer</summary>O((n+m) log(n+m)) with standard suffix array construction, or O(n+m) with SA-IS algorithm. This is better than DP's O(mn) when strings are long.</details>

5. **How does this differ from finding repeated substrings in one string?**
   <details><summary>Answer</summary>Longest repeated substring uses the same idea with X=Y but must ensure the two occurrences don't overlap at the same position. A suffix array approach with LCP is more natural for this.</details>

---

| [⬅️ Previous: Longest Common Subsequence](01-longest-common-subsequence.md) | [Next: Edit Distance ➡️](03-edit-distance.md) |
|:---|---:|
