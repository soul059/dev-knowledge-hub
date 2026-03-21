# Training Challenges

## Overview

GAN training is notoriously difficult. The adversarial dynamic can lead to **mode collapse** (generator produces limited variety), **training instability** (oscillating or diverging losses), **vanishing gradients** (generator gets no learning signal), and **evaluation difficulties** (hard to know when training is "done"). Understanding these problems is essential for diagnosing and fixing GAN training.

---

## Mode Collapse

```
Mode collapse: G learns to produce only a FEW types of outputs

  Real data: dogs, cats, birds, fish, horses
  Mode collapsed G: dogs, dogs, dogs, dogs, dogs
  
  G found ONE output that fools D → keeps producing it!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Real distribution:      Generator output:        │
  │                                                    │
  │    ●     ●               ●●●●●                    │
  │   ● ●   ● ●                                      │
  │  ●   ● ●   ●            (all same mode!)         │
  │   ● ●   ● ●                                      │
  │    ●     ●                                        │
  │  (multi-modal)           (collapsed to 1 mode)    │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Types of mode collapse:
  
  FULL collapse: generates one image (or near-identical)
    → Very obvious, easy to detect
    
  PARTIAL collapse: generates a few modes, misses others
    → Harder to detect, need diversity metrics

  Why it happens:
    G finds a safe "winning strategy" against current D
    D adapts → G shifts to another mode → cycle
    In the worst case, G keeps switching between a few modes
```

---

## Training Instability

```
Oscillation and divergence:

  ┌──────────────────────────────────────────────────┐
  │  Stable training:          Unstable training:    │
  │                                                    │
  │  Loss ▲                    Loss ▲                 │
  │       │╲  ╱╲               │    ╱╲       ╱╲     │
  │       │ ╲╱  ╲─────        │   ╱  ╲     ╱  ╲╱╲ │
  │       │       ─────        │  ╱    ╲   ╱       │
  │       └──────────►         │ ╱      ╲╱╱        │
  │                            └──────────────►     │
  │  Converging                Oscillating/diverging │
  └──────────────────────────────────────────────────┘

  Causes:
  1. Learning rate too high → overshooting
  2. D too strong → G gets no gradient
  3. D too weak → G gets misleading gradients
  4. Architecture mismatch → one dominates
  5. Batch normalization issues
```

---

## Vanishing Gradients

```
When D is too good, G's gradients vanish:

  D(G(z)) ≈ 0  for all z  (D perfectly identifies fakes)
  
  Original loss: log(1 - D(G(z))) ≈ log(1) = 0
  Gradient: ≈ 0  →  G can't learn!

  ┌──────────────────────────────────────────────────┐
  │ D is too confident → G gradient = 0              │
  │                                                    │
  │ Solutions:                                        │
  │   1. Non-saturating loss: -log(D(G(z)))          │
  │   2. WGAN loss: meaningful gradients always       │
  │   3. Don't train D too much                      │
  │   4. Label smoothing: D targets = 0.9 not 1.0   │
  │   5. Instance noise: add noise to D's inputs     │
  └──────────────────────────────────────────────────┘
```

---

## Non-Convergence

```
GANs don't have a well-defined convergence criterion:

  Classification: loss decreases → model improves → converge
  GAN: losses oscillate → hard to tell if training or failing!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  D loss ≈ 0.7 → could be:                       │
  │    (a) Perfect equilibrium (great!)               │
  │    (b) Both networks are bad (terrible!)          │
  │                                                    │
  │  Can't tell from losses alone!                   │
  │  Must inspect generated samples regularly         │
  │                                                    │
  │  Best practice: save samples every N iterations   │
  │  FID score periodically to track actual quality   │
  └──────────────────────────────────────────────────┘

  GAN training is more like "art" than "science"
  Lots of hyperparameter tuning and visual inspection
```

---

## Diagnosing Problems

```
Symptom → Diagnosis → Fix

  D loss → 0 quickly:
    D is too strong
    → Reduce D capacity, fewer D steps, add noise

  G loss → 0 quickly:
    G fooled D trivially (rare)
    → Increase D capacity, more D steps

  Both losses oscillate wildly:
    Learning rate too high
    → Lower LR, gradient clipping

  G produces identical outputs:
    Mode collapse
    → Use minibatch discrimination, unrolled GAN,
       Wasserstein loss, or spectral normalization

  G produces noise/artifacts:
    Training diverged
    → Restart with lower LR, different architecture

  Checkerboard artifacts:
    ConvTranspose2d issue
    → Use upsample + conv instead
```

---

## Python: Training with Monitoring

```python
import torch
import torchvision.utils as vutils

def train_gan_with_monitoring(G, D, dataloader, epochs, latent_dim,
                               device, log_interval=100):
    opt_G = torch.optim.Adam(G.parameters(), lr=2e-4, betas=(0.5, 0.999))
    opt_D = torch.optim.Adam(D.parameters(), lr=2e-4, betas=(0.5, 0.999))
    criterion = torch.nn.BCELoss()
    
    # Fixed noise for tracking progress
    fixed_z = torch.randn(64, latent_dim, device=device)
    
    for epoch in range(epochs):
        for i, (real, _) in enumerate(dataloader):
            real = real.to(device)
            bs = real.size(0)
            ones = torch.ones(bs, 1, device=device)
            zeros = torch.zeros(bs, 1, device=device)
            
            # Train D
            z = torch.randn(bs, latent_dim, device=device)
            fake = G(z).detach()
            d_loss = criterion(D(real), ones * 0.9) + criterion(D(fake), zeros)
            opt_D.zero_grad(); d_loss.backward(); opt_D.step()
            
            # Train G
            z = torch.randn(bs, latent_dim, device=device)
            fake = G(z)
            g_loss = criterion(D(fake), ones)
            opt_G.zero_grad(); g_loss.backward(); opt_G.step()
            
            if i % log_interval == 0:
                # Monitor D's predictions
                d_real_mean = D(real).mean().item()
                d_fake_mean = D(fake.detach()).mean().item()
                
                print(f"[{epoch}/{epochs}][{i}] "
                      f"D_loss: {d_loss:.3f} G_loss: {g_loss:.3f} "
                      f"D(real): {d_real_mean:.3f} D(fake): {d_fake_mean:.3f}")
                
                # Detect problems
                if d_real_mean > 0.99 and d_fake_mean < 0.01:
                    print("⚠️ WARNING: D too strong, G may have vanishing gradients")
                if d_fake_mean > 0.95:
                    print("⚠️ WARNING: G too strong or D collapsed")
        
        # Save samples for visual inspection
        with torch.no_grad():
            samples = G(fixed_z).cpu()
        vutils.save_image(samples, f'samples_epoch_{epoch}.png',
                         normalize=True, nrow=8)
```

---

## Revision Questions

1. **What is mode collapse and how does it manifest?**
2. **Why does a strong discriminator cause vanishing gradients?**
3. **Why can't you rely on loss values to judge GAN training quality?**
4. **What are the signs that a GAN has diverged?**
5. **How do you diagnose whether the discriminator is too strong or too weak?**

---

[Previous: 04-gan-loss-functions.md](04-gan-loss-functions.md) | [Next: 06-gan-training-tips.md](06-gan-training-tips.md)
