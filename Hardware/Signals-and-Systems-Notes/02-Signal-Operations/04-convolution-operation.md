# Chapter 2.4: Convolution Operation

## Overview

Convolution is the **single most important operation** in signals and systems. It determines the output of any Linear Time-Invariant (LTI) system given its input and impulse response. This chapter develops convolution from first principles for both continuous-time and discrete-time signals.

---

## 2.4.1 Motivation: Why Convolution?

For an LTI system with impulse response $h(t)$:

$$\text{Input } x(t) \xrightarrow{h(t)} \text{Output } y(t) = x(t) * h(t)$$

```
    ┌──────────────┐
    │  LTI System  │
    │              │
x(t)───►  h(t)    ├───► y(t) = x(t) * h(t)
    │              │
    └──────────────┘

If input = δ(t), then output = h(t)    [definition of impulse response]
If input = x(t), then output = x(t) * h(t)    [convolution]
```

---

## 2.4.2 Convolution Integral (Continuous-Time)

### Definition

$$y(t) = x(t) * h(t) = \int_{-\infty}^{\infty} x(\tau) \, h(t - \tau) \, d\tau$$

### Step-by-Step Procedure

1. **Replace** $t$ with dummy variable $\tau$ in both signals: $x(\tau)$ and $h(\tau)$
2. **Reverse** one signal: $h(-\tau)$ (flip $h$ about $\tau = 0$)
3. **Shift** the reversed signal by $t$: $h(t - \tau)$ (slide along $\tau$-axis)
4. **Multiply** $x(\tau) \cdot h(t - \tau)$ for each value of $t$
5. **Integrate** the product over all $\tau$ to get $y(t)$

### Visual Process

```
Step 1: Express as functions of τ

x(τ):                    h(τ):
    │                        │
  1 ┤  ┌──┐               1 ┤  ╲
    │  │  │                  │   ╲
  0 ┤──┘  └──→ τ          0 ┤────╲──→ τ
    0  1  2                  0  1  2

Step 2: Reverse h → h(-τ):
    │
  1 ┤        ╱
    │       ╱
  0 ┤──────╱──→ τ
   -2  -1  0

Step 3: Shift → h(t-τ) for various t:

t = -1:          t = 1:           t = 3:
    │                │                │
  1 ┤   ╱         1 ┤       ╱      1 ┤            ╱
    │  ╱             │      ╱         │           ╱
  0 ┤─╱──→ τ      0 ┤─────╱─→ τ   0 ┤──────────╱─→ τ
  -3 -2 -1           -1  0  1         1   2   3

Step 4-5: Multiply and integrate for each t → y(t)
```

---

## 2.4.3 Worked Example: CT Convolution

**Problem**: Compute $y(t) = x(t) * h(t)$ where:
- $x(t) = u(t) - u(t-2)$ (rectangular pulse, 0 to 2)
- $h(t) = u(t) - u(t-3)$ (rectangular pulse, 0 to 3)

```
x(t):                h(t):
    │                    │
  1 ┤  ┌──────┐       1 ┤  ┌──────────┐
    │  │      │          │  │          │
  0 ┤──┘      └──→ t  0 ┤──┘          └──→ t
    0     2              0        3
```

**Analysis by regions of overlap**:

$$y(t) = \int_{-\infty}^{\infty} x(\tau) \cdot h(t - \tau) \, d\tau$$

Since $x(\tau) = 1$ for $0 \leq \tau \leq 2$ and $h(t-\tau) = 1$ for $0 \leq t - \tau \leq 3$ (i.e., $t-3 \leq \tau \leq t$):

| Region | Condition | Overlap | $y(t)$ |
|--------|-----------|---------|--------|
| $t < 0$ | No overlap | — | $0$ |
| $0 \leq t < 2$ | Partial overlap | $[0, t]$ | $t$ |
| $2 \leq t < 3$ | Full overlap | $[0, 2]$ | $2$ |
| $3 \leq t < 5$ | Partial overlap | $[t-3, 2]$ | $5-t$ |
| $t \geq 5$ | No overlap | — | $0$ |

$$y(t) = \begin{cases} 0, & t < 0 \\ t, & 0 \leq t < 2 \\ 2, & 2 \leq t < 3 \\ 5 - t, & 3 \leq t < 5 \\ 0, & t \geq 5 \end{cases}$$

```
y(t) = x(t) * h(t):  [Trapezoidal shape]
    │
  2 ┤      ╱──────╲
    │     ╱        ╲
  1 ┤    ╱          ╲
    │   ╱            ╲
  0 ┤──╱──────────────╲──→ t
    0  1  2  3  4  5

Duration of y = (duration of x) + (duration of h) = 2 + 3 = 5 ✓
```

---

## 2.4.4 Convolution Sum (Discrete-Time)

### Definition

$$y[n] = x[n] * h[n] = \sum_{k=-\infty}^{\infty} x[k] \, h[n - k]$$

### Tabular Method (Most Practical for Hand Calculations)

**Problem**: Compute $y[n] = x[n] * h[n]$ where:
- $x[n] = \{1, 2, 3\}$ for $n = 0, 1, 2$
- $h[n] = \{1, 1, 1, 1\}$ for $n = 0, 1, 2, 3$

**Step 1**: Set up multiplication table

```
              h[0]=1   h[1]=1   h[2]=1   h[3]=1
               n=0      n=1      n=2      n=3      n=4      n=5
x[0]=1 ──►      1        1        1        1
x[1]=2 ──►               2        2        2        2
x[2]=3 ──►                        3        3        3        3
          ────────────────────────────────────────────────────
  SUM:           1        3        6        6        5        3
```

**Result**: $y[n] = \{1, 3, 6, 6, 5, 3\}$ for $n = 0, 1, 2, 3, 4, 5$

**Verification**:
- Length of $y = $ length($x$) + length($h$) $- 1 = 3 + 4 - 1 = 6$ ✓
- Sum of $y = $ sum($x$) × sum($h$) $= 6 \times 4 = 24$; sum of output = $1+3+6+6+5+3 = 24$ ✓

```
y[n]:
  6 ┤        ↑  ↑
  5 ┤              ↑
  4 ┤
  3 ┤     ↑              ↑
  2 ┤
  1 ┤  ↑
  0 ┼──┼──┼──┼──┼──┼──┼──→ n
     0  1  2  3  4  5
```

### Flip-and-Slide Method (Graphical)

For each value of $n$:

1. Flip $h[k]$ to get $h[-k]$
2. Shift to get $h[n-k]$
3. Multiply $x[k] \cdot h[n-k]$ for all $k$
4. Sum all products → $y[n]$

```
n = 0:  h[-k] centered at 0     n = 2:  h[2-k] centered at 2
    x[k]:  1  2  3               x[k]:  1  2  3
    h[-k]: 1                     h[2-k]:       1  1  1  1
           ↑                                      ↑
    Product: 1×1 = 1             Product: 1×1 + 2×1 + 3×1 = 6
    y[0] = 1                     y[2] = 6
```

---

## 2.4.5 Properties of Convolution

| Property | Formula | Significance |
|----------|---------|-------------|
| **Commutative** | $x * h = h * x$ | Either signal can be flipped |
| **Associative** | $(x * h_1) * h_2 = x * (h_1 * h_2)$ | Cascade systems |
| **Distributive** | $x * (h_1 + h_2) = x * h_1 + x * h_2$ | Parallel systems |
| **Identity** | $x * \delta = x$ | Impulse doesn't change signal |
| **Shift** | $x(t) * \delta(t - t_0) = x(t - t_0)$ | Delay by $t_0$ |
| **Width** | Width($y$) = Width($x$) + Width($h$) | Output is always wider |
| **Area** | Area($y$) = Area($x$) × Area($h$) | Areas multiply |

### Proof: Commutative Property

$$x * h = \int_{-\infty}^{\infty} x(\tau)h(t-\tau)d\tau$$

Let $u = t - \tau$, then $\tau = t - u$, $d\tau = -du$:

$$= \int_{\infty}^{-\infty} x(t-u)h(u)(-du) = \int_{-\infty}^{\infty} h(u)x(t-u)du = h * x \quad \checkmark$$

### Identity Property Illustration

```
    x(t) ───►[ δ(t) ]───► x(t)    (signal passes through unchanged)
    
    x(t) ───►[ δ(t - t₀) ]───► x(t - t₀)    (delayed by t₀)
```

---

## 2.4.6 Convolution with Standard Signals

### Convolution with Impulse

$$x(t) * \delta(t) = x(t) \quad \text{(identity)}$$

$$x(t) * \delta(t - t_0) = x(t - t_0) \quad \text{(delay)}$$

### Convolution with Step

$$x(t) * u(t) = \int_{-\infty}^{t} x(\tau) \, d\tau \quad \text{(running integral)}$$

### Convolution of Two Exponentials

$$e^{-at}u(t) * e^{-bt}u(t) = \frac{e^{-bt} - e^{-at}}{a - b} u(t), \quad a \neq b$$

$$e^{-at}u(t) * e^{-at}u(t) = te^{-at}u(t), \quad a = b$$

### Useful Convolution Results

| $x(t)$ | $h(t)$ | $y(t) = x * h$ |
|--------|--------|-----------------|
| $\delta(t)$ | $h(t)$ | $h(t)$ |
| $\delta(t - t_0)$ | $h(t)$ | $h(t - t_0)$ |
| $u(t)$ | $u(t)$ | $r(t) = tu(t)$ |
| $e^{-at}u(t)$ | $u(t)$ | $\frac{1}{a}(1 - e^{-at})u(t)$ |
| rect($t/T_1$) | rect($t/T_2$) | $T_{min} \cdot \text{tri}(t/T_{max})$ scaled |

---

## 2.4.7 Graphical Convolution — Complete Example

**Problem**: $y(t) = e^{-t}u(t) * u(t)$

**Step 1**: Set up the integral

$$y(t) = \int_{-\infty}^{\infty} e^{-\tau}u(\tau) \cdot u(t-\tau) \, d\tau$$

**Step 2**: Determine integration limits

- $u(\tau) = 1$ for $\tau \geq 0$
- $u(t - \tau) = 1$ for $\tau \leq t$
- Combined: $0 \leq \tau \leq t$ (for $t > 0$)

**Step 3**: Evaluate

$$y(t) = \int_{0}^{t} e^{-\tau} \, d\tau = \left[-e^{-\tau}\right]_0^t = 1 - e^{-t}, \quad t \geq 0$$

$$\boxed{y(t) = (1 - e^{-t})u(t)}$$

```
y(t) = (1 - e⁻ᵗ)u(t):
    │
  1 ┤ · · · · · · · · · · · · · · · ─────
    │                        ╭──────
    │                   ╱
    │              ╱
    │         ╱
    │      ╱
    │   ╱          ← RC charging curve!
  0 ┤──╱───────────────────────────→ t
    0  1  2  3  4  5  6  7

    This is the step response of an RC circuit!
```

---

## 2.4.8 Convolution in Block Diagram Notation

### Cascade (Series) Connection

$$y(t) = x(t) * h_1(t) * h_2(t) = x(t) * [h_1(t) * h_2(t)]$$

```
    x(t)──►[h₁(t)]──►[h₂(t)]──►y(t)
    
    Equivalent to:
    
    x(t)──►[h₁(t) * h₂(t)]──►y(t)
    
    h_eq(t) = h₁(t) * h₂(t)
```

### Parallel Connection

$$y(t) = x(t) * h_1(t) + x(t) * h_2(t) = x(t) * [h_1(t) + h_2(t)]$$

```
         ┌──[h₁(t)]──┐
    x(t)─┤            ├─(+)──►y(t)
         └──[h₂(t)]──┘
    
    h_eq(t) = h₁(t) + h₂(t)
```

---

## 2.4.9 Numerical Convolution Algorithm

For computer implementation of DT convolution:

```
Algorithm: Linear Convolution

Input:  x[0..M-1], h[0..N-1]
Output: y[0..M+N-2]

for n = 0 to M+N-2:
    y[n] = 0
    for k = max(0, n-N+1) to min(n, M-1):
        y[n] += x[k] * h[n-k]
```

**Complexity**: $O(MN)$ multiplications

> In practice, large convolutions use the **FFT method**: compute FFT of both signals, multiply in frequency domain, inverse FFT. Complexity: $O(N \log N)$.

---

## 2.4.10 Real-World Applications of Convolution

| Application | What is Convolved | Purpose |
|-------------|-------------------|---------|
| **Audio reverb** | Audio signal * room impulse response | Simulate acoustic environment |
| **Image blurring** | Image * Gaussian kernel | Noise reduction |
| **Edge detection** | Image * Sobel/Laplacian kernel | Feature extraction |
| **FIR filtering** | Signal * filter coefficients | Frequency selection |
| **Communication** | Signal * channel impulse response | Channel modeling |
| **Radar** | Received signal * matched filter | Target detection |

---

## Summary Table

| Concept | CT Formula | DT Formula |
|---------|-----------|-----------|
| Convolution | $\int_{-\infty}^{\infty} x(\tau)h(t-\tau)d\tau$ | $\sum_{k} x[k]h[n-k]$ |
| Commutative | $x * h = h * x$ | Same |
| Associative | $(x*h_1)*h_2 = x*(h_1*h_2)$ | Same |
| Distributive | $x*(h_1+h_2) = x*h_1 + x*h_2$ | Same |
| Identity | $x * \delta = x$ | Same |
| Output length | Width($x$) + Width($h$) | $M + N - 1$ samples |
| Output area | Area($x$) × Area($h$) | Sum($x$) × Sum($h$) |
| Cascade | $h_{eq} = h_1 * h_2$ | Same |
| Parallel | $h_{eq} = h_1 + h_2$ | Same |

---

## Quick Revision Questions

1. **What is the output length when convolving signals of length 5 and length 8?**
   - $5 + 8 - 1 = 12$ samples.

2. **What happens when you convolve any signal with $\delta[n-3]$?**
   - The signal is delayed by 3: $x[n] * \delta[n-3] = x[n-3]$.

3. **Compute $\{1, 2\} * \{1, 1, 1\}$ using the tabular method.**
   - Result: $\{1, 3, 3, 2\}$.

4. **What is $u(t) * u(t)$?**
   - $r(t) = tu(t)$ (ramp function).

5. **How does cascade connection of two LTI systems relate to convolution?**
   - Equivalent impulse response is $h_{eq} = h_1 * h_2$ (convolution of individual responses).

6. **Why is FFT-based convolution preferred for large signals?**
   - Direct convolution is $O(MN)$, while FFT-based is $O(N\log N)$, much faster for large $N$.

---

[← Previous: Signal Addition and Multiplication](03-signal-addition-and-multiplication.md) | [Back to Unit 2](README.md) | [Next Unit: Systems →](../03-Systems/README.md)
