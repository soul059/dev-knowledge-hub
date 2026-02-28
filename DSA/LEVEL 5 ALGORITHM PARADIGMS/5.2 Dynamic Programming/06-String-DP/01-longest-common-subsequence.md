# Chapter 1: Longest Common Subsequence (LCS)

## 📋 Overview
**Longest Common Subsequence** (LeetCode #1143) is the cornerstone of string DP. Given two strings, find the length of the longest subsequence common to both. A **subsequence** maintains relative order but doesn't need to be contiguous.

---

## 🧠 Core Concept

```
s1 = "ABCBDAB"
s2 = "BDCAB"

Subsequence: characters in order, not necessarily contiguous

LCS = "BCAB" (length 4)
       ↕  ↕  (other valid LCS: "BDAB")

Matching visualization:
  A B C B D A B
  │   │   │ │ │
  B D C A B
  │   │ │ │
  B   C A B  → "BCAB" ✓

Key: order preserved, gaps allowed
```

---

## 🔨 Step-by-Step Development

### Step 1: Define State
```
dp[i][j] = length of LCS of s1[0..i-1] and s2[0..j-1]
```

### Step 2: Recurrence
```
if s1[i-1] == s2[j-1]:
    dp[i][j] = dp[i-1][j-1] + 1      // characters match
else:
    dp[i][j] = max(dp[i-1][j], dp[i][j-1])  // skip one char

Intuition:
  Match: both chars contribute → extend previous LCS
  No match: try skipping each char, take the better result
```

### Step 3: Base Cases
```
dp[0][j] = 0 for all j  (empty s1)
dp[i][0] = 0 for all i  (empty s2)
```

---

## 🔬 Trace: s1 = "ABCDE", s2 = "ACE"

```
     ""  A  C  E
  "" [ 0  0  0  0 ]
  A  [ 0  1  1  1 ]
  B  [ 0  1  1  1 ]
  C  [ 0  1  2  2 ]
  D  [ 0  1  2  2 ]
  E  [ 0  1  2  3 ]

Step-by-step key cells:
  dp[1][1]: A==A → dp[0][0]+1 = 1
  dp[2][1]: B≠A → max(dp[1][1], dp[2][0]) = max(1,0) = 1
  dp[3][2]: C==C → dp[2][1]+1 = 2
  dp[5][3]: E==E → dp[4][2]+1 = 3

Answer: dp[5][3] = 3 (LCS = "ACE")
```

---

## 🔄 Reconstructing the LCS

```
function reconstructLCS(dp, s1, s2):
    i = len(s1), j = len(s2)
    lcs = ""
    
    while i > 0 and j > 0:
        if s1[i-1] == s2[j-1]:
            lcs = s1[i-1] + lcs    // this char is in LCS
            i -= 1; j -= 1
        else if dp[i-1][j] > dp[i][j-1]:
            i -= 1                  // came from above
        else:
            j -= 1                  // came from left
    
    return lcs

Trace for "ABCDE" / "ACE":
  (5,3): E==E → lcs="E", go to (4,2)
  (4,2): D≠C → dp[3][2]=2 vs dp[4][1]=1, go up (3,2)
  (3,2): C==C → lcs="CE", go to (2,1)
  (2,1): B≠A → dp[1][1]=1 vs dp[2][0]=0, go up (1,1)
  (1,1): A==A → lcs="ACE", go to (0,0)
  Done! LCS = "ACE" ✓
```

---

## 💻 Space-Optimized Version

```
function lcs(s1, s2):
    m = len(s1), n = len(s2)
    prev = array of size n+1, filled with 0
    
    for i = 1 to m:
        curr = array of size n+1, filled with 0
        for j = 1 to n:
            if s1[i-1] == s2[j-1]:
                curr[j] = prev[j-1] + 1
            else:
                curr[j] = max(prev[j], curr[j-1])
        prev = curr
    
    return prev[n]

Time: O(m·n), Space: O(n)

Note: Space optimization loses ability to reconstruct.
      For reconstruction, keep full dp table or use Hirschberg's algorithm.
```

---

## 🌐 Related Problems

```
┌─────────────────────────────────┬──────────────────────────────┐
│ Problem                         │ Relation to LCS              │
├─────────────────────────────────┼──────────────────────────────┤
│ Longest Common Substring        │ Only dp[i-1][j-1]+1 on match│
│ Edit Distance                   │ LCS variant with costs       │
│ Shortest Common Supersequence   │ m + n - LCS(s1, s2)         │
│ Diff utility (unix diff)       │ Based on LCS algorithm       │
│ DNA sequence alignment          │ LCS with scoring matrix      │
│ Longest Palindromic Subsequence │ LCS(s, reverse(s))           │
└─────────────────────────────────┴──────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = LCS length for s1[0..i-1], s2[0..j-1] |
| **Recurrence** | Match: dp[i-1][j-1]+1; No match: max(dp[i-1][j], dp[i][j-1]) |
| **Base** | dp[0][j] = dp[i][0] = 0 |
| **Complexity** | O(m·n) time, O(n) space |
| **Reconstruction** | Backtrack from dp[m][n], follow match/max |
| **Applications** | Diff, DNA alignment, version control |

---

## ❓ Quick Revision Questions

1. **What is the difference between a subsequence and a substring?**
2. **What is the recurrence for LCS when characters match vs don't match?**
3. **How do you reconstruct the actual LCS string?**
4. **How is Longest Palindromic Subsequence related to LCS?**
5. **What is the space-optimized approach for LCS?**
6. **How does LCS relate to Shortest Common Supersequence?**

---

[← Previous Unit: Cherry Pickup](../05-Grid-DP/06-cherry-pickup.md) | [Next: Longest Common Substring →](02-longest-common-substring.md)

[← Back to README](../README.md)
