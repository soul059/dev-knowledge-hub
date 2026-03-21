# Generalized Advantage Estimation (GAE)

## Overview

**Generalized Advantage Estimation (GAE)** provides a unified framework for computing advantage estimates by blending n-step returns using an exponentially-weighted average. It introduces a single parameter λ ∈ [0,1] that smoothly interpolates between high-bias/low-variance (TD) and low-bias/high-variance (MC) estimates. GAE is used in virtually all modern policy gradient methods including PPO.

---

## The Bias-Variance Spectrum

```
The core problem: How many steps of actual rewards to use?

  1-step:  Â = r_t + γV(s_{t+1}) - V(s_t)
           Low variance, HIGH bias (depends on V accuracy)

  2-step:  Â = r_t + γr_{t+1} + γ²V(s_{t+2}) - V(s_t)
           More variance, less bias

  n-step:  Â = Σ_{k=0}^{n-1} γ^k r_{t+k} + γ^n V(s_{t+n}) - V(s_t)
           
  ∞-step:  Â = G_t - V(s_t)     (Monte Carlo)
           HIGH variance, no bias

  ┌───────────────────────────────────────────┐
  │ Bias   ██████████░░░░░░░░░░░░  ←→  ░░░░  │
  │ Var    ░░░░░░░░░░██████████████████████████│
  │        1-step                    ∞-step   │
  │           TD(0)                    MC     │
  │                                           │
  │ GAE(λ): pick any point on this spectrum!  │
  └───────────────────────────────────────────┘
```

---

## TD Residual

```
The building block of GAE is the TD residual:

  δ_t = r_t + γV(s_{t+1}) - V(s_t)

  δ_t is a 1-step advantage estimate:
    E[δ_t] = E[r_t + γV(s_{t+1})] - V(s_t)
            = Q(s_t, a_t) - V(s_t)    (approximately)
            = A(s_t, a_t)

  The n-step advantage can be written as sum of TD residuals:
    Â_t^(1) = δ_t
    Â_t^(2) = δ_t + γδ_{t+1}
    Â_t^(3) = δ_t + γδ_{t+1} + γ²δ_{t+2}
    Â_t^(n) = Σ_{k=0}^{n-1} γ^k δ_{t+k}
```

---

## GAE Formula

```
GAE combines ALL n-step estimates with exponential weighting:

  Â_t^GAE(λ) = (1-λ)(Â_t^(1) + λÂ_t^(2) + λ²Â_t^(3) + ...)
  
  Simplifies to:
  
  ┌─────────────────────────────────────────────────┐
  │                                                   │
  │  Â_t^GAE(λ) = Σ_{k=0}^{∞} (γλ)^k × δ_{t+k}    │
  │                                                   │
  │  where δ_t = r_t + γV(s_{t+1}) - V(s_t)         │
  │                                                   │
  └─────────────────────────────────────────────────┘

  Special cases:
    λ = 0:  Â = δ_t                    (TD(0), high bias, low variance)
    λ = 1:  Â = Σ γ^k δ_{t+k} = G_t - V(s_t)  (MC, no bias, high variance)
    λ = 0.95-0.99: sweet spot for most tasks

  Compare with TD(λ):
    GAE(λ) for policy gradients ≈ TD(λ) for value learning
    Same λ parameter, same interpolation idea
```

---

## Efficient Computation

```
Computing GAE recursively (backwards through trajectory):

  Given trajectory: s_0, a_0, r_0, ..., s_T

  δ_t = r_t + γV(s_{t+1}) - V(s_t)    for each t

  Compute backwards:
    Â_T = 0
    Â_{T-1} = δ_{T-1}
    Â_{T-2} = δ_{T-2} + γλ × Â_{T-1}
    ...
    Â_t = δ_t + γλ × Â_{t+1}          ← recursive formula

  This is O(T) — very efficient!

  Pseudocode:
    advantages[T] = 0
    for t in range(T-1, -1, -1):
        delta = rewards[t] + gamma * values[t+1] * (1-done[t]) - values[t]
        advantages[t] = delta + gamma * lam * (1-done[t]) * advantages[t+1]
```

---

## GAE with PPO (Standard Pipeline)

```
Modern RL training loop:

  1. Collect N steps across M parallel environments
  2. Compute GAE advantages for all transitions
  3. Normalize advantages: Â ← (Â - μ) / (σ + ε)
  4. Run K epochs of minibatch PPO updates
  5. Repeat

  ┌──────────────────────────────────────────┐
  │ Collect    →  Compute GAE  →  PPO Update │
  │ M envs        λ=0.95          K epochs   │
  │ N steps       γ=0.99          clip=0.2   │
  │               normalize       minibatch  │
  └──────────────────────────────────────────┘
```

---

## Python Implementation

```python
import torch
import numpy as np

def compute_gae(rewards, values, dones, gamma=0.99, lam=0.95):
    """
    Compute Generalized Advantage Estimation.
    
    Args:
        rewards: [T] tensor of rewards
        values:  [T+1] tensor of value estimates (includes bootstrap)
        dones:   [T] tensor of done flags
        gamma:   discount factor
        lam:     GAE lambda parameter
    
    Returns:
        advantages: [T] tensor
        returns:    [T] tensor (advantages + values)
    """
    T = len(rewards)
    advantages = torch.zeros(T)
    last_gae = 0
    
    for t in reversed(range(T)):
        next_non_terminal = 1.0 - dones[t]
        delta = rewards[t] + gamma * values[t + 1] * next_non_terminal - values[t]
        advantages[t] = last_gae = delta + gamma * lam * next_non_terminal * last_gae
    
    returns = advantages + values[:T]
    return advantages, returns


# Vectorized version for multiple parallel environments
def compute_gae_vectorized(rewards, values, dones, gamma=0.99, lam=0.95):
    """
    Vectorized GAE for N parallel environments.
    
    rewards: [T, N]
    values:  [T+1, N]
    dones:   [T, N]
    """
    T, N = rewards.shape
    advantages = torch.zeros(T, N)
    last_gae = torch.zeros(N)
    
    for t in reversed(range(T)):
        mask = 1.0 - dones[t]
        delta = rewards[t] + gamma * values[t + 1] * mask - values[t]
        last_gae = delta + gamma * lam * mask * last_gae
        advantages[t] = last_gae
    
    returns = advantages + values[:T]
    return advantages, returns


# Example usage
T = 100      # trajectory length
gamma = 0.99
lam = 0.95

rewards = torch.randn(T)
values = torch.randn(T + 1)
dones = torch.zeros(T)  # no episode boundaries

advantages, returns = compute_gae(rewards, values, dones, gamma, lam)

# Normalize advantages (standard practice)
advantages = (advantages - advantages.mean()) / (advantages.std() + 1e-8)
```

---

## Effect of λ

```
λ = 0.0:   Only uses immediate TD error
  - Fast learning when V is accurate
  - Very biased when V is poor
  - Result: δ_t only

λ = 0.5:   Medium blend
  - Moderate bias and variance

λ = 0.95:  Common default (PPO, A2C)
  - Near-MC but with some smoothing
  - Good balance for most tasks

λ = 1.0:   Full Monte Carlo return minus baseline
  - No bias but high variance
  - Rarely used in practice

  ┌──────────────────────────────────────────┐
  │ λ     Effective horizon   Typical use     │
  │ 0.0   1 step              Simple envs     │
  │ 0.9   ~10 steps           Medium envs     │
  │ 0.95  ~20 steps           PPO default     │
  │ 0.99  ~100 steps          Long horizons   │
  │ 1.0   Full episode        Rare            │
  └──────────────────────────────────────────┘
  
  Effective horizon ≈ 1/(1 - γλ)
```

---

## Revision Questions

1. **What problem does GAE solve and how does it solve it?**
2. **How does the λ parameter control the bias-variance tradeoff?**
3. **Write the recursive formula for computing GAE.**
4. **What are the special cases for λ=0 and λ=1?**
5. **Why is advantage normalization important in practice?**

---

[Previous: 05-a2c-a3c.md](05-a2c-a3c.md) | [Back to README](../README.md)
