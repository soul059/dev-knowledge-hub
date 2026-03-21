# Chapter 2: Linear Function Approximation

> **Unit 6: Function Approximation** — *The workhorse of convergent RL: linear value functions with guaranteed stability.*

---

## Overview

Linear function approximation is the most important special case of parametric value
estimation. It combines the expressiveness needed for continuous state spaces with
**convergence guarantees** no nonlinear approximator can match. The value function is a
weighted sum of hand-crafted features — the design of which is the central engineering
challenge. Every convergence proof and error bound is cleanest in the linear setting.

```
 ╔══════════════════════════════════════════════════════════════════════╗
 ║              LINEAR FUNCTION APPROXIMATION                          ║
 ╠══════════════════════════════════════════════════════════════════════╣
 ║                                                                     ║
 ║  V̂(s, w) = w₁φ₁(s) + w₂φ₂(s) + … + wₙφₙ(s) = wᵀφ(s)             ║
 ║                                                                     ║
 ║  • φ(s) : feature vector — fixed, hand-designed                     ║
 ║  • w    : weight vector — learned from experience                   ║
 ║  • ∇V̂   = φ(s)  — gradient equals the feature vector!              ║
 ║  • VE(w) is quadratic → single global minimum                      ║
 ║  • TD(0) converges to a fixed point within bounded error            ║
 ║  • Tile coding, RBFs, polynomials, Fourier basis — all linear       ║
 ║                                                                     ║
 ╚══════════════════════════════════════════════════════════════════════╝
```

---

## 1. The Linear Value Function

### 1.1 Definition

A **linear function approximator** represents the value as a dot product between a
learned weight vector $\mathbf{w} \in \mathbb{R}^d$ and a fixed feature vector
$\boldsymbol{\phi}(s) \in \mathbb{R}^d$:

$$\hat{V}(s, \mathbf{w}) = \mathbf{w}^\top \boldsymbol{\phi}(s) = \sum_{i=1}^{d} w_i \, \phi_i(s)$$

Each $\phi_i : S \to \mathbb{R}$ extracts a numerical property of the state.
The function is **linear in the weights**, not in the state — the features $\phi_i(s)$
can be arbitrarily nonlinear (polynomials, Gaussians, indicators):

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │            LINEARITY IN WEIGHTS — NOT IN STATE                      │
  │                                                                     │
  │  State s ─┬─→ φ₁(s) ──→ × w₁ ──┐                                  │
  │           ├─→ φ₂(s) ──→ × w₂ ──┤                                  │
  │           ├─→ φ₃(s) ──→ × w₃ ──┼──→ Σ ──→ V̂(s, w)                │
  │           ├─→  ⋮         ⋮      │                                  │
  │           └─→ φ_d(s) ─→ × w_d ─┘                                  │
  │                                                                     │
  │  φᵢ(s) can be ANY function of s (nonlinear is fine!)               │
  │  V̂(s, w) must be LINEAR in w  ←── this is the constraint          │
  └──────────────────────────────────────────────────────────────────────┘
```

### 1.3 The Gradient is Simply φ(s)

The gradient of $\hat{V}$ with respect to $\mathbf{w}$ is the feature vector itself:

$$\nabla_\mathbf{w} \hat{V}(s, \mathbf{w}) = \nabla_\mathbf{w} \left( \sum_{i=1}^{d} w_i \, \phi_i(s) \right) = \boldsymbol{\phi}(s)$$

No chain rule, no backpropagation — just multiply the error by the feature vector.

### 1.4 Tabular as a Special Case

With **one-hot features** $\phi_i(s) = \mathbf{1}[s = s_i]$:

$$\hat{V}(s_j, \mathbf{w}) = \sum_{i=1}^{|S|} w_i \cdot \mathbf{1}[s_j = s_i] = w_j$$

Each weight stores the value of one state. Linear FA with $d = |S|$ one-hot features
*is* the tabular method.

---

## 2. Feature Vectors — Design and Intuition

Features are **fixed functions** $\boldsymbol{\phi} : S \to \mathbb{R}^d$ that
transform raw state into a representation for linear combination. The approximator
can only represent functions in the **span** of the features:

$$\mathcal{V}_\phi = \left\{ s \mapsto \mathbf{w}^\top \boldsymbol{\phi}(s) : \mathbf{w} \in \mathbb{R}^d \right\}$$

If $V^\pi$ lies outside this span, even the best $\mathbf{w}$ has nonzero error.

| Property | Description |
|----------|-------------|
| **Relevance** | Features should correlate with the true value function |
| **Independence** | Avoid redundant, multicollinear features |
| **Coverage** | Distinguish states with different values |
| **Boundedness** | Controlled magnitudes for stable learning |
| **Sparsity** | Mostly zeros → fast, efficient updates |

---

## 3. Feature Construction Methods

### 3.1 Polynomial Features

For continuous state $s = (s^{(1)}, \dots, s^{(k)})$, form polynomial terms.

**Degree-1:** $\boldsymbol{\phi}(s) = (1, s^{(1)}, s^{(2)}, \dots, s^{(k)})^\top$ — gives $d = k + 1$ features.

**Degree-2 (with cross-terms):**
$\boldsymbol{\phi}(s) = (1, s^{(1)}, s^{(2)}, (s^{(1)})^2, s^{(1)}s^{(2)}, (s^{(2)})^2)^\top$

For state dimension $k$ and degree $n$, feature count is $d = \binom{k + n}{n}$:

| State dim $k$ | Degree $n$ | Features $d$ |
|:-:|:-:|:-:|
| 2 | 2 | 6 |
| 2 | 3 | 10 |
| 4 | 3 | 35 |
| 10 | 3 | 286 |

**Advantage:** Simple, captures smooth functions. **Disadvantage:** Explodes in high
dimensions; poor at local structure.

### 3.2 State Aggregation (Special Case)

Partition $S$ into groups $G_1, \dots, G_d$ with indicator features
$\phi_i(s) = \mathbf{1}[s \in G_i]$. Each state activates exactly **one** feature.
This is piecewise-constant approximation — the coarsest linear FA.

### 3.3 Tile Coding (Coarse Coding with Overlapping Tilings)

The most practically important feature construction for linear RL. It provides
**sparse binary features** enabling both generalisation and discrimination.

**How it works:** Define multiple **tilings** (grids covering the state space), each
offset differently. The feature vector has a 1 for each tile containing the state.

```
  ┌────────────────────────────────────────────────────────────────────┐
  │                    TILE CODING — 3 TILINGS                        │
  │                                                                    │
  │  Tiling 1                Tiling 2                Tiling 3          │
  │  ┌───┬───┬───┬───┐      ┌───┬───┬───┬───┐      ┌───┬───┬───┬───┐ │
  │  │   │   │   │   │      │   │   │   │   │      │   │   │   │   │ │
  │  ├───┼───┼───┼───┤      ├───┼───┼───┼───┤      ├───┼───┼───┼───┤ │
  │  │   │ ● │   │   │      │   │   │   │   │      │   │   │   │   │ │
  │  ├───┼───┼───┼───┤      ├───┼─●─┼───┼───┤      ├───┼───┼───┼───┤ │
  │  │   │   │   │   │      │   │   │   │   │      │   │ ● │   │   │ │
  │  ├───┼───┼───┼───┤      ├───┼───┼───┼───┤      ├───┼───┼───┼───┤ │
  │  │   │   │   │   │      │   │   │   │   │      │   │   │   │   │ │
  │  └───┴───┴───┴───┘      └───┴───┴───┴───┘      └───┴───┴───┴───┘ │
  │        ↑                       ↑                       ↑           │
  │     offset (0,0)         offset (Δx,Δy)        offset (2Δx,2Δy)  │
  │                                                                    │
  │  State ● activates one tile per tiling → exactly n_tilings 1s     │
  │  φ(s) = (0,0,0,0,0,1,0,...,0,1,0,...,0,1,0,...,0)                 │
  │  Total features d = n_tilings × tiles_per_tiling                   │
  └────────────────────────────────────────────────────────────────────┘
```

| Property | Value |
|----------|-------|
| Features active per state | Exactly $n_\text{tilings}$ |
| Feature type | Binary (0 or 1) |
| Total features | $d = n_\text{tilings} \times \text{tiles per tiling}$ |
| Generalisation width | Controlled by tile size |
| Step size rule | $\alpha = \alpha_0 / n_\text{tilings}$ |

**Why multiple tilings?** A single tiling = state aggregation (identical values within
tile). Multiple offset tilings create intersecting receptive fields: nearby states
share many tiles → similar values; distant states share few → independent values.

### 3.4 Radial Basis Functions (RBFs)

RBFs produce **continuous-valued** features responding most strongly near a centre
$\mathbf{c}_i$:

$$\phi_i(s) = \exp\!\left( -\frac{\| s - \mathbf{c}_i \|^2}{2\sigma_i^2} \right)$$

where $\mathbf{c}_i$ is the centre and $\sigma_i$ controls the width.

```
  ┌──────────────────────────────────────────────────────────────────┐
  │              RADIAL BASIS FUNCTIONS (1D Example)                 │
  │                                                                  │
  │  φ(s)                                                            │
  │   1 ┤          ╱╲            ╱╲            ╱╲                    │
  │     │         ╱  ╲          ╱  ╲          ╱  ╲                   │
  │     │        ╱    ╲        ╱    ╲        ╱    ╲                  │
  │     │       ╱      ╲      ╱      ╲      ╱      ╲               │
  │     │      ╱        ╲   ╱        ╲   ╱        ╲                │
  │     │     ╱          ╲╱           ╲╱           ╲               │
  │   0 ┤────╱────────────────────────────────────────╲───→ s       │
  │              c₁           c₂           c₃                       │
  │                                                                  │
  │  Each RBF is a Gaussian centred at cᵢ with width σᵢ            │
  │  V̂(s, w) = w₁φ₁(s) + w₂φ₂(s) + w₃φ₃(s) + …                   │
  └──────────────────────────────────────────────────────────────────┘
```

**Properties:** Smooth continuous features; many nonzero activations → dense, slower
updates; width $\sigma_i$ controls generalisation breadth.

### 3.5 Fourier Basis

For state $s \in [0, 1]^k$, Fourier features are
$\phi_i(s) = \cos(\pi \, \mathbf{c}_i^\top s)$ where $\mathbf{c}_i$ is an integer
frequency vector. Order-$n$ uses all entries in $\{0, \dots, n\}$, giving $(n+1)^k$
features. Per-feature step sizes $\alpha_i = \alpha / \|\mathbf{c}_i\|$ improve
stability. Disadvantage: exponential in state dimension.

### 3.6 Comparison

| Method | Type | Sparsity | Generalisation | Best For |
|--------|:----:|:--------:|:--------------:|----------|
| Polynomial | Continuous | Dense | Global | Low-dim smooth problems |
| State Aggregation | Binary | 1-hot | Within group | Very simple problems |
| Tile Coding | Binary | $n$-hot | Local | Most RL problems |
| RBF | Continuous | Soft-sparse | Local (smooth) | Smooth value functions |
| Fourier | Continuous | Dense | Global | Smooth, periodic structure |

---

## 4. The Gradient Descent Update

### 4.1 SGD Update — Linear Specialisation

Since $\nabla_\mathbf{w} \hat{V} = \boldsymbol{\phi}(s)$, the general SGD update
$\mathbf{w} \leftarrow \mathbf{w} + \alpha[v_t - \hat{V}]\nabla\hat{V}$ simplifies to:

$$\boxed{\mathbf{w}_{t+1} = \mathbf{w}_t + \alpha \left[ v_t - \mathbf{w}_t^\top \boldsymbol{\phi}(s_t) \right] \boldsymbol{\phi}(s_t)}$$

Per-weight: $w_{i,t+1} = w_{i,t} + \alpha [v_t - \hat{V}(s_t, \mathbf{w}_t)] \phi_i(s_t)$.
Only weights with **nonzero features** change — with tile coding, cost is
$O(n_\text{tilings})$ not $O(d)$.

### 4.2 Different Targets

| Algorithm | Target $v_t$ | True Gradient? |
|-----------|:------------|:-:|
| Monte Carlo | $G_t$ (full return) | ✅ Yes |
| Semi-gradient TD(0) | $R_{t+1} + \gamma \hat{V}(S_{t+1}, \mathbf{w}_t)$ | ❌ Semi-gradient |
| Semi-gradient TD(λ) | Uses eligibility traces | ❌ Semi-gradient |

With MC targets, the update is true gradient descent. With TD targets it is
**semi-gradient** (target depends on $\mathbf{w}$) — but linear TD still converges.

---

## 5. The Value Error Surface

### 5.1 Quadratic Error Surface

$$\overline{\text{VE}}(\mathbf{w}) = \sum_{s \in S} \mu(s) \left[ V^\pi(s) - \mathbf{w}^\top \boldsymbol{\phi}(s) \right]^2 = \mathbf{w}^\top \mathbf{A} \mathbf{w} - 2\mathbf{b}^\top \mathbf{w} + c$$

where $\mathbf{A} = \sum_s \mu(s) \boldsymbol{\phi}(s)\boldsymbol{\phi}(s)^\top$ (PSD),
$\mathbf{b} = \sum_s \mu(s) V^\pi(s) \boldsymbol{\phi}(s)$, $c = \sum_s \mu(s) [V^\pi(s)]^2$.

### 5.2 Single Global Minimum

Because $\mathbf{A}$ is PSD, the surface is a **bowl** — one global minimum, **no
local minima**: $\mathbf{w}^* = \mathbf{A}^{-1} \mathbf{b}$

```
  ┌──────────────────────────────────────────────────────────────────┐
  │          VE(w) SURFACE — LINEAR vs NONLINEAR                    │
  │                                                                  │
  │  LINEAR (quadratic bowl):       NONLINEAR (complex surface):    │
  │  VE ▲                           VE ▲                            │
  │     │    ╲      ╱                   │  ╱╲    ╱╲                  │
  │     │     ╲    ╱                    │ ╱  ╲  ╱  ╲    ╱╲          │
  │     │      ╲  ╱                    │╱    ╲╱    ╲  ╱  ╲         │
  │     │       ╲╱                     │           ╲╱    ╲        │
  │     │       w*                     │     local    global       │
  │     │   (global min)               │     min      min          │
  │     └──────────→ w                 └──────────────────→ w      │
  │                                                                  │
  │  ONE minimum → guaranteed          MANY minima → SGD may        │
  │  convergence to optimum            get stuck in local min       │
  └──────────────────────────────────────────────────────────────────┘
```

---

## 6. Convergence Guarantees

### 6.1 Monte Carlo with Linear FA

With MC targets and step-size conditions $\sum \alpha_t = \infty$,
$\sum \alpha_t^2 < \infty$, linear MC converges to the **global minimum**
$\mathbf{w}^* = \arg\min_\mathbf{w} \overline{\text{VE}}(\mathbf{w})$.

### 6.2 TD(0) — The TD Fixed Point

Linear TD(0) converges to $\mathbf{w}_\text{TD}$ satisfying
$\mathbf{A}_\text{TD} \mathbf{w}_\text{TD} = \mathbf{b}_\text{TD}$, where:

$$\mathbf{A}_\text{TD} = \mathbb{E}\!\left[ \boldsymbol{\phi}(S_t) \left( \boldsymbol{\phi}(S_t) - \gamma \boldsymbol{\phi}(S_{t+1}) \right)^\top \right], \quad \mathbf{b}_\text{TD} = \mathbb{E}\!\left[ R_{t+1} \boldsymbol{\phi}(S_t) \right]$$

### 6.3 The Error Bound

$$\boxed{\overline{\text{VE}}(\mathbf{w}_\text{TD}) \;\leq\; \frac{1}{1 - \gamma} \min_\mathbf{w} \overline{\text{VE}}(\mathbf{w})}$$

**Interpretation:** TD's error is at most $\frac{1}{1-\gamma}$ times worse than the
best achievable (MC solution). The bound is often loose — TD typically performs much
closer to MC.

| $\gamma$ | Bound $\frac{1}{1-\gamma}$ | Meaning |
|:-:|:-:|:--|
| 0.0 | 1 | TD = MC |
| 0.9 | 10 | TD ≤ 10× MC |
| 0.99 | 100 | TD ≤ 100× MC |

Despite this, TD is preferred: lower variance, faster online learning, bound is worst-case.

### 6.4 Convergence Summary

```
Algorithm               Converges?     Converges To
─────────────────────────────────────────────────────────────────
MC + Linear FA          ✅ Yes         w*  (global minimum of VE)
TD(0) + Linear FA       ✅ Yes         w_TD (TD fixed point)
TD(λ) + Linear FA       ✅ Yes         Between w_TD and w*
TD(0) + Nonlinear FA    ❌ May diverge —
TD(0) + Off-policy      ❌ May diverge —
─────────────────────────────────────────────────────────────────
```

---

## 7. Tile Coding — Detailed Treatment

### 7.1 Multiple Tilings with Offsets

```
  ┌────────────────────────────────────────────────────────────────────────┐
  │           TILE CODING — 2D EXAMPLE (2 Tilings)                       │
  │                                                                       │
  │  Tiling 1 (no offset):              Tiling 2 (offset by half tile):  │
  │                                                                       │
  │   velocity                            velocity                        │
  │     ▲                                   ▲                             │
  │     │ ┌─────┬─────┬─────┐               │   ┌─────┬─────┬─────┐     │
  │     │ │  A  │  B  │  C  │               │   │  P  │  Q  │  R  │     │
  │     │ │     │  ●  │     │               │   │     │     │     │     │
  │     │ ├─────┼─────┼─────┤               │ ┌─┼─────┼─────┼─────┼─┐   │
  │     │ │  D  │  E  │  F  │               │ │ │  S  │  T● │  U  │ │   │
  │     │ ├─────┼─────┼─────┤               │ ├─┼─────┼─────┼─────┼─┤   │
  │     │ │  G  │  H  │  I  │               │ │ │  V  │  W  │  X  │ │   │
  │     │ └─────┴─────┴─────┘               │ └─┼─────┼─────┼─────┼─┘   │
  │     └──────────────────→ pos            │   └─────┴─────┴─────┘     │
  │                                         └──────────────────→ pos     │
  │                                                                       │
  │  State ● in tile B (T1) and tile T (T2): V̂(●) = w_B + w_T           │
  │  Nearby state ○ in tile B (T1) and S (T2): V̂(○) = w_B + w_S         │
  │  They share w_B → partial generalisation                             │
  └────────────────────────────────────────────────────────────────────────┘
```

### 7.2 Generalisation and Discrimination

With $n$ tilings, similarity between states equals the fraction of shared tiles:

$$\text{similarity}(s_1, s_2) = \frac{\text{shared active tiles}}{n_\text{tilings}}$$

### 7.3 Step Size and Hashing

Use $\alpha = \alpha_0 / n_\text{tilings}$ so the effective learning rate is $\alpha_0$.
For high-dimensional spaces, **hashing** maps tile indices to a fixed-size weight array,
trading minor aliasing for bounded memory.

---

## 8. Algorithms — Pseudocode

```
Algorithm: Semi-Gradient TD(0) with Linear FA
─────────────────────────────────────────────────────────────────
Input:  policy π, features φ: S → ℝᵈ, step size α, discount γ
Initialise: w = 0 ∈ ℝᵈ

Repeat (for each episode):
│  Initialise S
│  Repeat (for each step):
│  │  A ~ π(·|S), take A, observe R, S'
│  │  If S' is terminal:
│  │  │  w ← w + α [R - wᵀφ(S)] φ(S);  go to next episode
│  │  Else:
│  │  │  w ← w + α [R + γ wᵀφ(S') - wᵀφ(S)] φ(S);  S ← S'
```

For **Gradient MC**, replace the target: use the full return $G_t = \sum_k \gamma^k R_{t+k+1}$
instead of the bootstrapped TD target. MC is a true gradient method; TD is semi-gradient.

---

## 9. Python Implementation — Tile Coding for Mountain Car

```python
import numpy as np

class TileCoder:
    """Tile coding for continuous state spaces."""

    def __init__(self, n_tilings, tiles_per_dim, state_low, state_high):
        self.n_tilings = n_tilings
        self.tiles_per_dim = np.array(tiles_per_dim)
        self.state_low = np.array(state_low, dtype=np.float64)
        self.state_range = np.array(state_high) - self.state_low
        self.tiles_per_tiling = int(np.prod(self.tiles_per_dim))
        self.n_features = self.n_tilings * self.tiles_per_tiling

        # Asymmetric offsets for each tiling
        self.offsets = np.zeros((n_tilings, len(tiles_per_dim)))
        for i in range(n_tilings):
            for d in range(len(tiles_per_dim)):
                self.offsets[i, d] = (i / n_tilings) * (
                    self.state_range[d] / self.tiles_per_dim[d]
                )

    def get_active_tiles(self, state):
        """Return indices of active tiles for a given state."""
        state = np.asarray(state, dtype=np.float64)
        active = []
        for t in range(self.n_tilings):
            shifted = state - self.state_low + self.offsets[t]
            scaled = shifted / self.state_range * self.tiles_per_dim
            coords = np.clip(scaled.astype(int), 0, self.tiles_per_dim - 1)
            flat = t * self.tiles_per_tiling
            mult = 1
            for d in reversed(range(len(self.tiles_per_dim))):
                flat += coords[d] * mult
                mult *= self.tiles_per_dim[d]
            active.append(flat)
        return active


class SemiGradientTD:
    """Semi-gradient TD(0) with tile-coded linear FA."""

    def __init__(self, tile_coder, alpha_0=0.1, gamma=1.0):
        self.tc = tile_coder
        self.alpha = alpha_0 / tile_coder.n_tilings
        self.gamma = gamma
        self.w = np.zeros(tile_coder.n_features)

    def value(self, state):
        return sum(self.w[i] for i in self.tc.get_active_tiles(state))

    def update(self, state, reward, next_state, done):
        active = self.tc.get_active_tiles(state)
        v_s = sum(self.w[i] for i in active)
        target = reward if done else reward + self.gamma * self.value(next_state)
        td_error = target - v_s
        for i in active:
            self.w[i] += self.alpha * td_error


def run_mountain_car(n_episodes=500):
    """Train on Mountain Car with semi-gradient TD(0) + tile coding."""
    POS, VEL = (-1.2, 0.6), (-0.07, 0.07)
    tc = TileCoder(8, (8, 8), [POS[0], VEL[0]], [POS[1], VEL[1]])
    agent = SemiGradientTD(tc, alpha_0=0.1, gamma=1.0)

    for ep in range(n_episodes):
        pos, vel, steps = np.random.uniform(-0.6, -0.4), 0.0, 0
        while steps < 5000:
            action = np.random.choice([-1, 0, 1])
            nv = np.clip(vel + 0.001*action - 0.0025*np.cos(3*pos), *VEL)
            np_ = np.clip(pos + nv, *POS)
            if np_ == POS[0]: nv = 0.0
            done = np_ >= POS[1]; steps += 1
            agent.update([pos, vel], -1.0, [np_, nv], done)
            if done: break
            pos, vel = np_, nv
        if (ep+1) % 100 == 0:
            print(f"Ep {ep+1:4d} | Steps: {steps:4d} | "
                  f"V(bottom): {agent.value([-0.5, 0]):7.1f}")

if __name__ == "__main__":
    run_mountain_car()
```

**Walkthrough:** 8 tilings × 64 tiles = $d = 512$ features. Each state activates 8
tiles. Updates are $O(8)$. Step size $\alpha = 0.1/8 = 0.0125$.

---

## 10. Summary

```
╔═══════════════════════════════════════════════════════════════════════╗
║  CHAPTER 2 — KEY TAKEAWAYS                                          ║
╠═══════════════════════════════════════════════════════════════════════╣
║  1. Linear FA: V̂(s, w) = wᵀφ(s) — linear in weights, not state     ║
║  2. Gradient ∇V̂ = φ(s) — simplifies all update rules                ║
║  3. VE(w) is quadratic → single global minimum (no local minima)    ║
║  4. MC + linear FA → converges to global optimum w*                 ║
║  5. TD + linear FA → converges to TD fixed point w_TD               ║
║  6. Bound: VE(w_TD) ≤ 1/(1-γ) · min VE(w)                         ║
║  7. Tile coding: sparse binary features, efficient, practical       ║
║  8. RBFs: smooth continuous features, Gaussian bumps                ║
║  9. Feature design is THE critical decision in linear FA             ║
╚═══════════════════════════════════════════════════════════════════════╝
```

| Concept | Key Point |
|---------|-----------|
| Linear value function | $\hat{V}(s, \mathbf{w}) = \mathbf{w}^\top \boldsymbol{\phi}(s) = \sum_i w_i \phi_i(s)$ |
| Gradient property | $\nabla_\mathbf{w} \hat{V} = \boldsymbol{\phi}(s)$ — no backprop needed |
| SGD update (linear) | $\mathbf{w} \leftarrow \mathbf{w} + \alpha [v - \mathbf{w}^\top\boldsymbol{\phi}(s)] \boldsymbol{\phi}(s)$ |
| VE surface | Quadratic → single global minimum $\mathbf{w}^* = \mathbf{A}^{-1}\mathbf{b}$ |
| TD error bound | $\overline{\text{VE}}(\mathbf{w}_\text{TD}) \leq \frac{1}{1-\gamma} \min_\mathbf{w} \overline{\text{VE}}(\mathbf{w})$ |
| Tile coding | Sparse binary; $n_\text{tilings}$ active; $\alpha = \alpha_0 / n_\text{tilings}$ |
| RBF features | $\phi_i(s) = \exp(-\|s - c_i\|^2 / 2\sigma_i^2)$; smooth, local |
| Fourier basis | $\phi_i(s) = \cos(\pi \mathbf{c}_i^\top s)$; $(n+1)^k$ features |
| Tabular = special case | One-hot features: $d = |S|$, $w_i = V[s_i]$ |

---

## Revision Questions

1. **Derive the gradient of the linear value function.** Starting from
   $\hat{V}(s, \mathbf{w}) = \sum_i w_i \phi_i(s)$, show that
   $\nabla_\mathbf{w} \hat{V} = \boldsymbol{\phi}(s)$. Why does this make linear FA
   computationally attractive compared to neural networks?

2. **Explain why the VE surface has a single global minimum for linear FA.** Write out
   the quadratic form and explain the role of the matrix $\mathbf{A}$.

3. **State and interpret the TD fixed-point error bound.** For $\gamma = 0.95$, what
   does the bound say? Why might TD still be preferred over MC despite potentially
   higher asymptotic error?

4. **Design a tile coding scheme for Cart-Pole** (4D state: position, velocity, angle,
   angular velocity). How many tilings, tile dimensions, total features, and step size?

5. **Compare tile coding and RBFs.** Under what conditions would you prefer RBFs?
   What are the computational trade-offs (sparsity, update cost, memory)?

6. **Show that state aggregation is a special case of linear FA with one tiling.**
   What happens to generalisation? Why do we need multiple tilings?

---

## Navigation

| ← Previous | ↑ Up | Next → |
|:-----------:|:----:|:------:|
| [Why Function Approximation](./01-why-function-approximation.md) | [Unit 6: Function Approximation](./README.md) | [Neural Network Approximation](./03-neural-network-approximation.md) |
