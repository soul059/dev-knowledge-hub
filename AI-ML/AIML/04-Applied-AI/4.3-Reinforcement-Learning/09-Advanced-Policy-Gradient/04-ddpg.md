# Deep Deterministic Policy Gradient (DDPG)

## Overview

**DDPG** extends DQN to continuous action spaces by learning a deterministic policy μ(s; θ) that directly outputs the action, combined with a Q-function critic Q(s,a; φ). It's essentially "DQN for continuous control" — an off-policy actor-critic that uses a replay buffer and target networks for stability.

---

## Why Not Just Discretize?

```
Continuous action spaces can't use DQN directly:

  DQN: a* = argmax_a Q(s, a)  → needs to enumerate all actions!
  
  Discrete (5 actions):  Easy, check Q for all 5
  Continuous (1D):       Infinite actions, can't enumerate
  Continuous (30D):      Robot with 30 joints → impossible!
  
  Solutions:
  1. Discretize: 10 bins × 30 joints = 10^30 actions → impossible!
  2. Optimize: gradient ascent on Q w.r.t. a → expensive per step
  3. DDPG: learn a function μ(s) that outputs the best action directly ✓
  
  ┌──────────────────────────────────────────────────┐
  │ DQN:  State → [Q(s,a₁), Q(s,a₂), ..., Q(s,aₙ)]│
  │       Select argmax                               │
  │                                                    │
  │ DDPG: State → Actor → action a                   │
  │       State, a → Critic → Q(s,a)                  │
  │       Train actor to maximize Q                    │
  └──────────────────────────────────────────────────┘
```

---

## Architecture

```
  ┌──────────────────────────────────────────────────┐
  │ DDPG has 4 networks:                              │
  │                                                    │
  │ Actor:         μ_θ(s)   → deterministic action    │
  │ Critic:        Q_φ(s,a) → Q-value                │
  │ Target Actor:  μ_θ'(s)  → slowly updated copy    │
  │ Target Critic: Q_φ'(s,a)→ slowly updated copy    │
  │                                                    │
  │           s_t                                      │
  │            │                                       │
  │     ┌──────┴──────┐                                │
  │     │   Actor     │                                │
  │     │  μ_θ(s)     │                                │
  │     └──────┬──────┘                                │
  │            │ a_t = μ(s_t) + noise                  │
  │            │                                       │
  │     ┌──────┴──────┐                                │
  │     │   Critic    │                                │
  │     │ Q_φ(s, a)   │                                │
  │     └──────┬──────┘                                │
  │            │                                       │
  │       Q-value                                      │
  └──────────────────────────────────────────────────┘
```

---

## DDPG Algorithm

```
Initialize: Actor μ_θ, Critic Q_φ, Target μ_θ', Q_φ', Replay Buffer D

For each step:
  1. EXPLORE: a = μ_θ(s) + ε,  ε ~ N(0, σ)     (add noise)
  2. Execute a, observe r, s'
  3. Store (s, a, r, s', done) in D
  
  4. SAMPLE minibatch from D
  
  5. CRITIC UPDATE:
     y = r + γ(1-d) × Q_φ'(s', μ_θ'(s'))      (target value)
     L_critic = E[(Q_φ(s,a) - y)²]              (MSE loss)
     φ ← φ - α_Q ∇_φ L_critic
  
  6. ACTOR UPDATE:
     L_actor = -E[Q_φ(s, μ_θ(s))]              (maximize Q!)
     θ ← θ - α_μ ∇_θ L_actor
     
     Key: gradient flows through Q back to μ
     ∇_θ J = E[∇_a Q(s,a)|_{a=μ(s)} × ∇_θ μ(s)]  (chain rule)
  
  7. SOFT UPDATE targets:
     θ' ← τθ + (1-τ)θ'     (τ = 0.005)
     φ' ← τφ + (1-τ)φ'

The Deterministic Policy Gradient theorem:
  ∇_θ J = E_s[∇_a Q(s,a)|_{a=μ(s)} × ∇_θ μ_θ(s)]
  
  "How should we change μ to increase Q?"
  1. ∇_a Q: How does Q change with action?
  2. ∇_θ μ: How does action change with parameters?
  3. Chain them: How should parameters change to increase Q?
```

---

## Exploration in DDPG

```
Deterministic policy → no inherent exploration!

Must add noise:

  1. Action noise (ε-greedy continuous):
     a = μ(s) + ε,  ε ~ N(0, σ)
     σ decayed over training

  2. Ornstein-Uhlenbeck (OU) noise (original DDPG):
     dx = θ(μ_noise - x)dt + σdW
     Temporally correlated → smooth exploration
     Good for physical systems with momentum

  3. Parameter noise (alternative):
     θ̃ = θ + ε_params
     Perturb network weights → state-dependent exploration
     More consistent exploration within episodes

  In practice: Simple Gaussian noise works well
  OU noise is legacy; not needed for most tasks
```

---

## Python Implementation

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
import numpy as np
from collections import deque
import random

class Actor(nn.Module):
    def __init__(self, n_obs, n_actions, max_action=1.0):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(n_obs, 256), nn.ReLU(),
            nn.Linear(256, 256), nn.ReLU(),
            nn.Linear(256, n_actions), nn.Tanh())
        self.max_action = max_action
    
    def forward(self, state):
        return self.max_action * self.net(state)

class Critic(nn.Module):
    def __init__(self, n_obs, n_actions):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(n_obs + n_actions, 256), nn.ReLU(),
            nn.Linear(256, 256), nn.ReLU(),
            nn.Linear(256, 1))
    
    def forward(self, state, action):
        return self.net(torch.cat([state, action], dim=-1))

class ReplayBuffer:
    def __init__(self, capacity=1_000_000):
        self.buffer = deque(maxlen=capacity)
    
    def push(self, s, a, r, s_next, done):
        self.buffer.append((s, a, r, s_next, done))
    
    def sample(self, batch_size):
        batch = random.sample(self.buffer, batch_size)
        s, a, r, s2, d = zip(*batch)
        return (torch.FloatTensor(np.array(s)),
                torch.FloatTensor(np.array(a)),
                torch.FloatTensor(r).unsqueeze(1),
                torch.FloatTensor(np.array(s2)),
                torch.FloatTensor(d).unsqueeze(1))

class DDPG:
    def __init__(self, n_obs, n_actions, max_action=1.0):
        self.actor = Actor(n_obs, n_actions, max_action)
        self.critic = Critic(n_obs, n_actions)
        self.target_actor = Actor(n_obs, n_actions, max_action)
        self.target_critic = Critic(n_obs, n_actions)
        
        # Copy initial weights
        self.target_actor.load_state_dict(self.actor.state_dict())
        self.target_critic.load_state_dict(self.critic.state_dict())
        
        self.actor_opt = torch.optim.Adam(self.actor.parameters(), lr=1e-4)
        self.critic_opt = torch.optim.Adam(self.critic.parameters(), lr=1e-3)
        
        self.gamma = 0.99
        self.tau = 0.005
        self.max_action = max_action
    
    def select_action(self, state, noise=0.1):
        s = torch.FloatTensor(state).unsqueeze(0)
        action = self.actor(s).detach().numpy()[0]
        action += noise * np.random.randn(*action.shape)
        return np.clip(action, -self.max_action, self.max_action)
    
    def update(self, buffer, batch_size=256):
        s, a, r, s_next, done = buffer.sample(batch_size)
        
        # Critic update
        with torch.no_grad():
            target_a = self.target_actor(s_next)
            target_q = r + self.gamma * (1 - done) * self.target_critic(s_next, target_a)
        
        current_q = self.critic(s, a)
        critic_loss = F.mse_loss(current_q, target_q)
        
        self.critic_opt.zero_grad()
        critic_loss.backward()
        self.critic_opt.step()
        
        # Actor update: maximize Q
        actor_loss = -self.critic(s, self.actor(s)).mean()
        
        self.actor_opt.zero_grad()
        actor_loss.backward()
        self.actor_opt.step()
        
        # Soft update targets
        for tp, p in zip(self.target_actor.parameters(), self.actor.parameters()):
            tp.data.lerp_(p.data, self.tau)
        for tp, p in zip(self.target_critic.parameters(), self.critic.parameters()):
            tp.data.lerp_(p.data, self.tau)
```

---

## DDPG Limitations

```
1. Overestimation bias:
   Single Q-network tends to overestimate Q values
   → Fixed by TD3 (twin critics)

2. Brittle to hyperparameters:
   Sensitive to learning rates, noise scale, network size
   → Fixed by TD3 (delayed updates)

3. Exploration:
   Deterministic policy + Gaussian noise is crude
   → Fixed by SAC (maximum entropy)

4. Function approximation errors:
   Errors in Q accumulate through training
   → Fixed by TD3 (target policy smoothing)

  DDPG → TD3 → SAC (evolution of algorithms)
```

---

## Revision Questions

1. **Why can't DQN handle continuous action spaces?**
2. **What is the deterministic policy gradient theorem?**
3. **How does the actor update work (what gradient flows through)?**
4. **Why is exploration noise necessary in DDPG?**
5. **What are the main limitations of DDPG that TD3 addresses?**

---

[Previous: 03-sac.md](03-sac.md) | [Next: 05-td3.md](05-td3.md)
