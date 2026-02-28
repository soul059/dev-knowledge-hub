# Chapter 1.3: Significant Figures

[← Previous: Round-off and Truncation Errors](02-roundoff-and-truncation-errors.md) | [Next: Error Propagation →](04-error-propagation.md)

---

## Overview

Significant figures (or significant digits) provide a practical way to express the precision of a numerical value. They tell us **how many digits we can trust** in a computed or measured result.

---

## 1. Definition

A number $X_a$ is said to approximate the true value $X$ to **$n$ significant figures** if the absolute error satisfies:

$$|X - X_a| \leq \frac{1}{2} \times 10^{m-n+1}$$

where $m$ is the order of magnitude of $X$ (i.e., $X$ is of the form $d_1.d_2d_3\ldots \times 10^m$ with $d_1 \neq 0$).

### Intuitive Rule

A digit is **significant** if it is reliably known. The number of significant digits equals the count of digits from the first nonzero digit to the last reliably known digit.

---

## 2. Rules for Counting Significant Figures

| Rule | Example | Significant Figures |
|------|---------|:-------------------:|
| All nonzero digits are significant | $4573$ | 4 |
| Zeros between nonzero digits are significant | $3007$ | 4 |
| Leading zeros are NOT significant | $0.00452$ | 3 |
| Trailing zeros after decimal point ARE significant | $2.3400$ | 5 |
| Trailing zeros in integers are **ambiguous** | $1500$ | 2, 3, or 4 |

### Resolving Ambiguity with Scientific Notation

| Representation | Significant Figures |
|---------------|:-------------------:|
| $1.5 \times 10^3$ | 2 |
| $1.50 \times 10^3$ | 3 |
| $1.500 \times 10^3$ | 4 |

```
  Number:    0  .  0  0  4  5  2  0  0
             │     │  │  │  │  │  │  │
             │     │  │  ╰──┤──┤──┤──╯
             │     │  │     │  Significant (5)
             ╰─────╰──╯     │
              Not significant│
              (leading zeros)│
                             First nonzero digit
                             starts the count
```

---

## 3. Significant Figures and Error Bound

If $X_a$ has $n$ significant figures, we can bound the relative error:

$$E_r \leq \frac{1}{2} \times 10^{1-n} \times \frac{1}{d_1}$$

where $d_1$ is the leading significant digit.

### Practical Bound

$$E_r \leq 5 \times 10^{-n}$$

This means:
- 3 significant figures → relative error $\leq 0.5\%$
- 5 significant figures → relative error $\leq 0.005\%$
- 7 significant figures → relative error $\leq 0.00005\%$

---

## 4. Significant Figures in Arithmetic Operations

### Addition and Subtraction

**Rule**: The result should have the same number of **decimal places** as the operand with the fewest decimal places.

| Operation | Result | Rounded |
|-----------|--------|---------|
| $23.456 + 1.2$ | $24.656$ | $24.7$ (1 decimal place) |
| $100.0 - 0.0234$ | $99.9766$ | $100.0$ (1 decimal place) |

### Multiplication and Division

**Rule**: The result should have the same number of **significant figures** as the operand with the fewest significant figures.

| Operation | Result | Rounded |
|-----------|--------|---------|
| $3.14 \times 2.1$ | $6.594$ | $6.6$ (2 sig figs) |
| $45.678 / 2.3$ | $19.860...$ | $20.$ (2 sig figs) |

---

## 5. Loss of Significant Figures

### In Subtraction (Cancellation)

```
  Example:  3-digit arithmetic

    a = 4.56       (3 significant digits)
    b = 4.54       (3 significant digits)
    
    a - b = 0.02   (1 significant digit!)

  ┌─────────────────────────────────────────┐
  │  Start: 3 significant digits each      │
  │  Result: only 1 significant digit       │
  │  Lost: 2 significant digits!            │
  └─────────────────────────────────────────┘
```

This is why numerical methods try to avoid subtracting nearly equal quantities.

### In Division by Small Numbers

Dividing by a number close to zero amplifies existing errors:

$$\text{If } X_a = X + \epsilon, \text{ then } \frac{1}{X_a} \approx \frac{1}{X} - \frac{\epsilon}{X^2}$$

The error $\frac{\epsilon}{X^2}$ becomes large when $X$ is small.

---

## 6. Correct Significant Digits

We say $X_a$ is **correct to $n$ significant figures** if:

$$\left|\frac{X - X_a}{X}\right| \leq 5 \times 10^{-n}$$

### Example

Is $X_a = 3.1416$ correct to 4 significant figures for $\pi$?

$$E_r = \frac{|3.14159265 - 3.1416|}{3.14159265} = \frac{0.00000735}{3.14159265} = 2.34 \times 10^{-6}$$

Check: $5 \times 10^{-4} = 0.0005$

Since $2.34 \times 10^{-6} < 5 \times 10^{-4}$, **yes**, it's correct to 4 significant figures.

Actually, since $2.34 \times 10^{-6} < 5 \times 10^{-5}$, it's even correct to **5 significant figures**.

---

## 7. Significant Figures in Iterative Methods

In iterative numerical methods, we often use the **Scarborough criterion** to determine when to stop:

$$\left|\frac{x_n - x_{n-1}}{x_n}\right| < 0.5 \times 10^{2-n}$$

where $n$ is the number of significant figures desired.

```
  Iterative convergence example (finding √2):

  Iteration    x_n         |x_n - x_{n-1}|/|x_n|    Sig. Figs
  ─────────    ─────────   ──────────────────────    ─────────
     0         1.000000     ─                         ─
     1         1.500000     0.3333                    1
     2         1.416667     0.0588                    2
     3         1.414216     0.00173                   3
     4         1.414214     0.0000015                 6
     5         1.414214     < 10^-12                  12+
```

---

## 8. Real-World Applications

| Application | Required Sig. Figs | Why |
|-------------|:-----------------:|-----|
| Engineering drawings | 3-4 | Manufacturing tolerances |
| Chemical analysis | 4-5 | Reagent purity levels |
| Astronomical distances | 2-3 | Measurement limitations |
| Atomic physics constants | 8-12 | Precision experiments |
| GPS coordinates | 7-8 | Meter-level accuracy |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Significant figures | Count of reliably known digits from first nonzero |
| Leading zeros | Not significant |
| Trailing zeros (after decimal) | Significant |
| Trailing zeros (integer) | Ambiguous — use scientific notation |
| Addition/Subtraction | Match decimal places |
| Multiplication/Division | Match significant figures |
| Cancellation | Subtraction of near-equal numbers loses sig. figs |
| Scarborough criterion | Stopping rule for iterative methods |
| $n$ sig. figs ↔ error | $E_r \leq 5 \times 10^{-n}$ |

---

## Quick Revision Questions

1. **How many significant figures are in: (a) 0.00340, (b) 20500, (c) 1.0020, (d) $3.00 \times 10^5$?**

2. **A number has 5 significant figures. What is the maximum relative error?**

3. **Explain why $a - b$ can have fewer significant figures than $a$ or $b$. Give a numerical example.**

4. **Using the Scarborough criterion, how many iterations are needed for 4 significant figures if the relative approximate error sequence is: 0.5, 0.08, 0.003, 0.0001, 0.000002?**

5. **Round $2.34565$ to (a) 3, (b) 4, (c) 5 significant figures.**

6. **Why is scientific notation preferred for expressing significant figures? Illustrate with the number 4500.**

---

[← Previous: Round-off and Truncation Errors](02-roundoff-and-truncation-errors.md) | [Next: Error Propagation →](04-error-propagation.md)
