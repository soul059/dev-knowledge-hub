# 4.6 Comparison of Methods

## Unit 4: Collision Handling - Open Addressing

---

## All Collision Resolution Methods at a Glance

```
╔══════════════════════════════════════════════════════════════════════╗
║                  COLLISION RESOLUTION TAXONOMY                      ║
║                                                                      ║
║  Collision Handling                                                  ║
║  ├── Separate Chaining                                               ║
║  │   ├── Linked List chains                                          ║
║  │   ├── BST chains (Java 8+)                                       ║
║  │   └── Dynamic array chains                                       ║
║  │                                                                   ║
║  └── Open Addressing                                                 ║
║      ├── Linear Probing:    h(k,i) = (h(k) + i) mod m              ║
║      ├── Quadratic Probing: h(k,i) = (h(k) + c₁i + c₂i²) mod m    ║
║      └── Double Hashing:    h(k,i) = (h₁(k) + i·h₂(k)) mod m      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## Open Addressing Methods Comparison

| Feature | Linear Probing | Quadratic Probing | Double Hashing |
|---------|:---:|:---:|:---:|
| **Formula** | $h(k) + i$ | $h(k) + i^2$ | $h_1(k) + i \cdot h_2(k)$ |
| **Primary clustering** | Yes ✗ | No ✓ | No ✓ |
| **Secondary clustering** | Yes ✗ | Yes ✗ | No ✓ |
| **Cache performance** | Excellent | Good | Poor |
| **Probe coverage** | All slots | Needs prime $m$ | Needs coprime $h_2$ |
| **Hash functions needed** | 1 | 1 | 2 |
| **Implementation complexity** | Simple | Moderate | Complex |
| **Best for** | Low $\alpha$, cache-sensitive | Medium $\alpha$ | High $\alpha$ |

---

## Performance Comparison: Expected Probes

### Unsuccessful Search

| $\alpha$ | Linear | Quadratic | Double Hashing | Chaining |
|:---:|:---:|:---:|:---:|:---:|
| 0.25 | 1.39 | 1.33 | 1.33 | 1.25 |
| 0.50 | 2.50 | 2.19 | 2.00 | 1.50 |
| 0.75 | 8.50 | 4.64 | 4.00 | 1.75 |
| 0.90 | 50.50 | 11.40 | 10.00 | 1.90 |

### Successful Search

| $\alpha$ | Linear | Quadratic | Double Hashing | Chaining |
|:---:|:---:|:---:|:---:|:---:|
| 0.25 | 1.17 | 1.15 | 1.15 | 1.125 |
| 0.50 | 1.50 | 1.44 | 1.39 | 1.25 |
| 0.75 | 2.50 | 2.01 | 1.85 | 1.375 |
| 0.90 | 5.50 | 3.52 | 2.56 | 1.45 |

---

## Chaining vs Open Addressing

```
╔══════════════════════════════════════════════════════════════════╗
║          CHAINING  vs  OPEN ADDRESSING                          ║
║                                                                  ║
║  CHAINING:                     OPEN ADDRESSING:                  ║
║  ┌───┐                         ┌─────────────────┐              ║
║  │ 0 │──►[A]──►[D]            │ A │ D │ B │ C │  ← all inside ║
║  │ 1 │──►null                  │   │   │   │   │               ║
║  │ 2 │──►[B]                   └─────────────────┘              ║
║  │ 3 │──►[C]                                                    ║
║  └───┘                                                           ║
║  α can be > 1                  α must be ≤ 1                    ║
║  Extra pointers per node       No extra pointers                 ║
║  Scattered in memory           Contiguous array                  ║
║  Simple deletion               Tombstones needed                 ║
╚══════════════════════════════════════════════════════════════════╝
```

| Feature | Chaining | Open Addressing |
|---------|:---:|:---:|
| **Load factor limit** | Unlimited ($\alpha > 1$ ok) | $\alpha \leq 1$ (practical $< 0.7$) |
| **Memory overhead** | Pointers per node | No extra (just array) |
| **Cache performance** | Poor (pointer chasing) | Good (array sequential) |
| **Deletion** | Simple (remove node) | Complex (tombstones) |
| **Worst case** | $O(n)$ per chain | $O(n)$ probe entire table |
| **Implementation** | Simple | Moderate |
| **Resize frequency** | Less often | More often (keep $\alpha$ low) |
| **Small data** | Pointer overhead significant | More memory-efficient |
| **Large data** | Memory flexible | Need contiguous block |

---

## When to Use Which Method

```
╔══════════════════════════════════════════════════════════════════╗
║                  DECISION FLOWCHART                              ║
║                                                                  ║
║  Need hash table collision handling?                             ║
║  │                                                               ║
║  ├─ Unknown or variable load?                                    ║
║  │  └─ YES → CHAINING (handles α > 1 gracefully)               ║
║  │                                                               ║
║  ├─ Frequent deletions?                                          ║
║  │  └─ YES → CHAINING (no tombstone overhead)                   ║
║  │                                                               ║
║  ├─ Memory-constrained / cache-critical?                         ║
║  │  └─ YES → LINEAR PROBING (best cache, keep α < 0.5)         ║
║  │                                                               ║
║  ├─ Need guaranteed slot coverage with simple probe?             ║
║  │  └─ YES → QUADRATIC PROBING (prime table size)               ║
║  │                                                               ║
║  ├─ High load factor AND many collisions?                        ║
║  │  └─ YES → DOUBLE HASHING (best probe distribution)           ║
║  │                                                               ║
║  └─ General purpose / don't know?                                ║
║     └─ Default: CHAINING (most forgiving, least fragile)        ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## Language Implementation Choices

| Language | Default Implementation | Method |
|----------|----------------------|--------|
| Java `HashMap` | Chaining | Linked list → BST at 8 nodes |
| Python `dict` | Open addressing | Custom probing (perturbation) |
| C++ `unordered_map` | Chaining | Linked list |
| Go `map` | Chaining | Buckets of 8 |
| Rust `HashMap` | Open addressing | Robin Hood hashing |
| C# `Dictionary` | Chaining | Linked list |
| Ruby `Hash` | Open addressing | Open addressing (since 2.4) |
| Swift `Dictionary` | Open addressing | Linear probing variant |

```
Trend: Newer languages prefer open addressing
       for better cache performance.
       
       Older languages keep chaining for
       simpler implementation and flexibility.
```

---

## Memory Usage Comparison

For $n$ keys, table capacity $m$:

```
CHAINING (linked list):
  Array:  m × pointer_size         = 8m bytes
  Nodes:  n × (key + value + next) = n × 24 bytes (typical)
  Total ≈ 8m + 24n

OPEN ADDRESSING:
  Array:  m × (key + value + flag)  = m × 17 bytes (typical)
  Total ≈ 17m

Break-even point (at α = 0.75, m ≈ 1.33n):
  Chaining: 8(1.33n) + 24n = 34.64n bytes
  Open Addr: 17(1.33n)     = 22.61n bytes  ← 35% less memory

At α = 0.5 (m = 2n):
  Chaining: 8(2n) + 24n = 40n bytes
  Open Addr: 17(2n)      = 34n bytes  ← 15% less memory
```

---

## Summary: Strengths and Weaknesses

```
╔══════════════════════════════════════════════════════════════════╗
║                                                                  ║
║  LINEAR PROBING                                                  ║
║  ✓ Best cache performance        ✗ Primary clustering           ║
║  ✓ Simplest implementation       ✗ Bad at high load factors     ║
║  ✓ Low memory overhead           ✗ Needs α < 0.5 for speed     ║
║                                                                  ║
║  QUADRATIC PROBING                                               ║
║  ✓ No primary clustering         ✗ Secondary clustering         ║
║  ✓ Good cache (nearby probes)    ✗ May not visit all slots      ║
║  ✓ Moderate complexity           ✗ Needs prime table size        ║
║                                                                  ║
║  DOUBLE HASHING                                                  ║
║  ✓ No clustering at all          ✗ Poor cache performance       ║
║  ✓ Best probe distribution       ✗ Needs two hash functions     ║
║  ✓ Works well at higher α        ✗ Most complex to implement    ║
║                                                                  ║
║  SEPARATE CHAINING                                               ║
║  ✓ Simple deletion               ✗ Pointer overhead per node    ║
║  ✓ Handles α > 1                 ✗ Poor cache (scattered nodes) ║
║  ✓ Most forgiving overall        ✗ Memory not contiguous        ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: A system processes a known, fixed dataset of 1 million entries with no deletions. The system is memory-limited and performance-critical. Which collision handling method would you recommend, and what load factor would you target?**

<details>
<summary>Click to reveal answer</summary>

**Recommended: Linear probing with $\alpha \approx 0.5$** (table size ≈ 2 million).

**Reasoning**:
1. **No deletions** → eliminates linear probing's biggest weakness (tombstone management)
2. **Fixed dataset** → can size the table perfectly upfront, no dynamic resizing needed
3. **Memory-limited** → open addressing uses 35% less memory than chaining (no pointers)
4. **Performance-critical** → linear probing has the best cache locality, making it faster in practice than double hashing despite more theoretical probes
5. **$\alpha = 0.5$** → at this load factor, linear probing averages only 1.5 probes for successful search and 2.5 for unsuccessful — very fast

With 2M slots × (8 bytes key + 8 bytes value) = 32 MB, which is very reasonable for 1M entries.

</details>

---

| [← Previous: Clustering Problems](05-clustering-problems.md) | [Next: Unit 5 — Hash Table Operations →](../05-Hash-Table-Operations/01-insert-operation.md) |
|:---|---:|
