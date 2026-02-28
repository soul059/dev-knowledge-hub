# Chapter 6: Probabilistic Data Structures

[← Previous: Shuffle Algorithm](05-shuffle-algorithm.md) | [Back to README](../README.md) | [Next: Unit 8 - Point Representation →](../08-Geometry-Basics/01-point-representation.md)

---

## Overview

Probabilistic data structures trade small error probability for massive space savings. They are essential for big data, streaming, and systems where exact answers are too expensive.

---

## 6.1 Bloom Filter

```
  "Is element x in the set?" — with small false positive rate.
  
  ┌──────────────────────────────────────────────────────────┐
  │  Structure: Bit array of size m, k hash functions        │
  │                                                          │
  │  INSERT(x):                                              │
  │    For each hash function hᵢ (i = 1..k):                │
  │      Set bit[hᵢ(x) % m] = 1                            │
  │                                                          │
  │  QUERY(x):                                              │
  │    For each hash function hᵢ:                            │
  │      If bit[hᵢ(x) % m] == 0: return DEFINITELY NOT     │
  │    Return PROBABLY YES                                   │
  └──────────────────────────────────────────────────────────┘

  Visualization (m=10, k=3):
  
  Insert "cat":  h₁=2, h₂=5, h₃=8
  [0,0,1,0,0,1,0,0,1,0]
  
  Insert "dog":  h₁=1, h₂=5, h₃=9
  [0,1,1,0,0,1,0,0,1,1]
  
  Query "bird": h₁=2, h₂=4, h₃=9
  bit[2]=1 ✓, bit[4]=0 ✗ → DEFINITELY NOT in set
  
  Query "fox": h₁=1, h₂=5, h₃=8
  bit[1]=1 ✓, bit[5]=1 ✓, bit[8]=1 ✓ → PROBABLY YES (FALSE POSITIVE!)
```

### False Positive Rate

```
  P(false positive) ≈ (1 - e^(-kn/m))ᵏ
  
  Optimal k = (m/n) × ln 2 ≈ 0.693 × m/n
  
  At optimal k: P(fp) ≈ (0.6185)^(m/n)
  
  ┌─────────────────────────────────────────────────┐
  │  m/n (bits per element) │  P(false positive)    │
  ├─────────────────────────┼───────────────────────┤
  │         8               │  ~2.2%                │
  │        10               │  ~0.8%                │
  │        15               │  ~0.1%                │
  │        20               │  ~0.01%               │
  └─────────────────────────┴───────────────────────┘
  
  ★ No false negatives! If Bloom says "not in set", it's certain.
  ★ Cannot delete elements (use Counting Bloom Filter for that).
```

---

## 6.2 Count-Min Sketch

```
  Estimate frequency of element x in a stream.
  
  Structure: d × w array of counters (d hash functions, w width)
  
  UPDATE(x):
    FOR i = 1 TO d:
      counters[i][hᵢ(x) % w] += 1
  
  QUERY(x):
    RETURN min over i of counters[i][hᵢ(x) % w]

  ┌──────────────────────────────────────────────────┐
  │  Properties:                                      │
  │  • Never underestimates (always ≥ true count)    │
  │  • Overestimates by at most ε×n with P ≥ 1-δ    │
  │  • w = ⌈e/ε⌉, d = ⌈ln(1/δ)⌉                    │
  │  • Space: O((1/ε) × log(1/δ))                   │
  └──────────────────────────────────────────────────┘
  
  Example: Stream = [a,b,a,c,a,b,a]
  True freq: a=4, b=2, c=1
  
  CMS might report: a=4, b=3, c=1 (b overcounted due to collision)
```

---

## 6.3 HyperLogLog (Cardinality Estimation)

```
  Count the number of DISTINCT elements in a stream.
  
  Key insight: If you hash elements uniformly, the maximum
  number of leading zeros in the hash tells you about the
  cardinality n.
  
  E[max leading zeros] ≈ log₂(n)
  
  HyperLogLog improvement:
  • Split into 2^b buckets using first b bits of hash
  • Track max leading zeros in each bucket
  • Harmonic mean of estimates
  
  ┌──────────────────────────────────────────────────┐
  │  Space: O(m) where m = 2^b (typically 2^14)     │
  │  Error: σ ≈ 1.04 / √m                           │
  │                                                   │
  │  m = 2^14 = 16384 registers × 6 bits = ~12 KB   │
  │  Error ≈ 0.81% — for counting billions!          │
  └──────────────────────────────────────────────────┘
  
  Used by Redis, BigQuery, Presto for COUNT(DISTINCT).
```

---

## 6.4 Skip List (Randomized BST Alternative)

```
  A probabilistic alternative to balanced BSTs.
  
  Level 3: HEAD ──────────────────────────→ 9 ──→ NIL
  Level 2: HEAD ──→ 3 ──────────────────→ 9 ──→ NIL
  Level 1: HEAD ──→ 3 ──→ 5 ──→ 7 ──→ 9 ──→ NIL
  Level 0: HEAD ──→ 1 ──→ 3 ──→ 5 ──→ 7 ──→ 9 ──→ NIL
  
  INSERT: Flip coins to determine level (P=1/2 to go up).
  SEARCH: Start top-left, go right if possible, else drop down.
  
  ┌──────────────────────────────────────────────────┐
  │  Expected height: O(log n)                       │
  │  Search/Insert/Delete: O(log n) expected         │
  │  Space: O(n) expected                            │
  │  Simpler to implement than AVL/Red-Black trees!  │
  └──────────────────────────────────────────────────┘
```

---

## 6.5 Comparison Table

```
  ┌──────────────────┬──────────────┬───────────┬──────────────────┐
  │  Structure       │  Query       │  Space    │  Error Type      │
  ├──────────────────┼──────────────┼───────────┼──────────────────┤
  │  Bloom Filter    │  Membership  │  O(n)bits │  False positives │
  │  Count-Min Sketch│  Frequency   │  O(1/ε)  │  Overcounting    │
  │  HyperLogLog     │  Cardinality │  O(1/ε²) │  ±ε relative     │
  │  Skip List       │  BST ops     │  O(n)    │  None (Las Vegas)│
  └──────────────────┴──────────────┴───────────┴──────────────────┘
```

---

## 6.6 When to Use What

```
  ┌──────────────────────────────────────────────────────────┐
  │  "Is x in the set?"                                     │
  │  → Bloom Filter (or Cuckoo Filter for deletions)        │
  │                                                          │
  │  "How many times has x appeared?"                       │
  │  → Count-Min Sketch                                     │
  │                                                          │
  │  "How many distinct elements?"                          │
  │  → HyperLogLog                                          │
  │                                                          │
  │  "Need sorted set with O(log n) ops?"                   │
  │  → Skip List (simpler than balanced BST)                │
  └──────────────────────────────────────────────────────────┘
```

---

## Summary Table

| Structure | Purpose | Space | Error |
|-----------|---------|-------|-------|
| Bloom Filter | Set membership | ~10 bits/element | FP ≈ 1% |
| Count-Min Sketch | Frequency | O(1/ε × log(1/δ)) | Overcount ≤ εn |
| HyperLogLog | Distinct count | ~12 KB | ~0.8% |
| Skip List | Ordered ops | O(n) | None (randomized time) |

---

## Quick Revision Questions

1. **Can** a Bloom Filter produce false negatives? Why or why not?
2. **What is** the optimal number of hash functions for a Bloom Filter?
3. **How does** Count-Min Sketch guarantee it never undercounts?
4. **Why does** HyperLogLog use a harmonic mean?
5. **Compare** Skip List vs Red-Black Tree in implementation complexity.
6. **Design** a Bloom Filter for 10⁶ elements with 1% false positive rate.

---

[← Previous: Shuffle Algorithm](05-shuffle-algorithm.md) | [Back to README](../README.md) | [Next: Unit 8 - Point Representation →](../08-Geometry-Basics/01-point-representation.md)
