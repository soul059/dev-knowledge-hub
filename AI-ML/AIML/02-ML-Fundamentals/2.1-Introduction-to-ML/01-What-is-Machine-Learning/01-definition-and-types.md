# 📘 Chapter 1: Definition and Types of Machine Learning

> **Unit 2.1 – Introduction to Machine Learning** | File 1 of 7

---

## 📋 Overview

This chapter introduces the foundational definition of Machine Learning, traces its origins,
distinguishes it from traditional programming, and surveys the major types of ML paradigms.
By the end, you will understand **what ML is**, **when to use it**, and **how it differs**
from conventional rule-based software.

---

## 1. What is Machine Learning?

### 1.1 Arthur Samuel's Definition (1959)

> *"Machine Learning is a field of study that gives computers the ability to learn
> without being explicitly programmed."*

Arthur Samuel coined the term while working on a checkers-playing program at IBM.
The program improved its strategy by playing thousands of games against itself —
**learning from experience** rather than following hand-coded rules.

### 1.2 Tom Mitchell's Formal Definition (1997)

> *"A computer program is said to **learn** from experience **E** with respect to
> some class of tasks **T** and performance measure **P**, if its performance at
> tasks in **T**, as measured by **P**, improves with experience **E**."*

| Symbol | Meaning | Example (Email Spam Filter) |
|--------|---------|----------------------------|
| **T** | Task | Classifying emails as spam or not spam |
| **P** | Performance | Percentage of emails correctly classified |
| **E** | Experience | Observing thousands of labeled emails |

**Key insight:** ML systems improve automatically through exposure to data — they are
**not** hard-coded with domain rules.

---

## 2. ML vs Traditional Programming

```
┌──────────────────────────────────────────────────────────────┐
│            TRADITIONAL PROGRAMMING                           │
│                                                              │
│   ┌──────────┐                                               │
│   │   DATA   │──────┐                                        │
│   └──────────┘      │    ┌───────────┐    ┌──────────┐       │
│                     ├───▶│  PROGRAM  │───▶│  OUTPUT  │       │
│   ┌──────────┐      │    │  (Rules)  │    └──────────┘       │
│   │  RULES   │──────┘    └───────────┘                       │
│   └──────────┘                                               │
├──────────────────────────────────────────────────────────────┤
│            MACHINE LEARNING                                  │
│                                                              │
│   ┌──────────┐                                               │
│   │   DATA   │──────┐                                        │
│   └──────────┘      │    ┌───────────┐    ┌──────────┐       │
│                     ├───▶│ ML ALGO   │───▶│  RULES   │       │
│   ┌──────────┐      │    │ (Training)│    │ (Model)  │       │
│   │  OUTPUT  │──────┘    └───────────┘    └──────────┘       │
│   └──────────┘                                               │
└──────────────────────────────────────────────────────────────┘
```

**Traditional programming:** You write explicit rules. Input data + rules → output.
**Machine learning:** You provide data + desired outputs. The algorithm **discovers** the rules.

---

## 3. Types of Machine Learning

### 3.1 Supervised Learning

The model learns from **labeled** data — each input `x` has a known output `y`.

**Goal:** Learn a mapping function `f(x) → y` that generalizes to unseen data.

$$\hat{y} = f(x; \theta) \quad \text{where } \theta \text{ are learned parameters}$$

**Examples:** Spam detection, house price prediction, image classification.

### 3.2 Unsupervised Learning

The model discovers **hidden patterns** in **unlabeled** data — no target variable.

**Goal:** Find structure, groupings, or compressed representations in data.

**Examples:** Customer segmentation, anomaly detection, topic modeling.

### 3.3 Reinforcement Learning

An **agent** learns by interacting with an **environment**, receiving **rewards** or
**penalties** for its actions.

**Goal:** Maximize cumulative reward over time.

```
    ┌─────────┐  action   ┌─────────────┐
    │  AGENT  │──────────▶│ ENVIRONMENT │
    │         │◀──────────│             │
    └─────────┘  reward   └─────────────┘
                + state
```

**Examples:** Game playing (AlphaGo), autonomous driving, robotic control.

### 3.4 Semi-Supervised Learning

Uses a **small amount of labeled data** combined with a **large amount of unlabeled data**.

**Why?** Labeling is expensive. Semi-supervised methods leverage abundant unlabeled data
to improve performance beyond what the labeled subset alone could achieve.

**Examples:** Medical image analysis (few expert-labeled scans + many unlabeled scans).

---

## 4. ML Applications Across Industries

| Industry | Application | ML Type |
|------------|--------------------------------------|-----------------|
| **Healthcare** | Disease diagnosis from X-rays | Supervised (CV) |
| **Finance** | Fraud detection in transactions | Supervised / Unsupervised |
| **NLP** | Sentiment analysis, chatbots | Supervised |
| **Computer Vision** | Object detection, face recognition | Supervised (Deep Learning) |
| **Robotics** | Navigation, grasping objects | Reinforcement |
| **Retail** | Recommendation engines | Unsupervised / Supervised |
| **Agriculture** | Crop disease detection via drones | Supervised (CV) |
| **Cybersecurity** | Intrusion detection systems | Unsupervised |

---

## 5. When to Use ML vs Rule-Based Systems

| Criteria | Use ML | Use Rules |
|----------|--------|-----------|
| Pattern complexity | High-dimensional, non-linear patterns | Simple, well-defined logic |
| Data availability | Large datasets available | Little or no data |
| Problem stability | Patterns change over time | Rules are static and stable |
| Explainability need | Prediction accuracy matters most | Full transparency required |
| Development speed | Need to generalize from examples | Quick hard-coded solution works |

**Rule of thumb:** If you can't write down the rules easily — or they change frequently —
ML is likely the better approach.

---

## 6. Python Code — A Simple ML Example

```python
# Simple supervised learning: Iris flower classification
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeClassifier
from sklearn.metrics import accuracy_score

# Load dataset
iris = load_iris()
X, y = iris.data, iris.target   # features and labels

# Split into training and testing sets (80/20)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Train a Decision Tree classifier
model = DecisionTreeClassifier(max_depth=3, random_state=42)
model.fit(X_train, y_train)

# Evaluate on test data
y_pred = model.predict(X_test)
accuracy = accuracy_score(y_test, y_pred)
print(f"Test Accuracy: {accuracy:.2%}")
# Output: Test Accuracy: 100.00%
```

**What happened?** We gave the model labeled data (flowers with known species).
It learned decision rules automatically and predicted species on new, unseen flowers.

---

## 7. Summary Table — ML Types at a Glance

| Type | Data | Goal | Key Algorithms | Example |
|------|------|------|----------------|---------|
| **Supervised** | Labeled (x, y) | Predict y from x | Linear Regression, SVM, Random Forest | Email spam filter |
| **Unsupervised** | Unlabeled (x) | Find hidden structure | K-Means, PCA, DBSCAN | Customer segmentation |
| **Reinforcement** | Reward signals | Maximize cumulative reward | Q-Learning, PPO, DQN | Game-playing AI |
| **Semi-Supervised** | Few labeled + many unlabeled | Improve with unlabeled data | Label Propagation, Self-Training | Medical image analysis |

---

## ✅ Revision Questions

1. **State Tom Mitchell's formal definition of ML.** Identify T, P, and E for a
   self-driving car that learns to stay in its lane.

2. **Draw or describe** the key difference between traditional programming and
   machine learning in terms of inputs and outputs.

3. **Classify each scenario** into supervised, unsupervised, or reinforcement learning:
   - (a) Grouping news articles by topic without predefined categories
   - (b) Predicting house prices from square footage and location
   - (c) A robot learning to walk by trial and error

4. **Why is semi-supervised learning useful?** Give one real-world scenario where
   labeling all data is impractical.

5. **When should you prefer rule-based systems over ML?** List at least three criteria.

6. **In the Python example above**, what role does `train_test_split` play, and why
   is it important to evaluate on data the model has not seen during training?

---

## 🧭 Navigation

| Previous | Next |
|----------|------|
| — | [02 – Supervised, Unsupervised & Reinforcement Learning →](02-supervised-unsupervised-reinforcement.md) |

[🔼 Back to Unit 2.1 Overview](../README.md)
