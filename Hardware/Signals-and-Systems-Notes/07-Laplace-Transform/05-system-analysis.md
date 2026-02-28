# Chapter 7.5: System Analysis Using Laplace Transform

## Overview

The Laplace Transform provides a unified framework for analyzing LTI systems: solving differential equations with initial conditions, determining system stability, computing steady-state and transient responses, and designing controllers. This chapter brings together all the Laplace techniques for comprehensive system analysis.

---

## 7.5.1 Solving ODEs with Initial Conditions (Complete Response)

### General Procedure

```
COMPLETE ODE SOLUTION USING LAPLACE:

1. Take unilateral LT of both sides
   (include initial conditions in derivative terms)

2. Solve algebraically for Y(s)

3. Decompose Y(s):
   Y(s) = Y_zi(s) + Y_zs(s)
   ├── Y_zi: zero-input (due to ICs only)
   └── Y_zs: zero-state (due to input only)

4. Partial fraction expansion

5. Inverse LT → y(t)
```

### Example: First-Order System

$$\frac{dy}{dt} + 3y(t) = 2e^{-t}u(t), \quad y(0^-) = 5$$

**Step 1**: LT both sides

$$sY(s) - 5 + 3Y(s) = \frac{2}{s+1}$$

**Step 2**: Solve

$$Y(s)(s+3) = 5 + \frac{2}{s+1}$$

$$Y(s) = \frac{5}{s+3} + \frac{2}{(s+1)(s+3)}$$

- $Y_{zi}(s) = \frac{5}{s+3}$ (zero-input: from $y(0^-)=5$)
- $Y_{zs}(s) = \frac{2}{(s+1)(s+3)}$ (zero-state: from input)

**Step 3**: PFE of zero-state part

$$\frac{2}{(s+1)(s+3)} = \frac{1}{s+1} - \frac{1}{s+3}$$

**Step 4**: Total

$$Y(s) = \frac{5}{s+3} + \frac{1}{s+1} - \frac{1}{s+3} = \frac{1}{s+1} + \frac{4}{s+3}$$

$$y(t) = (e^{-t} + 4e^{-3t})u(t)$$

---

## 7.5.2 Natural and Forced Response

### Alternative Decomposition

$$Y(s) = \underbrace{Y_{natural}(s)}_{\text{from system poles}} + \underbrace{Y_{forced}(s)}_{\text{from input poles}}$$

| Component | Source | Behavior |
|-----------|--------|----------|
| **Zero-input** ($Y_{zi}$) | Initial conditions only | Depends on system poles |
| **Zero-state** ($Y_{zs}$) | Input only ($x = 0$ initially) | Both system and input poles |
| **Natural** | System poles (in any component) | Decays if stable |
| **Forced** | Input poles | Persists as long as input |
| **Transient** | LHP poles | Dies out over time |
| **Steady-state** | $j\omega$-axis poles of input | Persists forever |

---

## 7.5.3 Second-Order Example

$$\frac{d^2y}{dt^2} + 4\frac{dy}{dt} + 3y = 6u(t), \quad y(0^-) = 1, \; y'(0^-) = -1$$

**LT**:

$$s^2Y - s + 1 + 4(sY - 1) + 3Y = \frac{6}{s}$$

$$Y(s^2 + 4s + 3) = s - 1 + 4 + \frac{6}{s} = s + 3 + \frac{6}{s}$$

$$Y(s) = \frac{s^2 + 3s + 6}{s(s+1)(s+3)}$$

**PFE**:

$$Y(s) = \frac{A}{s} + \frac{B}{s+1} + \frac{C}{s+3}$$

$A = \frac{0+0+6}{(1)(3)} = 2$

$B = \frac{1-3+6}{(-1)(2)} = -2$

$C = \frac{9-9+6}{(-3)(-2)} = 1$

$$y(t) = (2 - 2e^{-t} + e^{-3t})u(t)$$

**Verification**: $y(0) = 2 - 2 + 1 = 1$ ✓, $y(\infty) = 2$ (matches final value theorem: $\lim_{s\to 0} sY(s) = 6/3 = 2$ ✓)

---

## 7.5.4 Stability Analysis

### Methods for Assessing Stability

| Method | Approach |
|--------|----------|
| **Pole location** | All poles in open LHP → stable |
| **Routh-Hurwitz** | Algebraic test without finding poles |
| **Final value theorem** | If applicable, check bounded response |

### Routh-Hurwitz Criterion

For characteristic polynomial $a_n s^n + a_{n-1}s^{n-1} + \cdots + a_1 s + a_0$:

**Necessary condition**: All coefficients $a_i > 0$ (same sign).

**Routh Array Construction**:

$$\begin{array}{c|cc}
s^n & a_n & a_{n-2} & a_{n-4} & \cdots \\
s^{n-1} & a_{n-1} & a_{n-3} & a_{n-5} & \cdots \\
s^{n-2} & b_1 & b_2 & \cdots \\
s^{n-3} & c_1 & c_2 & \cdots \\
\vdots & & & \\
s^0 & & & \\
\end{array}$$

where $b_1 = \frac{a_{n-1}a_{n-2} - a_n a_{n-3}}{a_{n-1}}$

**Stability**: No sign changes in the first column → all poles in LHP.

### Example

$$D(s) = s^3 + 6s^2 + 11s + 6$$

```
Routh Array:
  s³ │  1    11
  s² │  6     6
  s¹ │ (6·11 - 1·6)/6 = 10
  s⁰ │  6

First column: 1, 6, 10, 6 → all positive → NO sign changes → STABLE
(Roots: s = -1, -2, -3 — all in LHP ✓)
```

### Unstable Example

$$D(s) = s^3 + 2s^2 + s + 8$$

```
Routh Array:
  s³ │  1    1
  s² │  2    8
  s¹ │ (2·1 - 1·8)/2 = -3
  s⁰ │  8

First column: 1, 2, -3, 8 → TWO sign changes → TWO RHP poles → UNSTABLE
```

---

## 7.5.5 Impulse Response and System Response

### Finding System Response

Given $H(s)$ and input $X(s)$:

$$Y(s) = H(s) \cdot X(s)$$

### Common Input-Response Pairs

| Input | $X(s)$ | Output $Y(s)$ |
|-------|--------|----------------|
| Impulse $\delta(t)$ | $1$ | $H(s)$ |
| Step $u(t)$ | $1/s$ | $H(s)/s$ |
| Ramp $tu(t)$ | $1/s^2$ | $H(s)/s^2$ |
| Sinusoid $\sin\omega t$ | $\omega/(s^2+\omega^2)$ | $H(s)\omega/(s^2+\omega^2)$ |

### Example: Step Response

$$H(s) = \frac{10}{s+5}, \quad X(s) = \frac{1}{s}$$

$$Y(s) = \frac{10}{s(s+5)} = \frac{2}{s} - \frac{2}{s+5}$$

$$y(t) = 2(1 - e^{-5t})u(t)$$

```
Step response:
  y(t) ↑
  2.0 ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
       │         ╱───────────────
       │       ╱
       │     ╱
       │   ╱    Time constant τ = 1/5 = 0.2s
       │ ╱
  ─────┼╱─────────────────→ t
       0     0.2   0.4
```

---

## 7.5.6 Sinusoidal Steady-State Response

For a stable LTI system with sinusoidal input $x(t) = A\cos(\omega_0 t + \phi)$:

$$y_{ss}(t) = A|H(j\omega_0)|\cos(\omega_0 t + \phi + \angle H(j\omega_0))$$

The transient response (from system poles) dies out; only the forced response remains.

### Example

$$H(s) = \frac{10}{s+5}, \quad x(t) = 3\cos(4t)$$

$$H(j4) = \frac{10}{j4+5} = \frac{10}{5+j4}$$

$$|H(j4)| = \frac{10}{\sqrt{25+16}} = \frac{10}{\sqrt{41}} \approx 1.562$$

$$\angle H(j4) = -\arctan(4/5) \approx -38.66°$$

$$y_{ss}(t) = 3 \times 1.562 \cos(4t - 38.66°) \approx 4.69\cos(4t - 38.66°)$$

---

## 7.5.7 System Interconnections in $s$-Domain

### Mason's Gain Formula (for complex signal flow graphs)

$$H(s) = \frac{\sum_k P_k \Delta_k}{\Delta}$$

where:
- $P_k$ = forward path gain of $k$-th path
- $\Delta$ = $1 - \sum L_i + \sum L_iL_j - \cdots$ (loop gains)
- $\Delta_k$ = cofactor for $k$-th path

### Example: Feedback with Multiple Loops

```
        ┌────────────────────────────────┐
        │              G₃               │
  R(s) ──(+)──→[G₁]──(+)──→[G₂]──┬──→ Y(s)
          ↑-           ↑-         │
          │            │          │
          └────[H₂]────┘          │
          │                       │
          └────────[H₁]──────────┘
          
Forward paths: P₁ = G₁·G₂, P₂ = G₃
Inner loop: L₁ = -G₂·H₂
Outer loop: L₂ = -G₁·G₂·H₁
```

---

## 7.5.8 Application — Circuit Analysis

### Example: RC Circuit with Step Input

```
       R = 1kΩ
  ╶──┤►►►├──┬──╴
  +          │
  10u(t)   ┌─┴─┐ C = 1μF
           │   │
           └─┬─┘
  ╶──────────┘──╴
  -         Vₒᵤₜ
```

$$H(s) = \frac{1/RC}{s + 1/RC} = \frac{1000}{s + 1000}$$

$$V_{out}(s) = 10 \cdot \frac{1}{s} \cdot \frac{1000}{s+1000} = \frac{10000}{s(s+1000)}$$

$$= \frac{10}{s} - \frac{10}{s+1000}$$

$$v_{out}(t) = 10(1 - e^{-1000t})u(t) \text{ V}$$

Time constant: $\tau = RC = 10^3 \times 10^{-6} = 1\text{ ms}$

---

## 7.5.9 Unilateral vs Bilateral — When to Use

| Scenario | Use |
|----------|-----|
| Circuit with initial conditions | Unilateral LT |
| Causal system, causal input | Unilateral LT |
| Two-sided or non-causal signals | Bilateral LT |
| Stability analysis | Either (bilateral gives full picture) |
| Control system design | Unilateral LT (causal assumed) |

---

## Summary Table

| Analysis Task | Laplace Method |
|---------------|----------------|
| Solve ODE with ICs | Unilateral LT; include $x(0^-)$, $x'(0^-)$ |
| Find complete response | $Y = Y_{zi} + Y_{zs}$ |
| Check stability | Pole locations or Routh array |
| Step response | $Y(s) = H(s)/s$; inverse LT |
| Sinusoidal SS | Evaluate $H(j\omega_0)$; magnitude and phase |
| Circuit analysis | Replace $L \to Ls$, $C \to 1/Cs$, $R \to R$ |
| Feedback system | $H = G/(1+GF)$ |

---

## Quick Revision Questions

1. **Solve: $y' + 2y = 3u(t)$, $y(0^-) = 1$.**
   - $Y(s)(s+2) = 1 + 3/s$. $Y(s) = \frac{1}{s+2} + \frac{3}{s(s+2)} = \frac{1}{s+2} + \frac{3/2}{s} - \frac{3/2}{s+2} = \frac{3/2}{s} - \frac{1/2}{s+2}$. So $y(t) = (\frac{3}{2} - \frac{1}{2}e^{-2t})u(t)$.

2. **Is $D(s) = s^4 + 2s^3 + 3s^2 + 4s + 5$ stable? (Quick check.)**
   - All coefficients positive — necessary condition met. Need Routh array for sufficient condition.

3. **What is the steady-state output for input $5\sin(10t)$ to $H(s) = \frac{20}{s+20}$?**
   - $|H(j10)| = 20/\sqrt{500} = 0.894$, $\angle = -\arctan(0.5) = -26.57°$. $y_{ss} = 4.47\sin(10t - 26.57°)$.

4. **What is the zero-input response?**
   - The component of output due only to initial conditions, with input set to zero.

5. **How do you represent a capacitor in the $s$-domain?**
   - Impedance $Z_C(s) = 1/(Cs)$. Initial voltage appears as voltage source $v_C(0^-)/s$ in series.

---

[← Previous: Transfer Function](04-transfer-function.md) | [Next: Unit 8 — Z-Transform →](../08-Z-Transform/README.md)
