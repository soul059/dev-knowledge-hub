# Chapter 1: Pattern Matching Basics

[← Previous: String Comparison](../07-String-Fundamentals/06-string-comparison.md) | [Back to README](../README.md) | [Next: Brute Force String Matching →](02-brute-force-matching.md)

---

## Overview

**Pattern matching** (or string searching) is the problem of finding occurrences of a short string (pattern) within a longer string (text). It is one of the most fundamental problems in computer science, with applications from text editors to DNA analysis.

---

## Problem Statement

```
  Given:
    text    T = "ABCABCABDAB"   (length n)
    pattern P = "ABCABD"        (length m)

  Find: all positions where P occurs in T

  T: A B C A B C A B D A B
     0 1 2 3 4 5 6 7 8 9 10

  Try at position 0: ABCAB[C] ≠ ABCAB[D]  ✗
  Try at position 1: B ≠ A                 ✗
  ...
  Try at position 3: ABCABD                ✓ Found at index 3!
  
  Wait — let's check: T[3..8] = "ABCABD" = P  ✓
```

---

## Terminology

```
  ┌────────────────────────────────────────────────┐
  │  TEXT (or haystack):                            │
  │  The longer string we search in                │
  │  Length: n                                      │
  │                                                │
  │  PATTERN (or needle):                           │
  │  The shorter string we search for              │
  │  Length: m                                      │
  │                                                │
  │  MATCH (or occurrence):                         │
  │  Position i where T[i..i+m-1] = P              │
  │                                                │
  │  SHIFT:                                         │
  │  How far we move the pattern after a            │
  │  mismatch or match                             │
  └────────────────────────────────────────────────┘
```

---

## Approaches Overview

```
  ┌──────────────────────────────────────────────────────┐
  │                 PATTERN MATCHING ALGORITHMS           │
  │                                                      │
  │  1. Brute Force     O(n × m)    Simple, no preprocess│
  │  2. KMP             O(n + m)    Prefix function       │
  │  3. Rabin-Karp      O(n + m)*   Rolling hash          │
  │  4. Z-Algorithm     O(n + m)    Z-array               │
  │  5. Boyer-Moore     O(n/m) best Bad char + good suffix│
  │  6. Aho-Corasick    O(n + m + z) Multiple patterns    │
  │  7. Suffix Array    O(n log n)  Preprocess text       │
  │                                                      │
  │  * = average case                                    │
  └──────────────────────────────────────────────────────┘
```

---

## Visual: The Sliding Window View

```
  Pattern matching = sliding a window of size m across text

  T: [ A  B  C  A  B  C  A  B  D  A  B ]
  P: [ A  B  C  A  B  D ]
      ↑ compare window

  Shift by 1:
  T: [ A  B  C  A  B  C  A  B  D  A  B ]
  P:    [ A  B  C  A  B  D ]
         ↑ compare window

  Smart algorithms skip positions:
  T: [ A  B  C  A  B  C  A  B  D  A  B ]
  P:          [ A  B  C  A  B  D ]
               ↑ skip ahead! (KMP, Boyer-Moore)

  Key question: HOW MANY POSITIONS CAN WE SKIP?
  This is what separates O(nm) from O(n+m).
```

---

## Single Pattern vs Multiple Pattern

```
  Single pattern:
  "Find 'ABC' in text"
  → Brute force, KMP, Rabin-Karp, Z, Boyer-Moore

  Multiple patterns:
  "Find all of {'ABC', 'DEF', 'GHI'} in text"
  → Aho-Corasick, multiple Rabin-Karp

  ┌──────────────────────────────────────┐
  │  In this unit, we cover:             │
  │  • Brute Force (Chapter 2)          │
  │  • KMP (Chapter 3)                  │
  │  • Rabin-Karp (Chapter 4)           │
  │  • Z-Algorithm (Chapter 5)          │
  │  • Comparison (Chapter 6)           │
  └──────────────────────────────────────┘
```

---

## When Does It Matter?

```
  For small strings (n, m < 100):
  → Brute force is fine. Don't optimize!

  For large texts (n > 10⁶):
  → O(nm) can timeout. Need O(n+m) algorithms.

  For repeated searching in same text:
  → Preprocess text (suffix array/tree)

  For multiple patterns in one text:
  → Aho-Corasick or build a trie
```

---

## Real-World Applications

```
  ┌──────────────────────────────────────────────┐
  │ • Text editors: Ctrl+F / Find & Replace      │
  │ • Search engines: query matching              │
  │ • Bioinformatics: DNA sequence matching       │
  │ • Network security: packet inspection         │
  │ • Compilers: lexical analysis (tokenization)  │
  │ • Spam filters: keyword detection             │
  │ • Plagiarism detection: document comparison   │
  └──────────────────────────────────────────────┘
```

---

## Summary Table

| Concept | Detail |
|---------|--------|
| Pattern matching | Find pattern P in text T |
| Text length | n |
| Pattern length | m |
| Brute force | O(n × m), no preprocessing |
| KMP | O(n + m), uses prefix function |
| Rabin-Karp | O(n + m) avg, uses rolling hash |
| Z-Algorithm | O(n + m), uses Z-array |

---

## Quick Revision Questions

1. **What is the pattern matching problem?** Define text, pattern, and match.

2. **Why is brute force O(n × m)?** When is this acceptable?

3. **What do KMP, Rabin-Karp, and Z-Algorithm have in common?** How do they achieve O(n + m)?

4. **When would you preprocess the text instead of the pattern?**

5. **Name three real-world applications** of pattern matching.

---

[← Previous: String Comparison](../07-String-Fundamentals/06-string-comparison.md) | [Back to README](../README.md) | [Next: Brute Force String Matching →](02-brute-force-matching.md)
