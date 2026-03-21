# 📘 7.7 — Decision Trees for Regression

> **Unit 7 · Decision Trees** — Study Notes  
> *Part 7 of 7 — Regression trees, MSE/MAE splitting, piecewise constant approximation, comparison with linear regression, and practical sklearn examples.*

---

## 📑 Table of Contents

1. [What Is a Regression Tree?](#1-what-is-a-regression-tree)
2. [How Regression Trees Differ from Classification Trees](#2-how-regression-trees-differ-from-classification-trees)
3. [Splitting Criteria for Regression](#3-splitting-criteria-for-regression)
4. [Prediction — Leaf Mean](#4-prediction--leaf-mean)
5. [Step-by-Step Regression Tree Example](#5-step-by-step-regression-tree-example)
6. [Piecewise Constant Approximation](#6-piecewise-constant-approximation)
7. [Comparison with Linear Regression](#7-comparison-with-linear-regression)
8. [DecisionTreeRegressor in sklearn](#8-decisiontreeregressor-in-sklearn)
9. [Practical Example — House Price Prediction](#9-practical-example--house-price-prediction)
10. [Applications](#10-applications)
11. [Summary Table](#11-summary-table)
12. [Revision Questions](#12-revision-questions)
13. [Navigation](#13-navigation)

---

## 1. What Is a Regression Tree?

A **regression tree** is a decision tree where the target variable is **continuous** (a number, not a class label). Instead of predicting a class at each leaf, it predicts a **numerical value**.

```
Classification Tree (predict a label):     Regression Tree (predict a number):

     [Age ≤ 30?]                              [Area ≤ 1500 sqft?]
      /       \                                /              \
   "Young"  "Old"  ← class labels        [Bedrooms ≤ 2?]    [Age ≤ 10?]
                                           /        \          /        \
                                       $180K     $240K     $350K     $280K
                                         ↑         ↑         ↑         ↑
                                    numerical predictions (leaf means)
```

**Key idea:** The tree partitions the feature space into **rectangular regions**, and each region is assigned a **constant prediction** (the average of the training targets in that region).

---

## 2. How Regression Trees Differ from Classification Trees

| Aspect | Classification Tree | Regression Tree |
|--------|---------------------|-----------------|
| **Target type** | Categorical (class labels) | Continuous (numbers) |
| **Splitting criterion** | Gini impurity / Entropy | MSE / MAE / Friedman MSE |
| **Leaf prediction** | Majority class | Mean (or median) of targets |
| **Impurity measure** | Class distribution entropy | Variance of target values |
| **sklearn class** | `DecisionTreeClassifier` | `DecisionTreeRegressor` |
| **Evaluation metrics** | Accuracy, F1, AUC | MSE, RMSE, MAE, R² |
| **Output** | Probabilities + class | Continuous value |

---

## 3. Splitting Criteria for Regression

### 3.1 Mean Squared Error (MSE) — Default

The most common criterion. For a node with samples S:

```
                1     n
MSE(S) =  ─────── × Σ  (yᵢ − ȳ)²
               n    i=1

where ȳ = mean of all yᵢ in node S
```

This is simply the **variance** of the target values in the node.

### 3.2 MSE Reduction (Splitting)

For a binary split into left (Sₗ) and right (Sᵣ):

```
                     |Sₗ|                |Sᵣ|
MSE_split = ──────────── × MSE(Sₗ) + ──────────── × MSE(Sᵣ)
             |Sₗ| + |Sᵣ|              |Sₗ| + |Sᵣ|

MSE_reduction = MSE(parent) − MSE_split
```

The algorithm selects the feature and threshold that **maximise MSE reduction** (equivalently, minimise MSE_split).

### 3.3 Mean Absolute Error (MAE)

An alternative that is more **robust to outliers**:

```
                1     n
MAE(S) =  ─────── × Σ  |yᵢ − median(S)|
               n    i=1
```

When MAE is used, the leaf prediction is the **median** (not the mean).

### 3.4 Friedman MSE

A variation used in gradient boosting that weights by the number of samples in each child:

```
                     |Sₗ| × |Sᵣ|
Friedman_MSE = ──────────────────── × (ȳₗ − ȳᵣ)²
                 |Sₗ| + |Sᵣ|
```

This measures how different the left and right means are, weighted by sample sizes.

### 3.5 Comparison of Criteria

| Criterion | Formula | Leaf Prediction | Outlier Sensitivity | sklearn Parameter |
|-----------|---------|----------------|---------------------|-------------------|
| **MSE** | Variance | Mean | High | `criterion='squared_error'` |
| **MAE** | Median absolute deviation | Median | Low | `criterion='absolute_error'` |
| **Friedman MSE** | Weighted mean difference | Mean | Moderate | `criterion='friedman_mse'` |

---

## 4. Prediction — Leaf Mean

### 4.1 How Prediction Works

For a new sample **x**, the tree traverses from root to leaf, and the prediction is:

```
ŷ = ȳ_leaf = mean of all training targets in the leaf that x reaches

        1
ŷ = ─────── × Σ  yᵢ
     |Leaf|   i∈Leaf
```

### 4.2 Why the Mean?

The mean minimises the MSE within each leaf:

```
argmin  Σ  (yᵢ − c)²  =  ȳ  (the sample mean)
  c    i∈Leaf
```

If using MAE criterion, the **median** minimises the MAE:

```
argmin  Σ  |yᵢ − c|  =  median(y values in leaf)
  c    i∈Leaf
```

### 4.3 Prediction as a Piecewise Constant Function

The full regression tree can be written as:

```
         L
ŷ(x) =  Σ  cₗ × 𝟙(x ∈ Rₗ)
        l=1

where:
  L = number of leaves
  Rₗ = the region of feature space corresponding to leaf l
  cₗ = mean of training targets in leaf l
  𝟙(·) = indicator function (1 if true, 0 otherwise)
```

---

## 5. Step-by-Step Regression Tree Example

### 5.1 Dataset

Predict **house price** ($K) from **area** (sqft):

| Sample | Area (sqft) | Price ($K) |
|--------|-------------|------------|
| 1 | 600 | 150 |
| 2 | 800 | 180 |
| 3 | 1000 | 200 |
| 4 | 1200 | 210 |
| 5 | 1500 | 280 |
| 6 | 1800 | 310 |
| 7 | 2000 | 350 |
| 8 | 2500 | 400 |

### 5.2 Step 1 — Root Node

All 8 samples at root.

```
ȳ = (150+180+200+210+280+310+350+400) / 8 = 260.0

MSE(root) = [(150-260)² + (180-260)² + (200-260)² + (210-260)²
           + (280-260)² + (310-260)² + (350-260)² + (400-260)²] / 8
          = [12100 + 6400 + 3600 + 2500 + 400 + 2500 + 8100 + 19600] / 8
          = 55200 / 8
          = 6900.0
```

### 5.3 Step 2 — Evaluate All Split Thresholds

Candidate thresholds (midpoints between sorted values):

| Threshold | Left Samples | Right Samples | MSE_left | MSE_right | MSE_split | MSE_reduction |
|-----------|-------------|---------------|----------|-----------|-----------|---------------|
| 700 | {600} | {800,...,2500} | 0.0 | 5271.4 | 4612.5 | **2287.5** |
| 900 | {600,800} | {1000,...,2500} | 225.0 | 4333.3 | 3298.4 | **3601.6** |
| 1100 | {600,...,1000} | {1200,...,2500} | 422.2 | 3700.0 | 1808.3 | **5091.7** |
| 1350 | {600,...,1200} | {1500,...,2500} | 525.0 | 2075.0 | 1300.0 | **5600.0** |
| 1650 | {600,...,1500} | {1800,...,2500} | 2036.0 | 1400.0 | 1797.5 | **5102.5** |
| 1900 | {600,...,1800} | {2000,2500} | 2688.9 | 625.0 | 2173.4 | **4726.6** |
| 2250 | {600,...,2000} | {2500} | 4040.8 | 0.0 | 3535.7 | **3364.3** |

### 5.4 Step 3 — Select Best Split

**Best split: Area ≤ 1350** with MSE reduction = **5600.0**

```
Left:  {600, 800, 1000, 1200} → prices: {150, 180, 200, 210}
       ȳ_left = 185.0,  MSE_left = 525.0

Right: {1500, 1800, 2000, 2500} → prices: {280, 310, 350, 400}
       ȳ_right = 335.0,  MSE_right = 2075.0
```

### 5.5 Step 4 — Recurse on Left Child

Left child: {150, 180, 200, 210}, MSE = 525.0

| Threshold | MSE_split | MSE_reduction |
|-----------|-----------|---------------|
| 700 (Area ≤ 700) | 200.0 | **325.0** |
| 900 (Area ≤ 900) | 162.5 | **362.5** |
| 1100 (Area ≤ 1100) | 50.0 | **475.0** |

Best: **Area ≤ 1100** → reduction = 475.0

```
Left-Left:  {150, 180, 200} → ȳ = 176.67
Left-Right: {210}           → ȳ = 210.00
```

### 5.6 Step 5 — Recurse on Right Child

Right child: {280, 310, 350, 400}, MSE = 2075.0

Best split evaluation yields: **Area ≤ 1900** → reduction = 1800.0

```
Right-Left:  {280, 310} → ȳ = 295.0
Right-Right: {350, 400} → ȳ = 375.0
```

### 5.7 Final Tree (depth = 2)

```
                 [Area ≤ 1350?]
                /               \
         [Area ≤ 1100?]     [Area ≤ 1900?]
          /          \        /           \
     ȳ=176.67    ȳ=210.0  ȳ=295.0     ȳ=375.0
     {150,180,   {210}    {280,310}   {350,400}
      200}
```

### 5.8 Predictions

| Sample | Area | Leaf Reached | Prediction | Actual | Error |
|--------|------|-------------|------------|--------|-------|
| 1 | 600 | Left-Left | 176.67 | 150 | 26.67 |
| 2 | 800 | Left-Left | 176.67 | 180 | -3.33 |
| 3 | 1000 | Left-Left | 176.67 | 200 | -23.33 |
| 4 | 1200 | Left-Right | 210.00 | 210 | 0.00 |
| 5 | 1500 | Right-Left | 295.00 | 280 | 15.00 |
| 6 | 1800 | Right-Left | 295.00 | 310 | -15.00 |
| 7 | 2000 | Right-Right | 375.00 | 350 | 25.00 |
| 8 | 2500 | Right-Right | 375.00 | 400 | -25.00 |

---

## 6. Piecewise Constant Approximation

### 6.1 What It Looks Like

A regression tree creates a **step function** — it divides the feature space into regions and assigns a constant value to each:

```
Price ($K)
   ↑
400│                                    ┌─────────
   │                                    │ 375.0
350│                                    │
   │                          ┌─────────┘
300│                          │ 295.0
   │                          │
250│                          │
   │              ┌───────────┘
200│   ┌──────────┘ 210.0
   │   │ 176.67
150│   │
   │   │
   └───┴──────────┴───────────┴──────────┴──────── Area
      600   800  1000  1200  1350  1500  1800  2000  2500
                       ↑           ↑           ↑
                    split 1     split 2     split 3
```

### 6.2 Implications

| Property | Effect |
|----------|--------|
| **Within each region** | All predictions are identical (the mean) |
| **At boundaries** | Predictions change abruptly (discontinuous) |
| **Smooth functions** | Requires many splits to approximate well |
| **Adding depth** | More regions → finer approximation → risk of overfitting |

### 6.3 Improving the Approximation

```
Depth 1 (2 regions):           Depth 4 (up to 16 regions):

Price ↑                        Price ↑
      │     ┌──────────              │    ┌┐┌┐ ┌┐
      │─────┘                        │ ┌──┘└┘└─┘└──
      │                              │─┘
      └──────────── Area →           └──────────── Area →
                                     
      Coarse                         Fine (but still step-like)
```

For truly smooth predictions, consider:
- **Ensemble methods** (Random Forest averages many trees → smoother)
- **Gradient Boosting** (adds many shallow trees → effectively smooth)
- **Linear regression** (if the relationship is approximately linear)

---

## 7. Comparison with Linear Regression

### 7.1 Head-to-Head

| Aspect | Regression Tree | Linear Regression |
|--------|----------------|-------------------|
| **Model form** | Piecewise constant (step function) | y = β₀ + β₁x₁ + ... + βₚxₚ |
| **Decision boundary** | Axis-aligned rectangular regions | Hyperplane |
| **Non-linearity** | Captures naturally | Requires feature engineering |
| **Interactions** | Captures naturally | Must add interaction terms |
| **Extrapolation** | ❌ Cannot extrapolate | ✅ Extrapolates (for better or worse) |
| **Smoothness** | Discontinuous (steps) | Smooth (continuous line) |
| **Interpretability** | If-then rules | Coefficients |
| **Overfitting risk** | High (without pruning) | Low (with regularisation) |
| **Feature scaling** | Not required | Required for regularisation |
| **Outlier sensitivity** | MSE criterion: moderate; MAE: low | High (unless robust methods) |

### 7.2 When Each Wins

```
Scenario 1: True relationship is LINEAR         → Linear Regression wins

    y ↑   .  .  .                 Tree tries to approximate with steps:
      │  .    .    .              y ↑    ┌───┐
      │.   .    .    .                │ ┌─┘   └──┐
      │  .    .    .    .             │─┘         └──
      └──────────────── x →           └──────────────── x →

Scenario 2: True relationship is NONLINEAR       → Regression Tree wins

    y ↑       . .                 Linear regression gets the average:
      │     .     .               y ↑      /
      │   .         .                 │    /
      │ .             .               │   /     (misses the curve)
      │.               .              │  /
      └──────────────── x →           └──────────────── x →

Scenario 3: INTERACTIONS between features        → Regression Tree wins

    "Price depends on area, but the effect of area  
     differs depending on the neighbourhood."
    
    Tree naturally captures this via different branches.
    Linear regression needs explicit interaction terms.
```

### 7.3 Combined Approach: Model Trees

Some advanced trees use **linear regression at the leaves** instead of constant predictions. This combines the tree's ability to partition the space with linear regression's smoothness within each region. (Example: M5 model trees.)

---

## 8. DecisionTreeRegressor in sklearn

### 8.1 Key Parameters

```python
from sklearn.tree import DecisionTreeRegressor

reg = DecisionTreeRegressor(
    criterion='squared_error',    # 'squared_error', 'absolute_error', 'friedman_mse'
    max_depth=None,               # Maximum tree depth
    min_samples_split=2,          # Min samples to split a node
    min_samples_leaf=1,           # Min samples per leaf
    max_leaf_nodes=None,          # Max number of leaves
    min_impurity_decrease=0.0,    # Min impurity decrease for a split
    ccp_alpha=0.0,                # Cost-complexity pruning parameter
    random_state=42,
)
```

### 8.2 Key Attributes (After Fitting)

```python
reg.fit(X_train, y_train)

reg.feature_importances_   # MDI-based feature importance
reg.tree_.node_count       # Total nodes
reg.get_depth()            # Maximum depth
reg.get_n_leaves()         # Number of leaves
```

### 8.3 Key Methods

```python
reg.predict(X_new)                    # Predict target values
reg.score(X_test, y_test)             # R² score
reg.cost_complexity_pruning_path(X, y) # Pruning path
```

### 8.4 Evaluation Metrics

```python
from sklearn.metrics import (
    mean_squared_error, mean_absolute_error,
    r2_score, root_mean_squared_error
)

y_pred = reg.predict(X_test)

mse  = mean_squared_error(X_test, y_pred)       # MSE
rmse = root_mean_squared_error(y_test, y_pred)   # RMSE  
mae  = mean_absolute_error(y_test, y_pred)       # MAE
r2   = r2_score(y_test, y_pred)                  # R²
```

---

## 9. Practical Example — House Price Prediction

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.tree import DecisionTreeRegressor, export_text, plot_tree
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import mean_squared_error, r2_score

# ──────────────────────────────────────────────────────
#  1. Generate synthetic housing data
# ──────────────────────────────────────────────────────
np.random.seed(42)
n = 200

area = np.random.uniform(500, 3000, n)
bedrooms = np.random.choice([1, 2, 3, 4, 5], n, p=[0.1, 0.25, 0.35, 0.2, 0.1])
age = np.random.uniform(0, 50, n)

# Non-linear relationship with interaction
price = (
    50
    + 0.12 * area
    + 20 * bedrooms
    - 1.5 * age
    + 0.00005 * area ** 1.3     # non-linearity
    - 0.5 * age * (bedrooms > 3) # interaction
    + np.random.normal(0, 25, n)  # noise
)

X = np.column_stack([area, bedrooms, age])
y = price
feature_names = ['Area (sqft)', 'Bedrooms', 'Age (years)']

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42
)

# ──────────────────────────────────────────────────────
#  2. Fit regression tree vs linear regression
# ──────────────────────────────────────────────────────
# Unpruned tree
tree_full = DecisionTreeRegressor(random_state=42)
tree_full.fit(X_train, y_train)

# Pruned tree
tree_pruned = DecisionTreeRegressor(max_depth=5, min_samples_leaf=10,
                                     random_state=42)
tree_pruned.fit(X_train, y_train)

# Linear regression
lr = LinearRegression()
lr.fit(X_train, y_train)

# ──────────────────────────────────────────────────────
#  3. Evaluate
# ──────────────────────────────────────────────────────
print("=" * 65)
print(f"{'Model':<25} {'Train R²':<12} {'Test R²':<12} {'Test RMSE':<12}")
print("=" * 65)

for name, model in [("Unpruned Tree", tree_full),
                     ("Pruned Tree (d=5)", tree_pruned),
                     ("Linear Regression", lr)]:
    train_r2 = model.score(X_train, y_train)
    test_r2 = model.score(X_test, y_test)
    test_rmse = np.sqrt(mean_squared_error(y_test, model.predict(X_test)))
    print(f"{name:<25} {train_r2:<12.4f} {test_r2:<12.4f} {test_rmse:<12.2f}")

print("=" * 65)

# ──────────────────────────────────────────────────────
#  4. Print tree rules
# ──────────────────────────────────────────────────────
print("\n=== Pruned Tree Rules ===")
print(export_text(tree_pruned, feature_names=feature_names, max_depth=3))

# ──────────────────────────────────────────────────────
#  5. Feature importance
# ──────────────────────────────────────────────────────
print("\n=== Feature Importance (Pruned Tree) ===")
for name, imp in zip(feature_names, tree_pruned.feature_importances_):
    bar = "█" * int(imp * 50)
    print(f"  {name:20s}  {imp:.4f}  {bar}")

# ──────────────────────────────────────────────────────
#  6. Visualise: tree prediction vs linear regression (1D slice)
# ──────────────────────────────────────────────────────
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# 1D: fix bedrooms=3, age=15 → vary area
area_range = np.linspace(500, 3000, 500)
X_plot = np.column_stack([area_range,
                           np.full(500, 3),
                           np.full(500, 15)])

for ax, model, name in zip(axes,
                             [tree_pruned, lr],
                             ['Regression Tree (pruned)', 'Linear Regression']):
    y_plot = model.predict(X_plot)
    ax.scatter(X_test[:, 0], y_test, alpha=0.3, s=15, label='Test data')
    ax.plot(area_range, y_plot, 'r-', linewidth=2,
            label=f'{name} prediction')
    ax.set_xlabel('Area (sqft)')
    ax.set_ylabel('Price ($K)')
    ax.set_title(name)
    ax.legend()

plt.suptitle('Regression Tree vs Linear Regression (Bedrooms=3, Age=15)',
             fontsize=13)
plt.tight_layout()
plt.savefig('regression_tree_vs_linear.png', dpi=150)
plt.show()

# ──────────────────────────────────────────────────────
#  7. Plot tree graphically
# ──────────────────────────────────────────────────────
fig, ax = plt.subplots(figsize=(20, 10))
plot_tree(tree_pruned, feature_names=feature_names, filled=True,
          rounded=True, fontsize=8, ax=ax)
ax.set_title('Regression Tree (max_depth=5)', fontsize=14)
plt.tight_layout()
plt.savefig('regression_tree_structure.png', dpi=150)
plt.show()

# ──────────────────────────────────────────────────────
#  8. Cross-validation comparison
# ──────────────────────────────────────────────────────
print("\n=== 5-Fold Cross-Validation R² ===")
for name, model in [("Tree (d=3)", DecisionTreeRegressor(max_depth=3, random_state=42)),
                     ("Tree (d=5)", DecisionTreeRegressor(max_depth=5, random_state=42)),
                     ("Tree (d=10)", DecisionTreeRegressor(max_depth=10, random_state=42)),
                     ("Tree (full)", DecisionTreeRegressor(random_state=42)),
                     ("Linear Reg", LinearRegression())]:
    scores = cross_val_score(model, X, y, cv=5, scoring='r2')
    print(f"  {name:15s}  R² = {scores.mean():.4f} ± {scores.std():.4f}")

# ──────────────────────────────────────────────────────
#  9. Demonstrate extrapolation failure
# ──────────────────────────────────────────────────────
print("\n=== Extrapolation Test ===")
X_extrap = np.array([[4000, 4, 5]])  # Area beyond training range
tree_pred = tree_pruned.predict(X_extrap)[0]
lr_pred = lr.predict(X_extrap)[0]
print(f"  Predicting for Area=4000, Bedrooms=4, Age=5:")
print(f"    Tree prediction:   ${tree_pred:.1f}K  (capped at training range)")
print(f"    Linear prediction: ${lr_pred:.1f}K  (extrapolates)")
print(f"  → Tree CANNOT extrapolate beyond the range of training data!")
```

**Expected output pattern:**

```
=================================================================
Model                     Train R²     Test R²      Test RMSE   
=================================================================
Unpruned Tree             1.0000       0.85xx       xx.xx       
Pruned Tree (d=5)         0.96xx       0.92xx       xx.xx       
Linear Regression         0.90xx       0.89xx       xx.xx       
=================================================================

=== Extrapolation Test ===
  Predicting for Area=4000, Bedrooms=4, Age=5:
    Tree prediction:   $xxx.xK  (capped at training range)
    Linear prediction: $xxx.xK  (extrapolates)
  → Tree CANNOT extrapolate beyond the range of training data!
```

---

## 10. Applications

| Domain | Regression Tree Application |
|--------|----------------------------|
| **Real estate** | Predicting house prices from features (area, location, age) |
| **Finance** | Forecasting stock returns, option pricing |
| **Energy** | Predicting electricity demand based on weather and time |
| **Healthcare** | Estimating hospital length of stay |
| **Environmental** | Predicting air quality index from meteorological data |
| **Insurance** | Estimating claim amounts |
| **Retail** | Predicting sales revenue per store |

---

## 11. Summary Table

| Concept | Key Points |
|---------|-----------|
| **Regression tree** | Predicts a continuous target; leaves hold mean values |
| **MSE criterion** | Minimise variance within nodes; default in sklearn |
| **MAE criterion** | More robust to outliers; leaf predicts median |
| **Leaf prediction** | Mean (MSE) or median (MAE) of training targets |
| **Piecewise constant** | Tree output is a step function; not smooth |
| **vs Linear regression** | Tree: non-linear, no extrapolation. Linear: smooth, extrapolates |
| **Overfitting** | Unpruned tree → R²_train ≈ 1.0 but poor test R² |
| **Pruning** | max_depth, min_samples_leaf, ccp_alpha → better generalisation |
| **Feature importance** | Same MDI method as classification trees |
| **Extrapolation** | ❌ Tree cannot predict beyond the range of training data |
| **sklearn class** | `DecisionTreeRegressor(criterion='squared_error')` |
| **Ensemble extension** | Random Forest Regressor, Gradient Boosting Regressor |

---

## 12. Revision Questions

1. **Explain** how a regression tree makes predictions. What value does each leaf node hold, and why?

2. **Compute** the MSE for a node containing target values {100, 120, 140, 160}. Show your working.

3. A node with targets {10, 20, 30, 40, 50} is split into Left = {10, 20} and Right = {30, 40, 50}. **Calculate** the MSE reduction.

4. **Why** can't regression trees extrapolate? Illustrate with a simple 1D example.

5. **Compare** the piecewise constant output of a regression tree with the continuous output of linear regression. In which scenario does each perform better?

6. You fit a `DecisionTreeRegressor` with default parameters and get Train R² = 1.00 and Test R² = 0.65. **Diagnose** the problem and suggest three specific solutions.

---

## 13. Navigation

| Previous | Current | Next |
|----------|---------|------|
| [← 7.6 Feature Importance](06-feature-importance.md) | **7.7 Decision Trees for Regression** | — (End of Unit 7) |

| Module | Link |
|--------|------|
| 7.1 | [Tree Structure & Terminology](01-tree-structure-terminology.md) |
| 7.2 | [Splitting Criteria](02-splitting-criteria.md) |
| 7.3 | [Building a Decision Tree](03-building-decision-tree.md) |
| 7.4 | [Pruning Techniques](04-pruning-techniques.md) |
| 7.5 | [Advantages & Limitations](05-advantages-and-limitations.md) |
| 7.6 | [Feature Importance](06-feature-importance.md) |
| 7.7 | **Decision Trees for Regression** (you are here) |

---

> *"A regression tree doesn't draw a line through the data — it draws a staircase."*
