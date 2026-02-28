# 5.3 Delete Operation

## Unit 5: Hash Table Operations

---

## Overview

Deletion is the **trickiest** hash table operation. Its complexity depends on the collision strategy:

```
╔══════════════════════════════════════════════════════════════╗
║                DELETION COMPLEXITY                          ║
║                                                              ║
║  CHAINING:       Simple — just remove node from chain       ║
║  OPEN ADDRESSING: Complex — must use tombstones OR rehash   ║
║                                                              ║
║  Why the difference?                                         ║
║  • Chaining: removing a node doesn't affect other lookups   ║
║  • Open addressing: emptying a slot can BREAK probe chains  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Delete with Chaining

```
FUNCTION remove(key):
    index = hash(key) mod capacity

    // Handle if first node matches
    IF table[index] is not null AND table[index].key == key:
        table[index] = table[index].next
        size--
        RETURN true

    // Search in rest of chain
    prev = table[index]
    curr = prev.next

    WHILE curr is not null:
        IF curr.key == key:
            prev.next = curr.next    // Bypass node
            size--
            RETURN true
        prev = curr
        curr = curr.next

    RETURN false    // Key not found
```

### Trace: Chaining Delete

```
Table (capacity = 5):
  ┌───────┐
  │ 0: null│
  │ 1: null│
  │ 2: ──►│──[12:"cat"]──►[7:"bird"]──►[2:"ant"]──►null
  │ 3: null│
  │ 4: null│
  └───────┘

Delete key 7:
  h(7) = 2
  Chain at index 2:
    [12] → not 7 → set prev = [12], curr = [7]
    [7]  → MATCH!
    prev.next = curr.next → [12].next = [2:"ant"]
  
  Result:
  │ 2: ──►│──[12:"cat"]──►[2:"ant"]──►null
  
  size-- (simple linked list removal)

Delete key 12 (head of chain):
  h(12) = 2
  Head node [12] → MATCH!
  table[2] = [12].next = [2:"ant"]
  
  Result:
  │ 2: ──►│──[2:"ant"]──►null
```

---

## Delete with Open Addressing — The Problem

```
╔══════════════════════════════════════════════════════════════╗
║             WHY NAIVE DELETE BREAKS THINGS                  ║
║                                                              ║
║  Table:  [10][ 17 ][ 24 ][ 31 ][ ][ ][ ]                  ║
║           3     4     5     6   0  1  2                     ║
║  All keys hash to 3, inserted via linear probing            ║
║                                                              ║
║  NAIVE DELETE of 17 (set slot 4 to EMPTY):                  ║
║  [10][    ][ 24 ][ 31 ][ ][ ][ ]                           ║
║   3    4     5     6   0  1  2                              ║
║        ↑                                                     ║
║        empty gap                                             ║
║                                                              ║
║  Search for 24:  h(24) = 3                                  ║
║    Probe 0: slot 3 → 10 ≠ 24 → continue                    ║
║    Probe 1: slot 4 → EMPTY → "NOT FOUND" ✗ WRONG!          ║
║                                                              ║
║  24 IS in the table (at slot 5) but was never reached!     ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Solution 1: Tombstone (Lazy Delete)

Mark deleted slots with a special marker instead of setting them empty:

```
FUNCTION remove(key):
    index = hash(key) mod capacity

    FOR i = 0 TO capacity - 1:
        IF table[index] is EMPTY:
            RETURN false    // Key doesn't exist
        IF table[index] is not DELETED AND table[index].key == key:
            table[index] = DELETED    // Place tombstone
            size--
            RETURN true
        index = (index + 1) mod capacity

    RETURN false
```

### Trace: Tombstone Delete

```
Before delete:
  [10][ 17 ][ 24 ][ 31 ][ ][ ][ ]
   3     4     5     6   0  1  2

Delete 17 (place tombstone [X]):
  [10][ X  ][ 24 ][ 31 ][ ][ ][ ]
   3    4     5     6   0  1  2

Search for 24:  h(24) = 3
  Probe 0: slot 3 → 10 ≠ 24 → continue
  Probe 1: slot 4 → DELETED → continue (DON'T STOP)
  Probe 2: slot 5 → 24 == 24 → FOUND ✓

Insert 38:  h(38) = 3
  Probe 0: slot 3 → occupied
  Probe 1: slot 4 → DELETED → CAN REUSE → insert 38 here!
  [10][ 38 ][ 24 ][ 31 ][ ][ ][ ]
   3    4     5     6   0  1  2
```

### Tombstone Accumulation Problem

```
╔══════════════════════════════════════════════════════════════╗
║          TOMBSTONE ACCUMULATION                              ║
║                                                              ║
║  After many insert/delete cycles:                           ║
║                                                              ║
║  [A ][ X ][ X ][ B ][ X ][ C ][ X ][ X ][ D ][ X ]       ║
║   0    1    2    3    4    5    6    7    8    9             ║
║                                                              ║
║  4 live entries, 6 tombstones, 0 truly empty                ║
║  α = 4/10 = 0.4 — looks fine                               ║
║  But effective load = 10/10 = 1.0 — ALL slots occupied!    ║
║                                                              ║
║  Search for missing key must probe ALL 10 slots!            ║
║  Performance degrades to O(n) despite low logical load.     ║
║                                                              ║
║  Solution: Periodic rehashing to clean tombstones           ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Solution 2: Backward Shift Delete

An alternative that avoids tombstones entirely:

```
FUNCTION remove(key):
    index = hash(key) mod capacity
    
    // Find the key
    WHILE table[index] is not EMPTY:
        IF table[index].key == key:
            BREAK
        index = (index + 1) mod capacity
    
    IF table[index] is EMPTY:
        RETURN false    // Key not found
    
    // Shift subsequent entries backward
    size--
    next = (index + 1) mod capacity
    
    WHILE table[next] is not EMPTY:
        idealSlot = hash(table[next].key) mod capacity
        
        // Check if 'next' is displaced past 'index'
        IF (index < next AND (idealSlot <= index OR idealSlot > next)) OR
           (index > next AND (idealSlot <= index AND idealSlot > next)):
            table[index] = table[next]
            index = next
        next = (next + 1) mod capacity
    
    table[index] = EMPTY
    RETURN true
```

### Trace: Backward Shift Delete

```
Before:
  [10][ 17 ][ 24 ][ 31 ][   ][   ][   ]
   3     4     5     6    0    1    2
  All ideally hash to slot 3

Delete 17 (at slot 4):
  
  Step 1: Empty slot 4
  [10][    ][ 24 ][ 31 ][   ][   ][   ]
   3    4     5     6    0    1    2

  Step 2: Check slot 5 (key 24, ideal = 3)
    ideal=3 ≤ hole=4 → shift 24 backward
  [10][ 24 ][    ][ 31 ][   ][   ][   ]
   3    4     5     6    0    1    2

  Step 3: Check slot 6 (key 31, ideal = 3)
    ideal=3 ≤ hole=5 → shift 31 backward
  [10][ 24 ][ 31 ][    ][   ][   ][   ]
   3    4     5    6    0    1    2

  Step 4: Slot 6 is now empty → done!

  Result: NO tombstones! Probe chains preserved!
  Search for 24: h=3, probe 3→4 → FOUND at 4 ✓
```

---

## Comparison: Tombstone vs Backward Shift

| Feature | Tombstone | Backward Shift |
|---------|:---------:|:-------------:|
| Implementation | Simple | Complex |
| Delete cost | $O(1)$ | $O(\text{cluster size})$ |
| Search overhead | Tombstones lengthen probes | No extra overhead |
| Memory waste | Tombstones waste slots | No wasted slots |
| Need periodic rehash | Yes | No |
| Used in practice | Most implementations | Python `dict`, Rust |

---

## Delete Complexity

| Method | Best | Average | Worst |
|--------|:---:|:---:|:---:|
| Chaining | $O(1)$ | $O(1 + \alpha)$ | $O(n)$ |
| Open Addr (tombstone) | $O(1)$ | $O\left(\frac{1}{1-\alpha}\right)$ | $O(n)$ |
| Open Addr (back-shift) | $O(1)$ | $O\left(\frac{1}{1-\alpha}\right)$ | $O(n)$ |

---

## Quick Revision Question

**Q: Describe two different strategies for handling deletion in open addressing. What trade-off does each make?**

<details>
<summary>Click to reveal answer</summary>

**Strategy 1: Tombstones (Lazy Delete)**
- Mark deleted slot with a DELETED sentinel instead of EMPTY
- **Trade-off**: Delete is $O(1)$ (just mark), but tombstones accumulate over time, degrading search performance. Requires periodic rehashing to clean up.
- Search must skip over tombstones (continue probing), insert can reuse them.

**Strategy 2: Backward Shift Delete**
- After removing an entry, shift subsequent entries backward to fill the gap, maintaining the probe chain
- **Trade-off**: Delete is more expensive (up to cluster-size shifts), but leaves no tombstones. Search performance remains optimal without periodic cleanup.
- More complex to implement correctly, especially with wrap-around.

Tombstones are simpler and more common; backward shift is used in Python's `dict` and Rust's `HashMap` for better long-term performance.

</details>

---

| [← Previous: Search Operation](02-search-operation.md) | [Next: Resize & Rehash →](04-resize-and-rehash.md) |
|:---|---:|
