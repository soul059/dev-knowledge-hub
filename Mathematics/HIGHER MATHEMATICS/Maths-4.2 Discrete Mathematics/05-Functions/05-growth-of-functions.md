# Chapter 5.5: Growth of Functions (Asymptotic Analysis)

[← Previous: Recursive Functions](04-recursive-functions.md) | [Back to README](../README.md) | [Next: Counting Principles →](../06-Combinatorics/01-counting-principles.md)

---

## 📋 Chapter Overview

**Asymptotic analysis** describes how functions behave as their input grows large. The notations $O$, $\Omega$, and $\Theta$ classify algorithms by their efficiency, abstracting away constants and lower-order terms.

---

## 1. Big-O Notation (Upper Bound)

$f(n) = O(g(n))$ if there exist constants $c > 0$ and $n_0$ such that:

$$f(n) \leq c \cdot g(n) \quad \text{for all } n \geq n_0$$

```
  f(n) = O(g(n)):
  
  ↑ value
  │        c·g(n) ╱
  │             ╱
  │           ╱
  │     f(n)╱
  │       ╱  ← f stays BELOW c·g(n) after n₀
  │     ╱
  │   ╱
  │──────────────────→ n
  │  n₀
```

**Meaning:** $f$ grows **no faster** than $g$ (ignoring constants).

---

## 2. Big-Omega Notation (Lower Bound)

$f(n) = \Omega(g(n))$ if there exist constants $c > 0$ and $n_0$ such that:

$$f(n) \geq c \cdot g(n) \quad \text{for all } n \geq n_0$$

**Meaning:** $f$ grows **at least as fast** as $g$.

---

## 3. Big-Theta Notation (Tight Bound)

$f(n) = \Theta(g(n))$ if:

$$f(n) = O(g(n)) \text{ and } f(n) = \Omega(g(n))$$

Equivalently: $c_1 \cdot g(n) \leq f(n) \leq c_2 \cdot g(n)$ for large $n$.

```
  f(n) = Θ(g(n)):
  
  ↑ value
  │     c₂·g(n) ╱
  │           ╱
  │    f(n) ╱   ← f is SANDWICHED
  │       ╱      between c₁·g(n)
  │     ╱        and c₂·g(n)
  │   ╱ c₁·g(n)
  │──────────────────→ n
  │  n₀
```

**Meaning:** $f$ grows at **exactly the same rate** as $g$.

---

## 4. Common Growth Rates

Ordered from slowest to fastest:

$$O(1) \subset O(\log n) \subset O(\sqrt{n}) \subset O(n) \subset O(n \log n) \subset O(n^2) \subset O(n^3) \subset O(2^n) \subset O(n!)$$

| Name | $f(n)$ | $n = 10$ | $n = 100$ | $n = 1000$ |
|------|:------:|:--------:|:---------:|:----------:|
| Constant | $1$ | 1 | 1 | 1 |
| Logarithmic | $\log_2 n$ | 3.3 | 6.6 | 10 |
| Linear | $n$ | 10 | 100 | 1000 |
| Linearithmic | $n \log n$ | 33 | 664 | 9966 |
| Quadratic | $n^2$ | 100 | 10,000 | 1,000,000 |
| Cubic | $n^3$ | 1,000 | 1,000,000 | $10^9$ |
| Exponential | $2^n$ | 1,024 | $1.27 \times 10^{30}$ | ∞ |
| Factorial | $n!$ | 3,628,800 | ∞ | ∞ |

```
  Growth Rate Comparison (approximate shape):
  
  ↑          n!  2ⁿ  n²   n log n   n   log n  1
  │         │   │   ╱     ╱        ╱   ╱
  │         │   │  ╱    ╱        ╱   ╱
  │         │   │╱    ╱        ╱  ╱
  │         │  ╱    ╱        ╱ ╱
  │         │╱    ╱        ╱╱
  │        ╱    ╱       ╱╱──────────────
  │      ╱    ╱      ╱╱
  │    ╱    ╱     ╱╱
  │  ╱    ╱   ╱╱
  │╱    ╱ ╱╱
  └──────────────────────────────────→ n
```

---

## 5. Proving Big-O

### Example 1: $3n^2 + 5n + 2 = O(n^2)$

**Proof:** For $n \geq 1$:
$$3n^2 + 5n + 2 \leq 3n^2 + 5n^2 + 2n^2 = 10n^2$$

Choose $c = 10, n_0 = 1$. ✓

### Example 2: $\log_2(n!) = O(n \log n)$

**Proof:** $n! \leq n^n$, so $\log_2(n!) \leq \log_2(n^n) = n \log_2 n$.

Choose $c = 1, n_0 = 1$. ✓

### Example 3: $2^n \neq O(n^k)$ for any fixed $k$

Exponential growth eventually dominates any polynomial.

---

## 6. Useful Properties

| Rule | Statement |
|------|----------|
| Sum | $O(f) + O(g) = O(\max(f, g))$ |
| Product | $O(f) \cdot O(g) = O(f \cdot g)$ |
| Constant | $c \cdot O(f) = O(f)$ |
| Transitivity | $f = O(g)$ and $g = O(h)$ implies $f = O(h)$ |
| Polynomial | $a_k n^k + \cdots + a_0 = \Theta(n^k)$ |
| Logarithm base | $\log_a n = \Theta(\log_b n)$ (base doesn't matter) |

---

## 7. Algorithm Complexity Examples

| Algorithm | Time Complexity | Class |
|-----------|:-:|------|
| Array access | $O(1)$ | Constant |
| Binary search | $O(\log n)$ | Logarithmic |
| Linear search | $O(n)$ | Linear |
| Merge sort | $O(n \log n)$ | Linearithmic |
| Bubble sort | $O(n^2)$ | Quadratic |
| Matrix multiply | $O(n^3)$ | Cubic |
| Subset enumeration | $O(2^n)$ | Exponential |
| Permutation enumeration | $O(n!)$ | Factorial |

---

## 8. Little-o and Little-omega

| Notation | Meaning | Definition |
|----------|---------|-----------|
| $f = o(g)$ | $f$ grows **strictly slower** than $g$ | $\lim_{n \to \infty} f(n)/g(n) = 0$ |
| $f = \omega(g)$ | $f$ grows **strictly faster** than $g$ | $\lim_{n \to \infty} f(n)/g(n) = \infty$ |

**Examples:**
- $n = o(n^2)$ ← $n$ grows strictly slower than $n^2$
- $n^2 = o(2^n)$ ← quadratic grows strictly slower than exponential
- $n \log n = \omega(n)$ ← $n \log n$ grows strictly faster than $n$

---

## 9. Real-World Significance

```
  ┌──────────────────────────────────────────────────┐
  │         Why Asymptotic Analysis Matters           │
  │                                                   │
  │  Time to process n = 1,000,000 items:            │
  │                                                   │
  │  O(n)       →  ~1 second                         │
  │  O(n log n) →  ~20 seconds                       │
  │  O(n²)      →  ~12 days                          │
  │  O(2ⁿ)      →  longer than age of universe!      │
  │                                                   │
  │  The DIFFERENCE between polynomial and            │
  │  exponential is the difference between            │
  │  "feasible" and "impossible."                    │
  │                                                   │
  │  This is the foundation of the P vs NP problem!  │
  └──────────────────────────────────────────────────┘
```

---

## 📝 Summary Table

| Notation | Meaning | Analogy |
|----------|---------|---------|
| $f = O(g)$ | $f \leq c \cdot g$ eventually | $f \leq g$ |
| $f = \Omega(g)$ | $f \geq c \cdot g$ eventually | $f \geq g$ |
| $f = \Theta(g)$ | Both $O(g)$ and $\Omega(g)$ | $f = g$ |
| $f = o(g)$ | $f/g \to 0$ | $f < g$ |
| $f = \omega(g)$ | $f/g \to \infty$ | $f > g$ |

---

## ❓ Quick Revision Questions

1. **Prove that $5n^3 + 2n^2 + 1 = O(n^3)$.**

2. **Rank these in increasing order of growth: $n!, 2^n, n^2, \log n, n \log n, n^3, 1$.**

3. **Is $2^{n+1} = O(2^n)$? Prove or disprove.**

4. **If $f(n) = O(n^2)$ and $g(n) = O(n^3)$, what is $f(n) \cdot g(n)$?**

5. **Show that $\log_2 n = \Theta(\log_{10} n)$.**

6. **An algorithm takes $T(n) = 3n^2 + 100n + 500$ operations. What is its Big-O complexity?**

---

[← Previous: Recursive Functions](04-recursive-functions.md) | [Back to README](../README.md) | [Next: Counting Principles →](../06-Combinatorics/01-counting-principles.md)
