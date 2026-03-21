# Experience Replay

## Overview

**Experience replay** stores past transitions (s, a, r, s', done) in a buffer and randomly samples mini-batches for training. This simple idea is critical to DQN's success — it breaks correlated sequential data, enables data reuse, and stabilizes learning with function approximation.

---

## The Problem Without Replay

```
Online learning (no replay):
  Train on transitions as they arrive, sequentially

  Time:  t₁ → t₂ → t₃ → t₄ → t₅ → ...
  Data:  (s₁,a₁,r₁,s₂) → (s₂,a₂,r₂,s₃) → (s₃,a₃,r₃,s₄) → ...

  Problems:
  1. Correlated data:
     Consecutive transitions are highly correlated
     SGD assumes i.i.d. data → violates assumption → unstable
     
  2. Catastrophic forgetting:
     Agent in Room A → learns Room A well
     Moves to Room B → learns Room B, forgets Room A
     
  3. Data inefficiency:
     Each transition used once then discarded
     Rare/important transitions lost forever
```

---

## Experience Replay Buffer

```
  ┌────────────────────────────────────────┐
  │           Replay Buffer D              │
  │  capacity = N (e.g., 1,000,000)        │
  │                                        │
  │  ┌────────────────────────────────┐    │
  │  │ (s₁, a₁, r₁, s₂, done₁)     │    │
  │  │ (s₂, a₂, r₂, s₃, done₂)     │    │  ← oldest
  │  │ (s₃, a₃, r₃, s₄, done₃)     │    │
  │  │ ...                           │    │
  │  │ (sₖ, aₖ, rₖ, sₖ₊₁, doneₖ)  │    │  ← newest
  │  └────────────────────────────────┘    │
  │                                        │
  │  When full: overwrite oldest entries   │
  │  (circular buffer / FIFO)              │
  └────────────────────────────────────────┘

  Training step:
  1. Agent takes action → stores (s, a, r, s', done) in D
  2. Sample random mini-batch of B transitions from D
  3. Compute loss and update network
  
  ┌──────┐   store    ┌──────────┐  sample B  ┌──────────┐
  │Agent │ ──────────▶│  Buffer  │ ──────────▶│ Training │
  │      │            │  (FIFO)  │  randomly   │  Step    │
  └──────┘            └──────────┘            └──────────┘

  Benefits:
  ✓ Random sampling → breaks correlations → i.i.d.-like
  ✓ Each transition reused ~8× on average (1M buffer, 125K updates)
  ✓ Rare events persist in buffer for many updates
  ✓ Smoother learning curves
```

---

## Implementation

```python
import numpy as np
from collections import deque
import random

class ReplayBuffer:
    def __init__(self, capacity=100000):
        self.buffer = deque(maxlen=capacity)
    
    def push(self, state, action, reward, next_state, done):
        self.buffer.append((state, action, reward, next_state, done))
    
    def sample(self, batch_size):
        batch = random.sample(self.buffer, batch_size)
        states, actions, rewards, next_states, dones = zip(*batch)
        return (
            np.array(states, dtype=np.float32),
            np.array(actions, dtype=np.int64),
            np.array(rewards, dtype=np.float32),
            np.array(next_states, dtype=np.float32),
            np.array(dones, dtype=np.float32)
        )
    
    def __len__(self):
        return len(self.buffer)

# Usage
buffer = ReplayBuffer(capacity=100000)

# During interaction
buffer.push(state, action, reward, next_state, done)

# During training (only when buffer has enough samples)
if len(buffer) >= batch_size:
    states, actions, rewards, next_states, dones = buffer.sample(32)
    # ... compute loss and update
```

---

## Design Choices

```
Buffer size:
  Too small (1K):   Not enough diversity, correlations remain
  Too large (10M):  Memory-heavy, stale data, slow policy change
  Sweet spot:       100K - 1M (depends on problem)

  DQN original: 1,000,000 transitions
  At 84×84×4 uint8: ~28 GB → often use compression

Sampling strategy:
  Uniform:     Equal probability for all transitions
               Simple, effective baseline
               
  Prioritized: Higher probability for "surprising" transitions
               (see 06-prioritized-experience-replay.md)

When to start training:
  Wait until buffer has min_size samples (e.g., 50,000)
  Before that: only collect data with random policy
  Ensures enough diversity in initial batches

Update frequency:
  Train every K environment steps (K=1 or K=4 typical)
  Ratio of env steps to gradient steps matters
```

---

## Limitations

```
1. Uniform sampling ignores transition importance
   → Solved by Prioritized Experience Replay

2. Off-policy data may be very old/stale
   → Policy has changed significantly since collection
   → Partial fix: limit buffer size

3. Memory requirements
   For image-based RL with 1M buffer:
   → Frames: 84×84×1 byte × 4 frames × 1M = 28 GB
   → Tricks: store frames once, reconstruct stacks
   
4. Not suitable for on-policy methods
   On-policy (SARSA, A2C): need current policy data
   Replay data from old policy is off-policy!
   → Only works with off-policy algorithms (Q-learning, DQN)
```

---

## Revision Questions

1. **Why does training on sequential transitions cause instability?**
2. **How does random sampling from a replay buffer break correlations?**
3. **What is the trade-off in choosing replay buffer size?**
4. **Why can't experience replay be used with on-policy methods like SARSA?**
5. **How many times is each transition reused on average in DQN?**

---

[Previous: 01-dqn-architecture.md](01-dqn-architecture.md) | [Next: 03-target-networks.md](03-target-networks.md)
