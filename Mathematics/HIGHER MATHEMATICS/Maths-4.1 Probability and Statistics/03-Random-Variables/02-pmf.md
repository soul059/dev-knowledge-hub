# Chapter 3.2: Probability Mass Function (PMF)

[← Previous: Discrete Random Variables](01-discrete-random-variables.md) | [Back to README](../README.md) | [Next: Cumulative Distribution Function (CDF) →](03-cdf.md)

---

## Overview

The **Probability Mass Function (PMF)** completely describes the probability distribution of a discrete random variable. It gives the probability that the random variable takes on each specific value.

---

## 1. Definition

The PMF of a discrete random variable $X$ is:

$$\boxed{p_X(x) = P(X = x) \quad \text{for each } x \in R_X}$$

### Properties

A valid PMF must satisfy:

| Property | Condition |
|----------|-----------|
| Non-negativity | $p_X(x) \geq 0$ for all $x$ |
| Normalization | $\sum_{x \in R_X} p_X(x) = 1$ |
| Zero outside support | $p_X(x) = 0$ for $x \notin R_X$ |

---

## 2. Representations

### Tabular Form

**Example 1:** $X$ = number of heads in 2 fair coin tosses.

| $x$ | 0 | 1 | 2 |
|:---:|:-:|:-:|:-:|
| $p_X(x)$ | 1/4 | 2/4 | 1/4 |

Check: $1/4 + 2/4 + 1/4 = 1$ ✓

### Formula Form

**Example 2:** $X$ = roll of a fair die.

$$p_X(x) = \frac{1}{6}, \quad x \in \{1, 2, 3, 4, 5, 6\}$$

### Graphical Form (Bar Chart)

```
  PMF of X = number of heads in 3 coin tosses
  
  P(X=x)
  3/8 │       ▮     ▮
      │       ▮     ▮
  2/8 │       ▮     ▮
      │       ▮     ▮
  1/8 │  ▮    ▮     ▮    ▮
      │  ▮    ▮     ▮    ▮
    0 └──┴────┴─────┴────┴──► x
       0     1      2     3
  
  P(X=0) = 1/8    P(X=1) = 3/8
  P(X=2) = 3/8    P(X=3) = 1/8
  Sum = 1 ✓
```

---

## 3. Common Discrete PMFs (Preview)

| Distribution | PMF $p_X(x)$ | Support |
|-------------|:-------------:|:-------:|
| Bernoulli($p$) | $p^x(1-p)^{1-x}$ | $x \in \{0,1\}$ |
| Binomial($n,p$) | $\binom{n}{x}p^x(1-p)^{n-x}$ | $x \in \{0,1,...,n\}$ |
| Poisson($\lambda$) | $\frac{e^{-\lambda}\lambda^x}{x!}$ | $x \in \{0,1,2,...\}$ |
| Geometric($p$) | $(1-p)^{x-1}p$ | $x \in \{1,2,3,...\}$ |
| Uniform (discrete) | $\frac{1}{n}$ | $x \in \{x_1,...,x_n\}$ |

*(Each is covered in detail in Units 5.)*

---

## 4. Finding PMF from Experiments

### Step-by-Step Process

```
  ┌────────────────────────┐
  │ 1. Define experiment   │
  │    and sample space S  │
  └──────────┬─────────────┘
             ▼
  ┌────────────────────────┐
  │ 2. Define random       │
  │    variable X          │
  └──────────┬─────────────┘
             ▼
  ┌────────────────────────┐
  │ 3. Find support of X   │
  │    (possible values)   │
  └──────────┬─────────────┘
             ▼
  ┌────────────────────────┐
  │ 4. For each x in       │
  │    support, count      │
  │    favorable outcomes  │
  └──────────┬─────────────┘
             ▼
  ┌────────────────────────┐
  │ 5. P(X=x) = favorable  │
  │           / total      │
  └──────────┬─────────────┘
             ▼
  ┌────────────────────────┐
  │ 6. Verify: sum = 1     │
  └────────────────────────┘
```

### Example 3: Sum of Two Dice

$X$ = sum of two fair dice.

| $x$ | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
|:---:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:--:|:--:|:--:|
| favorable | 1 | 2 | 3 | 4 | 5 | 6 | 5 | 4 | 3 | 2 | 1 |
| $p_X(x)$ | 1/36 | 2/36 | 3/36 | 4/36 | 5/36 | 6/36 | 5/36 | 4/36 | 3/36 | 2/36 | 1/36 |

```
  PMF of Sum of Two Dice
  
  P(X=x)
  6/36 │              ▮
  5/36 │          ▮   ▮   ▮
  4/36 │       ▮  ▮   ▮   ▮  ▮
  3/36 │    ▮  ▮  ▮   ▮   ▮  ▮  ▮
  2/36 │ ▮  ▮  ▮  ▮   ▮   ▮  ▮  ▮  ▮
  1/36 │ ▮  ▮  ▮  ▮   ▮   ▮  ▮  ▮  ▮  ▮  ▮
     0 └─┴──┴──┴──┴───┴───┴──┴──┴──┴──┴──┴─► x
       2  3  4  5   6   7  8  9 10 11 12
  
  Symmetric around x = 7
```

---

## 5. Computing Probabilities from PMF

Once the PMF is known, any probability involving $X$ can be computed:

$$P(a \leq X \leq b) = \sum_{x=a}^{b} p_X(x)$$

$$P(X > a) = \sum_{x > a} p_X(x) = 1 - \sum_{x \leq a} p_X(x)$$

### Example 4

Using the two-dice sum PMF, find:

**(a)** $P(X \leq 4) = P(X=2) + P(X=3) + P(X=4) = \frac{1+2+3}{36} = \frac{6}{36} = \frac{1}{6}$

**(b)** $P(5 \leq X \leq 9) = \frac{4+5+6+5+4}{36} = \frac{24}{36} = \frac{2}{3}$

**(c)** $P(X > 10) = P(X=11) + P(X=12) = \frac{2+1}{36} = \frac{3}{36} = \frac{1}{12}$

**(d)** $P(X \text{ is even}) = P(2)+P(4)+P(6)+P(8)+P(10)+P(12) = \frac{1+3+5+5+3+1}{36} = \frac{18}{36} = \frac{1}{2}$

---

## 6. Joint and Marginal PMF (Preview)

For two random variables $X$ and $Y$:

**Joint PMF:** $p_{X,Y}(x,y) = P(X=x, Y=y)$

**Marginal PMFs:**

$$p_X(x) = \sum_{y} p_{X,Y}(x,y) \qquad p_Y(y) = \sum_{x} p_{X,Y}(x,y)$$

*(Covered in detail in Unit 7.)*

---

## 7. Worked Example: PMF Construction

**Problem:** A box has 5 red and 3 blue balls. Two are drawn without replacement. $X$ = number of red balls drawn. Find the PMF of $X$.

**Solution:**

Total ways to draw 2 from 8: $\binom{8}{2} = 28$

| $x$ | Favorable | $p_X(x)$ |
|:---:|:---------:|:---------:|
| 0 | $\binom{5}{0}\binom{3}{2} = 1 \times 3 = 3$ | $3/28$ |
| 1 | $\binom{5}{1}\binom{3}{1} = 5 \times 3 = 15$ | $15/28$ |
| 2 | $\binom{5}{2}\binom{3}{0} = 10 \times 1 = 10$ | $10/28$ |

Check: $3 + 15 + 10 = 28$ → $\frac{28}{28} = 1$ ✓

```
  PMF of X (number of red balls)
  
  P(X=x)
  15/28 │     ▮
        │     ▮
  10/28 │     ▮     ▮
        │     ▮     ▮
   3/28 │ ▮   ▮     ▮
        │ ▮   ▮     ▮
      0 └─┴───┴─────┴──► x
        0     1      2
```

---

## 8. Real-World Applications

| Scenario | Random Variable $X$ | PMF Type |
|----------|:-------------------:|:--------:|
| Number of heads in $n$ tosses | Binomial | $\binom{n}{x}p^x(1-p)^{n-x}$ |
| Emails per hour | Poisson | $e^{-\lambda}\lambda^x/x!$ |
| Die roll | Uniform | $1/6$ |
| Product passes/fails | Bernoulli | $p^x(1-p)^{1-x}$ |
| Survey responses (agree/disagree) | Bernoulli/Multinomial | Depends on categories |

---

## Summary Table

| Concept | Key Fact |
|---------|---------|
| PMF definition | $p_X(x) = P(X = x)$ |
| Non-negativity | $p_X(x) \geq 0$ |
| Normalization | $\sum p_X(x) = 1$ |
| Probability of a range | $P(a \leq X \leq b) = \sum_{x=a}^{b} p_X(x)$ |
| Complement | $P(X > a) = 1 - P(X \leq a)$ |
| Representations | Table, formula, bar chart |
| Construction | Count favorable outcomes for each $x$ value |

---

## Quick Revision Questions

1. A random variable has PMF $p_X(x) = c \cdot x$ for $x \in \{1, 2, 3, 4\}$. Find $c$.

2. Three coins are tossed. Let $X$ = number of tails. Find the PMF of $X$ in tabular form and verify the normalization property.

3. Is $p(x) = \frac{x}{10}$ for $x \in \{1, 2, 3, 4\}$ a valid PMF? Why or why not?

4. Given the PMF $p_X(x) = \frac{1}{2^x}$ for $x = 1, 2, 3, \ldots$. Verify it sums to 1 and find $P(X \leq 3)$.

5. A bag has 4 red, 3 white, and 2 blue balls. Three are drawn without replacement. Let $X$ = number of red balls. Construct the PMF of $X$.

6. Sketch the PMF of a fair die roll. How does it differ from the PMF of the sum of two dice?

---

[← Previous: Discrete Random Variables](01-discrete-random-variables.md) | [Back to README](../README.md) | [Next: Cumulative Distribution Function (CDF) →](03-cdf.md)
