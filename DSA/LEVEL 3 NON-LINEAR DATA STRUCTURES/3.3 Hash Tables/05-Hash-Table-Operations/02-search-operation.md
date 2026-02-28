# 5.2 Search Operation

## Unit 5: Hash Table Operations

---

## Overview

The **search** (or **get** / **lookup**) operation retrieves a value associated with a given key. This is the primary reason hash tables exist — $O(1)$ average-case lookups.

```
╔══════════════════════════════════════════════════════════════╗
║                   SEARCH PIPELINE                           ║
║                                                              ║
║  get(key)                                                    ║
║      │                                                       ║
║      ▼                                                       ║
║  ┌──────────────┐                                            ║
║  │ Compute hash │  index = h(key) mod capacity              ║
║  └──────┬───────┘                                            ║
║         ▼                                                    ║
║  ┌──────────────────┐                                        ║
║  │ Examine slot(s)  │                                        ║
║  └──────┬───────────┘                                        ║
║         │                                                    ║
║    ┌────┴────────────────┐                                   ║
║    ▼                     ▼                                   ║
║  KEY FOUND            KEY NOT FOUND                          ║
║  Return value         Return null / throw                    ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Search with Chaining

```
FUNCTION get(key):
    index = hash(key) mod capacity
    node = table[index]

    WHILE node is not null:
        IF node.key == key:
            RETURN node.value    // Found!
        node = node.next

    RETURN null    // Not found
```

### Trace: Chaining Search

**Table** (capacity = 5, hash = key mod 5):

```
  ┌───────┐
  │ 0: ──►│──[15:"dog"]──►null
  │ 1: null│
  │ 2: ──►│──[12:"cat"]──►[7:"bird"]──►null
  │ 3: ──►│──[8:"fish"]──►null
  │ 4: null│
  └───────┘

Search for key 7:
  h(7) = 7 mod 5 = 2
  Chain at index 2:
    Node [12:"cat"] → key 12 ≠ 7 → next
    Node [7:"bird"] → key 7 == 7 → FOUND! Return "bird"
  Comparisons: 2

Search for key 22:
  h(22) = 22 mod 5 = 2
  Chain at index 2:
    Node [12:"cat"] → key 12 ≠ 22 → next
    Node [7:"bird"] → key 7 ≠ 22 → next
    null → end of chain → NOT FOUND
  Comparisons: 2 + 1 null check = effectively 2 key comparisons
```

---

## Search with Open Addressing (Linear Probing)

```
FUNCTION get(key):
    index = hash(key) mod capacity

    WHILE table[index] is not EMPTY:
        IF table[index] is not DELETED:
            IF table[index].key == key:
                RETURN table[index].value    // Found!
        index = (index + 1) mod capacity

    RETURN null    // Hit empty slot → key doesn't exist
```

### Trace: Linear Probing Search

```
Table (capacity = 7, hash = key mod 7):
  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │ 38  │     │     │ 10  │ 17  │ 24  │ 31  │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    0     1     2     3     4     5     6

All keys hash to index 3 (mod 7)

Search for key 24:
  h(24) = 3
  Probe 0: index 3 → has 10 → 10 ≠ 24 → continue
  Probe 1: index 4 → has 17 → 17 ≠ 24 → continue
  Probe 2: index 5 → has 24 → 24 == 24 → FOUND!
  Return value at index 5
  Total probes: 3

Search for key 99:
  h(99) = 99 mod 7 = 1
  Probe 0: index 1 → EMPTY → NOT FOUND immediately
  Total probes: 1  ← best case for miss!

Search for key 45 (not in table):
  h(45) = 45 mod 7 = 3
  Probe 0: index 3 → 10 ≠ 45
  Probe 1: index 4 → 17 ≠ 45
  Probe 2: index 5 → 24 ≠ 45
  Probe 3: index 6 → 31 ≠ 45
  Probe 4: index 0 → 38 ≠ 45
  Probe 5: index 1 → EMPTY → NOT FOUND
  Total probes: 6  ← worst case, traversed entire cluster!
```

---

## Search with Tombstones

```
╔══════════════════════════════════════════════════════════════╗
║           SEARCH MUST SKIP TOMBSTONES                       ║
║                                                              ║
║  Table:  [10][ X ][ 24 ][ 31 ][ ]                          ║
║           0    1     2     3    4                            ║
║                ↑                                             ║
║            tombstone (17 was deleted)                        ║
║                                                              ║
║  Search for 24 (h(24) = 0):                                 ║
║  Probe 0: index 0 → has 10 → ≠ 24 → continue               ║
║  Probe 1: index 1 → DELETED → SKIP (don't stop!) → continue║
║  Probe 2: index 2 → has 24 → == 24 → FOUND! ✓              ║
║                                                              ║
║  If we stopped at tombstone → would miss key 24!            ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Successful vs Unsuccessful Search

```
SUCCESSFUL SEARCH:   Key exists → must find it
                     Cost depends on insertion order
                     Earlier inserts tend to be found faster

UNSUCCESSFUL SEARCH: Key doesn't exist → must confirm absence
                     Chaining:  traverse full chain + 1 null
                     Open Addr: probe until empty slot found
                     Usually MORE expensive than successful

                     Unsuccessful ≥ Successful (always)
```

### Expected Probes Formula

| Method | Successful | Unsuccessful |
|--------|:---:|:---:|
| Chaining | $1 + \frac{\alpha}{2}$ | $1 + \alpha$ |
| Linear Probing | $\frac{1}{2}\left(1 + \frac{1}{1-\alpha}\right)$ | $\frac{1}{2}\left(1 + \frac{1}{(1-\alpha)^2}\right)$ |
| Double Hashing | $\frac{1}{\alpha}\ln\frac{1}{1-\alpha}$ | $\frac{1}{1-\alpha}$ |

---

## Search Complexity Summary

| Case | Chaining | Open Addressing |
|------|:---:|:---:|
| **Best case** | $O(1)$ — first node matches | $O(1)$ — first slot matches |
| **Average (hit)** | $O(1 + \alpha/2)$ | $O\left(\frac{1}{1-\alpha}\right)$ |
| **Average (miss)** | $O(1 + \alpha)$ | $O\left(\frac{1}{1-\alpha}\right)$ |
| **Worst case** | $O(n)$ — all in one chain | $O(n)$ — probe entire table |

With constant load factor ($\alpha < 0.75$): **all are $O(1)$** average.

---

## Optimization: Early Termination

```
╔══════════════════════════════════════════════════════════════╗
║            MOVE-TO-FRONT HEURISTIC (Chaining)               ║
║                                                              ║
║  When a key is found deep in a chain,                       ║
║  move it to the front for faster future lookups:            ║
║                                                              ║
║  Before:  chain → [A] → [B] → [C] → null                   ║
║  Search for C:  3 comparisons                                ║
║                                                              ║
║  After move-to-front:  chain → [C] → [A] → [B] → null      ║
║  Next search for C:    1 comparison!                         ║
║                                                              ║
║  Useful when access patterns have temporal locality.        ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: In open addressing, why does an unsuccessful search typically require more probes than a successful search?**

<details>
<summary>Click to reveal answer</summary>

A **successful search** stops as soon as it finds the matching key, which could be at any point along the probe sequence. On average, it stops partway through.

An **unsuccessful search** must continue probing until it finds an **empty slot** to confirm the key doesn't exist. It cannot stop early — every occupied slot (whether matching or not) must be checked. This means it always probes at least as far as the longest successful search in that cluster, and potentially further.

With tombstones, the problem is even worse: deleted markers force the search to continue past slots that look "occupied," further increasing the average number of probes for misses.

Numerically at $\alpha = 0.75$: successful search ≈ 2.5 probes, unsuccessful ≈ 8.5 probes (linear probing).

</details>

---

| [← Previous: Insert Operation](01-insert-operation.md) | [Next: Delete Operation →](03-delete-operation.md) |
|:---|---:|
