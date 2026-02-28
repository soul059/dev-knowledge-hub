# 4.2 Linear Probing

## Unit 4: Collision Handling - Open Addressing

---

## Linear Probing Formula

$$h(k, i) = (h(k) + i) \bmod m$$

Where:
- $h(k)$ = primary hash function
- $i$ = probe number (0, 1, 2, ...)
- $m$ = table size

The probe sequence advances **one slot at a time**:

```
Position 0:  h(k)
Position 1:  h(k) + 1
Position 2:  h(k) + 2
Position 3:  h(k) + 3
...wraps around at m
```

---

## Step-by-Step Insertion Trace

**Setup**: Table size $m = 7$, hash function $h(k) = k \bmod 7$

**Insert keys: 10, 17, 24, 31, 38**

Note: All keys hash to $\text{index } 3$ ($10 \bmod 7 = 3$, $17 \bmod 7 = 3$, etc.)

```
Step 1: Insert 10
  h(10) = 10 mod 7 = 3
  Probe 0: slot 3 → EMPTY → insert here

  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │     │     │     │ 10  │     │     │     │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    0     1     2     3     4     5     6

Step 2: Insert 17
  h(17) = 17 mod 7 = 3
  Probe 0: slot 3 → OCCUPIED (10)
  Probe 1: slot 4 → EMPTY → insert here

  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │     │     │     │ 10  │ 17  │     │     │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    0     1     2     3     4     5     6

Step 3: Insert 24
  h(24) = 24 mod 7 = 3
  Probe 0: slot 3 → OCCUPIED (10)
  Probe 1: slot 4 → OCCUPIED (17)
  Probe 2: slot 5 → EMPTY → insert here

  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │     │     │     │ 10  │ 17  │ 24  │     │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    0     1     2     3     4     5     6

Step 4: Insert 31
  h(31) = 31 mod 7 = 3
  Probe 0: slot 3 → OCCUPIED (10)
  Probe 1: slot 4 → OCCUPIED (17)
  Probe 2: slot 5 → OCCUPIED (24)
  Probe 3: slot 6 → EMPTY → insert here

  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │     │     │     │ 10  │ 17  │ 24  │ 31  │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    0     1     2     3     4     5     6

Step 5: Insert 38
  h(38) = 38 mod 7 = 3
  Probe 0: slot 3 → OCCUPIED
  Probe 1: slot 4 → OCCUPIED
  Probe 2: slot 5 → OCCUPIED
  Probe 3: slot 6 → OCCUPIED
  Probe 4: slot (3+4) mod 7 = 0 → EMPTY → insert here  ← WRAPS AROUND!

  ┌─────┬─────┬─────┬─────┬─────┬─────┬─────┐
  │ 38  │     │     │ 10  │ 17  │ 24  │ 31  │
  └─────┴─────┴─────┴─────┴─────┴─────┴─────┘
    0     1     2     3     4     5     6
```

---

## Search Trace

**Search for key 24** in the final table above:

```
h(24) = 24 mod 7 = 3
Probe 0: slot 3 → has 10 → not 24 → continue
Probe 1: slot 4 → has 17 → not 24 → continue
Probe 2: slot 5 → has 24 → FOUND! ✓

Total probes: 3
```

**Search for key 25** (not in table):

```
h(25) = 25 mod 7 = 4
Probe 0: slot 4 → has 17 → not 25 → continue
Probe 1: slot 5 → has 24 → not 25 → continue
Probe 2: slot 6 → has 31 → not 25 → continue
Probe 3: slot 0 → has 38 → not 25 → continue
Probe 4: slot 1 → EMPTY → NOT FOUND ✗

Total probes: 5  (longer due to cluster!)
```

---

## Primary Clustering

```
╔═══════════════════════════════════════════════════════════════╗
║                   PRIMARY CLUSTERING                         ║
║                                                              ║
║   Clusters grow and MERGE, making the problem worse:        ║
║                                                              ║
║   Initially:  [A][ ][ ][ ][B][C][ ][ ][ ][D]               ║
║                ▬▬           ▬▬▬▬▬           ▬▬               ║
║                cluster 1   cluster 2     cluster 3           ║
║                                                              ║
║   After more inserts, clusters grow:                         ║
║                                                              ║
║               [A][X][ ][ ][B][C][Y][ ][ ][D]               ║
║                ▬▬▬▬▬        ▬▬▬▬▬▬▬▬▬     ▬▬                ║
║                                                              ║
║   Eventually they merge into ONE BIG cluster:               ║
║                                                              ║
║               [A][X][Z][W][B][C][Y][V][ ][D]               ║
║                ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬               ║
║                    One massive cluster!                       ║
║                                                              ║
║   Problem: Any new key hashing ANYWHERE in the cluster      ║
║   extends it further. Larger clusters attract more keys.    ║
╚═══════════════════════════════════════════════════════════════╝
```

### Why Clustering Happens

A cluster of size $s$ has probability $\frac{s+1}{m}$ of being extended (by a key hashing to any position in the cluster OR one position before it). Larger clusters are **more likely** to grow:

```
Cluster size 1:  probability 2/m of growing
Cluster size 5:  probability 6/m of growing  ← 3× more likely!
Cluster size 20: probability 21/m of growing ← 10.5× more likely!
```

---

## Performance Impact

Expected number of probes with load factor $\alpha$:

| Operation | Formula | $\alpha = 0.5$ | $\alpha = 0.7$ | $\alpha = 0.9$ |
|-----------|---------|:---:|:---:|:---:|
| Unsuccessful search | $\frac{1}{2}\left(1 + \frac{1}{(1-\alpha)^2}\right)$ | 2.5 | 6.1 | 50.5 |
| Successful search | $\frac{1}{2}\left(1 + \frac{1}{1-\alpha}\right)$ | 1.5 | 2.2 | 5.5 |

```
  Probes
    │
 50 ┤                                          ╱ unsuccessful
    │                                        ╱
 40 ┤                                      ╱
    │                                    ╱
 30 ┤                                  ╱
    │                                ╱
 20 ┤                              ╱
    │                            ╱
 10 ┤                          ╱
    │          ╱═══════════════   successful
  1 ┤═══════╱══════════════════════════════
    └──┬──────┬──────┬──────┬──────┬──────
      0.0   0.2    0.4    0.6    0.8   1.0
                   Load Factor α
```

---

## Implementation

```
CLASS LinearProbingHashTable:
    size = 0
    capacity = 16
    keys[]   = new Array(capacity)  // null = empty
    values[] = new Array(capacity)
    DELETED_MARKER = special sentinel

    FUNCTION hash(key):
        RETURN key.hashCode() mod capacity

    FUNCTION put(key, value):
        IF (size + tombstoneCount) >= capacity * 0.5:
            resize(capacity * 2)

        index = hash(key)
        firstTombstone = -1

        WHILE true:
            IF keys[index] is null:
                // Use earlier tombstone if found
                IF firstTombstone != -1:
                    index = firstTombstone
                keys[index] = key
                values[index] = value
                size++
                RETURN

            IF keys[index] == DELETED_MARKER:
                IF firstTombstone == -1:
                    firstTombstone = index

            ELSE IF keys[index] == key:
                values[index] = value  // Update existing
                RETURN

            index = (index + 1) mod capacity

    FUNCTION get(key):
        index = hash(key)
        WHILE keys[index] is not null:
            IF keys[index] != DELETED_MARKER AND keys[index] == key:
                RETURN values[index]
            index = (index + 1) mod capacity
        RETURN null

    FUNCTION remove(key):
        index = hash(key)
        WHILE keys[index] is not null:
            IF keys[index] != DELETED_MARKER AND keys[index] == key:
                keys[index] = DELETED_MARKER
                values[index] = null
                size--
                RETURN true
            index = (index + 1) mod capacity
        RETURN false
```

---

## Cache Friendliness

Linear probing's biggest advantage — sequential memory access:

```
╔═════════════════════════════════════════════════════╗
║            CACHE LINE ACCESS PATTERN               ║
║                                                     ║
║  CPU Cache Line (64 bytes, ~8 entries):             ║
║  ┌─────────────────────────────────────┐            ║
║  │ slot₀ │ slot₁ │ slot₂ │ ... │ slot₇│            ║
║  └─────────────────────────────────────┘            ║
║  ▲                                                  ║
║  One cache load fetches ~8 consecutive slots        ║
║                                                     ║
║  Linear probing: 1 cache miss for ~8 probes        ║
║  Double hashing: 1 cache miss PER probe            ║
║  Chaining:       1 cache miss PER node (pointers)  ║
╚═════════════════════════════════════════════════════╝
```

This makes linear probing **fastest in practice** at low load factors despite the clustering issue.

---

## Quick Revision Question

**Q: What is primary clustering, and why does it get progressively worse in linear probing?**

<details>
<summary>Click to reveal answer</summary>

**Primary clustering** occurs when occupied slots form contiguous blocks (clusters) in the hash table. In linear probing, when a collision occurs, the key is placed in the next available consecutive slot, which extends or merges clusters.

**Why it gets progressively worse**: A cluster of size $s$ has probability $\frac{s+1}{m}$ of being extended — any key hashing into the cluster OR to the slot just before it will extend it. Larger clusters therefore **attract more keys**, growing disproportionately faster than small clusters. When two clusters are separated by a single empty slot, filling that slot merges them into one even larger cluster, creating a positive feedback loop.

At high load factors (above 0.7), most of the table becomes one or two massive clusters, causing probe counts to grow dramatically.

</details>

---

| [← Previous: Open Addressing Concept](01-open-addressing-concept.md) | [Next: Quadratic Probing →](03-quadratic-probing.md) |
|:---|---:|
