# Soft Actor-Critic (SAC)

## Overview

**SAC** is an off-policy actor-critic algorithm that maximizes both expected return AND policy entropy. By adding an entropy bonus to the reward, SAC encourages exploration and learns robust, stochastic policies. SAC is considered state-of-the-art for continuous control tasks and combines three key ideas: off-policy learning, actor-critic architecture, and maximum entropy RL.

---

## Maximum Entropy RL

```
Standard RL objective:
  max E[Σ γ^t r_t]          (maximize return)

Maximum Entropy RL objective:
  max E[Σ γ^t (r_t + α H(π(·|s_t)))]

  = max E[Σ γ^t (r_t - α log π(a_t|s_t))]

  α = temperature parameter (how much to value entropy)

  Why entropy bonus?
  ┌──────────────────────────────────────────────────┐
  │ 1. Exploration: high entropy → try diverse actions│
  │ 2. Robustness: stochastic policy → handles noise │
  │ 3. Multi-modal: discovers multiple good solutions│
  │ 4. No collapse: prevents premature convergence   │
  │ 5. Transfer: general policies transfer better     │
  └──────────────────────────────────────────────────┘

  Standard RL: finds ONE optimal action (deterministic)
  MaxEnt RL:   finds ALL good actions (stochastic)
  
  Example: reaching a goal
    Standard: always takes shortest path
    MaxEnt:   takes many different routes (all near-optimal)
```

---

## SAC Architecture

```
SAC uses 5 networks:

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Actor: π_θ(a|s)     → stochastic policy          │
  │    Outputs: mean μ and std σ of Gaussian          │
  │    a = tanh(μ + σ × ε),  ε ~ N(0,1)              │
  │                                                    │
  │  Critic 1: Q_φ₁(s,a) → value estimate 1          │
  │  Critic 2: Q_φ₂(s,a) → value estimate 2          │
  │    (twin critics to reduce overestimation)         │
  │                                                    │
  │  Target 1: Q_φ₁'(s,a) → slowly updated copy      │
  │  Target 2: Q_φ₂'(s,a) → slowly updated copy      │
  │    (Polyak averaged: φ' ← τφ + (1-τ)φ')          │
  │                                                    │
  │  Temperature: α (auto-tuned)                      │
  └──────────────────────────────────────────────────┘

  Data flow:
    s → Actor → a (sampled from Gaussian)
         ↓
    (s,a) → Critic 1 → Q₁
    (s,a) → Critic 2 → Q₂
         ↓
    min(Q₁, Q₂)  → used for updates (pessimistic)
```

---

## SAC Algorithm

```
Initialize: π_θ, Q_φ₁, Q_φ₂, Q_φ₁', Q_φ₂', replay buffer D

For each step:
  1. INTERACT:
     a ~ π_θ(·|s)
     s', r = env.step(a)
     D.store(s, a, r, s', done)

  2. SAMPLE minibatch from D: {(s, a, r, s', d)}

  3. COMPUTE target:
     a' ~ π_θ(·|s')                          (sample next action)
     log_π' = log π_θ(a'|s')                 (entropy term)
     
     Q_target = r + γ(1-d) × (min(Q_φ₁'(s',a'), Q_φ₂'(s',a')) - α × log_π')
     
     ──── reward ─── ──── pessimistic Q ──── ── entropy ──

  4. UPDATE critics:
     L_Q₁ = E[(Q_φ₁(s,a) - Q_target)²]
     L_Q₂ = E[(Q_φ₂(s,a) - Q_target)²]

  5. UPDATE actor:
     a_new ~ π_θ(·|s)
     L_π = E[α × log π_θ(a_new|s) - min(Q_φ₁(s,a_new), Q_φ₂(s,a_new))]
     
     = "minimize entropy-weighted log prob + maximize Q"

  6. UPDATE temperature (automatic):
     L_α = E[-α × (log π_θ(a|s) + H_target)]
     
     H_target = -dim(action_space)   (heuristic)
     If policy entropy < target → decrease α → less exploration
     If policy entropy > target → increase α → more exploration

  7. SOFT UPDATE targets:
     φ₁' ← τ × φ₁ + (1-τ) × φ₁'
     φ₂' ← τ × φ₂ + (1-τ) × φ₂'
     
     τ = 0.005 (very slow update)
```

---

## Squashed Gaussian Policy

```
SAC uses a squashed Gaussian for continuous actions:

  z ~ N(μ_θ(s), σ_θ(s))     (sample from Gaussian)
  a = tanh(z)                 (squash to [-1, 1])

  Log probability correction (change of variables):
    log π(a|s) = log N(z; μ, σ) - Σ log(1 - tanh²(z_i))
    
    The second term corrects for the tanh transformation.
    Without it: log probs are wrong → training fails!

  Why tanh?
  - Actions bounded to [-1, 1] (standard for continuous control)
  - Differentiable everywhere
  - Smooth saturation at boundaries
```

---

## Python: SAC Core Components

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class GaussianActor(nn.Module):
    def __init__(self, n_obs, n_actions):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(n_obs, 256), nn.ReLU(),
            nn.Linear(256, 256), nn.ReLU())
        self.mu = nn.Linear(256, n_actions)
        self.log_std = nn.Linear(256, n_actions)
    
    def forward(self, state):
        x = self.net(state)
        mu = self.mu(x)
        log_std = self.log_std(x).clamp(-20, 2)
        return mu, log_std
    
    def sample(self, state):
        mu, log_std = self.forward(state)
        std = log_std.exp()
        dist = torch.distributions.Normal(mu, std)
        z = dist.rsample()  # reparameterization trick
        action = torch.tanh(z)
        
        # Log prob with squashing correction
        log_prob = dist.log_prob(z) - torch.log(1 - action.pow(2) + 1e-6)
        log_prob = log_prob.sum(dim=-1, keepdim=True)
        
        return action, log_prob

class TwinQCritic(nn.Module):
    def __init__(self, n_obs, n_actions):
        super().__init__()
        self.q1 = nn.Sequential(
            nn.Linear(n_obs + n_actions, 256), nn.ReLU(),
            nn.Linear(256, 256), nn.ReLU(),
            nn.Linear(256, 1))
        self.q2 = nn.Sequential(
            nn.Linear(n_obs + n_actions, 256), nn.ReLU(),
            nn.Linear(256, 256), nn.ReLU(),
            nn.Linear(256, 1))
    
    def forward(self, state, action):
        sa = torch.cat([state, action], dim=-1)
        return self.q1(sa), self.q2(sa)

class SAC:
    def __init__(self, n_obs, n_actions, gamma=0.99, tau=0.005, lr=3e-4):
        self.actor = GaussianActor(n_obs, n_actions)
        self.critic = TwinQCritic(n_obs, n_actions)
        self.target_critic = TwinQCritic(n_obs, n_actions)
        self.target_critic.load_state_dict(self.critic.state_dict())
        
        self.log_alpha = torch.zeros(1, requires_grad=True)
        self.target_entropy = -n_actions
        
        self.actor_opt = torch.optim.Adam(self.actor.parameters(), lr=lr)
        self.critic_opt = torch.optim.Adam(self.critic.parameters(), lr=lr)
        self.alpha_opt = torch.optim.Adam([self.log_alpha], lr=lr)
        
        self.gamma, self.tau = gamma, tau
    
    def update(self, batch):
        s, a, r, s_next, done = batch
        alpha = self.log_alpha.exp().detach()
        
        # --- Critic update ---
        with torch.no_grad():
            next_a, next_logp = self.actor.sample(s_next)
            tq1, tq2 = self.target_critic(s_next, next_a)
            target_q = r + self.gamma * (1 - done) * (
                torch.min(tq1, tq2) - alpha * next_logp)
        
        q1, q2 = self.critic(s, a)
        critic_loss = F.mse_loss(q1, target_q) + F.mse_loss(q2, target_q)
        
        self.critic_opt.zero_grad()
        critic_loss.backward()
        self.critic_opt.step()
        
        # --- Actor update ---
        new_a, logp = self.actor.sample(s)
        q1_new, q2_new = self.critic(s, new_a)
        actor_loss = (alpha * logp - torch.min(q1_new, q2_new)).mean()
        
        self.actor_opt.zero_grad()
        actor_loss.backward()
        self.actor_opt.step()
        
        # --- Temperature update ---
        alpha_loss = -(self.log_alpha.exp() * (logp + self.target_entropy).detach()).mean()
        self.alpha_opt.zero_grad()
        alpha_loss.backward()
        self.alpha_opt.step()
        
        # --- Soft update targets ---
        for tp, p in zip(self.target_critic.parameters(), self.critic.parameters()):
            tp.data.lerp_(p.data, self.tau)
```

---

## SAC vs Other Algorithms

```
  Feature          PPO        SAC        TD3
  ────────         ───        ───        ───
  Policy type      On-policy  Off-policy Off-policy
  Action space     Any        Continuous Continuous
  Exploration      Entropy    MaxEnt     Noise
  Replay buffer    No         Yes        Yes
  Stochastic π     Yes        Yes        No (det.)
  Sample eff.      Low        High       High
  Stability        High       High       High
  Typical use      Games,     Robotics,  Robotics,
                   discrete   continuous continuous
```

---

## Revision Questions

1. **What is maximum entropy RL and why is it beneficial?**
2. **Why does SAC use twin critics (clipped double Q)?**
3. **How does the automatic temperature tuning work?**
4. **What is the squashed Gaussian policy and why is the log-prob correction needed?**
5. **When would you choose SAC over PPO?**

---

[Previous: 02-ppo.md](02-ppo.md) | [Next: 04-ddpg.md](04-ddpg.md)
