# 8.4 Schwarz Lemma

[← Previous: Maximum Modulus Principle](03-maximum-modulus-principle.md) | [Next: Analytic Continuation →](05-analytic-continuation.md)

---

## Chapter Overview

The **Schwarz Lemma** is a rigidity result for analytic self-maps of the unit disk that fix the origin. It provides sharp bounds on $|f(z)|$ and $|f'(0)|$ and completely characterizes the equality case as a rotation. It is the gateway to hyperbolic geometry on the disk.

---

## 1. Statement

**Theorem (Schwarz Lemma).** Let $f: \mathbb{D} \to \mathbb{D}$ be analytic with $f(0) = 0$, where $\mathbb{D} = \{|z| < 1\}$. Then:

1. $|f(z)| \leq |z|$ for all $z \in \mathbb{D}$
2. $|f'(0)| \leq 1$

Moreover, if equality holds in (1) for some $z \neq 0$, or in (2), then $f(z) = e^{i\theta}z$ for some $\theta \in \mathbb{R}$ (a rotation).

$$\boxed{f: \mathbb{D} \to \mathbb{D}, \; f(0) = 0 \implies |f(z)| \leq |z| \text{ and } |f'(0)| \leq 1}$$

---

## 2. Proof

**Define** $g(z) = f(z)/z$ for $z \neq 0$, and $g(0) = f'(0)$ (removable singularity).

Then $g$ is analytic on $\mathbb{D}$.

**On $|z| = r < 1$:**

$$|g(z)| = \frac{|f(z)|}{|z|} \leq \frac{1}{r}$$

since $|f(z)| < 1$ (as $f: \mathbb{D} \to \mathbb{D}$).

By the Maximum Modulus Principle applied to $g$ on $|z| \leq r$:

$$|g(z)| \leq \frac{1}{r} \quad \text{for all } |z| \leq r$$

Letting $r \to 1^-$:

$$|g(z)| \leq 1 \quad \text{for all } z \in \mathbb{D}$$

This gives both $|f(z)| \leq |z|$ and $|g(0)| = |f'(0)| \leq 1$.

**Equality case:** If $|g(z_0)| = 1$ for some $z_0 \in \mathbb{D}$, then $|g|$ attains its maximum in the interior, so $g$ is constant by MMP: $g(z) = e^{i\theta}$, hence $f(z) = e^{i\theta}z$. $\square$

```
Schwarz Lemma geometrically:

  ╭──────╮  𝔻                If f: 𝔻 → 𝔻 and f(0) = 0:
 ╱        ╲
│     •    │   z              f "contracts" toward 0:
│    ╱      │                 |f(z)| ≤ |z|
│   •       │   f(z)
 ╲        ╱                   Equality ⟹ rotation
  ╰──────╯

  |f(z)| ≤ |z|  (closer to center)
```

---

## 3. Schwarz–Pick Lemma

### 3.1 Statement

Removes the assumption $f(0) = 0$ by using the hyperbolic metric.

**Theorem (Schwarz–Pick).** Let $f: \mathbb{D} \to \mathbb{D}$ be analytic. Then for all $z_1, z_2 \in \mathbb{D}$:

$$\left|\frac{f(z_1) - f(z_2)}{1 - \overline{f(z_1)}f(z_2)}\right| \leq \left|\frac{z_1 - z_2}{1 - \bar{z}_1 z_2}\right|$$

and

$$\frac{|f'(z)|}{1 - |f(z)|^2} \leq \frac{1}{1 - |z|^2}$$

Equality holds if and only if $f$ is a disk automorphism (Möbius transformation of $\mathbb{D}$).

### 3.2 Hyperbolic Metric Interpretation

The **Poincaré (hyperbolic) metric** on $\mathbb{D}$ is:

$$ds = \frac{2|dz|}{1 - |z|^2}$$

The **hyperbolic distance** is:

$$d_{\mathbb{D}}(z_1, z_2) = \tanh^{-1}\left|\frac{z_1 - z_2}{1 - \bar{z}_1 z_2}\right|$$

Schwarz–Pick says: **analytic self-maps of the disk are contractions in the hyperbolic metric**.

$$d_{\mathbb{D}}(f(z_1), f(z_2)) \leq d_{\mathbb{D}}(z_1, z_2)$$

Equality (isometry) ⟺ $f$ is a disk automorphism.

```
Hyperbolic geometry in 𝔻:

  ╭──────╮
 ╱   ╭──╮ ╲        Geodesics are arcs of circles
│   │    │  │       perpendicular to ∂𝔻
│   │  ○ │  │
│   │    │  │       Distances grow near the boundary
 ╲   ╰──╯ ╱        (hyperbolic metric blows up)
  ╰──────╯
```

---

## 4. Consequences and Applications

### 4.1 Automorphisms of the Disk

**Corollary.** Every automorphism of $\mathbb{D}$ has the form:

$$f(z) = e^{i\theta}\frac{z - a}{1 - \bar{a}z}, \quad |a| < 1, \; \theta \in \mathbb{R}$$

*Proof:* If $f$ is a bijection $\mathbb{D} \to \mathbb{D}$, apply Schwarz–Pick to both $f$ and $f^{-1}$ to get equality, forcing $f$ to be Möbius.

### 4.2 Fixed Point Theorem

**Corollary.** If $f: \mathbb{D} \to \mathbb{D}$ is analytic (not an automorphism) with a fixed point $f(a) = a$ in $\mathbb{D}$, then:

$$|f'(a)| < 1$$

The fixed point is **attracting**.

### 4.3 Denjoy–Wolff Theorem (Preview)

For $f: \mathbb{D} \to \mathbb{D}$ analytic with no fixed point in $\mathbb{D}$, the iterates $f^n = f \circ \cdots \circ f$ converge uniformly on compact subsets to a point on $\partial\mathbb{D}$.

---

## 5. Schwarz Reflection Principle

### 5.1 Statement

**Theorem.** Let $f$ be analytic on a domain $D$ that is symmetric about the real axis, with $D^+ = D \cap \{\text{Im}(z) > 0\}$. If $f$ is real-valued on $D \cap \mathbb{R}$, then:

$$f(\bar{z}) = \overline{f(z)}$$

More generally, if $f$ is analytic on $D^+$, continuous on $D^+ \cup I$ (where $I$ is an interval of $\mathbb{R}$), and maps $I$ to $\mathbb{R}$, then $f$ extends analytically to the reflection $D^+ \cup I \cup \overline{D^+}$ by:

$$f(z) = \overline{f(\bar{z})} \quad \text{for } z \in \overline{D^+}$$

```
  ╭────────╮
  │  D⁺    │   f analytic here
  │        │
  ├────────┤   f real on this axis
  │  D⁻    │   f extends here by reflection
  │        │
  ╰────────╯
```

### 5.2 Application

Used to extend the Riemann zeta function, construct analytic continuations, and simplify boundary-value problems.

---

## 6. Design Example

### Example: If $f: \mathbb{D} \to \mathbb{D}$ is analytic with $f(0) = 0$ and $f(1/2) = 0$, how large can $|f'(0)|$ be?

**Solution.** Since $f(0) = 0$ and $f(1/2) = 0$, define:

$$g(z) = \frac{f(z)}{z} \cdot \frac{1}{\phi_{1/2}(z)} \quad \text{where} \quad \phi_{1/2}(z) = \frac{z - 1/2}{1 - z/2}$$

Wait — more carefully: $f$ has zeros at $0$ and $1/2$. Write:

$$f(z) = z \cdot \frac{z - 1/2}{1 - z/2} \cdot h(z)$$

where $h: \mathbb{D} \to \mathbb{D}$ is analytic (by Schwarz–Pick, after dividing out the Blaschke factors).

Then $f'(0) = \frac{-1/2}{1} \cdot h(0) = -\frac{1}{2}h(0)$.

By Schwarz lemma on $h$: $|h(0)| \leq 1$.

So $|f'(0)| \leq 1/2$.

Equality when $h(z) = e^{i\theta}$ (constant of modulus 1).

$$\boxed{\max |f'(0)| = \frac{1}{2}}$$

---

## 7. Real-World Applications

| Application | How Schwarz Lemma is Used |
|-------------|--------------------------|
| **Hyperbolic geometry** | Self-maps contract hyperbolic distance |
| **Fixed point theory** | Attracting fixed points in iteration |
| **Complex dynamics** | Classifying dynamics near fixed points |
| **Entire function theory** | Growth bounds via generalizations |
| **Teichmüller theory** | Extremal mappings between Riemann surfaces |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Schwarz Lemma | $f: \mathbb{D} \to \mathbb{D}$, $f(0) = 0$ ⟹ $\|f(z)\| \leq \|z\|$, $\|f'(0)\| \leq 1$ |
| Equality | Iff $f(z) = e^{i\theta}z$ (rotation) |
| Schwarz–Pick | Analytic $\mathbb{D} \to \mathbb{D}$ contracts hyperbolic metric |
| Disk automorphisms | $e^{i\theta}(z-a)/(1-\bar{a}z)$ — the only isometries |
| Fixed points | Interior fixed point of non-automorphism is attracting |
| Reflection Principle | Real-on-axis ⟹ $f(\bar{z}) = \overline{f(z)}$ |

---

## Quick Revision Questions

1. **State** the Schwarz Lemma and prove it using the Maximum Modulus Principle.

2. **When** does equality hold in the Schwarz Lemma? What is the function?

3. **State** the Schwarz–Pick Lemma and interpret it in terms of hyperbolic distance.

4. **Prove** that every automorphism of the unit disk is a Möbius transformation.

5. **If** $f: \mathbb{D} \to \mathbb{D}$ is analytic with $f(0) = 0$ and $f(1/3) = 0$, find the maximum possible value of $|f'(0)|$.

6. **State** the Schwarz Reflection Principle and give one application.

---

[← Previous: Maximum Modulus Principle](03-maximum-modulus-principle.md) | [Next: Analytic Continuation →](05-analytic-continuation.md)
