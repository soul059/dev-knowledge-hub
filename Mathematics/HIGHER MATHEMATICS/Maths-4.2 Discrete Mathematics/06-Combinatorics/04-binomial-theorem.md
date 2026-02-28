# Chapter 6.4: Binomial Theorem

[← Previous: Combinations](03-combinations.md) | [Back to README](../README.md) | [Next: Inclusion-Exclusion Principle →](05-inclusion-exclusion.md)

---

## 📋 Chapter Overview

The **Binomial Theorem** provides a formula for expanding $(x + y)^n$ and reveals deep connections between algebra, combinatorics, and probability.

---

## 1. Statement of the Theorem

$$(x + y)^n = \sum_{k=0}^{n} \binom{n}{k} x^{n-k} y^k$$

$$= \binom{n}{0}x^n + \binom{n}{1}x^{n-1}y + \binom{n}{2}x^{n-2}y^2 + \cdots + \binom{n}{n}y^n$$

---

## 2. Expansions for Small $n$

| $n$ | $(x+y)^n$ |
|:---:|-----------|
| 0 | $1$ |
| 1 | $x + y$ |
| 2 | $x^2 + 2xy + y^2$ |
| 3 | $x^3 + 3x^2y + 3xy^2 + y^3$ |
| 4 | $x^4 + 4x^3y + 6x^2y^2 + 4xy^3 + y^4$ |
| 5 | $x^5 + 5x^4y + 10x^3y^2 + 10x^2y^3 + 5xy^4 + y^5$ |

```
  Coefficients match Pascal's Triangle:
  
  n=0:       1
  n=1:      1 1
  n=2:     1 2 1
  n=3:    1 3 3 1
  n=4:   1 4 6 4 1
  n=5:  1 5 10 10 5 1
```

---

## 3. Proof (Combinatorial Argument)

$(x + y)^n = \underbrace{(x+y)(x+y)\cdots(x+y)}_{n \text{ factors}}$

To get the term $x^{n-k}y^k$:
- Choose $k$ factors to contribute $y$ (the rest contribute $x$)
- Number of ways = $\binom{n}{k}$

Therefore the coefficient of $x^{n-k}y^k$ is $\binom{n}{k}$. $\blacksquare$

---

## 4. Special Substitutions

### 4.1 $x = 1, y = 1$

$$(1 + 1)^n = 2^n = \sum_{k=0}^{n} \binom{n}{k}$$

**Sum of all binomial coefficients = $2^n$**

### 4.2 $x = 1, y = -1$

$$(1 - 1)^n = 0 = \sum_{k=0}^{n} (-1)^k \binom{n}{k}$$

**Alternating sum of binomial coefficients = 0**

This means: $\binom{n}{0} + \binom{n}{2} + \binom{n}{4} + \cdots = \binom{n}{1} + \binom{n}{3} + \binom{n}{5} + \cdots = 2^{n-1}$

### 4.3 $x = 1, y = x$

$$(1 + x)^n = \sum_{k=0}^{n} \binom{n}{k} x^k$$

### 4.4 Differentiate $(1+x)^n$

$$n(1+x)^{n-1} = \sum_{k=1}^{n} k \binom{n}{k} x^{k-1}$$

Set $x = 1$: $\sum_{k=0}^{n} k\binom{n}{k} = n \cdot 2^{n-1}$

---

## 5. Multinomial Theorem

Generalization to more than two terms:

$$(x_1 + x_2 + \cdots + x_m)^n = \sum_{\substack{k_1+k_2+\cdots+k_m = n \\ k_i \geq 0}} \binom{n}{k_1, k_2, \ldots, k_m} x_1^{k_1} x_2^{k_2} \cdots x_m^{k_m}$$

where the **multinomial coefficient** is:

$$\binom{n}{k_1, k_2, \ldots, k_m} = \frac{n!}{k_1! k_2! \cdots k_m!}$$

**Example:** $(x + y + z)^3$

$$= x^3 + y^3 + z^3 + 3x^2y + 3x^2z + 3y^2x + 3y^2z + 3z^2x + 3z^2y + 6xyz$$

---

## 6. Negative and Fractional Binomial (Newton's Generalization)

For any real $\alpha$ and $|x| < 1$:

$$(1 + x)^\alpha = \sum_{k=0}^{\infty} \binom{\alpha}{k} x^k$$

where:

$$\binom{\alpha}{k} = \frac{\alpha(\alpha-1)(\alpha-2)\cdots(\alpha-k+1)}{k!}$$

**Example:**

$$(1+x)^{-1} = 1 - x + x^2 - x^3 + \cdots = \sum_{k=0}^{\infty} (-1)^k x^k$$

$$(1+x)^{1/2} = 1 + \frac{1}{2}x - \frac{1}{8}x^2 + \frac{1}{16}x^3 - \cdots$$

---

## 7. Worked Examples

### Example 1: Find the coefficient of $x^4$ in $(2x - 3)^7$

Using the binomial theorem with $a = 2x$ and $b = -3$:

$$\binom{7}{3}(2x)^4(-3)^3 = 35 \cdot 16x^4 \cdot (-27) = -15120x^4$$

Coefficient = $-15{,}120$

### Example 2: Prove $\sum_{k=0}^{n} \binom{n}{k}^2 = \binom{2n}{n}$

**Proof:** Consider $(1+x)^n(1+x)^n = (1+x)^{2n}$.

Coefficient of $x^n$ on the left: $\sum_{k=0}^{n} \binom{n}{k}\binom{n}{n-k} = \sum_{k=0}^{n} \binom{n}{k}^2$

Coefficient of $x^n$ on the right: $\binom{2n}{n}$

Therefore they're equal. $\blacksquare$

---

## 8. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Binomial Theorem in Practice              │
  │                                                   │
  │  1. Probability (Binomial Distribution)          │
  │     P(k successes in n trials) = C(n,k)p^k q^{n-k}  │
  │                                                   │
  │  2. Approximations                               │
  │     (1 + x)^n ≈ 1 + nx for small x              │
  │                                                   │
  │  3. Error Analysis                               │
  │     Propagation of small errors uses binomial     │
  │     expansion                                     │
  │                                                   │
  │  4. Combinatorial Identities                     │
  │     Generating proofs via algebraic manipulation  │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Formula |
|---------|---------|
| Binomial Theorem | $(x+y)^n = \sum \binom{n}{k}x^{n-k}y^k$ |
| Sum of coefficients | $\sum \binom{n}{k} = 2^n$ |
| Alternating sum | $\sum (-1)^k \binom{n}{k} = 0$ |
| Weighted sum | $\sum k\binom{n}{k} = n \cdot 2^{n-1}$ |
| Multinomial | $(x_1 + \cdots + x_m)^n = \sum \frac{n!}{k_1!\cdots k_m!}\prod x_i^{k_i}$ |
| General binomial | $(1+x)^\alpha = \sum \binom{\alpha}{k}x^k$ for $\|x\| < 1$ |

---

## ❓ Quick Revision Questions

1. **Expand $(x + 2)^4$ using the Binomial Theorem.**

2. **Find the coefficient of $x^3y^5$ in $(x + y)^8$.**

3. **Evaluate $\sum_{k=0}^{6} \binom{6}{k}$.**

4. **Find the constant term in $\left(x + \frac{1}{x}\right)^{10}$.**

5. **Prove: $\binom{n}{0} + \binom{n}{2} + \binom{n}{4} + \cdots = 2^{n-1}$.**

6. **What is the coefficient of $x^2y^2z$ in $(x + y + z)^5$?**

---

[← Previous: Combinations](03-combinations.md) | [Back to README](../README.md) | [Next: Inclusion-Exclusion Principle →](05-inclusion-exclusion.md)
