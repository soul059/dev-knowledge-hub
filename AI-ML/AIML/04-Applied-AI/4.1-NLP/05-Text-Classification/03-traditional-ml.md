# Traditional ML Approaches for Text Classification

## Overview

Before deep learning, traditional ML classifiers paired with TF-IDF features were the state-of-the-art for text classification — and they remain strong baselines. Naive Bayes, Logistic Regression, SVM, and Random Forest each have strengths that make them suitable for different scenarios.

---

## Naive Bayes

The classic text classification algorithm. Assumes word independence (naive assumption) but works surprisingly well.

```python
from sklearn.naive_bayes import MultinomialNB
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.pipeline import Pipeline

nb_pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(max_features=10000)),
    ('clf', MultinomialNB(alpha=1.0))  # alpha = Laplace smoothing
])

nb_pipeline.fit(X_train, y_train)
print(f"Accuracy: {nb_pipeline.score(X_test, y_test):.3f}")
```

**When to use:** Very fast, works with small datasets, good for spam detection.

---

## Logistic Regression

Often the best traditional classifier for text. Linear model with regularization.

```python
from sklearn.linear_model import LogisticRegression

lr_pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(max_features=20000, ngram_range=(1, 2))),
    ('clf', LogisticRegression(C=1.0, max_iter=1000, solver='lbfgs'))
])

lr_pipeline.fit(X_train, y_train)
predictions = lr_pipeline.predict(X_test)

# Inspect most important features per class
feature_names = lr_pipeline['tfidf'].get_feature_names_out()
coefs = lr_pipeline['clf'].coef_[0]  # for binary classification
top_positive = [feature_names[i] for i in coefs.argsort()[-10:]]
top_negative = [feature_names[i] for i in coefs.argsort()[:10]]
print(f"Top positive words: {top_positive}")
print(f"Top negative words: {top_negative}")
```

**When to use:** Best all-around traditional classifier, interpretable, fast.

---

## Support Vector Machine (SVM)

Linear SVM excels at high-dimensional sparse data (exactly what TF-IDF produces).

```python
from sklearn.svm import LinearSVC

svm_pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(max_features=20000, ngram_range=(1, 2))),
    ('clf', LinearSVC(C=1.0, max_iter=1000))
])

svm_pipeline.fit(X_train, y_train)
```

**When to use:** High-dimensional data, binary classification, often best linear classifier.

---

## Random Forest

Ensemble of decision trees. Handles non-linear relationships but less effective on sparse text features.

```python
from sklearn.ensemble import RandomForestClassifier

rf_pipeline = Pipeline([
    ('tfidf', TfidfVectorizer(max_features=5000)),
    ('clf', RandomForestClassifier(n_estimators=200, n_jobs=-1))
])
```

**When to use:** When combining text features with numerical/categorical features.

---

## Model Comparison

| Model | Speed | Accuracy | Interpretable | Handles Sparse | Best For |
|-------|:-----:|:--------:|:---:|:---:|----------|
| Naive Bayes | ⚡⚡⚡ | Good | ✅ | ✅ | Small data, spam, baseline |
| Logistic Reg. | ⚡⚡ | Very Good | ✅ | ✅ | General purpose, best default |
| Linear SVM | ⚡⚡ | Very Good | Partial | ✅ | Binary, high-dimensional |
| Random Forest | ⚡ | Good | Partial | ⚠️ | Mixed feature types |
| Gradient Boost | 🐢 | Very Good | ❌ | ⚠️ | Tabular + text features |

---

## Hyperparameter Tuning

```python
from sklearn.model_selection import GridSearchCV

pipeline = Pipeline([
    ('tfidf', TfidfVectorizer()),
    ('clf', LogisticRegression(max_iter=1000))
])

param_grid = {
    'tfidf__max_features': [5000, 10000, 20000],
    'tfidf__ngram_range': [(1, 1), (1, 2)],
    'clf__C': [0.1, 1.0, 10.0],
}

grid = GridSearchCV(pipeline, param_grid, cv=5, scoring='f1_macro', n_jobs=-1)
grid.fit(X_train, y_train)

print(f"Best params: {grid.best_params_}")
print(f"Best F1: {grid.best_score_:.3f}")
```

---

## Revision Questions

1. **Why does Naive Bayes work well for text despite its independence assumption?**
2. **How can you interpret which words drive predictions in Logistic Regression?**
3. **Why is Linear SVM particularly suited to TF-IDF feature vectors?**
4. **When would you prefer traditional ML over deep learning for text classification?**
5. **How does `ngram_range=(1,2)` improve classification over unigrams alone?**

---

[Previous: 02-feature-extraction.md](02-feature-extraction.md) | [Next: 04-deep-learning-approaches.md](04-deep-learning-approaches.md)
