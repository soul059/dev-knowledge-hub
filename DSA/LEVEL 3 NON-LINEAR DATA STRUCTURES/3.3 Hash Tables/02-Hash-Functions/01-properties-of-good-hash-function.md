# 2.1 Properties of a Good Hash Function

## Unit 2: Hash Functions

---

## Introduction

A hash function's quality directly determines a hash table's performance. A poor hash function causes clustering and collisions, degrading O(1) operations to O(n). This topic examines what makes a hash function "good."

---

## Essential Properties

```
╔══════════════════════════════════════════════════════════════════╗
║            PROPERTIES OF A GOOD HASH FUNCTION                   ║
║                                                                 ║
║   1. DETERMINISTIC                                              ║
║      Same input ──► Always same output                          ║
║      h("cat") = 5 today, tomorrow, always                      ║
║                                                                 ║
║   2. UNIFORM DISTRIBUTION                                       ║
║      Keys spread evenly across all slots                        ║
║      ┌──┬──┬──┬──┬──┬──┬──┬──┐                                ║
║      │██│██│██│██│██│██│██│██│  ◄── GOOD: even                 ║
║      └──┴──┴──┴──┴──┴──┴──┴──┘                                ║
║      ┌──┬──┬──┬──┬──┬──┬──┬──┐                                ║
║      │██████│  │  │█ │  │  │  │  ◄── BAD: clustered            ║
║      └──┴──┴──┴──┴──┴──┴──┴──┘                                ║
║                                                                 ║
║   3. EFFICIENCY                                                 ║
║      Computing h(k) must be O(1) or O(len(k))                  ║
║      A slow hash defeats the purpose of hashing                ║
║                                                                 ║
║   4. AVALANCHE EFFECT                                           ║
║      Small input change ──► Large output change                 ║
║      h("cat") = 42,  h("bat") = 789  (very different)         ║
║                                                                 ║
║   5. MINIMIZE COLLISIONS                                        ║
║      Different keys should map to different indices             ║
║      (impossible to eliminate, but minimize)                    ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## Property Details

### 1. Deterministic

```
GOOD (Deterministic):          BAD (Non-deterministic):
h("hello") → 5                h("hello") → 5
h("hello") → 5   ✓ Same!     h("hello") → 3   ✗ Different!
h("hello") → 5   ✓ Same!     h("hello") → 7   ✗ Different!

If the hash changes, you can never find your data again!
```

### 2. Uniform Distribution

```
10 keys hashed into 5 buckets:

GOOD distribution (≈2 per bucket):
Bucket 0: ██  (2 items)
Bucket 1: ██  (2 items)
Bucket 2: ██  (2 items)
Bucket 3: ██  (2 items)
Bucket 4: ██  (2 items)
Max chain length: 2 → O(1) lookups

BAD distribution (clustered):
Bucket 0: ████████  (8 items)
Bucket 1: █         (1 item)
Bucket 2:           (0 items)
Bucket 3: █         (1 item)
Bucket 4:           (0 items)
Max chain length: 8 → O(n) lookups!
```

### 3. Avalanche Effect

```
╔════════════════════════════════════════════════════════╗
║              AVALANCHE EFFECT                         ║
║                                                       ║
║   Input change: 1 bit                                 ║
║   Output change: ~50% of bits should flip             ║
║                                                       ║
║   GOOD avalanche:                                     ║
║   h("cat")  = 01101001 10110100                       ║
║   h("bat")  = 10010110 01001011  (12 of 16 bits flip)║
║                ^^^^^^^^ ^^^^^^^^                      ║
║                                                       ║
║   BAD avalanche:                                      ║
║   h("cat")  = 01101001 10110100                       ║
║   h("bat")  = 01101000 10110100  (only 1 bit flips)  ║
║                       ^                               ║
║   Similar inputs → similar outputs → clustering!     ║
╚════════════════════════════════════════════════════════╝
```

---

## Testing Hash Function Quality

```
FUNCTION test_hash_quality(hash_func, keys, table_size):
    buckets = new Array(table_size, 0)
    
    FOR each key in keys:
        index = hash_func(key) MOD table_size
        buckets[index] = buckets[index] + 1
    
    // Calculate statistics
    expected = len(keys) / table_size
    variance = 0
    FOR i = 0 TO table_size - 1:
        variance += (buckets[i] - expected)^2
    variance = variance / table_size
    
    // Chi-squared test
    chi_squared = variance * table_size / expected
    
    PRINT "Expected per bucket:", expected
    PRINT "Variance:", variance
    PRINT "Chi-squared:", chi_squared
    // chi_squared ≈ table_size means good distribution
```

---

## Common Pitfalls

| Pitfall | Example | Problem |
|---------|---------|---------|
| Using only part of key | `h(str) = str[0]` | Only 26 possible outputs for strings |
| Ignoring key structure | Sum of digits | Anagrams collide |
| Table size = even number | `% 100` | Doesn't use all bits |
| Non-prime table size | `% 10` | Patterns in data create patterns in hashes |

---

## Quick Revision Question

**Q: Explain the avalanche effect and why it's important for hash function quality. Give an example of what happens without it.**

<details>
<summary>Click to reveal answer</summary>

The **avalanche effect** means that a small change in the input (even a single bit) should cause a significant change in the output (roughly 50% of output bits flip). 

**Why it matters**: Without the avalanche effect, similar keys produce similar hash values, causing them to cluster in the same buckets. For example, if hashing student IDs "S001", "S002", "S003" produces consecutive hashes 1, 2, 3, all sequential IDs land in adjacent buckets, creating uneven load.

**Without avalanche**: Consider `h(x) = x % 100`. Keys 100, 200, 300, 400 all hash to 0. Keys 101, 201, 301 all hash to 1. Data with patterns creates hash patterns → clustering → poor performance.

</details>

---

| [← Unit 1: Hash Table Applications](../01-Hashing-Fundamentals/06-hash-table-applications.md) | [Next: Division Method →](02-division-method.md) |
|:---|---:|
