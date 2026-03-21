# Actor-Critic Methods

## Overview

**Actor-Critic** methods combine the best of policy gradient (actor) and value function (critic) approaches. The **actor** learns the policy π(a|s; θ), and the **critic** learns the value function V(s; w) to evaluate and guide the actor. Unlike REINFORCE, actor-critic methods can update at every step (no need to wait for episode completion).

---

## Architecture

```
  ┌──────────────────────────────────────────────────┐
  │               Actor-Critic                        │
  │                                                    │
  │  State s_t                                         │
  │     │                                              │
  │  ┌──┴──────────────────┐                           │
  │  │ Shared Features (optional)                      │
  │  └──┬──────────────────┘                           │
  │     │                                              │
  │  ┌──┴──┐         ┌──────┐                          │
  │  │Actor│         │Critic│                          │
  │  │π(a|s;θ)       │V(s;w)│                          │
  │  └──┬──┘         └──┬───┘                          │
  │     │               │                              │
  │  Action a_t      Value V(s_t)                      │
  │     │               │                              │
  │  ┌──▼──┐            │                              │
  │  │ Env │──── r_t, s_{t+1} ──┐                     │
  │  └─────┘                     │                     │
  │                              │                     │
  │  TD error: δ = r + γV(s') - V(s)                   │
  │                                                    │
  │  Actor update:  θ ← θ + α_θ × δ × ∇log π(a|s;θ)  │
  │  Critic update: w ← w + α_w × δ × ∇V(s;w)        │
  └──────────────────────────────────────────────────┘
```

---

## Algorithm

```
One-Step Actor-Critic (TD(0)):

  Initialize actor π(a|s;θ) and critic V(s;w)
  
  For each episode:
    s ← initial state
    
    For each step:
      a ~ π(·|s; θ)              (sample action from policy)
      Take a, observe r, s'
      
      δ = r + γ V(s'; w) - V(s; w)   (TD error)
      
      w ← w + α_w × δ × ∇_w V(s; w)    (update critic)
      θ ← θ + α_θ × δ × ∇_θ log π(a|s; θ)  (update actor)
      
      s ← s'

  Key difference from REINFORCE:
    REINFORCE:     Uses G_t (complete return) → needs full episode
    Actor-Critic:  Uses δ (TD error) → updates EVERY step!
    
    δ = r + γV(s') - V(s) ≈ A(s,a)  (one-step advantage estimate)
```

---

## Why Actor-Critic Is Better

```
                    REINFORCE          Actor-Critic
  ────────────────  ──────────         ────────────
  Update frequency  End of episode     Every step
  Variance          High (full G_t)    Lower (TD error)
  Bias              Unbiased           Slightly biased (bootstrap)
  Sample efficiency Low                Higher
  Online learning   No                 Yes
  
  Bias-Variance tradeoff:
  
  REINFORCE (MC):  Low bias, HIGH variance
    G_t = r_t + γr_{t+1} + γ²r_{t+2} + ...  (many random variables)
    
  Actor-Critic (TD): Some bias, LOWER variance
    δ = r_t + γV(s_{t+1}) - V(s_t)  (one random variable + bootstrap)
    
  In practice: lower variance wins → actor-critic learns faster
```

---

## N-Step Actor-Critic

```
Interpolate between 1-step TD and full MC:

  1-step: δ = r_t + γV(s_{t+1}) - V(s_t)
  2-step: δ = r_t + γr_{t+1} + γ²V(s_{t+2}) - V(s_t)
  n-step: δ = Σ_{k=0}^{n-1} γ^k r_{t+k} + γ^n V(s_{t+n}) - V(s_t)
  ∞-step: δ = G_t - V(s_t)    (= REINFORCE with baseline)

  ┌──────────────────────────────────────────┐
  │  n=1:  Low variance, more bias           │
  │  n=∞:  High variance, no bias            │
  │  n=5:  Often a good middle ground        │
  └──────────────────────────────────────────┘
  
  Choice of n depends on the environment.
```

---

## Shared vs Separate Networks

```
Option 1: Separate networks
  ┌────────┐     ┌────────┐
  │ Actor  │     │ Critic │    Independent parameters
  │ π(a|s) │     │ V(s)   │    More stable
  └────────┘     └────────┘    More parameters

Option 2: Shared backbone
  ┌────────────────┐
  │ Shared Backbone │           Shared features
  └───┬────────┬───┘           Fewer parameters
      │        │                Can help with feature reuse
  ┌───▼──┐ ┌──▼───┐            Risk: conflicting gradients
  │Actor │ │Critic│
  │ head │ │ head │
  └──────┘ └──────┘

  In practice: shared backbone is common
  Combined loss: L = L_actor + c × L_critic + β × H(π)
    c = 0.5 (critic coefficient)
    β = 0.01 (entropy bonus)
```

---

## Python Implementation

```python
import torch
import torch.nn as nn
import gymnasium as gym

class ActorCritic(nn.Module):
    def __init__(self, n_obs, n_actions):
        super().__init__()
        self.shared = nn.Sequential(
            nn.Linear(n_obs, 128), nn.ReLU())
        self.actor = nn.Linear(128, n_actions)
        self.critic = nn.Linear(128, 1)
    
    def forward(self, x):
        x = self.shared(x)
        policy = torch.distributions.Categorical(logits=self.actor(x))
        value = self.critic(x).squeeze(-1)
        return policy, value

env = gym.make("CartPole-v1")
model = ActorCritic(4, 2)
optimizer = torch.optim.Adam(model.parameters(), lr=3e-4)
gamma = 0.99

for episode in range(1000):
    state, _ = env.reset()
    done = False
    ep_reward = 0
    
    while not done:
        s_t = torch.FloatTensor(state)
        dist, value = model(s_t)
        action = dist.sample()
        
        next_state, reward, term, trunc, _ = env.step(action.item())
        done = term or trunc
        ep_reward += reward
        
        # Compute TD error
        with torch.no_grad():
            _, next_value = model(torch.FloatTensor(next_state))
            target = reward + gamma * next_value * (1 - float(done))
        
        td_error = target - value
        
        # Combined loss
        actor_loss = -dist.log_prob(action) * td_error.detach()
        critic_loss = td_error ** 2
        entropy = dist.entropy()
        
        loss = actor_loss + 0.5 * critic_loss - 0.01 * entropy
        
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()
        
        state = next_state
    
    if episode % 50 == 0:
        print(f"Episode {episode}, Reward: {ep_reward:.0f}")
```

---

## Revision Questions

1. **What are the two components of actor-critic and what does each learn?**
2. **Why can actor-critic update every step while REINFORCE cannot?**
3. **How does the TD error serve as an advantage estimate?**
4. **What is the bias-variance tradeoff between MC and TD actor-critic?**
5. **What are the pros and cons of shared vs separate networks?**

---

[Previous: 03-baseline-variance-reduction.md](03-baseline-variance-reduction.md) | [Next: 05-a2c-a3c.md](05-a2c-a3c.md)
