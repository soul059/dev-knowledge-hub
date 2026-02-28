# Unit 4 — Differentiability: Differentiability vs Continuity

[← Previous: Geometric Interpretation](02-geometric-interpretation.md) · [Next: Left and Right Derivatives →](04-left-right-derivatives.md)

---

## Chapter Overview

Continuity and differentiability are related but **not the same**. This chapter clarifies the precise relationship: **differentiability implies continuity**, but continuity does **not** imply differentiability. Understanding this distinction is crucial for rigorous calculus.

---

## 4.3.1 The Main Theorem

### Differentiability ⟹ Continuity

**Theorem:** If $f$ is differentiable at $x = a$, then $f$ is continuous at $x = a$.

**Proof:**

We need to show $\lim_{x \to a} f(x) = f(a)$, i.e., $\lim_{x \to a} [f(x) - f(a)] = 0$.

$$\lim_{x \to a} [f(x) - f(a)] = \lim_{x \to a} \frac{f(x) - f(a)}{x - a} \cdot (x - a)$$

$$= f'(a) \cdot 0 = 0 \quad \blacksquare$$

(Here we used the fact that $f'(a)$ exists and is finite, and $\lim_{x \to a}(x - a) = 0$.)

### The Converse is FALSE

**Continuity does NOT imply differentiability.**

A function can be continuous at a point but not differentiable there.

---

## 4.3.2 The Logical Relationship

```
  ┌─────────────────────────────────────────┐
  │           Continuous Functions           │
  │                                         │
  │    ┌────────────────────────────────┐   │
  │    │   Differentiable Functions     │   │
  │    │                                │   │
  │    │   (smaller set — every         │   │
  │    │    differentiable function     │   │
  │    │    is continuous, but not      │   │
  │    │    vice versa)                 │   │
  │    └────────────────────────────────┘   │
  │                                         │
  │   ← Functions continuous but NOT        │
  │     differentiable live in this gap     │
  └─────────────────────────────────────────┘
```

**Hierarchy:**

$$\text{Differentiable} \subset \text{Continuous} \subset \text{Defined (with limits)}$$

---

## 4.3.3 Counterexamples: Continuous but NOT Differentiable

### Example 1: $f(x) = |x|$ at $x = 0$ (Corner)

**Continuous at 0:** $\lim_{x \to 0} |x| = 0 = f(0)$ ✓

**Not differentiable at 0:**

$$\lim_{h \to 0^+} \frac{|0+h| - |0|}{h} = \lim_{h \to 0^+} \frac{h}{h} = 1$$

$$\lim_{h \to 0^-} \frac{|0+h| - |0|}{h} = \lim_{h \to 0^-} \frac{-h}{h} = -1$$

Since $1 \ne -1$, the derivative does not exist.

```
  y
  │╲       ╱
  │  ╲   ╱
  │    ╲╱  ← sharp corner (continuous, not differentiable)
  └─────┼──── x
        0
```

### Example 2: $f(x) = x^{1/3}$ at $x = 0$ (Vertical Tangent)

**Continuous at 0:** $\lim_{x \to 0} x^{1/3} = 0 = f(0)$ ✓

**Not differentiable at 0:**

$$f'(0) = \lim_{h \to 0} \frac{h^{1/3}}{h} = \lim_{h \to 0} h^{-2/3} = \infty$$

The tangent line is vertical — slope is infinite.

### Example 3: $f(x) = x^{2/3}$ at $x = 0$ (Cusp)

**Continuous at 0:** ✓ (follows from continuity of $x^{2/3}$)

**Not differentiable at 0:**

$$\lim_{h \to 0^+} \frac{h^{2/3}}{h} = \lim_{h \to 0^+} h^{-1/3} = +\infty$$
$$\lim_{h \to 0^-} \frac{|h|^{2/3}}{h} = \lim_{h \to 0^-} \frac{(-h)^{2/3}}{h} = -\infty$$

### Example 4: Weierstrass Function (Continuous Everywhere, Differentiable Nowhere!)

Karl Weierstrass constructed a function:

$$W(x) = \sum_{n=0}^{\infty} a^n \cos(b^n \pi x)$$

where $0 < a < 1$, $b$ is a positive odd integer, and $ab > 1 + \frac{3\pi}{2}$.

This function is:
- **Continuous** everywhere (uniform limit of continuous functions)
- **Differentiable** nowhere (the function has "corners" at every point)

This shows continuity and differentiability are fundamentally different properties.

---

## 4.3.4 Summary of Relationships

| Statement | True? | Example |
|-----------|-------|---------|
| Differentiable ⟹ Continuous | **YES** (theorem) | — |
| Continuous ⟹ Differentiable | **NO** | $\|x\|$ at 0 |
| Not continuous ⟹ Not differentiable | **YES** (contrapositive of above) | — |
| Not differentiable ⟹ Not continuous | **NO** | $\|x\|$ at 0 |
| Differentiable everywhere ⟹ Continuous everywhere | **YES** | — |
| Continuous everywhere ⟹ Differentiable somewhere | **NO** | Weierstrass function |

---

## 4.3.5 Decision Table: Checking Differentiability

```
  Step 1: Is f(a) defined?
          │
     ┌────┴────┐
    NO         YES
    ↓           │
  Not cont.   Step 2: Is lim[x→a] f(x) = f(a)?
  ∴ Not diff.       │
               ┌────┴────┐
              NO         YES
              ↓           │
           Not cont.    Step 3: Does f'(a) exist?
           ∴ Not diff.  (Check limit of diff. quotient)
                              │
                         ┌────┴────┐
                        NO         YES
                        ↓           ↓
                     Continuous   DIFFERENTIABLE
                     but NOT      (and therefore
                     differentiable continuous)
```

---

## 4.3.6 Practical Implications

| Property | Continuous Functions | Differentiable Functions |
|----------|---------------------|-------------------------|
| Can have corners | Yes | No |
| Can have cusps | Yes | No |
| Can have vertical tangents | Yes | No |
| Graph can be drawn without lifting pen | Yes | Yes |
| Graph is "smooth" | Not necessarily | Yes |
| Tangent line exists | Not necessarily | Yes |
| Linear approximation valid | Not necessarily | Yes |

---

## Summary Table

| Concept | Key Idea |
|---------|----------|
| Differentiable ⟹ Continuous | Always true; proven via limit arithmetic |
| Continuous ⟹ Differentiable | **FALSE** — counterexample: $\|x\|$ at 0 |
| Corner | Left and right derivatives differ (finite but unequal) |
| Cusp | One-sided derivatives are $\pm\infty$ |
| Vertical tangent | Derivative is $\pm\infty$ (both sides agree in magnitude) |
| Weierstrass function | Continuous everywhere, differentiable nowhere |
| Contrapositive | Not continuous ⟹ Not differentiable |

---

## Quick Revision Questions

**Q1.** Prove: if $f$ is differentiable at $a$, then $f$ is continuous at $a$.

**Q2.** Give three examples of functions continuous but not differentiable at a point.

**Q3.** True or False: "If $f$ is not continuous at $a$, it might still be differentiable at $a$."

**Q4.** Why does $|x|$ fail to be differentiable at $x = 0$? Compute both one-sided limits.

**Q5.** What is the Weierstrass function, and why is it significant?

**Q6.** If $f'(5) = 3$, can you conclude that $f$ is continuous at $x = 5$? Why?

---

[← Previous: Geometric Interpretation](02-geometric-interpretation.md) · [Next: Left and Right Derivatives →](04-left-right-derivatives.md)
