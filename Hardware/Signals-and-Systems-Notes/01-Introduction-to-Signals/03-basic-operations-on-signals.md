# Chapter 1.3: Basic Operations on Signals

## Overview

Signal operations are mathematical transformations applied to signals. These form the building blocks for signal processing and system analysis. Operations can be categorized into **independent variable (time) transformations** and **dependent variable (amplitude) transformations**.

---

## 1.3.1 Classification of Signal Operations

```
              Signal Operations
                    │
        ┌───────────┴───────────┐
        │                       │
   Time-Domain              Amplitude
   Operations               Operations
        │                       │
   ┌────┼────┐             ┌────┼────┐
   │    │    │             │    │    │
 Shift Scale Reflect     Scale  Add  Mult
```

---

## 1.3.2 Time Shifting

### Definition

Time shifting moves a signal forward or backward in time **without changing its shape**.

**Continuous-Time**: $y(t) = x(t - t_0)$
- $t_0 > 0$: Signal shifts **right** (delay)
- $t_0 < 0$: Signal shifts **left** (advance)

**Discrete-Time**: $y[n] = x[n - n_0]$
- $n_0 > 0$: Signal shifts **right** (delay)
- $n_0 < 0$: Signal shifts **left** (advance)

### ASCII Illustration

```
Original: x(t)              Delayed: x(t - 2)         Advanced: x(t + 2)
    │                           │                           │
  1 ┤  ╭──╮                  1 ┤        ╭──╮             1 ┤╭──╮
    │ ╱    ╲                   │       ╱    ╲              │╲    ╲
  0 ┤╱──────╲────→ t        0 ┤──────╱──────╲──→ t      0 ┤╱─────╲──→ t
    0  1  2  3                 0  1  2  3  4  5           -2 -1  0  1
    │                           │                           │
    │  Peak at t=1              │  Peak at t=3              │  Peak at t=-1
    │                           │  (shifted right by 2)     │  (shifted left by 2)
```

### Key Rule

> **"Replace $t$ with $(t - t_0)$"**: Every point on the signal moves to $t + t_0$.

### Example: DT Time Shift

Given $x[n] = \{1, \underline{2}, 3, 1\}$ (underline = $n = 0$):

```
x[n]:                    x[n-2] (delay by 2):
  3 ┤        ↑            3 ┤              ↑
  2 ┤     ↑               2 ┤           ↑
  1 ┤  ↑        ↑         1 ┤        ↑        ↑
  0 ┼──┼──┼──┼──┼──→      0 ┼──┼──┼──┼──┼──┼──┼──→
    -1  0  1  2  n            -1  0  1  2  3  4  n
                                      ↑
                                   origin shifted
```

---

## 1.3.3 Time Scaling

### Definition

Time scaling compresses or expands a signal along the time axis.

$$y(t) = x(at)$$

- $|a| > 1$: **Compression** (signal squeezed toward origin)
- $|a| < 1$: **Expansion** (signal stretched away from origin)
- $a < 0$: Also involves **time reversal**

### ASCII Illustration

```
Original: x(t)            Compressed: x(2t)         Expanded: x(t/2)
    │                           │                        │
  1 ┤  ╭────╮               1 ┤  ╭─╮                 1 ┤    ╭────────╮
    │ ╱      ╲                │ ╱   ╲                  │   ╱          ╲
  0 ┤╱────────╲──→ t       0 ┤╱─────╲───→ t        0 ┤──╱────────────╲──→ t
    0  1  2  3  4             0 0.5  1 1.5 2            0  2   4   6   8
    │                           │                        │
    │  Width = 4               │  Width = 2              │  Width = 8
    │                           │  (÷ 2)                 │  (× 2)
```

### Important Rules for Time Scaling

1. **Key time points get divided by $a$**: If $x(t)$ has a feature at $t = t_1$, then $x(at)$ has that feature at $t = t_1/a$
2. **For DT signals**: $x[an]$ is only defined when $an$ is an integer → some samples may be lost
3. **Area under the curve**: $\int x(at) dt = \frac{1}{|a|} \int x(\tau) d\tau$

### Example: Step-by-Step

Find $y(t) = x(2t - 3)$ given $x(t)$.

**Method**: Rewrite as $y(t) = x(2(t - 3/2))$

1. First shift: $x(t - 3/2)$ → shift right by 3/2
2. Then compress: $x(2(t - 3/2))$ → compress by factor 2 about $t = 3/2$

**OR** (alternative order):
1. First compress: $x(2t)$ → compress by factor 2
2. Then shift: $x(2(t - 3/2)) = x(2t - 3)$ → shift right by 3/2

> **Rule**: Always write in the form $x(a(t - t_0))$ to identify scaling factor $a$ and shift $t_0$.

---

## 1.3.4 Time Reversal (Reflection)

### Definition

Time reversal flips a signal about the vertical axis ($t = 0$ or $n = 0$).

$$y(t) = x(-t) \quad \text{or} \quad y[n] = x[-n]$$

### ASCII Illustration

```
x(t):                        x(-t):
    │                           │
  3 ┤     ╭╮                 3 ┤         ╭╮
  2 ┤    ╱  ╲               2 ┤        ╱  ╲
  1 ┤   ╱    ╲              1 ┤       ╱    ╲
  0 ┤──╱──────╲────→ t     0 ┤──────╱──────╲──→ t
   -1  0  1  2  3           -3  -2  -1  0  1
    │                           │
    │  Signal on [0,3]          │  Mirrored to [-3,0]
```

### DT Example

```
x[n] = {1, 2, 3, 4}  for n = 0, 1, 2, 3

x[n]:                     x[-n]:
  4 ┤              ↑       4 ┤  ↑
  3 ┤        ↑             3 ┤     ↑
  2 ┤     ↑                2 ┤        ↑
  1 ┤  ↑                   1 ┤           ↑
  0 ┼──┼──┼──┼──┼──→ n    0 ┼──┼──┼──┼──┼──→ n
     0  1  2  3            -3 -2 -1  0  1
```

---

## 1.3.5 Amplitude Scaling

### Definition

Multiply the signal amplitude by a constant $A$:

$$y(t) = A \cdot x(t)$$

- $A > 1$: **Amplification**
- $0 < A < 1$: **Attenuation**
- $A < 0$: **Inversion** (flip about time axis) + scaling

### ASCII Illustration

```
x(t):                2·x(t):              -x(t):
    │                    │                    │
  1 ┤  ╭──╮           2 ┤  ╭──╮            0 ┤──────────────→ t
    │ ╱    ╲             │ ╱    ╲             │ ╲    ╱
  0 ┤╱──────╲→ t      0 ┤╱──────╲→ t     -1 ┤  ╰──╯
    │                    │                    │
  Amplitude = 1       Amplitude = 2       Amplitude = -1
  (Original)          (Amplified)         (Inverted)
```

---

## 1.3.6 Signal Addition

### Definition

The sum of two signals is obtained by adding their values at each time instant:

$$y(t) = x_1(t) + x_2(t)$$

### Graphical Method

```
x₁(t):            x₂(t):            y(t) = x₁(t) + x₂(t):
    │                  │                     │
  2 ┤  ╭──╮         1 ┤──────╮            3 ┤  ╭╮
    │ ╱    ╲           │      ╲           2 ┤ ╱  ╲╮
  1 ┤╱      ╲        0 ┤──────╲───→ t    1 ┤╱     ╲
  0 ┤────────╲─→ t     0  1  2  3        0 ┤───────╲──→ t
    0  1  2  3  4                           0  1  2  3  4

At each t: y(t) = x₁(t) + x₂(t)
Example:   y(1) = x₁(1) + x₂(1) = 2 + 1 = 3
```

---

## 1.3.7 Signal Multiplication

### Definition

The product of two signals is obtained by multiplying their values at each time instant:

$$y(t) = x_1(t) \cdot x_2(t)$$

### Important Application: Modulation

Multiplying a message signal by a carrier signal is the basis of **amplitude modulation (AM)**:

```
Message: m(t)             Carrier: c(t) = cos(ωt)
    │                         │
  1 ┤  ╭──╮               1 ┤╱╲╱╲╱╲╱╲╱╲╱╲╱╲╱╲
    │ ╱    ╲                 │
  0 ┤╱──────╲─→ t         0 ┤─────────────────→ t
    │                        │
                          -1 ┤

AM Signal: y(t) = m(t)·cos(ωt)
    │
  1 ┤ ╱╲  ╱╲
    │╱  ╲╱╲ ╲╱╲
  0 ┤──────────╲╱╲──→ t
    │           ╲╱╲
 -1 ┤
    │  (Carrier modulated by message envelope)
```

---

## 1.3.8 Combined Operations — Order Matters!

When multiple operations are applied, the **order of operations** matters. Given $y(t) = x(at + b)$:

### Method 1: Shift First, Then Scale

1. Compute $x(t + b)$: Shift $x(t)$ by $-b$ (left if $b > 0$)
2. Compute $x(at + b)$: Replace $t$ by $at$ → scale about $t = 0$

### Method 2: Scale First, Then Shift (Recommended)

1. Write as $y(t) = x(a(t + b/a))$ or equivalently $x(a(t - t_0))$ where $t_0 = -b/a$
2. Compute $x(at)$: Scale about origin
3. Compute $x(a(t - t_0))$: Shift by $t_0$

### Worked Example

**Find** $y(t) = x(2t + 4)$ where $x(t) = t$ for $0 \leq t \leq 2$.

**Step 1**: Rewrite as $y(t) = x(2(t + 2))$

**Step 2**: Scale → $x(2t)$ exists for $0 \leq 2t \leq 2$, i.e., $0 \leq t \leq 1$

**Step 3**: Shift left by 2 → $x(2(t+2))$ exists for $-2 \leq t \leq -1$

```
x(t):               x(2t):              x(2t+4) = x(2(t+2)):
    │                    │                      │
  2 ┤        ╱         2 ┤    ╱               2 ┤    ╱
    │       ╱             │   ╱                  │   ╱
  1 ┤      ╱            1 ┤  ╱                 1 ┤  ╱
    │     ╱               │ ╱                    │ ╱
  0 ┤────╱────→ t      0 ┤╱──────→ t          0 ┤╱──────→ t
    0  1  2  3            0  1  2              -2  -1  0
```

---

## 1.3.9 Precedence Rule

For $y(t) = x(at - b)$:

| Operation | Effect on time points |
|-----------|----------------------|
| Time scaling by $a$ | Divide all time points by $a$ |
| Time shift by $b/a$ | Add $b/a$ to all time points |
| Time reversal ($a < 0$) | Flip about $t = 0$ |

> **Precedure**: Always factorize: $x(at - b) = x(a(t - b/a))$
> - $a$ = scaling factor
> - $b/a$ = shift amount (right if positive)

---

## 1.3.10 Real-World Applications

| Operation | Application |
|-----------|------------|
| Time shifting | Echo/delay in audio, radar pulse timing |
| Time scaling | Slow-motion / fast-forward video, frequency transposition |
| Time reversal | Matched filtering in radar, backward playback |
| Amplitude scaling | Volume control, gain adjustment |
| Addition | Mixing audio tracks, superposition in circuits |
| Multiplication | Modulation in communication, windowing in DSP |

---

## Summary Table

| Operation | CT Formula | DT Formula | Effect |
|-----------|-----------|-----------|--------|
| Time delay | $x(t - t_0)$ | $x[n - n_0]$ | Shift right by $t_0$ |
| Time advance | $x(t + t_0)$ | $x[n + n_0]$ | Shift left by $t_0$ |
| Time compression | $x(at)$, $|a|>1$ | $x[an]$ (if integer) | Squeeze toward origin |
| Time expansion | $x(at)$, $|a|<1$ | N/A directly | Stretch from origin |
| Time reversal | $x(-t)$ | $x[-n]$ | Mirror about origin |
| Amplitude scaling | $Ax(t)$ | $Ax[n]$ | Scale vertically |
| Addition | $x_1(t)+x_2(t)$ | $x_1[n]+x_2[n]$ | Add point by point |
| Multiplication | $x_1(t) \cdot x_2(t)$ | $x_1[n] \cdot x_2[n]$ | Multiply point by point |

---

## Quick Revision Questions

1. **If $x(t)$ has non-zero values for $1 \leq t \leq 4$, what is the support of $x(2t - 6)$?**
   - Rewrite as $x(2(t-3))$. Support: $1 \leq 2(t-3) \leq 4$ → $3.5 \leq t \leq 5$.

2. **Given $x[n] = \{2, \underline{1}, 3, 4\}$, find $x[2-n]$.**
   - First find $x[-n] = \{4, 3, \underline{1}, 2\}$, then $x[-(n-2)] = x[2-n]$ → shift right by 2: $\{4, 3, \underline{1}, 2\}$ at $n = -1, 0, 1, 2$ becomes at $n = 1, 2, 3, 4$.

3. **Does the order of time shifting and time scaling matter when computing $x(at - b)$?**
   - Yes, the order matters. Use factored form $x(a(t - b/a))$: scale by $a$, then shift by $b/a$.

4. **What is the physical meaning of $x(-t)$?**
   - Time reversal: playing the signal backward.

5. **What happens to the energy of a signal when it is amplitude-scaled by $A$?**
   - Energy scales by $|A|^2$: $E_y = |A|^2 E_x$.

6. **In amplitude modulation, what operation is used to combine the message and carrier?**
   - Multiplication: $y(t) = m(t) \cdot \cos(\omega_c t)$.

---

[← Previous: Signal Classification](02-signal-classification.md) | [Next: Standard Signals →](04-standard-signals.md)
