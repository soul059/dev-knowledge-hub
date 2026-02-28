# Chapter 1.5: Stability of Algorithms

[← Previous: Error Propagation](04-error-propagation.md) | [Next: Unit 2 — Bisection Method →](../02-Solution-of-Equations/01-bisection-method.md)

---

## Overview

An algorithm is **stable** if small perturbations in the input cause only small perturbations in the output. Even mathematically correct algorithms can produce wildly wrong answers if they are **unstable** — errors get amplified at each step rather than damped. This chapter explores numerical stability and its implications for choosing and designing algorithms.

---

## 1. Stability vs. Conditioning

These are related but distinct concepts:

| Concept | Refers to | Question |
|---------|-----------|----------|
| **Conditioning** | The mathematical **problem** | Is the problem sensitive to input changes? |
| **Stability** | The computational **algorithm** | Does the algorithm amplify errors beyond what the problem demands? |

```
  ┌──────────────────────────────────────────────────────────┐
  │                                                          │
  │  Well-conditioned problem + Stable algorithm = Good!     │
  │  Well-conditioned problem + Unstable algorithm = Bad!    │
  │  Ill-conditioned problem  + Stable algorithm = Limited   │
  │  Ill-conditioned problem  + Unstable algorithm = Disaster│
  │                                                          │
  └──────────────────────────────────────────────────────────┘
```

---

## 2. Forward and Backward Error

### Forward Error

The difference between the computed answer and the true answer:

$$\text{Forward Error} = |\tilde{y} - y|$$

where $\tilde{y}$ is computed and $y$ is exact.

### Backward Error

The smallest perturbation $\delta x$ to the input such that the computed answer is the **exact** answer to the perturbed problem:

$$\tilde{y} = f(x + \delta x)$$

$$\text{Backward Error} = |\delta x|$$

```
  Forward analysis:
       x ────► Algorithm ────► ỹ
                                │
                                ▼ compare
       x ────► f(x) = y ◄──────┘
                Forward Error = |ỹ - y|

  Backward analysis:
       x ────────► Algorithm ────► ỹ
       │                           ║
       │                           ║ (same answer)
       ▼                           ║
    x + δx ────► f(x+δx) ─────────╝
       Backward Error = |δx|
```

### Backward Stability

An algorithm is **backward stable** if the backward error is small (comparable to machine precision). This is the gold standard — it means the algorithm gives the exact answer to a slightly perturbed problem.

---

## 3. Stable vs. Unstable Algorithms

### Example: Computing $e^{-x}$ for large $x$

**Method A (Unstable)**: Use the Taylor series directly:

$$e^{-x} = 1 - x + \frac{x^2}{2!} - \frac{x^3}{3!} + \cdots$$

For $x = 20$:
- Individual terms: $1, -20, 200, -1333.3, 6666.7, \ldots$
- Large positive and negative terms cancel to give small result $e^{-20} \approx 2.06 \times 10^{-9}$
- **Catastrophic cancellation!**

**Method B (Stable)**: Compute $e^{x}$ first, then take reciprocal:

$$e^{-x} = \frac{1}{e^{x}}$$

- All terms in $e^x = 1 + x + x^2/2! + \cdots$ are **positive** — no cancellation!
- Then divide: stable operation.

```
  Method A (Unstable):             Method B (Stable):
  
  +6666.7                          Compute e^20 directly:
  -1333.3                          all positive terms
  +200.0                           → 4.85 × 10^8
  -20.0                            
  +1.0                             Then: 1 / (4.85 × 10^8)
  ───────                          = 2.06 × 10^-9 ✓
  ≈ ??? (massive cancellation)     
  
  ERROR!                           ACCURATE!
```

---

## 4. Error Growth: Linear vs. Exponential

### Linear Error Growth (Stable)

Error at step $n$: $E_n \approx c \cdot n \cdot \epsilon$

where $\epsilon$ is the error per step and $c$ is a constant. The total error grows slowly.

### Exponential Error Growth (Unstable)

Error at step $n$: $E_n \approx c \cdot r^n \cdot \epsilon$ with $r > 1$

The error doubles (or more) at each step — quickly becomes enormous.

```
  Error
    │                                    ╱ Exponential (UNSTABLE)
    │                                  ╱
    │                               ╱
    │                           ╱
    │                       ╱
    │                  ╱
    │             ╱
    │         ╱          ╱ Linear (STABLE)
    │      ╱          ╱
    │    ╱         ╱
    │  ╱       ╱
    │╱     ╱
    │  ╱
    ├──────────────────────────────── Step n
```

---

## 5. Stability in Recurrence Relations

Consider computing $I_n = \int_0^1 x^n e^{x-1} dx$ using the recurrence:

$$I_n = 1 - n \cdot I_{n-1}$$

Starting from $I_0 = 1 - e^{-1} \approx 0.6321$.

### Forward Recurrence (Unstable)

| $n$ | Computed $I_n$ | True $I_n$ (approx) | Error |
|:---:|:--------------:|:--------------------:|:-----:|
| 0 | 0.63212 | 0.63212 | 0 |
| 1 | 0.36788 | 0.36788 | 0 |
| 5 | 0.0516 | 0.0516 | ~0 |
| 10 | 0.034 | 0.0484 | 0.014 |
| 15 | -3.2 | 0.0389 | HUGE |
| 20 | 7218 | 0.0326 | DISASTER |

**Why?** The factor $n$ amplifies errors at each step: $\delta I_n = n \cdot \delta I_{n-1}$. After $n$ steps: $\delta I_n \approx n! \cdot \delta I_0$.

### Backward Recurrence (Stable)

Start from a rough estimate of $I_N$ for large $N$ and compute backward:

$$I_{n-1} = \frac{1 - I_n}{n}$$

Now the factor $\frac{1}{n}$ **damps** errors at each step!

```
  Forward:   error × n at each step  →  n! amplification  →  UNSTABLE
  Backward:  error ÷ n at each step  →  1/n! damping      →  STABLE
```

---

## 6. Stability of Differential Equation Solvers

Consider $y' = -10y$, $y(0) = 1$ with Euler's method:

$$y_{n+1} = y_n + h(-10 y_n) = (1 - 10h) y_n$$

For stability, we need $|1 - 10h| < 1$:

$$-1 < 1 - 10h < 1 \implies 0 < h < 0.2$$

```
  h = 0.1:  multiplier = 1 - 10(0.1) = 0     → stable (fast!)
  h = 0.15: multiplier = 1 - 10(0.15) = -0.5  → stable (oscillates)
  h = 0.19: multiplier = 1 - 10(0.19) = -0.9  → stable (barely)
  h = 0.21: multiplier = 1 - 10(0.21) = -1.1  → UNSTABLE!

  Stability Region for Euler's Method:
  
  Step size h
  ├───────┼───────┼───────┼───────┤
  0     0.05    0.1    0.15    0.2
  ◄────── STABLE REGION ──────────►
                                    ◄── UNSTABLE
```

---

## 7. Tips for Writing Stable Algorithms

| Principle | Example |
|-----------|---------|
| Avoid subtracting nearly equal numbers | Use $\frac{1}{\sqrt{x+1}+\sqrt{x}}$ instead of $\sqrt{x+1} - \sqrt{x}$ |
| Sum small numbers first | Add from smallest to largest to reduce round-off |
| Use mathematically equivalent but numerically stable forms | $e^{-x} = 1/e^x$ for large $x$ |
| Prefer backward recurrence when forward is unstable | $I_{n-1} = (1 - I_n)/n$ |
| Check stability conditions before choosing step size | Euler: $h < 2/|λ|$ for $y' = λy$ |
| Use pivoting in elimination methods | Partial pivoting in Gaussian elimination |

---

## 8. Real-World Applications

| Application | Stability Concern |
|-------------|-------------------|
| Weather simulation | Unstable time-stepping → solution blows up |
| Structural FEA | Ill-conditioned stiffness matrices |
| Control systems | Unstable digital filters cause oscillations |
| Molecular dynamics | Long simulations accumulate error; need symplectic integrators |
| Image processing | Deconvolution is ill-conditioned (regularization needed) |

---

## Summary Table

| Concept | Description |
|---------|-------------|
| **Stability** | Algorithm doesn't amplify errors beyond what the problem demands |
| **Conditioning** | Sensitivity of the problem itself to input perturbations |
| **Forward error** | $\|\tilde{y} - y\|$ — distance from true answer |
| **Backward error** | Size of input perturbation that explains computed output |
| **Backward stability** | Backward error $\approx$ machine epsilon |
| **Linear growth** | $E_n = O(n\epsilon)$ — acceptable |
| **Exponential growth** | $E_n = O(r^n \epsilon), r > 1$ — unstable |
| **Stability condition** | Problem-specific bound on step size / parameters |

---

## Quick Revision Questions

1. **Distinguish between the conditioning of a problem and the stability of an algorithm. Give an example where a stable algorithm is applied to an ill-conditioned problem.**

2. **What is backward stability? Why is it considered the gold standard for numerical algorithms?**

3. **Explain why computing $e^{-20}$ via the Taylor series of $e^{-x}$ is unstable, and how computing $1/e^{20}$ avoids the issue.**

4. **For the recurrence $I_n = 1 - nI_{n-1}$, explain why forward computation is unstable and backward computation is stable.**

5. **For Euler's method applied to $y' = -50y$, what is the maximum step size for stability?**

6. **List three practical strategies for writing numerically stable algorithms.**

---

[← Previous: Error Propagation](04-error-propagation.md) | [Next: Unit 2 — Bisection Method →](../02-Solution-of-Equations/01-bisection-method.md)
