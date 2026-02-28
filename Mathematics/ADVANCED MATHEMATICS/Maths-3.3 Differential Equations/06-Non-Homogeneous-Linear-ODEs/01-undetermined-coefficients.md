# Chapter 6.1: Method of Undetermined Coefficients

[← Previous: Characteristic Equation](../05-Higher-Order-Linear-ODEs/05-characteristic-equation.md) | [Back to README](../README.md) | [Next: Variation of Parameters →](02-variation-of-parameters.md)

---

## Overview

For non-homogeneous linear ODEs $L[y] = g(x)$ with constant coefficients, the method of undetermined coefficients finds a **particular solution** $y_p$ by "guessing" its form based on $g(x)$ and adjusting constants. The general solution is then $y = y_c + y_p$.

---

## 1. Structure of the General Solution

$$\boxed{y = y_c + y_p}$$

where:
- $y_c$ = complementary function (general solution of $L[y] = 0$)
- $y_p$ = any particular solution of $L[y] = g(x)$

```
NON-HOMOGENEOUS SOLUTION STRUCTURE

  L[y] = g(x)
      │
      ├──→  y_c : solve L[y] = 0  (homogeneous part)
      │         Contains arbitrary constants c₁, c₂, ...
      │
      └──→  y_p : find ONE solution of L[y] = g(x)
                No arbitrary constants
      │
      └──→  y = y_c + y_p  (GENERAL SOLUTION)
```

---

## 2. Admissible Forms of $g(x)$

The method works when $g(x)$ is a **finite linear combination** of:

| Function Type | Examples |
|--------------|---------|
| Polynomials | $1, x, x^2, 3x^3 - 2x + 1$ |
| Exponentials | $e^{ax}$ |
| Sines/Cosines | $\sin bx$, $\cos bx$ |
| Products of above | $x^2 e^{3x}$, $e^x\sin 2x$, $x\cos x$ |

> **Does NOT work for**: $\ln x$, $\tan x$, $1/x$, $\sec x$, etc. (Use variation of parameters for these.)

---

## 3. The Trial Solution Table

### Basic Forms (No Overlap with $y_c$)

| $g(x)$ | Trial particular solution $y_p$ |
|---------|-------------------------------|
| $ke^{ax}$ | $Ae^{ax}$ |
| $k\sin bx$ or $k\cos bx$ | $A\cos bx + B\sin bx$ |
| $kx^n$ | $A_n x^n + A_{n-1}x^{n-1} + \cdots + A_1 x + A_0$ |
| $ke^{ax}\sin bx$ or $ke^{ax}\cos bx$ | $e^{ax}(A\cos bx + B\sin bx)$ |
| $kx^n e^{ax}$ | $e^{ax}(A_n x^n + \cdots + A_0)$ |
| $kx^n\sin bx$ or $kx^n\cos bx$ | $(A_n x^n + \cdots + A_0)\cos bx + (B_n x^n + \cdots + B_0)\sin bx$ |

> **Rule**: Always include **both** $\sin$ and $\cos$ terms, even if $g(x)$ has only one.

---

## 4. The Modification Rule (When $y_p$ Overlaps $y_c$)

If any term of the trial $y_p$ is already a solution of the homogeneous equation (appears in $y_c$), **multiply by $x^s$** where $s$ is the smallest positive integer that removes the overlap.

```
MODIFICATION RULE DECISION TREE

  Write trial y_p based on g(x)
           │
           ▼
  Does any term of y_p
  appear in y_c?
      ╱         ╲
    NO           YES
     │            │
     ▼            ▼
  Use y_p      Multiply ENTIRE
  as is        trial by x^s
               (s = multiplicity
                of overlap)
```

### Common Situations

| $g(x)$ | Char. roots | $y_c$ overlap | $y_p$ form |
|---------|-------------|---------------|------------|
| $e^{2x}$ | $m = 2, 3$ | $e^{2x}$ ∈ $y_c$ | $Axe^{2x}$ |
| $e^{2x}$ | $m = 2, 2$ | $e^{2x}, xe^{2x}$ ∈ $y_c$ | $Ax^2 e^{2x}$ |
| $\sin 3x$ | $m = \pm 3i$ | $\sin 3x, \cos 3x$ ∈ $y_c$ | $x(A\cos 3x + B\sin 3x)$ |
| $5$ | $m = 0, 2$ | $1$ ∈ $y_c$ | $Ax$ |
| $3x$ | $m = 0, 0$ | $1, x$ ∈ $y_c$ | $x^2(Ax + B)$ |

---

## 5. Worked Examples

### Example 1: Simple Exponential — $y'' - 3y' + 2y = e^{5x}$

**Step 1**: $y_c$: $m^2 - 3m + 2 = 0 \implies m = 1, 2$

$y_c = c_1 e^x + c_2 e^{2x}$

**Step 2**: $g(x) = e^{5x}$. No overlap. Trial: $y_p = Ae^{5x}$

**Step 3**: $y_p'' - 3y_p' + 2y_p = 25Ae^{5x} - 15Ae^{5x} + 2Ae^{5x} = 12Ae^{5x} = e^{5x}$

$A = 1/12$

$$\boxed{y = c_1 e^x + c_2 e^{2x} + \frac{1}{12}e^{5x}}$$

### Example 2: Polynomial — $y'' + y = x^2$

$y_c = c_1\cos x + c_2\sin x$

Trial: $y_p = Ax^2 + Bx + C$

$y_p'' + y_p = 2A + Ax^2 + Bx + C = x^2$

Comparing: $A = 1$, $B = 0$, $2A + C = 0 \implies C = -2$

$$y_p = x^2 - 2$$

### Example 3: Overlap — $y'' - 2y' + y = e^x$

$y_c$: $m^2 - 2m + 1 = (m-1)^2 = 0 \implies y_c = (c_1 + c_2 x)e^x$

$g(x) = e^x$ overlaps! Both $e^x$ and $xe^x$ are in $y_c$. Multiply by $x^2$:

$y_p = Ax^2 e^x$

$y_p' = A(2x + x^2)e^x$, $y_p'' = A(2 + 4x + x^2)e^x$

Substitute: $A(2 + 4x + x^2)e^x - 2A(2x + x^2)e^x + Ax^2 e^x = e^x$

$A[2 + 4x + x^2 - 4x - 2x^2 + x^2]e^x = 2Ae^x = e^x$

$A = 1/2$

$$\boxed{y = (c_1 + c_2 x)e^x + \frac{1}{2}x^2 e^x}$$

### Example 4: Trigonometric — $y'' + 4y = \sin 2x$

$y_c$: $m^2 + 4 = 0 \implies m = \pm 2i \implies y_c = c_1\cos 2x + c_2\sin 2x$

$g(x) = \sin 2x$ overlaps! Multiply by $x$:

$y_p = x(A\cos 2x + B\sin 2x)$

$y_p' = (A\cos 2x + B\sin 2x) + x(-2A\sin 2x + 2B\cos 2x)$

$y_p'' = (-4A\sin 2x + 4B\cos 2x) + x(-4A\cos 2x - 4B\sin 2x)$

$y_p'' + 4y_p = -4A\sin 2x + 4B\cos 2x = \sin 2x$

$B = 0$, $A = -1/4$

$$\boxed{y = c_1\cos 2x + c_2\sin 2x - \frac{x}{4}\cos 2x}$$

### Example 5: Product Form — $y'' - y = xe^{2x}$

$y_c = c_1 e^x + c_2 e^{-x}$. No overlap.

$y_p = (Ax + B)e^{2x}$

$y_p' = (A + 2Ax + 2B)e^{2x}$

$y_p'' = (4A + 4Ax + 4B)e^{2x}$

$y_p'' - y_p = (4A + 4Ax + 4B - Ax - B)e^{2x} = (3Ax + 4A + 3B)e^{2x} = xe^{2x}$

$3A = 1 \implies A = 1/3$; $4A + 3B = 0 \implies B = -4/9$

$$y_p = \left(\frac{x}{3} - \frac{4}{9}\right)e^{2x}$$

### Example 6: Multiple Terms — $y'' + y' - 2y = 3x + e^x$

$y_c$: $m^2 + m - 2 = (m+2)(m-1) = 0 \implies y_c = c_1 e^{-2x} + c_2 e^x$

For $3x$: trial $Ax + B$ (no overlap)
For $e^x$: overlap with $e^x$! → trial $Cxe^x$

$y_p = Ax + B + Cxe^x$

Substituting and comparing coefficients:

- From $x$: $-2A = 3 \implies A = -3/2$
- From constant: $A - 2B = 0 \implies B = -3/4$
- From $e^x$: $C(1 + 2 + 1 - 2) + C(1 + 1 - 2)x \cdot e^x$... Let me compute properly:

$y_p'' + y_p' - 2y_p$:
- From polynomial: $0 + (A) + (-2Ax - 2B) = -2Ax + A - 2B = 3x + 0$
- From $Cxe^x$: $y_p'' = C(2+x)e^x$, $y_p' = C(1+x)e^x$

$C(2+x)e^x + C(1+x)e^x - 2Cxe^x = C(3)e^x = e^x \implies C = 1/3$

$$\boxed{y = c_1 e^{-2x} + c_2 e^x - \frac{3}{2}x - \frac{3}{4} + \frac{1}{3}xe^x}$$

---

## 6. Superposition for Multiple $g(x)$ Terms

If $L[y] = g_1(x) + g_2(x) + \cdots + g_k(x)$:

Find $y_{p_1}, y_{p_2}, \ldots, y_{p_k}$ separately, then:

$$y_p = y_{p_1} + y_{p_2} + \cdots + y_{p_k}$$

---

## Summary Table

| Concept | Details |
|---------|---------|
| **General solution** | $y = y_c + y_p$ |
| **Admissible $g(x)$** | Polynomials, exponentials, sin/cos, products |
| **Trial for $e^{ax}$** | $Ae^{ax}$ |
| **Trial for $\sin bx$** | $A\cos bx + B\sin bx$ |
| **Trial for $x^n$** | $A_nx^n + \cdots + A_0$ |
| **Modification rule** | If overlap with $y_c$, multiply trial by $x^s$ |
| **Superposition** | Split $g(x)$ into parts, solve each separately |

---

## Quick Revision Questions

1. Find $y_p$ for $y'' + y = 5e^{2x}$.

2. Solve $y'' - 4y = 8x^2$.

3. Find the correct trial form (with undetermined coefficients) for $y'' + 9y = 2\sin 3x$.

4. Solve $y'' + y' = x + 1$. *(Careful: $m = 0$ is a root!)*

5. Find $y_p$ for $y'' - y' = 3e^x\cos 2x$.

6. Determine the form of $y_p$ (do not solve) for $y'' - 4y' + 4y = e^{2x} + xe^{2x}$.

---

[← Previous: Characteristic Equation](../05-Higher-Order-Linear-ODEs/05-characteristic-equation.md) | [Back to README](../README.md) | [Next: Variation of Parameters →](02-variation-of-parameters.md)
