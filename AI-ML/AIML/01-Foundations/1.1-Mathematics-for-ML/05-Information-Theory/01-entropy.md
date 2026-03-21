# Chapter 1: Shannon Entropy

> **Unit 5 — Information Theory** | Mathematics for Machine Learning

---

## 1. Overview

**Shannon Entropy** is the foundational concept of information theory, introduced by Claude Shannon in 1948. It quantifies the **average amount of uncertainty** (or "surprise") in a random variable's outcomes. In ML, entropy drives decision tree splitting, feature selection, and underpins loss functions like cross-entropy.

---

## 2. Information Content of a Single Event

Before defining entropy, we define the **self-information** (or surprisal) of a single outcome x with probability p(x):

```
I(x) = -log₂ p(x)      (measured in bits)
I(x) = -ln  p(x)        (measured in nats)
```

**Intuition:** Rare events carry more information than common ones.

| Event probability p(x) | Self-information I(x) (bits) |
|------------------------|------------------------------|
| 1.0 (certain)          | 0.0                         |
| 0.5                    | 1.0                         |
| 0.25                   | 2.0                         |
| 0.125                  | 3.0                         |
| 0.01 (rare)            | 6.64                        |

---

## 3. Shannon Entropy — Definition

For a discrete random variable X with possible outcomes {x₁, x₂, …, xₙ} and probability mass function p(x):

```
              n
H(X) = - Σ  p(xᵢ) log₂ p(xᵢ)
             i=1
```

Entropy is the **expected value of self-information**: H(X) = E[I(X)] = E[-log p(X)]

### Convention
When p(xᵢ) = 0, we define 0 · log(0) = 0 (by continuity, since lim_{p→0⁺} p log p = 0).

---

## 4. Properties of Entropy

| Property               | Statement                                            |
|------------------------|------------------------------------------------------|
| **Non-negativity**     | H(X) ≥ 0, with equality iff X is deterministic      |
| **Maximum entropy**    | H(X) ≤ log₂(n), achieved when X ~ Uniform({1…n})    |
| **Concavity**          | H is a concave function of the distribution p        |
| **Additivity**         | For independent X, Y: H(X, Y) = H(X) + H(Y)        |
| **Conditioning reduces entropy** | H(X|Y) ≤ H(X)                            |

---

## 5. Binary Entropy Function

For a Bernoulli random variable X ∈ {0, 1} with P(X=1) = p:

```
H_b(p) = -p log₂(p) - (1-p) log₂(1-p)
```

### ASCII Plot of Binary Entropy H_b(p)

```
  H(p) (bits)
  1.0 |            * * * *
      |         *           *
  0.8 |       *               *
      |      *                 *
  0.6 |     *                   *
      |    *                     *
  0.4 |   *                       *
      |  *                         *
  0.2 | *                           *
      |*                             *
  0.0 +---+---+---+---+---+---+---+---→ p
      0  0.1 0.2 0.3 0.4 0.5 0.6 0.8 1.0

  Maximum at p = 0.5 → H_b(0.5) = 1.0 bit
  Minimum at p = 0 or p = 1 → H_b = 0 bits
```

**Key insight:** Maximum uncertainty occurs when both outcomes are equally likely.

---

## 6. Entropy for Common Distributions

### 6.1 Uniform Distribution (n outcomes)
```
H(X) = log₂(n)
```
| n (outcomes) | H(X) bits |
|-------------|-----------|
| 2           | 1.0       |
| 4           | 2.0       |
| 8           | 3.0       |
| 256         | 8.0       |

### 6.2 Geometric Distribution
For X ~ Geom(p):
```
H(X) = [ -(1-p) log₂(1-p) - p log₂(p) ] / p
```

### 6.3 Gaussian (Continuous — Differential Entropy)
For X ~ N(μ, σ²):
```
h(X) = ½ ln(2πeσ²)     (nats)
h(X) = ½ log₂(2πeσ²)   (bits)
```

---

## 7. Step-by-Step Examples

### Example 1: Fair Coin
X ∈ {H, T}, p(H) = p(T) = 0.5

```
H(X) = -0.5 log₂(0.5) - 0.5 log₂(0.5)
     = -0.5(-1) - 0.5(-1)
     = 0.5 + 0.5
     = 1.0 bit
```

### Example 2: Biased Coin
X ∈ {H, T}, p(H) = 0.9, p(T) = 0.1

```
H(X) = -0.9 log₂(0.9) - 0.1 log₂(0.1)
     = -0.9(-0.152) - 0.1(-3.322)
     = 0.137 + 0.332
     = 0.469 bits
```
Lower entropy → less uncertainty (we can predict heads most of the time).

### Example 3: Weather Forecast
X ∈ {Sunny, Rainy, Cloudy} with p = {0.7, 0.2, 0.1}

```
H(X) = -0.7 log₂(0.7) - 0.2 log₂(0.2) - 0.1 log₂(0.1)
     = -0.7(-0.515) - 0.2(-2.322) - 0.1(-3.322)
     = 0.360 + 0.464 + 0.332
     = 1.157 bits
```
Compare with maximum entropy: log₂(3) ≈ 1.585 bits (uniform over 3 outcomes).

### Example 4: Deterministic Variable
X always equals 5 → p(5) = 1.0

```
H(X) = -1.0 log₂(1.0) = -1.0 × 0 = 0 bits
```
No uncertainty at all.

---

## 8. Python Implementation

```python
import numpy as np

def entropy(probs, base=2):
    """Compute Shannon entropy of a discrete distribution."""
    probs = np.array(probs, dtype=float)
    probs = probs[probs > 0]  # Exclude zero probabilities
    return -np.sum(probs * np.log(probs)) / np.log(base)

# --- Examples ---
print(f"Fair coin:     H = {entropy([0.5, 0.5]):.4f} bits")
print(f"Biased coin:   H = {entropy([0.9, 0.1]):.4f} bits")
print(f"Weather:       H = {entropy([0.7, 0.2, 0.1]):.4f} bits")
print(f"Deterministic: H = {entropy([1.0]):.4f} bits")
print(f"Uniform(8):    H = {entropy([1/8]*8):.4f} bits")

# Binary entropy plot
import matplotlib.pyplot as plt

p_vals = np.linspace(0.001, 0.999, 500)
h_vals = -p_vals * np.log2(p_vals) - (1 - p_vals) * np.log2(1 - p_vals)

plt.plot(p_vals, h_vals)
plt.xlabel('p'); plt.ylabel('H(p) bits')
plt.title('Binary Entropy Function')
plt.grid(True); plt.show()
```

**Output:**
```
Fair coin:     H = 1.0000 bits
Biased coin:   H = 0.4690 bits
Weather:       H = 1.1568 bits
Deterministic: H = 0.0000 bits
Uniform(8):    H = 3.0000 bits
```

---

## 9. Connection to Decision Trees

Entropy is central to **ID3** and **C4.5** decision tree algorithms:

```
                  Dataset S
                 H(S) = 0.94
                /            \
         Feature A          Feature B
        /        \          /        \
   H(S₁)=0     H(S₂)=0.81   ...     ...

   IG(A) = H(S) - Σ (|Sᵢ|/|S|) H(Sᵢ)
```

- **High entropy** → impure node (mixed classes) → needs splitting
- **Low entropy** → pure node (one dominant class) → leaf node
- **Information Gain** = parent entropy − weighted child entropy
- The algorithm greedily picks the feature with the **highest information gain**

---

## 10. Real-World ML Applications

| Application                | How Entropy Is Used                                      |
|----------------------------|----------------------------------------------------------|
| **Decision Trees**         | Splitting criterion via information gain                 |
| **Random Forests**         | Entropy-based splits in each tree                        |
| **Cross-Entropy Loss**     | Derived from entropy; standard classification loss       |
| **Maximum Entropy Models** | Choose distribution with highest entropy given constraints|
| **Data Compression**       | Shannon's source coding theorem: avg bits ≥ H(X)        |
| **Anomaly Detection**      | High-surprise (low-probability) events flagged           |
| **NLP / Language Models**  | Perplexity = 2^H, measures model quality                 |

---

## 11. Summary Table

| Concept              | Formula / Key Fact                                     |
|----------------------|-------------------------------------------------------|
| Self-information     | I(x) = -log₂ p(x)                                    |
| Shannon Entropy      | H(X) = -Σ p(x) log₂ p(x)                            |
| Binary Entropy       | H_b(p) = -p log₂ p - (1-p) log₂(1-p)                |
| Min Entropy          | H(X) = 0 when deterministic                          |
| Max Entropy          | H(X) = log₂(n) for uniform over n outcomes            |
| Units                | bits (log₂), nats (ln), hartleys (log₁₀)             |
| Conditioning         | H(X|Y) ≤ H(X)                                        |
| Independence         | H(X,Y) = H(X) + H(Y) iff X ⊥ Y                      |

---

## 12. Quick Revision Questions

1. **What does entropy measure, and what are its units?**
   > Entropy measures the average uncertainty/surprise in a random variable. Units: bits (log₂), nats (ln).

2. **Why is the entropy of a fair coin flip exactly 1 bit?**
   > H = -2 × (0.5 × log₂ 0.5) = 1.0. One bit is the minimum information needed to encode one of two equally likely outcomes.

3. **What distribution maximizes entropy for n discrete outcomes, and what is the max value?**
   > The uniform distribution maximizes entropy at H = log₂(n).

4. **If p(Rain) = 0.01, what is the self-information of a "Rain" event?**
   > I(Rain) = -log₂(0.01) ≈ 6.64 bits — a rare event carries high information.

5. **How does entropy relate to decision tree construction?**
   > Decision trees split on the feature that reduces entropy the most (maximizes information gain), creating purer child nodes.

6. **Can entropy ever be negative for a discrete distribution? Why or why not?**
   > No. Each term p(x) log₂(1/p(x)) ≥ 0 since 0 ≤ p(x) ≤ 1, so the sum is non-negative.

---

[← Previous Chapter: 04-Mutual-Information](../05-Information-Theory/04-mutual-information.md) (Unit wrap-around) | [Next Chapter: 02-Cross-Entropy →](./02-cross-entropy.md)
