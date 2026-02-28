# Chapter 6.2: Variation of Parameters

[← Previous: Undetermined Coefficients](01-undetermined-coefficients.md) | [Back to README](../README.md) | [Next: D-Operator Methods →](03-d-operator-methods.md)

---

## Overview

Variation of parameters is a **general method** for finding particular solutions of non-homogeneous linear ODEs. Unlike undetermined coefficients, it works for **any** continuous $g(x)$ — not just polynomials, exponentials, and trigonometric functions.

---

## 1. The Idea

Replace the constants in $y_c = c_1 y_1 + c_2 y_2$ with **functions**: $y_p = u_1(x)y_1 + u_2(x)y_2$.

The "parameters" $c_1, c_2$ are "varied" — hence the name.

```
CONSTANT → FUNCTION

  y_c = c₁·y₁ + c₂·y₂     (homogeneous solution)
         │        │
         ▼        ▼
  y_p = u₁(x)·y₁ + u₂(x)·y₂   (particular solution)

  The constants "vary" to accommodate g(x)!
```

---

## 2. Derivation (Second-Order Case)

For: $y'' + P(x)y' + Q(x)y = g(x)$

Assume $y_p = u_1 y_1 + u_2 y_2$

$y_p' = u_1'y_1 + u_1 y_1' + u_2'y_2 + u_2 y_2'$

**Impose condition 1**: $u_1'y_1 + u_2'y_2 = 0$ (simplifies computation)

Then: $y_p' = u_1 y_1' + u_2 y_2'$

$y_p'' = u_1'y_1' + u_1 y_1'' + u_2'y_2' + u_2 y_2''$

Substituting into the ODE:

$$u_1\underbrace{(y_1'' + Py_1' + Qy_1)}_{=0} + u_2\underbrace{(y_2'' + Py_2' + Qy_2)}_{=0} + u_1'y_1' + u_2'y_2' = g(x)$$

**Condition 2**: $u_1'y_1' + u_2'y_2' = g(x)$

---

## 3. The System to Solve

$$\begin{cases} u_1'y_1 + u_2'y_2 = 0 \\ u_1'y_1' + u_2'y_2' = g(x) \end{cases}$$

Using Cramer's rule:

$$\boxed{u_1' = \frac{-y_2 \cdot g(x)}{W}, \qquad u_2' = \frac{y_1 \cdot g(x)}{W}}$$

where $W = W(y_1, y_2) = y_1 y_2' - y_2 y_1'$ is the Wronskian.

Then:

$$\boxed{u_1 = \int \frac{-y_2 \cdot g(x)}{W}\,dx, \qquad u_2 = \int \frac{y_1 \cdot g(x)}{W}\,dx}$$

$$\boxed{y_p = u_1 y_1 + u_2 y_2}$$

---

## 4. Step-by-Step Procedure

```
┌───────────────────────────────────────┐
│ 1. Write ODE in standard form:       │
│    y'' + P(x)y' + Q(x)y = g(x)      │
│    (leading coefficient = 1!)        │
├───────────────────────────────────────┤
│ 2. Find y_c = c₁y₁ + c₂y₂           │
├───────────────────────────────────────┤
│ 3. Compute Wronskian W(y₁, y₂)       │
├───────────────────────────────────────┤
│ 4. Compute:                          │
│    u₁' = -y₂·g / W                  │
│    u₂' = y₁·g / W                   │
├───────────────────────────────────────┤
│ 5. Integrate: u₁ = ∫u₁' dx          │
│              u₂ = ∫u₂' dx           │
│    (omit constants of integration)   │
├───────────────────────────────────────┤
│ 6. y_p = u₁·y₁ + u₂·y₂             │
│    y = y_c + y_p                     │
└───────────────────────────────────────┘
```

---

## 5. Worked Examples

### Example 1: $y'' + y = \sec x$

**Step 1**: Already in standard form. $g(x) = \sec x$.

**Step 2**: $m^2 + 1 = 0 \implies y_1 = \cos x, y_2 = \sin x$

**Step 3**: $W = \cos x \cdot \cos x - \sin x \cdot (-\sin x) = \cos^2 x + \sin^2 x = 1$

**Step 4**:
$u_1' = -\sin x \cdot \sec x = -\tan x$

$u_2' = \cos x \cdot \sec x = 1$

**Step 5**:
$u_1 = \int -\tan x\,dx = \ln|\cos x|$

$u_2 = \int 1\,dx = x$

**Step 6**:
$$y_p = \cos x \ln|\cos x| + x\sin x$$

$$\boxed{y = c_1\cos x + c_2\sin x + \cos x \ln|\cos x| + x\sin x}$$

### Example 2: $y'' + y = \csc x$

$y_1 = \cos x$, $y_2 = \sin x$, $W = 1$

$u_1' = -\sin x \cdot \csc x = -1 \implies u_1 = -x$

$u_2' = \cos x \cdot \csc x = \cot x \implies u_2 = \ln|\sin x|$

$$y_p = -x\cos x + \sin x \ln|\sin x|$$

### Example 3: $y'' - 2y' + y = \dfrac{e^x}{x}$

$y_c = (c_1 + c_2 x)e^x$, so $y_1 = e^x$, $y_2 = xe^x$

$W = e^x(e^x + xe^x) - xe^x \cdot e^x = e^{2x}$

$g(x) = e^x/x$

$u_1' = \dfrac{-xe^x \cdot (e^x/x)}{e^{2x}} = \dfrac{-e^{2x}}{e^{2x}} = -1 \implies u_1 = -x$

$u_2' = \dfrac{e^x \cdot (e^x/x)}{e^{2x}} = \dfrac{1}{x} \implies u_2 = \ln|x|$

$$y_p = -xe^x + xe^x\ln|x| = xe^x(\ln|x| - 1)$$

$$\boxed{y = (c_1 + c_2 x)e^x + xe^x\ln|x| - xe^x}$$

The $-xe^x$ term is absorbed into $y_c$ (it's $c_2 xe^x$ with $c_2 = -1$), so:

$$y = (c_1 + c_2 x)e^x + xe^x\ln|x|$$

### Example 4: $y'' + 4y = \tan 2x$

$y_1 = \cos 2x$, $y_2 = \sin 2x$, $W = 2$

$u_1' = \dfrac{-\sin 2x \cdot \tan 2x}{2} = \dfrac{-\sin^2 2x}{2\cos 2x} = \dfrac{\cos^2 2x - 1}{2\cos 2x} = \dfrac{\cos 2x}{2} - \dfrac{\sec 2x}{2}$

$u_1 = \dfrac{\sin 2x}{4} - \dfrac{1}{4}\ln|\sec 2x + \tan 2x|$

$u_2' = \dfrac{\cos 2x \cdot \tan 2x}{2} = \dfrac{\sin 2x}{2}$

$u_2 = -\dfrac{\cos 2x}{4}$

$$y_p = \cos 2x\left(\frac{\sin 2x}{4} - \frac{1}{4}\ln|\sec 2x + \tan 2x|\right) + \sin 2x\left(-\frac{\cos 2x}{4}\right)$$

$$y_p = -\frac{1}{4}\cos 2x \ln|\sec 2x + \tan 2x|$$

(The $\cos 2x \sin 2x$ terms cancel.)

### Example 5: Euler-Cauchy — $x^2y'' - 2xy' + 2y = x^3$

Standard form: $y'' - \dfrac{2}{x}y' + \dfrac{2}{x^2}y = x$

$y_1 = x$, $y_2 = x^2$, $W = x \cdot 2x - x^2 \cdot 1 = x^2$

$u_1' = \dfrac{-x^2 \cdot x}{x^2} = -x \implies u_1 = -\dfrac{x^2}{2}$

$u_2' = \dfrac{x \cdot x}{x^2} = 1 \implies u_2 = x$

$y_p = -\dfrac{x^2}{2}\cdot x + x \cdot x^2 = -\dfrac{x^3}{2} + x^3 = \dfrac{x^3}{2}$

$$y = c_1 x + c_2 x^2 + \frac{x^3}{2}$$

---

## 6. Extension to Higher Order

For $y''' + P_2 y'' + P_1 y' + P_0 y = g(x)$ with fundamental set $\{y_1, y_2, y_3\}$:

$$y_p = u_1 y_1 + u_2 y_2 + u_3 y_3$$

$$u_k' = \frac{W_k}{W}$$

where $W_k$ is the Wronskian $W$ with column $k$ replaced by $(0, 0, \ldots, 0, g)^T$.

---

## 7. Comparison: Undetermined Coefficients vs. Variation of Parameters

| Feature | Undetermined Coefficients | Variation of Parameters |
|---------|--------------------------|------------------------|
| Applicable to | Constant coefficients only | Any linear ODE |
| $g(x)$ restriction | Polynomials, exp, sin/cos | Any continuous function |
| Difficulty | Algebra (easy-moderate) | Integration (can be hard) |
| Modification needed | When overlap with $y_c$ | Never |
| Higher order | Tedious but systematic | Formula extends naturally |
| Best for | $e^{ax}$, polynomials, trig | $\sec x$, $\ln x$, $1/x$, etc. |

---

## Summary Table

| Concept | Details |
|---------|---------|
| **Formulas** | $u_1' = -y_2 g/W$, $u_2' = y_1 g/W$ |
| **Wronskian** | $W = y_1 y_2' - y_2 y_1'$ |
| **Particular solution** | $y_p = u_1 y_1 + u_2 y_2$ |
| **Key requirement** | Standard form (leading coeff. = 1) |
| **Advantage** | Works for ANY continuous $g(x)$ |
| **Disadvantage** | Integration may be difficult |

---

## Quick Revision Questions

1. Use variation of parameters to solve $y'' + y = \tan x$.

2. Solve $y'' - y = e^x$ using variation of parameters and compare with undetermined coefficients.

3. Find a particular solution of $y'' + 4y = \sec 2x$.

4. For the system $u_1'y_1 + u_2'y_2 = 0$, $u_1'y_1' + u_2'y_2' = g$, derive $u_1'$ and $u_2'$ using Cramer's rule.

5. Solve $x^2y'' - xy' + y = x\ln x$ using variation of parameters (given $y_1 = x$, $y_2 = x\ln x$).

6. Why must the ODE be in standard form (leading coefficient 1) before applying the formulas?

---

[← Previous: Undetermined Coefficients](01-undetermined-coefficients.md) | [Back to README](../README.md) | [Next: D-Operator Methods →](03-d-operator-methods.md)
