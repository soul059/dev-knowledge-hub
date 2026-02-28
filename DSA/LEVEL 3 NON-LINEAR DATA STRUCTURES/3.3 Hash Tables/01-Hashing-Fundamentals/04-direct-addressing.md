# 1.4 Direct Addressing

## Unit 1: Hashing Fundamentals

---

## What is Direct Addressing?

**Direct Addressing** is the simplest form of key-to-value mapping. The key itself is used directly as the array index — no hash function is needed. If the key is `k`, the value is stored at `table[k]`.

```
╔════════════════════════════════════════════════════════════╗
║                  DIRECT ADDRESS TABLE                     ║
║                                                           ║
║   Key IS the Index (no hash function needed)              ║
║                                                           ║
║   key = 3  ──► table[3]                                   ║
║   key = 7  ──► table[7]                                   ║
║   key = 1  ──► table[1]                                   ║
║                                                           ║
║   Index:  0    1    2    3    4    5    6    7    8    9   ║
║         ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
║         │    │ V1 │    │ V3 │    │    │    │ V7 │    │    │
║         └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
║                ▲         ▲                   ▲             ║
║              key=1     key=3               key=7           ║
║                                                           ║
║   ✓ O(1) guaranteed    ✗ Wastes space for sparse keys    ║
╚════════════════════════════════════════════════════════════╝
```

---

## How Direct Addressing Works

### Operations

```
CREATE: Allocate array of size = max possible key value + 1

INSERT(key, value):   table[key] = value         → O(1)

SEARCH(key):          return table[key]           → O(1)

DELETE(key):          table[key] = null           → O(1)
```

All operations are **guaranteed O(1)** — no collisions possible because each key has a unique slot.

---

## Pseudocode

```
CLASS DirectAddressTable:
    FUNCTION init(max_key):
        this.table = new Array(max_key + 1)    // size = max key + 1
        Initialize all entries to null
    
    FUNCTION insert(key, value):
        this.table[key] = value                // Direct mapping
    
    FUNCTION search(key):
        RETURN this.table[key]                 // Direct access
    
    FUNCTION delete(key):
        this.table[key] = null                 // Direct removal
```

---

## Step-by-Step Trace

```
Direct Address Table, max_key = 9

Operation 1: insert(2, "Bob")
  table[2] = "Bob"
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │     │     │"Bob"│     │     │     │     │     │     │     │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    0     1     2     3     4     5     6     7     8     9

Operation 2: insert(5, "Eve")
  table[5] = "Eve"
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │     │     │"Bob"│     │     │"Eve"│     │     │     │     │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    0     1     2     3     4     5     6     7     8     9

Operation 3: insert(8, "Hal")
  table[8] = "Hal"
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │     │     │"Bob"│     │     │"Eve"│     │     │"Hal"│     │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    0     1     2     3     4     5     6     7     8     9

Operation 4: search(5)
  return table[5] = "Eve"  ✓   O(1)

Operation 5: delete(2)
  table[2] = null
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │     │     │     │     │     │"Eve"│     │     │"Hal"│     │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    0     1     2     3     4     5     6     7     8     9

Only 3 entries stored in a 10-slot array → 70% wasted space!
```

---

## Limitations of Direct Addressing

```
╔════════════════════════════════════════════════════════════════╗
║              WHY DIRECT ADDRESSING FAILS                      ║
║                                                               ║
║   Problem 1: HUGE KEY SPACE                                   ║
║   ─────────────────────────                                   ║
║   Keys: phone numbers (10 digits) → 10^10 possible values    ║
║   Array size needed: 10,000,000,000 slots!                    ║
║   Memory: ~40 GB just for the array!                          ║
║                                                               ║
║   Problem 2: SPARSE DATA                                      ║
║   ──────────────────────                                      ║
║   Only 1000 students but IDs range 0-999999                   ║
║   ┌───────────────────────────────────────┐                   ║
║   │░░░░░█░░░░░░░░░░█░░░░░░░░░█░░░░░░░░░░│                   ║
║   └───────────────────────────────────────┘                   ║
║   Only 0.1% of array used → 99.9% wasted!                    ║
║                                                               ║
║   Problem 3: NON-INTEGER KEYS                                 ║
║   ───────────────────────────                                 ║
║   Keys like "alice", "bob" can't be used as indices           ║
║   Strings, floats, objects need conversion first              ║
╚════════════════════════════════════════════════════════════════╝
```

---

## Direct Addressing vs Hashing

| Aspect | Direct Addressing | Hash Table |
|--------|:-----------------:|:----------:|
| Key as index | Yes | No (uses hash function) |
| Collisions | None | Possible |
| Space | O(max key value) | O(n) |
| Key types | Integers only | Any hashable type |
| Worst-case lookup | O(1) | O(n) |
| Average lookup | O(1) | O(1) |
| Practical for large key ranges | No | Yes |

```
╔═══════════════════════════════════════════════════════════════╗
║          DIRECT ADDRESSING → HASHING MOTIVATION              ║
║                                                              ║
║   Direct Addressing:                                         ║
║   [  ][ ][ ][ ][ ][ ][ ][ ]...[ ][ ][ ]  size = max(key)  ║
║    ░░░ ░░░ ░░░ █  ░░░ ░░░ ░░░     █  ░░░                   ║
║                                                              ║
║   Hashing (compress to smaller table):                       ║
║   [  ][ ][ ][ ][ ]                         size = O(n)     ║
║    █   █       █   █                                         ║
║                                                              ║
║   Same data, much less space!                                ║
║   Trade-off: possible collisions                             ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## When to Use Direct Addressing

**Use When:**
- Key range is small and known (e.g., ASCII characters 0-127)
- Keys are dense (most values in range are used)
- Maximum speed is critical (no hash function overhead)

**Common Examples:**
- Counting character frequencies (key range 0-255)
- Boolean flag arrays
- Lookup tables for small enumerations

```
// Counting character frequencies — perfect for direct addressing
FUNCTION count_chars(string):
    freq = new Array(128)    // ASCII range
    FOR each char c in string:
        freq[ASCII(c)] = freq[ASCII(c)] + 1
    RETURN freq
```

---

## Quick Revision Question

**Q: A system stores records for employees with IDs between 1 and 50. Only 40 employees exist. Should you use direct addressing or a hash table? What if IDs range from 1 to 1,000,000?**

<details>
<summary>Click to reveal answer</summary>

**IDs 1-50, 40 employees**: Use **direct addressing**. The key range is small (50 slots), and 80% of slots are used (dense). The overhead of a hash function is unnecessary, and you get guaranteed O(1) with no collision handling.

**IDs 1-1,000,000**: Use a **hash table**. Direct addressing would require an array of 1,000,000 slots for only 40 entries — a utilization of just 0.004%. A hash table with ~50-80 slots and a good hash function provides O(1) average case with dramatically less memory waste.

</details>

---

| [← Previous: Hash Table Structure](03-hash-table-structure.md) | [Next: Why Hashing? →](05-why-hashing.md) |
|:---|---:|
