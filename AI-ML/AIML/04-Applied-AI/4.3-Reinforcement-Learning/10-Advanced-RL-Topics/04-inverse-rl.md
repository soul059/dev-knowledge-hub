# Inverse Reinforcement Learning (IRL)

## Overview

**Inverse RL** recovers the reward function from expert demonstrations instead of being given one. The key insight: it's often easier to demonstrate good behavior than to specify the reward function. IRL solves the problem "given that this expert is acting optimally, what reward function are they optimizing?" This is fundamental to imitation learning, value alignment, and understanding agent behavior.

---

## The Inverse Problem

```
Forward RL:  Given reward R → find optimal policy π*
Inverse RL:  Given optimal policy π* (demos) → find reward R

  ┌───────────────────────────────────────────┐
  │  Forward RL:                               │
  │    MDP + Reward → Policy                  │
  │    R(s,a) → π*(a|s)                       │
  │                                            │
  │  Inverse RL:                               │
  │    MDP + Expert Demos → Reward            │
  │    {τ₁, τ₂, ...} → R̂(s,a)               │
  │                                            │
  │  Then: R̂ → π* (train new policy using R̂) │
  └───────────────────────────────────────────┘

Why not just imitate (behavioral cloning)?
  BC: π(a|s) = copy expert
  Problem: distributional shift!
    ┌──────────────────────────────────────┐
    │ Expert: s₁ → a₁ → s₂ → a₂ → s₃     │
    │ Clone:  s₁ → a₁' → s₂' → ??? → 💥  │
    │                                       │
    │ Small errors compound!               │
    │ Clone visits states expert never saw │
    │ No training data for those states    │
    └──────────────────────────────────────┘

  IRL advantage: learns WHY expert acts that way
  → Recovered reward generalizes to new situations
```

---

## Feature-Based IRL

```
Assume reward is linear in features:
  R(s) = w · φ(s) = Σ wᵢ φᵢ(s)

  φ(s) = feature vector (hand-designed)
  w = weights to learn

  Expert demonstrations: {τ₁, τ₂, ..., τₙ}
  Feature expectations: μ_E = (1/N) Σ Σ_t γ^t φ(s_t)

  Goal: find w such that expert is optimal:
    μ_E = argmax_π E_π[Σ γ^t w · φ(s_t)]

  Ambiguity problem:
    w = 0 makes EVERY policy optimal!
    Many reward functions explain the same behavior
    Need additional constraints (max margin, max entropy)
```

---

## Maximum Entropy IRL

```
Key idea: among all reward functions consistent with demos,
choose the one that makes expert behavior MOST LIKELY
while keeping everything else as random as possible.

  P(τ) ∝ exp(Σ_t R(s_t, a_t))
  
  Expert trajectories should be exponentially more likely
  than non-expert trajectories, proportional to total reward.

  Optimization:
    max_R  E_expert[Σ R(s,a)] - log Z
    
    Z = Σ_τ exp(Σ_t R(s_t, a_t))   (partition function)

  This gives:
  - High reward to states/actions expert visits
  - Low reward to states/actions expert avoids
  - Maximum uncertainty (entropy) everywhere else

  ┌────────────────────────────────────────────────┐
  │ MaxEnt IRL resolves ambiguity:                  │
  │                                                  │
  │ Many rewards explain expert ────────────────┐   │
  │   R₁: slightly different     ←  choose the │   │
  │   R₂: very different            simplest one│   │
  │   R₃: matches exactly       ← (max entropy)│   │
  │   ...                       ────────────────┘   │
  └────────────────────────────────────────────────┘
```

---

## Generative Adversarial Imitation Learning (GAIL)

```
GAIL frames imitation as a GAN:

  Discriminator D(s,a): classifies expert vs. agent
    D(s,a) → 1 if expert, 0 if agent
    
  Generator (Policy) π: tries to fool discriminator
    Generates trajectories that look like expert's

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Expert demos: {(s,a)_expert}                     │
  │       ↓                                            │
  │  Discriminator D(s,a)  ←──── Is it expert?        │
  │       ↑                                            │
  │  Agent policy π(a|s)   ←──── Generate behavior    │
  │       ↓                                            │
  │  Reward = -log(1-D(s,a))   ← reward from D       │
  │       ↓                                            │
  │  Update π with PPO/TRPO using this reward          │
  │                                                    │
  └──────────────────────────────────────────────────┘

  Training loop:
  1. Collect agent trajectories with π
  2. Train D to distinguish expert from agent
  3. Use -log(1-D(s,a)) as reward for π
  4. Update π using any RL algorithm (PPO)
  5. Repeat

  GAIL doesn't explicitly recover R — reward is implicit in D
  But agent learns to behave like expert!
```

---

## Algorithms Summary

```
  Method                Key Idea                    Year
  ──────                ────────                    ────
  Linear IRL            w · φ(s), feature matching   2000
  Max Margin IRL        Maximize margin to expert    2004
  MaxEnt IRL            Maximum entropy principle     2008
  Deep MaxEnt IRL       Neural network reward         2015
  GAIL                  GAN-based, no explicit R      2016
  AIRL                  Recovers transferable R       2018
  DAgger                Interactive imitation          2011
  
  Linear → MaxEnt → Deep → GAN-based (evolution)
```

---

## Python: GAIL Sketch

```python
import torch
import torch.nn as nn

class Discriminator(nn.Module):
    """Distinguishes expert from agent state-action pairs."""
    def __init__(self, state_dim, action_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(state_dim + action_dim, 256), nn.ReLU(),
            nn.Linear(256, 256), nn.ReLU(),
            nn.Linear(256, 1), nn.Sigmoid())
    
    def forward(self, state, action):
        return self.net(torch.cat([state, action], dim=-1))

class GAILTrainer:
    def __init__(self, policy, discriminator, expert_data):
        self.policy = policy
        self.disc = discriminator
        self.expert_data = expert_data
        self.disc_opt = torch.optim.Adam(self.disc.parameters(), lr=3e-4)
    
    def update_discriminator(self, agent_states, agent_actions):
        # Expert data
        idx = torch.randint(0, len(self.expert_data[0]),
                           (len(agent_states),))
        expert_s = self.expert_data[0][idx]
        expert_a = self.expert_data[1][idx]
        
        # Discriminator loss (binary cross-entropy)
        expert_pred = self.disc(expert_s, expert_a)
        agent_pred = self.disc(agent_states, agent_actions)
        
        loss = (-torch.log(expert_pred + 1e-8).mean()
                - torch.log(1 - agent_pred + 1e-8).mean())
        
        self.disc_opt.zero_grad()
        loss.backward()
        self.disc_opt.step()
    
    def get_reward(self, states, actions):
        """GAIL reward: -log(1 - D(s,a))"""
        with torch.no_grad():
            d = self.disc(states, actions)
        return -torch.log(1 - d + 1e-8)
```

---

## Applications

```
1. Autonomous Driving:
   Expert: human driving demonstrations
   IRL → recover driving preferences (speed, safety, comfort)
   → Train self-driving policy using recovered reward

2. Robot Manipulation:
   Expert: human demonstrates task (kinesthetic teaching)
   IRL → understand what makes a good grasp/placement
   → Robot generalizes to new objects

3. Game AI:
   Expert: professional player replays
   IRL → understand play style and strategy
   → Generate diverse, human-like AI opponents

4. AI Safety / Alignment:
   Expert: human preferences
   IRL → understand human values
   → Align AI behavior with human intentions
```

---

## Revision Questions

1. **What is the fundamental difference between forward RL and inverse RL?**
2. **Why is IRL preferred over behavioral cloning for complex tasks?**
3. **What is the ambiguity problem in IRL and how does MaxEnt IRL resolve it?**
4. **How does GAIL use a GAN framework for imitation learning?**
5. **Why is IRL important for AI safety and value alignment?**

---

[Previous: 03-hierarchical-rl.md](03-hierarchical-rl.md) | [Next: 05-offline-rl.md](05-offline-rl.md)
