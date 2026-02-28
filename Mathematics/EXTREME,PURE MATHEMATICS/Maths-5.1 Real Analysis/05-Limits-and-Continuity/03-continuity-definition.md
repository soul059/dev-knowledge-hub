# Chapter 3: Continuity — Definition

[← Previous: Properties of Limits](02-properties-of-limits.md) | [Back to README](../README.md) | [Next: Properties of Continuous Functions →](04-properties-of-continuous-functions.md)

---

## Overview

A function is **continuous** at $c$ if the limit equals the function value: $\lim_{x \to c} f(x) = f(c)$. This chapter develops the $\varepsilon$-$\delta$, sequential, and topological definitions and their equivalences.

---

## 3.1 The ε-δ Definition

**Definition 3.1.** $f: A \to \mathbb{R}$ is **continuous at $c \in A$** if:

$$\forall\, \varepsilon > 0,\; \exists\, \delta > 0: \; |x - c| < \delta \;\text{ and }\; x \in A \quad\Rightarrow\quad |f(x) - f(c)| < \varepsilon$$

$f$ is **continuous on $A$** if it is continuous at every point of $A$.

```
  Continuity vs. limit:

  Limit: 0 < |x - c| < δ ⟹ |f(x) - L| < ε
                ↑                      ↑
          excludes c            L may ≠ f(c)

  Continuity: |x - c| < δ ⟹ |f(x) - f(c)| < ε
                ↑                         ↑
          includes c              must use f(c)

  Continuity = Limit exists AND equals f(c)
```

**At isolated points:** If $c$ is an isolated point of $A$, then $f$ is automatically continuous at $c$ (there exists $\delta$ with $A \cap V_\delta(c) = \{c\}$).

---

## 3.2 Sequential Characterization

**Theorem 3.2.** $f$ is continuous at $c$ $\iff$ for every sequence $(x_n) \subseteq A$ with $x_n \to c$, we have $f(x_n) \to f(c)$.

In short: **$f$ is continuous iff it preserves convergent sequences.**

```
  Sequential continuity:

  xₙ → c   ⟹   f(xₙ) → f(c)
  ───►          ───►
  Input:         Output:
  x₁ x₂ x₃ → c    f(x₁) f(x₂) f(x₃) → f(c)
```

---

## 3.3 Topological Characterization

**Theorem 3.3.** $f: \mathbb{R} \to \mathbb{R}$ is continuous $\iff$ $f^{-1}(O)$ is open for every open set $O \subseteq \mathbb{R}$.

Equivalently: preimages of open sets are open.

**Proof ($\Rightarrow$).** Let $O$ be open, $c \in f^{-1}(O)$. Then $f(c) \in O$, so $\exists\, \varepsilon > 0$ with $(f(c)-\varepsilon, f(c)+\varepsilon) \subseteq O$. By continuity, $\exists\, \delta > 0$ with $|f(x) - f(c)| < \varepsilon$ when $|x-c| < \delta$. So $(c-\delta, c+\delta) \subseteq f^{-1}(O)$, proving $f^{-1}(O)$ is open.

**Proof ($\Leftarrow$).** For $\varepsilon > 0$, $O = (f(c)-\varepsilon, f(c)+\varepsilon)$ is open, so $f^{-1}(O)$ is open, so $\exists\, \delta > 0$ with $(c-\delta, c+\delta) \subseteq f^{-1}(O)$, i.e., $|f(x) - f(c)| < \varepsilon$ when $|x-c| < \delta$. $\blacksquare$

---

## 3.4 Three Equivalent Definitions

| Definition | Statement |
|-----------|-----------|
| **ε-δ** | $\forall\, \varepsilon$, $\exists\, \delta$: $\|x - c\| < \delta \Rightarrow \|f(x) - f(c)\| < \varepsilon$ |
| **Sequential** | $x_n \to c \Rightarrow f(x_n) \to f(c)$ |
| **Topological** | $f^{-1}(O)$ open for every open $O$ |

All three are equivalent and used in different contexts:
- **ε-δ**: concrete proofs, finding explicit $\delta$
- **Sequential**: disproving continuity, algebraic arguments
- **Topological**: abstract arguments, generalizations

---

## 3.5 Examples and Non-Examples

**Example 1: Polynomials are continuous.** Build from $\lim_{x \to c} x = c$ (identity) and $\lim_{x \to c} k = k$ (constant), using ALT.

**Example 2: $f(x) = |x|$ is continuous.** $||x| - |c|| \leq |x - c|$. Take $\delta = \varepsilon$.

**Example 3: Dirichlet function is nowhere continuous.**
$$D(x) = \begin{cases} 1 & x \in \mathbb{Q} \\ 0 & x \notin \mathbb{Q} \end{cases}$$

For any $c$: take $x_n \in \mathbb{Q}$ with $x_n \to c$ and $y_n \notin \mathbb{Q}$ with $y_n \to c$. Then $f(x_n) = 1$ and $f(y_n) = 0$. Since limits disagree, $f$ is discontinuous at $c$.

**Example 4: Thomae's function is continuous at irrationals, discontinuous at rationals.**
$$T(x) = \begin{cases} 1/q & x = p/q \text{ (lowest terms)} \\ 0 & x \notin \mathbb{Q} \end{cases}$$

```
  Thomae's function (ruler function):
  
  y ▲
  1 │ ●                     ● 
    │
 1/2│    ●         ●
    │
 1/3│      ●   ●       ●
 1/4│       ● ●   ● ●     ●
    │ · · ·············· · · · (irrationals: y = 0)
    └──────────────────────────► x
    0                          1

  Continuous at every irrational, discontinuous at every rational!
```

---

## 3.6 Types of Discontinuity

| Type | Condition |
|------|-----------|
| **Removable** | $\lim_{x \to c} f(x)$ exists but $\neq f(c)$ (or $f(c)$ undefined) |
| **Jump** | Left and right limits exist but $\lim_{x \to c^-} f(x) \neq \lim_{x \to c^+} f(x)$ |
| **Essential** | At least one one-sided limit doesn't exist |

```
  Types of discontinuity:

  Removable:           Jump:              Essential:
  
  y ▲    ○  ← hole     y ▲   ●            y ▲  ╱╲╱╲╱╲
    │   ╱╲               │   │              │ ╱╲╱╲╱╲╱
    │  ╱  ╲              │   │ ← jump       │╱╲╱╲╱╲╱╲
    │ ╱ ●  ╲             │───○              │
    └──┼──────           └──┼──────         └──┼──────
       c                    c                  c

  Can be "fixed"       Cannot be fixed    sin(1/x) type
```

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Continuous at $c$ | $\lim_{x \to c} f(x) = f(c)$ |
| ε-δ definition | $\forall \varepsilon$, $\exists \delta$: $\|x-c\| < \delta \Rightarrow \|f(x)-f(c)\| < \varepsilon$ |
| Sequential | Preserves convergent sequences |
| Topological | Preimages of open sets are open |
| Dirichlet function | Nowhere continuous |
| Thomae's function | Continuous at irrationals only |
| Removable discontinuity | Limit exists but $\neq f(c)$ |
| Jump discontinuity | One-sided limits exist but differ |

---

## Quick Revision Questions

1. **State** the three equivalent definitions of continuity.

2. **Prove** $f(x) = x^2$ is continuous at $c = 3$ using $\varepsilon$-$\delta$.

3. **Show** the Dirichlet function is nowhere continuous.

4. **Show** Thomae's function is continuous at every irrational.

5. **Classify** the discontinuity of $f(x) = \frac{x^2 - 1}{x - 1}$ at $x = 1$.

6. **Prove** the topological characterization: $f$ continuous $\iff$ preimages of open sets are open.

---

[← Previous: Properties of Limits](02-properties-of-limits.md) | [Back to README](../README.md) | [Next: Properties of Continuous Functions →](04-properties-of-continuous-functions.md)
