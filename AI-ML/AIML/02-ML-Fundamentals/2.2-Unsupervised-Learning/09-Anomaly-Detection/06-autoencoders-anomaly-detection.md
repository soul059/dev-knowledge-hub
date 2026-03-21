# 🧬 Autoencoders for Anomaly Detection

> **Chapter 9.6 — Reconstruction Error as an Anomaly Signal**

---

## 📌 Overview

**Autoencoders** can be repurposed for anomaly detection by exploiting a simple idea: train the network to reconstruct **normal data**, then use **high reconstruction error** as an anomaly signal. Since the autoencoder learns the patterns of normal data, anomalies — which follow different patterns — will be poorly reconstructed.

### Core Idea

```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  TRAINING (normal data only):                                       │
│                                                                      │
│  Normal x ──► [Encoder] ──► z ──► [Decoder] ──► x̂ ≈ x              │
│                                                  ▲                   │
│                                       Low error! │ ← Learned to     │
│                                                      reconstruct    │
│                                                                      │
│  INFERENCE (test data):                                              │
│                                                                      │
│  Normal x ──► [Encoder] ──► z ──► [Decoder] ──► x̂ ≈ x  ✅ Low error│
│  Anomaly a ──► [Encoder] ──► z ──► [Decoder] ──► â ≠ a  🚨 HIGH error│
│                                                                      │
│  Decision: error > threshold → ANOMALY                               │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Why Autoencoders Work for Anomaly Detection

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  The autoencoder learns a COMPRESSED representation of       │
│  NORMAL data patterns.                                       │
│                                                              │
│  Normal data:                                                │
│  • Fits the learned patterns → easy to reconstruct           │
│  • Reconstruction error is LOW                               │
│                                                              │
│  Anomalous data:                                             │
│  • Doesn't fit learned patterns → hard to reconstruct        │
│  • Reconstruction error is HIGH                              │
│                                                              │
│  Analogy:                                                    │
│  • You learn to draw cats (training on cat images)           │
│  • Given a cat → you draw it well (low error)                │
│  • Given a dog → you draw it badly (high error)              │
│  • The "bad drawing" reveals it's NOT a cat!                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Reconstruction Error Distribution

```
  Number of
  Samples
  ▲
  │    Normal data
  │   ┌──────────┐
  │   │          │
  │   │          │    Anomalies
  │   │          │   ┌───┐
  │   │          │   │   │
  │   │          │   │   │
  │───┴──────────┴───┴───┴─────► Reconstruction Error
  │                  ▲
  │              Threshold
  │
  │  Low error       │  High error
  │  = Normal        │  = Anomaly
```

---

## 🏗️ Architecture Design

### Basic Autoencoder for Anomaly Detection

```
  Input Layer (d features)
       │
       ▼
  Dense(128, ReLU)     ─┐
  Dense(64, ReLU)       │ Encoder
  Dense(32, ReLU)       │
  Dense(16)            ─┘  ← Bottleneck
       │
       ▼
  Dense(32, ReLU)      ─┐
  Dense(64, ReLU)       │ Decoder
  Dense(128, ReLU)      │
  Dense(d, Sigmoid)    ─┘  ← Reconstruction
       │
       ▼
  Output Layer (d features)

  Loss = MSE(input, output) = (1/d) Σ(xᵢ - x̂ᵢ)²
```

### Architecture Guidelines

```
┌──────────────────────────────────────────────────────────────┐
│                ARCHITECTURE GUIDELINES                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│ 1. BOTTLENECK SIZE:                                          │
│    • Too large → AE copies input (learns identity)           │
│    • Too small → Loses important patterns                    │
│    • Rule of thumb: 5-30% of input dimension                │
│                                                              │
│ 2. DEPTH:                                                    │
│    • Deeper → captures more complex patterns                 │
│    • Too deep → harder to train, risk of overfitting         │
│    • 2-4 layers per side typically sufficient                │
│                                                              │
│ 3. SYMMETRY:                                                 │
│    • Encoder and decoder usually mirror each other           │
│    • Not strictly required but helps convergence             │
│                                                              │
│ 4. ACTIVATION:                                               │
│    • Hidden layers: ReLU (or LeakyReLU)                      │
│    • Output: Sigmoid (if input in [0,1]) or Linear           │
│                                                              │
│ 5. REGULARIZATION:                                           │
│    • Dropout, L2, or early stopping to prevent overfitting   │
│    • Critical: AE must NOT memorize normal data              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 📊 The Complete Pipeline

```
╔══════════════════════════════════════════════════════════════════╗
║          AUTOENCODER ANOMALY DETECTION PIPELINE                ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  1. DATA PREPARATION                                             ║
║     • Collect data (assumed mostly normal)                       ║
║     • Split: train (normal only) / validation / test             ║
║     • Normalize features (StandardScaler or MinMaxScaler)        ║
║                                                                  ║
║  2. TRAIN AUTOENCODER                                            ║
║     • Train on NORMAL data only                                  ║
║     • Loss = MSE or MAE                                          ║
║     • Monitor validation loss for early stopping                 ║
║                                                                  ║
║  3. COMPUTE RECONSTRUCTION ERRORS                                ║
║     • Pass all data through trained AE                           ║
║     • error(x) = ‖x - AE(x)‖²                                  ║
║                                                                  ║
║  4. SET THRESHOLD                                                ║
║     • Method A: Percentile (e.g., 95th or 99th percentile       ║
║       of training reconstruction errors)                        ║
║     • Method B: Mean + k·std of training errors                  ║
║     • Method C: Visual inspection of error distribution          ║
║     • Method D: Optimize on validation set (if labels exist)     ║
║                                                                  ║
║  5. CLASSIFY                                                     ║
║     • error(x) > threshold → ANOMALY                            ║
║     • error(x) ≤ threshold → NORMAL                             ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## 🐍 Python Implementation

### Complete Pipeline with Keras

```python
import numpy as np
import matplotlib.pyplot as plt
from tensorflow import keras
from tensorflow.keras import layers
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import train_test_split
from sklearn.metrics import precision_recall_curve, roc_auc_score

# ═══════════════════════════════════════
# 1. Generate Data
# ═══════════════════════════════════════
np.random.seed(42)

# Normal data: 2D Gaussian mixture
n_normal = 1000
X_normal = np.vstack([
    np.random.randn(500, 10) * 0.5 + 1,
    np.random.randn(500, 10) * 0.5 - 1,
])

# Anomalies: uniform random
n_anomalies = 50
X_anomalies = np.random.uniform(-5, 5, (n_anomalies, 10))

# Labels (for evaluation only)
y_normal = np.zeros(n_normal)
y_anomalies = np.ones(n_anomalies)

# ═══════════════════════════════════════
# 2. Prepare Data
# ═══════════════════════════════════════
# Train only on normal data
X_train, X_val = train_test_split(X_normal, test_size=0.2, random_state=42)

# Scale
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_val_scaled = scaler.transform(X_val)
X_test = np.vstack([X_normal[-100:], X_anomalies])
X_test_scaled = scaler.transform(X_test)
y_test = np.concatenate([np.zeros(100), np.ones(n_anomalies)])

# ═══════════════════════════════════════
# 3. Build Autoencoder
# ═══════════════════════════════════════
input_dim = X_train_scaled.shape[1]  # 10
encoding_dim = 3                      # Bottleneck

# Encoder
inputs = keras.Input(shape=(input_dim,))
encoded = layers.Dense(32, activation='relu')(inputs)
encoded = layers.Dense(16, activation='relu')(encoded)
encoded = layers.Dense(encoding_dim, activation='relu')(encoded)

# Decoder
decoded = layers.Dense(16, activation='relu')(encoded)
decoded = layers.Dense(32, activation='relu')(decoded)
decoded = layers.Dense(input_dim, activation='linear')(decoded)

autoencoder = keras.Model(inputs, decoded)
autoencoder.compile(optimizer='adam', loss='mse')

autoencoder.summary()

# ═══════════════════════════════════════
# 4. Train
# ═══════════════════════════════════════
history = autoencoder.fit(
    X_train_scaled, X_train_scaled,  # Input = Target
    epochs=100,
    batch_size=32,
    validation_data=(X_val_scaled, X_val_scaled),
    callbacks=[keras.callbacks.EarlyStopping(patience=10, restore_best_weights=True)],
    verbose=0
)

# ═══════════════════════════════════════
# 5. Compute Reconstruction Errors
# ═══════════════════════════════════════
def reconstruction_error(model, X):
    """Compute per-sample MSE reconstruction error."""
    X_pred = model.predict(X, verbose=0)
    return np.mean((X - X_pred) ** 2, axis=1)

# Errors on training data (for threshold)
train_errors = reconstruction_error(autoencoder, X_train_scaled)

# Errors on test data
test_errors = reconstruction_error(autoencoder, X_test_scaled)

# ═══════════════════════════════════════
# 6. Set Threshold
# ═══════════════════════════════════════
# Method: 95th percentile of training errors
threshold = np.percentile(train_errors, 95)
print(f"Threshold (95th percentile): {threshold:.4f}")

# Alternative: mean + 2*std
threshold_std = train_errors.mean() + 2 * train_errors.std()
print(f"Threshold (mean + 2σ): {threshold_std:.4f}")

# ═══════════════════════════════════════
# 7. Classify and Evaluate
# ═══════════════════════════════════════
y_pred = (test_errors > threshold).astype(int)

from sklearn.metrics import classification_report, confusion_matrix
print("\nClassification Report:")
print(classification_report(y_test, y_pred, target_names=['Normal', 'Anomaly']))

print("Confusion Matrix:")
print(confusion_matrix(y_test, y_pred))

# ROC-AUC
auc = roc_auc_score(y_test, test_errors)
print(f"\nROC-AUC: {auc:.4f}")

# ═══════════════════════════════════════
# 8. Visualization
# ═══════════════════════════════════════
fig, axes = plt.subplots(1, 3, figsize=(18, 5))

# Training loss
axes[0].plot(history.history['loss'], label='Train')
axes[0].plot(history.history['val_loss'], label='Validation')
axes[0].set_xlabel('Epoch')
axes[0].set_ylabel('MSE Loss')
axes[0].set_title('Training History')
axes[0].legend()

# Error distribution
axes[1].hist(test_errors[y_test == 0], bins=30, alpha=0.7, 
             label='Normal', color='blue')
axes[1].hist(test_errors[y_test == 1], bins=15, alpha=0.7, 
             label='Anomaly', color='red')
axes[1].axvline(threshold, color='black', linestyle='--', 
                label=f'Threshold={threshold:.3f}')
axes[1].set_xlabel('Reconstruction Error')
axes[1].set_ylabel('Count')
axes[1].set_title('Error Distribution')
axes[1].legend()

# Precision-Recall curve
precision, recall, thresholds_pr = precision_recall_curve(y_test, test_errors)
axes[2].plot(recall, precision)
axes[2].set_xlabel('Recall')
axes[2].set_ylabel('Precision')
axes[2].set_title(f'Precision-Recall Curve (AUC={auc:.3f})')

plt.tight_layout()
plt.show()
```

---

## 🔧 Threshold Selection Strategies

```
┌──────────────────────────────────────────────────────────────┐
│              THRESHOLD SELECTION METHODS                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. PERCENTILE-BASED                                         │
│     threshold = np.percentile(train_errors, 95)              │
│     + Simple, no labels needed                               │
│     - Assumes fixed contamination rate                       │
│                                                              │
│  2. STATISTICS-BASED                                         │
│     threshold = mean(errors) + k * std(errors)               │
│     k = 2 (95.4%), k = 3 (99.7%)                            │
│     + Principled, adapts to error distribution               │
│     - Assumes Gaussian errors                                │
│                                                              │
│  3. VALIDATION-BASED (if some labels exist)                  │
│     Optimize F1-score or ROC-AUC on validation set           │
│     + Best performance                                       │
│     - Requires labeled anomalies                             │
│                                                              │
│  4. VISUAL / ELBOW                                           │
│     Plot sorted errors, find the "elbow"                     │
│     + Intuitive                                              │
│     - Subjective                                             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## ⚖️ Comparison with Other Methods

```
┌──────────────────┬──────────────┬──────────────┬──────────────┐
│     Aspect       │ Autoencoder  │ Isolation    │ One-Class    │
│                  │ AD           │ Forest       │ SVM          │
├──────────────────┼──────────────┼──────────────┼──────────────┤
│ Approach         │ Reconstruction│ Isolation   │ Boundary     │
│ Linearity        │ Nonlinear    │ Nonlinear   │ Nonlinear    │
│ Training data    │ Normal only  │ Mixed       │ Normal only  │
│ Handles images   │ ✅ (CNN-AE)  │ ❌          │ ❌           │
│ Handles sequences│ ✅ (LSTM-AE) │ ❌          │ ❌           │
│ Feature learning │ ✅ Yes       │ ❌ No       │ ❌ No        │
│ Scalability      │ GPU needed   │ Very fast   │ O(N²)        │
│ Interpretability │ Recon error  │ Path length │ Decision fn  │
│ Hyperparameters  │ Many         │ Few         │ Few          │
│ Threshold        │ Must set     │ Built-in    │ ν parameter  │
│ Domain-specific  │ ✅ Yes       │ ❌ Generic  │ ❌ Generic   │
│ Best for         │ Complex data │ Tabular     │ Small data   │
│                  │ (img, text)  │ data        │              │
└──────────────────┴──────────────┴──────────────┴──────────────┘
```

---

## 🏭 Applications

| Domain | Application | Architecture |
|--------|-------------|-------------|
| **Manufacturing** | Defect detection in products | CNN Autoencoder on images |
| **Cybersecurity** | Network intrusion detection | Dense AE on packet features |
| **Finance** | Fraud detection | Dense AE on transaction features |
| **IoT/Sensors** | Equipment failure prediction | LSTM AE on time series |
| **Healthcare** | Abnormal ECG detection | 1D CNN AE on signals |
| **Video** | Surveillance anomaly detection | 3D CNN AE on video frames |

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| **Approach** | Train AE on normal data; high reconstruction error = anomaly |
| **Training** | Normal data only (semi-supervised) |
| **Score** | Reconstruction error: ‖x - AE(x)‖² |
| **Threshold** | Percentile, statistics-based, or validation-based |
| **Advantages** | Handles complex data (images, sequences), nonlinear |
| **Disadvantages** | Many hyperparameters, needs GPU, threshold selection |
| **Best For** | Complex data types where other methods fail |
| **Variants** | CNN-AE (images), LSTM-AE (time series), VAE |
| **Framework** | TensorFlow/Keras, PyTorch |

---

## ❓ Revision Questions

1. **Explain why an autoencoder trained on normal data produces high reconstruction error for anomalies.**
   Discuss the bottleneck constraint and learned data manifold.

2. **Design an autoencoder architecture for anomaly detection on a dataset with 50 features. Specify layer sizes, activations, and loss function.**
   Justify your bottleneck size and regularization choices.

3. **Compare three threshold selection strategies. Which would you use when you have (a) no labeled anomalies, (b) a small validation set with labels?**
   Discuss the trade-offs between precision and recall for each approach.

4. **An autoencoder achieves very low reconstruction error on BOTH normal and anomalous data. What might be wrong, and how would you fix it?**
   Consider bottleneck size, overfitting, and training data contamination.

5. **Compare autoencoder-based anomaly detection with Isolation Forest for tabular data. When would you choose each?**
   Discuss data complexity, computational resources, and interpretability.

6. **How would you adapt the autoencoder approach for time-series anomaly detection (e.g., sensor data)?**
   Discuss LSTM autoencoders, sliding windows, and temporal reconstruction errors.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← Local Outlier Factor](./05-local-outlier-factor.md) | [📂 Unsupervised Learning](../README.md) | [Clustering Evaluation →](../10-Clustering-Evaluation/01-internal-metrics.md) |

---

> **Key Takeaway:** Autoencoder-based anomaly detection leverages the principle that a model trained to compress and reconstruct normal data will fail to reconstruct anomalies. The reconstruction error serves as a natural anomaly score. This approach shines on complex data types (images, sequences, text) where traditional methods struggle, but requires careful architecture design and threshold selection.
