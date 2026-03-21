# 📘 Chapter 2: The Sigmoid Function

> **Unit 5: Logistic Regression** | **Section 2 of 7**
> Deep dive into the sigmoid (logistic) function — the mathematical heart of logistic regression. Understand its properties, behavior, derivative, numerical stability, and how it transforms linear outputs into probabilities.

---

## 📑 Table of Contents

1. [Introduction: Why We Need the Sigmoid](#1-introduction-why-we-need-the-sigmoid)
2. [The Sigmoid Function: Definition & Formula](#2-the-sigmoid-function-definition--formula)
3. [Key Properties of the Sigmoid](#3-key-properties-of-the-sigmoid)
4. [ASCII Plot of the Sigmoid](#4-ascii-plot-of-the-sigmoid)
5. [The Sigmoid Derivative](#5-the-sigmoid-derivative)
6. [Interpreting Sigmoid Output as Probability](#6-interpreting-sigmoid-output-as-probability)
7. [Sigmoid Saturation & Vanishing Gradients](#7-sigmoid-saturation--vanishing-gradients)
8. [Numerical Stability](#8-numerical-stability)
9. [Comparison with Other Activation Functions](#9-comparison-with-other-activation-functions)
10. [Python Implementation](#10-python-implementation)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)

---

## 1. Introduction: Why We Need the Sigmoid

Recall from [Chapter 1](01-classification-problem.md): linear regression outputs any real number, but we need probabilities in [0, 1].

We need a function g(z) that satisfies:
1. **Domain:** z ∈ (-∞, +∞) — accept any real input
2. **Range:** g(z) ∈ (0, 1) — output valid probabilities
3. **Monotonic:** larger z → larger g(z)
4. **Smooth:** differentiable everywhere (for gradient descent)
5. **Centered:** g(0) = 0.5 (natural threshold)

```
The Pipeline:
    
    Features        Linear         Sigmoid          Probability
    ┌───┐        Combination      Function          Output
    │ x │  ───→  z = wᵀx + b  ───→  σ(z)  ───→   P(y=1|x)
    └───┘        
    ℝⁿ            ℝ              (0, 1)            [0, 1]
                 any real        squashed!          probability
```

---

## 2. The Sigmoid Function: Definition & Formula

### The Formula

```
                    1
    σ(z) = ─────────────────
            1 + e^(-z)
```

Or equivalently:

```
                e^z
    σ(z) = ─────────────
            1 + e^z
```

Where:
- **z** = wᵀx + b (the linear combination, also called the **logit** or **log-odds**)
- **e** ≈ 2.71828 (Euler's number)
- **σ** (sigma) denotes the sigmoid function

### Breaking Down the Formula

```
Step by step for z = 2:

1. Compute -z:           -z = -2
2. Compute e^(-z):       e^(-2) = 0.1353
3. Compute 1 + e^(-z):   1 + 0.1353 = 1.1353
4. Take reciprocal:      1 / 1.1353 = 0.8808

    σ(2) = 1 / (1 + e^(-2)) = 1 / 1.1353 ≈ 0.8808
```

### Sample Values

| z | e^(-z) | 1 + e^(-z) | σ(z) |
|---|--------|------------|------|
| -5 | 148.41 | 149.41 | 0.0067 |
| -3 | 20.09 | 21.09 | 0.0474 |
| -2 | 7.389 | 8.389 | 0.1192 |
| -1 | 2.718 | 3.718 | 0.2689 |
| 0 | 1.000 | 2.000 | **0.5000** |
| 1 | 0.368 | 1.368 | 0.7311 |
| 2 | 0.135 | 1.135 | 0.8808 |
| 3 | 0.050 | 1.050 | 0.9526 |
| 5 | 0.007 | 1.007 | 0.9933 |

---

## 3. Key Properties of the Sigmoid

### Property 1: Range (0, 1)

```
    lim σ(z) = 0     (as z → -∞)
    z→-∞

    lim σ(z) = 1     (as z → +∞)
    z→+∞

    σ(0) = 0.5       (midpoint)

    ∀z ∈ ℝ:  0 < σ(z) < 1    (strictly between 0 and 1, never equals)
```

### Property 2: Symmetry

The sigmoid has a beautiful **point symmetry** around (0, 0.5):

```
    σ(-z) = 1 - σ(z)
    
    Proof:
    σ(-z) = 1 / (1 + e^z) 
          = 1 - e^z / (1 + e^z)
          = 1 - 1 / (1 + e^(-z))
          = 1 - σ(z)  ✓
    
    Example: σ(2) = 0.8808
             σ(-2) = 0.1192
             0.8808 + 0.1192 = 1.0  ✓
```

### Property 3: Monotonically Increasing

```
    dσ/dz > 0 for all z ∈ ℝ
    
    Larger z → larger σ(z)
    The function NEVER decreases.
```

### Property 4: S-Shaped (Sigmoidal) Curve

```
    The function has an "S" shape:
    - Flat near 0 for very negative z
    - Steep around z = 0
    - Flat near 1 for very positive z
```

### Property 5: Smooth and Differentiable

```
    σ(z) is infinitely differentiable (C∞)
    No corners, jumps, or discontinuities
    Perfect for gradient-based optimization!
```

### Property 6: Elegant Derivative

```
    dσ/dz = σ(z) · (1 - σ(z))
    
    The derivative is expressible in terms of the function itself!
    This makes computation extremely efficient.
```

---

## 4. ASCII Plot of the Sigmoid

### Standard Sigmoid σ(z)

```
    σ(z)
    1.0 ┤─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ── ─ ─ ─ ─ ─
        │                                       ·····■■■■■■■■■■■
        │                                  ····
    0.8 ┤                               ···
        │                             ··
        │                           ··
    0.6 ┤                         ··
        │                        ·
    0.5 ┤─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─◆─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
        │                      ·
    0.4 ┤                     ·
        │                   ··
        │                 ··
    0.2 ┤               ··
        │            ···
        │       ····
    0.0 ┤■■■■■■■····─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
        └───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───┬───
           -6  -5  -4  -3  -2  -1   0   1   2   3   4   5   6
                                     z

    Key observations:
    • σ(0) = 0.5 exactly (marked with ◆)
    • Nearly 0 for z < -5
    • Nearly 1 for z > 5
    • Steepest slope at z = 0
    • The "transition zone" is roughly z ∈ [-4, 4]
```

### Sigmoid with Different Scaling: σ(k·z)

```
    σ(z)
    1.0 ┤                          ····──── k=0.5 (gentle slope)
        │                     ·····
        │                ·····         ┌── k=1 (standard)
        │           ·····             ╱
    0.8 ┤       ····            ·····
        │     ···          ·····
        │    ··        ····
    0.6 ┤   ··       ···        ╱── k=5 (steep, near step function)
        │  ·       ··      ┌──╱
    0.5 ┤─ ─ ─ ─ ◆ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
        │       ··       ··
    0.4 ┤      ·       ··
        │     ··      ···
        │    ··     ····
    0.2 ┤   ··   ·····
        │  ··  ·····
        │    ·····
    0.0 ┤ ────····
        └───┬───┬───┬───┬───┬───┬───┬───
           -4  -3  -2  -1   0   1   2   3
    
    As k increases, sigmoid approaches the step function.
    As k decreases, sigmoid becomes more linear (around z=0).
```

---

## 5. The Sigmoid Derivative

### Derivation

```
    Given: σ(z) = 1 / (1 + e^(-z)) = (1 + e^(-z))^(-1)

    Using the chain rule:
    
    dσ/dz = -1 · (1 + e^(-z))^(-2) · (-e^(-z))
    
          = e^(-z) / (1 + e^(-z))²
    
          = [1 / (1 + e^(-z))] · [e^(-z) / (1 + e^(-z))]
    
          = σ(z) · [e^(-z) / (1 + e^(-z))]
    
    Now note:  e^(-z) / (1 + e^(-z)) = 1 - 1/(1 + e^(-z)) = 1 - σ(z)
    
    Therefore:
    ┌─────────────────────────────────────┐
    │  dσ/dz = σ(z) · (1 - σ(z))        │
    │                                     │
    │  or equivalently:                   │
    │  σ'(z) = σ(z) · σ(-z)             │
    └─────────────────────────────────────┘
```

### Derivative Values

| z | σ(z) | 1 - σ(z) | σ'(z) = σ(z)·(1-σ(z)) |
|---|------|----------|------------------------|
| -3 | 0.0474 | 0.9526 | 0.0452 |
| -2 | 0.1192 | 0.8808 | 0.1050 |
| -1 | 0.2689 | 0.7311 | 0.1966 |
| **0** | **0.5** | **0.5** | **0.25 (maximum!)** |
| 1 | 0.7311 | 0.2689 | 0.1966 |
| 2 | 0.8808 | 0.1192 | 0.1050 |
| 3 | 0.9526 | 0.0474 | 0.0452 |

### ASCII Plot of the Derivative

```
    σ'(z)
    0.25 ┤           ■■■
         │          ■   ■
         │         ■     ■
    0.20 ┤        ■       ■
         │       ■         ■
         │      ■           ■
    0.15 ┤     ■             ■
         │    ■               ■
    0.10 ┤   ■                 ■
         │  ■                   ■
    0.05 ┤ ■                     ■
         │■                       ■
    0.00 ┤■■                       ■■
         └───┬───┬───┬───┬───┬───┬───┬──
            -4  -3  -2  -1   0   1   2   3
    
    • Maximum derivative = 0.25 at z = 0
    • Bell-shaped curve (related to the logistic distribution PDF)
    • Approaches 0 for |z| > 4
```

### Why the Derivative Matters

The derivative σ'(z) appears directly in the **gradient descent update rule** for logistic regression. The fact that it equals σ(z)·(1-σ(z)) makes gradient computation extremely efficient — you compute σ(z) once, then the derivative is trivially obtained.

---

## 6. Interpreting Sigmoid Output as Probability

### The Probabilistic Model

Logistic regression models the **conditional probability**:

```
    P(y = 1 | x; w, b) = σ(wᵀx + b)
    P(y = 0 | x; w, b) = 1 - σ(wᵀx + b)
    
    These always sum to 1 ✓ (valid probability distribution)
```

### Compact Notation

Both cases can be written in a single expression:

```
    P(y | x; w, b) = σ(z)^y · (1 - σ(z))^(1-y)
    
    When y = 1:  P = σ(z)^1 · (1-σ(z))^0 = σ(z)       ✓
    When y = 0:  P = σ(z)^0 · (1-σ(z))^1 = 1 - σ(z)   ✓
```

### The Log-Odds (Logit) Interpretation

```
    σ(z) = P(y=1|x)
    
    The "odds" of y=1:
    odds = P(y=1) / P(y=0) = σ(z) / (1-σ(z))
    
    log(odds) = log(σ(z) / (1-σ(z)))
              = log(σ(z)) - log(1-σ(z))
              = z = wᵀx + b
    
    ┌─────────────────────────────────────────────────┐
    │  The linear combination z = wᵀx + b is the     │
    │  LOG-ODDS (logit) of the positive class!        │
    │                                                  │
    │  This is why logistic regression is also called  │
    │  "logit regression."                             │
    └─────────────────────────────────────────────────┘
```

### Interpreting the Weights

```
    z = w₁x₁ + w₂x₂ + ... + wₙxₙ + b
    
    A unit increase in xⱼ increases the log-odds by wⱼ:
    
    Δ(log-odds) = wⱼ · Δxⱼ
    
    This means the ODDS are multiplied by e^(wⱼ):
    
    odds_new = odds_old · e^(wⱼ)
    
    Example: w₁ = 0.7 for "word_free" in spam detection
    → Each additional occurrence of "free" multiplies
      the odds of spam by e^0.7 ≈ 2.01 (doubles the odds!)
```

---

## 7. Sigmoid Saturation & Vanishing Gradients

### The Saturation Problem

When |z| is large, the sigmoid "saturates" — it's nearly flat:

```
    Saturation Zones:
    
    σ(z)
    1.0 ┤████████████████                    ████████████████
        │  SATURATED   ████████         █████   SATURATED
        │  (flat)           ████████████       (flat)
    0.5 ┤
        │
    0.0 ┤
        └───────────────────────────────────────────────────
       -10  -8  -6  -4  -2   0   2   4   6   8  10
    
        ◄──── σ'(z) ≈ 0 ────►   ◄──── σ'(z) ≈ 0 ────►
             gradient vanishes!       gradient vanishes!
    
    Only the "active zone" (roughly z ∈ [-4, 4]) has meaningful gradients.
```

### Why This Matters

In deep neural networks (not just logistic regression), sigmoid saturation causes the **vanishing gradient problem**:

```
    When z is very large or very small:
    
    σ'(z) ≈ 0
    
    → Gradient updates become tiny
    → Learning slows to a crawl
    → Weights stop updating effectively
    
    This is why modern deep learning prefers ReLU over sigmoid
    for hidden layers (but sigmoid is still used for the output
    layer of binary classifiers).
```

### Impact on Logistic Regression

For logistic regression specifically, saturation is **less of a problem** because:
1. We only have one "layer" — no compounding of small gradients
2. The convex cost function still has a unique minimum
3. Feature scaling keeps z values in a reasonable range

---

## 8. Numerical Stability

### The Problem

For very large negative z, `e^(-z)` becomes astronomically large, causing **overflow**:

```python
# UNSTABLE: Direct computation
import numpy as np

z = -1000
# result = 1 / (1 + np.exp(-z))  # np.exp(1000) = inf → OVERFLOW!
```

For very large positive z, `e^(-z)` underflows to 0 (which is actually fine: σ(z) ≈ 1).

### The Numerically Stable Implementation

```
    For z ≥ 0:
        σ(z) = 1 / (1 + e^(-z))          ← standard formula (safe: e^(-z) small)
    
    For z < 0:
        σ(z) = e^z / (1 + e^z)           ← equivalent formula (safe: e^z small)
    
    Combined (using the log-sum-exp trick):
        σ(z) = 1 / (1 + e^(-|z|))  if z ≥ 0
             = e^z / (1 + e^z)      if z < 0
```

```python
def sigmoid_stable(z):
    """Numerically stable sigmoid function."""
    return np.where(
        z >= 0,
        1 / (1 + np.exp(-z)),       # safe for z >= 0
        np.exp(z) / (1 + np.exp(z)) # safe for z < 0
    )
```

### Even Simpler: Use scipy or clip

```python
from scipy.special import expit   # numerically stable sigmoid

# Or simply clip the input
def sigmoid_clipped(z):
    z = np.clip(z, -500, 500)  # prevent overflow
    return 1 / (1 + np.exp(-z))
```

---

## 9. Comparison with Other Activation Functions

### Sigmoid vs Step Function

```
    Step Function (Heaviside):      Sigmoid:
    
    y                               y
    1 ┤       ■■■■■■■■■             1 ┤              ····■■■■
      │       │                       │           ···
      │       │                     0.5┤─ ─ ─ ─ ◆ ─ ─ ─ ─ ─
      │       │                       │       ···
    0 ┤■■■■■■■                      0 ┤  ■■■····
      └───────┼───── z                └─────────┼───── z
              0                                 0
    
    ✗ Not differentiable at z=0     ✓ Smooth everywhere
    ✗ No gradient signal            ✓ Gradient for optimization
    ✗ No probability output         ✓ Output ∈ (0, 1)
```

### Sigmoid vs Tanh

```
    Sigmoid σ(z):                   Tanh:
    Range: (0, 1)                   Range: (-1, 1)
    
    y                               y
    1.0 ┤        ····■■■            1.0 ┤        ····■■■
        │     ···                       │     ···
    0.5 ┤─ ─◆─ ─ ─ ─ ─             0.0 ┤─ ─◆─ ─ ─ ─ ─
        │  ···                          │  ···
    0.0 ┤■■····                    -1.0 ┤■■····
        └────────── z                   └────────── z
    
    Relationship: tanh(z) = 2σ(2z) - 1
    
    tanh is "zero-centered" → better for hidden layers
    sigmoid is "probability-ready" → better for output layer
```

### Sigmoid vs ReLU

```
    Sigmoid:                        ReLU:
    
    y                               y
    1.0 ┤        ····■■■                │           ╱
        │     ···                       │         ╱
    0.5 ┤─ ─◆─ ─ ─ ─ ─                │       ╱
        │  ···                          │     ╱
    0.0 ┤■■····                     0.0 ┤■■■■■
        └────────── z                   └─────────── z
    
    Sigmoid:                        ReLU:
    ✓ Bounded output (0,1)          ✗ Unbounded (0, ∞)
    ✗ Saturates at extremes         ✓ No saturation for z > 0
    ✗ Vanishing gradients           ✓ Constant gradient for z > 0
    ✓ Probabilistic interpretation  ✗ Not a probability
    ✓ Great for output layers       ✓ Great for hidden layers
```

### Comparison Summary Table

| Property | Sigmoid | Step | Tanh | ReLU |
|----------|---------|------|------|------|
| **Formula** | 1/(1+e⁻ᶻ) | {0 if z<0, 1 if z≥0} | (eᶻ-e⁻ᶻ)/(eᶻ+e⁻ᶻ) | max(0, z) |
| **Range** | (0, 1) | {0, 1} | (-1, 1) | [0, ∞) |
| **Zero-centered** | No | No | **Yes** | No |
| **Differentiable** | **Yes** | No (at z=0) | **Yes** | No (at z=0) |
| **Saturates?** | Yes | N/A | Yes | No (for z>0) |
| **Vanishing gradient** | Yes | N/A | Less so | No |
| **Probabilistic** | **Yes** | No | No | No |
| **Use case** | Output layer | Perceptron | Hidden layers | Hidden layers |

---

## 10. Python Implementation {#10-python-implementation}

### Basic Sigmoid Implementation

```python
import numpy as np

# ─── Basic sigmoid ───
def sigmoid(z):
    """Standard sigmoid function."""
    return 1 / (1 + np.exp(-z))

# ─── Numerically stable sigmoid ───
def sigmoid_stable(z):
    """Numerically stable sigmoid for extreme values."""
    z = np.asarray(z, dtype=np.float64)
    return np.where(
        z >= 0,
        1 / (1 + np.exp(-z)),
        np.exp(z) / (1 + np.exp(z))
    )

# ─── Sigmoid derivative ───
def sigmoid_derivative(z):
    """Derivative of the sigmoid function: σ'(z) = σ(z)(1 - σ(z))."""
    s = sigmoid_stable(z)
    return s * (1 - s)

# Test values
test_values = [-5, -3, -1, 0, 1, 3, 5]
print("z\t\tσ(z)\t\tσ'(z)")
print("-" * 45)
for z in test_values:
    print(f"{z:+d}\t\t{sigmoid(z):.6f}\t{sigmoid_derivative(z):.6f}")
```

**Expected Output:**
```
z		σ(z)		σ'(z)
---------------------------------------------
-5		0.006693	0.006648
-3		0.047426	0.045177
-1		0.268941	0.196612
+0		0.500000	0.250000
+1		0.731059	0.196612
+3		0.952574	0.045177
+5		0.993307	0.006648
```

### Verifying Sigmoid Properties

```python
# ─── Property verification ───

# 1. Symmetry: σ(-z) = 1 - σ(z)
z = 3.0
assert np.isclose(sigmoid(-z), 1 - sigmoid(z)), "Symmetry failed!"
print(f"σ({z}) + σ({-z}) = {sigmoid(z) + sigmoid(-z):.10f}")  # Should be 1.0

# 2. σ(0) = 0.5
assert sigmoid(0) == 0.5, "Midpoint failed!"
print(f"σ(0) = {sigmoid(0)}")

# 3. Range: always in (0, 1)
z_extreme = np.array([-100, -10, 0, 10, 100])
s = sigmoid_stable(z_extreme)
assert np.all((s > 0) & (s < 1)), "Range violated!"
print(f"Extreme values: {s}")

# 4. Monotonically increasing
z_sorted = np.linspace(-10, 10, 1000)
s_sorted = sigmoid_stable(z_sorted)
assert np.all(np.diff(s_sorted) > 0), "Not monotonic!"
print("Monotonicity: verified ✓")

# 5. Derivative = σ(z) * (1 - σ(z))
z_test = 2.5
numerical_deriv = (sigmoid(z_test + 1e-7) - sigmoid(z_test - 1e-7)) / (2e-7)
analytical_deriv = sigmoid_derivative(z_test)
print(f"Numerical derivative:  {numerical_deriv:.8f}")
print(f"Analytical derivative: {analytical_deriv:.8f}")
print(f"Match: {np.isclose(numerical_deriv, analytical_deriv, atol=1e-6)}")
```

### Numerical Stability Demonstration

```python
# ─── Numerical stability comparison ───

def sigmoid_naive(z):
    """UNSTABLE version — do NOT use in production."""
    return 1 / (1 + np.exp(-z))

# Test with extreme values
extreme_z = np.array([-1000, -500, -100, 0, 100, 500, 1000])

print("Naive sigmoid (may produce warnings):")
import warnings
with warnings.catch_warnings():
    warnings.simplefilter("ignore")
    for z in extreme_z:
        try:
            result = sigmoid_naive(z)
            print(f"  σ({z:+5d}) = {result}")
        except Exception as e:
            print(f"  σ({z:+5d}) = ERROR: {e}")

print("\nStable sigmoid (always works):")
for z in extreme_z:
    result = sigmoid_stable(z)
    print(f"  σ({z:+5d}) = {result}")

# Using scipy (always stable)
from scipy.special import expit
print("\nScipy expit (always stable):")
print(f"  {expit(extreme_z)}")
```

### Visualizing Sigmoid Behavior with ASCII

```python
# ─── Text-based sigmoid plot ───

def ascii_plot_sigmoid(z_min=-6, z_max=6, width=60, height=20):
    """Print an ASCII sigmoid plot."""
    z_values = np.linspace(z_min, z_max, width)
    s_values = sigmoid_stable(z_values)
    
    print(f"\n  Sigmoid Function σ(z) = 1/(1+e^(-z))")
    print(f"  {'─' * (width + 6)}")
    
    for row in range(height, -1, -1):
        y_val = row / height
        line = f"  {y_val:4.2f} │"
        for col in range(width):
            if abs(s_values[col] - y_val) < 0.5 / height:
                line += "●"
            elif abs(y_val - 0.5) < 0.01:
                line += "─"
            else:
                line += " "
        print(line)
    
    # X-axis
    print(f"       └{'─' * width}")
    labels = f"       {z_min}"
    labels += " " * (width // 2 - 4) + "0"
    labels += " " * (width // 2 - 2) + f"{z_max}"
    print(labels)

ascii_plot_sigmoid()
```

### Using Sigmoid in Logistic Regression Prediction

```python
# ─── Putting it all together: Sigmoid for binary classification ───
from sklearn.datasets import make_classification
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split

# Generate data
X, y = make_classification(n_samples=500, n_features=2, n_informative=2,
                           n_redundant=0, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# Train sklearn logistic regression
model = LogisticRegression()
model.fit(X_train, y_train)

# Manual prediction using sigmoid
w = model.coef_[0]      # weights
b = model.intercept_[0] # bias

sample = X_test[0]
z = np.dot(w, sample) + b           # linear combination
prob = sigmoid_stable(z)             # sigmoid → probability
predicted_class = int(prob >= 0.5)   # threshold

# Compare with sklearn
sklearn_prob = model.predict_proba(X_test[0:1])[0, 1]
sklearn_class = model.predict(X_test[0:1])[0]

print(f"Manual computation:")
print(f"  z = w·x + b = {z:.6f}")
print(f"  σ(z) = {prob:.6f}")
print(f"  Predicted class: {predicted_class}")
print(f"\nsklearn computation:")
print(f"  P(y=1|x) = {sklearn_prob:.6f}")
print(f"  Predicted class: {sklearn_class}")
print(f"\nMatch: {np.isclose(prob, sklearn_prob)}")
```

### Comparison Plot: Sigmoid vs Tanh vs Step

```python
# ─── Comparison functions ───

def step_function(z):
    """Heaviside step function."""
    return np.where(z >= 0, 1.0, 0.0)

def tanh_fn(z):
    """Hyperbolic tangent."""
    return np.tanh(z)

def relu(z):
    """Rectified Linear Unit."""
    return np.maximum(0, z)

z = np.linspace(-5, 5, 1000)

print("Value comparison at key points:")
print(f"{'z':>4} {'Step':>8} {'Sigmoid':>8} {'Tanh':>8} {'ReLU':>8}")
print("-" * 40)
for z_val in [-3, -1, 0, 1, 3]:
    print(f"{z_val:>4d} {step_function(z_val):>8.4f} {sigmoid(z_val):>8.4f} "
          f"{tanh_fn(z_val):>8.4f} {relu(z_val):>8.4f}")

# Verify: tanh(z) = 2σ(2z) - 1
z_test = 1.5
tanh_direct = np.tanh(z_test)
tanh_from_sigmoid = 2 * sigmoid(2 * z_test) - 1
print(f"\nVerify tanh(z) = 2σ(2z) - 1:")
print(f"  tanh({z_test}) = {tanh_direct:.8f}")
print(f"  2σ(2·{z_test})-1 = {tanh_from_sigmoid:.8f}")
print(f"  Match: {np.isclose(tanh_direct, tanh_from_sigmoid)}")
```

---

## 11. Summary Table

| Concept | Formula / Value | Key Insight |
|---------|----------------|-------------|
| **Sigmoid function** | σ(z) = 1/(1+e⁻ᶻ) | Maps ℝ → (0, 1) |
| **σ(0)** | 0.5 | Natural threshold |
| **σ(z) as z→∞** | → 1 | Saturates high |
| **σ(z) as z→-∞** | → 0 | Saturates low |
| **Symmetry** | σ(-z) = 1 - σ(z) | Point symmetry around (0, 0.5) |
| **Derivative** | σ'(z) = σ(z)·(1-σ(z)) | Max at z=0: σ'(0) = 0.25 |
| **Log-odds** | z = log(P/(1-P)) = wᵀx + b | Linear part IS the logit |
| **Weight interpretation** | wⱼ = Δ(log-odds) per unit xⱼ | e^(wⱼ) = odds ratio |
| **Saturation** | σ'(z) ≈ 0 for \|z\| > 5 | Vanishing gradient zone |
| **Numerical stability** | Use piecewise formula | Avoid overflow with e^(-z) |
| **vs Tanh** | tanh(z) = 2σ(2z) - 1 | Tanh is zero-centered |
| **vs Step** | Sigmoid is smooth approximation | Differentiable everywhere |

---

## 12. Revision Questions

### Q1: Compute Sigmoid
**Q:** Compute σ(0), σ(ln 2), and σ(-ln 2) by hand. Verify the symmetry property.

**A:**
- σ(0) = 1/(1+e⁰) = 1/2 = **0.5**
- σ(ln 2) = 1/(1+e^(-ln 2)) = 1/(1+1/2) = 1/(3/2) = **2/3 ≈ 0.6667**
- σ(-ln 2) = 1/(1+e^(ln 2)) = 1/(1+2) = **1/3 ≈ 0.3333**
- Symmetry: σ(ln 2) + σ(-ln 2) = 2/3 + 1/3 = **1** ✓

---

### Q2: Derivative Proof
**Q:** Starting from σ(z) = 1/(1+e⁻ᶻ), derive that σ'(z) = σ(z)·(1-σ(z)). At what value of z is the derivative maximum, and what is that maximum value?

**A:** Using the chain rule: σ'(z) = e⁻ᶻ/(1+e⁻ᶻ)² = [1/(1+e⁻ᶻ)]·[e⁻ᶻ/(1+e⁻ᶻ)] = σ(z)·(1-σ(z)). The derivative σ'(z) = σ·(1-σ) is maximized when σ = 0.5, which occurs at **z = 0**. The maximum value is 0.5 × 0.5 = **0.25**.

---

### Q3: Log-Odds Interpretation
**Q:** A logistic regression model for spam detection has weight w = 0.8 for the feature "count of word 'free'". If an email currently has log-odds of being spam equal to 1.2, what happens when one more occurrence of "free" is added? Express the change in both log-odds and probability.

**A:**
- Original log-odds: 1.2 → P(spam) = σ(1.2) = 1/(1+e⁻¹·²) ≈ 0.7685
- New log-odds: 1.2 + 0.8 = 2.0 → P(spam) = σ(2.0) = 1/(1+e⁻²) ≈ 0.8808
- Log-odds increased by **0.8**; probability increased from **76.85% to 88.08%**
- Odds multiplied by e^0.8 ≈ 2.23

---

### Q4: Numerical Stability
**Q:** Why does computing `1/(1 + np.exp(-z))` directly fail for z = -1000? Write the numerically stable alternative and explain why it works.

**A:** For z = -1000, we compute e^(-(-1000)) = e^1000, which exceeds float64 max (~1.8×10³⁰⁸), causing **overflow to infinity**. Result: 1/(1+inf) = 0, which is actually correct by luck but unreliable. The stable alternative for z < 0 is `e^z / (1 + e^z)`, which computes e^(-1000) ≈ 0 (safe underflow), giving 0/(1+0) = 0.

---

### Q5: Sigmoid vs Tanh
**Q:** Prove that tanh(z) = 2σ(2z) - 1. Why might tanh be preferred over sigmoid in certain contexts?

**A:**
- tanh(z) = (eᶻ - e⁻ᶻ)/(eᶻ + e⁻ᶻ)
- 2σ(2z) - 1 = 2/(1+e⁻²ᶻ) - 1 = (2 - 1 - e⁻²ᶻ)/(1+e⁻²ᶻ) = (1 - e⁻²ᶻ)/(1+e⁻²ᶻ)
- Multiply numerator and denominator by eᶻ: (eᶻ - e⁻ᶻ)/(eᶻ + e⁻ᶻ) = tanh(z) ✓
- Tanh is preferred in hidden layers because it is **zero-centered** (range [-1, 1]), which helps gradient descent converge faster since outputs don't have a systematic positive bias.

---

### Q6: Saturation Analysis
**Q:** For what range of z values does the sigmoid derivative exceed 0.1? What does this imply about feature scaling in logistic regression?

**A:** σ'(z) = σ(z)(1-σ(z)) > 0.1. Solving: the quadratic σ² - σ + 0.1 < 0 gives σ ∈ (0.1127, 0.8873), corresponding to **z ∈ (-2.05, 2.05)** approximately. This means gradients are meaningful only when |z| < ~2. Implication: if features aren't scaled, z = wᵀx + b can easily be ±50 or more, placing all data points in the saturation zone where learning is extremely slow. **Feature scaling** (standardization) keeps z values in a reasonable range.

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Chapter 1: Classification Problem](01-classification-problem.md) | **Chapter 2: Sigmoid Function** | [Chapter 3: Decision Boundary →](03-decision-boundary.md) |

### Unit 5 Chapters
1. [Classification Problem](01-classification-problem.md)
2. **📍 Sigmoid Function** ← You are here
3. [Decision Boundary](03-decision-boundary.md)
4. [Cost Function & Log Loss](04-cost-function-log-loss.md)
5. [Gradient Descent for Logistic Regression](05-gradient-descent-logistic.md)
6. [Multi-Class Classification](06-multi-class-classification.md)
7. [Regularization](07-regularization.md)

---

> *"The sigmoid function is the bridge between linear algebra and probability — a simple formula that makes classification possible."*

© 2025 AI/ML Study Notes. All rights reserved.
