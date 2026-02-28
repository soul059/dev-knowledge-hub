# Chapter 6.2: Basic Trigonometric Equations

## Overview

This chapter covers the systematic approach to solving basic trigonometric equations of the form sin x = k, cos x = k, and tan x = k, where k is a constant. We'll work through finding principal values and expressing general solutions.

---

## 📐 Equations of the Form sin x = k

### Valid Range

For sin x = k to have solutions, we need **-1 ≤ k ≤ 1**.

### Solution Method

1. Find the principal value α where sin α = k (α ∈ [-π/2, π/2])
2. Apply general solution: x = nπ + (-1)ⁿα

### Case Analysis

```
    ┌────────────────────────────────────────────────────────────────┐
    │                      sin x = k                                 │
    ├────────────────────────────────────────────────────────────────┤
    │                                                                │
    │  Case 1: k > 0                                                │
    │  Principal value α in (0, π/2]                                 │
    │  Example: sin x = 1/2 → α = π/6                               │
    │                                                                │
    │  Case 2: k < 0                                                │
    │  Principal value α in [-π/2, 0)                               │
    │  Example: sin x = -1/2 → α = -π/6                             │
    │                                                                │
    │  Case 3: k = 0                                                │
    │  x = nπ                                                       │
    │                                                                │
    │  Case 4: k = 1                                                │
    │  x = 2nπ + π/2                                                │
    │                                                                │
    │  Case 5: k = -1                                               │
    │  x = 2nπ - π/2                                                │
    │                                                                │
    │  Case 6: |k| > 1                                              │
    │  NO SOLUTION                                                  │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📐 Equations of the Form cos x = k

### Valid Range

For cos x = k to have solutions, we need **-1 ≤ k ≤ 1**.

### Solution Method

1. Find the principal value α where cos α = k (α ∈ [0, π])
2. Apply general solution: x = 2nπ ± α

### Case Analysis

```
    ┌────────────────────────────────────────────────────────────────┐
    │                      cos x = k                                 │
    ├────────────────────────────────────────────────────────────────┤
    │                                                                │
    │  Case 1: k > 0                                                │
    │  Principal value α in [0, π/2)                                 │
    │  Example: cos x = √3/2 → α = π/6                              │
    │                                                                │
    │  Case 2: k < 0                                                │
    │  Principal value α in (π/2, π]                                 │
    │  Example: cos x = -√3/2 → α = 5π/6                            │
    │                                                                │
    │  Case 3: k = 0                                                │
    │  x = (2n+1)π/2                                                │
    │                                                                │
    │  Case 4: k = 1                                                │
    │  x = 2nπ                                                      │
    │                                                                │
    │  Case 5: k = -1                                               │
    │  x = (2n+1)π                                                  │
    │                                                                │
    │  Case 6: |k| > 1                                              │
    │  NO SOLUTION                                                  │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📐 Equations of the Form tan x = k

### Valid Range

tan x = k has solutions for **all real values of k**.

### Solution Method

1. Find the principal value α where tan α = k (α ∈ (-π/2, π/2))
2. Apply general solution: x = nπ + α

### Case Analysis

```
    ┌────────────────────────────────────────────────────────────────┐
    │                      tan x = k                                 │
    ├────────────────────────────────────────────────────────────────┤
    │                                                                │
    │  Case 1: k > 0                                                │
    │  Principal value α in (0, π/2)                                 │
    │  Example: tan x = 1 → α = π/4                                 │
    │                                                                │
    │  Case 2: k < 0                                                │
    │  Principal value α in (-π/2, 0)                               │
    │  Example: tan x = -1 → α = -π/4                               │
    │                                                                │
    │  Case 3: k = 0                                                │
    │  x = nπ                                                       │
    │                                                                │
    │  Always has a solution (no restriction on k)                  │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 🧮 Worked Examples

### Example 1: sin x = √3/2

**Solution:**
Principal value: α = π/3 (since sin(π/3) = √3/2)

General solution:
$$x = n\pi + (-1)^n \cdot \frac{\pi}{3}, \quad n \in \mathbb{Z}$$

**Verification:**
- n = 0: x = π/3 ✓
- n = 1: x = π - π/3 = 2π/3 ✓
- n = 2: x = 2π + π/3 = 7π/3 ✓

### Example 2: cos x = -1/2

**Solution:**
We need cos α = -1/2 where α ∈ [0, π].

Since cos(2π/3) = -1/2, we have α = 2π/3.

General solution:
$$x = 2n\pi \pm \frac{2\pi}{3}, \quad n \in \mathbb{Z}$$

**In [0, 2π]:**
- x = 2π/3 ✓
- x = 4π/3 (= 2π - 2π/3) ✓

### Example 3: tan x = √3

**Solution:**
Principal value: α = π/3 (since tan(π/3) = √3)

General solution:
$$x = n\pi + \frac{\pi}{3}, \quad n \in \mathbb{Z}$$

**In [0, 2π]:**
- n = 0: x = π/3 ✓
- n = 1: x = π + π/3 = 4π/3 ✓

### Example 4: 2sin x - 1 = 0

**Solution:**
$$2\sin x - 1 = 0$$
$$\sin x = \frac{1}{2}$$

Principal value: α = π/6

General solution:
$$x = n\pi + (-1)^n \cdot \frac{\pi}{6}, \quad n \in \mathbb{Z}$$

### Example 5: 2cos²x - 1 = 0

**Solution:**
$$2\cos^2 x = 1$$
$$\cos^2 x = \frac{1}{2}$$
$$\cos x = \pm\frac{1}{\sqrt{2}}$$

**Case 1:** cos x = 1/√2
Principal value: α = π/4
General solution: x = 2nπ ± π/4

**Case 2:** cos x = -1/√2
Principal value: α = 3π/4
General solution: x = 2nπ ± 3π/4

**Combined solution:**
$$x = n\frac{\pi}{2} + \frac{\pi}{4} = \frac{(2n+1)\pi}{4}, \quad n \in \mathbb{Z}$$

Or: x = (2n+1)π/4

### Example 6: tan²x = 3

**Solution:**
$$\tan^2 x = 3$$
$$\tan x = \pm\sqrt{3}$$

**Case 1:** tan x = √3
α = π/3
Solution: x = nπ + π/3

**Case 2:** tan x = -√3
α = -π/3
Solution: x = nπ - π/3

**Combined:**
$$x = n\pi \pm \frac{\pi}{3}, \quad n \in \mathbb{Z}$$

Or: x = nπ/3 where n is not divisible by 3

---

## 📊 Finding Solutions in Specific Intervals

### Method for [0, 2π]

1. Find the general solution
2. Substitute n = 0, ±1, ±2, ... 
3. Keep only values in [0, 2π]

### Example: Find all solutions of sin x = 1/2 in [0, 2π]

General solution: x = nπ + (-1)ⁿ(π/6)

Substituting:
- n = 0: x = π/6 ✓ (in range)
- n = 1: x = π - π/6 = 5π/6 ✓ (in range)
- n = 2: x = 2π + π/6 = 13π/6 ✗ (> 2π)
- n = -1: x = -π - π/6 = -7π/6 ✗ (< 0)

**Solutions in [0, 2π]: x = π/6, 5π/6**

---

## 📐 Standard Values Reference

### Common Sine Equations

| sin x = | Principal Value α | General Solution |
|---------|-------------------|------------------|
| 0 | 0 | nπ |
| 1/2 | π/6 | nπ + (-1)ⁿ(π/6) |
| √2/2 | π/4 | nπ + (-1)ⁿ(π/4) |
| √3/2 | π/3 | nπ + (-1)ⁿ(π/3) |
| 1 | π/2 | 2nπ + π/2 |
| -1/2 | -π/6 | nπ + (-1)ⁿ(-π/6) |
| -√2/2 | -π/4 | nπ + (-1)ⁿ(-π/4) |
| -√3/2 | -π/3 | nπ + (-1)ⁿ(-π/3) |
| -1 | -π/2 | 2nπ - π/2 |

### Common Cosine Equations

| cos x = | Principal Value α | General Solution |
|---------|-------------------|------------------|
| 0 | π/2 | (2n+1)π/2 |
| 1/2 | π/3 | 2nπ ± π/3 |
| √2/2 | π/4 | 2nπ ± π/4 |
| √3/2 | π/6 | 2nπ ± π/6 |
| 1 | 0 | 2nπ |
| -1/2 | 2π/3 | 2nπ ± 2π/3 |
| -√2/2 | 3π/4 | 2nπ ± 3π/4 |
| -√3/2 | 5π/6 | 2nπ ± 5π/6 |
| -1 | π | (2n+1)π |

### Common Tangent Equations

| tan x = | Principal Value α | General Solution |
|---------|-------------------|------------------|
| 0 | 0 | nπ |
| 1/√3 | π/6 | nπ + π/6 |
| 1 | π/4 | nπ + π/4 |
| √3 | π/3 | nπ + π/3 |
| -1/√3 | -π/6 | nπ - π/6 |
| -1 | -π/4 | nπ - π/4 |
| -√3 | -π/3 | nπ - π/3 |

---

## 📋 Summary Table

### Solution Process

| Step | Action |
|------|--------|
| 1 | Isolate the trigonometric function |
| 2 | Check if solution exists (for sin, cos: \|k\| ≤ 1) |
| 3 | Find principal value α |
| 4 | Apply appropriate general solution formula |
| 5 | If interval specified, find all solutions in range |

### Key Points

| Concept | Detail |
|---------|--------|
| sin x = k | Need -1 ≤ k ≤ 1 |
| cos x = k | Need -1 ≤ k ≤ 1 |
| tan x = k | No restriction on k |
| Squared equations | Give ± cases, solve both |
| Interval solutions | Substitute n values |

---

## ❓ Quick Revision Questions

1. **Find the general solution of sin x = -1/√2.**

2. **Find the general solution of cos x = √3/2.**

3. **Find all solutions of tan x = -√3 in the interval [0, 2π].**

4. **Solve: 4sin²x - 3 = 0 (general solution).**

5. **Find all solutions of cos x = 0.5 in [0, 4π].**

6. **Why does tan x = k always have a solution while sin x = k might not?**

<details>
<summary>Click to see answers</summary>

1. sin x = -1/√2 = -√2/2  
   Principal value: α = -π/4  
   General solution: **x = nπ + (-1)ⁿ(-π/4)**  
   Or: x = nπ - (-1)ⁿ(π/4)

2. cos x = √3/2  
   Principal value: α = π/6  
   General solution: **x = 2nπ ± π/6**

3. tan x = -√3  
   Principal value: α = -π/3  
   General solution: x = nπ - π/3  
   
   In [0, 2π]:  
   - n = 1: x = π - π/3 = 2π/3 ✓  
   - n = 2: x = 2π - π/3 = 5π/3 ✓  
   **Solutions: x = 2π/3, 5π/3**

4. 4sin²x - 3 = 0  
   sin²x = 3/4  
   sin x = ±√3/2  
   
   For sin x = √3/2: α = π/3, so x = nπ + (-1)ⁿ(π/3)  
   For sin x = -√3/2: α = -π/3, so x = nπ + (-1)ⁿ(-π/3)  
   
   **Combined: x = nπ ± π/3** (or x = nπ + (-1)ⁿ(±π/3))

5. cos x = 0.5 = 1/2  
   Principal value: α = π/3  
   General solution: x = 2nπ ± π/3  
   
   In [0, 4π]:  
   - 2(0)π + π/3 = π/3 ✓  
   - 2(0)π - π/3 = -π/3 ✗  
   - 2(1)π + π/3 = 7π/3 ✓  
   - 2(1)π - π/3 = 5π/3 ✓  
   - 2(2)π + π/3 = 13π/3 ✗ (> 4π)  
   - 2(2)π - π/3 = 11π/3 ✓  
   
   **Solutions: x = π/3, 5π/3, 7π/3, 11π/3**

6. The range of sin x and cos x is limited to [-1, 1], so if |k| > 1, no angle x exists with sin x = k or cos x = k.  
   
   However, tan x has range (-∞, +∞), covering all real numbers. For any real number k, there exists an angle x (in fact, infinitely many) such that tan x = k. This is because the tangent function is unbounded and covers all real values in each of its periods.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 6.1 General Solutions](01-general-solutions.md) | [Unit 6 Index](README.md) | [6.3 Quadratic Equations →](03-quadratic-equations.md) |

---

[← Back to Main Index](../README.md)
