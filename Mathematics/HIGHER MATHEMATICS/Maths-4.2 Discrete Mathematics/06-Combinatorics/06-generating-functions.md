# Chapter 6.6: Generating Functions

[← Previous: Inclusion-Exclusion Principle](05-inclusion-exclusion.md) | [Back to README](../README.md) | [Next: Recurrence Relations →](../07-Recurrence-Relations/01-recurrence-relations.md)

---

## 📋 Chapter Overview

**Generating functions** encode a sequence of numbers as coefficients of a formal power series. This transforms combinatorial problems into algebraic ones, providing a powerful unified framework for counting, solving recurrences, and proving identities.

---

## 1. Definition

The **ordinary generating function (OGF)** of a sequence $a_0, a_1, a_2, \ldots$ is:

$$G(x) = \sum_{n=0}^{\infty} a_n x^n = a_0 + a_1 x + a_2 x^2 + a_3 x^3 + \cdots$$

We don't worry about convergence — these are **formal** power series.

---

## 2. Basic Examples

| Sequence | Generating Function |
|----------|:-------------------:|
| $1, 1, 1, 1, \ldots$ | $\frac{1}{1-x}$ |
| $1, 0, 0, 0, \ldots$ | $1$ |
| $0, 1, 0, 0, \ldots$ | $x$ |
| $1, 2, 3, 4, \ldots$ | $\frac{1}{(1-x)^2}$ |
| $1, c, c^2, c^3, \ldots$ | $\frac{1}{1-cx}$ |
| $\binom{n}{0}, \binom{n}{1}, \ldots, \binom{n}{n}$ | $(1+x)^n$ |
| $1, 1, \frac{1}{2!}, \frac{1}{3!}, \ldots$ | $e^x$ |

**Key identity:**

$$\frac{1}{1-x} = 1 + x + x^2 + x^3 + \cdots = \sum_{n=0}^{\infty} x^n$$

---

## 3. Operations on Generating Functions

If $A(x) = \sum a_n x^n$ and $B(x) = \sum b_n x^n$:

| Operation | Result | Sequence |
|-----------|--------|----------|
| $A(x) + B(x)$ | $\sum (a_n + b_n)x^n$ | Term-by-term sum |
| $c \cdot A(x)$ | $\sum (c \cdot a_n)x^n$ | Scalar multiplication |
| $x \cdot A(x)$ | $\sum a_{n-1}x^n$ | Right shift |
| $A'(x)$ | $\sum (n+1)a_{n+1}x^n$ | Differentiate |
| $A(x) \cdot B(x)$ | $\sum \left(\sum_{k=0}^{n} a_k b_{n-k}\right)x^n$ | **Convolution** |

---

## 4. Solving Counting Problems

### Example: Coin Change

How many ways to make change for $n$ cents using pennies (1¢), nickels (5¢), and dimes (10¢)?

$$G(x) = \frac{1}{(1-x)(1-x^5)(1-x^{10})}$$

The coefficient of $x^n$ in $G(x)$ gives the answer.

```
  Each factor represents a coin type:
  
  Pennies:  1/(1-x)    = 1 + x + x² + x³ + ...
            (0,1,2,3,... pennies)
  
  Nickels:  1/(1-x⁵)   = 1 + x⁵ + x¹⁰ + ...
            (0,1,2,... nickels)
  
  Dimes:    1/(1-x¹⁰)  = 1 + x¹⁰ + x²⁰ + ...
            (0,1,2,... dimes)
  
  Product → coefficient of xⁿ = ways to make n cents
```

### Example: Binary Strings

Number of binary strings of length $n$:

$$G(x) = \frac{1}{1-2x} = \sum_{n=0}^{\infty} 2^n x^n$$

Coefficient of $x^n$ is $2^n$. ✓

---

## 5. Solving Recurrence Relations with Generating Functions

### Example: Fibonacci Numbers

$F(0) = 0, F(1) = 1, F(n) = F(n-1) + F(n-2)$

Let $G(x) = \sum_{n=0}^{\infty} F_n x^n$.

From the recurrence:

$$G(x) - 0 - x = x \cdot G(x) + x^2 \cdot G(x)$$

$$G(x)(1 - x - x^2) = x$$

$$G(x) = \frac{x}{1 - x - x^2}$$

**Partial fractions** yield the closed form:

$$F_n = \frac{1}{\sqrt{5}}\left[\left(\frac{1+\sqrt{5}}{2}\right)^n - \left(\frac{1-\sqrt{5}}{2}\right)^n\right]$$

This is **Binet's formula**!

---

## 6. Exponential Generating Functions (EGF)

For sequences where order matters, use:

$$\hat{G}(x) = \sum_{n=0}^{\infty} a_n \frac{x^n}{n!}$$

| Sequence | EGF |
|----------|:---:|
| $1, 1, 1, \ldots$ | $e^x$ |
| $0, 1, 1, 1, \ldots$ | $e^x - 1$ |
| $n!$ (all $a_n = n!$) | $\frac{1}{1-x}$ |
| Derangements $D_n$ | $\frac{e^{-x}}{1-x}$ |

---

## 7. Useful Generating Function Techniques

| Technique | Formula |
|-----------|---------|
| Extract coefficient | $[x^n]G(x) = a_n$ |
| Partial fractions | Split rational $G(x)$ into simpler terms |
| Multiply GFs | Convolution for combining independent choices |
| Composition | $G(H(x))$ for substitution |

```
  Solving a recurrence with GFs — workflow:
  
  ┌──────────────────────────────────────────┐
  │  1. Define G(x) = Σ aₙ xⁿ              │
  │  2. Multiply recurrence by xⁿ, sum      │
  │  3. Express in terms of G(x)            │
  │  4. Solve for G(x) algebraically        │
  │  5. Use partial fractions               │
  │  6. Extract [xⁿ] for closed form        │
  └──────────────────────────────────────────┘
```

---

## 8. Real-World Applications

```
  ┌──────────────────────────────────────────────────┐
  │         Generating Functions in Practice          │
  │                                                   │
  │  1. Algorithm Analysis                           │
  │     Solve recurrences for running times           │
  │                                                   │
  │  2. Probability Generating Functions             │
  │     G(x) = E[x^X] encodes distribution           │
  │     E[X] = G'(1), Var(X) = G''(1)+G'(1)-G'(1)² │
  │                                                   │
  │  3. Partition Theory                             │
  │     Count integer partitions via GFs              │
  │                                                   │
  │  4. Coding Theory                                │
  │     Weight enumerators of codes                   │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| OGF | $G(x) = \sum a_n x^n$ |
| EGF | $\hat{G}(x) = \sum a_n x^n/n!$ |
| $\frac{1}{1-x}$ | Generates $1, 1, 1, \ldots$ |
| $\frac{1}{1-cx}$ | Generates $1, c, c^2, \ldots$ |
| Product | Convolution (combining choices) |
| Recurrences | Translate to algebra, solve for $G(x)$ |

---

## ❓ Quick Revision Questions

1. **Find the generating function for the sequence $1, 3, 9, 27, \ldots$.**

2. **What sequence does $\frac{1}{(1-x)^2}$ generate?**

3. **Use generating functions to solve $a_n = 3a_{n-1}$ with $a_0 = 2$.**

4. **Find the generating function for the number of ways to select $n$ objects from 3 types with at most 5 of each.**

5. **What is the coefficient of $x^{10}$ in $\frac{1}{(1-x)(1-x^2)(1-x^5)}$?**

6. **Explain the connection between generating functions and the Binomial Theorem.**

---

[← Previous: Inclusion-Exclusion Principle](05-inclusion-exclusion.md) | [Back to README](../README.md) | [Next: Recurrence Relations →](../07-Recurrence-Relations/01-recurrence-relations.md)
