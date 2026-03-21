# Dueling DQN

## Overview

**Dueling DQN** (Wang et al., 2016) modifies the network architecture to separately estimate the **state value V(s)** and the **advantage A(s,a)** of each action. By decomposing Q(s,a) = V(s) + A(s,a), the network can learn which states are valuable independent of actions — greatly improving learning efficiency in states where the action choice doesn't matter.

---

## Motivation

```
In many states, the action choice doesn't matter:

  ┌─────────────────────────────────┐
  │  🚗         (empty road)       │   All actions roughly equal
  │  ─────────────────────────────  │   → V(s) matters, A(s,a) ≈ 0
  └─────────────────────────────────┘

  ┌─────────────────────────────────┐
  │  🚗  🚧   (obstacle ahead!)    │   Action choice is critical
  │  ─────────────────────────────  │   → A(s, steer_left) >> A(s, go_straight)
  └─────────────────────────────────┘

Standard DQN must update Q(s,a) for EACH action separately
Even when only V(s) changes → wasteful

Dueling DQN can update V(s) once → all Q(s,a) improve
→ Faster, more efficient learning
```

---

## Architecture

```
Standard DQN:
  ┌──────────┐     ┌────────┐     ┌──────────────┐
  │  State   │ →  │ Shared │ →  │ FC → Q(s,a)  │
  │  (input) │     │  CNN   │     │ for all a     │
  └──────────┘     └────────┘     └──────────────┘

Dueling DQN:
  ┌──────────┐     ┌────────┐     ┌─────────────────────┐
  │  State   │ →  │ Shared │ →  │ Value stream:        │
  │  (input) │     │  CNN   │     │   FC → V(s)  [1]    │
  └──────────┘     └────┬───┘     ├─────────────────────┤
                        │         │ Advantage stream:    │
                        └────────▶│   FC → A(s,a) [|A|]  │
                                  └──────────┬──────────┘
                                             │
                                    Q(s,a) = V(s) + A(s,a) - mean(A)

  Detailed architecture:
  
  Input (84×84×4)
       │
  ┌────▼──────────────────────────┐
  │  Shared Convolutional Layers  │
  │  (same as DQN: 3 conv layers) │
  └────┬──────────────────────────┘
       │
  ┌────┴────┐
  │         │
  ┌▼──────┐ ┌▼───────────┐
  │FC 512 │ │ FC 512     │
  │  ↓    │ │   ↓        │
  │FC → 1 │ │ FC → |A|   │
  │ V(s)  │ │ A(s,a)     │
  └───┬───┘ └────┬───────┘
      │          │
      └────┬─────┘
           │
    Q(s,a) = V(s) + (A(s,a) - mean_a'(A(s,a')))
```

---

## Identifiability Problem

```
Problem: Q(s,a) = V(s) + A(s,a) is not unique!
  
  V=10, A=[2, -1, -1]  → Q = [12, 9, 9]
  V=11, A=[1, -2, -2]  → Q = [12, 9, 9]   Same Q!
  
  Can't recover V and A uniquely from Q alone.

Solution 1: Subtract max advantage
  Q(s,a) = V(s) + (A(s,a) - max_{a'} A(s,a'))
  
  Forces A(s, a*) = 0 for best action
  → V(s) = Q(s, a*) exactly
  
Solution 2: Subtract mean advantage (preferred)
  Q(s,a) = V(s) + (A(s,a) - (1/|A|) Σ_{a'} A(s,a'))
  
  Forces advantages to sum to 0
  → V(s) = mean Q(s, ·)
  → More stable in practice (softer constraint)
```

---

## Python Implementation

```python
import torch
import torch.nn as nn

class DuelingDQN(nn.Module):
    def __init__(self, n_observations, n_actions):
        super().__init__()
        
        # Shared feature extractor
        self.features = nn.Sequential(
            nn.Linear(n_observations, 128),
            nn.ReLU(),
            nn.Linear(128, 128),
            nn.ReLU()
        )
        
        # Value stream: state → scalar value
        self.value_stream = nn.Sequential(
            nn.Linear(128, 64),
            nn.ReLU(),
            nn.Linear(64, 1)
        )
        
        # Advantage stream: state → advantage per action
        self.advantage_stream = nn.Sequential(
            nn.Linear(128, 64),
            nn.ReLU(),
            nn.Linear(64, n_actions)
        )
    
    def forward(self, x):
        features = self.features(x)
        value = self.value_stream(features)          # [B, 1]
        advantage = self.advantage_stream(features)  # [B, |A|]
        
        # Combine: Q = V + (A - mean(A))
        q_values = value + (advantage - advantage.mean(dim=1, keepdim=True))
        return q_values

# For Atari (CNN version)
class DuelingDQN_CNN(nn.Module):
    def __init__(self, n_actions):
        super().__init__()
        self.conv = nn.Sequential(
            nn.Conv2d(4, 32, 8, stride=4), nn.ReLU(),
            nn.Conv2d(32, 64, 4, stride=2), nn.ReLU(),
            nn.Conv2d(64, 64, 3, stride=1), nn.ReLU(),
            nn.Flatten()
        )
        self.value = nn.Sequential(
            nn.Linear(3136, 512), nn.ReLU(), nn.Linear(512, 1))
        self.advantage = nn.Sequential(
            nn.Linear(3136, 512), nn.ReLU(), nn.Linear(512, n_actions))
    
    def forward(self, x):
        features = self.conv(x / 255.0)
        v = self.value(features)
        a = self.advantage(features)
        return v + (a - a.mean(dim=1, keepdim=True))
```

---

## When Dueling Helps Most

```
Dueling DQN shines when:
  - Many actions have similar Q-values
  - State value is important regardless of action
  - Action space is large (more advantage values to learn)

  Game type           Benefit
  ─────────────────── ────────────
  Enduro (racing)     High (often just drive, rarely steer)
  Pong                Low (always need to move paddle)
  Large action spaces Very high (learn V once, A per action)
  
  Average across 57 Atari games:
    Dueling DQN outperforms DQN on 75% of games
    Combined with Double DQN + PER → even better
```

---

## Revision Questions

1. **What is the decomposition Q(s,a) = V(s) + A(s,a)?**
2. **Why does separating V and A improve learning efficiency?**
3. **What is the identifiability problem and how is it solved?**
4. **Why is mean subtraction preferred over max subtraction?**
5. **In what types of environments does Dueling DQN help the most?**

---

[Previous: 04-double-dqn.md](04-double-dqn.md) | [Next: 06-prioritized-experience-replay.md](06-prioritized-experience-replay.md)
