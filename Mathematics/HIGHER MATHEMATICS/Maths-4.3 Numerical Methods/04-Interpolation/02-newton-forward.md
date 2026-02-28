# Chapter 4.2: Newton's Forward Difference Interpolation

[← Previous: Finite Differences](01-finite-differences.md) | [Next: Newton's Backward Difference →](03-newton-backward.md)

---

## Overview

Newton's Forward Difference formula constructs an interpolating polynomial using the **forward difference table**. It is most accurate when interpolating near the **beginning** of the data table.

---

## 1. The Formula

Let $x = x_0 + sh$ where $s = \frac{x - x_0}{h}$. Then:

$$\boxed{f(x) = f_0 + s\Delta f_0 + \frac{s(s-1)}{2!}\Delta^2 f_0 + \frac{s(s-1)(s-2)}{3!}\Delta^3 f_0 + \cdots}$$

$$f(x) = \sum_{k=0}^{n} \binom{s}{k} \Delta^k f_0$$

where $\binom{s}{k} = \frac{s(s-1)(s-2)\cdots(s-k+1)}{k!}$

---

## 2. When to Use

- Data is **equally spaced**
- Interpolation point is near the **beginning** ($x_0$) of the table
- $s$ is between 0 and 1 (ideally) or small positive

```
  Data:  x₀    x₁    x₂    x₃    x₄    x₅
         ●─────●─────●─────●─────●─────●
         ↑                              
    Best accuracy                       
    for forward                         
    interpolation                       
```

---

## 3. Worked Example

Given:

| $x$ | 0 | 1 | 2 | 3 | 4 |
|:---:|:-:|:-:|:-:|:-:|:-:|
| $f(x)$ | 1 | 2 | 9 | 28 | 65 |

Find $f(1.5)$.

### Step 1: Build forward difference table

| $x$ | $f$ | $\Delta f$ | $\Delta^2 f$ | $\Delta^3 f$ | $\Delta^4 f$ |
|:---:|:---:|:----------:|:------------:|:------------:|:------------:|
| 0 | 1 | 1 | 6 | 6 | 0 |
| 1 | 2 | 7 | 12 | 6 | |
| 2 | 9 | 19 | 18 | | |
| 3 | 28 | 37 | | | |
| 4 | 65 | | | | |

### Step 2: Compute $s$

$$s = \frac{x - x_0}{h} = \frac{1.5 - 0}{1} = 1.5$$

### Step 3: Apply formula

$$f(1.5) = 1 + 1.5(1) + \frac{1.5(0.5)}{2}(6) + \frac{1.5(0.5)(-0.5)}{6}(6) + \frac{1.5(0.5)(-0.5)(-1.5)}{24}(0)$$

$$= 1 + 1.5 + 2.25 + (-0.375) + 0$$

$$= 4.375$$

**Check**: $f(x) = x^3 + 1$, so $f(1.5) = 3.375 + 1 = 4.375$ ✓

---

## 4. Error Term

The error in the $n$-th degree interpolation is:

$$R_n = \frac{s(s-1)(s-2)\cdots(s-n)}{(n+1)!} h^{n+1} f^{(n+1)}(\xi)$$

for some $\xi$ in the interval.

This can be approximated using the next difference:

$$R_n \approx \binom{s}{n+1} \Delta^{n+1} f_0$$

---

## 5. Derivation Sketch

Using the shift operator $E = 1 + \Delta$:

$$f(x_0 + sh) = E^s f_0 = (1 + \Delta)^s f_0$$

Expanding using the binomial theorem (generalized for non-integer $s$):

$$(1 + \Delta)^s = \sum_{k=0}^{\infty} \binom{s}{k} \Delta^k$$

This gives Newton's forward formula directly.

---

## 6. Real-World Applications

| Application | Example |
|-------------|---------|
| Population prediction | Estimate in-between census years |
| Engineering tables | Look up values between tabulated entries |
| Signal processing | Reconstruct signals from samples |
| Financial data | Estimate stock prices between trading days |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Formula** | $f(x) = \sum_{k=0}^{n} \binom{s}{k} \Delta^k f_0$ |
| **$s$ parameter** | $s = (x - x_0)/h$ |
| **Best for** | Interpolation near the beginning of table |
| **Requires** | Equally spaced data |
| **Uses** | Forward differences from top of table |
| **Error** | $\binom{s}{n+1} \Delta^{n+1} f_0$ |

---

## Quick Revision Questions

1. **State Newton's forward difference interpolation formula.**

2. **Given $f(0) = 1, f(0.5) = 1.6487, f(1.0) = 2.7183, f(1.5) = 4.4817$, find $f(0.25)$ using Newton's forward formula.**

3. **Derive Newton's forward formula from the operator relation $E = 1 + \Delta$.**

4. **Why is Newton's forward formula most accurate near the beginning of the table?**

5. **What determines how many terms to include in the formula?**

6. **Estimate the error when interpolating $f(0.3)$ from a table at $x = 0, 0.1, 0.2, 0.3, 0.4$ using a quadratic interpolation.**

---

[← Previous: Finite Differences](01-finite-differences.md) | [Next: Newton's Backward Difference →](03-newton-backward.md)
