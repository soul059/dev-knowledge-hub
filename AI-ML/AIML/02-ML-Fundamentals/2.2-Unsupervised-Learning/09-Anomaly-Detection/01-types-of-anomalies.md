# 🚨 Types of Anomalies

> **Chapter 9.1 — Understanding Anomalies: Point, Contextual, and Collective**

---

## 📌 Overview

**Anomaly detection** (also called **outlier detection**) is the identification of data points, events, or observations that deviate significantly from the expected pattern. Anomalies are rare but critical — they often represent fraud, system failures, medical conditions, or novel discoveries.

### Why Anomaly Detection Matters

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  Normal Data:  ●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●●  (99.9%)        │
│  Anomalies:    ★  ★                                   (0.1%)        │
│                                                                      │
│  But that 0.1% might be:                                            │
│  • A fraudulent credit card transaction ($50,000 loss)              │
│  • A cyberattack on your network                                    │
│  • An early sign of equipment failure                               │
│  • A rare disease requiring immediate treatment                     │
│  • A scientific discovery worth a Nobel Prize!                      │
│                                                                      │
│  "Finding needles in haystacks" — but the needles MATTER.           │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🔍 Three Types of Anomalies

### 1. Point Anomalies

A single data point that is anomalous with respect to the rest of the data.

```
  ┌────────────────────────────────────────────────────┐
  │                                                    │
  │  Feature 2                                         │
  │  ▲                                                 │
  │  │         ★ ← POINT ANOMALY                      │
  │  │                                                 │
  │  │     ●●●●●                                       │
  │  │    ●●●●●●●                                      │
  │  │   ●●●●●●●●                                      │
  │  │    ●●●●●●●                                      │
  │  │     ●●●●●                                       │
  │  │                                                 │
  │  └──────────────────────────────────────► Feature 1│
  │                                                    │
  │  Examples:                                         │
  │  • A credit card transaction of $50,000 when       │
  │    average is $100                                 │
  │  • A temperature reading of 200°F from a sensor    │
  │    that usually reads 70-80°F                      │
  │  • A student scoring 5% on an exam where mean      │
  │    is 75%                                          │
  │                                                    │
  └────────────────────────────────────────────────────┘
```

### 2. Contextual (Conditional) Anomalies

A data point that is anomalous **in a specific context** but might be normal in another context.

```
  ┌────────────────────────────────────────────────────┐
  │                                                    │
  │  Temperature                                       │
  │  ▲                                                 │
  │  │     Summer    Winter                            │
  │  │  ╭──╮     ╭──╮                                  │
  │ 35°│  │ │     │  │     ★ 35°C in Winter            │
  │    │  │ │     │  │       = ANOMALY!                │
  │ 20°│  │ │╭──╮ │  │                                  │
  │    │  │ ││  ││ │  │                                  │
  │  5°│  │ ││  ││ │  │     ★ 35°C in Summer           │
  │    │  ╰─╯╰──╯ ╰──╯       = NORMAL!                │
  │  └──────────────────────────────────► Month        │
  │                                                    │
  │  The CONTEXT (season/time) determines whether      │
  │  the value is anomalous.                           │
  │                                                    │
  │  Components:                                       │
  │  • Contextual attributes: time, location, etc.     │
  │  • Behavioral attributes: the measured value       │
  │                                                    │
  │  Examples:                                         │
  │  • High electricity usage at 3 AM (normal at 3 PM) │
  │  • Low traffic on a website on Christmas Day       │
  │  • Normal heartrate during sleep vs. during exam   │
  │                                                    │
  └────────────────────────────────────────────────────┘
```

### 3. Collective Anomalies

A collection of data points that is anomalous **as a group**, even though individual points may not be anomalous.

```
  ┌────────────────────────────────────────────────────┐
  │                                                    │
  │  ECG Signal (Heart Rhythm)                         │
  │  ▲                                                 │
  │  │  ╱╲    ╱╲    ╱╲  ╱╲╱╲╱╲╱╲  ╱╲    ╱╲           │
  │  │ ╱  ╲  ╱  ╲  ╱  ╲╱        ╲╱  ╲  ╱  ╲          │
  │  │╱    ╲╱    ╲╱    ╲          ╲    ╲╱    ╲         │
  │  │                  ╰──────────╯                   │
  │  │  Normal rhythm    COLLECTIVE     Normal rhythm  │
  │  │                   ANOMALY                       │
  │  └──────────────────────────────────► Time         │
  │                                                    │
  │  No individual heartbeat is "anomalous" —          │
  │  but the PATTERN of rapid irregular beats          │
  │  (as a sequence) indicates arrhythmia.             │
  │                                                    │
  │  Examples:                                         │
  │  • Sequence of rapid small bank transactions       │
  │    (structuring / smurfing to avoid reporting)      │
  │  • Unusual burst of network packets (DDoS attack)  │
  │  • Sequence of mouse clicks too fast for human     │
  │    (bot activity)                                   │
  │                                                    │
  └────────────────────────────────────────────────────┘
```

---

## ⚖️ Supervised vs. Unsupervised Anomaly Detection

```
┌─────────────────────┬─────────────────────┬─────────────────────┐
│     Aspect          │  Supervised         │  Unsupervised       │
├─────────────────────┼─────────────────────┼─────────────────────┤
│ Labels required?    │ Yes (normal +       │ No                  │
│                     │ anomaly labels)     │                     │
│ Training data       │ Both classes        │ Assumed mostly      │
│                     │                     │ normal data         │
│ Approach            │ Classification      │ Density, distance,  │
│                     │ (binary/multi)      │ or isolation based  │
│ Class imbalance     │ Severe (handle      │ Not an issue        │
│                     │ with SMOTE, etc.)   │                     │
│ Examples            │ Random Forest,      │ Isolation Forest,   │
│                     │ SVM, Neural Nets    │ LOF, One-Class SVM  │
│ When to use         │ Labeled historical  │ No labels, or       │
│                     │ anomalies available │ anomaly types       │
│                     │                     │ are unknown         │
│ Pros                │ High accuracy if    │ Can find novel      │
│                     │ labels are good     │ anomaly types       │
│ Cons                │ Requires expensive  │ Higher false        │
│                     │ labeling; limited   │ positive rate       │
│                     │ to known types      │                     │
└─────────────────────┴─────────────────────┴─────────────────────┘
```

### Semi-Supervised Anomaly Detection

```
┌──────────────────────────────────────────────────────────────┐
│ SEMI-SUPERVISED (Most common in practice):                   │
│                                                              │
│ Training data: Only NORMAL examples                          │
│ Goal: Learn what "normal" looks like                         │
│ Detection: Flag anything that deviates from "normal"         │
│                                                              │
│ Methods:                                                     │
│ • One-Class SVM                                              │
│ • Autoencoders (train on normal, high recon error = anomaly) │
│ • Gaussian models (fit normal, low probability = anomaly)    │
│                                                              │
│ Advantage: Don't need labeled anomalies (just normal data)   │
│ Challenge: Anomaly in training data corrupts the model       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 🔄 Anomaly Detection vs. Novelty Detection

```
┌────────────────────────┬─────────────────────┬─────────────────────┐
│     Aspect             │ Anomaly Detection   │ Novelty Detection   │
├────────────────────────┼─────────────────────┼─────────────────────┤
│ Training data          │ May contain         │ Clean (no           │
│                        │ anomalies           │ anomalies)          │
│ Goal                   │ Find anomalies IN   │ Detect if NEW       │
│                        │ the existing data   │ observations are    │
│                        │                     │ different           │
│ Setting                │ Transductive        │ Inductive           │
│                        │ (same data)         │ (new data)          │
│ sklearn term           │ outlier detection   │ novelty detection   │
│ Example                │ "Which existing     │ "Is this NEW        │
│                        │ transactions are    │ transaction         │
│                        │ fraudulent?"        │ fraudulent?"        │
└────────────────────────┴─────────────────────┴─────────────────────┘
```

---

## 🌍 Real-World Examples by Type

| Type | Domain | Example |
|------|--------|---------|
| **Point** | Finance | $50K transaction on card with $200 avg |
| **Point** | Manufacturing | Sensor reading 10x normal range |
| **Point** | Health | Blood pressure reading of 250/150 |
| **Contextual** | Energy | High AC usage in winter |
| **Contextual** | Retail | Low sales on Black Friday |
| **Contextual** | Network | Low traffic during business hours |
| **Collective** | Finance | Many small transactions just under reporting threshold |
| **Collective** | Security | Sequence of failed login attempts |
| **Collective** | Health | Irregular heartbeat pattern (arrhythmia) |

---

## 📊 Anomaly Detection Methods Overview

```
                      ANOMALY DETECTION METHODS
                      ─────────────────────────
                              │
              ┌───────────────┼───────────────┐
              │               │               │
        Statistical      ML-Based       Deep Learning
              │               │               │
        ┌─────┴─────┐   ┌────┴────┐    ┌─────┴─────┐
        │           │   │         │    │           │
     Z-score    Grubbs' │    Isolation│   Autoencoder│
     IQR       test    │    Forest  │   LSTM-AE   │
     Mahalanobis       │           │   GAN-based  │
                  One-Class   LOF  │              │
                  SVM    k-NN     DBSCAN          │
                                              (Ch 9.6)
                              │
              Covered in this unit: Ch 9.2-9.6
```

---

## 🐍 Python: Quick Overview

```python
from sklearn.ensemble import IsolationForest
from sklearn.svm import OneClassSVM
from sklearn.neighbors import LocalOutlierFactor
from sklearn.covariance import EllipticEnvelope
import numpy as np

# Generate normal data with anomalies
np.random.seed(42)
X_normal = np.random.randn(200, 2)
X_anomalies = np.random.uniform(low=-4, high=4, size=(10, 2))
X = np.vstack([X_normal, X_anomalies])
y_true = np.array([1]*200 + [-1]*10)  # 1=normal, -1=anomaly

# Quick comparison of methods
methods = {
    'Isolation Forest': IsolationForest(contamination=0.05, random_state=42),
    'One-Class SVM': OneClassSVM(nu=0.05, kernel='rbf'),
    'LOF': LocalOutlierFactor(n_neighbors=20, contamination=0.05),
    'Elliptic Envelope': EllipticEnvelope(contamination=0.05),
}

for name, clf in methods.items():
    if name == 'LOF':
        y_pred = clf.fit_predict(X)
    else:
        y_pred = clf.fit_predict(X)
    
    n_anomalies = (y_pred == -1).sum()
    print(f"{name:20s}: detected {n_anomalies} anomalies")
```

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| **Anomaly** | Data point deviating significantly from expected pattern |
| **Point Anomaly** | Single outlier point |
| **Contextual Anomaly** | Normal value in wrong context |
| **Collective Anomaly** | Group of points anomalous together |
| **Supervised** | Requires labeled normal + anomaly data |
| **Semi-supervised** | Only normal data for training |
| **Unsupervised** | No labels; assumes anomalies are rare |
| **Novelty Detection** | Detect anomalies in NEW data (clean training) |
| **Outlier Detection** | Find anomalies in EXISTING data |
| **Key Challenge** | Extreme class imbalance (anomalies are rare) |

---

## ❓ Revision Questions

1. **Define the three types of anomalies (point, contextual, collective) and provide a real-world example of each from a domain of your choice.**
   Ensure examples clearly demonstrate why each qualifies as its respective type.

2. **Explain the difference between supervised, semi-supervised, and unsupervised anomaly detection. When would you use each approach?**
   Consider label availability and the types of anomalies you need to detect.

3. **What is the difference between anomaly detection and novelty detection? How does this affect the choice of sklearn methods?**
   Discuss transductive vs. inductive settings and training data assumptions.

4. **Why is anomaly detection inherently challenging compared to standard classification?**
   Discuss class imbalance, lack of labeled data, evolving anomaly types, and evaluation difficulties.

5. **A hospital wants to detect unusual patient vital sign patterns. Which type(s) of anomaly might be relevant, and what detection approach would you recommend?**
   Consider that both individual readings and sequences of readings could be anomalous.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Association Rules Applications](../08-Association-Rules/05-applications.md) | [📂 Unsupervised Learning](../README.md) | [Statistical Methods →](./02-statistical-methods.md) |

---

> **Key Takeaway:** Anomalies come in three flavors — point (individual outliers), contextual (wrong context), and collective (anomalous groups). Most real-world anomaly detection is unsupervised or semi-supervised because labeled anomaly data is scarce. The right approach depends on your data type, label availability, and the kind of anomalies you expect to find.
