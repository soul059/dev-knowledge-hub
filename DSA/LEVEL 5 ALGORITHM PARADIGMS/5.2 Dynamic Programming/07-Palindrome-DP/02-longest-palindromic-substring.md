# Chapter 2: Longest Palindromic Substring

## 📋 Overview
**Longest Palindromic Substring** (LeetCode #5) finds the longest contiguous palindrome within a string. Unlike subsequence, the characters must be adjacent. The DP approach uses an interval table, but **expand-around-center** is often the preferred O(n²) method.

---

## 🧠 Core Concept

```
s = "babad"

Palindromic substrings: "b","a","b","a","d","bab","aba"
Longest: "bab" or "aba" (length 3)

Substring vs Subsequence:
  Substring: must be contiguous → "bab" ✓
  Subsequence: can skip chars → "bbbab"→"bbb" is subsequence ✓ but not substring
```

---

## 🔨 Approach 1: DP — O(n²) time, O(n²) space

```
State: dp[i][j] = true if s[i..j] is a palindrome

Recurrence:
  dp[i][j] = (s[i] == s[j]) AND dp[i+1][j-1]
  "Outer chars match AND inner string is palindrome"

Base cases:
  dp[i][i] = true           // single char
  dp[i][i+1] = (s[i]==s[i+1])  // two chars

Fill order: by increasing length

function longestPalindrome(s):
    n = len(s)
    dp = 2D boolean array n × n
    start = 0, maxLen = 1
    
    // All single chars
    for i = 0 to n-1: dp[i][i] = true
    
    // Check pairs
    for i = 0 to n-2:
        if s[i] == s[i+1]:
            dp[i][i+1] = true
            start = i; maxLen = 2
    
    // Length 3 and above
    for len = 3 to n:
        for i = 0 to n-len:
            j = i + len - 1
            if s[i] == s[j] and dp[i+1][j-1]:
                dp[i][j] = true
                start = i; maxLen = len
    
    return s[start .. start+maxLen-1]
```

---

## 🔨 Approach 2: Expand Around Center — O(n²) time, O(1) space

```
For each possible center, expand outward while characters match.

Centers:  n single chars + (n-1) pairs = 2n-1 centers total

function longestPalindrome(s):
    start = 0, maxLen = 1
    
    for center = 0 to n-1:
        // Odd length: center at i
        len1 = expand(s, center, center)
        // Even length: center between i and i+1  
        len2 = expand(s, center, center+1)
        
        len = max(len1, len2)
        if len > maxLen:
            maxLen = len
            start = center - (len-1)/2
    
    return s[start .. start+maxLen-1]

function expand(s, left, right):
    while left >= 0 and right < len(s) and s[left] == s[right]:
        left -= 1
        right += 1
    return right - left - 1
```

---

## 🔬 Trace: Expand Around Center for s = "babad"

```
Center 0 (b):
  Odd:  expand(0,0): b → can't expand → len=1
  Even: expand(0,1): b≠a → len=0

Center 1 (a):
  Odd:  expand(1,1): a → expand(0,2): b==b → expand(-1,3): stop → len=3 ("bab")
  Even: expand(1,2): a≠b → len=0

Center 2 (b):
  Odd:  expand(2,2): b → expand(1,3): a==a → expand(0,4): b≠d → len=3 ("aba")
  Even: expand(2,3): b≠a → len=0

Center 3 (a):
  Odd:  expand(3,3): a → expand(2,4): b≠d → len=1
  Even: expand(3,4): a≠d → len=0

Center 4 (d):
  Odd:  expand(4,4): d → len=1
  Even: out of bounds → len=0

Best: length 3, starting at index 0 → "bab"
```

---

## 🔬 DP Table Trace: s = "babad"

```
     b    a    b    a    d
b  [ T    F    T    F    F ]
a  [      T    F    T    F ]
b  [           T    F    F ]
a  [                T    F ]
d  [                     T ]

Reading: dp[0][2]=T → "bab" is palindrome ✓
         dp[1][3]=T → "aba" is palindrome ✓
         dp[0][4]=F → "babad" is not palindrome ✓
```

---

## 🌐 Manacher's Algorithm — O(n)

```
Achieves O(n) time using clever reuse of previous palindrome information.

Key idea: if we're inside a known palindrome, we can mirror
information from the left side to the right side.

Steps:
1. Insert '#' between characters: "abc" → "#a#b#c#"
2. For each center, use mirror of already-computed palindromes
3. Only expand beyond what we already know

P[i] = radius of palindrome centered at i

      # a # b # a # b # a # d #
P:    0 1 0 3 0 5 0 3 0 1 0 1 0

The center with P[i]=5 is 'a' at position 5.
Palindrome: "ababa" (radius 5 in transformed = length 5 in original)

Complexity: O(n) time, O(n) space
```

---

## 💡 Comparison of Approaches

```
┌─────────────────────┬──────────┬────────┬──────────────────┐
│ Approach            │ Time     │ Space  │ Difficulty       │
├─────────────────────┼──────────┼────────┼──────────────────┤
│ Brute Force         │ O(n³)    │ O(1)   │ Simple           │
│ DP Table            │ O(n²)    │ O(n²)  │ Moderate         │
│ Expand Around Center│ O(n²)    │ O(1)   │ Moderate         │
│ Manacher's          │ O(n)     │ O(n)   │ Hard             │
└─────────────────────┴──────────┴────────┴──────────────────┘

For interviews, Expand Around Center is the sweet spot:
  O(n²) time, O(1) space, easy to implement
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State (DP)** | dp[i][j] = true if s[i..j] is palindrome |
| **Recurrence** | dp[i][j] = s[i]==s[j] AND dp[i+1][j-1] |
| **Best Simple** | Expand around center: O(n²), O(1) |
| **Optimal** | Manacher's: O(n), O(n) |
| **Centers** | 2n-1 positions (n chars + n-1 gaps) |
| **vs Subsequence** | Substring = contiguous, resets on gap |

---

## ❓ Quick Revision Questions

1. **How does palindromic substring differ from subsequence?**
2. **How many possible centers exist for palindromes in a string of length n?**
3. **What is the expand-around-center approach?**
4. **What is the DP recurrence for palindromic substring?**
5. **What is Manacher's algorithm and when would you use it?**
6. **Find the longest palindromic substring of "cbbd".**

---

[← Previous: Longest Palindromic Subsequence](01-longest-palindromic-subsequence.md) | [Next: Palindrome Partitioning II →](03-palindrome-partitioning.md)

[← Back to README](../README.md)
