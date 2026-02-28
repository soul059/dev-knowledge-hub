# Chapter 8.3: Inverse Z-Transform

## Overview

The Inverse Z-Transform recovers the discrete-time sequence $x[n]$ from $X(z)$. Practical methods include **partial fraction expansion**, **power series (long division)**, and the **residue method**. The ROC determines whether terms are right-sided or left-sided.

---

## 8.3.1 Formal Definition

$$x[n] = \frac{1}{2\pi j} \oint_C X(z)\,z^{n-1}\,dz$$

where $C$ is a counterclockwise contour within the ROC. In practice, we use algebraic methods.

---

## 8.3.2 Method 1: Partial Fraction Expansion (PFE)

### Key Technique: Divide by $z$ First

For rational $X(z)$:

```
PFE PROCEDURE FOR INVERSE ZT:

1. Form X(z)/z

2. Decompose X(z)/z into partial fractions

3. Multiply each term by z to get X(z) terms

4. Use ROC to assign each term as:
   ├─ Right-sided: a^n·u[n]  (ROC outside pole)
   └─ Left-sided: -a^n·u[-n-1]  (ROC inside pole)

5. Sum all terms → x[n]
```

> **Why divide by $z$?** The ZT pairs are in the form $z/(z-a)$, not $1/(z-a)$. Dividing by $z$ first gives standard PFE form, then multiplying back yields recognizable pairs.

---

### Example 1: Distinct Poles

$$X(z) = \frac{2z^2 - 1.5z}{z^2 - 1.5z + 0.5}, \quad \text{ROC: } |z| > 1$$

**Step 1**: Check degree. Numerator degree = denominator degree = 2, so do long division:

$$X(z) = 2 + \frac{1.5z - 1}{z^2 - 1.5z + 0.5}$$

Actually, let's factor: $z^2 - 1.5z + 0.5 = (z-1)(z-0.5)$

$$\frac{X(z)}{z} = \frac{2z - 1.5}{(z-1)(z-0.5)}$$

Cover-up: $A = \frac{2-1.5}{1-0.5} = \frac{0.5}{0.5} = 1$, $B = \frac{1-1.5}{0.5-1} = \frac{-0.5}{-0.5} = 1$

$$\frac{X(z)}{z} = \frac{1}{z-1} + \frac{1}{z-0.5}$$

$$X(z) = \frac{z}{z-1} + \frac{z}{z-0.5}$$

ROC $|z| > 1$ → both terms right-sided:

$$x[n] = u[n] + (0.5)^n u[n] = (1 + (0.5)^n)u[n]$$

---

### Example 2: Repeated Poles

$$X(z) = \frac{z^2}{(z-0.5)^2}, \quad \text{ROC: } |z| > 0.5$$

$$\frac{X(z)}{z} = \frac{z}{(z-0.5)^2}$$

PFE for repeated poles:

$$\frac{z}{(z-0.5)^2} = \frac{A}{z-0.5} + \frac{B}{(z-0.5)^2}$$

$B = z\big|_{z=0.5} = 0.5$

$A = \frac{d}{dz}[z]\big|_{z=0.5} = 1$

$$X(z) = \frac{z}{z-0.5} + \frac{0.5z}{(z-0.5)^2}$$

$$x[n] = (0.5)^n u[n] + n(0.5)^n u[n] = (1+n)(0.5)^n u[n]$$

---

### Example 3: Two-Sided Signal

$$X(z) = \frac{z}{(z-2)(z-0.5)}, \quad \text{ROC: } 0.5 < |z| < 2$$

$$\frac{X(z)}{z} = \frac{1}{(z-2)(z-0.5)}$$

$A = \frac{1}{2-0.5} = \frac{2}{3}$, $B = \frac{1}{0.5-2} = -\frac{2}{3}$

$$X(z) = \frac{2z/3}{z-2} + \frac{-2z/3}{z-0.5}$$

ROC: $0.5 < |z| < 2$:
- Pole at $z = 2$: ROC is **inside** → left-sided: $-\frac{2}{3}(2)^n u[-n-1]$
- Pole at $z = 0.5$: ROC is **outside** → right-sided: $-\frac{2}{3}(0.5)^n u[n]$

$$x[n] = -\frac{2}{3}(2)^n u[-n-1] - \frac{2}{3}(0.5)^n u[n]$$

---

## 8.3.3 Method 2: Power Series (Long Division)

Since $X(z) = \sum x[n]z^{-n}$, the coefficients of $z^{-n}$ directly give $x[n]$.

### For Causal Signals: Divide in powers of $z^{-1}$

### Example

$$X(z) = \frac{1}{1 - 1.5z^{-1} + 0.5z^{-2}}, \quad \text{ROC: } |z| > 1$$

Perform long division:

```
                1 + 1.5z⁻¹ + 1.75z⁻² + ...
              ─────────────────────────────────
1-1.5z⁻¹+0.5z⁻² ) 1
                    1 - 1.5z⁻¹ + 0.5z⁻²
                    ─────────────────────
                        1.5z⁻¹ - 0.5z⁻²
                        1.5z⁻¹ - 2.25z⁻² + 0.75z⁻³
                        ──────────────────────────
                                1.75z⁻² - 0.75z⁻³
                                ...
```

$$X(z) = 1 + 1.5z^{-1} + 1.75z^{-2} + \cdots$$

$$x[0] = 1, \; x[1] = 1.5, \; x[2] = 1.75, \ldots$$

### For Anti-Causal Signals: Divide in powers of $z$

Rearrange and divide by ascending powers of $z$.

---

## 8.3.4 Method 3: Residue Method (Contour Integration)

$$x[n] = \sum_{\text{poles of } X(z)z^{n-1}} \text{Res}\left[X(z)z^{n-1}\right]$$

For simple poles at $z = z_k$:

$$\text{Res}_{z=z_k} = (z - z_k)X(z)z^{n-1}\Big|_{z=z_k}$$

### Example

$$X(z) = \frac{z}{(z-0.5)(z-0.8)}, \quad \text{ROC: } |z| > 0.8$$

$$x[n] = \text{Res}_{z=0.5}\left[\frac{z^n}{(z-0.5)(z-0.8)}\right] + \text{Res}_{z=0.8}\left[\frac{z^n}{(z-0.5)(z-0.8)}\right]$$

$$= \frac{(0.5)^n}{0.5-0.8} + \frac{(0.8)^n}{0.8-0.5} = \frac{-(0.5)^n}{0.3} + \frac{(0.8)^n}{0.3}$$

Wait — recompute: $X(z)z^{n-1} = \frac{z^n}{(z-0.5)(z-0.8)}$

Res at $z=0.5$: $(z-0.5)\frac{z^n}{(z-0.5)(z-0.8)}\big|_{z=0.5} = \frac{(0.5)^n}{0.5-0.8} = \frac{(0.5)^n}{-0.3}$

Res at $z=0.8$: $\frac{(0.8)^n}{0.8-0.5} = \frac{(0.8)^n}{0.3}$

$$x[n] = \frac{1}{0.3}\left[-(0.5)^n + (0.8)^n\right]u[n] = \frac{10}{3}\left[(0.8)^n - (0.5)^n\right]u[n]$$

---

## 8.3.5 Comparison of Methods

| Method | Best For | Limitations |
|--------|----------|-------------|
| **Partial Fractions** | Rational $X(z)$ with known poles | Requires factoring $D(z)$ |
| **Power Series** | Non-rational $X(z)$, first few values | Only yields individual samples |
| **Residue** | Quick for simple poles | Complex for repeated/complex poles |

```
CHOOSING THE RIGHT METHOD:

Is X(z) rational?
├─ NO → Power series (long division)
└─ YES → Can you factor D(z)?
         ├─ YES → PFE (most common)
         └─ NO → Power series or numerical methods

Need only first N samples?
└─ YES → Power series is fastest
```

---

## 8.3.6 Handling Improper $X(z)$

If degree of numerator ≥ degree of denominator, perform long division first.

### Example

$$X(z) = \frac{z^2 + z}{z - 0.5} = z + 1.5 + \frac{0.75}{z-0.5}$$

Wait — let me redo: $\frac{X(z)}{z} = \frac{z+1}{z-0.5} = 1 + \frac{1.5}{z-0.5}$

$$X(z) = z + \frac{1.5z}{z-0.5}$$

$$x[n] = \delta[n+1] + 1.5(0.5)^n u[n]$$

---

## Summary Table

| Method | Steps | Result Form |
|--------|-------|-------------|
| PFE | Divide by $z$ → PFE → multiply by $z$ → use table | Closed-form $x[n]$ |
| Power series | Long division in $z^{-1}$ | Individual samples |
| Residue | $\sum \text{Res}[X(z)z^{n-1}]$ | Closed-form $x[n]$ |
| ROC: pole outside | Left-sided term | $-a^n u[-n-1]$ |
| ROC: pole inside | Right-sided term | $a^n u[n]$ |

---

## Quick Revision Questions

1. **Why do we divide by $z$ before doing PFE?**
   - To get fractions of the form $1/(z-a)$ which, when multiplied back by $z$, give $z/(z-a)$ — the standard ZT pair form.

2. **Find $x[n]$ for $X(z) = 3z/(z-0.25)$, ROC: $|z| > 0.25$.**
   - Right-sided: $x[n] = 3(0.25)^n u[n]$.

3. **When is the power series method preferred?**
   - When $X(z)$ is not rational, when you need only the first few samples, or when factoring the denominator is difficult.

4. **Find $x[0]$ and $x[1]$ for $X(z) = z/(z^2-z+0.5)$ by long division.**
   - Divide: $X(z) = z^{-1}/(1-z^{-1}+0.5z^{-2})$. First terms: $0 \cdot z^0 + 1 \cdot z^{-1} + \ldots$ So $x[0] = 0$, $x[1] = 1$.

5. **For a two-sided ROC, how do you assign signal types?**
   - Poles outside the ROC → left-sided terms ($-a^nu[-n-1]$). Poles inside the ROC → right-sided terms ($a^nu[n]$).

---

[← Previous: Properties](02-properties.md) | [Next: Discrete-Time Transfer Function →](04-transfer-function.md)
