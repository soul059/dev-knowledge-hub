# Chapter 8.2: Properties of Z-Transform

## Overview

The properties of the Z-Transform parallel those of the Laplace Transform but are adapted for discrete-time signals. They enable efficient computation of transforms, solution of difference equations, and analysis of discrete-time systems.

---

## 8.2.1 Property Table

| Property | Time Domain | $z$-Domain | ROC |
|----------|-------------|------------|-----|
| **Linearity** | $ax_1[n] + bx_2[n]$ | $aX_1(z) + bX_2(z)$ | At least $R_1 \cap R_2$ |
| **Time Shift** | $x[n-k]$ | $z^{-k}X(z)$ | Same $R$ (except $z=0$ or $\infty$) |
| **$z$-Domain Scaling** | $a^n x[n]$ | $X(z/a)$ | $\|a\|R$ |
| **Time Reversal** | $x[-n]$ | $X(1/z)$ | $1/R$ |
| **Convolution** | $x_1[n] * x_2[n]$ | $X_1(z) \cdot X_2(z)$ | At least $R_1 \cap R_2$ |
| **Multiplication by $n$** | $nx[n]$ | $-z\frac{dX(z)}{dz}$ | Same $R$ |
| **Accumulation** | $\sum_{k=-\infty}^{n} x[k]$ | $\frac{1}{1-z^{-1}}X(z)$ | At least $R \cap \{\|z\|>1\}$ |
| **First Difference** | $x[n] - x[n-1]$ | $(1-z^{-1})X(z)$ | At least $R \cap \{\|z\|>0\}$ |
| **Initial Value** | $x[0]$ | $\lim_{z\to\infty} X(z)$ | — |
| **Final Value** | $x[\infty]$ | $\lim_{z\to 1}(1-z^{-1})X(z)$ | — |

---

## 8.2.2 Linearity

$$\mathcal{Z}\{ax_1[n] + bx_2[n]\} = aX_1(z) + bX_2(z)$$

ROC: at least $R_1 \cap R_2$ (may be larger if poles cancel).

### Example

$$x[n] = 3(0.5)^n u[n] - 2(0.8)^n u[n]$$

$$X(z) = \frac{3z}{z-0.5} - \frac{2z}{z-0.8}, \quad \text{ROC: } |z| > 0.8$$

---

## 8.2.3 Time Shifting

$$x[n-k] \xleftrightarrow{\mathcal{Z}} z^{-k}X(z)$$

$$x[n+k] \xleftrightarrow{\mathcal{Z}} z^{k}X(z)$$

### Delay by 1

$$x[n-1] \leftrightarrow z^{-1}X(z)$$

> $z^{-1}$ is the **unit delay** operator — fundamental in digital signal processing!

```
UNIT DELAY ELEMENT:

  x[n] ──→ [ z⁻¹ ] ──→ x[n-1]
  
  In block diagrams, z⁻¹ represents one sample delay.
```

### Unilateral ZT Time Shift (with initial conditions)

**Right shift** (delay):

$$\mathcal{Z}\{x[n-1]\} = z^{-1}X(z) + x[-1]$$

$$\mathcal{Z}\{x[n-2]\} = z^{-2}X(z) + x[-2] + x[-1]z^{-1}$$

**Left shift** (advance):

$$\mathcal{Z}\{x[n+1]\} = zX(z) - zx[0]$$

$$\mathcal{Z}\{x[n+2]\} = z^2X(z) - z^2x[0] - zx[1]$$

---

## 8.2.4 $z$-Domain Scaling (Multiplication by Exponential)

$$a^n x[n] \xleftrightarrow{\mathcal{Z}} X(z/a), \quad \text{ROC: } |a| \cdot R$$

### Example

If $x[n] = u[n] \leftrightarrow z/(z-1)$, then:

$$(0.9)^n u[n] \leftrightarrow \frac{z/0.9}{z/0.9 - 1} = \frac{z}{z - 0.9}$$

This is equivalent to replacing $z$ with $z/a$ in the original transform.

---

## 8.2.5 Convolution Property

$$x_1[n] * x_2[n] \xleftrightarrow{\mathcal{Z}} X_1(z) \cdot X_2(z)$$

This is the most important property for system analysis:

$$Y(z) = H(z) \cdot X(z)$$

```
CONVOLUTION → MULTIPLICATION:

  x[n] ──→ [ h[n] ] ──→ y[n] = x[n]*h[n]
  
  X(z) ──→ [ H(z) ] ──→ Y(z) = X(z)·H(z)
```

### Example

$x[n] = (0.5)^n u[n]$, $h[n] = (0.8)^n u[n]$

$$Y(z) = \frac{z}{z-0.5} \cdot \frac{z}{z-0.8} = \frac{z^2}{(z-0.5)(z-0.8)}$$

PFE (divide by $z$ first):

$$\frac{Y(z)}{z} = \frac{z}{(z-0.5)(z-0.8)} = \frac{-5/3}{z-0.5} + \frac{8/3}{z-0.8}$$

Wait — let me redo with the correct PFE approach:

$$\frac{Y(z)}{z} = \frac{z}{(z-0.5)(z-0.8)}$$

Using cover-up: $A = 0.5/(0.5-0.8) = 0.5/(-0.3) = -5/3$, $B = 0.8/(0.8-0.5) = 0.8/0.3 = 8/3$

$$Y(z) = \frac{-5z/3}{z-0.5} + \frac{8z/3}{z-0.8}$$

$$y[n] = \left(-\frac{5}{3}(0.5)^n + \frac{8}{3}(0.8)^n\right)u[n]$$

---

## 8.2.6 Multiplication by $n$ (Differentiation in $z$)

$$nx[n] \xleftrightarrow{\mathcal{Z}} -z\frac{dX(z)}{dz}$$

### Example

$$n \cdot a^n u[n]: \quad -z\frac{d}{dz}\left(\frac{z}{z-a}\right) = -z \cdot \frac{-a}{(z-a)^2} = \frac{az}{(z-a)^2}$$

---

## 8.2.7 Accumulation (Running Sum)

$$\sum_{k=-\infty}^{n} x[k] \xleftrightarrow{\mathcal{Z}} \frac{X(z)}{1 - z^{-1}} = \frac{z}{z-1} X(z)$$

This is the discrete-time analog of integration.

### Example

Accumulation of $\delta[n]$:

$$\sum_{k=-\infty}^{n} \delta[k] = u[n] \quad \leftrightarrow \quad \frac{1}{1-z^{-1}} = \frac{z}{z-1} \quad ✓$$

---

## 8.2.8 Initial and Final Value Theorems

### Initial Value Theorem

$$x[0] = \lim_{z \to \infty} X(z)$$

**Condition**: $x[n]$ is causal.

### Final Value Theorem

$$\lim_{n \to \infty} x[n] = \lim_{z \to 1} (1 - z^{-1})X(z) = \lim_{z \to 1} \frac{(z-1)X(z)}{z}$$

**Condition**: $(1-z^{-1})X(z)$ must have all poles inside the unit circle ($x[n]$ must converge).

### Example

$$X(z) = \frac{3z}{(z-0.5)(z-0.8)}, \quad \text{causal}$$

**Initial value**:

$$x[0] = \lim_{z\to\infty} \frac{3z}{(z-0.5)(z-0.8)} = \lim_{z\to\infty} \frac{3}{z-1.3+0.4/z} = 0$$

Actually: $\lim_{z\to\infty} \frac{3z}{z^2(1-0.5/z)(1-0.8/z)} = \lim_{z\to\infty} \frac{3}{z} = 0$.

**Final value**:

$$x[\infty] = \lim_{z\to 1} (1-z^{-1})\frac{3z}{(z-0.5)(z-0.8)} = \lim_{z\to 1} \frac{3(z-1)}{z} \cdot \frac{z}{(z-0.5)(z-0.8)}$$

Wait — let's simplify:

$$\lim_{z\to 1} \frac{(z-1) \cdot 3z}{z \cdot (z-0.5)(z-0.8)} = \frac{0 \cdot 3}{0.5 \cdot 0.2} = 0$$

Hmm, the $(z-1)$ in numerator → the limit is 0.

---

## 8.2.9 Solving Difference Equations

### Procedure

```
SOLVING DIFFERENCE EQUATIONS WITH ZT:

1. Take unilateral ZT of both sides
   (use time-shift with initial conditions)

2. Solve algebraically for Y(z)

3. Partial fraction expansion
   (divide by z first, then PFE, then multiply back)

4. Inverse Z-Transform → y[n]
```

### Example

$$y[n] - 0.5y[n-1] = x[n], \quad x[n] = u[n], \; y[-1] = 2$$

**Step 1**: ZT

$$Y(z) - 0.5(z^{-1}Y(z) + y[-1]) = \frac{z}{z-1}$$

$$Y(z)(1 - 0.5z^{-1}) = \frac{z}{z-1} + 0.5 \cdot 2$$

$$Y(z) \cdot \frac{z-0.5}{z} = \frac{z}{z-1} + 1$$

$$Y(z) = \frac{z}{(z-0.5)} \cdot \left(\frac{z}{z-1} + 1\right) = \frac{z}{(z-0.5)} \cdot \frac{2z-1}{z-1}$$

$$Y(z) = \frac{z(2z-1)}{(z-0.5)(z-1)}$$

**Step 2**: PFE (divide by $z$):

$$\frac{Y(z)}{z} = \frac{2z-1}{(z-0.5)(z-1)}$$

$A = \frac{2(0.5)-1}{0.5-1} = \frac{0}{-0.5} = 0$, $B = \frac{2(1)-1}{1-0.5} = \frac{1}{0.5} = 2$

$$Y(z) = \frac{2z}{z-1} \quad \Rightarrow \quad y[n] = 2u[n]$$

**Verify**: $y[0] = 0.5 \cdot 2 + 1 = 2$ ✓, $y[1] = 0.5 \cdot 2 + 1 = 2$ ✓

---

## Summary Table

| Property | Transform | ROC Effect |
|----------|-----------|------------|
| Linearity | $aX_1 + bX_2$ | $R_1 \cap R_2$ (at least) |
| Time shift by $k$ | $z^{-k}X(z)$ | Same (except $z=0/\infty$) |
| $z$-scaling ($a^n$) | $X(z/a)$ | Scaled by $\|a\|$ |
| Convolution | $X_1(z)X_2(z)$ | $R_1 \cap R_2$ |
| Multiply by $n$ | $-z \cdot dX/dz$ | Same ROC |
| Accumulation | $X(z)/(1-z^{-1})$ | $R \cap \{\|z\|>1\}$ |
| Initial value | $\lim_{z\to\infty} X(z)$ | Causal only |
| Final value | $\lim_{z\to 1} (1-z^{-1})X(z)$ | Stable convergence |

---

## Quick Revision Questions

1. **What does $z^{-1}$ represent in the time domain?**
   - A unit delay: $z^{-1}X(z) \leftrightarrow x[n-1]$.

2. **Find $\mathcal{Z}\{n(0.5)^n u[n]\}$.**
   - $-z\frac{d}{dz}\frac{z}{z-0.5} = \frac{0.5z}{(z-0.5)^2}$, ROC: $|z| > 0.5$.

3. **How do you handle initial conditions in the ZT?**
   - Use unilateral ZT shift property: $\mathcal{Z}\{x[n-1]\} = z^{-1}X(z) + x[-1]$.

4. **What is the ZT of the accumulation of $x[n] = (0.5)^n u[n]$?**
   - $\frac{z}{z-1} \cdot \frac{z}{z-0.5} = \frac{z^2}{(z-1)(z-0.5)}$, ROC: $|z| > 1$.

5. **Apply the final value theorem to $X(z) = \frac{z}{(z-1)(z-0.5)}$.**
   - $x[\infty] = \lim_{z\to 1}(z-1)\frac{z}{z(z-1)(z-0.5)} = \lim_{z\to 1}\frac{1}{z-0.5} = 2$. But $(1-z^{-1})X(z) = \frac{1}{z-0.5}$ has pole at $z=0.5$ (inside UC ✓).

---

[← Previous: Definition and ROC](01-definition-and-roc.md) | [Next: Inverse Z-Transform →](03-inverse-z-transform.md)
