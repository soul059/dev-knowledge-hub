# Chapter 4.4: System Interconnection

## Overview

Complex systems are built by interconnecting simpler subsystems. The three fundamental interconnection types — **cascade** (series), **parallel**, and **feedback** — each have specific rules for computing the equivalent impulse response or transfer function.

---

## 4.4.1 Cascade (Series) Connection

Two LTI systems in **cascade**: the output of the first system feeds into the second.

```
x(t) ──→ ┌────┐ ──→ ┌────┐ ──→ y(t)
          │ h₁ │     │ h₂ │
          └────┘     └────┘

Equivalent:
x(t) ──→ ┌──────────────┐ ──→ y(t)
          │ h = h₁ * h₂  │
          └──────────────┘
```

### Equivalent Impulse Response

$$h_{eq}(t) = h_1(t) * h_2(t)$$

### Equivalent Transfer Function

$$H_{eq}(s) = H_1(s) \cdot H_2(s)$$

> Convolution in time domain → multiplication in frequency domain.

### Properties

- **Order doesn't matter** (commutativity of convolution): $h_1 * h_2 = h_2 * h_1$
- **Multiple stages**: $h_{eq} = h_1 * h_2 * h_3 * \cdots$
- **Stability**: If each subsystem is stable, the cascade is stable

### Example 1

$$h_1(t) = e^{-t}u(t), \quad h_2(t) = e^{-2t}u(t)$$

$$H_1(s) = \frac{1}{s+1}, \quad H_2(s) = \frac{1}{s+2}$$

$$H_{eq}(s) = \frac{1}{(s+1)(s+2)} = \frac{1}{s+1} - \frac{1}{s+2}$$

$$h_{eq}(t) = (e^{-t} - e^{-2t})u(t)$$

---

## 4.4.2 Parallel Connection

Two LTI systems in **parallel**: both receive the same input, outputs are summed.

```
          ┌────┐
    ┌────→│ h₁ │────┐
    │     └────┘    │
x ──┤               ⊕──→ y
    │     ┌────┐    │
    └────→│ h₂ │────┘
          └────┘

Equivalent:
x(t) ──→ ┌──────────────┐ ──→ y(t)
          │ h = h₁ + h₂  │
          └──────────────┘
```

### Equivalent Impulse Response

$$h_{eq}(t) = h_1(t) + h_2(t)$$

### Equivalent Transfer Function

$$H_{eq}(s) = H_1(s) + H_2(s)$$

### Example 2

$$h_1(t) = 3\delta(t), \quad h_2(t) = e^{-t}u(t)$$

$$h_{eq}(t) = 3\delta(t) + e^{-t}u(t)$$

$$H_{eq}(s) = 3 + \frac{1}{s+1} = \frac{3s + 4}{s+1}$$

---

## 4.4.3 Feedback Connection

The output is **fed back** and combined with the input.

### Negative Feedback (most common)

```
              ┌──────────────────────┐
              │                      │
x(t) ──→ ⊕ ──→ ┌──────┐ ──→ y(t)   │
          ↑ -   │  H(s) │            │
          │     └──────┘            │
          │     ┌──────┐            │
          └─────│  G(s) │←───────────┘
                └──────┘
         Feedback path
```

### Closed-Loop Transfer Function

$$T(s) = \frac{H(s)}{1 + H(s)G(s)}$$

**Derivation**:
- Error: $E(s) = X(s) - G(s)Y(s)$
- Output: $Y(s) = H(s)E(s) = H(s)[X(s) - G(s)Y(s)]$
- $Y(s)[1 + H(s)G(s)] = H(s)X(s)$
- $T(s) = Y(s)/X(s) = H(s)/[1 + H(s)G(s)]$

### Unity Feedback ($G(s) = 1$)

$$T(s) = \frac{H(s)}{1 + H(s)}$$

```
Unity feedback:

x ──→ ⊕ ──→ ┌──────┐ ──→ y
      ↑ -    │  H(s) │
      │      └──────┘
      └──────────────────┘
      
T(s) = H(s) / (1 + H(s))
```

### Positive Feedback

Same structure but with positive sign at the summing junction:

$$T(s) = \frac{H(s)}{1 - H(s)G(s)}$$

> Positive feedback can lead to instability if $H(s)G(s) = 1$.

### Example 3

Forward path: $H(s) = \frac{10}{s+1}$, Feedback: $G(s) = 1$ (unity)

$$T(s) = \frac{10/(s+1)}{1 + 10/(s+1)} = \frac{10}{s + 1 + 10} = \frac{10}{s + 11}$$

- Open-loop pole: $s = -1$ (slow response)
- Closed-loop pole: $s = -11$ (faster, more stable response)

```
Effect of feedback on pole location:

  s-plane:
  jω ↑
     │
  ───┼──×──────────×──→ σ
     │  -11        -1
     │   CL         OL
     
  Feedback moves pole LEFT → faster, more stable
```

---

## 4.4.4 Mixed Interconnections

Real systems combine cascade, parallel, and feedback. Simplify step by step.

### Example 4

```
          ┌────┐   ┌────┐
    ┌────→│ h₁ │──→│ h₂ │────┐
    │     └────┘   └────┘    │
x ──┤                        ⊕──→ y
    │     ┌────┐             │
    └────→│ h₃ │─────────────┘
          └────┘
```

**Step 1**: Cascade $h_1$ and $h_2$: $h_{12} = h_1 * h_2$

**Step 2**: Parallel $h_{12}$ and $h_3$: $h_{eq} = h_1 * h_2 + h_3$

$$H_{eq}(s) = H_1(s)H_2(s) + H_3(s)$$

### Example 5: Cascade-Feedback Combination

```
              ┌──────────────────────────────┐
              │                              │
x ──→ ⊕ ──→ ┌────┐ ──→ ┌────┐ ──→ y        │
      ↑ -    │ h₁ │     │ h₂ │              │
      │      └────┘     └────┘              │
      │      ┌────┐                         │
      └──────│ h₃ │←────────────────────────┘
             └────┘
```

Forward path: $H_1(s)H_2(s)$
Feedback: $H_3(s)$

$$T(s) = \frac{H_1(s)H_2(s)}{1 + H_1(s)H_2(s)H_3(s)}$$

---

## 4.4.5 Block Diagram Reduction Rules

| Configuration | Equivalent |
|---------------|------------|
| Cascade: $H_1 \to H_2$ | $H_1 H_2$ |
| Parallel: $H_1 \parallel H_2$ | $H_1 + H_2$ |
| Negative feedback: $H$ with feedback $G$ | $\frac{H}{1+HG}$ |
| Positive feedback | $\frac{H}{1-HG}$ |
| Moving summing point ahead of block $H$ | Multiply other input by $H^{-1}$ |
| Moving takeoff point ahead of block $H$ | Multiply takeoff by $H$ |
| Moving summing point beyond block $H$ | Multiply other input by $H$ |
| Moving takeoff point beyond block $H$ | Multiply takeoff by $H^{-1}$ |

---

## 4.4.6 Inverse Systems

Two systems in cascade form an identity if:

$$h(t) * h_i(t) = \delta(t) \implies H(s) \cdot H_i(s) = 1$$

$$H_i(s) = \frac{1}{H(s)}$$

```
┌────┐   ┌──────┐
│ H  │──→│ 1/H  │──→ δ(t)    Identity system
└────┘   └──────┘

x ──→ [H] ──→ [1/H] ──→ x    Signal recovery
```

### Examples

| System | $H(s)$ | Inverse $H_i(s)$ | Inverse system |
|--------|--------|-------------------|----------------|
| Delay | $e^{-st_0}$ | $e^{st_0}$ | Advance (non-causal!) |
| Integrator | $1/s$ | $s$ | Differentiator |
| First-order | $1/(s+a)$ | $s+a$ | Diff + scale |
| Gain | $K$ | $1/K$ | Attenuator |

---

## 4.4.7 System Properties of Interconnections

| Property | Cascade | Parallel | Feedback |
|----------|---------|----------|----------|
| **Causal** | Both causal → causal | Both causal → causal | Depends on loop |
| **Stable** | Both stable → stable | Both stable → stable | Must analyze CL poles |
| **Linear** | Both linear → linear | Both linear → linear | Both linear → linear |
| **TI** | Both TI → TI | Both TI → TI | Both TI → TI |

---

## 4.4.8 Practical Examples

### Example 6: Audio Equalizer (Parallel)

```
              ┌─────────┐
         ┌───→│ Low Pass │───┐
         │    └─────────┘   │
         │    ┌─────────┐   │
x(t) ───┤───→│ Mid Pass │───⊕──→ y(t)
         │    └─────────┘   │
         │    ┌──────────┐  │
         └───→│ High Pass│───┘
              └──────────┘

H_eq = H_LP + H_MP + H_HP
Each band can be individually adjusted
```

### Example 7: PID Controller (Combined)

```
              ┌─────────┐
         ┌───→│ Kp (P)  │───┐
         │    └─────────┘   │
         │    ┌─────────┐   │
e(t) ───┤───→│ Ki∫ (I)  │───⊕──→ u(t)
         │    └─────────┘   │
         │    ┌─────────┐   │
         └───→│ Kd·d/dt │───┘
              └─────────┘

C(s) = Kp + Ki/s + Kd·s
     = (Kd·s² + Kp·s + Ki) / s
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Cascade | $h_{eq} = h_1 * h_2$; $H_{eq} = H_1 \cdot H_2$ |
| Parallel | $h_{eq} = h_1 + h_2$; $H_{eq} = H_1 + H_2$ |
| Negative feedback | $T = H/(1+HG)$ |
| Positive feedback | $T = H/(1-HG)$ |
| Inverse system | $H \cdot H_i = 1$ |
| Cascade order | Doesn't matter (commutativity) |
| Stability | Cascade/parallel: both stable → stable |
| Block diagram | Reduce systematically using rules |

---

## Quick Revision Questions

1. **What is the equivalent impulse response of two systems in parallel?**
   - Sum: $h_{eq} = h_1 + h_2$.

2. **What is the closed-loop transfer function for unity negative feedback?**
   - $T(s) = H(s)/[1 + H(s)]$.

3. **Can the cascade order of two LTI systems be swapped?**
   - Yes, due to commutativity of convolution.

4. **If $H(s) = 5/(s+3)$, what is the inverse system?**
   - $H_i(s) = (s+3)/5$ → a differentiator-type system with scaling.

5. **Two stable systems in cascade — is the result always stable?**
   - Yes. $\int|h_1 * h_2| \leq \int|h_1| \cdot \int|h_2| < \infty$.

6. **Draw the equivalent for a system $H$ in negative feedback with $G$, then cascaded with $H_3$.**
   - $T(s) = \frac{H(s) \cdot H_3(s)}{1 + H(s)G(s)}$.

---

[← Previous: Properties of LTI Systems](03-properties-of-lti-systems.md) | [Next: Unit 5 — Fourier Series →](../05-Fourier-Series/README.md)
