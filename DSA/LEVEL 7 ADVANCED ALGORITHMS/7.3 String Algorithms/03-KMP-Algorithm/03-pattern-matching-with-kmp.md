# Chapter 3.3 — Pattern Matching with KMP

> **Unit 3: KMP Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

With the LPS array in hand, we can now perform pattern matching in O(n + m)
time. This chapter shows the complete KMP matching algorithm with traces.

---

## 1. The KMP Matching Algorithm

```
  KMP_SEARCH(T, P):
      n ← |T|,  m ← |P|
      LPS ← BUILD_LPS(P)

      i ← 0    // index in text T
      j ← 0    // index in pattern P

      while i < n:
          if T[i] == P[j]:
              i ← i + 1
              j ← j + 1

          if j == m:
              // Full pattern matched at index (i - j)
              report match at position (i - j)
              j ← LPS[j - 1]    // continue searching for more matches

          else if i < n AND T[i] ≠ P[j]:
              if j ≠ 0:
                  j ← LPS[j - 1]    // fallback using LPS
              else:
                  i ← i + 1         // no fallback possible, advance text
```

---

## 2. How It Works — Visual Explanation

```
  Text:     ... X X X A B A B C A B A B Y Y Y ...
  Pattern:              A B A B C A B A B

  When characters match: advance both i and j.
  When mismatch at j:

  ┌───────────────────────────────────────────────────────┐
  │  j ≠ 0:  We've matched P[0..j-1].                   │
  │           Set j = LPS[j-1].                          │
  │           The prefix P[0..LPS[j-1]-1] already        │
  │           matches the suffix of what we matched.     │
  │           Continue comparing from this new j.        │
  │           DO NOT move i backward!                    │
  │                                                      │
  │  j == 0: First character doesn't match.             │
  │           Simply advance i by 1.                     │
  └───────────────────────────────────────────────────────┘
```

---

## 3. Complete Step-by-Step Trace

```
  T = "AABABCABABCABABD"   (n = 16)
  P = "ABABD"              (m = 5)

  Step 1: Build LPS for "ABABD"
  ─────────────────────────────
  i=0: LPS[0] = 0
  i=1: B≠A → LPS[1] = 0
  i=2: A==A → LPS[2] = 1
  i=3: B==B → LPS[3] = 2
  i=4: D≠A → fallback len=LPS[1]=0 → D≠A → LPS[4] = 0

  LPS = [0, 0, 1, 2, 0]

  Step 2: Match
  ──────────────
  i=0, j=0:  T[0]='A' == P[0]='A' → i=1, j=1
  i=1, j=1:  T[1]='A' ≠  P[1]='B' → j≠0 → j=LPS[0]=0
  i=1, j=0:  T[1]='A' == P[0]='A' → i=2, j=1
  i=2, j=1:  T[2]='B' == P[1]='B' → i=3, j=2
  i=3, j=2:  T[3]='A' == P[2]='A' → i=4, j=3
  i=4, j=3:  T[4]='B' == P[3]='B' → i=5, j=4
  i=5, j=4:  T[5]='C' ≠  P[4]='D' → j≠0 → j=LPS[3]=2
  i=5, j=2:  T[5]='C' ≠  P[2]='A' → j≠0 → j=LPS[1]=0
  i=5, j=0:  T[5]='C' ≠  P[0]='A' → j==0 → i=6
  i=6, j=0:  T[6]='A' == P[0]='A' → i=7, j=1
  i=7, j=1:  T[7]='B' == P[1]='B' → i=8, j=2
  i=8, j=2:  T[8]='A' == P[2]='A' → i=9, j=3
  i=9, j=3:  T[9]='B' == P[3]='B' → i=10, j=4
  i=10,j=4:  T[10]='C' ≠ P[4]='D' → j≠0 → j=LPS[3]=2
  i=10,j=2:  T[10]='C' ≠ P[2]='A' → j≠0 → j=LPS[1]=0
  i=10,j=0:  T[10]='C' ≠ P[0]='A' → j==0 → i=11
  i=11,j=0:  T[11]='A' == P[0]='A' → i=12, j=1
  i=12,j=1:  T[12]='B' == P[1]='B' → i=13, j=2
  i=13,j=2:  T[13]='A' == P[2]='A' → i=14, j=3
  i=14,j=3:  T[14]='B' == P[3]='B' → i=15, j=4
  i=15,j=4:  T[15]='D' == P[4]='D' → i=16, j=5

  j == m == 5 → MATCH at position (16 - 5) = 11 ✓

  T:  A A B A B C A B A B C A B A B D
                                ─────────
                                  11
```

---

## 4. Alignment Visualization

```
  T: A A B A B C A B A B C A B A B D
  
  Attempt 1 (i=0, j=0):
  P: A B A B D
     ▲ match, then mismatch at j=1

  Attempt 2 (i=1, j=0):        ← j fell back to 0
  P:   A B A B D
       ▲──▲──▲──▲ match 4, mismatch at j=4

  Attempt 3 (i=5, j=2):        ← j fell back to LPS[3]=2
  P:       A B A B D
               ▲ mismatch at j=2, then j=0

  Attempt 4 (i=6, j=0):
  P:             A B A B D
                 ▲──▲──▲──▲ match 4, mismatch at j=4

  Attempt 5 (i=10, j=2):       ← j fell back to LPS[3]=2
  P:                 A B A B D
                         ▲ mismatch, j=0

  Attempt 6 (i=11, j=0):
  P:                   A B A B D
                       ▲──▲──▲──▲──▲ FULL MATCH at 11! ✓

  Notice: i NEVER moves backward. Only j adjusts via LPS.
```

---

## 5. Code (Python)

```python
def kmp_search(text, pattern):
    n, m = len(text), len(pattern)
    if m == 0:
        return []
    
    lps = build_lps(pattern)
    matches = []
    
    i = 0  # index in text
    j = 0  # index in pattern
    
    while i < n:
        if text[i] == pattern[j]:
            i += 1
            j += 1
        
        if j == m:
            matches.append(i - j)
            j = lps[j - 1]  # look for next match
        elif i < n and text[i] != pattern[j]:
            if j != 0:
                j = lps[j - 1]
            else:
                i += 1
    
    return matches
```

---

## 6. Finding Overlapping Matches

```
  T = "AAAA",  P = "AA"

  LPS = [0, 1]

  i=0,j=0: A==A → i=1,j=1
  i=1,j=1: A==A → i=2,j=2 → j==m → match at 0, j=LPS[1]=1
  i=2,j=1: A==A → i=3,j=2 → j==m → match at 1, j=LPS[1]=1
  i=3,j=1: A==A → i=4,j=2 → j==m → match at 2, j=LPS[1]=1

  Matches: [0, 1, 2]  ← overlapping matches! ✓

  KMP naturally handles overlapping matches because after
  a match, j = LPS[m-1] (not 0), so it can detect the next
  overlapping occurrence immediately.
```

---

## 📝 Summary Table

| Aspect | Detail |
|--------|--------|
| Preprocessing | Build LPS array in O(m) |
| Matching | Single pass over text in O(n) |
| Total time | O(n + m) |
| Space | O(m) for LPS array |
| Text pointer | Never backtracks (only moves forward) |
| Pattern pointer | Falls back via LPS on mismatch |
| Overlapping matches | Automatically handled by j = LPS[m-1] |
| Non-overlapping | After match, set j = 0 instead |

---

## ❓ Quick Revision Questions

1. **What happens to j after a full match is found?**
   <details><summary>Answer</summary>j is set to LPS[m-1], enabling detection of overlapping matches.</details>

2. **Does the text pointer i ever move backward in KMP?**
   <details><summary>Answer</summary>No — i only moves forward. This is the key guarantee that makes KMP O(n).</details>

3. **How would you modify KMP to find only non-overlapping matches?**
   <details><summary>Answer</summary>After a match, set j = 0 instead of j = LPS[m-1].</details>

4. **What is the total time complexity of KMP including preprocessing?**
   <details><summary>Answer</summary>O(n + m): O(m) for LPS construction + O(n) for matching.</details>

5. **Why does KMP never need to re-examine a character in the text?**
   <details><summary>Answer</summary>The LPS array encodes which prefix of the pattern already matches a suffix of the matched portion, so we can continue from the right position without going back in the text.</details>

---

| [⬅️ Previous: LPS Array Construction](02-lps-array-construction.md) | [Next: Time Complexity ➡️](04-time-complexity.md) |
|:---|---:|
