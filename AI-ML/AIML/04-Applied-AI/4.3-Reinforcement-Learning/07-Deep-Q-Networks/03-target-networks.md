# Target Networks

## Overview

**Target networks** use a separate, slowly-updated copy of the Q-network to compute TD targets. Without this, the target shifts with every gradient step (since the same network computes both the prediction and the target), causing oscillations and divergence. Target networks stabilize training by holding targets fixed for many steps.

---

## The Instability Problem

```
Q-learning update:
  Q(s,a) ← Q(s,a) + α[r + γ max_a' Q(s',a') - Q(s,a)]
                              ↑ target        ↑ prediction

With neural networks, Q(s,a;θ) for ALL states changes with each θ update:

  Step 1: θ updated → Q(s',a';θ) changes → target shifts
  Step 2: Train toward new target → θ changes again → target shifts again
  
  ┌──────────────────────────────────────────┐
  │ Chasing a moving target:                  │
  │                                           │
  │  Prediction ──→ Target                    │
  │                  ↓ (target moves!)         │
  │  Prediction ──→ Target                    │
  │                  ↓ (target moves again!)   │
  │  Prediction ──→ Target                    │
  │                  ↓ ...                     │
  │                                           │
  │  → Oscillations, divergence, instability  │
  └──────────────────────────────────────────┘
```

---

## Solution: Target Network

```
Maintain TWO networks:

  Policy network Q(s, a; θ)    — updated every step
  Target network Q(s, a; θ⁻)   — updated every C steps

  ┌─────────────────┐        ┌─────────────────┐
  │  Policy Network │        │  Target Network │
  │  Q(s, a; θ)     │        │  Q(s, a; θ⁻)   │
  │                 │        │                 │
  │  Updated every  │        │  Frozen for C   │
  │  gradient step  │        │  steps          │
  └────────┬────────┘        └────────┬────────┘
           │                          │
     Predictions              Targets (stable!)
     Q(s, a; θ)               r + γ max Q(s', a'; θ⁻)

  Every C steps: θ⁻ ← θ  (hard copy)

  Effect:
    Target is FIXED for C steps
    → Stable regression target
    → No more chasing moving target
    → Convergent training!
```

---

## Hard vs Soft Updates

```
Hard update (original DQN):
  Every C steps: θ⁻ ← θ  (complete copy)
  
  θ⁻ ─────────────────────────────── → θ⁻ (unchanged)
                                       ↓ (copy at step C)
  θ  ── update ── update ── update ── → θ
  
  C = 10,000 steps in original DQN
  Discontinuous jumps in target values

Soft update (Polyak averaging, DDPG/TD3/SAC):
  Every step: θ⁻ ← τθ + (1-τ)θ⁻
  
  τ = 0.005 (small)
  
  θ⁻ smoothly tracks θ:
  θ⁻ ──── slowly ──── converges ──── toward θ
  
  No discontinuous jumps → smoother training
  More stable but slower to adapt

  Hard update: simple, works well with large C
  Soft update: smoother, preferred in modern algorithms
```

---

## Why It Works

```
Without target network:
  Loss = (r + γ max Q(s',a';θ) - Q(s,a;θ))²
  
  Gradient of loss w.r.t. θ:
    Both the target and prediction depend on θ
    → Gradient is biased (semi-gradient issue)
    → With neural nets: can amplify errors → divergence

With target network:
  Loss = (r + γ max Q(s',a';θ⁻) - Q(s,a;θ))²
  
  θ⁻ is treated as constant:
    Target is a fixed number for C steps
    → Standard supervised regression
    → Gradient is well-behaved
    → Training is stable

  Analogy:
    Without: trying to hit a moving target while running
    With:    target stands still for C shots, then moves
```

---

## Python Implementation

```python
import torch
import copy

class DQNAgent:
    def __init__(self, q_network, lr=1e-4, gamma=0.99,
                 target_update_freq=1000, soft_update_tau=None):
        self.policy_net = q_network
        self.target_net = copy.deepcopy(q_network)
        self.target_net.eval()  # no gradient tracking
        
        self.optimizer = torch.optim.Adam(
            self.policy_net.parameters(), lr=lr)
        self.gamma = gamma
        self.target_update_freq = target_update_freq
        self.tau = soft_update_tau
        self.steps = 0
    
    def compute_loss(self, states, actions, rewards, 
                     next_states, dones):
        q_values = self.policy_net(states).gather(
            1, actions.unsqueeze(1)).squeeze(1)
        
        with torch.no_grad():
            next_q = self.target_net(next_states).max(1)[0]
            targets = rewards + self.gamma * next_q * (1 - dones)
        
        return torch.nn.functional.mse_loss(q_values, targets)
    
    def update(self, batch):
        loss = self.compute_loss(*batch)
        self.optimizer.zero_grad()
        loss.backward()
        self.optimizer.step()
        
        self.steps += 1
        self._update_target()
        return loss.item()
    
    def _update_target(self):
        if self.tau:  # Soft update
            for tp, pp in zip(self.target_net.parameters(),
                              self.policy_net.parameters()):
                tp.data.copy_(self.tau * pp.data + 
                              (1 - self.tau) * tp.data)
        elif self.steps % self.target_update_freq == 0:  # Hard update
            self.target_net.load_state_dict(
                self.policy_net.state_dict())
```

---

## Revision Questions

1. **Why does training diverge without a target network?**
2. **How does freezing the target network stabilize training?**
3. **What is the difference between hard and soft target updates?**
4. **What is a typical value for C (hard update) and τ (soft update)?**
5. **Why is the target network set to eval mode (no gradient)?**

---

[Previous: 02-experience-replay.md](02-experience-replay.md) | [Next: 04-double-dqn.md](04-double-dqn.md)
