# Chapter 6.1: General Solutions

## Overview

Trigonometric equations typically have **infinitely many solutions** due to the periodic nature of trigonometric functions. This chapter introduces the concept of **general solutions**, which express all solutions using a single formula with an integer parameter.

---

## 📐 Why General Solutions?

### The Periodicity Problem

```
    Consider: sin θ = 1/2
    
    One solution: θ = 30° (or π/6)
    
    But sin is periodic with period 2π, so:
    θ = π/6, π/6 + 2π, π/6 + 4π, π/6 - 2π, ...
    
    Also, sin(π - θ) = sin θ, so:
    θ = 5π/6, 5π/6 + 2π, 5π/6 - 2π, ...
    
    We need ONE formula to capture ALL solutions!
```

### Visual Representation

```
    y = sin x
    
    y
    │      ___           ___           ___
  1 ┤     /   \         /   \         /   \
    │    /     \       /     \       /     \
1/2 ┤---/-------\-----/-------\-----/-------\----  y = 1/2
    │  /         \   /         \   /         \
  0 ┼─/───────────\─/───────────\─/───────────\─── x
    │/             X             X             \
 -1 ┤              ‾‾‾           ‾‾‾
    │
    └──────────────────────────────────────────────
      0   π/6  5π/6  2π        4π
          ↑    ↑      ↑    ↑
       Solutions repeat every 2π
```

---

## 📐 Principal Values

### Definition

The **principal value** (or principal solution) is the unique solution in a standard range.

### Standard Ranges

| Equation | Principal Value Range | Principal Value Symbol |
|----------|----------------------|------------------------|
| sin θ = k | [-π/2, π/2] | α |
| cos θ = k | [0, π] | α |
| tan θ = k | (-π/2, π/2) | α |

### Why These Ranges?

```
    ┌────────────────────────────────────────────────────────────────┐
    │  These ranges ensure EXACTLY ONE solution exists for each k    │
    │  in the valid domain:                                          │
    │                                                                │
    │  sin: From minimum to maximum in a "half-wave"                │
    │       Range: [-π/2, π/2] covers sin from -1 to 1              │
    │                                                                │
    │  cos: From maximum to minimum in a "half-wave"                │
    │       Range: [0, π] covers cos from 1 to -1                   │
    │                                                                │
    │  tan: One complete branch (avoiding asymptotes)                │
    │       Range: (-π/2, π/2) covers tan from -∞ to +∞            │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 📐 General Solution Formulas

### Formula 1: sin θ = sin α

$$\boxed{\theta = n\pi + (-1)^n \alpha, \quad n \in \mathbb{Z}}$$

**Understanding:**
- When n is even: θ = 2kπ + α (solutions in Q1 and Q4)
- When n is odd: θ = (2k+1)π - α (solutions in Q2 and Q3)

### Formula 2: cos θ = cos α

$$\boxed{\theta = 2n\pi \pm \alpha, \quad n \in \mathbb{Z}}$$

**Understanding:**
- θ = 2nπ + α (solutions where angle is α plus full rotations)
- θ = 2nπ - α (solutions where angle is -α plus full rotations)

### Formula 3: tan θ = tan α

$$\boxed{\theta = n\pi + \alpha, \quad n \in \mathbb{Z}}$$

**Understanding:**
- tan has period π, so all solutions are α apart by multiples of π

---

## 📊 Special Cases

### When sin θ = 0

$$\sin\theta = 0 \Rightarrow \theta = n\pi, \quad n \in \mathbb{Z}$$

Solutions: ..., -2π, -π, 0, π, 2π, 3π, ...

### When sin θ = 1

$$\sin\theta = 1 \Rightarrow \theta = 2n\pi + \frac{\pi}{2}, \quad n \in \mathbb{Z}$$

Or: θ = (4n + 1)π/2

### When sin θ = -1

$$\sin\theta = -1 \Rightarrow \theta = 2n\pi - \frac{\pi}{2}, \quad n \in \mathbb{Z}$$

Or: θ = (4n - 1)π/2

### When cos θ = 0

$$\cos\theta = 0 \Rightarrow \theta = (2n + 1)\frac{\pi}{2}, \quad n \in \mathbb{Z}$$

Solutions: ..., -3π/2, -π/2, π/2, 3π/2, ...

### When cos θ = 1

$$\cos\theta = 1 \Rightarrow \theta = 2n\pi, \quad n \in \mathbb{Z}$$

### When cos θ = -1

$$\cos\theta = -1 \Rightarrow \theta = (2n + 1)\pi, \quad n \in \mathbb{Z}$$

### When tan θ = 0

$$\tan\theta = 0 \Rightarrow \theta = n\pi, \quad n \in \mathbb{Z}$$

---

## 📊 Summary Chart

```
    ┌────────────────────────────────────────────────────────────────┐
    │                  GENERAL SOLUTION FORMULAS                     │
    ├────────────────────────────────────────────────────────────────┤
    │                                                                │
    │  Equation             General Solution                         │
    │  ─────────            ────────────────                         │
    │                                                                │
    │  sin θ = sin α        θ = nπ + (-1)ⁿα                         │
    │                                                                │
    │  cos θ = cos α        θ = 2nπ ± α                             │
    │                                                                │
    │  tan θ = tan α        θ = nπ + α                              │
    │                                                                │
    │  sin θ = 0            θ = nπ                                  │
    │                                                                │
    │  cos θ = 0            θ = (2n+1)π/2                           │
    │                                                                │
    │  tan θ = 0            θ = nπ                                  │
    │                                                                │
    │  sin θ = 1            θ = 2nπ + π/2                           │
    │                                                                │
    │  cos θ = 1            θ = 2nπ                                 │
    │                                                                │
    │  Where n ∈ ℤ (any integer)                                    │
    │                                                                │
    └────────────────────────────────────────────────────────────────┘
```

---

## 🧮 Worked Examples

### Example 1: Find the general solution of sin θ = 1/2

**Solution:**
First, find the principal value:
$$\sin\alpha = \frac{1}{2} \Rightarrow \alpha = \frac{\pi}{6}$$

Using the general solution formula for sin θ = sin α:
$$\theta = n\pi + (-1)^n \cdot \frac{\pi}{6}, \quad n \in \mathbb{Z}$$

**Verification (first few solutions):**
- n = 0: θ = 0 + π/6 = π/6 ✓
- n = 1: θ = π - π/6 = 5π/6 ✓
- n = 2: θ = 2π + π/6 = 13π/6 ✓
- n = -1: θ = -π - π/6 = -7π/6 ✓

### Example 2: Find the general solution of cos θ = -1/2

**Solution:**
First, find the principal value:
$$\cos\alpha = -\frac{1}{2} \Rightarrow \alpha = \frac{2\pi}{3}$$

(Note: α is in [0, π], and cos(2π/3) = -1/2)

Using the general solution formula for cos θ = cos α:
$$\theta = 2n\pi \pm \frac{2\pi}{3}, \quad n \in \mathbb{Z}$$

**This gives two families:**
- θ = 2nπ + 2π/3
- θ = 2nπ - 2π/3

### Example 3: Find the general solution of tan θ = 1

**Solution:**
First, find the principal value:
$$\tan\alpha = 1 \Rightarrow \alpha = \frac{\pi}{4}$$

Using the general solution formula for tan θ = tan α:
$$\theta = n\pi + \frac{\pi}{4}, \quad n \in \mathbb{Z}$$

**Verification:**
- n = 0: θ = π/4 (45°) ✓
- n = 1: θ = π + π/4 = 5π/4 (225°) ✓
- n = -1: θ = -π + π/4 = -3π/4 ✓

### Example 4: Find the general solution of sin 2θ = √3/2

**Solution:**
Let φ = 2θ. Then sin φ = √3/2.

Principal value: φ = π/3

General solution for φ:
$$\phi = n\pi + (-1)^n \cdot \frac{\pi}{3}$$

Substitute back φ = 2θ:
$$2\theta = n\pi + (-1)^n \cdot \frac{\pi}{3}$$
$$\theta = \frac{n\pi}{2} + (-1)^n \cdot \frac{\pi}{6}$$

### Example 5: Find the general solution of cos 3θ = 0

**Solution:**
Let φ = 3θ. Then cos φ = 0.

General solution: φ = (2n + 1)π/2

$$3\theta = (2n + 1)\frac{\pi}{2}$$
$$\theta = (2n + 1)\frac{\pi}{6}$$

---

## 🧠 Memory Techniques

### For sin θ = sin α

Remember: **"nπ plus or minus with alternating sign"**
- The (-1)ⁿ creates alternation between + and -

### For cos θ = cos α

Remember: **"2nπ with ± α"**
- The ± captures both α and -α solutions

### For tan θ = tan α

Remember: **"nπ + α"** (simplest formula)
- Just add multiples of π

### Mnemonic

**S**in: **S**pecial alternating formula (more complex)
**C**os: ± **C**hoice of sign
**T**an: **T**rivially simple (just add nπ)

---

## 📐 Visualization of Solutions

### sin θ = 1/2 Solutions

```
    Unit Circle View:
    
              y
              │
              │   • θ = π/6
         ─────┼─────
              │
        • θ = 5π/6
              │
    ──────────*────────── x
              │
              │
              │
              
    Both points have y-coordinate = 1/2
    
    All solutions: Rotate by 2π from these points
```

### tan θ = 1 Solutions

```
    Unit Circle View:
    
              y
              │     • θ = π/4
              │   ╱
              │ ╱
    ──────────*────────── x
            ╱ │
          ╱   │
        •     │ θ = 5π/4 (= π + π/4)
              │
              
    Solutions are π apart (tan's period is π)
```

---

## 📋 Summary Table

### Key Formulas

| Equation Type | General Solution |
|---------------|------------------|
| sin θ = sin α | θ = nπ + (-1)ⁿα |
| cos θ = cos α | θ = 2nπ ± α |
| tan θ = tan α | θ = nπ + α |
| sin θ = 0 | θ = nπ |
| cos θ = 0 | θ = (2n+1)π/2 |
| tan θ = 0 | θ = nπ |

### Key Points

| Concept | Detail |
|---------|--------|
| Principal value | Unique solution in standard range |
| General solution | Contains parameter n ∈ ℤ |
| Period of sin/cos | 2π |
| Period of tan | π |
| Checking solutions | Substitute back to verify |

---

## ❓ Quick Revision Questions

1. **Find the general solution of sin θ = -√3/2.**

2. **Find the general solution of cos θ = √2/2.**

3. **Find the general solution of tan θ = -1.**

4. **Find the general solution of sin 2θ = 1.**

5. **Find all solutions of cos θ = 1/2 in the interval [0, 2π].**

6. **Why is the general solution formula for tan simpler than for sin or cos?**

<details>
<summary>Click to see answers</summary>

1. Principal value: α = -π/3 (since sin(-π/3) = -√3/2)  
   General solution: **θ = nπ + (-1)ⁿ(-π/3) = nπ - (-1)ⁿ(π/3)**  
   Or equivalently: **θ = nπ + (-1)ⁿ⁺¹(π/3)**

2. Principal value: α = π/4 (since cos(π/4) = √2/2)  
   General solution: **θ = 2nπ ± π/4**

3. Principal value: α = -π/4 (since tan(-π/4) = -1)  
   General solution: **θ = nπ - π/4**  
   Or: θ = nπ + 3π/4 (using α = 3π/4)

4. Let φ = 2θ, then sin φ = 1  
   φ = 2nπ + π/2  
   2θ = 2nπ + π/2  
   **θ = nπ + π/4**

5. General solution: θ = 2nπ ± π/3  
   For n = 0:  
   - θ = π/3 ✓ (in [0, 2π])  
   - θ = -π/3 ✗ (not in [0, 2π])  
   For n = 1:  
   - θ = 2π + π/3 ✗ (not in [0, 2π])  
   - θ = 2π - π/3 = 5π/3 ✓ (in [0, 2π])  
   **Solutions: θ = π/3, 5π/3**

6. The tangent function has period π (not 2π like sin and cos), and within each period, there's only ONE value of θ that gives each value of tan θ. Sin and cos have period 2π and two solutions within each period (one going "up" and one going "down"), hence the more complex formulas needed to capture both.

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← Unit 5: Power Reduction](../05-Multiple-Submultiple-Angles/04-power-reduction.md) | [Unit 6 Index](README.md) | [6.2 Basic Equations →](02-basic-equations.md) |

---

[← Back to Main Index](../README.md)
