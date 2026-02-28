# Chapter 2: Open and Closed Sets in Metric Spaces

[← Previous: Definition and Examples](01-definition-and-examples.md) | [Back to README](../README.md) | [Next: Convergence →](03-convergence.md)

---

## Overview

The concepts of open and closed sets generalize from $\mathbb{R}$ to any metric space using the metric $d$ in place of $|\cdot|$. The **topology** of a metric space — its open sets — determines convergence, continuity, and compactness.

---

## 2.1 Open Balls

**Definition 2.1.** The **open ball** of radius $r > 0$ centered at $x$ in $(X, d)$:

$$B(x, r) = B_r(x) = \{y \in X : d(x, y) < r\}$$

The **closed ball:** $\overline{B}(x, r) = \{y \in X : d(x, y) \leq r\}$.

```
  Open ball B(x, r) in a metric space:

       ╭─────────╮
      │  ●        │     r
      │     x     │ ←──►
      │           │
       ╰─────────╯
  All y with d(x,y) < r
```

---

## 2.2 Open Sets

**Definition 2.2.** $U \subseteq X$ is **open** if:

$$\forall\, x \in U,\; \exists\, r > 0: B(x, r) \subseteq U$$

**Theorem 2.2.** The open sets of $(X, d)$ satisfy:
1. $\emptyset$ and $X$ are open
2. Arbitrary unions of open sets are open
3. Finite intersections of open sets are open

**Proof of (3).** If $U_1, \ldots, U_n$ open and $x \in \bigcap U_k$, then $\exists\, r_k > 0: B(x, r_k) \subseteq U_k$. Take $r = \min(r_1, \ldots, r_n) > 0$. Then $B(x, r) \subseteq \bigcap U_k$. $\blacksquare$

**Fact.** Every open ball $B(x, r)$ is an open set.

---

## 2.3 Closed Sets

**Definition 2.3.** $F \subseteq X$ is **closed** if $X \setminus F$ is open.

Equivalently: $F$ is closed $\iff$ $F$ contains all its limit points $\iff$ every convergent sequence in $F$ has its limit in $F$.

**Properties:** $\emptyset$ and $X$ are closed; arbitrary intersections of closed sets are closed; finite unions of closed sets are closed.

---

## 2.4 Interior, Closure, Boundary

| Concept | Definition |
|---------|-----------|
| **Interior** $A^\circ$ | $\{x \in A : \exists\, r > 0,\, B(x,r) \subseteq A\}$ (largest open subset) |
| **Closure** $\overline{A}$ | $\{x \in X : B(x,r) \cap A \neq \emptyset\;\forall r > 0\}$ (smallest closed superset) |
| **Boundary** $\partial A$ | $\overline{A} \setminus A^\circ$ |

$$A^\circ \subseteq A \subseteq \overline{A}$$

$A$ open $\iff$ $A = A^\circ$. $\quad$ $A$ closed $\iff$ $A = \overline{A}$.

---

## 2.5 Examples in Different Spaces

**In $(\mathbb{R}, |\cdot|)$:** Same as Unit 4.

**In $(\mathbb{R}^2, d_2)$:**
- $B(\mathbf{0}, 1)$ = open unit disk (open)
- $\{(x,y) : x^2 + y^2 \leq 1\}$ = closed unit disk (closed)
- $\{(x,y) : x^2 + y^2 \leq 1, (x,y) \neq (1,0)\}$ = neither open nor closed

**In discrete metric on $X$:**
- $B(x, 1) = \{x\}$ (just the center!)
- $B(x, 2) = X$ (everything)
- Every subset is both open and closed (**clopen**)

**In $C[0,1]$ with sup metric:**
- $B(f, \varepsilon) = \{g : \|f - g\|_\infty < \varepsilon\}$ = all functions within an $\varepsilon$-tube around $f$

```
  Open ball in C[0,1]:

  f(x)+ε ─ ─ ─ ─ ─ ─ ─ ─ ─
           g must lie in
  f(x)   ─────────────── f
           this tube
  f(x)-ε ─ ─ ─ ─ ─ ─ ─ ─ ─
```

---

## 2.6 Dense Sets

**Definition 2.6.** $A$ is **dense** in $X$ if $\overline{A} = X$, i.e., every open ball intersects $A$.

**Examples:**
- $\mathbb{Q}$ is dense in $\mathbb{R}$
- Polynomials are dense in $C[a,b]$ (Weierstrass approximation)

---

## Summary Table

| Concept | Definition | $\mathbb{R}$ | Discrete |
|---------|-----------|--------------|----------|
| Open ball | $B(x,r) = \{d(x,y)<r\}$ | Open interval | $\{x\}$ (if $r\leq 1$) |
| Open set | $\forall x$, $\exists r$: $B(x,r)\subseteq U$ | Union of intervals | Every set |
| Closed set | Complement open | Contains limits | Every set |
| Dense | $\overline{A} = X$ | $\mathbb{Q}$ | Only $X$ |

---

## Quick Revision Questions

1. **Define** open ball and open set in a general metric space.

2. **Prove** open balls are open sets.

3. **Show** every subset of a discrete metric space is both open and closed.

4. **Define** interior, closure, boundary. Show $A^\circ \subseteq A \subseteq \overline{A}$.

5. **Describe** the open ball $B(f, \varepsilon)$ in $C[0,1]$ with the sup metric.

6. **Prove** that arbitrary unions and finite intersections of open sets are open.

---

[← Previous: Definition and Examples](01-definition-and-examples.md) | [Back to README](../README.md) | [Next: Convergence →](03-convergence.md)
