# Chapter 9.3: Exponential and Logarithmic Fit

[← Previous: Polynomial Fitting](02-polynomial-fitting.md) | [Next: Cubic Spline Fitting →](04-cubic-spline-fitting.md)

---

## Overview

Many real-world phenomena (population growth, radioactive decay, power laws) follow **non-linear** models. By applying logarithmic or other transformations, these can be **linearised** and solved using linear least squares.

---

## 1. Exponential Model: $y = ae^{bx}$

### Linearisation

Take natural log: $\ln y = \ln a + bx$

Let $Y = \ln y$, $A = \ln a$. Then:

$$Y = A + bx \quad \text{(linear in }x\text{ and }Y\text{)}$$

Apply linear least squares on $(x_i, Y_i)$ to get $A$ and $b$, then $a = e^A$.

### Normal Equations

$$\begin{aligned}
nA + b\sum x_i &= \sum \ln y_i \\
A\sum x_i + b\sum x_i^2 &= \sum x_i \ln y_i
\end{aligned}$$

---

## 2. Power Model: $y = ax^b$

### Linearisation

Take natural log: $\ln y = \ln a + b \ln x$

Let $X = \ln x$, $Y = \ln y$, $A = \ln a$. Then:

$$Y = A + bX$$

Apply linear least squares on $(X_i, Y_i)$.

---

## 3. Growth Model: $y = ab^x$

$$\ln y = \ln a + x \ln b$$

Let $Y = \ln y$, $A = \ln a$, $B = \ln b$:

$$Y = A + Bx$$

After fitting: $a = e^A$, $b = e^B$.

---

## 4. Reciprocal Models

| Model | Transformation | Linear form |
|-------|---------------|-------------|
| $y = \frac{1}{a+bx}$ | $Z = 1/y$ | $Z = a + bx$ |
| $y = \frac{x}{a+bx}$ | $Z = x/y$ | $Z = a + bx$ |
| $y = \frac{ax}{b+x}$ (Michaelis–Menten) | $\frac{1}{y} = \frac{1}{a} + \frac{b}{a} \cdot \frac{1}{x}$ | $Z = A + BW$ |

---

## 5. Worked Example: Exponential Fit

Fit $y = ae^{bx}$ to:

| $x$ | 1 | 2 | 3 | 4 | 5 |
|-----|---|---|---|---|---|
| $y$ | 2.7 | 7.4 | 20.1 | 54.6 | 148.4 |

### Step 1: Transform

| $x_i$ | $y_i$ | $Y_i = \ln y_i$ | $x_i^2$ | $x_i Y_i$ |
|--------|--------|----------------|---------|-----------|
| 1 | 2.7 | 0.9933 | 1 | 0.9933 |
| 2 | 7.4 | 2.0015 | 4 | 4.0030 |
| 3 | 20.1 | 3.0000 | 9 | 9.0000 |
| 4 | 54.6 | 3.9988 | 16 | 15.9953 |
| 5 | 148.4 | 4.9993 | 25 | 24.9965 |
| **$\sum$** | **15** | | **14.9929** | **55** | **54.9881** |

$n = 5$

### Step 2: Solve Normal Equations

$$b = \frac{5(54.9881) - 15(14.9929)}{5(55) - 225} = \frac{274.9405 - 224.8935}{50} = \frac{50.047}{50} = 1.001$$

$$A = \frac{14.9929 - 1.001(15)}{5} = \frac{14.9929 - 15.015}{5} = -0.0044$$

$$a = e^{-0.0044} = 0.9956 \approx 1.0$$

$$\boxed{y \approx e^{x} \quad (a \approx 1, \; b \approx 1)}$$

This confirms $y = e^x$ as expected from the data.

---

## 6. Worked Example: Power Law Fit

Fit $y = ax^b$ to:

| $x$ | 1 | 2 | 3 | 4 | 5 |
|-----|---|---|---|---|---|
| $y$ | 1 | 5.7 | 14.0 | 25.6 | 41.0 |

### Step 1: Transform ($X = \ln x, Y = \ln y$)

| $x$ | $y$ | $X = \ln x$ | $Y = \ln y$ | $X^2$ | $XY$ |
|------|------|------------|------------|-------|------|
| 1 | 1.0 | 0 | 0 | 0 | 0 |
| 2 | 5.7 | 0.6931 | 1.7405 | 0.4804 | 1.2061 |
| 3 | 14.0 | 1.0986 | 2.6391 | 1.2069 | 2.8996 |
| 4 | 25.6 | 1.3863 | 3.2426 | 1.9218 | 4.4954 |
| 5 | 41.0 | 1.6094 | 3.7136 | 2.5902 | 5.9768 |
| **$\sum$** | | | **4.7874** | **11.3358** | **6.1993** | **14.5779** |

$n = 5$

### Step 2: Solve

$$b = \frac{5(14.5779) - 4.7874(11.3358)}{5(6.1993) - (4.7874)^2} = \frac{72.889 - 54.270}{30.997 - 22.919} = \frac{18.619}{8.078} = 2.305$$

$$A = \frac{11.3358 - 2.305(4.7874)}{5} = \frac{11.3358 - 11.035}{5} = 0.060$$

$$a = e^{0.060} = 1.062$$

$$\boxed{y \approx 1.06 \, x^{2.3}}$$

---

## 7. Important Note: Weighted Least Squares

When we transform $y = ae^{bx}$ to $\ln y = \ln a + bx$, minimising $\sum(\ln y_i - A - bx_i)^2$ is **not** the same as minimising $\sum(y_i - ae^{bx_i})^2$.

The transformed fit minimises **relative** errors (in log-space). For equal weighting in original space, use **non-linear least squares** (Gauss–Newton or Levenberg–Marquardt).

```
  Transformed fit:          Original non-linear fit:
  min Σ(ln y - Ŷ)²         min Σ(y - ae^bx)²
       │                         │
       ▼                         ▼
  Good for log-scale        Good for absolute scale
  Equal relative errors     Equal absolute errors
```

---

## Summary Table

| Model | Transformation | Variables | Back-transform |
|-------|---------------|-----------|--------------|
| $y = ae^{bx}$ | $\ln y = \ln a + bx$ | $Y$ vs $x$ | $a=e^A$ |
| $y = ax^b$ | $\ln y = \ln a + b\ln x$ | $Y$ vs $X$ | $a=e^A$ |
| $y = ab^x$ | $\ln y = \ln a + x\ln b$ | $Y$ vs $x$ | $a=e^A,\, b=e^B$ |
| $y = 1/(a+bx)$ | $1/y = a+bx$ | $Z$ vs $x$ | Direct |
| General non-linear | Gauss–Newton iteration | — | N/A |

---

## Quick Revision Questions

1. **How do you linearise the model $y = ae^{bx}$? What are the resulting normal equations?**

2. **Fit $y = ax^b$ to $(1,3), (2,12), (3,27), (4,48)$.**

3. **What is the difference between minimising $\sum(y - \hat{y})^2$ and $\sum(\ln y - \ln\hat{y})^2$?**

4. **Linearise the Michaelis–Menten model $y = \frac{ax}{b+x}$.**

5. **Fit $y = ab^x$ to $(0,2), (1,6), (2,18), (3,54)$.**

6. **When would you prefer non-linear least squares over linearisation?**

---

[← Previous: Polynomial Fitting](02-polynomial-fitting.md) | [Next: Cubic Spline Fitting →](04-cubic-spline-fitting.md)
