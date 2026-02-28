# Chapter 6.5: Weddle's Rule

[← Previous: Boole's Rule](04-booles-rule.md) | [Next: Romberg Integration →](06-romberg-integration.md)

---

## Overview

**Weddle's Rule** uses 7 equally spaced points (6 subintervals) and is popular in examinations because it has a simple weight pattern: $1, 5, 1, 6, 1, 5, 1$. It provides $O(h^8)$ accuracy in its exact form.

---

## 1. The Formula

$$\boxed{\int_{x_0}^{x_6} f(x)\,dx \approx \frac{3h}{10}[f_0 + 5f_1 + f_2 + 6f_3 + f_4 + 5f_5 + f_6]}$$

**Weights**: $1, 5, 1, 6, 1, 5, 1$ — easy to remember!

**Multiplier**: $\frac{3h}{10}$

---

## 2. Origin

Weddle's rule is a **simplified** version of the exact 7-point Newton-Cotes formula. The exact 7-point NC weights are $\frac{h}{140}(41, 216, 27, 272, 27, 216, 41)$; Weddle rounded these to $\frac{3h}{10}(1, 5, 1, 6, 1, 5, 1)$.

The simplification introduces a tiny additional error but keeps the formula very easy to apply by hand.

---

## 3. Error

$$E = -\frac{h^9}{1400}f^{(8)}(\xi)$$

- Order: $O(h^8)$ (very high accuracy)
- Exact for polynomials of degree $\leq 5$ (the simplified version)
- The exact 7-point NC is exact for degree $\leq 7$

---

## 4. Worked Example

Evaluate $\int_0^6 \frac{dx}{1+x}$ with $h = 1$ (7 points):

| $x$ | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|-----|---|---|---|---|---|---|---|
| $f(x) = \frac{1}{1+x}$ | 1.0000 | 0.5000 | 0.3333 | 0.2500 | 0.2000 | 0.1667 | 0.1429 |

$$I \approx \frac{3(1)}{10}[1.0000 + 5(0.5000) + 0.3333 + 6(0.2500) + 0.2000 + 5(0.1667) + 0.1429]$$

$$= 0.3\left[1.0000 + 2.5000 + 0.3333 + 1.5000 + 0.2000 + 0.8335 + 0.1429\right]$$

$$= 0.3 \times 6.5097 = 1.9529$$

**Exact value:** $\ln 7 = 1.9459$

**Error:** $0.0070$ (with only 7 points over a wide range — quite good!)

---

## 5. Composite Weddle's Rule

For $n$ subintervals ($n$ must be a multiple of 6):

Apply Weddle's rule to each group of 6 subintervals and sum the results.

$$\int_a^b f(x)\,dx = \sum_{k=0}^{n/6-1} \frac{3h}{10}[f_{6k} + 5f_{6k+1} + f_{6k+2} + 6f_{6k+3} + f_{6k+4} + 5f_{6k+5} + f_{6k+6}]$$

---

## 6. Comparison with Other Rules

| Rule | Points | Multiplier | Weights | Error |
|------|--------|-----------|---------|-------|
| Trapezoidal | 2 | $h/2$ | $1,1$ | $O(h^2)$ |
| Simpson 1/3 | 3 | $h/3$ | $1,4,1$ | $O(h^4)$ |
| Simpson 3/8 | 4 | $3h/8$ | $1,3,3,1$ | $O(h^4)$ |
| Boole | 5 | $2h/45$ | $7,32,12,32,7$ | $O(h^6)$ |
| **Weddle** | **7** | **$3h/10$** | **$1,5,1,6,1,5,1$** | **$O(h^8)$** |

---

## Summary Table

| Property | Value |
|----------|-------|
| **Points** | 7 (6 subintervals) |
| **Multiplier** | $\frac{3h}{10}$ |
| **Weights** | $1, 5, 1, 6, 1, 5, 1$ |
| **Error** | $O(h^8)$ |
| **Exact for** | Degree $\leq 5$ (simplified) |
| **Composite** | $n$ must be multiple of 6 |

---

## Quick Revision Questions

1. **State Weddle's rule with its multiplier and weights.**

2. **Evaluate $\int_0^6 x^2\,dx$ using Weddle's rule with $h=1$. Verify against the exact answer.**

3. **How is Weddle's rule related to the 7-point Newton-Cotes formula?**

4. **What constraint does $n$ have for the composite Weddle's rule?**

5. **Compare the accuracy of Weddle's rule with Simpson's 1/3 rule.**

6. **Why is Weddle's rule popular in examinations despite not being the most accurate 7-point formula?**

---

[← Previous: Boole's Rule](04-booles-rule.md) | [Next: Romberg Integration →](06-romberg-integration.md)
