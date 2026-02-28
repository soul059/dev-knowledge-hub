# Chapter 1.2: Order and Degree

[← Previous: Definition and Terminology](01-definition-and-terminology.md) | [Back to README](../README.md) | [Next: Formation of DEs →](03-formation-of-differential-equations.md)

---

## Overview

The **order** and **degree** of a differential equation are two fundamental classification properties. They determine the number of arbitrary constants in the general solution and influence which solution methods apply.

---

## 1. Order of a Differential Equation

### Definition

> The **order** of a differential equation is the order of the **highest derivative** that appears in the equation.

### Examples

| Equation | Highest Derivative | Order |
|----------|-------------------|-------|
| $\dfrac{dy}{dx} + y = e^x$ | $y'$ | **1** |
| $\dfrac{d^2 y}{dx^2} + 3\dfrac{dy}{dx} + 2y = 0$ | $y''$ | **2** |
| $y''' - 2y'' + y' = \sin x$ | $y'''$ | **3** |
| $\left(\dfrac{dy}{dx}\right)^5 + y = x$ | $y'$ (raised to power 5) | **1** |
| $\dfrac{d^4y}{dx^4} = y^2$ | $y^{(4)}$ | **4** |

> **Key Insight**: The **power** to which a derivative is raised does NOT affect the order. Only the **order of differentiation** matters.

### Visual Hierarchy

```
    Order of DE
    │
    ├── 1st Order:  involves y' (at most)
    │   └── One arbitrary constant in general solution
    │
    ├── 2nd Order:  involves y'' (at most)
    │   └── Two arbitrary constants in general solution
    │
    ├── 3rd Order:  involves y''' (at most)
    │   └── Three arbitrary constants in general solution
    │
    └── nth Order:  involves y⁽ⁿ⁾ (at most)
        └── n arbitrary constants in general solution
```

### Fundamental Principle

> The general solution of an $n$-th order ODE contains exactly **$n$ arbitrary constants**.

This is analogous to integration: each "layer" of differentiation requires one integration to undo, and each integration introduces one constant.

---

## 2. Degree of a Differential Equation

### Definition

> The **degree** of a differential equation is the **power (exponent)** of the highest-order derivative, **after** the equation has been made free of radicals and fractions involving derivatives.

### Prerequisites for Determining Degree

The equation must be:
1. **Polynomial** in all derivatives
2. Free of radicals (square roots, cube roots, etc.) on derivative terms
3. Free of fractions where derivatives appear in the denominator
4. Free of transcendental functions ($\sin$, $\cos$, $e$, $\ln$) of derivatives

> If these conditions cannot be met, the **degree is not defined**.

### Step-by-Step Process

```
Step 1: Identify the highest-order derivative
Step 2: Clear any radicals/fractions involving derivatives
Step 3: Check if equation is polynomial in derivatives
        ├── YES → Degree = power of highest-order derivative
        └── NO  → Degree is NOT DEFINED
```

### Examples with Analysis

#### Example 1: Simple case
$$\frac{d^2y}{dx^2} + 3\frac{dy}{dx} + 2y = 0$$

- Highest derivative: $y''$ (order 2)
- Power of $y''$: 1
- **Order = 2, Degree = 1**

#### Example 2: Power on highest derivative
$$\left(\frac{d^2y}{dx^2}\right)^3 + \left(\frac{dy}{dx}\right)^2 + y = 0$$

- Highest derivative: $y''$ (order 2)
- Power of $y''$: 3
- **Order = 2, Degree = 3**

#### Example 3: Radical that needs clearing
$$\frac{dy}{dx} = \sqrt{1 + \frac{d^2y}{dx^2}}$$

**Step 1**: Square both sides to remove the radical:
$$(y')^2 = 1 + y''$$

**Step 2**: Rearrange:
$$y'' - (y')^2 + 1 = 0$$

- Highest derivative: $y''$ (order 2)
- Power of $y''$: 1
- **Order = 2, Degree = 1**

#### Example 4: Degree not defined
$$\frac{d^2y}{dx^2} + \sin\left(\frac{dy}{dx}\right) = 0$$

- Contains $\sin(y')$ — a transcendental function of a derivative
- Cannot be made polynomial in derivatives
- **Order = 2, Degree = NOT DEFINED**

#### Example 5: Fractional powers
$$\left(\frac{d^2y}{dx^2}\right)^{2/3} = \frac{dy}{dx} + 1$$

**Step 1**: Raise both sides to power 3:
$$(y'')^2 = (y' + 1)^3$$

- Now polynomial in derivatives
- Highest derivative: $y''$, power = 2
- **Order = 2, Degree = 2**

---

## 3. Classification Table

| Equation | Order | Degree | Linear? |
|----------|-------|--------|---------|
| $y' + 2y = x$ | 1 | 1 | Yes |
| $(y')^3 + y = 0$ | 1 | 3 | No |
| $y'' + y' + y = 0$ | 2 | 1 | Yes |
| $(y'')^2 + (y')^3 = \sin x$ | 2 | 2 | No |
| $y''' + xy = e^x$ | 3 | 1 | Yes |
| $e^{y'} + y = x$ | 1 | N/D | No |
| $\sqrt{y''} + y' = 0 \Rightarrow y'' + (y')^2 = 0$ | 2 | 1 | No |

> **Important Pattern**: All linear DEs have degree 1, but not all degree-1 DEs are linear.

---

## 4. Relationship Between Order, Degree, and Solution

```
                        ┌─────────────┐
                        │   Given DE  │
                        └──────┬──────┘
                               │
                    ┌──────────┴──────────┐
                    │                     │
              Order = n              Degree = d
                    │                     │
        n arbitrary constants      d determines
        in general solution       solution method
                    │                complexity
                    │                     │
           ┌───────┴───────┐     ┌───────┴───────┐
           │               │     │               │
        n = 1           n ≥ 2   d = 1          d > 1
     1 constant      n consts  Easier        Harder
     First Order     Higher    methods       methods
                     Order
```

### Key Results

| Order $n$ | Number of Constants | Example General Solution |
|-----------|-------------------|--------------------------|
| 1 | 1 | $y = Ce^x$ |
| 2 | 2 | $y = C_1 e^x + C_2 e^{-x}$ |
| 3 | 3 | $y = C_1 e^x + C_2 e^{2x} + C_3 e^{3x}$ |
| $n$ | $n$ | $y = \sum_{i=1}^{n} C_i \phi_i(x)$ |

---

## 5. Common Pitfalls

### Pitfall 1: Confusing Power with Order
$$\left(\frac{dy}{dx}\right)^4 + y = 0$$

- This is **order 1** (highest derivative is $y'$), NOT order 4
- The exponent 4 is the **degree**, not the order

### Pitfall 2: Forgetting to Clear Radicals
$$y' = \sqrt{y'' + 1}$$

- **Before clearing**: You might incorrectly say order = 1
- **After squaring**: $(y')^2 = y'' + 1$ → order = 2, degree = 1

### Pitfall 3: Degree of a Linear DE
- Every **linear** ODE has degree **1** (by definition, derivatives appear to first power)
- But degree 1 does NOT imply linearity: $y'' + (y')y = 0$ has degree 1 but is non-linear

### Pitfall 4: Implicit Derivatives
$$x \cdot y' = y + \frac{1}{y'}$$

Multiply by $y'$: $x(y')^2 = yy' + 1$

- Now it's clear: Order = 1, Degree = 2

---

## 6. Worked Examples

### Example A
**Classify**: $\left(\dfrac{d^3y}{dx^3}\right)^2 + 5\dfrac{d^2y}{dx^2} + 6\dfrac{dy}{dx} + y = e^x$

**Solution**:
- Highest derivative: $y'''$ → **Order = 3**
- Power of $y'''$: 2 → **Degree = 2**
- Contains $(y''')^2$ → **Non-linear**

### Example B
**Classify**: $\dfrac{d^2y}{dx^2} = \left[1 + \left(\dfrac{dy}{dx}\right)^2\right]^{3/2}$

**Solution**:
- Square both sides: $(y'')^2 = [1 + (y')^2]^3$
- Expand: $(y'')^2 = 1 + 3(y')^2 + 3(y')^4 + (y')^6$
- Polynomial in derivatives ✓
- Highest derivative: $y''$ → **Order = 2**
- Power of $y''$: 2 → **Degree = 2**
- **Non-linear** (due to $(y')^6$ and $(y'')^2$)

> This is the equation for the **radius of curvature**: $\kappa = \dfrac{y''}{[1+(y')^2]^{3/2}}$

---

## Summary Table

| Concept | Key Point |
|---------|----------|
| **Order** | Order of the highest derivative present |
| **Degree** | Exponent of the highest-order derivative (after clearing radicals) |
| **Degree exists** | Only when equation is polynomial in all derivatives |
| **Degree undefined** | When $\sin(y')$, $e^{y''}$, $\ln(y')$, etc. are present |
| **Linear → Degree** | All linear DEs have degree 1 |
| **Degree 1 → Linear?** | Not necessarily (e.g., $y'' + yy' = 0$) |
| **Constants** | $n$-th order DE → $n$ arbitrary constants in general solution |
| **Clear radicals first** | Always simplify before determining degree |

---

## Quick Revision Questions

1. What is the order and degree of $\left(\dfrac{d^2y}{dx^2}\right)^4 + \left(\dfrac{dy}{dx}\right)^6 + y = 0$?

2. Why is the degree of $y'' + e^{y'} = 0$ not defined?

3. A differential equation has 5 arbitrary constants in its general solution. What is its order?

4. Find the order and degree of $\sqrt{\dfrac{d^2y}{dx^2}} + \dfrac{dy}{dx} + y = 0$ (after rationalizing).

5. True or False: "If the degree of a DE is 1, then it must be linear." Explain.

6. Classify the equation $\left(\dfrac{d^4y}{dx^4}\right)^3 + x^2 y = \sin x$ by order, degree, and linearity.

---

[← Previous: Definition and Terminology](01-definition-and-terminology.md) | [Back to README](../README.md) | [Next: Formation of DEs →](03-formation-of-differential-equations.md)
