# Chapter 9.1 — Longest Common Subsequence (LCS)

> **Unit 9: String DP** | [Course Home](../README.md)

---

## 📋 Chapter Overview

The **Longest Common Subsequence (LCS)** problem finds the longest
sequence of characters that appears in both strings in the same order
(not necessarily contiguous). It is a foundational string DP problem.

---

## 1. Problem Definition

```
  Given two strings X and Y, find the longest subsequence
  common to both.

  Example:
    X = "ABCBDAB"
    Y = "BDCAB"

    Common subsequences: "B", "AB", "BD", "BCA", "BCAB", ...
    LCS = "BCAB" (length 4)

  Note: Subsequence ≠ Substring
  ┌────────────────────────────────────────┐
  │  Subsequence: characters in order but  │
  │  not necessarily contiguous.           │
  │  "ACE" is subseq of "ABCDE"           │
  │  "AE" is subseq of "ABCDE"            │
  │  "EA" is NOT subseq of "ABCDE"        │
  └────────────────────────────────────────┘
```

---

## 2. Recursive Formulation

```
  Let X = x₁x₂...xₘ and Y = y₁y₂...yₙ

  LCS(i, j) = length of LCS of X[1..i] and Y[1..j]

  Base case:
    LCS(0, j) = 0   (empty X)
    LCS(i, 0) = 0   (empty Y)

  Recurrence:
    If X[i] == Y[j]:
        LCS(i, j) = 1 + LCS(i-1, j-1)     ← match! include it
    Else:
        LCS(i, j) = max(LCS(i-1, j),       ← skip X[i]
                        LCS(i, j-1))        ← skip Y[j]

  Answer: LCS(m, n)
```

---

## 3. DP Table Construction

```
  X = "ABCBDAB"  (m=7)
  Y = "BDCAB"    (n=5)

       ""  B  D  C  A  B
  ""  [ 0  0  0  0  0  0 ]
   A  [ 0  0  0  0  1  1 ]
   B  [ 0  1  1  1  1  2 ]
   C  [ 0  1  1  2  2  2 ]
   B  [ 0  1  1  2  2  3 ]
   D  [ 0  1  2  2  2  3 ]
   A  [ 0  1  2  2  3  3 ]
   B  [ 0  1  2  2  3  4 ]

  Answer: dp[7][5] = 4

  Fill direction: left-to-right, top-to-bottom.
  Each cell depends on: top (i-1,j), left (i,j-1), diagonal (i-1,j-1).

  ┌─────────────┐
  │ (i-1,j-1)  (i-1,j)  │
  │     ↘        ↓      │
  │ (i,j-1) → (i,j)    │
  └─────────────┘
```

---

## 4. Backtracking to Find the LCS

```
  Start at dp[7][5] = 4, trace back:

  (7,5): X[7]='B' == Y[5]='B' → include 'B', go to (6,4)
  (6,4): X[6]='A' == Y[4]='A' → include 'A', go to (5,3)
  (5,3): X[5]='D' ≠ Y[3]='C'
         dp[4,3]=2 vs dp[5,2]=2 → go to (4,3) (or (5,2))
  (4,3): X[4]='B' ≠ Y[3]='C'
         dp[3,3]=2 vs dp[4,2]=1 → go to (3,3)
  (3,3): X[3]='C' == Y[3]='C' → include 'C', go to (2,2)
  (2,2): X[2]='B' ≠ Y[2]='D'
         dp[1,2]=0 vs dp[2,1]=1 → go to (2,1)
  (2,1): X[2]='B' == Y[1]='B' → include 'B', go to (1,0)
  (1,0): base case → stop

  LCS (reversed): B, C, A, B → "BCAB" ✓
```

---

## 5. Implementation

```python
def lcs(X: str, Y: str) -> tuple:
    """Returns (length, LCS string)."""
    m, n = len(X), len(Y)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    
    # Fill DP table
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if X[i-1] == Y[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])
    
    # Backtrack to find the actual LCS
    result = []
    i, j = m, n
    while i > 0 and j > 0:
        if X[i-1] == Y[j-1]:
            result.append(X[i-1])
            i -= 1
            j -= 1
        elif dp[i-1][j] >= dp[i][j-1]:
            i -= 1
        else:
            j -= 1
    
    return dp[m][n], ''.join(reversed(result))

# Example:
length, seq = lcs("ABCBDAB", "BDCAB")
print(f"Length: {length}, LCS: {seq}")  # Length: 4, LCS: BCAB
```

---

## 6. Space Optimization

```
  Standard: O(m × n) space for full table.

  Optimization 1: Two rows — O(min(m,n)) space
  ──────────────────────────────────────────────
  Each row only depends on the previous row.

  def lcs_length(X, Y):
      if len(X) < len(Y):
          X, Y = Y, X     # Y is shorter
      m, n = len(X), len(Y)
      prev = [0] * (n + 1)
      curr = [0] * (n + 1)
      for i in range(1, m + 1):
          for j in range(1, n + 1):
              if X[i-1] == Y[j-1]:
                  curr[j] = prev[j-1] + 1
              else:
                  curr[j] = max(prev[j], curr[j-1])
          prev, curr = curr, [0] * (n + 1)
      return prev[n]

  Note: Two-row optimization gives LENGTH only.
  For the actual LCS with O(n) space: use Hirschberg's algorithm.
```

---

## 7. Complexity

```
  ┌──────────────┬────────────────────────────────┐
  │ Aspect       │ Value                          │
  ├──────────────┼────────────────────────────────┤
  │ Time         │ O(m × n)                       │
  │ Space (full) │ O(m × n)                       │
  │ Space (opt.) │ O(min(m, n))  length only      │
  │ Backtracking │ O(m + n)                       │
  └──────────────┴────────────────────────────────┘

  For very long strings:
  - Hunt-Szymanski: O((r + n) log n) where r = #matches
  - Bit-parallel: O(mn/w) where w = word size (64)
```

---

## 8. Applications

```
  ┌──────────────────────────────────────────────────────┐
  │  1. diff tools:  Unix diff, git diff use LCS        │
  │  2. DNA/protein sequence alignment                  │
  │  3. Version control: finding changes between files  │
  │  4. Spell checking: similarity measure              │
  │  5. Plagiarism detection                            │
  │  6. Merge conflict resolution                       │
  │                                                      │
  │  Related problems:                                   │
  │  - Shortest Common Supersequence = m + n - LCS      │
  │  - Edit Distance is related to LCS                  │
  │  - Number of distinct LCS (harder, O(mn) DP)       │
  └──────────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Variant | Time | Space | Notes |
|---------|------|-------|-------|
| Full DP table | O(mn) | O(mn) | Can backtrack for LCS |
| Two-row | O(mn) | O(n) | Length only |
| Hirschberg | O(mn) | O(n) | Divide & conquer, full LCS |
| Recursive + memo | O(mn) | O(mn) | Top-down alternative |

---

## ❓ Quick Revision Questions

1. **What is the recurrence for LCS?**
   <details><summary>Answer</summary>If X[i]==Y[j]: dp[i][j] = dp[i-1][j-1] + 1. Otherwise: dp[i][j] = max(dp[i-1][j], dp[i][j-1]). Base case: dp[0][j] = dp[i][0] = 0.</details>

2. **How do you recover the actual LCS from the DP table?**
   <details><summary>Answer</summary>Start at dp[m][n]. If X[i]==Y[j], include the character and go diagonally to (i-1,j-1). Otherwise, go to the larger of dp[i-1][j] or dp[i][j-1]. Continue until reaching row 0 or column 0.</details>

3. **What is the relationship between LCS and edit distance?**
   <details><summary>Answer</summary>If only insertions and deletions are allowed (no replacements), edit distance = m + n - 2×LCS(X,Y). LCS and edit distance are dual problems.</details>

4. **How can space be reduced to O(n)?**
   <details><summary>Answer</summary>Since each row depends only on the previous row, keep only two rows (prev and curr), alternating between them. This gives length only. For the actual LCS, use Hirschberg's divide-and-conquer algorithm.</details>

5. **What is the LCS of "ABCD" and "ACBD"?**
   <details><summary>Answer</summary>"ABD" or "ACD", both of length 3. The LCS is not necessarily unique — multiple subsequences of the same maximum length may exist.</details>

---

| [⬅️ Previous: Applications (Manacher's)](../08-Manachers-Algorithm/05-applications.md) | [Next: Longest Common Substring ➡️](02-longest-common-substring.md) |
|:---|---:|
