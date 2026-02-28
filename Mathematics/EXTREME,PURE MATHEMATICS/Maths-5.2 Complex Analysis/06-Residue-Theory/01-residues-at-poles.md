# 6.1 Residues at Poles

[← Previous: Zeros of Analytic Functions](../05-Series-Representations/04-zeros-of-analytic-functions.md) | [Next: The Residue Theorem →](02-residue-theorem.md)

---

## Chapter Overview

The **residue** of a function at an isolated singularity is the coefficient $a_{-1}$ in its Laurent series. It captures the *only* contribution to the contour integral. This chapter develops efficient techniques for computing residues, especially at poles, without needing to find the full Laurent series.

---

## 1. Definition of Residue

### 1.1 Via Laurent Series

If $f$ has an isolated singularity at $z_0$, the **residue** of $f$ at $z_0$ is:

$$\text{Res}(f, z_0) = a_{-1} = \frac{1}{2\pi i}\oint_C f(z)\,dz$$

where $C$ is any positively oriented simple closed curve enclosing $z_0$ and no other singularity.

### 1.2 Significance

$$\boxed{\oint_C f(z)\,dz = 2\pi i \cdot \text{Res}(f, z_0)}$$

if $C$ encloses only $z_0$.

---

## 2. Residue Formulas for Poles

### 2.1 Simple Pole ($m = 1$)

$$\boxed{\text{Res}(f, z_0) = \lim_{z \to z_0} (z - z_0)f(z)}$$

### 2.2 Pole of Order $m$

$$\boxed{\text{Res}(f, z_0) = \frac{1}{(m-1)!}\lim_{z \to z_0} \frac{d^{m-1}}{dz^{m-1}}\left[(z-z_0)^m f(z)\right]}$$

### 2.3 Quotient Rule (for simple poles)

If $f(z) = \frac{p(z)}{q(z)}$ where $p(z_0) \neq 0$, $q(z_0) = 0$, $q'(z_0) \neq 0$ (simple zero of $q$):

$$\boxed{\text{Res}\left(\frac{p}{q}, z_0\right) = \frac{p(z_0)}{q'(z_0)}}$$

---

## 3. Summary of Residue Computation Methods

| Situation | Method | Formula |
|-----------|--------|---------|
| Simple pole | Limit | $\lim_{z\to z_0}(z-z_0)f(z)$ |
| Simple pole of $p/q$ | Quotient rule | $p(z_0)/q'(z_0)$ |
| Pole of order $m$ | Derivative formula | $\frac{1}{(m-1)!}\frac{d^{m-1}}{dz^{m-1}}[(z-z_0)^m f(z)]$ |
| Any isolated singularity | Laurent series | Read off $a_{-1}$ |
| Essential singularity | Laurent series only | Expand and identify $a_{-1}$ |

```
Decision tree for computing residues:

           What type of singularity?
                    │
        ┌───────────┼───────────┐
        │           │           │
     Simple       Pole of     Essential
      pole       order m     singularity
        │           │           │
        ▼           ▼           ▼
   Use limit    Derivative   Find Laurent
   formula or    formula      series; read
   quotient                   off a_{-1}
    rule
```

---

## 4. Design Examples

### Example 1: $\text{Res}\left(\frac{e^z}{z^2+1}, i\right)$

$z^2 + 1 = (z-i)(z+i)$, so $z = i$ is a simple pole.

**Method 1 (Limit):**
$$\text{Res} = \lim_{z \to i}(z-i)\frac{e^z}{(z-i)(z+i)} = \frac{e^i}{2i}$$

**Method 2 (Quotient rule):** $p(z) = e^z$, $q(z) = z^2+1$, $q'(z) = 2z$:
$$\text{Res} = \frac{e^i}{2i} = \frac{\cos 1 + i\sin 1}{2i} = \frac{\sin 1 - i\cos 1}{2}$$

### Example 2: $\text{Res}\left(\frac{z}{(z-1)^3}, 1\right)$

Pole of order 3 at $z = 1$. Apply the formula with $m = 3$:

$$\text{Res} = \frac{1}{2!}\lim_{z\to 1}\frac{d^2}{dz^2}\left[(z-1)^3 \cdot \frac{z}{(z-1)^3}\right] = \frac{1}{2}\lim_{z\to 1}\frac{d^2}{dz^2}[z] = \frac{1}{2}\cdot 0 = 0$$

### Example 3: $\text{Res}\left(\frac{\cos z}{z^2}, 0\right)$

Pole of order 2 at $z = 0$. With $m = 2$:

$$\text{Res} = \frac{1}{1!}\lim_{z\to 0}\frac{d}{dz}\left[z^2 \cdot \frac{\cos z}{z^2}\right] = \lim_{z\to 0}\frac{d}{dz}[\cos z] = \lim_{z\to 0}(-\sin z) = 0$$

**Alternative (Laurent series):**
$$\frac{\cos z}{z^2} = \frac{1 - z^2/2 + z^4/24 - \cdots}{z^2} = \frac{1}{z^2} - \frac{1}{2} + \frac{z^2}{24} - \cdots$$

Coefficient of $1/z$ is $0$ → Residue $= 0$ ✓

### Example 4: $\text{Res}(e^{1/z}, 0)$ — Essential singularity

$$e^{1/z} = 1 + \frac{1}{z} + \frac{1}{2z^2} + \frac{1}{6z^3} + \cdots$$

$a_{-1} = 1$, so $\text{Res}(e^{1/z}, 0) = 1$.

### Example 5: $\text{Res}\left(\frac{1}{z^2\sin z}, 0\right)$

First determine the order. $\sin z = z - z^3/6 + \cdots$:

$$\frac{1}{z^2 \sin z} = \frac{1}{z^2 \cdot z(1 - z^2/6 + \cdots)} = \frac{1}{z^3}\cdot\frac{1}{1 - z^2/6 + \cdots}$$

$$= \frac{1}{z^3}\left(1 + \frac{z^2}{6} + \cdots\right) = \frac{1}{z^3} + \frac{1}{6z} + \cdots$$

Residue $= a_{-1} = 1/6$.

---

## 5. Residue at Infinity

If $f$ is analytic in $|z| > R$, the residue at infinity is:

$$\text{Res}(f, \infty) = -\frac{1}{2\pi i}\oint_{|z|=R'} f(z)\,dz = -a_{-1}$$

where the Laurent series is about $z = \infty$, i.e., $f(z) = \sum a_n z^n$ for large $|z|$.

**Alternatively:**

$$\text{Res}(f, \infty) = -\text{Res}\left(\frac{1}{z^2}f\left(\frac{1}{z}\right), 0\right)$$

**Key fact:** The sum of all residues (including infinity) is zero:

$$\sum_{\text{all } z_k} \text{Res}(f, z_k) + \text{Res}(f, \infty) = 0$$

---

## 6. Real-World Applications

| Application | Residue Role |
|-------------|-------------|
| **Circuit analysis** | Residues at poles → partial fraction → time-domain response |
| **Control theory** | Residues of transfer function ↔ modal contributions |
| **Particle physics** | Residues of propagators → particle masses and couplings |
| **Statistical mechanics** | Residues in partition function analysis |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| Residue definition | $\text{Res}(f, z_0) = a_{-1}$ in Laurent series |
| Simple pole | $\lim_{z\to z_0}(z-z_0)f(z)$ |
| Quotient form | $p(z_0)/q'(z_0)$ |
| Pole of order $m$ | $\frac{1}{(m-1)!}\frac{d^{m-1}}{dz^{m-1}}[(z-z_0)^m f(z)]\Big|_{z=z_0}$ |
| Essential singularity | Laurent series only |
| Residue at $\infty$ | $-\text{Res}(z^{-2}f(1/z), 0)$ |
| Sum of all residues | $= 0$ (including residue at $\infty$) |

---

## Quick Revision Questions

1. **Define** the residue of $f$ at $z_0$ and state its connection to the contour integral.

2. **Compute** $\text{Res}\left(\frac{z^2}{z^2+4}, 2i\right)$.

3. **Derive** the quotient rule $\text{Res}(p/q, z_0) = p(z_0)/q'(z_0)$ from the simple pole formula.

4. **Find** $\text{Res}\left(\frac{1}{z^2(z-1)^2}, 0\right)$ and $\text{Res}\left(\frac{1}{z^2(z-1)^2}, 1\right)$.

5. **Compute** $\text{Res}(z^2 e^{1/z}, 0)$ using the Laurent series.

6. **Verify** that the sum of all residues (including at $\infty$) of $\frac{1}{z(z-1)}$ is zero.

---

[← Previous: Zeros of Analytic Functions](../05-Series-Representations/04-zeros-of-analytic-functions.md) | [Next: The Residue Theorem →](02-residue-theorem.md)
