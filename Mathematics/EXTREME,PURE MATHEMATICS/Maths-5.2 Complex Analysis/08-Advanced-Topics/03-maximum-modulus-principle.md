# 8.3 Maximum Modulus Principle

[← Previous: Rouché's Theorem](02-rouches-theorem.md) | [Next: Schwarz Lemma →](04-schwarz-lemma.md)

---

## Chapter Overview

The **Maximum Modulus Principle** states that a non-constant analytic function cannot attain its maximum modulus in the interior of its domain — the maximum must occur on the boundary. This principle has profound consequences for bounding analytic functions, uniqueness, and the geometry of holomorphic maps.

---

## 1. Statement

### 1.1 Maximum Modulus Principle (MMP)

**Theorem.** If $f$ is analytic and non-constant on a domain $D$, then $|f(z)|$ has **no local maximum** in $D$.

**Corollary (Bounded domain form).** If $f$ is analytic on a bounded domain $D$ and continuous on $\overline{D}$, then:

$$\max_{z \in \overline{D}} |f(z)| = \max_{z \in \partial D} |f(z)|$$

The maximum of $|f|$ is attained **on the boundary** $\partial D$.

```
   ╭────────────╮
  ╱   Interior   ╲         |f| cannot have its max here
 │    |f| < M     │
 │                │         M = max on boundary
  ╲  ·····  ╱
   ╰────────╯
    boundary
   |f| = M here
```

### 1.2 Equivalent Formulation

If $f$ is analytic on $D$ and $|f(z_0)| \geq |f(z)|$ for all $z$ in some neighborhood of $z_0 \in D$, then $f$ is constant.

---

## 2. Proof

### 2.1 Via the Mean Value Property

Since $f$ is analytic, by the Cauchy integral formula (mean value property):

$$f(z_0) = \frac{1}{2\pi}\int_0^{2\pi} f(z_0 + re^{i\theta})\,d\theta$$

Taking moduli:

$$|f(z_0)| \leq \frac{1}{2\pi}\int_0^{2\pi} |f(z_0 + re^{i\theta})|\,d\theta \leq \max_{|z-z_0|=r}|f(z)|$$

If $|f(z_0)|$ is a maximum, equality holds throughout, which forces $|f|$ to be constant on the circle. Repeating for all radii and using connectedness shows $f$ is constant.

### 2.2 Via the Open Mapping Theorem

If $f$ is non-constant, $f$ is an open map. So the image $f(D)$ is open. If $|f(z_0)| = M = \max$, then the disk $|w| \leq M$ would need to contain an open neighborhood of $f(z_0)$, but $|w| > M$ points would be needed — contradiction.

---

## 3. Minimum Modulus Principle

**Theorem.** If $f$ is analytic and non-constant on $D$, and $f(z) \neq 0$ in $D$, then $|f|$ has **no local minimum** in $D$.

*Proof:* Apply MMP to $1/f$ (which is analytic since $f \neq 0$).

**Caution:** If $f$ has a zero in $D$, then $|f|$ trivially achieves minimum value $0$ in the interior. The hypothesis $f \neq 0$ is essential.

```
If f ≠ 0 in D:

  Max|f| on ∂D          Min|f| on ∂D
       │                     │
       ▼                     ▼
   ╭────────╮           ╭────────╮
  ╱ no max   ╲         ╱ no min   ╲
 │  inside    │       │  inside    │
  ╲          ╱         ╲          ╱
   ╰────────╯           ╰────────╯
```

---

## 4. Applications

### 4.1 Bound Propagation

If $|f(z)| \leq M$ on $\partial D$, then $|f(z)| \leq M$ throughout $D$.

This is used to bound solutions of PDEs and to prove uniqueness.

### 4.2 Uniqueness of Bounded Analytic Functions

If $f$ and $g$ are analytic on $D$, continuous on $\overline{D}$, and $f = g$ on $\partial D$, then $f = g$ on $D$.

*Proof:* $h = f - g$ satisfies $|h| = 0$ on $\partial D$, so $\max_{\overline{D}}|h| = 0$, hence $h \equiv 0$.

### 4.3 Liouville Revisited

MMP applied to $f$ on $|z| \leq R$ with $R \to \infty$ gives an alternative path to Liouville's theorem (bounded entire ⇒ constant).

---

## 5. Hadamard's Three-Lines Theorem

**Theorem.** Let $f$ be analytic on the strip $S = \{a < \text{Re}(z) < b\}$, continuous and bounded on $\overline{S}$. Let:

$$M(x) = \sup_{y \in \mathbb{R}} |f(x + iy)|$$

Then $\log M(x)$ is a **convex function** of $x$ on $[a, b]$:

$$M(x) \leq M(a)^{(b-x)/(b-a)} \cdot M(b)^{(x-a)/(b-a)}$$

```
  log M(x)
     ▲
     │    ╱
     │   ╱  ← convex: lies below the chord
     │  •
     │ ╱
     │╱
     ┼──────────► x
     a    x    b
```

This is a powerful interpolation inequality used in functional analysis.

---

## 6. Hadamard's Three-Circles Theorem

**Theorem.** Let $f$ be analytic on the annulus $r_1 < |z| < r_2$. Let:

$$M(r) = \max_{|z|=r} |f(z)|$$

Then $\log M(r)$ is a **convex function** of $\log r$:

$$\log M(r) \leq \frac{\log(r_2/r)}{\log(r_2/r_1)}\log M(r_1) + \frac{\log(r/r_1)}{\log(r_2/r_1)}\log M(r_2)$$

---

## 7. Phragmén–Lindelöf Principle

Extends MMP to **unbounded domains**. The basic idea: if $f$ is analytic on an unbounded domain, $|f| \leq M$ on the boundary, and $f$ doesn't grow too fast, then $|f| \leq M$ throughout.

**Theorem (Sector version).** If $f$ is analytic in a sector $|\arg z| < \pi/(2\alpha)$, continuous on the closure, $|f| \leq M$ on the boundary, and $|f(z)| = O(e^{|z|^\beta})$ for some $\beta < \alpha$, then $|f| \leq M$ in the sector.

---

## 8. Design Example

### Example: Show that $|e^z| \leq 1$ on the closed left half-plane implies $|e^z| < 1$ in the open left half-plane.

**Solution.** Let $D = \{z : \text{Re}(z) < 0\}$. On $\partial D$ (imaginary axis), $|e^{iy}| = 1$.

$f(z) = e^z$ is non-constant and analytic. On $\text{Re}(z) = 0$: $|f| = 1$.

As $\text{Re}(z) \to -\infty$, $|e^z| = e^{\text{Re}(z)} \to 0 < 1$.

By Phragmén–Lindelöf (since $|e^z| = e^{\text{Re}(z)} \leq e^{|z|}$, growth is at most exponential):

$$|e^z| < 1 \quad \text{for } \text{Re}(z) < 0$$

This is also directly verified: $|e^z| = e^{\text{Re}(z)} < e^0 = 1$ for $\text{Re}(z) < 0$. Direct computation works here, but Phragmén–Lindelöf handles cases where direct computation is impossible.

---

## 9. Real-World Applications

| Application | How MMP is Used |
|-------------|----------------|
| **PDE uniqueness** | Solution bounded by boundary data |
| **Stability analysis** | Transfer function bounded on imaginary axis → bounded in RHP |
| **Interpolation theory** | Three-lines theorem → Riesz–Thorin interpolation |
| **Number theory** | Bounding $L$-functions in critical strips |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Maximum Modulus Principle | $\max \|f\|$ occurs on $\partial D$, not interior |
| Minimum Modulus | Same if $f \neq 0$ (apply MMP to $1/f$) |
| Proof methods | Mean value property or Open Mapping Theorem |
| Boundary determines interior | If $\|f\| \leq M$ on $\partial D$, then throughout $D$ |
| Three-Lines Theorem | $\log M(x)$ convex on a strip |
| Three-Circles Theorem | $\log M(r)$ convex in $\log r$ on an annulus |
| Phragmén–Lindelöf | Extends MMP to unbounded domains with growth control |

---

## Quick Revision Questions

1. **State** the Maximum Modulus Principle and give two distinct proofs.

2. **Why** does the Minimum Modulus Principle require $f \neq 0$?

3. **Prove** that if $f$ is entire and $|f(z)| = 1$ for all $|z| = 1$, and $f$ has no zeros in $|z| < 1$, then $f$ is constant on $|z| \leq 1$.

4. **State** Hadamard's Three-Lines Theorem and describe its use in interpolation.

5. **Show** that a bounded entire function is constant using MMP (alternative to Liouville).

6. **What** additional hypothesis does the Phragmén–Lindelöf principle need beyond MMP?

---

[← Previous: Rouché's Theorem](02-rouches-theorem.md) | [Next: Schwarz Lemma →](04-schwarz-lemma.md)
