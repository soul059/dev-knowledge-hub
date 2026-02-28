# Chapter 4: Count Palindromic Subsequences

## 📋 Overview
**Count Different Palindromic Subsequences** (LeetCode #730) counts the number of distinct non-empty palindromic subsequences in a string. This is one of the more challenging palindrome DP problems, requiring careful handling of duplicates and modular arithmetic.

---

## 🧠 Core Concept

```
s = "bccb"

All distinct palindromic subsequences:
  Length 1: "b", "c"                        → 2
  Length 2: "bb", "cc"                      → 2
  Length 3: "bcb", "bCb"... just "bcb"      → 1
  Length 4: "bccb"                          → 1
  Total: 6

Key challenge: counting WITHOUT duplicates
  "bccb" has subsequences "b" at index 0 AND index 3
  But we count "b" only once.
```

---

## 🔨 Simpler Version First: Count All (with duplicates)

```
dp[i][j] = number of palindromic subsequences in s[i..j]
(may count duplicates — simpler to understand first)

if s[i] == s[j]:
    dp[i][j] = dp[i+1][j] + dp[i][j-1] + 1
    // All palindromes in [i+1..j] (not using s[i])
    // All palindromes in [i..j-1] (not using s[j])
    // +1 for the pair (s[i], s[j]) themselves
    // Note: dp[i+1][j-1] counted in both, but adding s[i]..s[j] around them creates NEW palindromes

else:
    dp[i][j] = dp[i+1][j] + dp[i][j-1] - dp[i+1][j-1]
    // Include-exclude: add both sides, subtract overlap
```

---

## 🔨 Distinct Count: LeetCode 730

```
State: dp[i][j] = count of distinct palindromic subsequences in s[i..j]
                   (modulo 10^9 + 7)

For s[i] == s[j]:
  Find leftmost and rightmost occurrence of s[i] in s[i+1..j-1]
  
  Case 1: No s[i] found inside → dp[i][j] = dp[i+1][j-1] × 2 + 2
    All inner palindromes + wrap each with s[i]...s[j] + "s[i]" + "s[i]s[j]"
    
  Case 2: One s[i] found inside → dp[i][j] = dp[i+1][j-1] × 2 + 1
    "s[i]" already counted in inner, so only +1 for "s[i]s[j]"
    
  Case 3: Two+ s[i] found inside → 
    dp[i][j] = dp[i+1][j-1] × 2 - dp[left+1][right-1]
    Subtract duplicates from the range between inner occurrences

For s[i] ≠ s[j]:
  dp[i][j] = dp[i+1][j] + dp[i][j-1] - dp[i+1][j-1]
```

---

## 🔬 Trace: s = "bccb"

```
s = b c c b
    0 1 2 3

Base: dp[i][i] = 1 for all i (each char is a palindrome)

Length 2:
  dp[1][2]: s[1]='c'==s[2]='c'
    Inner range [2..1] is empty → dp[inner]=0
    No 'c' inside → Case 1: 0×2+2 = 2 → "c","cc"
  dp[0][1]: s[0]='b'≠s[1]='c'
    dp[1][1]+dp[0][0]-dp[1][0] = 1+1-0 = 2 → "b","c"
  dp[2][3]: s[2]='c'≠s[3]='b'
    dp[3][3]+dp[2][2]-dp[3][2] = 1+1-0 = 2 → "c","b"

Length 3:
  dp[0][2]: s[0]='b'≠s[2]='c'
    dp[1][2]+dp[0][1]-dp[1][1] = 2+2-1 = 3 → "b","c","cc"
  dp[1][3]: s[1]='c'≠s[3]='b'
    dp[2][3]+dp[1][2]-dp[2][2] = 2+2-1 = 3 → "c","b","cc"

Length 4:
  dp[0][3]: s[0]='b'==s[3]='b'
    Inner [1..2]: look for 'b' in "cc" → none found
    Case 1: dp[1][2]×2+2 = 2×2+2 = 6
    → "b","c","cc","bcb","bccb","bb"   (6 total) ✓

Answer: 6
```

---

## 💻 Implementation

```
function countPalindromicSubsequences(s):
    n = len(s)
    MOD = 10^9 + 7
    dp = 2D array n × n, filled with 0
    
    for i = 0 to n-1: dp[i][i] = 1
    
    for length = 2 to n:
        for i = 0 to n-length:
            j = i + length - 1
            
            if s[i] == s[j]:
                // Find leftmost and rightmost s[i] in [i+1..j-1]
                left = i + 1
                right = j - 1
                while left <= right and s[left] != s[i]: left++
                while left <= right and s[right] != s[i]: right--
                
                if left > right:
                    // No duplicate inside
                    dp[i][j] = dp[i+1][j-1] * 2 + 2
                else if left == right:
                    // Exactly one duplicate
                    dp[i][j] = dp[i+1][j-1] * 2 + 1
                else:
                    // Two or more duplicates
                    dp[i][j] = dp[i+1][j-1] * 2 - dp[left+1][right-1]
            else:
                dp[i][j] = dp[i+1][j] + dp[i][j-1] - dp[i+1][j-1]
            
            dp[i][j] = ((dp[i][j] % MOD) + MOD) % MOD
    
    return dp[0][n-1]
```

---

## 💡 Understanding the Three Cases

```
s[i] == s[j] = 'a', inner string = s[i+1..j-1]

Case 1: No 'a' inside       Case 2: One 'a' inside
  a [ ... no a ... ] a        a [ ... a ... ] a
  
  Wrap all inner palindromes   Wrap all inner, but "a" already
  with 'a': new palindromes!   counted, so just +1 for "aa"
  Plus "a" and "aa" new.       
  Result: inner×2 + 2          Result: inner×2 + 1

Case 3: Two+ 'a' inside
  a [ ... a ... a ... ] a
        left    right
  
  Wrapping creates duplicates with palindromes
  that start/end with 'a' in [left+1..right-1]
  Result: inner×2 - dp[left+1][right-1]
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Problem** | Count distinct palindromic subsequences |
| **State** | dp[i][j] = count in s[i..j] |
| **Match Cases** | 3 cases based on inner occurrences of s[i] |
| **No Match** | Include-exclude: dp[i+1][j] + dp[i][j-1] - dp[i+1][j-1] |
| **Complexity** | O(n²) time (with O(n) per cell for searching → O(n³)), optimizable to O(n²) |
| **Modular** | Result modulo 10⁹+7 |

---

## ❓ Quick Revision Questions

1. **Why is counting distinct palindromes harder than counting all?**
2. **What are the three cases when s[i] == s[j]?**
3. **Why do we subtract dp[left+1][right-1] in Case 3?**
4. **What does the include-exclude formula handle for s[i] ≠ s[j]?**
5. **How can searching for left/right be optimized?**
6. **Count distinct palindromic subsequences of "aaa".**

---

[← Previous: Palindrome Partitioning II](03-palindrome-partitioning.md) | [Next: Palindrome Removal →](05-palindrome-removal.md)

[← Back to README](../README.md)
