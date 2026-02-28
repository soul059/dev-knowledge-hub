# 4.3 Cauchy–Goursat Theorem

[← Previous: Line Integrals](02-line-integrals.md) | [Next: Cauchy Integral Formula →](04-cauchy-integral-formula.md)

---

## Chapter Overview

The **Cauchy–Goursat Theorem** states that a contour integral of an analytic function over a closed curve is zero — provided the curve lies in a simply connected domain. This single result is the foundation for virtually everything that follows: the integral formula, series representations, and the residue theorem.

---

## 1. Statement of the Theorem

### 1.1 Cauchy's Theorem (Original — Cauchy, 1825)

If $f$ is analytic on a simply connected domain $D$ and $f'$ is continuous on $D$, then for every closed contour $\gamma$ in $D$:

$$\oint_\gamma f(z)\,dz = 0$$

### 1.2 Goursat's Enhancement (1900)

The hypothesis that $f'$ is continuous is **unnecessary**. Goursat proved:

> If $f$ is analytic (differentiable) on a simply connected domain $D$, then for every closed contour $\gamma$ in $D$:
> 
> $$\boxed{\oint_\gamma f(z)\,dz = 0}$$

The significance: analyticity already forces $f'$ to be continuous (and even infinitely differentiable), but proving this requires the Cauchy integral formula, which itself depends on this theorem. Goursat broke the circular reasoning.

---

## 2. Proof Outline (Goursat's Approach)

### For triangles (the key case):

```
Step 1: Subdivide triangle into 4 congruent sub-triangles:

        Δ                            Δ₁
       ╱ ╲                          ╱ ╲
      ╱   ╲            ──►        •───•
     ╱     ╲                     ╱ ╲ ╱ ╲
    ╱       ╲                   • Δ₃ • Δ₄•
   ╱_________╲                 ╱___╲╱___╲
                                  Δ₂
```

**Step 1.** For the triangle $\Delta$, subdivide into 4 sub-triangles $\Delta_1, \Delta_2, \Delta_3, \Delta_4$. Interior contributions cancel:

$$\oint_\Delta f\,dz = \sum_{k=1}^4 \oint_{\Delta_k} f\,dz$$

**Step 2.** Pick the sub-triangle $\Delta^{(1)}$ with the largest integral:

$$\left|\oint_\Delta f\,dz\right| \leq 4\left|\oint_{\Delta^{(1)}} f\,dz\right|$$

**Step 3.** Repeat: obtain nested triangles $\Delta \supset \Delta^{(1)} \supset \Delta^{(2)} \supset \cdots$ with:

$$\left|\oint_\Delta f\,dz\right| \leq 4^n\left|\oint_{\Delta^{(n)}} f\,dz\right|$$

**Step 4.** By the nested compact sets theorem, $\bigcap_{n} \Delta^{(n)} = \{z_0\}$. Use differentiability at $z_0$:

$$f(z) = f(z_0) + f'(z_0)(z - z_0) + \epsilon(z)(z - z_0)$$

where $\epsilon(z) \to 0$ as $z \to z_0$.

**Step 5.** The linear part integrates to zero. The error term gives:

$$\left|\oint_\Delta f\,dz\right| \leq 4^n \cdot \epsilon_n \cdot \frac{L^2}{4^n} = \epsilon_n L^2 \to 0$$

Therefore $\oint_\Delta f\,dz = 0$. $\square$

---

## 3. Extensions

### 3.1 Deformation of Contours

If $\gamma_1$ and $\gamma_2$ are homotopic in a domain where $f$ is analytic, then:

$$\int_{\gamma_1} f\,dz = \int_{\gamma_2} f\,dz$$

### 3.2 Multiply Connected Domains

For an annular region with outer contour $C_1$ and inner contour $C_2$ (both positively oriented):

$$\oint_{C_1} f\,dz = \oint_{C_2} f\,dz$$

```
Deformation principle in multiply connected domain:

   ╭──── C₁ (outer) ────╮
  ╱                       ╲
 │    ╭── C₂ (inner) ──╮   │
 │   │     singularity   │  │
 │   │         ×         │  │
 │    ╰─────────────────╯   │
  ╲     f analytic between  ╱
   ╰────────────────────────╯

  ∮_{C₁} f dz = ∮_{C₂} f dz
```

### 3.3 General Multiply Connected Version

If $f$ is analytic between a positively oriented outer contour $C_0$ and negatively oriented inner contours $C_1, \ldots, C_n$:

$$\oint_{C_0} f\,dz + \oint_{C_1} f\,dz + \cdots + \oint_{C_n} f\,dz = 0$$

(where $C_k$, $k \geq 1$, are oriented clockwise)

Or equivalently, with all contours counterclockwise:

$$\oint_{C_0} f\,dz = \sum_{k=1}^{n} \oint_{C_k} f\,dz$$

---

## 4. Consequences

| Consequence | Statement |
|------------|-----------|
| **Path independence** | $\int_\gamma f\,dz$ depends only on endpoints (in simply connected domain) |
| **Antiderivative existence** | Every analytic $f$ on simply connected $D$ has antiderivative $F$ |
| **Indefinite integral** | $F(z) = \int_{z_0}^z f(\zeta)\,d\zeta$ is well-defined and $F'(z) = f(z)$ |

---

## 5. Design Example

### Example: Evaluate $\oint_\gamma \frac{e^z}{z^2 + 4}\,dz$ where $\gamma$ is $|z| = 1$.

**Step 1.** Find singularities: $z^2 + 4 = 0 \Rightarrow z = \pm 2i$.

**Step 2.** Check: both $2i$ and $-2i$ have $|z| = 2 > 1$, so both are **outside** $\gamma$.

**Step 3.** $f(z) = e^z/(z^2+4)$ is analytic on the simply connected domain $|z| < 1.5$, which contains $\gamma$.

**Step 4.** By the Cauchy–Goursat theorem:

$$\oint_{|z|=1} \frac{e^z}{z^2+4}\,dz = 0$$

```
     ╭── |z| = 1 ──╮
    ╱                ╲
   │  f is analytic   │
   │  inside circle   │
    ╲                ╱        •  2i  (outside)
     ╰──────────────╯

                              • -2i  (outside)
```

### Example 2: Evaluate $\oint_{|z|=3} \frac{1}{z}\,dz$.

$1/z$ is analytic on $\{0 < |z| < \infty\}$ — but this domain is **not** simply connected. The origin is a singularity inside the contour. We **cannot** apply Cauchy–Goursat directly.

By deformation to $|z| = 1$ (or direct computation): the answer is $2\pi i$.

---

## 6. Cauchy's Theorem vs. Green's Theorem

Cauchy's original proof used Green's theorem (requires $f'$ continuous):

$$\oint_\gamma f\,dz = \oint_\gamma (u+iv)(dx+idy)$$
$$= \oint_\gamma (u\,dx - v\,dy) + i\oint_\gamma (v\,dx + u\,dy)$$

By Green's theorem:
$$= \iint_D \left(-\frac{\partial v}{\partial x} - \frac{\partial u}{\partial y}\right)dA + i\iint_D \left(\frac{\partial u}{\partial x} - \frac{\partial v}{\partial y}\right)dA$$

Both integrands vanish by the Cauchy–Riemann equations! Hence $= 0$.

---

## 7. Real-World Applications

| Application | How Cauchy–Goursat is Used |
|-------------|---------------------------|
| **Circuit analysis** | Deforming integration contours in Laplace-domain analysis |
| **Antenna design** | Path-independence of certain electromagnetic integrals |
| **Fluid dynamics** | Conservative velocity fields → zero circulation |
| **Quantum mechanics** | Deformation of Feynman integral contours |

---

## Summary Table

| Concept | Statement |
|---------|-----------|
| Cauchy–Goursat | $\oint_\gamma f\,dz = 0$ if $f$ analytic on simply connected $D \supset \gamma$ |
| Goursat improvement | No need to assume $f'$ continuous |
| Deformation principle | Homotopic contours give equal integrals |
| Multiply connected | $\oint_{C_{\text{outer}}} = \sum \oint_{C_{\text{inner}}}$ |
| Key hypothesis | Simply connected domain (or deformation version) |
| Proof technique | Subdivision of triangles + differentiability |

---

## Quick Revision Questions

1. **State** the Cauchy–Goursat theorem precisely, including all hypotheses.

2. **Explain** Goursat's improvement over Cauchy's original theorem and why it matters.

3. **True or False:** $\oint_{|z|=1} e^z\,dz = 0$. Justify.

4. **Evaluate** $\oint_{|z|=2} \frac{z}{z^2+9}\,dz$ using Cauchy–Goursat.

5. **State** the deformation principle for multiply connected domains and illustrate with $\oint \frac{1}{z}\,dz$.

6. **Why** does the Cauchy–Goursat theorem fail for $\oint_{|z|=1} \frac{1}{z}\,dz$?

---

[← Previous: Line Integrals](02-line-integrals.md) | [Next: Cauchy Integral Formula →](04-cauchy-integral-formula.md)
