# Proximal Policy Optimization (PPO)

## Overview

**PPO** is the most widely used policy gradient algorithm, designed as a simpler and more efficient alternative to TRPO. Instead of solving a constrained optimization problem, PPO clips the surrogate objective to prevent too-large policy updates. PPO achieves TRPO-level stability with first-order optimization (standard SGD/Adam), making it easy to implement and scale.

---

## Core Idea

```
TRPO:  max L(θ)  subject to  KL(π_old || π_new) ≤ δ
       → Complex: needs CG, FVP, line search

PPO:   max L_clip(θ)  (no constraint needed!)
       → Simple: just clip the objective, use Adam

  ┌──────────────────────────────────────────────────┐
  │ PPO keeps the policy close to old policy by      │
  │ CLIPPING the objective, not constraining KL.     │
  │                                                    │
  │ If the policy changes too much:                   │
  │   → the clipped term dominates                    │
  │   → gradient becomes zero for that sample         │
  │   → no further update in that direction            │
  └──────────────────────────────────────────────────┘
```

---

## PPO-Clip Objective

```
r_t(θ) = π_θ(a_t|s_t) / π_θ_old(a_t|s_t)    (importance ratio)

L^CLIP(θ) = E_t[ min(
    r_t(θ) × Â_t,                             (unclipped)
    clip(r_t(θ), 1-ε, 1+ε) × Â_t              (clipped)
)]

  ε = 0.2 (typical)  → ratio clipped to [0.8, 1.2]

  How it works:

  Case 1: Â > 0 (good action, want to increase probability)
    r > 1+ε:  clip! No more increase (already changed enough)
    r ≤ 1+ε:  normal gradient, increase probability
    
  Case 2: Â < 0 (bad action, want to decrease probability)
    r < 1-ε:  clip! No more decrease (already changed enough)
    r ≥ 1-ε:  normal gradient, decrease probability

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  L_clip when Â > 0:         L_clip when Â < 0:   │
  │                                                    │
  │  L ▲                        L ▲                   │
  │    │      ┌────── flat        │ flat ──────┐      │
  │    │     ╱                    │              ╲     │
  │    │    ╱                     │               ╲    │
  │    │   ╱                      │                ╲   │
  │    └──┼────────► r            └──┼────────► r  │  │
  │       1-ε 1 1+ε                 1-ε 1 1+ε     │  │
  │                                                    │
  │  Can't increase r          Can't decrease r       │
  │  beyond 1+ε                beyond 1-ε             │
  └──────────────────────────────────────────────────┘
```

---

## Full PPO Loss

```
Total loss = Policy loss + Value loss - Entropy bonus

  L = L^CLIP + c₁ × L^VF - c₂ × H(π)

  L^CLIP: Clipped surrogate (policy)
  L^VF:   Value function loss = (V(s) - V_target)²
  H(π):   Entropy bonus = -Σ π(a|s)log π(a|s)

  Typical hyperparameters:
    c₁ = 0.5       (value coefficient)
    c₂ = 0.01      (entropy coefficient)
    ε = 0.2         (clip range)
    γ = 0.99        (discount)
    λ = 0.95        (GAE lambda)
    K = 4           (epochs per batch)
    minibatch = 64  (minibatch size)
```

---

## PPO Algorithm

```
PPO with Clipped Objective:

  For iteration = 1, 2, ...:
    1. Run policy π_old in N parallel envs for T steps
       → Collect {s_t, a_t, r_t, s_{t+1}, done_t}
    
    2. Compute advantages using GAE(λ):
       Â_t = Σ_{k=0}^{∞} (γλ)^k δ_{t+k}
       
    3. Normalize advantages:
       Â ← (Â - μ) / (σ + ε)
    
    4. For epoch = 1 to K:
       Shuffle data into minibatches
       For each minibatch:
         Compute r_t(θ) = π_θ(a_t|s_t) / π_old(a_t|s_t)
         L_clip = min(r_t × Â_t, clip(r_t, 1-ε, 1+ε) × Â_t)
         L_value = (V_θ(s_t) - R_t)²
         L_entropy = -H(π_θ(·|s_t))
         
         L = -L_clip.mean() + 0.5 * L_value.mean() - 0.01 * L_entropy.mean()
         
         Gradient step on L

  Key: reuse same batch for K epochs (sample efficient)
  Old log probs are stored and fixed during updates
```

---

## PPO-Penalty (Alternative)

```
PPO-Penalty: adaptive KL penalty instead of clipping

  L^KLPEN = E[r_t × Â_t - β × KL(π_old || π_new)]

  β adapts automatically:
    if KL > 1.5 × δ_target: β ← 2β    (too much change, penalize more)
    if KL < δ_target / 1.5: β ← β/2   (too little change, allow more)

  Less common than PPO-Clip, but used in some implementations.
  PPO-Clip is simpler and usually works as well or better.
```

---

## Python: PPO Implementation

```python
import torch
import torch.nn as nn
import gymnasium as gym
import numpy as np

class PPOActorCritic(nn.Module):
    def __init__(self, n_obs, n_actions):
        super().__init__()
        self.shared = nn.Sequential(
            nn.Linear(n_obs, 64), nn.Tanh(),
            nn.Linear(64, 64), nn.Tanh())
        self.actor = nn.Linear(64, n_actions)
        self.critic = nn.Linear(64, 1)
    
    def forward(self, x):
        features = self.shared(x)
        dist = torch.distributions.Categorical(logits=self.actor(features))
        value = self.critic(features).squeeze(-1)
        return dist, value

def compute_gae(rewards, values, dones, gamma=0.99, lam=0.95):
    T = len(rewards)
    advantages = torch.zeros(T)
    last_gae = 0
    for t in reversed(range(T)):
        mask = 1.0 - dones[t]
        delta = rewards[t] + gamma * values[t+1] * mask - values[t]
        last_gae = delta + gamma * lam * mask * last_gae
        advantages[t] = last_gae
    return advantages, advantages + values[:T]

# Setup
env = gym.vector.make("CartPole-v1", num_envs=8)
model = PPOActorCritic(4, 2)
optimizer = torch.optim.Adam(model.parameters(), lr=3e-4)

# Hyperparameters
GAMMA, LAM = 0.99, 0.95
CLIP_EPS = 0.2
K_EPOCHS = 4
N_STEPS = 128
BATCH_SIZE = 256

states, _ = env.reset()

for update in range(1000):
    # --- Collect rollout ---
    all_states, all_actions, all_rewards = [], [], []
    all_dones, all_log_probs, all_values = [], [], []
    
    for step in range(N_STEPS):
        s_t = torch.FloatTensor(states)
        with torch.no_grad():
            dist, value = model(s_t)
        action = dist.sample()
        
        all_states.append(s_t)
        all_actions.append(action)
        all_log_probs.append(dist.log_prob(action))
        all_values.append(value)
        
        states, rewards, terms, truncs, _ = env.step(action.numpy())
        all_rewards.append(torch.FloatTensor(rewards))
        all_dones.append(torch.FloatTensor(terms | truncs))
    
    # Bootstrap
    with torch.no_grad():
        _, next_val = model(torch.FloatTensor(states))
    values = torch.stack(all_values + [next_val])
    
    # GAE
    advantages, returns = compute_gae(
        torch.stack(all_rewards), values,
        torch.stack(all_dones), GAMMA, LAM)
    
    # Flatten
    b_states = torch.cat(all_states)
    b_actions = torch.cat(all_actions)
    b_old_logp = torch.cat(all_log_probs).detach()
    b_advantages = advantages.reshape(-1)
    b_returns = returns.reshape(-1)
    
    # Normalize advantages
    b_advantages = (b_advantages - b_advantages.mean()) / (b_advantages.std() + 1e-8)
    
    # --- PPO update for K epochs ---
    for epoch in range(K_EPOCHS):
        indices = torch.randperm(len(b_states))
        for start in range(0, len(b_states), BATCH_SIZE):
            idx = indices[start:start + BATCH_SIZE]
            
            dist, value = model(b_states[idx])
            new_logp = dist.log_prob(b_actions[idx])
            entropy = dist.entropy().mean()
            
            # Importance ratio
            ratio = (new_logp - b_old_logp[idx]).exp()
            
            # Clipped surrogate
            adv = b_advantages[idx]
            surr1 = ratio * adv
            surr2 = torch.clamp(ratio, 1 - CLIP_EPS, 1 + CLIP_EPS) * adv
            policy_loss = -torch.min(surr1, surr2).mean()
            
            value_loss = (value - b_returns[idx]).pow(2).mean()
            
            loss = policy_loss + 0.5 * value_loss - 0.01 * entropy
            
            optimizer.zero_grad()
            loss.backward()
            nn.utils.clip_grad_norm_(model.parameters(), 0.5)
            optimizer.step()
```

---

## PPO Hyperparameter Guide

```
  Parameter      Default    Range           Notes
  ─────────      ───────    ─────           ─────
  clip ε         0.2        [0.1, 0.3]      Lower = more conservative
  γ              0.99       [0.95, 0.999]   Higher for longer horizons
  λ (GAE)        0.95       [0.9, 0.99]     Higher = less bias
  learning rate  3e-4       [1e-4, 1e-3]    Often annealed linearly
  K epochs       4          [3, 10]         More = more sample efficient
  minibatch      64         [32, 256]       Depends on batch size
  N envs         8-32       [4, 128]        More = more stable
  N steps        128-2048   [64, 4096]      Depends on episode length
  entropy coeff  0.01       [0, 0.05]       0 for deterministic envs
  grad clip      0.5        [0.5, 1.0]      Max gradient norm
```

---

## Revision Questions

1. **How does PPO-Clip prevent destructively large policy updates?**
2. **What does the importance ratio r_t(θ) represent?**
3. **Why can PPO reuse the same batch for multiple epochs?**
4. **What are the three components of the PPO loss function?**
5. **How does PPO compare to TRPO in terms of complexity and performance?**

---

[Previous: 01-trpo.md](01-trpo.md) | [Next: 03-sac.md](03-sac.md)
