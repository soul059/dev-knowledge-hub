# Trust Region Policy Optimization (TRPO)

## Overview

**TRPO** constrains policy updates so the new policy doesn't deviate too far from the old one, measured by KL divergence. Standard policy gradients are sensitive to step size — too large a step can catastrophically collapse performance. TRPO guarantees **monotonic improvement** by solving a constrained optimization problem at each update.

---

## The Problem with Vanilla Policy Gradient

```
Vanilla update: θ ← θ + α∇J(θ)

  Problem: policy is VERY sensitive to parameter changes!
  
  Small Δθ → policy barely changes → slow learning
  Large Δθ → policy collapses → catastrophic forgetting
  
  ┌───────────────────────────────────────────────┐
  │  Performance                                    │
  │  ▲                                              │
  │  │      ╱\                                      │
  │  │     ╱  \        step too large               │
  │  │    ╱    \       → collapse!                  │
  │  │   ╱      \                                   │
  │  │  ╱        \___                               │
  │  │ ╱                                            │
  │  │╱                                             │
  │  └──────────────────────────────► updates       │
  │                                                  │
  │  We need to control HOW MUCH the policy changes │
  └───────────────────────────────────────────────┘
```

---

## Trust Region Concept

```
Instead of constraining step size in PARAMETER space (Δθ),
constrain step size in POLICY space (how much π changes):

  max_θ  L(θ)                         (surrogate objective)
  s.t.   KL(π_old || π_new) ≤ δ       (trust region constraint)

  KL divergence measures how different two distributions are:
    KL(π_old || π_new) = E_s[ Σ_a π_old(a|s) log(π_old(a|s)/π_new(a|s)) ]

  ┌─────────────────────────────────────────────────┐
  │                                                   │
  │     Policy space                                  │
  │                                                   │
  │           ╭─────────╮                             │
  │          ╱  Trust    ╲    δ = KL constraint       │
  │         │  Region    │                            │
  │         │    ● π_old │                            │
  │         │       ↗    │                            │
  │          ╲  π_new  ╱                              │
  │           ╰───────╯                               │
  │                                                   │
  │  New policy must stay inside the trust region     │
  └─────────────────────────────────────────────────┘
```

---

## Surrogate Objective

```
TRPO optimizes a surrogate loss:

  L(θ) = E_t[ (π_θ(a_t|s_t) / π_θ_old(a_t|s_t)) × Â_t ]
       = E_t[ r_t(θ) × Â_t ]

  where r_t(θ) = π_new(a|s) / π_old(a|s)  (importance ratio)

  This is the expected advantage under the NEW policy,
  estimated using data from the OLD policy (importance sampling).

  If Â_t > 0 (good action):
    Want r_t > 1 → increase probability
    
  If Â_t < 0 (bad action):
    Want r_t < 1 → decrease probability

  But without constraint, r_t can become very large → instability
  TRPO constrains: KL(π_old || π_new) ≤ δ
```

---

## TRPO Algorithm

```
For each iteration:
  1. Collect trajectories with current policy π_old
  2. Compute advantages Â_t (typically using GAE)
  3. Compute surrogate objective gradient: g = ∇_θ L(θ)
  4. Compute Fisher Information Matrix: F = ∇²_θ KL(π_old || π_θ)
  5. Solve: F × step = g  (conjugate gradient method)
  6. Line search: find largest step within trust region
  7. Update: θ ← θ_old + α × step

  Fisher Information Matrix (FIM):
    F = E[∇log π × (∇log π)ᵀ]
    
    Measures curvature of policy space
    Used for natural gradient: F⁻¹g
    
  Conjugate Gradient (CG):
    Solves F × x = g WITHOUT computing F explicitly
    Only needs F × v products (Hessian-vector products)
    Typically 10 CG iterations suffice

  Line search with backtracking:
    Start with full step, halve until:
    - KL constraint satisfied: KL ≤ δ
    - Surrogate improves: L(θ_new) > L(θ_old)
```

---

## Natural Gradient

```
Standard gradient:     Δθ = α × ∇_θ J
Natural gradient:      Δθ = α × F⁻¹ × ∇_θ J

  Standard gradient is optimal in PARAMETER space (Euclidean)
  Natural gradient is optimal in DISTRIBUTION space (KL)

  Why it matters:
    Two parameters θ₁, θ₂ might:
    - Differ greatly in value (large Euclidean distance)
    - But produce nearly identical policies (small KL)
    
    Or:
    - Differ slightly in value (small Euclidean)
    - But produce very different policies (large KL)
    
    Natural gradient accounts for this → better updates!

  TRPO ≈ Natural gradient with trust region constraint
```

---

## Monotonic Improvement Guarantee

```
TRPO provides a theoretical guarantee:

  J(π_new) ≥ L(π_new) - C × max_s KL(π_old(·|s) || π_new(·|s))
  
  where C = 2εγ / (1-γ)²
  
  By keeping KL small (trust region), the penalty term is small,
  and since L approximates J at π_old, improvement is guaranteed.
  
  This is why TRPO is so stable:
  - Every update is guaranteed to improve (in theory)
  - No more catastrophic collapses
  - But: expensive due to conjugate gradient + line search
```

---

## Python: Simplified TRPO Concepts

```python
import torch
import torch.nn as nn

def conjugate_gradient(Fvp_fn, b, n_iters=10, tol=1e-10):
    """Solve Fx = b using conjugate gradient (without forming F)."""
    x = torch.zeros_like(b)
    r = b.clone()
    p = b.clone()
    rdotr = r.dot(r)
    
    for _ in range(n_iters):
        Fp = Fvp_fn(p)
        alpha = rdotr / (p.dot(Fp) + 1e-8)
        x += alpha * p
        r -= alpha * Fp
        new_rdotr = r.dot(r)
        if new_rdotr < tol:
            break
        p = r + (new_rdotr / rdotr) * p
        rdotr = new_rdotr
    return x

def fisher_vector_product(policy, states, v, damping=0.1):
    """Compute F @ v without forming the full Fisher matrix."""
    dist = policy(states)
    kl = torch.distributions.kl_divergence(dist, dist).mean()
    
    grads = torch.autograd.grad(kl, policy.parameters(),
                                create_graph=True)
    flat_grads = torch.cat([g.view(-1) for g in grads])
    
    gvp = flat_grads.dot(v)
    fvp = torch.autograd.grad(gvp, policy.parameters())
    flat_fvp = torch.cat([g.contiguous().view(-1) for g in fvp])
    
    return flat_fvp + damping * v

def trpo_step(policy, states, actions, advantages, max_kl=0.01):
    """One TRPO update step."""
    # 1. Compute policy gradient
    dist = policy(states)
    log_probs = dist.log_prob(actions)
    loss = -(log_probs * advantages).mean()
    
    grads = torch.autograd.grad(loss, policy.parameters())
    flat_grad = torch.cat([g.view(-1) for g in grads])
    
    # 2. Conjugate gradient to find natural gradient direction
    Fvp = lambda v: fisher_vector_product(policy, states, v)
    step_dir = conjugate_gradient(Fvp, flat_grad)
    
    # 3. Compute step size
    sFs = step_dir.dot(Fvp(step_dir))
    step_size = torch.sqrt(2 * max_kl / (sFs + 1e-8))
    
    # 4. Line search (simplified)
    full_step = step_size * step_dir
    
    return full_step  # Apply to parameters with backtracking
```

---

## TRPO vs Other Methods

```
  Method      Constraint     Computation     Stability
  ────────    ──────────     ───────────     ─────────
  Vanilla PG  None           O(N)            Low
  TRPO        KL ≤ δ         O(N²) CG       High
  PPO         Clip ratio     O(N)            High
  
  TRPO drawbacks:
  - Complex implementation (CG, FVP, line search)
  - Expensive per iteration
  - Hard to use with shared actor-critic networks
  
  → PPO was designed to approximate TRPO more simply
```

---

## Revision Questions

1. **What problem does TRPO solve that vanilla policy gradient has?**
2. **What is the trust region constraint and why use KL divergence?**
3. **What is the surrogate objective and what does the importance ratio represent?**
4. **Why is the natural gradient better than the standard gradient for policy optimization?**
5. **What is the monotonic improvement guarantee of TRPO?**

---

[Next: 02-ppo.md](02-ppo.md) | [Back to README](../README.md)
