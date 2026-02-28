# Chapter 3: Palindrome Partitioning II

## 📋 Overview
**Palindrome Partitioning II** (LeetCode #132) finds the minimum number of cuts so that every partition is a palindrome. It combines **palindrome detection DP** with a **1D cutting DP**, making it a great example of how two DP tables work together.

---

## 🧠 Core Concept

```
s = "aab"

Possible partitions:
  "a" | "a" | "b"  → 2 cuts (all palindromes ✓)
  "aa" | "b"        → 1 cut  (both palindromes ✓)
  "aab"             → 0 cuts (not palindrome ✗)

Minimum cuts: 1

s = "abcbm"
  "a"|"bcb"|"m"     → 2 cuts  ✓
  "a"|"b"|"c"|"b"|"m" → 4 cuts ✓
  Minimum: 2
```

---

## 🔨 Two-Table Approach

```
Table 1: isPalin[i][j] = true if s[i..j] is palindrome
  isPalin[i][j] = (s[i]==s[j]) AND (j-i<2 OR isPalin[i+1][j-1])

Table 2: cuts[i] = minimum cuts for s[0..i]
  cuts[i] = min over all j where isPalin[j][i]:
      if j == 0: 0                // whole prefix is palindrome
      else:      cuts[j-1] + 1    // cut before position j

Pseudocode:
function minCut(s):
    n = len(s)
    
    // Build palindrome table
    isPalin = 2D boolean n × n
    for i = n-1 down to 0:
        for j = i to n-1:
            isPalin[i][j] = (s[i]==s[j]) and (j-i<2 or isPalin[i+1][j-1])
    
    // Build cuts table
    cuts = array of size n
    for i = 0 to n-1:
        if isPalin[0][i]:
            cuts[i] = 0    // whole prefix is palindrome
        else:
            cuts[i] = i    // worst case: cut every character
            for j = 1 to i:
                if isPalin[j][i]:
                    cuts[i] = min(cuts[i], cuts[j-1] + 1)
    
    return cuts[n-1]
```

---

## 🔬 Trace: s = "aab"

```
Step 1: Palindrome table
  isPalin:
       a    a    b
  a  [ T    T    F ]
  a  [      T    F ]
  b  [           T ]

  "a"=T, "aa"=T, "aab"=F, "a"=T, "ab"=F, "b"=T

Step 2: Cuts table
  i=0: isPalin[0][0]=T → cuts[0]=0
  i=1: isPalin[0][1]=T → cuts[1]=0
  i=2: isPalin[0][2]=F
       worst case: cuts[2]=2
       j=1: isPalin[1][2]=F → skip
       j=2: isPalin[2][2]=T → cuts[2]=min(2, cuts[1]+1)=min(2,1)=1
       cuts[2]=1

Answer: 1 (partition: "aa"|"b")
```

---

## 🔬 Trace: s = "abcbm"

```
Palindrome table (key entries):
  isPalin[0][0]='a' T
  isPalin[1][1]='b' T
  isPalin[2][2]='c' T
  isPalin[3][3]='b' T
  isPalin[4][4]='m' T
  isPalin[1][3]='bcb' T  (b==b AND isPalin[2][2]='c' T)
  isPalin[0][4]='abcbm' F

Cuts table:
  i=0: isPalin[0][0]=T → cuts[0]=0
  i=1: isPalin[0][1]=F, worst=1
       j=1: isPalin[1][1]=T → cuts[1]=min(1, cuts[0]+1)=1
  i=2: isPalin[0][2]=F, worst=2
       j=1: isPalin[1][2]=F
       j=2: isPalin[2][2]=T → cuts[2]=min(2, cuts[1]+1)=2
  i=3: isPalin[0][3]=F, worst=3
       j=1: isPalin[1][3]=T → cuts[3]=min(3, cuts[0]+1)=1
       j=2: isPalin[2][3]=F
       j=3: isPalin[3][3]=T → cuts[3]=min(1, cuts[2]+1)=1
  i=4: isPalin[0][4]=F, worst=4
       j=1: isPalin[1][4]=F
       j=2: isPalin[2][4]=F
       j=3: isPalin[3][4]=F
       j=4: isPalin[4][4]=T → cuts[4]=min(4, cuts[3]+1)=2

Answer: 2 (partition: "a"|"bcb"|"m")
```

---

## 💻 Optimized: Build Palindrome Table While Computing Cuts

```
function minCut(s):
    n = len(s)
    cuts = array of size n
    isPalin = 2D boolean n × n
    
    for i = 0 to n-1:
        minCuts = i  // worst case
        for j = 0 to i:
            if s[j] == s[i] and (i-j < 2 or isPalin[j+1][i-1]):
                isPalin[j][i] = true
                if j == 0:
                    minCuts = 0
                else:
                    minCuts = min(minCuts, cuts[j-1] + 1)
        cuts[i] = minCuts
    
    return cuts[n-1]

Time: O(n²), Space: O(n²) for isPalin + O(n) for cuts
```

---

## 🌐 Expand-Around-Center Optimization

```
function minCut(s):
    n = len(s)
    cuts = [i for i in range(n)]  // worst case: i cuts for position i
    
    for center = 0 to n-1:
        // Odd-length palindromes
        i, j = center, center
        while i >= 0 and j < n and s[i] == s[j]:
            if i == 0:
                cuts[j] = 0
            else:
                cuts[j] = min(cuts[j], cuts[i-1] + 1)
            i -= 1; j += 1
        
        // Even-length palindromes
        i, j = center, center + 1
        while i >= 0 and j < n and s[i] == s[j]:
            if i == 0:
                cuts[j] = 0
            else:
                cuts[j] = min(cuts[j], cuts[i-1] + 1)
            i -= 1; j += 1
    
    return cuts[n-1]

Time: O(n²), Space: O(n) only!
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **Problem** | Min cuts so every part is palindrome |
| **Two Tables** | isPalin[i][j] + cuts[i] |
| **cuts[i]** | Min cuts for s[0..i] |
| **Recurrence** | cuts[i] = min(cuts[j-1]+1) where s[j..i] is palindrome |
| **Complexity** | O(n²) time, O(n) space (expand approach) |
| **Key Idea** | Combine palindrome check with cut optimization |

---

## ❓ Quick Revision Questions

1. **Why do we need two DP tables?**
2. **What does cuts[i] represent?**
3. **How do you check if s[j..i] is a palindrome efficiently?**
4. **What is the worst-case number of cuts for a string of length n?**
5. **How does the expand-around-center approach reduce space to O(n)?**
6. **Trace the algorithm for s = "aba".**

---

[← Previous: Longest Palindromic Substring](02-longest-palindromic-substring.md) | [Next: Count Palindromic Subsequences →](04-count-palindromic-subsequences.md)

[← Back to README](../README.md)
