# Chapter 9.4 — Wildcard Matching

> **Unit 9: String DP** | [Course Home](../README.md)

---

## 📋 Chapter Overview

**Wildcard matching** determines if a pattern with `'?'` (any single
character) and `'*'` (any sequence of characters, including empty)
matches an entire string. This is a classic DP problem (LeetCode 44).

---

## 1. Problem Definition

```
  Pattern characters:
    '?'  →  matches exactly ONE character (any)
    '*'  →  matches ANY SEQUENCE (zero or more characters)
    else →  matches itself exactly

  Examples:
    isMatch("aa", "a")      → false
    isMatch("aa", "*")      → true
    isMatch("cb", "?a")     → false ('?' matches 'c', but 'a'≠'b')
    isMatch("adceb", "*a*b") → true  (*="" , a=a, *="dce", b=b)
    isMatch("acdcb", "a*c?b") → false
```

---

## 2. DP Recurrence

```
  dp[i][j] = True if S[1..i] matches P[1..j]

  Base cases:
    dp[0][0] = True        (empty matches empty)
    dp[i][0] = False       (non-empty string, empty pattern)
    dp[0][j] = True only if P[1..j] is all '*'s

  Recurrence:
    If P[j] == '*':
        dp[i][j] = dp[i-1][j]     ← '*' matches one more char
                 OR dp[i][j-1]     ← '*' matches empty
    
    Else if P[j] == '?' or P[j] == S[i]:
        dp[i][j] = dp[i-1][j-1]   ← single char match
    
    Else:
        dp[i][j] = False

  ┌──────────────────────────────────────────────────┐
  │  '*' is special: dp[i][j-1] means '*' matches   │
  │  nothing; dp[i-1][j] means '*' matches S[i]     │
  │  and potentially more characters.                │
  └──────────────────────────────────────────────────┘
```

---

## 3. DP Table Example

```
  S = "adceb"   P = "*a*b"

         ""   *    a    *    b
    ""  [ T   T    F    F    F ]
     a  [ F   T    T    T    F ]
     d  [ F   T    F    T    F ]
     c  [ F   T    F    T    F ]
     e  [ F   T    F    T    F ]
     b  [ F   T    F    T    T ]

  dp[5][4] = T → match!

  Trace:
  dp[0][1]: P[1]='*', dp[0][0]=T → '*' matches empty → T
  dp[1][1]: P[1]='*', dp[0][1]=T (match empty) OR dp[1][0]=F → T
  dp[1][2]: P[2]='a'==S[1]='a', dp[0][1]=T → T
  dp[1][3]: P[3]='*', dp[0][3]=F OR dp[1][2]=T → T
  ...
  dp[5][4]: P[4]='b'==S[5]='b', dp[4][3]=T → T ✓
```

---

## 4. Implementation

```python
def isMatch(s: str, p: str) -> bool:
    m, n = len(s), len(p)
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    dp[0][0] = True
    
    # Base case: empty string with leading '*'s
    for j in range(1, n + 1):
        if p[j-1] == '*':
            dp[0][j] = dp[0][j-1]
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if p[j-1] == '*':
                dp[i][j] = dp[i-1][j] or dp[i][j-1]
            elif p[j-1] == '?' or p[j-1] == s[i-1]:
                dp[i][j] = dp[i-1][j-1]
            # else: dp[i][j] remains False
    
    return dp[m][n]
```

---

## 5. Space-Optimized Version

```python
def isMatch_opt(s: str, p: str) -> bool:
    m, n = len(s), len(p)
    prev = [False] * (n + 1)
    prev[0] = True
    for j in range(1, n + 1):
        if p[j-1] == '*':
            prev[j] = prev[j-1]
    
    for i in range(1, m + 1):
        curr = [False] * (n + 1)
        for j in range(1, n + 1):
            if p[j-1] == '*':
                curr[j] = curr[j-1] or prev[j]
            elif p[j-1] == '?' or p[j-1] == s[i-1]:
                curr[j] = prev[j-1]
        prev = curr
    
    return prev[n]
```

---

## 6. Greedy Approach (O(mn) worst, faster in practice)

```python
def isMatch_greedy(s: str, p: str) -> bool:
    """Two-pointer with backtracking for '*'."""
    si, pi = 0, 0
    star_pi, star_si = -1, -1
    
    while si < len(s):
        if pi < len(p) and (p[pi] == '?' or p[pi] == s[si]):
            si += 1
            pi += 1
        elif pi < len(p) and p[pi] == '*':
            star_pi = pi   # remember star position
            star_si = si   # remember string position
            pi += 1        # try matching '*' with empty
        elif star_pi != -1:
            # Backtrack: let '*' match one more character
            star_si += 1
            si = star_si
            pi = star_pi + 1
        else:
            return False
    
    # Check remaining pattern characters (must all be '*')
    while pi < len(p) and p[pi] == '*':
        pi += 1
    
    return pi == len(p)
```

```
  Greedy trace for S="adceb", P="*a*b":

  pi=0 ('*'): star_pi=0, star_si=0, pi=1
  pi=1 ('a'): s[0]='a' → match! si=1, pi=2
  pi=2 ('*'): star_pi=2, star_si=1, pi=3
  pi=3 ('b'): s[1]='d' ≠ 'b' → backtrack!
    star_si=2, si=2, pi=3
  pi=3 ('b'): s[2]='c' ≠ 'b' → backtrack!
    star_si=3, si=3, pi=3
  pi=3 ('b'): s[3]='e' ≠ 'b' → backtrack!
    star_si=4, si=4, pi=3
  pi=3 ('b'): s[4]='b' → match! si=5, pi=4
  si=5 = len(s), pi=4 = len(p) → TRUE ✓
```

---

## 7. Complexity

```
  ┌────────────────────┬────────┬─────────┐
  │ Approach           │ Time   │ Space   │
  ├────────────────────┼────────┼─────────┤
  │ DP (full table)    │ O(mn)  │ O(mn)   │
  │ DP (space opt)     │ O(mn)  │ O(n)    │
  │ Greedy (2 pointer) │ O(mn)  │ O(1)    │
  │ Recursive + memo   │ O(mn)  │ O(mn)   │
  └────────────────────┴────────┴─────────┘

  Greedy is often fastest in practice because it avoids
  filling the entire table — most inputs don't hit worst case.
```

---

## 📝 Summary Table

| Pattern | Meaning | DP Transition |
|---------|---------|---------------|
| `?` | Any single char | dp[i][j] = dp[i-1][j-1] |
| `*` | Any sequence (0+) | dp[i][j] = dp[i-1][j] OR dp[i][j-1] |
| `c` | Exact match | dp[i][j] = dp[i-1][j-1] if s[i]==c |
| _(mismatch)_ | No match | dp[i][j] = False |

---

## ❓ Quick Revision Questions

1. **How does '*' match in the DP?**
   <details><summary>Answer</summary>dp[i][j] = dp[i][j-1] (star matches empty) OR dp[i-1][j] (star matches s[i] and potentially more). The second case effectively allows '*' to consume any number of characters.</details>

2. **Why is dp[0][j] not simply False for all j?**
   <details><summary>Answer</summary>Leading '*' characters in the pattern can match the empty string. So dp[0][j] = True if P[1..j] consists entirely of '*' characters. For example, pattern "***" matches "".</details>

3. **What makes the greedy approach work?**
   <details><summary>Answer</summary>It remembers the last '*' position and backtracks to it when a mismatch occurs, trying to let the '*' match one more character. This avoids exploring all possibilities explicitly.</details>

4. **Difference between '?' in wildcard vs '.' in regex?**
   <details><summary>Answer</summary>'?' matches exactly one character (like regex '.'). In regex, '?' means "zero or one of the preceding element." Wildcard '*' means "any sequence," while regex '*' means "zero or more of the preceding element."</details>

5. **What is the time complexity of the greedy approach?**
   <details><summary>Answer</summary>O(mn) worst case (e.g., s="aaaa...b", p="*b"), but typically much faster because backtracking is limited. Average case is closer to O(m + n).</details>

---

| [⬅️ Previous: Edit Distance](03-edit-distance.md) | [Next: Regular Expression Matching ➡️](05-regular-expression-matching.md) |
|:---|---:|
