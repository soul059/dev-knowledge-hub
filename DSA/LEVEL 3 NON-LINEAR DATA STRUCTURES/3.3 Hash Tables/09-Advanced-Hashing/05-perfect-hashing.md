# 9.5 Perfect Hashing

## Unit 9: Advanced Hashing

---

## What is Perfect Hashing?

A hash function that maps $n$ keys to $n$ distinct positions with **zero collisions**. Lookups are $O(1)$ worst-case with no probing needed.

```
╔══════════════════════════════════════════════════════════════╗
║  PERFECT HASHING: No collisions (for a KNOWN set of keys)  ║
║                                                              ║
║  ┌──────────────────┬────────────────────────────────┐     ║
║  │ Type             │ Description                     │     ║
║  ├──────────────────┼────────────────────────────────┤     ║
║  │ Perfect          │ Maps n keys → m slots (m ≥ n)  │     ║
║  │                  │ No collisions                   │     ║
║  ├──────────────────┼────────────────────────────────┤     ║
║  │ Minimal Perfect  │ Maps n keys → exactly n slots  │     ║
║  │ (MPHF)           │ No collisions, no wasted space │     ║
║  └──────────────────┴────────────────────────────────┘     ║
║                                                              ║
║  Limitation: Works only for STATIC sets (keys known ahead) ║
║  Dynamic insertions require rebuilding the hash function   ║
╚══════════════════════════════════════════════════════════════╝
```

---

## FKS Perfect Hashing (Two-Level Scheme)

The **Fredman-Komlós-Szemerédi** scheme uses two levels of hashing:

```
╔══════════════════════════════════════════════════════════════╗
║  LEVEL 1: Universal hash → n buckets (some collisions)     ║
║                                                              ║
║  Keys: {5, 12, 18, 23, 35, 42, 51, 67}                    ║
║                                                              ║
║  h₁(k) = ((ak + b) mod p) mod 8                            ║
║                                                              ║
║  Bucket 0: {12, 42}                                         ║
║  Bucket 1: {}                                               ║
║  Bucket 2: {18}                                             ║
║  Bucket 3: {5, 23, 51}                                      ║
║  Bucket 4: {35}                                             ║
║  Bucket 5: {67}                                             ║
║  Bucket 6: {}                                               ║
║  Bucket 7: {}                                               ║
║                                                              ║
║  LEVEL 2: For each bucket with bᵢ items,                   ║
║           create a perfect hash of size bᵢ²                ║
║                                                              ║
║  Bucket 0 (2 items): secondary table of size 4             ║
║    h₂(k) = ((a₀k + b₀) mod p) mod 4                       ║
║    12 → slot 1, 42 → slot 3  (no collision!)               ║
║                                                              ║
║  Bucket 3 (3 items): secondary table of size 9             ║
║    h₃(k) = ((a₃k + b₃) mod p) mod 9                       ║
║    5 → slot 2, 23 → slot 7, 51 → slot 0  (no collision!)  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Why $b_i^2$ Size Works

```
╔══════════════════════════════════════════════════════════════╗
║  Birthday Paradox (reversed):                               ║
║                                                              ║
║  If bᵢ items hash into bᵢ² slots using a random hash:     ║
║  Expected collisions = bᵢ(bᵢ-1) / (2·bᵢ²) < 1/2         ║
║                                                              ║
║  By Markov's inequality:                                    ║
║  Pr[any collision] < 1/2                                    ║
║                                                              ║
║  → Try random hash functions until one has NO collisions   ║
║  → Expected number of tries: 2 (constant!)                 ║
║                                                              ║
║  TOTAL SPACE: Σ bᵢ² = O(n) (if level-1 uses universal h) ║
║  Proof: E[Σ bᵢ²] = n + 2·(collisions at level 1)         ║
║         With universal hashing, collisions ≤ n             ║
║         → E[Σ bᵢ²] ≤ 3n = O(n)                           ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Lookup Process

```
FUNCTION lookup(key):
    // Level 1: Find bucket
    bucket = h₁(key) % m

    // Level 2: Find position within bucket's secondary table
    table = secondaryTables[bucket]
    IF table is null: RETURN NOT_FOUND

    pos = h₂_bucket(key) % table.size
    IF table[pos] == key:
        RETURN table[pos].value
    ELSE:
        RETURN NOT_FOUND

// ALWAYS O(1): one hash at level 1 + one hash at level 2
// No probing, no chaining, no loops
```

---

## Construction Trace

```
Keys: {10, 22, 37, 40, 52, 70}
n = 6, choose m = 6

LEVEL 1: h₁(k) = k mod 6
  Bucket 0: {70}      → b₀=1
  Bucket 1: {37}      → b₁=1
  Bucket 2: {52}      → b₂=1
  Bucket 3: {}        → b₃=0
  Bucket 4: {10, 22, 40}  → b₄=3
  Bucket 5: {}        → b₅=0

  Σ bᵢ² = 1+1+1+0+9+0 = 12 = O(n) ✓

LEVEL 2 for Bucket 4 (keys {10, 22, 40}):
  Secondary table size = 3² = 9

  Try h₂(k) = k mod 9:
    10 → 1, 22 → 4, 40 → 4  ← COLLISION! Try again.

  Try h₂(k) = (2k+5) mod 9:
    10 → 25 mod 9 = 7
    22 → 49 mod 9 = 4
    40 → 85 mod 9 = 4  ← COLLISION! Try again.

  Try h₂(k) = (3k+1) mod 9:
    10 → 31 mod 9 = 4
    22 → 67 mod 9 = 4  ← COLLISION! Try again.

  Try h₂(k) = (k+3) mod 9:
    10 → 13 mod 9 = 4
    22 → 25 mod 9 = 7
    40 → 43 mod 9 = 7  ← COLLISION!

  Try h₂(k) = (5k+2) mod 9:
    10 → 52 mod 9 = 7
    22 → 112 mod 9 = 4
    40 → 202 mod 9 = 4  ← COLLISION!

  Try h₂(k) = (4k) mod 9:
    10 → 40 mod 9 = 4
    22 → 88 mod 9 = 7
    40 → 160 mod 9 = 7  ← COLLISION!

  Try h₂(k) = (k*7+3) mod 9:
    10 → 73 mod 9 = 1
    22 → 157 mod 9 = 4
    40 → 283 mod 9 = 4  ← COLLISION!

  Try h₂(k) = (k*2+1) mod 9:
    10 → 21 mod 9 = 3
    22 → 45 mod 9 = 0
    40 → 81 mod 9 = 0  ← COLLISION!

  Try h₂(k) = (k+1) mod 9:
    10 → 11 mod 9 = 2
    22 → 23 mod 9 = 5
    40 → 41 mod 9 = 5  ← COLLISION!

  Try h₂(k) = (k*3+2) mod 9:
    10 → 32 mod 9 = 5
    22 → 68 mod 9 = 5  ← COLLISION!

  Try h₂(k) = (k*11) mod 9:
    10 → 110 mod 9 = 2
    22 → 242 mod 9 = 8
    40 → 440 mod 9 = 8  ← COLLISION!

  In practice, with proper universal hash family h(k)=((a·k+b) mod p) mod 9
  where p is prime > max key, success is guaranteed within ~2 tries on average.
  
  h₂(k) = ((7k+3) mod 43) mod 9:
    10 → 73 mod 43 = 30, 30 mod 9 = 3
    22 → 157 mod 43 = 28, 28 mod 9 = 1
    40 → 283 mod 43 = 26, 26 mod 9 = 8
    NO COLLISION! ✓

LOOKUP(22):
  Level 1: 22 mod 6 = 4 → bucket 4
  Level 2: ((7·22+3) mod 43) mod 9 = 1
  secondary[4][1] = 22 ✓ → FOUND in O(1)
```

---

## Complexity Summary

| Operation | Time |
|-----------|:---:|
| Lookup | $O(1)$ worst-case |
| Construction | $O(n)$ expected |
| Space | $O(n)$ |
| Insert/Delete | Reconstruct: $O(n)$ |

---

## When to Use Perfect Hashing

```
╔══════════════════════════════════════════════════════════════╗
║  ✓ USE WHEN:                                                ║
║    • Key set is static (known in advance)                  ║
║    • Lookups are frequent, updates are rare                ║
║    • O(1) worst-case lookup is required                    ║
║    • Examples: keyword tables, compiler reserved words,    ║
║      country codes, command dispatch tables                ║
║                                                              ║
║  ✗ DON'T USE WHEN:                                         ║
║    • Keys change frequently (dynamic sets)                 ║
║    • Key set is unknown at build time                      ║
║    • Simple hash table is fast enough                      ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: In the FKS two-level perfect hashing scheme, why do we use tables of size $b_i^2$ at the second level instead of just $b_i$? What would happen with linear-sized tables?**

<details>
<summary>Click to reveal answer</summary>

With a table of size $b_i$ for $b_i$ items (linear-sized), collisions are **highly likely**. By the birthday paradox, the expected number of collisions with a random hash function is:

$$E[\text{collisions}] = \frac{b_i(b_i - 1)}{2b_i} = \frac{b_i - 1}{2}$$

For even $b_i = 10$ items, we'd expect ~4.5 collisions — making collision-free hashing very unlikely. We'd need many retries, and there's no guarantee of termination.

With **quadratic-sized** tables ($b_i^2$ slots for $b_i$ items):

$$E[\text{collisions}] = \frac{b_i(b_i - 1)}{2b_i^2} = \frac{b_i - 1}{2b_i} < \frac{1}{2}$$

The expected number of collisions is less than $\frac{1}{2}$ regardless of $b_i$. By Markov's inequality, the probability of having **any** collision is $< \frac{1}{2}$. This means:
- On average, 2 random hash functions suffice
- Construction is efficient — $O(b_i)$ per bucket

The total space across all buckets is still $O(n)$ because:
$$\sum b_i^2 \leq 2n + n = O(n) \text{ (using universal hashing at level 1)}$$

</details>

---

| [← Cuckoo Hashing](04-cuckoo-hashing.md) | [Next: Locality-Sensitive Hashing →](06-locality-sensitive-hashing.md) |
|:---|---:|
