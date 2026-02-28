# Chapter 7.1: Linear Recurrence Relations

[← Previous: Generating Functions](../06-Combinatorics/06-generating-functions.md) | [Back to README](../README.md) | [Next: Homogeneous Recurrence →](02-homogeneous-recurrence.md)

---

## 📋 Chapter Overview

A **recurrence relation** expresses each term of a sequence in terms of earlier terms. They arise naturally in algorithm analysis, population modeling, and combinatorial counting. This chapter introduces the fundamentals: definitions, classification, and techniques for recognizing which method to apply.

---

## 1. Definition

A **recurrence relation** for a sequence $\{a_n\}$ is an equation that defines $a_n$ in terms of one or more previous terms $a_{n-1}, a_{n-2}, \ldots$

It consists of:
- **Recurrence equation** — the rule relating terms
- **Initial conditions** — enough starting values to determine the sequence uniquely

### Example

$$a_n = 3a_{n-1} + 2, \quad a_0 = 1$$

$$a_0 = 1, \quad a_1 = 5, \quad a_2 = 17, \quad a_3 = 53, \ldots$$

---

## 2. Order of a Recurrence

The **order** of a recurrence is the difference between the highest and lowest subscripts.

| Recurrence | Order |
|:----------:|:-----:|
| $a_n = 2a_{n-1}$ | 1 |
| $a_n = a_{n-1} + a_{n-2}$ | 2 |
| $a_n = a_{n-1} + a_{n-2} + a_{n-3}$ | 3 |
| $a_n = a_{n-1} \cdot a_{n-2}$ | 2 |

---

## 3. Linear Recurrence Relations

A recurrence is **linear** if $a_n$ is expressed as a **linear combination** of previous terms (possibly plus a function of $n$):

$$a_n = c_1 a_{n-1} + c_2 a_{n-2} + \cdots + c_k a_{n-k} + f(n)$$

where $c_1, c_2, \ldots, c_k$ are constants and $f(n)$ depends only on $n$.

```
  Classification of Linear Recurrence Relations:
  
  ┌─────────────────────────────────────┐
  │        Linear Recurrence            │
  │                                     │
  │   f(n) = 0?                         │
  │   ┌─── YES ──┐   ┌─── NO ──┐       │
  │   │          │   │          │       │
  │   ▼          │   ▼          │       │
  │  Homogeneous │  Non-Homo-  │       │
  │              │  geneous    │       │
  │              │              │       │
  │  Constant    │  Constant   │       │
  │  coeffs?     │  coeffs?    │       │
  │  YES → Ch.2  │  YES → Ch.3│       │
  └─────────────────────────────────────┘
```

### Linear Examples

| Recurrence | Type |
|:----------:|:----:|
| $a_n = 5a_{n-1} - 6a_{n-2}$ | Linear, homogeneous, constant coefficients, order 2 |
| $a_n = 2a_{n-1} + 3^n$ | Linear, non-homogeneous, constant coefficients, order 1 |
| $a_n = na_{n-1}$ | Linear, homogeneous, variable coefficients, order 1 |

### Non-Linear Examples

| Recurrence | Why Non-Linear |
|:----------:|:-------------:|
| $a_n = a_{n-1}^2$ | Squared term |
| $a_n = a_{n-1} \cdot a_{n-2}$ | Product of terms |
| $a_n = \sqrt{a_{n-1}}$ | Non-linear function |

---

## 4. Constant Coefficients

A linear recurrence has **constant coefficients** if $c_1, c_2, \ldots, c_k$ are all constants (not functions of $n$).

$$a_n = c_1 a_{n-1} + c_2 a_{n-2} + \cdots + c_k a_{n-k} + f(n)$$

These are the recurrences we can solve most systematically.

---

## 5. Famous Recurrence Relations

| Name | Recurrence | Initial | Closed Form |
|------|:----------:|:-------:|:-----------:|
| Fibonacci | $F_n = F_{n-1} + F_{n-2}$ | $F_0=0, F_1=1$ | $\frac{\phi^n - \hat\phi^n}{\sqrt 5}$ |
| Tower of Hanoi | $T_n = 2T_{n-1} + 1$ | $T_0 = 0$ | $2^n - 1$ |
| Factorial | $n! = n \cdot (n-1)!$ | $0! = 1$ | $n!$ |
| Catalan | $C_n = \sum_{k=0}^{n-1} C_k C_{n-1-k}$ | $C_0 = 1$ | $\frac{1}{n+1}\binom{2n}{n}$ |
| Merge Sort | $T(n) = 2T(n/2)+n$ | $T(1) = 0$ | $n \log_2 n$ |

where $\phi = \frac{1+\sqrt{5}}{2} \approx 1.618$ (golden ratio) and $\hat\phi = \frac{1-\sqrt{5}}{2}$.

---

## 6. Modeling with Recurrence Relations

### Example 1: Compound Interest

Balance $B_n$ after $n$ periods at rate $r$ with deposit $d$:

$$B_n = (1+r) \cdot B_{n-1} + d, \quad B_0 = P$$

- Linear, non-homogeneous, order 1, constant coefficients

### Example 2: Population with predation

$$P_n = 1.5 \cdot P_{n-1} - 0.2 \cdot P_{n-2}$$

- Linear, homogeneous, order 2, constant coefficients

### Example 3: Binary Strings without "00"

Let $a_n$ = number of binary strings of length $n$ with no two consecutive 0s.

If the string ends in 1: previous $n-1$ characters can be any valid string → $a_{n-1}$  
If the string ends in 0: must end in "10", previous $n-2$ characters valid → $a_{n-2}$

$$a_n = a_{n-1} + a_{n-2}, \quad a_1 = 2, \quad a_2 = 3$$

This is the Fibonacci sequence (shifted)!

---

## 7. Methods Overview

| Method | Applicable To | Chapter |
|--------|---------------|:-------:|
| Characteristic equation | Linear, homogeneous, constant coeff. | 7.2 |
| Undetermined coefficients | Linear, non-homogeneous, constant coeff. | 7.3 |
| Generating functions | Any linear recurrence | 7.4 |
| Iteration / Back-substitution | Simple recurrences | 7.5 |
| Master Theorem | Divide-and-conquer recurrences | 7.5 |

---

## 8. Checking Solutions

To verify a proposed closed form $a_n = g(n)$:

1. **Check initial conditions** — Does $g(0) = a_0$? Does $g(1) = a_1$?
2. **Substitute into recurrence** — Does $g(n) = c_1 g(n-1) + c_2 g(n-2) + \cdots + f(n)$?

### Example

Verify $a_n = 2^{n+1} - 1$ solves $a_n = 2a_{n-1} + 1, \; a_0 = 1$.

- $a_0 = 2^1 - 1 = 1$ ✓
- $2a_{n-1} + 1 = 2(2^n - 1) + 1 = 2^{n+1} - 2 + 1 = 2^{n+1} - 1 = a_n$ ✓

---

## 9. Real-World Applications

```
  ┌─────────────────────────────────────────────────┐
  │       Recurrences in Computer Science            │
  │                                                  │
  │  1. Algorithm Complexity                         │
  │     Merge Sort: T(n) = 2T(n/2) + O(n)           │
  │     Binary Search: T(n) = T(n/2) + O(1)         │
  │                                                  │
  │  2. Dynamic Programming                          │
  │     Fibonacci, shortest paths, knapsack          │
  │                                                  │
  │  3. Combinatorial Counting                       │
  │     Strings, tilings, lattice paths              │
  │                                                  │
  │  4. Financial Modeling                           │
  │     Compound interest, amortization              │
  │                                                  │
  │  5. Biology                                      │
  │     Population dynamics, gene frequencies        │
  └─────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| Recurrence relation | Defines $a_n$ in terms of earlier terms |
| Order | Difference between highest and lowest subscripts |
| Linear | $a_n$ is a linear combination of previous terms (+ $f(n)$) |
| Homogeneous | $f(n) = 0$ (no extra function of $n$) |
| Constant coefficients | All $c_i$ are constants, not functions of $n$ |
| Initial conditions | Starting values that determine the sequence uniquely |

---

## ❓ Quick Revision Questions

1. **Classify the recurrence $a_n = 4a_{n-1} - 4a_{n-2}$ by order, linearity, homogeneity, and coefficient type.**

2. **Write a recurrence relation for the number of ways to tile a $2 \times n$ board with $2 \times 1$ dominoes.**

3. **Is $a_n = a_{n-1} + n^2$ homogeneous or non-homogeneous? Why?**

4. **Verify that $a_n = 3 \cdot 2^n$ satisfies $a_n = 2a_{n-1}$ with $a_0 = 3$.**

5. **Write a recurrence for the number of binary strings of length $n$ that do not contain "111".**

---

[← Previous: Generating Functions](../06-Combinatorics/06-generating-functions.md) | [Back to README](../README.md) | [Next: Homogeneous Recurrence →](02-homogeneous-recurrence.md)
