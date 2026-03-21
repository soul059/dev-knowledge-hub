# DQN Architecture

## Overview

**Deep Q-Network (DQN)** (Mnih et al., 2015) was the breakthrough that launched deep reinforcement learning. It combines Q-learning with deep neural networks to learn directly from high-dimensional inputs (raw pixels), achieving human-level performance on Atari games. DQN introduced two critical innovations — experience replay and target networks — to stabilize training.

---

## From Q-Learning to DQN

```
Tabular Q-Learning:
  Q-table: |S| × |A| entries
  Works for small, discrete state spaces
  
  Atari: 210×160×3 pixels, 256 colors
    States ≈ 256^(210×160×3) → impossibly large table!

DQN solution: Approximate Q with a neural network

  Tabular:               DQN:
  ┌────────────────┐     ┌────────────────┐
  │  Q(s, a)       │     │  State (pixels)│
  │  ┌───┬───┬───┐ │     │       │        │
  │  │2.1│3.5│1.2│ │     │  ┌────▼────┐   │
  │  ├───┼───┼───┤ │     │  │  CNN    │   │
  │  │...│...│...│ │     │  │  layers │   │
  │  └───┴───┴───┘ │     │  └────┬────┘   │
  │ |S| × |A| table│     │  ┌────▼────┐   │
  └────────────────┘     │  │  FC     │   │
                          │  │  layers │   │
                          │  └────┬────┘   │
                          │  [Q(s,a₁), Q(s,a₂), ..., Q(s,aₙ)]
                          └────────────────┘
```

---

## Network Architecture

```
DQN for Atari:

  Input: 84×84×4 (4 stacked grayscale frames)
         ↓
  ┌──────────────────────────────────────┐
  │ Conv1: 32 filters, 8×8, stride 4    │ → 20×20×32
  │ ReLU                                │
  ├──────────────────────────────────────┤
  │ Conv2: 64 filters, 4×4, stride 2    │ → 9×9×64
  │ ReLU                                │
  ├──────────────────────────────────────┤
  │ Conv3: 64 filters, 3×3, stride 1    │ → 7×7×64
  │ ReLU                                │
  ├──────────────────────────────────────┤
  │ Flatten: 7×7×64 = 3136              │
  ├──────────────────────────────────────┤
  │ FC1: 3136 → 512, ReLU               │
  ├──────────────────────────────────────┤
  │ FC2: 512 → |A| (output Q values)    │
  └──────────────────────────────────────┘

  Why 4 stacked frames?
    Single frame can't show motion direction/speed
    4 frames → velocity and acceleration visible
    
  Why output all Q(s,a) at once?
    One forward pass → Q-values for ALL actions
    Much more efficient than separate pass per action
```

---

## DQN Algorithm

```
Initialize:
  Q-network with random weights θ
  Target network with weights θ⁻ = θ
  Replay buffer D (capacity N)

For each episode:
  Preprocess initial state s₁
  
  For each step t:
    With probability ε: select random action a_t
    Otherwise: a_t = argmax_a Q(s_t, a; θ)
    
    Execute a_t, observe reward r_t, next state s_{t+1}
    Store transition (s_t, a_t, r_t, s_{t+1}, done) in D
    
    Sample random minibatch of transitions from D
    
    For each transition (s, a, r, s', done):
      If done:
        y = r
      Else:
        y = r + γ × max_{a'} Q(s', a'; θ⁻)   ← target network!
      
      Loss = (y - Q(s, a; θ))²
    
    Gradient descent on θ to minimize loss
    
    Every C steps: θ⁻ ← θ   (update target network)

  Decrease ε over time (e.g., 1.0 → 0.1 over 1M frames)
```

---

## Key Innovations

```
1. Experience Replay (see 02-experience-replay.md)
   Store transitions, sample randomly for training
   → Breaks temporal correlations
   → Reuses data efficiently

2. Target Network (see 03-target-networks.md)  
   Separate network for computing TD targets
   → Stabilizes training by fixing target for C steps

3. Frame Preprocessing
   Raw Atari: 210×160×3 → Grayscale → Resize 84×84 → Stack 4 frames
   → Reduces input dimensionality
   → Encodes temporal information

4. Reward Clipping
   Clip all rewards to [-1, +1]
   → Same learning rate works across different games
   → Loses magnitude info but improves stability

5. Gradient Clipping
   Clip error term to [-1, 1] (equivalent to Huber loss)
   → Prevents large gradient updates
```

---

## Training Details

```
Hyperparameters (original DQN):

  Parameter                Value
  ─────────────────────── ──────
  Replay buffer size       1,000,000
  Minibatch size           32
  Discount γ               0.99
  Learning rate α          0.00025
  Target update freq C     10,000 steps
  ε start                  1.0
  ε end                    0.1
  ε decay over             1,000,000 frames
  Training starts after    50,000 frames
  Optimizer                RMSprop
  Total training           50,000,000 frames (~38 days)

  ε-greedy schedule:
  ε
  1.0 ┤\
      │  \
  0.5 ┤    \
      │      \
  0.1 ┤────────────────────────────
      ┼───┬───┬───┬───┬───┬───┬──
      0  0.2  0.5  1M     10M   50M  frames
```

---

## Python: DQN Implementation

```python
import torch
import torch.nn as nn
import numpy as np
from collections import deque
import random

class DQN(nn.Module):
    def __init__(self, n_observations, n_actions):
        super().__init__()
        self.network = nn.Sequential(
            nn.Linear(n_observations, 128),
            nn.ReLU(),
            nn.Linear(128, 128),
            nn.ReLU(),
            nn.Linear(128, n_actions)
        )
    
    def forward(self, x):
        return self.network(x)

class ReplayBuffer:
    def __init__(self, capacity=10000):
        self.buffer = deque(maxlen=capacity)
    
    def push(self, state, action, reward, next_state, done):
        self.buffer.append((state, action, reward, next_state, done))
    
    def sample(self, batch_size):
        batch = random.sample(self.buffer, batch_size)
        states, actions, rewards, next_states, dones = zip(*batch)
        return (np.array(states), np.array(actions),
                np.array(rewards), np.array(next_states),
                np.array(dones))

# Training loop sketch
import gymnasium as gym

env = gym.make("CartPole-v1")
n_obs = env.observation_space.shape[0]
n_act = env.action_space.n

policy_net = DQN(n_obs, n_act)
target_net = DQN(n_obs, n_act)
target_net.load_state_dict(policy_net.state_dict())

optimizer = torch.optim.Adam(policy_net.parameters(), lr=1e-4)
buffer = ReplayBuffer(10000)
gamma = 0.99
epsilon = 1.0

for episode in range(500):
    state, _ = env.reset()
    total_reward = 0
    
    for step in range(500):
        # ε-greedy action selection
        if random.random() < epsilon:
            action = env.action_space.sample()
        else:
            with torch.no_grad():
                q_vals = policy_net(torch.FloatTensor(state))
                action = q_vals.argmax().item()
        
        next_state, reward, term, trunc, _ = env.step(action)
        done = term or trunc
        buffer.push(state, action, reward, next_state, done)
        state = next_state
        total_reward += reward
        
        # Train
        if len(buffer.buffer) >= 64:
            s, a, r, s2, d = buffer.sample(64)
            s_t = torch.FloatTensor(s)
            a_t = torch.LongTensor(a)
            r_t = torch.FloatTensor(r)
            s2_t = torch.FloatTensor(s2)
            d_t = torch.FloatTensor(d)
            
            q_values = policy_net(s_t).gather(1, a_t.unsqueeze(1)).squeeze()
            with torch.no_grad():
                next_q = target_net(s2_t).max(1)[0]
                targets = r_t + gamma * next_q * (1 - d_t)
            
            loss = nn.MSELoss()(q_values, targets)
            optimizer.zero_grad()
            loss.backward()
            optimizer.step()
        
        if done:
            break
    
    epsilon = max(0.01, epsilon * 0.995)
    if episode % 10 == 0:
        target_net.load_state_dict(policy_net.state_dict())
```

---

## Revision Questions

1. **Why can't we use tabular Q-learning for Atari games?**
2. **Why does DQN output Q-values for all actions simultaneously?**
3. **What are the two key innovations that stabilize DQN training?**
4. **Why are 4 frames stacked as input instead of 1?**
5. **How does the ε-greedy schedule change during training?**

---

[Next: 02-experience-replay.md](02-experience-replay.md)
