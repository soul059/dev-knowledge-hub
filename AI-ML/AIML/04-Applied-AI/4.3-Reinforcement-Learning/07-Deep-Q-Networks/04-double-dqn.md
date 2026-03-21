# Double DQN

## Overview

**Double DQN** (van Hasselt et al., 2016) fixes the overestimation bias in standard DQN. Regular Q-learning uses the same network to both select and evaluate actions, which systematically overestimates Q-values. Double DQN decouples selection from evaluation using the policy and target networks.

---

## The Overestimation Problem

```
Standard DQN target:
  y = r + γ max_{a'} Q(s', a'; θ⁻)

  max operator is biased upward:
    E[max(X₁, X₂, ..., Xₙ)] ≥ max(E[X₁], E[X₂], ..., E[Xₙ])
    
  If Q-values have noise (they always do):
    Q(s', a₁) = true + noise₁
    Q(s', a₂) = true + noise₂
    
    max(Q(s',a₁), Q(s',a₂)) ≥ max(true₁, true₂)
    
  Example:
    True Q-values:     Q*(s',a₁) = 5.0,  Q*(s',a₂) = 5.0
    Noisy estimates:   Q(s',a₁)  = 5.3,  Q(s',a₂)  = 4.7
    
    max Q = 5.3 > 5.0 = max Q*    ← overestimated!
    
  Over many updates:
    Overestimation accumulates and propagates
    → Suboptimal policies
    → Unstable training

  ┌─────────────────────────────────────┐
  │  True Q    ─────────────            │
  │  DQN Q     ────────────── ↑ biased  │
  │                           ↑ upward  │
  │  Overestimation grows with:         │
  │  - More actions (more noise terms)  │
  │  - More noise in estimates          │
  │  - Longer training                  │
  └─────────────────────────────────────┘
```

---

## Double Q-Learning Solution

```
Key insight: Use DIFFERENT networks for selection and evaluation

  Standard DQN (single network for both):
    a* = argmax_{a'} Q(s', a'; θ⁻)     ← select action
    y  = r + γ Q(s', a*; θ⁻)            ← evaluate with SAME network
    
  Double DQN (decouple selection and evaluation):
    a* = argmax_{a'} Q(s', a'; θ)       ← select with POLICY network
    y  = r + γ Q(s', a*; θ⁻)            ← evaluate with TARGET network
    
  The noise that causes selection of a* is independent from
  the noise in the evaluation Q(s', a*; θ⁻)
  → Overestimation is greatly reduced!

  Comparison:
  ┌───────────────────────────────────────────────┐
  │ DQN:        y = r + γ Q(s', argmax Q(s',·;θ⁻); θ⁻) │
  │                                same ↑     same ↑     │
  │                                                       │
  │ Double DQN: y = r + γ Q(s', argmax Q(s',·;θ); θ⁻)   │
  │                           policy ↑    target ↑        │
  │                           (select)    (evaluate)      │
  └───────────────────────────────────────────────┘
```

---

## Algorithm

```
Double DQN — only ONE line changes from DQN:

  Sample minibatch (s, a, r, s', done) from replay buffer
  
  DQN target:
    y = r + γ × max_{a'} Q(s', a'; θ⁻)
  
  Double DQN target:                              ← ONLY CHANGE
    a* = argmax_{a'} Q(s', a'; θ)                 ← policy net selects
    y  = r + γ × Q(s', a*; θ⁻)                    ← target net evaluates
  
  Loss = (y - Q(s, a; θ))²
  Update θ via gradient descent
  
  Periodically: θ⁻ ← θ

  That's it! Minimal code change, significant improvement.
```

---

## Python Implementation

```python
import torch

def compute_dqn_target(policy_net, target_net, next_states, 
                       rewards, dones, gamma, double=True):
    with torch.no_grad():
        if double:
            # Double DQN: select actions with policy, evaluate with target
            best_actions = policy_net(next_states).argmax(1)
            next_q = target_net(next_states).gather(
                1, best_actions.unsqueeze(1)).squeeze(1)
        else:
            # Standard DQN: both select and evaluate with target
            next_q = target_net(next_states).max(1)[0]
        
        targets = rewards + gamma * next_q * (1 - dones)
    return targets

# Usage in training loop
targets = compute_dqn_target(
    policy_net, target_net, next_states_batch,
    rewards_batch, dones_batch, gamma=0.99, double=True
)
q_values = policy_net(states_batch).gather(
    1, actions_batch.unsqueeze(1)).squeeze(1)
loss = torch.nn.functional.mse_loss(q_values, targets)
```

---

## Impact

```
Results on Atari (from original paper):

  Game          DQN Score   Double DQN Score
  ──────────── ──────────  ────────────────
  Asterix       6,012       17,356   (+189%)
  Breakout        401          418   (+4%)
  Pong             21           21   (=)
  Space Invaders 1,976        3,154  (+60%)
  
  Average across 49 games:
    Double DQN outperforms DQN on 80% of games
    Reduces overestimation without hurting performance
    
  Overestimation comparison:
  
  Q-value
   ↑
   │    ╱╲  DQN (overestimated)
   │   ╱  ╲
   │  ╱    ╲────────────── 
   │ ╱ Double DQN (accurate)
   │╱─────────────────────
   │───── True value
   └──────────────────────→ Training steps
```

---

## Revision Questions

1. **Why does the max operator cause overestimation in Q-learning?**
2. **How does Double DQN decouple action selection from evaluation?**
3. **What is the exact difference in the target computation between DQN and Double DQN?**
4. **Why does using two different networks reduce overestimation?**
5. **How much code needs to change to convert DQN to Double DQN?**

---

[Previous: 03-target-networks.md](03-target-networks.md) | [Next: 05-dueling-dqn.md](05-dueling-dqn.md)
