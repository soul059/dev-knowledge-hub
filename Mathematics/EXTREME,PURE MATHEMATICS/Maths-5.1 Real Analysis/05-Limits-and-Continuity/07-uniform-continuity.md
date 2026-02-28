# Chapter 7: Uniform Continuity

[← Previous: Extreme Value Theorem](06-extreme-value-theorem.md) | [Back to README](../README.md) | [Next: Unit 6 — Definition and Basic Properties of Derivatives →](../06-Differentiation/01-definition-and-basic-properties.md)

---

## Overview

**Uniform continuity** strengthens continuity: the $\delta$ works for *all* points simultaneously, not each point individually. On compact sets, continuity automatically upgrades to uniform continuity. This is essential for proving the Riemann integrability of continuous functions.

---

## 7.1 Definition

**Definition 7.1.** $f: A \to \mathbb{R}$ is **uniformly continuous** on $A$ if:

$$\forall\, \varepsilon > 0,\; \exists\, \delta > 0: \; |x - y| < \delta \;\text{ and }\; x, y \in A \quad\Rightarrow\quad |f(x) - f(y)| < \varepsilon$$

```
  Continuity vs. Uniform Continuity:

  Continuity at c:              Uniform continuity on A:
  ∀ε > 0                        ∀ε > 0
    ∃δ > 0  ← depends on ε,c     ∃δ > 0  ← depends on ε ONLY
      ∀x ∈ A:                       ∀x, y ∈ A:
        |x-c| < δ                     |x-y| < δ
        ⟹ |f(x)-f(c)| < ε           ⟹ |f(x)-f(y)| < ε

  KEY DIFFERENCE: "∀c" is INSIDE vs. OUTSIDE the ∃δ

  Continuous:          ∀c ∀ε ∃δ(ε,c) ...   (δ can vary with c)
  Uniformly cont.:     ∀ε ∃δ(ε) ∀x,y ...   (one δ works everywhere)
```

---

## 7.2 Negation

$f$ is **not** uniformly continuous on $A$ if:

$$\exists\, \varepsilon_0 > 0,\; \forall\, \delta > 0,\; \exists\, x, y \in A: \; |x - y| < \delta \;\text{ but }\; |f(x) - f(y)| \geq \varepsilon_0$$

**Sequential Criterion for Non-uniform Continuity.** $f$ is not uniformly continuous on $A$ $\iff$ $\exists\, \varepsilon_0 > 0$ and sequences $(x_n), (y_n) \subseteq A$ with $|x_n - y_n| \to 0$ but $|f(x_n) - f(y_n)| \geq \varepsilon_0$.

---

## 7.3 Examples

| Function | Domain | Uniformly continuous? | Reason |
|----------|--------|:--------------------:|--------|
| $f(x) = 3x + 1$ | $\mathbb{R}$ | ✅ | $\delta = \varepsilon/3$ works everywhere |
| $f(x) = x^2$ | $[0, 5]$ | ✅ | Compact domain |
| $f(x) = x^2$ | $\mathbb{R}$ | ❌ | $x_n = n$, $y_n = n + 1/n$; $\|x_n - y_n\| \to 0$ but $\|f(x_n) - f(y_n)\| \to 2$ |
| $f(x) = \sin x$ | $\mathbb{R}$ | ✅ | $\|\sin x - \sin y\| \leq \|x - y\|$ |
| $f(x) = 1/x$ | $(0, 1)$ | ❌ | $x_n = 1/n$, $y_n = 1/(n+1)$ |
| $f(x) = \sqrt{x}$ | $[0, \infty)$ | ✅ | (non-trivial proof) |

**Example: $f(x) = x^2$ not uniformly continuous on $\mathbb{R}$.**

Take $x_n = n$, $y_n = n + 1/n$. Then $|x_n - y_n| = 1/n \to 0$, but:
$$|f(x_n) - f(y_n)| = |n^2 - (n + 1/n)^2| = |2 + 1/n^2| \to 2$$

So with $\varepsilon_0 = 1$, no $\delta$ works. $\blacksquare$

```
  Why x² is not uniformly continuous on ℝ:

  f(x) = x²
  ▲
  │         ╱   The slope increases without bound.
  │       ╱     Near x = 100: f changes by ~200Δx
  │     ╱       Near x = 1:   f changes by ~2Δx  
  │   ╱╱        
  │ ╱╱          Same Δx gives MUCH bigger Δf for large x.
  │╱            No single δ can control all points.
  └──────────────► x
```

---

## 7.4 Theorem: Compact Domain ⟹ Uniform Continuity

**Theorem 7.2 (Heine–Cantor).** If $f: K \to \mathbb{R}$ is continuous and $K$ is compact, then $f$ is uniformly continuous on $K$.

**Proof.** Suppose not. Then $\exists\, \varepsilon_0 > 0$ and $(x_n), (y_n) \subseteq K$ with $|x_n - y_n| < 1/n$ but $|f(x_n) - f(y_n)| \geq \varepsilon_0$.

By compactness, $(x_n)$ has a subsequence $x_{n_k} \to c \in K$. Then $y_{n_k} \to c$ as well (since $|x_{n_k} - y_{n_k}| < 1/n_k \to 0$).

By continuity: $f(x_{n_k}) \to f(c)$ and $f(y_{n_k}) \to f(c)$. So $|f(x_{n_k}) - f(y_{n_k})| \to 0$, contradicting $|f(x_{n_k}) - f(y_{n_k})| \geq \varepsilon_0$. $\blacksquare$

```
  Heine–Cantor: why compactness saves you

  On compact K: sequences can't "escape to infinity"
  → Any problematic sequences have convergent subsequences
  → Continuity at the limit point forces a contradiction

  ┌─────────────────────────────────────┐
  │  Continuous on compact ⟹           │
  │  Uniformly continuous               │
  │                                     │
  │  Converse: UC ⟹ continuous (yes)  │
  │  But UC ⇏ compact domain           │
  │  (e.g., sin x on ℝ)                │
  └─────────────────────────────────────┘
```

---

## 7.5 Lipschitz Continuity

**Definition 7.3.** $f: A \to \mathbb{R}$ is **Lipschitz continuous** if $\exists\, L \geq 0$:

$$|f(x) - f(y)| \leq L|x - y| \quad \forall\, x, y \in A$$

**Hierarchy:**

$$\text{Lipschitz} \;\Rightarrow\; \text{Uniformly continuous} \;\Rightarrow\; \text{Continuous}$$

Neither converse holds:
- $\sqrt{x}$ on $[0,1]$: uniformly continuous but not Lipschitz (slope $\to \infty$ at 0)
- $x^2$ on $\mathbb{R}$: continuous but not uniformly continuous

---

## 7.6 Uniform Continuity and Extensions

**Theorem 7.4.** If $f: (a, b) \to \mathbb{R}$ is uniformly continuous, then $f$ can be uniquely extended to a continuous function $\tilde{f}: [a, b] \to \mathbb{R}$.

*The limits $\lim_{x \to a^+} f(x)$ and $\lim_{x \to b^-} f(x)$ exist (Cauchy criterion using uniform continuity).*

**Theorem 7.5.** If $f: A \to \mathbb{R}$ is uniformly continuous and $(x_n).$ is Cauchy in $A$, then $(f(x_n))$ is Cauchy. (Ordinary continuity does NOT guarantee this.)

---

## Summary Table

| Concept | Key Result |
|---------|------------|
| Uniform continuity | One $\delta$ works for all points |
| vs. continuity | $\delta$ depends on $\varepsilon$ only (not on the point) |
| Compact domain | Continuous $\Rightarrow$ uniformly continuous (Heine–Cantor) |
| Non-UC test | Find $(x_n), (y_n)$ with $\|x_n - y_n\| \to 0$ but $\|f(x_n) - f(y_n)\| \geq \varepsilon_0$ |
| Lipschitz | $\|f(x) - f(y)\| \leq L\|x-y\|$; strongest of the three |
| Extensions | UC on $(a,b)$ → extends to $[a,b]$ |
| Preserves Cauchy | UC maps Cauchy to Cauchy; continuity does not |

---

## Quick Revision Questions

1. **Write** the precise definition of uniform continuity and compare with pointwise continuity.

2. **Show** $f(x) = 1/x$ is not uniformly continuous on $(0, 1)$.

3. **Prove** the Heine–Cantor theorem: continuous on compact $\Rightarrow$ uniformly continuous.

4. **Is** $f(x) = x \sin(1/x)$ uniformly continuous on $(0, 1)$? Justify.

5. **Define** Lipschitz continuity and show $\text{Lip.} \Rightarrow \text{UC} \Rightarrow \text{Cont.}$

6. **Prove:** If $f$ is uniformly continuous and $(x_n)$ is Cauchy, then $(f(x_n))$ is Cauchy. Give a counterexample for merely continuous $f$.

---

[← Previous: Extreme Value Theorem](06-extreme-value-theorem.md) | [Back to README](../README.md) | [Next: Unit 6 — Definition and Basic Properties of Derivatives →](../06-Differentiation/01-definition-and-basic-properties.md)
