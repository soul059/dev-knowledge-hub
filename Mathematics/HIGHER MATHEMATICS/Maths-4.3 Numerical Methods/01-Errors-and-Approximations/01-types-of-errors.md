# Chapter 1.1: Types of Errors

[← Back to README](../README.md) | [Next: Round-off and Truncation Errors →](02-roundoff-and-truncation-errors.md)

---

## Overview

Every numerical computation introduces some degree of error. Understanding, classifying, and quantifying errors is the foundation of numerical methods — without this, we cannot judge whether a computed answer is trustworthy.

This chapter introduces the three fundamental error measures: **absolute error**, **relative error**, and **percentage error**.

---

## 1. Why Errors Occur in Numerical Computation

```
  ┌─────────────────────────────────────────────────────┐
  │              SOURCES OF ERROR                        │
  │                                                      │
  │   Mathematical    ──►  Modelling Error               │
  │   Model                (idealization of reality)     │
  │        │                                             │
  │        ▼                                             │
  │   Numerical       ──►  Truncation Error              │
  │   Method               (finite approximation)       │
  │        │                                             │
  │        ▼                                             │
  │   Computer        ──►  Round-off Error               │
  │   Arithmetic           (finite word length)          │
  │        │                                             │
  │        ▼                                             │
  │   Human           ──►  Blunders / Data Errors        │
  │   Input                (measurement, typos)          │
  └─────────────────────────────────────────────────────┘
```

---

## 2. True Value vs. Approximate Value

Let:
- $X$ = **true (exact) value** of a quantity
- $X_a$ = **approximate (computed) value**

The **error** is the difference between these two:

$$E = X - X_a$$

> **Note**: The true value is often unknown — that's why we compute! So in practice we work with estimates and bounds on the error.

---

## 3. Absolute Error

### Definition

The **absolute error** is the magnitude of the difference between the true value and the approximate value:

$$E_a = |X - X_a|$$

### Properties

| Property | Detail |
|----------|--------|
| Always non-negative | $E_a \geq 0$ |
| Has same units as $X$ | If $X$ is in meters, $E_a$ is in meters |
| Does not indicate severity alone | An error of 1 cm is huge for a watch part, tiny for a building |

### Example

If the true value of $\pi = 3.14159265...$ and we approximate $\pi_a = 3.14$:

$$E_a = |3.14159265... - 3.14| = 0.00159265...$$

---

## 4. Relative Error

### Definition

The **relative error** normalizes the absolute error by the true value:

$$E_r = \frac{|X - X_a|}{|X|} = \frac{E_a}{|X|}, \quad X \neq 0$$

### Why Relative Error Matters

Absolute error alone is misleading:

```
  Case A:  True = 1000,  Approx = 999    →  Ea = 1     Er = 0.001
  Case B:  True = 2,     Approx = 1      →  Ea = 1     Er = 0.5

  Same absolute error, but Case B is FAR worse!
  ──────────────────────────────────────────────────
  Relative error captures the "significance" of the error.
```

### Example

For $\pi_a = 3.14$:

$$E_r = \frac{0.00159265...}{3.14159265...} \approx 0.000507$$

---

## 5. Percentage Error

### Definition

Simply the relative error expressed as a percentage:

$$E_p = E_r \times 100\% = \frac{|X - X_a|}{|X|} \times 100\%$$

### Example

For $\pi_a = 3.14$:

$$E_p = 0.000507 \times 100\% \approx 0.0507\%$$

---

## 6. Error Bound

When the true value $X$ is unknown, we often state an **error bound** $\epsilon$ such that:

$$|X - X_a| \leq \epsilon$$

This means the true value lies in the interval:

$$X \in [X_a - \epsilon, \; X_a + \epsilon]$$

```
       X_a - ε          X_a          X_a + ε
  ───────┤───────────────┼───────────────┤───────
         ◄──── error bound ε ────────────►
                    True value X lies
                    somewhere in here
```

---

## 7. Worked Examples

### Example 1: Measuring a Rod

A steel rod has a true length of $X = 25.462$ cm. A measurement yields $X_a = 25.5$ cm.

| Error Type | Calculation | Result |
|-----------|-------------|--------|
| Absolute | $\|25.462 - 25.5\| = 0.038$ | $0.038$ cm |
| Relative | $0.038 / 25.462 = 0.001493$ | $0.001493$ |
| Percentage | $0.001493 \times 100$ | $0.1493\%$ |

### Example 2: Comparing Two Approximations

Which is a better approximation?
- $X_1 = 0.004$ approximated as $X_{1a} = 0.003$
- $X_2 = 4.000$ approximated as $X_{2a} = 3.997$

| Quantity | $X_1$ | $X_2$ |
|----------|-------|-------|
| Absolute Error | $0.001$ | $0.003$ |
| Relative Error | $0.001/0.004 = 0.25$ | $0.003/4.000 = 0.00075$ |
| Percentage Error | $25\%$ | $0.075\%$ |

**Conclusion**: Despite having a larger absolute error, $X_2$ is far more accurate in relative terms. **Always compare using relative or percentage error.**

### Example 3: Error in Function Evaluation

Compute $f(x) = \sqrt{x}$ at $x = 50$.

True value: $\sqrt{50} = 7.07106781...$

Approximation: $\sqrt{50} \approx 7.07$

$$E_a = |7.07106781 - 7.07| = 0.00106781$$

$$E_r = \frac{0.00106781}{7.07106781} = 0.000151$$

$$E_p = 0.0151\%$$

---

## 8. Relationship Between Error Types

```
  ┌──────────────┐       ÷ |X|        ┌──────────────┐       × 100%      ┌──────────────┐
  │   Absolute   │ ──────────────────► │   Relative   │ ────────────────► │  Percentage  │
  │    Error     │                     │    Error     │                    │    Error     │
  │   Ea = |X-Xa|│ ◄────────────────── │  Er = Ea/|X| │ ◄──────────────── │  Ep = Er×100 │
  └──────────────┘       × |X|        └──────────────┘       ÷ 100%      └──────────────┘
```

---

## 9. Real-World Applications

| Field | Measurement | Typical Error Tolerance |
|-------|------------|------------------------|
| Civil Engineering | Bridge length | Absolute: ±1 cm |
| Pharmacy | Drug dosage | Relative: < 0.1% |
| GPS Navigation | Position | Absolute: ±3 m |
| Financial Computing | Interest rates | Relative: < 0.001% |
| Scientific Research | Physical constants | Percentage: < 0.0001% |

---

## Summary Table

| Concept | Formula | Key Point |
|---------|---------|-----------|
| True Error | $E = X - X_a$ | Can be positive or negative |
| Absolute Error | $E_a = \|X - X_a\|$ | Magnitude only; same units as $X$ |
| Relative Error | $E_r = E_a / \|X\|$ | Dimensionless; shows significance |
| Percentage Error | $E_p = E_r \times 100\%$ | Easy to interpret |
| Error Bound | $\|X - X_a\| \leq \epsilon$ | Guarantees interval for true value |

---

## Quick Revision Questions

1. **Define absolute error and relative error. Why is relative error often preferred?**

2. **The true value of a quantity is 5.6745 and its approximate value is 5.67. Calculate all three types of errors.**

3. **Two measurements have absolute errors of 0.5 and 0.05 respectively. Can you determine which is more accurate? Why or why not?**

4. **If the relative error in measuring the radius of a circle is 0.02, what can you say about the error in computing its area $A = \pi r^2$?**

5. **A calculator gives $e = 2.718$. The true value is $e = 2.718281828...$. Find the absolute, relative, and percentage errors.**

6. **Explain with an example why absolute error alone is insufficient to judge the quality of an approximation.**

---

[← Back to README](../README.md) | [Next: Round-off and Truncation Errors →](02-roundoff-and-truncation-errors.md)
