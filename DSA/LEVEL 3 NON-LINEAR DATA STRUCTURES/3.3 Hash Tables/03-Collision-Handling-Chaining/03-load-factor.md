# 3.3 Load Factor

## Unit 3: Collision Handling - Chaining

---

## Definition

The **load factor** ($\alpha$) measures how full a hash table is. It's the ratio of stored elements to table size.

$$\alpha = \frac{n}{m}$$

Where: $n$ = number of elements stored, $m$ = number of buckets (table size)

```
╔═══════════════════════════════════════════════════════════╗
║                    LOAD FACTOR                           ║
║                                                          ║
║   n = 6 elements, m = 8 buckets                         ║
║   α = 6/8 = 0.75                                        ║
║                                                          ║
║   ┌───┬───┬───┬───┬───┬───┬───┬───┐                    ║
║   │ █ │   │ █ │ █ │   │ █ │ █ │ █ │                    ║
║   └───┴───┴───┴───┴───┴───┴───┴───┘                    ║
║     0   1   2   3   4   5   6   7                       ║
║                                                          ║
║   6 out of 8 buckets occupied = 75% full                ║
║                                                          ║
║   With chaining, α CAN exceed 1.0:                      ║
║   n = 12, m = 8 → α = 1.5                              ║
║   (chains hold multiple elements per bucket)            ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Load Factor and Performance

```
╔════════════════════════════════════════════════════════════════╗
║          LOAD FACTOR vs PERFORMANCE (CHAINING)                ║
║                                                               ║
║   α = 0.25   Very fast lookups, lots of wasted space         ║
║   │████                                                       ║
║   │                                                           ║
║   α = 0.50   Good balance of speed and space                 ║
║   │████████                                                   ║
║   │                                                           ║
║   α = 0.75   Default threshold (Java HashMap)   ◄── SWEET   ║
║   │████████████                                      SPOT    ║
║   │                                                           ║
║   α = 1.00   Average 1 element per bucket                    ║
║   │████████████████                                           ║
║   │                                                           ║
║   α = 2.00   Average 2 elements per chain                    ║
║   │████████████████████████████████                           ║
║   │                                                           ║
║   α = 5.00   Heavy chains, slow lookups                      ║
║   │████████████████████████████████████████████████████████   ║
║                                                               ║
║   Expected chain length = α                                  ║
║   Expected search time = O(1 + α)                            ║
╚════════════════════════════════════════════════════════════════╝
```

---

## Impact on Operations

| Load Factor (α) | Avg Chain Length | Successful Search | Unsuccessful Search | Status |
|:---:|:---:|:---:|:---:|:---|
| 0.25 | 0.25 | ~1.1 probes | ~1.3 probes | Under-utilized |
| 0.50 | 0.50 | ~1.25 probes | ~1.5 probes | Good |
| **0.75** | **0.75** | **~1.4 probes** | **~1.75 probes** | **Optimal** |
| 1.00 | 1.00 | ~1.5 probes | ~2.0 probes | Acceptable |
| 2.00 | 2.00 | ~2.0 probes | ~3.0 probes | Slow |
| 10.0 | 10.0 | ~5.5 probes | ~11 probes | Very slow |

---

## When to Resize

```
╔═══════════════════════════════════════════════════════════════╗
║                RESIZE TRIGGER                                ║
║                                                              ║
║   Threshold: typically α > 0.75                             ║
║                                                              ║
║   BEFORE resize (α = 0.75, n=6, m=8):                      ║
║   ┌──┬──┬──┬──┬──┬──┬──┬──┐                                ║
║   │██│  │██│██│  │██│██│██│                                 ║
║   └──┴──┴──┴──┴──┴──┴──┴──┘                                ║
║                                                              ║
║   Insert one more → α = 7/8 = 0.875 > 0.75                 ║
║   TRIGGER RESIZE!                                            ║
║                                                              ║
║   AFTER resize (double capacity, rehash all):               ║
║   ┌──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┬──┐       ║
║   │  │██│  │  │██│  │██│  │  │██│  │██│  │  │██│  │       ║
║   └──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┴──┘       ║
║   α = 7/16 = 0.4375   ◄── Much lower, faster lookups       ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Default Load Factors in Languages

| Language | Structure | Default α | Resize Factor |
|----------|-----------|:---------:|:------------:|
| Java | `HashMap` | 0.75 | 2x |
| Python | `dict` | 0.667 | 2x |
| C++ | `unordered_map` | 1.0 | ~2x |
| C# | `Dictionary` | 1.0 | ~2x |
| Go | `map` | 6.5 (per bucket) | 2x |

---

## Trace: Load Factor During Insertions

```
Table size m = 4, threshold α = 0.75

Insert 1: n=1, α = 1/4 = 0.25  ✓
Insert 2: n=2, α = 2/4 = 0.50  ✓
Insert 3: n=3, α = 3/4 = 0.75  ✓ (at threshold)
Insert 4: n=4, α = 4/4 = 1.00  ✗ RESIZE!
  → New m = 8, rehash all 4 elements
  → α = 4/8 = 0.50  ✓

Insert 5: n=5, α = 5/8 = 0.625 ✓
Insert 6: n=6, α = 6/8 = 0.75  ✓ (at threshold)
Insert 7: n=7, α = 7/8 = 0.875 ✗ RESIZE!
  → New m = 16, rehash all 7 elements
  → α = 7/16 = 0.4375  ✓

Pattern: resize at n = 3, 6, 12, 24, 48, ...
```

---

## Quick Revision Question

**Q: A hash table has 100 buckets and 60 elements. What is the load factor? If the resize threshold is 0.75, how many more elements can be inserted before a resize is triggered?**

<details>
<summary>Click to reveal answer</summary>

**Load factor**: $\alpha = n/m = 60/100 = 0.6$

**Elements until resize**: The threshold is $0.75 \times 100 = 75$ elements. Currently at 60, so $75 - 60 = 15$ more elements can be inserted before resize is triggered.

After resize: table grows to 200 buckets, all 75 elements are rehashed, and new $\alpha = 75/200 = 0.375$.

</details>

---

| [← Previous: Linked List Implementation](02-linked-list-implementation.md) | [Next: Average Case Analysis →](04-average-case-analysis.md) |
|:---|---:|
