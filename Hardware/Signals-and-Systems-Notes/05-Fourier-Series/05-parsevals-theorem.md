# Chapter 5.5: Parseval's Theorem and Power Spectrum

## Overview

Parseval's theorem establishes the equivalence between **time-domain power** and **frequency-domain power**. It states that the total average power of a periodic signal equals the sum of powers in each harmonic component — enabling power analysis entirely in the frequency domain.

---

## 5.5.1 Parseval's Theorem — Statement

### Exponential Form

$$P = \frac{1}{T} \int_T |x(t)|^2 \, dt = \sum_{n=-\infty}^{\infty} |c_n|^2$$

### Trigonometric Form

$$P = a_0^2 + \frac{1}{2}\sum_{n=1}^{\infty} (a_n^2 + b_n^2) = A_0^2 + \frac{1}{2}\sum_{n=1}^{\infty} A_n^2$$

> **Total average power** = sum of power in DC + power in each harmonic.

```
PARSEVAL'S THEOREM:

Time domain:                  Frequency domain:
                              
  │  ╱╲   ╱╲   ╱╲            |cₙ|²
  │ ╱  ╲ ╱  ╲ ╱  ╲             │  ↑
  ┼╱────╳────╳────╲→ t         │  │  ↑
  │                             │  │  │  ↑
  Average of |x(t)|²           ┼──┴──┴──┴──→ n
                              
  ∫|x|²/T     =               Σ|cₙ|²
  
  SAME POWER — computed two different ways
```

---

## 5.5.2 Power Contribution by Harmonic

The power in the $n$-th harmonic is:

$$P_n = |c_n|^2 + |c_{-n}|^2 = 2|c_n|^2 \quad (n > 0)$$

$$P_0 = |c_0|^2 = a_0^2 \quad \text{(DC power)}$$

### Power Distribution Table (Example: Square Wave)

$c_n = \frac{2}{jn\pi}$ for odd $n$, amplitude $A = 1$:

| Harmonic | $\|c_n\|$ | $P_n = 2\|c_n\|^2$ | Cumulative $\%$ |
|----------|-----------|---------------------|------------------|
| $n=0$ (DC) | 0 | 0 | 0% |
| $n=\pm 1$ | $2/\pi$ | $8/\pi^2 \approx 0.811$ | 81.1% |
| $n=\pm 3$ | $2/3\pi$ | $8/9\pi^2 \approx 0.090$ | 90.1% |
| $n=\pm 5$ | $2/5\pi$ | $8/25\pi^2 \approx 0.032$ | 93.4% |
| $n=\pm 7$ | $2/7\pi$ | $8/49\pi^2 \approx 0.017$ | 95.0% |
| $\vdots$ | $\vdots$ | $\vdots$ | $\to 100\%$ |

Total: $P = 1$ (since $|x(t)| = 1$ everywhere → $P = 1$).

**Verification**: $\sum_{n \text{ odd}} \frac{8}{n^2\pi^2} = \frac{8}{\pi^2}\cdot\frac{\pi^2}{8} = 1$ ✓

```
Power contribution by harmonic:

Power
  │  ████
  │  ████
  │  ████
  │  ████ ███
  │  ████ ███ ██
  │  ████ ███ ██ █
  ┼──┴────┴───┴──┴──→ harmonic
     1    3   5  7
     
  81% of power is in the FUNDAMENTAL alone!
```

---

## 5.5.3 Power Spectral Density (PSD)

The **power spectrum** (or power spectral density for periodic signals) is:

$$S_n = |c_n|^2$$

Plotted as a **line spectrum** at frequencies $n\omega_0$:

```
Power spectrum of square wave:

  |cₙ|²
  0.41 ┤  ↑
       │  │
       │  │
  0.05 ┤  │     ↑
  0.02 ┤  │     │  ↑
       │  │     │  │  ↑
  ─────┼──┴─────┴──┴──┴──→ nω₀
       0  ω₀  3ω₀ 5ω₀ 7ω₀
       
  Decreases as 1/n² → 90%+ power in first few harmonics
```

---

## 5.5.4 Bandwidth

The **bandwidth** of a periodic signal is the range of frequencies containing significant power.

### Definitions

| Definition | Description |
|------------|-------------|
| **Essential BW** | Range containing 90% (or 95%, 99%) of total power |
| **3 dB BW** | Range where $\|c_n\|^2 \geq \frac{1}{2}\|c_n\|^2_{\max}$ |
| **Null-to-null BW** | Width of the main lobe (first nulls) |

For the pulse train $c_n = Ad\,\text{sinc}(nd)$:

- Main lobe width (null-to-null): $BW = 2/\tau = 2/(dT)$ in terms of harmonic index
- In Hz: $BW = 1/\tau$

```
Bandwidth concepts:

|cₙ|
  ↑   ╱╲
  │  ╱  ╲
  │ ╱    ╲     ╱╲     ╱╲
  │╱      ╲   ╱  ╲   ╱  ╲
  ┼────────╲─╱────╲─╱────╲─→ f
  0        1/τ   2/τ   3/τ
  
  ←─ Main lobe ─→
  ← Null-to-null BW →
```

> **Key insight**: Narrower pulse → wider bandwidth (time-bandwidth product is approximately constant).

---

## 5.5.5 Parseval's Theorem for DT (DTFS)

$$P = \frac{1}{N} \sum_{n=0}^{N-1} |x[n]|^2 = \sum_{k=0}^{N-1} |c_k|^2$$

**Example**: $x[n] = \{1, 0, 1, 0\}$ with $N = 4$:

Time domain: $P = \frac{1}{4}(1+0+1+0) = \frac{1}{2}$

Frequency domain: $c_0 = 1/2$, $c_1 = 0$, $c_2 = 1/2$, $c_3 = 0$

$\sum|c_k|^2 = (1/2)^2 + 0 + (1/2)^2 + 0 = 1/2$ ✓

---

## 5.5.6 Applications of Parseval's Theorem

### Application 1: Computing Difficult Sums

Since $P_{\text{time}} = P_{\text{freq}}$, known power in time domain can evaluate infinite sums.

**Example**: For square wave with $|x(t)| = 1$:

$$1 = \sum_{n \text{ odd}} \frac{8}{n^2\pi^2} = \frac{8}{\pi^2}\left(1 + \frac{1}{9} + \frac{1}{25} + \cdots\right)$$

$$\therefore \sum_{n=0}^{\infty} \frac{1}{(2n+1)^2} = \frac{\pi^2}{8}$$

### Application 2: SNR and Filtering

If a filter passes only harmonics $|n| \leq N$:

$$\text{Retained power} = \sum_{|n| \leq N} |c_n|^2$$

$$\text{Fraction of power retained} = \frac{\sum_{|n| \leq N}|c_n|^2}{\sum_{-\infty}^{\infty}|c_n|^2}$$

### Application 3: RMS Value

$$x_{\text{rms}} = \sqrt{P} = \sqrt{\sum |c_n|^2}$$

For a signal $x(t) = 3 + 4\cos(\omega_0 t) + 2\sin(2\omega_0 t)$:

$$P = 3^2 + \frac{4^2}{2} + \frac{2^2}{2} = 9 + 8 + 2 = 19$$

$$x_{\text{rms}} = \sqrt{19} \approx 4.36$$

---

## 5.5.7 Generalized Parseval (Cross-Power)

$$\frac{1}{T}\int_T x(t)y^*(t)dt = \sum_{n=-\infty}^{\infty} c_n d_n^*$$

If $x = y$, this reduces to the standard Parseval's theorem.

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Parseval's theorem | $\frac{1}{T}\int\|x\|^2 dt = \sum\|c_n\|^2$ |
| Power in $n$-th harmonic | $P_n = 2\|c_n\|^2$ ($n > 0$) |
| DC power | $P_0 = \|c_0\|^2 = a_0^2$ |
| RMS value | $x_{\text{rms}} = \sqrt{\sum\|c_n\|^2}$ |
| Bandwidth | Narrower pulse → wider BW |
| Square wave power | 81% in fundamental, 90% in first two harmonics |
| Sum evaluation | Time-domain power can evaluate frequency sums |

---

## Quick Revision Questions

1. **State Parseval's theorem for FS.**
   - $P = \frac{1}{T}\int_T |x(t)|^2 dt = \sum_{n=-\infty}^{\infty} |c_n|^2$.

2. **What fraction of a square wave's power is in the fundamental?**
   - About 81% ($8/\pi^2$).

3. **Find the RMS of $x(t) = 5\cos(\omega_0 t)$.**
   - $P = A^2/2 = 25/2$, so $x_{\text{rms}} = 5/\sqrt{2} \approx 3.54$.

4. **If a pulse narrows, what happens to its bandwidth?**
   - Bandwidth increases (inversely proportional to pulse width).

5. **Use Parseval's to find $\sum_{n=1,3,5,...} 1/n^2$.**
   - From the square wave: $1 = \frac{8}{\pi^2}\sum_{n \text{ odd}} 1/n^2$, so $\sum = \pi^2/8$.

---

[← Previous: Properties of Fourier Series](04-properties-of-fourier-series.md) | [Next: Unit 6 — Fourier Transform →](../06-Fourier-Transform/README.md)
