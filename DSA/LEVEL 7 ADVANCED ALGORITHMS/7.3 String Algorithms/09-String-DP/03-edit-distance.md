# Chapter 9.3 — Edit Distance (Levenshtein Distance)

> **Unit 9: String DP** | [Course Home](../README.md)

---

## 📋 Chapter Overview

**Edit distance** measures the minimum number of single-character
operations (insert, delete, replace) to transform one string into
another. It is fundamental to spell checking, DNA alignment, and
diff algorithms.

---

## 1. Problem Definition

```
  Given strings X and Y, find the minimum number of operations
  to transform X into Y.

  Operations (each costs 1):
    INSERT:   add a character       "ab" → "abc"
    DELETE:   remove a character    "abc" → "ab"
    REPLACE:  change a character    "abc" → "adc"

  Examples:
    edit("kitten", "sitting") = 3
      kitten → sitten (replace k→s)
      sitten → sittin (replace e→i)
      sittin → sitting (insert g)

    edit("", "abc") = 3  (insert a, b, c)
    edit("abc", "abc") = 0
```

---

## 2. DP Recurrence

```
  dp[i][j] = edit distance between X[1..i] and Y[1..j]

  Base cases:
    dp[0][j] = j    (insert j characters)
    dp[i][0] = i    (delete i characters)

  Recurrence:
    If X[i] == Y[j]:
        dp[i][j] = dp[i-1][j-1]        ← characters match, no cost
    Else:
        dp[i][j] = 1 + min(
            dp[i-1][j],      ← DELETE  X[i]
            dp[i][j-1],      ← INSERT  Y[j]
            dp[i-1][j-1]     ← REPLACE X[i] with Y[j]
        )

  ┌─────────────────────────────────────────┐
  │  dp[i-1][j-1]     dp[i-1][j]           │
  │  (replace/match)  (delete)              │
  │        ↘              ↓                 │
  │  dp[i][j-1]  ────→  dp[i][j]           │
  │  (insert)                               │
  └─────────────────────────────────────────┘
```

---

## 3. DP Table Example

```
  X = "HORSE"
  Y = "ROS"

         ""  R  O  S
    ""  [ 0  1  2  3 ]
     H  [ 1  1  2  3 ]
     O  [ 2  2  1  2 ]
     R  [ 3  2  2  2 ]
     S  [ 4  3  3  2 ]
     E  [ 5  4  4  3 ]

  Answer: dp[5][3] = 3

  Trace (one possible sequence):
  (5,3)→(4,3): X[5]='E'≠Y[3]='S', delete E     → dp[4][3]=2+1?
  
  Better trace:
  dp[5][3]=3 came from dp[4][2]+1 (replace E→S? No...)

  Let's trace properly:
  dp[5][3]=3: min(dp[4][3]+1=3, dp[5][2]+1=5, dp[4][2]+1=3)
    → dp[4][2]+1: replace E→S. X="HORS", matched with "RO" → dp[4][2]=2
  dp[4][2]=2: X[4]='S'≠Y[2]='O', min(dp[3][2]+1=3, dp[4][1]+1=4, dp[3][1]+1=3)
    → dp[3][1]+1: replace S→O? No, delete. Actually dp[3][2]=2 came from...

  Operations: HORSE → RORSE (replace H→R) → ROSE (delete R) → ROS (delete E)
```

---

## 4. Implementation

```python
def edit_distance(X: str, Y: str) -> int:
    m, n = len(X), len(Y)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    # Base cases
    for i in range(m + 1):
        dp[i][0] = i
    for j in range(n + 1):
        dp[0][j] = j
    
    # Fill table
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if X[i-1] == Y[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(
                    dp[i-1][j],      # delete
                    dp[i][j-1],      # insert
                    dp[i-1][j-1]     # replace
                )
    
    return dp[m][n]

# Space-optimized version (O(n) space):
def edit_distance_opt(X: str, Y: str) -> int:
    m, n = len(X), len(Y)
    prev = list(range(n + 1))
    
    for i in range(1, m + 1):
        curr = [i] + [0] * n
        for j in range(1, n + 1):
            if X[i-1] == Y[j-1]:
                curr[j] = prev[j-1]
            else:
                curr[j] = 1 + min(prev[j], curr[j-1], prev[j-1])
        prev = curr
    
    return prev[n]
```

---

## 5. Backtracking: Finding Operations

```python
def edit_operations(X: str, Y: str) -> list:
    m, n = len(X), len(Y)
    dp = [[0] * (n+1) for _ in range(m+1)]
    for i in range(m+1): dp[i][0] = i
    for j in range(n+1): dp[0][j] = j
    
    for i in range(1, m+1):
        for j in range(1, n+1):
            if X[i-1] == Y[j-1]:
                dp[i][j] = dp[i-1][j-1]
            else:
                dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])
    
    # Backtrack
    ops = []
    i, j = m, n
    while i > 0 or j > 0:
        if i > 0 and j > 0 and X[i-1] == Y[j-1]:
            i -= 1; j -= 1  # match, no operation
        elif i > 0 and j > 0 and dp[i][j] == dp[i-1][j-1] + 1:
            ops.append(f"Replace X[{i-1}]='{X[i-1]}' with '{Y[j-1]}'")
            i -= 1; j -= 1
        elif i > 0 and dp[i][j] == dp[i-1][j] + 1:
            ops.append(f"Delete X[{i-1}]='{X[i-1]}'")
            i -= 1
        else:
            ops.append(f"Insert '{Y[j-1]}'")
            j -= 1
    
    return list(reversed(ops))
```

---

## 6. Variations

```
  ┌──────────────────────────────────────────────────────────┐
  │  Variation 1: Weighted edit distance                     │
  │  Different costs for insert/delete/replace.              │
  │  Just change the +1 to the respective cost.              │
  │                                                           │
  │  Variation 2: Only insert + delete (no replace)          │
  │  edit_distance = m + n - 2 × LCS(X, Y)                  │
  │                                                           │
  │  Variation 3: Damerau-Levenshtein                        │
  │  Adds TRANSPOSITION: swap adjacent characters.           │
  │  "ab" → "ba" costs 1 instead of 2.                      │
  │                                                           │
  │  Variation 4: Edit distance with k threshold             │
  │  Only compute dp values within band of width 2k+1.       │
  │  Time: O(nk) instead of O(mn).                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 7. Applications

```
  1. Spell Checkers:   suggest words within edit distance 1-2
  2. DNA Alignment:    measure sequence similarity
  3. Fuzzy String Matching: approximate pattern search
  4. Diff Tools:       computing file differences
  5. OCR Correction:   fixing character recognition errors
  6. NLP:              string similarity for entity matching

  Relationship to other metrics:
  ┌─────────────────────────────────────────────────┐
  │  Hamming distance:  only substitutions,         │
  │                      same-length strings only    │
  │  Levenshtein:       insert + delete + replace    │
  │  Damerau-Lev:       + transposition              │
  │  Jaro-Winkler:      different formula entirely   │
  └─────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Aspect | Details |
|--------|---------|
| Operations | Insert (1), Delete (1), Replace (1) |
| Recurrence | dp[i][j] = dp[i-1][j-1] if match, else 1+min(3 neighbors) |
| Base cases | dp[i][0]=i, dp[0][j]=j |
| Time | O(m × n) |
| Space | O(m × n), optimizable to O(min(m,n)) |
| Answer | dp[m][n] |
| Relation to LCS | edit = m + n - 2×LCS (insert/delete only) |

---

## ❓ Quick Revision Questions

1. **What are the base cases and why?**
   <details><summary>Answer</summary>dp[i][0] = i (delete all i characters from X to get empty string), dp[0][j] = j (insert all j characters of Y into empty string). These represent transforming to/from the empty string.</details>

2. **What does each of the three transitions represent?**
   <details><summary>Answer</summary>dp[i-1][j]+1 = delete X[i]; dp[i][j-1]+1 = insert Y[j]; dp[i-1][j-1]+1 = replace X[i] with Y[j]. When X[i]==Y[j], dp[i-1][j-1]+0 (match, no operation).</details>

3. **How is edit distance related to LCS?**
   <details><summary>Answer</summary>When only insertions and deletions are allowed (no replacements), edit distance = m + n - 2×LCS(X,Y). With replacements allowed, the relationship is more complex.</details>

4. **What is Damerau-Levenshtein distance?**
   <details><summary>Answer</summary>It extends Levenshtein by adding transposition (swapping two adjacent characters) as a fourth operation. "ab"→"ba" costs 1 instead of 2. Useful for real-world typos.</details>

5. **How to reduce time when edit distance is known to be small?**
   <details><summary>Answer</summary>Use banded DP: only compute dp[i][j] where |i-j| ≤ k (the maximum expected edit distance). This reduces time from O(mn) to O(nk). If dp[m][n] > k, no solution within the band.</details>

6. **What is edit("sunday", "saturday")?**
   <details><summary>Answer</summary>3: sunday → saturday requires: insert 'a' and 't' after 's', replace 'n' with 'r'. Or: sunday → snday (delete u) → ... Actually: edit distance = 3.</details>

---

| [⬅️ Previous: Longest Common Substring](02-longest-common-substring.md) | [Next: Wildcard Matching ➡️](04-wildcard-matching.md) |
|:---|---:|
