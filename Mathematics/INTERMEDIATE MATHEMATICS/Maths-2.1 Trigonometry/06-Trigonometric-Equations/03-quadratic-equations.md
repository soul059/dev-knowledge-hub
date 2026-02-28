# Chapter 6.3: Quadratic Trigonometric Equations

## Overview

Quadratic trigonometric equations are equations that can be expressed in the form **at² + bt + c = 0**, where t is a trigonometric function (sin x, cos x, or tan x). These are solved by treating the trigonometric function as a variable, solving the quadratic, then finding the angles.

---

## 📐 General Approach

### Step-by-Step Method

```
    ┌─────────────────────────────────────────────────────────────────┐
    │           SOLVING QUADRATIC TRIG EQUATIONS                      │
    ├─────────────────────────────────────────────────────────────────┤
    │                                                                 │
    │  Step 1: Identify the trigonometric function (sin x, cos x,    │
    │          or tan x) and let t = that function                   │
    │                                                                 │
    │  Step 2: Rewrite equation as at² + bt + c = 0                  │
    │                                                                 │
    │  Step 3: Solve the quadratic using:                            │
    │          • Factoring                                           │
    │          • Quadratic formula: t = (-b ± √(b²-4ac))/2a         │
    │                                                                 │
    │  Step 4: Check validity of solutions:                          │
    │          • For sin x = t or cos x = t: need |t| ≤ 1           │
    │          • For tan x = t: all real t are valid                 │
    │                                                                 │
    │  Step 5: Find x using general solution formulas                │
    │                                                                 │
    └─────────────────────────────────────────────────────────────────┘
```

---

## 🧮 Worked Examples

### Example 1: 2sin²x - 3sin x + 1 = 0

**Solution:**
Let t = sin x. The equation becomes:
$$2t^2 - 3t + 1 = 0$$

Factoring:
$$(2t - 1)(t - 1) = 0$$
$$t = \frac{1}{2} \quad \text{or} \quad t = 1$$

**Case 1:** sin x = 1/2
$$x = n\pi + (-1)^n \cdot \frac{\pi}{6}$$

**Case 2:** sin x = 1
$$x = 2n\pi + \frac{\pi}{2}$$

**Complete solution:**
$$x = n\pi + (-1)^n \cdot \frac{\pi}{6} \quad \text{or} \quad x = 2n\pi + \frac{\pi}{2}, \quad n \in \mathbb{Z}$$

### Example 2: 2cos²x + 3cos x - 2 = 0

**Solution:**
Let t = cos x. The equation becomes:
$$2t^2 + 3t - 2 = 0$$

Factoring:
$$(2t - 1)(t + 2) = 0$$
$$t = \frac{1}{2} \quad \text{or} \quad t = -2$$

**Case 1:** cos x = 1/2 ✓ (valid since |1/2| ≤ 1)
$$x = 2n\pi \pm \frac{\pi}{3}$$

**Case 2:** cos x = -2 ✗ (invalid since |-2| > 1)
No solution from this case.

**Complete solution:**
$$x = 2n\pi \pm \frac{\pi}{3}, \quad n \in \mathbb{Z}$$

### Example 3: tan²x - tan x - 2 = 0

**Solution:**
Let t = tan x. The equation becomes:
$$t^2 - t - 2 = 0$$

Factoring:
$$(t - 2)(t + 1) = 0$$
$$t = 2 \quad \text{or} \quad t = -1$$

**Case 1:** tan x = 2
$$x = n\pi + \arctan(2)$$

**Case 2:** tan x = -1
$$x = n\pi - \frac{\pi}{4}$$

**Complete solution:**
$$x = n\pi + \arctan(2) \quad \text{or} \quad x = n\pi - \frac{\pi}{4}, \quad n \in \mathbb{Z}$$

### Example 4: 2sin²x - sin x - 1 = 0 in [0, 2π]

**Solution:**
Let t = sin x:
$$2t^2 - t - 1 = 0$$
$$(2t + 1)(t - 1) = 0$$
$$t = -\frac{1}{2} \quad \text{or} \quad t = 1$$

**Case 1:** sin x = -1/2
In [0, 2π]: x = 7π/6, 11π/6

**Case 2:** sin x = 1
In [0, 2π]: x = π/2

**Solutions in [0, 2π]: x = π/2, 7π/6, 11π/6**

### Example 5: 4cos²x - 4cos x + 1 = 0

**Solution:**
Let t = cos x:
$$4t^2 - 4t + 1 = 0$$
$$(2t - 1)^2 = 0$$
$$t = \frac{1}{2}$$ (double root)

cos x = 1/2:
$$x = 2n\pi \pm \frac{\pi}{3}, \quad n \in \mathbb{Z}$$

### Example 6: Using Quadratic Formula

Solve: cos²x + 4cos x + 2 = 0

**Solution:**
Let t = cos x:
$$t^2 + 4t + 2 = 0$$

Using quadratic formula:
$$t = \frac{-4 \pm \sqrt{16 - 8}}{2} = \frac{-4 \pm \sqrt{8}}{2} = \frac{-4 \pm 2\sqrt{2}}{2} = -2 \pm \sqrt{2}$$

$$t = -2 + \sqrt{2} \approx -0.586 \quad \text{or} \quad t = -2 - \sqrt{2} \approx -3.414$$

**Check validity:**
- t = -2 + √2 ≈ -0.586 ✓ (|t| < 1)
- t = -2 - √2 ≈ -3.414 ✗ (|t| > 1)

cos x = -2 + √2:
$$x = 2n\pi \pm \arccos(-2 + \sqrt{2}), \quad n \in \mathbb{Z}$$

---

## 📐 Equations Involving Multiple Functions

### Converting to Single Function

When an equation contains both sin x and cos x, use identities to convert to a single function.

### Example 7: 2cos²x + 3sin x = 3

**Solution:**
Use cos²x = 1 - sin²x:
$$2(1 - \sin^2 x) + 3\sin x = 3$$
$$2 - 2\sin^2 x + 3\sin x = 3$$
$$-2\sin^2 x + 3\sin x - 1 = 0$$
$$2\sin^2 x - 3\sin x + 1 = 0$$

Let t = sin x:
$$2t^2 - 3t + 1 = 0$$
$$(2t - 1)(t - 1) = 0$$
$$t = \frac{1}{2} \quad \text{or} \quad t = 1$$

**Case 1:** sin x = 1/2: x = nπ + (-1)ⁿ(π/6)
**Case 2:** sin x = 1: x = 2nπ + π/2

### Example 8: sin²x - 2cos x + 1 = 0

**Solution:**
Use sin²x = 1 - cos²x:
$$1 - \cos^2 x - 2\cos x + 1 = 0$$
$$-\cos^2 x - 2\cos x + 2 = 0$$
$$\cos^2 x + 2\cos x - 2 = 0$$

Let t = cos x:
$$t^2 + 2t - 2 = 0$$
$$t = \frac{-2 \pm \sqrt{4 + 8}}{2} = \frac{-2 \pm \sqrt{12}}{2} = -1 \pm \sqrt{3}$$

**Check validity:**
- t = -1 + √3 ≈ 0.732 ✓
- t = -1 - √3 ≈ -2.732 ✗

cos x = -1 + √3:
$$x = 2n\pi \pm \arccos(-1 + \sqrt{3})$$

---

## 📊 Common Patterns

### Perfect Square Trinomials

| Equation | Factored Form | Solution |
|----------|---------------|----------|
| sin²x - 2sin x + 1 = 0 | (sin x - 1)² = 0 | sin x = 1 |
| cos²x + 2cos x + 1 = 0 | (cos x + 1)² = 0 | cos x = -1 |
| 4sin²x - 4sin x + 1 = 0 | (2sin x - 1)² = 0 | sin x = 1/2 |

### Difference of Squares

| Equation | Factored Form | Solutions |
|----------|---------------|-----------|
| sin²x - 1 = 0 | (sin x - 1)(sin x + 1) = 0 | sin x = ±1 |
| 4cos²x - 1 = 0 | (2cos x - 1)(2cos x + 1) = 0 | cos x = ±1/2 |
| tan²x - 1 = 0 | (tan x - 1)(tan x + 1) = 0 | tan x = ±1 |

---

## 📐 Visual Approach

### Graphical Interpretation

```
    Finding solutions of 2sin²x - sin x - 1 = 0
    
    y = 2sin²x - sin x - 1
    
    y
    │      
  2 ┤  ╲      ╱     ╲      ╱
    │   ╲    ╱       ╲    ╱
  0 ┼────*──────*────*──────*──── x
    │     ╲  ╱         ╲  ╱
 -2 ┤      ╲╱           ╲╱
    │
    └──────────────────────────────
        π/2   π    3π/2   2π
              ↑      ↑     ↑
         Solutions where curve crosses x-axis
```

---

## 📋 Summary Table

### Solution Process

| Step | Action |
|------|--------|
| 1 | Let t = sin x, cos x, or tan x |
| 2 | Rewrite as at² + bt + c = 0 |
| 3 | Solve quadratic (factor or formula) |
| 4 | Check: \|t\| ≤ 1 for sin/cos |
| 5 | Apply general solution formulas |

### Common Substitutions

| If equation has | Substitute |
|-----------------|------------|
| sin²x and sin x only | t = sin x |
| cos²x and cos x only | t = cos x |
| tan²x and tan x only | t = tan x |
| sin²x and cos x | sin²x = 1 - cos²x, then t = cos x |
| cos²x and sin x | cos²x = 1 - sin²x, then t = sin x |

### Validity Check

| Function | Valid t values |
|----------|----------------|
| sin x = t | -1 ≤ t ≤ 1 |
| cos x = t | -1 ≤ t ≤ 1 |
| tan x = t | All real t |

---

## ❓ Quick Revision Questions

1. **Solve: 2sin²x - 5sin x + 2 = 0 in [0, 2π].**

2. **Solve: 3cos²x - 5cos x + 2 = 0 (general solution).**

3. **Solve: tan²x + tan x - 6 = 0 (general solution).**

4. **Solve: 2cos²x - sin x - 1 = 0 in [0, 2π].**

5. **For the equation cos²x + 2cos x + 3 = 0, explain why there is no solution.**

6. **Solve: sin²x - sin x = 0 (general solution).**

<details>
<summary>Click to see answers</summary>

1. Let t = sin x: 2t² - 5t + 2 = 0  
   (2t - 1)(t - 2) = 0  
   t = 1/2 or t = 2  
   
   sin x = 2 is invalid (|2| > 1)  
   sin x = 1/2: x = π/6, 5π/6 in [0, 2π]  
   **Solutions: x = π/6, 5π/6**

2. Let t = cos x: 3t² - 5t + 2 = 0  
   (3t - 2)(t - 1) = 0  
   t = 2/3 or t = 1  
   
   cos x = 2/3: x = 2nπ ± arccos(2/3)  
   cos x = 1: x = 2nπ  
   **General solution: x = 2nπ ± arccos(2/3) or x = 2nπ**

3. Let t = tan x: t² + t - 6 = 0  
   (t + 3)(t - 2) = 0  
   t = -3 or t = 2  
   
   tan x = -3: x = nπ + arctan(-3) = nπ - arctan(3)  
   tan x = 2: x = nπ + arctan(2)  
   **General solution: x = nπ - arctan(3) or x = nπ + arctan(2)**

4. 2cos²x - sin x - 1 = 0  
   2(1 - sin²x) - sin x - 1 = 0  
   2 - 2sin²x - sin x - 1 = 0  
   -2sin²x - sin x + 1 = 0  
   2sin²x + sin x - 1 = 0  
   (2sin x - 1)(sin x + 1) = 0  
   sin x = 1/2 or sin x = -1  
   
   In [0, 2π]:  
   sin x = 1/2: x = π/6, 5π/6  
   sin x = -1: x = 3π/2  
   **Solutions: x = π/6, 5π/6, 3π/2**

5. Let t = cos x: t² + 2t + 3 = 0  
   Discriminant = 4 - 12 = -8 < 0  
   The quadratic has no real solutions, so there are no values of t.  
   Therefore, **no solution exists** for the original equation.

6. sin²x - sin x = 0  
   sin x(sin x - 1) = 0  
   sin x = 0 or sin x = 1  
   
   sin x = 0: x = nπ  
   sin x = 1: x = 2nπ + π/2  
   **General solution: x = nπ or x = 2nπ + π/2 (or combined: x = nπ/2, n ∈ ℤ)**

</details>

---

## Navigation

| Previous | Up | Next |
|----------|-------|------|
| [← 6.2 Basic Equations](02-basic-equations.md) | [Unit 6 Index](README.md) | [6.4 Advanced Techniques →](04-advanced-techniques.md) |

---

[← Back to Main Index](../README.md)
