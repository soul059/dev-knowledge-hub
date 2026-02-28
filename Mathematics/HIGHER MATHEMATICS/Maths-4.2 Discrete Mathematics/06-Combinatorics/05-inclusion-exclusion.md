# Chapter 6.5: Inclusion-Exclusion Principle (Advanced)

[← Previous: Binomial Theorem](04-binomial-theorem.md) | [Back to README](../README.md) | [Next: Generating Functions →](06-generating-functions.md)

---

## 📋 Chapter Overview

The **Principle of Inclusion-Exclusion (PIE)** generalizes the sum rule to handle overlapping sets. It's one of the most powerful counting tools, used to count derangements, surjections, Euler's totient, and more.

---

## 1. General Formula

For sets $A_1, A_2, \ldots, A_n$:

$$\left|\bigcup_{i=1}^{n} A_i\right| = \sum_{i} |A_i| - \sum_{i<j} |A_i \cap A_j| + \sum_{i<j<k} |A_i \cap A_j \cap A_k| - \cdots + (-1)^{n+1}|A_1 \cap \cdots \cap A_n|$$

| Sets | Formula |
|:----:|---------|
| 2 | $\|A \cup B\| = \|A\| + \|B\| - \|A \cap B\|$ |
| 3 | $\|A \cup B \cup C\| = \|A\|+\|B\|+\|C\| - \|A{\cap}B\| - \|A{\cap}C\| - \|B{\cap}C\| + \|A{\cap}B{\cap}C\|$ |

```
  PIE for 3 sets — Venn Diagram:
  
           ┌───────────────────────┐
           │A       ┌─────────────┤B
           │    ┌───┤             │
           │    │ A∩│B            │
           │    │ ∩C│             │
           │  A ├───┤   B        │
           │  ∩C│   │   ∩C       │
           └────┤   ├────────────┘
                │ C │
                └───┘
  
  Add singles, subtract pairs, add triple.
```

---

## 2. Complementary Form

To count elements in **none** of $A_1, \ldots, A_n$:

$$\left|\overline{A_1} \cap \overline{A_2} \cap \cdots \cap \overline{A_n}\right| = |U| - \left|\bigcup_{i=1}^{n} A_i\right|$$

This is the "at least / none" form — incredibly useful in practice.

---

## 3. Application: Counting Surjections

The number of surjections from an $m$-element set to an $n$-element set:

$$S(m, n) = \sum_{k=0}^{n} (-1)^k \binom{n}{k}(n-k)^m$$

**Derivation:** Let $A_i$ = functions that miss element $i$ in the codomain.

$$|A_i| = (n-1)^m, \quad |A_i \cap A_j| = (n-2)^m, \quad \ldots$$

By PIE: Functions missing at least one element = $\sum (-1)^{k+1}\binom{n}{k}(n-k)^m$

Surjections = Total $-$ missing at least one.

**Example:** Surjections from $\{1,2,3,4\}$ to $\{a,b,c\}$:

$$S(4,3) = \binom{3}{0}3^4 - \binom{3}{1}2^4 + \binom{3}{2}1^4 - \binom{3}{3}0^4$$
$$= 81 - 48 + 3 - 0 = 36$$

---

## 4. Application: Derangements (Revisited)

A derangement avoids $A_i = \{\text{permutations fixing element } i\}$.

$$D_n = n! - \binom{n}{1}(n-1)! + \binom{n}{2}(n-2)! - \cdots + (-1)^n\binom{n}{n}0!$$

$$= n! \sum_{k=0}^{n} \frac{(-1)^k}{k!}$$

---

## 5. Application: Euler's Totient Function

$\phi(n)$ = count of integers from 1 to $n$ coprime to $n$.

If $n = p_1^{a_1} \cdots p_k^{a_k}$:

$$\phi(n) = n \prod_{p \mid n}\left(1 - \frac{1}{p}\right)$$

**Example:** $\phi(12) = 12(1 - 1/2)(1 - 1/3) = 12 \cdot 1/2 \cdot 2/3 = 4$

The 4 numbers coprime to 12 in $\{1,...,12\}$: $\{1, 5, 7, 11\}$.

---

## 6. Application: Problème des Ménages

How many ways can $n$ married couples sit at a round table so that no husband sits next to his wife?

This classic problem uses PIE with circular permutations. The formula involves derangement-like reasoning applied in a circular setting.

---

## 7. Worked Example: Counting with Forbidden Properties

**Problem:** How many integers from 1 to 1000 are NOT divisible by 2, 3, or 5?

Let $A_2, A_3, A_5$ = multiples of 2, 3, 5 in $\{1, ..., 1000\}$.

| Set | Size |
|-----|:----:|
| $\|A_2\|$ | $\lfloor 1000/2 \rfloor = 500$ |
| $\|A_3\|$ | $\lfloor 1000/3 \rfloor = 333$ |
| $\|A_5\|$ | $\lfloor 1000/5 \rfloor = 200$ |
| $\|A_2 \cap A_3\|$ | $\lfloor 1000/6 \rfloor = 166$ |
| $\|A_2 \cap A_5\|$ | $\lfloor 1000/10 \rfloor = 100$ |
| $\|A_3 \cap A_5\|$ | $\lfloor 1000/15 \rfloor = 66$ |
| $\|A_2 \cap A_3 \cap A_5\|$ | $\lfloor 1000/30 \rfloor = 33$ |

$$|A_2 \cup A_3 \cup A_5| = 500+333+200 - 166-100-66 + 33 = 734$$

$$\text{Not divisible by 2, 3, or 5} = 1000 - 734 = 266$$

---

## 8. The Sieve of Eratosthenes Connection

The Sieve of Eratosthenes can be viewed as an application of PIE:

```
  Finding primes up to 30:
  
  Start: 2  3  4  5  6  7  8  9 10 11 12 13 14 15
        16 17 18 19 20 21 22 23 24 25 26 27 28 29 30
  
  Remove multiples of 2: ╳4, ╳6, ╳8, ╳10, ╳12, ...
  Remove multiples of 3: ╳9, ╳15, ╳21, ╳27, ...
  Remove multiples of 5: ╳25, ...
  
  (Only need primes up to √30 ≈ 5.5)
  
  Remaining primes: 2, 3, 5, 7, 11, 13, 17, 19, 23, 29
```

---

## 📝 Summary Table

| Application | What PIE Counts |
|-------------|----------------|
| Union of sets | $\|\bigcup A_i\|$ via alternating sums |
| Surjections | Functions hitting all targets |
| Derangements | Permutations with no fixed points |
| Euler's totient | Integers coprime to $n$ |
| Forbidden patterns | Objects avoiding specified properties |

---

## ❓ Quick Revision Questions

1. **How many integers from 1 to 200 are divisible by 4 or 6?**

2. **Compute the number of surjections from a 5-element set to a 3-element set.**

3. **Use PIE to compute $D_4$ (derangements of 4).**

4. **Compute $\phi(30)$.**

5. **Among 100 students, 40 take math, 30 take physics, 20 take chemistry, 15 take math and physics, 10 take math and chemistry, 8 take physics and chemistry, 5 take all three. How many take none?**

6. **State the general Inclusion-Exclusion formula for $n$ sets.**

---

[← Previous: Binomial Theorem](04-binomial-theorem.md) | [Back to README](../README.md) | [Next: Generating Functions →](06-generating-functions.md)
