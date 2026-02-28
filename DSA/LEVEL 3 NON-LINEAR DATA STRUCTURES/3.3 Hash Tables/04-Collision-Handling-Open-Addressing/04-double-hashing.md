# 4.4 Double Hashing

## Unit 4: Collision Handling - Open Addressing

---

## Double Hashing Formula

$$h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod m$$

Where:
- $h_1(k)$ = primary hash (determines initial slot)
- $h_2(k)$ = secondary hash (determines probe step size)
- $i$ = probe number (0, 1, 2, ...)

```
╔══════════════════════════════════════════════════════════╗
║           WHY DOUBLE HASHING IS SUPERIOR                 ║
║                                                          ║
║  Each key gets a UNIQUE probe sequence based on          ║
║  TWO independent hash functions:                         ║
║                                                          ║
║  Key A: h₁(A)=3, h₂(A)=2 → probes: 3, 5, 7, 9, 0...  ║
║  Key B: h₁(B)=3, h₂(B)=5 → probes: 3, 8, 2, 7, 1...  ║
║                                                          ║
║  Same initial slot (3) but DIFFERENT sequences!          ║
║  This eliminates BOTH primary AND secondary clustering.  ║
╚══════════════════════════════════════════════════════════╝
```

---

## Choosing h₂(k)

**Requirements for h₂(k)**:
1. $h_2(k) \neq 0$ (otherwise no probing occurs)
2. $h_2(k)$ must be **relatively prime to $m$** (to visit all slots)

### Common choices:

| Table size $m$ | $h_2(k)$ formula | Guarantee |
|-----------|------------|-----------|
| Prime $m$ | $1 + (k \bmod (m - 1))$ | Always visits all $m$ slots |
| Prime $m$ | $q - (k \bmod q)$ where $q < m$ is prime | Visits all slots |
| Power of 2 | Always return odd number | Visits all slots |

```
Most common pair:
  h₁(k) = k mod m           (m is prime)
  h₂(k) = 1 + (k mod (m-1))

Example with m = 13:
  h₁(k) = k mod 13
  h₂(k) = 1 + (k mod 12)    ← always 1..12, never 0
```

---

## Step-by-Step Trace

**Setup**: $m = 13$, $h_1(k) = k \bmod 13$, $h_2(k) = 1 + (k \bmod 12)$

**Insert keys: 14, 27, 40, 53**

All have $h_1(k) = k \bmod 13 = 1$, but **different step sizes**!

```
Step sizes:
  h₂(14) = 1 + (14 mod 12) = 1 + 2 = 3
  h₂(27) = 1 + (27 mod 12) = 1 + 3 = 4
  h₂(40) = 1 + (40 mod 12) = 1 + 4 = 5
  h₂(53) = 1 + (53 mod 12) = 1 + 5 = 6
```

```
Step 1: Insert 14 (h₁=1, h₂=3)
  Probe 0: (1 + 0×3) mod 13 = 1 → EMPTY → insert

  Index: 0    1    2    3    4    5    6    7    8    9   10   11   12
        [   ][14 ][   ][   ][   ][   ][   ][   ][   ][   ][   ][   ][   ]

Step 2: Insert 27 (h₁=1, h₂=4)
  Probe 0: (1 + 0×4) mod 13 = 1 → OCCUPIED (14)
  Probe 1: (1 + 1×4) mod 13 = 5 → EMPTY → insert

  Index: 0    1    2    3    4    5    6    7    8    9   10   11   12
        [   ][14 ][   ][   ][   ][27 ][   ][   ][   ][   ][   ][   ][   ]

Step 3: Insert 40 (h₁=1, h₂=5)
  Probe 0: (1 + 0×5) mod 13 = 1  → OCCUPIED (14)
  Probe 1: (1 + 1×5) mod 13 = 6  → EMPTY → insert

  Index: 0    1    2    3    4    5    6    7    8    9   10   11   12
        [   ][14 ][   ][   ][   ][27 ][40 ][   ][   ][   ][   ][   ][   ]

Step 4: Insert 53 (h₁=1, h₂=6)
  Probe 0: (1 + 0×6) mod 13 = 1  → OCCUPIED (14)
  Probe 1: (1 + 1×6) mod 13 = 7  → EMPTY → insert

  Index: 0    1    2    3    4    5    6    7    8    9   10   11   12
        [   ][14 ][   ][   ][   ][27 ][40 ][53 ][   ][   ][   ][   ][   ]

  Results: Keys are SPREAD across slots 1, 5, 6, 7
  
  Compare — all four had h₁ = 1, but different h₂ values
  sent them to different slots with only 1-2 probes each!
```

---

## Contrast with Linear and Quadratic Probing

Same scenario ($h_1 = 1$ for all four keys):

```
LINEAR PROBING:
  14 → slot 1
  27 → slot 2  (probe past 1)
  40 → slot 3  (probe past 1,2)
  53 → slot 4  (probe past 1,2,3)
  Probes: 1, 2, 3, 4 = 10 total
  Creates cluster: [14][27][40][53]

QUADRATIC PROBING:
  14 → slot 1         (probe 0)
  27 → slot 2         (probe 1: 1+1=2)
  40 → slot 5         (probe 2: 1+4=5)
  53 → slot 10        (probe 3: 1+9=10)
  Probes: 1, 2, 3, 4 = 10 total
  Same number! But keys follow same sequence → secondary clustering

DOUBLE HASHING:
  14 → slot 1  (probe 0)
  27 → slot 5  (probe 1, step=4)
  40 → slot 6  (probe 1, step=5)
  53 → slot 7  (probe 1, step=6)
  Probes: 1, 2, 2, 2 = 7 total  ← FEWEST!
  No clustering at all!
```

---

## Performance Analysis

Expected probes under **uniform hashing** assumption:

| Operation | Formula | $\alpha = 0.5$ | $\alpha = 0.7$ | $\alpha = 0.9$ |
|-----------|---------|:---:|:---:|:---:|
| Unsuccessful search | $\frac{1}{1-\alpha}$ | 2.0 | 3.3 | 10.0 |
| Successful search | $\frac{1}{\alpha}\ln\frac{1}{1-\alpha}$ | 1.4 | 1.7 | 2.6 |

Compare with linear probing at $\alpha = 0.9$: unsuccessful = 50.5 vs double hashing = 10.0!

---

## Implementation

```
CLASS DoubleHashingTable:
    capacity = 13   // Prime
    size = 0
    keys[]   = new Array(capacity)
    values[] = new Array(capacity)
    DELETED  = sentinel

    FUNCTION h1(key):
        RETURN key.hashCode() mod capacity

    FUNCTION h2(key):
        // Must never return 0
        RETURN 1 + (key.hashCode() mod (capacity - 1))

    FUNCTION probe(key, i):
        RETURN (h1(key) + i * h2(key)) mod capacity

    FUNCTION put(key, value):
        IF size >= capacity * 0.6:
            resize(nextPrime(capacity * 2))

        FOR i = 0 TO capacity - 1:
            index = probe(key, i)
            IF keys[index] is null OR keys[index] is DELETED:
                keys[index] = key
                values[index] = value
                size++
                RETURN
            IF keys[index] == key:
                values[index] = value    // Update
                RETURN

    FUNCTION get(key):
        FOR i = 0 TO capacity - 1:
            index = probe(key, i)
            IF keys[index] is null:
                RETURN null
            IF keys[index] != DELETED AND keys[index] == key:
                RETURN values[index]
        RETURN null

    FUNCTION remove(key):
        FOR i = 0 TO capacity - 1:
            index = probe(key, i)
            IF keys[index] is null:
                RETURN false
            IF keys[index] != DELETED AND keys[index] == key:
                keys[index] = DELETED
                values[index] = null
                size--
                RETURN true
        RETURN false
```

---

## Quick Revision Question

**Q: In double hashing, why must $h_2(k)$ never return 0, and why must it be relatively prime to the table size $m$?**

<details>
<summary>Click to reveal answer</summary>

**Why $h_2(k) \neq 0$**: If $h_2(k) = 0$, the probe sequence becomes $h_1(k), h_1(k), h_1(k), \ldots$ — the same slot forever. The probing gets stuck and never finds an alternative slot.

**Why relatively prime to $m$**: The step size $h_2(k)$ and table size $m$ must be coprime so the probe sequence visits **all $m$ slots** before cycling. If $\gcd(h_2(k), m) = d > 1$, the sequence only visits $m/d$ distinct slots, potentially missing empty slots even when the table isn't full.

**Practical solution**: Make $m$ prime, and define $h_2(k) = 1 + (k \bmod (m-1))$. Since $m$ is prime and $h_2$ ranges from 1 to $m-1$, every possible step size is coprime to $m$.

</details>

---

| [← Previous: Quadratic Probing](03-quadratic-probing.md) | [Next: Clustering Problems →](05-clustering-problems.md) |
|:---|---:|
