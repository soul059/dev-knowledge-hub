# 4.3 Quadratic Probing

## Unit 4: Collision Handling - Open Addressing

---

## Quadratic Probing Formula

$$h(k, i) = (h(k) + c_1 \cdot i + c_2 \cdot i^2) \bmod m$$

Common simplified form (with $c_1 = 0$, $c_2 = 1$):

$$h(k, i) = (h(k) + i^2) \bmod m$$

The probe sequence jumps by **increasing squares**:

```
Probe 0:  h(k)
Probe 1:  h(k) + 1
Probe 2:  h(k) + 4
Probe 3:  h(k) + 9
Probe 4:  h(k) + 16
Probe 5:  h(k) + 25
...all mod m
```

---

## How Quadratic Probing Breaks Clusters

```
╔══════════════════════════════════════════════════════════╗
║         LINEAR  vs  QUADRATIC  PROBING                  ║
║                                                          ║
║  Cluster:  [A][B][C][D][E][ ][ ][ ][ ][ ][ ]           ║
║             0  1  2  3  4  5  6  7  8  9  10            ║
║                                                          ║
║  New key K with h(K) = 0:                               ║
║                                                          ║
║  LINEAR:     probe 0→1→2→3→4→5 (slot 5)               ║
║              Extends the cluster!                        ║
║              ▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬▬                          ║
║                                                          ║
║  QUADRATIC:  probe 0→1→4→9 (slot 9)                   ║
║              Jumps OVER the cluster!                     ║
║              ▬▬▬▬▬▬▬▬▬▬  ← cluster NOT extended       ║
╚══════════════════════════════════════════════════════════╝
```

---

## Step-by-Step Insertion Trace

**Setup**: Table size $m = 11$, hash function $h(k) = k \bmod 11$, probe $h(k,i) = (h(k) + i^2) \bmod 11$

**Insert keys: 22, 33, 44, 55, 77**

All hash to index 0: $22 \bmod 11 = 0$, $33 \bmod 11 = 0$, etc.

```
Step 1: Insert 22
  h(22,0) = (0 + 0) mod 11 = 0 → EMPTY → insert

  ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
  │ 22 │    │    │    │    │    │    │    │    │    │    │
  └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
    0    1    2    3    4    5    6    7    8    9    10

Step 2: Insert 33
  h(33,0) = (0 + 0) mod 11 = 0  → OCCUPIED (22)
  h(33,1) = (0 + 1) mod 11 = 1  → EMPTY → insert

  ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
  │ 22 │ 33 │    │    │    │    │    │    │    │    │    │
  └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
    0    1    2    3    4    5    6    7    8    9    10

Step 3: Insert 44
  h(44,0) = (0 + 0)  mod 11 = 0  → OCCUPIED (22)
  h(44,1) = (0 + 1)  mod 11 = 1  → OCCUPIED (33)
  h(44,2) = (0 + 4)  mod 11 = 4  → EMPTY → insert  ← jumped to 4!

  ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
  │ 22 │ 33 │    │    │ 44 │    │    │    │    │    │    │
  └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
    0    1    2    3    4    5    6    7    8    9    10

Step 4: Insert 55
  h(55,0) = (0 + 0)  mod 11 = 0  → OCCUPIED (22)
  h(55,1) = (0 + 1)  mod 11 = 1  → OCCUPIED (33)
  h(55,2) = (0 + 4)  mod 11 = 4  → OCCUPIED (44)
  h(55,3) = (0 + 9)  mod 11 = 9  → EMPTY → insert  ← jumped to 9!

  ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
  │ 22 │ 33 │    │    │ 44 │    │    │    │    │ 55 │    │
  └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
    0    1    2    3    4    5    6    7    8    9    10

Step 5: Insert 77
  h(77,0) = (0 + 0)  mod 11 = 0   → OCCUPIED (22)
  h(77,1) = (0 + 1)  mod 11 = 1   → OCCUPIED (33)
  h(77,2) = (0 + 4)  mod 11 = 4   → OCCUPIED (44)
  h(77,3) = (0 + 9)  mod 11 = 9   → OCCUPIED (55)
  h(77,4) = (0 + 16) mod 11 = 5   → EMPTY → insert

  ┌────┬────┬────┬────┬────┬────┬────┬────┬────┬────┬────┐
  │ 22 │ 33 │    │    │ 44 │ 77 │    │    │    │ 55 │    │
  └────┴────┴────┴────┴────┴────┴────┴────┴────┴────┴────┘
    0    1    2    3    4    5    6    7    8    9    10

  Notice: Keys are SPREAD OUT (0, 1, 4, 9, 5)
  Compare with linear probing: they'd be at (0, 1, 2, 3, 4)
```

---

## Secondary Clustering

```
╔══════════════════════════════════════════════════════════╗
║              SECONDARY CLUSTERING                        ║
║                                                          ║
║  Problem: Keys that hash to the SAME initial slot        ║
║  follow the EXACT SAME probe sequence.                   ║
║                                                          ║
║  If h(A) = h(B) = 5:                                    ║
║    A's probes:  5, 6, 9, 3, 10, 8, ...                  ║
║    B's probes:  5, 6, 9, 3, 10, 8, ...  ← identical!   ║
║                                                          ║
║  This is LESS severe than primary clustering:            ║
║  - Only affects keys with same hash value                ║
║  - Primary clustering affects ALL nearby keys            ║
║                                                          ║
║  But still causes degraded performance compared to       ║
║  double hashing, which gives each key a unique sequence. ║
╚══════════════════════════════════════════════════════════╝
```

---

## Coverage Guarantee

**Not all table slots may be reachable!** The probe sequence $\{i^2 \bmod m\}$ doesn't always visit every slot.

```
Example with m = 8:
  i:    0  1  2  3  4  5  6  7
  i²:   0  1  4  9  16 25 36 49
  mod8: 0  1  4  1  0  1  4  1

  Only visits slots: {0, 1, 4}  ← misses 2, 3, 5, 6, 7!
```

### Guarantee conditions:

| Condition | Coverage |
|-----------|----------|
| $m$ is prime | At least $\lfloor m/2 \rfloor$ slots visited |
| $m$ is prime AND $\alpha < 0.5$ | Insertion always succeeds |
| $m = 2^k$, use $h(k,i) = h(k) + \frac{i(i+1)}{2}$ | All slots visited |

```
Example with m = 11 (prime):
  i:     0  1  2  3  4   5   6   7   8   9   10
  i²:    0  1  4  9  16  25  36  49  64  81  100
  mod11: 0  1  4  9  5   3   3   5   9   4   1

  Visits: {0, 1, 3, 4, 5, 9} ← 6 out of 11 = ⌊11/2⌋ + 1 ✓
```

---

## Pseudocode

```
CLASS QuadraticProbingHashTable:
    capacity = 11     // Prime number
    size = 0
    table[] = new Array(capacity)  // null entries
    DELETED = sentinel

    FUNCTION probe(key, i):
        RETURN (hash(key) + i * i) mod capacity

    FUNCTION put(key, value):
        IF size >= capacity / 2:    // Keep α < 0.5
            resize(nextPrime(capacity * 2))

        FOR i = 0 TO capacity - 1:
            index = probe(key, i)
            IF table[index] is null OR table[index] is DELETED:
                table[index] = Entry(key, value)
                size++
                RETURN
            IF table[index].key == key:
                table[index].value = value    // Update
                RETURN

    FUNCTION get(key):
        FOR i = 0 TO capacity - 1:
            index = probe(key, i)
            IF table[index] is null:
                RETURN null
            IF table[index] is not DELETED AND table[index].key == key:
                RETURN table[index].value
        RETURN null

    FUNCTION nextPrime(n):
        WHILE not isPrime(n):
            n++
        RETURN n
```

---

## Quick Revision Question

**Q: What is secondary clustering, and how does it differ from primary clustering? Which probing method is affected by each?**

<details>
<summary>Click to reveal answer</summary>

**Primary clustering** (linear probing): Contiguous blocks of occupied slots form and grow. ANY key hashing to ANY slot within or adjacent to the cluster extends it. Affects keys with *different* hash values.

**Secondary clustering** (quadratic probing): Keys that hash to the *same* initial slot follow the *exact same* probe sequence. Only keys with identical hash values compete, but they all step on each other's paths.

| Feature | Primary | Secondary |
|---------|---------|-----------|
| Probing method | Linear | Quadratic |
| Affected keys | All nearby keys | Only same-hash keys |
| Severity | More severe | Less severe |
| Solution | Quadratic/double hashing | Double hashing |

Double hashing eliminates both types of clustering because each key gets a unique probe sequence determined by *two* independent hash functions.

</details>

---

| [← Previous: Linear Probing](02-linear-probing.md) | [Next: Double Hashing →](04-double-hashing.md) |
|:---|---:|
