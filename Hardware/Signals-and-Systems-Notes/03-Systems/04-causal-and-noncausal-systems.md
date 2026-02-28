# Chapter 3.4: Causal and Non-Causal Systems

## Overview

Causality is a property that determines whether a system can be **physically realized**. A causal system produces output that depends only on present and past inputs — it cannot "see" the future. This is a necessary condition for any real-time system.

---

## 3.4.1 Definition

### Causal System

A system is **causal** if the output at any time $t_0$ depends only on input values at $t \leq t_0$:

$$y(t_0) = f(x(t), t \leq t_0)$$

> The output does NOT depend on **future** values of the input.

### Non-Causal System

A system is **non-causal** if the output depends on **future** values of the input:

$$y(t_0) \text{ depends on } x(t) \text{ for some } t > t_0$$

### Anti-Causal System

A system is **anti-causal** if the output depends ONLY on future inputs:

$$y(t_0) = f(x(t), t \geq t_0)$$

```
CAUSAL:                       NON-CAUSAL:
Uses past & present only      Uses future values too

──────┬──────                ──────┬──────
 past │ future                past │ future
      │                           │
  ████│                       ████│████
  used│ NOT used              used│ also used
      ↑                           ↑
    t = t₀                      t = t₀
```

---

## 3.4.2 Testing for Causality

### Method

Examine the system equation and check: **Does computing $y(t_0)$ require knowledge of $x(t)$ for any $t > t_0$?**

- If NO → **Causal**
- If YES → **Non-Causal**

### Key Patterns

| Pattern in System Equation | Causal? | Reason |
|---------------------------|---------|--------|
| $x(t)$, $x(t-a)$ where $a > 0$ | ✓ | Present and past only |
| $x(t+a)$ where $a > 0$ | ✗ | Requires future input |
| $\int_{-\infty}^{t} x(\tau)d\tau$ | ✓ | Integrates past only |
| $\int_{-\infty}^{\infty} x(\tau)d\tau$ | ✗ | Integrates over all time (including future) |
| $x(2t)$ | ✗ | For $t > 0$: needs $x(2t)$, a future value |
| $x(-t)$ | ✗ | For $t < 0$: needs $x(-t)$ where $-t > 0$ |

---

## 3.4.3 Examples — Causal Systems

### Example 1: $y(t) = x(t)$

Output depends only on present input. **Causal** ✓

### Example 2: $y(t) = x(t-5)$

Output depends on input 5 seconds in the past. **Causal** ✓

### Example 3: $y[n] = \sum_{k=0}^{n} x[k]$ (Accumulator)

Sums from $k = 0$ to current $n$ — only past and present. **Causal** ✓

### Example 4: $y(t) = x(t) + 2x(t-1) - x(t-3)$

Uses $x(t)$, $x(t-1)$, $x(t-3)$ — only present and past. **Causal** ✓

### Example 5: RC Circuit

```
         R
    ──/\/\/──┬──── v_out(t)
             │
    v_in(t)  C
             │
    ─────────┴──── GND

    RC(dv_out/dt) + v_out = v_in

    Output v_out(t) depends on v_in(τ) for τ ≤ t → CAUSAL ✓
    (Capacitor stores past information)
```

---

## 3.4.4 Examples — Non-Causal Systems

### Example 6: $y(t) = x(t + 2)$

At time $t = 0$: $y(0) = x(2)$ — needs input 2 seconds in the **future**. **Non-Causal** ✗

### Example 7: $y[n] = x[n] + x[n+1]$

At $n = 0$: $y[0] = x[0] + x[1]$ — needs the **next** sample. **Non-Causal** ✗

### Example 8: $y(t) = x(-t)$ (Time reversal)

At $t = -1$: $y(-1) = x(1)$ — needs a future value. **Non-Causal** ✗

### Example 9: $y[n] = \frac{1}{2N+1}\sum_{k=-N}^{N} x[n-k]$ (Symmetric moving average)

Uses $x[n+N]$, which is $N$ samples in the future. **Non-Causal** ✗

### Example 10: $y(t) = \int_{t-1}^{t+1} x(\tau) d\tau$

Upper limit is $t + 1$ → requires future input. **Non-Causal** ✗

---

## 3.4.5 Causality of LTI Systems

For an LTI system with impulse response $h(t)$:

> An LTI system is causal if and only if $h(t) = 0$ for $t < 0$.

**Proof**: Output $y(t) = \int_{-\infty}^{\infty} h(\tau)x(t-\tau)d\tau$. If $h(\tau) = 0$ for $\tau < 0$, then:

$$y(t) = \int_{0}^{\infty} h(\tau)x(t-\tau)d\tau$$

Since $t - \tau \leq t$ for $\tau \geq 0$, only past/present input values are used.

```
CAUSAL impulse response:        NON-CAUSAL impulse response:
    │                               │
    │  h(t)                         │    h(t)
  1 ┤  ╲                         1 ┤  ╭──╮
    │   ╲                           │ ╱    ╲
    │    ╲╲                         │╱      ╲
  0 ┤─────╲╲──→ t              0 ┤────────╲──→ t
    ↑                           ↑   ↑
    t=0                         t<0 t=0
    
    h(t) = 0 for t < 0             h(t) ≠ 0 for t < 0
    → CAUSAL                        → NON-CAUSAL
```

### Discrete-Time

An LTI system with $h[n]$ is causal ⟺ $h[n] = 0$ for $n < 0$.

---

## 3.4.6 Making Non-Causal Systems Causal

A non-causal system can often be made causal by introducing a **delay**:

### Example

**Non-causal**: $y[n] = x[n-1] + x[n] + x[n+1]$

**Made causal**: Introduce 1-sample delay → $y'[n] = x[n-2] + x[n-1] + x[n]$

This is the same system but with output delayed by 1. Acceptable in offline (non-real-time) processing.

```
Non-causal:              Made causal (with delay):
    
h[n]:                    h'[n]:
  1 ┤  ↑  ↑  ↑           1 ┤  ↑  ↑  ↑
    │  │  │  │              │  │  │  │
  0 ┼──┼──┼──┼──→ n      0 ┼──┼──┼──┼──→ n
    -1  0  1                 0  1  2
    
  h[-1]≠0 → Non-causal    All h[n]=0 for n<0 → Causal
```

---

## 3.4.7 Physical Realizability

| Property | Realizable in Real-Time? |
|----------|-------------------------|
| Causal | ✓ Yes — can be built with hardware/software |
| Non-Causal | ✗ Not in real-time — requires buffering and delay |
| Anti-Causal | ✗ Only in post-processing (reverse-time) |

### Where Non-Causal Systems Are Used

| Application | Why Non-Causal is OK |
|-------------|---------------------|
| Image processing | All pixels available simultaneously |
| Offline audio editing | Entire recording available |
| Video post-production | All frames available |
| Scientific data analysis | Data already collected |
| Non-real-time filtering | Can buffer and add delay |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Causal | $y(t_0)$ depends only on $x(t)$ for $t \leq t_0$ |
| Non-Causal | $y(t_0)$ depends on $x(t)$ for some $t > t_0$ |
| Anti-Causal | $y(t_0)$ depends only on $x(t)$ for $t \geq t_0$ |
| LTI Causality | $h(t) = 0$ for $t < 0$ |
| Physical realizability | Only causal systems work in real-time |
| Making causal | Add delay to shift non-causal part |
| Indicators of non-causality | $x(t+a)$, $x(-t)$, $x(at)$ for $a > 1$, future limits in integrals |

---

## Quick Revision Questions

1. **Is $y(t) = x(t - 10)$ causal or non-causal?**
   - Causal. It uses the input from 10 seconds **ago** (past).

2. **Is $y[n] = x[2n]$ causal?**
   - Non-causal. At $n = 3$: $y[3] = x[6]$, which is a future value relative to $n = 3$.

3. **For an LTI system, what condition on $h(t)$ ensures causality?**
   - $h(t) = 0$ for all $t < 0$.

4. **Can a non-causal system be implemented in practice?**
   - Yes, but only in offline/non-real-time processing where all data is available beforehand.

5. **Is $y(t) = x(t)\cos(5t)$ causal?**
   - Yes. At any time $t$, only $x(t)$ (present value) is needed.

6. **How can the non-causal system $y[n] = x[n+2]$ be made causal?**
   - Add a delay of at least 2: $y'[n] = x[n+2-2] = x[n]$, or equivalently produce $y[n-2] = x[n]$.

---

[← Previous: Time-Invariant and Time-Variant](03-time-invariant-and-time-variant.md) | [Next: Stable and Unstable Systems →](05-stable-and-unstable-systems.md)
