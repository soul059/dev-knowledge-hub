# Hierarchical Reinforcement Learning (HRL)

## Overview

**Hierarchical RL** decomposes complex tasks into a hierarchy of subtasks, enabling temporal abstraction. Instead of making decisions at every timestep, higher-level policies set goals or select skills, while lower-level policies execute them. HRL addresses the curse of long horizons — tasks that require thousands of steps become tractable when broken into manageable subgoals.

---

## Why Hierarchy?

```
Problem: Making a sandwich
  Flat RL: 1000+ individual motor commands
    Step 1: move arm left 2cm
    Step 2: close gripper 5%
    Step 3: ...
    Very long horizon → sparse reward → hard to learn!

  Hierarchical RL:
    High level: [Get bread] → [Get filling] → [Assemble]
    Mid level:  [Get bread] = [Open cabinet] → [Reach] → [Grab]
    Low level:  [Reach] = individual motor commands

  ┌──────────────────────────────────────────────────┐
  │  High-level policy:   "Get bread"                │
  │       │                                           │
  │  Mid-level policy:    "Open cabinet"              │
  │       │                                           │
  │  Low-level policy:    [move, grip, pull, ...]    │
  │                                                   │
  │  Temporal abstraction:                           │
  │    High: decides every ~100 steps                │
  │    Mid:  decides every ~20 steps                 │
  │    Low:  decides every step                      │
  └──────────────────────────────────────────────────┘

  Benefits:
  1. Shorter effective horizon at each level
  2. Subpolicies can transfer across tasks
  3. Exploration is more structured
  4. Interpretable decisions
```

---

## Options Framework

```
An OPTION is a temporally extended action:

  Option ω = (I, π, β)
    I: initiation set  — states where option can start
    π: policy          — how to act while option is active
    β: termination     — probability of stopping at each state

  Semi-MDP with options:
    Agent selects option ω
    Option executes for multiple steps (following π)
    Option terminates (according to β)
    Agent selects next option

  ┌──────────────────────────────────────────┐
  │  Regular MDP:                             │
  │  s₀ →a₁→ s₁ →a₂→ s₂ →a₃→ s₃ →a₄→ s₄   │
  │  Decision at every step                   │
  │                                           │
  │  Options (Semi-MDP):                      │
  │  s₀ →[option 1]→ s₃ →[option 2]→ s₇     │
  │  Decision every few steps                 │
  │  Much shorter effective horizon!           │
  └──────────────────────────────────────────┘

  Option-value function:
    Q(s, ω) = E[r₁ + γr₂ + ... + γ^k r_k + γ^k V(s_k)]
    where k is when option ω terminates
```

---

## Goal-Conditioned RL

```
Policy is conditioned on a goal: π(a|s, g)

  "Given current state and goal, what action to take?"

  High-level policy:  g = π_high(s)        (selects goal)
  Low-level policy:   a = π_low(s, g)      (achieves goal)

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Manager (high-level):                            │
  │    Every k steps: g_t = π_high(s_t)               │
  │    Reward: external environment reward             │
  │                                                    │
  │  Worker (low-level):                              │
  │    Every step: a_t = π_low(s_t, g_t)              │
  │    Reward: -||s_{t+1} - g_t|| (did we reach goal?)│
  │                                                    │
  │  Manager provides subgoals                        │
  │  Worker tries to reach them                       │
  │  Decoupled learning!                              │
  └──────────────────────────────────────────────────┘

  Universal Value Function Approximator (UVFA):
    V(s, g) or Q(s, a, g) — value depends on goal
    One network handles all possible goals
```

---

## Feudal Networks (FeUdal Networks, FuN)

```
Manager-Worker architecture:

  Manager:
    Operates at lower temporal resolution
    Produces direction vector in latent space
    Receives external reward
    
  Worker:
    Operates at every timestep
    Tries to move in direction suggested by manager
    Intrinsic reward: cosine similarity between
      actual movement and manager's direction

  ┌──────────────────────────────────────┐
  │  s_t → Manager (every c steps)       │
  │          ↓ goal direction g_t        │
  │                                      │
  │  s_t, g_t → Worker (every step)     │
  │               ↓ action a_t          │
  │                                      │
  │  Worker reward:                      │
  │    r_intr = cos(s_{t+c} - s_t, g_t) │
  │    (did state change in goal dir?)   │
  └──────────────────────────────────────┘
```

---

## HAM and MAXQ

```
HAM (Hierarchy of Abstract Machines):
  Define a hierarchy of finite-state machines
  Each machine can call sub-machines
  Constrains exploration to valid action sequences
  Programmer specifies partial program structure

MAXQ Value Decomposition:
  Q(s, a) = V_subtask(s) + C(s, subtask, a)
  
  Total value = value of completing subtask
              + value of completing remaining task
  
  Decomposes value hierarchically:
    Q_root = C_root + Q_subtask1
    Q_subtask1 = C_subtask1 + Q_primitive_action
    
  Enables reuse: subtask values transfer across contexts
```

---

## Hindsight Goal Relabeling

```
For goal-conditioned RL, goals may rarely be achieved:

  Original: trajectory (s₀,a₀,...,s_T), goal g → reward 0 (failed)
  
  Hindsight Experience Replay (HER):
    Relabel goal as g' = s_T (where we actually ended up)
    Now trajectory "succeeded" at reaching g'!
    
    Original:  (s₀, g) → reward 0 (didn't reach g)
    Relabeled: (s₀, g'=s_T) → reward 1 (reached s_T!)
    
    Learn from failure: "I didn't go where I wanted,
    but I learned how to go where I went"
    
  Dramatically improves learning in sparse reward settings
```

---

## Python: Goal-Conditioned HRL

```python
import torch
import torch.nn as nn
import numpy as np

class HighLevelPolicy(nn.Module):
    """Manager: selects subgoals every k steps."""
    def __init__(self, state_dim, goal_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(state_dim, 128), nn.ReLU(),
            nn.Linear(128, 128), nn.ReLU(),
            nn.Linear(128, goal_dim), nn.Tanh())
    
    def forward(self, state):
        return self.net(state)

class LowLevelPolicy(nn.Module):
    """Worker: takes actions to reach subgoal."""
    def __init__(self, state_dim, goal_dim, action_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(state_dim + goal_dim, 128), nn.ReLU(),
            nn.Linear(128, 128), nn.ReLU(),
            nn.Linear(128, action_dim), nn.Tanh())
    
    def forward(self, state, goal):
        return self.net(torch.cat([state, goal], dim=-1))

class HierarchicalAgent:
    def __init__(self, state_dim, goal_dim, action_dim, k=10):
        self.manager = HighLevelPolicy(state_dim, goal_dim)
        self.worker = LowLevelPolicy(state_dim, goal_dim, action_dim)
        self.k = k  # manager decision frequency
        self.step_count = 0
        self.current_goal = None
    
    def act(self, state):
        s = torch.FloatTensor(state).unsqueeze(0)
        
        # Manager sets new goal every k steps
        if self.step_count % self.k == 0:
            with torch.no_grad():
                self.current_goal = self.manager(s)
        
        # Worker selects action to reach goal
        with torch.no_grad():
            action = self.worker(s, self.current_goal)
        
        self.step_count += 1
        return action.squeeze(0).numpy()
    
    def compute_intrinsic_reward(self, state, next_state, goal):
        """Worker reward: progress toward goal."""
        direction = next_state - state
        goal_direction = goal - state
        
        # Cosine similarity
        cos_sim = (torch.nn.functional.cosine_similarity(
            direction.unsqueeze(0), goal_direction.unsqueeze(0)))
        return cos_sim.item()
```

---

## Comparison of HRL Methods

```
  Method       Hierarchy Type    Key Idea             Year
  ──────       ──────────────    ────────             ────
  Options      Temporal          Extended actions     1999
  MAXQ         Value decomp.    Subtask values       2000
  FuN          Goal-cond.       Manager-worker       2017
  HIRO         Goal-cond.       Off-policy HRL       2018
  HAC          Goal-cond.       Hindsight at all     2019
                                levels
```

---

## Revision Questions

1. **Why is temporal abstraction important for RL in complex tasks?**
2. **What are the three components of an option?**
3. **How does goal-conditioned RL decompose the learning problem?**
4. **What is Hindsight Experience Replay and why is it effective?**
5. **What is the manager-worker paradigm in hierarchical RL?**

---

[Previous: 02-multi-agent-rl.md](02-multi-agent-rl.md) | [Next: 04-inverse-rl.md](04-inverse-rl.md)
