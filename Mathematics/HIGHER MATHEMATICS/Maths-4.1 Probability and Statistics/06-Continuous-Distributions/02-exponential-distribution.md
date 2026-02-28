# Chapter 6.2: Exponential Distribution

[← Previous: Uniform Distribution](01-uniform-distribution.md) | [Back to README](../README.md) | [Next: Normal Distribution →](03-normal-distribution.md)

---

## Overview

The **Exponential distribution** models the **time between events** in a Poisson process. It is the continuous analog of the Geometric distribution and the only continuous distribution with the **memoryless property**.

---

## 1. Definition

$$X \sim \text{Exponential}(\lambda), \quad \lambda > 0$$

### PDF

$$\boxed{f(x) = \lambda e^{-\lambda x}, \quad x \geq 0}$$

### CDF

$$\boxed{F(x) = 1 - e^{-\lambda x}, \quad x \geq 0}$$

### Survival Function

$$\boxed{P(X > x) = e^{-\lambda x}}$$

```
  PDF (λ = 1)                      CDF (λ = 1)
  
  f(x)                             F(x)
  1.0│╲                             1.0│          ━━━━━━━━━
     │  ╲                              │       ╱
  0.5│    ╲                          0.5│    ╱
     │      ╲                           │  ╱
     │        ╲─────                    │╱
  0.0└──────────────► x             0.0└──────────────► x
     0  1  2  3  4                     0  1  2  3  4
```

---

## 2. Key Properties

| Property | Value |
|----------|-------|
| Mean | $E[X] = 1/\lambda$ |
| Variance | $\text{Var}(X) = 1/\lambda^2$ |
| Std Dev | $\sigma = 1/\lambda$ |
| Median | $\ln(2)/\lambda \approx 0.693/\lambda$ |
| Mode | 0 |
| MGF | $\frac{\lambda}{\lambda - t}$, $t < \lambda$ |
| Skewness | 2 |
| Kurtosis | 9 |

> Note: Mean = Standard Deviation = $1/\lambda$. The median is about 69.3% of the mean.

### Alternative Parameterization

Some texts use $\beta = 1/\lambda$ (mean) as the parameter:

$$f(x) = \frac{1}{\beta}e^{-x/\beta}, \quad E[X] = \beta$$

---

## 3. Memoryless Property

$$\boxed{P(X > s + t \mid X > s) = P(X > t)}$$

**Proof**:

$$P(X > s+t \mid X > s) = \frac{P(X > s+t)}{P(X > s)} = \frac{e^{-\lambda(s+t)}}{e^{-\lambda s}} = e^{-\lambda t} = P(X > t) \quad \blacksquare$$

> "No wear and tear" — a component that has survived $s$ hours is stochastically identical to a new component.

```
  Memoryless Property
  
  ─────────┬──────────────────────► time
           s          s+t
  
  Given: survived to time s
  Question: P(survive t MORE time)?
  Answer: Same as P(survive t from the start)!
  
  The Exponential "doesn't remember" the past.
```

### Uniqueness

The Exponential is the **only** continuous distribution with the memoryless property.

---

## 4. Relationship to Poisson Process

```
  Poisson Process (rate λ)
  
  Events:  ×     ×  ×       ×      ×  ×     ×
  ─────────┼─────┼──┼───────┼──────┼──┼─────┼──► time
           │     │  │       │      │  │     │
           ├─T₁──┤  │       │      │  │     │
                 ├T₂┤       │      │  │     │
                    ├──T₃───┤      │  │     │
                            ├──T₄──┤  │     │
                                   ├T₅┤     │
                                      ├─T₆──┤
  
  Each Tᵢ ~ Exponential(λ)    (inter-arrival times)
  Count in [0,t] ~ Poisson(λt) (number of events)
```

---

## 5. Worked Examples

### Example 1: Light Bulb Lifetime

Mean lifetime = 1000 hours, so $\lambda = 1/1000$.

$$P(X > 1500) = e^{-1500/1000} = e^{-1.5} \approx 0.2231$$

$$P(X > 2000 \mid X > 500) = P(X > 1500) = e^{-1.5} \approx 0.2231 \quad \text{(memoryless!)}$$

### Example 2: Customer Arrivals

Customers arrive at rate 3 per hour. Time until next customer:

$X \sim \text{Exp}(3)$, $E[X] = 1/3$ hour = 20 minutes

$$P(X > 0.5) = e^{-3(0.5)} = e^{-1.5} \approx 0.2231$$

$$P(X < 0.1) = 1 - e^{-0.3} \approx 0.2592$$

### Example 3: Minimum of Exponentials

If $X_1 \sim \text{Exp}(\lambda_1)$ and $X_2 \sim \text{Exp}(\lambda_2)$ are independent:

$$\min(X_1, X_2) \sim \text{Exp}(\lambda_1 + \lambda_2)$$

**Proof**: $P(\min > t) = P(X_1 > t)P(X_2 > t) = e^{-\lambda_1 t}e^{-\lambda_2 t} = e^{-(\lambda_1+\lambda_2)t}$

---

## 6. Key Relationships

| From / To | Relationship |
|-----------|-------------|
| **Poisson** | Inter-arrival times of Poisson($\lambda$) are Exp($\lambda$) |
| **Geometric** | Discrete analog (memoryless property) |
| **Gamma** | Sum of $n$ i.i.d. Exp($\lambda$) is Gamma($n, \lambda$) |
| **Weibull** | Generalization: $X^{1/k}$ where $X \sim \text{Exp}$ |
| **Uniform** | If $U \sim U(0,1)$, then $-\ln(U)/\lambda \sim \text{Exp}(\lambda)$ |

---

## 7. Hazard Rate

$$h(x) = \frac{f(x)}{1 - F(x)} = \frac{\lambda e^{-\lambda x}}{e^{-\lambda x}} = \lambda$$

The **constant hazard rate** $\lambda$ reflects the memoryless property — the failure rate doesn't change with age.

| Distribution | Hazard Rate | Interpretation |
|:--:|:--:|:--|
| Exponential | Constant $\lambda$ | No aging |
| Weibull ($k > 1$) | Increasing | Wear out |
| Weibull ($k < 1$) | Decreasing | Infant mortality |

---

## 8. Real-World Applications

| Scenario | Rate $\lambda$ | Mean $1/\lambda$ |
|----------|:-:|:-:|
| Time between phone calls | 5/hr | 12 min |
| Time between earthquakes | 0.2/yr | 5 years |
| Radioactive decay | varies | half-life/$\ln 2$ |
| Hardware component life | 0.001/hr | 1000 hrs |
| Time between bus arrivals | 4/hr | 15 min |

---

## Summary Table

| Concept | Formula |
|---------|---------|
| PDF | $\lambda e^{-\lambda x}$, $x \geq 0$ |
| CDF | $1 - e^{-\lambda x}$ |
| Survival | $e^{-\lambda x}$ |
| Mean | $1/\lambda$ |
| Variance | $1/\lambda^2$ |
| Memoryless | $P(X>s+t \mid X>s) = P(X>t)$ |
| Min of independents | $\text{Exp}(\lambda_1 + \lambda_2)$ |
| Hazard rate | Constant $= \lambda$ |

---

## Quick Revision Questions

1. If $X \sim \text{Exp}(0.5)$, find $P(X > 2)$, $P(1 < X < 3)$, and the median.

2. The average time between failures is 500 hours. Find the probability it lasts at least 600 hours. Given it has lasted 200 hours, find the probability it lasts an additional 400 hours.

3. **Derive** $E[X] = 1/\lambda$ for the Exponential distribution using integration by parts.

4. Three servers have lifetimes $\text{Exp}(0.01)$, $\text{Exp}(0.02)$, $\text{Exp}(0.03)$ independently. What is the distribution of the time until the first failure?

5. Prove that the hazard rate of the Exponential is constant and explain why this means "no aging."

6. If $X_1, X_2, \ldots, X_n$ are i.i.d. $\text{Exp}(\lambda)$, what is the distribution of $S = X_1 + X_2 + \cdots + X_n$?

---

[← Previous: Uniform Distribution](01-uniform-distribution.md) | [Back to README](../README.md) | [Next: Normal Distribution →](03-normal-distribution.md)
