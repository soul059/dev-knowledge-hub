# 4.6 Morera's Theorem

[← Previous: Derivatives of Analytic Functions](05-derivatives-of-analytic-functions.md) | [Next: Unit 5 — Taylor Series →](../05-Series-Representations/01-taylor-series.md)

---

## Chapter Overview

**Morera's Theorem** provides a powerful *converse* to the Cauchy–Goursat theorem: if all contour integrals of a continuous function vanish, then the function is analytic. It is frequently used to prove analyticity indirectly — especially for limits, integrals depending on parameters, and series.

---

## 1. Statement

### 1.1 Morera's Theorem

> **Theorem (Morera, 1886).** Let $f$ be continuous on a domain $D$. If
> $$\oint_\gamma f(z)\,dz = 0$$
> for every closed contour $\gamma$ in $D$, then $f$ is **analytic** in $D$.

### 1.2 Sufficient to Check Triangles

It suffices to check the condition for all triangular contours $\Delta$ in $D$:

$$\oint_\Delta f(z)\,dz = 0 \quad \text{for all triangles } \Delta \subset D$$

### 1.3 Relationship to Cauchy–Goursat

```
Cauchy–Goursat and Morera — Logical Equivalence
(on simply connected domains):

   f analytic on D ══════════════════ ∮_γ f dz = 0
                    Cauchy─Goursat
                    ──────────────►
                    ◄──────────────
                       Morera
```

| Direction | Theorem | Hypotheses |
|-----------|---------|------------|
| $\Rightarrow$ | Cauchy–Goursat | $f$ analytic, $D$ simply connected |
| $\Leftarrow$ | Morera | $f$ continuous, all contour integrals vanish |

---

## 2. Proof

**Step 1.** Fix $z_0 \in D$. Define:

$$F(z) = \int_{z_0}^{z} f(\zeta)\,d\zeta$$

Since all closed contour integrals vanish, the integral is path-independent, so $F(z)$ is well-defined.

**Step 2.** Show $F'(z) = f(z)$:

$$F(z+h) - F(z) = \int_z^{z+h} f(\zeta)\,d\zeta$$

$$\frac{F(z+h)-F(z)}{h} - f(z) = \frac{1}{h}\int_z^{z+h} [f(\zeta) - f(z)]\,d\zeta$$

By continuity: for any $\epsilon > 0$, if $|h|$ is small enough, $|f(\zeta) - f(z)| < \epsilon$ on the segment. By ML-inequality:

$$\left|\frac{F(z+h)-F(z)}{h} - f(z)\right| \leq \frac{1}{|h|} \cdot \epsilon \cdot |h| = \epsilon$$

So $F'(z) = f(z)$.

**Step 3.** $F$ is analytic (has a derivative $f$). By the infinite differentiability theorem (Chapter 4.5), $F' = f$ is also analytic. $\square$

---

## 3. Applications of Morera's Theorem

### 3.1 Uniform Limits of Analytic Functions

> **Theorem.** If $\{f_n\}$ is a sequence of analytic functions on $D$ converging uniformly to $f$ on compact subsets of $D$, then $f$ is analytic on $D$.

**Proof:** 
- Each $f_n$ is continuous → $f$ is continuous (uniform limit).
- For any closed contour $\gamma$ in $D$:

$$\oint_\gamma f\,dz = \oint_\gamma \lim f_n\,dz = \lim \oint_\gamma f_n\,dz = \lim 0 = 0$$

(interchange of limit and integral justified by uniform convergence.)

- By Morera: $f$ is analytic. $\square$

### 3.2 Integrals Depending on a Parameter

> **Theorem.** If $g(z, t)$ is continuous on $D \times [a,b]$, analytic in $z$ for each fixed $t$, then
> $$F(z) = \int_a^b g(z, t)\,dt$$
> is analytic on $D$.

**Proof:** $F$ is continuous. For any triangle $\Delta$ in $D$:

$$\oint_\Delta F(z)\,dz = \oint_\Delta \int_a^b g(z,t)\,dt\,dz = \int_a^b \oint_\Delta g(z,t)\,dz\,dt = \int_a^b 0\,dt = 0$$

By Morera: $F$ is analytic. $\square$

### 3.3 Analyticity of Power Series

If $\sum a_n z^n$ converges uniformly on compact subsets of $|z| < R$, each partial sum is analytic (polynomial), so the sum is analytic by §3.1.

---

## 4. Comparison of Methods for Proving Analyticity

| Method | When to Use |
|--------|------------|
| Cauchy–Riemann equations | $f$ given explicitly as $u + iv$ |
| Direct derivative computation | $f$ given by a simple formula |
| Morera's theorem | $f$ defined as limit, integral, or series |
| Composition rules | $f = g \circ h$ with $g, h$ analytic |
| Algebraic closure | Sums, products, quotients of analytic functions |

---

## 5. Design Example

### Example: Show that $F(z) = \int_0^1 \frac{t^z}{1+t}\,dt$ is analytic for $\text{Re}(z) > 0$.

**Step 1.** Set $g(z,t) = \frac{t^z}{1+t} = \frac{e^{z \ln t}}{1+t}$ for $t \in (0, 1]$.

**Step 2.** For fixed $t \in (0,1]$, $g(z,t) = \frac{e^{z \ln t}}{1+t}$ is entire in $z$ (exponential of a linear function).

**Step 3.** Check continuity on $\{z : \text{Re}(z) \geq \delta\} \times [0, 1]$ for any $\delta > 0$:

$$|g(z,t)| = \frac{t^{\text{Re}(z)}}{1+t} \leq t^\delta$$

which is integrable on $[0,1]$.

**Step 4.** By the parameter-dependent integral theorem (§3.2), $F(z)$ is analytic for $\text{Re}(z) > 0$.

---

## 6. Real-World Applications

| Application | Morera's Role |
|-------------|--------------|
| **Quantum mechanics** | Analyticity of Green's functions defined as integrals |
| **Probability** | Analyticity of characteristic functions |
| **Number theory** | Analyticity of Dirichlet series in half-planes |
| **PDE theory** | Regularity: solutions to elliptic PDEs are analytic |

---

## Summary Table

| Concept | Statement |
|---------|-----------|
| Morera's theorem | Continuous + all contour integrals vanish → analytic |
| Converse of | Cauchy–Goursat theorem |
| Key technique | Construct antiderivative $F(z) = \int_{z_0}^z f$ |
| Uniform limits | Uniform limit of analytic functions is analytic |
| Parameter integrals | $\int_a^b g(z,t)\,dt$ is analytic if $g$ is analytic in $z$ |
| Triangle version | Suffices to check $\oint_\Delta f\,dz = 0$ for all triangles |

---

## Quick Revision Questions

1. **State** Morera's theorem precisely. What is the relationship with the Cauchy–Goursat theorem?

2. **Prove** that if $f_n \to f$ uniformly on compact subsets and each $f_n$ is analytic, then $f$ is analytic.

3. **Why** is it sufficient to check only triangular contours in Morera's theorem?

4. **Show** that $f(z) = \sum_{n=0}^\infty z^n/n^2$ is analytic for $|z| < 1$ using Morera's theorem.

5. **Explain** why Morera's theorem is useful even though we already have the Cauchy–Riemann equations.

6. **Apply** Morera's theorem to prove that $F(z) = \int_0^\infty e^{-zt}\,dt$ is analytic for $\text{Re}(z) > 0$.

---

[← Previous: Derivatives of Analytic Functions](05-derivatives-of-analytic-functions.md) | [Next: Unit 5 — Taylor Series →](../05-Series-Representations/01-taylor-series.md)
