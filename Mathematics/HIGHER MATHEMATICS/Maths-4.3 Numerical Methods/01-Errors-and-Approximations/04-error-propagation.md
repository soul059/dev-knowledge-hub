# Chapter 1.4: Error Propagation

[← Previous: Significant Figures](03-significant-figures.md) | [Next: Stability of Algorithms →](05-stability-of-algorithms.md)

---

## Overview

When we perform arithmetic operations on approximate values, the errors in the operands **propagate** into the result. Understanding how errors grow (or shrink) through calculations is crucial for assessing the reliability of final answers.

---

## 1. The Fundamental Idea

If $u = f(x_1, x_2, \ldots, x_k)$ and each $x_i$ has an error $\delta x_i$, then the error in $u$ is:

$$\delta u \approx \frac{\partial f}{\partial x_1} \delta x_1 + \frac{\partial f}{\partial x_2} \delta x_2 + \cdots + \frac{\partial f}{\partial x_k} \delta x_k$$

This comes from the **total differential** (first-order Taylor approximation).

```
  Input errors                    Output error
  ┌────────┐                     ┌────────┐
  │ δx₁    │──┐                  │        │
  ├────────┤  │   ┌──────────┐   │  δu    │
  │ δx₂    │──┼──►│ f(x₁,x₂ │──►│        │
  ├────────┤  │   │  ...,xₖ) │   │ = Σ ∂f │
  │  ...   │──┤   └──────────┘   │   ──·δxᵢ│
  ├────────┤  │                  │   ∂xᵢ  │
  │ δxₖ    │──┘                  └────────┘
  └────────┘
```

---

## 2. Error Propagation in Basic Arithmetic

### Addition: $u = x + y$

$$\delta u = \delta x + \delta y$$

**Absolute errors add.**

$$E_r(u) = \frac{|\delta u|}{|u|} = \frac{|\delta x| + |\delta y|}{|x + y|}$$

### Subtraction: $u = x - y$

$$\delta u = \delta x - \delta y \quad \Rightarrow \quad |\delta u| \leq |\delta x| + |\delta y|$$

**Absolute errors add**, but if $x \approx y$, then $|u| = |x - y|$ is small, making the **relative error large!**

```
  SUBTRACTION DANGER ZONE:

  x = 1.234 ± 0.001     y = 1.233 ± 0.001

  u = x - y = 0.001     |δu| ≤ 0.002

  Relative error = 0.002 / 0.001 = 200% !!
  
  ┌──────────────────────────────────────────┐
  │  Rule: Avoid subtracting nearly equal    │
  │  numbers in numerical computation!       │
  └──────────────────────────────────────────┘
```

### Multiplication: $u = x \cdot y$

$$\frac{\delta u}{u} = \frac{\delta x}{x} + \frac{\delta y}{y}$$

**Relative errors add.**

### Division: $u = x / y$

$$\frac{\delta u}{u} = \frac{\delta x}{x} + \frac{\delta y}{y}$$

**Relative errors add** (same as multiplication).

---

## 3. Summary of Error Propagation Rules

| Operation | Absolute Error | Relative Error |
|-----------|---------------|----------------|
| $u = x + y$ | $\|\delta u\| \leq \|\delta x\| + \|\delta y\|$ | Depends on $\|u\|$ |
| $u = x - y$ | $\|\delta u\| \leq \|\delta x\| + \|\delta y\|$ | Can be very large if $x \approx y$ |
| $u = x \cdot y$ | Complex | $E_r(u) \leq E_r(x) + E_r(y)$ |
| $u = x / y$ | Complex | $E_r(u) \leq E_r(x) + E_r(y)$ |
| $u = x^n$ | $\|\delta u\| = \|nx^{n-1}\| \|\delta x\|$ | $E_r(u) = \|n\| \cdot E_r(x)$ |
| $u = \ln x$ | $\|\delta u\| = \|\delta x / x\|$ | $E_r(u) = E_r(x) / \|\ln x\|$ |
| $u = e^x$ | $\|\delta u\| = e^x \|\delta x\|$ | $E_r(u) = \|x\| \cdot E_a(x)$ |

---

## 4. General Formula for Functions of Multiple Variables

For $u = f(x_1, x_2, \ldots, x_k)$:

### Maximum (Worst-Case) Error

$$|\delta u|_{\max} = \sum_{i=1}^{k} \left|\frac{\partial f}{\partial x_i}\right| |\delta x_i|$$

### Probable (Statistical) Error

If errors are independent and random:

$$|\delta u|_{\text{prob}} = \sqrt{\sum_{i=1}^{k} \left(\frac{\partial f}{\partial x_i}\right)^2 (\delta x_i)^2}$$

```
  Comparison:

  Maximum error:  |δu| ≤ |∂f/∂x₁|·|δx₁| + |∂f/∂x₂|·|δx₂| + ...
                  (all errors conspire in worst direction)

  Probable error: |δu| ≈ √[(∂f/∂x₁·δx₁)² + (∂f/∂x₂·δx₂)² + ...]
                  (errors partially cancel randomly)

  In practice: true error usually lies between the two.
```

---

## 5. Condition Number

The **condition number** of a function measures how sensitive the output is to input perturbations:

$$\kappa = \left|\frac{x \cdot f'(x)}{f(x)}\right|$$

| $\kappa$ Value | Interpretation |
|:-------------:|:--------------:|
| $\kappa \approx 1$ | Well-conditioned |
| $\kappa \gg 1$ | Ill-conditioned |
| $\kappa = \infty$ | Singular problem |

### Examples

| Function | Condition Number $\kappa$ | Well/Ill-Conditioned |
|----------|:------------------------:|:--------------------:|
| $f(x) = x + c$ | $\frac{x}{x+c}$ | Ill if $x \approx -c$ |
| $f(x) = x^n$ | $n$ | Well for small $n$ |
| $f(x) = \sqrt{x}$ | $\frac{1}{2}$ | Always well-conditioned |
| $f(x) = \ln x$ | $\frac{1}{\|\ln x\|}$ | Ill near $x = 1$ |
| $f(x) = \sin x$ | $\|x \cot x\|$ | Ill near $x = n\pi$ |

---

## 6. Worked Examples

### Example 1: Error in Computing Area

A rectangle has measured dimensions $l = 10.0 \pm 0.1$ cm and $w = 5.0 \pm 0.05$ cm.

Area: $A = l \times w = 50.0$ cm²

Error:

$$\delta A = w \cdot \delta l + l \cdot \delta w = 5.0(0.1) + 10.0(0.05) = 0.5 + 0.5 = 1.0 \text{ cm}^2$$

$$E_r(A) = \frac{1.0}{50.0} = 0.02 = 2\%$$

Alternative (relative error addition):

$$E_r(A) = E_r(l) + E_r(w) = \frac{0.1}{10.0} + \frac{0.05}{5.0} = 0.01 + 0.01 = 0.02 = 2\% \checkmark$$

### Example 2: Error in Volume of a Sphere

$V = \frac{4}{3}\pi r^3$, with $r = 5.0 \pm 0.02$ cm.

$$\delta V = 4\pi r^2 \cdot \delta r = 4\pi(25)(0.02) = 6.283 \text{ cm}^3$$

$$V = \frac{4}{3}\pi(125) = 523.6 \text{ cm}^3$$

$$E_r(V) = 3 \cdot E_r(r) = 3 \times \frac{0.02}{5.0} = 0.012 = 1.2\%$$

**Key insight**: For $V = cr^3$, relative error in $V$ is **3 times** relative error in $r$.

### Example 3: Error in a Complex Expression

Compute $u = \frac{x^2 y}{z}$ where $x = 3.0 \pm 0.03$, $y = 2.0 \pm 0.01$, $z = 4.0 \pm 0.02$.

$$E_r(u) = 2 \cdot E_r(x) + E_r(y) + E_r(z)$$

$$= 2\left(\frac{0.03}{3.0}\right) + \frac{0.01}{2.0} + \frac{0.02}{4.0}$$

$$= 2(0.01) + 0.005 + 0.005 = 0.03 = 3\%$$

---

## 7. Error Propagation Flow Diagram

```
  Given: u = (a + b)² / c

  Step 1: s = a + b
           δs = δa + δb         [absolute errors add]

  Step 2: t = s²
           Er(t) = 2·Er(s)      [power rule]
           = 2·(δa + δb)/|a+b|

  Step 3: u = t / c
           Er(u) = Er(t) + Er(c) [division rule]
           = 2·(δa + δb)/|a+b| + δc/|c|

  ┌───────┐     ┌───────┐     ┌───────┐     ┌───────┐
  │ a ± δa│──┐  │       │     │       │     │       │
  └───────┘  ├─►│ s=a+b │────►│ t=s²  │──┐  │       │
  ┌───────┐  │  │δs=δa+δb│    │Er=2Er(s)│  ├─►│ u=t/c │
  │ b ± δb│──┘  └───────┘     └───────┘  │  │Er(u)  │
  └───────┘                              │  │       │
  ┌───────┐                              │  │       │
  │ c ± δc│──────────────────────────────┘  │       │
  └───────┘                                 └───────┘
```

---

## 8. Real-World Applications

| Application | Error Propagation Concern |
|-------------|--------------------------|
| GPS positioning | Errors in satellite signals propagate into position errors |
| Medical imaging | Noise in sensor readings affects reconstructed images |
| Chemical reactions | Measurement errors in concentrations affect yield predictions |
| Circuit analysis | Component tolerances (±5% resistors) affect circuit behavior |
| Survey/mapping | Small angle errors propagate into large distance errors |

---

## Summary Table

| Concept | Formula | Key Insight |
|---------|---------|-------------|
| General error propagation | $\delta u = \sum \frac{\partial f}{\partial x_i} \delta x_i$ | First-order Taylor approx |
| Addition/Subtraction | Absolute errors add | Subtraction magnifies relative error |
| Multiplication/Division | Relative errors add | Convenient for products |
| Power $x^n$ | $E_r(u) = n \cdot E_r(x)$ | Error amplified by power |
| Probable error | $\sqrt{\sum(\partial f/\partial x_i \cdot \delta x_i)^2}$ | More realistic than worst-case |
| Condition number | $\kappa = \|x f'(x)/f(x)\|$ | $\kappa \gg 1$ means ill-conditioned |

---

## Quick Revision Questions

1. **If $u = x^2 - y^2$ and $x = 4.00 \pm 0.01$, $y = 3.00 \pm 0.01$, find the absolute and relative errors in $u$.**

2. **Show that for $u = xy$, the relative error in $u$ equals the sum of relative errors in $x$ and $y$.**

3. **Why is subtraction of nearly equal numbers dangerous? Express this in terms of condition number.**

4. **The radius of a sphere is measured as $r = 10 \pm 0.1$ cm. Find the error in computing the surface area $S = 4\pi r^2$.**

5. **Distinguish between maximum error and probable error. When is each appropriate?**

6. **A function $f(x) = \tan x$ is evaluated near $x = \pi/2$. What is the condition number? Is the problem well-conditioned?**

---

[← Previous: Significant Figures](03-significant-figures.md) | [Next: Stability of Algorithms →](05-stability-of-algorithms.md)
