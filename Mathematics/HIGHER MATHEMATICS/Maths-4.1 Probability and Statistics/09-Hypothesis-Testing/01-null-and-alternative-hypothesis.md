# Chapter 9.1: Null and Alternative Hypothesis

[← Previous: Confidence Intervals](../08-Sampling-and-Estimation/05-confidence-intervals.md) | [Back to README](../README.md) | [Next: Type I and Type II Errors →](02-type-I-and-type-II-errors.md)

---

## Overview

**Hypothesis testing** is a formal procedure for deciding between two competing claims about a population parameter using sample data. This chapter introduces the framework and terminology.

---

## 1. What Is a Hypothesis?

A **statistical hypothesis** is a statement (claim) about a population parameter.

| Term | Symbol | Definition |
|------|--------|-----------|
| **Null Hypothesis** | $H_0$ | The "status quo" or default claim. Assumed true until evidence against it. |
| **Alternative Hypothesis** | $H_1$ (or $H_a$) | What we suspect or want to show. Accepted only with strong evidence. |

```
  The Legal Analogy
  
  ┌──────────────────────────────────────────────┐
  │  Courtroom          Statistics                │
  │  ─────────          ──────────                │
  │  Defendant          H₀ (null hypothesis)      │
  │  is "innocent"      is "true"                 │
  │                                               │
  │  Prosecution        Researcher gathers         │
  │  presents evidence  sample data                │
  │                                               │
  │  "Guilty" verdict   Reject H₀                 │
  │  only if evidence   only if evidence           │
  │  beyond reasonable  is statistically           │
  │  doubt              significant                │
  │                                               │
  │  "Not guilty" ≠     "Fail to reject H₀" ≠     │
  │  "Innocent"         "H₀ is true"              │
  └──────────────────────────────────────────────┘
```

---

## 2. Types of Hypotheses

### One-Tailed (One-Sided) Tests

| Test Type | $H_0$ | $H_1$ | Use When |
|-----------|--------|--------|----------|
| Right-tailed | $\mu \leq \mu_0$ | $\mu > \mu_0$ | Suspect parameter is larger |
| Left-tailed | $\mu \geq \mu_0$ | $\mu < \mu_0$ | Suspect parameter is smaller |

### Two-Tailed (Two-Sided) Test

| $H_0$ | $H_1$ | Use When |
|--------|--------|----------|
| $\mu = \mu_0$ | $\mu \neq \mu_0$ | Suspect parameter differs (either direction) |

```
  Right-tailed:        Left-tailed:         Two-tailed:
  
  H₀: μ ≤ μ₀          H₀: μ ≥ μ₀          H₀: μ = μ₀
  H₁: μ > μ₀          H₁: μ < μ₀          H₁: μ ≠ μ₀
  
       ╱╲                   ╱╲                  ╱╲
      ╱  ╲                 ╱  ╲                ╱  ╲
     ╱    ╲               ╱    ╲              ╱    ╲
    ╱      ╲█            █╱      ╲           █╱      ╲█
  ─╱────────╲█──       ──█╲──────╲─        ──█╲──────╲█──
              →          ←                  ←          →
  Reject region      Reject region     Reject regions
  in right tail      in left tail      in both tails
```

---

## 3. Setting Up Hypotheses

### Guidelines

1. $H_0$ always contains the **equality** ($=$, $\leq$, or $\geq$)
2. $H_1$ is what the researcher wants to **prove**
3. The burden of proof is on $H_1$
4. $H_0$ and $H_1$ are **mutually exclusive** and **exhaustive**

### Examples

| Scenario | $H_0$ | $H_1$ |
|----------|--------|--------|
| "New drug is more effective" | $\mu_{\text{drug}} \leq \mu_{\text{placebo}}$ | $\mu_{\text{drug}} > \mu_{\text{placebo}}$ |
| "Average ≠ 100" | $\mu = 100$ | $\mu \neq 100$ |
| "Defect rate < 5%" | $p \geq 0.05$ | $p < 0.05$ |
| "Coin is fair" | $p = 0.5$ | $p \neq 0.5$ |
| "Treatment reduces time" | $\mu \geq \mu_0$ | $\mu < \mu_0$ |

---

## 4. General Procedure

```
  Step-by-Step Hypothesis Testing
  
  ┌─────────────────────────────────┐
  │ 1. State H₀ and H₁             │
  └──────────────┬──────────────────┘
                 ▼
  ┌─────────────────────────────────┐
  │ 2. Choose significance level α  │
  │    (typically 0.05)             │
  └──────────────┬──────────────────┘
                 ▼
  ┌─────────────────────────────────┐
  │ 3. Select test statistic        │
  │    (Z, t, χ², F)               │
  └──────────────┬──────────────────┘
                 ▼
  ┌─────────────────────────────────┐
  │ 4. Determine rejection region   │
  │    OR compute p-value           │
  └──────────────┬──────────────────┘
                 ▼
  ┌─────────────────────────────────┐
  │ 5. Compute test statistic from  │
  │    sample data                  │
  └──────────────┬──────────────────┘
                 ▼
  ┌─────────────────────────────────┐
  │ 6. Make decision:               │
  │    Reject H₀ or Fail to reject │
  └──────────────┬──────────────────┘
                 ▼
  ┌─────────────────────────────────┐
  │ 7. State conclusion in context  │
  └─────────────────────────────────┘
```

---

## 5. Significance Level $\alpha$

$$\alpha = P(\text{Reject } H_0 \mid H_0 \text{ is true})$$

This is the maximum probability of a **Type I error** (false positive).

| $\alpha$ | How conservative? | Common use |
|---------|-------------------|-----------|
| 0.10 | Lenient | Exploratory studies |
| 0.05 | Standard | Most research |
| 0.01 | Strict | High-stakes decisions |
| 0.001 | Very strict | Particle physics, genomics |

$\alpha$ is chosen **before** collecting data!

---

## 6. Test Statistic and Critical Value

### Test Statistic

A value computed from sample data that measures how far the sample result is from what $H_0$ predicts:

$$\text{Test statistic} = \frac{\text{Sample statistic} - \text{Null parameter value}}{\text{Standard error}}$$

### Critical Value

The threshold that separates the rejection region from the non-rejection region.

```
  Two-tailed test at α = 0.05
  
                    Fail to reject H₀
        Reject H₀ ◄─────────────────────► Reject H₀
  
  ─────█╱╲╲╲╲╲╲╲╲╲╲╲╲╲╲╱╲╲╲╲╲╲╲╲╲╲╲█─────
       │                             │
    -z_{α/2}                      z_{α/2}
    = -1.96                       = 1.96
       │         │                   │
    α/2 = 2.5%   │               α/2 = 2.5%
                  0
```

---

## 7. Decision Rules

### Critical Value Approach

- **Right-tailed:** Reject $H_0$ if test statistic $> z_{\alpha}$
- **Left-tailed:** Reject $H_0$ if test statistic $< -z_{\alpha}$
- **Two-tailed:** Reject $H_0$ if $|$test statistic$| > z_{\alpha/2}$

### P-value Approach (next chapter)

Reject $H_0$ if p-value $< \alpha$.

---

## 8. Important Language

| Correct | Incorrect |
|---------|-----------|
| "Reject $H_0$" | "Accept $H_1$" (too strong) |
| "Fail to reject $H_0$" | "Accept $H_0$" (never proven!) |
| "Evidence suggests..." | "$H_1$ is true" |
| "Statistically significant" | "Practically important" |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| $H_0$ | Null hypothesis — status quo, contains $=$ |
| $H_1$ | Alternative — what we want to show |
| $\alpha$ | Significance level — P(Type I error) |
| One-tailed | $H_1$: $\mu > \mu_0$ or $\mu < \mu_0$ |
| Two-tailed | $H_1$: $\mu \neq \mu_0$ |
| Test statistic | Measures departure from $H_0$ |
| Critical value | Threshold for rejection |
| Language | "Fail to reject" ≠ "Accept" |

---

## Quick Revision Questions

1. For each scenario, state $H_0$ and $H_1$: (a) A manufacturer claims batteries last at least 500 hours. (b) A school claims the average GPA is 3.0.

2. What is the difference between a one-tailed and two-tailed test? When would you use each?

3. Explain using the courtroom analogy why we say "fail to reject $H_0$" instead of "accept $H_0$."

4. A researcher sets $\alpha = 0.01$. What does this mean in practical terms?

5. Why must $\alpha$ be chosen before data collection?

---

[← Previous: Confidence Intervals](../08-Sampling-and-Estimation/05-confidence-intervals.md) | [Back to README](../README.md) | [Next: Type I and Type II Errors →](02-type-I-and-type-II-errors.md)
