# 📘 Chapter 3: Decision Boundary

> **Unit 5: Logistic Regression** | **Section 3 of 7**
> Understand how logistic regression draws the line (literally) between classes. Explore linear and nonlinear decision boundaries, the role of weights and bias, and how to visualize decision regions in 2D and higher dimensions.

---

## 📑 Table of Contents

1. [What is a Decision Boundary?](#1-what-is-a-decision-boundary)
2. [Linear Decision Boundary: The Math](#2-linear-decision-boundary-the-math)
3. [Effect of Weights and Bias on the Boundary](#3-effect-of-weights-and-bias-on-the-boundary)
4. [ASCII Visualization of Decision Boundaries](#4-ascii-visualization-of-decision-boundaries)
5. [Nonlinear Decision Boundaries via Feature Engineering](#5-nonlinear-decision-boundaries-via-feature-engineering)
6. [Multi-Class Decision Boundaries](#6-multi-class-decision-boundaries)
7. [Decision Boundary in Higher Dimensions](#7-decision-boundary-in-higher-dimensions)
8. [Python Implementation with sklearn](#8-python-implementation-with-sklearn)
9. [Summary Table](#9-summary-table)
10. [Revision Questions](#10-revision-questions)

---

## 1. What is a Decision Boundary?

### Definition

The **decision boundary** is the surface in feature space where the model's predicted probability equals the threshold (typically 0.5). It separates the regions where the model predicts different classes.

```
Decision Boundary Concept:

    x₂
    │
    │   ■ ■ ■ ■         ← Region: P(y=1|x) ≥ 0.5 → Predict 1
    │ ■ ■ ■ ■ ■ ■
    │   ■ ■ ■  ╱─── Decision Boundary: P(y=1|x) = 0.5
    │ ■ ■ ■  ╱
    │   ■  ╱  ● ● ●
    │     ╱ ● ● ● ●
    │   ╱  ● ● ● ● ●   ← Region: P(y=1|x) < 0.5 → Predict 0
    │ ╱  ● ● ● ● ●
    └──────────────── x₁
```

### Mathematical Condition

For logistic regression:

```
    ŷ = σ(z) = σ(wᵀx + b)

    Decision rule:  predict 1 if σ(z) ≥ 0.5
                    predict 0 if σ(z) < 0.5
    
    Since σ(z) = 0.5 exactly when z = 0:
    
    ┌──────────────────────────────────────────────┐
    │  Decision Boundary:  wᵀx + b = 0            │
    │                                               │
    │  Predict 1 when:     wᵀx + b ≥ 0  (z ≥ 0)  │
    │  Predict 0 when:     wᵀx + b < 0  (z < 0)  │
    └──────────────────────────────────────────────┘
```

---

## 2. Linear Decision Boundary: The Math

### Two Features (2D)

With features x₁ and x₂:

```
    z = w₁x₁ + w₂x₂ + b = 0
    
    Solving for x₂:
    ┌───────────────────────────────────┐
    │  x₂ = -(w₁/w₂)·x₁ - b/w₂       │
    │                                   │
    │  slope = -w₁/w₂                  │
    │  intercept = -b/w₂               │
    └───────────────────────────────────┘
```

### Example Calculation

```
Given: w₁ = 2, w₂ = 3, b = -6

Decision boundary: 2x₁ + 3x₂ - 6 = 0

Solving for x₂:  x₂ = -(2/3)x₁ + 2

    Points on the boundary:
    x₁ = 0  →  x₂ = 2
    x₁ = 3  →  x₂ = 0
    x₁ = 1.5 → x₂ = 1

    x₂
    3 ┤
      │
    2 ┤■
      │ ╲
    1 ┤   ╲  ← boundary line
      │     ╲
    0 ┤───────■──── x₁
      0   1   2   3
    
    Above the line (2x₁ + 3x₂ - 6 > 0): Predict 1
    Below the line (2x₁ + 3x₂ - 6 < 0): Predict 0
```

### One Feature (1D)

```
    z = w₁x₁ + b = 0  →  x₁ = -b/w₁
    
    Example: w₁ = 2, b = -5  →  boundary at x₁ = 2.5
    
    ● ● ● ● ● │ ■ ■ ■ ■ ■
    Class 0    2.5   Class 1
               ↑
          decision boundary
          (a single point)
```

### General Form (n Features)

```
    z = w₁x₁ + w₂x₂ + ... + wₙxₙ + b = 0
    
    This defines a HYPERPLANE in n-dimensional space.
    
    Vector form:  wᵀx + b = 0
    
    where w = [w₁, w₂, ..., wₙ]ᵀ  (weight vector)
          x = [x₁, x₂, ..., xₙ]ᵀ  (feature vector)
          b = bias (scalar)
```

---

## 3. Effect of Weights and Bias on the Boundary

### The Weight Vector is Normal to the Boundary

```
    The weight vector w = [w₁, w₂] is PERPENDICULAR 
    to the decision boundary!

    x₂
    │
    │    ╲             w = [w₁, w₂]
    │     ╲           ↗
    │      ╲        ╱  (normal to boundary)
    │       ╲     ╱
    │        ╲  ╱
    │         ╲╱─────── decision boundary (wᵀx + b = 0)
    │        ╱╲
    │      ╱    ╲
    │    ╱        ╲
    └──────────────── x₁
    
    w points toward the POSITIVE class region (class 1).
    The length ‖w‖ controls how "steep" the probability 
    transition is at the boundary.
```

### Effect of Changing Weights

```
Case 1: w = [1, 0], b = -2
    
    Boundary: x₁ = 2 (vertical line)
    Only x₁ matters — x₂ is irrelevant!
    
    x₂ │  ●  │  ■
       │  ●  │  ■
       │  ●  │  ■
       └──┼──┼──── x₁
          0  2

Case 2: w = [0, 1], b = -2
    
    Boundary: x₂ = 2 (horizontal line)
    Only x₂ matters — x₁ is irrelevant!
    
    x₂ │  ■  ■  ■
    2 ─┤──────────
       │  ●  ●  ●
       └──────── x₁

Case 3: w = [1, 1], b = -2
    
    Boundary: x₁ + x₂ = 2 (diagonal)
    Both features contribute equally.
    
    x₂ │■
    2 ─┤ ■╲
       │■  ╲ ●
       │  ■ ╲ ●
       └──────╲── x₁
          0  2

Case 4: w = [2, 1], b = -2
    
    Boundary: 2x₁ + x₂ = 2
    x₁ has TWICE the influence of x₂.
    
    x₂ │■ ■
    2 ─┤■ ╲
       │■  ╲● ●
       │  ╲ ● ●
       └──╲───── x₁
          0  1
```

### Effect of Changing Bias

```
The bias b SHIFTS the boundary without rotating it:

    b = -4:               b = -2:               b = 0:
    x₂│     ╲             x₂│   ╲               x₂│╲
       │      ╲               │    ╲                 │ ╲
       │       ╲              │     ╲                │  ╲
       │        ╲             │      ╲               │   ╲
       └─────────╲ x₁        └───────╲ x₁           └────╲ x₁
    
    Larger |b| pushes the boundary further from the origin.
    Sign of b determines which direction.
```

### Weight Magnitude and Confidence

```
Small weights ‖w‖:                Large weights ‖w‖:

    P(y=1)                           P(y=1)
    1.0 ┤        ·····────           1.0 ┤           ┌────
        │     ···                        │           │
    0.5 ┤─ ─◆─ ─ ─ ─ ─ ─           0.5 ┤─ ─ ─ ─ ─ ◆ ─ ─
        │  ···                           │           │
    0.0 ┤────·····                   0.0 ┤────────── ┘
        └─────────── x                   └─────────── x
    
    Gradual transition                Sharp transition
    (uncertain near boundary)         (confident, step-like)
```

---

## 4. ASCII Visualization of Decision Boundaries

### Simple Linear Boundary

```
    x₂
    5 ┤ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■
      │ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■
    4 ┤ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■
      │ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ .
    3 ┤ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ . . ●
      │ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ . . ● ● ●
    2 ┤ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ ■ . . . ● ● ● ● ●
      │ ■ ■ ■ ■ ■ ■ ■ ■ ■ . . . ● ● ● ● ● ● ● ●
    1 ┤ ■ ■ ■ ■ ■ ■ . . . ● ● ● ● ● ● ● ● ● ● ●
      │ ■ ■ ■ . . . ● ● ● ● ● ● ● ● ● ● ● ● ● ●
    0 ┤ . . . ● ● ● ● ● ● ● ● ● ● ● ● ● ● ● ● ●
      └─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┬─┤
        0 1 2 3 4 5 6 7 8 9            ...       x₁
    
    Boundary: w₁x₁ + w₂x₂ + b = 0  (diagonal line)
    ■ = Class 1 (above)    ● = Class 0 (below)
    . = transition zone (near 0.5 probability)
```

### Probability Heat Map View

```
    Decision region with probability gradient:
    
    x₂
    │ 0.99 0.97 0.93 0.85 0.73 0.58 0.42 0.27 0.15 0.07
    │ 0.97 0.93 0.85 0.73 0.58 0.42 0.27 0.15 0.07 0.03
    │ 0.93 0.85 0.73 0.58 0.42 0.27 0.15 0.07 0.03 0.01
    │ 0.85 0.73 0.58 0.42 0.27 0.15 0.07 0.03 0.01 0.00
    │ 0.73 0.58 0.42 0.27 0.15 0.07 0.03 0.01 0.00 0.00
    └──────────────────────────────────────────────────── x₁
                          ↑
                   Decision boundary
                  (where P = 0.50)
    
    Notice how probability changes SMOOTHLY, not abruptly.
    The boundary is where P transitions through 0.5.
```

---

## 5. Nonlinear Decision Boundaries via Feature Engineering

### The Limitation of Linear Boundaries

Standard logistic regression can only draw **straight lines** (or hyperplanes). What if the data isn't linearly separable?

```
Linearly Separable:              NOT Linearly Separable:
    x₂                              x₂
    │ ■ ■ ■                          │      ● ●
    │ ■ ■ ╱                          │    ● ■ ■ ●
    │ ■ ╱ ● ●                       │   ● ■ ■ ■ ●
    │ ╱ ● ● ●                       │    ● ■ ■ ●
    └───────── x₁                    │      ● ●
                                     └────────── x₁
    ✓ One line separates             ✗ No single line works!
```

### Solution: Add Polynomial Features

By adding **engineered features** like x₁², x₂², x₁x₂, we can create **nonlinear** boundaries while still using logistic regression:

```
Original features: [x₁, x₂]

With degree-2 polynomial features:
    [x₁, x₂, x₁², x₂², x₁x₂]

Decision boundary:
    w₁x₁ + w₂x₂ + w₃x₁² + w₄x₂² + w₅x₁x₂ + b = 0
    
    This can represent:
    • Circles     (w₃ = w₄, w₅ = 0)
    • Ellipses    (w₃ ≠ w₄, w₅ = 0)
    • Parabolas   (one of w₃, w₄ = 0)
    • Hyperbolas  (w₃ and w₄ have opposite signs)
```

### Example: Circular Boundary

```
    Suppose: w = [0, 0, 1, 1], b = -4
    Features: [x₁, x₂, x₁², x₂²]
    
    Boundary: x₁² + x₂² - 4 = 0
    →         x₁² + x₂² = 4
    →         This is a CIRCLE with radius 2!
    
    x₂
    2 ┤       ●●●●●
      │     ●●■■■■●●
      │    ●●■■■■■■●●
    1 ┤   ●●■■■■■■■●●
      │   ●■■■■■■■■●
    0 ┤   ●■■■■■■■■●
      │   ●●■■■■■■●●
   -1 ┤   ●●■■■■■●●●
      │    ●●■■■■●●
   -2 ┤      ●●●●●
      └─┬─┬─┬─┬─┬─┬─┤
       -2-1  0  1  2  x₁
    
    Inside circle (x₁²+x₂² < 4):  Class 1 (■)
    Outside circle (x₁²+x₂² > 4): Class 0 (●)
```

### Higher-Degree Polynomials

```
Degree 1: Linear boundaries (lines, planes)
    z = w₁x₁ + w₂x₂ + b

Degree 2: Quadratic boundaries (circles, ellipses, parabolas)
    z = w₁x₁ + w₂x₂ + w₃x₁² + w₄x₂² + w₅x₁x₂ + b

Degree 3: Cubic boundaries (complex curves)
    z = ... + w₆x₁³ + w₇x₁²x₂ + w₈x₁x₂² + w₉x₂³ + b

    x₂                           x₂
    │     ╱                       │  ╱╲    ╱
    │   ╱                         │╱    ╲╱
    │ ╱                           │     ╱╲
    └──── x₁                     └──╱────── x₁
    Degree 1                      Degree 3
    (2 parameters)                (9 parameters)
    
    ⚠️ Higher degree = more flexible but risk of OVERFITTING!
```

---

## 6. Multi-Class Decision Boundaries

### One-vs-Rest Boundaries

With K classes, we learn K decision boundaries:

```
    3-Class Classification (One-vs-Rest):
    
    x₂
    │ ▲ ▲ ▲ ▲ ▲ ▲ ▲ ╲
    │ ▲ ▲ ▲ ▲ ▲ ▲ ╱  ╲
    │ ▲ ▲ ▲ ▲ ▲ ╱     ╲ ■ ■
    │ ▲ ▲ ▲ ▲ ╱    ■ ■ ■ ■ ■
    │ ▲ ▲ ▲ ╱  ■ ■ ■ ■ ■ ■ ■
    │ ▲ ▲ ╱ ■ ■ ■ ■ ■ ■ ■ ■
    │ ▲ ╱ ─ ─ ─ ─ ─ ─ ─ ─ ─ 
    │ ╱ ● ● ● ● ● ● ● ● ● ●
    │ ● ● ● ● ● ● ● ● ● ● ●
    └─────────────────────── x₁
    
    Three boundaries divide the space into 3 regions:
    ● Class 0 region    ■ Class 1 region    ▲ Class 2 region
```

### Softmax Decision Regions

```
    With softmax (multinomial logistic regression),
    boundaries are the loci where two classes have
    equal probability:
    
    P(class i | x) = P(class j | x)
    
    This creates PAIRWISE LINEAR boundaries that
    meet at vertices, forming a Voronoi-like partition:
    
    x₂
    │╲  Class 2  ╱
    │  ╲        ╱
    │    ╲    ╱
    │      ╲╱       ← boundaries meet at a point
    │  C0  ╱╲  C1
    │    ╱    ╲
    │  ╱        ╲
    └──────────── x₁
```

---

## 7. Decision Boundary in Higher Dimensions

### Geometric Interpretation

| n (features) | Boundary Type | Example |
|---|---|---|
| 1 | Point | x₁ = 2.5 |
| 2 | Line | 2x₁ + 3x₂ - 6 = 0 |
| 3 | Plane | w₁x₁ + w₂x₂ + w₃x₃ + b = 0 |
| n | Hyperplane | wᵀx + b = 0 |

### The Normal Vector Interpretation

```
    The weight vector w is always PERPENDICULAR (normal)
    to the decision boundary hyperplane.
    
    3D Example:
    
              x₃
              │     ╱╱╱╱╱╱╱╱ ← decision plane
              │   ╱╱╱╱╱╱╱╱
              │ ╱╱╱╱╱╱╱╱
              │╱╱╱╱╱╱╱
              │╱╱╱╱╱ ──→ w (normal to the plane)
              │╱╱╱
             ╱│
           ╱  │
         ╱    │
    x₂ ╱     └─────────── x₁
    
    Distance from origin to boundary: |b|/‖w‖
```

### Margin and Distance

The **distance** from a point x₀ to the decision boundary wᵀx + b = 0 is:

```
    distance = |wᵀx₀ + b| / ‖w‖

    This is the "functional margin" / ‖w‖
    
    Points far from the boundary → high confidence
    Points near the boundary → low confidence (close to 0.5)
```

---

## 8. Python Implementation with sklearn

### Basic Decision Boundary Visualization

```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.linear_model import LogisticRegression
from sklearn.preprocessing import StandardScaler

# ─── Generate 2D binary classification data ───
np.random.seed(42)
X, y = make_classification(
    n_samples=200, n_features=2, n_informative=2,
    n_redundant=0, n_clusters_per_class=1, random_state=42
)

# Scale features
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# Train logistic regression
model = LogisticRegression(random_state=42)
model.fit(X_scaled, y)

# Extract boundary parameters
w = model.coef_[0]
b = model.intercept_[0]

print(f"Weights: w₁ = {w[0]:.4f}, w₂ = {w[1]:.4f}")
print(f"Bias:    b  = {b:.4f}")
print(f"Boundary equation: {w[0]:.4f}·x₁ + {w[1]:.4f}·x₂ + ({b:.4f}) = 0")
print(f"Slope:     {-w[0]/w[1]:.4f}")
print(f"Intercept: {-b/w[1]:.4f}")

# Compute decision boundary line
x1_range = np.linspace(X_scaled[:, 0].min() - 1, X_scaled[:, 0].max() + 1, 100)
x2_boundary = -(w[0] * x1_range + b) / w[1]

print(f"\nBoundary passes through:")
for x1_val in [-1, 0, 1]:
    x2_val = -(w[0] * x1_val + b) / w[1]
    print(f"  ({x1_val:.1f}, {x2_val:.4f})")
```

### ASCII Decision Boundary Plot

```python
def ascii_decision_boundary(X, y, model, resolution=40, height=20):
    """Plot a text-based decision boundary."""
    x1_min, x1_max = X[:, 0].min() - 0.5, X[:, 0].max() + 0.5
    x2_min, x2_max = X[:, 1].min() - 0.5, X[:, 1].max() + 0.5
    
    # Create grid
    x1_grid = np.linspace(x1_min, x1_max, resolution)
    x2_grid = np.linspace(x2_max, x2_min, height)  # reversed for display
    
    print(f"\n  Decision Boundary Visualization")
    print(f"  {'─' * (resolution + 4)}")
    
    for i, x2 in enumerate(x2_grid):
        row = f"  {x2:+5.1f} │"
        for j, x1 in enumerate(x1_grid):
            point = np.array([[x1, x2]])
            prob = model.predict_proba(point)[0, 1]
            
            # Check if any data point is near this grid cell
            near_point = False
            for k in range(len(X)):
                if (abs(X[k, 0] - x1) < (x1_max - x1_min) / resolution and
                    abs(X[k, 1] - x2) < (x2_max - x2_min) / height):
                    row += "●" if y[k] == 0 else "■"
                    near_point = True
                    break
            
            if not near_point:
                if abs(prob - 0.5) < 0.03:
                    row += "─"  # boundary
                elif prob >= 0.5:
                    row += "░"  # class 1 region (light)
                else:
                    row += " "  # class 0 region
        
        print(row)
    
    print(f"       └{'─' * resolution}")
    print(f"        {x1_min:+.1f}" + " " * (resolution // 2 - 5) + 
          f"x₁" + " " * (resolution // 2 - 5) + f"{x1_max:+.1f}")

ascii_decision_boundary(X_scaled, y, model)
```

### Nonlinear Decision Boundary with Polynomial Features

```python
from sklearn.preprocessing import PolynomialFeatures
from sklearn.pipeline import Pipeline

# ─── Create non-linearly separable data (circles) ───
np.random.seed(42)
n = 200
theta = np.random.uniform(0, 2 * np.pi, n)
r_inner = np.random.uniform(0, 1.5, n // 2)
r_outer = np.random.uniform(2.5, 4.0, n // 2)

X_inner = np.column_stack([r_inner * np.cos(theta[:n//2]), 
                            r_inner * np.sin(theta[:n//2])])
X_outer = np.column_stack([r_outer * np.cos(theta[n//2:]),
                            r_outer * np.sin(theta[n//2:])])

X_circle = np.vstack([X_inner, X_outer])
y_circle = np.array([1] * (n // 2) + [0] * (n // 2))

# Linear model (will fail)
model_linear = LogisticRegression(random_state=42)
model_linear.fit(X_circle, y_circle)
acc_linear = model_linear.score(X_circle, y_circle)

# Polynomial model (degree 2 — will capture the circle!)
model_poly = Pipeline([
    ('poly', PolynomialFeatures(degree=2, include_bias=False)),
    ('lr', LogisticRegression(random_state=42, max_iter=1000))
])
model_poly.fit(X_circle, y_circle)
acc_poly = model_poly.score(X_circle, y_circle)

print(f"Linear Logistic Regression Accuracy:    {acc_linear:.4f}")
print(f"Polynomial (deg=2) LR Accuracy:         {acc_poly:.4f}")

# Examine polynomial features
poly = PolynomialFeatures(degree=2, include_bias=False)
X_poly = poly.fit_transform(X_circle[:1])
print(f"\nOriginal features: {X_circle[0]}")
print(f"Polynomial features: {X_poly[0]}")
print(f"Feature names: {poly.get_feature_names_out()}")
```

### Visualizing Effect of Weight Changes

```python
# ─── Show how weights affect the boundary ───

def show_boundary(w1, w2, b, label=""):
    """Print boundary equation and key points."""
    print(f"\n{label}")
    print(f"  Boundary: {w1:.1f}·x₁ + {w2:.1f}·x₂ + ({b:.1f}) = 0")
    
    if w2 != 0:
        slope = -w1 / w2
        intercept = -b / w2
        print(f"  x₂ = {slope:.2f}·x₁ + {intercept:.2f}")
        print(f"  Slope: {slope:.2f}, y-intercept: {intercept:.2f}")
    else:
        print(f"  x₁ = {-b/w1:.2f} (vertical line)")
    
    # Weight vector direction
    norm = np.sqrt(w1**2 + w2**2)
    print(f"  Weight vector: [{w1:.1f}, {w2:.1f}], ‖w‖ = {norm:.2f}")
    print(f"  Distance from origin: {abs(b)/norm:.2f}")

show_boundary(1, 1, -2, "Equal weights (45° diagonal)")
show_boundary(2, 1, -2, "x₁ twice as important")
show_boundary(1, 2, -2, "x₂ twice as important")
show_boundary(1, 0, -3, "Only x₁ matters (vertical)")
show_boundary(0, 1, -3, "Only x₂ matters (horizontal)")
show_boundary(1, 1, 0,  "Through origin")
show_boundary(1, 1, -4, "Shifted far from origin")
```

### Complete sklearn Pipeline for Decision Boundary

```python
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report

# ─── Full pipeline ───
X, y = make_classification(
    n_samples=500, n_features=2, n_informative=2,
    n_redundant=0, random_state=42
)

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Scale features (important for boundary interpretation)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# Train model
model = LogisticRegression(random_state=42)
model.fit(X_train_s, y_train)

# Evaluate
y_pred = model.predict(X_test_s)
print(f"Test Accuracy: {accuracy_score(y_test, y_pred):.4f}")
print(f"\nClassification Report:")
print(classification_report(y_test, y_pred))

# Boundary analysis
w = model.coef_[0]
b = model.intercept_[0]
print(f"Decision Boundary: {w[0]:.4f}·x₁ + {w[1]:.4f}·x₂ + ({b:.4f}) = 0")

# Points near the boundary (uncertain predictions)
probs = model.predict_proba(X_test_s)[:, 1]
uncertain = np.abs(probs - 0.5) < 0.1
print(f"\nUncertain predictions (|P-0.5| < 0.1): {uncertain.sum()} out of {len(y_test)}")
print(f"Most confident P(y=1): {probs.max():.4f}")
print(f"Least confident P(y=1): {probs[np.argmin(np.abs(probs - 0.5))]:.4f}")
```

### Multi-Feature Decision Boundary (3D+)

```python
# ─── 3D decision boundary (plane) ───
from sklearn.datasets import make_classification

X_3d, y_3d = make_classification(
    n_samples=300, n_features=3, n_informative=3,
    n_redundant=0, random_state=42
)

model_3d = LogisticRegression(random_state=42)
model_3d.fit(X_3d, y_3d)

w = model_3d.coef_[0]
b = model_3d.intercept_[0]

print(f"3D Decision Boundary (a plane):")
print(f"  {w[0]:.4f}·x₁ + {w[1]:.4f}·x₂ + {w[2]:.4f}·x₃ + ({b:.4f}) = 0")
print(f"\nNormal vector: [{w[0]:.4f}, {w[1]:.4f}, {w[2]:.4f}]")
print(f"Normal vector magnitude: {np.linalg.norm(w):.4f}")
print(f"Distance from origin: {abs(b) / np.linalg.norm(w):.4f}")

# In higher dimensions, we can only visualize 2D slices
# Fix x₃ = 0 and plot the remaining 2D boundary
print(f"\nSlice at x₃ = 0:")
print(f"  {w[0]:.4f}·x₁ + {w[1]:.4f}·x₂ + ({b:.4f}) = 0")
if w[1] != 0:
    print(f"  x₂ = {-w[0]/w[1]:.4f}·x₁ + ({-b/w[1]:.4f})")
```

---

## 9. Summary Table

| Concept | Key Equation | Description |
|---------|-------------|-------------|
| **Decision boundary** | wᵀx + b = 0 | Surface where P(y=1) = 0.5 |
| **Positive region** | wᵀx + b > 0 | Predict class 1 (P > 0.5) |
| **Negative region** | wᵀx + b < 0 | Predict class 0 (P < 0.5) |
| **Boundary slope** (2D) | -w₁/w₂ | Determined by weight ratio |
| **Boundary intercept** (2D) | -b/w₂ | Controlled by bias |
| **Normal to boundary** | w = [w₁, ..., wₙ] | Points toward class 1 region |
| **Distance from origin** | \|b\|/‖w‖ | How far boundary is from center |
| **Point-to-boundary distance** | \|wᵀx₀ + b\|/‖w‖ | Confidence measure |
| **Nonlinear boundary** | Add polynomial features | x², x₁x₂, etc. |
| **Weight magnitude** | ‖w‖ | Controls sharpness of transition |

---

## 10. Revision Questions

### Q1: Boundary Equation
**Q:** A logistic regression model has w = [3, -2] and b = 6. Write the decision boundary equation, find the slope, and determine which class is predicted for the point (1, 5).

**A:**
- Boundary: 3x₁ - 2x₂ + 6 = 0 → x₂ = (3/2)x₁ + 3
- Slope = -w₁/w₂ = -3/(-2) = **3/2 = 1.5**
- For point (1, 5): z = 3(1) - 2(5) + 6 = 3 - 10 + 6 = **-1**
- Since z < 0 → σ(z) < 0.5 → **Predict class 0**

---

### Q2: Weight Interpretation
**Q:** If you double all weights (w → 2w) but keep the bias the same, what happens to: (a) the position of the decision boundary, (b) the probability predictions far from the boundary?

**A:**
- (a) Boundary position **changes** because wᵀx + b = 0 is not the same as 2wᵀx + b = 0 (the bias didn't scale). The boundary shifts toward the origin by a factor related to b.
- (b) Far from the boundary, |z| becomes even larger (doubled), so σ(z) approaches 0 or 1 even more quickly. **Predictions become more confident** (sharper transition).

---

### Q3: Nonlinear Boundary
**Q:** Can standard logistic regression (without feature engineering) produce a circular decision boundary? How would you modify the approach to achieve this?

**A:** **No**, standard logistic regression produces only linear (hyperplane) boundaries. For a circular boundary x₁² + x₂² = r², add polynomial features: create new features x₁², x₂², and x₁x₂ using `PolynomialFeatures(degree=2)`. The model becomes linear in the **expanded** feature space but produces a nonlinear boundary in the **original** space. The learned weights on x₁² and x₂² will be equal if the boundary is a perfect circle.

---

### Q4: Distance from Boundary
**Q:** Given weights w = [4, 3] and bias b = -10, compute the distance from the decision boundary to: (a) the origin, (b) the point (5, 5).

**A:**
- ‖w‖ = √(4² + 3²) = √(16 + 9) = √25 = **5**
- (a) Distance from origin: |wᵀ·0 + b|/‖w‖ = |0 + (-10)|/5 = **2.0** units
- (b) Distance from (5,5): |4(5) + 3(5) - 10|/5 = |20 + 15 - 10|/5 = 25/5 = **5.0** units

---

### Q5: Bias Effect
**Q:** A decision boundary currently passes through the origin (b = 0) with weights w = [1, -1]. What value of b would shift the boundary so it passes through the point (2, 2)?

**A:** For the boundary to pass through (2, 2): w₁(2) + w₂(2) + b = 0 → 1(2) + (-1)(2) + b = 0 → 2 - 2 + b = 0 → **b = 0**. The point (2, 2) already lies on the boundary! This is because w = [1, -1] creates the boundary x₁ - x₂ = 0 (i.e., x₁ = x₂), and (2, 2) satisfies this. For the boundary to pass through (3, 1): 1(3) + (-1)(1) + b = 0 → b = **-2**.

---

### Q6: Polynomial Features Count
**Q:** If you have 3 original features (x₁, x₂, x₃) and create polynomial features up to degree 2, list all the resulting features and count them. How does this affect the decision boundary?

**A:** Features: x₁, x₂, x₃, x₁², x₂², x₃², x₁x₂, x₁x₃, x₂x₃ = **9 features** (plus bias). The decision boundary becomes a **quadric surface** in 3D (can be an ellipsoid, hyperboloid, paraboloid, etc.), enabling the model to separate classes with curved surfaces. The general formula for degree d features with n original features is C(n+d, d) - 1.

---

## 🧭 Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← Chapter 2: Sigmoid Function](02-sigmoid-function.md) | **Chapter 3: Decision Boundary** | [Chapter 4: Cost Function & Log Loss →](04-cost-function-log-loss.md) |

### Unit 5 Chapters
1. [Classification Problem](01-classification-problem.md)
2. [Sigmoid Function](02-sigmoid-function.md)
3. **📍 Decision Boundary** ← You are here
4. [Cost Function & Log Loss](04-cost-function-log-loss.md)
5. [Gradient Descent for Logistic Regression](05-gradient-descent-logistic.md)
6. [Multi-Class Classification](06-multi-class-classification.md)
7. [Regularization](07-regularization.md)

---

> *"The decision boundary is where the model is most uncertain — and therefore where the most interesting things happen."*

© 2025 AI/ML Study Notes. All rights reserved.
