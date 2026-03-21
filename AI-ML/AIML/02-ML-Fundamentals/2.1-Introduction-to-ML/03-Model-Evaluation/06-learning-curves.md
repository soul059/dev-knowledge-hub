# 📈 Chapter 6: Learning Curves & Validation Curves

> **Unit 3 – Model Evaluation** | Diagnose bias, variance, and hyperparameter sensitivity with learning and validation curves.

---

## 6.1 What Are Learning Curves?

A **learning curve** plots model performance (score) against **training set size**. Two curves are drawn simultaneously:

| Curve | What It Shows |
|---|---|
| **Training score** | How well the model fits the data it was trained on |
| **Validation score** | How well the model generalises to unseen data |

As training size `m` increases:
- **Training score** typically *decreases* slightly (harder to memorise more data).
- **Validation score** typically *increases* (more data → better generalisation).

The relationship can be approximated as:

```
Validation Error ≈ irreducible_error + O(1/m)
Training Error   ≈ irreducible_error − O(1/m)
```

### Training Curve vs Validation Curve — Key Distinction

| Aspect | Learning Curve | Validation Curve |
|---|---|---|
| **X-axis** | Training set size | Hyperparameter value |
| **Y-axis** | Score (accuracy, F1, etc.) | Score (accuracy, F1, etc.) |
| **Purpose** | Diagnose bias/variance | Tune hyperparameters |
| **Plots** | Train score + Val score vs `m` | Train score + Val score vs param |

---

## 6.2 Diagnosing Problems from Learning Curves

### Pattern 1 — High Bias (Underfitting)

```
Score
1.0 |
    |
0.8 |
    |
0.6 |  ──────────────────────── Training Score
    |  ──────────────────────── Validation Score
0.4 |
    |
0.2 |
    +──────────────────────────────▶
              Training Set Size
```

- **Both** curves plateau at a **low score**.
- Gap between curves is **small**.
- Adding more data will **not** help.
- **Remedies**: increase model complexity, add polynomial/interaction features, reduce regularisation.

### Pattern 2 — High Variance (Overfitting)

```
Score
1.0 |  ━━━━━━━━━━━━━━━━━━━━━━━━ Training Score
    |                    ╭──────
0.8 |               ╭───╯
    |          ╭───╯
0.6 |     ╭───╯          Validation Score
    | ───╯
0.4 |
    |        ⟵ LARGE GAP ⟶
0.2 |
    +──────────────────────────────▶
              Training Set Size
```

- Training score stays **high**; validation score remains **much lower**.
- Gap between curves is **large** and may slowly narrow with more data.
- **Remedies**: get more training data, reduce model complexity, increase regularisation, apply dropout / early stopping.

### Pattern 3 — Good Fit

```
Score
1.0 |  ━━━━━━━━━━━━━━━━━━━━━━━━ Training Score
    |               ╭━━━━━━━━━━ Validation Score
0.8 |          ╭━━━╯
    |     ╭━━━╯
0.6 | ━━━╯         ⟵ small gap ⟶
    |
0.4 |
    +──────────────────────────────▶
              Training Set Size
```

- Both curves converge at a **high score**.
- Gap is **small** and narrows as `m` grows.
- Model complexity and data volume are well-matched.

---

## 6.3 Interpreting Learning Curves — Actionable Insights

| Observation | Diagnosis | Action |
|---|---|---|
| Both curves low, small gap | High bias | More features, complex model |
| Train high, val low, big gap | High variance | More data, regularise |
| Curves converge high | Good fit | Deploy / fine-tune threshold |
| Val score still rising at max `m` | Need more data | Collect additional samples |
| Both curves oscillate | Noisy data or bad CV | Increase CV folds, clean data |

---

## 6.4 Validation Curves (Score vs Hyperparameter)

A **validation curve** fixes the training set size and sweeps a single hyperparameter:

```
Score
1.0 |      ╭━━━━━━╮
    |    ╭╯       ╰╮         Training Score
0.8 |  ╭╯          ╰╮
    | ╭╯             ╰─── Validation Score
0.6 |╯
    |
0.4 | Under-      Optimal     Over-
    |  fit         zone         fit
    +──────────────────────────────▶
         Hyperparameter Value
            (e.g., max_depth)
```

- **Left region** (low complexity): both scores low → underfitting.
- **Middle region**: validation score peaks → optimal hyperparameter.
- **Right region** (high complexity): training score high but validation drops → overfitting.

---

## 6.5 Python Code — `learning_curve` & `validation_curve`

### Learning Curve with scikit-learn

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.model_selection import learning_curve
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import load_breast_cancer

X, y = load_breast_cancer(return_X_y=True)
model = RandomForestClassifier(n_estimators=50, random_state=42)

train_sizes, train_scores, val_scores = learning_curve(
    model, X, y,
    train_sizes=np.linspace(0.1, 1.0, 10),
    cv=5,
    scoring="accuracy",
    n_jobs=-1
)

train_mean = train_scores.mean(axis=1)
val_mean   = val_scores.mean(axis=1)
train_std  = train_scores.std(axis=1)
val_std    = val_scores.std(axis=1)

plt.figure(figsize=(8, 5))
plt.plot(train_sizes, train_mean, "o-", label="Training Score")
plt.plot(train_sizes, val_mean,   "o-", label="Validation Score")
plt.fill_between(train_sizes, train_mean - train_std,
                 train_mean + train_std, alpha=0.15)
plt.fill_between(train_sizes, val_mean - val_std,
                 val_mean + val_std, alpha=0.15)
plt.xlabel("Training Set Size")
plt.ylabel("Accuracy")
plt.title("Learning Curve — Random Forest (Breast Cancer)")
plt.legend(loc="lower right")
plt.grid(True)
plt.tight_layout()
plt.show()
```

### Validation Curve with scikit-learn

```python
from sklearn.model_selection import validation_curve

param_range = np.arange(1, 21)

train_scores, val_scores = validation_curve(
    RandomForestClassifier(n_estimators=50, random_state=42),
    X, y,
    param_name="max_depth",
    param_range=param_range,
    cv=5,
    scoring="accuracy",
    n_jobs=-1
)

plt.figure(figsize=(8, 5))
plt.plot(param_range, train_scores.mean(axis=1), "o-", label="Training")
plt.plot(param_range, val_scores.mean(axis=1),   "o-", label="Validation")
plt.xlabel("max_depth")
plt.ylabel("Accuracy")
plt.title("Validation Curve — max_depth")
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```

---

## 6.6 Real-World Example — Spam Classifier Diagnosis

**Scenario**: A Naïve Bayes spam classifier achieves 92 % training accuracy and 91 % validation accuracy, but both are below the 97 % target.

**Learning Curve Reading**:

| Size | Train Acc | Val Acc |
|------|-----------|---------|
| 500 | 0.93 | 0.88 |
| 2 000 | 0.92 | 0.90 |
| 5 000 | 0.92 | 0.91 |
| 10 000 | 0.92 | 0.91 |

**Diagnosis**: Both curves plateau ≈ 0.92 with a small gap → **high bias**.

**Actions taken**:
1. Switched from unigrams to **bigram TF-IDF** features → accuracy rose to 0.95.
2. Replaced Naïve Bayes with **Logistic Regression** (higher capacity) → accuracy reached 0.97.
3. Verified the gap remained small (no new variance problem).

---

## 6.7 Summary — Curve Patterns & Recommended Actions

| Pattern | Train Score | Val Score | Gap | Diagnosis | Remedy |
|---|---|---|---|---|---|
| Both plateau low | Low | Low | Small | High Bias | Complex model, more features |
| Train high, val low | High | Low | Large | High Variance | More data, regularise, simplify |
| Both converge high | High | High | Small | Good Fit | Deploy model |
| Val still rising | Moderate | Rising | Moderate | Insufficient data | Collect more samples |
| Val curve peaks then drops | — | Peak visible | Varies | Optimal param found | Use peak value |

---

## 6.8 Revision Questions

1. **Define** a learning curve and explain what its two component curves represent.
2. A learning curve shows both training and validation scores plateauing at 0.60. **Diagnose** the problem and propose two concrete remedies.
3. You observe a persistent 20-percentage-point gap between training and validation scores even at maximum training size. What does this indicate, and how would you address it?
4. **Distinguish** between a learning curve and a validation curve in terms of purpose, x-axis variable, and typical use case.
5. In a validation curve for `max_depth`, the training score keeps rising while the validation score peaks at `max_depth = 8` and then declines. **Explain** what is happening and what value you should choose.
6. Why is it important to plot **confidence bands** (± std) on learning curves, and what does high variance across folds suggest?

---

## Navigation

| ← Previous | Next → |
|---|---|
| [05 – Hyperparameter Tuning](05-hyperparameter-tuning.md) | [07 – Validation Strategies](07-validation-strategies.md) |
