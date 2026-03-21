# Baseline and Variance Reduction

## Overview

Raw REINFORCE suffers from high variance, making learning slow and unstable. **Baselines** subtract a state-dependent value from the return, reducing variance without introducing bias. The most common baseline is the learned value function V(s), which leads naturally to the **advantage function** A(s,a) = Q(s,a) - V(s).

---

## Why Baselines Work

```
REINFORCE gradient:
  ∇J = E[∇log π(a|s;θ) × G_t]

Problem: If all returns are positive (common!), all actions get reinforced:
  G = [10, 8, 12, 7]  → all positive → all actions "good"
  
  But relative to average (9.25):
  G - b = [0.75, -1.25, 2.75, -2.25]
  → Now some actions reinforced, some discouraged!

REINFORCE with baseline:
  ∇J = E[∇log π(a|s;θ) × (G_t - b(s_t))]

  b(s) can be ANY function of state (not action!)
  Still unbiased: E[∇log π × b(s)] = 0  (provably)
  But variance is reduced!

  Proof that baseline doesn't add bias:
    E_a[∇log π(a|s) × b(s)] = b(s) × E_a[∇log π(a|s)]
                              = b(s) × ∇Σ_a π(a|s)
                              = b(s) × ∇1 = 0  ✓
```

---

## Common Baselines

```
1. Constant baseline: b = average return
   b = (1/N) Σ G_t
   Simple but doesn't adapt per state

2. Moving average: b = running mean of returns
   Better but still state-independent

3. Learned value function: b(s) = V̂(s; w)  ← BEST
   State-dependent → adapts to each state's expected return
   
   G_t - V̂(s_t) ≈ Q(s_t, a_t) - V(s_t) = A(s_t, a_t)
   
   = Advantage function!
   
   Positive advantage: action is BETTER than average → reinforce
   Negative advantage: action is WORSE than average → discourage
   Zero advantage: action is average → no change

  ┌────────────────────────────────────────┐
  │ Without baseline:                       │
  │   G = [10, 8, 12, 7]                   │
  │   All positive → all reinforced 😕     │
  │                                         │
  │ With V(s) baseline:                     │
  │   A = G - V = [+1, -1.5, +3, -2.5]    │
  │   Clear signal: +3 is great, -2.5 bad! │
  └────────────────────────────────────────┘
```

---

## Advantage Function

```
A^π(s, a) = Q^π(s, a) - V^π(s)

  "How much better is action a than average in state s?"

  A(s, a) > 0:  Action better than expected
  A(s, a) = 0:  Action is average
  A(s, a) < 0:  Action worse than expected

  Property: E_a[A(s,a)] = 0 (advantages sum to zero)
  
  This is exactly what we want for the gradient:
    ∇J = E[∇log π(a|s;θ) × A(s, a)]
    
  Good actions (A > 0) → increase probability
  Bad actions (A < 0) → decrease probability
  Average actions (A ≈ 0) → barely change
```

---

## REINFORCE with Baseline

```
Algorithm:

  Initialize policy π(a|s; θ) and value V̂(s; w)
  
  For each episode:
    Collect trajectory: s₀,a₀,r₀,...,s_T,a_T,r_T
    
    For t = 0 to T:
      G_t = Σ_{k=t}^T γ^(k-t) r_k
      δ_t = G_t - V̂(s_t; w)          ← advantage estimate
      
      θ ← θ + α_π × ∇log π(a_t|s_t;θ) × δ_t    (policy update)
      w ← w + α_v × δ_t × ∇V̂(s_t; w)            (value update)

  Two networks to learn:
    Policy π(a|s; θ): actor  — selects actions
    Value V̂(s; w):    critic — evaluates states
    
  → This is the Actor-Critic framework!
```

---

## Other Variance Reduction Techniques

```
1. Reward-to-go (causality):
   Only use future rewards, not past:
   
   Full return:      ∇J ∝ ∇log π(a_t|s_t) × Σ_{k=0}^T r_k
   Reward-to-go:     ∇J ∝ ∇log π(a_t|s_t) × Σ_{k=t}^T r_k  ← smaller!
   
   Past rewards are independent of a_t → they add noise

2. Return normalization:
   G_t ← (G_t - μ_G) / (σ_G + ε)
   Centers and scales returns → more stable gradients

3. Entropy regularization:
   Add H(π) bonus to encourage exploration:
   L = -E[log π × A] - β × H(π)
   H(π) = -Σ π(a|s) log π(a|s)
   Prevents premature convergence to deterministic policy

4. Multiple parallel environments:
   Run N environments simultaneously
   Average gradients → lower variance by √N
```

---

## Python: REINFORCE with Baseline

```python
import torch
import torch.nn as nn
import gymnasium as gym

class ActorCriticNet(nn.Module):
    def __init__(self, n_obs, n_actions):
        super().__init__()
        self.shared = nn.Sequential(
            nn.Linear(n_obs, 128), nn.ReLU())
        self.actor = nn.Linear(128, n_actions)    # policy
        self.critic = nn.Linear(128, 1)           # value baseline
    
    def forward(self, x):
        features = self.shared(x)
        logits = self.actor(features)
        value = self.critic(features)
        return torch.distributions.Categorical(logits=logits), value

env = gym.make("CartPole-v1")
model = ActorCriticNet(4, 2)
optimizer = torch.optim.Adam(model.parameters(), lr=1e-3)
gamma = 0.99

for episode in range(1000):
    log_probs, values, rewards = [], [], []
    state, _ = env.reset()
    done = False
    
    while not done:
        s_t = torch.FloatTensor(state)
        dist, value = model(s_t)
        action = dist.sample()
        
        log_probs.append(dist.log_prob(action))
        values.append(value.squeeze())
        
        state, reward, term, trunc, _ = env.step(action.item())
        rewards.append(reward)
        done = term or trunc
    
    # Compute returns
    returns = []
    G = 0
    for r in reversed(rewards):
        G = r + gamma * G
        returns.insert(0, G)
    returns = torch.FloatTensor(returns)
    
    # Advantages = returns - baseline
    values_t = torch.stack(values)
    advantages = returns - values_t.detach()
    
    # Losses
    policy_loss = -(torch.stack(log_probs) * advantages).sum()
    value_loss = nn.functional.mse_loss(values_t, returns)
    
    loss = policy_loss + 0.5 * value_loss
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()
```

---

## Revision Questions

1. **Why does subtracting a baseline reduce variance without adding bias?**
2. **What is the advantage function and what does its sign mean?**
3. **Why is V(s) the optimal baseline choice?**
4. **How does reward-to-go reduce variance compared to full returns?**
5. **What is entropy regularization and why is it useful?**

---

[Previous: 02-reinforce.md](02-reinforce.md) | [Next: 04-actor-critic.md](04-actor-critic.md)
