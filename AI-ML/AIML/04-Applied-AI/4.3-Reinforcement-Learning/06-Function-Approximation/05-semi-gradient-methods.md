# Semi-Gradient Methods

## Overview

In RL with function approximation, the target for updating our value estimate itself depends on the current parameters (because we bootstrap). **Semi-gradient methods** acknowledge this by only differentiating part of the update — treating the bootstrapped target as fixed. This is computationally simpler and converges well with linear function approximation, though it can diverge with nonlinear approximators.

---

## Why Not Full Gradient?

```
Value approximation objective:
  Minimize: J(w) = E[(v_π(s) - v̂(s, w))²]

  True gradient:
    ∇J(w) = -2 E[(v_π(s) - v̂(s, w)) × ∇v̂(s, w)]

  Problem: We don't know v_π(s)!
  
  MC approach: Use G_t (actual return) as target
    w ← w + α[G_t - v̂(s, w)] × ∇v̂(s, w)
    ✓ G_t doesn't depend on w → true gradient!

  TD approach: Use R + γv̂(s', w) as target
    w ← w + α[R + γv̂(s', w) - v̂(s, w)] × ∇v̂(s, w)
    
    But the target R + γv̂(s', w) ALSO depends on w!
    We ignore this dependency → "semi-gradient"
    
    Full gradient would be:
    ∇[R + γv̂(s',w) - v̂(s,w)]² includes ∇v̂(s',w) term
    Semi-gradient drops the ∇v̂(s',w) term
```

---

## Semi-Gradient TD(0)

```
Algorithm:

  Initialize w arbitrarily
  
  For each episode:
    Initialize S
    For each step:
      Choose A from policy π
      Take action A, observe R, S'
      
      δ = R + γ v̂(S', w) - v̂(S, w)     ← TD error
      w ← w + α × δ × ∇v̂(S, w)          ← semi-gradient update
                        ↑
                Only gradient of v̂(S, w),
                NOT v̂(S', w) in the target
      
      S ← S'

  With linear approximation v̂(s, w) = w^T φ(s):
    ∇v̂(s, w) = φ(s)
    w ← w + α × [R + γ w^T φ(s') - w^T φ(s)] × φ(s)
```

---

## Semi-Gradient SARSA

```
On-policy TD control with function approximation:

  Initialize w arbitrarily
  
  For each episode:
    S ← initial state
    A ← ε-greedy from q̂(S, ·, w)
    
    For each step:
      Take action A, observe R, S'
      If S' is terminal:
        w ← w + α[R - q̂(S, A, w)] × ∇q̂(S, A, w)
        break
      A' ← ε-greedy from q̂(S', ·, w)
      
      δ = R + γ q̂(S', A', w) - q̂(S, A, w)
      w ← w + α × δ × ∇q̂(S, A, w)
      
      S ← S', A ← A'

  Linear case: q̂(s, a, w) = w^T φ(s, a)
    Feature vector φ(s, a) encodes state-action pair
```

---

## Semi-Gradient Q-Learning

```
Off-policy TD control with function approximation:

  δ = R + γ max_a' q̂(S', a', w) - q̂(S, A, w)
  w ← w + α × δ × ∇q̂(S, A, w)

  ⚠️ WARNING: Off-policy + function approximation + bootstrapping
              = "The Deadly Triad" → can DIVERGE!

  The Deadly Triad:
  ┌──────────────────────────────────────┐
  │ 1. Function approximation            │
  │ 2. Bootstrapping (TD targets)        │
  │ 3. Off-policy training               │
  │                                      │
  │ Any TWO are fine.                    │
  │ All THREE together → instability!    │
  └──────────────────────────────────────┘

  Solutions:
    - Experience replay + target networks (DQN)
    - Gradient TD methods (GTD, GTD2, TDC)
    - Emphatic TD methods
```

---

## Convergence Properties

```
                  Linear FA     Nonlinear FA
                  ─────────     ────────────
  MC (on-policy)  Converges ✓   Converges ✓
  
  Semi-grad TD    Converges ✓   May diverge ✗
  (on-policy)     (to TD fixed   (no guarantees)
                   point)
  
  Semi-grad TD    May diverge ✗  May diverge ✗
  (off-policy)    (deadly triad) (deadly triad)

  TD fixed point (linear):
    w_TD = A⁻¹ b
    where A = E[φ(s)(φ(s) - γφ(s'))^T]
          b = E[R × φ(s)]
    
    Approximation error bound:
    ||v̂_TD - v_π|| ≤ (1/√(1-γ²)) × min_w ||v̂(·,w) - v_π||
    
    At most 1/√(1-γ²) times worse than best possible linear approx
```

---

## Gradient TD Methods

```
True gradient methods that are stable even off-policy:

  GTD2 (Gradient TD):
    Minimizes projected Bellman error using true gradient
    
    Two sets of parameters: w (value), u (auxiliary)
    
    u ← u + β(δ - u^T φ) × φ
    w ← w + α(φ - γφ')(φ^T u)
    
    β = step size for u (typically larger than α)
    
    Convergent under all three deadly triad conditions!
    But: slower convergence, two parameter vectors

  TDC (TD with Correction):
    w ← w + α × δ × φ - α × γ × (φ^T u) × φ'
    u ← u + β × (δ - φ^T u) × φ
    
    The correction term -αγ(φ^T u)φ' fixes the semi-gradient bias
```

---

## Python: Semi-Gradient TD

```python
import numpy as np
import gymnasium as gym

class SemiGradientTD:
    def __init__(self, n_features, alpha=0.01, gamma=0.99):
        self.w = np.zeros(n_features)
        self.alpha = alpha
        self.gamma = gamma
    
    def value(self, phi):
        return np.dot(self.w, phi)
    
    def update(self, phi_s, reward, phi_s_next, done):
        target = reward if done else reward + self.gamma * self.value(phi_s_next)
        td_error = target - self.value(phi_s)
        self.w += self.alpha * td_error * phi_s
        return td_error

def tile_coding_features(state, n_tilings=8, n_tiles=8):
    """Simple tile coding for continuous states."""
    features = np.zeros(n_tilings * n_tiles * len(state))
    for tiling in range(n_tilings):
        offset = tiling / n_tilings
        for dim, s in enumerate(state):
            idx = int((s + offset) * n_tiles) % n_tiles
            features[tiling * n_tiles * len(state) + dim * n_tiles + idx] = 1.0
    return features

# Train on MountainCar
env = gym.make("MountainCar-v0")
n_features = 8 * 8 * 2
agent = SemiGradientTD(n_features, alpha=0.001)

for episode in range(500):
    state, _ = env.reset()
    phi = tile_coding_features(state)
    total_reward = 0
    
    for step in range(200):
        action = env.action_space.sample()
        next_state, reward, terminated, truncated, _ = env.step(action)
        done = terminated or truncated
        phi_next = tile_coding_features(next_state)
        
        agent.update(phi, reward, phi_next, done)
        phi = phi_next
        total_reward += reward
        
        if done:
            break
    
    if episode % 100 == 0:
        print(f"Episode {episode}, Total reward: {total_reward:.0f}")
```

---

## Summary

| Method | Gradient | Bootstraps | Convergence (Linear) | Off-Policy Safe |
|--------|:---:|:---:|:---:|:---:|
| MC | True | No | ✓ | ✓ (with IS) |
| Semi-grad TD | Semi | Yes | ✓ (on-policy) | ✗ |
| GTD2/TDC | True | Yes | ✓ | ✓ |
| Residual gradient | True | Yes | ✓ | ✓ (biased) |

---

## Revision Questions

1. **Why is the TD update called "semi-gradient"?**
2. **What is the Deadly Triad and which components cause divergence?**
3. **Why does MC give a true gradient but TD does not?**
4. **What is the TD fixed point and how good is its approximation?**
5. **How do Gradient TD methods (GTD2, TDC) fix the semi-gradient problem?**

---

[Previous: 04-gradient-descent-methods.md](04-gradient-descent-methods.md) | [Back to README](../README.md)
