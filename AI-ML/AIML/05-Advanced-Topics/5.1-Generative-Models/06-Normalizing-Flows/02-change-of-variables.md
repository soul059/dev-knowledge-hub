# Change of Variables

## Overview

The **change of variables** formula is the mathematical foundation of normalizing flows. It describes how a probability density transforms under an invertible mapping. When we apply a function f to transform a random variable z into x = f(z), the density of x depends on both the density of z and how much f "stretches" or "compresses" space — measured by the **Jacobian determinant**.

---

## One-Dimensional Case

```
If z ~ p_z(z) and x = f(z), then:

  p_x(x) = p_z(f⁻¹(x)) × |df⁻¹/dx|

  Example: x = 2z + 3  (stretch by 2, shift by 3)
  
    z = (x - 3) / 2
    dz/dx = 1/2
    
    p_x(x) = p_z((x-3)/2) × |1/2|
    
  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  z ~ N(0,1):          x = 2z + 3:                │
  │                                                    │
  │       ╱╲                    ╱╲                    │
  │      ╱  ╲                  ╱  ╲                   │
  │     ╱    ╲               ╱    ╲                   │
  │    ╱      ╲            ╱        ╲                 │
  │  ──────────────      ────────────────             │
  │   -2  0  2           -1  1  3  5  7               │
  │                                                    │
  │  Stretching by 2 → density halved (area = 1)     │
  │  The |1/2| factor accounts for this!             │
  └──────────────────────────────────────────────────┘

  Intuition: if f stretches space by factor k,
  density must shrink by factor 1/k to preserve total probability = 1
```

---

## Multi-Dimensional Case

```
If z ∈ ℝⁿ and x = f(z), the stretching factor becomes
the ABSOLUTE VALUE of the DETERMINANT of the JACOBIAN:

  p_x(x) = p_z(f⁻¹(x)) × |det J_{f⁻¹}(x)|

  Jacobian matrix J of f⁻¹:

  J = ┌ ∂z₁/∂x₁  ∂z₁/∂x₂  ...  ∂z₁/∂xₙ ┐
      │ ∂z₂/∂x₁  ∂z₂/∂x₂  ...  ∂z₂/∂xₙ │
      │    ⋮         ⋮      ⋱      ⋮      │
      └ ∂zₙ/∂x₁  ∂zₙ/∂x₂  ...  ∂zₙ/∂xₙ ┘

  det(J) = how much the transformation changes volume

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  2D Example:                                      │
  │                                                    │
  │  Original     After f      det(J) = ?            │
  │  ┌──┐        ┌────────┐                          │
  │  │  │   f→   │        │   det(J) = 4             │
  │  │  │        │        │   Volume grew by 4×      │
  │  └──┘        └────────┘   Density shrinks by 4×  │
  │  area=1      area=4                               │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## Log-Likelihood Formulation

```
In practice, work with log-probabilities:

  log p(x) = log p_z(f⁻¹(x)) + log |det J_{f⁻¹}(x)|

For composed flows f = f_K ∘ ... ∘ f_1:

  log p(x) = log p_z(z₀) + Σᵢ log |det J_{f_i⁻¹}|

  where z₀ = f₁⁻¹(f₂⁻¹(...f_K⁻¹(x)))

  Training objective:
    maximize  E_data[log p(x)]
  = maximize  E_data[log p_z(f⁻¹(x)) + log |det J|]

  ┌──────────────────────────────────────────────────┐
  │                                                    │
  │  Two terms to compute:                           │
  │                                                    │
  │  1. log p_z(z):  Easy! z is Gaussian            │
  │     = -½ ||z||² - (d/2) log(2π)                 │
  │                                                    │
  │  2. log |det J|:  This is the HARD part!         │
  │     General: O(n³) to compute determinant        │
  │     Flow design: make this O(n) or O(n log n)    │
  │                                                    │
  └──────────────────────────────────────────────────┘
```

---

## The Jacobian Determinant Challenge

```
For n-dimensional data:
  Full Jacobian: n × n matrix
  Determinant: O(n³) computation

  For images (64×64×3 = 12288 dimensions):
  Jacobian: 12288 × 12288 matrix → impossible!

  Solution: Design f so Jacobian is TRIANGULAR:

  Triangular Jacobian:
  J = ┌ a₁₁  0    0   ...  0   ┐
      │ a₂₁  a₂₂  0   ...  0   │
      │ a₃₁  a₃₂  a₃₃ ...  0   │
      │  ⋮    ⋮    ⋮   ⋱   ⋮   │
      └ aₙ₁  aₙ₂  aₙ₃ ... aₙₙ ┘

  det(J) = a₁₁ × a₂₂ × a₃₃ × ... × aₙₙ  (product of diagonal!)
  
  O(n) computation!  This is the key insight for efficient flows.

  Flow architectures are designed specifically to produce
  triangular (or block-triangular) Jacobians.
```

---

## Efficient Jacobian Structures

```
Design Pattern             Jacobian Structure      Cost
───────────────            ──────────────────      ────
Coupling layers (RealNVP)  Lower triangular        O(n)
Autoregressive (MAF/IAF)   Triangular              O(n)
1×1 Convolutions (Glow)    Dense but small (c×c)   O(c³)
Residual flows             Approximate (trace)     O(n)
LU decomposition           L × U                   O(n)

  Coupling layer Jacobian (RealNVP):

  x₁ = z₁                      (identity, no transform)
  x₂ = z₂ × exp(s(z₁)) + t(z₁) (affine, conditioned on z₁)

  J = ┌  I       0            ┐
      └ ∂x₂/∂z₁  diag(exp(s)) ┘

  det(J) = exp(Σ s_i)  ← just sum of s values!
```

---

## Python: Change of Variables in Practice

```python
import torch
import torch.nn as nn
import math

class PlanarFlow(nn.Module):
    """Planar flow: f(z) = z + u × h(wᵀz + b)."""
    def __init__(self, dim):
        super().__init__()
        self.w = nn.Parameter(torch.randn(1, dim))
        self.u = nn.Parameter(torch.randn(1, dim))
        self.b = nn.Parameter(torch.zeros(1))
    
    def forward(self, z):
        linear = (z @ self.w.t()) + self.b         # (batch, 1)
        h = torch.tanh(linear)                      # (batch, 1)
        x = z + self.u * h                          # (batch, dim)
        
        # Log determinant using matrix determinant lemma
        h_prime = 1 - h ** 2                        # tanh derivative
        psi = h_prime @ self.w                      # (batch, dim)
        det = 1 + (psi @ self.u.t()).squeeze()      # (batch,)
        log_det = torch.log(torch.abs(det) + 1e-8)
        return x, log_det

class NormalizingFlow(nn.Module):
    def __init__(self, dim, n_flows=10):
        super().__init__()
        self.flows = nn.ModuleList([PlanarFlow(dim) for _ in n_flows])
        self.dim = dim
    
    def log_prob(self, x):
        log_det_sum = 0
        z = x
        for flow in reversed(self.flows):
            z, log_det = flow(z)
            log_det_sum += log_det
        
        # Base distribution log prob (standard Gaussian)
        log_pz = -0.5 * (z ** 2 + math.log(2 * math.pi)).sum(dim=-1)
        return log_pz + log_det_sum
    
    def sample(self, n_samples):
        z = torch.randn(n_samples, self.dim)
        x = z
        for flow in self.flows:
            x, _ = flow(x)
        return x

# Train on 2D data
flow = NormalizingFlow(dim=2, n_flows=16)
optimizer = torch.optim.Adam(flow.parameters(), lr=1e-3)

for step in range(5000):
    x = sample_2d_data(batch_size=512)
    loss = -flow.log_prob(x).mean()
    optimizer.zero_grad(); loss.backward(); optimizer.step()
```

---

## Revision Questions

1. **What does the Jacobian determinant measure in the context of density transformation?**
2. **Why does stretching space by factor k reduce density by 1/k?**
3. **Why is computing the full Jacobian determinant infeasible for high-dimensional data?**
4. **How does a triangular Jacobian make determinant computation efficient?**
5. **What is the log-likelihood training objective for normalizing flows?**

---

[Previous: 01-flow-based-models-concept.md](01-flow-based-models-concept.md) | [Next: 03-invertible-transformations.md](03-invertible-transformations.md)
