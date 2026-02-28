# Chapter 4: Continuity in Metric Spaces

[← Previous: Convergence](03-convergence.md) | [Back to README](../README.md) | [Next: Completeness →](05-completeness.md)

---

## Overview

Continuity in metric spaces is defined using the metric $d$, generalizing the ε-δ definition. The topological characterization — preimages of open sets are open — becomes especially powerful and leads to general topology.

---

## 4.1 Definition

**Definition 4.1.** Let $(X, d_X)$ and $(Y, d_Y)$ be metric spaces. $f: X \to Y$ is **continuous at** $p \in X$ if:

$$\forall\, \varepsilon > 0,\; \exists\, \delta > 0: \; d_X(x, p) < \delta \Rightarrow d_Y(f(x), f(p)) < \varepsilon$$

$f$ is **continuous on $X$** if continuous at every $p \in X$.

---

## 4.2 Sequential Characterization

**Theorem 4.2.** $f: X \to Y$ is continuous at $p$ $\iff$ for every sequence $x_n \to p$ in $X$, $f(x_n) \to f(p)$ in $Y$.

**Proof.** Same as for $\mathbb{R}$, replacing $|\cdot|$ with the appropriate metrics. $\blacksquare$

---

## 4.3 Topological Characterization

**Theorem 4.3.** $f: X \to Y$ is continuous $\iff$ $f^{-1}(V)$ is open in $X$ for every open $V \subseteq Y$.

Equivalently: $f^{-1}(F)$ is closed for every closed $F \subseteq Y$.

**Proof ($\Rightarrow$).** Let $V$ open, $p \in f^{-1}(V)$. Then $f(p) \in V$, so $\exists\, \varepsilon$: $B(f(p), \varepsilon) \subseteq V$. By continuity, $\exists\, \delta$: $f(B(p, \delta)) \subseteq B(f(p), \varepsilon) \subseteq V$, so $B(p, \delta) \subseteq f^{-1}(V)$. Hence $f^{-1}(V)$ is open.

**Proof ($\Leftarrow$).** Fix $p$, $\varepsilon > 0$. $V = B(f(p), \varepsilon)$ is open, so $f^{-1}(V)$ is open and contains $p$. Thus $\exists\, \delta$: $B(p, \delta) \subseteq f^{-1}(V)$, giving $d_X(x,p) < \delta \Rightarrow f(x) \in V$, i.e., $d_Y(f(x), f(p)) < \varepsilon$. $\blacksquare$

```
  Three equivalent definitions:

  ┌──────────────────────────────────────────────────┐
  │                                                  │
  │  (1)  ε-δ:  d(x,p) < δ ⟹ d(f(x),f(p)) < ε    │
  │                    ⟺                             │
  │  (2)  Sequential:  xₙ → p ⟹ f(xₙ) → f(p)      │
  │                    ⟺                             │
  │  (3)  Topological: f⁻¹(open) = open             │
  │                                                  │
  └──────────────────────────────────────────────────┘
```

---

## 4.4 Uniform Continuity

**Definition 4.4.** $f: X \to Y$ is **uniformly continuous** if:

$$\forall\, \varepsilon > 0,\; \exists\, \delta > 0: \; d_X(x, y) < \delta \Rightarrow d_Y(f(x), f(y)) < \varepsilon$$

(Same $\delta$ for all pairs $x, y$.)

---

## 4.5 Lipschitz and Contraction Mappings

**Definition 4.5.** $f: X \to Y$ is **Lipschitz** with constant $L$ if:

$$d_Y(f(x), f(y)) \leq L\, d_X(x, y) \quad \forall\, x, y$$

- Lipschitz $\Rightarrow$ uniformly continuous $\Rightarrow$ continuous.
- $L < 1$: **contraction mapping**.

---

## 4.6 Properties of Continuous Functions

**Theorem 4.6.** If $f: X \to Y$ and $g: Y \to Z$ are continuous, then $g \circ f: X \to Z$ is continuous.

**Theorem 4.6'.** Preservations under continuity:
- Continuous image of a **compact** set is compact.
- Continuous image of a **connected** set is connected.
- Continuous image of a **bounded** set need NOT be bounded (in general).

---

## 4.7 Isometries and Homeomorphisms

| Map type | Preserves | Definition |
|----------|-----------|-----------|
| **Isometry** | Distances | $d_Y(f(x), f(y)) = d_X(x, y)$ |
| **Homeomorphism** | Topology | $f$ bijective, $f$ and $f^{-1}$ continuous |

Isometry $\Rightarrow$ homeomorphism $\Rightarrow$ same topological properties.

---

## 4.8 Banach Fixed Point Theorem

**Theorem 4.8 (Banach Contraction Principle).** If $(X, d)$ is a **complete** metric space and $f: X \to X$ is a **contraction** ($d(f(x), f(y)) \leq c\, d(x,y)$ for some $0 \leq c < 1$), then $f$ has a **unique fixed point** $x^* = f(x^*)$.

Moreover, for any $x_0 \in X$, the iteration $x_{n+1} = f(x_n)$ converges to $x^*$, with:

$$d(x_n, x^*) \leq \frac{c^n}{1 - c}\, d(x_0, x_1)$$

**Proof sketch.** $(x_n)$ is Cauchy: $d(x_{n+1}, x_n) \leq c^n d(x_1, x_0)$, and $\sum c^n < \infty$. By completeness, $x_n \to x^*$. By continuity: $f(x^*) = \lim f(x_n) = \lim x_{n+1} = x^*$. Uniqueness: if $f(y) = y$, then $d(x^*, y) = d(f(x^*), f(y)) \leq c\, d(x^*, y)$, so $d(x^*, y) = 0$. $\blacksquare$

```
  Banach Fixed Point Iteration:

  x₀ → x₁ = f(x₀) → x₂ = f(x₁) → ··· → x*

  Each step contracts distance by factor c:

  |──────────────| d(x₀, x*)
     |────────| d(x₁, x*)       ← factor c
        |────| d(x₂, x*)       ← factor c²
          |─| d(x₃, x*)        ← factor c³
           * x*
```

---

## Summary Table

| Concept | Definition |
|---------|-----------|
| Continuous at $p$ | $d(x,p)<\delta \Rightarrow d(f(x),f(p))<\varepsilon$ |
| Sequential | $x_n \to p \Rightarrow f(x_n) \to f(p)$ |
| Topological | $f^{-1}(\text{open}) = \text{open}$ |
| Uniformly continuous | $\delta$ independent of point |
| Lipschitz | $d(f(x),f(y)) \leq Ld(x,y)$ |
| Contraction | Lipschitz with $L < 1$ |
| Banach FPT | Complete + contraction ⟹ unique fixed point |

---

## Quick Revision Questions

1. **Define** continuity in metric spaces. State three equivalent characterizations.

2. **Prove** the topological characterization: continuous $\iff$ preimage of open is open.

3. **Define** Lipschitz continuity. Show Lipschitz $\Rightarrow$ uniformly continuous.

4. **State and prove** the Banach Fixed Point Theorem.

5. **Apply** the contraction principle to show $f(x) = \cos x$ has a unique fixed point in $[0,1]$.

6. **Compare** isometries and homeomorphisms. Give an example of a homeomorphism that is not an isometry.

---

[← Previous: Convergence](03-convergence.md) | [Back to README](../README.md) | [Next: Completeness →](05-completeness.md)
