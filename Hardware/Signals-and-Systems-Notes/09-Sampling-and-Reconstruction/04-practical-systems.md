# Chapter 9.4: Practical Sampling Systems

## Overview

Real-world sampling involves imperfect components: sample-and-hold circuits with finite acquisition time, analog-to-digital converters with quantization errors, and digital-to-analog converters with limited resolution. This chapter covers ADC/DAC architectures, quantization theory, oversampling, and sigma-delta modulation — bridging the gap between theoretical sampling and practical implementation.

---

## 9.4.1 Complete Sampling System

```
PRACTICAL ANALOG-TO-DIGITAL-TO-ANALOG CHAIN:

         ANALOG FRONT-END                DIGITAL                   ANALOG OUTPUT
  x(t) ──→ [Anti-alias] ──→ [S/H] ──→ [ADC] ──→ [DSP] ──→ [DAC] ──→ [Smooth] ──→ y(t)
             LPF              Sample     Quantize   Process    Convert    LPF
                               & Hold     + Code              to analog
```

---

## 9.4.2 Sample-and-Hold (S/H) Circuit

### Purpose

Holds the analog input constant during ADC conversion time. Without S/H, the input would change during conversion, causing errors.

```
SAMPLE-AND-HOLD OPERATION:

  x(t):              S/H output:
    ╱╲    ╱╲          ┌──┐   ┌──┐
   ╱  ╲  ╱  ╲         │  │   │  │ ┌──┐
  ╱    ╲╱    ╲     ┌──┘  └───┘  │ │  │
                   │            └─┘  └──
  ──────────→ t    ────────────────────→ t
                     Sample  Hold  Sample  Hold
```

### Key Parameters

| Parameter | Description | Typical Value |
|-----------|-------------|---------------|
| Acquisition time | Time to capture input | 100 ns – 1 μs |
| Aperture jitter | Uncertainty in sample instant | 1 – 100 ps |
| Droop rate | Voltage decay during hold | 1 – 100 mV/ms |
| Hold settling | Time for output to settle | 50 – 500 ns |

### Aperture Jitter Effect

Timing uncertainty $\Delta t$ limits maximum signal frequency:

$$\text{SNR}_{jitter} = -20\log_{10}(2\pi f_m \Delta t) \text{ dB}$$

---

## 9.4.3 Quantization

### Definition

Quantization maps the continuous amplitude to a finite set of discrete levels:

$$x_q[n] = Q\{x[n]\} = \Delta \cdot \text{round}\left(\frac{x[n]}{\Delta}\right)$$

where $\Delta = \frac{V_{FS}}{2^B}$ is the quantization step size, $B$ is the number of bits, and $V_{FS}$ is the full-scale voltage.

```
QUANTIZATION (3-bit example, 8 levels):

  x(t) → Quantized levels:
  
  7Δ ─ ─ ─ ─ ─ ─ ─ ─ ─ ━━━━   Level 111
  6Δ ─ ─ ─ ─ ─ ─ ━━━━━       Level 110
  5Δ ─ ─ ─ ─ ━━━━             Level 101
  4Δ ─ ─ ━━━━                  Level 100
  3Δ ━━━━ ╱╲                   Level 011
  2Δ    ╱╱  ╲╲                 Level 010
  1Δ   ╱      ╲               Level 001
  0  ──╱────────╲──→ t        Level 000
  
  Error |e| ≤ Δ/2  (for rounding quantizer)
```

### Quantization Error

$$e[n] = x_q[n] - x[n], \quad -\frac{\Delta}{2} \leq e[n] < \frac{\Delta}{2}$$

For a uniformly distributed input over many quantization levels:

$$\sigma_e^2 = \frac{\Delta^2}{12}$$

### Signal-to-Quantization-Noise Ratio (SQNR)

For a full-scale sinusoidal input with $B$ bits:

$$\boxed{\text{SQNR} = 6.02B + 1.76 \text{ dB}}$$

| Bits ($B$) | SQNR (dB) | Dynamic Range |
|------------|-----------|---------------|
| 8 | 49.9 | Audio (telephone) |
| 12 | 74.0 | Industrial |
| 16 | 98.1 | CD audio |
| 24 | 146.2 | Professional audio |

> Each additional bit adds **6.02 dB** of dynamic range.

---

## 9.4.4 ADC Architectures

| Architecture | Speed | Resolution | Power | Application |
|-------------|-------|------------|-------|-------------|
| **Flash** | Very fast (GHz) | Low (6-8 bits) | High | Video, radar |
| **SAR** | Medium (1-10 MHz) | Medium-High (8-18 bits) | Low | General purpose |
| **Pipeline** | Fast (100+ MHz) | Medium (8-16 bits) | Medium | Communications |
| **Sigma-Delta** | Slow base (MHz) | Very High (16-24 bits) | Low | Audio, sensors |
| **Dual-Slope** | Very slow (Hz) | High (12-20 bits) | Very low | Multimeters |

### Flash ADC

```
FLASH ADC (3-bit):

  Vᵢₙ ──┬──→ [Comp 7] → 7Δ/2?  ──→ ┌────────┐
         ├──→ [Comp 6] → 6Δ/2?      │Priority│ ──→ 3-bit
         ├──→ [Comp 5] → 5Δ/2?      │Encoder │     digital
         ├──→ [Comp 4] → 4Δ/2?  ──→ │        │     output
         ├──→ [Comp 3] → 3Δ/2?      └────────┘
         ├──→ [Comp 2] → 2Δ/2?
         └──→ [Comp 1] → 1Δ/2?
         
  2^B - 1 = 7 comparators needed
  Very fast (single clock cycle) but exponential hardware
```

### Successive Approximation Register (SAR)

```
SAR ADC:

  Vᵢₙ ──→(+)──→ [Comp] ──→ [SAR Logic] ──→ Digital output
          ↑-                     │
          │                      │
          └──── [DAC] ←──────────┘
          
  Binary search: B clock cycles for B bits
  1. Set MSB, compare
  2. Keep or clear MSB based on result
  3. Move to next bit
  4. Repeat until LSB
```

---

## 9.4.5 Oversampling and Noise Shaping

### Oversampling ADC

Sample at $f_s = K \cdot 2f_m$ where $K$ is the oversampling ratio:

- Spreads quantization noise over bandwidth $[0, Kf_m]$
- After digital LPF and decimation, only noise in $[0, f_m]$ remains
- Effective SQNR improvement: $10\log_{10}(K)$ dB

$$\text{SQNR}_{oversampled} = 6.02B + 1.76 + 10\log_{10}(K) \text{ dB}$$

> Doubling $K$ gains **3 dB** (≈ 0.5 bit of resolution)

```
OVERSAMPLING NOISE SPREADING:

Nyquist-rate (K=1):               4× Oversampling (K=4):
  Noise                              Noise
  ┌────────┐                         ┌────────────────────┐
  │████████│ All noise                │█████│              │
  │████████│ in fm band               │█████│  filtered out│
  └────────┘──→ f                     └─────┘──────────────┘→ f
  0      fm                          0   fm              4fm

  Same total noise, but                Noise density = 1/4
  all in signal band                   Filter keeps only fm
                                       → 6 dB less noise!
```

---

## 9.4.6 Sigma-Delta ($\Sigma\Delta$) Modulation

Combines oversampling with **noise shaping** — pushing quantization noise to higher frequencies where it is filtered out.

```
SIGMA-DELTA MODULATOR (1st order):

  x[n] ──→(+)──→ [∫] ──→ [1-bit] ──→ y[n] (1-bit stream)
           ↑-     Σ      Quantizer
           │               │
           └───────────────┘
           
  Noise Transfer Function: NTF(z) = 1 - z⁻¹
  → High-pass shapes noise away from DC/low freq
```

### Noise Shaping Effect

| Modulator Order | Noise Shaping | SQNR per doubling of OSR |
|----------------|---------------|--------------------------|
| 0 (no shaping) | Flat | +3 dB (0.5 bit) |
| 1st order | $(1 - z^{-1})$ | +9 dB (1.5 bits) |
| 2nd order | $(1 - z^{-1})^2$ | +15 dB (2.5 bits) |
| $L$th order | $(1 - z^{-1})^L$ | $+(6L+3)$ dB |

### Sigma-Delta ADC System

```
SIGMA-DELTA ADC:

  x(t) → [Anti-alias] → [ΣΔ Modulator] → [Digital Decimation] → x[n]
           (gentle)       at K·fs            Filter + ↓K           at fs
           
  Advantages:
  - Simple analog circuitry (1-bit quantizer)
  - High resolution from oversampling + noise shaping
  - Relaxed anti-aliasing filter
  - 16-24 bit resolution achievable
```

---

## 9.4.7 DAC Architectures

| Architecture | Speed | Resolution | Use |
|-------------|-------|------------|-----|
| Binary-weighted resistor | Medium | 8-12 bits | Simple DAC |
| R-2R ladder | Medium | 8-16 bits | General purpose |
| Current-steering | Fast | 8-16 bits | Video, communications |
| Sigma-delta DAC | Slow | 16-24 bits | Audio |
| PWM | Slow | 8-16 bits | Motor control, audio |

### R-2R Ladder DAC

```
R-2R LADDER (4-bit):

  Vref ─┤►R─┬─►R─┬─►R─┬─►R─┬───→ Vout
             │     │     │     │
            2R    2R    2R    2R
             │     │     │     │
            b₃    b₂    b₁    b₀
           (MSB)              (LSB)
           
  Vout = Vref × (b₃/2 + b₂/4 + b₁/8 + b₀/16)
```

---

## 9.4.8 Practical Design Considerations

| Consideration | Impact | Solution |
|---------------|--------|----------|
| Clock jitter | Limits SNDR at high frequencies | Low-jitter clock sources |
| Component mismatch | DAC linearity errors | Calibration, dynamic element matching |
| Power supply noise | Injects into signal | Proper decoupling, separate analog/digital supplies |
| Thermal noise | Sets noise floor | Low-noise amplifiers |
| Bandwidth vs resolution | Tradeoff | Choose architecture accordingly |
| Latency | Pipeline/ΣΔ have delay | Budget for processing delay |

### End-to-End Performance Metric

$$\text{ENOB} = \frac{\text{SINAD} - 1.76}{6.02}$$

where **ENOB** = Effective Number of Bits, **SINAD** = Signal-to-Noise-and-Distortion ratio.

---

## Summary Table

| Component | Function | Key Parameter |
|-----------|----------|---------------|
| Anti-aliasing filter | Remove frequencies > $f_s/2$ | Stopband attenuation |
| Sample-and-hold | Freeze input during conversion | Aperture jitter |
| ADC | Continuous → discrete amplitude | Bits, SQNR, speed |
| Quantization | Amplitude discretization | $\Delta = V_{FS}/2^B$ |
| SQNR | $6.02B + 1.76$ dB | Each bit = 6 dB |
| Oversampling | Spread noise over wider BW | +3 dB per doubling |
| Sigma-delta | Noise shaping + oversampling | +9 dB per doubling (1st order) |
| DAC | Discrete → continuous | R-2R, current steering |

---

## Quick Revision Questions

1. **A 12-bit ADC has what theoretical SQNR?**
   - $6.02 \times 12 + 1.76 = 74.0$ dB.

2. **How much does oversampling by 4× improve SQNR?**
   - $10\log_{10}(4) = 6.02$ dB (≈ 1 extra bit of resolution).

3. **What is the purpose of a sample-and-hold circuit?**
   - Holds the analog input constant during ADC conversion to prevent errors from a changing input.

4. **Why do sigma-delta ADCs achieve high resolution with a 1-bit quantizer?**
   - They use high oversampling ratios combined with noise shaping, which pushes quantization noise to high frequencies where it is filtered out.

5. **What is ENOB and why is it important?**
   - Effective Number of Bits — measures actual ADC performance including noise and distortion, not just nominal bit count. ENOB = (SINAD − 1.76) / 6.02.

6. **A signal bandwidth is 20 kHz. A sigma-delta ADC uses 64× oversampling. What is the sampling rate?**
   - $f_s = 64 \times 2 \times 20\text{k} = 2.56$ MHz.

---

[← Previous: Reconstruction](03-reconstruction.md) | [Back to Unit 9 README →](README.md) | [Back to Main README →](../README.md)
