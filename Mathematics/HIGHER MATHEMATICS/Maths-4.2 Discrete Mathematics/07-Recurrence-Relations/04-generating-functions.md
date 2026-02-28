# Chapter 7.4: Generating Functions for Recurrences

[← Previous: Non-Homogeneous Recurrence](03-non-homogeneous-recurrence.md) | [Back to README](../README.md) | [Next: Solving Recurrences →](05-solving-recurrences.md)

---

## 📋 Chapter Overview

Generating functions provide a **universal method** for solving linear recurrences — both homogeneous and non-homogeneous. Instead of guessing solution forms, we transform the recurrence into an algebraic equation, solve it, and extract the coefficients.

---

## 1. The GF Method — Step by Step

```
  ┌──────────────────────────────────────────────────┐
  │  Generating Function Method for Recurrences       │
  │                                                    │
  │  1. Define G(x) = Σ aₙ xⁿ                        │
  │  2. Multiply both sides of recurrence by xⁿ       │
  │  3. Sum over all valid n                           │
  │  4. Express sums in terms of G(x)                 │
  │  5. Solve algebraically for G(x)                  │
  │  6. Decompose G(x) using partial fractions        │
  │  7. Extract [xⁿ] = aₙ (the coefficient of xⁿ)    │
  └──────────────────────────────────────────────────┘
```

### Key identities for step 4:

$$\sum_{n=1}^{\infty} a_{n-1} x^n = x \cdot G(x)$$

$$\sum_{n=2}^{\infty} a_{n-2} x^n = x^2 \cdot G(x)$$

$$\sum_{n=0}^{\infty} a_n x^n = G(x)$$

---

## 2. Example: First-Order Recurrence

Solve $a_n = 3a_{n-1} + 2$, $a_0 = 1$.

**Step 1.** $G(x) = \sum_{n=0}^{\infty} a_n x^n$

**Step 2-3.** Multiply by $x^n$ and sum for $n \geq 1$:

$$\sum_{n=1}^{\infty} a_n x^n = 3\sum_{n=1}^{\infty} a_{n-1} x^n + \sum_{n=1}^{\infty} 2x^n$$

$$G(x) - a_0 = 3x \cdot G(x) + \frac{2x}{1-x}$$

**Step 4-5.** $G(x)(1-3x) = 1 + \frac{2x}{1-x}$

$$G(x) = \frac{1}{1-3x} + \frac{2x}{(1-x)(1-3x)}$$

**Partial fractions** on second term: $\frac{2x}{(1-x)(1-3x)} = \frac{-1}{1-x} + \frac{1}{1-3x}$

$$G(x) = \frac{2}{1-3x} - \frac{1}{1-x}$$

**Step 6-7.** Extract coefficients:

$$a_n = 2 \cdot 3^n - 1$$

**Verify:** $a_0 = 2-1 = 1$ ✓, $a_1 = 6-1 = 5 = 3(1)+2$ ✓

---

## 3. Example: Fibonacci Numbers

$F_n = F_{n-1} + F_{n-2}$, $F_0 = 0, F_1 = 1$.

Let $G(x) = \sum_{n=0}^{\infty} F_n x^n$.

For $n \geq 2$: multiply recurrence by $x^n$ and sum:

$$G(x) - F_0 - F_1 x = x(G(x) - F_0) + x^2 G(x)$$

$$G(x) - x = xG(x) + x^2 G(x)$$

$$G(x)(1 - x - x^2) = x$$

$$G(x) = \frac{x}{1-x-x^2}$$

**Partial fractions:** Factor $1-x-x^2 = -(x-\phi^{-1})(x-\hat\phi^{-1})$ where $\phi = \frac{1+\sqrt{5}}{2}$.

$$G(x) = \frac{1}{\sqrt{5}}\left(\frac{1}{1-\phi x} - \frac{1}{1-\hat\phi x}\right)$$

$$F_n = [x^n]G(x) = \frac{1}{\sqrt{5}}(\phi^n - \hat\phi^n)$$

This matches **Binet's formula**.

---

## 4. Example: Non-Homogeneous with Exponential

Solve $a_n = 2a_{n-1} + 2^n$, $a_0 = 0$.

$G(x) - 0 = 2x \cdot G(x) + \sum_{n=1}^{\infty} 2^n x^n = 2xG(x) + \frac{2x}{1-2x}$

$$G(x)(1-2x) = \frac{2x}{1-2x}$$

$$G(x) = \frac{2x}{(1-2x)^2}$$

Using the identity $\frac{1}{(1-cx)^2} = \sum_{n=0}^{\infty} (n+1)c^n x^n$:

$$\frac{2x}{(1-2x)^2} = \sum_{n=1}^{\infty} n \cdot 2^n x^n$$

$$\boxed{a_n = n \cdot 2^n}$$

---

## 5. Partial Fraction Decomposition

This is the critical algebraic step. Common cases:

### Distinct Linear Factors

$$\frac{P(x)}{(1-ax)(1-bx)} = \frac{A}{1-ax} + \frac{B}{1-bx}$$

Extract: $[x^n] = A \cdot a^n + B \cdot b^n$

### Repeated Linear Factor

$$\frac{P(x)}{(1-ax)^2} = \frac{A}{1-ax} + \frac{B}{(1-ax)^2}$$

Using $[x^n]\frac{1}{(1-ax)^k} = \binom{n+k-1}{k-1}a^n$:

$$[x^n] = A \cdot a^n + B(n+1) \cdot a^n$$

### Quick Reference

| GF Term | Coefficient of $x^n$ |
|---------|:--------------------:|
| $\frac{1}{1-ax}$ | $a^n$ |
| $\frac{1}{(1-ax)^2}$ | $(n+1)a^n$ |
| $\frac{1}{(1-ax)^3}$ | $\binom{n+2}{2}a^n$ |
| $\frac{x}{(1-ax)^2}$ | $na^{n-1}$ |

---

## 6. Catalan Numbers via GFs

The Catalan numbers satisfy: $C_n = \sum_{k=0}^{n-1} C_k C_{n-1-k}$, $C_0 = 1$.

Let $G(x) = \sum C_n x^n$. The convolution sum means:

$$G(x) = 1 + x \cdot G(x)^2$$

This is a **quadratic** in $G(x)$:

$$xG^2 - G + 1 = 0 \implies G(x) = \frac{1 - \sqrt{1-4x}}{2x}$$

Using the generalized binomial theorem:

$$C_n = [x^n]G(x) = \frac{1}{n+1}\binom{2n}{n}$$

---

## 7. Advantages and Limitations

| Aspect | GF Method |
|--------|-----------|
| Universality | Works for any linear recurrence |
| Non-linear | Can handle some (e.g., Catalan) |
| Difficulty | Partial fractions can be tedious |
| When to use | When undetermined coefficients fails, or for complicated $f(n)$ |
| Alternative | Characteristic equation is faster for simple cases |

---

## 8. Real-World Applications

```
  ┌───────────────────────────────────────────────┐
  │       Generating Functions in Practice         │
  │                                                │
  │  1. Probability                                │
  │     GF of PMF = probability generating         │
  │     function → E[X] = G'(1)                    │
  │                                                │
  │  2. Queueing Theory                            │
  │     Steady-state distributions via GFs          │
  │                                                │
  │  3. Enumeration                                │
  │     Lattice paths, partitions, tilings          │
  │                                                │
  │  4. Compiler Design                            │
  │     Ambiguity in grammars analyzed via GFs      │
  └───────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| $G(x) = \sum a_n x^n$ | Define generating function |
| Shift: $a_{n-1}$ → $xG(x)$ | Right-shift by multiplication |
| Solve algebraically | Get $G(x) = \text{rational function}$ |
| Partial fractions | Decompose into simple terms |
| Extract $[x^n]$ | Read off closed-form for $a_n$ |

---

## ❓ Quick Revision Questions

1. **Use GFs to solve $a_n = a_{n-1} + 1$, $a_0 = 0$.**

2. **Find the GF for the recurrence $a_n = 3a_{n-1}$, $a_0 = 2$. What is the closed form?**

3. **What is $[x^n]\frac{1}{(1-2x)^2}$?**

4. **How does the shift $a_{n-2}$ translate in GF terms?**

5. **Use GFs to derive the closed form for the Tower of Hanoi recurrence: $T_n = 2T_{n-1}+1$, $T_0 = 0$.**

---

[← Previous: Non-Homogeneous Recurrence](03-non-homogeneous-recurrence.md) | [Back to README](../README.md) | [Next: Solving Recurrences →](05-solving-recurrences.md)
