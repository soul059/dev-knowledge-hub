# Chapter 4.3 — Spurious Hits & Collision Handling

> **Unit 4: Rabin-Karp Algorithm** | [Course Home](../README.md)

---

## 📋 Chapter Overview

A **spurious hit** occurs when the hash of a text window matches the pattern's
hash but the actual strings are different. Understanding and dealing with
spurious hits is crucial for correctness and performance.

---

## 1. What Is a Spurious Hit?

```
  Spurious Hit = Hash Match BUT String Mismatch

  ┌─────────────────────────────────────────────┐
  │  Pattern:  "abc"  →  hash = 28              │
  │  Window:   "xyz"  →  hash = 28   (same!)    │
  │                                              │
  │  Hash says MATCH, but "abc" ≠ "xyz"         │
  │  This is a SPURIOUS HIT (false positive)    │
  └─────────────────────────────────────────────┘

  Why it happens:
  ────────────────
  Hash maps an infinite set of strings →  finite set [0, q-1]
  By Pigeonhole Principle, collisions are INEVITABLE.
```

---

## 2. Visual Example

```
  Text:     A B C D E A B F G
  Pattern:  A B F      hash(P) = 42

  Slide window of length 3:

  Window    Hash    Match?    Verify?    Result
  ──────    ────    ──────    ───────    ──────
  A B C      35      No        —          —
  B C D      48      No        —          —
  C D E      61      No        —          —
  D E A      42      Yes!      Check...   "DEA" ≠ "ABF" → SPURIOUS HIT ❌
  E A B      53      No        —          —
  A B F      42      Yes!      Check...   "ABF" = "ABF" → TRUE MATCH ✅
  B F G      55      No        —          —

  1 spurious hit + 1 true match = 2 verifications total
```

---

## 3. Why Spurious Hits Happen

```
  Cause 1: Modular Arithmetic
  ───────────────────────────
  We compute hash mod q.  q values compress a much larger space.

  Number of distinct strings of length m: |Σ|^m
  Number of hash slots: q

  If m = 10, Σ = 26:  26¹⁰ ≈ 1.4 × 10¹⁴ strings → 10⁹ slots
  Each slot maps ~140,000 strings on average!

  Cause 2: Small Modulus
  ──────────────────────
  Smaller q → more collisions.
  q = 13:  only 13 possible hash values → very frequent hits.
  q = 10⁹+7:  1 billion slots → rare collisions.
```

---

## 4. Handling Spurious Hits — Verification

```
  Algorithm:
  ──────────
  1. Compute hash of pattern: hp
  2. Compute hash of first window: ht
  3. For each position i:
       if ht == hp:
           VERIFY: compare P[0..m-1] with T[i..i+m-1] character by character
           if all characters match:
               report match at position i
       Roll hash to next window

  ┌──────────────────────────────────────────────────┐
  │  Hash filters out most non-matches in O(1).      │
  │  Verification costs O(m) but happens rarely      │
  │  (only on hash match).                           │
  └──────────────────────────────────────────────────┘
```

### Pseudocode

```
function RabinKarp(T, P, d, q):
    n = len(T), m = len(P)
    hp = hash(P)
    ht = hash(T[0..m-1])
    h = d^(m-1) mod q          // precomputed

    for i = 0 to n - m:
        if ht == hp:                       // ← hash match
            if T[i..i+m-1] == P[0..m-1]:   // ← O(m) verification
                print "Match at", i
            else:
                // SPURIOUS HIT — hash matched but strings differ
                spurious_count += 1

        if i < n - m:
            ht = rolling_update(ht, T[i], T[i+m], d, h, q)
```

---

## 5. Impact on Time Complexity

```
  Best / Average Case:
  ────────────────────
  - Hash matches are rare → few verifications
  - Time: O(n + m)

  Worst Case:
  ───────────
  - Every window gives a hash match (all spurious or real)
  - Each match triggers O(m) verification
  - Time: O(n × m)

  When does worst case occur?
  ──────────────────────────
  T = "AAAAAAAAAA"     (all same character)
  P = "AAAA"
  Every window has the same hash AND same content.
  → n - m + 1 verifications, each O(m) = O(nm)

  ┌──────────────────────────────────────────────┐
  │  Worst case is same as brute force!          │
  │  But average case is MUCH better: O(n + m)   │
  └──────────────────────────────────────────────┘
```

---

## 6. Reducing Spurious Hits

### Strategy 1: Use a Large Prime

```
  P(collision per window) ≈ 1/q

  q = 13       →  P ≈ 7.7%    (many spurious hits)
  q = 101      →  P ≈ 1%
  q = 10⁹+7   →  P ≈ 10⁻⁹   (practically zero)
```

### Strategy 2: Double Hashing

```
  Use TWO independent hash functions:

  Hash 1:  d₁ = 31,  q₁ = 10⁹ + 7
  Hash 2:  d₂ = 37,  q₂ = 10⁹ + 9

  Report match only if BOTH hashes agree.

  P(spurious hit) = 1/q₁ × 1/q₂ ≈ 10⁻¹⁸

  ┌──────────────────────────────────────┐
  │  Double hashing virtually eliminates │
  │  spurious hits at 2× hash cost.     │
  │  Still O(n) overall.                │
  └──────────────────────────────────────┘
```

### Strategy 3: Random Base

```
  Choose d randomly at each run.
  Makes it impossible for adversarial inputs to always cause collisions.
  (Used in competitive programming for anti-hack)
```

---

## 7. Counting Spurious Hits — Example

```
  Given: T = "2359023141526739921", P = "31415"
  d = 10, q = 13

  hash(P) = 31415 mod 13 = 7

  Windows of length 5:
  ┌─────────┬──────────────────┬─────────┬────────────┐
  │ Position│ Window           │ Hash    │ Type       │
  ├─────────┼──────────────────┼─────────┼────────────┤
  │    0    │ 23590            │ 8       │ No match   │
  │    1    │ 35902            │ 9       │ No match   │
  │    2    │ 59023            │ 9       │ No match   │
  │    3    │ 90231            │ 3       │ No match   │
  │    4    │ 02314            │ 11      │ No match   │
  │    5    │ 23141            │ 7  ✓    │ SPURIOUS   │ ← hash match, strings differ
  │    6    │ 31415            │ 7  ✓    │ TRUE MATCH │ ← hash match, strings equal
  │    7    │ 14152            │ 9       │ No match   │
  │   ...   │ ...              │ ...     │ ...        │
  └─────────┴──────────────────┴─────────┴────────────┘

  Spurious Hits: 1   (position 5)
  True Matches:  1   (position 6)

  With larger q, the spurious hit at position 5 likely disappears.
```

---

## 8. Spurious Hit Rate Analysis

```
  Expected number of spurious hits for random text:

  E[spurious hits] = (n - m + 1 - k) / q

  where k = number of true matches

  For n = 10⁶, m = 10, k = 5, q = 10⁹+7:
  E ≈ (10⁶ - 10 + 1 - 5) / 10⁹ ≈ 0.001

  Less than 1 spurious hit expected on average! ✓
```

---

## 📝 Summary Table

| Concept | Details |
|---------|---------|
| Spurious hit | Hash match + string mismatch |
| Cause | Modular arithmetic compresses hash space |
| Frequency | ≈ (n-m)/q per search |
| Handling | Character-by-character verification on hash match |
| Best-case overhead | 0 spurious hits → pure O(n+m) |
| Worst case | All windows match hash → O(nm) |
| Mitigation 1 | Large prime q (10⁹+7) |
| Mitigation 2 | Double hashing (two hash functions) |
| Mitigation 3 | Randomized base d |

---

## ❓ Quick Revision Questions

1. **What causes a spurious hit?**
   <details><summary>Answer</summary>Two different strings map to the same hash value due to the finite range of the modular hash function (pigeonhole principle).</details>

2. **Does Rabin-Karp ever miss a true match?**
   <details><summary>Answer</summary>No. If two strings are equal, their hashes are always equal. Rabin-Karp has no false negatives — only false positives (spurious hits).</details>

3. **What is the worst-case time complexity and when does it occur?**
   <details><summary>Answer</summary>O(nm). Occurs when every window's hash matches the pattern's hash (e.g., T = "AAAA...A", P = "AAA"), requiring O(m) verification at each of the n-m+1 positions.</details>

4. **How does double hashing help?**
   <details><summary>Answer</summary>Both independent hash functions must produce matching values simultaneously for a spurious hit. The combined probability drops to 1/(q₁×q₂), making spurious hits virtually impossible.</details>

5. **Which is more important for reducing spurious hits — increasing d or increasing q?**
   <details><summary>Answer</summary>Increasing q. The base d affects distribution quality, but the modulus q directly determines the number of available hash slots and thus collision probability ≈ 1/q.</details>

---

| [⬅️ Previous: Hash Function Design](02-hash-function-design.md) | [Next: Multiple Pattern Matching ➡️](04-multiple-pattern-matching.md) |
|:---|---:|
