# Chapter 3.3: Time-Invariant and Time-Variant Systems

## Overview

A time-invariant system behaves identically regardless of **when** the input is applied. If you delay the input, the output is delayed by exactly the same amount — the system's characteristics don't change over time. This property, combined with linearity, gives us the powerful LTI framework.

---

## 3.3.1 Definition

A system is **time-invariant (TI)** if a time shift in the input causes an identical time shift in the output:

$$\text{If } x(t) \to y(t), \text{ then } x(t - t_0) \to y(t - t_0) \quad \forall t_0$$

A system that does not satisfy this is **time-variant (TV)**.

```
TIME-INVARIANT SYSTEM:

    x(t)      ──►[System]──►  y(t)
    x(t - t₀) ──►[System]──►  y(t - t₀)    ← SAME delay in output

TIME-VARIANT SYSTEM:

    x(t)      ──►[System]──►  y(t)
    x(t - t₀) ──►[System]──►  y₁(t) ≠ y(t - t₀)    ← DIFFERENT output
```

---

## 3.3.2 Step-by-Step Testing Procedure

### Method

1. Find the output $y(t)$ for input $x(t)$
2. Compute $y(t - t_0)$ — this is the delayed output (what we expect)
3. Replace $x(t)$ with $x(t - t_0)$ in the system equation — compute $y_1(t)$ (actual output for delayed input)
4. Compare: if $y_1(t) = y(t - t_0)$ for all $t_0$ → Time-Invariant

```
TESTING FLOWCHART:

Step 1: x(t) ──►[T]──► y(t)
Step 2: Compute y(t - t₀) by replacing t with (t - t₀) in y(t)
Step 3: x(t - t₀) ──►[T]──► y₁(t) by replacing x(t) with x(t-t₀) in system equation
Step 4: y₁(t) = y(t - t₀)?
         ╱           ╲
       YES            NO
        │              │
   TIME-INVARIANT  TIME-VARIANT
```

---

## 3.3.3 Examples — Time-Invariant Systems

### Example 1: $y(t) = 3x(t)$

- Step 1: $y(t) = 3x(t)$
- Step 2: $y(t - t_0) = 3x(t - t_0)$
- Step 3: Input $x(t-t_0)$ → $y_1(t) = 3x(t - t_0)$
- Step 4: $y_1(t) = y(t-t_0)$ ✓ → **Time-Invariant** ✓

### Example 2: $y(t) = x(t - 5)$

- Step 2: $y(t - t_0) = x(t - t_0 - 5)$
- Step 3: $y_1(t) = x((t - t_0) - 5) = x(t - t_0 - 5)$
- Step 4: Equal ✓ → **Time-Invariant** ✓

### Example 3: $y(t) = \int_{-\infty}^{t} x(\tau)d\tau$

- Step 2: $y(t-t_0) = \int_{-\infty}^{t-t_0} x(\tau)d\tau$
- Step 3: $y_1(t) = \int_{-\infty}^{t} x(\tau - t_0)d\tau$. Let $u = \tau - t_0$: $= \int_{-\infty}^{t - t_0} x(u)du$
- Step 4: Equal ✓ → **Time-Invariant** ✓

### Example 4: $y[n] = x[n] - x[n-1]$ (First difference)

- Step 2: $y[n - n_0] = x[n-n_0] - x[n-n_0-1]$
- Step 3: $y_1[n] = x[n-n_0] - x[(n-n_0)-1] = x[n-n_0] - x[n-n_0-1]$
- Step 4: Equal ✓ → **Time-Invariant** ✓

---

## 3.3.4 Examples — Time-Variant Systems

### Example 5: $y(t) = t \cdot x(t)$

- Step 2: $y(t - t_0) = (t - t_0) \cdot x(t - t_0)$
- Step 3: $y_1(t) = t \cdot x(t - t_0)$
- Step 4: $t \cdot x(t-t_0) \neq (t-t_0) \cdot x(t-t_0)$ in general → **Time-Variant** ✗

> The coefficient $t$ explicitly depends on time, changing the system's gain.

### Example 6: $y(t) = x(2t)$ (Time scaling)

- Step 2: $y(t - t_0) = x(2(t - t_0)) = x(2t - 2t_0)$
- Step 3: $y_1(t) = x(2t - t_0)$  ← replacing $x(t)$ with $x(t-t_0)$ gives $x(2t - t_0)$
- Step 4: $x(2t - t_0) \neq x(2t - 2t_0)$ → **Time-Variant** ✗

> Time scaling changes the delay by factor 2, so the shift is not preserved.

### Example 7: $y(t) = x(t) \cdot \cos(\omega_0 t)$

- Step 2: $y(t - t_0) = x(t-t_0) \cos(\omega_0(t-t_0))$
- Step 3: $y_1(t) = x(t-t_0) \cos(\omega_0 t)$
- Step 4: $\cos(\omega_0 t) \neq \cos(\omega_0(t-t_0))$ → **Time-Variant** ✗

> Even though we showed this is linear earlier, it is NOT time-invariant. The modulator is **linear but time-variant (LTV)**.

### Example 8: $y[n] = x[-n]$ (Time reversal)

- Step 2: $y[n - n_0] = x[-(n - n_0)] = x[-n + n_0]$
- Step 3: $y_1[n] = x[-(n - n_0)]$? No: $y_1[n] = x[-n] → $ replace $x[n]$ with $x[n-n_0]$ → $y_1[n] = x[-n - n_0]$
- Step 4: $x[-n - n_0] \neq x[-n + n_0]$ → **Time-Variant** ✗

---

## 3.3.5 Identification Rules (Quick Reference)

### Time-Invariant Indicators

The system equation contains **only** operations that don't depend on absolute time:
- Scaling by constants: $ax(t)$
- Time delay: $x(t - c)$ where $c$ is constant
- Integration/summation with fixed limits relative to $t$
- Differentiation: $dx/dt$

### Time-Variant Indicators

The system equation contains:
- **Explicit time multiplication**: $t \cdot x(t)$, $n \cdot x[n]$
- **Time-varying coefficients**: $a(t) \cdot x(t)$
- **Non-unity time scaling**: $x(at)$, $a \neq 1$ (inside the argument of $x$)
- **Time reversal**: $x(-t)$

| System | TI? | Red Flag |
|--------|-----|----------|
| $y = 5x(t-3)$ | ✓ | None |
| $y = (t+1)x(t)$ | ✗ | $t$ multiplied with $x$ |
| $y = x(3t)$ | ✗ | Time scaling by 3 |
| $y = x(-t+2)$ | ✗ | Reflection |
| $y = x(t)\sin(t)$ | ✗ | Time-varying coefficient |
| $y = x^2(t)$ | ✓ | Non-linear but TI |
| $y = x(t-2)+x(t-5)$ | ✓ | Only fixed delays |

---

## 3.3.6 Physical Interpretation

### Time-Invariant Systems

The system has **fixed parameters** that do not change over time:
- A fixed resistor: $R$ doesn't change → $v = iR$ is TI
- A channel with constant delay → TI
- An amplifier with fixed gain → TI

### Time-Variant Systems

The system parameters **change** with time:
- A fading wireless channel: path loss varies → TV
- A modulator: carrier changes output timing → TV
- An aging component: resistance drifts → TV
- A system with adaptive gain control → TV

```
TIME-INVARIANT:                      TIME-VARIANT:
    │                                    │
    │    ┌─────────────┐                │    ┌─────────────┐
    │    │ Parameters  │                │    │ Parameters  │
    │    │ are FIXED   │                │    │ CHANGE with │
    │    │ (constant)  │                │    │   time      │
    │    └─────────────┘                │    └─────────────┘
    │                                    │
    │  Same input at t=0 or t=100       │  Same input at t=0 or t=100
    │  → Same output shape              │  → DIFFERENT output shape
```

---

## 3.3.7 Timing Diagram: TI vs TV

```
TIME-INVARIANT system: y(t) = x(t-1)

Input x(t):                  Output y(t) = x(t-1):
    │                            │
  1 ┤  ┌──┐                   1 ┤     ┌──┐
    │  │  │                      │     │  │
  0 ┤──┘  └────→ t           0 ┤─────┘  └────→ t
    0  1  2                      0  1  2  3

Input x(t-3):                Output for x(t-3):
    │                            │
  1 ┤        ┌──┐             1 ┤           ┌──┐
    │        │  │                │           │  │
  0 ┤────────┘  └──→ t       0 ┤───────────┘  └──→ t
    0  3  4  5                   0  4  5  6
    
    Shifted by 3 → output also shifted by EXACTLY 3 ✓


TIME-VARIANT system: y(t) = t·x(t)

Input x(t) = u(t)-u(t-1):   Output y(t):
    │                            │
  1 ┤  ┌──┐                   1 ┤     ╱|
    │  │  │                      │    ╱ |
  0 ┤──┘  └────→ t           0 ┤───╱──┘───→ t
    0  1  2                      0  1  2
                                 y = t for 0≤t≤1

Input x(t-2):                Output for x(t-2):
    │                            │
  1 ┤        ┌──┐             3 ┤        ╱|
    │        │  │             2 ┤       ╱ |
  0 ┤────────┘  └──→ t       0 ┤──────╱──┘───→ t
    0  2  3  4                   0  2  3  4
                                 y = t for 2≤t≤3 (DIFFERENT shape!)
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Time-Invariant | $x(t-t_0) \to y(t-t_0)$: shift input → shift output |
| Time-Variant | Input shift ≠ output shift |
| TI indicator | No explicit $t$ in coefficients; no $x(at)$ with $a \neq 1$ |
| TV indicator | $t \cdot x(t)$, $x(at)$, time-varying coefficients |
| LTI system | Linear + Time-Invariant → fully described by $h(t)$ |
| Physical TI | Fixed system parameters |
| Physical TV | Parameters change with time |

---

## Quick Revision Questions

1. **Is $y(t) = x(t/3)$ time-invariant?**
   - No. $y_1(t) = x((t-t_0)/3) \neq x(t/3 - t_0) = y(t-t_0)$. Time scaling → TV.

2. **Is $y[n] = x[n] + 2x[n-1] + x[n-2]$ time-invariant?**
   - Yes. Only fixed delays ($n-1$, $n-2$) — no $n$-dependent coefficients.

3. **A system has $y(t) = e^{-t}x(t)$. Is it TI?**
   - No. $e^{-t}$ is a time-varying coefficient.

4. **Can a system be linear but time-variant?**
   - Yes. Example: $y(t) = t \cdot x(t)$ is linear (superposition holds) but time-variant.

5. **Why is time-invariance important for the convolution approach?**
   - If the system is TI, the impulse response $h(t, \tau) = h(t - \tau)$ depends only on the difference, enabling the standard convolution integral.

6. **Is $y(t) = x(t) + x(t+3)$ time-invariant?**
   - Yes. Both $x(t)$ and $x(t+3)$ involve fixed time shifts — the system equation has no explicit time dependence.

---

[← Previous: Linear and Non-Linear Systems](02-linear-and-nonlinear-systems.md) | [Next: Causal and Non-Causal Systems →](04-causal-and-noncausal-systems.md)
