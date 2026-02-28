# Chapter 9.5 — Regular Expression Matching

> **Unit 9: String DP** | [Course Home](../README.md)

---

## 📋 Chapter Overview

**Regular expression matching** with `'.'` (any single character) and
`'*'` (zero or more of the preceding element) is a harder variant of
pattern matching (LeetCode 10). The `'*'` here modifies the previous
character, unlike wildcard matching.

---

## 1. Problem Definition

```
  Pattern characters:
    '.'  →  matches any SINGLE character
    '*'  →  zero or more of the PRECEDING element
    else →  matches itself

  Examples:
    isMatch("aa", "a")     → false
    isMatch("aa", "a*")    → true   (a* = "aa")
    isMatch("ab", ".*")    → true   (.* = "ab")
    isMatch("aab", "c*a*b") → true  (c*="", a*="aa", b="b")
    isMatch("mississippi", "mis*is*p*.") → false

  Key difference from wildcard:
  ┌──────────────────────────────────────────────────────┐
  │  Wildcard '*':  standalone, matches any sequence     │
  │  Regex '*':     modifies PREVIOUS char,              │
  │                 matches 0+ of that char              │
  │                                                       │
  │  Regex "a*" = {"", "a", "aa", "aaa", ...}           │
  │  Regex ".*" = {"", "a", "ab", "abc", ...}  anything! │
  └──────────────────────────────────────────────────────┘
```

---

## 2. DP Recurrence

```
  dp[i][j] = True if S[1..i] matches P[1..j]

  Base cases:
    dp[0][0] = True
    dp[i][0] = False for i > 0
    dp[0][j]: True if P[j] == '*' and dp[0][j-2] is True
              (x* matches empty string, removing both x and *)

  Recurrence:
    Case 1: P[j] == '*'
        // '*' with preceding element P[j-1]:
        dp[i][j] = dp[i][j-2]     ← zero occurrences (skip x*)
                 OR (match(S[i], P[j-1]) AND dp[i-1][j])
                    ← one+ occurrences: S[i] matches P[j-1],
                       and rest of S[1..i-1] matches with same pattern

    Case 2: P[j] == '.' or P[j] == S[i]
        dp[i][j] = dp[i-1][j-1]   ← single char match

    Case 3: mismatch
        dp[i][j] = False

  where match(c, p) = (p == '.' or p == c)
```

---

## 3. DP Table Example

```
  S = "aab"    P = "c*a*b"

         ""   c    *    a    *    b
    ""  [ T   F    T    F    T    F ]
     a  [ F   F    F    T    T    F ]
     a  [ F   F    F    F    T    F ]
     b  [ F   F    F    F    F    T ]

  dp[3][5] = T → match!

  Key cells explained:
  dp[0][2]: P[2]='*', dp[0][0]=T → "c*" matches "" → T
  dp[0][4]: P[4]='*', dp[0][2]=T → "c*a*" matches "" → T
  dp[1][3]: P[3]='a'==S[1]='a', dp[0][2]=T → T
  dp[1][4]: P[4]='*', dp[1][2]=F OR (match('a','a') AND dp[0][4]=T) → T
  dp[2][4]: P[4]='*', dp[2][2]=F OR (match('a','a') AND dp[1][4]=T) → T
  dp[3][5]: P[5]='b'==S[3]='b', dp[2][4]=T → T ✓
```

---

## 4. Implementation

```python
def isMatch(s: str, p: str) -> bool:
    m, n = len(s), len(p)
    dp = [[False] * (n + 1) for _ in range(m + 1)]
    dp[0][0] = True
    
    # Base case: empty string
    for j in range(1, n + 1):
        if p[j-1] == '*':
            dp[0][j] = dp[0][j-2]  # x* matches empty
    
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if p[j-1] == '*':
                # Zero occurrences of p[j-2]
                dp[i][j] = dp[i][j-2]
                # One or more occurrences
                if p[j-2] == '.' or p[j-2] == s[i-1]:
                    dp[i][j] = dp[i][j] or dp[i-1][j]
            elif p[j-1] == '.' or p[j-1] == s[i-1]:
                dp[i][j] = dp[i-1][j-1]
    
    return dp[m][n]
```

---

## 5. The '*' Transitions Explained

```
  Pattern "a*" matching against string "aaa":

  Zero occurrences: dp[i][j-2]
  ┌──────────────────────────────────────────┐
  │  Skip 'a' and '*' entirely.              │
  │  "aaa" must match the pattern BEFORE a*  │
  └──────────────────────────────────────────┘

  One occurrence: match S[i] with P[j-1], then dp[i-1][j]
  ┌──────────────────────────────────────────────────────┐
  │  S[i] must match 'a' (or '.').                      │
  │  Then S[1..i-1] must match the SAME pattern          │
  │  (including "a*"), allowing more 'a's to match.      │
  │                                                       │
  │  This recursion naturally handles 2, 3, ... matches: │
  │  dp[3][j]: 'a' matches, check dp[2][j]              │
  │  dp[2][j]: 'a' matches, check dp[1][j]              │
  │  dp[1][j]: 'a' matches, check dp[0][j]              │
  │  dp[0][j]: use zero-occurrence → dp[0][j-2]         │
  └──────────────────────────────────────────────────────┘
```

---

## 6. Comparison: Wildcard vs Regex Matching

```
  ┌──────────────────┬──────────────────┬────────────────────┐
  │ Aspect           │ Wildcard (LC 44) │ Regex (LC 10)      │
  ├──────────────────┼──────────────────┼────────────────────┤
  │ '?' / '.'        │ any single char  │ any single char    │
  │ '*' meaning      │ any sequence     │ 0+ of previous     │
  │ '*' transitions  │ dp[i-1][j] or    │ dp[i][j-2] or      │
  │                  │ dp[i][j-1]       │ dp[i-1][j]         │
  │ '*' standalone?  │ YES              │ NO (needs prev)    │
  │ Base case dp[0]  │ leading *'s      │ even-pos *'s       │
  │ Complexity       │ O(mn)            │ O(mn)              │
  └──────────────────┴──────────────────┴────────────────────┘
```

---

## 7. Common Pitfalls

```
  ┌────────────────────────────────────────────────────────┐
  │  1. '*' can match ZERO characters                     │
  │     "c*" matches "" (zero c's)                       │
  │                                                        │
  │  2. ".*" matches ANYTHING (any sequence)              │
  │     '.' matches any char, '*' repeats it              │
  │                                                        │
  │  3. Pattern "a*a" CAN match "a" or "aaa" or "aaaa"   │
  │     a* = "", then a = "a"  → matches "a"             │
  │     a* = "aa", then a = "a" → matches "aaa"          │
  │                                                        │
  │  4. dp[i][j-2] for zero matches (not j-1!)           │
  │     We skip BOTH the preceding char and '*'           │
  │                                                        │
  │  5. Never assume '*' appears after every char         │
  │     Pattern "abc" has no '*' at all                   │
  └────────────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Pattern | Meaning | DP Transition |
|---------|---------|---------------|
| `.` | Any single char | dp[i-1][j-1] |
| `x*` | Zero of x | dp[i][j-2] |
| `x*` | One+ of x | dp[i-1][j] if match(s[i],x) |
| `c` | Exact char | dp[i-1][j-1] if s[i]==c |

---

## ❓ Quick Revision Questions

1. **How does regex '*' differ from wildcard '*'?**
   <details><summary>Answer</summary>Regex '*' modifies the preceding character: "a*" means zero or more 'a's. Wildcard '*' is standalone and matches any sequence of any characters. They require different DP formulations.</details>

2. **Why do we use dp[i][j-2] for zero occurrences?**
   <details><summary>Answer</summary>We skip both the character before '*' and '*' itself (2 pattern positions). dp[i][j-1] would only skip '*' but keep the character, which doesn't make sense since 'x*' is a unit.</details>

3. **What does '.*' match?**
   <details><summary>Answer</summary>'.' matches any single character, and '*' means zero or more of it. So '.*' matches any sequence of any characters (including empty). It's the most permissive pattern.</details>

4. **How does the DP handle "a*a*a" matching "aaa"?**
   <details><summary>Answer</summary>The first "a*" can match 0-3 a's, the second "a*" matches some/remaining a's, and the final "a" must match exactly one 'a'. dp naturally explores all valid splits through its transitions.</details>

5. **What is the base case for dp[0][j]?**
   <details><summary>Answer</summary>dp[0][j] = True only if P[1..j] can match the empty string. This happens when pattern consists of x* pairs: dp[0][j] = dp[0][j-2] if P[j]=='*'. Otherwise dp[0][j] = False.</details>

---

| [⬅️ Previous: Wildcard Matching](04-wildcard-matching.md) | [Next: Interleaving Strings ➡️](06-interleaving-strings.md) |
|:---|---:|
