# Unit 6 — Derivatives of Standard Functions: Exponential & Logarithmic Derivatives

[← Previous: Inverse Trigonometric Derivatives](03-inverse-trig-derivatives.md) · [Next: Hyperbolic Function Derivatives →](05-hyperbolic-derivatives.md)

---

## Chapter Overview

The exponential function $e^x$ is unique: **it is its own derivative**. This chapter covers derivatives of exponential, general exponential ($a^x$), natural logarithmic ($\ln x$), and general logarithmic ($\log_a x$) functions with full proofs and applications.

---

## 6.4.1 The Exponential Function

### Derivative of $e^x$

$$\frac{d}{dx}(e^x) = e^x$$

**Proof (from definition of $e$):**

$$\frac{d}{dx}(e^x) = \lim_{h \to 0}\frac{e^{x+h} - e^x}{h} = e^x \cdot \lim_{h \to 0}\frac{e^h - 1}{h} = e^x \cdot 1 = e^x$$

since $\lim_{h \to 0}\frac{e^h - 1}{h} = 1$ is a defining property of $e$.

```
   y
   |            *
   |           *     y = eˣ 
   |          *      slope at any point = y-value at that point
   |        *
   |      *
   |    *
   | *
  1+*
   |*
   +---+---+---→ x
  -2  -1   0   1
```

### Derivative of $e^{u}$ (chain rule)

$$\frac{d}{dx}(e^{u}) = e^{u} \cdot u'$$

**Examples:**

| $y$ | $\frac{dy}{dx}$ |
|-----|------------------|
| $e^{3x}$ | $3e^{3x}$ |
| $e^{x^2}$ | $2xe^{x^2}$ |
| $e^{\sin x}$ | $\cos x \cdot e^{\sin x}$ |
| $e^{-x^2/2}$ | $-xe^{-x^2/2}$ |
| $x^2 e^x$ | $e^x(x^2 + 2x)$ |

---

## 6.4.2 General Exponential: $a^x$

$$\frac{d}{dx}(a^x) = a^x \ln a \quad (a > 0, a \neq 1)$$

**Proof:** Write $a^x = e^{x \ln a}$:

$$\frac{d}{dx}(e^{x \ln a}) = e^{x \ln a} \cdot \ln a = a^x \ln a \quad \blacksquare$$

**Chain rule form:**

$$\frac{d}{dx}(a^{u}) = a^{u} \cdot \ln a \cdot u'$$

**Examples:**

| $y$ | $\frac{dy}{dx}$ |
|-----|------------------|
| $2^x$ | $2^x \ln 2$ |
| $10^x$ | $10^x \ln 10$ |
| $3^{x^2}$ | $3^{x^2} \cdot \ln 3 \cdot 2x$ |

---

## 6.4.3 The Natural Logarithm

### Derivative of $\ln x$

$$\frac{d}{dx}(\ln x) = \frac{1}{x} \quad (x > 0)$$

**Proof:** Let $y = \ln x$, so $e^y = x$.

Differentiate implicitly: $e^y \cdot \frac{dy}{dx} = 1$

$$\frac{dy}{dx} = \frac{1}{e^y} = \frac{1}{x} \quad \blacksquare$$

### Derivative of $\ln|x|$

$$\frac{d}{dx}(\ln|x|) = \frac{1}{x} \quad (x \neq 0)$$

This extends the domain to include negative $x$.

### Chain rule form

$$\frac{d}{dx}[\ln u] = \frac{u'}{u}$$

**Examples:**

| $y$ | $\frac{dy}{dx}$ |
|-----|------------------|
| $\ln(3x)$ | $\frac{1}{x}$ |
| $\ln(x^2 + 1)$ | $\frac{2x}{x^2 + 1}$ |
| $\ln(\sin x)$ | $\frac{\cos x}{\sin x} = \cot x$ |
| $\ln(\cos x)$ | $\frac{-\sin x}{\cos x} = -\tan x$ |
| $\ln(\sec x + \tan x)$ | $\sec x$ |
| $[\ln x]^2$ | $\frac{2\ln x}{x}$ |
| $\ln(\ln x)$ | $\frac{1}{x \ln x}$ |

---

## 6.4.4 General Logarithm: $\log_a x$

$$\frac{d}{dx}(\log_a x) = \frac{1}{x \ln a}$$

**Proof:** Use the change of base formula: $\log_a x = \frac{\ln x}{\ln a}$

$$\frac{d}{dx}\left(\frac{\ln x}{\ln a}\right) = \frac{1}{\ln a} \cdot \frac{1}{x} = \frac{1}{x \ln a} \quad \blacksquare$$

**Special case:** $\frac{d}{dx}(\log_{10} x) = \frac{1}{x \ln 10} \approx \frac{0.4343}{x}$

---

## 6.4.5 Complete Reference Table

| $f(x)$ | $f'(x)$ | Domain |
|---------|----------|--------|
| $e^x$ | $e^x$ | $\mathbb{R}$ |
| $e^{kx}$ | $ke^{kx}$ | $\mathbb{R}$ |
| $a^x$ | $a^x \ln a$ | $\mathbb{R}$, $a > 0$ |
| $\ln x$ | $\frac{1}{x}$ | $x > 0$ |
| $\ln\|x\|$ | $\frac{1}{x}$ | $x \neq 0$ |
| $\log_a x$ | $\frac{1}{x \ln a}$ | $x > 0$ |

---

## 6.4.6 Worked Examples

### Example 1: $y = xe^x$

$$\frac{dy}{dx} = e^x + xe^x = e^x(1 + x)$$

### Example 2: $y = \frac{e^x - e^{-x}}{2}$ (this is $\sinh x$)

$$\frac{dy}{dx} = \frac{e^x + e^{-x}}{2}$$

### Example 3: $y = e^x \sin x$

$$\frac{dy}{dx} = e^x \sin x + e^x \cos x = e^x(\sin x + \cos x)$$

### Example 4: $y = \ln\sqrt{x^2 + 1}$

Simplify first: $y = \frac{1}{2}\ln(x^2+1)$

$$\frac{dy}{dx} = \frac{1}{2} \cdot \frac{2x}{x^2 + 1} = \frac{x}{x^2 + 1}$$

### Example 5: $y = x^x$ (revisited via exponentials)

Write $y = e^{x\ln x}$:

$$\frac{dy}{dx} = e^{x \ln x} \cdot (1 \cdot \ln x + x \cdot \frac{1}{x}) = x^x(1 + \ln x)$$

This matches the logarithmic differentiation result from Unit 5.

---

## 6.4.7 Important Results to Remember

$$\frac{d}{dx}[\ln(\sin x)] = \cot x$$

$$\frac{d}{dx}[\ln(\cos x)] = -\tan x$$

$$\frac{d}{dx}[\ln(\sec x + \tan x)] = \sec x$$

$$\frac{d}{dx}[\ln(\csc x - \cot x)] = \csc x$$

These appear frequently in integration and are worth memorizing.

---

## Summary Table

| Concept | Key Fact |
|---------|----------|
| $e^x$ is its own derivative | $\frac{d}{dx}(e^x) = e^x$ |
| General exponential | $\frac{d}{dx}(a^x) = a^x \ln a$ |
| Natural log | $\frac{d}{dx}(\ln x) = 1/x$ |
| General log | $\frac{d}{dx}(\log_a x) = 1/(x \ln a)$ |
| Chain rule for $\ln$ | $\frac{d}{dx}[\ln u] = u'/u$ (logarithmic derivative) |
| $e^x$ via limits | $e = \lim_{n\to\infty}(1 + 1/n)^n$ |

---

## Quick Revision Questions

**Q1.** Differentiate $y = e^{3x^2 - x + 1}$.

**Q2.** Find $\frac{dy}{dx}$ for $y = \ln\left(\frac{x+1}{x-1}\right)$.

**Q3.** Differentiate $y = 5^{\sin x}$.

**Q4.** Show that if $y = \frac{e^x - e^{-x}}{e^x + e^{-x}}$, then $\frac{dy}{dx} = 1 - y^2$.

**Q5.** Find the second derivative of $y = x^2 \ln x$.

**Q6.** Differentiate $y = \log_{10}(\cos x)$.

---

[← Previous: Inverse Trigonometric Derivatives](03-inverse-trig-derivatives.md) · [Next: Hyperbolic Function Derivatives →](05-hyperbolic-derivatives.md)
