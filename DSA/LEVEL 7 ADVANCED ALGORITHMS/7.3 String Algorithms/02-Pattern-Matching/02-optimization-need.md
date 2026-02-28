# Chapter 2.2 — Optimization Need

> **Unit 2: Pattern Matching** | [Course Home](../README.md)

---

## 📋 Chapter Overview

Why can't we just use brute force? This chapter quantifies the cost, identifies
the redundancy, and motivates the need for KMP, Rabin-Karp, and Z-Algorithm
through concrete examples and real-world scale arguments.

---

## 1. The Redundancy Problem

```
  T = "ABABABABC"
  P = "ABABC"

  Brute Force Attempt at i=0:
  ──────────────────────────
  T: A B A B A B A B C
  P: A B A B C
               ▲
     Matched 4 chars, mismatch at j=4  (A ≠ C)

  What we KNOW after this mismatch:
  ─────────────────────────────────
  T[0] = A,  T[1] = B,  T[2] = A,  T[3] = B     (verified!)

  What brute force does NEXT (i=1):
  ─────────────────────────────────
  T: . B A B A ...
  P:   A B A B C
       ▲
     B ≠ A → fail after 1 comparison.

  But we already KNEW T[1] = B ≠ A!

  What brute force does at i=2:
  ─────────────────────────────
  T: . . A B A B A B C
  P:     A B A B C
         ✓ ✓ ✓ ✓ ▲ mismatch again

  We re-compared T[2]='A' and T[3]='B' — but we already knew this
  from the i=0 attempt! Those characters are part of the pattern prefix.
```

### The Wasted Work

```
  ┌───────────────────────────────────────────────────────┐
  │                                                       │
  │  Every time we shift by just 1 and restart from j=0,  │
  │  we're re-examining characters we've already seen.    │
  │                                                       │
  │  Number of redundant comparisons can reach O(n × m)   │
  │  in the worst case.                                   │
  │                                                       │
  └───────────────────────────────────────────────────────┘
```

---

## 2. Scale of the Problem

Consider real-world scenarios:

```
  ┌──────────────────────────┬─────────┬──────────┬──────────────┐
  │ Application              │ n       │ m        │ n × m        │
  ├──────────────────────────┼─────────┼──────────┼──────────────┤
  │ Find word in a book      │ 500K    │ 10       │ 5 million    │
  │ Search in DNA sequence   │ 3B      │ 1000     │ 3 trillion!  │
  │ Log file search          │ 10M     │ 100      │ 1 billion    │
  │ Code search in repo      │ 50M     │ 50       │ 2.5 billion  │
  │ Plagiarism detection     │ 100K    │ varies   │ varies       │
  └──────────────────────────┴─────────┴──────────┴──────────────┘

  At O(n × m), DNA search would need 3 × 10¹² operations!
  At O(n + m),  it's just 3 × 10⁹ + 10³ ≈ 3 × 10⁹.

  That's a 1000× speedup.
```

---

## 3. What Optimization Looks Like

### The Key Insight: Use Pattern Structure

```
  Pattern P = "ABABC"

  The prefix "AB" is also a suffix of "ABAB" (the matched portion).
  This means when we mismatch at j=4, we can:
    - Skip to where the prefix "AB" aligns with the suffix "AB"
    - Resume comparison from j=2, not j=0

  BEFORE (brute force):
  ──────────────────────
  T: A B A B A B A B C
  P: A B A B C               mismatch at j=4
     shift by 1 → restart j=0

  AFTER (KMP):
  ─────────────
  T: A B A B A B A B C
  P:     A B A B C            shift by 2 → resume at j=2
             ▲
             Already matched!

  We never re-examine T[0] and T[1]!
```

---

## 4. Comparison of Approaches

```
  ┌────────────────────┬───────────┬──────────────────────────────┐
  │ Algorithm          │  Time     │  Key Idea                    │
  ├────────────────────┼───────────┼──────────────────────────────┤
  │ Brute Force        │ O(n × m) │  Try every position          │
  │ KMP                │ O(n + m) │  Failure function (LPS)      │
  │ Rabin-Karp         │ O(n + m) │  Rolling hash comparison     │
  │                    │ avg      │  (O(n×m) worst with hash     │
  │                    │          │   collisions)                │
  │ Z-Algorithm        │ O(n + m) │  Z-array prefix matching     │
  │ Boyer-Moore        │ O(n/m)   │  Bad char + good suffix      │
  │                    │ best     │  (skips large chunks)        │
  │ Aho-Corasick       │ O(n + Σm)│  Multi-pattern via trie      │
  └────────────────────┴───────────┴──────────────────────────────┘
```

### Visual Time Comparison

```
  n = 1,000,000    m = 1,000

  Brute Force: ███████████████████████████████████████  10⁹ ops
  KMP:         █                                       10⁶ ops
  Rabin-Karp:  █                                       10⁶ ops (avg)
  Z-Algorithm: █                                       10⁶ ops

  That's a 1000× improvement!
```

---

## 5. Types of Optimization

### 5.1 Avoid Re-scanning (KMP, Z)

```
  Use preprocessed information about P to determine the next
  safe comparison point after a mismatch.

  ┌─────────────────────────────────┐
  │  Preprocessing: O(m) time      │
  │  Matching:      O(n) time      │
  │  Total:         O(n + m) time  │
  └─────────────────────────────────┘
```

### 5.2 Compare Fingerprints, Not Characters (Rabin-Karp)

```
  Instead of comparing m characters:
    Compute a hash of the window → compare hash with pattern hash.
    Hash comparison is O(1).
    
  Rolling hash: update hash in O(1) as window slides.

  Hash match → verify (O(m) only when hash matches).
  Hash mismatch → definitely no match (O(1) to skip).
```

### 5.3 Skip Ahead (Boyer-Moore)

```
  Read pattern right-to-left.
  When mismatch at T[i+j]:
    - Bad character rule: skip based on T[i+j]'s position in P
    - Good suffix rule: skip based on matched suffix structure

  Best case: O(n/m) — can skip entire pattern lengths!
```

---

## 6. When Each Algorithm Shines

```
  ┌─────────────────────────────┬──────────────────────────────┐
  │  Scenario                   │  Best Algorithm              │
  ├─────────────────────────────┼──────────────────────────────┤
  │  Single pattern, guaranteed │  KMP (deterministic O(n+m)) │
  │  O(n+m)                     │                              │
  │                             │                              │
  │  Multiple patterns at once  │  Rabin-Karp or Aho-Corasick │
  │                             │                              │
  │  Large alphabet, long skips │  Boyer-Moore                 │
  │                             │                              │
  │  Simple implementation      │  Z-Algorithm                 │
  │                             │                              │
  │  Substring comparison       │  String Hashing              │
  │  (many queries)             │                              │
  └─────────────────────────────┴──────────────────────────────┘
```

---

## 7. The Preprocessing Philosophy

```
  ┌──────────────────────────────────────────────┐
  │                                              │
  │  💡 CORE PRINCIPLE                           │
  │                                              │
  │  Spend O(m) time upfront to analyze the      │
  │  pattern's internal structure (prefixes,     │
  │  suffixes, repetitions), so that during      │
  │  matching, you never waste a comparison.     │
  │                                              │
  │  Preprocessing + Matching = O(m) + O(n)      │
  │                           = O(n + m)         │
  │                                              │
  │  vs. Brute Force = O(n × m)                  │
  │                                              │
  └──────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Key Point |
|---------|-----------|
| Brute force weakness | Re-examines already-matched characters |
| Redundant comparisons | Can reach O(n × m) on structured inputs |
| Real-world scale | DNA: 3 billion chars — O(n×m) is infeasible |
| KMP/Z optimization | Preprocess pattern once → never backtrack in text |
| Rabin-Karp optimization | Compare hashes in O(1) instead of strings in O(m) |
| Preprocessing | O(m) upfront cost, amortized across all comparisons |

---

## ❓ Quick Revision Questions

1. **Why does brute force reach O(n × m) worst case?**
   <details><summary>Answer</summary>When the pattern almost matches at every position (e.g., T = "AAA...A", P = "AA...AB"), each window requires m-1 comparisons before failing.</details>

2. **What does "never backtrack in text" mean for KMP?**
   <details><summary>Answer</summary>The text pointer only moves forward — we never re-read a character of T we've already examined. This guarantees O(n) matching time.</details>

3. **Why is Rabin-Karp O(n+m) on average but O(n×m) in the worst case?**
   <details><summary>Answer</summary>If hash collisions (spurious hits) occur frequently, every window requires O(m) verification. Average case assumes few collisions.</details>

4. **What is the preprocessing step in KMP?**
   <details><summary>Answer</summary>Building the Failure Function (LPS array) for the pattern in O(m) time.</details>

5. **In what scenario can Boyer-Moore achieve O(n/m) time?**
   <details><summary>Answer</summary>When the last character of the pattern frequently mismatches and doesn't appear in the text, the algorithm can skip m positions at a time.</details>

---

| [⬅️ Previous: Brute Force Approach](01-brute-force-approach.md) | [Next: Pattern Matching Applications ➡️](03-pattern-matching-applications.md) |
|:---|---:|
