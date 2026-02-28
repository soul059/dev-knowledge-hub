# Chapter 3.6: Countable and Uncountable Sets

[← Previous: Cardinality](05-cardinality.md) | [Back to README](../README.md) | [Next: Cantor's Theorem →](07-cantors-theorem.md)

---

## 📋 Chapter Overview

One of the most surprising discoveries in mathematics is that **not all infinities are equal**. Some infinite sets are "countable" (the same size as $\mathbb{N}$), while others are "uncountable" (strictly larger). This chapter explores the boundary between these two types.

---

## 1. Countably Infinite Sets

A set $A$ is **countably infinite** if there exists a bijection $f: \mathbb{N} \rightarrow A$.

Equivalently, the elements of $A$ can be listed in an infinite sequence:

$$A = \{a_0, a_1, a_2, a_3, \ldots\}$$

A set is **countable** if it is finite or countably infinite.

---

## 2. Proving $\mathbb{Z}$ is Countable

List the integers in this clever order:

$$0, 1, -1, 2, -2, 3, -3, 4, -4, \ldots$$

```
  ℕ:  0   1   2   3   4   5   6   7   8  ...
       ↓   ↓   ↓   ↓   ↓   ↓   ↓   ↓   ↓
  ℤ:  0   1  -1   2  -2   3  -3   4  -4  ...
```

**Bijection:**

$$f(n) = \begin{cases} n/2 & \text{if } n \text{ is even} \\ -(n+1)/2 & \text{if } n \text{ is odd} \end{cases}$$

Since every integer appears exactly once in the list, $|\mathbb{Z}| = |\mathbb{N}| = \aleph_0$.

---

## 3. Proving $\mathbb{Q}$ is Countable (Cantor's Zigzag)

Arrange positive rationals in a grid and traverse diagonally:

```
           1    2    3    4    5   ...
         ┌────┬────┬────┬────┬────┐
    1    │1/1 →1/2  1/3 →1/4  1/5 │
         │  ↙    ↗    ↙    ↗      │
    2    │2/1  2/2  2/3  2/4  2/5 │
         │↗    ↙    ↗              │
    3    │3/1  3/2  3/3  3/4  3/5 │
         │  ↙    ↗                 │
    4    │4/1  4/2  4/3  4/4  4/5 │
         │↗                        │
    5    │5/1  5/2  5/3  5/4  5/5 │
         └────┴────┴────┴────┴────┘
  
  Zigzag path: 1/1, 1/2, 2/1, 3/1, 2/2, 1/3, 1/4, 2/3, 3/2, 4/1, ...
  Skip duplicates (like 2/2 = 1/1)
```

This enumerates every positive rational. Including 0 and negatives:

$$0, \frac{1}{1}, -\frac{1}{1}, \frac{1}{2}, -\frac{1}{2}, \frac{2}{1}, -\frac{2}{1}, \frac{1}{3}, -\frac{1}{3}, \ldots$$

Therefore $|\mathbb{Q}| = \aleph_0$.

---

## 4. Proving $\mathbb{N} \times \mathbb{N}$ is Countable

**Cantor's Pairing Function:**

$$\pi(m, n) = \frac{(m + n)(m + n + 1)}{2} + m$$

```
  n\m │  0     1     2     3     4
  ────┼───────────────────────────
   0  │  0     1     3     6    10
   1  │  2     4     7    11    ...
   2  │  5     8    12    ...
   3  │  9    13    ...
   4  │ 14    ...
  
  Traversal follows anti-diagonals:
  (0,0), (1,0), (0,1), (2,0), (1,1), (0,2), (3,0), ...
```

This is a bijection from $\mathbb{N} \times \mathbb{N}$ to $\mathbb{N}$.

---

## 5. Uncountable Sets: Cantor's Diagonal Argument

**Theorem:** The interval $(0, 1)$ is **uncountable** (i.e., $|(0,1)| > \aleph_0$).

**Proof by Contradiction:**

Suppose $(0, 1)$ is countable. Then we can list all real numbers in $(0, 1)$:

$$r_1 = 0.\mathbf{d_{11}} d_{12} d_{13} d_{14} \ldots$$
$$r_2 = 0.d_{21} \mathbf{d_{22}} d_{23} d_{24} \ldots$$
$$r_3 = 0.d_{31} d_{32} \mathbf{d_{33}} d_{34} \ldots$$
$$r_4 = 0.d_{41} d_{42} d_{43} \mathbf{d_{44}} \ldots$$
$$\vdots$$

Now construct a new number $x = 0.x_1 x_2 x_3 x_4 \ldots$ where:

$$x_i = \begin{cases} 5 & \text{if } d_{ii} \neq 5 \\ 6 & \text{if } d_{ii} = 5 \end{cases}$$

```
  Supposed complete list:
  
  r₁ = 0.[4] 1 5 9 2 6 ...
  r₂ = 0. 3[7]3 8 0 1 ...
  r₃ = 0. 0 2[1]5 7 3 ...
  r₄ = 0. 8 6 0[5]4 9 ...
  r₅ = 0. 1 4 9 3[2]8 ...
  r₆ = 0. 7 0 6 1 5[0]...
            ↓ ↓ ↓ ↓ ↓ ↓
  Diagonal: 4 7 1 5 2 0
  
  x =       5 5 5 6 5 5 ...  (differs on every diagonal digit)
```

Then $x \neq r_i$ for any $i$ (they differ in the $i$-th decimal place).

**Contradiction!** $x \in (0, 1)$ but $x$ is not in our "complete" list.

Therefore $(0, 1)$ is uncountable. $\blacksquare$

---

## 6. $|\mathbb{R}| = |(0, 1)|$

We can construct a bijection between $(0, 1)$ and $\mathbb{R}$:

$$f(x) = \tan\left(\pi x - \frac{\pi}{2}\right)$$

```
  f(x) = tan(πx - π/2)
  
        ↑ f(x)
        │          ╱
        │        ╱
        │      ╱
  ──────│────╱──────→ x
   0    │  ╱     1
        │╱
       ╱│
     ╱  │
  
  Maps (0,1) → ℝ bijectively
```

Since $|(0,1)| > \aleph_0$, we have $|\mathbb{R}| > \aleph_0$.

---

## 7. Hierarchy of Countable Closures

Countable sets are closed under many operations:

| Operation | Result |
|-----------|--------|
| Subset of countable | Countable |
| Countable $\cup$ Countable | Countable |
| Countable union of countable sets | Countable |
| $A \times B$ (both countable) | Countable |
| $A^n$ (finite power of countable) | Countable |

**Not preserved:**

| Operation | Result |
|-----------|--------|
| $\mathcal{P}(\mathbb{N})$ | Uncountable |
| $\{0, 1\}^{\mathbb{N}}$ (infinite sequences) | Uncountable |
| $\mathbb{N}^{\mathbb{N}}$ (functions $\mathbb{N} \to \mathbb{N}$) | Uncountable |

---

## 8. Real-World Analogy

```
  ┌──────────────────────────────────────────────────┐
  │         Countable vs. Uncountable                 │
  │                                                   │
  │  Think of countable sets as items you can label   │
  │  with a ticket number (1, 2, 3, ...).             │
  │                                                   │
  │  Uncountable sets are like "continuous" entities   │
  │  — no matter how you try to number them, there    │
  │  will always be unlabeled items left over.         │
  │                                                   │
  │  Applications:                                    │
  │  • Computability: Only countably many programs    │
  │    but uncountably many problems → some problems  │
  │    are unsolvable!                                │
  │  • Measure Theory: Countable sets have "measure   │
  │    zero" on the real line                         │
  │  • Database Theory: Countable domains allow       │
  │    enumeration-based algorithms                   │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Definition |
|---------|------------|
| Countably infinite | Bijection with $\mathbb{N}$ exists |
| Countable | Finite or countably infinite |
| Uncountable | Not countable ($\|A\| > \aleph_0$) |
| $\|\mathbb{Z}\| = \aleph_0$ | Listed as $0, 1, -1, 2, -2, \ldots$ |
| $\|\mathbb{Q}\| = \aleph_0$ | Cantor's zigzag enumeration |
| $\|(0,1)\| > \aleph_0$ | Cantor's diagonal argument |
| $\|\mathbb{R}\| = \mathfrak{c}$ | Same cardinality as $(0,1)$ |

---

## ❓ Quick Revision Questions

1. **Define "countably infinite." Give two examples.**

2. **Explain why the set of all even numbers is countable.**

3. **Describe Cantor's zigzag argument for $\mathbb{Q}$.**

4. **Walk through Cantor's diagonal argument showing $(0,1)$ is uncountable.**

5. **Is the set of all finite subsets of $\mathbb{N}$ countable or uncountable?**

6. **Why does the existence of uncountable sets matter for computer science?**

---

[← Previous: Cardinality](05-cardinality.md) | [Back to README](../README.md) | [Next: Cantor's Theorem →](07-cantors-theorem.md)
