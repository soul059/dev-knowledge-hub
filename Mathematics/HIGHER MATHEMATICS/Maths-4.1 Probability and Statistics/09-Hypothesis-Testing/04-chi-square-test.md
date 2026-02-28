# Chapter 9.4: Chi-Square Test

[← Previous: Z-Test and t-Test](03-z-test-and-t-test.md) | [Back to README](../README.md) | [Next: ANOVA →](05-anova.md)

---

## Overview

The **chi-square ($\chi^2$) test** is used for categorical data — testing whether observed frequencies match expected frequencies (goodness-of-fit) or whether two categorical variables are related (independence).

---

## 1. Chi-Square Goodness-of-Fit Test

### Purpose

Test whether observed data follow a hypothesized distribution.

### Test Statistic

$$\boxed{\chi^2 = \sum_{i=1}^k \frac{(O_i - E_i)^2}{E_i}}$$

where:
- $O_i$ = observed frequency in category $i$
- $E_i$ = expected frequency in category $i$ under $H_0$
- $k$ = number of categories

Under $H_0$: $\chi^2 \sim \chi^2(k - 1 - m)$ where $m$ = number of parameters estimated from data.

### Hypotheses

$$H_0: \text{Data follow the specified distribution}$$
$$H_1: \text{Data do not follow the specified distribution}$$

**Always right-tailed:** Reject $H_0$ if $\chi^2 > \chi^2_{\alpha, \text{df}}$.

### Example: Fair Die

Roll a die 120 times:

| Face | 1 | 2 | 3 | 4 | 5 | 6 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| $O_i$ | 25 | 17 | 15 | 23 | 24 | 16 |
| $E_i$ | 20 | 20 | 20 | 20 | 20 | 20 |

$$\chi^2 = \frac{(25-20)^2}{20} + \frac{(17-20)^2}{20} + \frac{(15-20)^2}{20} + \frac{(23-20)^2}{20} + \frac{(24-20)^2}{20} + \frac{(16-20)^2}{20}$$

$$= \frac{25 + 9 + 25 + 9 + 16 + 16}{20} = \frac{100}{20} = 5.0$$

$\text{df} = 6 - 1 = 5$, $\chi^2_{0.05, 5} = 11.07$.

Since $5.0 < 11.07$: **Fail to reject $H_0$**. No evidence die is unfair.

```
  Chi-square Decision
  
  f(x)
  │╲
  │ ╲                          Fail to reject H₀
  │  ╲
  │   ╲
  │    ╲                  ██████
  │     ╲________________█████████
  └──────────────────────────────► x
  0               5.0       11.07
                  ↑           ↑
                χ²_calc    χ²_crit
                           
              NOT in rejection region → Fail to reject
```

---

## 2. Chi-Square Test for Independence

### Purpose

Test whether two categorical variables are independent (no association).

### Setup: Contingency Table

| | $B_1$ | $B_2$ | $\cdots$ | $B_c$ | **Row Total** |
|:-:|:-:|:-:|:-:|:-:|:-:|
| $A_1$ | $O_{11}$ | $O_{12}$ | $\cdots$ | $O_{1c}$ | $R_1$ |
| $A_2$ | $O_{21}$ | $O_{22}$ | $\cdots$ | $O_{2c}$ | $R_2$ |
| $\vdots$ | | | | | |
| $A_r$ | $O_{r1}$ | $O_{r2}$ | $\cdots$ | $O_{rc}$ | $R_r$ |
| **Col Total** | $C_1$ | $C_2$ | $\cdots$ | $C_c$ | $n$ |

### Expected Frequencies

$$\boxed{E_{ij} = \frac{R_i \times C_j}{n}}$$

### Test Statistic

$$\chi^2 = \sum_{i=1}^r \sum_{j=1}^c \frac{(O_{ij} - E_{ij})^2}{E_{ij}}, \quad \text{df} = (r-1)(c-1)$$

### Example: Gender and Preference

| | Product A | Product B | Product C | Total |
|:-:|:-:|:-:|:-:|:-:|
| Male | 30 | 40 | 30 | 100 |
| Female | 50 | 30 | 20 | 100 |
| Total | 80 | 70 | 50 | 200 |

**Expected frequencies** (under independence):

| | Product A | Product B | Product C |
|:-:|:-:|:-:|:-:|
| Male | $100 \times 80/200 = 40$ | $100 \times 70/200 = 35$ | $100 \times 50/200 = 25$ |
| Female | $100 \times 80/200 = 40$ | $100 \times 70/200 = 35$ | $100 \times 50/200 = 25$ |

$$\chi^2 = \frac{(30-40)^2}{40} + \frac{(40-35)^2}{35} + \frac{(30-25)^2}{25} + \frac{(50-40)^2}{40} + \frac{(30-35)^2}{35} + \frac{(20-25)^2}{25}$$

$$= 2.5 + 0.714 + 1.0 + 2.5 + 0.714 + 1.0 = 8.43$$

$\text{df} = (2-1)(3-1) = 2$, $\chi^2_{0.05, 2} = 5.991$.

Since $8.43 > 5.991$: **Reject $H_0$**. Gender and product preference are related.

---

## 3. Chi-Square Test for Homogeneity

Same formula as independence test, but different sampling:

| Independence Test | Homogeneity Test |
|-------------------|------------------|
| One sample, two variables | Two+ populations, one variable |
| "Are X and Y related?" | "Same distribution across groups?" |
| $\text{df} = (r-1)(c-1)$ | Same formula |

---

## 4. Assumptions and Conditions

| Condition | Requirement |
|-----------|-------------|
| Expected frequencies | $E_i \geq 5$ for all cells |
| Sample | Random and independent observations |
| Variables | Categorical (nominal or ordinal) |
| If $E_i < 5$ | Combine categories or use Fisher's exact test |

```
  Checking Assumptions
  
  Expected frequencies:
  ┌──────┬──────┬──────┐
  │  40  │  35  │  25  │  All ≥ 5 ✓
  ├──────┼──────┼──────┤
  │  40  │  35  │  25  │  All ≥ 5 ✓
  └──────┴──────┴──────┘
  
  If any cell < 5:
  ┌──────┬──────┬──────┐
  │  40  │  35  │   3  │  ← Problem!
  ├──────┼──────┼──────┤       Combine with
  │  40  │  35  │   7  │       adjacent category
  └──────┴──────┴──────┘
```

---

## 5. Yates' Continuity Correction

For $2 \times 2$ tables with small expected frequencies:

$$\chi^2_{\text{Yates}} = \sum \frac{(|O_{ij} - E_{ij}| - 0.5)^2}{E_{ij}}$$

Makes the discrete $\chi^2$ statistic better approximate the continuous $\chi^2$ distribution.

---

## 6. Measuring Association Strength

After finding a significant chi-square, measure **how strong** the association is:

| Measure | Formula | Range |
|---------|---------|-------|
| Cramér's V | $V = \sqrt{\chi^2/(n \cdot \min(r-1, c-1))}$ | $[0, 1]$ |
| Phi coefficient ($\phi$) | $\phi = \sqrt{\chi^2/n}$ | $[0, 1]$ (for $2 \times 2$) |
| Contingency coefficient | $C = \sqrt{\chi^2/(\chi^2 + n)}$ | $[0, 1)$ |

| $V$ or $\phi$ | Interpretation |
|--------------|----------------|
| $< 0.1$ | Negligible |
| $0.1 - 0.3$ | Weak |
| $0.3 - 0.5$ | Moderate |
| $> 0.5$ | Strong |

---

## Summary Table

| Test | Purpose | df | Reject if |
|------|---------|-----|----------|
| Goodness-of-fit | Data matches distribution? | $k - 1 - m$ | $\chi^2 > \chi^2_{\alpha, \text{df}}$ |
| Independence | Two variables related? | $(r-1)(c-1)$ | $\chi^2 > \chi^2_{\alpha, \text{df}}$ |
| Homogeneity | Same distribution across groups? | $(r-1)(c-1)$ | $\chi^2 > \chi^2_{\alpha, \text{df}}$ |
| All tests | $\chi^2 = \sum(O-E)^2/E$ | | Always right-tailed |

---

## Quick Revision Questions

1. A coin is flipped 200 times: 115 heads, 85 tails. Is the coin fair at $\alpha = 0.05$?

2. In a $3 \times 4$ contingency table with $n = 300$, compute the degrees of freedom.

3. What should you do if expected frequencies are less than 5?

4. Compute Cramér's $V$ for the gender-preference example above.

5. Explain the difference between the test for independence and the test for homogeneity.

6. Given observed = {20, 30, 50} and expected under $H_0$ = {25, 25, 50}: compute $\chi^2$ and the p-value conclusion at $\alpha = 0.05$.

---

[← Previous: Z-Test and t-Test](03-z-test-and-t-test.md) | [Back to README](../README.md) | [Next: ANOVA →](05-anova.md)
