# Model-Based Reinforcement Learning

## Overview

**Model-based RL** learns a dynamics model of the environment (transition function and reward function) and uses it for planning. Instead of relying solely on real interactions, the agent can "imagine" trajectories using the learned model, dramatically improving sample efficiency. The tradeoff is model accuracy — errors in the learned model can lead to poor policies.

---

## Model-Free vs Model-Based

```
Model-Free (DQN, PPO, SAC):
  Agent → Action → Environment → Reward, Next State
  Learn directly from interaction
  No explicit model of how the world works

Model-Based:
  Agent → Action → Learned Model → Predicted Reward, Next State
  Learn a model first, then plan using the model
  Can also mix real and simulated experience

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Model-Free:                                      │
  │    Real env → (s,a,r,s') → Update policy          │
  │    Simple but needs millions of interactions       │
  │                                                    │
  │  Model-Based:                                     │
  │    Real env → (s,a,r,s') → Learn model            │
  │    Model → simulated (s,a,r,s') → Update policy  │
  │    Much fewer real interactions needed!             │
  │                                                    │
  │  Sample efficiency comparison:                    │
  │    Model-free:  ~10M steps (Atari)                │
  │    Model-based: ~100K steps (100x fewer!)          │
  └──────────────────────────────────────────────────┘
```

---

## Learning the Model

```
The model consists of:
  Transition: s_{t+1} = f_θ(s_t, a_t)    or P(s'|s,a)
  Reward:     r_t = g_φ(s_t, a_t)         or R(s,a)

  Training: supervised learning on collected data
    Input:  (state, action)
    Output: (next_state, reward)
    Loss:   MSE or negative log-likelihood

  Types of models:
  
  1. Deterministic: s' = f(s, a)
     Simple neural network
     Fast but can't capture stochasticity

  2. Probabilistic: P(s'|s,a) = N(μ(s,a), σ(s,a))
     Gaussian output → captures uncertainty
     Essential for knowing what the model doesn't know

  3. Ensemble: Train K models → disagreement = uncertainty
     ┌──────┐
     │Model 1│ → s'₁
     │Model 2│ → s'₂   Agreement = confident
     │Model 3│ → s'₃   Disagreement = uncertain
     │...    │
     │Model K│ → s'_K
     └──────┘
```

---

## Planning with Learned Models

```
Once we have a model, several planning approaches:

1. RANDOM SHOOTING (simplest):
   Generate N random action sequences
   Simulate each through model
   Pick the one with highest predicted return
   Execute first action, repeat (MPC style)

2. CROSS-ENTROPY METHOD (CEM):
   Initialize action distribution
   Sample N sequences → simulate → evaluate
   Keep top K sequences → refit distribution
   Repeat for M iterations
   
3. BACKPROPAGATION THROUGH MODEL:
   Unroll model for H steps
   Compute total predicted reward
   Backpropagate gradient to optimize actions or policy
   
   J = Σ_{t=0}^H γ^t r_model(s_t, π(s_t))
   ∇_θ J through the model computation graph

4. DYNA-STYLE (mix real + simulated):
   Collect real data → update model
   Generate synthetic data from model
   Update policy with both real + synthetic
```

---

## Dyna Architecture

```
Dyna-Q (Sutton, 1991):

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Real Experience:  s,a → env → r,s'               │
  │       │                                            │
  │       ├──→ Direct RL update (model-free)           │
  │       │                                            │
  │       └──→ Update model: f(s,a) ≈ (r,s')          │
  │                │                                    │
  │                └──→ Simulated planning:             │
  │                     Repeat K times:                │
  │                       Sample s,a from experience  │
  │                       r̂,ŝ' = model(s,a)           │
  │                       RL update with (s,a,r̂,ŝ')   │
  │                                                    │
  └──────────────────────────────────────────────────┘

  K simulated updates per real step → K× faster learning
  Model accuracy limits how much simulation helps
```

---

## Modern Model-Based Methods

```
1. WORLD MODELS (Ha & Schmidhuber, 2018):
   VAE encodes observations → latent state z
   RNN predicts future latent states
   Controller (small network) acts in latent space
   Train entirely in "dream" of the environment!

2. DREAMER (Hafner et al., 2020):
   Learn latent dynamics model
   Imagine trajectories in latent space
   Train actor-critic on imagined data
   State-of-the-art on continuous control from pixels

3. MBPO (Janner et al., 2019):
   Ensemble of probabilistic models
   Generate short model rollouts (branched from real data)
   Train SAC on real + model data
   Key: short rollouts → less model error compounding

4. MuZero (Schrittwieser et al., 2020):
   Learn model in latent space (not pixel space)
   Plan with MCTS using learned model
   Superhuman Atari, Chess, Go without knowing rules!
```

---

## Python: Simple Dyna-Q

```python
import numpy as np
from collections import defaultdict

class DynaQ:
    def __init__(self, n_states, n_actions, alpha=0.1, gamma=0.99,
                 epsilon=0.1, n_planning=5):
        self.Q = np.zeros((n_states, n_actions))
        self.model = {}
        self.alpha = alpha
        self.gamma = gamma
        self.epsilon = epsilon
        self.n_planning = n_planning
        self.visited = []
    
    def select_action(self, state):
        if np.random.random() < self.epsilon:
            return np.random.randint(self.Q.shape[1])
        return np.argmax(self.Q[state])
    
    def update(self, s, a, r, s_next, done):
        # 1. Direct RL (Q-learning update)
        target = r + self.gamma * np.max(self.Q[s_next]) * (1 - done)
        self.Q[s, a] += self.alpha * (target - self.Q[s, a])
        
        # 2. Update model
        self.model[(s, a)] = (r, s_next, done)
        if (s, a) not in self.visited:
            self.visited.append((s, a))
        
        # 3. Planning: K simulated updates
        for _ in range(self.n_planning):
            idx = np.random.randint(len(self.visited))
            s_sim, a_sim = self.visited[idx]
            r_sim, s_next_sim, done_sim = self.model[(s_sim, a_sim)]
            
            target = r_sim + self.gamma * np.max(self.Q[s_next_sim]) * (1 - done_sim)
            self.Q[s_sim, a_sim] += self.alpha * (target - self.Q[s_sim, a_sim])
```

---

## Challenges

```
1. Model error compounds over long rollouts:
   1-step error: small
   10-step error: 10× larger (roughly)
   100-step error: policy may be completely wrong
   → Solution: use short rollouts (MBPO uses 1-5 steps)

2. Distribution shift:
   Model trained on data from old policy
   New policy visits new states → model is wrong there
   → Solution: ensemble disagreement, continual model updates

3. Computational cost:
   Learning + using model adds overhead
   Planning (especially MCTS) can be expensive
   → Tradeoff: fewer real interactions but more computation

4. Partial observability:
   Model needs Markov state, but observations may not be
   → Solution: learn latent state (Dreamer, MuZero)
```

---

## Revision Questions

1. **What is the key advantage of model-based over model-free RL?**
2. **What is the Dyna architecture and how does it combine real and simulated experience?**
3. **Why do model errors compound over long rollouts?**
4. **How do ensemble models help with model uncertainty?**
5. **What is MuZero and why is it significant?**

---

[Next: 02-multi-agent-rl.md](02-multi-agent-rl.md) | [Back to README](../README.md)
