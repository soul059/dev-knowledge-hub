# 1. Mixture Models Concept

[← Back to GMM Unit](./README.md) | [Next: GMM Intuition →](./02-gmm-intuition.md)

---

## Chapter Overview

A **mixture model** assumes that observed data is generated from a combination (mixture) of several underlying probability distributions. Each distribution represents a distinct subpopulation or cluster within the data. **Gaussian Mixture Models (GMMs)** are the most common type, where each component is a multivariate Gaussian (normal) distribution. This chapter introduces the mixture model framework, the concept of latent variables, mixture weights, and the generative story behind GMMs.

---

## 1.1 What is a Mixture Distribution?

### Intuition

Imagine measuring the heights of students in a school. If you measure both boys and girls together, the combined histogram won't look like a single bell curve — it will be a **superposition** of two bell curves (one for boys, one for girls) with different means.

```
  Combined Distribution (what we observe):
  
  Frequency
     │       ╱╲           ╱╲
     │      ╱  ╲         ╱  ╲
     │     ╱    ╲       ╱    ╲
     │    ╱      ╲     ╱      ╲
     │   ╱        ╲   ╱        ╲
     │  ╱          ╲ ╱          ╲
     │ ╱            ╳            ╲
     │╱           ╱   ╲           ╲
     └──────────────────────────────── Height
           Girls          Boys
           μ₁=160cm       μ₂=175cm

  The observed distribution = π₁·N(μ₁,σ₁²) + π₂·N(μ₂,σ₂²)
  where π₁ = fraction of girls, π₂ = fraction of boys
```

### Formal Definition

A **mixture distribution** with K components is:

```
p(x) = Σ_{k=1}^{K} πₖ · f_k(x)

where:
  πₖ = mixing weight (proportion) of component k
  f_k(x) = probability density of component k
  
  Constraints:
    πₖ ≥ 0        for all k
    Σ πₖ = 1       (weights sum to 1)
```

### For Gaussian Mixture Models:

```
p(x) = Σ_{k=1}^{K} πₖ · N(x | μₖ, Σₖ)

where:
  N(x | μₖ, Σₖ) = Gaussian (normal) density with:
    μₖ = mean vector of component k
    Σₖ = covariance matrix of component k
```

---

## 1.2 K Gaussian Components

Each component in a GMM is a **multivariate Gaussian distribution** characterized by:

### 1.2.1 Mean Vector μₖ

The center of the k-th component.

```
For d-dimensional data:
  μₖ = [μₖ₁, μₖ₂, ..., μₖd]ᵀ

Example (2D, 3 components):
  μ₁ = [2, 3]ᵀ    (center of component 1)
  μ₂ = [6, 7]ᵀ    (center of component 2)
  μ₃ = [4, 1]ᵀ    (center of component 3)
```

### 1.2.2 Covariance Matrix Σₖ

Controls the shape, size, and orientation of the k-th component.

```
For 2D data:
  Σₖ = [ σ²ₖ₁₁   σₖ₁₂ ]     Diagonal: circular/axis-aligned ellipse
       [ σₖ₂₁   σ²ₖ₂₂ ]     Full: rotated ellipse

  σ²ₖ₁₁ = variance along dimension 1
  σ²ₖ₂₂ = variance along dimension 2
  σₖ₁₂ = σₖ₂₁ = covariance (controls rotation)
```

### 1.2.3 Visual: Different Covariance Structures

```
  Spherical (Σ = σ²I):     Diagonal:              Full:
  
       · · ·                  · ·               · ·
     · · · · ·              · · · ·            · · · · ·
    · · · · · ·            · · · · ·          · · · · · ·
     · · · · ·              · · · ·            · · · · ·
       · · ·                  · ·               · ·
                                                  
  Perfect circle           Axis-aligned          Rotated
  (equal variance)         ellipse               ellipse
  (1 parameter)            (d parameters)        (d(d+1)/2 params)
```

---

## 1.3 Latent Variable z

The **latent variable** (hidden variable) **z** indicates which component generated each data point. We can't observe z directly — we only see the data x.

```
For each data point xₙ:
  zₙ ∈ {1, 2, ..., K}    (which component generated xₙ?)
  
  zₙ is UNOBSERVED (latent/hidden)
  xₙ is OBSERVED (the data we see)

  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │  Observed:  x₁, x₂, x₃, ..., xₙ    (the data)     │
  │  Hidden:    z₁, z₂, z₃, ..., zₙ    (unknown!)      │
  │                                                      │
  │  Goal: Infer z (cluster assignments) from x (data)  │
  │                                                      │
  └──────────────────────────────────────────────────────┘
```

### One-Hot Encoding of z

Often z is represented as a one-hot vector:

```
If K = 3 and point n came from component 2:

  zₙ = [0, 1, 0]ᵀ

  zₙₖ = 1 if point n from component k
  zₙₖ = 0 otherwise

  Distribution:
  P(zₙₖ = 1) = πₖ   (prior probability of component k)
```

---

## 1.4 Mixture Weights πₖ

The **mixture weight** πₖ represents the prior probability that a randomly chosen data point belongs to component k.

```
π = [π₁, π₂, ..., πₖ]

Properties:
  1. πₖ ≥ 0   for all k     (non-negative)
  2. Σ πₖ = 1               (sum to 1)
  3. πₖ ≈ Nₖ/N              (fraction of data from component k)

Example (3 components):
  π₁ = 0.5    →  50% of data from component 1
  π₂ = 0.3    →  30% of data from component 2  
  π₃ = 0.2    →  20% of data from component 3
       ─────
       1.0       (sum = 1 ✓)
```

### Interpretation

```
Population:
  ┌──────────────────────────────────────────────────┐
  │ Component 1 (50%) │ Component 2 (30%) │ Comp 3   │
  │    π₁ = 0.5       │    π₂ = 0.3       │ π₃=0.2  │
  │  ████████████████  │  ██████████       │ ██████  │
  └──────────────────────────────────────────────────┘
  
  When generating a new data point:
    50% chance it comes from component 1
    30% chance it comes from component 2
    20% chance it comes from component 3
```

---

## 1.5 The Generative Story

A GMM tells a **generative story** — a probabilistic recipe for how the data was created:

### Step-by-Step Generation

```
To generate ONE data point xₙ:

  Step 1: CHOOSE a component
    ┌─────────────────────────────────────────────┐
    │  Roll a K-sided weighted die:               │
    │                                             │
    │    P(choose component k) = πₖ               │
    │                                             │
    │  Example (K=3):                             │
    │    Roll → 0.42  →  falls in [0, 0.5)       │
    │    → Choose component 1 (since π₁ = 0.5)   │
    │                                             │
    │    [───────0.5───────|────0.3────|──0.2──]  │
    │     Component 1       Component 2  Comp 3   │
    │              ↑                               │
    │          0.42 lands here                     │
    └─────────────────────────────────────────────┘

  Step 2: GENERATE from chosen component
    ┌─────────────────────────────────────────────┐
    │  Sample x from N(μₖ, Σₖ):                  │
    │                                             │
    │  If component 1 was chosen:                 │
    │    x ~ N(μ₁, Σ₁)                           │
    │    x ~ N([2,3], [[1,0],[0,1]])              │
    │    → x = [2.3, 3.7]  (one possible sample) │
    └─────────────────────────────────────────────┘

  Repeat N times → dataset of N points
```

### Full Generative Process

```
FOR n = 1 to N:
  1. Sample zₙ ~ Categorical(π₁, π₂, ..., πₖ)     # Choose component
  2. Sample xₙ ~ N(μ_{zₙ}, Σ_{zₙ})                 # Generate from it
  
  This gives us:
    - Observed data: {x₁, x₂, ..., xₙ}
    - Hidden labels: {z₁, z₂, ..., zₙ}   (we don't see these!)
```

### Joint and Marginal Distributions

```
Joint probability (for a specific point and component):

  p(x, z=k) = p(z=k) · p(x | z=k)
             = πₖ · N(x | μₖ, Σₖ)

Marginal probability (what we observe, summing over all components):

  p(x) = Σ_{k=1}^{K} p(x, z=k)
       = Σ_{k=1}^{K} πₖ · N(x | μₖ, Σₖ)
       
  This is the GMM density!
```

### Graphical Model

```
  ┌─────────────────────────────────────────────────┐
  │   Plate notation (repeated N times):            │
  │                                                 │
  │         π                                       │
  │         │                                       │
  │         ▼                                       │
  │   ┌── [zₙ] ──┐                                │
  │   │           │                                 │
  │   │     │     │                                 │
  │   │     ▼     │                                 │
  │   │   (xₙ) ◄── μₖ, Σₖ                         │
  │   │           │                                 │
  │   └───────────┘  × N                           │
  │                                                 │
  │   [·] = latent (unobserved)                    │
  │   (·) = observed                                │
  │   Arrows = conditional dependence               │
  └─────────────────────────────────────────────────┘
```

---

## 1.6 The Multivariate Gaussian Density

For completeness, the probability density function of a d-dimensional Gaussian:

```
N(x | μ, Σ) = ────────────────1──────────────── · exp(-½ (x-μ)ᵀ Σ⁻¹ (x-μ))
               (2π)^{d/2} |Σ|^{1/2}

where:
  x = d-dimensional data point
  μ = d-dimensional mean vector
  Σ = d×d covariance matrix
  |Σ| = determinant of Σ
  Σ⁻¹ = inverse of Σ
  (x-μ)ᵀ Σ⁻¹ (x-μ) = Mahalanobis distance squared
```

### Univariate Special Case (d=1)

```
N(x | μ, σ²) = ──────1────── · exp(-(x-μ)² / 2σ²)
                √(2πσ²)
```

### Example Calculation

```
Given: μ = [1, 2]ᵀ,  Σ = [2  0.5]
                           [0.5  1]

For point x = [1.5, 2.5]ᵀ:

  Step 1: x - μ = [0.5, 0.5]ᵀ

  Step 2: Σ⁻¹ = 1/(2·1 - 0.5·0.5) · [1   -0.5]  =  [0.571  -0.286]
                                       [-0.5   2]     [-0.286  1.143]

  Step 3: (x-μ)ᵀ Σ⁻¹ (x-μ) = [0.5, 0.5] · [0.571·0.5 + (-0.286)·0.5]
                                             [(-0.286)·0.5 + 1.143·0.5]
                              = [0.5, 0.5] · [0.143]
                                              [0.429]
                              = 0.071 + 0.214 = 0.286

  Step 4: |Σ| = 2·1 - 0.5·0.5 = 1.75

  Step 5: N(x|μ,Σ) = 1/(2π·√1.75) · exp(-0.286/2)
                    = 1/(2π·1.323) · exp(-0.143)
                    = 0.120 · 0.867
                    = 0.104
```

---

## 1.7 Why Gaussian Components?

```
┌────────────────────────────────────────────────────────────────┐
│  Why are Gaussians the default choice for mixture components?  │
│                                                                │
│  1. Central Limit Theorem                                     │
│     Many natural phenomena are approximately Gaussian.         │
│     Sums of many independent factors → Gaussian shape.        │
│                                                                │
│  2. Mathematical Tractability                                 │
│     Gaussian has nice closed-form expressions for:             │
│     - Marginalization                                          │
│     - Conditioning                                             │
│     - Products                                                 │
│     - KL divergence                                            │
│                                                                │
│  3. Universal Approximation                                   │
│     A sufficient number of Gaussian components can             │
│     approximate ANY continuous density to arbitrary            │
│     precision (like a Fourier series for distributions).       │
│                                                                │
│  4. Only Two Parameters                                       │
│     Fully characterized by mean and covariance.               │
│     Maximum entropy distribution for given mean/variance.      │
│                                                                │
│  5. Efficient Estimation                                      │
│     EM algorithm has closed-form M-step updates.              │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 1.8 Python: Generating Data from a Mixture

```python
import numpy as np

def generate_gmm_data(n_samples, means, covariances, weights, random_state=42):
    """
    Generate data from a Gaussian Mixture Model.
    
    Parameters:
        n_samples: total number of points
        means: list of K mean vectors
        covariances: list of K covariance matrices
        weights: list of K mixing weights (sum to 1)
        random_state: for reproducibility
    
    Returns:
        X: generated data (n_samples, d)
        z: true component labels (n_samples,)
    """
    rng = np.random.RandomState(random_state)
    K = len(means)
    d = len(means[0])
    
    # Step 1: Choose components based on weights
    z = rng.choice(K, size=n_samples, p=weights)
    
    # Step 2: Generate from chosen components
    X = np.zeros((n_samples, d))
    for k in range(K):
        mask = z == k
        n_k = mask.sum()
        X[mask] = rng.multivariate_normal(means[k], covariances[k], n_k)
    
    return X, z

# Define a 3-component GMM
means = [
    [2, 3],     # Component 1
    [7, 8],     # Component 2
    [5, 2],     # Component 3
]
covariances = [
    [[1, 0.3], [0.3, 1]],        # Component 1: slight positive correlation
    [[1.5, -0.5], [-0.5, 1.5]],  # Component 2: negative correlation
    [[0.5, 0], [0, 2]],          # Component 3: elongated vertically
]
weights = [0.4, 0.35, 0.25]      # 40%, 35%, 25%

# Generate data
X, z = generate_gmm_data(500, means, covariances, weights)

# Summary statistics
for k in range(3):
    mask = z == k
    print(f"Component {k+1}:")
    print(f"  True weight: {weights[k]}")
    print(f"  Actual fraction: {mask.sum()/len(z):.3f}")
    print(f"  True mean: {means[k]}")
    print(f"  Sample mean: {X[mask].mean(axis=0).round(3)}")
    print()
```

**Expected Output:**
```
Component 1:
  True weight: 0.4
  Actual fraction: 0.394
  True mean: [2, 3]
  Sample mean: [2.017 3.048]

Component 2:
  True weight: 0.35
  Actual fraction: 0.354
  True mean: [7, 8]
  Sample mean: [6.953 7.982]

Component 3:
  True weight: 0.25
  Actual fraction: 0.252
  True mean: [5, 2]
  Sample mean: [4.988 2.091]
```

---

## 1.9 Summary Table

| Concept | Symbol | Description | Example |
|---------|--------|-------------|---------|
| **Mixture distribution** | p(x) | Weighted sum of component distributions | p(x) = 0.5·N₁ + 0.3·N₂ + 0.2·N₃ |
| **Component** | f_k(x) | Individual probability distribution | N(x \| μₖ, Σₖ) |
| **Number of components** | K | How many sub-distributions | K = 3 (three clusters) |
| **Mixture weight** | πₖ | Prior probability of component k | π₁=0.5, π₂=0.3, π₃=0.2 |
| **Mean** | μₖ | Center of component k | μ₁ = [2, 3]ᵀ |
| **Covariance** | Σₖ | Shape/spread of component k | Σ₁ = [[1,0],[0,1]] |
| **Latent variable** | zₙ | Hidden: which component generated xₙ | zₙ = 2 (from component 2) |
| **Generative story** | — | Recipe: pick component, then sample | z~Cat(π), x~N(μ_z, Σ_z) |

---

## 1.10 Revision Questions

1. **Define a mixture model** mathematically. What are the three sets of parameters in a Gaussian Mixture Model?

2. **What is a latent variable** in the context of GMMs? Why can't we observe it directly, and why does it matter for the clustering task?

3. **Explain the generative story** of a GMM. Walk through the process of generating a single data point, step by step.

4. **Given a 2-component GMM** with π₁=0.6, μ₁=0, σ₁=1 and π₂=0.4, μ₂=5, σ₂=2, compute p(x=3). Show all steps.

5. **Why are Gaussian distributions** the standard choice for mixture components? List at least three reasons.

6. **What constraint must mixture weights satisfy**, and what is their intuitive meaning? What would happen if we allowed negative mixture weights?

---

[← Back to GMM Unit](./README.md) | [Next: GMM Intuition →](./02-gmm-intuition.md)
