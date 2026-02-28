# Chapter 4.5: Newton's Divided Difference Interpolation

[← Previous: Lagrange Interpolation](04-lagrange-interpolation.md) | [Next: Inverse Interpolation →](06-inverse-interpolation.md)

---

## Overview

**Newton's Divided Difference Interpolation** is the most general Newton-type formula. It works for **unequally spaced** data, is easy to extend when new data points are added, and naturally connects to the forward/backward difference formulas for equally spaced data.

---

## 1. Divided Differences — Definition

### First Divided Difference

$$f[x_i, x_j] = \frac{f(x_j) - f(x_i)}{x_j - x_i}$$

### Second Divided Difference

$$f[x_i, x_j, x_k] = \frac{f[x_j, x_k] - f[x_i, x_j]}{x_k - x_i}$$

### General $n$-th Divided Difference

$$\boxed{f[x_0, x_1, \ldots, x_n] = \frac{f[x_1, \ldots, x_n] - f[x_0, \ldots, x_{n-1}]}{x_n - x_0}}$$

---

## 2. Divided Difference Table

```
   x      f(x)     1st DD      2nd DD      3rd DD
  ┌────┬────────┬───────────┬───────────┬───────────┐
  │ x₀ │ f[x₀]  │           │           │           │
  │    │        │ f[x₀,x₁] │           │           │
  │ x₁ │ f[x₁]  │           │f[x₀,x₁,x₂]│         │
  │    │        │ f[x₁,x₂] │           │f[x₀,x₁,x₂,x₃]│
  │ x₂ │ f[x₂]  │           │f[x₁,x₂,x₃]│         │
  │    │        │ f[x₂,x₃] │           │           │
  │ x₃ │ f[x₃]  │           │           │           │
  └────┴────────┴───────────┴───────────┴───────────┘
         ↑           ↑            ↑           ↑
       Column 0   Column 1    Column 2    Column 3
```

---

## 3. Newton's Divided Difference Formula

$$\boxed{P_n(x) = f[x_0] + (x-x_0)f[x_0,x_1] + (x-x_0)(x-x_1)f[x_0,x_1,x_2] + \cdots}$$

In compact notation:

$$P_n(x) = f[x_0] + \sum_{k=1}^{n} f[x_0, x_1, \ldots, x_k] \prod_{j=0}^{k-1} (x - x_j)$$

---

## 4. Worked Example

Construct the interpolating polynomial for:

| $x$ | 1 | 2 | 4 | 7 |
|-----|---|---|---|---|
| $f(x)$ | 22 | 30 | 82 | 274 |

### Step 1: Build the Divided Difference Table

**1st Divided Differences:**

$$f[1,2] = \frac{30-22}{2-1} = 8$$

$$f[2,4] = \frac{82-30}{4-2} = 26$$

$$f[4,7] = \frac{274-82}{7-4} = 64$$

**2nd Divided Differences:**

$$f[1,2,4] = \frac{26-8}{4-1} = 6$$

$$f[2,4,7] = \frac{64-26}{7-2} = \frac{38}{5} = 7.6$$

**3rd Divided Difference:**

$$f[1,2,4,7] = \frac{7.6 - 6}{7-1} = \frac{1.6}{6} \approx 0.2\overline{6}$$

### Complete Table

| $x$ | $f(x)$ | 1st DD | 2nd DD | 3rd DD |
|-----|---------|--------|--------|--------|
| 1 | 22 | | | |
| | | 8 | | |
| 2 | 30 | | 6 | |
| | | 26 | | 0.267 |
| 4 | 82 | | 7.6 | |
| | | 64 | | |
| 7 | 274 | | | |

### Step 2: Write the Polynomial

$$P_3(x) = 22 + 8(x-1) + 6(x-1)(x-2) + 0.267(x-1)(x-2)(x-4)$$

### Verification

At $x = 2$: $P_3(2) = 22 + 8(1) + 6(1)(0) + 0.267(1)(0)(-2) = 30$ ✓

---

## 5. Connection to Equal Spacing

When $x_i = x_0 + ih$ (equal spacing), divided differences reduce to ordinary differences:

$$f[x_0, x_1, \ldots, x_n] = \frac{\Delta^n f_0}{n! \, h^n}$$

Substituting into the divided difference formula gives Newton's forward formula.

---

## 6. Key Properties

1. **Symmetry**: Divided differences are symmetric functions of their arguments:
   $$f[x_0, x_1] = f[x_1, x_0]$$

2. **Leibniz formula**: $f[x_0, x_1, \ldots, x_n] = \sum_{i=0}^{n} \frac{f(x_i)}{\prod_{j \neq i}(x_i - x_j)}$

3. **Mean Value Theorem Connection**: $f[x_0, x_1] = f'(\xi)$ for some $\xi \in (x_0, x_1)$

4. **Adding a Point**: Only one new column entry needed — no recomputation required!

---

## 7. Algorithm (Pseudocode)

```
INPUT: (x₀,f₀), (x₁,f₁), ..., (xₙ,fₙ) and point x

  Step 1: Build DD table
    For i = 0,...,n: d[i][0] = f[i]
    For j = 1,...,n:
      For i = 0,...,n-j:
        d[i][j] = (d[i+1][j-1] - d[i][j-1]) / (x[i+j] - x[i])

  Step 2: Evaluate polynomial
    P = d[0][n]
    For i = n-1 down to 0:
      P = P * (x - x[i]) + d[0][i]

OUTPUT: P = Pₙ(x)
```

---

## Summary Table

| Property | Value |
|----------|-------|
| **Data Spacing** | Any (unequal or equal) |
| **Uses** | Top diagonal of DD table: $f[x_0], f[x_0,x_1], \ldots$ |
| **Adding a Point** | Adds one new column — very efficient |
| **Complexity** | $O(n^2)$ to build table |
| **Error** | $f[x_0, \ldots, x_n, x] \prod_{i=0}^{n}(x-x_i)$ |
| **Relation** | $\Delta^n f_0 = n! \, h^n \, f[x_0, \ldots, x_n]$ for equal $h$ |

---

## Quick Revision Questions

1. **Define the third divided difference $f[x_0, x_1, x_2, x_3]$.**

2. **Construct the divided difference table for $(0,1), (1,2), (3,10), (4,17)$ and find $P_3(x)$.**

3. **Show that $f[x_0, x_1, \ldots, x_n]$ is a symmetric function of its arguments.**

4. **Relate Newton's divided difference formula to the forward difference formula for equally spaced data.**

5. **What is the advantage of the divided difference form over Lagrange's form when a new data point is added?**

6. **Prove: $f[x_0, x_1] = f[x_1, x_0]$.**

---

[← Previous: Lagrange Interpolation](04-lagrange-interpolation.md) | [Next: Inverse Interpolation →](06-inverse-interpolation.md)
