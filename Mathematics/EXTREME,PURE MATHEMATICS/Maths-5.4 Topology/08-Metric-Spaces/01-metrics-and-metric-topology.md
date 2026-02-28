# Chapter 8.1 — Metrics and Metric Topology

[← Previous: Lindelöf Spaces](../07-Countability-Axioms/03-lindelof-spaces.md) | [Back to README](../README.md) | [Next: Complete Metric Spaces →](02-complete-metric-spaces.md)

---

## 1. Overview

Metric spaces are the most concrete and intuitive topological spaces. A **metric** assigns a distance between points, and the resulting topology inherits all the structure studied so far. This chapter defines metrics, the induced topology, and considers metrizability.

---

## 2. Definitions

> **Def 1.1 (Metric).** A **metric** on a set $X$ is a function $d : X \times X \to [0, \infty)$ satisfying for all $x, y, z \in X$:
>
> | Axiom | Condition |
> |-------|-----------|
> | **(M1)** Positivity | $d(x,y) \geq 0$; $d(x,y) = 0 \iff x = y$ |
> | **(M2)** Symmetry | $d(x,y) = d(y,x)$ |
> | **(M3)** Triangle inequality | $d(x,z) \leq d(x,y) + d(y,z)$ |

> **Def 1.2 (Open Ball).** $B(x, \epsilon) = \{y \in X : d(x,y) < \epsilon\}$.
>
> **Def 1.3 (Closed Ball).** $\overline{B}(x, \epsilon) = \{y \in X : d(x,y) \leq \epsilon\}$.
>
> **Def 1.4 (Sphere).** $S(x, \epsilon) = \{y \in X : d(x,y) = \epsilon\}$.

> **Def 1.5 (Metric Topology).** The topology induced by $d$ has basis $\{B(x, \epsilon) : x \in X, \epsilon > 0\}$.

```
  Open ball B(x, ε):
  ┌───────────────────┐
  │     ·  ·  ·       │
  │   ·          ·     │
  │  ·     • x    ·    │
  │   ·    ↕ε    ·     │
  │     ·  ·  ·       │
  │                    │
  └───────────────────┘
  All points within distance ε of x
```

---

## 3. Standard Examples

| Space | Metric | Notation |
|-------|--------|----------|
| Real line | $d(x,y) = \|x-y\|$ | $(\mathbb{R}, d)$ |
| Euclidean $n$-space | $d_2(x,y) = \sqrt{\sum (x_i-y_i)^2}$ | $(\mathbb{R}^n, d_2)$ |
| Taxicab / Manhattan | $d_1(x,y) = \sum \|x_i-y_i\|$ | $(\mathbb{R}^n, d_1)$ |
| Max / Chebyshev | $d_\infty(x,y) = \max \|x_i-y_i\|$ | $(\mathbb{R}^n, d_\infty)$ |
| Discrete metric | $d(x,y) = \begin{cases}0 & x=y \\ 1 & x \ne y\end{cases}$ | — |
| $p$-adic metric | $d_p(x,y) = p^{-v_p(x-y)}$ | $(\mathbb{Q}, d_p)$ |
| Function space | $d(f,g) = \sup\|f(t)-g(t)\|$ | $C([0,1])$ |

**Note:** $d_1, d_2, d_\infty$ on $\mathbb{R}^n$ all induce the **same** topology (the standard Euclidean topology).

---

## 4. Equivalent Metrics

> **Def 1.6.** Metrics $d$ and $d'$ on $X$ are **topologically equivalent** if they induce the same topology.

> **Thm 1.7.** $d$ and $d'$ are equivalent iff for every $x$ and $\epsilon > 0$, there exist $\delta_1, \delta_2 > 0$ with $B_d(x,\delta_1) \subseteq B_{d'}(x,\epsilon)$ and $B_{d'}(x,\delta_2) \subseteq B_d(x,\epsilon)$.

> **Def 1.8.** Metrics $d, d'$ are **strongly equivalent** if there exist constants $c, C > 0$ with $c \cdot d \leq d' \leq C \cdot d$. Strong equivalence ⟹ topological equivalence.

**Example on $\mathbb{R}^n$:**
$$d_\infty \leq d_2 \leq d_1 \leq n \cdot d_\infty$$
so all three are strongly (hence topologically) equivalent.

---

## 5. The Standard Bounded Metric

> **Def 1.9.** Given metric $d$, define $\bar{d}(x,y) = \min(d(x,y), 1)$. Then $\bar{d}$ is a metric equivalent to $d$.

This trick is useful for infinite products: define $D(x,y) = \sup_n \frac{\bar{d}_n(x_n, y_n)}{n}$ on $\prod X_n$ to metrize the product topology (when countably many factors).

---

## 6. Metrizability

> **Def 1.10.** A topological space is **metrizable** if its topology is induced by some metric.

```
Metrizable? Hierarchy:
  Metrizable ──→ Normal (T₄) ──→ Regular (T₃) ──→ Hausdorff (T₂)
  Metrizable ──→ First countable
  Metrizable ──→ Paracompact
```

**Key metrization theorems:**
| Theorem | Condition |
|---------|-----------|
| Urysohn metrization | 2nd countable + T₃ ⟹ metrizable |
| Nagata–Smirnov | T₃ + σ-locally finite basis ⟹ metrizable |
| Bing | T₃ + σ-discrete basis ⟹ metrizable |

**Non-metrizable examples:** Sorgenfrey line (separable, not 2nd countable — but first countable); $\mathbb{R}$ with cofinite topology; $\omega_1$ with order topology.

---

## 7. Continuity in Metric Spaces

> **Thm 1.11.** $f : (X, d_X) \to (Y, d_Y)$ is continuous iff $\forall x \in X$, $\forall \epsilon > 0$, $\exists \delta > 0$ with $d_X(a,x) < \delta \Rightarrow d_Y(f(a), f(x)) < \epsilon$.

This is the classical $\epsilon$-$\delta$ definition — which is equivalent to the general topological definition when both spaces are metrizable.

**Uniform continuity:** $\delta$ depends only on $\epsilon$, not on $x$. This is a metric (not topological) concept.

---

## 8. Summary Table

| Concept | Details |
|---------|---------|
| Metric | Distance function satisfying M1–M3 |
| Open ball | $B(x,\epsilon) = \{y : d(x,y) < \epsilon\}$ |
| Metric topology | Basis = all open balls |
| Equivalent metrics | Same topology |
| Strongly equivalent | $c \cdot d \leq d' \leq C \cdot d$ |
| Metrizable | Topology comes from some metric |
| $\epsilon$-$\delta$ | Recovers classical continuity |

---

## 9. Quick Revision Questions

1. State the metric axioms (M1–M3) and verify them for the discrete metric.

2. Prove that $d_1, d_2, d_\infty$ are strongly equivalent on $\mathbb{R}^n$.

3. Define topological equivalence of metrics. Give an example of equivalent metrics that are not strongly equivalent.

4. What is the standard bounded metric? Why is it useful for products?

5. State the Urysohn Metrization Theorem and give an example of a non-metrizable space.

6. Derive the $\epsilon$-$\delta$ definition of continuity from the topological definition.

---

[← Previous: Lindelöf Spaces](../07-Countability-Axioms/03-lindelof-spaces.md) | [Back to README](../README.md) | [Next: Complete Metric Spaces →](02-complete-metric-spaces.md)
