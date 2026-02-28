# 5.2 Laurent Series

[← Previous: Taylor Series](01-taylor-series.md) | [Next: Classification of Singularities →](03-classification-of-singularities.md)

---

## Chapter Overview

When a function has a singularity at $z_0$, its Taylor series about $z_0$ does not exist. The **Laurent series** extends the Taylor series by allowing **negative powers** of $(z - z_0)$, giving a representation valid in an *annulus* around the singularity. The Laurent series is the bridge to residue theory (Unit 6).

---

## 1. Laurent's Theorem

### 1.1 Statement

> **Theorem.** If $f$ is analytic in the annulus $r < |z - z_0| < R$ (where $0 \leq r < R \leq \infty$), then:
> $$\boxed{f(z) = \sum_{n=-\infty}^{\infty} a_n(z - z_0)^n = \underbrace{\sum_{n=0}^{\infty} a_n(z-z_0)^n}_{\text{analytic part}} + \underbrace{\sum_{n=1}^{\infty} a_{-n}(z-z_0)^{-n}}_{\text{principal part}}}$$

### 1.2 Coefficient Formula

$$a_n = \frac{1}{2\pi i}\oint_C \frac{f(z)}{(z - z_0)^{n+1}}\,dz, \quad n \in \mathbb{Z}$$

where $C$ is any positively oriented circle $|z - z_0| = \rho$ with $r < \rho < R$.

```
Annulus of convergence:

         ╭────── R ──────╮
        ╱   ╭── r ──╮     ╲
       ╱   ╱         ╲     ╲
      │   │   sing.   │     │
      │   │    at z₀  │     │    f analytic in the annulus
      │   │     ×     │     │    r < |z - z₀| < R
       ╲   ╲         ╱     ╱
        ╲   ╰───────╯     ╱
         ╰────────────────╯

  Taylor part converges for |z - z₀| < R
  Principal part converges for |z - z₀| > r
  Both converge in the annulus
```

---

## 2. Structure of the Laurent Series

| Part | Terms | Convergence |
|------|-------|-------------|
| **Analytic part** | $\sum_{n=0}^{\infty} a_n(z-z_0)^n$ | $|z-z_0| < R$ |
| **Principal part** | $\sum_{n=1}^{\infty} a_{-n}(z-z_0)^{-n}$ | $|z-z_0| > r$ |
| **Full series** | Both combined | Annulus $r < |z-z_0| < R$ |

### 2.1 Special Cases

| Condition | Consequence |
|-----------|------------|
| No principal part ($a_{-n} = 0$ for all $n \geq 1$) | $f$ is analytic at $z_0$ (Taylor series) |
| Finite principal part ($a_{-n} = 0$ for $n > N$) | **Pole** of order $N$ at $z_0$ |
| Infinite principal part | **Essential singularity** at $z_0$ |

---

## 3. Uniqueness

The Laurent series representation in a given annulus is **unique**. This means:
- You can find Laurent coefficients by *any* valid method (partial fractions, substitution, known series, etc.).
- If you obtain a series $\sum c_n(z-z_0)^n$ valid in an annulus, then $c_n = a_n$.

---

## 4. Finding Laurent Series in Practice

### 4.1 Different Annuli, Different Series

A function can have **different** Laurent series in different annuli around the same point!

**Example:** $f(z) = \frac{1}{z(1-z)}$, $z_0 = 0$.

**Annulus 1: $0 < |z| < 1$** (between singularities at $z = 0$ and $z = 1$):

$$\frac{1}{z(1-z)} = \frac{1}{z}\cdot\frac{1}{1-z} = \frac{1}{z}\sum_{n=0}^\infty z^n = \sum_{n=0}^\infty z^{n-1} = \frac{1}{z} + 1 + z + z^2 + \cdots$$

**Annulus 2: $|z| > 1$** (outside both singularities):

$$\frac{1}{z(1-z)} = -\frac{1}{z^2}\cdot\frac{1}{1-1/z} = -\frac{1}{z^2}\sum_{n=0}^\infty \frac{1}{z^n} = -\sum_{n=0}^\infty \frac{1}{z^{n+2}} = -\frac{1}{z^2} - \frac{1}{z^3} - \cdots$$

```
Two different Laurent series for the same function:

  ──────•═══════════•──────────────────────► Re
        0    Annulus 1    1    Annulus 2
             0<|z|<1           |z|>1
```

### 4.2 Strategy Summary

| Technique | When to Use |
|-----------|------------|
| Partial fractions + geometric series | Rational functions |
| Substitution into known series | $e^{1/z}$, $\sin(1/z)$, etc. |
| Long division | When leading behavior is clear |
| Multiply/divide known series | Products of simple functions |
| Coefficient formula (integral) | Theoretical or complicated cases |

---

## 5. The Residue (Preview)

The coefficient $a_{-1}$ in the Laurent series has special significance:

$$a_{-1} = \frac{1}{2\pi i}\oint_C f(z)\,dz$$

This is the **residue** of $f$ at $z_0$:

$$\text{Res}(f, z_0) = a_{-1}$$

It is the *only* Laurent coefficient that contributes to the contour integral!

---

## 6. Design Examples

### Example 1: Laurent series of $e^{1/z}$ about $z_0 = 0$.

Substitute $w = 1/z$ into $e^w = \sum_{n=0}^\infty w^n/n!$:

$$e^{1/z} = \sum_{n=0}^{\infty} \frac{1}{n!\,z^n} = 1 + \frac{1}{z} + \frac{1}{2z^2} + \frac{1}{6z^3} + \cdots$$

Valid for $|z| > 0$ (i.e., all $z \neq 0$). The principal part is **infinite** → essential singularity at $z = 0$.

Residue: $a_{-1} = 1$.

### Example 2: Find the Laurent series of $\frac{1}{(z-1)(z-2)}$ in the annulus $1 < |z| < 2$.

**Step 1.** Partial fractions:

$$\frac{1}{(z-1)(z-2)} = \frac{-1}{z-1} + \frac{1}{z-2}$$

**Step 2.** For $|z| > 1$: $\frac{-1}{z-1} = \frac{-1}{z}\cdot\frac{1}{1 - 1/z} = -\sum_{n=0}^\infty \frac{1}{z^{n+1}}$

**Step 3.** For $|z| < 2$: $\frac{1}{z-2} = \frac{-1}{2}\cdot\frac{1}{1-z/2} = -\sum_{n=0}^\infty \frac{z^n}{2^{n+1}}$

**Step 4.** Combine:

$$f(z) = -\sum_{n=0}^\infty \frac{1}{z^{n+1}} - \sum_{n=0}^\infty \frac{z^n}{2^{n+1}}$$

$$= \cdots - \frac{1}{z^2} - \frac{1}{z} - \frac{1}{2} - \frac{z}{4} - \frac{z^2}{8} - \cdots$$

### Example 3: Laurent series of $\frac{\sin z}{z^3}$ about $z = 0$.

$$\frac{\sin z}{z^3} = \frac{1}{z^3}\left(z - \frac{z^3}{6} + \frac{z^5}{120} - \cdots\right) = \frac{1}{z^2} - \frac{1}{6} + \frac{z^2}{120} - \cdots$$

Principal part: $\frac{1}{z^2}$ (finite). Pole of order 2. Residue $a_{-1} = 0$.

---

## 7. Real-World Applications

| Application | Laurent Series Role |
|-------------|-------------------|
| **Signal processing** | $z$-transform: Laurent series in annulus of convergence |
| **Antenna theory** | Near-field vs. far-field expansions = different annuli |
| **Electrostatics** | Multipole expansion = Laurent series of potential |
| **Fluid dynamics** | Far-field velocity as Laurent series |

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| Laurent series | $f(z) = \sum_{n=-\infty}^{\infty} a_n(z-z_0)^n$ |
| Valid in | Annulus $r < |z-z_0| < R$ |
| Coefficient formula | $a_n = \frac{1}{2\pi i}\oint_C \frac{f(z)}{(z-z_0)^{n+1}}\,dz$ |
| Uniqueness | Series is unique in a given annulus |
| Different annuli | Same function may have different Laurent series |
| Analytic part | Non-negative powers |
| Principal part | Negative powers; determines singularity type |
| Residue | $a_{-1}$ = coefficient of $(z-z_0)^{-1}$ |

---

## Quick Revision Questions

1. **State** Laurent's theorem, specifying the domain of convergence and the coefficient formula.

2. **Find** the Laurent series of $\frac{1}{z^2(1+z)}$ about $z = 0$ for $0 < |z| < 1$.

3. **Why** can the same function have different Laurent series about the same point?

4. **Find** the Laurent series of $e^{1/z}$ and identify the type of singularity at $z = 0$.

5. **Compute** the residue of $\frac{\cos z}{z^2}$ at $z = 0$ from its Laurent series.

6. **Explain** why the residue $a_{-1}$ is the only Laurent coefficient that contributes to $\oint f\,dz$.

---

[← Previous: Taylor Series](01-taylor-series.md) | [Next: Classification of Singularities →](03-classification-of-singularities.md)
