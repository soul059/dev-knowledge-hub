# Chapter 9.5: ANOVA (Analysis of Variance)

[← Previous: Chi-Square Test](04-chi-square-test.md) | [Back to README](../README.md) | [Next: P-Values →](06-p-values.md)

---

## Overview

**ANOVA** (Analysis of Variance) tests whether the means of three or more groups are equal. Despite its name, it compares means by analyzing the variance structure of the data.

---

## 1. The Problem ANOVA Solves

**Why not just do multiple t-tests?**

For $k = 4$ groups, you'd need $\binom{4}{2} = 6$ pairwise t-tests. At $\alpha = 0.05$ each:

$$P(\text{at least one Type I error}) = 1 - (1 - 0.05)^6 = 1 - 0.735 = 0.265$$

That's a 26.5% false positive rate! ANOVA tests all groups simultaneously at the desired $\alpha$.

```
  Multiple t-tests:                  ANOVA:
  
  Group 1 ←→ Group 2                Group 1 ┐
  Group 1 ←→ Group 3                Group 2 ┤── One test!
  Group 1 ←→ Group 4                Group 3 ┤   α = 0.05
  Group 2 ←→ Group 3                Group 4 ┘
  Group 2 ←→ Group 4
  Group 3 ←→ Group 4
  
  6 tests, inflated α              1 test, controlled α
```

---

## 2. One-Way ANOVA

### Hypotheses

$$H_0: \mu_1 = \mu_2 = \cdots = \mu_k \quad (\text{all group means equal})$$

$$H_1: \text{At least one } \mu_i \text{ differs}$$

### Model

$$X_{ij} = \mu + \alpha_i + \varepsilon_{ij}, \quad \varepsilon_{ij} \overset{\text{iid}}{\sim} N(0, \sigma^2)$$

where $i = 1, \ldots, k$ (groups), $j = 1, \ldots, n_i$ (observations).

### The Key Idea: Decomposing Variance

$$\underbrace{\text{Total Variation}}_{\text{SST}} = \underbrace{\text{Between-Group Variation}}_{\text{SSB}} + \underbrace{\text{Within-Group Variation}}_{\text{SSW}}$$

```
  Variance Decomposition
  
  ●  ●     ●●  ●     ●  ●●
   ● ●     ●  ●●     ●● ●
  ──┬──   ──┬──    ──┬──
    x̄₁      x̄₂      x̄₃      Group means
  ──────────┬──────────
            x̄           Grand mean
  
  SST = How far each point is from x̄  (total)
  SSB = How far each x̄ᵢ is from x̄    (between groups)
  SSW = How far each point is from its x̄ᵢ (within group)
```

---

## 3. Computing the Sums of Squares

$$\boxed{SST = \sum_{i=1}^k \sum_{j=1}^{n_i} (X_{ij} - \bar{X})^2}$$

$$\boxed{SSB = \sum_{i=1}^k n_i(\bar{X}_i - \bar{X})^2}$$

$$\boxed{SSW = \sum_{i=1}^k \sum_{j=1}^{n_i} (X_{ij} - \bar{X}_i)^2}$$

**Identity:** $SST = SSB + SSW$

### Mean Squares

$$MSB = \frac{SSB}{k-1}, \quad MSW = \frac{SSW}{N-k}$$

where $N = \sum n_i$ (total sample size).

---

## 4. The F-Test Statistic

$$\boxed{F = \frac{MSB}{MSW} = \frac{SSB/(k-1)}{SSW/(N-k)} \sim F(k-1, N-k)}$$

under $H_0$.

**Decision:** Reject $H_0$ if $F > F_{\alpha, k-1, N-k}$ (always right-tailed).

**Intuition:**
- If $H_0$ true: group means are similar → $SSB$ small → $F \approx 1$
- If $H_0$ false: group means differ → $SSB$ large → $F$ large

---

## 5. ANOVA Table

| Source | SS | df | MS | F |
|--------|-----|------|------|------|
| Between groups | $SSB$ | $k-1$ | $MSB = SSB/(k-1)$ | $F = MSB/MSW$ |
| Within groups | $SSW$ | $N-k$ | $MSW = SSW/(N-k)$ | |
| **Total** | $SST$ | $N-1$ | | |

---

## 6. Worked Example

Three fertilizers tested on plant growth (cm):

| Fertilizer A | Fertilizer B | Fertilizer C |
|:-:|:-:|:-:|
| 20 | 22 | 25 |
| 21 | 24 | 28 |
| 19 | 23 | 26 |
| 22 | 25 | 27 |

$n_1 = n_2 = n_3 = 4$, $N = 12$, $k = 3$

**Group means:** $\bar{X}_1 = 20.5$, $\bar{X}_2 = 23.5$, $\bar{X}_3 = 26.5$

**Grand mean:** $\bar{X} = (82 + 94 + 106)/12 = 23.5$

**SSB:** $4(20.5-23.5)^2 + 4(23.5-23.5)^2 + 4(26.5-23.5)^2 = 4(9) + 0 + 4(9) = 72$

**SSW:** $(20-20.5)^2 + (21-20.5)^2 + (19-20.5)^2 + (22-20.5)^2 + \cdots$

$= (0.25+0.25+2.25+2.25) + (0.25+0.25+2.25+2.25) + (2.25+2.25+0.25+0.25) = 5 + 5 + 5 = 15$

| Source | SS | df | MS | F |
|--------|-----|------|------|------|
| Between | 72 | 2 | 36 | **21.6** |
| Within | 15 | 9 | 1.667 | |
| Total | 87 | 11 | | |

$F_{0.05, 2, 9} = 4.26$. Since $21.6 > 4.26$: **Reject $H_0$**.

At least one fertilizer produces significantly different growth.

---

## 7. Post-Hoc Tests

ANOVA tells us *at least one mean differs* but not *which ones*. Post-hoc tests determine specific differences:

| Method | Description | Controls |
|--------|-------------|----------|
| Tukey's HSD | All pairwise comparisons | Family-wise error |
| Bonferroni | Adjust $\alpha' = \alpha/m$ for $m$ comparisons | Family-wise error |
| Scheffé | Conservative, any contrast | Family-wise error |
| Dunnett | Compare all groups to control | Family-wise error |

### Tukey's HSD Formula

$$HSD = q_{\alpha, k, N-k} \cdot \sqrt{\frac{MSW}{n}}$$

Reject if $|\bar{X}_i - \bar{X}_j| > HSD$.

---

## 8. Assumptions

| Assumption | How to Check | If Violated |
|------------|-------------|-------------|
| Normality | Shapiro-Wilk test, Q-Q plot | Kruskal-Wallis (non-parametric) |
| Equal variances | Levene's test, Bartlett's test | Welch's ANOVA |
| Independence | Study design | Consider repeated measures |

```
  ANOVA Assumptions
  
  Group 1:    ╱╲        Equal spread (σ₁ = σ₂ = σ₃) ✓
  Group 2:     ╱╲       Normal within each group ✓
  Group 3:      ╱╲      Independent observations ✓
  
  ──────┬──────┬──────┬──── 
       μ₁     μ₂     μ₃
  
  Means may differ, but shapes and spreads are the same
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| Purpose | Compare $k \geq 3$ group means |
| $H_0$ | $\mu_1 = \mu_2 = \cdots = \mu_k$ |
| Decomposition | $SST = SSB + SSW$ |
| Test statistic | $F = MSB/MSW \sim F(k-1, N-k)$ |
| Reject $H_0$ | If $F > F_{\alpha, k-1, N-k}$ |
| If $F$ significant | Use post-hoc tests (Tukey, Bonferroni) |
| Assumptions | Normality, equal variances, independence |

---

## Quick Revision Questions

1. Why is ANOVA preferred over multiple t-tests when comparing three or more groups?

2. From the ANOVA table: SSB = 120, SSW = 80, $k = 4$, $N = 40$. Complete the table and perform the F-test at $\alpha = 0.05$.

3. What does a significant F-test tell you? What does it NOT tell you?

4. List three post-hoc tests and when you would use each.

5. If $F = 1.2$ with df $= (3, 36)$ at $\alpha = 0.05$: what is your conclusion and interpretation?

6. State the assumptions for one-way ANOVA and how to handle violations.

---

[← Previous: Chi-Square Test](04-chi-square-test.md) | [Back to README](../README.md) | [Next: P-Values →](06-p-values.md)
