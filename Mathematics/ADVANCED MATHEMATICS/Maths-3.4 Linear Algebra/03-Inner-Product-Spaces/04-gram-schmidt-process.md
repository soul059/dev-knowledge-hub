# 3.4 Gram-Schmidt Process

[← Previous: Orthogonality](03-orthogonality.md) · [Back to README](../README.md) · [Next: Orthogonal Projection →](05-orthogonal-projection.md)

---

## Chapter Overview

The **Gram-Schmidt process** is an algorithm that takes any basis and produces an **orthogonal** (or orthonormal) basis for the same subspace. It is the constructive proof that every finite-dimensional inner product space has an orthonormal basis.

---

## 1. The Idea

Given a basis $\{\mathbf{v}_1, \mathbf{v}_2, \dots, \mathbf{v}_k\}$, we build orthogonal vectors $\{\mathbf{u}_1, \mathbf{u}_2, \dots, \mathbf{u}_k\}$ one at a time by **subtracting off projections** onto the previously constructed vectors.

```
  Step 1: u₁ = v₁                  (keep first vector)
  
  Step 2: u₂ = v₂ - proj_{u₁}(v₂)  (subtract component along u₁)
  
        ▲ v₂
        │╲
        │  ╲
  proj  │    ╲ u₂ (the "residual" ⊥ u₁)
        │     ╲
  ──────┼──────╲──────▶ u₁
        │  proj_{u₁}(v₂)
  
  Step 3: u₃ = v₃ - proj_{u₁}(v₃) - proj_{u₂}(v₃)
  ...and so on.
```

---

## 2. The Algorithm

**Input:** Linearly independent set $\{\mathbf{v}_1, \mathbf{v}_2, \dots, \mathbf{v}_k\}$

**Output:** Orthogonal set $\{\mathbf{u}_1, \mathbf{u}_2, \dots, \mathbf{u}_k\}$ with $\text{span}\{\mathbf{u}_1, \dots, \mathbf{u}_j\} = \text{span}\{\mathbf{v}_1, \dots, \mathbf{v}_j\}$ at every step.

$$\mathbf{u}_1 = \mathbf{v}_1$$

$$\mathbf{u}_2 = \mathbf{v}_2 - \frac{\langle \mathbf{v}_2, \mathbf{u}_1 \rangle}{\langle \mathbf{u}_1, \mathbf{u}_1 \rangle} \mathbf{u}_1$$

$$\mathbf{u}_3 = \mathbf{v}_3 - \frac{\langle \mathbf{v}_3, \mathbf{u}_1 \rangle}{\langle \mathbf{u}_1, \mathbf{u}_1 \rangle} \mathbf{u}_1 - \frac{\langle \mathbf{v}_3, \mathbf{u}_2 \rangle}{\langle \mathbf{u}_2, \mathbf{u}_2 \rangle} \mathbf{u}_2$$

In general:

$$\boxed{\mathbf{u}_j = \mathbf{v}_j - \sum_{i=1}^{j-1} \frac{\langle \mathbf{v}_j, \mathbf{u}_i \rangle}{\langle \mathbf{u}_i, \mathbf{u}_i \rangle} \mathbf{u}_i}$$

To get an **orthonormal** basis, normalise each: $\hat{\mathbf{u}}_j = \frac{\mathbf{u}_j}{\|\mathbf{u}_j\|}$.

---

## 3. Flowchart

```
  ┌────────────────────────┐
  │ Start with {v₁,...,vₖ} │
  └───────────┬────────────┘
              │
              ▼
  ┌────────────────────────┐
  │ Set u₁ = v₁            │
  └───────────┬────────────┘
              │
              ▼
  ┌────────────────────────────────────────┐
  │ For j = 2, 3, ..., k:                 │
  │                                        │
  │   uⱼ = vⱼ - Σᵢ₌₁ʲ⁻¹ (⟨vⱼ,uᵢ⟩/⟨uᵢ,uᵢ⟩)·uᵢ │
  │                                        │
  └───────────┬────────────────────────────┘
              │
              ▼
  ┌────────────────────────────────────────┐
  │ (Optional) Normalise:                  │
  │   ûⱼ = uⱼ / ‖uⱼ‖  for each j         │
  └───────────┬────────────────────────────┘
              │
              ▼
  ┌────────────────────────┐
  │ Output: {û₁,...,ûₖ}    │
  │ (orthonormal basis)    │
  └────────────────────────┘
```

---

## 4. Worked Example

**Orthogonalise** $\mathbf{v}_1 = (1, 1, 0)$, $\mathbf{v}_2 = (1, 0, 1)$, $\mathbf{v}_3 = (0, 1, 1)$.

### Step 1

$\mathbf{u}_1 = \mathbf{v}_1 = (1, 1, 0)$

### Step 2

$\langle \mathbf{v}_2, \mathbf{u}_1 \rangle = 1 \cdot 1 + 0 \cdot 1 + 1 \cdot 0 = 1$

$\langle \mathbf{u}_1, \mathbf{u}_1 \rangle = 1 + 1 + 0 = 2$

$$\mathbf{u}_2 = (1, 0, 1) - \frac{1}{2}(1, 1, 0) = \left(\frac{1}{2}, -\frac{1}{2}, 1\right)$$

**Verify:** $\langle \mathbf{u}_1, \mathbf{u}_2 \rangle = \frac{1}{2} - \frac{1}{2} + 0 = 0$ ✓

### Step 3

$\langle \mathbf{v}_3, \mathbf{u}_1 \rangle = 0 + 1 + 0 = 1$

$\langle \mathbf{v}_3, \mathbf{u}_2 \rangle = 0 - \frac{1}{2} + 1 = \frac{1}{2}$

$\langle \mathbf{u}_2, \mathbf{u}_2 \rangle = \frac{1}{4} + \frac{1}{4} + 1 = \frac{3}{2}$

$$\mathbf{u}_3 = (0, 1, 1) - \frac{1}{2}(1, 1, 0) - \frac{1/2}{3/2}\left(\frac{1}{2}, -\frac{1}{2}, 1\right)$$

$$= (0, 1, 1) - \left(\frac{1}{2}, \frac{1}{2}, 0\right) - \frac{1}{3}\left(\frac{1}{2}, -\frac{1}{2}, 1\right)$$

$$= \left(-\frac{1}{2} - \frac{1}{6},\; \frac{1}{2} + \frac{1}{6},\; 1 - \frac{1}{3}\right) = \left(-\frac{2}{3}, \frac{2}{3}, \frac{2}{3}\right)$$

**Verify:** $\langle \mathbf{u}_1, \mathbf{u}_3 \rangle = -\frac{2}{3} + \frac{2}{3} + 0 = 0$ ✓  
$\langle \mathbf{u}_2, \mathbf{u}_3 \rangle = -\frac{1}{3} - \frac{1}{3} + \frac{2}{3} = 0$ ✓

### Normalise

$$\hat{\mathbf{u}}_1 = \frac{1}{\sqrt{2}}(1, 1, 0), \quad \hat{\mathbf{u}}_2 = \frac{1}{\sqrt{3/2}}\left(\frac{1}{2}, -\frac{1}{2}, 1\right) = \frac{1}{\sqrt{6}}(1, -1, 2)$$

$$\hat{\mathbf{u}}_3 = \frac{1}{\sqrt{4/3}}\left(-\frac{2}{3}, \frac{2}{3}, \frac{2}{3}\right) = \frac{1}{\sqrt{3}}(-1, 1, 1)$$

---

## 5. Connection to QR Factorisation

The Gram-Schmidt process directly gives the **QR factorisation** of a matrix.

If $A = [\mathbf{v}_1 \mid \mathbf{v}_2 \mid \cdots \mid \mathbf{v}_n]$ and we apply Gram-Schmidt to the columns:

$$A = QR$$

where:
- $Q = [\hat{\mathbf{u}}_1 \mid \hat{\mathbf{u}}_2 \mid \cdots \mid \hat{\mathbf{u}}_n]$ — orthonormal columns
- $R$ — upper triangular with $r_{ij} = \langle \mathbf{v}_j, \hat{\mathbf{u}}_i \rangle$

```
  A    =    Q    ·    R
  
  ┌     ┐   ┌     ┐   ┌             ┐
  │ |   │   │ |   │   │ r₁₁ r₁₂ r₁₃│
  │ v₁  │ = │ û₁  │ · │  0  r₂₂ r₂₃│
  │ |   │   │ |   │   │  0   0  r₃₃│
  └     ┘   └     ┘   └             ┘
```

---

## 6. Gram-Schmidt on Functions

**Example:** Orthogonalise $\{1, x, x^2\}$ on $[0, 1]$ with $\langle f, g \rangle = \int_0^1 f(x)g(x)\,dx$.

**Step 1:** $u_1 = 1$, $\|u_1\|^2 = \int_0^1 1\,dx = 1$.

**Step 2:** $\langle x, 1 \rangle = \int_0^1 x\,dx = \frac{1}{2}$

$u_2 = x - \frac{1/2}{1} \cdot 1 = x - \frac{1}{2}$

**Step 3:** $\langle x^2, u_1 \rangle = \int_0^1 x^2\,dx = \frac{1}{3}$

$\langle x^2, u_2 \rangle = \int_0^1 x^2(x - \frac{1}{2})\,dx = \frac{1}{4} - \frac{1}{6} = \frac{1}{12}$

$\|u_2\|^2 = \int_0^1 (x-\frac{1}{2})^2\,dx = \frac{1}{12}$

$u_3 = x^2 - \frac{1}{3} - \frac{1/12}{1/12}(x - \frac{1}{2}) = x^2 - x + \frac{1}{6}$

These are (up to scaling) the first three **Legendre polynomials** shifted to $[0,1]$!

---

## 7. Numerical Stability Note

The classical Gram-Schmidt algorithm can suffer from **numerical rounding errors** when vectors are nearly parallel. The **Modified Gram-Schmidt** (MGS) algorithm is more stable:

| Method | Approach | Stability |
|--------|----------|-----------|
| Classical GS | Subtract all projections at once | Can lose orthogonality |
| Modified GS | Subtract projections one at a time, updating after each | Much better |
| Householder reflections | Use reflections instead | Best for QR |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Purpose | Convert any basis → orthogonal/orthonormal basis |
| Formula | $\mathbf{u}_j = \mathbf{v}_j - \sum \frac{\langle \mathbf{v}_j, \mathbf{u}_i \rangle}{\langle \mathbf{u}_i, \mathbf{u}_i \rangle}\mathbf{u}_i$ |
| Normalise | $\hat{\mathbf{u}}_j = \mathbf{u}_j / \|\mathbf{u}_j\|$ |
| Span preserved | $\text{span}\{\mathbf{u}_1,\dots,\mathbf{u}_j\} = \text{span}\{\mathbf{v}_1,\dots,\mathbf{v}_j\}$ |
| QR factorisation | $A = QR$ directly from Gram-Schmidt |
| Works on functions | Same algorithm with integral inner product |

---

## Quick Revision Questions

1. Apply Gram-Schmidt to $\{(1, 0, 0), (1, 1, 0), (1, 1, 1)\}$ and normalise.

2. What is the relationship between the Gram-Schmidt process and QR factorisation?

3. Orthogonalise $\{1, x\}$ on $[-1, 1]$ with $\langle f, g \rangle = \int_{-1}^{1} fg\,dx$.

4. Why does the Gram-Schmidt process require the input vectors to be linearly independent?

5. What goes wrong numerically with classical Gram-Schmidt? What is the fix?

6. After applying Gram-Schmidt and normalising, verify that $\hat{\mathbf{u}}_i \cdot \hat{\mathbf{u}}_j = \delta_{ij}$.

---

[← Previous: Orthogonality](03-orthogonality.md) · [Back to README](../README.md) · [Next: Orthogonal Projection →](05-orthogonal-projection.md)
