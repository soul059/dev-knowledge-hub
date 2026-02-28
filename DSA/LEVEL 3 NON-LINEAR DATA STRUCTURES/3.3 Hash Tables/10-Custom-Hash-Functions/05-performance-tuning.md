# 10.5 Performance Tuning

## Unit 10: Custom Hash Functions

---

## The Performance Levers

```
╔══════════════════════════════════════════════════════════════╗
║         HASH TABLE PERFORMANCE DEPENDS ON:                  ║
║                                                              ║
║  1. LOAD FACTOR (α = n/m)                                   ║
║     → Controls space-time tradeoff                          ║
║     → Triggers resizing                                     ║
║                                                              ║
║  2. HASH FUNCTION QUALITY                                   ║
║     → Determines collision rate                             ║
║     → Affects bucket distribution                           ║
║                                                              ║
║  3. COLLISION RESOLUTION                                    ║
║     → Chaining vs Open Addressing                           ║
║     → Determines degradation behavior                       ║
║                                                              ║
║  4. INITIAL CAPACITY                                        ║
║     → Avoids expensive early resizes                        ║
║                                                              ║
║  5. MEMORY LAYOUT                                           ║
║     → Cache-friendliness matters for real performance       ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Load Factor Tuning

### The Tradeoff

```
Low α (e.g., 0.25)         High α (e.g., 0.9)
────────────────────        ────────────────────
✓ Fewer collisions          ✗ More collisions
✓ Faster lookups            ✗ Slower lookups
✗ Wastes memory             ✓ Memory efficient
✗ Poor cache utilization    ✓ Better data density

           SWEET SPOT: α ≈ 0.5 to 0.75
```

### Expected Probes by Load Factor

For **separate chaining** (successful search):

$$E[\text{probes}] = 1 + \frac{\alpha}{2}$$

For **linear probing** (successful search):

$$E[\text{probes}] = \frac{1}{2}\left(1 + \frac{1}{1 - \alpha}\right)$$

```
┌───────────┬─────────────┬─────────────────┐
│ Load (α)  │  Chaining   │  Linear Probing │
├───────────┼─────────────┼─────────────────┤
│   0.25    │    1.13     │      1.17       │
│   0.50    │    1.25     │      1.50       │
│   0.75    │    1.38     │      2.50       │
│   0.90    │    1.45     │      5.50       │
│   0.95    │    1.48     │     10.50       │
│   0.99    │    1.50     │     50.50       │
└───────────┴─────────────┴─────────────────┘

╔═══════════════════════════════════════════════╗
║  Key Insight:                                 ║
║  Chaining degrades LINEARLY with load         ║
║  Open addressing degrades EXPLOSIVELY         ║
║  past α ≈ 0.7                                 ║
╚═══════════════════════════════════════════════╝
```

---

## Initial Capacity Planning

```
╔══════════════════════════════════════════════════════════════╗
║  PROBLEM: Default initial capacity is small (e.g., 16)     ║
║  If you're inserting 1 million entries, the table resizes   ║
║  many times: 16 → 32 → 64 → ... → 2,097,152              ║
║                                                              ║
║  Each resize: O(n) rehashing + memory allocation!           ║
║                                                              ║
║  RESIZE COUNT: log₂(1,000,000 / 16) ≈ 16 resizes!          ║
║                                                              ║
║  SOLUTION: Pre-size the table                               ║
║                                                              ║
║  // Java: account for load factor                           ║
║  int capacity = (int)(expectedSize / loadFactor) + 1;       ║
║  Map<K,V> map = new HashMap<>(capacity, loadFactor);        ║
║                                                              ║
║  // For 1M entries with α = 0.75:                           ║
║  capacity = (int)(1_000_000 / 0.75) + 1 = 1_333_334       ║
║  → Rounds up to next power of 2 = 2,097,152                ║
║  → ZERO resizes during insertion!                           ║
╚══════════════════════════════════════════════════════════════╝
```

### Time Saved by Pre-Sizing

```
Without pre-sizing (1M insertions):
  Resize at 12, 24, 48, 96, ..., 786432
  Total rehash operations: ~2M extra element moves
  Time: ~1.5x slower than optimal

With pre-sizing:
  Zero resizes
  Time: Optimal

           ┌──────────────────────────────────┐
           │ Time vs Insertions               │
     Time  │                        /         │
           │                      /  No pre-  │
           │                    /    sizing    │
           │                  /               │
           │              . /                 │
           │           ../  ─── Pre-sized     │
           │        .../                      │
           │   ..../                          │
           └──────────────────────────────────┘
                    Insertions →
```

---

## Hash Function Speed vs Quality

```
FUNCTION hashSimple(key):       // FAST, lower quality
    return key * 2654435761     // Knuth's multiplicative hash
    // ~1 ns per hash

FUNCTION hashGood(key):         // MODERATE, good quality
    h = key
    h ^= (h >>> 16)
    h *= 0x45d9f3b
    h ^= (h >>> 16)
    return h
    // ~3 ns per hash

FUNCTION hashCrypto(key):       // SLOW, cryptographic quality
    return SHA256(key)
    // ~100 ns per hash — OVERKILL for hash tables!
```

```
╔══════════════════════════════════════════════════════════════╗
║  GUIDELINE:                                                 ║
║                                                              ║
║  Hash computation time << Expected probe time               ║
║                                                              ║
║  If your hash function takes 10ns but saves 5 probes        ║
║  (each ~5ns for cache miss), net saving = 15ns              ║
║                                                              ║
║  If your hash function takes 100ns but only saves 2 probes  ║
║  net LOSS = 90ns → simpler hash would be faster             ║
║                                                              ║
║  RULE: Use the SIMPLEST hash that gives good distribution   ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Cache-Friendly Hash Tables

### The Memory Hierarchy Problem

```
    CPU Register   ~0.3 ns
    L1 Cache       ~1 ns
    L2 Cache       ~4 ns
    L3 Cache       ~10 ns
    Main RAM       ~100 ns     ← Chaining pointer chase!
    SSD            ~100,000 ns
```

### Chaining vs Open Addressing — Cache Effects

```
CHAINING (pointer-based):
┌───┐    ┌───────┐    ┌───────┐    ┌───────┐
│ •─┼───→│ key,v │───→│ key,v │───→│ key,v │
├───┤    └───────┘    └───────┘    └───────┘
│ •─┼───→ ...         ↑ Each node allocated separately
├───┤                 ↑ Random memory locations
│ / │                 ↑ CACHE MISS on every hop!
└───┘

OPEN ADDRESSING (linear probing):
┌───────┬───────┬───────┬───────┬───────┬───────┐
│ key,v │ key,v │ key,v │ empty │ key,v │ empty │
└───────┴───────┴───────┴───────┴───────┴───────┘
  ↑ Contiguous memory → cache line loads ~16 entries
  ↑ Sequential scan → hardware prefetcher helps
  ↑ MUCH fewer cache misses!
```

```
╔══════════════════════════════════════════════════════════════╗
║  BENCHMARK REALITY (1M lookups, α = 0.5):                  ║
║                                                              ║
║  Chaining:         ~150 ns/lookup  (pointer chasing)        ║
║  Linear Probing:   ~40  ns/lookup  (cache-friendly)         ║
║  Robin Hood:       ~45  ns/lookup  (cache-friendly + fair)  ║
║  Cuckoo:           ~35  ns/lookup  (exactly 2 lookups)      ║
║                                                              ║
║  Linear probing can be 3-4x FASTER than chaining at same   ║
║  load factor, purely from cache effects!                    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Resize Strategy

```
╔══════════════════════════════════════════════════════════════╗
║  STANDARD: Double when α exceeds threshold                  ║
║    Resize factor: 2x                                        ║
║    Amortized O(1) insert                                    ║
║    ⚠ Spike: one insert takes O(n) to rehash everything     ║
║                                                              ║
║  INCREMENTAL REHASHING (Redis approach):                    ║
║    Keep OLD and NEW table simultaneously                    ║
║    Each operation migrates a few entries                     ║
║    ✓ No latency spikes                                      ║
║    ✗ More complex, temporarily uses 2x memory              ║
║                                                              ║
║  SHRINKING:                                                 ║
║    Halve when α drops below threshold (e.g., 0.125)         ║
║    Some implementations never shrink (Java HashMap)         ║
║    Manual: new HashMap<>(old) to right-size                 ║
╚══════════════════════════════════════════════════════════════╝
```

### Incremental Rehashing Trace

```
State: Old table (4 buckets), New table (8 buckets)

Operation 1: get("apple")
  1. Check new table → not found
  2. Check old table → found in bucket 2
  3. Migrate bucket 2 entries to new table
  4. Return value

Operation 2: put("banana", 5)
  1. Insert into new table
  2. Migrate 1 more bucket from old → new

... after enough operations ...

Old table fully migrated → discard old table
Only new table remains
```

---

## Platform-Specific Defaults

| Platform | Default Capacity | Default Load Factor | Resize Factor | Collision Strategy |
|----------|:---:|:---:|:---:|---|
| Java HashMap | 16 | 0.75 | 2x | Chaining → TreeMap at 8 |
| Python dict | 8 | 0.67 | 2x (or 4x if small) | Open addressing (probing) |
| C++ unordered_map | impl-defined | 1.0 | ~2x | Chaining |
| Go map | impl-defined | 6.5 (per bucket) | 2x | Chaining (8 per bucket) |
| Rust HashMap | 0 (lazy) | ~0.875 | 2x | Swiss Table (SIMD probing) |

---

## Performance Tuning Checklist

```
□ Pre-size table if you know approximate entry count
□ Choose load factor based on collision strategy:
    Chaining: 0.75 (default is usually fine)
    Open addressing: 0.5 - 0.7
□ Profile hash function: quality vs speed tradeoff
□ Apply bit mixing if using power-of-2 table sizes
□ Consider open addressing for cache-sensitive workloads
□ Monitor collision rate: > 30% of buckets with 2+ entries = problem
□ Avoid resizing during latency-critical operations
□ For known static data: consider perfect hashing (zero collisions)
□ For concurrent access: use concurrent hash map variants
```

---

## Quick Revision Question

**Q: You have a Java `HashMap` that will store exactly 1,000 entries. The default load factor is 0.75 and default initial capacity is 16. How many resize operations will occur if you don't pre-size? What initial capacity should you set to avoid all resizes?**

<details>
<summary>Click to reveal answer</summary>

**Resize count without pre-sizing:**

The table resizes when `size > capacity × loadFactor`:

```
Capacity  │ Threshold (×0.75) │ Resize?
──────────┼───────────────────┼────────
   16     │       12          │ Yes, when 13th added
   32     │       24          │ Yes, when 25th added
   64     │       48          │ Yes, when 49th added
  128     │       96          │ Yes, when 97th added
  256     │      192          │ Yes, when 193rd added
  512     │      384          │ Yes, when 385th added
 1024     │      768          │ Yes, when 769th added
 2048     │     1536          │ No (1000 < 1536)
```

**7 resize operations** before the table is large enough.

**Optimal initial capacity:**
```java
int expectedSize = 1000;
float loadFactor = 0.75f;
int capacity = (int)(expectedSize / loadFactor) + 1;  // 1334
// Java rounds up to next power of 2 → 2048

Map<K,V> map = new HashMap<>(1334, 0.75f);
// Or simply: new HashMap<>(2048)
```

This eliminates all 7 resizes, avoiding ~2,000 extra element rehash operations.

</details>

---

| [← Collision Minimization](04-collision-minimization.md) | [Next: Common Mistakes →](06-common-mistakes.md) |
|:---|---:|
