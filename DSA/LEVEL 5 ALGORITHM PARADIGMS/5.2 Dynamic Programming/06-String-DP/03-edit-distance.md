# Chapter 3: Edit Distance (Levenshtein Distance)

## 📋 Overview
**Edit Distance** (LeetCode #72) finds the minimum number of operations (insert, delete, replace) to transform one string into another. It's one of the most important string DP problems with applications in spell checking, DNA alignment, and diff tools.

---

## 🧠 Core Concept

```
Operations:
  INSERT:  "abc" → "abXc"    (insert X at position)
  DELETE:  "abc" → "ac"      (delete b)
  REPLACE: "abc" → "aXc"     (replace b with X)

Example: "horse" → "ros"
  horse → rorse  (replace h with r)
  rorse → rose   (delete r at position 3)
  rose  → ros    (delete e)
  = 3 operations

State: dp[i][j] = edit distance between s1[0..i-1] and s2[0..j-1]
```

---

## 🔨 Recurrence

```
if s1[i-1] == s2[j-1]:
    dp[i][j] = dp[i-1][j-1]           // characters match, no operation

else:
    dp[i][j] = 1 + min(
        dp[i-1][j],      // DELETE from s1 (or INSERT into s2)
        dp[i][j-1],      // INSERT into s1 (or DELETE from s2)
        dp[i-1][j-1]     // REPLACE s1[i-1] with s2[j-1]
    )

Visualization of transitions:
  ┌─────────┬─────────┐
  │dp[i-1]  │dp[i-1]  │
  │  [j-1]  │  [j]    │
  │ REPLACE │ DELETE  │
  ├─────────┼─────────┤
  │dp[i]    │dp[i][j] │
  │  [j-1]  │  = ?    │
  │ INSERT  │         │
  └─────────┴─────────┘
```

---

## 🔬 Trace: s1 = "horse", s2 = "ros"

```
      ""  r  o  s
  ""  [ 0  1  2  3 ]
  h   [ 1  1  2  3 ]
  o   [ 2  2  1  2 ]
  r   [ 3  2  2  2 ]
  s   [ 4  3  3  2 ]
  e   [ 5  4  4  3 ]

Key cells:
  dp[1][1]: h≠r → 1+min(dp[0][0]=0, dp[0][1]=1, dp[1][0]=1) = 1 (replace)
  dp[2][2]: o==o → dp[1][1] = 1
  dp[3][1]: r==r → dp[2][0] = 2
  dp[5][3]: e≠s → 1+min(dp[4][2]=3, dp[5][2]=4, dp[4][3]=2) = 3

Answer: dp[5][3] = 3

Operations (backtrack):
  (5,3): e≠s, came from dp[4][3]=2 (DELETE e)
  (4,3): s==s, came from dp[3][2] (match)
  (3,2): r≠o, came from dp[2][1]=2? Actually dp[2][2]=1... 
  
  Let me trace more carefully:
  (5,3)=3: e≠s, min(dp[4][2]=3,dp[5][2]=4,dp[4][3]=2)=2→REPLACE e→s? No...
  
  Better: dp[4][3]=2 via delete. dp[4][3]: s==s, dp[3][2]=2.
  dp[3][2]: r≠o, min(dp[2][1]=2,dp[3][1]=2,dp[2][2]=1)=1→REPLACE r→o? 
  
  Actually multiple valid backtrack paths exist.
  One valid sequence: replace h→r, delete r, delete e = 3 ops ✓
```

---

## 💻 Implementation

```
function editDistance(s1, s2):
    m = len(s1), n = len(s2)
    
    // Space optimized: two rows
    prev = [0, 1, 2, ..., n]
    
    for i = 1 to m:
        curr = array of size n+1
        curr[0] = i
        for j = 1 to n:
            if s1[i-1] == s2[j-1]:
                curr[j] = prev[j-1]
            else:
                curr[j] = 1 + min(prev[j], curr[j-1], prev[j-1])
        prev = curr
    
    return prev[n]

Time: O(m·n), Space: O(n)
```

---

## 🌐 Variants

### Variant 1: Only insertions and deletions (no replace)
```
dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1])  when mismatch
         = dp[i-1][j-1]                       when match

This equals: m + n - 2 × LCS(s1, s2)
  (delete all non-LCS chars from s1, insert all non-LCS chars of s2)
```

### Variant 2: Different costs per operation
```
dp[i][j] = min(
    dp[i-1][j]   + deleteCost,
    dp[i][j-1]   + insertCost,
    dp[i-1][j-1] + (0 if match else replaceCost)
)
```

### Variant 3: One Edit Distance (LeetCode #161)
```
Are two strings exactly one edit apart?
Can be solved in O(n) without DP:

function isOneEditDistance(s, t):
    if abs(len(s) - len(t)) > 1: return false
    for i in range(min(len(s), len(t))):
        if s[i] != t[i]:
            if len(s) == len(t): return s[i+1:] == t[i+1:]  // replace
            if len(s) < len(t):  return s[i:] == t[i+1:]    // insert
            return s[i+1:] == t[i:]                          // delete
    return abs(len(s) - len(t)) == 1
```

---

## 💡 Applications

```
┌────────────────────────────┬──────────────────────────────┐
│ Application                │ How Edit Distance is used    │
├────────────────────────────┼──────────────────────────────┤
│ Spell checkers             │ Suggest words within distance│
│ DNA sequence alignment     │ Mutations = edits            │
│ Plagiarism detection       │ Similarity score             │
│ Fuzzy string matching      │ Approximate pattern search   │
│ diff / version control     │ Minimum file changes         │
│ OCR correction             │ Fix recognition errors       │
│ Natural Language Processing│ Word/sentence similarity     │
└────────────────────────────┴──────────────────────────────┘
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = min edits for s1[0..i-1] → s2[0..j-1] |
| **Match** | dp[i][j] = dp[i-1][j-1] (free) |
| **No Match** | 1 + min(delete, insert, replace) |
| **Base** | dp[i][0]=i, dp[0][j]=j |
| **Complexity** | O(m·n) time, O(n) space |
| **Also Known As** | Levenshtein distance |

---

## ❓ Quick Revision Questions

1. **What are the three operations in edit distance?**
2. **What does dp[i][j] represent?**
3. **Which cell in the dp table corresponds to each operation?**
4. **How is edit distance related to LCS when only insert/delete are allowed?**
5. **What are the base cases and why?**
6. **How do you reconstruct the sequence of operations?**

---

[← Previous: Longest Common Substring](02-longest-common-substring.md) | [Next: Distinct Subsequences →](04-distinct-subsequences.md)

[← Back to README](../README.md)
