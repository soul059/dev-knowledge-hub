# Exploration Strategies

## Overview

**Exploration** is one of the fundamental challenges in RL: how should an agent try new actions to discover better strategies? Too little exploration leads to suboptimal policies (getting stuck in local optima), while too much wastes time on clearly bad actions. This chapter covers strategies from simple ε-greedy to sophisticated curiosity-driven and count-based approaches.

---

## The Exploration-Exploitation Dilemma

```
Exploitation: Use what you know → maximize short-term reward
Exploration:  Try new things → discover better strategies

  ┌──────────────────────────────────────────────────┐
  │  Restaurant analogy:                              │
  │                                                    │
  │  Exploit: Go to your favorite restaurant          │
  │    → Guaranteed good meal                         │
  │    → But might miss an even better place!         │
  │                                                    │
  │  Explore: Try a new restaurant                    │
  │    → Might discover something amazing             │
  │    → Might waste money on bad food                │
  │                                                    │
  │  Challenge: How to balance? When to explore?      │
  └──────────────────────────────────────────────────┘

  Types of exploration problems:
  1. Dense rewards: Easy, simple exploration works
  2. Sparse rewards: Hard, need directed exploration
  3. Deceptive rewards: Very hard, local optima everywhere
```

---

## Basic Strategies

```
1. ε-GREEDY:
   With prob ε: random action (explore)
   With prob 1-ε: greedy action (exploit)
   
   ε decays over time: ε₀=1.0 → ε_final=0.01
   
   Pros: Simple, works for many problems
   Cons: Explores UNIFORMLY (wastes time on known-bad actions)
         Doesn't direct exploration toward uncertainty

2. BOLTZMANN (SOFTMAX):
   P(a) = exp(Q(s,a)/τ) / Σ exp(Q(s,a')/τ)
   
   τ = temperature
   τ → ∞: uniform random (explore)
   τ → 0: greedy (exploit)
   
   Better than ε-greedy: favors actions with higher Q
   Still doesn't consider uncertainty

3. NOISE INJECTION:
   Add noise to actions (continuous): a = μ(s) + ε
   Add noise to parameters: θ̃ = θ + ε
   
   NoisyNet: learnable noise parameters
   σ = f(features) → state-dependent exploration!
```

---

## Optimism in the Face of Uncertainty (OFU)

```
Principle: Act as if the best possible outcome will happen

  UCB (Upper Confidence Bound):
    a* = argmax_a [Q(s,a) + c × √(ln(t) / N(s,a))]
    
    Q(s,a):              exploitation term (estimated value)
    c × √(ln(t)/N(s,a)): exploration bonus (uncertainty)
    
    N(s,a) = visit count
    
    Rarely visited → large bonus → explore!
    Frequently visited → small bonus → exploit!

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Action A: Q=5, visited 100 times                │
  │    UCB = 5 + 0.3 = 5.3                           │
  │                                                    │
  │  Action B: Q=4, visited 2 times                  │
  │    UCB = 4 + 2.1 = 6.1  ← Explore this one!     │
  │                                                    │
  │  B has lower Q but high uncertainty               │
  │  → Worth trying to see if it's actually better   │
  └──────────────────────────────────────────────────┘
```

---

## Count-Based Exploration

```
Track how many times each state (or state-action) is visited:
  N(s) = number of visits to state s

Exploration bonus: r_bonus = β / √N(s)
  Rarely seen states → high bonus → explore!

Problem: In large/continuous spaces, N(s)=0 for almost all states!

Solutions:

1. Hash-based counts:
   φ(s) → hash code → count hash collisions
   SimHash, locality-sensitive hashing

2. Density models (pseudo-counts):
   Train density model ρ(s) on visited states
   Pseudo-count: N̂(s) = ρ(s) × n / (1 - ρ(s))
   
   Low density → low pseudo-count → explore!

3. RND (Random Network Distillation):
   Fixed random network: f(s)
   Trained network: f̂(s; θ) trying to match f
   
   Prediction error: ||f̂(s) - f(s)||²
   
   Familiar states → low error (well-trained) → low bonus
   Novel states → high error (never seen) → high bonus!
   
   ┌──────────────────────────────────────────┐
   │ RND: Novelty ≈ prediction error         │
   │                                          │
   │ Visited: f̂(s) ≈ f(s) → error ≈ 0      │
   │ Novel:   f̂(s) ≠ f(s) → error ≫ 0      │
   │                                          │
   │ Bonus = ||f̂(s) - f(s)||²               │
   └──────────────────────────────────────────┘
```

---

## Curiosity-Driven Exploration

```
Intrinsic Curiosity Module (ICM):

  The agent is "curious" about states it can't predict well
  
  Three components:
    Feature encoder: φ(s) → compact state features
    Forward model:   φ̂(s_{t+1}) = f(φ(s_t), a_t)  (predict next)
    Inverse model:   â_t = g(φ(s_t), φ(s_{t+1}))  (predict action)

  Intrinsic reward:
    r_intrinsic = ||φ̂(s_{t+1}) - φ(s_{t+1})||²
    
    "Surprise" = how wrong the forward prediction was
    High surprise → novel transition → explore!

  Why the inverse model?
    Trains φ to encode only action-relevant features
    Ignores "noisy TV" problem:
      Without: agent stares at TV static (always surprising!)
      With:    φ ignores TV (not controllable) → no curiosity

  ┌──────────────────────────────────────────────────┐
  │  s_t, a_t ─→ Forward Model ─→ φ̂(s_{t+1})      │
  │                                  │                │
  │  s_{t+1}  ─→ Encoder ───→ φ(s_{t+1})            │
  │                                  │                │
  │  Curiosity = ||φ̂(s_{t+1}) - φ(s_{t+1})||²      │
  │                                                    │
  │  High curiosity → explore this transition!        │
  └──────────────────────────────────────────────────┘
```

---

## Go-Explore

```
For hard exploration problems (Montezuma's Revenge):

  Standard RL: never finds first reward (after 1000+ steps)
  
  Go-Explore (2 phases):
  
  Phase 1: EXPLORE (no learning, just find states)
    Maintain archive of interesting states
    Repeatedly:
      1. Go to a state in archive (deterministically)
      2. Explore randomly from there
      3. Add new interesting states to archive
    
    Key: "Go" back to promising states before exploring
    Avoids: losing progress by starting from scratch

  Phase 2: ROBUSTIFY
    Have trajectories that reach goal
    Train policy via imitation + RL to be robust
    Handle stochasticity (demos may not replay exactly)

  ┌────────────────────────────────────────┐
  │ Standard:  Start → explore → get lost │
  │ Go-Explore: Start → archive state     │
  │             → go back → explore more  │
  │             → archive new states      │
  │             → go back → explore more  │
  │             → eventually reach goal!  │
  └────────────────────────────────────────┘
```

---

## Python: RND Exploration Bonus

```python
import torch
import torch.nn as nn

class RNDExploration:
    """Random Network Distillation for exploration bonuses."""
    
    def __init__(self, obs_dim, feature_dim=64, lr=1e-3):
        # Fixed random network (never trained)
        self.target = nn.Sequential(
            nn.Linear(obs_dim, 128), nn.ReLU(),
            nn.Linear(128, feature_dim))
        for p in self.target.parameters():
            p.requires_grad = False
        
        # Predictor network (trained to match target)
        self.predictor = nn.Sequential(
            nn.Linear(obs_dim, 128), nn.ReLU(),
            nn.Linear(128, 128), nn.ReLU(),
            nn.Linear(128, feature_dim))
        
        self.optimizer = torch.optim.Adam(
            self.predictor.parameters(), lr=lr)
        
        # Running statistics for normalization
        self.reward_mean = 0
        self.reward_var = 1
        self.count = 0
    
    def compute_bonus(self, obs):
        """Exploration bonus = prediction error."""
        with torch.no_grad():
            target_features = self.target(obs)
            predicted_features = self.predictor(obs)
            bonus = (target_features - predicted_features).pow(2).sum(dim=-1)
        return bonus
    
    def update(self, obs):
        """Train predictor on visited observations."""
        target_features = self.target(obs).detach()
        predicted_features = self.predictor(obs)
        loss = (target_features - predicted_features).pow(2).sum(dim=-1).mean()
        
        self.optimizer.zero_grad()
        loss.backward()
        self.optimizer.step()
        
        return loss.item()
    
    def normalize_bonus(self, bonus):
        """Normalize exploration bonus for stable training."""
        self.count += 1
        delta = bonus.mean().item() - self.reward_mean
        self.reward_mean += delta / self.count
        self.reward_var = (bonus.var().item() + self.reward_var) / 2
        
        return bonus / (self.reward_var ** 0.5 + 1e-8)


# Usage in training loop
rnd = RNDExploration(obs_dim=84*84)

for step in range(total_steps):
    obs = get_observation()
    
    # Compute exploration bonus
    bonus = rnd.compute_bonus(torch.FloatTensor(obs))
    bonus = rnd.normalize_bonus(bonus)
    
    # Total reward = extrinsic + intrinsic
    total_reward = extrinsic_reward + 0.1 * bonus
    
    # Update RND predictor
    rnd.update(torch.FloatTensor(obs))
    
    # Use total_reward in PPO/DQN/etc.
```

---

## Comparison of Strategies

```
  Strategy         Type          Best For          Complexity
  ────────         ────          ────────          ──────────
  ε-greedy         Undirected    Simple envs       Low
  Boltzmann        Undirected    Discrete actions  Low
  UCB              Optimistic    Bandits, tabular  Low
  Count-based      Directed      Discrete states   Medium
  RND              Directed      Any env           Medium
  ICM              Curiosity     Sparse reward     High
  Go-Explore       Archive       Very sparse       High
  NoisyNet         Parametric    Deep RL           Low
```

---

## Revision Questions

1. **What is the exploration-exploitation dilemma?**
2. **How does UCB balance exploration and exploitation?**
3. **How does RND measure state novelty without explicit counts?**
4. **What is the "noisy TV" problem and how does ICM address it?**
5. **Why is Go-Explore effective for hard exploration problems?**

---

[Previous: 06-rlhf.md](06-rlhf.md) | [Back to README](../README.md)
