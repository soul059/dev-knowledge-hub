# 9.6 Locality-Sensitive Hashing (LSH)

## Unit 9: Advanced Hashing

---

## The Problem

> Find **similar** items efficiently among billions of items.

Standard hash functions are designed so that **similar inputs produce different hashes**. LSH does the **opposite**: similar items should hash to the **same bucket** with high probability.

```
╔══════════════════════════════════════════════════════════════╗
║  REGULAR HASHING:                                           ║
║    "hello" → 0x3A7F     "hallo" → 0xB291                  ║
║    Tiny change in input → completely different hash         ║
║    Goal: MINIMIZE collisions                                ║
║                                                              ║
║  LOCALITY-SENSITIVE HASHING:                                ║
║    "hello" → bucket 5    "hallo" → bucket 5                ║
║    Similar inputs → SAME bucket (with high probability)     ║
║    Goal: MAXIMIZE collisions for similar items              ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Definition

A hash family $\mathcal{H}$ is $(d_1, d_2, p_1, p_2)$-sensitive if for any two points $x, y$:

$$\text{If } d(x, y) \leq d_1 \text{ then } \Pr[h(x) = h(y)] \geq p_1$$
$$\text{If } d(x, y) \geq d_2 \text{ then } \Pr[h(x) = h(y)] \leq p_2$$

Where $d_1 < d_2$ and $p_1 > p_2$.

```
╔══════════════════════════════════════════════════════════════╗
║  Probability of same bucket                                 ║
║  │                                                          ║
║  │  p₁ ──────────╲                                         ║
║  │                  ╲                                       ║
║  │                    ╲                                     ║
║  │                      ╲                                   ║
║  │                        ╲                                 ║
║  │  p₂                     ╲──────────                     ║
║  │                                                          ║
║  └──────────┼─────────────┼─────────→ distance             ║
║             d₁            d₂                                ║
║                                                              ║
║  CLOSE items (d < d₁): high collision probability           ║
║  FAR items (d > d₂):   low collision probability            ║
╚══════════════════════════════════════════════════════════════╝
```

---

## LSH for Jaccard Similarity: MinHash

The **Jaccard similarity** of two sets $A$ and $B$:

$$J(A, B) = \frac{|A \cap B|}{|A \cup B|}$$

**MinHash** creates a signature that preserves Jaccard similarity:

```
FUNCTION minHash(set, numHashFunctions):
    signature = []

    FOR i = 1 TO numHashFunctions:
        minVal = INFINITY
        FOR each element in set:
            hashed = hᵢ(element)
            minVal = min(minVal, hashed)
        signature.add(minVal)

    RETURN signature

KEY PROPERTY:
  Pr[minHash_i(A) == minHash_i(B)] = J(A, B)
```

### Trace

```
A = {1, 2, 3}     B = {2, 3, 4}

J(A,B) = |{2,3}| / |{1,2,3,4}| = 2/4 = 0.5

Using 4 hash functions (simplified: hᵢ(x) = (a·x + b) mod p):

h₁: A→{3, 5, 2} min=2    B→{5, 2, 1} min=1    2≠1
h₂: A→{7, 1, 4} min=1    B→{1, 4, 6} min=1    1==1 ✓
h₃: A→{2, 8, 3} min=2    B→{8, 3, 5} min=3    2≠3
h₄: A→{6, 4, 9} min=4    B→{4, 9, 2} min=2    4≠2

Signature A: [2, 1, 2, 4]
Signature B: [1, 1, 3, 2]
Estimated J = matches/total = 1/4 = 0.25

(With more hash functions, estimate converges to true J=0.5)
```

---

## LSH Banding Technique

To amplify the gap between $p_1$ and $p_2$, divide the signature into **bands**:

```
╔══════════════════════════════════════════════════════════════╗
║  Signature of 100 hash values → 20 bands of 5 rows each   ║
║                                                              ║
║  Band 1: [h₁, h₂, h₃, h₄, h₅]                            ║
║  Band 2: [h₆, h₇, h₈, h₉, h₁₀]                          ║
║  ...                                                        ║
║  Band 20: [h₉₆, h₉₇, h₉₈, h₉₉, h₁₀₀]                  ║
║                                                              ║
║  Two items are candidates if they agree on ALL rows in     ║
║  ANY band:                                                  ║
║                                                              ║
║  Pr[match in one band] = s^r     (s=similarity, r=rows)    ║
║  Pr[match in no band] = (1 - s^r)^b                        ║
║  Pr[candidate pair] = 1 - (1 - s^r)^b                      ║
║                                                              ║
║  With b=20, r=5:                                            ║
║    s=0.8: Pr = 1-(1-0.8⁵)²⁰ = 1-(1-0.328)²⁰ ≈ 0.9996    ║
║    s=0.2: Pr = 1-(1-0.2⁵)²⁰ = 1-(1-0.00032)²⁰ ≈ 0.0064  ║
║                                                              ║
║  Sharp threshold! Similar items almost always found,        ║
║  dissimilar items almost never paired.                      ║
╚══════════════════════════════════════════════════════════════╝
```

---

## LSH for Cosine Similarity: Random Hyperplanes

For vector data, use random hyperplane projections:

```
FUNCTION cosineLSH(vector, randomVectors):
    hash = 0
    FOR i = 0 TO k - 1:
        // Dot product with random vector
        IF dot(vector, randomVectors[i]) >= 0:
            hash = hash | (1 << i)
    RETURN hash

PROPERTY:
  Pr[h(u) == h(v)] = 1 - θ(u,v)/π
  where θ is the angle between vectors u and v
```

---

## Complete LSH Pipeline

```
╔══════════════════════════════════════════════════════════════╗
║  1. COMPUTE SIGNATURES: MinHash/SimHash for all items      ║
║     Each item → fixed-size signature vector                 ║
║                                                              ║
║  2. BAND & HASH: Divide signatures into bands              ║
║     Hash each band to find candidate pairs                  ║
║                                                              ║
║  3. CANDIDATE PAIRS: Items that match in any band           ║
║     These are "probably similar" → few false negatives      ║
║                                                              ║
║  4. VERIFY: Compute true similarity for candidates only     ║
║     Filter out false positives                              ║
║                                                              ║
║  Result: All truly similar pairs found (with high prob)     ║
║          without checking all O(n²) pairs                   ║
╚══════════════════════════════════════════════════════════════╝
```

---

## Applications

| Application | Similarity Type | Domain |
|-------------|:-:|---------|
| Near-duplicate web pages | Jaccard (shingles) | Search engines |
| Similar images | Cosine (feature vectors) | Image search |
| Recommendation systems | Cosine/Jaccard | User preferences |
| Genome comparison | Jaccard (k-mers) | Bioinformatics |
| Plagiarism detection | Jaccard (n-grams) | Academic tools |
| Malware detection | Feature similarity | Security |

---

## Complexity Comparison

| Approach | Time | Space |
|----------|:---:|:---:|
| Brute force (all pairs) | $O(n^2 \cdot d)$ | $O(1)$ |
| LSH | $O(n)$ expected | $O(n)$ |
| KD-tree | $O(n \log n)$ | $O(n)$ |

Where $n$ = number of items, $d$ = dimension. LSH is the only method that scales well for high-dimensional data (avoids the "curse of dimensionality" that affects KD-trees).

---

## Quick Revision Question

**Q: How is LSH fundamentally different from traditional hash functions? What property of a good traditional hash function does LSH deliberately violate?**

<details>
<summary>Click to reveal answer</summary>

Traditional hash functions are designed for **uniform distribution** — even slightly different inputs should produce completely different, uniformly distributed hash values. This property (called the **avalanche effect**) minimizes collisions in hash tables.

LSH **deliberately violates** this property. Instead of minimizing collisions, LSH **maximizes collisions for similar items**:

| Property | Traditional Hash | LSH |
|----------|:---:|:---:|
| Goal | Minimize collisions | Maximize similar-item collisions |
| Similar inputs | Different hashes | Same hash (high probability) |
| Different inputs | Different hashes | Different hashes (high probability) |
| Use case | Hash tables, integrity | Similarity search |

The key insight is that LSH is **not used for data storage** (like hash tables). It's used for **approximate nearest neighbor search** — converting an $O(n^2)$ all-pairs similarity problem into an $O(n)$ candidate generation problem.

LSH trades **exactness** for **speed**: it might miss some similar pairs (false negatives) and need to verify some dissimilar pairs (false positives), but it explores only a tiny fraction of the search space.

</details>

---

| [← Perfect Hashing](05-perfect-hashing.md) | [Unit 10: Custom Hash Functions →](../10-Custom-Hash-Functions/01-hashing-custom-objects.md) |
|:---|---:|
