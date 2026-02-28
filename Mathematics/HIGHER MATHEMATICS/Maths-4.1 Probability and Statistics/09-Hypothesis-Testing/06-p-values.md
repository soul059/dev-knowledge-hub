# Chapter 9.6: P-Values

[← Previous: ANOVA](05-anova.md) | [Back to README](../README.md) | [Next: Linear Regression →](../10-Regression-and-Correlation/01-linear-regression.md)

---

## Overview

The **p-value** is the most commonly reported measure in hypothesis testing. It quantifies the strength of evidence against $H_0$ by answering: "If $H_0$ were true, how surprising is our observed data?"

---

## 1. Definition

$$\boxed{p\text{-value} = P(\text{Test statistic as extreme or more extreme than observed} \mid H_0 \text{ true})}$$

```
  "As extreme or more extreme" depends on H₁:
  
  Right-tailed (H₁: μ > μ₀):
  p = P(Z ≥ z_obs)
  
         ╱╲
        ╱  ╲
       ╱    ╲
      ╱      ╲████████  ← p-value = this area
  ───╱────────╲████████──
               z_obs
  
  Left-tailed (H₁: μ < μ₀):
  p = P(Z ≤ z_obs)
  
            ╱╲
           ╱  ╲
          ╱    ╲
  ████████╱      ╲
  ████████╲───────╲──
        z_obs
  
  Two-tailed (H₁: μ ≠ μ₀):
  p = 2 · P(Z ≥ |z_obs|)
  
         ╱╲
        ╱  ╲
  █████╱    ╲█████
  █████╲    ╱█████  ← p = both shaded areas
  ──────╲──╱──────
   -|z|    |z|
```

---

## 2. Decision Rule

$$\boxed{\text{Reject } H_0 \iff p\text{-value} < \alpha}$$

| p-value vs $\alpha$ | Decision |
|---------------------|----------|
| $p < \alpha$ | Reject $H_0$ (statistically significant) |
| $p \geq \alpha$ | Fail to reject $H_0$ (not significant) |

---

## 3. Interpreting P-Values

### Strength of Evidence

| P-value Range | Evidence Against $H_0$ |
|---------------|----------------------|
| $p > 0.10$ | Little or no evidence |
| $0.05 < p \leq 0.10$ | Weak (marginal) |
| $0.01 < p \leq 0.05$ | Moderate |
| $0.001 < p \leq 0.01$ | Strong |
| $p \leq 0.001$ | Very strong |

### What a P-Value IS and IS NOT

| ✅ P-value IS | ❌ P-value IS NOT |
|--------------|-------------------|
| P(data this extreme \| $H_0$ true) | P($H_0$ is true) |
| A measure of evidence strength | P(result due to chance) |
| Calculated assuming $H_0$ | The probability of making an error |
| Dependent on sample size | A measure of practical importance |

---

## 4. Computing P-Values

### For Z-Tests

| $H_1$ | p-value | Example: $z_{obs} = 2.3$ |
|--------|---------|--------------------------|
| $\mu > \mu_0$ | $P(Z > z_{obs})$ | $P(Z > 2.3) = 0.0107$ |
| $\mu < \mu_0$ | $P(Z < z_{obs})$ | $P(Z < 2.3) = 0.9893$ |
| $\mu \neq \mu_0$ | $2P(Z > |z_{obs}|)$ | $2(0.0107) = 0.0214$ |

### For t-Tests

Same logic, but use the $t$-distribution with appropriate df.

### For $\chi^2$-Tests

$$p\text{-value} = P(\chi^2 > \chi^2_{obs})$$

(Always right-tailed for GOF and independence tests.)

### For F-Tests (ANOVA)

$$p\text{-value} = P(F > F_{obs})$$

---

## 5. Worked Examples

### Example 1: Z-Test

$H_0: \mu = 100$ vs $H_1: \mu \neq 100$. $\bar{x} = 104$, $\sigma = 15$, $n = 36$.

$$Z = \frac{104 - 100}{15/6} = 1.60$$

$$p\text{-value} = 2P(Z > 1.60) = 2(0.0548) = 0.1096$$

At $\alpha = 0.05$: $p = 0.1096 > 0.05$ → **Fail to reject $H_0$**.

### Example 2: One-Sided t-Test

$H_0: \mu \leq 50$ vs $H_1: \mu > 50$. $\bar{x} = 53$, $s = 8$, $n = 25$.

$$T = \frac{53 - 50}{8/5} = 1.875$$

From $t(24)$ table: $P(T > 1.875)$. Since $t_{0.05, 24} = 1.711$ and $t_{0.025, 24} = 2.064$:

$$0.025 < p\text{-value} < 0.05$$

At $\alpha = 0.05$: **Reject $H_0$**.

---

## 6. P-Value vs Critical Value Approach

| Aspect | P-Value | Critical Value |
|--------|---------|----------------|
| What you compute | P(data \| $H_0$) | Fixed threshold |
| Comparison | p vs $\alpha$ | Test stat vs critical value |
| Flexibility | Works for any $\alpha$ | Tied to chosen $\alpha$ |
| Information | Shows HOW significant | Only yes/no |
| Preferred | Modern practice | Traditional approach |

They always give the **same decision**:

$$p < \alpha \iff |\text{test stat}| > \text{critical value}$$

---

## 7. Common Pitfalls

### Statistical vs Practical Significance

```
  Large sample (n = 10,000):
  
  H₀: μ = 100
  Observed: x̄ = 100.5
  
  Z = (100.5 - 100)/(σ/√10000) can be very large
  → Tiny p-value (p < 0.001)
  → "Highly significant!"
  
  But... a difference of 0.5 may be MEANINGLESS in practice!
  
  ┌──────────────────────────────────────────────────┐
  │ Statistical significance ≠ Practical importance  │
  │                                                  │
  │ Always report EFFECT SIZE alongside p-value      │
  └──────────────────────────────────────────────────┘
```

### P-Hacking (Data Dredging)

| Problem | Description |
|---------|-------------|
| Multiple testing | Testing many hypotheses → false positives |
| Stopping rule | Collecting data until $p < 0.05$ |
| Selective reporting | Only publishing significant results |
| Post-hoc hypotheses | Forming $H_1$ after seeing data |

**Solution:** Pre-register hypotheses; adjust for multiple comparisons (Bonferroni, FDR).

---

## 8. Relationship to Confidence Intervals

$$\text{Reject } H_0: \mu = \mu_0 \text{ at level } \alpha \iff \mu_0 \notin \text{the } (1-\alpha) \text{ CI}$$

```
  95% CI: (102.5, 107.5)
  
  Test H₀: μ = 100 → 100 NOT in CI → Reject H₀ ✓
  Test H₀: μ = 105 → 105 IS in CI  → Fail to reject ✓
  
  ──────────┤══════════════════├──────────
           102.5    105     107.5
       100 ↑
       NOT in CI
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| P-value | $P(\text{data this extreme} \mid H_0)$ |
| Decision | Reject $H_0$ if $p < \alpha$ |
| Small $p$ | Strong evidence against $H_0$ |
| Large $p$ | Weak evidence; can't conclude $H_0$ is true |
| $p \neq P(H_0 \text{ true})$ | Common misconception! |
| CI connection | $\mu_0 \notin$ CI $\iff$ $p < \alpha$ |
| Practical significance | Report effect size, not just $p$ |

---

## Quick Revision Questions

1. In a right-tailed test, $z_{obs} = 1.85$. Compute the p-value and decide at $\alpha = 0.05$.

2. Explain why "p = 0.03 means there's a 3% chance $H_0$ is true" is **incorrect**.

3. A study with $n = 50{,}000$ finds $p = 0.0001$ for a mean difference of 0.1 points. Comment on statistical vs practical significance.

4. $\chi^2_{obs} = 7.5$ with df = 3. Is $p < 0.05$? (Use: $\chi^2_{0.05, 3} = 7.815$.)

5. Explain the relationship between p-values and confidence intervals.

6. What is p-hacking and how can it be prevented?

---

[← Previous: ANOVA](05-anova.md) | [Back to README](../README.md) | [Next: Linear Regression →](../10-Regression-and-Correlation/01-linear-regression.md)
