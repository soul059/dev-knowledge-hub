# Offline Reinforcement Learning

## Overview

**Offline RL** (also called batch RL) learns policies entirely from a fixed dataset of previously collected interactions, with no further environment interaction. This is critical for domains where online exploration is dangerous, expensive, or impossible — healthcare, autonomous driving, robotics, and recommendation systems. The key challenge is **distributional shift**: the learned policy may take actions the dataset never covered.

---

## Online vs Offline RL

```
Online RL:
  Agent → Environment → Data → Update → Agent → Env → ...
  Can explore freely, collect new data
  But: needs simulator or real-time interaction

Offline RL:
  Fixed Dataset D = {(s, a, r, s')₁, ..., (s, a, r, s')_N}
  No new interaction allowed!
  Learn best policy from whatever data exists

  ┌──────────────────────────────────────────────────┐
  │  Online RL:                                       │
  │    ┌──────┐  ←─ act ──  ┌───────┐                │
  │    │  Env │  ── obs ──→ │ Agent │                │
  │    └──────┘              └───────┘                │
  │    Iterative interaction                          │
  │                                                    │
  │  Offline RL:                                      │
  │    ┌─────────┐           ┌───────┐                │
  │    │ Dataset │  ──────→  │ Agent │                │
  │    │  (fixed)│           └───────┘                │
  │    └─────────┘                                    │
  │    No interaction, just learn from data            │
  └──────────────────────────────────────────────────┘

  Why offline RL matters:
  • Healthcare: can't experiment on patients
  • Driving: can't crash to learn
  • Finance: historical trading data only
  • Robotics: real robots are slow/expensive
```

---

## The Distributional Shift Problem

```
The core challenge: policy visits states/actions NOT in dataset

  Dataset collected by behavior policy β:
    D = {(s, a, r, s') | a ~ β(·|s)}
  
  Learned policy π may choose different actions:
    If π(a|s) ≠ β(a|s), we have NO data for those actions!

  What goes wrong:
    Q(s, a) for unseen actions → extrapolation → garbage values
    Policy exploits Q errors → selects bad actions confidently!

  ┌──────────────────────────────────────────────────┐
  │  Q-value                                          │
  │  ▲                                                │
  │  │     ████    True Q                             │
  │  │   ██    ██                                     │
  │  │  █        █    Estimated Q (with errors)       │
  │  │ █          █                                   │
  │  │█     ╱╲    ██████      ← spurious peak!       │
  │  │     ╱  ╲              (no data here)           │
  │  │    ╱    ╲                                      │
  │  └──────────────────────► actions                 │
  │       │data│ │no data│                            │
  │                                                    │
  │  Policy picks the spurious peak → terrible action!│
  └──────────────────────────────────────────────────┘
  
  Naive Q-learning on offline data → catastrophic failure
```

---

## Conservative Q-Learning (CQL)

```
Core idea: penalize Q-values for out-of-distribution actions

  Standard Q: min_φ E[(Q_φ(s,a) - (r + γ max_a' Q(s',a')))²]
  
  CQL adds a regularizer:
    min_φ  α × E_s[log Σ_a exp(Q(s,a)) - E_{a~β}[Q(s,a)]]
           + standard TD loss

  This pushes DOWN Q-values for all actions (log-sum-exp)
  and pushes UP Q-values for actions in the dataset (second term)
  
  Result: Q is CONSERVATIVE (lower bound on true Q)
  → Policy won't exploit overestimated values
  → Safe, pessimistic decisions

  ┌────────────────────────────────────────┐
  │ CQL effect:                             │
  │                                         │
  │ Before: Q overestimates unseen actions │
  │ After:  Q is pessimistic everywhere    │
  │         but accurate for seen actions   │
  │                                         │
  │ Policy sticks to familiar actions! ✓   │
  └────────────────────────────────────────┘
```

---

## Behavior Cloning with Constraints

```
Several approaches constrain policy to stay near behavior policy:

1. BCQ (Batch-Constrained Q-learning):
   Train generative model of behavior: G(s) → a
   Policy only considers actions near G's output
   
   a* = argmax_{a_i ~ G(s)} Q(s, a_i + perturbation)
   
   "Only consider actions the dataset would take"

2. BEAR (Bootstrapping Error Accumulation Reduction):
   Constraint: MMD(π(·|s), β(·|s)) ≤ ε
   Policy must stay close to behavior policy
   Uses support constraint (allow any in-distribution action)

3. TD3+BC:
   Simple: TD3 + behavior cloning penalty
   
   L_actor = -Q(s, π(s)) + λ × ||π(s) - a_data||²
   
   Balance: maximize Q but stay near data actions
   Extremely simple, surprisingly effective!
```

---

## Decision Transformer

```
Treat offline RL as a SEQUENCE MODELING problem!

  Input:  (R̂₁, s₁, a₁, R̂₂, s₂, a₂, ..., R̂_T, s_T, a_T)
  
  R̂_t = desired return-to-go from step t
  
  At test time:
    Set R̂₁ = desired total return (e.g., 1000)
    Model predicts: what action would achieve this return?
    
  ┌──────────────────────────────────────────────────┐
  │  Decision Transformer:                            │
  │                                                    │
  │  Input: [R̂=1000, s₀] → Output: a₀               │
  │  Input: [R̂=995, s₁]  → Output: a₁               │
  │  Input: [R̂=990, s₂]  → Output: a₂               │
  │         ...                                        │
  │                                                    │
  │  Uses GPT-style architecture!                     │
  │  No Q-values, no value functions                  │
  │  Just sequence prediction                         │
  │                                                    │
  │  Conditioning on HIGH return →                    │
  │    model outputs actions that ACHIEVE high return │
  └──────────────────────────────────────────────────┘

  Architecture: GPT with three token types per timestep:
    [return_token, state_token, action_token]
    
  Causal self-attention over the sequence
  Predict next action given history + desired return
```

---

## Python: Conservative Q-Learning (Simplified)

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class CQLCritic(nn.Module):
    def __init__(self, state_dim, action_dim):
        super().__init__()
        self.q1 = nn.Sequential(
            nn.Linear(state_dim + action_dim, 256), nn.ReLU(),
            nn.Linear(256, 256), nn.ReLU(),
            nn.Linear(256, 1))
        self.q2 = nn.Sequential(
            nn.Linear(state_dim + action_dim, 256), nn.ReLU(),
            nn.Linear(256, 256), nn.ReLU(),
            nn.Linear(256, 1))
    
    def forward(self, s, a):
        sa = torch.cat([s, a], -1)
        return self.q1(sa), self.q2(sa)

def cql_loss(critic, states, actions, rewards, next_states, dones,
             target_critic, actor, gamma=0.99, alpha=5.0, n_samples=10):
    """CQL: standard TD + conservative regularization."""
    
    # Standard TD target
    with torch.no_grad():
        next_actions, next_logp = actor.sample(next_states)
        tq1, tq2 = target_critic(next_states, next_actions)
        target_q = rewards + gamma * (1 - dones) * torch.min(tq1, tq2)
    
    # Current Q-values
    q1, q2 = critic(states, actions)
    td_loss = F.mse_loss(q1, target_q) + F.mse_loss(q2, target_q)
    
    # CQL regularization: penalize high Q for random actions
    batch_size = states.shape[0]
    
    # Sample random actions
    random_actions = torch.FloatTensor(
        batch_size * n_samples, actions.shape[-1]
    ).uniform_(-1, 1)
    
    states_rep = states.unsqueeze(1).repeat(1, n_samples, 1).reshape(-1, states.shape[-1])
    
    random_q1, random_q2 = critic(states_rep, random_actions)
    random_q1 = random_q1.reshape(batch_size, n_samples)
    random_q2 = random_q2.reshape(batch_size, n_samples)
    
    # log-sum-exp of Q for random actions (push down)
    cql_q1 = torch.logsumexp(random_q1, dim=1).mean()
    cql_q2 = torch.logsumexp(random_q2, dim=1).mean()
    
    # Q for dataset actions (push up)
    data_q1 = q1.mean()
    data_q2 = q2.mean()
    
    cql_reg = (cql_q1 - data_q1) + (cql_q2 - data_q2)
    
    return td_loss + alpha * cql_reg
```

---

## Key Methods Comparison

```
  Method      Approach           Complexity   Performance
  ──────      ────────           ──────────   ───────────
  BCQ         Constrain to data  Medium       Good
  BEAR        MMD constraint     High         Good
  CQL         Conservative Q     Medium       Very good
  TD3+BC      TD3 + BC penalty   Low          Good
  Decision    Sequence model     High         Very good
  Transformer (GPT-style)
  IQL         In-sample learning Low          Very good
```

---

## Revision Questions

1. **Why can't standard online RL algorithms (DQN, SAC) be directly applied to offline data?**
2. **What is distributional shift and how does it cause Q-value extrapolation errors?**
3. **How does CQL prevent overestimation of out-of-distribution actions?**
4. **How does the Decision Transformer frame RL as sequence modeling?**
5. **What real-world domains benefit most from offline RL and why?**

---

[Previous: 04-inverse-rl.md](04-inverse-rl.md) | [Next: 06-rlhf.md](06-rlhf.md)
