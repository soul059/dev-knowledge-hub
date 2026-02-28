# 2.5 Analytic Functions

[← Previous: Cauchy–Riemann Equations](04-cauchy-riemann-equations.md) | [Next: Harmonic Functions →](06-harmonic-functions.md)

---

## Chapter Overview

A function that is complex-differentiable in a neighborhood of every point of its domain is called **analytic** (or **holomorphic**). This is the central object of study in complex analysis. Analytic functions possess an extraordinary collection of properties — far beyond what their definition would suggest — making them among the most well-behaved objects in all of mathematics.

---

## 1. Definitions

### 1.1 Analytic at a Point

$f$ is **analytic at $z_0$** if $f$ is complex-differentiable at every point in some open neighborhood of $z_0$.

> Note: being differentiable at $z_0$ alone is NOT enough. We need differentiability in an open disk around $z_0$.

### 1.2 Analytic on a Domain

$f$ is **analytic on an open set $D$** if it is analytic at every point of $D$.

### 1.3 Entire Function

$f$ is **entire** if it is analytic on all of $\mathbb{C}$.

### 1.4 Terminology

| Term | Meaning | Usage |
|------|---------|-------|
| **Analytic** | Complex-differentiable in a neighborhood | Standard in applied math |
| **Holomorphic** | Same as analytic | Standard in pure math |
| **Regular** | Same as analytic | Older terminology |
| **Entire** | Analytic on all of $\mathbb{C}$ | $e^z$, polynomials, $\sin z$ |

---

## 2. Examples and Non-Examples

### 2.1 Entire Functions

| Function | Why Entire |
|----------|-----------|
| Polynomials $P(z)$ | Differentiable everywhere |
| $e^z$ | C–R equations hold everywhere |
| $\sin z$, $\cos z$ | Defined by everywhere-convergent series |
| $\sinh z$, $\cosh z$ | Same reason |

### 2.2 Analytic on a Domain (Not Entire)

| Function | Domain of Analyticity | Singularity |
|----------|----------------------|-------------|
| $1/z$ | $\mathbb{C} \setminus \{0\}$ | Pole at $0$ |
| $1/(z^2+1)$ | $\mathbb{C} \setminus \{i, -i\}$ | Poles at $\pm i$ |
| $\text{Log}(z)$ | $\mathbb{C} \setminus (-\infty, 0]$ | Branch cut |
| $\sqrt{z}$ | $\mathbb{C} \setminus (-\infty, 0]$ | Branch point at $0$ |

### 2.3 Nowhere Analytic

| Function | Why Not Analytic |
|----------|-----------------|
| $\bar{z}$ | C–R equations fail everywhere |
| $\text{Re}(z)$ | C–R equations fail everywhere |
| $|z|$ | Not even differentiable (except possibly at 0) |

---

## 3. Remarkable Properties of Analytic Functions

### 3.1 The Cascade of Consequences

A single assumption — complex differentiability in a neighborhood — triggers an astonishing chain:

```
f is analytic (differentiable in a neighborhood)
    │
    ├──► f is infinitely differentiable (C^∞)
    │
    ├──► f has a convergent Taylor series at every point
    │
    ├──► f satisfies Cauchy's integral formula
    │
    ├──► f satisfies the maximum modulus principle
    │
    ├──► f is conformal where f'≠ 0
    │
    ├──► Re(f) and Im(f) are harmonic
    │
    ├──► f is determined by its values on any arc
    │        (Identity Theorem)
    │
    └──► Zeros of f are isolated (unless f ≡ 0)
```

> **Nothing remotely like this happens for real-differentiable functions.** This is the miracle of complex analysis.

### 3.2 Comparison with Real Analysis

| Property | Real $C^1$ functions | Analytic (complex) functions |
|----------|---------------------|------------------------------|
| Differentiable once ⟹ twice? | **No** | **Yes** (infinitely many times!) |
| Power series representation? | **No** (need $C^\infty$ + convergence) | **Always** |
| Determined by values on a set with limit point? | **No** | **Yes** (Identity Theorem) |
| Max/min in interior? | Can occur | **No** max of $\|f\|$ inside domain |
| Zeros isolated? | **No** | **Yes** (unless $f \equiv 0$) |

---

## 4. Key Theorems (Previews)

### 4.1 Identity Theorem

> If $f$ and $g$ are analytic on a connected domain $D$ and $f(z_n) = g(z_n)$ for a sequence $z_n \to z_0 \in D$, then $f \equiv g$ on $D$.

**Consequence**: An analytic function is completely determined by its values on any set with a limit point in the domain.

### 4.2 Liouville's Theorem

> A bounded entire function is constant.

**Application**: Proof of the Fundamental Theorem of Algebra — every non-constant polynomial has a root.

### 4.3 Open Mapping Theorem

> A non-constant analytic function maps open sets to open sets.

### 4.4 Maximum Modulus Principle (Preview)

> If $f$ is analytic and non-constant on a connected domain $D$, then $|f|$ has no maximum on $D$.

---

## 5. Singular Points

### 5.1 Definition

A point $z_0$ is a **singular point** (or **singularity**) of $f$ if $f$ is not analytic at $z_0$ but is analytic at some point in every neighborhood of $z_0$.

### 5.2 Classification (Preview)

| Type | Characterization | Example |
|------|-----------------|---------|
| **Removable** | $\lim_{z \to z_0} f(z)$ exists and is finite | $\frac{\sin z}{z}$ at $z=0$ |
| **Pole** | $\lim_{z \to z_0} |f(z)| = \infty$ | $\frac{1}{z}$ at $z=0$ |
| **Essential** | Neither removable nor pole | $e^{1/z}$ at $z=0$ |
| **Branch point** | Multi-valuedness | $\sqrt{z}$ at $z=0$ |

---

## 6. Constructing Analytic Functions

### 6.1 Given $u$, Find $v$ (Harmonic Conjugate)

If $u(x,y)$ is harmonic (satisfies $\nabla^2 u = 0$), find $v$ such that $f = u + iv$ is analytic:

1. From $v_x = -u_y$, integrate w.r.t. $x$: $v = -\int u_y\, dx + \phi(y)$
2. From $v_y = u_x$, determine $\phi(y)$

### 6.2 Example: Given $u = x^2 - y^2$, find analytic $f$.

**Step 1.** Check harmonic: $u_{xx} + u_{yy} = 2 + (-2) = 0$ ✓

**Step 2.** From C–R: $v_y = u_x = 2x$ ⟹ $v = 2xy + g(x)$

**Step 3.** From C–R: $v_x = -u_y = 2y$ ⟹ $v_x = 2y + g'(x) = 2y$, so $g'(x) = 0$, $g(x) = C$

**Step 4.** $v = 2xy + C$

**Result**: $f(z) = (x^2 - y^2) + i(2xy) + iC = z^2 + iC$

---

## 7. The Milne-Thomson Method

An efficient alternative: given $u(x,y)$, form $f(z)$ by formally substituting $x = z$, $y = 0$ in $u_x - iu_y$ and integrating:

$$f'(z) = u_x(z, 0) - iu_y(z, 0)$$

$$f(z) = \int f'(z)\, dz + C$$

### Example: $u = x^3 - 3xy^2$

$u_x = 3x^2 - 3y^2$, $u_y = -6xy$

$f'(z) = u_x(z,0) - iu_y(z,0) = 3z^2 - 0 = 3z^2$

$f(z) = z^3 + C$

**Check**: $z^3 = (x+iy)^3 = x^3 - 3xy^2 + i(3x^2y - y^3)$, so $u = x^3 - 3xy^2$ ✓

---

## 8. Real-World Applications

| Application | How Analyticity Is Used |
|-------------|------------------------|
| **Aerodynamics** | Conformal mapping of airfoil shapes (Joukowski transform) |
| **Fluid Flow** | Complex potential $\Phi(z) = \phi + i\psi$ is analytic for irrotational, incompressible flow |
| **Electrostatics** | Complex potential for 2D charge distributions |
| **Quantum Mechanics** | Analytic continuation of scattering amplitudes |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Analytic at $z_0$ | Differentiable in a neighborhood of $z_0$ |
| Entire | Analytic on all of $\mathbb{C}$ |
| Holomorphic = Analytic | Same concept, different names |
| Cascade | Differentiable ⟹ $C^\infty$ ⟹ power series ⟹ ... |
| Identity Theorem | Values on any convergent sequence determine $f$ |
| Liouville | Bounded + entire ⟹ constant |
| Singularities | Removable, pole, essential, branch point |
| Harmonic conjugate | Given $u$, find $v$ via C–R equations |
| Milne-Thomson | $f'(z) = u_x(z,0) - iu_y(z,0)$ |

---

## Quick Revision Questions

1. **Define** what it means for $f$ to be analytic at $z_0$ and explain why differentiability at $z_0$ alone is insufficient.

2. **List** five properties that an analytic function possesses which a merely real-differentiable function does not.

3. **Classify** the singularities of $f(z) = \frac{z}{(z-1)(z-2)^2}$.

4. **Find** the analytic function $f(z)$ whose real part is $u = e^x \cos y$.

5. **State** Liouville's theorem and explain how it implies the Fundamental Theorem of Algebra.

6. **Use** the Milne-Thomson method to find the analytic function whose real part is $u = x^2 - y^2 + x$.

---

[← Previous: Cauchy–Riemann Equations](04-cauchy-riemann-equations.md) | [Next: Harmonic Functions →](06-harmonic-functions.md)
