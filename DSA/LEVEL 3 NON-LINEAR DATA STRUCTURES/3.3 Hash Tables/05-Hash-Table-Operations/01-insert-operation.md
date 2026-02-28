# 5.1 Insert Operation

## Unit 5: Hash Table Operations

---

## Overview

The **insert** (or **put**) operation stores a key-value pair in the hash table. It must handle three scenarios:
1. Slot is empty → store directly
2. Key already exists → update value
3. Collision → resolve using chosen strategy

```
╔══════════════════════════════════════════════════════════════╗
║                   INSERT PIPELINE                           ║
║                                                              ║
║  put(key, value)                                             ║
║      │                                                       ║
║      ▼                                                       ║
║  ┌──────────────┐                                            ║
║  │ Compute hash │  index = h(key) mod capacity              ║
║  └──────┬───────┘                                            ║
║         ▼                                                    ║
║  ┌──────────────┐                                            ║
║  │ Check slot   │                                            ║
║  └──────┬───────┘                                            ║
║         │                                                    ║
║    ┌────┼──────────────┐                                     ║
║    ▼    ▼              ▼                                     ║
║  EMPTY  SAME KEY    COLLISION                                ║
║  Store  Update      Resolve via                              ║
║  here   value       chaining / probing                       ║
║         │              │                                     ║
║         ▼              ▼                                     ║
║  ┌──────────────┐  ┌────────────┐                            ║
║  │ Check load   │  │ Check load │                            ║
║  │ factor       │  │ factor     │                            ║
║  └──────┬───────┘  └──────┬─────┘                            ║
║         ▼                 ▼                                   ║
║     α > threshold?  →  YES → Resize & Rehash                ║
║                     →  NO  → Done                            ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Insert with Chaining

```
FUNCTION put(key, value):
    index = hash(key) mod capacity
    chain = table[index]

    // Check if key already exists
    FOR each node in chain:
        IF node.key == key:
            node.value = value    // Update
            RETURN

    // Key not found — prepend new node (O(1))
    newNode = Node(key, value)
    newNode.next = table[index]
    table[index] = newNode
    size++

    // Check load factor
    IF size / capacity > MAX_LOAD_FACTOR:
        resize(capacity * 2)
```

### Trace: Chaining Insert

**Table**: capacity = 5, hash = key mod 5

```
Insert (7, "apple"):
  h(7) = 7 mod 5 = 2
  table[2] is null → create new node
  table[2] → [7:"apple"] → null
  size = 1, α = 0.2

  ┌───────┐
  │ 0: null│
  │ 1: null│
  │ 2: ──►│──[7:"apple"]──►null
  │ 3: null│
  │ 4: null│
  └───────┘

Insert (12, "banana"):
  h(12) = 12 mod 5 = 2        ← COLLISION with key 7!
  Traverse chain at table[2]:
    node [7:"apple"] → key 7 ≠ 12 → continue
    reach null → key not found
  Prepend new node
  table[2] → [12:"banana"] → [7:"apple"] → null
  size = 2, α = 0.4

  ┌───────┐
  │ 0: null│
  │ 1: null│
  │ 2: ──►│──[12:"banana"]──►[7:"apple"]──►null
  │ 3: null│
  │ 4: null│
  └───────┘

Insert (7, "cherry"):
  h(7) = 7 mod 5 = 2
  Traverse chain at table[2]:
    node [12:"banana"] → key 12 ≠ 7 → continue
    node [7:"apple"]   → key 7 == 7 → UPDATE value!
  table[2] → [12:"banana"] → [7:"cherry"] → null
  size = 2 (unchanged — was update, not insert)
```

---

## Insert with Open Addressing (Linear Probing)

```
FUNCTION put(key, value):
    IF size >= capacity * MAX_LOAD_FACTOR:
        resize(capacity * 2)

    index = hash(key) mod capacity

    WHILE table[index] is not EMPTY:
        IF table[index] is DELETED:
            BREAK    // Can reuse tombstone
        IF table[index].key == key:
            table[index].value = value    // Update
            RETURN
        index = (index + 1) mod capacity   // Linear probe

    table[index] = Entry(key, value)
    size++
```

### Trace: Linear Probing Insert

**Table**: capacity = 7, hash = key mod 7

```
Insert (10, "A"):
  h(10) = 3 → EMPTY → store
  [  ][  ][  ][10:A][    ][    ][  ]
   0   1   2    3     4     5    6

Insert (17, "B"):
  h(17) = 3 → OCCUPIED → probe
  3+1=4 → EMPTY → store
  [  ][  ][  ][10:A][17:B][    ][  ]
   0   1   2    3     4     5    6

Insert (24, "C"):
  h(24) = 3 → OCCUPIED → probe
  4 → OCCUPIED → probe
  5 → EMPTY → store
  [  ][  ][  ][10:A][17:B][24:C][  ]
   0   1   2    3     4     5    6

Insert (10, "X"):      ← UPDATE existing key
  h(10) = 3 → has key 10 → UPDATE value to "X"
  [  ][  ][  ][10:X][17:B][24:C][  ]
   0   1   2    3     4     5    6
```

---

## Amortized Analysis of Insert

```
╔══════════════════════════════════════════════════════════════╗
║              AMORTIZED INSERT = O(1)                        ║
║                                                              ║
║  Most inserts:  O(1)   — constant work                      ║
║  Rare resize:   O(n)   — must rehash all entries            ║
║                                                              ║
║  Amortized over n insertions:                               ║
║                                                              ║
║  Total cost = n × O(1) + O(n) resize operations            ║
║                                                              ║
║  With doubling strategy, resizes happen at:                  ║
║  n = 1, 2, 4, 8, 16, 32, ...                               ║
║                                                              ║
║  Total resize cost = 1 + 2 + 4 + ... + n = 2n - 1 = O(n)   ║
║                                                              ║
║  Amortized per insert = O(n) / n = O(1) ✓                   ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Insert Complexities

| Method | Best Case | Average Case | Worst Case | Amortized |
|--------|:---------:|:------------:|:----------:|:---------:|
| Chaining | $O(1)$ | $O(1)$ | $O(n)$ | $O(1)$ |
| Linear Probing | $O(1)$ | $O\left(\frac{1}{1-\alpha}\right)$ | $O(n)$ | $O(1)$ |
| Quadratic Probing | $O(1)$ | $O\left(\frac{1}{1-\alpha}\right)$ | $O(n)$ | $O(1)$ |
| Double Hashing | $O(1)$ | $O\left(\frac{1}{1-\alpha}\right)$ | $O(n)$ | $O(1)$ |

---

## Quick Revision Question

**Q: When inserting a key that already exists in the hash table, what should happen? How does this differ between chaining and open addressing implementations?**

<details>
<summary>Click to reveal answer</summary>

When a key already exists, the **value should be updated** (not duplicated). This makes hash tables behave as a map/dictionary with unique keys.

**Chaining**: Walk through the linked list at the computed index, comparing each node's key. If found, update the value in-place. If not found after traversing the entire chain, create a new node.

**Open addressing**: During the probing sequence, if we encounter a slot with the matching key, update its value. If we reach an empty slot first, the key doesn't exist — insert a new entry.

The key difference is that chaining must check the entire chain before confirming a miss, while open addressing stops at the first empty slot (or tombstone for insert).

</details>

---

| [← Unit 4: Comparison of Methods](../04-Collision-Handling-Open-Addressing/06-comparison-of-methods.md) | [Next: Search Operation →](02-search-operation.md) |
|:---|---:|
