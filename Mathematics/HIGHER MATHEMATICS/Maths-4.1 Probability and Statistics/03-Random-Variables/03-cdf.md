# Chapter 3.3: Cumulative Distribution Function (CDF)

[← Previous: Probability Mass Function (PMF)](02-pmf.md) | [Back to README](../README.md) | [Next: Continuous Random Variables →](04-continuous-random-variables.md)

---

## Overview

The **Cumulative Distribution Function (CDF)** gives the probability that a random variable is less than or equal to a given value. Unlike the PMF (discrete only) or PDF (continuous only), the CDF works for **all** random variables — discrete, continuous, or mixed.

---

## 1. Definition

The CDF of a random variable $X$ is:

$$\boxed{F_X(x) = P(X \leq x) \quad \text{for all } x \in \mathbb{R}}$$

### For Discrete RVs

$$F_X(x) = \sum_{x_i \leq x} p_X(x_i) = \sum_{x_i \leq x} P(X = x_i)$$

---

## 2. Properties of the CDF

| Property | Statement | Intuition |
|----------|-----------|-----------|
| Bounded | $0 \leq F_X(x) \leq 1$ | Probability is bounded |
| Left limit | $\lim_{x \to -\infty} F_X(x) = 0$ | No probability below all values |
| Right limit | $\lim_{x \to +\infty} F_X(x) = 1$ | Total probability is 1 |
| Non-decreasing | If $a < b$, then $F_X(a) \leq F_X(b)$ | More values ⇒ more probability |
| Right-continuous | $F_X(x) = F_X(x^+)$ | Convention for discrete jumps |
| Jump at $x_0$ | $P(X = x_0) = F_X(x_0) - F_X(x_0^-)$ | Jump size = PMF value |

```
  CDF Properties — Visual
  
  F(x)
  1.0 │                          ─────────────
      │                         │
      │                         │
      │                    ─────┘
      │                   │
      │              ─────┘
      │             │
      │        ─────┘
      │       │
      │  ─────┘
  0.0 │──
      └─────────────────────────────────────► x
      
  • Staircase shape for discrete RVs
  • Horizontal between value points
  • Jump at each value = P(X = that value)
  • Each step includes the left point (●) but not the upper right (○)
```

---

## 3. Example: Die Roll

$X$ = fair die roll, $P(X = k) = 1/6$ for $k = 1, 2, 3, 4, 5, 6$.

| $x$ | $F_X(x) = P(X \leq x)$ |
|:---:|:-----------------------:|
| $x < 1$ | 0 |
| $1 \leq x < 2$ | 1/6 |
| $2 \leq x < 3$ | 2/6 |
| $3 \leq x < 4$ | 3/6 |
| $4 \leq x < 5$ | 4/6 |
| $5 \leq x < 6$ | 5/6 |
| $x \geq 6$ | 1 |

```
  CDF of Fair Die Roll
  
  F(x)
  6/6 │                                    ●─────────
  5/6 │                            ○───────●
  4/6 │                    ○───────●
  3/6 │            ○───────●
  2/6 │    ○───────●
  1/6 │ ○──●
  0/6 │────○
      └──┬──┬──┬──┬──┬──┬──────────────────► x
         1  2  3  4  5  6
      
  ● = included (closed)    ○ = excluded (open)
  Jump at each integer = 1/6
```

---

## 4. Computing Probabilities from CDF

The CDF gives us a powerful way to compute any probability:

| Probability | Formula |
|-------------|---------|
| $P(X \leq a)$ | $F_X(a)$ |
| $P(X > a)$ | $1 - F_X(a)$ |
| $P(a < X \leq b)$ | $F_X(b) - F_X(a)$ |
| $P(X = a)$ | $F_X(a) - F_X(a^-)$ (for discrete) |
| $P(a \leq X \leq b)$ | $F_X(b) - F_X(a^-)$ |
| $P(a < X < b)$ | $F_X(b^-) - F_X(a)$ |

### Example: Using Die CDF

- $P(X \leq 3) = F_X(3) = 3/6 = 1/2$
- $P(X > 4) = 1 - F_X(4) = 1 - 4/6 = 2/6 = 1/3$
- $P(2 < X \leq 5) = F_X(5) - F_X(2) = 5/6 - 2/6 = 3/6 = 1/2$
- $P(X = 3) = F_X(3) - F_X(2) = 3/6 - 2/6 = 1/6$ ✓

---

## 5. CDF for Number of Heads (3 Coins)

$X$ = number of heads. $p_X(0) = 1/8, p_X(1) = 3/8, p_X(2) = 3/8, p_X(3) = 1/8$.

| $x$ range | $F_X(x)$ |
|:---------:|:---------:|
| $x < 0$ | 0 |
| $0 \leq x < 1$ | 1/8 |
| $1 \leq x < 2$ | 4/8 = 1/2 |
| $2 \leq x < 3$ | 7/8 |
| $x \geq 3$ | 1 |

```
  PMF (bars)                    CDF (stairs)
  
  p(x)                          F(x)
  3/8 │   ▮   ▮                 1   │                  ●────
      │   ▮   ▮                 7/8 │          ○───────●
  1/8 │▮  ▮   ▮  ▮              1/2 │  ○───────●
      │▮  ▮   ▮  ▮              1/8 │  ●
    0 └┴──┴───┴──┴─► x          0   │──○
      0  1   2   3               └──┬──┬──┬──┬──► x
                                    0  1  2  3
  
  PMF: height of each bar       CDF: cumulative sum = staircase
```

---

## 6. Relationship Between PMF and CDF

### From PMF to CDF

$$F_X(x) = \sum_{x_i \leq x} p_X(x_i)$$

### From CDF to PMF

$$p_X(x_k) = F_X(x_k) - F_X(x_{k-1})$$

(Jump at $x_k$ = $F$ value at $x_k$ minus $F$ value just before $x_k$)

```
  Conversion Diagram
  
  PMF ──────────► CDF
      Cumulative     
      Summation      
       (sum up)      
                     
  PMF ◄──────────  CDF
      Differences    
      (subtract      
       consecutive   
       values)       
```

---

## 7. Quantiles and Percentiles

The **$p$-th quantile** (or 100$p$-th percentile) of $X$ is:

$$x_p = \inf\{x : F_X(x) \geq p\}$$

| Quantile | Name |
|:--------:|------|
| $x_{0.25}$ | First quartile (Q1) |
| $x_{0.50}$ | Median (Q2) |
| $x_{0.75}$ | Third quartile (Q3) |
| $x_{0.90}$ | 90th percentile |

### Example

For the die roll: What is the median?

$F_X(3) = 3/6 = 0.5 \geq 0.5$ → median = 3 (smallest $x$ where $F(x) \geq 0.5$)

---

## 8. Real-World Applications

| Application | CDF Question |
|------------|-------------|
| **Quality** | $P(\text{defects} \leq 2)$ = acceptable rate |
| **Inventory** | $P(\text{demand} \leq k)$ = stock-out probability |
| **Testing** | $P(\text{score} \leq s)$ = percentile rank |
| **Reliability** | $F(t) = P(\text{failure by time } t)$ |
| **Finance** | Value at Risk: find $x$ where $P(\text{loss} > x) = 0.05$ |

---

## Summary Table

| Concept | Formula / Key Fact |
|---------|-------------------|
| CDF definition | $F_X(x) = P(X \leq x)$ |
| Discrete CDF | $F_X(x) = \sum_{x_i \leq x} p_X(x_i)$ (staircase) |
| $P(X > a)$ | $1 - F_X(a)$ |
| $P(a < X \leq b)$ | $F_X(b) - F_X(a)$ |
| PMF from CDF | Jump at $x_k = F_X(x_k) - F_X(x_k^-)$ |
| Properties | $0 \leq F \leq 1$, non-decreasing, right-continuous |
| Median | Smallest $x$ such that $F_X(x) \geq 0.5$ |

---

## Quick Revision Questions

1. Given PMF $p_X(1) = 0.2, p_X(2) = 0.3, p_X(3) = 0.3, p_X(4) = 0.2$, construct the CDF $F_X(x)$ and sketch it.

2. Using the CDF from Q1, find $P(X > 2)$, $P(1 < X \leq 3)$, and the median of $X$.

3. The CDF of $X$ is given by: $F(x) = 0$ for $x < 0$, $F(x) = 0.25$ for $0 \leq x < 1$, $F(x) = 0.75$ for $1 \leq x < 2$, $F(x) = 1$ for $x \geq 2$. Find the PMF of $X$.

4. **Prove** that the CDF is non-decreasing (i.e., $a < b \implies F(a) \leq F(b)$).

5. Why must the CDF be right-continuous for discrete RVs? What would go wrong if it were left-continuous?

6. The CDF satisfies $F(5) = 0.8$. What is $P(X > 5)$? Can you determine $P(X = 5)$ from this information alone?

---

[← Previous: Probability Mass Function (PMF)](02-pmf.md) | [Back to README](../README.md) | [Next: Continuous Random Variables →](04-continuous-random-variables.md)
