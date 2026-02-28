# Chapter 7.3: Non-Homogeneous Recurrence Relations

[← Previous: Homogeneous Recurrence](02-homogeneous-recurrence.md) | [Back to README](../README.md) | [Next: Generating Functions →](04-generating-functions.md)

---

## 📋 Chapter Overview

A **non-homogeneous** linear recurrence has the form $a_n = c_1 a_{n-1} + \cdots + c_k a_{n-k} + f(n)$ where $f(n) \neq 0$. The solution combines the **homogeneous solution** with a **particular solution**.

---

## 1. Structure of the Solution

$$\boxed{a_n = a_n^{(h)} + a_n^{(p)}}$$

- $a_n^{(h)}$ = general solution of the **associated homogeneous** recurrence (set $f(n)=0$)
- $a_n^{(p)}$ = any **particular solution** of the non-homogeneous recurrence

---

## 2. Method of Undetermined Coefficients

Guess the form of $a_n^{(p)}$ based on $f(n)$, then solve for the unknown constants.

### Standard Guesses

| $f(n)$ | Initial Guess for $a_n^{(p)}$ |
|:------:|:-----------------------------:|
| $d$ (constant) | $C$ |
| $dn + e$ | $Cn + D$ |
| $dn^2$ | $Cn^2 + Dn + E$ |
| $d \cdot s^n$ | $C \cdot s^n$ |
| $d \cdot n \cdot s^n$ | $(Cn + D) \cdot s^n$ |

### ⚠️ Modification Rule

If the initial guess is **already a solution** of the homogeneous equation, multiply it by $n$ (or $n^2$, etc.) until it is no longer a solution.

| Situation | Modified Guess |
|-----------|:--------------:|
| $C \cdot s^n$ is a homogeneous solution | $Cn \cdot s^n$ |
| $Cn \cdot s^n$ is also a homogeneous solution | $Cn^2 \cdot s^n$ |

---

## 3. Worked Example: Constant $f(n)$

Solve $a_n = 2a_{n-1} + 3$, $a_0 = 1$.

**Homogeneous:** $a_n = 2a_{n-1}$ → char. eq. $r = 2$ → $a_n^{(h)} = A \cdot 2^n$

**Particular:** Guess $a_n^{(p)} = C$.

Substitute: $C = 2C + 3 \implies C = -3$

**General solution:** $a_n = A \cdot 2^n - 3$

**Initial condition:** $a_0 = A - 3 = 1 \implies A = 4$

$$\boxed{a_n = 4 \cdot 2^n - 3 = 2^{n+2} - 3}$$

**Verify:** $a_1 = 8 - 3 = 5$. Check: $2(1) + 3 = 5$ ✓

---

## 4. Worked Example: Polynomial $f(n)$

Solve $a_n = a_{n-1} + 2n + 1$, $a_0 = 0$.

**Homogeneous:** $r = 1$ → $a_n^{(h)} = A$

**Particular:** Guess $a_n^{(p)} = Cn + D$?

But wait — $C$ (constant) is already a homogeneous solution ($r=1$). So multiply by $n$:

Guess $a_n^{(p)} = Cn^2 + Dn$

Substitute: $Cn^2 + Dn = C(n-1)^2 + D(n-1) + 2n + 1$

$Cn^2 + Dn = Cn^2 - 2Cn + C + Dn - D + 2n + 1$

$0 = (-2C + 2)n + (C - D + 1)$

$-2C + 2 = 0 \implies C = 1$  
$C - D + 1 = 0 \implies D = 2$

$a_n^{(p)} = n^2 + 2n$

**General solution:** $a_n = A + n^2 + 2n$

$a_0 = A = 0$

$$\boxed{a_n = n^2 + 2n = n(n+2)}$$

---

## 5. Worked Example: Exponential $f(n)$

Solve $a_n = 5a_{n-1} - 6a_{n-2} + 3^n$, $a_0 = 1, a_1 = 4$.

**Homogeneous:** $r^2 - 5r + 6 = 0 \implies (r-2)(r-3) = 0$, roots $r = 2, 3$

$a_n^{(h)} = A \cdot 2^n + B \cdot 3^n$

**Particular:** $f(n) = 3^n$. But $r = 3$ is already a characteristic root!

Multiply by $n$: Guess $a_n^{(p)} = Cn \cdot 3^n$

Substitute into recurrence:

$Cn \cdot 3^n = 5C(n-1) \cdot 3^{n-1} - 6C(n-2) \cdot 3^{n-2} + 3^n$

Divide by $3^{n-2}$:

$9Cn = 15C(n-1) - 6C(n-2) + 9$

$9Cn = 15Cn - 15C - 6Cn + 12C + 9$

$9Cn = 9Cn - 3C + 9$

$0 = -3C + 9 \implies C = 3$

$a_n^{(p)} = 3n \cdot 3^n = n \cdot 3^{n+1}$

**General solution:** $a_n = A \cdot 2^n + B \cdot 3^n + n \cdot 3^{n+1}$

**Initial conditions:**

$a_0 = A + B = 1$  
$a_1 = 2A + 3B + 9 = 4 \implies 2A + 3B = -5$

$A = 8, \; B = -7$

$$\boxed{a_n = 8 \cdot 2^n - 7 \cdot 3^n + n \cdot 3^{n+1}}$$

---

## 6. Back-Substitution (Iteration) Method

For simpler recurrences, repeatedly substitute until a pattern emerges.

### Example

$a_n = 2a_{n-1} + 1, \quad a_0 = 0$

```
  a_n = 2·a_{n-1} + 1
      = 2(2·a_{n-2} + 1) + 1      = 4·a_{n-2} + 2 + 1
      = 4(2·a_{n-3} + 1) + 2 + 1  = 8·a_{n-3} + 4 + 2 + 1
      ...
      = 2^n · a_0 + 2^{n-1} + 2^{n-2} + ... + 2 + 1
      = 0 + (2^n - 1)
      = 2^n - 1
```

$$a_n = 2^n - 1$$

---

## 7. Telescoping Method

If the recurrence can be written as a difference:

$a_n - g(n) = a_{n-1} - g(n-1) + h(n)$

Sum both sides from 1 to $n$ to collapse (telescope) the sum.

### Example

$a_n = a_{n-1} + n$, $a_0 = 0$

$$a_n - a_{n-1} = n$$

Summing from 1 to $n$:

$$a_n - a_0 = \sum_{k=1}^{n} k = \frac{n(n+1)}{2}$$

$$a_n = \frac{n(n+1)}{2}$$

---

## 8. Decision Flowchart

```
  ┌────────────────────────────────────┐
  │    Non-Homogeneous Recurrence      │
  │    aₙ = c₁aₙ₋₁ + ... + f(n)      │
  │                                    │
  │  Step 1: Solve homogeneous part    │
  │  Step 2: Examine f(n)             │
  │                                    │
  │  f(n) = polynomial?               │
  │  ├── YES → Guess polynomial        │
  │  │         same degree             │
  │  │                                 │
  │  f(n) = exponential sⁿ?           │
  │  ├── YES → Guess C·sⁿ             │
  │  │                                 │
  │  f(n) = poly × exponential?        │
  │  ├── YES → Guess poly × sⁿ        │
  │  │                                 │
  │  Step 3: Is guess a homogeneous    │
  │          solution?                 │
  │  ├── YES → Multiply by n           │
  │  │                                 │
  │  Step 4: Substitute, solve for     │
  │          undetermined coefficients │
  │  Step 5: Combine: a(h) + a(p)     │
  │  Step 6: Apply initial conditions  │
  └────────────────────────────────────┘
```

---

## 📝 Summary Table

| Concept | Description |
|---------|-------------|
| General solution | $a_n = a_n^{(h)} + a_n^{(p)}$ |
| Homogeneous part | Solve with characteristic equation |
| Particular solution | Guess based on form of $f(n)$ |
| Modification rule | Multiply guess by $n$ if it solves the homogeneous equation |
| Back-substitution | Iterate recurrence to find a pattern |
| Telescoping | Sum differences to collapse |

---

## ❓ Quick Revision Questions

1. **Solve $a_n = 3a_{n-1} + 2$ with $a_0 = 1$.**

2. **Find the general solution of $a_n = 2a_{n-1} + n$ with $a_0 = 0$.**

3. **Why must we multiply the particular solution guess by $n$ when $f(n) = 3^n$ and $r=3$ is a characteristic root?**

4. **Solve $a_n = a_{n-1} + 2^n$ with $a_0 = 0$ using back-substitution.**

5. **Find $a_n$ if $a_n = 4a_{n-1} - 4a_{n-2} + 2^n$ with $a_0 = 0, a_1 = 1$.**

6. **For the Tower of Hanoi $T_n = 2T_{n-1} + 1$, $T_0 = 0$, derive the closed form.**

---

[← Previous: Homogeneous Recurrence](02-homogeneous-recurrence.md) | [Back to README](../README.md) | [Next: Generating Functions →](04-generating-functions.md)
