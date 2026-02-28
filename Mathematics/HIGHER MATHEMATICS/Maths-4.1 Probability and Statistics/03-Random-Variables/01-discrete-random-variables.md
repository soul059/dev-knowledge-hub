# Chapter 3.1: Discrete Random Variables

[← Previous: Applications in Probability](../02-Permutations-and-Combinations/04-applications-in-probability.md) | [Back to README](../README.md) | [Next: Probability Mass Function (PMF) →](02-pmf.md)

---

## Overview

A **random variable** bridges the gap between abstract probability theory and quantitative analysis. It assigns a **numerical value** to each outcome in the sample space, allowing us to use algebraic and calculus-based methods.

---

## 1. Definition

A **random variable** $X$ is a function that maps each outcome $\omega$ in the sample space $S$ to a real number:

$$X: S \to \mathbb{R}$$

$$\omega \mapsto X(\omega)$$

A random variable is **discrete** if it takes on a **countable** number of values (finite or countably infinite).

```
  Random Variable as a Function
  
  Sample Space S                Real Number Line ℝ
  ┌──────────────┐              ◄──────────────────►
  │  ω₁ ●───────────────────────────► X(ω₁) = x₁
  │  ω₂ ●───────────────────────────► X(ω₂) = x₂
  │  ω₃ ●───────────────────────────► X(ω₃) = x₃
  │  ω₄ ●───────────────────────────► X(ω₄) = x₁  (same value!)
  │  ω₅ ●───────────────────────────► X(ω₅) = x₄
  └──────────────┘
  
  Multiple outcomes can map to the same number.
  X is a FUNCTION from S to ℝ.
```

---

## 2. Examples

### Example 1: Coin Tosses (2 coins)

$S = \{HH, HT, TH, TT\}$

Define $X$ = "number of heads":

| Outcome $\omega$ | $X(\omega)$ |
|:-----------------:|:-----------:|
| TT | 0 |
| TH | 1 |
| HT | 1 |
| HH | 2 |

$X$ takes values $\{0, 1, 2\}$ → **discrete** random variable.

### Example 2: Sum of Two Dice

$X$ = sum of two dice values.

$X$ takes values $\{2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12\}$.

```
  Die 1 + Die 2 = X
  
  X:  2   3   4   5   6   7   8   9  10  11  12
      │   │   │   │   │   │   │   │   │   │   │
  #:  1   2   3   4   5   6   5   4   3   2   1
      
  Total outcomes = 36
```

### Example 3: Countably Infinite

$X$ = "number of tosses until first Head" (geometric setting).

$X$ takes values $\{1, 2, 3, 4, \ldots\}$ — countably infinite, still discrete.

---

## 3. Range (Support) of a Random Variable

The **range** (or **support**) of $X$ is the set of all possible values:

$$R_X = \{x \in \mathbb{R} : P(X = x) > 0\}$$

| Example | Support |
|---------|---------|
| Number of heads in 3 tosses | $\{0, 1, 2, 3\}$ |
| Roll of a die | $\{1, 2, 3, 4, 5, 6\}$ |
| Tosses until first head | $\{1, 2, 3, \ldots\}$ |
| Number of defects per item | $\{0, 1, 2, \ldots\}$ |

---

## 4. Events Defined by Random Variables

Once we define $X$, we can express events in terms of $X$:

| Event Expression | Meaning | Set in $S$ |
|:----------------:|---------|-----------|
| $\{X = k\}$ | $X$ equals $k$ | $\{\omega : X(\omega) = k\}$ |
| $\{X \leq k\}$ | $X$ is at most $k$ | $\{\omega : X(\omega) \leq k\}$ |
| $\{X > k\}$ | $X$ is greater than $k$ | $\{\omega : X(\omega) > k\}$ |
| $\{a < X \leq b\}$ | $X$ is between $a$ and $b$ | $\{\omega : a < X(\omega) \leq b\}$ |

### Example

For $X$ = number of heads in 2 coin tosses:
- $\{X = 1\} = \{HT, TH\}$ → $P(X = 1) = 2/4 = 1/2$
- $\{X \geq 1\} = \{HT, TH, HH\}$ → $P(X \geq 1) = 3/4$
- $\{X < 2\} = \{TT, HT, TH\}$ → $P(X < 2) = 3/4$

---

## 5. Functions of Random Variables

If $X$ is a random variable and $g$ is a function, then $Y = g(X)$ is also a random variable.

### Example 4

$X$ = a die roll. Define $Y = X^2$.

| $X = x$ | 1 | 2 | 3 | 4 | 5 | 6 |
|:--------:|:-:|:-:|:-:|:-:|:-:|:-:|
| $Y = x^2$ | 1 | 4 | 9 | 16 | 25 | 36 |
| $P(Y = y)$ | 1/6 | 1/6 | 1/6 | 1/6 | 1/6 | 1/6 |

### Example 5: Indicator Random Variable

An **indicator** (or Bernoulli) random variable for event $A$:

$$I_A(\omega) = \begin{cases} 1 & \text{if } \omega \in A \\ 0 & \text{if } \omega \notin A \end{cases}$$

$P(I_A = 1) = P(A)$, $P(I_A = 0) = 1 - P(A)$

---

## 6. Discrete vs. Continuous — Preview

| Feature | Discrete RV | Continuous RV |
|---------|:-----------:|:-------------:|
| Values | Countable | Uncountable (intervals) |
| Described by | PMF $P(X = x)$ | PDF $f(x)$ |
| $P(X = x)$ | Can be $> 0$ | Always $= 0$ |
| Sum/Integral | $\sum P(X = x) = 1$ | $\int f(x) dx = 1$ |
| Examples | Coins, dice, counts | Time, weight, temperature |

```
  Discrete RV                    Continuous RV
  
  P(X=x)                         f(x)
  │                               │
  │  ▮                            │      ╱╲
  │  ▮  ▮                        │    ╱    ╲
  │  ▮  ▮  ▮                     │  ╱        ╲
  │  ▮  ▮  ▮  ▮                  │╱            ╲
  └──┴──┴──┴──┴──► x             └──────────────► x
  
  Probability at points          Probability = area under curve
  (spikes/bars)                  (P at single point = 0)
```

---

## 7. Real-World Applications

| Domain | Discrete Random Variable |
|--------|-------------------------|
| **Quality Control** | Number of defective items in a batch |
| **Networking** | Number of packets received per second |
| **Insurance** | Number of claims filed per year |
| **Biology** | Number of mutations per gene |
| **Customer Service** | Number of customers in a queue |
| **Sports** | Number of goals scored per match |

---

## Summary Table

| Concept | Definition / Key Idea |
|---------|----------------------|
| Random Variable | Function $X: S \to \mathbb{R}$ |
| Discrete RV | Takes countable number of values |
| Support / Range | $R_X = \{x : P(X=x) > 0\}$ |
| Event notation | $\{X = k\}$, $\{X \leq k\}$, etc. |
| Indicator RV | $I_A = 1$ if $A$ occurs, 0 otherwise |
| Function of RV | $Y = g(X)$ is also an RV |
| Discrete examples | Counts, rolls, tosses, rankings |

---

## Quick Revision Questions

1. Define a random variable. Why do we call it a "variable" even though it's technically a function?

2. Two dice are rolled. Define $X$ = absolute difference of the two values. What is the support of $X$? List $P(X = k)$ for each $k$.

3. Define $X$ as the number of tails in 3 coin tosses. List all outcomes, their $X$ values, and compute $P(X = k)$ for all $k$.

4. Explain the difference between a discrete and continuous random variable with one example each from real life.

5. If $X$ is the roll of a fair die, and $Y = 2X - 1$, find the range of $Y$ and the probability distribution of $Y$.

6. What is an indicator random variable? If $A$ is the event "it rains today" with $P(A) = 0.3$, define $I_A$ and find $E[I_A]$.

---

[← Previous: Applications in Probability](../02-Permutations-and-Combinations/04-applications-in-probability.md) | [Back to README](../README.md) | [Next: Probability Mass Function (PMF) →](02-pmf.md)
