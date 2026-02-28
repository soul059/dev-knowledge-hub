# Chapter 3: KMP Algorithm (Knuth-Morris-Pratt)

[← Previous: Brute Force String Matching](02-brute-force-matching.md) | [Back to README](../README.md) | [Next: Rabin-Karp Algorithm →](04-rabin-karp.md)

---

## Overview

The **Knuth-Morris-Pratt (KMP)** algorithm searches for a pattern in text in **O(n + m)** time by preprocessing the pattern to build a **failure function** (also called the prefix function or partial match table). When a mismatch occurs, instead of starting over, KMP uses this table to skip ahead intelligently.

---

## Core Idea

```
  Brute force problem: after a mismatch, we restart 
  pattern comparison from the beginning.

  text:    A B C A B C A B D
  pattern: A B C A B D
                     ↑ mismatch at j=5

  Brute force: go back to i=1, j=0 (wasteful!)
  
  KMP insight: We already matched "ABCAB".
  Within "ABCAB", the prefix "AB" = the suffix "AB"!

  So we know the text has "AB" just before the mismatch.
  We can continue matching from pattern position j=2!

  text:    A B C A B C A B D
  pattern:       A B C A B D     ← shifted by 3, j=2
                     ↑ continue from here!
```

---

## The Failure Function (LPS Array)

```
  LPS = Longest Proper Prefix which is also Suffix

  For pattern P, lps[i] = length of the longest proper 
  prefix of P[0..i] that is also a suffix of P[0..i].

  "Proper" means the prefix ≠ the entire string.

  Pattern: A B C A B D
  Index:   0 1 2 3 4 5

  lps[0] = 0    "A"       → no proper prefix = suffix
  lps[1] = 0    "AB"      → "A" ≠ "B"
  lps[2] = 0    "ABC"     → "A"≠"C", "AB"≠"BC"
  lps[3] = 1    "ABCA"    → "A" = "A" ✓ (length 1)
  lps[4] = 2    "ABCAB"   → "AB" = "AB" ✓ (length 2)
  lps[5] = 0    "ABCABD"  → no match

  lps = [0, 0, 0, 1, 2, 0]
```

### Visual of LPS

```
  P = "ABCABD"

  "ABCAB":
   A B           ← prefix of length 2
         A B     ← suffix of length 2
   ─────────
   A B C A B     ← they match! lps[4] = 2

  This means: if we matched up to position 4 and then
  fail at position 5, we can restart at position 2
  (because the first 2 chars are already matched).
```

---

## Building the LPS Array

```
FUNCTION buildLPS(pattern, m):
    lps = array of size m, all zeros
    length = 0         // length of previous longest prefix suffix
    i = 1

    WHILE i < m:
        IF pattern[i] == pattern[length]:
            length = length + 1
            lps[i] = length
            i = i + 1
        ELSE:
            IF length > 0:
                length = lps[length - 1]    // fall back!
                // Don't increment i
            ELSE:
                lps[i] = 0
                i = i + 1

    RETURN lps
```

### Building LPS: Trace

```
  pattern = "ABABAC"
  lps = [0, 0, 0, 0, 0, 0]
  length = 0, i = 1

  i=1: P[1]='B' ≠ P[0]='A', length=0 → lps[1]=0, i=2
  i=2: P[2]='A' = P[0]='A' → length=1, lps[2]=1, i=3
  i=3: P[3]='B' = P[1]='B' → length=2, lps[3]=2, i=4
  i=4: P[4]='A' = P[2]='A' → length=3, lps[4]=3, i=5
  i=5: P[5]='C' ≠ P[3]='B'
       length=3 > 0 → length = lps[2] = 1
       P[5]='C' ≠ P[1]='B'
       length=1 > 0 → length = lps[0] = 0
       P[5]='C' ≠ P[0]='A'
       length=0 → lps[5]=0, i=6

  lps = [0, 0, 1, 2, 3, 0]
```

---

## KMP Search Algorithm

```
FUNCTION kmpSearch(text, pattern):
    n = length(text)
    m = length(pattern)
    lps = buildLPS(pattern, m)
    matches = []

    i = 0    // index in text
    j = 0    // index in pattern

    WHILE i < n:
        IF text[i] == pattern[j]:
            i = i + 1
            j = j + 1

        IF j == m:
            matches.append(i - j)    // match found!
            j = lps[j - 1]          // look for next match
        ELSE IF i < n AND text[i] ≠ pattern[j]:
            IF j > 0:
                j = lps[j - 1]      // use failure function
            ELSE:
                i = i + 1            // no fallback, advance text
    
    RETURN matches
```

---

## Full KMP Trace

```
  text    = "ABABDABABAC"
  pattern = "ABABAC"
  lps     = [0, 0, 1, 2, 3, 0]

  i=0, j=0: T[0]='A' = P[0]='A' → i=1, j=1
  i=1, j=1: T[1]='B' = P[1]='B' → i=2, j=2
  i=2, j=2: T[2]='A' = P[2]='A' → i=3, j=3
  i=3, j=3: T[3]='B' = P[3]='B' → i=4, j=4
  i=4, j=4: T[4]='D' ≠ P[4]='A'
             j=4 > 0 → j = lps[3] = 2
             (We know T[2..3]="AB" = P[0..1], skip!)

  i=4, j=2: T[4]='D' ≠ P[2]='A'
             j=2 > 0 → j = lps[1] = 0

  i=4, j=0: T[4]='D' ≠ P[0]='A'
             j=0 → i=5

  i=5, j=0: T[5]='A' = P[0]='A' → i=6, j=1
  i=6, j=1: T[6]='B' = P[1]='B' → i=7, j=2
  i=7, j=2: T[7]='A' = P[2]='A' → i=8, j=3
  i=8, j=3: T[8]='B' = P[3]='B' → i=9, j=4
  i=9, j=4: T[9]='A' = P[4]='A' → i=10, j=5
  i=10, j=5: T[10]='C' = P[5]='C' → i=11, j=6

  j=6 = m → MATCH at position 11 - 6 = 5!
  j = lps[5] = 0

  i=11: loop ends.

  Result: Match at position 5 ("ABABAC" starts at index 5)
```

---

## Why KMP Is O(n + m)

```
  Key insight: i NEVER goes backward!

  In brute force: when mismatch, i goes back to i-j+1
  In KMP: i only moves forward, j falls back via lps

  i advances at most n times.
  j can fall back, but each fallback is "paid for" by
  a previous advance of j.

  Total: at most 2n comparisons for search + O(m) for LPS build
  
  Time: O(n + m)
  
  ┌─────────────────────────────────────────────┐
  │  Amortized argument:                         │
  │  Each comparison either:                     │
  │  1. Advances i (at most n times)             │
  │  2. Reduces j (j was increased before)       │
  │                                              │
  │  Total work bounded by 2n + m               │
  └─────────────────────────────────────────────┘
```

---

## LPS Examples

```
  Pattern: "AABAAAB"
  lps:     [0,1,0,1,2,2,3]

  Pattern: "ABCABC"
  lps:     [0,0,0,1,2,3]

  Pattern: "AAAA"
  lps:     [0,1,2,3]

  Pattern: "ABCD"
  lps:     [0,0,0,0]

  Pattern: "AABCAAB"
  lps:     [0,1,0,0,1,2,3]
```

---

## Complexity

| Metric | Value |
|--------|-------|
| Preprocessing (LPS) | O(m) |
| Searching | O(n) |
| Total | O(n + m) |
| Space | O(m) for LPS array |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Core idea | Use prefix-suffix info to skip comparisons |
| LPS array | Longest proper prefix = suffix for each position |
| Key property | Text pointer i never moves backward |
| Preprocessing | O(m) to build LPS |
| Search | O(n) — each char examined at most twice |
| Total time | O(n + m) |
| Space | O(m) |

---

## Quick Revision Questions

1. **What is the LPS (failure function)?** Compute it for "AABAAAC".

2. **Why does the text pointer never go backward in KMP?** What is the amortized argument?

3. **Trace KMP search** for pattern "ABAB" in text "ABABCABABAB".

4. **What happens when there's a mismatch at j=0?** What does KMP do differently from j > 0?

5. **Why is KMP O(n + m) but brute force O(n × m)?** What "knowledge" does KMP preserve?

6. **Build the LPS array for "AABAAB".** Show each step.

---

[← Previous: Brute Force String Matching](02-brute-force-matching.md) | [Back to README](../README.md) | [Next: Rabin-Karp Algorithm →](04-rabin-karp.md)
