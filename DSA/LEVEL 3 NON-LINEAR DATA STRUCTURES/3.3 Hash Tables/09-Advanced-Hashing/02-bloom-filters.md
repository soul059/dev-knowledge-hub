# 9.2 Bloom Filters

## Unit 9: Advanced Hashing

---

## What is a Bloom Filter?

A **space-efficient probabilistic** data structure that answers membership queries:

- **"Is X in the set?"**
- Answer: **"Definitely NO"** or **"Probably YES"**

```
╔══════════════════════════════════════════════════════════════╗
║  BLOOM FILTER GUARANTEES:                                   ║
║                                                              ║
║  ┌────────────────────────────┬─────────────────────┐      ║
║  │ If answer is NO            │ Element is DEFINITELY│      ║
║  │                            │ NOT in the set       │      ║
║  ├────────────────────────────┼─────────────────────┤      ║
║  │ If answer is YES           │ Element is PROBABLY  │      ║
║  │                            │ in the set (might be │      ║
║  │                            │ a false positive)    │      ║
║  └────────────────────────────┴─────────────────────┘      ║
║                                                              ║
║  False positives: POSSIBLE (controllable)                   ║
║  False negatives: IMPOSSIBLE                                ║
║  Deletion:        NOT supported (standard version)          ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Structure

A bit array of $m$ bits, all initially 0, with $k$ independent hash functions.

```
╔══════════════════════════════════════════════════════════════╗
║  Bit array (m=16):                                          ║
║  Index: 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15  ║
║  Bits:  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0  0   ║
║                                                              ║
║  k=3 hash functions: h₁, h₂, h₃                           ║
║                                                              ║
║  INSERT "apple":                                            ║
║    h₁("apple") = 1                                          ║
║    h₂("apple") = 5                                          ║
║    h₃("apple") = 13                                         ║
║    Set bits 1, 5, 13 to 1                                   ║
║                                                              ║
║  Index: 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15  ║
║  Bits:  0  1  0  0  0  1  0  0  0  0  0  0  0  1  0  0   ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Operations

### Insert

```
FUNCTION insert(element):
    FOR i = 1 TO k:
        index = hashᵢ(element) % m
        bits[index] = 1
```

### Query

```
FUNCTION mightContain(element):
    FOR i = 1 TO k:
        index = hashᵢ(element) % m
        IF bits[index] == 0:
            RETURN false    // DEFINITELY not in set
    RETURN true             // PROBABLY in set
```

---

## Complete Trace

```
m = 10, k = 3 hash functions

Bit array: [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]

INSERT "cat":
  h₁("cat")%10 = 1, h₂("cat")%10 = 4, h₃("cat")%10 = 7
  Bits: [0, 1, 0, 0, 1, 0, 0, 1, 0, 0]

INSERT "dog":
  h₁("dog")%10 = 3, h₂("dog")%10 = 7, h₃("dog")%10 = 9
  Bits: [0, 1, 0, 1, 1, 0, 0, 1, 0, 1]
                     ↑              ↑     ↑
                    new          shared   new

QUERY "cat":
  h₁ → bit[1]=1 ✓, h₂ → bit[4]=1 ✓, h₃ → bit[7]=1 ✓
  → PROBABLY YES (correct, "cat" was inserted)

QUERY "bird":
  h₁("bird")%10 = 1, h₂("bird")%10 = 4, h₃("bird")%10 = 9
  bit[1]=1 ✓, bit[4]=1 ✓, bit[9]=1 ✓
  → PROBABLY YES ← FALSE POSITIVE! "bird" was never inserted!
  (Bits happen to be set by "cat" and "dog" combined)

QUERY "fish":
  h₁("fish")%10 = 2, h₂("fish")%10 = 5, h₃("fish")%10 = 8
  bit[2]=0 ✗
  → DEFINITELY NO (correct, "fish" was never inserted)
  (Even one 0 bit is enough to reject)
```

---

## False Positive Probability

After inserting $n$ elements into an $m$-bit array with $k$ hash functions:

$$P_{\text{fp}} \approx \left(1 - e^{-kn/m}\right)^k$$

### Optimal Number of Hash Functions

$k_{\text{opt}} = \frac{m}{n} \ln 2 \approx 0.693 \cdot \frac{m}{n}$

```
╔══════════════════════════════════════════════════════════════╗
║  Example: n=1000 elements, desired FP rate < 1%            ║
║                                                              ║
║  m = -n·ln(p) / (ln2)²                                     ║
║    = -1000·ln(0.01) / (0.693)²                             ║
║    = -1000·(-4.605) / 0.480                                ║
║    = 9592 bits ≈ 1.2 KB                                    ║
║                                                              ║
║  k = (m/n)·ln2 = 9.592·0.693 ≈ 7 hash functions           ║
║                                                              ║
║  Compare: Storing 1000 elements as a HashSet:              ║
║    ~40-80 KB (40-80 bytes/element)                         ║
║                                                              ║
║  Bloom filter: 1.2 KB                                       ║
║  Space savings: 30-60x!                                     ║
╚══════════════════════════════════════════════════════════════╝
```

### FP Rate Table

| Bits per element ($m/n$) | Optimal $k$ | FP Rate |
|:---:|:---:|:---:|
| 4 | 3 | 14.7% |
| 8 | 6 | 2.2% |
| 10 | 7 | 0.82% |
| 16 | 11 | 0.046% |
| 20 | 14 | 0.006% |

---

## Why No Deletion?

```
╔══════════════════════════════════════════════════════════════╗
║  After inserting "cat" and "dog":                           ║
║  Bits: [0, 1, 0, 1, 1, 0, 0, 1, 0, 1]                    ║
║                                                              ║
║  DELETE "cat" → clear bits 1, 4, 7:                        ║
║  Bits: [0, 0, 0, 1, 0, 0, 0, 0, 0, 1]                    ║
║                                 ↑                           ║
║                  bit 7 was SHARED with "dog"!               ║
║                                                              ║
║  Now QUERY "dog":                                           ║
║    h₃ → bit[7] = 0 → DEFINITELY NO                        ║
║    But "dog" IS in the set! → FALSE NEGATIVE               ║
║                                                              ║
║  This violates the core guarantee of Bloom filters.        ║
║                                                              ║
║  Solution: COUNTING BLOOM FILTER                            ║
║    Use counters instead of bits                             ║
║    Insert: increment, Delete: decrement                    ║
║    Trade-off: 3-4x more space                              ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Real-World Applications

| Application | How Bloom Filter Helps |
|-------------|----------------------|
| Web crawlers | "Have I visited this URL before?" — avoid re-crawling |
| Databases (LSM trees) | "Is this key in this SSTable?" — skip disk reads |
| CDN caching | Check if content exists before fetching from origin |
| Spell checkers | Quick "is this a valid word?" test |
| Bitcoin SPV | Lightweight transaction verification |
| Network routing | Packet duplicate detection |

---

## Implementation in Practice

```
// Using double hashing to simulate k hash functions:
// hᵢ(x) = h₁(x) + i · h₂(x)    (Kirsch-Mitzenmacker)

CLASS BloomFilter:
    bits = new BitArray(m)

    FUNCTION insert(element):
        h1 = murmurHash(element)
        h2 = fnvHash(element)
        FOR i = 0 TO k - 1:
            index = (h1 + i * h2) % m
            bits.set(index)

    FUNCTION mightContain(element):
        h1 = murmurHash(element)
        h2 = fnvHash(element)
        FOR i = 0 TO k - 1:
            index = (h1 + i * h2) % m
            IF NOT bits.get(index):
                RETURN false
        RETURN true
```

---

## Quick Revision Question

**Q: A Bloom filter with $m = 10000$ bits and $k = 7$ hash functions has had $n = 1000$ elements inserted. Approximately what is the false positive rate? What happens to the FP rate as more elements are inserted?**

<details>
<summary>Click to reveal answer</summary>

Using the formula: $P_{\text{fp}} \approx (1 - e^{-kn/m})^k$

$$P_{\text{fp}} \approx \left(1 - e^{-7 \times 1000 / 10000}\right)^7 = \left(1 - e^{-0.7}\right)^7$$

$e^{-0.7} \approx 0.4966$

$$P_{\text{fp}} \approx (1 - 0.4966)^7 = (0.5034)^7 \approx 0.0082$$

**False positive rate ≈ 0.82%** (about 1 in 122 queries on non-existent elements gives a false positive).

**As more elements are inserted:**
- More bits get set to 1
- The probability that ALL $k$ bits for a random element are 1 increases
- FP rate increases **exponentially** with $n$:
  - $n = 2000$: FP ≈ 4.5%
  - $n = 5000$: FP ≈ 45%
  - $n = 10000$: FP ≈ 93%

The filter becomes "saturated" — almost all bits are 1, so almost every query returns positive. This is why sizing $m$ appropriately for the expected $n$ is critical.

</details>

---

| [← Consistent Hashing](01-consistent-hashing.md) | [Next: Count-Min Sketch →](03-count-min-sketch.md) |
|:---|---:|
