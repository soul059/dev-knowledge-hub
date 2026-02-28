# Chapter 9.2: Type I and Type II Errors

[← Previous: Null and Alternative Hypothesis](01-null-and-alternative-hypothesis.md) | [Back to README](../README.md) | [Next: Z-Test and t-Test →](03-z-test-and-t-test.md)

---

## Overview

Every hypothesis test can make two kinds of mistakes. Understanding these errors — and the tradeoffs between them — is essential for designing good tests and interpreting results correctly.

---

## 1. The Two Types of Errors

| | $H_0$ True (Reality) | $H_0$ False (Reality) |
|:-:|:-:|:-:|
| **Fail to Reject $H_0$** | ✅ Correct Decision | ❌ **Type II Error** ($\beta$) |
| **Reject $H_0$** | ❌ **Type I Error** ($\alpha$) | ✅ Correct Decision (**Power**) |

$$\boxed{\alpha = P(\text{Type I Error}) = P(\text{Reject } H_0 \mid H_0 \text{ true})}$$

$$\boxed{\beta = P(\text{Type II Error}) = P(\text{Fail to reject } H_0 \mid H_0 \text{ false})}$$

$$\boxed{\text{Power} = 1 - \beta = P(\text{Reject } H_0 \mid H_0 \text{ false})}$$

```
  The Error Matrix (Confusion Matrix)
  
                        REALITY
                   H₀ True    H₀ False
                 ┌──────────┬──────────┐
  DECISION       │          │          │
  Fail to   ─────│ Correct  │ Type II  │
  Reject H₀     │  (1-α)   │ Error(β) │
                 ├──────────┼──────────┤
  Reject    ─────│ Type I   │ Correct  │
  H₀             │ Error(α) │ Power    │
                 │          │  (1-β)   │
                 └──────────┴──────────┘
```

---

## 2. Real-World Analogies

### Medical Testing

| | Patient Healthy ($H_0$) | Patient Sick ($H_1$) |
|:-:|:-:|:-:|
| Test Negative | True Negative ✅ | **False Negative** (Type II) |
| Test Positive | **False Positive** (Type I) | True Positive ✅ |

### Criminal Justice

| | Innocent ($H_0$) | Guilty ($H_1$) |
|:-:|:-:|:-:|
| Acquit | Correct ✅ | **Guilty goes free** (Type II) |
| Convict | **Innocent convicted** (Type I) | Correct ✅ |

---

## 3. The Tradeoff

$$\text{Decreasing } \alpha \text{ (fewer false positives)} \Rightarrow \text{Increasing } \beta \text{ (more false negatives)}$$

```
  The α-β Tradeoff (for fixed n)
  
  Distribution under H₀         Distribution under H₁
  
          ╱╲                              ╱╲
         ╱  ╲                            ╱  ╲
        ╱    ╲                          ╱    ╲
       ╱      ╲        │              ╱      ╲
      ╱        ╲       │             ╱        ╲
     ╱  (1-α)   ╲  α   │    β      ╱  (1-β)   ╲
  ──╱────────────╲██████│████████──╱─────────────╲──
                  │     c         │
                  │               │
           Critical value c
  
  α = shaded area under H₀ (right of c)
  β = shaded area under H₁ (left of c)
  
  Moving c right → smaller α, larger β
  Moving c left  → larger α, smaller β
```

### How to Reduce BOTH Errors

**Increase the sample size $n$!** This makes both distributions narrower, reducing overlap.

```
  Small n:                       Large n:
  
      ╱╲         ╱╲                  ╱╲      ╱╲
     ╱  ╲  BIG  ╱  ╲               ╱╱╲╲    ╱╱╲╲
    ╱    ╲OVERLAP╱    ╲            ╱╱  ╲╲  ╱╱  ╲╲
  ─╱──────╲────╱──────╲─       ──╱╱────╲╲╱╱────╲╲──
       H₀        H₁                H₀  LESS  H₁
                                      OVERLAP
```

---

## 4. Power of a Test

$$\boxed{\text{Power} = 1 - \beta = P(\text{Reject } H_0 \mid H_1 \text{ true})}$$

Power depends on:

| Factor | Effect on Power |
|--------|-----------------|
| $\uparrow$ Sample size $n$ | $\uparrow$ Power |
| $\uparrow$ Significance level $\alpha$ | $\uparrow$ Power (but more Type I errors) |
| $\uparrow$ Effect size $|\mu - \mu_0|$ | $\uparrow$ Power (easier to detect) |
| $\downarrow$ Population variability $\sigma$ | $\uparrow$ Power |

### Typical Power Target

$$\text{Power} \geq 0.80 \quad (\beta \leq 0.20)$$

---

## 5. Computing Power (Example)

$H_0: \mu = 100$ vs $H_1: \mu = 105$, with $\sigma = 15$, $n = 36$, $\alpha = 0.05$ (right-tailed).

**Step 1:** Critical value under $H_0$:

$$c = \mu_0 + z_\alpha \cdot \frac{\sigma}{\sqrt{n}} = 100 + 1.645 \cdot \frac{15}{6} = 100 + 4.11 = 104.11$$

**Step 2:** Power = $P(\bar{X} > 104.11 \mid \mu = 105)$

$$Z = \frac{104.11 - 105}{15/6} = \frac{-0.89}{2.5} = -0.356$$

$$\text{Power} = P(Z > -0.356) = 0.639$$

This power is below 0.80 — we need a larger sample!

**Required $n$ for power = 0.80:**

$$n = \left(\frac{\sigma(z_\alpha + z_\beta)}{\mu_1 - \mu_0}\right)^2 = \left(\frac{15(1.645 + 0.842)}{5}\right)^2 = \left(\frac{15 \times 2.487}{5}\right)^2 = (7.46)^2 \approx 56$$

---

## 6. Power Function and OC Curve

The **power function** $\pi(\theta) = P(\text{Reject } H_0 \mid \theta)$ shows power for different parameter values.

```
  Power Function π(θ) for H₀: μ = μ₀ vs H₁: μ > μ₀
  
  Power
  1.0 │                              ●────────
      │                         ●──●
  0.8 │                    ●──●     ← desired power
      │               ●──●
      │          ●──●
  0.5 │     ●──●
      │  ●●
  α   │●                            ← power at μ₀ = α
  0.0 │
      └──────────────────────────────────────── θ
         μ₀              μ₁
  
  The farther θ is from μ₀, the higher the power
  (easier to detect bigger effects)
```

The **Operating Characteristic (OC) Curve** is $1 - \pi(\theta) = \beta(\theta)$.

---

## 7. Effect Size

**Effect size** standardizes the magnitude of the departure from $H_0$:

$$\boxed{d = \frac{|\mu_1 - \mu_0|}{\sigma}}$$

| Cohen's $d$ | Interpretation |
|------------|----------------|
| $d = 0.2$ | Small effect |
| $d = 0.5$ | Medium effect |
| $d = 0.8$ | Large effect |

Larger effect sizes require smaller samples to detect.

---

## 8. Sample Size Determination

For a one-sample $Z$-test:

$$\boxed{n = \left(\frac{z_\alpha + z_\beta}{\delta/\sigma}\right)^2}$$

where $\delta = |\mu_1 - \mu_0|$ and $z_\beta$ is the critical value for the desired power.

| $\alpha$ | Power | $z_\alpha + z_\beta$ |
|---------|-------|---------------------|
| 0.05 | 0.80 | 1.645 + 0.842 = 2.487 |
| 0.05 | 0.90 | 1.645 + 1.282 = 2.927 |
| 0.01 | 0.80 | 2.326 + 0.842 = 3.168 |
| 0.01 | 0.90 | 2.326 + 1.282 = 3.608 |

---

## Summary Table

| Concept | Formula/Definition |
|---------|-------------------|
| Type I Error ($\alpha$) | Reject $H_0$ when $H_0$ is true |
| Type II Error ($\beta$) | Fail to reject $H_0$ when $H_1$ is true |
| Power | $1 - \beta$ = P(correctly reject $H_0$) |
| $\alpha$-$\beta$ tradeoff | Reducing one increases the other (fixed $n$) |
| Increase power by | ↑ $n$, ↑ $\alpha$, ↑ effect size, ↓ $\sigma$ |
| Effect size | $d = \|\mu_1 - \mu_0\|/\sigma$ |
| Sample size | $n = ((z_\alpha + z_\beta)/d)^2$ |

---

## Quick Revision Questions

1. In the medical testing context, which is generally worse: a Type I or Type II error? Discuss for (a) cancer screening, (b) COVID test for boarding a flight.

2. If $\alpha = 0.05$, $\sigma = 10$, $n = 100$, testing $H_0: \mu = 50$ vs $H_1: \mu = 52$: compute the power of the test.

3. Explain why increasing $n$ reduces both $\alpha$ and $\beta$ simultaneously.

4. A study reports power = 0.60. Is this adequate? What does it mean?

5. Calculate the sample size needed for $\alpha = 0.05$, power = 0.90, $\sigma = 20$, detecting a difference of $\delta = 5$.

---

[← Previous: Null and Alternative Hypothesis](01-null-and-alternative-hypothesis.md) | [Back to README](../README.md) | [Next: Z-Test and t-Test →](03-z-test-and-t-test.md)
