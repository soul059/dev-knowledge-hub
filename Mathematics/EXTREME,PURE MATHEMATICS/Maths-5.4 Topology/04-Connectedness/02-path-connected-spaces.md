# Chapter 4.2 — Path-Connected Spaces

[← Previous: Connected Spaces](01-connected-spaces.md) | [Back to README](../README.md) | [Next: Components →](03-components.md)

---

## 1. Overview

**Path-connectedness** is a stronger form of connectedness: not only is the space "one piece," but any two points can be joined by a continuous path. Path-connectedness implies connectedness, but not conversely.

---

## 2. Definition

> **Def 2.1 (Path).** A **path** in $X$ from $x$ to $y$ is a continuous function $\gamma: [0,1] \to X$ with $\gamma(0) = x$ and $\gamma(1) = y$.

> **Def 2.2 (Path-Connected).** $X$ is **path-connected** if for every $x, y \in X$, there exists a path from $x$ to $y$.

```
A path from x to y:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
            X
    ┌───────────────────┐
    │                   │
    │  x •──────╮       │
    │           │       │
    │           ╰───• y │
    │     γ([0,1])      │
    └───────────────────┘

    γ: [0,1] → X
    γ(0) = x,  γ(1) = y
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 3. Path Operations

Paths can be composed, reversed, and reparametrized:

### 3.1 Reverse Path

If $\gamma$ is a path from $x$ to $y$, then $\bar{\gamma}(t) = \gamma(1-t)$ is a path from $y$ to $x$.

### 3.2 Concatenation

If $\gamma_1$ goes from $x$ to $y$ and $\gamma_2$ from $y$ to $z$:

$$(\gamma_1 * \gamma_2)(t) = \begin{cases} \gamma_1(2t) & 0 \leq t \leq 1/2 \\ \gamma_2(2t - 1) & 1/2 \leq t \leq 1 \end{cases}$$

This is continuous by the pasting lemma (gluing at $t = 1/2$, a closed set).

### 3.3 Constant Path

$c_x(t) = x$ for all $t$. A path from $x$ to $x$.

```
Path operations:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Reverse:        x ●←─────── y     (γ̄)
  
  Concatenation:  x ●───→ y ●───→ z (γ₁ * γ₂)
  
  Timing:    γ₁ on [0,½]   γ₂ on [½,1]
             (double speed) (double speed)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## 4. Path-Connected ⟹ Connected

> **Thm 2.3.** Every path-connected space is connected.

**Proof:** Suppose $X$ is path-connected but $X = U \cup V$ is a separation. Pick $x \in U, y \in V$. Let $\gamma: [0,1] \to X$ be a path from $x$ to $y$. Then $[0,1] = \gamma^{-1}(U) \cup \gamma^{-1}(V)$ is a separation of $[0,1]$. But $[0,1]$ is connected. Contradiction. ∎

---

## 5. Connected ⟹̸ Path-Connected

> **Ex 2.4 (Topologist's Sine Curve).** Let
>
> $$S = \left\{\left(x, \sin\frac{1}{x}\right) : x > 0\right\} \cup \{(0, y) : -1 \leq y \leq 1\}$$
>
> This is the closure of the graph of $\sin(1/x)$ for $x > 0$.

```
Topologist's sine curve:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  y
  1 ┤  │╲    ╱╲    ╱╲
    │  │ ╲  ╱  ╲  ╱  ╲  ╱~~
  0 ┤──│──╳────╳────╳──╳~~~~~~~~→  x
    │  │ ╱  ╲  ╱  ╲  ╱
 -1 ┤  │╱    ╲╱    ╲╱
    │  │
    │ {0}×[-1,1]
    │ (vertical segment)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Properties:**
- **Connected:** The graph $\{(x, \sin(1/x)) : x > 0\}$ is the continuous image of $(0,\infty)$, hence connected. $S$ is its closure, hence connected (closure of connected is connected).
- **Not path-connected:** No continuous path can reach from $(0,0)$ to any point $(x_0, \sin(1/x_0))$ because the oscillation forces the path to "travel infinitely."

---

## 6. Spaces Where Connected = Path-Connected

| Condition | Connected ⟹ path-connected? |
|-----------|:---:|
| Open subsets of $\mathbb{R}^n$ | ✓ |
| Locally path-connected spaces | ✓ |
| Topological manifolds | ✓ |
| CW-complexes | ✓ |
| General spaces | ✗ |

---

## 7. Continuous Images

> **Thm 2.5.** If $f: X \to Y$ is continuous and $X$ is path-connected, then $f(X)$ is path-connected.

**Proof:** Given $f(x), f(y) \in f(X)$, let $\gamma$ be a path from $x$ to $y$ in $X$. Then $f \circ \gamma$ is a path from $f(x)$ to $f(y)$ in $f(X)$. ∎

---

## 8. Products

> **Thm 2.6.** $\prod_\alpha X_\alpha$ is path-connected (product topology) iff each $X_\alpha$ is path-connected.

**Proof ($\Leftarrow$):** Given points $(x_\alpha)$ and $(y_\alpha)$, for each $\alpha$ let $\gamma_\alpha$ be a path from $x_\alpha$ to $y_\alpha$. Then $\gamma(t) = (\gamma_\alpha(t))_\alpha$ is continuous (by the universal property of products) and is a path from $(x_\alpha)$ to $(y_\alpha)$. ∎

---

## 9. Summary Table

| Concept | Definition / Key Point |
|---------|----------------------|
| Path | Continuous $\gamma: [0,1] \to X$ |
| Path-connected | Any two points joined by a path |
| Path-connected ⟹ connected | Always |
| Connected ⟹̸ path-connected | Counterexample: topologist's sine curve |
| Continuous image | Path-connected → path-connected |
| Products | Path-connected iff all factors are |
| Open ⊆ ℝⁿ | Connected ⟺ path-connected |
| Path concatenation | $\gamma_1 * \gamma_2$: run $\gamma_1$ then $\gamma_2$ at double speed |

---

## 10. Quick Revision Questions

1. Define path-connectedness and prove that it implies connectedness.

2. Describe the topologist's sine curve and explain why it is connected but not path-connected.

3. If $f: X \to Y$ is continuous and surjective with $X$ path-connected, is $Y$ path-connected?

4. Prove that the product of two path-connected spaces is path-connected.

5. Is $\mathbb{R}^2 \setminus \{0\}$ path-connected? Is $\mathbb{R} \setminus \{0\}$ path-connected?

6. Under what conditions on $X$ does connectedness imply path-connectedness?

---

[← Previous: Connected Spaces](01-connected-spaces.md) | [Back to README](../README.md) | [Next: Components →](03-components.md)
