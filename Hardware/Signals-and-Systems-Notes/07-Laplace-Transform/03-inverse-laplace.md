# Chapter 7.3: Inverse Laplace Transform

## Overview

The Inverse Laplace Transform recovers $x(t)$ from $X(s)$. While the formal definition uses a contour integral, practical computation relies on **partial fraction expansion (PFE)** combined with known transform pairs. The ROC determines whether each term corresponds to a right-sided or left-sided signal.

---

## 7.3.1 Formal Definition

$$x(t) = \frac{1}{2\pi j}\int_{\sigma - j\infty}^{\sigma + j\infty} X(s)\,e^{st}\,ds$$

where $\sigma$ lies within the ROC. In practice, we rarely evaluate this integral directly.

---

## 7.3.2 Partial Fraction Expansion Method

For rational $X(s) = N(s)/D(s)$ where $\deg(N) < \deg(D)$:

### Step-by-Step Procedure

```
INVERSE LAPLACE TRANSFORM PROCEDURE:

1. Check: Is deg(N) ≥ deg(D)?
   ├─ YES → Do polynomial long division first
   └─ NO  → Proceed to step 2

2. Factor D(s) into roots (poles)

3. Set up partial fractions based on pole type:
   ├─ Simple real poles → A/(s+a)
   ├─ Repeated poles → A/(s+a) + B/(s+a)² + ...
   └─ Complex conjugate → (As+B)/(s²+bs+c)

4. Solve for coefficients

5. Use ROC to determine each term's signal type:
   ├─ ROC right of pole → right-sided: e^(-at)u(t)
   └─ ROC left of pole → left-sided: -e^(-at)u(-t)

6. Inverse transform each term using table
```

---

## 7.3.3 Case 1: Distinct Real Poles

$$X(s) = \frac{N(s)}{(s+p_1)(s+p_2)\cdots(s+p_n)}$$

$$= \frac{A_1}{s+p_1} + \frac{A_2}{s+p_2} + \cdots + \frac{A_n}{s+p_n}$$

**Coefficient formula** (cover-up method):

$$A_k = (s+p_k)X(s)\Big|_{s = -p_k}$$

### Example

$$X(s) = \frac{2s + 4}{(s+1)(s+3)}, \quad \text{ROC: } \sigma > -1$$

$$A_1 = (s+1)\frac{2s+4}{(s+1)(s+3)}\bigg|_{s=-1} = \frac{-2+4}{-1+3} = \frac{2}{2} = 1$$

$$A_2 = (s+3)\frac{2s+4}{(s+1)(s+3)}\bigg|_{s=-3} = \frac{-6+4}{-3+1} = \frac{-2}{-2} = 1$$

$$X(s) = \frac{1}{s+1} + \frac{1}{s+3}$$

ROC $\sigma > -1$ → both poles to the left of ROC → both terms right-sided:

$$x(t) = (e^{-t} + e^{-3t})u(t)$$

---

## 7.3.4 Case 2: Repeated Poles

$$\frac{N(s)}{(s+p)^r} = \frac{A_1}{s+p} + \frac{A_2}{(s+p)^2} + \cdots + \frac{A_r}{(s+p)^r}$$

**Coefficients**:

$$A_r = (s+p)^r X(s)\Big|_{s=-p}$$

$$A_{r-1} = \frac{d}{ds}\left[(s+p)^r X(s)\right]\bigg|_{s=-p}$$

$$A_{r-k} = \frac{1}{k!}\frac{d^k}{ds^k}\left[(s+p)^r X(s)\right]\bigg|_{s=-p}$$

### Example

$$X(s) = \frac{s+5}{(s+1)^2(s+3)}, \quad \text{ROC: } \sigma > -1$$

Partial fractions:

$$X(s) = \frac{A_1}{s+1} + \frac{A_2}{(s+1)^2} + \frac{B}{s+3}$$

$$A_2 = (s+1)^2 \cdot X(s)\big|_{s=-1} = \frac{-1+5}{-1+3} = \frac{4}{2} = 2$$

$$A_1 = \frac{d}{ds}\left[\frac{s+5}{s+3}\right]_{s=-1} = \frac{(s+3) - (s+5)}{(s+3)^2}\bigg|_{s=-1} = \frac{-2}{4} = -\frac{1}{2}$$

$$B = (s+3)X(s)\big|_{s=-3} = \frac{-3+5}{(-3+1)^2} = \frac{2}{4} = \frac{1}{2}$$

$$x(t) = \left(-\frac{1}{2}e^{-t} + 2te^{-t} + \frac{1}{2}e^{-3t}\right)u(t)$$

Using: $\frac{1}{(s+a)^2} \leftrightarrow te^{-at}u(t)$

---

## 7.3.5 Case 3: Complex Conjugate Poles

Poles $s = -\alpha \pm j\beta$:

$$\frac{As + B}{(s+\alpha)^2 + \beta^2} = A\frac{s+\alpha}{(s+\alpha)^2+\beta^2} + \frac{B-A\alpha}{\beta}\cdot\frac{\beta}{(s+\alpha)^2+\beta^2}$$

$$\Rightarrow e^{-\alpha t}\left[A\cos\beta t + \frac{B-A\alpha}{\beta}\sin\beta t\right]u(t)$$

### Example

$$X(s) = \frac{2s+10}{s^2 + 4s + 13}, \quad \text{ROC: } \sigma > -2$$

Complete the square: $s^2 + 4s + 13 = (s+2)^2 + 9 = (s+2)^2 + 3^2$

$$X(s) = \frac{2(s+2) + 6}{(s+2)^2 + 9} = 2\cdot\frac{s+2}{(s+2)^2+9} + 2\cdot\frac{3}{(s+2)^2+9}$$

$$x(t) = e^{-2t}(2\cos 3t + 2\sin 3t)\,u(t)$$

```
Complex poles → damped sinusoid:

x(t):
  ╱╲  
 ╱  ╲   ╱╲
╱    ╲ ╱  ╲    ╱╲
───────╳────╲──╱──╲───→ t
       ╲╱    ╲╱    ╲╱
       
Envelope: e^(-αt)
Frequency: β rad/s
Poles at s = -α ± jβ
```

---

## 7.3.6 Case 4: Improper Fractions ($\deg N \geq \deg D$)

Perform **polynomial long division** first.

### Example

$$X(s) = \frac{s^3 + 2s^2 + 6}{s^2 + 3s + 2}$$

Long division: $X(s) = (s - 1) + \frac{s + 8}{(s+1)(s+2)}$

Then do partial fractions on the remainder:

$$\frac{s+8}{(s+1)(s+2)} = \frac{7}{s+1} - \frac{6}{s+2}$$

$$x(t) = \delta'(t) - \delta(t) + (7e^{-t} - 6e^{-2t})u(t)$$

where $s \leftrightarrow \delta'(t)$ and $1 \leftrightarrow \delta(t)$.

---

## 7.3.7 ROC and Signal Identification

```
Given X(s) = 1/[(s+1)(s-2)], poles at s = -1 and s = 2

Three possible ROCs → three different signals:

CASE 1: σ > 2 (right-sided)      
  jω ↑                            
  ───┤──┤──║══════→ σ              
     -1  2                         
  x(t) = [Ae^(-t) + Be^(2t)]u(t) 

CASE 2: -1 < σ < 2 (two-sided)   
  jω ↑                            
  ───┤══╬══┤───→ σ                 
     -1    2                       
  x(t) = Ae^(-t)u(t) - Be^(2t)u(-t)

CASE 3: σ < -1 (left-sided)      
  jω ↑                            
  ══════║──┤──┤───→ σ              
        -1  2                      
  x(t) = -[Ae^(-t) + Be^(2t)]u(-t)
```

### Worked Example

$$X(s) = \frac{3}{(s+1)(s-2)}, \quad \text{ROC: } -1 < \sigma < 2$$

$$X(s) = \frac{-1}{s+1} + \frac{1}{s-2}$$

- Pole at $s = -1$: ROC is to the **right** → $-e^{-t}u(t)$
- Pole at $s = 2$: ROC is to the **left** → $-e^{2t}u(-t) \cdot 1 = e^{2t}u(-t)$

Wait — let's be precise:

- $\frac{-1}{s+1}$, ROC right of $-1$ → $-e^{-t}u(t)$
- $\frac{1}{s-2}$, ROC left of $2$ → $-e^{2t}u(-t)$

$$x(t) = -e^{-t}u(t) - e^{2t}u(-t)$$

---

## 7.3.8 Using MATLAB/Computational Verification

The inverse Laplace can be verified using the residue function:

```
Given X(s) = (2s+4)/[(s+1)(s+3)]

Numerator:   N = [2, 4]
Denominator: D = [1, 4, 3]

Residues:    r = [1, 1]
Poles:       p = [-1, -3]

x(t) = 1·e^(-t)·u(t) + 1·e^(-3t)·u(t)
```

---

## 7.3.9 Summary of Inverse Transform Forms

| $X(s)$ term | $x(t)$ (causal, ROC right) | $x(t)$ (anti-causal, ROC left) |
|-------------|---------------------------|--------------------------------|
| $\frac{A}{s+a}$ | $Ae^{-at}u(t)$ | $-Ae^{-at}u(-t)$ |
| $\frac{A}{(s+a)^2}$ | $Ate^{-at}u(t)$ | $-(-t)Ae^{-at}u(-t)$ |
| $\frac{A}{(s+a)^n}$ | $\frac{At^{n-1}}{(n-1)!}e^{-at}u(t)$ | $-\frac{A(-t)^{n-1}}{(n-1)!}e^{-at}u(-t)$ |
| Complex pair | Damped sinusoid $\times u(t)$ | Growing sinusoid $\times u(-t)$ |

---

## Summary Table

| Method | When to Use | Key Formula |
|--------|-------------|-------------|
| Cover-up | Distinct real poles | $A_k = (s+p_k)X(s)\|_{s=-p_k}$ |
| Derivative method | Repeated poles | $A_{r-k} = \frac{1}{k!}\frac{d^k}{ds^k}[(s+p)^rX(s)]$ |
| Complete the square | Complex poles | $(s+\alpha)^2 + \beta^2$ form |
| Long division | $\deg N \geq \deg D$ | Divide first, then PFE |
| ROC rule | Signal type | Right of pole → right-sided |

---

## Quick Revision Questions

1. **Find $x(t)$ for $X(s) = \frac{5}{(s+2)(s+5)}$, ROC: $\sigma > -2$.**
   - PFE: $\frac{5/3}{s+2} - \frac{5/3}{s+5}$. Both right-sided: $x(t) = \frac{5}{3}(e^{-2t} - e^{-5t})u(t)$.

2. **Find $x(t)$ for $X(s) = \frac{3}{(s+1)^2}$, ROC: $\sigma > -1$.**
   - $x(t) = 3te^{-t}u(t)$.

3. **How do you handle the case when numerator degree equals denominator degree?**
   - Perform polynomial long division to get a constant plus a proper fraction, then apply PFE.

4. **For $X(s) = \frac{1}{s+2}$ with ROC $\sigma < -2$, what is $x(t)$?**
   - Left-sided: $x(t) = -e^{-2t}u(-t)$.

5. **What is the inverse LT of $\frac{s+3}{(s+3)^2 + 16}$?**
   - $e^{-3t}\cos(4t)\,u(t)$.

---

[← Previous: Properties](02-properties.md) | [Next: Transfer Function →](04-transfer-function.md)
