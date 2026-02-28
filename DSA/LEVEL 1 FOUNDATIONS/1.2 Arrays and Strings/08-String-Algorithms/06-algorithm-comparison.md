# Chapter 6: Algorithm Comparison

[← Previous: Z-Algorithm](05-z-algorithm.md) | [Back to README](../README.md) | [Next: Reverse String Problems →](../09-String-Manipulation/01-reverse-string-problems.md)

---

## Overview

We've now studied four string matching algorithms: Brute Force, KMP, Rabin-Karp, and the Z-Algorithm. This chapter compares them head-to-head so you can choose the right tool for each situation.

---

## Side-by-Side Comparison

```
  ┌────────────────┬──────────┬──────────┬──────────┬──────────┐
  │ Feature        │ Brute    │ KMP      │ Rabin    │ Z-Algo   │
  │                │ Force    │          │ Karp     │          │
  ├────────────────┼──────────┼──────────┼──────────┼──────────┤
  │ Preprocess     │ None     │ O(m)     │ O(m)     │ O(n+m)   │
  │ Search         │ O(nm)    │ O(n)     │ O(n)*    │ O(n+m)   │
  │ Total          │ O(nm)    │ O(n+m)   │ O(n+m)*  │ O(n+m)   │
  │ Worst case     │ O(nm)    │ O(n+m)   │ O(nm)    │ O(n+m)   │
  │ Space          │ O(1)     │ O(m)     │ O(1)     │ O(n+m)   │
  │ Multi-pattern  │ Poor     │ No†      │ Good     │ No       │
  │ Online‡        │ Yes      │ Yes      │ Yes      │ No       │
  │ Simplicity     │ ★★★★★   │ ★★★      │ ★★★★    │ ★★★★    │
  └────────────────┴──────────┴──────────┴──────────┴──────────┘
  
  * = average case
  † KMP alone; Aho-Corasick extends it for multiple patterns
  ‡ Online = can process text character by character
```

---

## When to Use Which?

```
  ┌──────────────────────────────────────────────────────────┐
  │  DECISION GUIDE                                          │
  │                                                          │
  │  Is the input small (n, m < 1000)?                      │
  │    YES → Brute Force (simplest)                         │
  │                                                          │
  │  Need guaranteed O(n+m) worst case?                     │
  │    YES → KMP or Z-Algorithm                             │
  │                                                          │
  │  Searching for MULTIPLE patterns?                       │
  │    YES → Rabin-Karp (easy) or Aho-Corasick (advanced)  │
  │                                                          │
  │  Need to find all occurrences?                          │
  │    ANY → All four work, KMP/Z are most efficient        │
  │                                                          │
  │  Working with very large alphabets?                     │
  │    → Rabin-Karp (alphabet-independent hashing)          │
  │                                                          │
  │  Need online processing (streaming)?                    │
  │    → KMP (processes text left to right)                 │
  │                                                          │
  │  Competitive programming / interviews?                  │
  │    → KMP (most popular) or Z-algo (easy to code)        │
  └──────────────────────────────────────────────────────────┘
```

---

## Preprocessing Comparison

```
  BRUTE FORCE: No preprocessing
  - Just start comparing immediately
  - Good when pattern is tiny or you search once

  KMP: Build LPS (failure function) — O(m)
  - LPS[i] = longest proper prefix = suffix of P[0..i]
  - "ABABAC" → [0, 0, 1, 2, 3, 0]
  - Tells us WHERE to restart on mismatch

  RABIN-KARP: Compute pattern hash — O(m)
  - hash(P) using polynomial rolling hash
  - Choose good prime q for fewer collisions

  Z-ALGORITHM: Build Z-array — O(n+m)
  - Concatenate P + "$" + T, build full Z-array
  - More space but very elegant
```

---

## Worst Case Scenarios

```
  BRUTE FORCE worst case: O(nm)
  text = "AAAAAAA...A"  pattern = "AAAB"
  Almost matches at every position, fails at last char.

  KMP worst case: O(n+m) — always!
  Text pointer never goes backward.
  Even on adversarial input: guaranteed linear.

  RABIN-KARP worst case: O(nm)
  All windows hash to same value (extreme collision).
  Every window triggers O(m) verification.
  In practice: extremely rare with good hash.

  Z-ALGORITHM worst case: O(n+m) — always!
  R pointer only increases; guaranteed linear.

  ┌─────────────────────────────────────────┐
  │  KMP and Z have GUARANTEED O(n+m)      │
  │  Rabin-Karp is O(n+m) EXPECTED         │
  │  Brute Force is O(nm) worst case       │
  └─────────────────────────────────────────┘
```

---

## Space Comparison

```
  BRUTE FORCE:   O(1) — just two indices
  KMP:           O(m) — LPS array
  RABIN-KARP:    O(1) — two hash values
  Z-ALGORITHM:   O(n+m) — full Z-array on concatenated string

  For memory-constrained environments:
  Brute Force or Rabin-Karp (O(1) extra space)
```

---

## Multi-Pattern Search

```
  Find multiple patterns P₁, P₂, ..., Pₖ in text T?

  BRUTE FORCE:   Run k times → O(k × n × m)
  KMP:           Run k times → O(k × (n + m))
  RABIN-KARP:    Hash all patterns, one pass → O(n + k×m)
  Z-ALGORITHM:   Run k times → O(k × (n + m))
  AHO-CORASICK:  One pass → O(n + total pattern length + matches)

  Rabin-Karp multi-pattern:
  1. Hash each pattern → put in hash set
  2. Roll hash through text
  3. Check if hash is in set → O(1) lookup per window!

  This is Rabin-Karp's BIGGEST advantage.
```

---

## Practical Performance

```
  In practice (real-world text, large alphabets):

  ┌───────────────────────────────────┐
  │  Boyer-Moore  >>>>  (skips chars) │  ← not covered in detail
  │  Brute Force  >>    (cache-friendly)
  │  KMP          >>    (guaranteed)  │
  │  Rabin-Karp   >>    (hash-based)  │
  │  Z-Algorithm  >>    (elegant)     │
  └───────────────────────────────────┘

  Surprise: brute force is often FAST in practice!
  - Cache-friendly (sequential access)
  - Large alphabets → quick rejection
  - No preprocessing overhead
  - Simple code → well-optimized by compiler

  For competitive programming:
  KMP and Z are the go-to algorithms.
```

---

## Boyer-Moore (Bonus)

```
  Not covered in detail, but important to know:

  Key ideas:
  1. Compare pattern RIGHT to LEFT
  2. Bad character rule: skip based on mismatched char
  3. Good suffix rule: skip based on matched suffix

  Best case: O(n/m) — can skip entire pattern!
  Worst case: O(nm)
  Average: sub-linear for large alphabets

  text:    "THIS IS A SIMPLE EXAMPLE"
  pattern: "EXAMPLE"

  Compare from right:
  "THIS IS" ... 'S' ≠ 'E' and 'S' not in "EXAMPL"
  → skip 7 characters at once!

  Most practical text search tools use Boyer-Moore.
```

---

## Interview Cheat Sheet

```
  ┌──────────────────────────────────────────────────┐
  │  INTERVIEW TIP                                    │
  │                                                   │
  │  "How would you search for a pattern in text?"    │
  │                                                   │
  │  Level 1: Mention brute force O(nm)              │
  │  Level 2: Explain KMP with LPS array O(n+m)     │
  │  Level 3: Discuss trade-offs (space, simplicity) │
  │  Level 4: Mention multi-pattern (Rabin-Karp)     │
  │  Level 5: Mention Boyer-Moore for practical use  │
  │                                                   │
  │  Most asked: KMP (build LPS, explain why O(n+m))│
  └──────────────────────────────────────────────────┘
```

---

## Summary Table

| Algorithm | Time | Space | Worst Case | Best For |
|-----------|------|-------|------------|----------|
| Brute Force | O(nm) | O(1) | O(nm) | Small inputs, simplicity |
| KMP | O(n+m) | O(m) | O(n+m) | Guaranteed linear, streaming |
| Rabin-Karp | O(n+m)* | O(1) | O(nm) | Multi-pattern search |
| Z-Algorithm | O(n+m) | O(n+m) | O(n+m) | Easy to code, string periods |
| Boyer-Moore | O(n/m)† | O(m+σ) | O(nm) | Large alphabets, practice |

*average †best case

---

## Quick Revision Questions

1. **Rank the four algorithms by worst-case time complexity.** Which two guarantee O(n+m)?

2. **When would you choose Rabin-Karp over KMP?** Give a concrete scenario.

3. **Why is Z-Algorithm not considered "online"?** What does online mean here?

4. **Why might brute force outperform KMP on typical English text?**

5. **Which algorithm uses the most space?** Which uses the least? Why?

6. **Explain the key preprocessing difference** between KMP (LPS) and Z-Algorithm (Z-array).

---

[← Previous: Z-Algorithm](05-z-algorithm.md) | [Back to README](../README.md) | [Next: Reverse String Problems →](../09-String-Manipulation/01-reverse-string-problems.md)
