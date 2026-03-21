# Chapter 4: Gradient Descent Methods

> **Unit 6: Function Approximation** — *Optimizing value function parameters through gradient-based learning.*

---

## Overview

Every function approximation method in RL ultimately needs a way to adjust its
parameters so that predictions become more accurate. **Gradient descent** provides
this mechanism: we define a loss function that measures prediction error, compute its
gradient with respect to the weight vector, and step in the direction that reduces the
error. The critical complication in RL is that the **true target is unknown** — we
never observe $v_\pi(s)$ directly — so we must substitute estimates (Monte Carlo
returns, TD targets), each introducing its own bias–variance tradeoff and altering the
convergence properties of the algorithm.

```
 ╔══════════════════════════════════════════════════════════════════════╗
 ║              GRADIENT DESCENT METHODS FOR RL                        ║
 ╠══════════════════════════════════════════════════════════════════════╣
 ║                                                                     ║
 ║  Objective: minimise  VE(w) = E_μ[(v_π(S) − V̂(S,w))²]             ║
 ║                                                                     ║
 ║  • True gradient:  ∇VE  requires v_π(S) — unknown in practice      ║
 ║  • MC substitute:  G_t  → unbiased, high variance, true gradient   ║
 ║  • TD substitute:  R + γV̂ → biased, low variance, semi-gradient    ║
 ║  • Learning rate:  must satisfy Robbins–Monro conditions            ║
 ║  • Batch methods:  experience replay, LSTD — reuse data            ║
 ║  • Convergence:    MC → global opt (linear FA), TD → fixed point   ║
 ║                                                                     ║
 ╚══════════════════════════════════════════════════════════════════════╝
```

---

## 1. The Value Error Objective

### 1.1 Mean Squared Value Error

We want $\hat{V}(s, \mathbf{w}) \approx v_\pi(s)$ for all states. The natural
objective is the **Mean Squared Value Error (VE)**, weighted by the on-policy state
distribution $\mu(s)$:

$$\overline{\text{VE}}(\mathbf{w}) \doteq \sum_{s \in \mathcal{S}} \mu(s) \left[ v_\pi(s) - \hat{V}(s, \mathbf{w}) \right]^2$$

In expectation form:

$$\overline{\text{VE}}(\mathbf{w}) = \mathbb{E}_\mu \left[ \left( v_\pi(S) - \hat{V}(S, \mathbf{w}) \right)^2 \right]$$

The weighting by $\mu(s)$ is essential: states visited more often under $\pi$ matter
more for prediction accuracy. For episodic tasks, $\mu(s)$ is the fraction of time
steps spent in state $s$ on average.

The $\mu$-weighting ensures the approximation is best where it matters most — along
trajectories the policy actually generates. On-policy samples naturally weight states
by $\mu$, making SGD updates automatically follow this distribution.

---

## 2. The Gradient of VE

### 2.1 Deriving the Full Gradient

Taking the gradient of $\overline{\text{VE}}$ with respect to $\mathbf{w}$:

$$\nabla_\mathbf{w} \overline{\text{VE}}(\mathbf{w}) = \nabla_\mathbf{w} \mathbb{E}_\mu \left[ \left( v_\pi(S) - \hat{V}(S, \mathbf{w}) \right)^2 \right]$$

Since $v_\pi(S)$ does not depend on $\mathbf{w}$, the chain rule gives:

$$\nabla_\mathbf{w} \overline{\text{VE}}(\mathbf{w}) = -2 \, \mathbb{E}_\mu \left[ \left( v_\pi(S) - \hat{V}(S, \mathbf{w}) \right) \nabla_\mathbf{w} \hat{V}(S, \mathbf{w}) \right]$$

### 2.2 Gradient Descent Update

Standard gradient descent steps in the negative gradient direction:

$$\mathbf{w}_{t+1} = \mathbf{w}_t - \frac{1}{2} \alpha \nabla_\mathbf{w} \overline{\text{VE}}(\mathbf{w}_t)$$

Substituting (the $\frac{1}{2}$ cancels the $-2$):

$$\mathbf{w}_{t+1} = \mathbf{w}_t + \alpha \, \mathbb{E}_\mu \left[ \left( v_\pi(S) - \hat{V}(S, \mathbf{w}_t) \right) \nabla_\mathbf{w} \hat{V}(S, \mathbf{w}_t) \right]$$

This is the **expected update**. We cannot compute it exactly because it requires
summing over all states weighted by $\mu$ and knowing $v_\pi$.

---

## 3. Stochastic Gradient Descent (SGD)

### 3.1 The SGD Principle

Replace the expectation with a **single-sample estimate**. If $S_t$ is sampled from
$\mu$ (which happens naturally by following $\pi$), the SGD update is:

$$\boxed{\mathbf{w}_{t+1} = \mathbf{w}_t + \alpha \left[ v_\pi(S_t) - \hat{V}(S_t, \mathbf{w}_t) \right] \nabla_\mathbf{w} \hat{V}(S_t, \mathbf{w}_t)}$$

This is an unbiased estimate of the true gradient direction.

### 3.2 The Fundamental Problem

We **do not know** $v_\pi(S_t)$. If we did, we wouldn't need to learn. Instead, we
substitute an estimate $U_t$ as the target:

$$\mathbf{w}_{t+1} = \mathbf{w}_t + \alpha \left[ U_t - \hat{V}(S_t, \mathbf{w}_t) \right] \nabla_\mathbf{w} \hat{V}(S_t, \mathbf{w}_t)$$

The choice of $U_t$ determines both the **bias** and the **convergence** properties:

| Target $U_t$ | Bias | Variance | True Gradient? | Converges to |
|:---|:---:|:---:|:---:|:---|
| $v_\pi(S_t)$ (oracle) | None | None | ✓ | Global minimum of $\overline{\text{VE}}$ |
| $G_t$ (MC return) | None | High | ✓ | Global minimum of $\overline{\text{VE}}$ |
| $R_{t+1} + \gamma \hat{V}(S_{t+1}, \mathbf{w})$ (TD(0)) | Yes | Low | ✗ | TD fixed point |
| $G_t^\lambda$ (TD(λ)) | Partial | Medium | ✗ | Between MC and TD |

---

## 4. Monte Carlo Gradient Descent

### 4.1 Using the Return as Target

The Monte Carlo return $G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots$
is an **unbiased** estimate of $v_\pi(S_t)$:

$$\mathbb{E}_\pi[G_t \mid S_t = s] = v_\pi(s)$$

Therefore the MC gradient update is:

$$\mathbf{w}_{t+1} = \mathbf{w}_t + \alpha \left[ G_t - \hat{V}(S_t, \mathbf{w}_t) \right] \nabla_\mathbf{w} \hat{V}(S_t, \mathbf{w}_t)$$

Since $G_t$ does not depend on $\mathbf{w}$ (it is computed from a completed
trajectory), this is a **true stochastic gradient method**. The update is an unbiased
sample of $-\frac{1}{2}\nabla_\mathbf{w}\overline{\text{VE}}$.

### 4.2 Convergence Guarantee

**Theorem (MC with Linear FA):** Under the Robbins–Monro conditions on $\alpha_t$
(Section 6), gradient Monte Carlo with linear function approximation
$\hat{V}(s, \mathbf{w}) = \mathbf{w}^\top \boldsymbol{\phi}(s)$ converges to the
**global optimum** of $\overline{\text{VE}}$:

$$\mathbf{w}^* = \arg\min_\mathbf{w} \overline{\text{VE}}(\mathbf{w})$$

This is because $\overline{\text{VE}}$ is convex in $\mathbf{w}$ for linear FA, and
the MC update follows the true gradient.

### 4.3 MC Gradient Pseudocode

```
Algorithm: Gradient Monte Carlo Value Estimation
─────────────────────────────────────────────────
Input:  policy π, step size α, feature vector φ(s)
Output: weight vector w ≈ w*

Initialize w ∈ ℝ^d arbitrarily (e.g., w = 0)

Loop for each episode:
│  Generate episode S₀, A₀, R₁, S₁, A₁, R₂, ..., S_T following π
│  
│  Loop for t = T-1, T-2, ..., 0:
│  │  Gₜ ← Rₜ₊₁ + γGₜ₊₁           (compute return backward)
│  │  w ← w + α [Gₜ − V̂(Sₜ, w)] ∇_w V̂(Sₜ, w)
│  │
│  │  For linear FA:
│  │    V̂(Sₜ, w) = w^T φ(Sₜ)
│  │    ∇_w V̂(Sₜ, w) = φ(Sₜ)
│  │    w ← w + α [Gₜ − w^T φ(Sₜ)] φ(Sₜ)
```

---

## 5. TD Gradient Descent (Semi-Gradient)

### 5.1 The TD Target

TD(0) uses the bootstrapped target $U_t = R_{t+1} + \gamma \hat{V}(S_{t+1}, \mathbf{w}_t)$:

$$\mathbf{w}_{t+1} = \mathbf{w}_t + \alpha \left[ R_{t+1} + \gamma \hat{V}(S_{t+1}, \mathbf{w}_t) - \hat{V}(S_t, \mathbf{w}_t) \right] \nabla_\mathbf{w} \hat{V}(S_t, \mathbf{w}_t)$$

### 5.2 Why TD Is NOT a True Gradient Method

The TD target **depends on $\mathbf{w}$**. The full gradient of the squared TD error
would include $\gamma \nabla_\mathbf{w} \hat{V}(S_{t+1}, \mathbf{w})$, but TD
ignores this term — hence the name **semi-gradient**. The full gradient is:

$$\nabla_\mathbf{w} \left[ \delta_t \right]^2 = 2\delta_t \left[ \gamma \nabla_\mathbf{w}\hat{V}(S_{t+1}, \mathbf{w}) - \nabla_\mathbf{w}\hat{V}(S_t, \mathbf{w}) \right]$$

TD keeps only $-\nabla_\mathbf{w}\hat{V}(S_t, \mathbf{w})$, dropping the bootstrapped
gradient term.

### 5.3 The TD Fixed Point

Semi-gradient TD(0) with linear FA converges to $\mathbf{w}_\text{TD}$ satisfying
$\mathbf{A}\mathbf{w}_\text{TD} = \mathbf{b}$ where:

$$\mathbf{A} = \mathbb{E}_\mu \left[ \boldsymbol{\phi}(S_t) \left( \boldsymbol{\phi}(S_t) - \gamma \boldsymbol{\phi}(S_{t+1}) \right)^\top \right], \quad \mathbf{b} = \mathbb{E}_\mu \left[ R_{t+1} \boldsymbol{\phi}(S_t) \right]$$

The error at this fixed point is bounded:

$$\overline{\text{VE}}(\mathbf{w}_\text{TD}) \leq \frac{1}{1 - \gamma} \min_\mathbf{w} \overline{\text{VE}}(\mathbf{w})$$

---

## 6. Learning Rate Schedules

### 6.1 The Robbins–Monro Conditions

For stochastic approximation to converge, the step-size sequence $\{\alpha_t\}$ must
satisfy:

$$\sum_{t=0}^{\infty} \alpha_t = \infty \qquad \text{and} \qquad \sum_{t=0}^{\infty} \alpha_t^2 < \infty$$

**Intuition:**
- $\sum \alpha_t = \infty$ ensures steps are large enough to reach any optimum,
  regardless of starting point.
- $\sum \alpha_t^2 < \infty$ ensures steps shrink fast enough that noise averages out.

**Example:** $\alpha_t = \frac{1}{t}$ satisfies both conditions (harmonic series
diverges, $\sum 1/t^2 = \pi^2/6$ converges).

### 6.2 Common Schedules

| Schedule | Formula | Robbins–Monro? | Notes |
|----------|---------|:--------------:|-------|
| **Constant** | $\alpha_t = \alpha$ | ✗ | Does not converge to a point; oscillates in a region. Good for non-stationary problems |
| **1/t decay** | $\alpha_t = \frac{\alpha_0}{1 + \beta t}$ | ✓ | Classic choice. $\beta$ controls decay speed |
| **Exponential decay** | $\alpha_t = \alpha_0 \cdot \lambda^t$ | ✗ | Decays too fast ($\sum \alpha_t < \infty$). Used in practice for finite horizons |
| **Cosine annealing** | $\alpha_t = \alpha_\min + \frac{1}{2}(\alpha_0 - \alpha_\min)(1 + \cos\frac{\pi t}{T})$ | ✗ | Smooth warm-restarts. Popular in deep RL |
| **Linear decay** | $\alpha_t = \alpha_0(1 - t/T)$ | ✗ | Simple, reaches zero at step $T$ |

### 6.3 Practical Advice

A **constant step size** is often preferred in RL because:
1. The problem is inherently **non-stationary** (policy changes during control).
2. We want recent experience to have more influence than old experience.
3. Theoretical convergence to a point is less important than tracking a moving target.

Typical constant step sizes: $\alpha \in [10^{-4}, 10^{-2}]$ depending on the
representation and problem scale.

---

## 7. Batch vs Online vs Mini-Batch

### 7.1 Update Granularity

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │              BATCH vs MINI-BATCH vs ONLINE (SGD)                    │
  │                                                                     │
  │  Full Batch (N samples):                                            │
  │    w ← w + α · (1/N) Σᵢ [Uᵢ − V̂(Sᵢ,w)] ∇V̂(Sᵢ,w)                │
  │    • Exact gradient estimate over dataset                           │
  │    • Smooth but expensive per step                                  │
  │    • One update per sweep through all data                          │
  │                                                                     │
  │  Mini-Batch (B samples, B << N):                                    │
  │    w ← w + α · (1/B) Σⱼ [Uⱼ − V̂(Sⱼ,w)] ∇V̂(Sⱼ,w)                │
  │    • Reduced variance vs single sample                              │
  │    • GPU-friendly parallelism                                       │
  │    • B ∈ {32, 64, 128, 256} typical                                 │
  │                                                                     │
  │  Online / SGD (1 sample):                                           │
  │    w ← w + α [Uₜ − V̂(Sₜ,w)] ∇V̂(Sₜ,w)                            │
  │    • Maximum noise, cheapest per step                               │
  │    • Can learn during episode (TD)                                   │
  │    • Natural for streaming RL data                                  │
  └──────────────────────────────────────────────────────────────────────┘
```

### 7.2 Mini-Batch in Deep RL

In deep RL, mini-batch updates from an **experience replay buffer** are standard:

1. Collect experience $(s, a, r, s')$ into replay buffer $\mathcal{D}$.
2. Sample a mini-batch $\mathcal{B} \subset \mathcal{D}$ uniformly at random.
3. Compute the gradient on $\mathcal{B}$ and update weights.

This breaks temporal correlation and reuses data for better sample efficiency.

---

## 8. Batch Methods: Experience Replay and LSTD

### 8.1 Experience Replay (Revisited)

Given a finite dataset of transitions $\mathcal{D} = \{(S_i, U_i)\}_{i=1}^N$,
repeatedly resample and apply gradient updates until convergence:

```
Algorithm: Batch Gradient Descent with Experience Replay
────────────────────────────────────────────────────────
Input:  dataset D = {(S₁,U₁), ..., (S_N, U_N)}, step size α
Output: weight vector w

Initialize w arbitrarily

Repeat until convergence:
│  Shuffle D or sample mini-batch B ⊂ D
│  For each (Sᵢ, Uᵢ) in B:
│  │  w ← w + α [Uᵢ − V̂(Sᵢ, w)] ∇_w V̂(Sᵢ, w)
│
│  (Check ‖Δw‖ < ε for convergence)
```

With linear FA, this converges to the **least-squares solution**:

$$\mathbf{w}^* = \left( \boldsymbol{\Phi}^\top \boldsymbol{\Phi} \right)^{-1} \boldsymbol{\Phi}^\top \mathbf{u}$$

where $\boldsymbol{\Phi}$ is the $N \times d$ feature matrix and $\mathbf{u}$ is the
vector of targets.

### 8.2 Least-Squares TD (LSTD)

LSTD computes the TD fixed point **directly** without iterative updates or a step
size $\alpha$. For linear FA $\hat{V}(s, \mathbf{w}) = \mathbf{w}^\top \boldsymbol{\phi}(s)$:

$$\boxed{\mathbf{w}_\text{LSTD} = \hat{\mathbf{A}}^{-1} \hat{\mathbf{b}}}$$

where the empirical estimates are:

$$\hat{\mathbf{A}} = \sum_{t=0}^{T-1} \boldsymbol{\phi}(S_t) \left( \boldsymbol{\phi}(S_t) - \gamma \boldsymbol{\phi}(S_{t+1}) \right)^\top + \epsilon \mathbf{I}$$

$$\hat{\mathbf{b}} = \sum_{t=0}^{T-1} R_{t+1} \, \boldsymbol{\phi}(S_t)$$

The small regularisation $\epsilon \mathbf{I}$ ensures invertibility.

### 8.3 LSTD Properties

| Property | SGD TD | LSTD |
|----------|:------:|:----:|
| **Step size $\alpha$** | Required | Not needed |
| **Convergence speed** | $O(1/\sqrt{T})$ | Immediate (one-shot) |
| **Computation** | $O(d)$ per step | $O(d^2)$ per step, $O(d^3)$ for solve |
| **Memory** | $O(d)$ | $O(d^2)$ (stores $\hat{\mathbf{A}}$) |
| **Incremental update** | Natural | Use Sherman–Morrison for $O(d^2)$ inverse update |
| **Scalability** | Scales to millions of params | Limited to $d \lesssim 10{,}000$ |
| **Non-stationarity** | Adapts with constant $\alpha$ | Must re-solve for new policy |

### 8.4 Sherman–Morrison Incremental Update

Maintain $\hat{\mathbf{A}}^{-1}$ online in $O(d^2)$ per step instead of $O(d^3)$:

$$\hat{\mathbf{A}}_{t+1}^{-1} = \hat{\mathbf{A}}_t^{-1} - \frac{\hat{\mathbf{A}}_t^{-1} \boldsymbol{\phi}(S_t)(\boldsymbol{\phi}(S_t) - \gamma\boldsymbol{\phi}(S_{t+1}))^\top \hat{\mathbf{A}}_t^{-1}}{1 + (\boldsymbol{\phi}(S_t) - \gamma\boldsymbol{\phi}(S_{t+1}))^\top \hat{\mathbf{A}}_t^{-1} \boldsymbol{\phi}(S_t)}$$

---

## 9. Convergence Properties

### 9.1 Convergence Summary

```
  ╔══════════════════════════════════════════════════════════════════════╗
  ║              CONVERGENCE OF GRADIENT METHODS                        ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║                                                                     ║
  ║                    LINEAR FA              NONLINEAR FA              ║
  ║              ┌──────────────────┐   ┌──────────────────┐           ║
  ║  MC          │ Global optimum   │   │ Local optimum    │           ║
  ║  (true grad) │ w* = argmin VE   │   │ (no guarantee of │           ║
  ║              │ ✓ Guaranteed     │   │  global minimum) │           ║
  ║              └──────────────────┘   └──────────────────┘           ║
  ║              ┌──────────────────┐   ┌──────────────────┐           ║
  ║  TD(0)       │ TD fixed point   │   │ May diverge!     │           ║
  ║  (semi-grad) │ VE ≤ 1/(1-γ) min│   │ (Deadly Triad)   │           ║
  ║              │ ✓ Guaranteed     │   │ ✗ No guarantee   │           ║
  ║              └──────────────────┘   └──────────────────┘           ║
  ║              ┌──────────────────┐   ┌──────────────────┐           ║
  ║  LSTD        │ TD fixed point   │   │ Not applicable   │           ║
  ║  (direct)    │ One-shot exact   │   │ (linear only)    │           ║
  ║              │ ✓ Guaranteed     │   │                  │           ║
  ║              └──────────────────┘   └──────────────────┘           ║
  ║                                                                     ║
  ╚══════════════════════════════════════════════════════════════════════╝
```

### 9.2 The Loss Surface for Linear FA

For linear FA, $\overline{\text{VE}}(\mathbf{w})$ is a **convex quadratic** in
$\mathbf{w}$ with Hessian $2\mathbb{E}_\mu[\boldsymbol{\phi}(S)\boldsymbol{\phi}(S)^\top] \succeq 0$.
There is a unique minimum (if features are linearly independent) and gradient descent
follows nested elliptical contours down to it.

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │     LOSS SURFACE AND GRADIENT DESCENT PATH (Linear FA, d=2)         │
  │                                                                     │
  │  w₂ ↑                                                               │
  │     │          ___________                                          │
  │     │        /             \          Start ★                       │
  │     │      /    _________    \         ↓                            │
  │     │    /    /           \    \      ╲                             │
  │     │   |   |   _______   |   |       ╲                            │
  │     │   |   |  |       |  |   |    ← ─ ╲  gradient steps           │
  │     │   |   |  |  w*●  |  |   |        ╱ ╲                         │
  │     │   |   |  |_______|  |   |       ╱   ╲                        │
  │     │   |   |             |   |      ╱                             │
  │     │    \   \_________  /   /      ╱  SGD path (noisy)            │
  │     │      \            /   /                                       │
  │     │        \_________/   /    Elliptical contours: VE(w) = c     │
  │     │                                                               │
  │     └───────────────────────────────────────────────────→ w₁        │
  └──────────────────────────────────────────────────────────────────────┘
```

---

## 10. Python Implementation

The following implements gradient MC, semi-gradient TD(0), and LSTD with configurable
learning rate schedules on a random-walk environment with linear function approximation.

```python
import numpy as np

# ── Learning Rate Schedules ────────────────────────────────────────────
def constant_lr(a0):
    return lambda t: a0

def inverse_decay_lr(a0, beta=0.001):
    """α_t = α₀/(1+βt). Satisfies Robbins–Monro."""
    return lambda t: a0 / (1.0 + beta * t)

def exponential_decay_lr(a0, decay=0.999):
    return lambda t: a0 * (decay ** t)

def cosine_annealing_lr(a0, a_min, T):
    return lambda t: a_min + 0.5*(a0 - a_min)*(1 + np.cos(np.pi*min(t,T)/T))

# ── Random Walk Environment ──────────────────────────────────────────
class RandomWalk:
    """1D random walk with N states. Terminal at 0 and N+1."""
    def __init__(self, n=19):
        self.n = n; self.state = n//2 + 1
    def reset(self):
        self.state = self.n//2 + 1; return self.state
    def step(self):
        self.state += np.random.choice([-1, 1])
        if self.state <= 0:    return self.state, -1.0, True
        if self.state > self.n: return self.state,  1.0, True
        return self.state, 0.0, False

def phi(s, n):
    """One-hot feature vector."""
    x = np.zeros(n); x[s-1] = 1.0; return x

# ── Gradient Monte Carlo ─────────────────────────────────────────────
def gradient_mc(env, n_ep=500, lr=None, gamma=1.0):
    lr = lr or constant_lr(0.01)
    w, t, true_v = np.zeros(env.n), 0, np.linspace(-1,1,env.n+2)[1:-1]
    errors = []
    for _ in range(n_ep):
        s, traj = env.reset(), []
        while True:
            s2, r, done = env.step(); traj.append((s, r))
            if done: break
            s = s2
        G = 0.0
        for s, r in reversed(traj):
            G = gamma*G + r
            p = phi(s, env.n)
            w += lr(t) * (G - w@p) * p;  t += 1       # true gradient
        preds = np.array([w@phi(i,env.n) for i in range(1,env.n+1)])
        errors.append(np.sqrt(np.mean((preds - true_v)**2)))
    return w, errors

# ── Semi-Gradient TD(0) ──────────────────────────────────────────────
def semi_gradient_td0(env, n_ep=500, lr=None, gamma=1.0):
    lr = lr or constant_lr(0.01)
    w, t, true_v = np.zeros(env.n), 0, np.linspace(-1,1,env.n+2)[1:-1]
    errors = []
    for _ in range(n_ep):
        s = env.reset()
        while True:
            p = phi(s, env.n)
            s2, r, done = env.step()
            target = r if done else r + gamma*(w@phi(s2,env.n))
            w += lr(t) * (target - w@p) * p;  t += 1   # semi-gradient
            if done: break
            s = s2
        preds = np.array([w@phi(i,env.n) for i in range(1,env.n+1)])
        errors.append(np.sqrt(np.mean((preds - true_v)**2)))
    return w, errors

# ── LSTD ──────────────────────────────────────────────────────────────
def lstd(env, n_ep=100, gamma=1.0, eps=1e-4):
    """Direct solution: w = A⁻¹b. No step size needed."""
    A, b = eps*np.eye(env.n), np.zeros(env.n)
    for _ in range(n_ep):
        s = env.reset()
        while True:
            p = phi(s, env.n)
            s2, r, done = env.step()
            p2 = np.zeros(env.n) if done else phi(s2, env.n)
            A += np.outer(p, p - gamma*p2);  b += r*p
            if done: break
            s = s2
    return np.linalg.solve(A, b)

# ── Compare Methods ──────────────────────────────────────────────────
if __name__ == "__main__":
    env = RandomWalk(19)
    schedules = {
        "Constant (α=0.01)":  constant_lr(0.01),
        "1/t decay":          inverse_decay_lr(0.05, 0.005),
        "Exponential decay":  exponential_decay_lr(0.05, 0.9995),
        "Cosine annealing":   cosine_annealing_lr(0.05, 0.001, 800),
    }
    print("=" * 55)
    print("  Gradient MC — Learning Rate Schedule Comparison")
    print("=" * 55)
    for name, sched in schedules.items():
        np.random.seed(42)
        _, errs = gradient_mc(env, 500, lr=sched)
        print(f"  {name:28s} RMSVE = {errs[-1]:.4f}")

    print("\n" + "=" * 55)
    print("  MC  vs  TD(0)  vs  LSTD")
    print("=" * 55)
    for label, fn in [("Gradient MC", lambda: gradient_mc(env, 500)),
                      ("Semi-grad TD", lambda: semi_gradient_td0(env, 500))]:
        np.random.seed(42);  _, e = fn()
        print(f"  {label:16s} RMSVE = {e[-1]:.4f}")
    np.random.seed(42);  w_l = lstd(env, 500)
    tv = np.linspace(-1,1,21)[1:-1]
    print(f"  {'LSTD':16s} RMSVE = "
          f"{np.sqrt(np.mean((np.array([w_l@phi(i,19) for i in range(1,20)])-tv)**2)):.4f}")
```

---

## 11. Summary

| Concept | Key Point |
|---------|-----------|
| Value Error objective | $\overline{\text{VE}}(\mathbf{w}) = \mathbb{E}_\mu[(v_\pi(S) - \hat{V}(S,\mathbf{w}))^2]$ weighted by on-policy distribution |
| True gradient | Requires knowing $v_\pi$ — impractical; must substitute targets |
| SGD update | $\mathbf{w} \leftarrow \mathbf{w} + \alpha[U_t - \hat{V}(S_t,\mathbf{w})]\nabla\hat{V}(S_t,\mathbf{w})$ |
| MC target ($G_t$) | Unbiased, true gradient method; converges to global minimum (linear FA) |
| TD target ($R + \gamma\hat{V}$) | Biased, semi-gradient; converges to TD fixed point (linear FA) |
| Robbins–Monro | $\sum\alpha_t = \infty$ and $\sum\alpha_t^2 < \infty$ for convergence |
| Constant step size | Violates Robbins–Monro but tracks non-stationary problems |
| Batch vs online | Full batch = exact gradient; mini-batch = variance reduction; online = cheapest |
| Experience replay | Resamples from buffer; breaks correlation; improves data efficiency |
| LSTD | Direct solution $\mathbf{w} = \hat{\mathbf{A}}^{-1}\hat{\mathbf{b}}$; no step size; $O(d^2)$ memory |
| TD fixed point bound | $\overline{\text{VE}}(\mathbf{w}_\text{TD}) \leq \frac{1}{1-\gamma}\min_\mathbf{w}\overline{\text{VE}}(\mathbf{w})$ |

---

## Revision Questions

1. **Derive the SGD update for the Mean Squared Value Error.** Starting from
   $\overline{\text{VE}}(\mathbf{w}) = \mathbb{E}_\mu[(v_\pi(S) - \hat{V}(S, \mathbf{w}))^2]$,
   show how the gradient leads to the weight update rule, and explain why we use a
   single-sample estimate instead of the full expectation.

2. **Explain why TD(0) is not a true gradient method.** Identify which term in the
   gradient is dropped and why. What implicit objective does semi-gradient TD(0)
   minimise, and why does this still lead to useful convergence results for linear FA?

3. **State the Robbins–Monro conditions and verify them for $\alpha_t = c/t$.** Which
   of the common learning rate schedules (constant, exponential decay, cosine
   annealing) satisfy these conditions? Why might we prefer a schedule that violates
   them in practice?

4. **Compare the computational and statistical tradeoffs of LSTD versus semi-gradient
   TD(0).** For a problem with $d = 500$ features and $10^6$ transitions, which method
   would you choose and why? What about $d = 50{,}000$?

5. **Design an experiment to compare MC gradient descent with four learning rate
   schedules.** Specify the environment, feature representation, performance metric,
   number of runs, and what you would plot. Predict which schedule will perform best in
   early learning vs. asymptotically.

6. **Prove that $\overline{\text{VE}}(\mathbf{w})$ is convex for linear FA.** Compute
   the Hessian $\nabla^2_\mathbf{w}\overline{\text{VE}}$ and show it is positive
   semi-definite. Under what condition on the feature vectors is the minimum unique?

---

## Navigation

| ← Previous | ↑ Up | Next → |
|:-----------:|:----:|:------:|
| [Neural Network Approximation](./03-neural-network-approximation.md) | [Unit 6: Function Approximation](./README.md) | [Semi-Gradient Methods](./05-semi-gradient-methods.md) |
