# Chapter 2.3: Combinations

[← Previous: Permutations](02-permutations.md) | [Back to README](../README.md) | [Next: Applications in Probability →](04-applications-in-probability.md)

---

## Overview

A **combination** is a selection of objects where **order does NOT matter**. While permutations count arrangements, combinations count selections. Choosing {A, B, C} is the same combination as {C, A, B}.

---

## 1. Definition

The number of ways to choose $r$ objects from $n$ distinct objects (without regard to order) is:

$$\boxed{\binom{n}{r} = C(n, r) = \frac{n!}{r!(n-r)!}}$$

Also read as "$n$ choose $r$."

### Relationship to Permutations

$$\binom{n}{r} = \frac{P(n,r)}{r!}$$

**Logic:** $P(n,r)$ counts all ordered selections. Since each group of $r$ objects can be arranged $r!$ ways, we divide by $r!$ to remove the ordering.

```
  Permutation vs. Combination (n=4, r=2, objects: A,B,C,D)
  
  PERMUTATIONS P(4,2) = 12      COMBINATIONS C(4,2) = 6
  (order matters)                (order doesn't matter)
  
  AB  BA                         {A,B}
  AC  CA                         {A,C}
  AD  DA                         {A,D}
  BC  CB                         {B,C}
  BD  DB                         {B,D}
  CD  DC                         {C,D}
  
  Each combination generates r! = 2! = 2 permutations
  12 / 2 = 6 ✓
```

---

## 2. Key Properties

| Property | Formula |
|----------|---------|
| Symmetry | $\binom{n}{r} = \binom{n}{n-r}$ |
| Boundary | $\binom{n}{0} = \binom{n}{n} = 1$ |
| Single selection | $\binom{n}{1} = n$ |
| Pascal's Identity | $\binom{n}{r} = \binom{n-1}{r-1} + \binom{n-1}{r}$ |
| Sum of row | $\sum_{r=0}^{n} \binom{n}{r} = 2^n$ |
| Vandermonde's Identity | $\binom{m+n}{r} = \sum_{k=0}^{r} \binom{m}{k}\binom{n}{r-k}$ |

### Pascal's Triangle

```
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
  Row sum = 2ⁿ
```

---

## 3. Examples

### Example 1: Committee Selection

Choose a committee of 4 from 10 people.

$$\binom{10}{4} = \frac{10!}{4! \cdot 6!} = \frac{10 \times 9 \times 8 \times 7}{4 \times 3 \times 2 \times 1} = \frac{5040}{24} = 210$$

### Example 2: Handshakes

In a room of 15 people, everyone shakes hands with everyone else. How many handshakes?

Each handshake involves choosing 2 people (order doesn't matter):

$$\binom{15}{2} = \frac{15 \times 14}{2} = 105$$

### Example 3: Lottery

A lottery picks 6 numbers from 1–49. How many possible tickets?

$$\binom{49}{6} = \frac{49!}{6! \cdot 43!} = 13{,}983{,}816$$

---

## 4. Combinations with Restrictions

### "At Least" / "At Most" Problems

**Example 4:** Choose a committee of 5 from 6 men and 4 women, with at least 2 women.

| Women | Men | Number of Committees |
|:-----:|:---:|:-------------------:|
| 2 | 3 | $\binom{4}{2} \times \binom{6}{3} = 6 \times 20 = 120$ |
| 3 | 2 | $\binom{4}{3} \times \binom{6}{2} = 4 \times 15 = 60$ |
| 4 | 1 | $\binom{4}{4} \times \binom{6}{1} = 1 \times 6 = 6$ |
| **Total** | | **186** |

### "Specific Members" Constraints

**Example 5:** Committee of 5 from 10 people. Person A must be included, Person B must be excluded.

- Fix A as in → 1 spot taken, 3 remaining spots from 8 people (exclude B)
- $\binom{8}{4} = 70$

---

## 5. Combinations with Repetition (Stars and Bars)

When choosing $r$ items from $n$ types **with repetition** (order doesn't matter):

$$\boxed{\binom{n + r - 1}{r} = \binom{n + r - 1}{n - 1}}$$

### Stars and Bars Method

Distributing $r$ identical objects into $n$ distinct bins is equivalent to placing $r$ stars and $(n-1)$ bars in a row.

**Example 6:** Buy 5 fruits from 3 types (apples, bananas, cherries). How many selections?

$$\binom{3 + 5 - 1}{5} = \binom{7}{5} = 21$$

```
  Stars and Bars: 5 fruits, 3 types
  
  ★★|★★|★    → 2 apples, 2 bananas, 1 cherry
  ★★★★★||    → 5 apples, 0 bananas, 0 cherries
  ||★★★★★    → 0 apples, 0 bananas, 5 cherries
  ★|★★|★★    → 1 apple, 2 bananas, 2 cherries
  
  Total: C(7,5) = C(7,2) = 21 arrangements
  
  n-1 = 2 bars divide into n = 3 regions
  r = 5 stars represent items
```

---

## 6. Binomial Theorem

The combination formula appears in the **Binomial Theorem**:

$$\boxed{(a + b)^n = \sum_{r=0}^{n} \binom{n}{r} a^{n-r} b^r}$$

### Special Cases

| Identity | Value |
|----------|-------|
| $a=b=1$ | $\sum \binom{n}{r} = 2^n$ |
| $a=1, b=-1$ | $\sum (-1)^r \binom{n}{r} = 0$ |
| $a=1, b=x$ | $(1+x)^n = \sum \binom{n}{r} x^r$ |

### Example 7

Expand $(x + 2)^4$:

$$= \binom{4}{0}x^4(2)^0 + \binom{4}{1}x^3(2)^1 + \binom{4}{2}x^2(2)^2 + \binom{4}{3}x^1(2)^3 + \binom{4}{4}x^0(2)^4$$

$$= x^4 + 8x^3 + 24x^2 + 32x + 16$$

---

## 7. Permutation vs. Combination — Decision Guide

```
  ┌─────────────────────────────────┐
  │     Does ORDER matter?          │
  └────────────┬────────────────────┘
               │
       ┌───────┴───────┐
       ▼               ▼
    ┌──YES──┐       ┌──NO──┐
    │       │       │      │
    ▼       │       ▼      │
  PERMU-    │     COMBI-   │
  TATION    │     NATION   │
    │       │       │      │
    ▼       │       ▼      │
  ┌─────────┴───┐  ┌──────┴──────┐
  │ Repetition? │  │ Repetition? │
  ├──Yes: nʳ    │  ├──Yes:       │
  │             │  │  C(n+r-1,r) │
  ├──No:        │  │             │
  │  P(n,r)     │  ├──No:       │
  └─────────────┘  │  C(n,r)    │
                   └─────────────┘
```

### Complete Comparison

| | Order Matters | Order Doesn't Matter |
|---|:---:|:---:|
| **Without Repetition** | $P(n,r) = \frac{n!}{(n-r)!}$ | $\binom{n}{r} = \frac{n!}{r!(n-r)!}$ |
| **With Repetition** | $n^r$ | $\binom{n+r-1}{r}$ |

---

## 8. Real-World Applications

| Application | Type |
|------------|------|
| **Lottery** | Combinations (order doesn't matter) |
| **Card hands** | $\binom{52}{5}$ poker hands |
| **Team selection** | Committee from a group |
| **Sampling** | Choose sample from population |
| **Networking** | Number of connections in $n$-node network: $\binom{n}{2}$ |
| **Binary strings** | Strings with exactly $k$ ones: $\binom{n}{k}$ |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Combination ($r$ from $n$, no rep.) | $\binom{n}{r} = \frac{n!}{r!(n-r)!}$ |
| Symmetry | $\binom{n}{r} = \binom{n}{n-r}$ |
| Pascal's Identity | $\binom{n}{r} = \binom{n-1}{r-1} + \binom{n-1}{r}$ |
| Sum of row | $\sum \binom{n}{r} = 2^n$ |
| Combination with repetition | $\binom{n+r-1}{r}$ |
| Binomial Theorem | $(a+b)^n = \sum \binom{n}{r} a^{n-r} b^r$ |
| Vandermonde's Identity | $\binom{m+n}{r} = \sum_k \binom{m}{k}\binom{n}{r-k}$ |

---

## Quick Revision Questions

1. Compute $\binom{12}{4}$ and $\binom{12}{8}$. Verify the symmetry property.

2. A pizza shop offers 10 toppings. How many ways can you choose (a) exactly 3 toppings, (b) at most 3 toppings, (c) any number of toppings?

3. From 7 men and 5 women, a committee of 6 is to be formed with at least 3 women. How many ways?

4. Using Stars and Bars, find the number of non-negative integer solutions to $x_1 + x_2 + x_3 + x_4 = 10$.

5. **Prove** Pascal's Identity $\binom{n}{r} = \binom{n-1}{r-1} + \binom{n-1}{r}$ algebraically and give a combinatorial argument.

6. How many 5-card poker hands contain exactly 2 aces?

---

[← Previous: Permutations](02-permutations.md) | [Back to README](../README.md) | [Next: Applications in Probability →](04-applications-in-probability.md)
