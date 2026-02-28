# 4.5 Clustering Problems

## Unit 4: Collision Handling - Open Addressing

---

## What is Clustering?

Clustering occurs when occupied slots **group together** in a hash table, forcing keys to probe through long chains of occupied slots before finding an empty one.

```
╔══════════════════════════════════════════════════════════╗
║                  IDEAL vs CLUSTERED                      ║
║                                                          ║
║  IDEAL (uniform):                                        ║
║  [A][ ][B][ ][C][ ][D][ ][E][ ][F][ ][G][ ]           ║
║  Max probe length: 1                                     ║
║                                                          ║
║  CLUSTERED:                                              ║
║  [A][B][C][D][E][F][G][ ][ ][ ][ ][ ][ ][ ]           ║
║  ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬                                 ║
║  Max probe length: 7                                     ║
║                                                          ║
║  Both have 7 keys and 14 slots (same load factor)       ║
║  but vastly different performance!                       ║
╚══════════════════════════════════════════════════════════╝
```

---

## Primary Clustering (Linear Probing)

### Definition

Primary clustering occurs when **any key** that hashes to **any slot within or adjacent to a cluster** extends the cluster.

```
╔══════════════════════════════════════════════════════════════╗
║              PRIMARY CLUSTERING — GROWTH                     ║
║                                                              ║
║  Cluster at slots 3-6:                                       ║
║  [ ][ ][ ][A][B][C][D][ ][ ][ ]                            ║
║   0  1  2  3  4  5  6  7  8  9                              ║
║            ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬                                  ║
║                                                              ║
║  Keys that EXTEND this cluster:                              ║
║  • h(K) = 3 → probes 3,4,5,6,7 → lands at 7 (extends!)    ║
║  • h(K) = 4 → probes 4,5,6,7   → lands at 7 (extends!)    ║
║  • h(K) = 5 → probes 5,6,7     → lands at 7 (extends!)    ║
║  • h(K) = 6 → probes 6,7       → lands at 7 (extends!)    ║
║  • h(K) = 7 → probes 7         → lands at 7 (extends!)    ║
║                                                              ║
║  That's 5 out of 10 possible hash values extend it!         ║
║  Probability = (cluster_size + 1) / m = 5/10 = 50%         ║
╚══════════════════════════════════════════════════════════════╝
```

### Cluster Merging

```
Before: Two separate clusters

  [A][B][ ][ ][C][D][E][ ][ ][ ]
   0  1  2  3  4  5  6  7  8  9
   ▬▬▬▬▬        ▬▬▬▬▬▬▬▬▬

Insert key with h(K) = 1:
  Probes: 1(taken), 2(empty) → inserted at 2

  [A][B][K][ ][C][D][E][ ][ ][ ]
   0  1  2  3  4  5  6  7  8  9
   ▬▬▬▬▬▬▬▬▬

Insert key with h(K) = 2:
  Probes: 2(taken), 3(empty) → inserted at 3

  [A][B][K][X][C][D][E][ ][ ][ ]
   0  1  2  3  4  5  6  7  8  9
   ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬

  Two clusters MERGED into one giant cluster!
  Size went from 2 + 3 = 5 to 7
```

### Growth Feedback Loop

```
  Cluster size: 1 → P(grow) = 2/m     ─┐
  Cluster size: 2 → P(grow) = 3/m      │
  Cluster size: 5 → P(grow) = 6/m      │  Positive
  Cluster size: 10 → P(grow) = 11/m    │  Feedback
  Cluster size: 20 → P(grow) = 21/m    │  Loop!
  Cluster size: 50 → P(grow) = 51/m   ─┘

  Bigger clusters grow FASTER → dominate the table
```

---

## Secondary Clustering (Quadratic Probing)

### Definition

Secondary clustering occurs when keys with the **same hash value** follow the **exact same probe sequence**.

```
╔══════════════════════════════════════════════════════════════╗
║              SECONDARY CLUSTERING                            ║
║                                                              ║
║  h(k,i) = (h(k) + i²) mod 11                               ║
║                                                              ║
║  All keys with h(k) = 3:                                    ║
║                                                              ║
║  Key A: 3, 4, 7, 1, 8, 6, ...   (3+0, 3+1, 3+4, 3+9...)  ║
║  Key B: 3, 4, 7, 1, 8, 6, ...   ← SAME sequence!          ║
║  Key C: 3, 4, 7, 1, 8, 6, ...   ← SAME sequence!          ║
║                                                              ║
║  But keys with h(k) = 5 get a DIFFERENT sequence:           ║
║  Key D: 5, 6, 9, 3, 10, 8, ...                             ║
║                                                              ║
║  Only keys with the same initial hash collide repeatedly.   ║
╚══════════════════════════════════════════════════════════════╝
```

### Why It's Less Severe

```
PRIMARY CLUSTERING:
  Cluster [3][4][5][6] affects keys hashing to 3, 4, 5, 6, or 7
  → 5 out of m hash values cause extension

SECONDARY CLUSTERING:
  Only keys where h(k) = 3 follow each other's path
  → 1 out of m hash values causes this specific conflict

  Impact ratio: 5:1 in favor of secondary being milder
```

---

## No Clustering (Double Hashing)

```
╔══════════════════════════════════════════════════════════════╗
║              DOUBLE HASHING — NO CLUSTERING                  ║
║                                                              ║
║  h(k,i) = (h₁(k) + i · h₂(k)) mod m                       ║
║                                                              ║
║  Even if h₁(A) = h₁(B) = 3:                                ║
║                                                              ║
║  Key A: h₂(A) = 2 → sequence: 3, 5, 7, 9, 0, 2, ...       ║
║  Key B: h₂(B) = 5 → sequence: 3, 8, 0, 5, 10, 2, ...      ║
║                                                              ║
║  After the first collision, paths DIVERGE!                   ║
║                                                              ║
║  Each key effectively has its own private probe sequence.    ║
║  Approximates "uniform hashing" — the theoretical ideal.     ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Visual Comparison: 50% Load Factor

```
  LINEAR PROBING (primary clustering):
  [A][B][C][ ][ ][ ][D][E][F][G][ ][ ][ ][ ][H][ ]
  ▬▬▬▬▬▬▬▬▬              ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬
  Big clusters form even at α = 0.5

  QUADRATIC PROBING (secondary clustering):
  [A][ ][B][C][ ][D][ ][ ][E][F][ ][G][ ][ ][H][ ]
  Better spread, but some grouping

  DOUBLE HASHING (no clustering):
  [A][ ][B][ ][C][ ][D][ ][E][ ][F][ ][G][ ][H][ ]
  Near-uniform distribution
```

---

## Performance Comparison at Various Load Factors

### Unsuccessful Search (Expected Probes)

| $\alpha$ | Linear | Quadratic | Double Hashing |
|:---:|:---:|:---:|:---:|
| 0.10 | 1.1 | 1.1 | 1.1 |
| 0.25 | 1.4 | 1.3 | 1.3 |
| 0.50 | 2.5 | 2.2 | 2.0 |
| 0.75 | 8.5 | 4.6 | 4.0 |
| 0.90 | 50.5 | 11.4 | 10.0 |
| 0.95 | 200.5 | 22.0 | 20.0 |

```
  Probes
    │
200 ┤                     ╱ LINEAR
    │                   ╱
150 ┤                 ╱
    │               ╱
100 ┤             ╱
    │           ╱
 50 ┤         ╱
    │       ╱
 20 ┤     ╱         ╱──── QUADRATIC
    │   ╱         ╱ ╱──── DOUBLE
 10 ┤ ╱       ╱─╱
    │╱    ╱─╱
  1 ┤═══╱═══════════════════════
    └──┬──────┬──────┬──────┬──
      0.0   0.25   0.50   0.75   0.95
                   Load Factor α
```

---

## Real-World Mitigations

```
╔══════════════════════════════════════════════════════════════╗
║            STRATEGIES TO MINIMIZE CLUSTERING                 ║
║                                                              ║
║  1. KEEP α LOW                                               ║
║     Linear probing: α < 0.5 → avg 2.5 probes               ║
║     Quadratic:      α < 0.5 → insertion guaranteed          ║
║     Double hashing: α < 0.7 → still fast                    ║
║                                                              ║
║  2. USE GOOD HASH FUNCTIONS                                  ║
║     Poor hash → many same-slot collisions → clustering      ║
║     Bit-mixing (murmur, xxhash) → uniform spread            ║
║                                                              ║
║  3. RESIZE PROACTIVELY                                       ║
║     Don't wait until table is full                           ║
║     Resize at 50-70% capacity                               ║
║                                                              ║
║  4. ROBIN HOOD HASHING (advanced)                            ║
║     When inserting, if probe length of new key >             ║
║     probe length of existing key, SWAP them.                 ║
║     Equalizes probe lengths across all keys!                ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Quick Revision Question

**Q: Three keys all hash to slot 5 in a table of size 13. Show the probe sequence each key follows under (a) linear probing, (b) quadratic probing, and (c) double hashing with $h_2$ values of 3, 7, and 4. Which method produces clustering?**

<details>
<summary>Click to reveal answer</summary>

**(a) Linear probing** — all follow the same sequence:
- Key 1: 5, 6, 7, 8, 9, 10, 11, 12, 0, 1, 2, 3, 4
- Key 2: 5, 6, 7, 8, 9, ... (identical!)
- Key 3: 5, 6, 7, 8, 9, ... (identical!)
- **Primary clustering**: forms a contiguous block 5, 6, 7.

**(b) Quadratic probing** — all follow the same sequence:
- Key 1: 5, 6, 9, 3, 12, 10, ...
- Key 2: 5, 6, 9, 3, 12, 10, ... (identical!)
- Key 3: 5, 6, 9, 3, 12, 10, ... (identical!)
- **Secondary clustering**: same probe path for all three.

**(c) Double hashing** — each gets a unique sequence:
- Key 1 ($h_2 = 3$): 5, 8, 11, 1, 4, 7, 10, 0, 3, 6, 9, 12, 2
- Key 2 ($h_2 = 7$): 5, 12, 6, 0, 7, 1, 8, 2, 9, 3, 10, 4, 11
- Key 3 ($h_2 = 4$): 5, 9, 0, 4, 8, 12, 3, 7, 11, 2, 6, 10, 1
- **No clustering**: paths diverge after first collision.

</details>

---

| [← Previous: Double Hashing](04-double-hashing.md) | [Next: Comparison of Methods →](06-comparison-of-methods.md) |
|:---|---:|
