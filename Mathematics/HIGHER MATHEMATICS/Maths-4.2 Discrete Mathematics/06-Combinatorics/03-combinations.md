# Chapter 6.3: Combinations

[← Previous: Permutations](02-permutations.md) | [Back to README](../README.md) | [Next: Binomial Theorem →](04-binomial-theorem.md)

---

## 📋 Chapter Overview

A **combination** is a selection of objects where **order does not matter**. While permutations answer "how many arrangements?", combinations answer "how many groups?"

---

## 1. Definition

The number of ways to choose $r$ objects from $n$ distinct objects (order doesn't matter):

$$\binom{n}{r} = C(n, r) = \frac{n!}{r!(n-r)!} = \frac{P(n,r)}{r!}$$

```
  Permutation vs. Combination:
  
  Choosing 2 from {A, B, C}:
  
  Permutations (order matters):     Combinations (order doesn't):
  AB, BA, AC, CA, BC, CB            {A,B}, {A,C}, {B,C}
  P(3,2) = 6                        C(3,2) = 3
  
  C(n,r) = P(n,r) / r!    ← divide out the orderings
```

---

## 2. Key Values

| $\binom{n}{r}$ | Value | Meaning |
|:-:|:-:|---------|
| $\binom{n}{0}$ | 1 | One way to choose nothing |
| $\binom{n}{1}$ | $n$ | Choose one item |
| $\binom{n}{n}$ | 1 | Choose everything |
| $\binom{n}{r}$ | $\binom{n}{n-r}$ | Symmetry |
| $\binom{5}{2}$ | 10 | |
| $\binom{10}{3}$ | 120 | |
| $\binom{52}{5}$ | 2,598,960 | Poker hands |

---

## 3. Pascal's Triangle

$$\binom{n}{r} = \binom{n-1}{r-1} + \binom{n-1}{r}$$

```
  Pascal's Triangle:
  
  n=0:                    1
  n=1:                  1   1
  n=2:                1   2   1
  n=3:              1   3   3   1
  n=4:            1   4   6   4   1
  n=5:          1   5  10  10   5   1
  n=6:        1   6  15  20  15   6   1
  n=7:      1   7  21  35  35  21   7   1
  
  Each entry = sum of two entries above it
  Row n gives C(n,0), C(n,1), ..., C(n,n)
```

**Properties visible in Pascal's Triangle:**
- Symmetry: $\binom{n}{r} = \binom{n}{n-r}$
- Row sum: $\sum_{r=0}^{n} \binom{n}{r} = 2^n$
- Alternating sum: $\sum_{r=0}^{n} (-1)^r \binom{n}{r} = 0$

---

## 4. Important Identities

| Identity | Formula |
|----------|---------|
| Symmetry | $\binom{n}{r} = \binom{n}{n-r}$ |
| Pascal's rule | $\binom{n}{r} = \binom{n-1}{r-1} + \binom{n-1}{r}$ |
| Vandermonde's | $\binom{m+n}{r} = \sum_{k=0}^{r} \binom{m}{k}\binom{n}{r-k}$ |
| Sum of row | $\sum_{r=0}^{n} \binom{n}{r} = 2^n$ |
| Hockey stick | $\sum_{i=r}^{n} \binom{i}{r} = \binom{n+1}{r+1}$ |
| Absorption | $\binom{n}{r} = \frac{n}{r}\binom{n-1}{r-1}$ |

---

## 5. Combinations with Repetition

Choose $r$ items from $n$ types **with repetition allowed** (order doesn't matter):

$$\binom{n + r - 1}{r} = \binom{n + r - 1}{n - 1}$$

This is the "stars and bars" formula.

**Example:** How many ways to buy 5 donuts from 3 flavors (chocolate, vanilla, strawberry)?

$$\binom{3 + 5 - 1}{5} = \binom{7}{5} = 21$$

```
  Stars and Bars: 5 donuts (★), 2 dividers (|)
  
  ★★★|★|★      → 3 choc, 1 van, 1 straw
  ★★|★★|★      → 2 choc, 2 van, 1 straw
  |★★★★|★      → 0 choc, 4 van, 1 straw
  ★★★★★||      → 5 choc, 0 van, 0 straw
  
  Arrange 5 ★'s and 2 |'s: C(7,2) = 21 ways
```

---

## 6. Comparing Counting Techniques

| | Order Matters | Order Doesn't Matter |
|-|:---:|:---:|
| **No Repetition** | $P(n,r) = \frac{n!}{(n-r)!}$ | $\binom{n}{r} = \frac{n!}{r!(n-r)!}$ |
| **Repetition** | $n^r$ | $\binom{n+r-1}{r}$ |

```
  Decision flowchart:
  
  Does ORDER matter?
      │
      ├── YES → Is REPETITION allowed?
      │         ├── YES → nʳ
      │         └── NO  → P(n,r) = n!/(n-r)!
      │
      └── NO  → Is REPETITION allowed?
                ├── YES → C(n+r-1, r) "stars & bars"
                └── NO  → C(n,r) = n!/(r!(n-r)!)
```

---

## 7. Worked Examples

### Example 1: Committee Selection

Choose a committee of 4 from 10 people.

$$\binom{10}{4} = \frac{10!}{4! \cdot 6!} = 210$$

### Example 2: Committee with Constraints

From 6 men and 4 women, form a committee of 5 with at least 2 women.

| Women | Men | Count |
|:---:|:---:|:---:|
| 2 | 3 | $\binom{4}{2}\binom{6}{3} = 6 \times 20 = 120$ |
| 3 | 2 | $\binom{4}{3}\binom{6}{2} = 4 \times 15 = 60$ |
| 4 | 1 | $\binom{4}{4}\binom{6}{1} = 1 \times 6 = 6$ |
| | **Total** | **186** |

### Example 3: Distributing Identical Objects

Distribute 10 identical balls into 4 distinct boxes.

$$\binom{10 + 4 - 1}{10} = \binom{13}{10} = \binom{13}{3} = 286$$

---

## 8. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Combinations in Practice                  │
  │                                                   │
  │  1. Lottery Probability                          │
  │     Pick 6 from 49: C(49,6) = 13,983,816        │
  │     P(jackpot) = 1/13,983,816 ≈ 0.000007%       │
  │                                                   │
  │  2. Network Design                               │
  │     C(n,2) = links needed for full mesh network  │
  │                                                   │
  │  3. Binary Strings                               │
  │     C(n,k) = strings of length n with k ones     │
  │                                                   │
  │  4. Sampling                                     │
  │     Choose test cases from a larger set           │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Formula |
|---------|:-------:|
| $r$ from $n$ (no repeat, no order) | $\binom{n}{r}$ |
| With repetition (no order) | $\binom{n+r-1}{r}$ |
| Pascal's Rule | $\binom{n}{r} = \binom{n-1}{r-1} + \binom{n-1}{r}$ |
| Row sum | $\sum \binom{n}{r} = 2^n$ |
| Stars and bars | Distribute $r$ identical items into $n$ boxes |

---

## ❓ Quick Revision Questions

1. **Compute $\binom{8}{3}$ and $\binom{8}{5}$. Why are they equal?**

2. **How many 5-card poker hands can be dealt from a 52-card deck?**

3. **How many non-negative integer solutions does $x_1 + x_2 + x_3 = 10$ have?**

4. **A class has 12 boys and 8 girls. How many committees of 5 have exactly 3 boys?**

5. **Prove Pascal's identity: $\binom{n}{r} = \binom{n-1}{r-1} + \binom{n-1}{r}$.**

6. **How many ways can you choose 4 flavors of ice cream from 10 flavors if repeats are allowed?**

---

[← Previous: Permutations](02-permutations.md) | [Back to README](../README.md) | [Next: Binomial Theorem →](04-binomial-theorem.md)
