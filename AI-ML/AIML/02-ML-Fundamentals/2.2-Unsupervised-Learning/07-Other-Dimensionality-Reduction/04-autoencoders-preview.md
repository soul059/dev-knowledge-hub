# 🧬 Autoencoders for Dimensionality Reduction (Preview)

> **Chapter 7.4 — Neural Network-Based Nonlinear Dimensionality Reduction**

---

## 📌 Overview

An **autoencoder** is a neural network trained to **reconstruct its own input** through a compressed intermediate representation (the **bottleneck**). By forcing data through a lower-dimensional bottleneck, the network learns a compact, nonlinear representation of the data — making autoencoders a powerful tool for **nonlinear dimensionality reduction**.

### Architecture Overview

```
┌────────────────────────────────────────────────────────────────────┐
│                                                                    │
│   INPUT          ENCODER      BOTTLENECK      DECODER      OUTPUT  │
│   (d dims)       (layers)     (k dims)        (layers)     (d dims)│
│                                                                    │
│   x₁ ─┐     ┌──── h₁ ────┐                ┌──── h₁'────┐  ┌─ x̂₁  │
│   x₂ ─┤     │             │    z₁          │            │  ├─ x̂₂  │
│   x₃ ─┼──►  │   Hidden    ├──► z₂ ──►     │   Hidden   ├──┤  x̂₃  │
│   x₄ ─┤     │   Layers    │    z₃          │   Layers   │  ├─ x̂₄  │
│   x₅ ─┘     └─────────────┘                └────────────┘  └─ x̂₅  │
│                                                                    │
│   ◄─── d ───►              ◄── k ──►              ◄─── d ───►     │
│                                                                    │
│              ENCODER              │           DECODER               │
│         f(x) → z                  │       g(z) → x̂                 │
│                                                                    │
│   GOAL: x̂ ≈ x  (minimize reconstruction error)                    │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Core Concept

### The Idea

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  1. COMPRESS: Encoder maps high-D input to low-D code        │
│                                                              │
│     x ∈ ℝᵈ  ──encoder──►  z ∈ ℝᵏ    (where k << d)         │
│                                                              │
│  2. RECONSTRUCT: Decoder maps low-D code back to high-D      │
│                                                              │
│     z ∈ ℝᵏ  ──decoder──►  x̂ ∈ ℝᵈ                            │
│                                                              │
│  3. LEARN: Train to minimize reconstruction error             │
│                                                              │
│     Loss = ‖x - x̂‖²  (MSE between input and output)         │
│                                                              │
│  RESULT: The bottleneck z is a useful low-D representation!  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Mathematical Formulation

```
Encoder:   z = f_θ(x)        (parameterized by θ)
Decoder:   x̂ = g_φ(z)        (parameterized by φ)
Loss:      L(θ, φ) = Σᵢ ‖xᵢ - g_φ(f_θ(xᵢ))‖²

Optimization:
  θ*, φ* = argmin  Σᵢ ‖xᵢ - g_φ(f_θ(xᵢ))‖²
            θ,φ
```

```
  Training Process:
  
  Iteration 1:  x ──► [encoder] ──► z ──► [decoder] ──► x̂   Loss = 5.2
  Iteration 10: x ──► [encoder] ──► z ──► [decoder] ──► x̂   Loss = 2.1
  Iteration 100: x ──► [encoder] ──► z ──► [decoder] ──► x̂  Loss = 0.3
                                                              ▲
                                                              │
                                                         x̂ ≈ x !
```

---

## 🔄 Autoencoder vs PCA

### Linear Autoencoder = PCA

A remarkable result: **a single-layer linear autoencoder with MSE loss recovers the same subspace as PCA!**

```
Linear Autoencoder:
  z = Wx        (encoder: linear, no activation)
  x̂ = W'z       (decoder: linear, no activation)
  
  Solution: W spans the same subspace as top-k PCA components

┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Linear AE with k units  ≡  PCA with k components           │
│                                                              │
│  But: Nonlinear AE can capture CURVED manifolds that         │
│       PCA cannot!                                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Why Go Nonlinear?

```
  Data on a Curve (Swiss Roll):

  PCA (linear):                    Autoencoder (nonlinear):
  ┌──────────────────┐            ┌──────────────────┐
  │     ╱╲            │            │                  │
  │    ╱  ╲           │   ────►   │  ●●●●●●●●●●●●●   │
  │   ╱    ╲──►       │            │  (unfolded!)     │
  │  ╱ overlaps!      │            │                  │
  └──────────────────┘            └──────────────────┘

  PCA can only find linear         AE can learn the nonlinear
  subspaces → lossy for            mapping to "unroll" the
  curved manifolds                 manifold
```

### Comparison Table

```
┌────────────────────────┬─────────────────┬──────────────────────┐
│      Aspect            │      PCA        │    Autoencoder       │
├────────────────────────┼─────────────────┼──────────────────────┤
│ Mapping type           │ Linear          │ Nonlinear            │
│ Closed-form solution   │ Yes (SVD)       │ No (gradient descent)│
│ Training required      │ No              │ Yes (neural network) │
│ Computation cost       │ Low             │ High (GPU helpful)   │
│ Interpretability       │ High (loadings) │ Low (black box)      │
│ Hyperparameters        │ n_components    │ Architecture, lr, etc│
│ Overfitting risk       │ None            │ Yes (need regularize)│
│ Handles nonlinearity   │ No              │ Yes                  │
│ Inverse transform      │ Exact           │ Approximate          │
│ Deterministic          │ Yes             │ No (random init)     │
│ New point embedding    │ Yes             │ Yes (encoder.predict)│
└────────────────────────┴─────────────────┴──────────────────────┘
```

---

## 🏗️ Types of Autoencoders

### 1. Vanilla (Undercomplete) Autoencoder

The basic architecture where the bottleneck has **fewer dimensions** than the input.

```
  Input(784) → Dense(256) → Dense(64) → Dense(32) → Dense(64) → Dense(256) → Output(784)
              ╰──── Encoder ────╯  ╰bottleneck╯  ╰──── Decoder ────╯
```

### 2. Deep Autoencoder

Multiple hidden layers for more complex representations:

```
  Input(784)
    │
    ▼
  Dense(512, ReLU)    ─┐
  Dense(256, ReLU)     │ Encoder
  Dense(128, ReLU)     │
  Dense(32)           ─┘  ← Bottleneck (latent space)
    │
    ▼
  Dense(128, ReLU)    ─┐
  Dense(256, ReLU)     │ Decoder
  Dense(512, ReLU)     │
  Dense(784, Sigmoid) ─┘
    │
    ▼
  Output (reconstructed)
```

### 3. Convolutional Autoencoder

For image data — uses Conv/Pool layers for encoder and Conv/Upsample for decoder:

```
  Image(28×28×1)
    │
  Conv2D(32, 3×3) → MaxPool(2×2) → Conv2D(64, 3×3) → MaxPool(2×2)
    │                                                        │
    ▼                                                        ▼
  Flatten → Dense(32)  ← Bottleneck
    │
  Reshape → Conv2DT(64, 3×3) → UpSample(2×2) → Conv2DT(32, 3×3) → UpSample(2×2)
    │
  Output(28×28×1)
```

### 4. Sparse Autoencoder

Adds a **sparsity constraint** on the bottleneck activations:

```
Loss = ‖x - x̂‖² + λ · Σ|zⱼ|     (L1 regularization)
                   or
Loss = ‖x - x̂‖² + β · KL(ρ ‖ ρ̂ⱼ)  (KL sparsity penalty)

Forces most bottleneck neurons to be near zero → learns features!
```

### 5. Denoising Autoencoder

Trained to reconstruct **clean** input from **corrupted** input:

```
  x̃ = x + noise     (add Gaussian noise or set random features to 0)
  
  x̃ ──► [encoder] ──► z ──► [decoder] ──► x̂
  
  Loss = ‖x - x̂‖²   (compare with CLEAN x, not noisy x̃)
  
  → Learns robust features that ignore noise
```

### 6. Variational Autoencoder (VAE) — Preview

The VAE doesn't encode to a single point but to a **probability distribution**:

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  Standard AE:                                                    │
│    x ──► encoder ──► z (point in latent space) ──► decoder ──► x̂ │
│                                                                  │
│  VAE:                                                            │
│    x ──► encoder ──► μ, σ² (distribution parameters)             │
│                        │                                         │
│                        ▼                                         │
│                    z ~ N(μ, σ²)  (sample from distribution)      │
│                        │                                         │
│                        ▼                                         │
│                    decoder ──► x̂                                  │
│                                                                  │
│  Loss = Reconstruction + KL(q(z|x) ‖ p(z))                      │
│                          │                                       │
│                     Regularization (pushes toward N(0,1))        │
│                                                                  │
│  BENEFIT: Smooth, continuous latent space → can GENERATE         │
│           new data by sampling from latent space!                │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

```
  Standard AE Latent Space:         VAE Latent Space:
  ┌──────────────────┐              ┌──────────────────┐
  │    ●              │              │   ●●●●●●●●       │
  │         ●         │              │   ●●●●●●●●       │
  │  ●                │              │   ●●●●●●●●       │
  │            ●      │              │   ●●●●●●●●       │
  │      ●            │              │                  │
  │ Sparse, holes     │              │ Dense, smooth    │
  │ → can't sample    │              │ → can sample!    │
  └──────────────────┘              └──────────────────┘
```

---

## 🐍 Python Implementation

### Basic Autoencoder with Keras/TensorFlow

```python
import numpy as np
import matplotlib.pyplot as plt
from tensorflow import keras
from tensorflow.keras import layers
from sklearn.datasets import load_digits
from sklearn.preprocessing import MinMaxScaler

# Load and preprocess data
digits = load_digits()
X = digits.data
y = digits.target

scaler = MinMaxScaler()
X_scaled = scaler.fit_transform(X)

# ══════════════════════════════════════
# BUILD AUTOENCODER
# ══════════════════════════════════════

input_dim = X_scaled.shape[1]  # 64
encoding_dim = 2               # Bottleneck dimension (for visualization)

# Encoder
encoder_input = keras.Input(shape=(input_dim,))
encoded = layers.Dense(32, activation='relu')(encoder_input)
encoded = layers.Dense(16, activation='relu')(encoded)
encoded = layers.Dense(encoding_dim, activation='linear')(encoded)

encoder = keras.Model(encoder_input, encoded, name='encoder')

# Decoder
decoder_input = keras.Input(shape=(encoding_dim,))
decoded = layers.Dense(16, activation='relu')(decoder_input)
decoded = layers.Dense(32, activation='relu')(decoded)
decoded = layers.Dense(input_dim, activation='sigmoid')(decoded)

decoder = keras.Model(decoder_input, decoded, name='decoder')

# Full autoencoder
autoencoder_input = keras.Input(shape=(input_dim,))
z = encoder(autoencoder_input)
reconstructed = decoder(z)
autoencoder = keras.Model(autoencoder_input, reconstructed, name='autoencoder')

# Compile and train
autoencoder.compile(optimizer='adam', loss='mse')
history = autoencoder.fit(
    X_scaled, X_scaled,        # Input = Target (reconstruction)
    epochs=200,
    batch_size=32,
    validation_split=0.2,
    verbose=0
)

# ══════════════════════════════════════
# VISUALIZE LATENT SPACE
# ══════════════════════════════════════

# Encode all data to 2D
X_encoded = encoder.predict(X_scaled)

plt.figure(figsize=(10, 8))
scatter = plt.scatter(X_encoded[:, 0], X_encoded[:, 1],
                      c=y, cmap='tab10', s=15, alpha=0.7)
plt.colorbar(scatter, ticks=range(10), label='Digit')
plt.title('Autoencoder Latent Space (2D)')
plt.xlabel('Latent Dimension 1')
plt.ylabel('Latent Dimension 2')
plt.tight_layout()
plt.show()

# ══════════════════════════════════════
# VISUALIZE RECONSTRUCTIONS
# ══════════════════════════════════════

X_reconstructed = autoencoder.predict(X_scaled)

fig, axes = plt.subplots(2, 10, figsize=(15, 3))
for i in range(10):
    # Original
    axes[0, i].imshow(X_scaled[i].reshape(8, 8), cmap='gray')
    axes[0, i].set_title(f'{y[i]}')
    axes[0, i].axis('off')
    # Reconstructed
    axes[1, i].imshow(X_reconstructed[i].reshape(8, 8), cmap='gray')
    axes[1, i].axis('off')

axes[0, 0].set_ylabel('Original')
axes[1, 0].set_ylabel('Reconstructed')
plt.suptitle('Original vs Reconstructed Digits')
plt.tight_layout()
plt.show()

# Training loss
plt.figure(figsize=(8, 4))
plt.plot(history.history['loss'], label='Train Loss')
plt.plot(history.history['val_loss'], label='Val Loss')
plt.xlabel('Epoch')
plt.ylabel('MSE Loss')
plt.title('Autoencoder Training')
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()
```

### Comparing AE vs PCA

```python
from sklearn.decomposition import PCA

# PCA to 2D
pca = PCA(n_components=2)
X_pca = pca.fit_transform(X_scaled)

# Autoencoder to 2D (already computed above)
# X_encoded = encoder.predict(X_scaled)

fig, axes = plt.subplots(1, 2, figsize=(16, 6))

axes[0].scatter(X_pca[:, 0], X_pca[:, 1], c=y, cmap='tab10', s=10, alpha=0.6)
axes[0].set_title('PCA (2D)')
axes[0].set_xlabel('PC1')
axes[0].set_ylabel('PC2')

axes[1].scatter(X_encoded[:, 0], X_encoded[:, 1], c=y, cmap='tab10', s=10, alpha=0.6)
axes[1].set_title('Autoencoder (2D)')
axes[1].set_xlabel('Latent 1')
axes[1].set_ylabel('Latent 2')

plt.suptitle('PCA vs Autoencoder: Dimensionality Reduction to 2D')
plt.tight_layout()
plt.show()

# Reconstruction error comparison
X_pca_reconstructed = pca.inverse_transform(X_pca)
pca_mse = np.mean((X_scaled - X_pca_reconstructed) ** 2)
ae_mse = np.mean((X_scaled - X_reconstructed) ** 2)

print(f"PCA Reconstruction MSE:         {pca_mse:.6f}")
print(f"Autoencoder Reconstruction MSE: {ae_mse:.6f}")
```

---

## 🏭 Applications

| Domain | Application |
|--------|-------------|
| **Image Compression** | Reduce image size while preserving quality |
| **Anomaly Detection** | High reconstruction error → anomaly |
| **Denoising** | Remove noise from images/signals |
| **Feature Learning** | Learn representations for downstream tasks |
| **Generative Models** | VAE generates new data samples |
| **Recommendation** | Learn user/item latent factors |
| **Drug Discovery** | Learn molecular representations |

---

## 📋 Summary Table

| Concept | Details |
|---------|---------|
| **Architecture** | Encoder → Bottleneck → Decoder |
| **Objective** | Minimize reconstruction error ‖x - x̂‖² |
| **Type** | Nonlinear dimensionality reduction |
| **Relation to PCA** | Linear AE ≡ PCA; nonlinear AE goes beyond |
| **Variants** | Vanilla, Deep, Sparse, Denoising, Convolutional, Variational |
| **Key Advantage** | Can learn complex nonlinear manifolds |
| **Key Disadvantage** | Requires training, more hyperparameters, less interpretable |
| **New Points** | Yes — pass through encoder |
| **Framework** | TensorFlow/Keras, PyTorch |
| **Link to Deep Learning** | Foundation for VAEs, GANs, and representation learning |

---

## ❓ Revision Questions

1. **What is the fundamental difference between PCA and a nonlinear autoencoder for dimensionality reduction?**
   Discuss linear vs. nonlinear mappings and the types of manifolds each can capture.

2. **Prove (or explain intuitively) why a linear autoencoder with MSE loss learns the same subspace as PCA.**
   Consider the loss function and the optimal weight matrices.

3. **What is the bottleneck layer, and why is it critical for learning useful representations?**
   Discuss information compression and the information bottleneck principle.

4. **Compare and contrast a standard autoencoder with a variational autoencoder (VAE). What does the VAE add, and why?**
   Discuss the distributional latent space, KL regularization, and generation capability.

5. **Design an autoencoder architecture for MNIST (28×28 images). Specify layer sizes, activations, and loss function.**
   Consider the tradeoff between bottleneck size and reconstruction quality.

6. **What are the main drawbacks of using autoencoders compared to simpler methods like PCA or UMAP?**
   Discuss training complexity, hyperparameter sensitivity, and interpretability.

---

## 🧭 Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← UMAP](./03-umap.md) | [📂 Unsupervised Learning](../README.md) | [Method Comparison →](./05-comparison-of-methods.md) |

---

> **Key Takeaway:** Autoencoders extend dimensionality reduction to the nonlinear regime using neural networks. While more powerful than PCA, they require careful architecture design, training, and regularization. They serve as the foundation for advanced generative models like VAEs, covered in the deep learning module.
