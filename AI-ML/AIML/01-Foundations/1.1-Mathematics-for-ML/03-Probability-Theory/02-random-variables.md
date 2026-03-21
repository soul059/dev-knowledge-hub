# Chapter 2: Random Variables

> **Unit 3: Probability Theory** | **Topic**: Discrete & Continuous Random Variables, PMF, PDF, CDF

---

## 1. Overview

A **random variable** (RV) is a function that maps outcomes in a sample space to real numbers. It is
the bridge between abstract probability theory and the quantitative world of data and machine learning.
Every feature in a dataset, every model prediction, and every loss value can be viewed as a random variable.

---

## 2. Definition

A random variable X is a function X: Ω → ℝ that assigns a numerical value to each outcome in the
sample space Ω.

```
Experiment: Roll a die       Ω = {1, 2, 3, 4, 5, 6}
Random variable X = face value    X(ω) = ω

Experiment: Flip 3 coins     Ω = {HHH, HHT, HTH, ...}
Random variable X = # of heads    X(HHT) = 2, X(TTT) = 0
```

---

## 3. Discrete vs Continuous Random Variables

| Property            | Discrete RV                        | Continuous RV                        |
|---------------------|------------------------------------|--------------------------------------|
| Values              | Countable set (finite or infinite) | Uncountable (interval of ℝ)          |
| Described by        | PMF (Probability Mass Function)    | PDF (Probability Density Function)   |
| P(X = x)            | Can be > 0                        | Always = 0                           |
| Summation           | Σ P(X = xᵢ) = 1                  | ∫ f(x) dx = 1                       |
| Examples            | Coin flips, word counts, labels   | Height, weight, pixel intensity      |

---

## 4. Probability Mass Function (PMF)

For a **discrete** random variable X, the PMF gives the probability of each value:

```
p(x) = P(X = x)

Properties:
  1. p(x) ≥ 0  for all x
  2. Σ_x p(x) = 1
```

### Example: Fair Die

```
x     |  1    2    3    4    5    6
------+--------------------------------
p(x)  | 1/6  1/6  1/6  1/6  1/6  1/6

PMF Visualization:

p(x)
1/6 │  █    █    █    █    █    █
    │  █    █    █    █    █    █
    │  █    █    █    █    █    █
    └──1────2────3────4────5────6──→ x
```

### Example: Biased Coin (X = 1 for heads, 0 for tails, p = 0.7)

```
x     |  0      1
------+-------------
p(x)  |  0.3    0.7
```

---

## 5. Probability Density Function (PDF)

For a **continuous** random variable X, the PDF f(x) describes the relative likelihood:

```
Properties:
  1. f(x) ≥ 0  for all x
  2. ∫_{-∞}^{∞} f(x) dx = 1
  3. P(a ≤ X ≤ b) = ∫_a^b f(x) dx
```

> ⚠️ **Important**: f(x) is NOT a probability. It can be greater than 1. Only the integral
> over an interval gives a probability.

### Example: Uniform Distribution on [0, 1]

```
f(x) = 1   for 0 ≤ x ≤ 1
       0   otherwise

PDF Visualization:

f(x)
  1 │ ┌────────────────────┐
    │ │████████████████████│
    │ │████████████████████│
  0 └─┴────────────────────┴──→ x
    0                       1
```

**P(0.2 ≤ X ≤ 0.5) = ∫₀.₂^0.5 1 dx = 0.5 − 0.2 = 0.3**

---

## 6. Cumulative Distribution Function (CDF)

The CDF gives the probability that X takes a value **less than or equal to** x:

```
F(x) = P(X ≤ x)

For discrete:    F(x) = Σ_{xᵢ ≤ x} p(xᵢ)
For continuous:  F(x) = ∫_{-∞}^{x} f(t) dt
```

### Properties of CDF

```
1. F(-∞) = 0,  F(∞) = 1
2. F is non-decreasing: if a < b, then F(a) ≤ F(b)
3. P(a < X ≤ b) = F(b) - F(a)
4. For continuous RVs: f(x) = dF(x)/dx  (PDF is derivative of CDF)
```

### Example: CDF of a Fair Die

```
x      |  <1    1     2     3     4     5     6    >6
-------+----------------------------------------------
F(x)   |  0    1/6   2/6   3/6   4/6   5/6   6/6   1

CDF Visualization (step function):

F(x)
  1 │                                    ●────────
5/6 │                              ●─────┘
4/6 │                        ●─────┘
3/6 │                  ●─────┘
2/6 │            ●─────┘
1/6 │      ●─────┘
  0 │──────┘
    └──0──1──2──3──4──5──6──7──→ x
```

### Example: CDF of Uniform[0,1]

```
         ⎧ 0       if x < 0
F(x) =  ⎨ x       if 0 ≤ x ≤ 1
         ⎩ 1       if x > 1

F(x)
  1 │            ●──────────
    │          ╱
    │        ╱
    │      ╱
    │    ╱
  0 │──●
    └──0────0.5────1────→ x
```

---

## 7. Relationships Between PMF/PDF and CDF

```
┌────────────┐         ┌────────────┐
│   PMF/PDF  │ ──────→ │    CDF     │
│   p(x)/f(x)│  Sum/   │   F(x)     │
│            │ Integrate│            │
└────────────┘         └────────────┘
      ↑                      │
      │    Difference/       │
      └──── Differentiate ───┘
```

---

## 8. Step-by-Step Example: Custom Discrete RV

**Problem**: A game costs $2 to play. You roll a die: win $5 for 6, $1 for 4-5, nothing otherwise.
Let X = net profit. Find the PMF and CDF.

```
Step 1: Define X
  Roll 1,2,3  → Win $0, Net = 0 - 2 = -$2
  Roll 4,5    → Win $1, Net = 1 - 2 = -$1
  Roll 6      → Win $5, Net = 5 - 2 = +$3

Step 2: PMF
  P(X = -2) = 3/6 = 1/2
  P(X = -1) = 2/6 = 1/3
  P(X = +3) = 1/6

  Verify: 1/2 + 1/3 + 1/6 = 3/6 + 2/6 + 1/6 = 6/6 = 1 ✓

Step 3: CDF
  F(x) = 0       for x < -2
  F(x) = 1/2     for -2 ≤ x < -1
  F(x) = 5/6     for -1 ≤ x < 3
  F(x) = 1       for x ≥ 3
```

---

## 9. Python Code: Visualizing PMF, PDF, CDF

```python
import numpy as np
import matplotlib.pyplot as plt

# --- Discrete RV: PMF and CDF ---
x_disc = np.array([-2, -1, 3])
pmf = np.array([1/2, 1/3, 1/6])

fig, axes = plt.subplots(1, 2, figsize=(12, 4))

axes[0].stem(x_disc, pmf)
axes[0].set_title("PMF of Net Profit")
axes[0].set_xlabel("x"); axes[0].set_ylabel("P(X=x)")

cdf_vals = np.cumsum(pmf)
axes[1].step(x_disc, cdf_vals, where='post')
axes[1].set_title("CDF of Net Profit")
axes[1].set_xlabel("x"); axes[1].set_ylabel("F(x)")
plt.tight_layout(); plt.show()
```

```python
# --- Continuous RV: Normal Distribution PDF and CDF ---
from scipy import stats

x = np.linspace(-4, 4, 300)
mu, sigma = 0, 1
pdf = stats.norm.pdf(x, mu, sigma)
cdf = stats.norm.cdf(x, mu, sigma)

fig, axes = plt.subplots(1, 2, figsize=(12, 4))
axes[0].plot(x, pdf, 'b-', lw=2)
axes[0].fill_between(x, pdf, alpha=0.2)
axes[0].set_title(f"PDF: N({mu},{sigma}²)")

axes[1].plot(x, cdf, 'r-', lw=2)
axes[1].set_title(f"CDF: N({mu},{sigma}²)")
plt.tight_layout(); plt.show()
```

---

## 10. Random Variables in Machine Learning

In ML, **every data feature is a random variable**:

```
Dataset: House Prices
┌──────────┬───────────┬────────┬─────────┐
│ Sq. Feet │  Bedrooms │  Age   │  Price   │
│ (cont.)  │ (discrete)│(cont.) │ (cont.) │
├──────────┼───────────┼────────┼─────────┤
│  X₁      │  X₂       │  X₃    │  Y      │
└──────────┴───────────┴────────┴─────────┘

X₁ ~ Continuous RV (PDF describes distribution of house sizes)
X₂ ~ Discrete RV   (PMF gives probability of 1, 2, 3, ... bedrooms)
Y  ~ Target RV     (what we want to predict)
```

| ML Context                  | Random Variable Interpretation                    |
|-----------------------------|---------------------------------------------------|
| Classification output       | Discrete RV over class labels                     |
| Regression output           | Continuous RV (predicted value)                   |
| Softmax probabilities       | PMF over classes                                  |
| Neural network weights      | RVs in Bayesian deep learning                     |
| Training loss               | RV that depends on random mini-batch selection    |
| Data augmentation            | Introduces randomness → RV transformations       |

---

## 11. Summary Table

| Concept       | Symbol       | Discrete                      | Continuous                     |
|---------------|-------------|-------------------------------|--------------------------------|
| Distribution  | —           | PMF: p(x)                     | PDF: f(x)                     |
| Point prob.   | P(X=x)      | p(x) ≥ 0                     | Always 0                      |
| Range prob.   | P(a≤X≤b)    | Σ p(xᵢ) for a≤xᵢ≤b          | ∫_a^b f(x)dx                  |
| Normalization | —           | Σ p(x) = 1                   | ∫f(x)dx = 1                   |
| CDF           | F(x)        | Σ_{xᵢ≤x} p(xᵢ)              | ∫_{-∞}^x f(t)dt               |
| PDF from CDF  | —           | Differences in F              | f(x) = dF/dx                  |

---

## 12. Quick Revision Questions

1. **What is the difference between a random variable and an event?**
2. **Can a PDF value exceed 1?** Why or why not?
3. **Given PMF: P(X=1)=0.2, P(X=2)=0.5, P(X=3)=0.3,** compute F(2).
4. **Why is P(X = 3.5) = 0 for a continuous RV?** Explain using the PDF.
5. **In a classification model with 4 classes,** is the output a discrete or continuous RV?
6. **How do you recover the PDF from the CDF** for a continuous random variable?

---

## 13. Navigation

| ← Previous | Next → |
|:-----------:|:------:|
| [Chapter 1: Probability Basics](01-probability-basics.md) | [Chapter 3: Probability Distributions](03-probability-distributions.md) |

---

*Unit 3 · Probability Theory · Chapter 2 of 9*
