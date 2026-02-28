# Chapter 9.1: Method of Least Squares

[← Previous: QR Algorithm](../08-Eigenvalue-Problems/03-qr-algorithm.md) | [Next: Polynomial Fitting →](02-polynomial-fitting.md)

---

## Overview

The **Method of Least Squares** finds the **best-fit** curve for a set of data points by minimising the sum of squared residuals. Unlike interpolation (which passes through every point), least squares finds the curve that best represents the overall trend, even with noisy data.

---

## 1. Problem Statement

Given $n$ data points $(x_1, y_1), (x_2, y_2), \ldots, (x_n, y_n)$, find a fitting function $\hat{y} = \phi(x)$ that minimises:

$$\boxed{S = \sum_{i=1}^{n} [y_i - \phi(x_i)]^2 = \sum_{i=1}^{n} r_i^2}$$

where $r_i = y_i - \phi(x_i)$ is the **residual** at point $i$.

---

## 2. Linear Least Squares: $y = a + bx$

Minimise $S = \sum_{i=1}^{n}(y_i - a - bx_i)^2$.

Setting $\frac{\partial S}{\partial a} = 0$ and $\frac{\partial S}{\partial b} = 0$:

$$\boxed{\begin{aligned} na + b\sum x_i &= \sum y_i \\ a\sum x_i + b\sum x_i^2 &= \sum x_i y_i \end{aligned}}$$

These are the **normal equations**. Solving:

$$\boxed{b = \frac{n\sum x_i y_i - \sum x_i \sum y_i}{n\sum x_i^2 - (\sum x_i)^2}, \quad a = \bar{y} - b\bar{x}}$$

---

## 3. Geometric Interpretation

```
  y
  │      ●
  │        ●       Best-fit line
  │     ●  │╱─────────────────
  │   ↕  ╱─│──●
  │  ●╱────●     ← residuals (vertical distances)
  │ ╱   ●
  │╱  ●
  └──────────────────── x

  The line minimises the sum of squared vertical distances
```

---

## 4. Worked Example

Fit $y = a + bx$ to:

| $x$ | 1 | 2 | 3 | 4 | 5 |
|-----|---|---|---|---|---|
| $y$ | 2.1 | 3.9 | 6.2 | 7.8 | 10.1 |

### Computation Table

| $x_i$ | $y_i$ | $x_i^2$ | $x_i y_i$ |
|--------|--------|---------|-----------|
| 1 | 2.1 | 1 | 2.1 |
| 2 | 3.9 | 4 | 7.8 |
| 3 | 6.2 | 9 | 18.6 |
| 4 | 7.8 | 16 | 31.2 |
| 5 | 10.1 | 25 | 50.5 |
| **$\sum$** | **15** | **30.1** | **55** | **110.2** |

$n = 5$

$$b = \frac{5(110.2) - 15(30.1)}{5(55) - 15^2} = \frac{551 - 451.5}{275 - 225} = \frac{99.5}{50} = 1.99$$

$$a = \frac{30.1}{5} - 1.99 \times \frac{15}{5} = 6.02 - 5.97 = 0.05$$

$$\boxed{y = 0.05 + 1.99x}$$

---

## 5. Goodness of Fit: $R^2$

The **coefficient of determination**:

$$R^2 = 1 - \frac{\sum(y_i - \hat{y}_i)^2}{\sum(y_i - \bar{y})^2}$$

- $R^2 = 1$: perfect fit
- $R^2 = 0$: no better than using $\bar{y}$
- $R^2 \geq 0.9$: generally indicates a good fit

---

## 6. Matrix Formulation

For any linear model $\hat{y} = \sum_{j} a_j \phi_j(x)$:

$$\underbrace{\begin{bmatrix} \phi_1(x_1) & \phi_2(x_1) & \cdots \\ \phi_1(x_2) & \phi_2(x_2) & \cdots \\ \vdots & & \ddots \end{bmatrix}}_{\Phi} \begin{bmatrix} a_1 \\ a_2 \\ \vdots \end{bmatrix} = \begin{bmatrix} y_1 \\ y_2 \\ \vdots \end{bmatrix}$$

Normal equations: $\boxed{\Phi^T \Phi \, \mathbf{a} = \Phi^T \mathbf{y}}$

---

## 7. Fitting Non-Linear Models via Transformation

| Model | Transformation | Linear form |
|-------|---------------|-------------|
| $y = ae^{bx}$ | $\ln y = \ln a + bx$ | $Y = A + bx$ |
| $y = ax^b$ | $\ln y = \ln a + b \ln x$ | $Y = A + bX$ |
| $y = a \cdot b^x$ | $\ln y = \ln a + x \ln b$ | $Y = A + Bx$ |
| $y = \frac{1}{a + bx}$ | $1/y = a + bx$ | $Z = a + bx$ |

Fit the transformed model using linear least squares, then back-transform.

---

## Summary Table

| Property | Value |
|----------|-------|
| **Objective** | Minimise $\sum r_i^2$ |
| **Normal equations** | $\Phi^T\Phi \mathbf{a} = \Phi^T \mathbf{y}$ |
| **Linear $y=a+bx$** | $b = \frac{n\sum xy - \sum x \sum y}{n\sum x^2 - (\sum x)^2}$ |
| **Goodness of fit** | $R^2 = 1 - \frac{SS_{\text{res}}}{SS_{\text{tot}}}$ |
| **Non-linear models** | Transform to linear form, then fit |

---

## Quick Revision Questions

1. **Derive the normal equations for fitting $y = a + bx$ by least squares.**

2. **Fit $y = a + bx$ to $(1,1), (2,1.5), (3,2.2), (4,2.8)$.**

3. **What does the $R^2$ value represent? How is it calculated?**

4. **Fit an exponential $y = ae^{bx}$ to $(0,2), (1,5.4), (2,14.8)$.**

5. **Write the normal equations in matrix form for polynomial fitting.**

6. **Why do we minimise the sum of squared residuals rather than the sum of absolute residuals?**

---

[← Previous: QR Algorithm](../08-Eigenvalue-Problems/03-qr-algorithm.md) | [Next: Polynomial Fitting →](02-polynomial-fitting.md)
