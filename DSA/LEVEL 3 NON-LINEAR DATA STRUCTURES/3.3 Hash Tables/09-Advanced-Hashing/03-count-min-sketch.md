# 9.3 Count-Min Sketch

## Unit 9: Advanced Hashing

---

## What is a Count-Min Sketch?

A **probabilistic frequency estimation** data structure. It answers: **"How many times has element X been seen?"** — with a small overcount error.

```
╔══════════════════════════════════════════════════════════════╗
║  GUARANTEES:                                                ║
║                                                              ║
║  True count: f(x)     Estimated count: f̂(x)                ║
║                                                              ║
║  f̂(x) ≥ f(x)          (NEVER undercounts)                  ║
║  f̂(x) ≤ f(x) + εN    (with probability ≥ 1 - δ)          ║
║                                                              ║
║  where N = total count of all elements                      ║
║        ε = error factor (controls overcount)                ║
║        δ = failure probability                              ║
║                                                              ║
║  Space: O(1/ε · log(1/δ))                                  ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Structure

A 2D array of counters — $d$ rows × $w$ columns — with one hash function per row.

```
╔══════════════════════════════════════════════════════════════╗
║                                                              ║
║  w = ⌈e/ε⌉ columns (width)                                ║
║  d = ⌈ln(1/δ)⌉ rows (depth)                               ║
║                                                              ║
║         col 0   col 1   col 2   col 3   col 4              ║
║  h₁:  [  0  ] [  0  ] [  0  ] [  0  ] [  0  ]             ║
║  h₂:  [  0  ] [  0  ] [  0  ] [  0  ] [  0  ]             ║
║  h₃:  [  0  ] [  0  ] [  0  ] [  0  ] [  0  ]             ║
║                                                              ║
║  Each row uses a DIFFERENT hash function                    ║
║  Each hash function maps elements to columns [0, w-1]      ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Operations

### Update (Increment Count)

```
FUNCTION update(element, count=1):
    FOR i = 0 TO d - 1:
        j = hᵢ(element) % w
        table[i][j] += count
```

### Query (Estimate Count)

```
FUNCTION estimate(element):
    minCount = INFINITY
    FOR i = 0 TO d - 1:
        j = hᵢ(element) % w
        minCount = min(minCount, table[i][j])
    RETURN minCount
```

The **minimum** across all rows is the best estimate — it's the row least affected by collisions.

---

## Complete Trace

```
w = 5, d = 3 (3 hash functions)

Initial table:
       0   1   2   3   4
h₁: [  0   0   0   0   0 ]
h₂: [  0   0   0   0   0 ]
h₃: [  0   0   0   0   0 ]

UPDATE("apple", 1):
  h₁("apple")%5 = 1, h₂("apple")%5 = 3, h₃("apple")%5 = 0
       0   1   2   3   4
h₁: [  0   1   0   0   0 ]
h₂: [  0   0   0   1   0 ]
h₃: [  1   0   0   0   0 ]

UPDATE("banana", 1):
  h₁("banana")%5 = 3, h₂("banana")%5 = 0, h₃("banana")%5 = 4
       0   1   2   3   4
h₁: [  0   1   0   1   0 ]
h₂: [  1   0   0   1   0 ]
h₃: [  1   0   0   0   1 ]

UPDATE("apple", 1):  (second occurrence)
  h₁("apple")%5 = 1, h₂("apple")%5 = 3, h₃("apple")%5 = 0
       0   1   2   3   4
h₁: [  0   2   0   1   0 ]
h₂: [  1   0   0   2   0 ]
h₃: [  2   0   0   0   1 ]

UPDATE("cherry", 1):
  h₁("cherry")%5 = 1, h₂("cherry")%5 = 4, h₃("cherry")%5 = 0
       0   1   2   3   4
h₁: [  0   3   0   1   0 ]   ← col 1 has apple+cherry
h₂: [  1   0   0   2   1 ]
h₃: [  3   0   0   0   1 ]   ← col 0 has apple+cherry+banana(h₂)

QUERY("apple"):
  h₁ → table[0][1] = 3    (inflated by "cherry" collision)
  h₂ → table[1][3] = 2    ← CORRECT!
  h₃ → table[2][0] = 3    (inflated by collisions)
  Answer: min(3, 2, 3) = 2 ✓ (exact: 2)

QUERY("banana"):
  h₁ → table[0][3] = 1    ← CORRECT!
  h₂ → table[1][0] = 1    ← CORRECT!
  h₃ → table[2][4] = 1    ← CORRECT!
  Answer: min(1, 1, 1) = 1 ✓

QUERY("grape"):   (never inserted)
  h₁("grape")%5 = 3 → table[0][3] = 1
  h₂("grape")%5 = 2 → table[1][2] = 0
  h₃("grape")%5 = 1 → table[2][1] = 0
  Answer: min(1, 0, 0) = 0 ✓ (if any row gives 0, answer is 0)
```

---

## Why "Min" Works

```
╔══════════════════════════════════════════════════════════════╗
║  Each cell accumulates counts from ALL elements that       ║
║  hash to that position.                                     ║
║                                                              ║
║  table[i][j] = f(x) + (collisions from other elements)     ║
║                                                              ║
║  Collisions can only ADD to the cell, never subtract.       ║
║  → Every row gives an OVERESTIMATE                          ║
║  → The MINIMUM is the least-overestimated = best estimate  ║
║                                                              ║
║  With d independent hash functions, the probability that    ║
║  ALL d rows have large collisions is:                       ║
║  (Pr[single row overcount > εN])^d ≤ (1/e)^d = δ          ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Sizing Parameters

| Desired guarantee | Width $w$ | Depth $d$ |
|:---:|:---:|:---:|
| ε = 0.1%, δ = 0.01% | 2,719 | 10 |
| ε = 0.1%, δ = 1% | 2,719 | 5 |
| ε = 1%, δ = 0.1% | 272 | 7 |
| ε = 1%, δ = 1% | 272 | 5 |

Total space: $w \times d$ counters (a few KB typically).

---

## Bloom Filter vs Count-Min Sketch

| Feature | Bloom Filter | Count-Min Sketch |
|---------|:---:|:---:|
| Answers | "Is X in set?" | "How often is X?" |
| Error type | False positive only | Overcount only |
| Structure | 1D bit array | 2D counter array |
| Space | $O(n/\ln 2)$ bits | $O(1/ε \cdot \log(1/δ))$ counters |
| Deletion | No (unless counting) | Yes (decrement) |

---

## Applications

| Use Case | What It Tracks |
|----------|---------------|
| Network traffic monitoring | Packet frequencies per IP/flow |
| Heavy hitters detection | Find elements with count > threshold |
| Database query optimization | Approximate histogram of column values |
| Streaming analytics | Popular URLs, trending hashtags |
| NLP | Word frequency estimation |

---

## Quick Revision Question

**Q: Can the Count-Min Sketch ever undercount an element's frequency? Why or why not?**

<details>
<summary>Click to reveal answer</summary>

**No, it can NEVER undercount** (assuming no negative updates).

**Proof:** For element $x$ with true count $f(x)$:

Each cell `table[i][hᵢ(x)]` contains:
$$\text{table}[i][h_i(x)] = f(x) + \sum_{\substack{y \neq x \\ h_i(y) = h_i(x)}} f(y)$$

The second term is the sum of counts of OTHER elements that collide with $x$ in row $i$. Since all counts are non-negative:
$$\text{table}[i][h_i(x)] \geq f(x)$$

This holds for **every** row $i$. Therefore:
$$\hat{f}(x) = \min_i \text{table}[i][h_i(x)] \geq f(x)$$

The estimate is always ≥ the true count. This is why the Count-Min Sketch is a one-sided error structure — it can overcount but never undercount.

The only exception is if `update(element, -count)` is called (decrement), which could cause undercounting. Standard CMS only uses positive increments.

</details>

---

| [← Bloom Filters](02-bloom-filters.md) | [Next: Cuckoo Hashing →](04-cuckoo-hashing.md) |
|:---|---:|
