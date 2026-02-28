# Chapter 5: Interleaving String

## 📋 Overview
**Interleaving String** (LeetCode #97) determines whether string `s3` is formed by interleaving `s1` and `s2`. Characters from `s1` and `s2` must maintain their relative order in `s3`. This is a 2D DP problem where we simultaneously consume characters from both input strings.

---

## 🧠 Core Concept

```
s1 = "aabcc"
s2 = "dbbca"
s3 = "aadbbcbcac"

Is s3 a valid interleaving of s1 and s2?

  s3: a  a  d  b  b  c  b  c  a  c
      ↕  ↕        ↕     ↕     ↕
  s1: a  a     b     c     c     c  → Wait, s1 = "aabcc"
      
Let me color it:
  s3: a  a  d  b  b  c  b  c  a  c
      s1 s1 s2 s2 s2 s2 s1 s1 s1 s1  ← who contributes each char?
      a  a  d  b  b  c  b  c  ?  ?
      
Actually: "aa" from s1, "dbb" from s2, "c" from s1, "b" from s2, 
          "c" from s1, "a" from s2, "c" from s1

  s1: a,a,_,_,_,_,b,c,_,c  → "aabcc" ✓ (order preserved)
  s2: _,_,d,b,b,c,_,_,a,_  → "dbbca" ✓ (order preserved)

Answer: true
```

---

## 🔨 Recurrence

```
State: dp[i][j] = can s1[0..i-1] and s2[0..j-1] interleave to form s3[0..i+j-1]?

Key insight: s3[i+j-1] must come from either s1[i-1] or s2[j-1]

dp[i][j] = (dp[i-1][j] AND s1[i-1] == s3[i+j-1])   // s3's last char from s1
         OR (dp[i][j-1] AND s2[j-1] == s3[i+j-1])   // s3's last char from s2

Base: dp[0][0] = true (both empty → empty s3)
      dp[i][0] = dp[i-1][0] AND s1[i-1] == s3[i-1]  (only using s1)
      dp[0][j] = dp[0][j-1] AND s2[j-1] == s3[j-1]  (only using s2)

Prerequisite: len(s1) + len(s2) must equal len(s3)
```

---

## 🔬 Trace: s1 = "ab", s2 = "cd", s3 = "acbd"

```
     ""   c    d
  "" [ T   F   F ]
  a  [ T   T   F ]    ← dp[1][0]: s1[0]='a'==s3[0]='a' ✓
  b  [ F   T   T ]    ← dp[2][2]: check below

Step-by-step:
  dp[0][0] = true
  dp[0][1]: s2[0]='c' vs s3[0]='a' → false
  dp[0][2]: false (dp[0][1] is false)
  
  dp[1][0]: dp[0][0]=T AND s1[0]='a'==s3[0]='a' → true
  dp[1][1]: dp[0][1]=F AND match? → false
         OR dp[1][0]=T AND s2[0]='c'==s3[1]='c' → TRUE
  dp[1][2]: dp[0][2]=F AND match? → false
         OR dp[1][1]=T AND s2[1]='d'==s3[2]='b' → false → false... 
         
  Hmm, let me recheck s3="acbd":
  dp[1][1]: s3[1]='c', dp[1][0]=T AND s2[0]='c'=='c' → TRUE ✓
  dp[1][2]: s3[2]='b', dp[1][1]=T AND s2[1]='d'=='b' → false
            dp[0][2]=F → false → FALSE
  dp[2][1]: s3[2]='b', dp[1][1]=T AND s1[1]='b'=='b' → TRUE ✓
            OR dp[2][0]=F → TRUE
  dp[2][2]: s3[3]='d', dp[1][2]=F → skip
            dp[2][1]=T AND s2[1]='d'=='d' → TRUE ✓

Answer: TRUE (s3 = "acbd" is interleaving of "ab" and "cd")
Interleaving: a(s1) c(s2) b(s1) d(s2)
```

---

## 💻 Implementation

```
function isInterleave(s1, s2, s3):
    m = len(s1), n = len(s2)
    if m + n != len(s3): return false
    
    // Space optimized: 1D array
    dp = array of size n+1
    
    for i = 0 to m:
        for j = 0 to n:
            if i == 0 and j == 0:
                dp[j] = true
            else:
                fromS1 = (i > 0 and dp[j] and s1[i-1] == s3[i+j-1])
                fromS2 = (j > 0 and dp[j-1] and s2[j-1] == s3[i+j-1])
                dp[j] = fromS1 or fromS2
    
    return dp[n]

Time: O(m·n), Space: O(n)
```

---

## 🌐 Visualization: Decision Path

```
s1 = "ab", s2 = "cd", s3 = "acbd"

Grid of decisions:
  (0,0) → 'a' matches s1[0]
       ↓
  (1,0) → 'c' matches s2[0]
       →
  (1,1) → 'b' matches s1[1]
       ↓
  (2,1) → 'd' matches s2[1]
       →
  (2,2) → DONE ✓

  Think of it as navigating a grid:
  - Going DOWN = consume next char from s1
  - Going RIGHT = consume next char from s2
  - At each step, s3[i+j-1] tells you which move to make
  - Multiple valid paths may exist (true interleaving)
```

---

## 💡 Common Pitfalls

```
1. Forgetting length check: len(s1)+len(s2) must equal len(s3)

2. Greedy doesn't work:
   s1="aa", s2="ab", s3="aaba"
   Greedy might pick s1's 'a', then s2's 'a', then stuck!
   Correct: s2's 'a', s1's 'a', s2's 'b', s1's 'a' → "aaba" ✓

3. The boolean DP is elegant:
   dp[i][j] is true/false (not counting or optimizing)
   This is a FEASIBILITY problem, not optimization.
```

---

## 📊 Summary Table

| Concept | Description |
|---------|-------------|
| **State** | dp[i][j] = can s1[0..i-1], s2[0..j-1] form s3[0..i+j-1] |
| **Recurrence** | (dp[i-1][j] AND s1 match) OR (dp[i][j-1] AND s2 match) |
| **Prerequisite** | len(s1) + len(s2) == len(s3) |
| **Type** | Boolean DP (feasibility) |
| **Complexity** | O(m·n) time, O(n) space |
| **Insight** | Navigate grid: down=use s1, right=use s2 |

---

## ❓ Quick Revision Questions

1. **What is the prerequisite check before running DP?**
2. **What does "interleaving" mean precisely?**
3. **Why doesn't a greedy approach work?**
4. **What does going "down" vs "right" represent in the dp grid?**
5. **Is this an optimization problem or a feasibility problem?**
6. **How do you space-optimize from O(m·n) to O(n)?**

---

[← Previous: Distinct Subsequences](04-distinct-subsequences.md) | [Next: Shortest Common Supersequence →](06-shortest-common-supersequence.md)

[← Back to README](../README.md)
