# Chapter 3.6 — Step-by-Step Example

> **Unit 3: KMP Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

A complete, detailed walkthrough of KMP on a realistic example — from LPS
construction to finding all matches — with every comparison shown.

---

## 1. Problem

```
  Text:     T = "AABAACAADAABAABA"    (n = 15)
  Pattern:  P = "AABA"                (m = 4)

  Find all occurrences of P in T.
```

---

## 2. Phase 1: Build the LPS Array

```
  P = "AABA"
  Initialize: LPS = [0, _, _, _],  len = 0,  i = 1

  ┌────────────────────────────────────────────────────┐
  │ Step 1: i=1                                        │
  │   P[1]='A'  vs  P[0]='A'  →  MATCH                │
  │   len = 1,  LPS[1] = 1,  i = 2                    │
  │   LPS = [0, 1, _, _]                              │
  │                                                    │
  │ Step 2: i=2                                        │
  │   P[2]='B'  vs  P[1]='A'  →  MISMATCH             │
  │   len = 1 ≠ 0 → fallback: len = LPS[0] = 0       │
  │                                                    │
  │   P[2]='B'  vs  P[0]='A'  →  MISMATCH             │
  │   len = 0 → LPS[2] = 0,  i = 3                    │
  │   LPS = [0, 1, 0, _]                              │
  │                                                    │
  │ Step 3: i=3                                        │
  │   P[3]='A'  vs  P[0]='A'  →  MATCH                │
  │   len = 1,  LPS[3] = 1,  i = 4                    │
  │   LPS = [0, 1, 0, 1]                              │
  └────────────────────────────────────────────────────┘

  Final LPS = [0, 1, 0, 1]

    Index:   0   1   2   3
  Pattern:   A   A   B   A
      LPS:   0   1   0   1
```

### LPS Interpretation

```
  LPS[3] = 1:  "AABA" has prefix "A" = suffix "A"
  LPS[1] = 1:  "AA"   has prefix "A" = suffix "A"
  LPS[0] = 0:  "A"    — no proper prefix
  LPS[2] = 0:  "AAB"  — no common prefix/suffix
```

---

## 3. Phase 2: Pattern Matching

```
  T = "AABAACAADAABAABA"
  P = "AABA"
  LPS = [0, 1, 0, 1]

  i = 0 (text pointer),  j = 0 (pattern pointer)
```

### Detailed Comparison Table

```
  Step │ i │ j │ T[i] │ P[j] │ Action          │ After: i  j │ Note
  ─────┼───┼───┼──────┼──────┼─────────────────┼─────────────┼──────────────
    1  │ 0 │ 0 │  A   │  A   │ Match           │  1    1     │
    2  │ 1 │ 1 │  A   │  A   │ Match           │  2    2     │
    3  │ 2 │ 2 │  B   │  B   │ Match           │  3    3     │
    4  │ 3 │ 3 │  A   │  A   │ Match           │  4    4     │
       │   │   │      │      │ j==m=4!         │             │ 🎯 MATCH at 0
       │   │   │      │      │ j=LPS[3]=1      │  4    1     │ Continue
    5  │ 4 │ 1 │  A   │  A   │ Match           │  5    2     │
    6  │ 5 │ 2 │  C   │  B   │ Mismatch        │             │
       │   │   │      │      │ j=LPS[1]=1      │  5    1     │ Fallback
    7  │ 5 │ 1 │  C   │  A   │ Mismatch        │             │
       │   │   │      │      │ j=LPS[0]=0      │  5    0     │ Fallback
    8  │ 5 │ 0 │  C   │  A   │ Mismatch, j=0   │  6    0     │ Advance i
    9  │ 6 │ 0 │  A   │  A   │ Match           │  7    1     │
   10  │ 7 │ 1 │  A   │  A   │ Match           │  8    2     │
   11  │ 8 │ 2 │  D   │  B   │ Mismatch        │             │
       │   │   │      │      │ j=LPS[1]=1      │  8    1     │ Fallback
   12  │ 8 │ 1 │  D   │  A   │ Mismatch        │             │
       │   │   │      │      │ j=LPS[0]=0      │  8    0     │ Fallback
   13  │ 8 │ 0 │  D   │  A   │ Mismatch, j=0   │  9    0     │ Advance i
   14  │ 9 │ 0 │  A   │  A   │ Match           │ 10    1     │
   15  │10 │ 1 │  A   │  A   │ Match           │ 11    2     │
   16  │11 │ 2 │  B   │  B   │ Match           │ 12    3     │
   17  │12 │ 3 │  A   │  A   │ Match           │ 13    4     │
       │   │   │      │      │ j==m=4!         │             │ 🎯 MATCH at 9
       │   │   │      │      │ j=LPS[3]=1      │ 13    1     │ Continue
   18  │13 │ 1 │  A   │  A   │ Match           │ 14    2     │
   19  │14 │ 2 │  B   │  B   │ Match           │ 15    3     │  
   20  │15 │ -- │      │      │ i=15=n → STOP   │             │
```

Wait — let me recount. T has indices 0–14 (length 15). Let me recheck step 19.

```
  Step 19: i=14, j=2:  T[14]='B', P[2]='B' → Match → i=15, j=3
  
  i=15 = n → loop ends.
  
  j = 3 ≠ m = 4, so no final match here.
  
  But wait — let me recheck T.
  T = "AABAACAADAABAABA"
       0123456789...

  T[0..14]:
  0:A 1:A 2:B 3:A 4:A 5:C 6:A 7:A 8:D 9:A 10:A 11:B 12:A 13:A 14:B

  Hmm, that's only 15 chars. Let me count again:
  A-A-B-A-A-C-A-A-D-A-A-B-A-A-B-A  → 16 chars!
  0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15

  n = 16. Let me redo from step 18.
```

### Corrected Continuation (n = 16)

```
  Step │ i │ j │ T[i] │ P[j] │ Action          │ After: i  j │ Note
  ─────┼───┼───┼──────┼──────┼─────────────────┼─────────────┼──────────────
   18  │13 │ 1 │  A   │  A   │ Match           │ 14    2     │
   19  │14 │ 2 │  B   │  B   │ Match           │ 15    3     │
   20  │15 │ 3 │  A   │  A   │ Match           │ 16    4     │
       │   │   │      │      │ j==m=4!         │             │ 🎯 MATCH at 12
       │   │   │      │      │ j=LPS[3]=1      │ 16    1     │
       │   │   │      │      │ i=16=n → STOP   │             │
```

---

## 4. Visual Alignment

```
  T:  A  A  B  A  A  C  A  A  D  A  A  B  A  A  B  A
      0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15

  Match 1 at index 0:
  T:  A  A  B  A  A  C  A  A  D  A  A  B  A  A  B  A
      ───────────
  P:  A  A  B  A                                         ✓

  Match 2 at index 9:
  T:  A  A  B  A  A  C  A  A  D  A  A  B  A  A  B  A
                                 ───────────
  P:                             A  A  B  A              ✓

  Match 3 at index 12:
  T:  A  A  B  A  A  C  A  A  D  A  A  B  A  A  B  A
                                          ───────────
  P:                                      A  A  B  A    ✓

  Result: [0, 9, 12]
```

---

## 5. Comparison Count Analysis

```
  Total comparisons made:
  Steps: 1-4 (4 matches), 5 (1 match), 6-8 (3 mismatches + 1 advance),
         9-10 (2 matches), 11-13 (3 mismatches + 1 advance),
         14-17 (4 matches), 18-20 (3 matches)

  Total ≈ 20 comparisons

  Brute force would need: (16 - 4 + 1) × 4 = 52 worst case

  KMP savings: ~60% fewer comparisons!

  Note: Total comparisons ≤ 2n - 1 = 31.
  Actual = 20 < 31 ✓
```

---

## 6. Summary of This Example

```
  ┌────────────────────────────────────────────┐
  │  Input                                     │
  │    T = "AABAACAADAABAABA"  (n = 16)        │
  │    P = "AABA"              (m = 4)         │
  │                                            │
  │  LPS Array: [0, 1, 0, 1]                  │
  │                                            │
  │  Matches found at: [0, 9, 12]             │
  │                                            │
  │  Total comparisons: ~20                    │
  │  Brute force would need: up to 52          │
  │                                            │
  │  i never went backward (only forward)      │
  │  j fell back via LPS 4 times               │
  └────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Phase | Work Done |
|-------|-----------|
| LPS Build | 5 comparisons for m = 4 |
| Matching | ~20 comparisons for n = 16 |
| Total | ~25 comparisons |
| Matches found | 3 (at indices 0, 9, 12) |
| Brute force alternative | Up to 52 comparisons |

---

## ❓ Quick Revision Questions

1. **In this example, how many times did the text pointer i go backward?**
   <details><summary>Answer</summary>Zero times — the text pointer never goes backward in KMP.</details>

2. **What is LPS[3] for pattern "AABA" and what does it mean?**
   <details><summary>Answer</summary>LPS[3] = 1, meaning the longest proper prefix of "AABA" that is also a suffix is "A" (length 1).</details>

3. **After finding a match at position 0, what is j set to?**
   <details><summary>Answer</summary>j = LPS[3] = 1, meaning we already know 1 character matches for the next potential overlap.</details>

4. **How many matches are overlapping in this example?**
   <details><summary>Answer</summary>Matches at 9 and 12 overlap (9 + 4 = 13 > 12), sharing positions 12-12.</details>

5. **What would happen if we set j = 0 after each match instead of j = LPS[m-1]?**
   <details><summary>Answer</summary>We would miss overlapping matches. In this case, the match at index 12 might still be found (since non-overlap can still detect it here), but in general, overlapping matches would be missed.</details>

---

| [⬅️ Previous: Applications](05-applications.md) | [Next Unit: Rabin-Karp Algorithm ➡️](../04-Rabin-Karp-Algorithm/01-rolling-hash-concept.md) |
|:---|---:|
