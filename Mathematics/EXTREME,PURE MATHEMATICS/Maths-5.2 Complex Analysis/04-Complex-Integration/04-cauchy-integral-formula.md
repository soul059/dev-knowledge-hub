# 4.4 Cauchy Integral Formula

[← Previous: Cauchy–Goursat Theorem](03-cauchy-goursat-theorem.md) | [Next: Derivatives of Analytic Functions →](05-derivatives-of-analytic-functions.md)

---

## Chapter Overview

The **Cauchy Integral Formula** (CIF) is arguably the most powerful result in complex analysis. It says that the values of an analytic function inside a contour are completely determined by its values *on* the contour. This has staggering consequences: infinite differentiability, power series representations, the maximum modulus principle, and much more.

---

## 1. The Cauchy Integral Formula

### 1.1 Statement

Let $f$ be analytic on and inside a simple closed contour $C$ (positively oriented), and let $z_0$ be any point inside $C$. Then:

$$\boxed{f(z_0) = \frac{1}{2\pi i}\oint_C \frac{f(z)}{z - z_0}\,dz}$$

### 1.2 Interpretation

- The value $f(z_0)$ at an interior point is determined by values of $f$ on the boundary.
- The kernel $\frac{1}{z - z_0}$ is the **Cauchy kernel**.
- This is a *reproducing formula* — the function reproduces itself from its boundary values.

```
Cauchy Integral Formula — geometric picture:

        ╭────── C ──────╮
       ╱                  ╲
      │                    │
      │       • z₀         │       f(z₀) = (1/2πi) ∮_C f(z)/(z-z₀) dz
      │                    │
       ╲                  ╱
        ╰────────────────╯

  Values on the boundary ──► Value at any interior point
```

---

## 2. Proof Outline

**Step 1.** $\frac{f(z)}{z - z_0}$ is analytic everywhere inside $C$ **except** at $z = z_0$.

**Step 2.** By the deformation principle, replace $C$ with a small circle $C_\epsilon: |z - z_0| = \epsilon$:

$$\oint_C \frac{f(z)}{z - z_0}\,dz = \oint_{C_\epsilon} \frac{f(z)}{z - z_0}\,dz$$

**Step 3.** On $C_\epsilon$, write $z = z_0 + \epsilon e^{it}$:

$$\oint_{C_\epsilon} \frac{f(z)}{z - z_0}\,dz = \int_0^{2\pi} \frac{f(z_0 + \epsilon e^{it})}{\epsilon e^{it}} \cdot i\epsilon e^{it}\,dt = i\int_0^{2\pi} f(z_0 + \epsilon e^{it})\,dt$$

**Step 4.** As $\epsilon \to 0$, by continuity of $f$:

$$\to i\int_0^{2\pi} f(z_0)\,dt = 2\pi i\,f(z_0)$$

Therefore $\oint_C \frac{f(z)}{z - z_0}\,dz = 2\pi i\,f(z_0)$. $\square$

---

## 3. The Mean Value Property

From the proof (Step 3 with $C_\epsilon$ replaced by $C_R$: $|z - z_0| = R$):

$$f(z_0) = \frac{1}{2\pi}\int_0^{2\pi} f(z_0 + Re^{it})\,dt$$

> *The value of an analytic function at the center of a circle equals the average of its values on the circle.*

This is the **mean value property** — a hallmark of harmonic and analytic functions.

---

## 4. Generalized CIF for Derivatives

### 4.1 Statement

Under the same hypotheses:

$$\boxed{f^{(n)}(z_0) = \frac{n!}{2\pi i}\oint_C \frac{f(z)}{(z - z_0)^{n+1}}\,dz}$$

### 4.2 Key Consequence

Every analytic function is **infinitely differentiable**! This stands in stark contrast to real analysis, where differentiable functions need not be twice differentiable.

| Property | Real Analysis | Complex Analysis |
|----------|--------------|-----------------|
| $f$ differentiable | $f'$ may not exist | $f'$ exists and is analytic |
| $f$ once differentiable | May not be twice differentiable | $f$ is $C^\infty$ |
| $f$ smooth | Need not be analytic | Automatically analytic |

---

## 5. Cauchy's Inequality

From the generalized CIF and the ML-inequality:

$$|f^{(n)}(z_0)| \leq \frac{n!\,M(R)}{R^n}$$

where $M(R) = \max_{|z-z_0|=R} |f(z)|$ and $R$ is the radius of a circle around $z_0$ on which $f$ is analytic.

---

## 6. Design Examples

### Example 1: Evaluate $\oint_{|z|=2} \frac{e^z}{z-1}\,dz$

**Step 1.** Identify the form: $\oint_C \frac{f(z)}{z - z_0}\,dz$ with $f(z) = e^z$, $z_0 = 1$.

**Step 2.** Check: $z_0 = 1$ is inside $|z| = 2$? Yes ($|1| < 2$).

**Step 3.** Apply CIF:

$$\oint_{|z|=2} \frac{e^z}{z-1}\,dz = 2\pi i\,f(1) = 2\pi i\,e$$

### Example 2: Evaluate $\oint_{|z|=1} \frac{\sin z}{z^3}\,dz$

**Step 1.** Write as $\oint_C \frac{f(z)}{(z-0)^{2+1}}\,dz$ with $f(z) = \sin z$, $z_0 = 0$, $n = 2$.

**Step 2.** Apply the generalized CIF:

$$\oint_{|z|=1} \frac{\sin z}{z^3}\,dz = \frac{2\pi i}{2!}\,f''(0)$$

**Step 3.** $f(z) = \sin z$, $f''(z) = -\sin z$, $f''(0) = 0$.

$$= \frac{2\pi i}{2} \cdot 0 = 0$$

### Example 3: Evaluate $\oint_{|z|=2} \frac{z}{(z-1)^2}\,dz$

**Step 1.** Form: $\frac{f(z)}{(z - z_0)^{n+1}}$ with $f(z) = z$, $z_0 = 1$, $n = 1$.

**Step 2.** Apply generalized CIF:

$$= \frac{2\pi i}{1!}\,f'(1) = 2\pi i \cdot 1 = 2\pi i$$

---

## 7. Real-World Applications

| Application | How CIF Appears |
|-------------|-----------------|
| **Signal processing** | Recovering interior signal from boundary measurements |
| **Holography** | Reconstructing 3D information from 2D boundary data |
| **Potential theory** | Interior potential from boundary potential (Poisson formula) |
| **Control theory** | Transfer function recovery from frequency response |

---

## Summary Table

| Result | Formula |
|--------|---------|
| Cauchy Integral Formula | $f(z_0) = \frac{1}{2\pi i}\oint_C \frac{f(z)}{z-z_0}\,dz$ |
| Generalized CIF | $f^{(n)}(z_0) = \frac{n!}{2\pi i}\oint_C \frac{f(z)}{(z-z_0)^{n+1}}\,dz$ |
| Mean Value Property | $f(z_0) = \frac{1}{2\pi}\int_0^{2\pi} f(z_0+Re^{it})\,dt$ |
| Cauchy's Inequality | $|f^{(n)}(z_0)| \leq \frac{n!\,M(R)}{R^n}$ |
| Infinite differentiability | Analytic $\Rightarrow C^\infty$ |

---

## Quick Revision Questions

1. **State** the Cauchy Integral Formula and its generalization to $n$-th derivatives.

2. **Evaluate** $\oint_{|z|=3} \frac{\cos z}{z-\pi}\,dz$.

3. **Evaluate** $\oint_{|z|=1} \frac{e^z}{z^4}\,dz$ using the generalized CIF.

4. **Prove** Cauchy's inequality from the generalized CIF and the ML-inequality.

5. **Explain** why the CIF implies that an analytic function is infinitely differentiable.

6. **Evaluate** $\oint_{|z|=2} \frac{z^2+1}{(z-1)^3}\,dz$.

---

[← Previous: Cauchy–Goursat Theorem](03-cauchy-goursat-theorem.md) | [Next: Derivatives of Analytic Functions →](05-derivatives-of-analytic-functions.md)
