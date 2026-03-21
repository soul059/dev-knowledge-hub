# 🎛️ Chapter 5: Hyperparameter Tuning

> **Module:** 03-Model-Evaluation | **Topic:** Systematic Hyperparameter Optimization
> **Objective:** Understand the difference between parameters and hyperparameters, master Grid Search, Random Search, Bayesian Optimization, and Successive Halving, and apply best practices for efficient tuning.

---

## 5.1 Hyperparameters vs. Model Parameters

| Aspect | Model Parameters | Hyperparameters |
|--------|-----------------|-----------------|
| **Set by** | Learning algorithm (training) | Engineer (before training) |
| **Learned from data?** | ✓ Yes | ✗ No |
| **Examples** | Weights, biases, coefficients | Learning rate, K in KNN, tree depth |
| **Stored in** | Trained model object | Configuration / search space |

```
                ┌───────────────────────────────┐
Hyperparameters │  learning_rate = 0.01         │  ← YOU choose
(before train)  │  max_depth = 5                │
                └──────────────┬────────────────┘
                               ▼
                ┌───────────────────────────────┐
                │      TRAINING ALGORITHM       │
                │  Minimizes loss on data       │
                └──────────────┬────────────────┘
                               ▼
Model Parameters│  w₁ = 0.73, w₂ = -1.24 ...   │  ← ALGORITHM learns
(after train)   │  bias = 0.42                  │
                └───────────────────────────────┘
```

### Concrete Examples

| Algorithm | Hyperparameters | Model Parameters |
|-----------|----------------|------------------|
| Linear Regression | Regularization strength (α) | Weights (w), intercept (b) |
| Decision Tree | max_depth, min_samples_split | Split thresholds, feature indices |
| Random Forest | n_estimators, max_features | Individual tree structures |
| Neural Network | Learning rate, layers, neurons | Weights and biases of each layer |
| KNN | K (neighbors), distance metric | — (non-parametric, stores data) |
| SVM | C, kernel, gamma | Support vectors, dual coefficients |

---

## 5.2 Grid Search (Exhaustive Search)

Grid Search evaluates **every combination** in a predefined parameter grid.
```
Parameter Grid Example:
  max_depth   = [3, 5, 10]           → 3 values
  n_estimators = [50, 100, 200]      → 3 values
  min_samples_split = [2, 5]         → 2 values

Total combinations = 3 × 3 × 2 = 18 fits per CV fold
With 5-fold CV    = 18 × 5 = 90 total model fits
```

### Python — GridSearchCV

```python
from sklearn.model_selection import GridSearchCV
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_iris

X, y = load_iris(return_X_y=True)
model = RandomForestClassifier(random_state=42)

param_grid = {
    'n_estimators': [50, 100, 200],
    'max_depth': [3, 5, 10, None],
    'min_samples_split': [2, 5, 10]
}

grid_search = GridSearchCV(
    estimator=model,
    param_grid=param_grid,
    cv=5,
    scoring='accuracy',
    n_jobs=-1,          # use all CPU cores
    verbose=1
)
grid_search.fit(X, y)

print(f"Best Params : {grid_search.best_params_}")
print(f"Best Score  : {grid_search.best_score_:.4f}")
best_model = grid_search.best_estimator_
```

| Pros | Cons |
|------|------|
| Exhaustive — guaranteed to find the best in the grid | **Exponentially expensive** as dimensions grow |
| Simple to implement and interpret | Wastes budget on unimportant parameters |
| Reproducible results | Cannot explore outside predefined values |

---

## 5.3 Random Search

Instead of evaluating every combination, Random Search **samples random configurations** from defined distributions. Bergstra & Bengio (2012) showed it often outperforms Grid Search with the same computational budget.

```
Grid Search (9 trials)          Random Search (9 trials)
┌───┬───┬───┐                   ┌─────────────────────┐
│ × │ × │ × │                   │  ×    ×         ×   │
├───┼───┼───┤                   │       ×    ×        │
│ × │ × │ × │                   │ ×          ×    ×   │
├───┼───┼───┤                   │      ×          ×   │
│ × │ × │ × │                   └─────────────────────┘
└───┴───┴───┘
Only 3 unique values            9 unique values per
per dimension explored          dimension explored!

If only one parameter matters, Random Search covers it better.
```

### Python — RandomizedSearchCV

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import randint, uniform

param_distributions = {
    'n_estimators': randint(50, 500),        # uniform integer [50, 500)
    'max_depth': randint(3, 30),             # uniform integer [3, 30)
    'min_samples_split': randint(2, 20),
    'max_features': uniform(0.1, 0.9),       # continuous [0.1, 1.0)
}

random_search = RandomizedSearchCV(
    estimator=RandomForestClassifier(random_state=42),
    param_distributions=param_distributions,
    n_iter=50,           # number of random samples
    cv=5,
    scoring='accuracy',
    random_state=42,
    n_jobs=-1
)
random_search.fit(X, y)

print(f"Best Params : {random_search.best_params_}")
print(f"Best Score  : {random_search.best_score_:.4f}")
```

| Pros | Cons |
|------|------|
| More efficient for high-dimensional spaces | Not exhaustive — may miss the optimum |
| Can sample from continuous distributions | Results vary with random seed |
| Easy to control budget via `n_iter` | No intelligent exploration strategy |

---

## 5.4 Bayesian Optimization

Bayesian Optimization builds a **surrogate model** (typically a Gaussian Process) of the objective function and uses an **acquisition function** to decide which configuration to try next.

```
  ┌───────────────────────────────────────────────┐
  │ 1. Evaluate a few random initial points        │
  │ 2. Fit surrogate model (GP) to observations    │
  │ 3. Acquisition function picks next best point  │──┐
  │ 4. Evaluate that point (train + CV score)      │  │
  │ 5. Update surrogate with new observation       │  │
  └───────────────────────────┬───────────────────┘  │
                              └──────────────────────┘
                              Repeat until budget exhausted
```

**Key Components:**

| Component | Role | Common Choice |
|-----------|------|---------------|
| Surrogate Model | Approximates the true objective | Gaussian Process, TPE |
| Acquisition Function | Balances exploration vs. exploitation | Expected Improvement (EI) |

> **Libraries:** `scikit-optimize` (skopt), `Optuna`, `Hyperopt`, `BayesSearchCV`.

---

## 5.5 Successive Halving (HalvingGridSearchCV)

Starts by evaluating **all candidates** with a **small resource budget** (e.g., few training samples or iterations), then progressively **doubles** the budget while **halving** the number of surviving candidates.

```
Round 1:  64 candidates × 1/8 budget  → keep top 32
Round 2:  32 candidates × 1/4 budget  → keep top 16
Round 3:  16 candidates × 1/2 budget  → keep top 8
Round 4:   8 candidates × full budget → pick winner ★

Much faster than evaluating all 64 at full budget!
```

```python
from sklearn.experimental import enable_halving_search_cv  # noqa
from sklearn.model_selection import HalvingGridSearchCV

halving_search = HalvingGridSearchCV(
    estimator=RandomForestClassifier(random_state=42),
    param_grid=param_grid,
    factor=2,            # halve candidates each round
    cv=5,
    scoring='accuracy',
    random_state=42,
    n_jobs=-1
)
halving_search.fit(X, y)
print(f"Best Params: {halving_search.best_params_}")
```

---

## 5.6 Common Hyperparameters for Popular Algorithms

| Algorithm | Key Hyperparameters | Typical Range / Values |
|-----------|--------------------|-----------------------|
| **Logistic Regression** | C (inverse regularization), penalty | C: [0.001–100], penalty: l1/l2 |
| **Decision Tree** | max_depth, min_samples_split, criterion | depth: [3–30], criterion: gini/entropy |
| **Random Forest** | n_estimators, max_depth, max_features | trees: [50–500], features: sqrt/log2 |
| **Gradient Boosting** | learning_rate, n_estimators, max_depth | lr: [0.01–0.3], depth: [3–10] |
| **SVM** | C, kernel, gamma | C: [0.1–100], gamma: scale/auto |
| **KNN** | n_neighbors, weights, metric | K: [3–25], weights: uniform/distance |
| **Neural Network** | learning_rate, hidden_layers, dropout, batch_size | lr: [1e-4–1e-2], dropout: [0.1–0.5] |

---

## 5.7 Best Practices for Hyperparameter Tuning

### Strategy: Coarse-to-Fine Search

```
Step 1 — Coarse Random Search (wide ranges, ~50 iterations)
  learning_rate: [0.001, 0.5]
  max_depth:     [2, 50]
                    │
                    ▼  Identify promising region
Step 2 — Fine Grid Search (narrow ranges, exhaustive)
  learning_rate: [0.01, 0.03, 0.05, 0.1]
  max_depth:     [4, 5, 6, 7, 8]
                    │
                    ▼  Best configuration
  learning_rate = 0.05, max_depth = 6  ★
```

### Checklist

| Practice | Why |
|----------|-----|
| **Always use CV** inside the search | Prevents overfitting to a single validation set |
| **Use `n_jobs=-1`** | Parallelizes fits across CPU cores |
| **Start with Random Search** | Explore broadly before narrowing down |
| **Log all results** | `cv_results_` attribute stores every trial |
| **Fix `random_state`** | Ensures reproducibility |
| **Set a computational budget** | Balance accuracy gain vs. time cost |
| **Use Nested CV** for final reporting | Avoid optimistic bias (see Ch. 4) |

---

## 5.8 Practical Example — Tuning a Random Forest End-to-End

```python
from sklearn.datasets import load_wine
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import RandomizedSearchCV, StratifiedKFold, cross_val_score
from scipy.stats import randint

X, y = load_wine(return_X_y=True)

# Step 1: Coarse Random Search
coarse_params = {
    'n_estimators': randint(50, 500),
    'max_depth': randint(3, 30),
    'min_samples_split': randint(2, 20),
    'max_features': ['sqrt', 'log2', None],
}
cv_inner = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
search = RandomizedSearchCV(
    RandomForestClassifier(random_state=42), coarse_params,
    n_iter=60, cv=cv_inner, scoring='accuracy', random_state=42, n_jobs=-1
)
search.fit(X, y)
print(f"Best Params: {search.best_params_}, Score: {search.best_score_:.4f}")

# Step 2: Unbiased estimate with Nested CV
cv_outer = StratifiedKFold(n_splits=5, shuffle=True, random_state=0)
nested = cross_val_score(search, X, y, cv=cv_outer, scoring='accuracy')
print(f"Nested CV: {nested.mean():.4f} ± {nested.std():.4f}")
```

---

## 📋 Summary Comparison Table

| Method | Search Strategy | Budget Control | Handles Continuous? | Intelligence | Computation |
|--------|----------------|---------------|---------------------|-------------|-------------|
| **Grid Search** | Exhaustive | By grid size | ✗ (discrete grid) | None | High |
| **Random Search** | Stochastic sampling | `n_iter` | ✓ | None | Medium |
| **Bayesian Opt.** | Surrogate + acquisition | Iterations | ✓ | Learns from history | Medium |
| **Halving Search** | Tournament elimination | Factor / rounds | ✗ (grid-based) | Resource-adaptive | Low–Medium |

---

## ❓ Revision Questions

1. **Explain the difference between hyperparameters and model parameters.** Give two examples of each for a Random Forest classifier.

2. **A Grid Search over 4 hyperparameters with 5 values each and 5-fold CV requires how many model fits?** Show the calculation.

3. **Why does Random Search often outperform Grid Search** with the same computational budget? Refer to the Bergstra & Bengio argument.

4. **Describe the Bayesian Optimization loop.** What role do the surrogate model and acquisition function play?

5. **What is Successive Halving?** How does it reduce computation compared to standard Grid Search?

6. **You are tuning a Gradient Boosting model on a 100K-row dataset.** Outline your tuning strategy step by step, including which search method you would use and why.

---

## 🧭 Navigation

| ← Previous | ↑ Up | Next → |
|------------|------|--------|
| [04 — Cross-Validation Techniques](04-cross-validation-techniques.md) | [Model Evaluation](README.md) | [06 — Learning Curves](06-learning-curves.md) |
