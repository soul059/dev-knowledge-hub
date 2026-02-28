# Chapter 7.2: Homogeneous Recurrence Relations

[← Previous: Linear Recurrence Relations](01-linear-recurrence-relations.md) | [Back to README](../README.md) | [Next: Non-Homogeneous Recurrence →](03-non-homogeneous-recurrence.md)

---

## 📋 Chapter Overview

A **linear homogeneous recurrence relation with constant coefficients** has the form $a_n = c_1 a_{n-1} + c_2 a_{n-2} + \cdots + c_k a_{n-k}$. The **characteristic equation** method provides a systematic way to find closed-form solutions.

---

## 1. Standard Form

$$a_n - c_1 a_{n-1} - c_2 a_{n-2} - \cdots - c_k a_{n-k} = 0$$

Requirements:
- **Linear** in $a_{n}, a_{n-1}, \ldots$
- **Homogeneous** — right side is 0 (no $f(n)$ term)
- **Constant coefficients** — all $c_i$ are constants

---

## 2. The Characteristic Equation

**Key idea:** Try solutions of the form $a_n = r^n$.

Substituting into $a_n = c_1 a_{n-1} + c_2 a_{n-2} + \cdots + c_k a_{n-k}$:

$$r^n = c_1 r^{n-1} + c_2 r^{n-2} + \cdots + c_k r^{n-k}$$

Divide by $r^{n-k}$:

$$r^k - c_1 r^{k-1} - c_2 r^{k-2} - \cdots - c_k = 0$$

This is the **characteristic equation** and its roots are the **characteristic roots**.

---

## 3. Case 1: Distinct Real Roots

If the characteristic equation has $k$ **distinct** roots $r_1, r_2, \ldots, r_k$, the general solution is:

$$\boxed{a_n = A_1 r_1^n + A_2 r_2^n + \cdots + A_k r_k^n}$$

where $A_1, A_2, \ldots, A_k$ are determined by initial conditions.

### Example: Second-Order, Distinct Roots

Solve $a_n = 5a_{n-1} - 6a_{n-2}$, with $a_0 = 1, a_1 = 4$.

**Step 1.** Characteristic equation: $r^2 - 5r + 6 = 0$

$$(r-2)(r-3) = 0 \implies r_1 = 2, \; r_2 = 3$$

**Step 2.** General solution: $a_n = A \cdot 2^n + B \cdot 3^n$

**Step 3.** Apply initial conditions:

$$a_0: A + B = 1$$
$$a_1: 2A + 3B = 4$$

Solving: $A = -1, B = 2$

$$\boxed{a_n = -2^n + 2 \cdot 3^n}$$

**Verify:** $a_2 = -4 + 18 = 14$. Check: $5(4) - 6(1) = 14$ ✓

---

## 4. Case 2: Repeated Roots

If root $r$ has **multiplicity** $m$, it contributes $m$ solutions:

$$r^n, \; n \cdot r^n, \; n^2 \cdot r^n, \; \ldots, \; n^{m-1} \cdot r^n$$

### General Rule

If $r$ is a root of multiplicity $m$:

$$\text{Contribution: } (A_0 + A_1 n + A_2 n^2 + \cdots + A_{m-1} n^{m-1}) \cdot r^n$$

### Example: Repeated Root

Solve $a_n = 4a_{n-1} - 4a_{n-2}$, with $a_0 = 1, a_1 = 6$.

**Step 1.** Characteristic equation: $r^2 - 4r + 4 = 0$

$$(r-2)^2 = 0 \implies r = 2 \text{ (multiplicity 2)}$$

**Step 2.** General solution: $a_n = (A + Bn) \cdot 2^n$

**Step 3.** Apply initial conditions:

$$a_0: A = 1$$
$$a_1: (1 + B) \cdot 2 = 6 \implies B = 2$$

$$\boxed{a_n = (1 + 2n) \cdot 2^n}$$

**Verify:** $a_2 = 5 \cdot 4 = 20$. Check: $4(6) - 4(1) = 20$ ✓

---

## 5. Case 3: Complex Roots

If the characteristic equation has complex roots $r = \alpha \pm \beta i$, they always come in conjugate pairs. Use:

$$r = \rho (\cos\theta + i\sin\theta)$$

where $\rho = \sqrt{\alpha^2 + \beta^2}$ and $\theta = \arctan(\beta/\alpha)$.

The contribution is:

$$a_n = \rho^n (A\cos(n\theta) + B\sin(n\theta))$$

### Example

Solve $a_n = 2a_{n-1} - 2a_{n-2}$, with $a_0 = 0, a_1 = 1$.

**Step 1.** $r^2 - 2r + 2 = 0 \implies r = 1 \pm i$

$\rho = \sqrt{1^2 + 1^2} = \sqrt{2}$, $\theta = \arctan(1/1) = \pi/4$

**Step 2.** $a_n = (\sqrt{2})^n (A\cos(n\pi/4) + B\sin(n\pi/4))$

**Step 3.** $a_0 = 0 \implies A = 0$

$a_1 = \sqrt{2} \cdot B \cdot \sin(\pi/4) = \sqrt{2} \cdot B \cdot \frac{\sqrt{2}}{2} = B = 1$

$$\boxed{a_n = (\sqrt{2})^n \sin(n\pi/4)}$$

---

## 6. Solution Procedure Summary

```
  ┌──────────────────────────────────────────────┐
  │   Solving Homogeneous Linear Recurrences      │
  │                                                │
  │  1. Write characteristic equation r^k - ...   │
  │  2. Find all roots r₁, r₂, ..., rₖ           │
  │  3. Build general solution:                   │
  │     ┌──────────────────────────────┐          │
  │     │ Distinct r : A·rⁿ           │          │
  │     │ Repeated r (mult m):        │          │
  │     │   (A₀+A₁n+...+Aₘ₋₁n^{m-1})rⁿ│         │
  │     │ Complex α±βi :              │          │
  │     │   ρⁿ(A cos nθ + B sin nθ)  │          │
  │     └──────────────────────────────┘          │
  │  4. Apply initial conditions → solve for A,B  │
  │  5. Write closed-form solution                │
  └──────────────────────────────────────────────┘
```

---

## 7. Fibonacci Numbers — Complete Solution

$$F_n = F_{n-1} + F_{n-2}, \quad F_0 = 0, F_1 = 1$$

**Characteristic equation:** $r^2 - r - 1 = 0$

$$r = \frac{1 \pm \sqrt{5}}{2}$$

Let $\phi = \frac{1+\sqrt{5}}{2}$ and $\hat\phi = \frac{1-\sqrt{5}}{2}$.

**General solution:** $F_n = A\phi^n + B\hat\phi^n$

**Initial conditions:**
$$F_0 = 0: \quad A + B = 0 \implies B = -A$$
$$F_1 = 1: \quad A\phi - A\hat\phi = 1 \implies A(\phi - \hat\phi) = 1 \implies A\sqrt{5} = 1$$

$$A = \frac{1}{\sqrt{5}}, \quad B = -\frac{1}{\sqrt{5}}$$

$$\boxed{F_n = \frac{1}{\sqrt{5}}\left(\phi^n - \hat\phi^n\right) = \frac{1}{\sqrt{5}}\left[\left(\frac{1+\sqrt{5}}{2}\right)^n - \left(\frac{1-\sqrt{5}}{2}\right)^n\right]}$$

This is **Binet's formula**. Since $|\hat\phi| < 1$, $\hat\phi^n \to 0$, so $F_n \approx \frac{\phi^n}{\sqrt{5}}$.

---

## 8. Third-Order Example

Solve $a_n = 6a_{n-1} - 11a_{n-2} + 6a_{n-3}$, with $a_0 = 2, a_1 = 5, a_2 = 15$.

**Characteristic equation:** $r^3 - 6r^2 + 11r - 6 = 0$

$$(r-1)(r-2)(r-3) = 0 \implies r_1 = 1, r_2 = 2, r_3 = 3$$

**General solution:** $a_n = A \cdot 1^n + B \cdot 2^n + C \cdot 3^n = A + B \cdot 2^n + C \cdot 3^n$

**Apply initial conditions:**

| | Equation |
|---|---------|
| $a_0 = 2$ | $A + B + C = 2$ |
| $a_1 = 5$ | $A + 2B + 3C = 5$ |
| $a_2 = 15$ | $A + 4B + 9C = 15$ |

Solving: $A = 1, B = -1, C = 2$

$$\boxed{a_n = 1 - 2^n + 2 \cdot 3^n}$$

---

## 9. Important Theorems

**Theorem 1 (Superposition):** If $a_n^{(1)}$ and $a_n^{(2)}$ are solutions of a linear homogeneous recurrence, then so is $c_1 a_n^{(1)} + c_2 a_n^{(2)}$ for any constants $c_1, c_2$.

**Theorem 2 (Uniqueness):** A linear recurrence of order $k$ with $k$ initial conditions has a **unique** solution.

**Theorem 3:** The general solution of a $k$-th order linear homogeneous recurrence with constant coefficients is determined entirely by the characteristic roots.

---

## 📝 Summary Table

| Case | Roots | Solution Form |
|------|-------|---------------|
| Distinct real roots | $r_1, r_2, \ldots, r_k$ | $A_1 r_1^n + A_2 r_2^n + \cdots$ |
| Repeated root | $r$ with multiplicity $m$ | $(A_0 + A_1 n + \cdots + A_{m-1}n^{m-1})r^n$ |
| Complex roots | $\alpha \pm \beta i$ | $\rho^n(A\cos n\theta + B\sin n\theta)$ |
| Superposition | — | Linear combinations of solutions are solutions |

---

## ❓ Quick Revision Questions

1. **Find the closed form for $a_n = 7a_{n-1} - 10a_{n-2}$, with $a_0 = 2, a_1 = 1$.**

2. **Solve $a_n = 6a_{n-1} - 9a_{n-2}$, with $a_0 = 1, a_1 = 6$. (Hint: repeated root)**

3. **What are the characteristic roots of $a_n = a_{n-1} + 2a_{n-2}$? Find the general solution.**

4. **Why does a repeated root of multiplicity 2 require both $r^n$ and $nr^n$ in the solution?**

5. **Use Binet's formula to compute $F_{10}$. Verify against the definition.**

6. **Solve $a_n = -a_{n-2}$, with $a_0 = 1, a_1 = 0$. (Complex roots)**

---

[← Previous: Linear Recurrence Relations](01-linear-recurrence-relations.md) | [Back to README](../README.md) | [Next: Non-Homogeneous Recurrence →](03-non-homogeneous-recurrence.md)
