# Chapter 9.2: Polynomial Fitting

[← Previous: Method of Least Squares](01-least-squares.md) | [Next: Exponential & Logarithmic Fit →](03-exponential-fit.md)

---

## Overview

When data shows a curved trend, a straight line is inadequate. **Polynomial regression** extends least squares to fit polynomials of degree $m$:

$$\hat{y} = a_0 + a_1 x + a_2 x^2 + \cdots + a_m x^m$$

The key constraint is $m < n - 1$ (under-determined to avoid overfitting).

---

## 1. Normal Equations for Degree-$m$ Polynomial

Minimise $S = \sum_{i=1}^{n}(y_i - a_0 - a_1 x_i - a_2 x_i^2 - \cdots - a_m x_i^m)^2$

Setting $\frac{\partial S}{\partial a_j} = 0$ for $j = 0, 1, \ldots, m$ gives $m+1$ normal equations:

$$\boxed{\begin{bmatrix} n & \sum x & \sum x^2 & \cdots & \sum x^m \\ \sum x & \sum x^2 & \sum x^3 & \cdots & \sum x^{m+1} \\ \vdots & & & \ddots & \vdots \\ \sum x^m & \sum x^{m+1} & & \cdots & \sum x^{2m} \end{bmatrix} \begin{bmatrix} a_0 \\ a_1 \\ \vdots \\ a_m \end{bmatrix} = \begin{bmatrix} \sum y \\ \sum xy \\ \vdots \\ \sum x^m y \end{bmatrix}}$$

This is the **Vandermonde system** with the Gram matrix $\Phi^T\Phi$.

---

## 2. Degree Selection

```
  Error
  │ ·                   ← Training error
  │  \                  
  │   \      ....... ← Test/validation error
  │    \ ...·
  │     ·  /
  │       /
  │      ← Sweet spot (optimal degree)
  └───────────────────── Degree m
     1   2   3   4   5
```

| Criterion | Formula / Rule |
|-----------|---------------|
| **Visual inspection** | Plot residuals; look for patterns |
| **$R^2$ adjusted** | $R^2_{\text{adj}} = 1 - \frac{(1-R^2)(n-1)}{n-m-1}$ |
| **AIC** | $\text{AIC} = n\ln(S/n) + 2(m+1)$ |
| **Residual analysis** | Residuals should be random, not systematic |

---

## 3. Quadratic Fit ($m=2$): $y = a_0 + a_1 x + a_2 x^2$

Normal equations:

$$\begin{aligned}
na_0 + a_1\sum x + a_2\sum x^2 &= \sum y \\
a_0\sum x + a_1\sum x^2 + a_2\sum x^3 &= \sum xy \\
a_0\sum x^2 + a_1\sum x^3 + a_2\sum x^4 &= \sum x^2 y
\end{aligned}$$

---

## 4. Worked Example: Quadratic Fit

Fit $y = a_0 + a_1 x + a_2 x^2$ to:

| $x$ | 0 | 1 | 2 | 3 | 4 |
|-----|---|---|---|---|---|
| $y$ | 1 | 1.8 | 1.3 | 2.5 | 6.3 |

$n = 5$

### Computation Table

| $x$ | $y$ | $x^2$ | $x^3$ | $x^4$ | $xy$ | $x^2 y$ |
|------|------|-------|-------|-------|------|---------|
| 0 | 1.0 | 0 | 0 | 0 | 0 | 0 |
| 1 | 1.8 | 1 | 1 | 1 | 1.8 | 1.8 |
| 2 | 1.3 | 4 | 8 | 16 | 2.6 | 5.2 |
| 3 | 2.5 | 9 | 27 | 81 | 7.5 | 22.5 |
| 4 | 6.3 | 16 | 64 | 256 | 25.2 | 100.8 |
| **$\sum$** | **10** | **12.9** | **30** | **100** | **354** | **37.1** | **130.3** |

Normal equations:

$$\begin{bmatrix} 5 & 10 & 30 \\ 10 & 30 & 100 \\ 30 & 100 & 354 \end{bmatrix} \begin{bmatrix} a_0 \\ a_1 \\ a_2 \end{bmatrix} = \begin{bmatrix} 12.9 \\ 37.1 \\ 130.3 \end{bmatrix}$$

Solving (by Gaussian elimination or any method):

$$a_0 = 1.42, \quad a_1 = -0.86, \quad a_2 = 0.54$$

$$\boxed{y = 1.42 - 0.86x + 0.54x^2}$$

### Verification

| $x$ | $y$ | $\hat{y}$ | Residual |
|------|------|-----------|----------|
| 0 | 1.0 | 1.42 | −0.42 |
| 1 | 1.8 | 1.10 | +0.70 |
| 2 | 1.3 | 1.86 | −0.56 |
| 3 | 2.5 | 3.70 | −1.20 |
| 4 | 6.3 | 6.62 | −0.32 |

---

## 5. Ill-Conditioning of Vandermonde Matrices

The Gram matrix $\Phi^T\Phi$ becomes **ill-conditioned** for high-degree polynomials:

$$\text{cond}(\Phi^T\Phi) \sim \left(\frac{x_{\max}}{x_{\min}}\right)^{2m}$$

**Remedies:**
| Technique | Effect |
|-----------|--------|
| **Centering**: use $\bar{x} = x - \text{mean}$ | Reduces powers |
| **Orthogonal polynomials** (Chebyshev, Legendre) | Diagonalises $\Phi^T\Phi$ |
| **QR decomposition** | Numerically stable solve |
| **Scaling**: normalise $x$ to $[-1,1]$ | Bounds condition number |

---

## 6. Orthogonal Polynomial Fitting

Use orthogonal polynomials $P_0, P_1, \ldots, P_m$ satisfying $\sum P_j(x_i)P_k(x_i) = 0$ for $j \neq k$.

Then the coefficients decouple:

$$a_j = \frac{\sum y_i P_j(x_i)}{\sum [P_j(x_i)]^2}$$

No system of equations to solve — each coefficient computed independently!

**Gram–Schmidt on monomials** gives the discrete orthogonal polynomial family for the given data.

---

## Summary Table

| Aspect | Detail |
|--------|--------|
| **Model** | $\hat{y} = a_0 + a_1 x + \cdots + a_m x^m$ |
| **Normal equations** | $(m+1) \times (m+1)$ system via $\Phi^T\Phi \mathbf{a} = \Phi^T \mathbf{y}$ |
| **Degree choice** | Use $R^2_{\text{adj}}$, AIC, or residual analysis |
| **Ill-conditioning** | Grows with degree; mitigate with centering/orthogonal polys |
| **Orthogonal polys** | Decouple coefficients, numerically stable |
| **Overfitting** | $m \ll n-1$ to generalise well |

---

## Quick Revision Questions

1. **Write the normal equations for fitting a second-degree polynomial to $n$ data points.**

2. **Fit $y = a_0 + a_1 x + a_2 x^2$ to $(0,1), (1,2), (2,5), (3,10)$.**

3. **Why does the Vandermonde matrix become ill-conditioned for large degree $m$?**

4. **What is the advantage of orthogonal polynomial fitting?**

5. **How does adjusted $R^2$ differ from ordinary $R^2$? Why is it preferred?**

6. **Describe two practical ways to reduce ill-conditioning in polynomial regression.**

---

[← Previous: Method of Least Squares](01-least-squares.md) | [Next: Exponential & Logarithmic Fit →](03-exponential-fit.md)
