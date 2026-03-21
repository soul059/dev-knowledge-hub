# Chapter 6: Model Selection in Machine Learning

> **Unit**: 01 — What is Machine Learning | **Prereqs**: Bias-Variance Tradeoff, Supervised Learning basics

---

## 6.1 Why Model Selection Matters

No single algorithm dominates all problems. Model selection is the disciplined
process of comparing candidates and picking the one that best balances
**performance**, **cost**, and **constraints**.

```
  Raw Data ──► Candidate Models ──► Evaluation ──► Best Model ──► Deploy
                 ├─ Linear Reg.        │
                 ├─ Decision Tree       │  compare on
                 ├─ SVM                 │  validation set
                 └─ Neural Net          ▼
                                  Selection Criteria
```

## 6.2 Core Selection Criteria

| Criterion | Measures | Dominates When |
|-----------|---------|----------------|
| **Accuracy** | Prediction quality (RMSE, F1, AUC) | Research, competitions |
| **Interpretability** | Ability to explain predictions | Healthcare, finance, legal |
| **Training Time** | Time to fit | Rapid prototyping, limited GPU |
| **Inference Latency** | Prediction speed per sample | Real-time / edge devices |
| **Scalability** | Handling large N or D | Big-data pipelines |
| **Memory Footprint** | RAM / disk for model | Mobile, embedded deployment |

```
 High Interpretability ◄──────────────────────► Low Interpretability
 ┌──────────┬──────────┬───────────┬──────────┬──────────────┐
 │ Linear   │ Decision │ Logistic  │ Random   │ Deep Neural  │
 │ Regress. │ Tree     │ Regress.  │ Forest   │ Networks     │
 └──────────┴──────────┴───────────┴──────────┴──────────────┘
```

## 6.3 The No Free Lunch Theorem

> *"Any two algorithms are equivalent when averaged across all possible
> problems."* — Wolpert & Macready, 1997

For algorithms **A₁** and **A₂** over every target function **f**:

```
  Σ P(error | A₁, f)  =  Σ P(error | A₂, f)
```

**Implications**: (1) No universal best model. (2) Domain knowledge guides
algorithm choice. (3) Always benchmark empirically. (4) Assumptions about the
data are what let specific algorithms win.

## 6.4 Algorithm Selection by Data Characteristics

| Data Characteristic | Try First | Avoid |
|--------------------|-----------|-------|
| Small N (< 1 k) | SVM, k-NN, Logistic Reg. | Deep Learning |
| Large N (> 100 k) | XGBoost, SGD models, Neural Nets | Kernel SVM O(N³) |
| High-dim (D >> N) | Lasso, Ridge, Linear SVM | Plain Decision Tree |
| Many categoricals | CatBoost, LightGBM, Trees | Linear Regression |
| Need probabilities | Logistic Reg., Naive Bayes | SVM (uncalibrated) |
| Non-linear patterns | Random Forest, GBM, MLP | Linear models |

## 6.5 Validation-Based Selection (Cross-Validation)

```
  Fold 1:  [VAL][ Train ][ Train ][ Train ][ Train ]  → s₁
  Fold 2:  [ Train ][VAL][ Train ][ Train ][ Train ]  → s₂
  Fold 3:  [ Train ][ Train ][VAL][ Train ][ Train ]  → s₃
  Fold 4:  [ Train ][ Train ][ Train ][VAL][ Train ]  → s₄
  Fold 5:  [ Train ][ Train ][ Train ][ Train ][VAL]  → s₅
  CV Score = (s₁ + s₂ + … + s₅) / 5
```

Use **stratified CV** for classification to preserve class ratios in each fold.

## 6.6 Information Criteria: AIC & BIC

**AIC** = 2k − 2 ln(L̂)  &nbsp;&nbsp;|&nbsp;&nbsp; **BIC** = k · ln(n) − 2 ln(L̂)

- **k** = number of parameters, **n** = samples, **L̂** = max likelihood
- Lower is better. BIC penalizes complexity more when n ≥ 8.

| Aspect | AIC | BIC |
|--------|-----|-----|
| Penalty / param | 2 | ln(n) |
| Favors | Flexible models | Simpler models |
| Best for | Prediction | True model identification |

**Example** (n = 100): Model A (k=3, lnL̂=−120): AIC=246, BIC≈253.8.
Model B (k=6, lnL̂=−115): AIC=242, BIC≈257.6. AIC picks B; BIC picks A.

## 6.7 Practical Decision Framework

```
                ┌────────────────────────┐
                │ Define success metric  │
                │ & constraints          │
                └──────────┬─────────────┘
                           ▼
                ┌────────────────────────┐
         ┌─YES──┤ Need interpretability? │
         │      └──────────┬─────────────┘
         ▼                 │ NO
  ┌──────────────┐         ▼
  │ Linear/Tree  │  ┌──────────────────┐
  │ models       │  │ Small or Large N?│
  └──────────────┘  └──┬──────────┬────┘
                   Small N      Large N
                     ▼            ▼
              ┌───────────┐ ┌────────────┐
              │ SVM, k-NN │ │ GBM, NN    │
              └─────┬─────┘ └─────┬──────┘
                    └──────┬──────┘
                           ▼
                ┌────────────────────────┐
                │ CV top 2-3 candidates  │
                └──────────┬─────────────┘
                           ▼
                ┌────────────────────────┐
                │ Check deploy limits    │
                │ (latency/memory/cost)  │
                └──────────┬─────────────┘
                           ▼
                ┌────────────────────────┐
                │ Select final model     │
                └────────────────────────┘
```

## 6.8 Python — Comparing Multiple Models

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import cross_val_score
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.svm import SVC
from sklearn.neighbors import KNeighborsClassifier
import time

X, y = load_iris(return_X_y=True)
models = {
    "Logistic Reg.":     LogisticRegression(max_iter=200),
    "Decision Tree":     DecisionTreeClassifier(),
    "Random Forest":     RandomForestClassifier(n_estimators=100),
    "Gradient Boost":    GradientBoostingClassifier(n_estimators=100),
    "SVM (RBF)":         SVC(),
    "k-NN (k=5)":        KNeighborsClassifier(n_neighbors=5),
}

print(f"{'Model':<20} {'CV Acc':>8} {'Std':>7} {'Time(s)':>8}")
print("-" * 46)
for name, m in models.items():
    t = time.time()
    sc = cross_val_score(m, X, y, cv=5, scoring="accuracy")
    print(f"{name:<20} {sc.mean():>8.4f} {sc.std():>7.4f} {time.time()-t:>8.4f}")
```

## 6.9 Real-World Considerations

| Concern | Question | Impact |
|---------|----------|--------|
| Latency | < 10 ms per prediction? | Avoid large ensembles / deep nets |
| Memory | Deploying on mobile? | Prefer LR, small tree |
| Retraining | Data shifts weekly? | Choose fast-to-train models |
| Regulatory | Must explain every prediction? | Use interpretable models + SHAP |
| Cost | Limited GPU budget? | GBM often beats NN on tabular data |

**Example — Fraud Detection**: Need high recall + explainability + real-time
scoring → XGBoost with SHAP explanations.

## 6.10 Summary Table

| Topic | Key Takeaway |
|-------|-------------|
| Selection criteria | Balance accuracy, interpretability, speed, scalability |
| No Free Lunch | No universal best — always benchmark |
| Data-driven choice | Match algorithm to data size, dims, type |
| Cross-validation | Gold standard for unbiased comparison |
| AIC / BIC | Compare likelihood models without validation set |
| Decision framework | Filter by constraints, then CV the shortlist |
| Deployment | Latency & memory often override raw accuracy |

## 6.11 Revision Questions

1. State the No Free Lunch theorem. Why does it make empirical comparison necessary?
2. A dataset has 500 samples, 10 000 features. Which algorithms to try first and why?
3. Compute AIC and BIC for k=4, n=200, ln(L̂)=−300. Show your work.
4. Why does BIC select simpler models than AIC when n is large?
5. Rank Logistic Regression, Random Forest (500 trees), and a 50-layer CNN
   for deployment on a 2 GB RAM smartphone. Justify.
6. Explain 5-fold stratified CV. Why is stratification important for imbalanced data?

---

| ← Previous | Next → |
|:-----------:|:------:|
| [05 — Bias-Variance Tradeoff](05-bias-variance-tradeoff.md) | [07 — ML Workflow](07-ml-workflow.md) |
