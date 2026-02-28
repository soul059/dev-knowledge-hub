# Chapter 3.6: Memory and Memoryless Systems

## Overview

A system's output may depend **only on the current input** or **also on past/future inputs**. This distinction is fundamental — memoryless systems are simpler to analyze and implement, while systems with memory can perform filtering, integration, and other powerful operations.

---

## 3.6.1 Definitions

### Memoryless (Static) System

A system is **memoryless** if the output at any time $t_0$ depends **only** on the input at **that same time** $t_0$:

$$y(t_0) = f\big(x(t_0)\big) \qquad \text{(CT)}$$
$$y[n_0] = f\big(x[n_0]\big) \qquad \text{(DT)}$$

### System with Memory (Dynamic) System

A system **has memory** if the output at time $t_0$ depends on input values at times **other than** $t_0$:

$$y(t_0) = f\big(x(t), \; t \neq t_0 \text{ also}\big)$$

```
MEMORYLESS SYSTEM:                SYSTEM WITH MEMORY:
                                  
  x(t₀) ──→ ┌────────┐ → y(t₀)    x(t) ──→ ┌────────┐ → y(t₀)
              │ y = f(x)│           for all   │        │
              └────────┘            relevant  │ Memory │
                                    t values  │ Block  │
  Uses ONLY x(t₀)                             └────────┘
                                    Uses x(t₀), x(t₀-1), ...
```

---

## 3.6.2 Identifying Memory — Key Patterns

| Pattern | Memory? | Reason |
|---------|---------|--------|
| $y(t) = a \cdot x(t)$ | No | Only current input |
| $y(t) = x(t)^2$ | No | Only current input |
| $y(t) = x(t-1)$ | Yes | Uses past input |
| $y(t) = x(2t)$ | Depends* | At $t=3$, uses $x(6)$ — future value |
| $y(t) = \int_{-\infty}^{t} x(\tau) d\tau$ | Yes | Uses all past inputs |
| $y[n] = x[n] + x[n-1]$ | Yes | Uses current AND past |
| $y[n] = n \cdot x[n]$ | No | Only current input (scaled by index) |
| $y(t) = x(t)\cos(\omega t)$ | No | Only current input × time function |
| $y[n] = \max\{x[n], x[n-1]\}$ | Yes | Uses current and past |
| $y(t) = \frac{dx(t)}{dt}$ | Yes | Derivative requires nearby values |

*Time scaling can reference other time instants, implying memory.

---

## 3.6.3 Examples — Memoryless Systems

### Example 1: Resistor — $y(t) = Rx(t)$

$$v(t) = R \cdot i(t)$$

Current at time $t$ → voltage at time $t$. No past values needed. **Memoryless** ✓

```
  i(t) ──→ ┌───┐ ──→ v(t) = R·i(t)
            │ R │
            └───┘
  
  Output depends ONLY on current input
```

### Example 2: Squarer — $y(t) = x^2(t)$

At any $t_0$: $y(t_0) = [x(t_0)]^2$. Only uses $x(t_0)$. **Memoryless** ✓

### Example 3: Full-Wave Rectifier — $y(t) = |x(t)|$

$$y(t_0) = |x(t_0)|$$

Only the current sample matters. **Memoryless** ✓

### Example 4: Quantizer — $y[n] = Q(x[n])$

Each sample is independently rounded/quantized. **Memoryless** ✓

### Example 5: $y(t) = \cos(x(t))$

Any function $f(\cdot)$ applied to only $x(t)$ makes the system memoryless. **Memoryless** ✓

---

## 3.6.4 Examples — Systems with Memory

### Example 6: Ideal Delay — $y(t) = x(t - T)$

Output at time $t$ depends on input at time $t - T$ (past). **Has Memory** ✓

```
  t = 5, T = 2:
  y(5) = x(3)  ← needs to "remember" x from 2 seconds ago
  
  Input:       x: ──╲╱──╲╱──╲╱──
               t:  1  2  3  4  5
  
  Output:      y: ────╲╱──╲╱──╲╱
               t:  1  2  3  4  5   (shifted right by T=2)
```

### Example 7: Integrator — $y(t) = \int_{-\infty}^{t} x(\tau) d\tau$

Output depends on **all past** values of input. **Has Memory** ✓

### Example 8: Moving Average — $y[n] = \frac{1}{3}(x[n] + x[n-1] + x[n-2])$

Uses current and two previous samples. **Has Memory** ✓

```
  Computing y[5]:
  
  x: ... x[3]  x[4]  x[5] ...
          │      │      │
          ↓      ↓      ↓
        ┌────────────────┐
        │  (x[3]+x[4]+x[5])/3 │
        └────────────────┘
                 │
                 ↓
               y[5]
  
  Needs to store past 2 samples → MEMORY
```

### Example 9: Differentiator — $y(t) = \frac{dx(t)}{dt}$

$$y(t) = \lim_{\Delta \to 0} \frac{x(t) - x(t - \Delta)}{\Delta}$$

Requires knowledge of $x$ at nearby past instants. **Has Memory** ✓

### Example 10: Accumulator — $y[n] = \sum_{k=-\infty}^{n} x[k]$

Depends on all past and current samples. **Has Memory** ✓

---

## 3.6.5 LTI Systems and Memory

For an **LTI system**, the memory property is determined by the impulse response:

| Impulse Response | Memory? |
|------------------|---------|
| $h(t) = a\delta(t)$ | Memoryless |
| $h(t) \neq a\delta(t)$ | Has Memory |

> An LTI system is memoryless **if and only if** $h(t) = K\delta(t)$ for some constant $K$.

**Proof**: If $h(t) = K\delta(t)$:

$$y(t) = x(t) * K\delta(t) = Kx(t)$$

The output depends only on $x(t)$ → memoryless.

If $h(t)$ has any nonzero value for $t \neq 0$, convolution brings in $x$ at other time instants → memory.

---

## 3.6.6 Physical Interpretation

### Why Memory Matters

| Memoryless | With Memory |
|------------|-------------|
| Instantaneous processing | Retains information over time |
| No storage elements needed | Requires capacitors, flip-flops, registers |
| Resistive circuits | RC, RL, RLC circuits |
| Cannot filter | Can perform filtering |
| Cannot differentiate or integrate | Can compute derivatives and integrals |

### Storage Elements in Hardware

```
MEMORYLESS CIRCUIT:              CIRCUIT WITH MEMORY:

  ┌───┐                          ┌───┐     ┌───┐
──┤ R ├──  No storage           ──┤ R ├──┬──┤   ├──
  └───┘    element                └───┘  │  └───┘
                                      ┌──┴──┐
  Voltage divider:                    │  C  │  ← Capacitor
  Purely resistive                    └─────┘    stores charge
                                
                                  RC circuit: output
                                  depends on past input
```

---

## 3.6.7 Flowchart for Determining Memory

```
             START
               │
               ▼
      ┌────────────────┐
      │ Write y in     │
      │ terms of x     │
      └───────┬────────┘
              │
              ▼
   ┌──────────────────────┐
   │ Does y(t₀) depend    │
   │ ONLY on x(t₀)?       │──── YES ──→ MEMORYLESS
   └──────────┬───────────┘
              │ NO
              ▼
   ┌──────────────────────┐
   │ Does y(t₀) use:      │
   │ • x(t₀ ± d) ?        │
   │ • integral of x ?    │
   │ • derivative of x ?  │──── YES ──→ HAS MEMORY
   │ • sum over past x ?  │
   └──────────────────────┘
```

---

## 3.6.8 Tricky Cases

### Case 1: $y(t) = x(t) \cdot t$

$y(t_0) = x(t_0) \cdot t_0$. Only uses $x(t_0)$. The $t$ is just a time-varying coefficient. **Memoryless** ✓

### Case 2: $y[n] = x[-n]$

At $n = 3$: $y[3] = x[-3]$. Uses input at a different time. **Has Memory** ✓

### Case 3: $y(t) = x(0)$

For all $t$, the output is $x(0)$ — a constant. At $t \neq 0$, the system must "remember" $x(0)$. **Has Memory** ✓

### Case 4: $y[n] = x[n] \cdot x[n]$

Same as $y[n] = x^2[n]$. Only current sample used. **Memoryless** ✓

### Case 5: $y(t) = x(t) + y(t-1)$ (Recursive)

$y(t)$ depends on previous output, which was computed from earlier inputs. **Has Memory** ✓

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Memoryless | Output at $t_0$ depends **only** on $x(t_0)$ |
| Memory | Output uses past/future input values |
| LTI memoryless | $h(t) = K\delta(t)$ only |
| Derivative | Has memory (needs neighboring values) |
| Integrator/Accumulator | Has memory (needs all past values) |
| Delay $x(t-d)$ | Has memory ($d \neq 0$) |
| $f(x(t))$ | Memoryless for any function $f$ of current input |
| Physical | Memory → storage elements (C, L, registers) |

---

## Quick Revision Questions

1. **Is $y(t) = x(t) + 5$ memoryless?**
   - Yes. Output depends only on $x(t)$ at time $t$.

2. **Is the ideal delay $y[n] = x[n-3]$ a memoryless system?**
   - No. Output at time $n$ depends on input at time $n-3$.

3. **What is the necessary and sufficient condition for an LTI system to be memoryless?**
   - $h(t) = K\delta(t)$ for some constant $K$.

4. **Is a differentiator memoryless or does it have memory?**
   - It has memory — computing the derivative requires values at nearby time instants.

5. **Classify: $y[n] = nx[n] + 2x[n]$**
   - Memoryless. $y[n] = (n+2)x[n]$ — only current input $x[n]$ is used.

6. **Does a purely resistive circuit have memory?**
   - No. Resistors produce instantaneous $v = Ri$ — no energy storage.

---

[← Previous: Stable and Unstable Systems](05-stable-and-unstable-systems.md) | [Next: Unit 4 — LTI Systems →](../04-LTI-Systems/README.md)
