# Policy Gradient Theorem

## Overview

**Policy gradient methods** directly parameterize and optimize the policy π(a|s; θ), rather than learning a value function and deriving a policy from it. The **policy gradient theorem** provides the mathematical foundation — an expression for the gradient of expected return with respect to policy parameters, enabling gradient ascent on performance.

---

## Why Policy Gradients?

```
Value-based (DQN):  Learn Q(s,a) → derive policy (argmax)
  Limitations:
    - Only discrete actions (can't argmax over continuous)
    - Deterministic policy (can't represent stochastic)
    - Small changes in Q → drastic policy changes

Policy-based:  Directly optimize π(a|s; θ)
  Advantages:
    ✓ Continuous action spaces (natural parameterization)
    ✓ Stochastic policies (exploration built-in)
    ✓ Smooth policy updates (small θ change → small policy change)
    ✓ Can learn true optimal stochastic policies

  Example — Rock-Paper-Scissors:
    Optimal policy is stochastic: π(R)=π(P)=π(S)=1/3
    Value-based methods can't represent this naturally!
```

---

## Policy Parameterization

```
Discrete actions — Softmax policy:
  π(a|s; θ) = exp(h(s,a;θ)) / Σ_{a'} exp(h(s,a';θ))
  
  h(s,a;θ) = preference function (e.g., neural network output)
  
  ┌────────┐     ┌────────┐     softmax     ┌──────────────┐
  │ State s│ →  │ NN(θ) │ ──────────────→ │ π(a₁|s)=0.7 │
  └────────┘     │        │                 │ π(a₂|s)=0.2 │
                  │ h(s,·) │                 │ π(a₃|s)=0.1 │
                  └────────┘                 └──────────────┘

Continuous actions — Gaussian policy:
  π(a|s; θ) = N(a | μ(s;θ), σ²(s;θ))
  
  NN outputs mean μ and log-std σ for each action dimension
  
  ┌────────┐     ┌────────┐     ┌───────────────────┐
  │ State s│ →  │ NN(θ) │ →  │ μ = 2.3           │
  └────────┘     └────────┘     │ σ = 0.5           │
                                 │ a ~ N(2.3, 0.25)  │
                                 └───────────────────┘
```

---

## The Objective

```
Maximize expected return:
  J(θ) = E_{τ~π_θ}[R(τ)] = E[Σ_t γ^t r_t]
  
  τ = trajectory: (s₀, a₀, r₀, s₁, a₁, r₁, ...)
  
  We want: θ* = argmax_θ J(θ)
  
  Approach: Gradient ascent
    θ ← θ + α ∇_θ J(θ)
    
  But how to compute ∇_θ J(θ)?
    J(θ) depends on θ through the policy AND the trajectory distribution
    The environment dynamics P(s'|s,a) are unknown!
```

---

## The Policy Gradient Theorem

```
Theorem (Sutton et al., 1999):

  ∇_θ J(θ) = E_{π_θ}[Σ_t ∇_θ log π(a_t|s_t; θ) × G_t]

  where G_t = Σ_{k=t}^T γ^(k-t) r_k  (return from time t)

Derivation sketch (using log-derivative trick):

  ∇_θ P(τ; θ) = P(τ; θ) × ∇_θ log P(τ; θ)    (log-derivative trick)
  
  log P(τ; θ) = log P(s₀) + Σ_t [log π(a_t|s_t;θ) + log P(s_{t+1}|s_t,a_t)]
                              ↑ depends on θ           ↑ doesn't depend on θ!
  
  ∇_θ log P(τ; θ) = Σ_t ∇_θ log π(a_t|s_t; θ)
  
  Therefore:
  ∇_θ J(θ) = E_τ[R(τ) × Σ_t ∇_θ log π(a_t|s_t; θ)]

  Key insight: Don't need to know P(s'|s,a)!
  Only need ∇_θ log π(a_t|s_t; θ) — the score function

Intuition:
  ∇_θ log π(a|s; θ)  =  direction to make action a MORE likely
  
  If G_t > 0: Move θ to make action a more likely (good outcome)
  If G_t < 0: Move θ to make action a less likely (bad outcome)
  
  → Reinforce actions that led to good returns!
```

---

## Score Function

```
The score function ∇_θ log π(a|s; θ):

  Discrete (softmax):
    ∇_θ log π(a|s; θ) = ∇_θ h(s,a;θ) - E_{a'~π}[∇_θ h(s,a';θ)]
    = feature of chosen action - expected feature
    
  Continuous (Gaussian):
    π(a|s;θ) = N(μ_θ(s), σ²_θ(s))
    
    ∇_μ log π = (a - μ) / σ²        (push μ toward a if good)
    ∇_σ log π = ((a-μ)² - σ²) / σ³  (adjust spread)
    
  In practice: Use automatic differentiation!
    log_prob = distribution.log_prob(action)
    log_prob.backward()  → computes ∇_θ log π automatically
```

---

## Python: Computing Policy Gradient

```python
import torch
import torch.nn as nn
import torch.distributions as D

class PolicyNetwork(nn.Module):
    def __init__(self, n_obs, n_actions):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(n_obs, 64), nn.ReLU(),
            nn.Linear(64, 64), nn.ReLU(),
            nn.Linear(64, n_actions)
        )
    
    def forward(self, x):
        logits = self.net(x)
        return D.Categorical(logits=logits)

# Computing the policy gradient
policy = PolicyNetwork(4, 2)
state = torch.FloatTensor([0.1, -0.2, 0.3, 0.0])

dist = policy(state)
action = dist.sample()           # a ~ π(·|s; θ)
log_prob = dist.log_prob(action)  # log π(a|s; θ)

# After collecting return G_t:
G_t = 15.0
loss = -log_prob * G_t   # negative because we maximize
loss.backward()           # ∇_θ [log π(a|s;θ) × G_t]
```

---

## Revision Questions

1. **What advantages do policy gradient methods have over value-based methods?**
2. **What is the policy gradient theorem and what does it say?**
3. **Why doesn't the policy gradient require knowledge of environment dynamics?**
4. **What is the log-derivative trick and how is it used?**
5. **How does the sign of G_t determine whether an action is reinforced or discouraged?**

---

[Next: 02-reinforce.md](02-reinforce.md)
