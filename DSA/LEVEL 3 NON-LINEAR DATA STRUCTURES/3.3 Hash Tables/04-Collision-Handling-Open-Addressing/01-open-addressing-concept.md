# 4.1 Open Addressing Concept

## Unit 4: Collision Handling - Open Addressing

---

## What is Open Addressing?

In **open addressing**, all elements are stored **directly in the hash table array** — no linked lists or external chains. When a collision occurs, we **probe** (search) for the next available slot within the array itself.

```
╔═══════════════════════════════════════════════════════════════╗
║          CHAINING  vs  OPEN ADDRESSING                       ║
║                                                              ║
║   CHAINING:                    OPEN ADDRESSING:              ║
║   ┌───┐                       ┌───────────────┐             ║
║   │ 0 │──►[A]──►[D]──►null   │ 0 │ A         │             ║
║   │ 1 │──►null                │ 1 │ D  ◄──probe│            ║
║   │ 2 │──►[B]──►null         │ 2 │ B         │             ║
║   │ 3 │──►[C]──►null         │ 3 │ C         │             ║
║   └───┘                       │ 4 │ (empty)   │             ║
║                                └───────────────┘             ║
║   External storage             Everything inside array       ║
║   Extra pointers               No extra pointers             ║
║   α can be > 1                 α must be ≤ 1                ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## How Probing Works

```
╔════════════════════════════════════════════════════════════╗
║                   PROBING CONCEPT                         ║
║                                                           ║
║   INSERT key K:                                           ║
║   1. Compute index = h(K)                                ║
║   2. If table[index] is empty → store K here            ║
║   3. If table[index] is occupied → PROBE next slot      ║
║   4. Keep probing until an empty slot is found           ║
║                                                           ║
║   Example: Insert "D" where h("D") = 0, but 0 is taken  ║
║                                                           ║
║   ┌─────┬─────┬─────┬─────┬─────┐                       ║
║   │  A  │     │  B  │  C  │     │                       ║
║   └─────┴─────┴─────┴─────┴─────┘                       ║
║     0↑     1     2     3     4                           ║
║     │                                                     ║
║     Collision! Probe to next slot                        ║
║                                                           ║
║   ┌─────┬─────┬─────┬─────┬─────┐                       ║
║   │  A  │  D  │  B  │  C  │     │                       ║
║   └─────┴─────┴─────┴─────┴─────┘                       ║
║     0     1↑     2     3     4                           ║
║           │                                               ║
║           Found empty slot! Store D here.                ║
╚════════════════════════════════════════════════════════════╝
```

---

## Probe Sequence

A **probe sequence** is the order in which we check slots. For key $k$, the probe sequence is:

$$h(k, 0), h(k, 1), h(k, 2), \ldots, h(k, m-1)$$

The sequence must be a **permutation** of $\{0, 1, 2, \ldots, m-1\}$ to ensure every slot is checked.

```
Three main probing strategies:

1. LINEAR PROBING:     h(k,i) = (h(k) + i) mod m
   Probe: h, h+1, h+2, h+3, ...

2. QUADRATIC PROBING:  h(k,i) = (h(k) + c₁i + c₂i²) mod m
   Probe: h, h+1, h+4, h+9, ...

3. DOUBLE HASHING:     h(k,i) = (h₁(k) + i·h₂(k)) mod m
   Probe: h₁, h₁+h₂, h₁+2h₂, ...
```

---

## Open Addressing Operations

```
// INSERT
FUNCTION insert(key, value):
    FOR i = 0 TO m - 1:
        index = probe(key, i)
        IF table[index] is EMPTY or DELETED:
            table[index] = (key, value)
            RETURN index
    ERROR "Hash table is full!"

// SEARCH
FUNCTION search(key):
    FOR i = 0 TO m - 1:
        index = probe(key, i)
        IF table[index] is EMPTY:
            RETURN null        // Key not in table
        IF table[index].key == key:
            RETURN table[index].value  // Found!
    RETURN null

// DELETE (using tombstones)
FUNCTION delete(key):
    FOR i = 0 TO m - 1:
        index = probe(key, i)
        IF table[index] is EMPTY:
            RETURN null        // Key not found
        IF table[index].key == key:
            table[index] = DELETED     // Mark as tombstone
            RETURN key
    RETURN null
```

---

## The Tombstone Problem

```
╔═══════════════════════════════════════════════════════════════╗
║                   TOMBSTONE (DELETED MARKER)                 ║
║                                                              ║
║   Why not just set deleted slot to EMPTY?                    ║
║                                                              ║
║   Table: [A][ ][B][C][ ]    h(A)=0, h(B)=0, h(C)=0        ║
║           0  1  2  3  4     B and C were probed past A      ║
║                                                              ║
║   Delete A, set to EMPTY:                                    ║
║   [ ][ ][B][C][ ]                                           ║
║    0  1  2  3  4                                             ║
║    ↑                                                         ║
║    Search for B: h(B)=0, slot 0 is EMPTY → "NOT FOUND" ✗  ║
║    B is actually at index 2, but we stopped too early!      ║
║                                                              ║
║   Solution — use TOMBSTONE marker [X]:                      ║
║   [X][ ][B][C][ ]                                           ║
║    0  1  2  3  4                                             ║
║    ↑                                                         ║
║    Search for B: h(B)=0, slot 0 is DELETED → keep probing  ║
║    Check slot 1: EMPTY? No, continue... slot 2: B! FOUND ✓ ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Constraints

| Constraint | Reason |
|-----------|--------|
| $\alpha \leq 1$ | Every element needs its own slot, no external storage |
| $\alpha < 0.7$ recommended | Performance degrades rapidly above 0.7 |
| Deletion is tricky | Must use tombstones, not empty markers |
| Good for cache | All data in contiguous array |

---

## Quick Revision Question

**Q: Why can't you simply set a deleted slot to "empty" in open addressing? What alternative is used, and what problem does it introduce?**

<details>
<summary>Click to reveal answer</summary>

Setting a deleted slot to "empty" would **break the probe chain**. If key B was inserted after probing past a slot now marked empty, a search for B would stop at the empty slot and incorrectly report "not found."

The alternative is a **tombstone** (deleted marker). During search, tombstones are treated as occupied (continue probing past them). During insert, tombstones can be overwritten with new data.

**Problem introduced**: Over time, many tombstones accumulate, lengthening probe sequences even for keys that aren't involved in collisions. This degrades performance. The solution is periodic **rehashing** — rebuilding the table while discarding tombstones.

</details>

---

| [← Unit 3: Implementation Details](../03-Collision-Handling-Chaining/06-implementation-details.md) | [Next: Linear Probing →](02-linear-probing.md) |
|:---|---:|
