# Chapter 3.1: System Classification

## Overview

A system is a mathematical model that transforms input signals into output signals. Before analyzing any system, we must identify its properties — these dictate the analytical tools available and the system's behavior guarantees.

---

## 3.1.1 What is a System?

A system $\mathcal{T}$ maps input $x$ to output $y$:

$$y(t) = \mathcal{T}\{x(t)\} \quad \text{(CT)} \qquad y[n] = \mathcal{T}\{x[n]\} \quad \text{(DT)}$$

```
    ┌────────────────┐
    │    System       │
    │   T{·}         │
x(t)───►             ├───► y(t) = T{x(t)}
    │                │
    └────────────────┘

Examples:
  y(t) = 2x(t)              ← Amplifier
  y(t) = x(t-2)             ← Delay line
  y(t) = x²(t)              ← Squarer
  y(t) = dx(t)/dt            ← Differentiator
  y[n] = x[n] + x[n-1]      ← Discrete-time adder
```

---

## 3.1.2 Classification Properties

Systems are classified along **six independent axes**:

```
                    SYSTEM PROPERTIES
                         │
    ┌────────┬───────┬───┴───┬────────┬──────────┐
    │        │       │       │        │          │
 Linear   Time-   Causal  Stable  Memory    Invertible
   vs     Invariant  vs      vs      vs         vs
 Non-     vs        Non-   Unstable Memory-   Non-
 linear  Time-     Causal          less     Invertible
         Variant
```

| Property | Test Criterion | Ideal for Analysis? |
|----------|---------------|---------------------|
| **Linearity** | Superposition: $T\{ax_1 + bx_2\} = aT\{x_1\} + bT\{x_2\}$ | Yes |
| **Time-Invariance** | Shift input → same shift in output | Yes |
| **Causality** | Output depends only on present/past input | Yes (realizable) |
| **Stability** | Bounded input → bounded output (BIBO) | Yes (safe) |
| **Memory** | Output depends on input at times other than $t$ | — |
| **Invertibility** | Distinct inputs produce distinct outputs | — |

### The Golden Combination: LTI

A system that is both **Linear** and **Time-Invariant** is called an **LTI system**. LTI systems are fully characterized by their **impulse response** $h(t)$ and can be analyzed using:
- Convolution
- Fourier Transform
- Laplace Transform / Z-Transform

```
    ┌─────────────┐
    │  LTI System │
    │             │
    │  Fully      │      Characterized by h(t) or H(s) or H(jω)
    │  characterized    
    │  by impulse │      Output = x(t) * h(t)
    │  response   │
    └─────────────┘
```

---

## 3.1.3 Testing Methodology

### General Approach for Each Property

```
1. LINEARITY TEST:
   ┌────────────────────────────────────────────┐
   │ Let x₁(t) → y₁(t) and x₂(t) → y₂(t)     │
   │ Check: T{ax₁ + bx₂} = ay₁ + by₂ ?        │
   │ If YES → Linear; If NO → Non-linear        │
   └────────────────────────────────────────────┘

2. TIME-INVARIANCE TEST:
   ┌────────────────────────────────────────────┐
   │ Let x(t) → y(t)                            │
   │ Delay input: x(t-t₀) → y₁(t)              │
   │ Check: y₁(t) = y(t-t₀) ?                   │
   │ If YES → Time-invariant; If NO → Variant   │
   └────────────────────────────────────────────┘

3. CAUSALITY TEST:
   ┌────────────────────────────────────────────┐
   │ Does y(t₀) depend on x(t) for t > t₀?      │
   │ If NO → Causal; If YES → Non-causal        │
   └────────────────────────────────────────────┘

4. STABILITY TEST (BIBO):
   ┌────────────────────────────────────────────┐
   │ If |x(t)| ≤ M < ∞ for all t,               │
   │ is |y(t)| ≤ N < ∞ for all t?               │
   │ If YES → Stable; If NO → Unstable          │
   └────────────────────────────────────────────┘

5. MEMORY TEST:
   ┌────────────────────────────────────────────┐
   │ Does y(t₀) depend ONLY on x(t₀)?           │
   │ If YES → Memoryless; If NO → Has memory    │
   └────────────────────────────────────────────┘
```

---

## 3.1.4 Quick Classification Examples

| System | Linear? | TI? | Causal? | Stable? | Memory? |
|--------|---------|-----|---------|---------|---------|
| $y(t) = 3x(t)$ | ✓ | ✓ | ✓ | ✓ | No |
| $y(t) = x(t-2)$ | ✓ | ✓ | ✓ | ✓ | Yes |
| $y(t) = x^2(t)$ | ✗ | ✓ | ✓ | ✓* | No |
| $y(t) = tx(t)$ | ✓ | ✗ | ✓ | ✓ | No |
| $y(t) = x(t+1)$ | ✓ | ✓ | ✗ | ✓ | Yes |
| $y(t) = e^{x(t)}$ | ✗ | ✓ | ✓ | ✓ | No |
| $y(t) = \int_{-\infty}^{t} x(\tau)d\tau$ | ✓ | ✓ | ✓ | ✗ | Yes |
| $y[n] = nx[n]$ | ✓ | ✗ | ✓ | ✗ | No |

*For bounded input signals.

---

## 3.1.5 Physical System Examples

### Example 1: Resistor (Memoryless, Linear, TI)

```
    ┌──/\/\/──┐
    │    R    │
v(t)│        │i(t) = v(t)/R
    │        │
    └────────┘

y = x/R → Linear, TI, Causal, Stable, Memoryless
```

### Example 2: Capacitor (Memory, Linear, TI)

```
    ┌───||───┐
    │    C   │
i(t)│        │v(t) = (1/C)∫i(τ)dτ
    │        │
    └────────┘

y depends on past values of x → Has memory
y = (1/C)∫x(τ)dτ → Linear, TI, Causal, Memory
```

### Example 3: Diode (Non-Linear)

```
    ──►|──
    
i(t) = I₀(e^(v/V_T) - 1)  → Non-linear (exponential)
                              TI, Causal, Memoryless
```

---

## 3.1.6 Real-World Applications

| System Type | Example | Key Properties |
|-------------|---------|----------------|
| Audio amplifier | $y = Ax$ | LTI (ideally), Causal, Stable |
| Communication channel | $y = h*x + n$ | LTI (approximately), Causal |
| Image enhancement | Histogram equalization | Non-linear, TI |
| Adaptive filter | LMS algorithm | Linear but Time-Variant |
| Digital thermostat | On/Off control | Non-linear, Causal, Memory |
| Echo canceller | FIR filter | LTI, Causal, Stable |

---

## Summary Table

| Property | Definition | Why It Matters |
|----------|-----------|----------------|
| Linear | Superposition holds | Enables convolution, transforms |
| Time-Invariant | Behavior doesn't change with time | Impulse response is fixed |
| Causal | No future inputs needed | Physically realizable |
| BIBO Stable | Bounded input → bounded output | System won't blow up |
| Memoryless | Output depends only on current input | Simple algebraic relation |
| Invertible | Unique input-output mapping | Can recover input from output |

---

## Quick Revision Questions

1. **What is the superposition principle and why is it important?**
   - $T\{ax_1 + bx_2\} = aT\{x_1\} + bT\{x_2\}$. It enables analysis using decomposition into simpler signals.

2. **What makes LTI systems special compared to other system types?**
   - Completely characterized by impulse response; output = convolution of input and impulse response.

3. **Is the system $y(t) = x(t) + 3$ linear?**
   - No. When $x = 0$, $y = 3 \neq 0$ (violates zero-input/zero-output).

4. **Can a system be both causal and have its output depend on $x(t+1)$?**
   - No. Dependence on $x(t+1)$ means the system needs future input → non-causal.

5. **Name a physical system that is non-linear but time-invariant.**
   - A diode: $i = I_0(e^{v/V_T} - 1)$ — non-linear but the relation doesn't change with time.

---

[← Back to Unit 3](README.md) | [Next: Linear and Non-Linear Systems →](02-linear-and-nonlinear-systems.md)
