# 5.5 Time Complexity Analysis

## Unit 5: Hash Table Operations

---

## Summary Table: All Operations

| Operation | Best | Average | Worst | Amortized |
|-----------|:---:|:---:|:---:|:---:|
| **Insert** | $O(1)$ | $O(1)$ | $O(n)$ | $O(1)$ |
| **Search** | $O(1)$ | $O(1)$ | $O(n)$ | — |
| **Delete** | $O(1)$ | $O(1)$ | $O(n)$ | — |
| **Resize** | $O(n)$ | $O(n)$ | $O(n)$ | $O(1)$ per insert |

---

## When Each Case Occurs

```
╔══════════════════════════════════════════════════════════════╗
║  BEST CASE O(1):                                           ║
║  • No collision — key maps directly to an empty slot       ║
║  • First slot checked is the answer                        ║
║                                                              ║
║  AVERAGE CASE O(1):                                         ║
║  • Constant load factor α maintained                       ║
║  • Good hash function distributes uniformly                ║
║  • Few collisions — 1-3 probes on average                  ║
║                                                              ║
║  WORST CASE O(n):                                          ║
║  • All keys hash to same slot                              ║
║  • Chaining: one chain of length n                         ║
║  • Open addressing: probe entire table                     ║
║  • Caused by: bad hash function, adversarial input,        ║
║    or extremely unlucky distribution                       ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Detailed Analysis by Method

### Chaining

Under **Simple Uniform Hashing Assumption (SUHA)**:

```
  Expected chain length = α = n/m

  UNSUCCESSFUL SEARCH:
    Examine full chain → O(1 + α) comparisons
    The "1" is for computing the hash

  SUCCESSFUL SEARCH:
    On average, examine half the chain → O(1 + α/2)

  INSERT (at head):
    O(1) + search to check for duplicate → O(1 + α)
    Without duplicate check: O(1)

  DELETE:
    O(1) to find + O(1) to remove → O(1 + α) total
```

### Open Addressing (Double Hashing)

Under **Uniform Hashing Assumption**:

$$\text{Unsuccessful search: } \frac{1}{1-\alpha} \text{ expected probes}$$

$$\text{Successful search: } \frac{1}{\alpha}\ln\frac{1}{1-\alpha} \text{ expected probes}$$

```
Numerical values:

  α     │ Unsuccessful │ Successful
  ──────┼──────────────┼───────────
  0.10  │     1.11     │   1.05
  0.25  │     1.33     │   1.15
  0.50  │     2.00     │   1.39
  0.75  │     4.00     │   1.85
  0.90  │    10.00     │   2.56
  0.99  │   100.00     │   4.65
```

---

## Hash Table vs Other Data Structures

```
╔══════════════════════════════════════════════════════════════════╗
║         COMPLEXITY COMPARISON                                   ║
║                                                                  ║
║  Operation    │ Hash Table │ BST (balanced) │ Sorted Array      ║
║  ─────────────┼────────────┼────────────────┼──────────────     ║
║  Search       │ O(1) avg   │ O(log n)       │ O(log n)         ║
║  Insert       │ O(1) amort │ O(log n)       │ O(n)             ║
║  Delete       │ O(1) avg   │ O(log n)       │ O(n)             ║
║  Min/Max      │ O(n)       │ O(log n)       │ O(1)             ║
║  Ordered iter │ O(n log n) │ O(n)           │ O(n)             ║
║  Range query  │ O(n)       │ O(log n + k)   │ O(log n + k)    ║
║  Space        │ O(n)       │ O(n)           │ O(n)             ║
║                                                                  ║
║  Hash tables WIN at point lookups                               ║
║  BSTs WIN when ordering matters                                 ║
║  Sorted arrays WIN for static data with range queries           ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## The O(1) Caveat

```
╔══════════════════════════════════════════════════════════════╗
║  Hash table O(1) is EXPECTED / AMORTIZED, not guaranteed!  ║
║                                                              ║
║  Requirements for O(1):                                     ║
║  ✓ Good hash function (uniform distribution)               ║
║  ✓ Bounded load factor (resize when needed)                ║
║  ✓ Non-adversarial input (or universal hashing)            ║
║                                                              ║
║  IF any requirement breaks:                                  ║
║  ✗ Bad hash → O(n) chains/probes                           ║
║  ✗ No resize → α grows → O(n) degradation                 ║
║  ✗ Adversarial input → forced O(n) collisions             ║
║                                                              ║
║  In competitive programming / interviews:                   ║
║  Usually assume O(1) but mention the caveats!              ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Hash Function Cost

The $O(1)$ assumes hash computation is $O(1)$. For variable-length keys:

| Key Type | Hash Cost | Total Lookup |
|----------|:---------:|:---:|
| Integer | $O(1)$ | $O(1)$ |
| Fixed-length string | $O(L)$ where $L$ = length | $O(L)$ |
| Variable-length string | $O(L)$ | $O(L)$ |
| Object (composite key) | $O(\text{fields})$ | $O(\text{fields})$ |

```
Example: String keys of average length 100
  Hash computation: O(100) per lookup
  Still O(1) in terms of n (number of entries)
  But string comparison in chains also takes O(L)
  
  Total: O(L × (1 + α)) for chaining
         O(L / (1 - α)) for open addressing
```

---

## Amortized Analysis Deep Dive

```
╔══════════════════════════════════════════════════════════════╗
║          ACCOUNTING METHOD FOR RESIZE                       ║
║                                                              ║
║  Imagine each insert pays $3:                               ║
║  • $1 for the insert itself                                 ║
║  • $2 saved as "credit" on the entry                       ║
║                                                              ║
║  When resize doubles from m to 2m:                          ║
║  • Must rehash m/2 entries (those inserted since last       ║
║    resize)                                                   ║
║  • Each of those entries has $2 credit saved                ║
║  • Cost of rehashing = m/2 entries × $1 = $m/2             ║
║  • Credits available = m/2 entries × $2 = $m               ║
║  • Credits ≥ Cost → resize is "free"!                      ║
║                                                              ║
║  Therefore: amortized cost per insert = $3 = O(1)          ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Performance in Practice

Real-world performance differs from theory due to:

```
Factor              Impact
──────────────      ──────────────────────────────
Cache locality      Linear probing >> chaining
Memory allocation   Chaining needs malloc per node
Hash quality        Poor hash destroys all guarantees
Key comparison      String keys: equality check is O(L)
Memory overhead     Chaining: ~24 bytes/entry extra
Branch prediction   Predictable access >> random jumps
```

### Practical Throughput (operations/second, approximate)

```
  Millions of operations per second:

  100M ┤  ████
       │  ████
  80M  ┤  ████  ████
       │  ████  ████
  60M  ┤  ████  ████  ████
       │  ████  ████  ████
  40M  ┤  ████  ████  ████  ████
       │  ████  ████  ████  ████
  20M  ┤  ████  ████  ████  ████  ████
       │  ████  ████  ████  ████  ████
   0   └──────────────────────────────
        Robin  Linear Python  C++   Java
        Hood   Probe  dict    umap  HashMap
```

---

## Quick Revision Question

**Q: A hash table stores $n = 10{,}000$ entries with a load factor $\alpha = 0.75$. Calculate: (a) the table capacity, (b) expected probes for an unsuccessful search using chaining, (c) expected probes for an unsuccessful search using double hashing.**

<details>
<summary>Click to reveal answer</summary>

**(a) Table capacity:**
$\alpha = n/m \implies m = n/\alpha = 10{,}000 / 0.75 \approx 13{,}334$

**(b) Chaining — unsuccessful search:**
Expected probes $= 1 + \alpha = 1 + 0.75 = 1.75$

This means on average, we compute the hash (1 operation) and then traverse 0.75 nodes in the chain before hitting null.

**(c) Double hashing — unsuccessful search:**
Expected probes $= \frac{1}{1-\alpha} = \frac{1}{1-0.75} = \frac{1}{0.25} = 4.0$

Double hashing requires about 4 probes on average, vs 1.75 for chaining at the same load factor. However, double hashing uses less memory (no pointer overhead), which may make it faster in practice due to cache effects.

</details>

---

| [← Previous: Resize & Rehash](04-resize-and-rehash.md) | [Next: Space Complexity →](06-space-complexity.md) |
|:---|---:|
