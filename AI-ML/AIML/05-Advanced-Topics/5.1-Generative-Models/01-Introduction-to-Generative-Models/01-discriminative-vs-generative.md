# Discriminative vs Generative Models

## Overview

The fundamental distinction in machine learning is between **discriminative** models (which learn decision boundaries between classes) and **generative** models (which learn the underlying data distribution to generate new samples). Understanding this distinction is essential for knowing when and why to use generative approaches.

---

## Core Difference

```
Discriminative: Learns P(y|x)
  "Given this input, what's the label?"
  Models the boundary between classes
  
Generative: Learns P(x) or P(x,y) = P(x|y) × P(y)
  "What does the data look like?"
  Models the full data distribution
  Can generate NEW data!

  ┌─────────────────────────────────────────────────┐
  │                                                   │
  │  Discriminative         Generative                │
  │                                                   │
  │    ○ ○ ○ │ × × ×        ○ ○ ○   × × ×           │
  │   ○ ○ ○  │  × × ×      ○ ○ ○ ○  × × × ×        │
  │    ○ ○ ○ │ × × ×        ○ ○ ○   × × ×           │
  │          │               (cloud)  (cloud)         │
  │    Learns the          Learns how each             │
  │    boundary            class is distributed        │
  │                                                   │
  └─────────────────────────────────────────────────┘
```

---

## Mathematical Formulation

```
Given data x and labels y:

Discriminative:
  Model P(y|x) directly
  
  Examples:
    Logistic Regression: P(y=1|x) = σ(wᵀx + b)
    Neural Network:      P(y|x) = softmax(f_θ(x))
    SVM:                 y = sign(wᵀx + b)
  
  Only needs to find the decision boundary
  Doesn't model how x is distributed

Generative:
  Model P(x) or P(x|y)
  
  Can classify using Bayes' rule:
    P(y|x) = P(x|y) × P(y) / P(x)
  
  But ALSO can:
    Sample new x ~ P(x)     ← Generate new data!
    Evaluate P(x)            ← Density estimation
    Detect anomalies         ← Low P(x) = unusual

  ┌────────────────────────────────────────────────┐
  │ Generative models can do everything             │
  │ discriminative models can do (classify),        │
  │ PLUS generate, evaluate density, and more.     │
  │                                                  │
  │ But: discriminative models are usually BETTER   │
  │ at pure classification (they focus on boundary) │
  └────────────────────────────────────────────────┘
```

---

## Comparison

```
  Aspect              Discriminative       Generative
  ──────              ──────────────       ──────────
  Learns              P(y|x)               P(x) or P(x,y)
  Goal                Classification       Data distribution
  Can generate?       No                   Yes ✓
  Can classify?       Yes (primary)        Yes (via Bayes)
  Density estimate?   No                   Yes ✓
  Anomaly detection?  Limited              Yes ✓
  Training data       Labeled (x,y)        Often unlabeled (x)
  Accuracy            Higher (focused)     Lower (broader goal)
  Examples            Logistic Reg, SVM,   Naive Bayes, GMM,
                      Neural Networks,     VAE, GAN, Diffusion,
                      Random Forest        GPT, Flow models

  Key insight:
    Discriminative: "I don't know what a cat looks like,
                     but I can tell cats from dogs"
    Generative:     "I know what cats look like,
                     so I can draw one AND recognize one"
```

---

## Explicit vs Implicit Generative Models

```
Explicit density: model defines P(x) you can evaluate
  
  Tractable density:
    Autoregressive: P(x) = Π P(x_i|x_{<i})
    Normalizing Flows: exact likelihood via change of variables
  
  Approximate density:
    VAE: lower bound on P(x) via ELBO
    Boltzmann machines: approximate sampling

Implicit density: model generates samples but can't evaluate P(x)
  
    GAN: generates x, but no P(x) formula
    Diffusion: generates x via iterative denoising

  ┌──────────────────────────────────────────────┐
  │ Generative Models                             │
  │ ├── Explicit density                          │
  │ │   ├── Tractable: Autoregressive, Flows     │
  │ │   └── Approximate: VAE, Boltzmann          │
  │ └── Implicit density                          │
  │     ├── GAN                                   │
  │     └── Diffusion models                      │
  └──────────────────────────────────────────────┘
```

---

## When to Use Which

```
Use Discriminative when:
  ✓ Pure classification or regression
  ✓ Labeled data available
  ✓ Accuracy is the primary metric
  ✓ Don't need to generate new data

Use Generative when:
  ✓ Need to create new content (images, text, audio)
  ✓ Anomaly detection (model normal → flag abnormal)
  ✓ Data augmentation (generate more training data)
  ✓ Semi-supervised learning (few labels + lots of unlabeled)
  ✓ Understanding data distribution
  ✓ Handling missing data
```

---

## Revision Questions

1. **What is the fundamental difference between P(y|x) and P(x)?**
2. **Why are discriminative models typically better at classification?**
3. **What capabilities do generative models have that discriminative models lack?**
4. **What is the difference between explicit and implicit density models?**
5. **Give an example where a generative model is preferred over discriminative.**

---

[Next: 02-types-of-generative-models.md](02-types-of-generative-models.md) | [Back to README](../README.md)
