# Chapter 7: The End-to-End Machine Learning Workflow

> **Unit**: 01 — What is Machine Learning | **Prereqs**: Model Selection, Bias-Variance Tradeoff

---

## 7.1 Why a Structured Workflow?

ML is **iterative, not linear**. A disciplined pipeline ensures
reproducibility, catches errors early, and enables collaboration. Data
scientists spend ~80 % of time on data preparation — a good workflow keeps
that effort organized.

## 7.2 Complete ML Pipeline Overview

```
 ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
 │ 1. Problem   │───►│ 2. Data      │───►│ 3. EDA       │
 │   Definition │    │   Collection │    │              │
 └──────────────┘    └──────────────┘    └──────┬───────┘
                                                ▼
 ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
 │ 6. Model     │◄───│ 5. Feature   │◄───│ 4. Data      │
 │   Selection  │    │   Engineering│    │   Preprocess.│
 └──────┬───────┘    └──────────────┘    └──────────────┘
        ▼
 ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
 │ 7. Training  │───►│ 8. Evaluation│───►│ 9. Tuning    │
 └──────────────┘    └──────────────┘    └──────┬───────┘
                                                ▼
                                         ┌──────────────┐
                              feedback ◄─│10. Deploy &  │
                              loops      │   Monitor    │
                                         └──────────────┘
```

## 7.3 Each Stage in Detail

### Stage 1 — Problem Definition
Translate business question into an ML task. Define success metric, decide
supervised vs unsupervised, set baseline.  
**Deliverable**: Problem statement with KPI target.  
**Pitfall**: Jumping to modeling before understanding the problem.

> *Example*: "Reduce churn by 15 %" → binary classification, metric = recall ≥ 0.85.

### Stage 2 — Data Collection
Gather raw data from databases, APIs, web scraping. Merge sources.  
**Deliverable**: Raw dataset + data dictionary.  
**Pitfall**: Ignoring sampling bias or privacy regulations.

### Stage 3 — Exploratory Data Analysis (EDA)
Compute summary stats, plot distributions, check correlations & class balance.

```
  Mean  μ = (1/n) Σ xᵢ       Std  σ = √[(1/n) Σ (xᵢ − μ)²]
```

**Deliverable**: EDA report with visualizations.  
**Pitfall**: Skipping EDA and missing data quality issues.

### Stage 4 — Data Preprocessing
Handle missing values, encode categoricals, scale numerics.

```
  ┌────────────┬──────────────────────────────┐
  │ Drop       │ If < 5 % missing and MCAR    │
  │ Mean/Mode  │ Simple, distorts variance     │
  │ KNN Impute │ Uses similar samples          │
  └────────────┴──────────────────────────────┘
```

**Deliverable**: Clean, model-ready dataset.  
**Pitfall**: Fitting scaler on test data → **data leakage**.

### Stage 5 — Feature Engineering
Create informative features: polynomial terms, log transforms, domain features.

```
  price_per_sqft = price / sqft
  house_age      = current_year - year_built
```

**Deliverable**: Final feature matrix X.  
**Pitfall**: Creating features from future data (temporal leakage).

### Stage 6 — Model Selection
Cross-validate 3-5 candidates, compare scores.  
**Deliverable**: Ranked model list with CV scores.  
**Pitfall**: Only trying one algorithm. *(See Chapter 6 for full framework.)*

### Stage 7 — Model Training
Fit selected model. Optimization objective (linear regression example):

```
  Minimize  J(θ) = (1/2n) Σ (hθ(xᵢ) − yᵢ)²
```

**Deliverable**: Trained model / saved weights.  
**Pitfall**: Training on entire dataset with no held-out test set.

### Stage 8 — Model Evaluation
Assess on unseen test data. Compute precision, recall, F1, AUC, RMSE.

```
               Predicted
            │  Pos │  Neg │
  Actual P  │  TP  │  FN  │       Precision = TP / (TP+FP)
  Actual N  │  FP  │  TN  │       Recall    = TP / (TP+FN)
                                  F1 = 2·P·R / (P+R)
```

**Pitfall**: Using accuracy alone on imbalanced data.

### Stage 9 — Hyperparameter Tuning

```
  Grid Search        Random Search      Bayesian Opt.
  ┌─┬─┬─┬─┐         ┌─ ─ ─ ─ ─┐       ┌─ ─ ─ ─ ─┐
  │●│●│●│●│         │ ●    ●  │       │    ►●    │
  │●│●│●│●│         │   ●     │       │  ►●►●   │
  │●│●│●│●│         │ ●    ●  │       │ (smart)  │
  └─┴─┴─┴─┘         └─ ─ ─ ─ ─┘       └─ ─ ─ ─ ─┘
  Exhaustive O(k^d)  Random combos     Fewest evals
```

**Pitfall**: Tuning on the test set instead of validation set.

### Stage 10 — Deployment & Monitoring
Serialize model, build API, set up drift monitoring dashboards.  
**Pitfall**: No monitoring — model degrades silently.

## 7.4 Iterative Nature of ML

```
  Problem ──► Data ──► Model ──► Evaluate
     ▲                               │
     │           ITERATE             │
     └───────────────────────────────┘
```

- Poor results? → Collect more data  
- Overfitting? → Simplify model / regularize  
- Underfitting? → Better features  
- Data drift? → Retrain on fresh data  

Most projects cycle **4-10 iterations** before production quality.

## 7.5 Python — Complete Mini-Pipeline

```python
import pandas as pd
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split, GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, accuracy_score
from sklearn.pipeline import Pipeline

# Stage 1-2: Problem + Data
data = load_breast_cancer()
X = pd.DataFrame(data.data, columns=data.feature_names)
y = data.target  # 0 = malignant, 1 = benign

# Stage 3: Quick EDA
print("Shape:", X.shape, "| Missing:", X.isnull().sum().sum())
print("Classes:", pd.Series(y).value_counts().to_dict())

# Stage 4-7: Preprocess + Train (Pipeline prevents leakage)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y)

pipe = Pipeline([
    ("scaler", StandardScaler()),
    ("clf", RandomForestClassifier(random_state=42)),
])

# Stage 9: Hyperparameter Tuning
param_grid = {"clf__n_estimators": [50, 100, 200],
              "clf__max_depth": [None, 5, 10]}
grid = GridSearchCV(pipe, param_grid, cv=5, scoring="f1", n_jobs=-1)
grid.fit(X_train, y_train)
print("Best params:", grid.best_params_)
print("Best CV F1:", round(grid.best_score_, 4))

# Stage 8: Evaluate on held-out test set
y_pred = grid.predict(X_test)
print("Test Accuracy:", round(accuracy_score(y_test, y_pred), 4))
print(classification_report(y_test, y_pred, target_names=data.target_names))
```

## 7.6 Real-World Example — Spam Classifier End-to-End

| Stage | Activity |
|-------|----------|
| 1. Problem Def. | Binary classification: spam vs ham, F1 ≥ 0.95 |
| 2. Data | 50 k labeled emails from company mail server |
| 3. EDA | 30 % spam, inspect top spam keywords |
| 4. Preprocess | Lowercase, strip HTML, tokenize, handle missing subjects |
| 5. Features | TF-IDF vectors, email length, link count, urgency words |
| 6. Selection | CV: Naive Bayes, Logistic Reg., SVM, LightGBM |
| 7. Training | Fit LightGBM (winner) on 80 % train split |
| 8. Evaluation | Test F1 = 0.96, 2 % false positive rate |
| 9. Tuning | Random search: learning rate, max depth, num leaves |
| 10. Deploy | Serialize with joblib, REST API, monitor drift weekly |

## 7.7 Summary Table

| # | Stage | Input | Output | Key Tool |
|---|-------|-------|--------|----------|
| 1 | Problem Definition | Business question | ML task + metric | Domain expertise |
| 2 | Data Collection | Source list | Raw dataset | SQL, APIs |
| 3 | EDA | Raw data | Insights report | pandas, matplotlib |
| 4 | Preprocessing | Raw data | Clean data | Imputer, Encoder |
| 5 | Feature Engineering | Clean data | Feature matrix | Domain logic |
| 6 | Model Selection | X, y | Top candidates | cross_val_score |
| 7 | Training | X_train, y_train | Fitted model | .fit() |
| 8 | Evaluation | X_test, y_test | Metrics report | classification_report |
| 9 | Tuning | Model + grid | Best params | GridSearchCV |
| 10 | Deployment | Final model | Live API | Flask, joblib |

## 7.8 Revision Questions

1. List the 10 ML pipeline stages in order. Which stage consumes the most
   time and why?
2. What is data leakage? Give one preprocessing and one feature engineering
   example. How does `sklearn.pipeline.Pipeline` prevent it?
3. Compare Grid Search vs Random Search. When is Random Search preferred?
4. A spam classifier has 98 % accuracy but only 3 % of emails are spam. Why
   is accuracy misleading? Which metric(s) would you use instead?
5. Name two things to monitor after deployment. What action would you take
   on detecting concept drift?
6. Walk through building a house-price predictor end-to-end, naming specific
   techniques at each pipeline stage.

---

| ← Previous | Next → |
|:-----------:|:------:|
| [06 — Model Selection](06-model-selection.md) | *End of Unit 01* |
