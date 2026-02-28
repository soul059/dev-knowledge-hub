# Chapter 6.3: Simpson's 3/8 Rule

[← Previous: Simpson's 1/3 Rule](02-simpsons-one-third.md) | [Next: Boole's Rule →](04-booles-rule.md)

---

## Overview

**Simpson's 3/8 Rule** fits a **cubic polynomial** through four equally spaced points. It uses 3 subintervals per application and is particularly useful when $n$ is a multiple of 3, or to handle leftovers when $n$ is odd (pairing with the 1/3 rule).

---

## 1. Single-Interval Formula

Over 3 subintervals of size $h$:

$$\boxed{\int_{x_0}^{x_3} f(x)\,dx \approx \frac{3h}{8}[f_0 + 3f_1 + 3f_2 + f_3]}$$

---

## 2. Composite Simpson's 3/8 Rule

Divide $[a,b]$ into $n$ subintervals ($n$ **must be a multiple of 3**):

$$\int_a^b f(x)\,dx \approx \frac{3h}{8}\left[f_0 + 3f_1 + 3f_2 + 2f_3 + 3f_4 + 3f_5 + 2f_6 + \cdots + f_n\right]$$

**Weight pattern**: $1, 3, 3, 2, 3, 3, 2, \ldots, 3, 3, 1$

---

## 3. Error Analysis

### Single application

$$E = -\frac{3h^5}{80}f^{(4)}(\xi)$$

### Composite rule

$$\boxed{E = -\frac{(b-a)h^4}{80}f^{(4)}(\xi)}$$

- Same **order** $O(h^4)$ as Simpson's 1/3 rule
- But slightly **larger constant** ($1/80$ vs $1/180$)
- Also exact for polynomials of degree $\leq 3$

---

## 4. Worked Example

Evaluate $\int_0^{0.9} \sqrt{1+x^2}\,dx$ with $n = 3$, $h = 0.3$.

| $x$ | 0.0 | 0.3 | 0.6 | 0.9 |
|-----|-----|-----|-----|-----|
| $f(x)$ | 1.0000 | 1.0440 | 1.1662 | 1.3454 |

$$I \approx \frac{3(0.3)}{8}[1.0000 + 3(1.0440) + 3(1.1662) + 1.3454]$$

$$= \frac{0.9}{8}[1.0000 + 3.1320 + 3.4986 + 1.3454]$$

$$= 0.1125 \times 8.9760 = 1.0098$$

---

## 5. Combining 1/3 and 3/8 Rules

When $n$ doesn't satisfy either rule alone:

| $n$ | Strategy |
|-----|----------|
| Even | Use Simpson's 1/3 for all |
| Multiple of 3 | Use Simpson's 3/8 for all |
| $n = 5$ | 3/8 for first 3 + 1/3 for last 2 |
| $n = 7$ | 3/8 for first 3 + 1/3 for last 4 |
| General odd | 3/8 for 3 + 1/3 for remaining (even) |

---

## 6. Comparison: 1/3 vs 3/8

| Property | Simpson's 1/3 | Simpson's 3/8 |
|----------|--------------|---------------|
| Polynomial degree | 2 (parabola) | 3 (cubic) |
| Points per panel | 3 (2 intervals) | 4 (3 intervals) |
| Weights | 1, 4, 1 | 1, 3, 3, 1 |
| Error constant | $1/180$ | $1/80$ |
| Exact for | $\leq$ degree 3 | $\leq$ degree 3 |
| Preferred when | $n$ even | $n$ multiple of 3 |

> **Simpson's 1/3 is generally preferred** because it has a smaller error constant.

---

## Summary Table

| Property | Value |
|----------|-------|
| **Type** | Closed Newton-Cotes, degree 3 |
| **Formula** | $\frac{3h}{8}[f_0 + 3f_1 + 3f_2 + f_3]$ |
| **Requirement** | $n$ must be a **multiple of 3** |
| **Error** | $-\frac{(b-a)h^4}{80}f^{(4)}(\xi)$ |
| **Order** | $O(h^4)$ |
| **Exact for** | Polynomials of degree $\leq 3$ |

---

## Quick Revision Questions

1. **State Simpson's 3/8 rule and its error term.**

2. **Evaluate $\int_0^3 x^3\,dx$ using the 3/8 rule with $h=1$. Verify it's exact.**

3. **Why is Simpson's 1/3 rule generally preferred over the 3/8 rule?**

4. **When is Simpson's 3/8 rule particularly useful?**

5. **How can you handle $n = 7$ subintervals using a combination of rules?**

6. **Compare the error constants of the 1/3 and 3/8 rules.**

---

[← Previous: Simpson's 1/3 Rule](02-simpsons-one-third.md) | [Next: Boole's Rule →](04-booles-rule.md)
