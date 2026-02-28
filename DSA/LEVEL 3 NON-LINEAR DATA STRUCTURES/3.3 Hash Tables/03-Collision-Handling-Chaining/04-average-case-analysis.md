# 3.4 Average Case Analysis

## Unit 3: Collision Handling - Chaining

---

## Simple Uniform Hashing Assumption (SUHA)

The analysis assumes **Simple Uniform Hashing**: each key is equally likely to hash to any of the $m$ slots, independent of other keys.

$$\Pr[h(k) = j] = \frac{1}{m} \quad \text{for all slots } j$$

---

## Expected Chain Length

With $n$ keys and $m$ buckets under SUHA:

$$\text{Expected chain length} = \frac{n}{m} = \alpha$$

```
╔═══════════════════════════════════════════════════════════════╗
║            EXPECTED DISTRIBUTION (SUHA)                      ║
║                                                              ║
║   n = 12 keys, m = 4 buckets, α = 3                        ║
║                                                              ║
║   Expected: 3 keys per bucket                               ║
║   ┌───┐                                                     ║
║   │ 0 │──► [a] ──► [b] ──► [c] ──► null       (3 keys)    ║
║   ├───┤                                                     ║
║   │ 1 │──► [d] ──► [e] ──► [f] ──► null       (3 keys)    ║
║   ├───┤                                                     ║
║   │ 2 │──► [g] ──► [h] ──► [i] ──► null       (3 keys)    ║
║   ├───┤                                                     ║
║   │ 3 │──► [j] ──► [k] ──► [l] ──► null       (3 keys)    ║
║   └───┘                                                     ║
║                                                              ║
║   Each bucket stores exactly α = 3 elements (expected)      ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Unsuccessful Search Analysis

An **unsuccessful search** looks through an entire chain without finding the key.

$$E[\text{unsuccessful search}] = \alpha = \frac{n}{m}$$

```
╔═══════════════════════════════════════════════════════════╗
║           UNSUCCESSFUL SEARCH                            ║
║                                                          ║
║   Searching for key K (not in table):                   ║
║                                                          ║
║   1. Compute index = h(K)           → O(1)             ║
║   2. Traverse entire chain at index → O(α)             ║
║   3. Reach null → "NOT FOUND"       → O(1)             ║
║                                                          ║
║   Total: O(1 + α)                                       ║
║                                                          ║
║   If α = O(1) (constant load factor):                   ║
║   Unsuccessful search = O(1)                            ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Successful Search Analysis

A **successful search** finds the key somewhere in the chain. On average, it searches about half the chain.

$$E[\text{successful search}] = 1 + \frac{\alpha}{2} - \frac{1}{2m}$$

For large $m$: $\approx 1 + \frac{\alpha}{2}$

```
╔═══════════════════════════════════════════════════════════╗
║            SUCCESSFUL SEARCH                             ║
║                                                          ║
║   Chain: [K₁] ──► [K₂] ──► [K₃] ──► [K₄] ──► null    ║
║                                                          ║
║   If searching for K₁: 1 comparison                     ║
║   If searching for K₂: 2 comparisons                    ║
║   If searching for K₃: 3 comparisons                    ║
║   If searching for K₄: 4 comparisons                    ║
║                                                          ║
║   Average = (1+2+3+4)/4 = 2.5 comparisons              ║
║           = 1 + (4-1)/2 = 1 + α/2  (when α = 3)       ║
║                                                          ║
║   On average, we search HALF the chain                  ║
╚═══════════════════════════════════════════════════════════╝
```

---

## Numerical Examples

| n | m | α = n/m | Unsuccessful Search | Successful Search |
|:-:|:-:|:-------:|:-------------------:|:-----------------:|
| 10 | 10 | 1.0 | 1 + 1.0 = 2.0 | 1 + 0.5 = 1.5 |
| 10 | 20 | 0.5 | 1 + 0.5 = 1.5 | 1 + 0.25 = 1.25 |
| 100 | 100 | 1.0 | 1 + 1.0 = 2.0 | 1 + 0.5 = 1.5 |
| 100 | 50 | 2.0 | 1 + 2.0 = 3.0 | 1 + 1.0 = 2.0 |
| 1000 | 100 | 10.0 | 1 + 10.0 = 11.0 | 1 + 5.0 = 6.0 |

> **Key Insight**: As long as $\alpha = O(1)$ (i.e., $m = \Theta(n)$), all operations are $O(1)$ average case!

---

## Proof Sketch: Unsuccessful Search

```
╔═══════════════════════════════════════════════════════════════╗
║              PROOF: UNSUCCESSFUL SEARCH = O(1 + α)          ║
║                                                              ║
║   1. Compute h(k) takes O(1)                                ║
║                                                              ║
║   2. Expected chain length at any bucket:                    ║
║      E[chain at bucket j] = n × Pr[key maps to j]          ║
║                            = n × (1/m)                      ║
║                            = n/m = α                        ║
║                                                              ║
║   3. Must traverse entire chain (key not present)            ║
║      Expected comparisons = α                               ║
║                                                              ║
║   4. Total expected time = O(1 + α)                         ║
║                                                              ║
║   If m = cn for constant c (e.g., m = 2n):                  ║
║      α = n/(cn) = 1/c = O(1)                               ║
║      Total = O(1 + O(1)) = O(1) ✓                          ║
╚═══════════════════════════════════════════════════════════════╝
```

---

## Complexity Summary

| Operation | Best Case | Average Case | Worst Case |
|-----------|:---------:|:------------:|:----------:|
| Insert | O(1) | O(1) | O(n) |
| Search (successful) | O(1) | O(1 + α/2) | O(n) |
| Search (unsuccessful) | O(1) | O(1 + α) | O(n) |
| Delete | O(1) | O(1 + α/2) | O(n) |

> All operations are **O(1) average** when $\alpha = O(1)$, which is maintained by resizing when $\alpha$ exceeds a threshold.

---

## Quick Revision Question

**Q: A chained hash table has 200 elements in 50 buckets. What is the expected number of comparisons for (a) a successful search and (b) an unsuccessful search?**

<details>
<summary>Click to reveal answer</summary>

$\alpha = n/m = 200/50 = 4$

**(a) Successful search**: $1 + \alpha/2 = 1 + 4/2 = 1 + 2 = 3$ comparisons expected.

**(b) Unsuccessful search**: $1 + \alpha = 1 + 4 = 5$ comparisons expected (must check entire chain).

This tells us the table should be resized — a load factor of 4 is quite high. Doubling to 100 buckets would give $\alpha = 2$, and using 200 buckets would give $\alpha = 1$.

</details>

---

| [← Previous: Load Factor](03-load-factor.md) | [Next: Worst Case Scenario →](05-worst-case-scenario.md) |
|:---|---:|
