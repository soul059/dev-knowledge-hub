# Chapter 7: Data Splitting Strategies

> **Unit 2 · Data Preprocessing** — Split your data correctly to get **honest** performance estimates and avoid data leakage.

---

## 7.1 Why Splitting Matters

```
     ┌───────────────────────────────────┐
     │       FULL DATASET (N rows)       │
     └────┬──────────┬──────────┬────────┘
     ┌────▼───┐ ┌────▼─────┐ ┌─▼─────┐
     │ TRAIN  │ │VALIDATE  │ │ TEST  │     Golden Rule: Test set
     │ 60-80% │ │ 10-20%   │ │10-20% │     stays UNTOUCHED until
     │Fit mdl │ │Tune hyp. │ │Final  │     final evaluation.
     └────────┘ └──────────┘ └───────┘
```

## 7.2 Splitting Strategies Compared

```
Random Split       Stratified Split    Time-Based Split    Group-Based Split
────────────       ────────────────    ────────────────    ─────────────────
● ○ ● ○ ● ○       Class A: 70/30     ─────────────►      Patient 1 → Train
Shuffle &          Class B: 70/30      Past │ Future      Patient 2 → Train
divide randomly    Same ratio kept    Train │  Test       Patient 3 → Test
```

---

## 7.3 Random Split

The simplest strategy — rows are shuffled and divided. Risk: minority class may end up entirely in one split; temporal patterns ignored.

```python
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
```

| Pros | Cons |
|---|---|
| Simple, fast | Ignores class balance and time ordering |

---

## 7.4 Stratified Split

Maintains the **same class distribution** in every split — essential for imbalanced datasets.

```
Original → Class 0: 90%  Class 1: 10%
Train    → Class 0: 90%  Class 1: 10%  ✓ preserved
Test     → Class 0: 90%  Class 1: 10%  ✓ preserved
```

```python
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42   # ← preserves ratio
)
```

| Pros | Cons |
|---|---|
| Preserves class ratios | Still ignores temporal order |

---

## 7.5 Time-Based Split

For **temporal** data, train on the **past** and test on the **future** — never the reverse.

```
Timeline ─────────────────────────────────────────►
         │◄─── Training Data ───►│◄── Test Data ──►│
         Jan 2022          Dec 2023          Dec 2024
WRONG:  Randomly mixing 2024 rows into training → future leakage!
```

```python
df = df.sort_values('date')
train = df[df['date'] < '2024-01-01']
test  = df[df['date'] >= '2024-01-01']
```

### Time-Series Cross-Validation — training window expands; validation always follows:
```
Fold 1:  [Train]  [Val]              from sklearn.model_selection import TimeSeriesSplit
Fold 2:  [Train ····]  [Val]         tscv = TimeSeriesSplit(n_splits=5)
Fold 3:  [Train ·········]  [Val]    for i, (tr, va) in enumerate(tscv.split(X)):
         ─────────────────────► time      print(f"Fold {i}: train={len(tr)}")
```

| Pros | Cons |
|---|---|
| Prevents future data leakage | Reduces usable training data |

---

## 7.6 Group-Based Split

When rows from the **same group** (patient, user, store) must stay together to prevent leakage.

```
Patient A has 12 scans → ALL go to Train
Patient B has  8 scans → ALL go to Test
If scans from Patient A appear in both → model memorises the patient!
```

```python
from sklearn.model_selection import GroupShuffleSplit, GroupKFold

gss = GroupShuffleSplit(n_splits=1, test_size=0.2, random_state=42)
train_idx, test_idx = next(gss.split(X, y, groups=groups))
assert set(groups[train_idx]).isdisjoint(set(groups[test_idx]))  # no overlap

gkf = GroupKFold(n_splits=5)  # GroupKFold for cross-validation
for fold, (tr, va) in enumerate(gkf.split(X, y, groups)):
    print(f"Fold {fold}: train groups={len(set(groups[tr]))}")
```

| Pros | Cons |
|---|---|
| Prevents group-level leakage | Uneven fold sizes; needs group column |

---

## 7.7 K-Fold Cross-Validation Preparation

K-Fold gives a **robust** performance estimate by rotating the validation set:

```
Fold 1:  [Val ] [Train] [Train] [Train] [Train]
Fold 2:  [Train] [Val ] [Train] [Train] [Train]
Fold 3:  [Train] [Train] [Val ] [Train] [Train]
Fold 4:  [Train] [Train] [Train] [Val ] [Train]
Fold 5:  [Train] [Train] [Train] [Train] [Val ]
Final score = mean(score₁ … score_k) ± std
```

```python
from sklearn.model_selection import cross_val_score, StratifiedKFold
from sklearn.ensemble import RandomForestClassifier

skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
model = RandomForestClassifier(n_estimators=100, random_state=42)
scores = cross_val_score(model, X, y, cv=skf, scoring='f1_macro')
print(f"F1 (macro): {scores.mean():.4f} ± {scores.std():.4f}")
```

---

## 7.8 Handling Imbalanced Datasets

When one class dominates (e.g., fraud = 0.5%), models learn to always predict the majority.

```
┌──────────────────────────────────────────────────────────┐
│              IMBALANCED DATA STRATEGIES                  │
├───────────────────┬───────────────────┬──────────────────┤
│  OVERSAMPLING     │  UNDERSAMPLING    │ SYNTHETIC (SMOTE)│
│  Duplicate minor. │  Remove majority  │ Interpolate NEW  │
│  Risk: overfit    │  Risk: info loss  │ minority samples │
└───────────────────┴───────────────────┴──────────────────┘
```

### SMOTE — Synthetic Minority Over-sampling Technique
```
For each minority sample xᵢ:
  1. Find k nearest minority neighbours
  2. Pick one neighbour xₙ at random
  3. x_new = xᵢ + rand(0,1) · (xₙ − xᵢ)
```

### Python — Resampling with imblearn
```python
from imblearn.over_sampling import SMOTE, RandomOverSampler
from imblearn.under_sampling import RandomUnderSampler
from imblearn.pipeline import Pipeline as ImbPipeline
from collections import Counter

print("Before:", Counter(y_train))
X_ros, y_ros = RandomOverSampler(random_state=42).fit_resample(X_train, y_train)
X_sm, y_sm = SMOTE(random_state=42, k_neighbors=5).fit_resample(X_train, y_train)

# Combined pipeline: SMOTE + Undersampling
pipeline = ImbPipeline([
    ('over',  SMOTE(sampling_strategy=0.5, random_state=42)),
    ('under', RandomUnderSampler(sampling_strategy=0.8, random_state=42)),
])
X_bal, y_bal = pipeline.fit_resample(X_train, y_train)
```

> ⚠️ **Critical:** Apply SMOTE / resampling **only on the training set** — never on validation or test data.

---

## 7.9 Decision Guide — When to Use Each Strategy

| Scenario | Recommended Strategy |
|---|---|
| Large balanced dataset, no time/groups | Random split |
| Imbalanced classification | Stratified split + SMOTE on train |
| Stock prices, weather, sales forecasting | Time-based split |
| Medical trials, user-level data | Group-based split (GroupKFold) |
| Small dataset, need robust estimate | Stratified K-Fold CV |
| Imbalanced + temporal | Time-based split + SMOTE on train |

---

## 7.10 Common Pitfalls

| Pitfall | Why It's Dangerous | Fix |
|---|---|---|
| SMOTE before splitting | Synthetic test points leak train info | SMOTE only on train fold |
| Random split on time-series | Future data in training | Use TimeSeriesSplit |
| Ignoring group structure | Same patient in train & test | Use GroupKFold |
| Tuning on test set | Optimistic final metric | Separate validation set |
| No `random_state` | Non-reproducible results | Always seed splits |

---

## 7.11 Summary

| Concept | Key Takeaway |
|---|---|
| Random split | Simple baseline; assumes i.i.d. data |
| Stratified split | Preserves class ratios; use for classification |
| Time-based split | Train on past, test on future; prevents temporal leakage |
| Group-based split | Keeps related rows together; prevents group leakage |
| K-Fold CV | Averages over K rotations for robust scoring |
| SMOTE | Generates synthetic minority samples via interpolation |
| Golden rule | Never let test-set information influence training |

---

## 7.12 Revision Questions

1. **Why** is a random 80/20 split insufficient for a fraud dataset with 0.3% fraud rate?
2. **Explain** data leakage in time-series forecasting. How does `TimeSeriesSplit` prevent it?
3. A hospital dataset has multiple X-rays per patient. **Why** must you use `GroupKFold`?
4. **Describe** how SMOTE creates a synthetic sample. Why apply it **only** on the training fold?
5. **Compare** random oversampling with SMOTE. Which is more prone to overfitting and why?
6. You have 10,000 rows, 5% positive class, timestamps, and user IDs. **Design** a complete splitting and resampling strategy.

---

[⬅️ Previous: 06-feature-selection.md](06-feature-selection.md) | 🏁 **End of Unit 2 — Data Preprocessing**
