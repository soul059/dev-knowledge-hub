# Chapter 4: Distinct Subsequences

## 📋 Overview
**Distinct Subsequences** (LeetCode #115) counts the number of distinct subsequences of string `s` that equal string `t`. It combines counting with string matching in a beautiful DP formulation.

---

## 🧠 Core Concept

```
s = "rabbbit", t = "rabbit"

How many ways can we choose characters from s (in order) to form t?

r a b b b i t
↕ ↕ ↕ ↕   ↕ ↕   → choose b at position 3, skip position 5
↕ ↕ ↕   ↕ ↕ ↕   → choose b at position 4, skip position 3
↕ ↕   ↕ ↕ ↕ ↕   → choose b at position 3&4&5? No...

Actually:
  r a b b i t  →  using b[2],b[3]  (skip b[4])
  r a b b i t  →  using b[2],b[4]  (skip b[3])
  r a b b i t  →  using b[3],b[4]  (skip b[2])

Answer: 3
```

---

## 🔨 Recurrence

```
dp[i][j] = number of distinct subsequences of s[0..i-1] that equal t[0..j-1]

if s[i-1] == t[j-1]:
    dp[i][j] = dp[i-1][j-1] + dp[i-1][j]
               ↑ use this char   ↑ skip this char
    "We can match this character OR skip it"

else:
    dp[i][j] = dp[i-1][j]
               ↑ must skip this char
    "Character doesn't match, skip it"

Base cases:
    dp[i][0] = 1  for all i  (empty t is a subsequence of anything)
    dp[0][j] = 0  for j > 0  (can't form non-empty t from empty s)
```

---

## 🔬 Trace: s = "rabbbit", t = "rabbit"

```
      ""  r  a  b  b  i  t
  ""  [ 1  0  0  0  0  0  0 ]
  r   [ 1  1  0  0  0  0  0 ]
  a   [ 1  1  1  0  0  0  0 ]
  b   [ 1  1  1  1  0  0  0 ]
  b   [ 1  1  1  2  1  0  0 ]
  b   [ 1  1  1  3  3  0  0 ]
  i   [ 1  1  1  3  3  3  0 ]
  t   [ 1  1  1  3  3  3  3 ]

Key cells:
  dp[3][3]: s[2]='b'==t[2]='b' → dp[2][2]+dp[2][3] = 1+0 = 1
  dp[4][3]: s[3]='b'==t[2]='b' → dp[3][2]+dp[3][3] = 1+1 = 2
  dp[5][3]: s[4]='b'==t[2]='b' → dp[4][2]+dp[4][3] = 1+2 = 3
  dp[4][4]: s[3]='b'==t[3]='b' → dp[3][3]+dp[3][4] = 1+0 = 1
  dp[5][4]: s[4]='b'==t[3]='b' → dp[4][3]+dp[4][4] = 2+1 = 3

Answer: dp[7][6] = 3 ✓
```

---

## 💻 Implementation

### Tabulation
```
function numDistinct(s, t):
    m = len(s), n = len(t)
    dp = 2D array (m+1) × (n+1)
    
    // Base: empty t matched by any prefix of s
    for i = 0 to m: dp[i][0] = 1
    // Base: non-empty t can't be matched by empty s
    for j = 1 to n: dp[0][j] = 0
    
    for i = 1 to m:
        for j = 1 to n:
            dp[i][j] = dp[i-1][j]        // skip s[i-1]
            if s[i-1] == t[j-1]:
                dp[i][j] += dp[i-1][j-1]  // use s[i-1]
    
    return dp[m][n]

Time: O(m·n), Space: O(m·n)
```

### Space Optimized — O(n) space
```
function numDistinct(s, t):
    m = len(s), n = len(t)
    dp = array of size n+1, filled with 0
    dp[0] = 1  // empty t
    
    for i = 1 to m:
        // MUST iterate right to left to avoid overwriting
        for j = n down to 1:
            if s[i-1] == t[j-1]:
                dp[j] += dp[j-1]
    
    return dp[n]

Why right-to-left?
  dp[j] needs old dp[j-1] (from previous row).
  Left-to-right would overwrite dp[j-1] before we use it.
  
  This is analogous to 0/1 knapsack space optimization!
```

---

## 🔬 Trace: Space-Optimized, s = "aab", t = "ab"

```
dp = [1, 0, 0]  (base)

i=1, s[0]='a':
  j=2: s[0]='a' ≠ t[1]='b' → skip
  j=1: s[0]='a' == t[0]='a' → dp[1] += dp[0] = 0+1 = 1
  dp = [1, 1, 0]

i=2, s[1]='a':
  j=2: s[1]='a' ≠ t[1]='b' → skip
  j=1: s[1]='a' == t[0]='a' → dp[1] += dp[0] = 1+1 = 2
  dp = [1, 2, 0]

i=3, s[2]='b':
  j=2: s[2]='b' == t[1]='b' → dp[2] += dp[1] = 0+2 = 2
  j=1: s[2]='b' ≠ t[0]='a' → skip
  dp = [1, 2, 2]

Answer: dp[2] = 2

Verification: "aab" → "ab" can use a[0]+b[2] or a[1]+b[2] = 2 ways ✓
```

---

## 💡 Intuition Builder

```
Think of it as: for each character in t,
which characters in s can match it?

s = "aab", t = "ab"

t[0]='a': can match s[0]='a' OR s[1]='a'  → 2 choices
t[1]='b': can match s[2]='b'              → 1 choice

But choices are constrained:
  If t[0] uses s[1], then t[1] can only use s[2] (index > 1) ✓
  If t[0] uses s[0], then t[1] can only use s[1] or s[2] → but only s[2] is 'b'

Total: 2 ways ✓

The DP efficiently handles these ordering constraints!
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = ways to form t[0..j-1] from s[0..i-1] |
| **Match** | dp[i][j] = dp[i-1][j-1] + dp[i-1][j] |
| **No Match** | dp[i][j] = dp[i-1][j] |
| **Base** | dp[i][0]=1, dp[0][j]=0 for j>0 |
| **Space Opt** | O(n), iterate right-to-left |
| **Complexity** | O(m·n) time |

---

## ❓ Quick Revision Questions

1. **What does dp[i][j] represent in this problem?**
2. **Why do we add dp[i-1][j-1] + dp[i-1][j] on a match?**
3. **What does "skip this character" correspond to in the dp?**
4. **Why must the space-optimized version iterate right-to-left?**
5. **How many distinct subsequences of "aaa" equal "a"?**
6. **What happens if len(t) > len(s)?**

---

[← Previous: Edit Distance](03-edit-distance.md) | [Next: Interleaving String →](05-interleaving-string.md)

[← Back to README](../README.md)
