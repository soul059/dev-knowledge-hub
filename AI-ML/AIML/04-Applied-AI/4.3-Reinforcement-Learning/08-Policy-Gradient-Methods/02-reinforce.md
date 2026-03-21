# REINFORCE Algorithm

## Overview

**REINFORCE** (Williams, 1992) is the simplest policy gradient algorithm. It collects complete episodes, computes returns, and uses them to estimate the policy gradient. Despite high variance, it is the foundation for all modern policy gradient methods.

---

## Algorithm

```
REINFORCE (Monte Carlo Policy Gradient):

  Initialize policy network π(a|s; θ) with random θ
  Set learning rate α

  For each episode:
    1. Generate episode: s₀,a₀,r₀, s₁,a₁,r₁, ..., s_T,a_T,r_T
       (follow π(·|s; θ))
    
    2. For each step t = 0, 1, ..., T:
       Compute return: G_t = Σ_{k=t}^{T} γ^(k-t) r_k
    
    3. Update: θ ← θ + α Σ_t γ^t ∇_θ log π(a_t|s_t; θ) × G_t

  Equivalent loss for gradient descent frameworks:
    L(θ) = -Σ_t log π(a_t|s_t; θ) × G_t
    
  Minimize L → maximize expected return

  ┌─────────────────────────────────────────────┐
  │  Collect       Compute       Update         │
  │  episode  →   returns  →   parameters      │
  │                                              │
  │  s₀,a₀,r₀    G₀ = r₀+γr₁+...  θ ← θ+α∇J  │
  │  s₁,a₁,r₁    G₁ = r₁+γr₂+...              │
  │  ...          ...                            │
  │  s_T,a_T,r_T  G_T = r_T                     │
  └─────────────────────────────────────────────┘
```

---

## Step-by-Step Example

```
CartPole episode (simplified):

  Step  State        Action  Reward  Return (γ=0.99)
  0     [0.01,...]   Right   1       G₀ = 1+0.99+0.98+0.97 = 3.94
  1     [0.03,...]   Left    1       G₁ = 1+0.99+0.98 = 2.97
  2     [0.02,...]   Right   1       G₂ = 1+0.99 = 1.99
  3     [0.04,...]   Left    1       G₃ = 1

  Gradient update:
    ∇J ≈ ∇log π(Right|s₀) × 3.94
        + ∇log π(Left|s₁)  × 2.97
        + ∇log π(Right|s₂) × 1.99
        + ∇log π(Left|s₃)  × 1.00
    
    All returns positive → ALL actions reinforced
    But early actions reinforced MORE (higher G)
    → Makes sense: early actions affect more future reward
```

---

## High Variance Problem

```
REINFORCE has high variance because:

  1. G_t depends on entire future trajectory
     Slight randomness → very different G values
     
  2. All returns might be positive
     Even bad actions get reinforced (just less)
     
  Episode 1: G = [10, 8, 5]     → all actions reinforced ✓
  Episode 2: G = [2, 1, 0.5]    → all actions reinforced ✓ (less)
  Episode 3: G = [15, 12, 8]    → all actions reinforced ✓ (most)
  
  Gradient estimate is noisy:
    Var[∇J] is HIGH → slow, unstable learning
    
  Solutions:
    - Subtract baseline (see 03-baseline-variance-reduction.md)
    - Use more episodes per update
    - Normalize returns
```

---

## Python Implementation

```python
import torch
import torch.nn as nn
import torch.optim as optim
import torch.distributions as D
import gymnasium as gym
import numpy as np

class PolicyNet(nn.Module):
    def __init__(self, n_obs, n_actions):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(n_obs, 128), nn.ReLU(),
            nn.Linear(128, 128), nn.ReLU(),
            nn.Linear(128, n_actions)
        )
    
    def forward(self, x):
        return D.Categorical(logits=self.net(x))

def reinforce(env_name="CartPole-v1", n_episodes=1000, gamma=0.99, lr=1e-3):
    env = gym.make(env_name)
    n_obs = env.observation_space.shape[0]
    n_act = env.action_space.n
    
    policy = PolicyNet(n_obs, n_act)
    optimizer = optim.Adam(policy.parameters(), lr=lr)
    
    for episode in range(n_episodes):
        log_probs = []
        rewards = []
        
        state, _ = env.reset()
        done = False
        
        # 1. Collect episode
        while not done:
            state_t = torch.FloatTensor(state)
            dist = policy(state_t)
            action = dist.sample()
            log_probs.append(dist.log_prob(action))
            
            state, reward, terminated, truncated, _ = env.step(action.item())
            rewards.append(reward)
            done = terminated or truncated
        
        # 2. Compute returns
        returns = []
        G = 0
        for r in reversed(rewards):
            G = r + gamma * G
            returns.insert(0, G)
        returns = torch.FloatTensor(returns)
        
        # Normalize returns (variance reduction trick)
        returns = (returns - returns.mean()) / (returns.std() + 1e-8)
        
        # 3. Compute loss and update
        loss = 0
        for log_prob, G_t in zip(log_probs, returns):
            loss -= log_prob * G_t
        
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        
        if episode % 50 == 0:
            print(f"Episode {episode}, "
                  f"Total reward: {sum(rewards):.0f}")

reinforce()
```

---

## Properties

| Property | REINFORCE |
|----------|-----------|
| Type | On-policy, Monte Carlo |
| Requires | Complete episodes |
| Variance | High |
| Bias | Unbiased gradient estimate |
| Convergence | Guaranteed (to local optimum) |
| Action space | Discrete and continuous |
| Sample efficiency | Low (one episode per update) |

---

## Revision Questions

1. **What are the three steps of the REINFORCE algorithm?**
2. **Why must REINFORCE wait for complete episodes before updating?**
3. **Why does REINFORCE have high variance?**
4. **What does normalizing returns accomplish?**
5. **Why is REINFORCE considered an unbiased gradient estimator?**

---

[Previous: 01-policy-gradient-theorem.md](01-policy-gradient-theorem.md) | [Next: 03-baseline-variance-reduction.md](03-baseline-variance-reduction.md)
