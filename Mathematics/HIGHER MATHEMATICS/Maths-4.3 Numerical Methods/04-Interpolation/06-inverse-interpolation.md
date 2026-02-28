# Chapter 4.6: Inverse Interpolation

[← Previous: Divided Differences](05-divided-differences.md) | [Next: Spline Interpolation →](07-spline-interpolation.md)

---

## Overview

**Inverse interpolation** finds the value of $x$ corresponding to a given value of $y = f(x)$, using a table of known $(x_i, f_i)$ values. Instead of interpolating $f$ as a function of $x$, we interpolate $x$ as a function of $f$.

This is particularly useful for **finding roots**: set $y = 0$ and find the corresponding $x$.

---

## 1. Methods of Inverse Interpolation

### Method 1: Interchange Roles of $x$ and $f$

If we wish to find $x$ for a given $f$:

- Treat $f$ as the independent variable and $x$ as the dependent variable.
- Apply Lagrange's formula with arguments swapped:

$$x = \sum_{i=0}^{n} x_i \prod_{\substack{j=0 \\ j \neq i}}^{n} \frac{f - f_j}{f_i - f_j}$$

### Method 2: Iterative use of Forward Interpolation

Use Newton's forward formula to express $f(x)$ as a polynomial in $x$, then solve $f(x) = y$ for $x$.

---

## 2. Worked Example 1 — Lagrange Method

Given the data:

| $x$ | 2 | 3 | 4 | 5 |
|-----|---|---|---|---|
| $f(x)$ | 0.6931 | 1.0986 | 1.3863 | 1.6094 |

Find $x$ when $f(x) = 1.2$.

**Solution**: Apply Lagrange's formula with $f$ ↔ $x$ swapped.

$$x = \sum_{i=0}^{3} x_i \prod_{\substack{j=0 \\ j \neq i}}^{3} \frac{f - f_j}{f_i - f_j}$$

With $f = 1.2$:

$$L_0 = \frac{(1.2 - 1.0986)(1.2 - 1.3863)(1.2 - 1.6094)}{(0.6931 - 1.0986)(0.6931 - 1.3863)(0.6931 - 1.6094)}$$

$$= \frac{(0.1014)(-0.1863)(-0.4094)}{(-0.4055)(-0.6932)(-0.9163)}$$

$$= \frac{0.007735}{-0.25739} = -0.03004$$

Similarly computing $L_1, L_2, L_3$ and summing:

$$x \approx 2(-0.03004) + 3(0.6938) + 4(0.3942) + 5(-0.0580)$$

$$x \approx -0.0601 + 2.0814 + 1.5768 - 0.2900 \approx 3.3081$$

Since $f(x) = \ln x$, the exact answer is $e^{1.2} = 3.3201$. Our approximation is close.

---

## 3. Worked Example 2 — Finding a Root

Find a root of $f(x) = 0$ using:

| $x$ | 0.3 | 0.5 | 0.7 | 0.9 |
|-----|-----|-----|-----|-----|
| $f(x)$ | −0.52 | −0.08 | 0.33 | 0.72 |

Using Lagrange inverse interpolation with $f = 0$:

$$x = 0.3 \cdot \frac{(0-(-0.08))(0-0.33)(0-0.72)}{(-0.52-(-0.08))(-0.52-0.33)(-0.52-0.72)}$$

$$+ 0.5 \cdot \frac{(0-(-0.52))(0-0.33)(0-0.72)}{(-0.08-(-0.52))(-0.08-0.33)(-0.08-0.72)}$$

$$+ 0.7 \cdot \frac{(0-(-0.52))(0-(-0.08))(0-0.72)}{(0.33-(-0.52))(0.33-(-0.08))(0.33-0.72)}$$

$$+ 0.9 \cdot \frac{(0-(-0.52))(0-(-0.08))(0-0.33)}{(0.72-(-0.52))(0.72-(-0.08))(0.72-0.33)}$$

Computing each term:

| Term | Numerator | Denominator | $L_i(0)$ | $x_i \cdot L_i$ |
|------|-----------|-------------|-----------|-----------------|
| $i=0$ | $(0.08)(-0.33)(-0.72) = 0.01901$ | $(-0.44)(-0.85)(-1.24) = -0.46376$ | $-0.04099$ | $-0.01230$ |
| $i=1$ | $(0.52)(-0.33)(-0.72) = 0.12355$ | $(0.44)(-0.41)(-0.80) = 0.14432$ | $0.85612$ | $0.42806$ |
| $i=2$ | $(0.52)(0.08)(-0.72) = -0.02995$ | $(0.85)(0.41)(-0.39) = -0.13592$ | $0.22035$ | $0.15425$ |
| $i=3$ | $(0.52)(0.08)(-0.33) = -0.01373$ | $(1.24)(0.80)(0.39) = 0.38688$ | $-0.03549$ | $-0.03194$ |

$$x = -0.01230 + 0.42806 + 0.15425 - 0.03194 = 0.5381$$

The root of $f(x) = 0$ is approximately $x \approx 0.538$.

---

## 4. When to Use Inverse Interpolation

```
  ┌─────────────────────────────────────────┐
  │         Direct Interpolation            │
  │   Given: x ──► Find: f(x)              │
  │   Method: Standard Lagrange/Newton      │
  ├─────────────────────────────────────────┤
  │         Inverse Interpolation           │
  │   Given: f ──► Find: x                 │
  │   Method: Swap x and f, then apply     │
  │           Lagrange/Newton               │
  │                                         │
  │   Applications:                         │
  │     • Finding roots: f(x) = 0          │
  │     • Finding x for a target value     │
  │     • Solving implicit equations        │
  └─────────────────────────────────────────┘
```

---

## 5. Precautions

- **Monotonicity**: $f(x)$ should be monotonic in the interval (one-to-one) for inverse interpolation to be valid.
- **Sensitivity**: If $f'(x)$ is small near the answer, inverse interpolation amplifies errors.
- **Multiple roots**: If $f$ is not injective, the inverse function is multi-valued — the method may give incorrect results.

---

## Summary Table

| Property | Value |
|----------|-------|
| **Purpose** | Find $x$ given $y = f(x)$ |
| **Primary Method** | Lagrange with $x$ and $f$ swapped |
| **Root Finding** | Set $f = 0$ in the inverse formula |
| **Requirement** | $f$ should be monotonic |
| **Accuracy** | Same error bounds as direct interpolation |
| **Limitation** | Fails if $f$ is not one-to-one |

---

## Quick Revision Questions

1. **What is inverse interpolation? When is it used?**

2. **Describe two methods for performing inverse interpolation.**

3. **Using the data $(1,2), (2,11), (3,34)$, find $x$ when $f(x) = 20$ by inverse Lagrange interpolation.**

4. **Why must $f(x)$ be monotonic for inverse interpolation to work?**

5. **How is inverse interpolation used for root-finding?**

6. **What precaution is needed when $f'(x)$ is near zero in the region of interest?**

---

[← Previous: Divided Differences](05-divided-differences.md) | [Next: Spline Interpolation →](07-spline-interpolation.md)
