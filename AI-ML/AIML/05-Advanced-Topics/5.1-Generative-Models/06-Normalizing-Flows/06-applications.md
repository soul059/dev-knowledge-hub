# Normalizing Flows Applications

## Overview

Normalizing flows' unique properties — exact likelihood computation, exact inference, and invertible mappings — make them valuable beyond image generation. This chapter surveys applications across density estimation, variational inference, speech synthesis, anomaly detection, and scientific computing, highlighting scenarios where flows' strengths give them clear advantages over VAEs and GANs.

---

## Density Estimation

```
Flows provide EXACT log-likelihoods:

  Standard task: given dataset, estimate p(x)
  
  Flows are natural density estimators:
    • Train by maximizing log p(x)
    • No approximation (unlike VAE's ELBO)
    • No adversarial training needed
    
  Applications:
    • Financial modeling (risk, pricing)
    • Scientific data analysis
    • Compression (better density → better compression)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Method       Density   Quality   Stable?        │
  │  ─────────    ───────   ───────   ──────         │
  │  KDE          Exact     Poor      Yes            │
  │  GMM          Exact     Limited   Yes            │
  │  VAE          ELBO      OK        Yes            │
  │  GAN          None!     Great     No             │
  │  Flow         Exact     Good      Yes            │
  │  Autoregr.    Exact     Good      Yes (slow gen) │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Variational Inference Enhancement

```
Flows improve VAE posterior approximation:

  Standard VAE: q(z|x) = N(μ, σ²)  (simple Gaussian)
  Flow-enhanced: q(z|x) = Flow(N(μ, σ²))  (complex posterior!)

  z₀ ~ N(μ(x), σ²(x))
  z₁ = f₁(z₀)
  z₂ = f₂(z₁)
  ...
  z_K = f_K(z_{K-1})  ← this is the final latent code

  ELBO becomes:
    E[log p(x|z_K)] - KL(q_K(z_K|x) || p(z_K))
    
  KL term uses change of variables through the flow

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Gaussian q(z|x):    ○  (limited expressiveness) │
  │                                                    │
  │  Flow-enhanced q:    ◈  (arbitrary shape!)       │
  │                                                    │
  │  Better posterior → tighter ELBO → better model  │
  │                                                    │
  │  Key papers:                                      │
  │  • Normalizing Flows (Rezende & Mohamed, 2015)   │
  │  • IAF-VAE (Kingma et al., 2016)                │
  │  • Sylvester Flows (van den Berg et al., 2018)   │
  └──────────────────────────────────────────────────┘
```

---

## Speech Synthesis

```
WaveGlow: Flow-based speech synthesis (NVIDIA, 2019)

  Input: mel spectrogram (conditioning)
  Output: raw audio waveform (22kHz, 16-bit)
  
  Architecture:
    • 12 coupling layers
    • Conditioned on mel spectrogram
    • WaveNet-like dilated convolutions inside coupling networks
    
  Advantages over WaveNet:
    • Real-time synthesis (25× faster than WaveNet)
    • Parallel generation (not autoregressive!)
    • Same quality as WaveNet
    
  Advantages over GAN-based TTS:
    • Stable training (no mode collapse)
    • Exact likelihood → can measure quality

  FloWaveNet: another flow-based TTS model
    • Similar approach, similar quality
```

---

## Anomaly Detection

```
Flows assign exact log-likelihoods to test data:

  High log p(x) → normal data
  Low log p(x)  → anomalous!

  Train flow on normal data → compute threshold

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  log p(x):                                       │
  │                                                    │
  │  Normal data:     log p ≈ -500   (high)          │
  │  Novel/anomaly:   log p ≈ -2000  (low!)          │
  │                                                    │
  │  Set threshold τ:                                │
  │  if log p(x) < τ → flag as anomaly              │
  │                                                    │
  │  Caveat: flows can assign high likelihood        │
  │  to out-of-distribution data sometimes!          │
  │  (Nalisnick et al., 2019 — CIFAR/SVHN issue)    │
  │  Need typicality test or other corrections       │
  └──────────────────────────────────────────────────┘

  Applications:
    • Manufacturing defect detection
    • Medical imaging (abnormal scans)
    • Network intrusion detection
    • Fraud detection
```

---

## Image Manipulation

```
Glow-style latent space manipulation:

  1. Encode real image: z = f⁻¹(x)
  2. Modify z in meaningful direction
  3. Decode back: x' = f(z')

  Attribute editing:
    z' = z + α × direction_vector

  Applications:
    • Face editing (age, expression, hair)
    • Style transfer
    • Image completion
    • Super-resolution

  Advantage over GAN-based editing:
    • Perfect reconstruction (invertible!)
    • No mode dropping
    • Quantifiable: know exact latent representation
```

---

## Scientific Computing

```
Flows model complex probability distributions in physics:

  1. Lattice Field Theory
     • Sample field configurations
     • Replace expensive MCMC with flow sampling
     • Orders of magnitude speedup

  2. Molecular Dynamics
     • Boltzmann distribution sampling
     • Free energy estimation
     • Protein folding distributions

  3. Cosmology
     • Posterior inference for cosmological parameters
     • Gravitational wave analysis
     • Galaxy simulation

  4. Bayesian Neural Networks
     • Flow-based weight posteriors
     • Better uncertainty estimation

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Why flows for science?                          │
  │                                                    │
  │  • Exact probabilities (needed for physics!)     │
  │  • Invertible (can go data ↔ latent)            │
  │  • Can compute expectations analytically         │
  │  • No mode collapse (cover full distribution)    │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Data Compression

```
Flows enable optimal compression via bits-back coding:

  Theorem: optimal code length = -log₂ p(x) bits
  
  Flow gives exact p(x) → optimal compression!

  Practical compression with flows:
    • Encode x → z = f⁻¹(x)
    • Quantize z
    • Compress quantized z (Gaussian → simple)
    • Transmit compressed z
    • Receiver: decompress z, compute x = f(z)

  Results:
    • Competitive with learned image codecs
    • Lossless compression possible
    • Better than PNG/JPEG for specific domains
```

---

## Python: Anomaly Detection with Flow

```python
import torch
import torch.nn as nn

class FlowAnomalyDetector:
    def __init__(self, flow_model, threshold_percentile=5):
        self.flow = flow_model
        self.threshold = None
        self.percentile = threshold_percentile
    
    def fit(self, normal_data_loader):
        """Train flow on normal data and compute threshold."""
        optimizer = torch.optim.Adam(self.flow.parameters(), lr=1e-3)
        
        # Train
        for epoch in range(100):
            for batch in normal_data_loader:
                loss = -self.flow.log_prob(batch).mean()
                optimizer.zero_grad(); loss.backward(); optimizer.step()
        
        # Compute threshold from training data log-likelihoods
        all_log_probs = []
        self.flow.eval()
        with torch.no_grad():
            for batch in normal_data_loader:
                lp = self.flow.log_prob(batch)
                all_log_probs.append(lp)
        
        all_lp = torch.cat(all_log_probs)
        self.threshold = torch.quantile(all_lp, self.percentile / 100)
    
    def predict(self, x):
        """Returns True for anomalies."""
        self.flow.eval()
        with torch.no_grad():
            log_prob = self.flow.log_prob(x)
        return log_prob < self.threshold
    
    def score(self, x):
        """Anomaly score (lower = more anomalous)."""
        self.flow.eval()
        with torch.no_grad():
            return self.flow.log_prob(x)
```

---

## Summary Table

| Application | Why Flows? | Alternative | Flow Advantage |
|-------------|-----------|-------------|----------------|
| Density estimation | Exact likelihood | KDE, GMM | Scales to high dim |
| VAE enhancement | Flexible posterior | Mean-field | Tighter ELBO |
| Speech synthesis | Parallel sampling | WaveNet | Real-time speed |
| Anomaly detection | Probability scores | Autoencoders | Principled threshold |
| Image editing | Perfect inversion | GAN inversion | Exact reconstruction |
| Scientific computing | Exact probabilities | MCMC | Much faster |
| Compression | Optimal code length | Learned codecs | Theoretically optimal |

---

## Revision Questions

1. **Why are flows better than VAEs for density estimation?**
2. **How do flows improve variational inference in VAEs?**
3. **What makes flow-based speech synthesis faster than autoregressive models?**
4. **What is the limitation of using flow likelihood for anomaly detection?**
5. **Why are flows particularly valuable in scientific computing?**

---

[Previous: 05-glow.md](05-glow.md) | [Next: ../07-Diffusion-Models/01-diffusion-process.md](../07-Diffusion-Models/01-diffusion-process.md)
