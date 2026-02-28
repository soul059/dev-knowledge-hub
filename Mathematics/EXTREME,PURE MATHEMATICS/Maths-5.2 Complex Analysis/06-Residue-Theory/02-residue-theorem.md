# 6.2 The Residue Theorem

[← Previous: Residues at Poles](01-residues-at-poles.md) | [Next: Evaluation of Real Integrals →](03-evaluation-of-real-integrals.md)

---

## Chapter Overview

The **Residue Theorem** is the grand unification of complex integration. It reduces any contour integral of a meromorphic function to a sum of residues — a finite algebraic computation. This single theorem subsumes the Cauchy–Goursat theorem, the Cauchy integral formula, and provides the primary computational tool for evaluating integrals.

---

## 1. Statement

### 1.1 The Residue Theorem

> Let $f$ be analytic inside and on a simple closed positively oriented contour $C$, except at finitely many isolated singularities $z_1, z_2, \ldots, z_n$ inside $C$. Then:
>
> $$\boxed{\oint_C f(z)\,dz = 2\pi i \sum_{k=1}^{n} \text{Res}(f, z_k)}$$

```
The Residue Theorem:

       ╭──────── C ────────╮
      ╱                     ╲
     │    ×z₁    ×z₂        │      ∮_C f dz = 2πi [Res(f,z₁)
     │                      │                     + Res(f,z₂)
     │         ×z₃          │                     + Res(f,z₃)]
      ╲                    ╱
       ╰───────────────────╯
```

### 1.2 Proof Sketch

Surround each $z_k$ by a small circle $C_k$. By the deformation principle:

$$\oint_C f\,dz = \sum_{k=1}^{n}\oint_{C_k} f\,dz = \sum_{k=1}^{n} 2\pi i\,\text{Res}(f, z_k) \quad \square$$

---

## 2. Special Cases Recovery

| If... | Then the Residue Theorem gives... |
|-------|----------------------------------|
| No singularities inside $C$ | $\oint_C f\,dz = 0$ (Cauchy–Goursat) |
| One simple pole at $z_0$ with $f = g/(z-z_0)$ | $\oint_C \frac{g(z)}{z-z_0}\,dz = 2\pi i\,g(z_0)$ (Cauchy Integral Formula) |
| Pole of order $n+1$ at $z_0$ | Gives the generalized CIF |

The Residue Theorem is the **master theorem** from which all earlier integration results follow.

---

## 3. Design Examples

### Example 1: $\oint_{|z|=3} \frac{1}{z^2-1}\,dz$

**Step 1.** Factor: $\frac{1}{z^2-1} = \frac{1}{(z-1)(z+1)}$.

**Step 2.** Singularities: $z = 1$ and $z = -1$, both inside $|z| = 3$.

**Step 3.** Residues (simple poles, quotient rule with $p = 1$, $q = z^2 - 1$, $q' = 2z$):

$$\text{Res}(f, 1) = \frac{1}{2(1)} = \frac{1}{2}, \quad \text{Res}(f, -1) = \frac{1}{2(-1)} = -\frac{1}{2}$$

**Step 4.** Result:

$$\oint_{|z|=3} \frac{dz}{z^2-1} = 2\pi i\left(\frac{1}{2} - \frac{1}{2}\right) = 0$$

### Example 2: $\oint_{|z|=2} \frac{e^z}{z(z-1)}\,dz$

**Step 1.** Simple poles at $z = 0$ and $z = 1$, both inside $|z| = 2$.

**Step 2.** Residues:
$$\text{Res}(f, 0) = \lim_{z\to 0} z \cdot \frac{e^z}{z(z-1)} = \frac{e^0}{0-1} = -1$$

$$\text{Res}(f, 1) = \lim_{z\to 1}(z-1)\frac{e^z}{z(z-1)} = \frac{e^1}{1} = e$$

**Step 3.** Result:
$$\oint_{|z|=2}\frac{e^z}{z(z-1)}\,dz = 2\pi i(-1 + e) = 2\pi i(e-1)$$

### Example 3: $\oint_{|z|=1} \frac{z^2}{(z-2)(z+3)}\,dz$

Singularities at $z = 2$ and $z = -3$. Both are **outside** $|z| = 1$.

By Cauchy–Goursat: $\oint = 0$.

### Example 4: $\oint_{|z|=1} z^3 e^{1/z}\,dz$

Essential singularity at $z = 0$. Find $a_{-1}$ of $z^3 e^{1/z}$:

$$z^3 e^{1/z} = z^3\sum_{n=0}^\infty \frac{1}{n!\,z^n} = z^3 + z^2 + \frac{z}{2} + \frac{1}{6} + \frac{1}{24z} + \cdots$$

$a_{-1} = 1/24$, so:

$$\oint_{|z|=1} z^3 e^{1/z}\,dz = 2\pi i \cdot \frac{1}{24} = \frac{\pi i}{12}$$

### Example 5: $\oint_{|z|=4} \frac{dz}{z^4+1}$

**Step 1.** Roots of $z^4 = -1 = e^{i\pi}$:

$$z_k = e^{i(\pi + 2k\pi)/4}, \quad k = 0,1,2,3$$

$$z_0 = e^{i\pi/4}, \; z_1 = e^{i3\pi/4}, \; z_2 = e^{i5\pi/4}, \; z_3 = e^{i7\pi/4}$$

All have $|z_k| = 1 < 4$, so all are inside the contour.

**Step 2.** All simple poles. Using $\text{Res} = \frac{1}{q'(z_k)} = \frac{1}{4z_k^3} = \frac{z_k}{4z_k^4} = \frac{z_k}{4(-1)} = -\frac{z_k}{4}$.

**Step 3.** Sum:

$$\sum_{k=0}^3 \text{Res}(f, z_k) = -\frac{1}{4}\sum_{k=0}^3 z_k = -\frac{1}{4} \cdot 0 = 0$$

(The roots sum to zero.) So $\oint = 0$.

---

## 4. Computational Strategy

```
Steps for evaluating ∮_C f(z) dz via Residue Theorem:

  ┌─────────────────────────────────┐
  │  1. Find all singularities      │
  │     of f in ℂ                   │
  ├─────────────────────────────────┤
  │  2. Determine which are         │
  │     INSIDE contour C            │
  ├─────────────────────────────────┤
  │  3. Classify each: pole         │
  │     (order?), essential         │
  ├─────────────────────────────────┤
  │  4. Compute Res(f, z_k) for     │
  │     each interior singularity   │
  ├─────────────────────────────────┤
  │  5. Sum: ∮ = 2πi Σ Res(f, z_k) │
  └─────────────────────────────────┘
```

---

## 5. Real-World Applications

| Application | How the Residue Theorem is Used |
|-------------|-------------------------------|
| **Inverse Laplace transform** | Contour integral → sum of residues → time-domain signal |
| **Control systems** | Stability analysis via pole residues |
| **Electrostatics** | Force/potential via residue computation |
| **Quantum field theory** | Feynman diagram evaluation |
| **Number theory** | Asymptotic formulas via contour integration |

---

## Summary Table

| Concept | Statement |
|---------|-----------|
| Residue Theorem | $\oint_C f\,dz = 2\pi i \sum \text{Res}(f, z_k)$ |
| Subsumes Cauchy–Goursat | No singularities → $\oint = 0$ |
| Subsumes CIF | One simple pole of $g/(z-z_0)$ → $2\pi i\,g(z_0)$ |
| Computation steps | Find singularities → determine inside → compute residues → sum |
| Essential singularities | Must use Laurent series for residue |
| All poles simple | Often use quotient rule: $p(z_0)/q'(z_0)$ |

---

## Quick Revision Questions

1. **State** the Residue Theorem and outline its proof.

2. **Evaluate** $\oint_{|z|=2} \frac{\sin z}{z^2}\,dz$.

3. **Evaluate** $\oint_{|z|=3} \frac{z+1}{z^2-2z}\,dz$.

4. **Show** that the Cauchy Integral Formula is a special case of the Residue Theorem.

5. **Evaluate** $\oint_{|z|=1} \frac{dz}{z^2+4}$ and explain why the answer is zero.

6. **Evaluate** $\oint_{|z|=2} \frac{e^z}{(z-1)^3}\,dz$ using the generalized residue formula.

---

[← Previous: Residues at Poles](01-residues-at-poles.md) | [Next: Evaluation of Real Integrals →](03-evaluation-of-real-integrals.md)
