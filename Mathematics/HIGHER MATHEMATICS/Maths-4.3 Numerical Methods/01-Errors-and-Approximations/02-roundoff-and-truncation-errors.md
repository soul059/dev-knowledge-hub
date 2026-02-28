# Chapter 1.2: Round-off and Truncation Errors

[← Previous: Types of Errors](01-types-of-errors.md) | [Next: Significant Figures →](03-significant-figures.md)

---

## Overview

The two most fundamental sources of error in numerical computation are **round-off errors** (from finite machine arithmetic) and **truncation errors** (from replacing infinite processes with finite ones). Understanding their distinct origins and behaviors is essential for choosing and analyzing numerical methods.

---

## 1. Round-off Errors

### Definition

**Round-off error** arises because computers use a finite number of digits (bits) to represent real numbers. Most real numbers cannot be stored exactly.

### How Computers Store Numbers

A floating-point number in base 10 is stored as:

$$x = \pm d_1.d_2 d_3 \ldots d_t \times 10^e$$

where $t$ is the number of significant digits (precision) and $e$ is the exponent.

```
  ┌─────────────────────────────────────────────────────────────┐
  │          IEEE 754 Single Precision (32 bits)                 │
  │                                                              │
  │  ┌───┬──────────┬───────────────────────────┐               │
  │  │ S │ Exponent │        Mantissa            │               │
  │  │1b │   8 bits  │       23 bits              │               │
  │  └───┴──────────┴───────────────────────────┘               │
  │                                                              │
  │  S = sign bit (0 = positive, 1 = negative)                  │
  │  Exponent: biased by 127                                     │
  │  Mantissa: fractional part (leading 1 implied)               │
  │                                                              │
  │  Precision: ~7 decimal digits                                │
  │  Range: ~±3.4 × 10^38                                       │
  └─────────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────────┐
  │          IEEE 754 Double Precision (64 bits)                  │
  │                                                              │
  │  ┌───┬──────────────┬──────────────────────────────────┐    │
  │  │ S │   Exponent    │           Mantissa                │    │
  │  │1b │   11 bits     │          52 bits                  │    │
  │  └───┴──────────────┴──────────────────────────────────┘    │
  │                                                              │
  │  Precision: ~15-16 decimal digits                            │
  │  Range: ~±1.7 × 10^308                                      │
  └─────────────────────────────────────────────────────────────┘
```

### Rounding vs. Chopping

Two strategies to fit a number into $t$ digits:

**Chopping (Truncation)**: Simply drop all digits beyond position $t$.

$$\pi = 3.14159265... \xrightarrow{\text{chop to 4 digits}} 3.141$$

**Rounding**: Look at digit $d_{t+1}$; if $\geq 5$, round up.

$$\pi = 3.14159265... \xrightarrow{\text{round to 4 digits}} 3.142$$

| Method | Result (4 digits) | Absolute Error |
|--------|-------------------|----------------|
| Chopping | $3.141$ | $0.00059...$ |
| Rounding | $3.142$ | $0.00041...$ |

**Rounding is generally more accurate** — maximum error is $0.5 \times 10^{1-t}$ compared to $10^{1-t}$ for chopping.

### Catastrophic Cancellation

When two nearly equal numbers are subtracted, significant digits are lost:

```
  Example: 5-digit arithmetic

    a = 0.12345 × 10^1  =  1.2345
    b = 0.12344 × 10^1  =  1.2344

    a - b = 0.00001 × 10^1  =  0.1 × 10^-3

  We started with 5 significant digits but the result
  has only 1 significant digit!

  ┌─────────────────────────────────────────────┐
  │  BEFORE subtraction:  5 significant digits  │
  │  AFTER subtraction:   1 significant digit   │
  │  Lost: 4 digits of precision!               │
  └─────────────────────────────────────────────┘
```

### Accumulation of Round-off Errors

In long computations with many operations, round-off errors can accumulate:

```
  Step 1:  small error e₁
  Step 2:  small error e₂
    ...
  Step n:  small error eₙ

  Total error ≈ e₁ + e₂ + ... + eₙ  (worst case)
             ≈ √(e₁² + e₂² + ... + eₙ²)  (statistical average)
```

---

## 2. Truncation Errors

### Definition

**Truncation error** arises when an infinite mathematical process is approximated by a finite one. This is a property of the **method**, not the machine.

### Classic Example: Taylor Series

The exponential function has the exact expansion:

$$e^x = 1 + x + \frac{x^2}{2!} + \frac{x^3}{3!} + \frac{x^4}{4!} + \cdots$$

If we truncate after $n$ terms:

$$e^x \approx \sum_{k=0}^{n-1} \frac{x^k}{k!}$$

The truncation error is the remainder:

$$R_n = \frac{x^n}{n!} + \frac{x^{n+1}}{(n+1)!} + \cdots$$

```
  Approximating e^1 with increasing terms:
  
  Terms   Approximation    Absolute Error
  ─────   ─────────────    ──────────────
    1       1.000000        1.718282
    2       2.000000        0.718282
    3       2.500000        0.218282
    4       2.666667        0.051615
    5       2.708333        0.009948
    6       2.716667        0.001615
    7       2.718056        0.000227
    8       2.718254        0.000028    ◄── convergence!
```

### Truncation Error in Derivatives

The derivative $f'(x)$ is defined as:

$$f'(x) = \lim_{h \to 0} \frac{f(x+h) - f(x)}{h}$$

Numerically, we use a **finite** $h$:

$$f'(x) \approx \frac{f(x+h) - f(x)}{h}$$

By Taylor expansion:

$$f(x+h) = f(x) + hf'(x) + \frac{h^2}{2}f''(x) + \cdots$$

So:

$$\frac{f(x+h) - f(x)}{h} = f'(x) + \frac{h}{2}f''(x) + \cdots$$

The truncation error is $O(h)$ — it decreases linearly with $h$.

---

## 3. Round-off vs. Truncation: The Fundamental Trade-off

```
  Error
    │
    │ ╲                          ╱
    │   ╲  Round-off error     ╱
    │     ╲  (increases as   ╱
    │       ╲  h → 0)      ╱
    │         ╲           ╱
    │           ╲       ╱
    │    Total    ╲   ╱  Truncation error
    │    Error  ────╳────  (decreases as h → 0)
    │             ╱  ╲
    │           ╱      ╲
    │         ╱          ╲
    │       ╱              ╲
    │     ╱                  ╲
    ├─────────────────────────────────── h
    │         h_opt
    │    (optimal step size)
```

| Aspect | Round-off Error | Truncation Error |
|--------|----------------|-----------------|
| **Source** | Finite machine precision | Finite approximation of math |
| **Depends on** | Computer hardware, word length | Algorithm, step size |
| **Behavior with smaller $h$** | Increases (more operations) | Decreases |
| **Can be eliminated?** | No (inherent to digital computing) | Reduce by taking more terms |
| **Controlled by** | Using higher precision arithmetic | Using better algorithms |

---

## 4. Order of Approximation (Big-O Notation)

We say a method has truncation error $O(h^p)$ if:

$$|E_T| \leq C \cdot h^p \quad \text{for small } h$$

where $C$ is a constant independent of $h$.

### What This Means Practically

If a method is $O(h^2)$ and you halve $h$:
- Error reduces by factor $\left(\frac{1}{2}\right)^2 = \frac{1}{4}$

| Order | Name | Halving $h$ reduces error by |
|-------|------|------------------------------|
| $O(h)$ | First order | $\times \frac{1}{2}$ |
| $O(h^2)$ | Second order | $\times \frac{1}{4}$ |
| $O(h^3)$ | Third order | $\times \frac{1}{8}$ |
| $O(h^4)$ | Fourth order | $\times \frac{1}{16}$ |

---

## 5. Worked Examples

### Example 1: Truncation Error of Forward Difference

Compute $f'(1)$ for $f(x) = x^3$ using forward difference with $h = 0.1$ and $h = 0.01$.

True value: $f'(x) = 3x^2 \Rightarrow f'(1) = 3$

| $h$ | $\frac{f(1+h) - f(1)}{h}$ | Approx | Absolute Error |
|-----|---------------------------|--------|----------------|
| 0.1 | $\frac{1.331 - 1}{0.1}$ | $3.31$ | $0.31$ |
| 0.01 | $\frac{1.030301 - 1}{0.01}$ | $3.0301$ | $0.0301$ |

Halving $h$ from $0.1$ to $0.01$ (factor 10) reduced error by factor ~10, confirming $O(h)$.

### Example 2: Round-off in Subtraction

Compute $f(x) = \frac{1 - \cos x}{x^2}$ at $x = 0.0001$ with 10-digit arithmetic.

$$\cos(0.0001) = 0.9999999950000...\text{ (exact)}$$

With 10 digits: $\cos(0.0001) = 0.9999999950$

$$1 - 0.9999999950 = 0.0000000050 = 5.0 \times 10^{-9}$$

$$\frac{5.0 \times 10^{-9}}{(10^{-4})^2} = \frac{5.0 \times 10^{-9}}{10^{-8}} = 0.50$$

True value: $\frac{1 - \cos x}{x^2} \to \frac{1}{2} = 0.5$ as $x \to 0$.

Here we got lucky, but with fewer digits, cancellation would cause severe error. A **reformulation** avoids the problem:

$$\frac{1 - \cos x}{x^2} = \frac{\sin^2 x}{x^2(1 + \cos x)} = \frac{1}{1 + \cos x} \cdot \left(\frac{\sin x}{x}\right)^2$$

---

## 6. Strategies to Minimize Errors

| Strategy | Addresses | Example |
|----------|-----------|---------|
| Use higher-order methods | Truncation | Simpson's rule vs. Trapezoidal |
| Use double precision | Round-off | 64-bit vs. 32-bit |
| Reformulate expressions | Round-off (cancellation) | $\sqrt{x+1} - \sqrt{x} = \frac{1}{\sqrt{x+1}+\sqrt{x}}$ |
| Use smaller step size | Truncation | $h = 0.01$ vs. $h = 0.1$ |
| Compensated summation | Round-off (accumulation) | Kahan summation algorithm |

---

## 7. Real-World Applications

- **Weather Forecasting**: Truncation errors in PDEs can cause incorrect predictions days ahead.
- **Financial Systems**: Round-off in currency calculations can cause discrepancies worth millions.
- **Spacecraft Trajectory**: The Patriot missile failure (1991) was due to round-off in time tracking.
- **Structural Analysis**: FEA meshes trade truncation error (coarse mesh) vs. round-off (very fine mesh).

---

## Summary Table

| Concept | Description | Formula/Bound |
|---------|-------------|---------------|
| Round-off Error | Due to finite precision arithmetic | Machine epsilon $\approx 10^{-16}$ (double) |
| Truncation Error | Due to finite approximation | $O(h^p)$ for step size $h$, order $p$ |
| Chopping Error | Drop extra digits | $\leq 10^{1-t}$ |
| Rounding Error | Round last digit | $\leq 0.5 \times 10^{1-t}$ |
| Catastrophic Cancellation | Loss of significant digits in subtraction | Reformulate to avoid |
| Optimal Step Size | Balances round-off and truncation | Problem-dependent |

---

## Quick Revision Questions

1. **Distinguish between round-off error and truncation error with one example each.**

2. **Why does reducing the step size $h$ indefinitely NOT always improve accuracy? Explain with the error trade-off diagram.**

3. **Compute the truncation error when $e^{0.5}$ is approximated by $1 + 0.5 + 0.5^2/2! + 0.5^3/3!$. What is the order of the next neglected term?**

4. **What is catastrophic cancellation? Give an example and show how to reformulate the expression to avoid it.**

5. **A method is $O(h^2)$. If the error is $0.04$ for $h = 0.1$, estimate the error for $h = 0.05$.**

6. **Explain why rounding is preferred over chopping. What is the maximum error bound for each in $t$-digit arithmetic?**

---

[← Previous: Types of Errors](01-types-of-errors.md) | [Next: Significant Figures →](03-significant-figures.md)
