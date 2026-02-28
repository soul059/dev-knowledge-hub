# Chapter 2.1: Time Shifting and Scaling

## Overview

Time shifting and time scaling are the two most fundamental independent-variable operations on signals. Combined operations (shift + scale + reversal) appear frequently in convolution, modulation, and system analysis. This chapter provides a rigorous, step-by-step methodology.

---

## 2.1.1 Time Shifting — Detailed Treatment

### Continuous-Time

$$y(t) = x(t - t_0)$$

| Condition | Effect | Physical Meaning |
|-----------|--------|-----------------|
| $t_0 > 0$ | Shift **right** by $t_0$ | **Delay**: signal arrives later |
| $t_0 < 0$ | Shift **left** by $\|t_0\|$ | **Advance**: signal arrives earlier |

### How Shifting Works

If $x(t)$ has a feature (peak, zero-crossing, edge) at time $t = t_1$, then $x(t - t_0)$ has that same feature at $t = t_1 + t_0$.

**Proof**: For $x(t - t_0)$ to equal $x(t_1)$, we need $t - t_0 = t_1$, so $t = t_1 + t_0$.

```
Example: x(t) defined on [1, 4], peak at t = 2

x(t):                   x(t - 3):  [shift right by 3]
    │                       │
  2 ┤     ╭╮             2 ┤              ╭╮
    │    ╱  ╲               │             ╱  ╲
  1 ┤   ╱    ╲           1 ┤            ╱    ╲
    │  ╱      ╲             │           ╱      ╲
  0 ┤─╱────────╲──→ t    0 ┤──────────╱────────╲──→ t
    0  1  2  3  4  5        0  1  2  3  4  5  6  7
       ↑  ↑     ↑                       ↑  ↑     ↑
     start peak end                   start peak end
     1→4  2→5  4→7  (all shifted by +3)
```

### Discrete-Time Shifting

$$y[n] = x[n - n_0]$$

Every sample index shifts by $n_0$ ($n_0$ must be integer for DT).

```
x[n] = {2, 3, 1, 4} at n = 0,1,2,3

x[n]:                    x[n-2]:
  4 ┤              ↑      4 ┤                    ↑
  3 ┤     ↑               3 ┤           ↑
  2 ┤  ↑                  2 ┤        ↑
  1 ┤        ↑             1 ┤              ↑
  0 ┼──┼──┼──┼──┼──→ n    0 ┼──┼──┼──┼──┼──┼──┼──→ n
     0  1  2  3              0  1  2  3  4  5
```

### Physical Realization: Delay Line

```
                    ┌─────────────┐
    x(t) ──────────►│  Delay = t₀  ├──────────► x(t - t₀)
                    │   (z⁻¹ for  │
                    │   DT: x[n-1]│
                    └─────────────┘

In digital systems, each z⁻¹ block represents a one-sample delay:

    x[n] ──►[z⁻¹]──►[z⁻¹]──►[z⁻¹]──► x[n-3]
              │         │        │
            x[n-1]   x[n-2]   x[n-3]
```

---

## 2.1.2 Time Scaling — Detailed Treatment

### Continuous-Time

$$y(t) = x(at), \quad a \neq 0$$

| Condition | Effect | Duration Change |
|-----------|--------|----------------|
| $\|a\| > 1$ | **Compression** | Duration ÷ $\|a\|$ |
| $0 < \|a\| < 1$ | **Expansion** | Duration × $1/\|a\|$ |
| $a < 0$ | **Reversal** included | Mirror + scale |

### Core Principle

> Every time point $t_k$ in $x(t)$ maps to $t_k/a$ in $x(at)$.

```
x(t) defined on [0, 6]:       x(2t) [compress by 2]:     x(t/2) [expand by 2]:
    │                              │                           │
  1 ┤  ┌──────────┐            1 ┤  ┌─────┐               1 ┤  ┌────────────────────┐
    │  │          │              │  │     │                 │  │                    │
  0 ┤──┘          └──→ t     0 ┤──┘     └────→ t       0 ┤──┘                    └──→ t
    0  1  2  3  4  5  6          0  1  2  3                 0  2  4  6  8  10  12
    │                              │                           │
    │  Width = 6                   │  Width = 3                │  Width = 12
    │                              │  (6 ÷ 2)                 │  (6 × 2)
```

### Effect on Signal Properties

| Property | $x(t)$ | $x(at)$, $\|a\| > 1$ |
|----------|--------|----------------------|
| Duration | $D$ | $D/\|a\|$ |
| Frequency content | $\omega$ | $\|a\|\omega$ (frequencies scale up) |
| Energy | $E$ | $E/\|a\|$ |
| Area | $A$ | $A/\|a\|$ |

$$\int_{-\infty}^{\infty} x(at) \, dt = \frac{1}{|a|} \int_{-\infty}^{\infty} x(\tau) \, d\tau$$

### Discrete-Time Scaling: Decimation and Interpolation

DT scaling is fundamentally different because $x[an]$ is only defined when $an$ is an integer.

**Decimation (Downsampling by $M$)**: $y[n] = x[Mn]$

- Keep every $M$-th sample, discard the rest
- Information is **lost** (potential aliasing)

```
x[n]:                     y[n] = x[2n] (downsample by 2):
  4 ┤  ↑     ↑  ↑          4 ┤  ↑        ↑
  3 ┤     ↑        ↑       3 ┤
  2 ┤                       2 ┤     ↑
  1 ┤                       1 ┤
  0 ┼──┼──┼──┼──┼──┼──→    0 ┼──┼──┼──┼──→
     0  1  2  3  4  5         0  1  2  3
     ↑     ↑     ↑            Keep n=0,2,4 only
```

**Interpolation (Upsampling by $L$)**: Insert $L-1$ zeros between samples

```
x[n]:                    x_up[n] (upsample by 2, zero-insert):
  3 ┤  ↑                 3 ┤  ↑
  2 ┤     ↑  ↑            2 ┤        ↑     ↑
  1 ┤              ↑      1 ┤                    ↑
  0 ┼──┼──┼──┼──┼──→     0 ┼──┼──┼──┼──┼──┼──┼──┼──→
     0  1  2  3              0  1  2  3  4  5  6  7
                             Zeros inserted at odd indices
```

---

## 2.1.3 Time Reversal

$$y(t) = x(-t) \qquad \text{or} \qquad y[n] = x[-n]$$

Time reversal is a special case of scaling with $a = -1$.

### Effect

Every time point $t_k$ maps to $-t_k$: the signal is **mirrored** about the vertical axis.

```
x(t) on [1, 5]:              x(-t) on [-5, -1]:
    │                              │
  2 ┤    ╱╲                     2 ┤            ╱╲
    │   ╱  ╲                      │           ╱  ╲
  1 ┤  ╱    ╲╲                 1 ┤         ╱╱    ╲
    │ ╱      ╲╲                   │       ╱╱      ╲
  0 ┤╱────────╲╲──→ t          0 ┤──────╱╱────────╲──→ t
    0  1  2  3  4  5            -5  -4  -3  -2  -1  0
```

### Physical Interpretation

| Context | Meaning |
|---------|---------|
| Audio | Playing a recording backward |
| Radar | Matched filter uses reversed replica |
| Convolution | One signal is reversed and slid |

---

## 2.1.4 Combined Operations: $y(t) = x(at + b)$

### The Canonical Approach

Always factor: $x(at + b) = x\left(a\left(t + \frac{b}{a}\right)\right) = x\left(a\left(t - \left(-\frac{b}{a}\right)\right)\right)$

Identify:
- **Scaling factor**: $a$ (compress if $|a|>1$, expand if $|a|<1$, reverse if $a<0$)
- **Effective shift**: $t_0 = -b/a$ (right if positive, left if negative)

### Method: Apply Operations to Key Points

Given $x(t)$ with key points at $t_1, t_2, \ldots$:

For $y(t) = x(at + b)$, find $t$ such that $at + b = t_k$:

$$t = \frac{t_k - b}{a}$$

### Comprehensive Example

**Problem**: Given $x(t)$ is a triangular pulse on $[-1, 3]$ with peak at $t = 1$. Sketch $y(t) = x(-2t + 4)$.

**Step 1**: Factor: $y(t) = x(-2(t - 2))$

Parameters: $a = -2$ (compression by 2 + reversal), shift right by 2.

**Step 2**: Transform key points using $t = (t_k - b)/a = (t_k - 4)/(-2)$:

| Key point $t_k$ in $x$ | Calculation | New position in $y$ |
|-------------------------|-------------|---------------------|
| $t_k = -1$ (start) | $(-1 - 4)/(-2) = 2.5$ | $t = 2.5$ |
| $t_k = 1$ (peak) | $(1 - 4)/(-2) = 1.5$ | $t = 1.5$ |
| $t_k = 3$ (end) | $(3 - 4)/(-2) = 0.5$ | $t = 0.5$ |

```
x(t):                          y(t) = x(-2t + 4):
    │                              │
  1 ┤      ╱╲                   1 ┤      ╱╲
    │     ╱  ╲                    │     ╱  ╲
    │    ╱    ╲                   │    ╱    ╲
    │   ╱      ╲                  │   ╱      ╲
  0 ┤──╱────────╲──→ t         0 ┤──╱────────╲──→ t
    -1    1    3                 0.5   1.5   2.5
    │                              │
    │  Width = 4                   │  Width = 2 (compressed ÷2)
    │  Peak at 1                   │  Peak at 1.5 (reversed+shifted)
```

---

## 2.1.5 Common Mistakes to Avoid

| Mistake | Correct Approach |
|---------|-----------------|
| Shifting first then scaling while using the un-factored form | Always factor: $x(at+b) = x(a(t - (-b/a)))$ |
| Forgetting reversal when $a < 0$ | Check sign of $a$ first |
| Applying DT scaling without checking integer condition | $x[an]$ valid only when $an \in \mathbb{Z}$ |
| Confusing delay (+right) and advance (+left) | $x(t - t_0)$: **subtract** → delay (right) |

---

## 2.1.6 Timing Diagrams for Signal Transformations

```
TIMING DIAGRAM: Effects of operations on a pulse x(t) = rect(t)

Original x(t):
         ┌───┐
         │   │
    ─────┘   └─────
    -0.5  0  0.5

x(t - 2) [delay by 2]:
              ┌───┐
              │   │
    ──────────┘   └─────
         0   1.5  2  2.5

x(2t) [compress by 2]:
        ┌─┐
        │ │
    ────┘ └────
    -0.25 0 0.25

x(-t) [reverse]:
         ┌───┐
         │   │
    ─────┘   └─────
    -0.5  0  0.5    (same! Because rect is even)

x(-t + 1) = x(-(t-1)) [reverse + shift right by 1]:
              ┌───┐
              │   │
    ──────────┘   └──
           0  0.5  1  1.5
```

---

## 2.1.7 Real-World Applications

| Operation | Application | Example |
|-----------|------------|---------|
| Time delay | Communication | Signal propagation over cable |
| Time advance | Prediction | Look-ahead in video encoding |
| Compression | Fast-forward | Speeding up audio/video |
| Expansion | Slow-motion | Slowing down replay |
| Reversal | Matched filtering | Radar pulse detection |
| Combined | Channel equalization | Compensating for delay + dispersion |

---

## Summary Table

| Operation | CT Formula | Key Points Transform | Effect |
|-----------|-----------|---------------------|--------|
| Right shift by $t_0$ | $x(t-t_0)$ | $t_k \to t_k + t_0$ | Delay |
| Left shift by $t_0$ | $x(t+t_0)$ | $t_k \to t_k - t_0$ | Advance |
| Compress by $a>1$ | $x(at)$ | $t_k \to t_k/a$ | Duration shrinks |
| Expand by $0<a<1$ | $x(at)$ | $t_k \to t_k/a$ | Duration grows |
| Reverse | $x(-t)$ | $t_k \to -t_k$ | Mirror |
| General | $x(at+b)$ | $t_k \to (t_k-b)/a$ | Factor first! |

---

## Quick Revision Questions

1. **If $x(t)$ is defined on $[2, 8]$, what is the support of $y(t) = x(3t - 6)$?**
   - Factor: $x(3(t-2))$. Solve $2 \leq 3(t-2) \leq 8$ → $2.67 \leq t \leq 4.67$.

2. **Given a DT signal $x[n]$ for $n = 0$ to $5$, which samples survive in $y[n] = x[3n]$?**
   - $n = 0$: $x[0]$; $n = 1$: $x[3]$. Only $n = 0$ and $n = 1$ give valid indices $\leq 5$.

3. **What is the width of $x(at)$ if $x(t)$ has width $W$?**
   - Width = $W/|a|$.

4. **For $x(t-t_0)$, does the signal shift left or right when $t_0 > 0$?**
   - Right (delay).

5. **If $x(t)$ has energy $E$, what is the energy of $x(at)$?**
   - $E/|a|$.

---

[← Back to Unit 2](README.md) | [Next: Amplitude Scaling →](02-amplitude-scaling.md)
