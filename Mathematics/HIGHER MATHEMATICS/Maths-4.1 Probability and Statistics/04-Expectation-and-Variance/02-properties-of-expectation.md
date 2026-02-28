# Chapter 4.2: Properties of Expectation

[← Previous: Expected Value](01-expected-value.md) | [Back to README](../README.md) | [Next: Variance and Standard Deviation →](03-variance-and-standard-deviation.md)

---

## Overview

Expectation is a **linear operator** — it respects addition and scalar multiplication. This linearity makes it one of the most powerful tools in probability, allowing us to break complex problems into simpler pieces.

---

## 1. Linearity of Expectation

$$\boxed{E[aX + bY + c] = aE[X] + bE[Y] + c}$$

This holds for **any** random variables $X$ and $Y$, **regardless of dependence**.

### General Form

$$E\left[\sum_{i=1}^n a_i X_i + c\right] = \sum_{i=1}^n a_i E[X_i] + c$$

### Why Linearity is Remarkable

```
  ╔═══════════════════════════════════════════════════╗
  ║  Linearity works even when X and Y are DEPENDENT  ║
  ║                                                    ║
  ║  No independence assumption needed!                ║
  ║                                                    ║
  ║  E[X + Y] = E[X] + E[Y]       ALWAYS TRUE         ║
  ║  E[X · Y] = E[X] · E[Y]      ONLY IF INDEPENDENT  ║
  ╚═══════════════════════════════════════════════════╝
```

---

## 2. Complete List of Properties

| Property | Formula | Condition |
|----------|---------|-----------|
| Constant | $E[c] = c$ | Always |
| Scaling | $E[aX] = aE[X]$ | Always |
| Shift | $E[X + c] = E[X] + c$ | Always |
| Sum | $E[X + Y] = E[X] + E[Y]$ | Always |
| **Linearity** | $E[aX + bY + c] = aE[X] + bE[Y] + c$ | **Always** |
| Product | $E[XY] = E[X] \cdot E[Y]$ | **Only if independent** |
| Monotonicity | If $X \geq Y$ a.s., then $E[X] \geq E[Y]$ | Always |
| Triangle Inequality | $|E[X]| \leq E[|X|]$ | Always |
| Non-negativity | If $X \geq 0$, then $E[X] \geq 0$ | Always |

---

## 3. Proofs of Key Properties

### Proof: $E[X + Y] = E[X] + E[Y]$ (Discrete case)

$$E[X+Y] = \sum_x \sum_y (x+y) \cdot P(X=x, Y=y)$$
$$= \sum_x \sum_y x \cdot P(X=x, Y=y) + \sum_x \sum_y y \cdot P(X=x, Y=y)$$
$$= \sum_x x \sum_y P(X=x, Y=y) + \sum_y y \sum_x P(X=x, Y=y)$$
$$= \sum_x x \cdot P(X=x) + \sum_y y \cdot P(Y=y)$$
$$= E[X] + E[Y] \quad \blacksquare$$

### Proof: $E[XY] = E[X]E[Y]$ when independent

If $X \perp Y$: $P(X=x, Y=y) = P(X=x) \cdot P(Y=y)$

$$E[XY] = \sum_x \sum_y xy \cdot P(X=x, Y=y) = \sum_x \sum_y xy \cdot P(X=x) P(Y=y)$$
$$= \left(\sum_x x P(X=x)\right)\left(\sum_y y P(Y=y)\right) = E[X] \cdot E[Y] \quad \blacksquare$$

---

## 4. Power of Linearity — Worked Examples

### Example 1: Expected Value of Binomial

$X \sim \text{Binomial}(n, p)$: Number of successes in $n$ independent trials.

Write $X = X_1 + X_2 + \cdots + X_n$ where each $X_i \sim \text{Bernoulli}(p)$

$$E[X] = E[X_1] + E[X_2] + \cdots + E[X_n] = np$$

> No messy binomial coefficient calculation needed!

### Example 2: Hat Problem (Matching)

$n$ people throw their hats into a pile and each picks one at random. What is the expected number of people who get their own hat?

Let $X_i = 1$ if person $i$ gets their own hat, 0 otherwise.

$$E[X_i] = P(\text{person } i \text{ gets own hat}) = \frac{1}{n}$$

$$E[X] = E\left[\sum_{i=1}^n X_i\right] = \sum_{i=1}^n E[X_i] = n \cdot \frac{1}{n} = 1$$

Answer: **Expected 1 person**, regardless of $n$! (Even for $n = 1{,}000{,}000$)

### Example 3: Coupon Collector Problem

Collect $n$ different coupons. Each purchase gives one coupon uniformly at random. How many purchases needed to collect all $n$?

Let $T_i$ = purchases needed to get the $i$-th new coupon (after having $i-1$).

$$P(\text{new coupon} \mid i-1 \text{ collected}) = \frac{n-(i-1)}{n}$$

$T_i \sim \text{Geometric}\left(\frac{n-i+1}{n}\right)$, so $E[T_i] = \frac{n}{n-i+1}$

$$E[T] = \sum_{i=1}^n E[T_i] = \sum_{i=1}^n \frac{n}{n-i+1} = n \sum_{k=1}^{n} \frac{1}{k} = nH_n$$

where $H_n = 1 + \frac{1}{2} + \frac{1}{3} + \cdots + \frac{1}{n} \approx \ln n + 0.5772$

For $n = 50$ stickers: $E[T] \approx 50 \times 4.499 \approx 225$ purchases.

### Example 4: Sum of Two Dice

$$E[X+Y] = E[X] + E[Y] = 3.5 + 3.5 = 7$$

---

## 5. Indicator Random Variables Technique

The **indicator method** decomposes a count into a sum of 0/1 variables:

$$X = \sum_{i=1}^n \mathbb{1}_{A_i}$$

Then:

$$E[X] = \sum_{i=1}^n P(A_i)$$

```
  Problem: Count how many events happen
  
  Step 1: Define X_i = 1 if event i occurs
  Step 2: X = X_1 + X_2 + ... + X_n
  Step 3: E[X] = P(A_1) + P(A_2) + ... + P(A_n)
  
  ┌────────────────────────────────────────────┐
  │  No need to worry about dependence between │
  │  the X_i's — linearity handles it!         │
  └────────────────────────────────────────────┘
```

### Example 5: Expected Inversions in a Random Permutation

A random permutation of $\{1, 2, \ldots, n\}$. An **inversion** is a pair $(i,j)$ where $i < j$ but $\pi(i) > \pi(j)$.

For each pair $(i,j)$: $P(\text{inversion}) = 1/2$ (by symmetry)

$$E[\text{inversions}] = \binom{n}{2} \cdot \frac{1}{2} = \frac{n(n-1)}{4}$$

---

## 6. E[X] Does NOT Tell You E[g(X)] in General

**Jensen's Inequality** for convex functions:

$$E[g(X)] \geq g(E[X]) \quad \text{if } g \text{ is convex}$$
$$E[g(X)] \leq g(E[X]) \quad \text{if } g \text{ is concave}$$

```
  Convex function g:            Concave function g:
  
  g(x)                          g(x)
  │       ╱                     │   ╱──────╲
  │     ╱                       │ ╱          ╲
  │   ╱   chord above curve     │╱  chord below
  │ ╱                           │
  └──────────► x                └──────────► x
  
  E[g(X)] ≥ g(E[X])            E[g(X)] ≤ g(E[X])
```

### Common Consequence

$$E[X^2] \geq (E[X])^2$$

because $g(x) = x^2$ is convex. (This fact is central to variance!)

---

## 7. Real-World Applications

| Application | How Linearity Helps |
|------------|---------------------|
| **Portfolio returns** | $E[\text{portfolio}] = \sum w_i E[R_i]$, no need for joint distribution |
| **Quality control** | Expected defects = $n \cdot p_{\text{defect}}$ |
| **Network design** | Expected hops = sum of per-link expectations |
| **Algorithms** | Expected runtime of randomized algorithms |
| **Genetics** | Expected number of shared alleles |
| **Sports analytics** | Expected points = sum of play-by-play expectations |

---

## Summary Table

| Property | Statement | Needs Independence? |
|----------|-----------|:-------------------:|
| $E[c]$ | $= c$ | No |
| $E[aX + b]$ | $= aE[X] + b$ | No |
| $E[X+Y]$ | $= E[X] + E[Y]$ | **No** |
| $E[XY]$ | $= E[X]E[Y]$ | **Yes** |
| Indicator sum | $E[\sum \mathbb{1}_{A_i}] = \sum P(A_i)$ | No |
| Jensen (convex $g$) | $E[g(X)] \geq g(E[X])$ | No |
| Monotonicity | $X \geq Y \Rightarrow E[X] \geq E[Y]$ | No |

---

## Quick Revision Questions

1. Given $E[X] = 3$ and $E[Y] = 5$, find $E[2X - 3Y + 7]$.

2. In a class of 30 students, each independently has probability 0.6 of passing an exam. What is the expected number of students who pass? (Use indicator variables.)

3. **Coupon collector**: If there are 6 types of prizes in cereal boxes, what is the expected number of boxes you must buy to collect all 6?

4. A random variable $X$ has $E[X] = 4$ and $E[X^2] = 20$. Is it possible that $P(X < 0) > 0$? How do you know $E[X^2] \geq (E[X])^2$?

5. Given $X$ and $Y$ are independent with $E[X] = 2$, $E[Y] = 3$, $E[X^2] = 6$, $E[Y^2] = 11$. Find $E[(X+Y)^2]$.

6. 10 married couples are seated randomly in a row. What is the expected number of couples sitting next to each other? (Use indicator variables.)

---

[← Previous: Expected Value](01-expected-value.md) | [Back to README](../README.md) | [Next: Variance and Standard Deviation →](03-variance-and-standard-deviation.md)
