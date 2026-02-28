# 5.4 Resize and Rehash

## Unit 5: Hash Table Operations

---

## Why Resize?

As the load factor $\alpha$ grows, performance degrades:

```
╔══════════════════════════════════════════════════════════════╗
║              PERFORMANCE vs LOAD FACTOR                     ║
║                                                              ║
║  α = 0.25 → avg 1.3 probes  (fast)                        ║
║  α = 0.50 → avg 2.0 probes  (good)                        ║
║  α = 0.75 → avg 4.0 probes  (acceptable)                  ║
║  α = 0.90 → avg 10.0 probes (slow!)                       ║
║  α = 0.99 → avg 100.0 probes (unusable!)                   ║
║                                                              ║
║  Solution: When α exceeds a threshold, RESIZE the table    ║
║  to restore performance.                                     ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Resize Threshold Defaults

| Language | Default Load Factor | Resize Strategy |
|----------|:---:|:---|
| Java `HashMap` | 0.75 | Double capacity |
| Python `dict` | 0.667 | Quadruple capacity (when small) |
| C++ `unordered_map` | 1.0 | Impl-defined (usually double) |
| Go `map` | 0.65 | Double bucket count |
| Rust `HashMap` | 0.875 | Double capacity |

---

## The Rehash Process

**Resizing is NOT just copying** — every entry must be **rehashed** because positions depend on table size.

```
╔══════════════════════════════════════════════════════════════╗
║                   REHASH PROCESS                            ║
║                                                              ║
║  1. Allocate new table (2× capacity)                        ║
║  2. For each entry in old table:                            ║
║     a. Compute NEW index using new capacity                 ║
║     b. Insert into new table                                ║
║  3. Replace old table with new table                        ║
║  4. Old table gets garbage collected                        ║
║                                                              ║
║  KEY INSIGHT: h(k) mod 8 ≠ h(k) mod 16                    ║
║  Every single entry may move to a new position!             ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Step-by-Step Rehash Trace

**Old table**: capacity = 4, max load = 0.75, 3 entries (α = 0.75 → trigger!)

```
Old Table (capacity 4, h(k) = k mod 4):
  ┌─────┬─────┬─────┬─────┐
  │  8  │  5  │ 14  │     │
  └─────┴─────┴─────┴─────┘
    0     1     2     3
  
  8 mod 4 = 0  ✓
  5 mod 4 = 1  ✓
  14 mod 4 = 2 ✓

Insert (7, value):  h(7) = 7 mod 4 = 3
  After insert: α = 4/4 = 1.0 > 0.75 → RESIZE!

Step 1: Allocate new table, capacity = 8

  New Table (capacity 8):
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │     │     │     │     │     │     │     │     │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    0     1     2     3     4     5     6     7

Step 2: Rehash 8
  8 mod 8 = 0 → insert at 0
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │  8  │     │     │     │     │     │     │     │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Step 3: Rehash 5
  5 mod 8 = 5 → insert at 5  ← MOVED from index 1 to 5!
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │  8  │     │     │     │     │  5  │     │     │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Step 4: Rehash 14
  14 mod 8 = 6 → insert at 6  ← MOVED from index 2 to 6!
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │  8  │     │     │     │     │  5  │ 14  │     │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Step 5: Rehash 7
  7 mod 8 = 7 → insert at 7
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │  8  │     │     │     │     │  5  │ 14  │  7  │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘

Done! New α = 4/8 = 0.5 — performance restored!
```

---

## Power-of-Two Optimization

```
╔══════════════════════════════════════════════════════════════╗
║         POWER-OF-TWO TABLE SIZE TRICK                       ║
║                                                              ║
║  When capacity is always 2^k:                               ║
║  • h(k) mod 2^k  =  h(k) & (2^k - 1)                     ║
║  • Bitwise AND is faster than modulo!                       ║
║                                                              ║
║  Example: capacity = 16 = 2^4                               ║
║  h(k) mod 16  ≡  h(k) & 0b1111  ≡  h(k) & 15             ║
║                                                              ║
║  During resize (16 → 32):                                   ║
║  Each key either STAYS at index i                           ║
║  or MOVES to index i + 16.                                  ║
║                                                              ║
║  Why? Only one extra bit matters:                           ║
║  h(k) & 0b01111 → old index                               ║
║  h(k) & 0b11111 → if new bit is 0: same index             ║
║                    if new bit is 1: index + oldCapacity     ║
╚══════════════════════════════════════════════════════════════╝
```

### Bit Pattern Example

```
key = 42, hash = 42 = 0b101010

Old capacity 16:  42 & 0b01111 = 0b01010 = 10
New capacity 32:  42 & 0b11111 = 0b01010 = 10  ← stays!

key = 26, hash = 26 = 0b011010

Old capacity 16:  26 & 0b01111 = 0b01010 = 10
New capacity 32:  26 & 0b11111 = 0b11010 = 26 = 10 + 16  ← moves!
```

---

## Shrinking (Downsizing)

```
╔══════════════════════════════════════════════════════════════╗
║                TABLE SHRINKING                              ║
║                                                              ║
║  If many entries are deleted, the table may be too large:   ║
║                                                              ║
║  capacity = 1024, entries = 50 → α = 0.05 (wasting memory) ║
║                                                              ║
║  Common strategy:                                            ║
║  • Shrink when α < 0.125 (or similar low threshold)        ║
║  • Halve the capacity: new capacity = capacity / 2          ║
║  • Must rehash all entries (same process as growing)        ║
║                                                              ║
║  Hysteresis:                                                 ║
║  • Grow at α > 0.75                                         ║
║  • Shrink at α < 0.125 (not 0.375 = 0.75/2)               ║
║  • Gap prevents rapid grow/shrink oscillation               ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Incremental Rehashing

For very large tables, rehashing all at once is expensive. **Incremental rehashing** spreads the cost:

```
╔══════════════════════════════════════════════════════════════╗
║             INCREMENTAL REHASHING (Redis approach)          ║
║                                                              ║
║  1. Keep BOTH old and new tables active                     ║
║  2. On each operation (get/put/delete):                     ║
║     - Move a few entries from old → new table               ║
║     - Perform the operation on whichever table has the key  ║
║  3. When old table is empty, discard it                     ║
║                                                              ║
║  ┌─────────────┐    ┌──────────────────────────┐            ║
║  │  Old Table   │    │       New Table           │           ║
║  │  (shrinking) │    │       (growing)           │           ║
║  │  [A][B][ ]   │───►│  [A][B][ ][ ][ ][ ]     │           ║
║  │  [C][ ][ ]   │    │  [C][ ][ ][ ][ ][ ]     │           ║
║  └─────────────┘    └──────────────────────────┘            ║
║                                                              ║
║  Advantage: No single O(n) spike                            ║
║  Disadvantage: More complex, uses 2× memory temporarily    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Amortized Cost of Resize

```
Resize operations with doubling strategy:

Insert #  │ Capacity │ Resize?  │ Entries Moved
──────────┼──────────┼──────────┼──────────────
1         │ 4        │ No       │ 0
2         │ 4        │ No       │ 0
3         │ 4        │ No       │ 0
4         │ 4 → 8   │ YES      │ 3
5         │ 8        │ No       │ 0
6         │ 8        │ No       │ 0
7         │ 8 → 16  │ YES      │ 6
...       │          │          │
n         │ → 2n     │ YES      │ n-1

Total moves across n inserts:
  3 + 6 + 12 + ... ≈ 2n

Amortized cost per insert = O(2n/n) = O(1)
```

---

## Pseudocode: Complete Resize

```
FUNCTION resize(newCapacity):
    oldTable = table
    oldCapacity = capacity

    // Allocate new table
    table = new Array(newCapacity)
    capacity = newCapacity
    size = 0    // Will be re-counted during reinsert

    // Rehash all entries from old table
    FOR i = 0 TO oldCapacity - 1:
        IF oldTable[i] is not EMPTY and not DELETED:
            put(oldTable[i].key, oldTable[i].value)
            // ↑ Uses NEW table and NEW capacity

    // Old table is freed/garbage collected
```

---

## Quick Revision Question

**Q: Why can't you simply copy entries to the same indices in a resized table? What must be done instead, and what is the amortized cost?**

<details>
<summary>Click to reveal answer</summary>

You can't copy to the same indices because the **index depends on the table capacity**: $\text{index} = h(k) \bmod m$. When $m$ changes, the computed index changes for most keys. For example, key 14 maps to index 2 in a table of size 4 ($14 \bmod 4 = 2$), but to index 6 in a table of size 8 ($14 \bmod 8 = 6$).

Instead, you must **rehash** every entry: compute its new index using the new capacity and insert it into the new table. This takes $O(n)$ for one resize.

However, with a doubling strategy, resizes occur at intervals $1, 2, 4, 8, \ldots, n$, with total rehash work $\sum = O(n)$. Spread over $n$ inserts, the **amortized cost per insert is $O(1)$**.

</details>

---

| [← Previous: Delete Operation](03-delete-operation.md) | [Next: Time Complexity Analysis →](05-time-complexity-analysis.md) |
|:---|---:|
