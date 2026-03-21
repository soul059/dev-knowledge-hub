# Chapter 5: Joint and Conditional Probability

> **Unit 3: Probability Theory** | **Topic**: Joint, Marginal, Conditional Distributions & Independence

---

## 1. Overview

Most machine learning problems involve **multiple** random variables — features, labels, latent
variables. Understanding how these variables relate to each other is captured by joint, marginal,
and conditional probabilities. These concepts form the backbone of probabilistic graphical models,
Bayesian inference, and generative modeling.

---

## 2. Joint Probability

The **joint probability** P(X = x, Y = y) gives the probability that X takes value x AND Y
takes value y simultaneously.

### 2.1 Joint PMF (Discrete)

```
p(x, y) = P(X = x, Y = y)

Properties:
  1. p(x, y) ≥ 0  for all x, y
  2. ΣₓΣᵧ p(x, y) = 1
```

### 2.2 Joint PDF (Continuous)

```
f(x, y) ≥ 0   and   ∫∫ f(x, y) dx dy = 1

P(a ≤ X ≤ b, c ≤ Y ≤ d) = ∫ₐᵇ ∫_c^d f(x,y) dy dx
```

### 2.3 Example: Joint Probability Table

Two dice are rolled. X = first die, Y = second die. Let A = "X is even", B = "Y ≤ 3".

| | Y=1 | Y=2 | Y=3 | Y=4 | Y=5 | Y=6 | **P(X=x)** |
|---|---|---|---|---|---|---|---|
| X=1 | 1/36 | 1/36 | 1/36 | 1/36 | 1/36 | 1/36 | **1/6** |
| X=2 | 1/36 | 1/36 | 1/36 | 1/36 | 1/36 | 1/36 | **1/6** |
| X=3 | 1/36 | 1/36 | 1/36 | 1/36 | 1/36 | 1/36 | **1/6** |
| X=4 | 1/36 | 1/36 | 1/36 | 1/36 | 1/36 | 1/36 | **1/6** |
| X=5 | 1/36 | 1/36 | 1/36 | 1/36 | 1/36 | 1/36 | **1/6** |
| X=6 | 1/36 | 1/36 | 1/36 | 1/36 | 1/36 | 1/36 | **1/6** |
| **P(Y=y)** | **1/6** | **1/6** | **1/6** | **1/6** | **1/6** | **1/6** | **1** |

---

## 3. Marginal Probability

The **marginal probability** is obtained by summing (or integrating) the joint over the other variable:

```
Discrete:    P(X = x) = Σ_y P(X = x, Y = y)
Continuous:  f_X(x)   = ∫ f(x, y) dy
```

This process is called **marginalization** — "marginalizing out" Y.

### Visual: Marginalization

```
Joint Distribution Table:
             Y=0    Y=1    │ P(X=x)
         ┌────────────────┼────────
  X=0    │  0.1    0.2    │  0.3      ← Sum across Y
  X=1    │  0.3    0.4    │  0.7      ← Sum across Y
         └────────────────┼────────
  P(Y=y) │  0.4    0.6    │  1.0
            ↑       ↑
         Sum down X
```

---

## 4. Conditional Probability

### 4.1 Definition

```
P(Y = y | X = x) = P(X = x, Y = y) / P(X = x)
```

"The probability of Y given we know X."

### 4.2 From the Table Above

```
P(Y=1 | X=0) = P(X=0, Y=1) / P(X=0)
              = 0.2 / 0.3
              = 2/3 ≈ 0.667
```

### 4.3 Conditional Distribution Table

Given X=0:

| Y | P(Y=y \| X=0) |
|---|----------------|
| 0 | 0.1/0.3 = 1/3  |
| 1 | 0.2/0.3 = 2/3  |
| **Sum** | **1** |

Given X=1:

| Y | P(Y=y \| X=1) |
|---|----------------|
| 0 | 0.3/0.7 = 3/7  |
| 1 | 0.4/0.7 = 4/7  |
| **Sum** | **1** |

---

## 5. The Product Rule and Chain Rule

### 5.1 Product Rule

```
P(X, Y) = P(Y|X) · P(X) = P(X|Y) · P(Y)
```

### 5.2 Chain Rule (Generalized)

For multiple variables:

```
P(X₁, X₂, ..., Xₙ) = P(X₁) · P(X₂|X₁) · P(X₃|X₁,X₂) · ... · P(Xₙ|X₁,...,Xₙ₋₁)
```

**ML Use**: Autoregressive language models decompose P(sentence) this way:
```
P(w₁, w₂, ..., wₙ) = P(w₁) · P(w₂|w₁) · P(w₃|w₁,w₂) · ...
```

---

## 6. Independence and Conditional Independence

### 6.1 Independence

X and Y are **independent** (X ⊥ Y) if:

```
P(X, Y) = P(X) · P(Y)       for all x, y
Equivalently: P(Y|X) = P(Y)
```

### 6.2 Conditional Independence

X and Y are **conditionally independent** given Z (X ⊥ Y | Z) if:

```
P(X, Y | Z) = P(X|Z) · P(Y|Z)
Equivalently: P(X | Y, Z) = P(X | Z)
```

> ⚠️ **Key insight**: Independence does NOT imply conditional independence, and vice versa!

### Example: Conditional Independence in Naive Bayes

```
Naive Bayes assumes: P(x₁, x₂, ..., xₙ | C) = ∏ᵢ P(xᵢ | C)

Features are conditionally independent GIVEN the class label C,
even if they are NOT marginally independent.

Example: "free" and "money" are correlated in emails (not independent),
but given class=spam, each word appears independently.
```

---

## 7. Probability Tree Diagrams

### Example: Medical Testing

```
Disease prevalence: P(D) = 0.01
Test sensitivity:   P(+|D) = 0.95
Test specificity:   P(-|D') = 0.90  →  P(+|D') = 0.10

                    ┌── P(+|D)  = 0.95 → P(D,+)  = 0.01 × 0.95 = 0.0095
        P(D)=0.01──┤
                    └── P(-|D)  = 0.05 → P(D,-)  = 0.01 × 0.05 = 0.0005
       ┤
                    ┌── P(+|D') = 0.10 → P(D',+) = 0.99 × 0.10 = 0.0990
        P(D')=0.99─┤
                    └── P(-|D') = 0.90 → P(D',-) = 0.99 × 0.90 = 0.8910
                                                              Total = 1.0000 ✓
```

### From the Tree:

```
P(+) = P(D,+) + P(D',+) = 0.0095 + 0.0990 = 0.1085   (marginal)
P(D|+) = P(D,+) / P(+) = 0.0095 / 0.1085 ≈ 0.0876    (Bayes' theorem preview!)
```

Only ~8.8% chance of actually having the disease even with a positive test!

---

## 8. Bayes' Rule Preview

From the product rule: P(X,Y) = P(Y|X)·P(X) = P(X|Y)·P(Y)

Rearranging:

```
P(Y|X) = P(X|Y) · P(Y) / P(X)
```

This is **Bayes' theorem** — covered in depth in Chapter 6.

---

## 9. Step-by-Step Worked Example

**Problem**: A factory has two machines. Machine A produces 60% of items, Machine B produces 40%.
Defect rates: A → 3%, B → 5%. An item is defective. What machine likely produced it?

```
Step 1: Define variables
  M ∈ {A, B}    (machine)
  D ∈ {0, 1}    (defective)

Step 2: Given information
  P(M=A) = 0.60,  P(M=B) = 0.40
  P(D=1|M=A) = 0.03,  P(D=1|M=B) = 0.05

Step 3: Joint probabilities
  P(M=A, D=1) = P(D=1|M=A) · P(M=A) = 0.03 × 0.60 = 0.018
  P(M=B, D=1) = P(D=1|M=B) · P(M=B) = 0.05 × 0.40 = 0.020

Step 4: Marginal P(D=1)
  P(D=1) = 0.018 + 0.020 = 0.038

Step 5: Conditional (Bayes')
  P(M=A | D=1) = 0.018 / 0.038 ≈ 0.474  (47.4%)
  P(M=B | D=1) = 0.020 / 0.038 ≈ 0.526  (52.6%)

Answer: Machine B is slightly more likely (52.6%) to have produced the defective item.
```

---

## 10. Python Code

```python
import numpy as np

# Joint probability table
joint = np.array([
    [0.1, 0.2],   # X=0: Y=0, Y=1
    [0.3, 0.4]    # X=1: Y=0, Y=1
])

# Marginals
p_x = joint.sum(axis=1)   # Sum over Y
p_y = joint.sum(axis=0)   # Sum over X

print("Joint distribution:")
print(joint)
print(f"\nMarginal P(X): {p_x}")
print(f"Marginal P(Y): {p_y}")

# Conditional P(Y|X=0)
p_y_given_x0 = joint[0] / p_x[0]
print(f"\nP(Y|X=0): {p_y_given_x0}")

# Test independence: P(X,Y) == P(X)·P(Y)?
outer_product = np.outer(p_x, p_y)
print(f"\nP(X)·P(Y):\n{outer_product}")
print(f"Independent? {np.allclose(joint, outer_product)}")
```

```python
# Factory example
p_A, p_B = 0.60, 0.40
p_D_A, p_D_B = 0.03, 0.05

p_D = p_D_A * p_A + p_D_B * p_B  # Total probability
p_A_D = (p_D_A * p_A) / p_D      # Bayes' theorem
p_B_D = (p_D_B * p_B) / p_D

print(f"P(Defective) = {p_D:.4f}")
print(f"P(Machine A | Defective) = {p_A_D:.4f}")
print(f"P(Machine B | Defective) = {p_B_D:.4f}")
```

---

## 11. Key Relationships Diagram

```
┌─────────────────────────────────────────────────────────┐
│                                                         │
│   Joint P(X,Y)                                          │
│       │                                                 │
│       ├── Marginalize over Y ──→ P(X)                   │
│       ├── Marginalize over X ──→ P(Y)                   │
│       │                                                 │
│       ├── Divide by P(X) ──→ P(Y|X)  (Conditional)     │
│       ├── Divide by P(Y) ──→ P(X|Y)  (Conditional)     │
│       │                                                 │
│       └── If P(X,Y) = P(X)·P(Y) → Independent         │
│                                                         │
│   Chain Rule: P(X,Y,Z) = P(X)·P(Y|X)·P(Z|X,Y)        │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 12. Real-World ML Applications

| Concept                  | ML Application                                        |
|--------------------------|-------------------------------------------------------|
| Joint distribution       | Generative models learn P(X, Y) to generate data     |
| Marginal distribution    | Computing P(Y) by marginalizing over latent Z         |
| Conditional distribution | P(Y\|X) is the core of discriminative classifiers     |
| Chain rule               | Autoregressive models (GPT) decompose sequence prob.  |
| Conditional independence | Naive Bayes, graphical models, d-separation           |
| Marginalization          | EM algorithm, variational inference                   |

---

## 13. Summary Table

| Concept                | Formula                                    | Key Point                        |
|------------------------|--------------------------------------------|----------------------------------|
| Joint prob.            | P(X=x, Y=y)                              | Probability of both events       |
| Marginal prob.         | P(X=x) = Σ_y P(X=x, Y=y)                | Sum out the other variable       |
| Conditional prob.      | P(Y\|X) = P(X,Y) / P(X)                  | Prob of Y knowing X              |
| Product rule           | P(X,Y) = P(Y\|X)·P(X)                    | Joint from conditional           |
| Chain rule             | P(X₁,...,Xₙ) = ∏ P(Xᵢ\|X₁,...,Xᵢ₋₁)    | Decompose any joint              |
| Independence           | P(X,Y) = P(X)·P(Y)                       | Knowing X tells nothing about Y  |
| Cond. independence     | P(X,Y\|Z) = P(X\|Z)·P(Y\|Z)             | Independent once Z is known      |

---

## 14. Quick Revision Questions

1. **Given a joint table, compute P(X=1) and P(Y=0\|X=1).** Show your marginalization.
2. **What is the difference between independence and conditional independence?** Give an example.
3. **Write the chain rule for P(A, B, C, D)** in two different orderings.
4. **In a Naive Bayes classifier, what conditional independence assumption is made?**
5. **Why is P(Disease\|Positive Test) often much lower than P(Positive Test\|Disease)?**
6. **How does marginalization relate to the EM algorithm?**

---

## 15. Navigation

| ← Previous | Next → |
|:-----------:|:------:|
| [Chapter 4: Expectation & Variance](04-expectation-and-variance.md) | [Chapter 6: Bayes' Theorem](06-bayes-theorem.md) |

---

*Unit 3 · Probability Theory · Chapter 5 of 9*
