# Chapter 3.4: Singular Solutions

[← Previous: Lagrange's Equation](03-lagranges-equation.md) | [Back to README](../README.md) | [Next: Growth and Decay →](../04-Applications-First-Order/01-growth-and-decay.md)

---

## Overview

A **singular solution** is a solution of a DE that cannot be obtained from the general solution for any value of the arbitrary constant. Singular solutions arise as **envelopes** of families of curves and are unique to higher-degree first-order equations.

---

## 1. Definition

> A solution $y = \psi(x)$ is a **singular solution** of $F(x, y, p) = 0$ if:
> 1. It satisfies $F(x, y, p) = 0$
> 2. It cannot be obtained from the general solution $\Phi(x, y, C) = 0$ for any $C$
> 3. At every point, it is tangent to some member of the general solution family

---

## 2. Methods for Finding Singular Solutions

### Method 1: p-Discriminant

From $F(x, y, p) = 0$, eliminate $p$ between:

$$F(x, y, p) = 0 \quad \text{and} \quad \frac{\partial F}{\partial p} = 0$$

The resulting curve $\Delta_p(x, y) = 0$ is the **p-discriminant locus**.

### Method 2: C-Discriminant

From the general solution $\Phi(x, y, C) = 0$, eliminate $C$ between:

$$\Phi(x, y, C) = 0 \quad \text{and} \quad \frac{\partial \Phi}{\partial C} = 0$$

The resulting curve $\Delta_C(x, y) = 0$ is the **C-discriminant locus**.

### Relationship

```
F(x, y, p) = 0                    Φ(x, y, C) = 0
(the DE)                           (general solution)
     │                                   │
     ▼                                   ▼
∂F/∂p = 0                          ∂Φ/∂C = 0
     │                                   │
     ▼                                   ▼
Eliminate p                         Eliminate C
     │                                   │
     ▼                                   ▼
p-discriminant                    C-discriminant
     │                                   │
     └────────────┬──────────────────────┘
                  │
                  ▼
         May contain:
         • Singular solution (envelope)
         • Cusp locus
         • Tac locus (nodal locus)
```

> **Caution**: Not every component of the discriminant locus is a singular solution. Verify by substitution!

---

## 3. Worked Examples

### Example 1: Using C-Discriminant

**DE**: $p^2 - 4y = 0$, i.e., $(y')^2 = 4y$

**Step 1** — Solve: $\dfrac{dy}{\sqrt{4y}} = dx \implies \sqrt{y} = x + C$

**General solution**: $\Phi = y - (x + C)^2 = 0$, i.e., $y = (x + C)^2$

**Step 2** — C-discriminant: $\dfrac{\partial \Phi}{\partial C} = -2(x + C) = 0 \implies C = -x$

**Step 3** — Substitute: $y = (x - x)^2 = 0$

**Singular solution**: $\boxed{y = 0}$

**Verify**: $y = 0 \implies y' = 0 \implies (y')^2 = 0 = 4 \cdot 0$ ✓

```
   y
   │  ╱      ╲      ╱      ╲
   │ ╱   C=1  ╲    ╱  C=-1  ╲
   │╱          ╲  ╱          ╲     Family: y = (x+C)²
   ├────────────╲╱────────────── x  (parabolas)
   │ ═══════════════════════════   y = 0 (singular solution)
   │  Envelope: the x-axis          touching all parabolas
   │  touches every parabola         at their vertices
```

### Example 2: Using p-Discriminant

**DE**: $4xp^2 - (3x - 1)^2 = 0$

$F = 4xp^2 - (3x-1)^2 = 0$

$\dfrac{\partial F}{\partial p} = 8xp = 0$

From $\partial F/\partial p = 0$: either $x = 0$ or $p = 0$.

**Case $p = 0$**: Substitute in $F$: $0 - (3x-1)^2 = 0 \implies x = 1/3$

Check $x = 1/3$: this is a vertical line, which has $p = \infty$, contradicting $p = 0$. **Not a solution.**

**Case $x = 0$**: Substitute: $0 - (0-1)^2 = -1 \neq 0$. **Not a solution.**

So this DE actually has **no singular solution** — the discriminant locus gave extraneous results.

### Example 3: Clairaut Equation Revisited

**DE**: $y = xp + p^2$

**General solution**: $y = Cx + C^2$

**C-discriminant**: $\partial\Phi/\partial C$: let $\Phi = y - Cx - C^2 = 0$

$\dfrac{\partial\Phi}{\partial C} = -x - 2C = 0 \implies C = -x/2$

Substitute: $y = (-x/2)x + (-x/2)^2 = -x^2/2 + x^2/4 = -x^2/4$

**Singular solution**: $y = -x^2/4$

**p-discriminant**: $F = y - xp - p^2 = 0$

$\partial F/\partial p = -x - 2p = 0 \implies p = -x/2$

Substitute: $y = x(-x/2) + (-x/2)^2 \doesn't... wait$:

$y - x(-x/2) - (-x/2)^2 = y + x^2/2 - x^2/4 = y + x^2/4 = 0$

$y = -x^2/4$ ✓ Same result!

---

## 4. Extraneous Loci

The discriminant may include curves that are **NOT** singular solutions:

### Cusp Locus

A curve where solution curves have cusps (sharp turns). These satisfy the discriminant equation but are not envelopes.

### Tac Locus (Nodal Locus)

A curve where two different members of the family are tangent to each other (not to the locus itself).

### How to Distinguish

| Locus | Appears in p-disc? | Appears in C-disc? | Is solution? |
|-------|-------------------|-------------------|-------------|
| **Envelope** (singular sol.) | Yes (once) | Yes (once) | **Yes** |
| **Cusp locus** | Yes (cubed) | No | No |
| **Tac locus** | No | Yes (squared) | No |

> **Rule of thumb**: Always **verify** candidates by substituting back into the original DE.

---

## 5. Existence of Singular Solutions

### When They Exist

```
F(x, y, p) = 0
      │
      ▼
Is the DE of degree > 1 in p?
      │
   ┌──┴──┐
  YES    NO
   │      │
   │      └── Degree 1: No singular solution
   │           (for first-order DEs)
   ▼
 May have singular solution.
 Find discriminant, verify.
```

### Key Theorem

> A first-order ODE of **degree 1** (i.e., linear in $p$) has **no singular solution**.

Singular solutions are a phenomenon of **higher-degree** equations only.

---

## 6. Connection to the Theory of Envelopes

For a family $F(x, y, C) = 0$, the envelope satisfies:

$$F = 0 \quad \text{and} \quad F_C = 0$$

The discriminant $\Delta$ can be expressed as:

$$\Delta = \text{Resultant}(F, F_C, C)$$

When $\Delta(x, y) = 0$, we get the envelope (and possibly extraneous loci).

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Singular solution** | Satisfies DE but not obtainable from general solution |
| **Geometric meaning** | Envelope of the family of solution curves |
| **p-discriminant** | Eliminate $p$ from $F = 0$ and $\partial F/\partial p = 0$ |
| **C-discriminant** | Eliminate $C$ from $\Phi = 0$ and $\partial\Phi/\partial C = 0$ |
| **Extraneous loci** | Cusp locus, tac locus — not genuine solutions |
| **Verification** | Always substitute candidate back into DE |
| **Existence** | Only for DEs of degree $> 1$ in $p$ |

---

## Quick Revision Questions

1. Find the singular solution of $(y')^2 = 4(y - 1)$ using the C-discriminant method.

2. The general solution of a DE is $y = Cx - C^3$. Find the singular solution.

3. Explain why $y' + 2y = x$ cannot have a singular solution.

4. What is the geometric interpretation of a singular solution as an envelope?

5. Find both discriminant loci for $y = xp + e^p$ and determine the singular solution.

6. A discriminant locus contains $y^2 = 0$, i.e., $y = 0$ appearing squared. Is this a singular solution, cusp locus, or tac locus?

---

[← Previous: Lagrange's Equation](03-lagranges-equation.md) | [Back to README](../README.md) | [Next: Growth and Decay →](../04-Applications-First-Order/01-growth-and-decay.md)
