# Chapter 1: Riemann Sums and Partitions

[← Previous: Unit 6 — Higher-Order Derivatives](../06-Differentiation/05-higher-order-derivatives.md) | [Back to README](../README.md) | [Next: Upper and Lower Integrals →](02-upper-and-lower-integrals.md)

---

## Overview

The **Riemann integral** formalizes the notion of "area under a curve." We begin with **partitions** that divide $[a,b]$ into subintervals and **Riemann sums** that approximate the integral using sample points.

---

## 1.1 Partitions

**Definition 1.1.** A **partition** $P$ of $[a,b]$ is a finite set:

$$P = \{a = x_0 < x_1 < x_2 < \cdots < x_n = b\}$$

The **mesh** (or norm) of $P$ is $\|P\| = \max_{1 \leq k \leq n}(x_k - x_{k-1})$.

A partition $Q$ **refines** $P$ (written $Q \supseteq P$) if $P \subseteq Q$ (more points).

```
  A partition of [0, 3]:

  P = {0, 0.5, 1.5, 2, 3}:
  
  ├────┼────────┼───┼──────┤
  0   0.5     1.5  2      3
    Δx₁  Δx₂  Δx₃   Δx₄
    0.5   1.0  0.5   1.0

  ||P|| = max(0.5, 1.0, 0.5, 1.0) = 1.0

  Refinement Q = P ∪ {1} = {0, 0.5, 1, 1.5, 2, 3}:
  ├────┼────┼────┼───┼──────┤
  0   0.5   1  1.5  2      3
  (more points, smaller intervals)
```

---

## 1.2 Riemann Sums

**Definition 1.2.** Given $f: [a,b] \to \mathbb{R}$, a partition $P = \{x_0, \ldots, x_n\}$, and **tags** $t_k \in [x_{k-1}, x_k]$, the **Riemann sum** is:

$$S(f, P, t) = \sum_{k=1}^{n} f(t_k)(x_k - x_{k-1}) = \sum_{k=1}^{n} f(t_k)\Delta x_k$$

```
  Riemann sum = total area of rectangles:

  f(x) ▲
       │    ┌──┐┌─┐
       │ ┌──┤  ││ │┌───┐
       │ │  │  ││ ││   │
       │ │  │  ││ ││   │
       │ │  │  ││ ││   │
       └─┼──┼──┼┼─┼┼───┼──► x
         x₀ x₁ x₂ x₃  x₄
         
  Height = f(tₖ)  at chosen tag point
  Width = Δxₖ = xₖ - xₖ₋₁
  Area of k-th rectangle = f(tₖ)·Δxₖ
```

**Special Riemann sums:**
| Type | Tags $t_k$ |
|------|-----------|
| Left sum | $t_k = x_{k-1}$ |
| Right sum | $t_k = x_k$ |
| Midpoint sum | $t_k = (x_{k-1} + x_k)/2$ |

---

## 1.3 Upper and Lower Sums

**Definition 1.3.** For bounded $f: [a,b] \to \mathbb{R}$ and partition $P$:

$$M_k = \sup_{[x_{k-1}, x_k]} f, \qquad m_k = \inf_{[x_{k-1}, x_k]} f$$

$$U(f, P) = \sum_{k=1}^n M_k \Delta x_k \quad\text{(upper sum)}$$
$$L(f, P) = \sum_{k=1}^n m_k \Delta x_k \quad\text{(lower sum)}$$

For any tags: $L(f, P) \leq S(f, P, t) \leq U(f, P)$.

```
  Upper sum (circumscribed rectangles):

  f(x) ▲ ┌──────┐
       │ │      │┌──┐
       │ │  ╱╲  ││  │
       │ │╱    ╲││  │
       │ │      ╲│  │
       └─┼──────┼┼──┼──► x

  Lower sum (inscribed rectangles):

  f(x) ▲
       │      ╱╲
       │    ╱    ╲
       │  ╱  ┌──┐╲
       │╱┌──┐│  │ ╲
       │ │  ││  │
       └─┼──┼┼──┼──► x

  L(f,P) ≤ ∫f ≤ U(f,P)
```

---

## 1.4 Refinement Lemma

**Lemma 1.4.** If $Q$ refines $P$ ($Q \supseteq P$), then:

$$L(f, P) \leq L(f, Q) \leq U(f, Q) \leq U(f, P)$$

Refinement **increases** lower sums and **decreases** upper sums.

**Proof.** Adding one point $x^*$ to $P$ in $[x_{k-1}, x_k]$ splits one term:

$$m_k \Delta x_k \leq m_k' \Delta x_k' + m_k'' \Delta x_k''$$

where $m_k', m_k'' \geq m_k$ (sup over smaller set $\geq$ sup over larger set... actually inf is $\leq$, reversed for infimum). So the lower sum increases. $\blacksquare$

---

## 1.5 Key Inequality

**Lemma 1.5.** For ANY two partitions $P_1, P_2$:

$$L(f, P_1) \leq U(f, P_2)$$

**Proof.** Let $Q = P_1 \cup P_2$ (common refinement). Then:
$$L(f, P_1) \leq L(f, Q) \leq U(f, Q) \leq U(f, P_2). \quad\blacksquare$$

---

## Summary Table

| Concept | Definition |
|---------|-----------|
| Partition $P$ | $\{a = x_0 < x_1 < \cdots < x_n = b\}$ |
| Mesh $\\|P\\|$ | $\max \Delta x_k$ |
| Riemann sum | $\sum f(t_k)\Delta x_k$ |
| Upper sum $U(f,P)$ | $\sum M_k \Delta x_k$ ($M_k = \sup f$ on $[x_{k-1}, x_k]$) |
| Lower sum $L(f,P)$ | $\sum m_k \Delta x_k$ ($m_k = \inf f$ on $[x_{k-1}, x_k]$) |
| Refinement | More points → $L$ increases, $U$ decreases |
| Key inequality | $L(f, P_1) \leq U(f, P_2)$ for all $P_1, P_2$ |

---

## Quick Revision Questions

1. **Define** a partition and its mesh. What does refinement mean?

2. **Compute** $U(f, P)$ and $L(f, P)$ for $f(x) = x^2$ on $[0, 1]$ with $P = \{0, 1/3, 2/3, 1\}$.

3. **Prove** the refinement lemma: adding points increases $L$ and decreases $U$.

4. **Prove** $L(f, P_1) \leq U(f, P_2)$ for any two partitions.

5. **What** is the relationship between Riemann sums and upper/lower sums?

6. **Compute** the left, right, and midpoint Riemann sums for $f(x) = \sin x$ on $[0, \pi]$ with 4 equal subintervals.

---

[← Previous: Unit 6 — Higher-Order Derivatives](../06-Differentiation/05-higher-order-derivatives.md) | [Back to README](../README.md) | [Next: Upper and Lower Integrals →](02-upper-and-lower-integrals.md)
