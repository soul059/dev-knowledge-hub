# Chapter 2: Brute Force String Matching

[← Previous: Pattern Matching Basics](01-pattern-matching-basics.md) | [Back to README](../README.md) | [Next: KMP Algorithm →](03-kmp-algorithm.md)

---

## Overview

The brute force (naive) approach tries every possible alignment of the pattern against the text. It's simple, requires no preprocessing, and works well for short strings. Understanding its limitations motivates the advanced algorithms.

---

## Algorithm

```
FUNCTION bruteForceSearch(text, pattern):
    n = length(text)
    m = length(pattern)
    matches = []

    FOR i = 0 TO n - m:
        j = 0
        WHILE j < m AND text[i + j] == pattern[j]:
            j = j + 1

        IF j == m:
            matches.append(i)    // Found match at position i

    RETURN matches
```

---

## Visual Trace

```
  text    = "AABAACAADAAB"
  pattern = "AABA"

  i=0: A A B A       compare
       A A B A  ✓✓✓✓  MATCH at 0!

  i=1: A A B A A C A A D A A B
        A A B A
        A ✓ B≠A ✗     shift

  i=2: A A B A A C A A D A A B
          A A B A
          B≠A ✗        shift

  i=3: A A B A A C A A D A A B
            A A B A
            A ✓ A ✓ C≠B ✗  shift

  i=4: A A B A A C A A D A A B
              A A B A
              A ✓ C≠A ✗    shift

  i=5: A A B A A C A A D A A B
                A A B A
                C≠A ✗       shift

  i=6: A A B A A C A A D A A B
                  A A B A
                  A ✓ A ✓ D≠B ✗  shift

  i=7: A A B A A C A A D A A B
                    A A B A
                    A ✓ D≠A ✗    shift

  i=8: A A B A A C A A D A A B
                      A A B A
                      D≠A ✗       shift

  Matches found: [0]
```

---

## Worst Case: O(n × m)

```
  text    = "AAAAAAAAAB"  (n = 10)
  pattern = "AAAAB"       (m = 5)

  i=0: AAAAB
       AAAA✓ B≠A✗    compared 5 chars, failed at last

  i=1:  AAAAB
       AAAA✓ A≠B✗    compared 5 chars again!

  i=2:   AAAAB
       AAAA✓ A≠B✗    and again...

  ...every position compares m characters before failing!

  Total comparisons: (n - m + 1) × m
                   = 6 × 5 = 30

  In general: O(n × m)
```

---

## Best Case: O(n)

```
  text    = "ABCDEFGHIJ"
  pattern = "XYZ"

  i=0: A≠X ✗        1 comparison
  i=1: B≠X ✗        1 comparison
  i=2: C≠X ✗        1 comparison
  ...

  Each position fails at the FIRST character!
  Total: n - m + 1 comparisons ≈ O(n)

  Best case happens when first char of pattern
  doesn't appear (or rarely appears) in text.
```

---

## Average Case

```
  For random text over alphabet of size σ:

  Expected comparisons per position ≈ σ/(σ-1)

  For σ = 26 (lowercase English): ~1.04 per position
  → Average case ≈ O(n) for large alphabets!

  For σ = 2 (binary): ~2 per position
  → Average case closer to O(n × m) for small alphabets

  DNA (σ = 4): in between, more false starts
```

---

## Variations

### 1. Find First Occurrence Only

```
FUNCTION findFirst(text, pattern):
    n = length(text)
    m = length(pattern)

    FOR i = 0 TO n - m:
        j = 0
        WHILE j < m AND text[i + j] == pattern[j]:
            j = j + 1
        IF j == m:
            RETURN i        // return immediately
    
    RETURN -1               // not found
```

### 2. Count Occurrences

```
FUNCTION countOccurrences(text, pattern):
    count = 0
    n = length(text)
    m = length(pattern)

    FOR i = 0 TO n - m:
        j = 0
        WHILE j < m AND text[i + j] == pattern[j]:
            j = j + 1
        IF j == m:
            count = count + 1

    RETURN count
```

### 3. Overlapping vs Non-Overlapping

```
  text = "AAAA", pattern = "AA"

  Overlapping: found at 0, 1, 2 → count = 3
  Non-overlapping: found at 0, 2 → count = 2

  For non-overlapping: after a match at i, skip to i + m
```

---

## Why It's Not Good Enough

```
  Problem: brute force FORGETS what it learned!

  text:    A A B A A B A A B A B
  pattern: A A B A B

  i=0: A A B A [B≠A] ✗
       We learned T[0..3] = "AABA"
       But at i=1, we start comparing from scratch!

  i=1: A ✓ B≠A ✗
       We ALREADY knew T[1] = 'A' from previous!

  Smart algorithms (KMP, Z) REMEMBER partial matches
  and skip unnecessary comparisons.

  ┌─────────────────────────────────────────────┐
  │  Brute force: up to n×m comparisons         │
  │  KMP:         exactly n+m comparisons        │
  │                                              │
  │  For n=10⁶, m=10⁶:                          │
  │  Brute force: up to 10¹² operations (TLE!)  │
  │  KMP:         2×10⁶ operations (fast!)       │
  └─────────────────────────────────────────────┘
```

---

## Complexity Analysis

| Case | Comparisons | When |
|------|------------|------|
| Best | O(n) | Pattern's first char rarely in text |
| Average | O(n) | Large alphabet, random text |
| Worst | O(n × m) | Many partial matches (e.g., "AAAA..." and "AAAB") |
| Space | O(1) | No preprocessing |

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| Approach | Try every position, compare char by char |
| Preprocessing | None |
| Best case | O(n) — quick first-char rejections |
| Worst case | O(n × m) — many almost-matches |
| Space | O(1) |
| When to use | Short patterns, large alphabets, simple code needed |

---

## Quick Revision Questions

1. **Why is brute force O(n × m) in the worst case?** Give a concrete worst-case input.

2. **Why is the average case O(n) for English text?** What role does alphabet size play?

3. **What information does brute force "forget"** that smarter algorithms remember?

4. **Trace brute force search** for pattern "AB" in text "AABABAB". List all match positions.

5. **What is the difference between overlapping and non-overlapping matches?** Give an example.

6. **When is brute force the best choice?** Why bother with it at all?

---

[← Previous: Pattern Matching Basics](01-pattern-matching-basics.md) | [Back to README](../README.md) | [Next: KMP Algorithm →](03-kmp-algorithm.md)
