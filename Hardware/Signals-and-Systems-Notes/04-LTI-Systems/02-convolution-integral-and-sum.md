# Chapter 4.2: Convolution Integral and Sum

## Overview

Convolution is the mathematical operation that computes the output of any LTI system given its impulse response and input signal. The **convolution integral** (CT) and **convolution sum** (DT) are the most fundamental computational tools in signals and systems.

---

## 4.2.1 The Convolution Integral (CT)

$$y(t) = x(t) * h(t) = \int_{-\infty}^{\infty} x(\tau) h(t - \tau) \, d\tau$$

### Step-by-Step Procedure

1. **Replace** $t$ with $\tau$ in both $x$ and $h$: $x(\tau)$, $h(\tau)$
2. **Flip** $h(\tau)$ to get $h(-\tau)$
3. **Shift** by $t$ to get $h(t-\tau)$
4. **Multiply** $x(\tau) \cdot h(t-\tau)$
5. **Integrate** the product over all $\tau$
6. **Repeat** for every value of $t$

```
Step-by-step graphical convolution:

Step 1: x(τ)           Step 2: h(-τ) [flip]
    │                       │
    │ ╱╲                  ╱╲│
    │╱  ╲               ╱  ╲│
  ──┼────╲──→ τ       ──╲──╱┼──→ τ
    0                    0

Step 3: h(t-τ) [shift by t]
    │
    │     ╱╲
    │    ╱  ╲
  ──┼───╱────╲──→ τ
    0   t

Step 4-5: Multiply & Integrate
    │
    │ ╱╲ ╱╲      Area of overlap
    │╱  ╳  ╲     = y(t)
  ──┼───╱╲──╲──→ τ
```

---

## 4.2.2 Worked Example: CT Convolution

**Find** $y(t) = x(t) * h(t)$ where:

$$x(t) = e^{-t}u(t), \quad h(t) = e^{-2t}u(t)$$

**Setup**:

$$y(t) = \int_{-\infty}^{\infty} e^{-\tau}u(\tau) \cdot e^{-2(t-\tau)}u(t-\tau) \, d\tau$$

**Determine limits** using $u(\tau) \cdot u(t-\tau)$:
- $u(\tau) = 1$ when $\tau \geq 0$
- $u(t-\tau) = 1$ when $\tau \leq t$
- Both nonzero: $0 \leq \tau \leq t$ (for $t \geq 0$)

**Compute** (for $t \geq 0$):

$$y(t) = \int_{0}^{t} e^{-\tau} \cdot e^{-2(t-\tau)} d\tau = e^{-2t} \int_{0}^{t} e^{\tau} d\tau$$

$$= e^{-2t}[e^{\tau}]_0^t = e^{-2t}(e^t - 1) = e^{-t} - e^{-2t}$$

$$\boxed{y(t) = (e^{-t} - e^{-2t})u(t)}$$

```
x(t) = e⁻ᵗu(t):      h(t) = e⁻²ᵗu(t):     y(t) = (e⁻ᵗ-e⁻²ᵗ)u(t):
  1 ┤╲                  1 ┤╲                  │   ╱╲
    │ ╲                    │╲                  │  ╱  ╲
    │  ╲                   │ ╲╲                │ ╱    ╲╲
    │   ╲╲                 │   ╲╲              │╱       ╲╲
  0 ┤─────╲→ t          0 ┤─────╲→ t        0 ┤──────────╲→ t
    0  1  2  3            0  1  2  3          0  1  2  3  4

  Slow decay             Faster decay        Rise then decay
                                              Peak ≈ 0.25 at t ≈ 0.69
```

### Verification

At $t = 0$: $y(0) = 1 - 1 = 0$ ✓ (starts at zero)

As $t \to \infty$: $y(t) \to 0$ ✓ (both terms decay)

---

## 4.2.3 The Convolution Sum (DT)

$$y[n] = x[n] * h[n] = \sum_{k=-\infty}^{\infty} x[k] \cdot h[n-k]$$

### Step-by-Step Procedure

1. **List** $x[k]$ and $h[k]$
2. **Flip** $h[k]$ to get $h[-k]$
3. **Shift** by $n$ to get $h[n-k]$
4. **Multiply** $x[k] \cdot h[n-k]$ element-wise
5. **Sum** all products → $y[n]$
6. **Repeat** for each $n$

---

## 4.2.4 DT Convolution — Tabular Method

The tabular (matrix) method is efficient for finite-length sequences.

**Example**: $x[n] = \{1, 2, 3\}$ (starting at $n=0$) and $h[n] = \{1, 1, 1\}$ (starting at $n=0$)

| | $h[0]=1$ | $h[1]=1$ | $h[2]=1$ |
|---|---|---|---|
| $x[0]=1$ | 1 | 1 | 1 |
| $x[1]=2$ | | 2 | 2 | 2 |
| $x[2]=3$ | | | 3 | 3 | 3 |

Sum along diagonals:

$$y[0] = 1, \quad y[1] = 1+2 = 3, \quad y[2] = 1+2+3 = 6$$
$$y[3] = 2+3 = 5, \quad y[4] = 3$$

$$\boxed{y[n] = \{1, 3, 6, 5, 3\}, \quad n = 0,1,2,3,4}$$

```
Tabular method layout:

           n=0  n=1  n=2  n=3  n=4
x[0]·h:    1    1    1
x[1]·h:         2    2    2
x[2]·h:              3    3    3
          ─────────────────────────
y[n]:      1    3    6    5    3
```

### Output Length Rule

If $x$ has length $N$ and $h$ has length $M$:

$$\text{Length of } y = N + M - 1$$

Here: $3 + 3 - 1 = 5$ ✓

---

## 4.2.5 Graphical Convolution — DT Detailed Example

**Find** $y[n] = x[n] * h[n]$ where $x[n] = u[n]$ and $h[n] = (0.5)^n u[n]$.

$$y[n] = \sum_{k=-\infty}^{\infty} u[k] \cdot (0.5)^{n-k} u[n-k]$$

**Limits**: $u[k] = 1$ for $k \geq 0$; $u[n-k] = 1$ for $k \leq n$.

For $n \geq 0$:

$$y[n] = \sum_{k=0}^{n} (0.5)^{n-k} = (0.5)^n \sum_{k=0}^{n} 2^k = (0.5)^n \cdot \frac{2^{n+1}-1}{2-1}$$

$$= (0.5)^n(2^{n+1} - 1) = 2 - (0.5)^n$$

$$\boxed{y[n] = (2 - 2^{-n})u[n]}$$

```
y[n] values:
n:    0    1    2    3    4    5    ...
y[n]: 1   1.5  1.75 1.875  →  2

  2 ┤─────────────────── (asymptote)
    │        ↑   ↑   ↑
    │     ↑
  1 ┤  ↑
    │
  0 ┼──┴──┴──┴──┴──┴──→ n
     0  1  2  3  4  5
  
  y[n] → 2 as n → ∞ (step response of stable system)
```

---

## 4.2.6 Convolution with Standard Signals

These results are extremely useful and should be memorized:

| Convolution | Result |
|-------------|--------|
| $x(t) * \delta(t)$ | $x(t)$ (identity) |
| $x(t) * \delta(t-t_0)$ | $x(t-t_0)$ (shift) |
| $x[n] * \delta[n]$ | $x[n]$ (identity) |
| $u(t) * u(t)$ | $tu(t)$ (ramp) |
| $e^{-at}u(t) * u(t)$ | $\frac{1}{a}(1 - e^{-at})u(t)$ |
| $e^{-at}u(t) * e^{-bt}u(t)$ | $\frac{e^{-bt} - e^{-at}}{a-b}u(t)$, $a \neq b$ |
| $e^{-at}u(t) * e^{-at}u(t)$ | $te^{-at}u(t)$ |

### The Sifting Property in Action

$$x(t) * \delta(t - t_0) = \int_{-\infty}^{\infty} x(\tau)\delta(t - t_0 - \tau) d\tau = x(t - t_0)$$

> Convolving with a shifted impulse simply **shifts** the signal.

---

## 4.2.7 Graphical Convolution — CT Example

**Find** $y(t) = x(t) * h(t)$ where:

$$x(t) = u(t) - u(t-2) \quad \text{(rect pulse, width 2)}$$
$$h(t) = u(t) - u(t-3) \quad \text{(rect pulse, width 3)}$$

```
x(t):                 h(t):
  1 ┤  ┌───┐           1 ┤  ┌────────┐
    │  │   │              │  │        │
  0 ┤──┘   └──→ t     0 ┤──┘        └──→ t
    0  1   2             0  1  2  3
    Width = 2            Width = 3
```

### Graphical Solution by Regions

Flip $h(\tau)$ → $h(-\tau)$, then slide $h(t-\tau)$ across $x(\tau)$:

**Region 1**: $t < 0$ → No overlap → $y(t) = 0$

**Region 2**: $0 \leq t < 2$ → Partial overlap:
$$y(t) = \int_0^t 1 \cdot 1 \, d\tau = t$$

**Region 3**: $2 \leq t < 3$ → Full overlap of $x$:
$$y(t) = \int_0^2 1 \cdot 1 \, d\tau = 2$$

**Region 4**: $3 \leq t < 5$ → Partial overlap:
$$y(t) = \int_{t-3}^2 1 \cdot 1 \, d\tau = 5-t$$

**Region 5**: $t \geq 5$ → No overlap → $y(t) = 0$

```
Result y(t) — Trapezoidal pulse:

  2 ┤     ┌─────┐
    │    ╱       ╲
  1 ┤   ╱         ╲
    │  ╱           ╲
  0 ┤─╱─────────────╲──→ t
    0  1  2  3  4  5

  y(t) = { t,     0 ≤ t < 2
          { 2,     2 ≤ t < 3
          { 5-t,   3 ≤ t < 5
          { 0,     otherwise

  Output width = 2 + 3 - 0 = 5  ✓
```

---

## 4.2.8 Convolution of Two Rectangles — General Formula

For $x(t) = \text{rect}_{T_1}(t)$ and $h(t) = \text{rect}_{T_2}(t)$ (both starting at $t=0$):

- Output is a **trapezoid** (or triangle if $T_1 = T_2$)
- Output duration: $T_1 + T_2$
- Maximum value: $\min(T_1, T_2)$
- Flat top from $\min(T_1,T_2)$ to $\max(T_1,T_2)$

```
Two equal rectangles → Triangle:

  x * x:     T ┤    ╱╲
               │   ╱  ╲
               │  ╱    ╲
             0 ┤─╱──────╲──→ t
               0    T    2T

Two unequal rectangles → Trapezoid:

  x * h:  T₁ ┤   ┌──────┐      (T₁ < T₂)
              │  ╱        ╲
              │ ╱          ╲
            0 ┤╱────────────╲──→ t
              0 T₁  T₂  T₁+T₂
```

---

## 4.2.9 Numerical Convolution Algorithm

For computer implementation:

```
Algorithm: DT_Convolution(x, h)
──────────────────────────────
Input:  x[0..N-1], h[0..M-1]
Output: y[0..N+M-2]

For n = 0 to N+M-2:
    y[n] = 0
    For k = max(0, n-M+1) to min(n, N-1):
        y[n] += x[k] * h[n-k]
    End
End
Return y
```

**Complexity**: $O(NM)$ multiplications.

**Fast convolution** using FFT: $O((N+M)\log(N+M))$ — much faster for large sequences.

---

## 4.2.10 Important Observations

1. **Commutativity**: $x * h = h * x$ — you can flip either signal
2. **Width rule**: Output duration = sum of input durations (for finite signals)
3. **Area rule**: $\text{Area}(y) = \text{Area}(x) \times \text{Area}(h)$
4. **Smoothing effect**: Convolution generally smooths signals
5. **Polynomial multiplication**: DT convolution of coefficient sequences = polynomial multiplication

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| CT convolution | $y(t) = \int x(\tau)h(t-\tau)d\tau$ |
| DT convolution | $y[n] = \sum x[k]h[n-k]$ |
| Graphical method | Flip, shift, multiply, integrate/sum |
| Tabular method | Arrange products, sum diagonals |
| Output length | $N + M - 1$ (finite-length sequences) |
| $x * \delta$ | $= x$ (identity element) |
| Rect * Rect | Trapezoid (or triangle if equal widths) |
| Complexity | $O(NM)$ direct, $O(N\log N)$ via FFT |

---

## Quick Revision Questions

1. **Compute $\{1, 2\} * \{1, -1\}$.**
   - $y = \{1\cdot1,\; 1(-1)+2(1),\; 2(-1)\} = \{1, 1, -2\}$.

2. **What is $x(t) * \delta(t - 5)$?**
   - $x(t-5)$ — shifts the signal by 5 units.

3. **If $x$ has length 4 and $h$ has length 6, what is the output length?**
   - $4 + 6 - 1 = 9$.

4. **Is convolution commutative?**
   - Yes. $x * h = h * x$.

5. **What does convolution of two rectangular pulses produce?**
   - A trapezoidal (or triangular) pulse.

6. **Verify: $u(t) * u(t) = tu(t)$.**
   - $\int_0^t 1 \cdot 1 \, d\tau = t$ for $t \geq 0$. Combined: $tu(t)$ ✓.

---

[← Previous: Impulse Response](01-impulse-response.md) | [Next: Properties of LTI Systems →](03-properties-of-lti-systems.md)
