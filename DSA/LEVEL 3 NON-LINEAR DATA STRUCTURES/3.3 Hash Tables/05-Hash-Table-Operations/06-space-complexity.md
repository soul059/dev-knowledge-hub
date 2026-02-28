# 5.6 Space Complexity

## Unit 5: Hash Table Operations

---

## Space Components

```
╔══════════════════════════════════════════════════════════════╗
║              HASH TABLE MEMORY USAGE                        ║
║                                                              ║
║  Total Space = Table Array + Stored Data + Overhead         ║
║                                                              ║
║  ┌──────────────┐                                            ║
║  │ Table Array   │ m slots × slot_size                      ║
║  │ (always       │ Allocated even if empty                  ║
║  │  allocated)   │                                           ║
║  ├──────────────┤                                            ║
║  │ Stored Data   │ n entries × entry_size                   ║
║  │ (keys+values) │ Only for occupied slots/nodes            ║
║  ├──────────────┤                                            ║
║  │ Overhead      │ Pointers, metadata, padding              ║
║  │ (per-entry)   │ Depends on implementation                ║
║  └──────────────┘                                            ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Chaining: Memory Layout

```
Table array: m pointers (8 bytes each on 64-bit)
Each node:   key + value + next pointer + object header

Example: m = 1000, n = 750 (α = 0.75)
         Keys: 8 bytes (long), Values: 8 bytes (long)

┌────────────────────────────────────────────────────────┐
│ Component          │ Per Unit  │ Count │ Total         │
├────────────────────┼──────────┼───────┼───────────────┤
│ Table array        │ 8 bytes  │ 1000  │  8,000 bytes  │
│ Node key           │ 8 bytes  │  750  │  6,000 bytes  │
│ Node value         │ 8 bytes  │  750  │  6,000 bytes  │
│ Node next pointer  │ 8 bytes  │  750  │  6,000 bytes  │
│ Node object header │ 16 bytes │  750  │ 12,000 bytes  │
├────────────────────┼──────────┼───────┼───────────────┤
│ TOTAL              │          │       │ 38,000 bytes  │
│ Per entry          │          │       │ ~50.7 bytes   │
│ Vs raw data (16B)  │          │       │ 3.2× overhead │
└────────────────────────────────────────────────────────┘
```

---

## Open Addressing: Memory Layout

```
Table array: m entries stored inline (no pointers)
Each slot: key + value + status flag

Example: m = 1000, n = 750 (α = 0.75)
         Keys: 8 bytes (long), Values: 8 bytes (long)

┌────────────────────────────────────────────────────────┐
│ Component          │ Per Unit  │ Count │ Total         │
├────────────────────┼──────────┼───────┼───────────────┤
│ Slot key           │ 8 bytes  │ 1000  │  8,000 bytes  │
│ Slot value         │ 8 bytes  │ 1000  │  8,000 bytes  │
│ Slot status flag   │ 1 byte   │ 1000  │  1,000 bytes  │
│ Padding            │ ~7 bytes │ 1000  │  7,000 bytes  │
├────────────────────┼──────────┼───────┼───────────────┤
│ TOTAL              │          │       │ 24,000 bytes  │
│ Per entry (actual) │          │       │ 32 bytes      │
│ Vs raw data (16B)  │          │       │ 2.0× overhead │
└────────────────────────────────────────────────────────┘
```

---

## Side-by-Side Comparison

| Metric | Chaining | Open Addressing |
|--------|:---:|:---:|
| **Total for 750 entries** | 38 KB | 24 KB |
| **Per entry** | 50.7 bytes | 32 bytes |
| **Overhead ratio** | 3.2× raw data | 2.0× raw data |
| **Empty slot cost** | 8 bytes (pointer) | 17 bytes (full slot) |
| **Scales with** | $n$ (entries) | $m$ (capacity) |
| **Memory allocation** | Per-node (heap) | One contiguous array |
| **Memory fragmentation** | High | None |

```
╔══════════════════════════════════════════════════════════════╗
║  KEY INSIGHT:                                               ║
║                                                              ║
║  Chaining: Memory ∝ m + n (table + nodes)                  ║
║  Open Addr: Memory ∝ m (fixed per capacity)                ║
║                                                              ║
║  At high load (α → 1):                                     ║
║  Open addressing wins because no pointer overhead           ║
║                                                              ║
║  At low load (α → 0):                                      ║
║  Chaining wins because empty slots are just 8B pointers    ║
║  vs 17B empty entries in open addressing                    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Space Complexity Formula

Overall space complexity: **O(n)** (or more precisely, $O(m)$ where $m = \Theta(n)$)

```
With bounded load factor:
  m = n / α_max

  Chaining:     Space = O(m + n) = O(n/α + n) = O(n)
  Open Addr:    Space = O(m) = O(n/α) = O(n)

  Both are O(n) because α is a constant.
```

---

## Wasted Space Analysis

```
╔══════════════════════════════════════════════════════════════╗
║           SPACE WASTE SCENARIOS                             ║
║                                                              ║
║  1. JUST AFTER RESIZE (α drops from 0.75 to 0.375):        ║
║     Table doubled, but entries same                         ║
║     ~62.5% of slots empty → wasted                         ║
║                                                              ║
║  2. MANY DELETIONS (without shrinking):                     ║
║     Capacity stays large, entries few                       ║
║     Can waste 90%+ of allocated memory                     ║
║                                                              ║
║  3. TOMBSTONE ACCUMULATION:                                 ║
║     Dead slots prevent shrinking                            ║
║     Logical α = 0.1, physical α = 0.8                      ║
║                                                              ║
║  MITIGATION:                                                ║
║  • Shrink table when α < threshold                          ║
║  • Periodic rehash to clean tombstones                     ║
║  • Use exact-size tables for known datasets                ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Memory by Language

| Language | Structure | Memory per Entry (approx.) |
|----------|-----------|:---:|
| Java `HashMap` | Node: key + value + hash + next | ~48 bytes + key/value objects |
| Python `dict` | Compact: hash + key_ptr + value_ptr | ~72 bytes (3.6+) |
| C++ `unordered_map` | Node: key + value + next | ~40 bytes |
| Go `map` | Bucket: 8 entries packed | ~20 bytes per entry |
| Rust `HashMap` | Swiss table: inline key+value | ~17 bytes per entry |

```
Java HashMap entry breakdown (64-bit JVM):

┌─────────────────────────────────────┐
│ HashMap.Node object:                │
│ ├─ Object header:  16 bytes        │
│ ├─ int hash:        4 bytes        │
│ ├─ Object key ref:  8 bytes  ──────┼──► Key object (variable)
│ ├─ Object val ref:  8 bytes  ──────┼──► Value object (variable)
│ ├─ Node next ref:   8 bytes        │
│ └─ Padding:         4 bytes        │
│ = 48 bytes per node                │
│                                     │
│ + table[i] reference: 8 bytes      │
│ = ~56 bytes overhead per entry     │
└─────────────────────────────────────┘
```

---

## Optimizing Space

```
1. CHOOSE APPROPRIATE INITIAL CAPACITY
   IF you know n ≈ 1000:
     Set capacity = 1000 / α_max ≈ 1334
     Avoids multiple resizes that waste time AND memory

2. USE OPEN ADDRESSING FOR SMALL ENTRIES
   When key+value < 32 bytes, pointer overhead of chaining
   can double the total memory.

3. COMPACT REPRESENTATIONS
   Python 3.6+ uses compact dict:
   • Sparse index array (1-byte indices for small tables)
   • Dense entries array (no gaps)
   • Reduces memory by ~25%

4. FLAT HASH MAPS
   Store keys and values in separate arrays:
   keys[]   = [k₀, k₁, k₂, ...]
   values[] = [v₀, v₁, v₂, ...]
   Better cache utilization for key-only searches
```

---

## Space vs Time Trade-off

```
  Time (avg probes)
    │
 10 ┤                 ╱
    │               ╱
  8 ┤             ╱
    │           ╱
  6 ┤         ╱
    │       ╱
  4 ┤     ╱
    │   ╱
  2 ┤ ╱             More memory = fewer collisions
    │╱               = faster operations
  1 ┤═══════════
    └─┬────┬────┬────┬────┬────
     1×   2×   3×   4×   5×  
         Space (relative to data)

  Sweet spot: ~2× data size (α ≈ 0.5)
  Good balance of speed and memory
```

---

## Quick Revision Question

**Q: A system stores 1 million key-value pairs where each key is 8 bytes and each value is 8 bytes. Compare the total memory usage of chaining vs open addressing (with $\alpha = 0.75$) including all overhead.**

<details>
<summary>Click to reveal answer</summary>

$n = 1{,}000{,}000$, $\alpha = 0.75$, so $m = n / 0.75 \approx 1{,}333{,}334$

**Chaining:**
- Table array: $m \times 8$ bytes = $1{,}333{,}334 \times 8 = 10.67$ MB
- Nodes: $n \times (8 + 8 + 8 + 16)$ = $1{,}000{,}000 \times 40 = 40$ MB
  - (key + value + next pointer + object header)
- **Total ≈ 50.67 MB** (≈ 50.7 bytes/entry)

**Open Addressing:**
- Table: $m \times (8 + 8 + 1 + 7)$ = $1{,}333{,}334 \times 24 = 32$ MB
  - (key + value + flag + padding per slot)
- **Total ≈ 32 MB** (≈ 32 bytes/entry)

Open addressing uses about **37% less memory** than chaining for this workload, primarily because it avoids the per-node pointer and object header overhead. This is a significant saving at scale (18.67 MB difference for 1M entries).

</details>

---

| [← Previous: Time Complexity](05-time-complexity-analysis.md) | [Next: Unit 6 — Two Sum Problem →](../06-Classic-Hash-Problems/01-two-sum-problem.md) |
|:---|---:|
