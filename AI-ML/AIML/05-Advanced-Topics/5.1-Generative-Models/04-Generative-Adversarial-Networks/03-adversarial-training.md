# Adversarial Training

## Overview

**Adversarial training** is the process of simultaneously training the generator and discriminator through their competitive dynamic. Understanding the training dynamics — gradient flow, equilibrium conditions, and common failure patterns — is essential for successfully training GANs. This chapter dives into the mechanics of how G and D learn from each other.

---

## Training Algorithm

```
Algorithm: GAN Training

  Initialize G_θ and D_φ with random weights

  For each training iteration:
  
    ── Train Discriminator (1 or more steps) ──
    
    1. Sample real data: x ~ P_data  (minibatch of m)
    2. Sample noise:     z ~ P_z     (minibatch of m)
    3. Generate fakes:   x_fake = G(z)
    
    4. D loss = -[mean(log D(x)) + mean(log(1 - D(x_fake)))]
    5. Update φ by ascending: ∇_φ loss_D
    
    ── Train Generator (1 step) ──
    
    6. Sample noise: z ~ P_z (new minibatch)
    7. Generate fakes: x_fake = G(z)
    
    8. G loss = -mean(log D(G(z)))      (non-saturating)
       OR: G loss = mean(log(1 - D(G(z))))   (original)
    
    9. Update θ by descending: ∇_θ loss_G

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Key: G never sees real data directly!            │
  │  G only receives gradients THROUGH D              │
  │                                                    │
  │  D tells G: "this fake looks like/unlike real"    │
  │  G adjusts to make fakes D can't distinguish      │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Gradient Flow

```
How does G learn? Through D's gradients:

  Forward: z → G → x_fake → D → D(x_fake)
  
  Backward:
    ∂L_G/∂θ = ∂L_G/∂D(x_fake) × ∂D(x_fake)/∂x_fake × ∂x_fake/∂θ
              ─────────────────   ──────────────────   ───────────
              Loss gradient        D's gradient w.r.t   G's gradient
                                   its input             w.r.t params

  D provides the "teaching signal" to G:
    D's gradient tells G HOW to change x_fake to look more real
    
  ┌──────────────────────────────────────────────────┐
  │  z → [G_θ] → x_fake → [D_φ] → score            │
  │       ↑                  ↑                        │
  │   update G            FROZEN                     │
  │   (using D's          (don't update D            │
  │    gradients)          when training G)           │
  └──────────────────────────────────────────────────┘

  IMPORTANT: When training G, D's weights are FROZEN
             When training D, G's weights are FROZEN
             Never update both simultaneously!
```

---

## Training Dynamics

```
Early training:
  G produces garbage → D easily classifies
  D loss ≈ 0, G loss is high
  Risk: vanishing gradients for G

Mid training:
  G improves, D struggles more
  D loss increases, G loss decreases
  Both losses oscillate

Late training (ideal):
  G produces realistic data
  D loss ≈ log(2) ≈ 0.693 (random guessing)
  D(x) ≈ 0.5 for all x

  ┌──────────────────────────────────────────────────┐
  │  Loss ▲                                           │
  │       │  D loss                                   │
  │  0.7  │  ─────────────────────── ideal D loss    │
  │       │╲                       ╱                  │
  │       │ ╲       ╱╲      ╱╲   ╱                   │
  │       │  ╲    ╱    ╲  ╱    ╲╱                    │
  │       │   ╲ ╱       ╲╱                           │
  │       │    ╲                                      │
  │       │     G loss                                │
  │       └──────────────────────────► epoch          │
  │                                                    │
  │  Healthy training: losses oscillate, converge     │
  │  Unhealthy: one loss → 0, other → ∞              │
  └──────────────────────────────────────────────────┘
```

---

## Practical Training Code

```python
import torch
import torch.nn as nn

def train_gan_step(G, D, opt_G, opt_D, real_data, latent_dim, device):
    batch_size = real_data.size(0)
    real = real_data.to(device)
    
    # Labels
    real_label = torch.ones(batch_size, 1, device=device)
    fake_label = torch.zeros(batch_size, 1, device=device)
    criterion = nn.BCELoss()
    
    # ═══ Train Discriminator ═══
    z = torch.randn(batch_size, latent_dim, device=device)
    fake = G(z).detach()  # detach: don't backprop into G
    
    d_real = D(real)
    d_fake = D(fake)
    
    d_loss_real = criterion(d_real, real_label * 0.9)  # label smoothing
    d_loss_fake = criterion(d_fake, fake_label)
    d_loss = d_loss_real + d_loss_fake
    
    opt_D.zero_grad()
    d_loss.backward()
    opt_D.step()
    
    # ═══ Train Generator ═══
    z = torch.randn(batch_size, latent_dim, device=device)
    fake = G(z)
    d_fake = D(fake)
    
    g_loss = criterion(d_fake, real_label)  # fool D
    
    opt_G.zero_grad()
    g_loss.backward()
    opt_G.step()
    
    return d_loss.item(), g_loss.item()
```

---

## Nash Equilibrium

```
Theoretical convergence:

  At Nash equilibrium:
    P_G(x) = P_data(x)     (G perfectly models data)
    D(x) = 0.5  ∀x         (D can't distinguish)

  In practice:
    • GAN training rarely reaches true equilibrium
    • Losses oscillate rather than converge
    • G and D "chase" each other
    • We stop when samples look good (empirical)

  Convergence is NOT guaranteed for deep GANs!
  This is a fundamental challenge of adversarial training.
```

---

## Revision Questions

1. **How does the generator receive learning signal if it never sees real data?**
2. **Why must we freeze one network while training the other?**
3. **What does a D loss of ~0.693 indicate about training?**
4. **What is label smoothing and why is it used in GAN training?**
5. **Why don't GANs guarantee convergence to Nash equilibrium?**

---

[Previous: 02-generator-and-discriminator.md](02-generator-and-discriminator.md) | [Next: 04-gan-loss-functions.md](04-gan-loss-functions.md)
