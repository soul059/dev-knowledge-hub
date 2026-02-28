# 6.4 Principal Value Integrals

[← Previous: Evaluation of Real Integrals](03-evaluation-of-real-integrals.md) | [Next: Integrals Involving Branch Points →](05-integrals-involving-branch-points.md)

---

## Chapter Overview

When the integrand has a **pole on the real axis**, the integral $\int_{-\infty}^{\infty} f(x)\,dx$ diverges in the ordinary sense. The **Cauchy principal value** regularizes such integrals by symmetrically excluding the singularity. We develop contour techniques using **indented contours** (semicircular indentations) to evaluate these.

---

## 1. Definition of Principal Value

### 1.1 At Infinity

$$\text{P.V.}\int_{-\infty}^{\infty} f(x)\,dx = \lim_{R\to\infty}\int_{-R}^{R} f(x)\,dx$$

(Symmetric limit, not independent limits at $\pm\infty$.)

### 1.2 At a Finite Singularity

If $f$ has a singularity at $x = c$ on $[a,b]$:

$$\text{P.V.}\int_a^b f(x)\,dx = \lim_{\epsilon \to 0^+}\left[\int_a^{c-\epsilon} f(x)\,dx + \int_{c+\epsilon}^b f(x)\,dx\right]$$

### 1.3 Example

$$\text{P.V.}\int_{-1}^{1} \frac{dx}{x} = \lim_{\epsilon\to 0^+}\left[\int_{-1}^{-\epsilon}\frac{dx}{x} + \int_{\epsilon}^{1}\frac{dx}{x}\right] = \lim_{\epsilon\to 0^+}[-\ln\epsilon + \ln\epsilon] = 0$$

(The ordinary integral diverges, but the P.V. exists by symmetry.)

---

## 2. Indented Contour Method

### 2.1 The Indented Contour

When $f$ has a **simple pole on the real axis** at $x = x_0$, we indent the contour:

```
Indented contour for pole at x₀ on the real axis:

          ╭──── Γ_R (large semicircle) ────╮
         ╱                                  ╲
        ╱         × (poles in UHP)           ╲
       ╱                                      ╲
  ────•──────────╮    ╭──────────────────────────•────
     -R          ╰────╯                           R
                  γ_ε
               (small semicircle
                over x₀, radius ε)

  Indent ABOVE x₀ to EXCLUDE it from the interior
```

### 2.2 Contribution of Small Semicircle

> **Lemma.** If $f$ has a simple pole at $z_0 \in \mathbb{R}$ and $\gamma_\epsilon$ is a semicircular arc of radius $\epsilon$ centered at $z_0$, traversed counterclockwise (from $z_0 - \epsilon$ to $z_0 + \epsilon$ over the top), then:
>
> $$\lim_{\epsilon\to 0}\int_{\gamma_\epsilon} f(z)\,dz = \pi i\cdot\text{Res}(f, z_0)$$

*Half* of $2\pi i$ times the residue — because the arc subtends angle $\pi$ (a semicircle), not $2\pi$ (a full circle).

### 2.3 The Formula

$$\text{P.V.}\int_{-\infty}^{\infty} f(x)\,dx = 2\pi i\sum_{\text{UHP}} \text{Res}(f, z_k) + \pi i\sum_{\text{real axis}} \text{Res}(f, x_j)$$

---

## 3. Design Examples

### Example 1: $\text{P.V.}\int_{-\infty}^{\infty} \frac{dx}{x(x^2+1)}$

**Step 1.** Simple pole at $x = 0$ (on real axis) and at $z = \pm i$.

**Step 2.** Use indented contour. UHP pole: $z = i$.

$$\text{Res}(f, i) = \frac{1}{i(2i)} = \frac{1}{-2} = -\frac{1}{2}$$

$$\text{Res}(f, 0) = \lim_{z\to 0} \frac{z}{z(z^2+1)} = 1$$

**Step 3.** Apply the formula:

$$\text{P.V.}\int_{-\infty}^{\infty}\frac{dx}{x(x^2+1)} = 2\pi i\left(-\frac{1}{2}\right) + \pi i(1) = -\pi i + \pi i = 0$$

**Verification:** The integrand is odd, so the P.V. should be $0$. ✓

### Example 2: $\text{P.V.}\int_{-\infty}^{\infty} \frac{\sin x}{x}\,dx$

**Step 1.** Consider $\int \frac{e^{iz}}{z}\,dz$. Simple pole at $z = 0$.

**Step 2.** Use indented contour (indent above $z = 0$). No poles in UHP.

$$\text{Res}(e^{iz}/z, 0) = \lim_{z\to 0} z \cdot \frac{e^{iz}}{z} = e^0 = 1$$

**Step 3.** Contributions:
- Large semicircle: $\to 0$ (Jordan's lemma with $a = 1 > 0$)
- Small semicircle: $\to -\pi i \cdot 1 = -\pi i$ (negative because traversed clockwise!)

**Step 4.** Full contour integral $= 0$ (no poles inside). So:

$$\text{P.V.}\int_{-\infty}^{\infty}\frac{e^{ix}}{x}\,dx + 0 - \pi i = 0$$

$$\text{P.V.}\int_{-\infty}^{\infty}\frac{e^{ix}}{x}\,dx = \pi i$$

**Step 5.** Take imaginary part:

$$\text{P.V.}\int_{-\infty}^{\infty}\frac{\sin x}{x}\,dx = \text{Im}(\pi i) = \pi$$

Since $\frac{\sin x}{x}$ is actually integrable (improper integral converges), P.V. = ordinary integral:

$$\boxed{\int_{-\infty}^{\infty}\frac{\sin x}{x}\,dx = \pi}$$

---

## 4. When P.V. Equals the Ordinary Integral

| Condition | P.V. = ordinary integral? |
|-----------|--------------------------|
| $f$ has no real singularities and $\int$ converges | Yes |
| $f$ is even: P.V. at $\pm\infty$ needed | Yes (symmetric) |
| $f$ has integrable singularity ($|f| \sim |x-c|^{-\alpha}$, $\alpha < 1$) | Yes |
| Non-integrable singularity ($\alpha = 1$) | P.V. may exist but $\neq$ ordinary integral |

---

## 5. Real-World Applications

| Application | P.V. Integrals Used |
|-------------|---------------------|
| **Kramers–Kronig relations** | P.V. integrals relate real/imaginary parts of susceptibility |
| **Hilbert transform** | $\mathcal{H}[f](t) = \frac{1}{\pi}\text{P.V.}\int \frac{f(\tau)}{t-\tau}\,d\tau$ |
| **Dispersion relations** | P.V. integrals in scattering theory |
| **Signal processing** | Analytic signal construction via Hilbert transform |

---

## Summary Table

| Concept | Formula / Fact |
|---------|---------------|
| P.V. at infinity | $\lim_{R\to\infty}\int_{-R}^R f\,dx$ |
| P.V. at singularity $c$ | $\lim_{\epsilon\to 0^+}\left[\int_{-\infty}^{c-\epsilon} + \int_{c+\epsilon}^{\infty}\right]$ |
| Small semicircle (simple pole) | Contributes $\pm \pi i \cdot \text{Res}$ |
| Indented contour formula | P.V.$\int = 2\pi i\sum$(UHP) $+ \pi i\sum$(real axis) |
| Dirichlet integral | $\int_{-\infty}^{\infty}\frac{\sin x}{x}\,dx = \pi$ |

---

## Quick Revision Questions

1. **Define** the Cauchy principal value integral at a finite singularity and at infinity.

2. **Prove** the half-residue lemma: a semicircular indent of angle $\alpha$ around a simple pole contributes $\alpha i \cdot \text{Res}$.

3. **Evaluate** $\text{P.V.}\int_{-\infty}^{\infty}\frac{dx}{x^2 - 1}$.

4. **Evaluate** $\int_{-\infty}^{\infty}\frac{\sin x}{x}\,dx$ using the indented contour method.

5. **Explain** the Kramers–Kronig relations and their connection to principal value integrals.

6. **Evaluate** $\text{P.V.}\int_{-\infty}^{\infty}\frac{\cos x}{x}\,dx$ and explain why this is zero.

---

[← Previous: Evaluation of Real Integrals](03-evaluation-of-real-integrals.md) | [Next: Integrals Involving Branch Points →](05-integrals-involving-branch-points.md)
