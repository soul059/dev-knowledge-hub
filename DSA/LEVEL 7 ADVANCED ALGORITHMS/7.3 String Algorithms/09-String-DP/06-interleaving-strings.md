# Chapter 9.6 — Interleaving Strings

> **Unit 9: String DP** | [Course Home](../README.md)

---

## 📋 Chapter Overview

**Interleaving strings** asks whether string S3 can be formed by
interleaving two strings S1 and S2 while maintaining the relative
order of characters from each (LeetCode 97).

---

## 1. Problem Definition

```
  Given S1, S2, and S3:
  Can S3 be formed by interleaving S1 and S2?

  S1 = "aab"
  S2 = "axy"
  S3 = "aaxaby"

  YES: a a x a b y
       S1 S1 S2 S1 S1 S2   Wait — let's trace properly.
       
  S1 = "aab" → a(0) a(1) b(2)
  S2 = "axy" → a(0) x(1) y(2)

  S3 = "aaxaby"
       a  → from S1[0]
       a  → from S2[0]
       x  → from S2[1]
       a  → from S1[1]
       b  → from S1[2]
       y  → from S2[2]

  Relative order preserved:
    S1 chars: a(pos 0), a(pos 3), b(pos 4) → order maintained ✓
    S2 chars: a(pos 1), x(pos 2), y(pos 5) → order maintained ✓

  ┌────────────────────────────────────────────────────┐
  │  Interleaving ≠ merging arbitrarily.               │
  │  Characters from S1 must appear in S1's order.     │
  │  Characters from S2 must appear in S2's order.     │
  │  len(S3) must equal len(S1) + len(S2).             │
  └────────────────────────────────────────────────────┘
```

---

## 2. DP Recurrence

```
  dp[i][j] = True if S3[1..i+j] can be formed by interleaving
             S1[1..i] and S2[1..j]

  Base cases:
    dp[0][0] = True
    dp[i][0] = dp[i-1][0] AND S1[i] == S3[i]
    dp[0][j] = dp[0][j-1] AND S2[j] == S3[j]

  Recurrence:
    dp[i][j] = (dp[i-1][j] AND S1[i] == S3[i+j])    ← take from S1
            OR (dp[i][j-1] AND S2[j] == S3[i+j])    ← take from S2

  Intuition:
  ┌────────────────────────────────────────────────────┐
  │  At position i+j in S3, the character came from    │
  │  either S1[i] or S2[j].                            │
  │                                                     │
  │  If from S1: check S1[i]==S3[i+j] and dp[i-1][j]  │
  │  If from S2: check S2[j]==S3[i+j] and dp[i][j-1]  │
  └────────────────────────────────────────────────────┘
```

---

## 3. DP Table Example

```
  S1 = "aab"   (m=3)
  S2 = "axy"   (n=3)
  S3 = "aaxaby" (len=6 = 3+3 ✓)

  S3 indices:  a  a  x  a  b  y
               1  2  3  4  5  6

         ""    a    x    y
    ""  [ T    F    F    F ]
     a  [ T    F    F    F ]
     a  [ T    T    F    F ]
     b  [ F    T    F    F ]

  Wait, let me recompute:

         j=0   j=1(a) j=2(x) j=3(y)
  i=0  [  T     F      F      F   ]
  i=1  [  T     ?      ?      ?   ]
    a   dp[1][0]: dp[0][0]=T AND S1[1]='a'==S3[1]='a' → T
        dp[1][1]: dp[0][1]=F AND.. OR dp[1][0]=T AND S2[1]='a'==S3[2]='a' → T
        dp[1][2]: dp[0][2]=F OR dp[1][1]=T AND S2[2]='x'==S3[3]='x' → T
        dp[1][3]: dp[0][3]=F OR dp[1][2]=T AND S2[3]='y'==S3[4]='a'? NO → F
  i=2  a:
        dp[2][0]: dp[1][0]=T AND S1[2]='a'==S3[2]='a' → T
        dp[2][1]: dp[1][1]=T AND S1[2]='a'==S3[3]='x'? NO
                  OR dp[2][0]=T AND S2[1]='a'==S3[3]='x'? NO → F
        dp[2][2]: dp[1][2]=T AND S1[2]='a'==S3[4]='a' → T!
                  OR dp[2][1]=F → T
        dp[2][3]: dp[1][3]=F OR dp[2][2]=T AND S2[3]='y'==S3[5]='b'? NO → F
  i=3  b:
        dp[3][0]: dp[2][0]=T AND S1[3]='b'==S3[3]='x'? NO → F
        dp[3][1]: dp[2][1]=F OR dp[3][0]=F → F
        dp[3][2]: dp[2][2]=T AND S1[3]='b'==S3[5]='b' → T!
        dp[3][3]: dp[2][3]=F OR dp[3][2]=T AND S2[3]='y'==S3[6]='y' → T!

  dp[3][3] = T → YES, S3 is an interleaving of S1 and S2 ✓
```

---

## 4. Implementation

```python
def isInterleave(s1: str, s2: str, s3: str) -> bool:
    m, n = len(s1), len(s2)
    if m + n != len(s3):
        return False
    
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    dp[0][0] = True
    
    # Base cases
    for i in range(1, m + 1):
        dp[i][0] = dp[i-1][0] and s1[i-1] == s3[i-1]
    for j in range(1, n + 1):
        dp[0][j] = dp[0][j-1] and s2[j-1] == s3[j-1]
    
    # Fill table
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            dp[i][j] = (dp[i-1][j] and s1[i-1] == s3[i+j-1]) or \
                       (dp[i][j-1] and s2[j-1] == s3[i+j-1])
    
    return dp[m][n]
```

---

## 5. Space-Optimized Version

```python
def isInterleave_opt(s1: str, s2: str, s3: str) -> bool:
    m, n = len(s1), len(s2)
    if m + n != len(s3):
        return False
    
    # Use shorter string as columns for O(min(m,n)) space
    if m < n:
        s1, s2, m, n = s2, s1, n, m
    
    dp = [False] * (n + 1)
    dp[0] = True
    
    for j in range(1, n + 1):
        dp[j] = dp[j-1] and s2[j-1] == s3[j-1]
    
    for i in range(1, m + 1):
        dp[0] = dp[0] and s1[i-1] == s3[i-1]
        for j in range(1, n + 1):
            dp[j] = (dp[j] and s1[i-1] == s3[i+j-1]) or \
                    (dp[j-1] and s2[j-1] == s3[i+j-1])
    
    return dp[n]
```

---

## 6. BFS Alternative

```
  The problem can also be modeled as a GRAPH SEARCH.

  State: (i, j) = consumed i chars from S1, j from S2.
  Edge (i,j) → (i+1,j) if S1[i+1] == S3[i+j+1]
  Edge (i,j) → (i,j+1) if S2[j+1] == S3[i+j+1]

  Goal: reach state (m, n).

  BFS/DFS from (0,0):
  ┌──────────────────────────────────────┐
  │  BFS avoids revisiting states with   │
  │  a visited set. Same O(mn) time/space│
  │  as DP, but sometimes clearer to     │
  │  think about.                        │
  └──────────────────────────────────────┘
```

---

## 7. Complexity & Visualization

```
  ┌──────────────────┬────────┬─────────┐
  │ Approach         │ Time   │ Space   │
  ├──────────────────┼────────┼─────────┤
  │ DP (full table)  │ O(mn)  │ O(mn)   │
  │ DP (1D array)    │ O(mn)  │ O(n)    │
  │ BFS              │ O(mn)  │ O(mn)   │
  │ DFS + memo       │ O(mn)  │ O(mn)   │
  └──────────────────┴────────┴─────────┘

  Grid visualization:

       S2 →  a  x  y
  S1 ↓    ┌──┬──┬──┬──┐
          │  │  │  │  │  Take from S2 →
     a    ├──┼──┼──┼──┤
          │  │  │  │  │
     a    ├──┼──┼──┼──┤  Take from S1 ↓
          │  │  │  │  │
     b    ├──┼──┼──┼──┤
          │  │  │  │  │
          └──┴──┴──┴──┘
  Start: top-left (0,0)
  Goal:  bottom-right (m,n)
  Path = sequence of → and ↓ moves
```

---

## 📝 Summary Table

| Aspect | Details |
|--------|---------|
| Precondition | len(S3) = len(S1) + len(S2) |
| State | dp[i][j] = can interleave S1[1..i], S2[1..j] to form S3[1..i+j] |
| Transition | From top (S1) or from left (S2) if char matches S3 |
| Time | O(m × n) |
| Space | O(m × n), optimizable to O(min(m,n)) |
| Alternative | BFS/DFS on (i,j) state graph |

---

## ❓ Quick Revision Questions

1. **What is the quick check before running DP?**
   <details><summary>Answer</summary>Check if len(S3) == len(S1) + len(S2). If not, interleaving is impossible regardless of character content.</details>

2. **What does dp[i][j] represent?**
   <details><summary>Answer</summary>dp[i][j] is True if S3[1..i+j] can be formed by interleaving S1[1..i] and S2[1..j] while maintaining the relative character order from both strings.</details>

3. **Why is there an OR in the transition?**
   <details><summary>Answer</summary>The current character s3[i+j] could come from either S1 (if s1[i]==s3[i+j]) or S2 (if s2[j]==s3[i+j]). Either source is valid, hence OR.</details>

4. **How is this different from merging two sorted arrays?**
   <details><summary>Answer</summary>Merging sorted arrays has a greedy choice (always pick smaller element). Interleaving doesn't — the choice of which string to take from depends on future characters, requiring DP to explore both options.</details>

5. **Can BFS solve this? How?**
   <details><summary>Answer</summary>Yes. Model as a graph with states (i,j). Start at (0,0), goal at (m,n). Edges go right (take from S2) or down (take from S1) when characters match S3. BFS finds if (m,n) is reachable. Use visited set to avoid revisiting states.</details>

---

| [⬅️ Previous: Regular Expression Matching](05-regular-expression-matching.md) | [Next: Suffix Array ➡️](../10-Advanced-String-Concepts/01-suffix-array.md) |
|:---|---:|
