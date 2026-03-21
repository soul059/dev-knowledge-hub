# Chapter 8.6: XGBoost, LightGBM & CatBoost

> **Unit 8: Ensemble Methods** — File 6 of 8

---

## Table of Contents

1. [Introduction — The Modern GBM Ecosystem](#1-introduction--the-modern-gbm-ecosystem)
2. [XGBoost (eXtreme Gradient Boosting)](#2-xgboost-extreme-gradient-boosting)
3. [LightGBM (Light Gradient Boosting Machine)](#3-lightgbm-light-gradient-boosting-machine)
4. [CatBoost (Categorical Boosting)](#4-catboost-categorical-boosting)
5. [Head-to-Head Comparison](#5-head-to-head-comparison)
6. [Installation & Basic Usage](#6-installation--basic-usage)
7. [Hyperparameter Tuning Guide](#7-hyperparameter-tuning-guide)
8. [Applications](#8-applications)
9. [Summary Table](#9-summary-table)
10. [Revision Questions](#10-revision-questions)

---

## 1. Introduction — The Modern GBM Ecosystem

While sklearn's `GradientBoostingClassifier` implements the original Friedman algorithm, three libraries have revolutionized gradient boosting with engineering optimizations and algorithmic innovations:

```
                    Gradient Boosting (Friedman, 1999)
                              │
              ┌───────────────┼───────────────┐
              │               │               │
         ┌────┴────┐    ┌────┴────┐    ┌─────┴─────┐
         │ XGBoost │    │LightGBM │    │  CatBoost  │
         │ (2014)  │    │ (2017)  │    │  (2017)    │
         │ Chen &  │    │Microsoft│    │  Yandex    │
         │ Guestrin │    │Research │    │           │
         └─────────┘    └─────────┘    └───────────┘
         
         Regularized     Fastest         Best with
         objective,      training,       categorical
         first to        leaf-wise       features,
         popularize      growth          ordered
         GBM at scale                    boosting
```

---

## 2. XGBoost (eXtreme Gradient Boosting)

### Key Innovations

#### 2.1 Regularized Objective Function

XGBoost adds **explicit regularization** to the objective:

```
Obj(t) = Σ L(yᵢ, ŷᵢ⁽ᵗ⁾) + Σ Ω(fₖ)
         i=1                k=1
         ─────────────      ──────────
         Training loss      Regularization

Where:
  Ω(f) = γT + ½λ Σ wⱼ²
                  j=1

  T = number of leaves in tree
  wⱼ = weight (prediction value) of leaf j
  γ = penalty for number of leaves (complexity cost)
  λ = L2 regularization on leaf weights
```

This is different from sklearn's GBM which has NO explicit regularization on tree structure.

#### 2.2 Second-Order Approximation

XGBoost uses a **second-order Taylor expansion** of the loss:

```
Standard GBM:  Uses first derivative (gradient) only
XGBoost:       Uses first AND second derivatives

L(yᵢ, ŷᵢ⁽ᵗ⁻¹⁾ + fₜ(xᵢ)) ≈ L(yᵢ, ŷᵢ⁽ᵗ⁻¹⁾) + gᵢfₜ(xᵢ) + ½hᵢfₜ²(xᵢ)

Where:
  gᵢ = ∂L/∂ŷ           (first derivative = gradient)
  hᵢ = ∂²L/∂ŷ²         (second derivative = Hessian)

Optimal leaf weight for leaf j:
  wⱼ* = -Σᵢ∈ⱼ gᵢ / (Σᵢ∈ⱼ hᵢ + λ)

Optimal split gain:
  Gain = ½ [G_L²/(H_L+λ) + G_R²/(H_R+λ) - (G_L+G_R)²/(H_L+H_R+λ)] - γ

  Where G = Σgᵢ, H = Σhᵢ for left (L) and right (R) children
```

#### 2.3 Tree Pruning

```
sklearn GBM:  Pre-pruning only (max_depth limits growth)

XGBoost:      Post-pruning with the Gain formula
              • Grows tree to max_depth
              • Prunes back nodes where Gain < 0
              • γ parameter controls pruning aggressiveness

              ┌───┐
              │   │  Gain = +5.2 ✓ Keep
              └─┬─┘
               / \
          ┌───┐   ┌───┐
          │   │   │   │  Gain = -1.3 ✗ Prune!
          └─┬─┘   └───┘
           / \
      ┌───┐   ┌───┐
      │   │   │   │  Gain = +0.5 ✓ Keep (but parent pruned)
      └───┘   └───┘
```

#### 2.4 Column Subsampling

XGBoost supports feature subsampling at three levels:

```
colsample_bytree:   Random features per TREE
colsample_bylevel:  Random features per DEPTH LEVEL
colsample_bynode:   Random features per SPLIT (like Random Forest)

These can be combined:
  If colsample_bytree=0.8 and colsample_bylevel=0.8:
  At depth 0: 80% of features available
  At depth 1: 80% of those → 64% of original features
```

#### 2.5 System Optimizations

```
┌──────────────────────────────────────────────────────────┐
│  XGBoost System-Level Optimizations                      │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  • Parallel tree construction (within each tree)         │
│    - Split finding parallelized across features          │
│    - NOT parallel across trees (still sequential)        │
│                                                          │
│  • Cache-aware access patterns                           │
│    - Sorted data stored in compressed column format      │
│    - Reduces cache misses during split finding           │
│                                                          │
│  • Out-of-core computation                               │
│    - Can handle data larger than RAM                     │
│    - Block-based data storage on disk                    │
│                                                          │
│  • Sparsity-aware algorithm                              │
│    - Handles missing values natively                     │
│    - Learns optimal direction for missing values         │
│                                                          │
│  • Weighted quantile sketch                              │
│    - Approximate split finding for large datasets        │
│    - Distributed version for multi-machine training      │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### XGBoost Missing Value Handling

```
At each split node, XGBoost learns whether to send
missing values LEFT or RIGHT:

        ┌──────────┐
        │ Feature A │
        │   ≤ 5 ?   │
        └────┬─────┘
            / \
      Yes  /   \  No
          /     \
    ┌────┐       ┌────┐
    │Left│       │Right│
    └────┘       └────┘
        ↑
    Missing values go HERE
    (XGBoost tried both directions and this was better)
```

---

## 3. LightGBM (Light Gradient Boosting Machine)

### Key Innovations

#### 3.1 Leaf-Wise (Best-First) Tree Growth

The most important difference from XGBoost:

```
Level-Wise Growth (XGBoost default):
══════════════════════════════════════
  Grows ALL nodes at the same depth before going deeper.
  Balanced, but wastes computation on low-gain splits.

  Depth 0:        ┌───────┐
                   │ Split │
                   └───┬───┘
                      / \
  Depth 1:    ┌─────┐   ┌─────┐     ← Both nodes split
              │Split│   │Split│        regardless of gain
              └──┬──┘   └──┬──┘
                / \       / \
  Depth 2:  ┌──┐ ┌──┐ ┌──┐ ┌──┐     ← All 4 nodes split
            │  │ │  │ │  │ │  │
            └──┘ └──┘ └──┘ └──┘


Leaf-Wise Growth (LightGBM):
══════════════════════════════
  Always splits the leaf with the HIGHEST gain.
  More efficient — focuses computation where it matters.

  Step 1:         ┌───────┐
                   │ Split │
                   └───┬───┘
                      / \
  Step 2:    ┌─────┐   ┌─────┐
              │Split│   │     │     ← Only the HIGH-GAIN
              └──┬──┘   └─────┘       leaf is split!
                / \
  Step 3:  ┌──┐ ┌─────┐
           │  │ │Split│              ← Next highest gain leaf
           └──┘ └──┬──┘
                  / \
  Step 4:      ┌──┐ ┌──┐
               │  │ │  │
               └──┘ └──┘

  Result: DEEPER tree on one side, SHALLOWER on another
          More accurate with fewer leaves
```

**Why Leaf-Wise is faster:**
- For the same number of leaves, leaf-wise achieves lower loss
- Or equivalently: reaches the same loss with fewer leaves
- But: more prone to overfitting on small datasets → use `max_depth` or `num_leaves` to control

#### 3.2 GOSS (Gradient-based One-Side Sampling)

```
Key insight: Samples with LARGE gradients contribute more
to the information gain. Keep all of them, but subsample
the ones with small gradients.

Algorithm:
  1. Sort samples by absolute gradient |gᵢ|
  2. Keep top a% of samples (large gradients)
  3. Randomly sample b% of remaining samples (small gradients)
  4. Multiply small-gradient samples by (1-a)/b to maintain
     the correct data distribution

Example (1000 samples, a=20%, b=10%):
  • Keep 200 samples with largest |gᵢ| (all of them)
  • Randomly sample 80 from remaining 800
  • Weight the 80 samples by (1-0.2)/0.1 = 8.0
  • Train tree on 280 samples instead of 1000 (3.6× faster!)
  • Nearly identical accuracy to using all 1000

┌─────────────────────────────────────────────────────────────┐
│  Gradient magnitude: |████████████████████░░░░░░░░░░░░░░░│ │
│                       ▲ Top 20%: KEEP ALL    ▲ Bottom 80% │ │
│                                                Sample 10%  │
└─────────────────────────────────────────────────────────────┘
```

#### 3.3 EFB (Exclusive Feature Bundling)

```
Observation: In sparse datasets, many features are EXCLUSIVE
(they rarely have non-zero values simultaneously).

Example: One-hot encoded features
  Feature_A: [1, 0, 0, 0, 1, 0, ...]
  Feature_B: [0, 1, 0, 0, 0, 0, ...]
  Feature_C: [0, 0, 1, 0, 0, 1, ...]
  
  These are mutually exclusive! They can be BUNDLED into one feature.

Before EFB: 1000 sparse features → 1000 split candidates per node
After EFB:  Bundle into ~50 dense features → 50 split candidates

Result: Dramatic speedup on high-dimensional sparse data
        (common in NLP, click prediction, etc.)
```

#### 3.4 Histogram-Based Split Finding

```
Exact split finding (XGBoost default):
  Sort all N samples by each feature → O(N·p·log N)
  Try every possible threshold
  Very accurate but SLOW for large N

Histogram-based (LightGBM):
  Bucket continuous features into ~255 bins → O(N·p)
  Only try 255 thresholds instead of N
  Much faster with minimal accuracy loss

  Feature values: [0.1, 0.3, 0.5, 0.8, 1.2, 1.5, 2.0, 3.1, ...]
                   ↓ Bin into histogram
  Bins:          [  Bin 0  |  Bin 1  |  Bin 2  |  Bin 3  | ...]
                  [0,0.5)    [0.5,1.0)  [1.0,2.0)  [2.0,4.0)
  
  Instead of N thresholds → only 255 thresholds to evaluate!
  
  XGBoost later adopted this too ("tree_method='hist'")
```

---

## 4. CatBoost (Categorical Boosting)

### Key Innovations

#### 4.1 Ordered Boosting (Combating Target Leakage)

Standard gradient boosting has a subtle issue: **prediction shift**.

```
The Problem:
  When computing residuals, we use the same data that was used
  to train previous trees. This causes a subtle overfitting bias.

  Round 1: Train h₁ on ALL data → compute residuals on ALL data
  Round 2: Train h₂ on residuals → but residuals computed using
           models that already SAW this data!
           
  This is like computing "training error" and treating it as
  "test error" — biased!

CatBoost's Solution — Ordered Boosting:
  1. Randomly permute the training data
  2. For sample i, compute the model using ONLY samples 1...(i-1)
  3. Sample i's residual is computed from a model that NEVER saw sample i
  
  Example (n=5 samples):
    Sample 1: Model trained on {} (no data) → use prior
    Sample 2: Model trained on {1} → predict for 2
    Sample 3: Model trained on {1,2} → predict for 3
    Sample 4: Model trained on {1,2,3} → predict for 4
    Sample 5: Model trained on {1,2,3,4} → predict for 5
    
  This eliminates target leakage! (at some computational cost)
```

#### 4.2 Native Categorical Feature Handling

```
Standard approach (manual):
  Categorical feature "City" → One-hot encoding or Label encoding
  
  Problem with one-hot: Curse of dimensionality (1000 cities = 1000 features)
  Problem with label:   Imposes artificial ordering

CatBoost's approach — Ordered Target Statistics:
  For categorical feature with value v and sample i:
  
  TS(xᵢ = v) = (Σⱼ<ᵢ [xⱼ=v] · yⱼ + prior) / (Σⱼ<ᵢ [xⱼ=v] + 1)
  
  Translation: "Average target value for this category,
                computed using ONLY samples that appear
                BEFORE sample i in the random permutation"
  
  This avoids target leakage (ordered computation)
  and handles high-cardinality categoricals naturally!

Example:
  Data (ordered by permutation):
  i=1: City=NYC,   y=10  → TS = (0 + prior)/(0 + 1) = prior
  i=2: City=LA,    y=20  → TS = (0 + prior)/(0 + 1) = prior
  i=3: City=NYC,   y=15  → TS = (10 + prior)/(1 + 1)
  i=4: City=NYC,   y=12  → TS = (10+15 + prior)/(2 + 1)
  i=5: City=LA,    y=25  → TS = (20 + prior)/(1 + 1)
```

#### 4.3 Symmetric (Oblivious) Trees

```
Standard Decision Trees:           CatBoost Oblivious Trees:
  Different splits at each node     SAME split at each depth level

       ┌───┐                              ┌───┐
       │f₃>2│                             │f₃>2│
       └─┬──┘                             └─┬──┘
        / \                                / \
   ┌───┐   ┌───┐                     ┌───┐   ┌───┐
   │f₁>5│  │f₇>3│  ← Different!      │f₁>5│  │f₁>5│  ← SAME!
   └─┬──┘  └─┬──┘                    └─┬──┘  └─┬──┘
    / \     / \                        / \     / \
  [A] [B] [C] [D]                   [A] [B] [C] [D]

Benefits of oblivious trees:
  • Faster prediction (can use bitwise operations)
  • Built-in regularization (fewer distinct split conditions)
  • Robust to overfitting
  • Very fast inference — important for production
```

---

## 5. Head-to-Head Comparison

### Feature Comparison Table

```
┌──────────────────────┬──────────────┬──────────────┬──────────────┐
│     Feature          │   XGBoost    │   LightGBM   │   CatBoost   │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ Tree Growth          │ Level-wise   │ Leaf-wise    │ Symmetric    │
│                      │ (or hist)    │ (best-first) │ (oblivious)  │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ Split Finding        │ Exact or     │ Histogram    │ Histogram    │
│                      │ Histogram    │ (always)     │              │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ Categorical Support  │ Experimental │ Built-in     │ Native       │
│                      │ (recent)     │ (label enc.) │ (ordered TS) │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ Missing Values       │ Native       │ Native       │ Native       │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ Regularization       │ L1 + L2 +    │ L1 + L2 +    │ Ordered      │
│                      │ γ (tree)     │ num_leaves   │ boosting     │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ Training Speed       │ Fast         │ Fastest      │ Moderate     │
│ (large data)         │              │ (2-10×)      │              │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ Prediction Speed     │ Fast         │ Fast         │ Fastest      │
│                      │              │              │ (oblivious)  │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ Overfitting Control  │ Good         │ Needs tuning │ Best default │
│                      │              │ (leaf-wise   │ (ordered     │
│                      │              │  can overfit)│  boosting)   │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ GPU Support          │ Yes          │ Yes          │ Yes          │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ Distributed          │ Yes          │ Yes          │ Yes          │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ Language Support     │ Python, R,   │ Python, R,   │ Python, R,   │
│                      │ Java, C++,   │ C++, Java    │ C++, Java    │
│                      │ Julia, etc.  │              │              │
├──────────────────────┼──────────────┼──────────────┼──────────────┤
│ First Release        │ 2014         │ 2017         │ 2017         │
└──────────────────────┴──────────────┴──────────────┴──────────────┘
```

### When to Use Each

```
Choose XGBoost when:
  ✓ You want the most widely used, battle-tested library
  ✓ You need extensive documentation and community support
  ✓ Medium-sized datasets (up to millions of rows)
  ✓ You want fine-grained control over regularization

Choose LightGBM when:
  ✓ You have LARGE datasets (millions+ rows)
  ✓ Training speed is critical
  ✓ You have high-dimensional or sparse data
  ✓ You're in a competition (often marginally more accurate)

Choose CatBoost when:
  ✓ You have many CATEGORICAL features
  ✓ You want good results with MINIMAL tuning
  ✓ You need fast INFERENCE (prediction) speed
  ✓ You want robust out-of-the-box performance
```

### Speed Benchmark (Typical Results)

```
  Training Time (relative to XGBoost = 1.0)
  on a dataset with 1M rows, 100 features:

  XGBoost:    ████████████████████  1.0×
  LightGBM:   █████                0.25×  (4× faster!)
  CatBoost:   ██████████████       0.7×

  Note: Actual speedup varies with data characteristics
        LightGBM's advantage grows with larger datasets
```

---

## 6. Installation & Basic Usage

### Installation

```bash
# XGBoost
pip install xgboost

# LightGBM
pip install lightgbm

# CatBoost
pip install catboost
```

### XGBoost Example

```python
import xgboost as xgb
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# --- Scikit-Learn API ---
model = xgb.XGBClassifier(
    n_estimators=200,
    max_depth=6,
    learning_rate=0.1,
    subsample=0.8,
    colsample_bytree=0.8,
    reg_alpha=0.1,        # L1 regularization
    reg_lambda=1.0,       # L2 regularization
    gamma=0.1,            # min split gain
    use_label_encoder=False,
    eval_metric='logloss',
    random_state=42
)
model.fit(
    X_train, y_train,
    eval_set=[(X_test, y_test)],
    verbose=False
)
y_pred = model.predict(X_test)
print(f"XGBoost Accuracy: {accuracy_score(y_test, y_pred):.4f}")

# --- Native API (more control) ---
dtrain = xgb.DMatrix(X_train, label=y_train)
dtest = xgb.DMatrix(X_test, label=y_test)

params = {
    'max_depth': 6,
    'eta': 0.1,
    'objective': 'binary:logistic',
    'eval_metric': 'logloss',
    'subsample': 0.8,
    'colsample_bytree': 0.8
}

model_native = xgb.train(
    params, dtrain,
    num_boost_round=200,
    evals=[(dtest, 'test')],
    early_stopping_rounds=20,
    verbose_eval=False
)
y_pred_prob = model_native.predict(dtest)
y_pred_native = (y_pred_prob > 0.5).astype(int)
print(f"XGBoost (native) Accuracy: {accuracy_score(y_test, y_pred_native):.4f}")
```

### LightGBM Example

```python
import lightgbm as lgb
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# --- Scikit-Learn API ---
model = lgb.LGBMClassifier(
    n_estimators=200,
    num_leaves=31,          # max leaves per tree (key param!)
    max_depth=-1,           # no limit (controlled by num_leaves)
    learning_rate=0.1,
    subsample=0.8,
    colsample_bytree=0.8,
    reg_alpha=0.1,
    reg_lambda=1.0,
    min_child_samples=20,   # min samples in leaf
    random_state=42,
    verbose=-1
)
model.fit(
    X_train, y_train,
    eval_set=[(X_test, y_test)],
    callbacks=[lgb.early_stopping(20), lgb.log_evaluation(0)]
)
y_pred = model.predict(X_test)
print(f"LightGBM Accuracy: {accuracy_score(y_test, y_pred):.4f}")

# --- Native API ---
train_data = lgb.Dataset(X_train, label=y_train)
test_data = lgb.Dataset(X_test, label=y_test, reference=train_data)

params = {
    'objective': 'binary',
    'metric': 'binary_logloss',
    'num_leaves': 31,
    'learning_rate': 0.1,
    'verbose': -1
}

model_native = lgb.train(
    params, train_data,
    num_boost_round=200,
    valid_sets=[test_data],
    callbacks=[lgb.early_stopping(20), lgb.log_evaluation(0)]
)
y_pred_prob = model_native.predict(X_test)
y_pred_native = (y_pred_prob > 0.5).astype(int)
print(f"LightGBM (native) Accuracy: {accuracy_score(y_test, y_pred_native):.4f}")
```

### CatBoost Example

```python
from catboost import CatBoostClassifier, Pool
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score

X, y = load_breast_cancer(return_X_y=True)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# --- Basic Usage ---
model = CatBoostClassifier(
    iterations=200,
    depth=6,
    learning_rate=0.1,
    l2_leaf_reg=3.0,       # L2 regularization
    random_seed=42,
    verbose=0              # suppress output
)
model.fit(X_train, y_train, eval_set=(X_test, y_test))
y_pred = model.predict(X_test)
print(f"CatBoost Accuracy: {accuracy_score(y_test, y_pred):.4f}")

# --- With Categorical Features ---
import pandas as pd
import numpy as np

# Simulate data with categorical features
df = pd.DataFrame({
    'age': np.random.randint(20, 70, 1000),
    'city': np.random.choice(['NYC', 'LA', 'Chicago', 'Houston'], 1000),
    'job': np.random.choice(['Engineer', 'Doctor', 'Teacher', 'Artist'], 1000),
    'income': np.random.normal(50000, 15000, 1000)
})
y_sim = (df['income'] > 50000).astype(int)

cat_features = ['city', 'job']  # specify which columns are categorical

train_pool = Pool(
    data=df.drop('income', axis=1),
    label=y_sim,
    cat_features=cat_features  # CatBoost handles encoding automatically!
)

model_cat = CatBoostClassifier(iterations=100, depth=4, verbose=0)
model_cat.fit(train_pool)
print(f"CatBoost with categoricals: trained successfully!")
print(f"Feature importance: {dict(zip(df.drop('income', axis=1).columns, model_cat.feature_importances_))}")
```

---

## 7. Hyperparameter Tuning Guide

### XGBoost Hyperparameters

```python
# Most important XGBoost hyperparameters (in order of importance)
xgb_params = {
    # === Learning ===
    'learning_rate': 0.1,        # [0.01, 0.3] — start here
    'n_estimators': 500,         # [100, 5000] — use early stopping
    
    # === Tree Structure ===
    'max_depth': 6,              # [3, 10] — most important tree param
    'min_child_weight': 1,       # [1, 10] — like min_samples_leaf
    
    # === Regularization ===
    'gamma': 0,                  # [0, 5] — min loss reduction for split
    'reg_alpha': 0,              # [0, 1] — L1 regularization
    'reg_lambda': 1,             # [0, 10] — L2 regularization
    
    # === Sampling ===
    'subsample': 0.8,            # [0.5, 1.0]
    'colsample_bytree': 0.8,    # [0.5, 1.0]
}
```

### LightGBM Hyperparameters

```python
# Most important LightGBM hyperparameters
lgb_params = {
    # === Learning ===
    'learning_rate': 0.1,        # [0.01, 0.3]
    'n_estimators': 500,         # [100, 5000]
    
    # === Tree Structure ===
    'num_leaves': 31,            # [15, 255] — MOST IMPORTANT!
    'max_depth': -1,             # [-1, 15] — -1 means no limit
    'min_child_samples': 20,     # [5, 100]
    
    # === Regularization ===
    'reg_alpha': 0,              # [0, 1]
    'reg_lambda': 0,             # [0, 1]
    'min_split_gain': 0,         # [0, 1]
    
    # === Sampling ===
    'subsample': 0.8,            # [0.5, 1.0]
    'colsample_bytree': 0.8,    # [0.5, 1.0]
    
    # === Speed ===
    'max_bin': 255,              # [63, 512]
}
```

### CatBoost Hyperparameters

```python
# Most important CatBoost hyperparameters
cat_params = {
    # === Learning ===
    'learning_rate': 0.1,        # [0.01, 0.3] — auto if not set
    'iterations': 500,           # [100, 5000]
    
    # === Tree Structure ===
    'depth': 6,                  # [4, 10]
    'min_data_in_leaf': 1,       # [1, 50]
    
    # === Regularization ===
    'l2_leaf_reg': 3.0,          # [1, 10]
    'random_strength': 1.0,     # [0, 10] — randomness in scoring
    'bagging_temperature': 1.0, # [0, 10] — Bayesian bootstrap
    
    # === Categorical ===
    'one_hot_max_size': 25,     # max categories for one-hot
}
```

### Universal Tuning Strategy

```
Step 1: Fix learning_rate = 0.1, find optimal n_estimators
        Use early stopping to automatically determine n_estimators

Step 2: Tune tree-specific parameters
        XGBoost: max_depth, min_child_weight
        LightGBM: num_leaves, min_child_samples
        CatBoost: depth, min_data_in_leaf

Step 3: Tune regularization
        reg_alpha, reg_lambda, gamma (XGBoost)
        
Step 4: Tune sampling parameters
        subsample, colsample_bytree

Step 5: Lower learning_rate (0.01-0.05) and increase n_estimators
        This often gives a final boost in accuracy

Optuna Example:
```

```python
import optuna
from sklearn.model_selection import cross_val_score

def objective(trial):
    params = {
        'n_estimators': 500,
        'learning_rate': trial.suggest_float('learning_rate', 0.01, 0.3, log=True),
        'max_depth': trial.suggest_int('max_depth', 3, 10),
        'subsample': trial.suggest_float('subsample', 0.5, 1.0),
        'colsample_bytree': trial.suggest_float('colsample_bytree', 0.5, 1.0),
        'reg_alpha': trial.suggest_float('reg_alpha', 1e-8, 10.0, log=True),
        'reg_lambda': trial.suggest_float('reg_lambda', 1e-8, 10.0, log=True),
        'gamma': trial.suggest_float('gamma', 1e-8, 5.0, log=True),
        'min_child_weight': trial.suggest_int('min_child_weight', 1, 10),
        'random_state': 42,
        'use_label_encoder': False,
        'eval_metric': 'logloss'
    }
    
    model = xgb.XGBClassifier(**params)
    scores = cross_val_score(model, X_train, y_train, cv=5, scoring='accuracy')
    return scores.mean()

study = optuna.create_study(direction='maximize')
study.optimize(objective, n_trials=100)
print(f"Best accuracy: {study.best_value:.4f}")
print(f"Best params: {study.best_params}")
```

---

## 8. Applications

| Domain | Preferred Library | Why |
|--------|------------------|-----|
| **Kaggle Competitions** | XGBoost / LightGBM | Consistently top-performing |
| **Ad Click Prediction** | LightGBM | Speed on billions of impressions |
| **Search Ranking** | LambdaMART (XGBoost) | Pairwise ranking loss |
| **Fraud Detection** | CatBoost | Many categorical features |
| **Recommendation Systems** | LightGBM | Speed + high-dimensional sparse features |
| **Financial Modeling** | XGBoost | Robust, well-understood |
| **Healthcare** | CatBoost | Handles medical codes (categorical) natively |
| **NLP Feature-Based** | LightGBM | Fast on high-dimensional TF-IDF features |

---

## 9. Summary Table

| Aspect | XGBoost | LightGBM | CatBoost |
|--------|---------|----------|----------|
| **Developer** | DMLC (Chen & Guestrin) | Microsoft Research | Yandex |
| **Year** | 2014 | 2017 | 2017 |
| **Tree Growth** | Level-wise (default) | Leaf-wise | Symmetric/Oblivious |
| **Key Innovation** | Regularized objective | GOSS + EFB | Ordered boosting |
| **Categorical Handling** | Manual encoding | Label encoding | Native ordered TS |
| **Training Speed** | Fast | Fastest | Moderate |
| **Inference Speed** | Fast | Fast | Fastest |
| **Default Performance** | Good | Good | Best |
| **Tuning Needed** | Moderate | More | Least |
| **Missing Values** | Native | Native | Native |
| **GPU Support** | Yes | Yes | Yes |
| **Best For** | General purpose | Large data, speed | Categorical-heavy data |

---

## 10. Revision Questions

**Q1.** Explain the mathematical difference between XGBoost's regularized objective and standard gradient boosting. How does the regularization term affect the optimal leaf weights?

<details>
<summary>Answer</summary>

**Standard GBM objective:**
Obj = Σᵢ L(yᵢ, ŷᵢ)

**XGBoost objective:**
Obj = Σᵢ L(yᵢ, ŷᵢ) + γT + ½λΣⱼ wⱼ²

Where T = number of leaves, wⱼ = leaf weight, γ = leaf penalty, λ = L2 penalty.

**Effect on optimal leaf weights:**
Without regularization: wⱼ* = -Gⱼ/Hⱼ (where G = sum of gradients, H = sum of Hessians)
With regularization: wⱼ* = -Gⱼ/(Hⱼ + λ)

The λ term in the denominator **shrinks** leaf weights toward zero, preventing any single leaf from having extreme predictions. The γT term discourages having too many leaves. Together, they prevent overfitting more effectively than depth limiting alone.
</details>

**Q2.** Compare level-wise (XGBoost) and leaf-wise (LightGBM) tree growth strategies. Why might leaf-wise overfit on small datasets?

<details>
<summary>Answer</summary>

**Level-wise:** Splits ALL nodes at the current depth before moving to the next level. This creates balanced trees and ensures each depth level is fully developed before going deeper.

**Leaf-wise:** Always splits the leaf with the highest gain, regardless of depth. This creates asymmetric trees that can be very deep on one side.

**Why leaf-wise overfits on small data:**
1. Leaf-wise trees can grow very deep, creating specialized leaves with very few samples
2. On small datasets, the "highest gain" leaf might represent noise rather than signal
3. Deep, asymmetric trees have more parameters relative to data size
4. Level-wise growth implicitly regularizes by keeping trees balanced

**Mitigation:** LightGBM uses `num_leaves` (not `max_depth`) as the primary constraint. Setting `num_leaves=31` limits the total leaves regardless of depth. For small datasets, use `num_leaves ≤ 2^max_depth` (e.g., 31 ≤ 32 = 2⁵).
</details>

**Q3.** Explain GOSS (Gradient-based One-Side Sampling) in LightGBM. Why can we safely undersample low-gradient instances?

<details>
<summary>Answer</summary>

**GOSS Algorithm:**
1. Sort instances by absolute gradient |gᵢ|
2. Keep top a% (large gradients) → these contribute most to information gain
3. Randomly sample b% of remaining → these contribute less
4. Multiply sampled instances' weights by (1-a)/b to maintain distribution

**Why undersampling low-gradient instances is safe:**

Information gain for a split depends on the gradient sum. Instances with **large gradients** (|gᵢ|) are:
- Far from the current model's prediction → they're the "hard" examples
- They contribute most to the gradient sum → most influence on split decisions
- Removing them would significantly change the optimal split

Instances with **small gradients** are:
- Close to the current model's prediction → already well-modeled
- They contribute little to the gradient sum
- Removing a fraction of them barely changes the optimal split
- Compensating with a weight multiplier (1-a)/b maintains the expected gradient sum

Result: ~3-4× speedup with negligible accuracy loss.
</details>

**Q4.** How does CatBoost's ordered target statistic handle categorical features differently from one-hot encoding and label encoding? What is the target leakage problem it solves?

<details>
<summary>Answer</summary>

**One-hot encoding:** Creates a binary column per category. Problems: dimensionality explosion with high-cardinality features, doesn't capture target-feature relationship.

**Label encoding:** Assigns integers (0, 1, 2, ...) to categories. Problem: implies ordinal relationship where none exists (is "NYC" > "LA"?).

**Ordered Target Statistics (CatBoost):**
Replaces each categorical value with the average target value for that category, computed using only preceding samples in a random permutation:

TS(xᵢ = v) = (Σⱼ<ᵢ [xⱼ=v]·yⱼ + prior) / (Σⱼ<ᵢ [xⱼ=v] + 1)

**Target leakage problem:** If you compute "mean target per category" using ALL data, the encoding for sample i uses i's own target value. This creates optimistic estimates — the model sees information it shouldn't during training. It's like including the answer in the question.

CatBoost's ordered approach ensures sample i's encoding uses only samples with index < i in the permutation, completely eliminating target leakage. Multiple random permutations are used for robustness.
</details>

**Q5.** You have a dataset with 10 million rows, 500 features (50 of which are categorical with high cardinality), and need a model deployed for real-time predictions. Which library would you choose and why?

<details>
<summary>Answer</summary>

**CatBoost** is the best choice for this scenario:

1. **Categorical features (50 high-cardinality):** CatBoost handles these natively with ordered target statistics, avoiding the need for manual encoding that would explode the feature space.

2. **Large dataset (10M rows):** While LightGBM is faster for training, CatBoost is still efficient and the difference matters less with a one-time training job.

3. **Real-time predictions:** CatBoost's **oblivious trees** provide the **fastest inference speed** among the three. Oblivious trees can be evaluated using bitwise operations, making prediction ~2-3× faster than regular trees.

4. **Out-of-the-box performance:** CatBoost requires minimal hyperparameter tuning, reducing development time. Ordered boosting provides good generalization without extensive validation.

**Alternative consideration:** If training speed is critical (e.g., frequent retraining), LightGBM would be chosen, with manual categorical encoding. For the deployment constraint (real-time predictions), CatBoost's inference speed advantage is decisive.
</details>

**Q6.** Explain XGBoost's handling of missing values. How does it decide which direction to send missing values at each split?

<details>
<summary>Answer</summary>

**XGBoost's Missing Value Algorithm:**

At each candidate split, XGBoost evaluates TWO scenarios:
1. Send all instances with missing values to the LEFT child
2. Send all instances with missing values to the RIGHT child

It picks whichever direction gives a **higher gain**.

**Detailed process:**
1. When computing the gain for a split threshold θ on feature j:
   - Compute gain assuming missing → left (add their gradients/Hessians to left)
   - Compute gain assuming missing → right (add their gradients/Hessians to right)
   - Choose the direction with higher gain
   - Store this direction as the "default direction" for this split

2. At prediction time:
   - If a sample has a missing value for the split feature
   - Send it in the "default direction" learned during training

**Key insight:** This is a **learned** approach — the model discovers the best treatment for missing values from the data, rather than requiring imputation. Different splits can send missing values in different directions, adapting to the data structure.

This is why XGBoost (and LightGBM/CatBoost) often outperform models that require imputation as preprocessing.
</details>

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← 8.5 Gradient Boosting](./05-gradient-boosting.md) | [Unit 8: Ensemble Methods](../README.md) | [8.7 Stacking →](./07-stacking.md) |

---

*© 2025 AIML Study Notes. Unit 8: Ensemble Methods — Chapter 6: XGBoost, LightGBM & CatBoost.*
