# Chapter 3: Neural Network Function Approximation

> **Unit 6: Function Approximation** — *Using deep networks as powerful, flexible value and policy approximators.*

---

## Overview

Linear function approximation requires hand-crafted features and can only represent
value functions in the span of those features. Many real-world value surfaces are
**highly nonlinear** — cliffs, ridges, pockets — and no fixed feature set captures them
all. Neural networks **learn their own features** end-to-end, acting as universal
function approximators that can represent arbitrarily complex mappings from states to
values. This power comes at a cost: we lose the clean convergence guarantees of the
linear case, and a set of interacting instabilities — the **Deadly Triad** — emerges
when we combine neural networks with bootstrapping and off-policy learning.

```
 ╔══════════════════════════════════════════════════════════════════════╗
 ║           NEURAL NETWORK FUNCTION APPROXIMATION                     ║
 ╠══════════════════════════════════════════════════════════════════════╣
 ║                                                                     ║
 ║  V̂(s, w) = f_w(s)  where f is a deep neural network                ║
 ║                                                                     ║
 ║  • Learns features automatically — no hand-engineering              ║
 ║  • Universal approximation — can fit any continuous function        ║
 ║  • ∇_w V̂  computed via backpropagation (chain rule)                 ║
 ║  • Loss surface is non-convex → local minima, saddle points         ║
 ║  • No convergence guarantees with bootstrapping + off-policy        ║
 ║  • The Deadly Triad: FA + bootstrapping + off-policy → divergence   ║
 ║  • Solutions: experience replay, target networks, gradient clipping ║
 ║                                                                     ║
 ╚══════════════════════════════════════════════════════════════════════╝
```

---

## 1. Why Nonlinear Function Approximation?

### 1.1 Limitations of Linear FA

Linear FA represents values as $\hat{V}(s, \mathbf{w}) = \mathbf{w}^\top \boldsymbol{\phi}(s)$.
This is powerful when we can design good features, but fundamentally limited:

| Limitation | Explanation |
|-----------|-------------|
| **Fixed feature space** | $\boldsymbol{\phi}(s)$ is chosen before learning — the agent cannot discover new abstractions |
| **Representation ceiling** | If $V^\pi \notin \text{span}(\boldsymbol{\phi})$, no weight vector can eliminate the approximation error |
| **Feature engineering burden** | Designing features requires deep domain expertise and extensive experimentation |
| **Curse of dimensionality** | Polynomial or Fourier bases grow exponentially with state dimension |
| **No compositional structure** | Cannot represent hierarchical features (edges → shapes → objects) |

### 1.2 The Value Surface Problem

Consider a navigation task on a 2D grid with obstacles. The true value function has
sharp discontinuities around walls and smooth gradients in open space. A polynomial
basis would need extremely high degree to capture these transitions without ringing
artifacts (Gibbs phenomenon). Tile coding can approximate this but requires careful
tuning of tile sizes and offsets for each region.

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │     VALUE SURFACES THAT LINEAR FA CANNOT EASILY REPRESENT           │
  │                                                                     │
  │  True V(s):          Linear FA fit:       Neural net fit:           │
  │                                                                     │
  │  ▓▓░░░░░░▓▓          ──────────           ▓▓░░░░░░▓▓               │
  │  ▓▓░░██░░▓▓          ────────             ▓▓░░██░░▓▓               │
  │  ▓▓░░██░░▓▓   vs     ──────        vs     ▓▓░░██░░▓▓               │
  │  ▓▓░░░░░░▓▓          ────────             ▓▓░░░░░░▓▓               │
  │  ▓▓▓▓▓▓▓▓▓▓          ──────────           ▓▓▓▓▓▓▓▓▓▓               │
  │                                                                     │
  │  Sharp edges,         Smooth polynomial    Network learns edges     │
  │  local structure      misses structure     and smooth regions       │
  └──────────────────────────────────────────────────────────────────────┘
```

### 1.3 What We Need

A function class that:
- **Learns its own features** from data (no manual design)
- **Scales** to high-dimensional state spaces (images, joint angles)
- **Adapts** its representational capacity during training
- Is **differentiable** so gradient-based RL algorithms apply

Neural networks satisfy all four requirements.

---

## 2. Neural Networks as Universal Function Approximators

### 2.1 The Universal Approximation Theorem

**Theorem (Cybenko 1989, Hornik 1991):** A feedforward network with a single hidden
layer of sufficient width and a non-polynomial activation function can approximate any
continuous function on a compact subset of $\mathbb{R}^n$ to arbitrary precision.

Formally, for any continuous $g : K \to \mathbb{R}$ on compact $K \subset \mathbb{R}^n$
and any $\epsilon > 0$, there exists a network $f_\mathbf{w}$ with one hidden layer such
that:

$$\sup_{s \in K} |f_\mathbf{w}(s) - g(s)| < \epsilon$$

**Important caveats:**
- The theorem guarantees *existence*, not that SGD will *find* the right weights
- The required width may be exponentially large for some functions
- Depth (more layers) provides exponentially more efficient representations than width

### 2.2 From Linear to Nonlinear

Linear FA: fixed features, learned weights.
Neural network: **learned features AND learned weights**.

$$\underbrace{\hat{V}(s, \mathbf{w}) = \mathbf{w}_L^\top \boldsymbol{\phi}_\mathbf{w}(s)}_{\text{linear output on learned features}}$$

The hidden layers compute a **learned feature representation**
$\boldsymbol{\phi}_\mathbf{w}(s)$ that is itself a function of the weights. The output
layer is linear in those features, but the overall function is nonlinear in $\mathbf{w}$.

---

## 3. The Value Network: V̂(s, w)

### 3.1 Architecture

A state-value network maps a state vector to a single scalar value:

$$\hat{V}(s, \mathbf{w}) = f_\mathbf{w}(s) : \mathbb{R}^n \to \mathbb{R}$$

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │               STATE-VALUE NETWORK V̂(s, w)                          │
  │                                                                     │
  │  Input         Hidden Layer 1    Hidden Layer 2     Output          │
  │  (state s)     (128 neurons)     (64 neurons)       (scalar)        │
  │                                                                     │
  │  s₁ ─────┐    ┌─ h₁⁽¹⁾ ──┐     ┌─ h₁⁽²⁾ ──┐                      │
  │  s₂ ─────┤    ├─ h₂⁽¹⁾ ──┤     ├─ h₂⁽²⁾ ──┤                      │
  │  s₃ ─────┼──W₁+b₁──→ ReLU──┼──W₂+b₂──→ ReLU──┼──W₃+b₃──→ V̂(s)   │
  │  s₄ ─────┤    ├─  ⋮    ──┤     ├─  ⋮    ──┤                      │
  │   ⋮  ─────┘    └─ h₁₂₈⁽¹⁾┘     └─ h₆₄⁽²⁾ ┘                      │
  │                                                                     │
  │  w = {W₁, b₁, W₂, b₂, W₃, b₃}  — all trainable parameters       │
  │  Total params: n×128 + 128 + 128×64 + 64 + 64×1 + 1               │
  └──────────────────────────────────────────────────────────────────────┘
```

### 3.2 Forward Pass (Mathematical)

For a two-hidden-layer network with ReLU activations:

$$\mathbf{h}^{(1)} = \text{ReLU}(\mathbf{W}_1 \mathbf{s} + \mathbf{b}_1)$$

$$\mathbf{h}^{(2)} = \text{ReLU}(\mathbf{W}_2 \mathbf{h}^{(1)} + \mathbf{b}_2)$$

$$\hat{V}(s, \mathbf{w}) = \mathbf{W}_3 \mathbf{h}^{(2)} + b_3$$

where $\text{ReLU}(x) = \max(0, x)$ is applied element-wise, and:
- $\mathbf{W}_1 \in \mathbb{R}^{128 \times n}$, $\mathbf{b}_1 \in \mathbb{R}^{128}$
- $\mathbf{W}_2 \in \mathbb{R}^{64 \times 128}$, $\mathbf{b}_2 \in \mathbb{R}^{64}$
- $\mathbf{W}_3 \in \mathbb{R}^{1 \times 64}$, $b_3 \in \mathbb{R}$

The full weight vector is $\mathbf{w} = \text{vec}(\mathbf{W}_1, \mathbf{b}_1, \mathbf{W}_2, \mathbf{b}_2, \mathbf{W}_3, b_3)$.

---

## 4. The Action-Value Network: Q̂(s, a, w)

There are two fundamental architectures for representing action-value functions with
neural networks, each with distinct trade-offs.

### 4.1 Architecture A: (s, a) → Q

A single state-action pair maps to a single Q-value:

$$\hat{Q}(s, a, \mathbf{w}) = f_\mathbf{w}([s; a]) : \mathbb{R}^{n+m} \to \mathbb{R}$$

where $[s; a]$ denotes the concatenation of state and action vectors.

**Advantages:** Works with continuous action spaces (any $a$ can be evaluated).
**Disadvantages:** Requires $|\mathcal{A}|$ forward passes to find $\max_a \hat{Q}(s, a)$
in discrete settings.

### 4.2 Architecture B: s → Q(·, a) for All Actions

The network takes only the state and outputs Q-values for *every* discrete action:

$$f_\mathbf{w}(s) = [\hat{Q}(s, a_1, \mathbf{w}), \hat{Q}(s, a_2, \mathbf{w}), \dots, \hat{Q}(s, a_{|\mathcal{A}|}, \mathbf{w})]$$

**Advantages:** A single forward pass gives all Q-values — greedy action selection is
just $\arg\max$ over the output vector. This is the architecture used by DQN.
**Disadvantages:** Only works for finite, discrete action spaces.

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │          TWO Q-NETWORK ARCHITECTURES COMPARED                       │
  │                                                                     │
  │  Architecture A: (s,a) → Q           Architecture B: s → Q(·,a)    │
  │                                                                     │
  │   s ──┐                                s ──┐                        │
  │       ├──→ [hidden] ──→ Q(s,a)             ├──→ [hidden] ─┬→ Q₁    │
  │   a ──┘                                    │              ├→ Q₂    │
  │                                            │              ├→ Q₃    │
  │   Need |A| passes to find                  │              └→ Q₄    │
  │   max_a Q(s,a)                             │                       │
  │                                        One pass → all Q-values     │
  │   ✓ Continuous actions                 ✓ Efficient max_a            │
  │   ✗ Slow for discrete                 ✗ Discrete actions only      │
  └──────────────────────────────────────────────────────────────────────┘
```

### 4.3 Comparison Table

| Property | Architecture A: (s,a)→Q | Architecture B: s→Q(·,a) |
|----------|:-----------------------:|:------------------------:|
| Input | $[s; a]$ concatenated | $s$ only |
| Output | Single scalar $Q$ | Vector of $|\mathcal{A}|$ Q-values |
| Forward passes for greedy | $|\mathcal{A}|$ | 1 |
| Continuous actions | ✓ | ✗ |
| Used in | DDPG, SAC | DQN, Double DQN, Dueling DQN |

---

## 5. Gradient Computation via Backpropagation

### 5.1 The Key Equation

The semi-gradient TD update for a neural network value function is:

$$\mathbf{w} \leftarrow \mathbf{w} + \alpha \left[ R + \gamma \hat{V}(S', \mathbf{w}) - \hat{V}(S, \mathbf{w}) \right] \nabla_\mathbf{w} \hat{V}(S, \mathbf{w})$$

In the linear case, $\nabla_\mathbf{w} \hat{V} = \boldsymbol{\phi}(s)$ — trivial. For
neural networks, $\nabla_\mathbf{w} \hat{V}(s, \mathbf{w})$ involves the **chain rule**
through every layer:

$$\nabla_\mathbf{w} \hat{V}(s, \mathbf{w}) = \nabla_\mathbf{w} f_\mathbf{w}(s)$$

### 5.2 Backpropagation Through the Network

For a two-layer network $\hat{V} = \mathbf{W}_3 \text{ReLU}(\mathbf{W}_2 \text{ReLU}(\mathbf{W}_1 s + \mathbf{b}_1) + \mathbf{b}_2) + b_3$:

**Output layer gradient:**

$$\frac{\partial \hat{V}}{\partial \mathbf{W}_3} = (\mathbf{h}^{(2)})^\top, \qquad \frac{\partial \hat{V}}{\partial b_3} = 1$$

**Hidden layer 2 gradient:**

$$\frac{\partial \hat{V}}{\partial \mathbf{W}_2} = \mathbf{W}_3^\top \odot \mathbf{1}[\mathbf{h}^{(2)} > 0] \cdot (\mathbf{h}^{(1)})^\top$$

**Hidden layer 1 gradient:**

$$\frac{\partial \hat{V}}{\partial \mathbf{W}_1} = \left(\mathbf{W}_3^\top \odot \mathbf{1}[\mathbf{h}^{(2)} > 0] \cdot \mathbf{W}_2^\top \odot \mathbf{1}[\mathbf{h}^{(1)} > 0]\right) \cdot \mathbf{s}^\top$$

In practice, **automatic differentiation** (PyTorch `autograd`, TensorFlow `GradientTape`)
handles this — we never compute these derivatives by hand.

### 5.3 Contrast with Linear FA

| Property | Linear FA | Neural Network FA |
|----------|:---------:|:-----------------:|
| Gradient $\nabla_\mathbf{w} \hat{V}$ | $\boldsymbol{\phi}(s)$ — the feature vector | Requires full backpropagation |
| Computation cost | $O(d)$ — one dot product | $O(\sum_l d_l \cdot d_{l+1})$ — proportional to total params |
| Loss surface | Quadratic, convex → one global min | Non-convex → local minima, saddle points |
| Second-order info | $\nabla^2 = \boldsymbol{\Phi}^\top \mathbf{D} \boldsymbol{\Phi}$ — constant | Hessian varies with $\mathbf{w}$ — expensive to compute |

---

## 6. The Deadly Triad

The **Deadly Triad** (Sutton & Barto, 2018) identifies three elements that, when
combined, can cause RL algorithms to **diverge** — weights growing without bound,
oscillating, or converging to the wrong values.

```
 ╔══════════════════════════════════════════════════════════════════════╗
 ║                        THE DEADLY TRIAD                             ║
 ╠══════════════════════════════════════════════════════════════════════╣
 ║                                                                     ║
 ║                    Function Approximation                           ║
 ║                         /         \                                 ║
 ║                        /           \                                ║
 ║                       /    DANGER    \                               ║
 ║                      /     ZONE      \                              ║
 ║                     /   (divergence)  \                              ║
 ║                    /                   \                             ║
 ║           Bootstrapping ──────── Off-Policy                         ║
 ║                                                                     ║
 ║  Any TWO elements:  stable (usually)                                ║
 ║  All THREE together: divergence can occur                           ║
 ║                                                                     ║
 ║  Examples of all three:                                             ║
 ║    • Q-learning + neural network (the DQN setting)                  ║
 ║    • Off-policy TD with linear FA (Baird's counterexample)          ║
 ╚══════════════════════════════════════════════════════════════════════╝
```

### 6.1 Element 1: Function Approximation

When we approximate $V$ or $Q$ rather than storing exact values, an update to one
state's value **changes the values of nearby states** through weight sharing. Reducing
the error at state $s$ may *increase* the error at state $s'$. In the tabular case,
updates are independent — each state has its own entry.

### 6.2 Element 2: Bootstrapping

Bootstrapping means using our own estimate as part of the update target:

$$\text{Target} = R + \gamma \hat{V}(S', \mathbf{w})$$

The target depends on the very parameters we are updating. This creates a **moving
target problem**: as $\mathbf{w}$ changes, the target changes too. With a fixed target
(as in Monte Carlo), this is just standard supervised learning.

### 6.3 Element 3: Off-Policy Learning

Off-policy methods learn about a policy $\pi$ (target) from data generated by a
different policy $b$ (behavior). The distribution of states visited under $b$ may not
match the distribution under $\pi$. The FA is trained on the $b$-distribution but
evaluated on the $\pi$-distribution — this **distribution mismatch** can amplify
approximation errors.

### 6.4 Why Each Pair Can Cause Instability

**FA + Bootstrapping (on-policy):**
Updating $\mathbf{w}$ to reduce TD error at $s$ shifts the value at neighboring states.
If the bootstrapped target $\hat{V}(S')$ is one of those neighbors, reducing error at
$s$ may increase the target, increasing error at $s$. This feedback loop can cause
oscillation. Linear FA has the **TD fixed point** theorem to guarantee bounded error;
nonlinear FA has no such guarantee.

**FA + Off-Policy (without bootstrapping):**
Monte Carlo with FA and off-policy data requires importance sampling ratios. The variance
of these ratios grows exponentially with trajectory length, making gradient estimates
extremely noisy. The FA fits to high-variance targets, learning a poor approximation.

**Bootstrapping + Off-Policy (tabular):**
Even without FA, off-policy bootstrapping (e.g., Q-learning) can have convergence
issues in expectation under some conditions, though tabular Q-learning does converge
with suitable step sizes. The real danger emerges when FA enters the picture.

**All Three Together:**
Function approximation creates **generalization coupling** between states.
Bootstrapping creates **self-referential targets**. Off-policy creates **distribution
mismatch**. Together: the agent updates weights based on a biased target computed from
an approximation trained on the wrong distribution, and these errors compound through
generalization across the state space. Baird's counterexample demonstrates divergence
even with linear FA.

---

## 7. Instability Issues in Detail

### 7.1 Correlated Samples

In online RL, the agent generates a trajectory $s_0, a_0, r_1, s_1, a_1, r_2, \dots$
Consecutive transitions are **temporally correlated**: $s_{t+1}$ is always near $s_t$
in state space. Standard SGD assumes i.i.d. samples from the training distribution.

$$\text{Cov}[(s_t, a_t, r_{t+1}, s_{t+1}),\; (s_{t+k}, a_{t+k}, r_{t+k+1}, s_{t+k+1})] \neq 0 \quad \text{for small } k$$

**Effect:** The network repeatedly updates on similar states, fitting the current region
of state space at the expense of previously visited regions. Gradient estimates are
biased toward the recent trajectory, not the true expected gradient.

### 7.2 Non-Stationary Targets

With bootstrapping, the target $y_t = R_{t+1} + \gamma \hat{V}(S_{t+1}, \mathbf{w})$
changes as $\mathbf{w}$ changes. This is unlike supervised learning where labels are fixed.

$$\text{Loss}(\mathbf{w}) = \mathbb{E}\left[(y_t - \hat{V}(S_t, \mathbf{w}))^2\right]$$

But $y_t = y_t(\mathbf{w})$ — the loss landscape itself shifts every gradient step.
The agent is chasing a moving target, which can prevent convergence:

$$\mathbf{w}_{t+1} = \mathbf{w}_t + \alpha \underbrace{[y_t(\mathbf{w}_t) - \hat{V}(S_t, \mathbf{w}_t)]}_{\text{depends on } \mathbf{w}_t} \nabla_\mathbf{w} \hat{V}(S_t, \mathbf{w}_t)$$

### 7.3 Catastrophic Forgetting

Neural networks are global approximators — updating weights to improve predictions in
one region of state space **degrades** predictions in other regions. When the agent
moves to a new part of the environment, it overwrites knowledge about previously
visited states.

**Sequential state visitation** exacerbates this: the network over-fits to the current
trajectory segment, then forgets what it learned about earlier regions when it moves
elsewhere. This is a well-known problem in continual learning.

---

## 8. Solutions Preview

The following techniques address the instabilities identified above. Each is covered
in depth in the DQN unit; here we present the key ideas.

### 8.1 Experience Replay

Store transitions $(s, a, r, s', \text{done})$ in a **replay buffer** $\mathcal{D}$
of fixed capacity. Sample mini-batches **uniformly at random** from $\mathcal{D}$:

$$\mathbf{w} \leftarrow \mathbf{w} - \alpha \nabla_\mathbf{w} \frac{1}{|\mathcal{B}|} \sum_{(s,a,r,s') \in \mathcal{B}} \left(y - \hat{Q}(s, a, \mathbf{w})\right)^2$$

**Solves:**
- **Correlated samples** — random sampling breaks temporal correlation
- **Data efficiency** — each transition is used in many updates
- **Catastrophic forgetting** — old transitions are revisited

### 8.2 Target Networks

Maintain a separate **target network** $\hat{Q}(s, a, \mathbf{w}^-)$ with frozen
weights $\mathbf{w}^-$. The target is computed using $\mathbf{w}^-$:

$$y = R + \gamma \max_{a'} \hat{Q}(S', a', \mathbf{w}^-)$$

Periodically copy $\mathbf{w} \to \mathbf{w}^-$ (hard update) or use a soft update:

$$\mathbf{w}^- \leftarrow \tau \mathbf{w} + (1 - \tau) \mathbf{w}^- \qquad (\tau \ll 1)$$

**Solves:**
- **Non-stationary targets** — the target is fixed for many steps
- **Feedback loops** — reduces the coupling between prediction and target

### 8.3 Gradient Clipping

Limit the magnitude of the gradient to prevent large, destabilizing updates:

**Clip by value:**

$$g_i \leftarrow \text{clip}(g_i, -c, c)$$

**Clip by global norm (preferred):**

$$\mathbf{g} \leftarrow \mathbf{g} \cdot \min\left(1, \frac{c}{\|\mathbf{g}\|}\right)$$

**Solves:**
- **Exploding gradients** — especially in early training or with large TD errors
- **Training stability** — bounds the step size regardless of error magnitude

**Huber loss** (smooth L1) is a related approach that reduces gradient magnitude for
large errors automatically:

$$L_\delta(e) = \begin{cases} \frac{1}{2}e^2 & \text{if } |e| \leq \delta \\ \delta(|e| - \frac{1}{2}\delta) & \text{otherwise} \end{cases}$$

---

## 9. Network Architectures for RL

### 9.1 Fully Connected (MLP)

The most common architecture for low-dimensional state spaces (positions, velocities,
joint angles). Two hidden layers of 64–256 neurons with ReLU activation are standard.

```
  ┌──────────────────────────────────────────────────────────────────────┐
  │          TYPICAL MLP ARCHITECTURE FOR RL                            │
  │                                                                     │
  │  Layer        Shape              Activation    Notes                │
  │  ─────────────────────────────────────────────────────────────────  │
  │  Input        (batch, n_state)   —             Raw state vector     │
  │  Hidden 1     (batch, 128)       ReLU          Dense / Linear       │
  │  Hidden 2     (batch, 128)       ReLU          Dense / Linear       │
  │  Output       (batch, n_act)     None          Q-values (no act.)   │
  │                                                                     │
  │  Common choices:                                                    │
  │   • Activation: ReLU (default), Tanh (bounded), LeakyReLU           │
  │   • Initialization: Kaiming (He) for ReLU, Xavier for Tanh          │
  │   • Optimizer: Adam (lr=1e-4 to 3e-4 typical for RL)               │
  │   • No dropout, no batch norm (usually hurt RL stability)           │
  └──────────────────────────────────────────────────────────────────────┘
```

### 9.2 Convolutional Networks (for Pixel Input)

When the state is a raw image (Atari, robotic vision), convolutional layers extract
spatial features before fully connected layers output values:

$$s \in \mathbb{R}^{H \times W \times C} \xrightarrow{\text{Conv}} \xrightarrow{\text{Conv}} \xrightarrow{\text{Conv}} \xrightarrow{\text{Flatten}} \xrightarrow{\text{FC}} \hat{Q}(s, \cdot)$$

This is the DQN architecture (Mnih et al., 2015). Detailed in the DQN chapter.

### 9.3 Weight Initialization

Poor initialization can cause dead neurons (ReLU) or vanishing gradients. Standard
practice:

| Activation | Initialization | Variance |
|:----------:|:--------------:|:--------:|
| ReLU | He (Kaiming) | $\text{Var}(w) = 2/n_\text{in}$ |
| Tanh / Sigmoid | Xavier (Glorot) | $\text{Var}(w) = 2/(n_\text{in} + n_\text{out})$ |
| Output layer | Uniform small | $w \sim U(-3\times10^{-3}, 3\times10^{-3})$ |

---

## 10. Python Example: Value Network for CartPole

The following implements a Q-network with experience replay and a target network —
a minimal but complete deep RL agent.

```python
import torch
import torch.nn as nn
import torch.optim as optim
import numpy as np
from collections import deque
import random

# ── Q-Network ──────────────────────────────────────────────────────────
class QNetwork(nn.Module):
    """Two-hidden-layer MLP for Q-value approximation."""

    def __init__(self, state_dim, action_dim, hidden=128):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(state_dim, hidden),
            nn.ReLU(),
            nn.Linear(hidden, hidden),
            nn.ReLU(),
            nn.Linear(hidden, action_dim),  # one output per action
        )

    def forward(self, x):
        return self.net(x)


# ── Replay Buffer ─────────────────────────────────────────────────────
class ReplayBuffer:
    """Fixed-size FIFO buffer for experience replay."""

    def __init__(self, capacity=10_000):
        self.buffer = deque(maxlen=capacity)

    def push(self, state, action, reward, next_state, done):
        self.buffer.append((state, action, reward, next_state, done))

    def sample(self, batch_size):
        batch = random.sample(self.buffer, batch_size)
        states, actions, rewards, next_states, dones = zip(*batch)
        return (
            torch.FloatTensor(np.array(states)),
            torch.LongTensor(actions),
            torch.FloatTensor(rewards),
            torch.FloatTensor(np.array(next_states)),
            torch.FloatTensor(dones),
        )

    def __len__(self):
        return len(self.buffer)


# ── DQN Agent ─────────────────────────────────────────────────────────
class DQNAgent:
    """Minimal DQN agent with target network and experience replay."""

    def __init__(self, state_dim, action_dim, lr=1e-3, gamma=0.99,
                 eps_start=1.0, eps_end=0.01, eps_decay=500,
                 target_update=50, batch_size=64):
        self.action_dim = action_dim
        self.gamma = gamma
        self.eps_start = eps_start
        self.eps_end = eps_end
        self.eps_decay = eps_decay
        self.target_update = target_update
        self.batch_size = batch_size
        self.steps = 0

        # Online and target networks
        self.q_net = QNetwork(state_dim, action_dim)
        self.target_net = QNetwork(state_dim, action_dim)
        self.target_net.load_state_dict(self.q_net.state_dict())

        self.optimizer = optim.Adam(self.q_net.parameters(), lr=lr)
        self.buffer = ReplayBuffer()

    def select_action(self, state):
        """Epsilon-greedy action selection."""
        eps = self.eps_end + (self.eps_start - self.eps_end) * \
              np.exp(-self.steps / self.eps_decay)
        self.steps += 1
        if random.random() < eps:
            return random.randrange(self.action_dim)
        with torch.no_grad():
            q_vals = self.q_net(torch.FloatTensor(state).unsqueeze(0))
            return q_vals.argmax(dim=1).item()

    def update(self):
        """One step of gradient descent on a mini-batch."""
        if len(self.buffer) < self.batch_size:
            return None

        states, actions, rewards, next_states, dones = \
            self.buffer.sample(self.batch_size)

        # Current Q-values for chosen actions
        q_values = self.q_net(states).gather(1, actions.unsqueeze(1)).squeeze(1)

        # Target Q-values from frozen target network
        with torch.no_grad():
            next_q = self.target_net(next_states).max(dim=1)[0]
            targets = rewards + self.gamma * next_q * (1 - dones)

        # Huber loss (smooth L1) for gradient clipping effect
        loss = nn.SmoothL1Loss()(q_values, targets)

        self.optimizer.zero_grad()
        loss.backward()          # backpropagation computes ∇_w L
        nn.utils.clip_grad_norm_(self.q_net.parameters(), max_norm=10.0)
        self.optimizer.step()

        return loss.item()

    def sync_target(self, episode):
        """Hard update: copy online weights to target network."""
        if episode % self.target_update == 0:
            self.target_net.load_state_dict(self.q_net.state_dict())


# ── Training Loop ─────────────────────────────────────────────────────
def train_cartpole(n_episodes=400):
    """Train DQN on CartPole-v1 (no gym dependency — simplified env)."""
    # Simplified CartPole dynamics for self-contained example
    # State: [x, x_dot, theta, theta_dot]
    # Actions: 0 (push left), 1 (push right)

    agent = DQNAgent(state_dim=4, action_dim=2)

    def reset_env():
        return np.random.uniform(-0.05, 0.05, size=4)

    def step_env(state, action):
        x, x_dot, theta, theta_dot = state
        force = 10.0 if action == 1 else -10.0
        cos_th, sin_th = np.cos(theta), np.sin(theta)
        temp = (force + 0.05 * theta_dot**2 * sin_th) / 1.1
        th_acc = (9.8*sin_th - cos_th*temp) / \
                 (0.5 * (4.0/3.0 - 0.1*cos_th**2 / 1.1))
        x_acc = temp - 0.05 * th_acc * cos_th / 1.1
        dt = 0.02
        x      += dt * x_dot
        x_dot  += dt * x_acc
        theta  += dt * theta_dot
        theta_dot += dt * th_acc
        ns = np.array([x, x_dot, theta, theta_dot])
        done = abs(x) > 2.4 or abs(theta) > 0.2095
        reward = 0.0 if done else 1.0
        return ns, reward, done

    for ep in range(n_episodes):
        state = reset_env()
        total_reward = 0
        for t in range(500):
            action = agent.select_action(state)
            next_state, reward, done = step_env(state, action)
            agent.buffer.push(state, action, reward, next_state, float(done))
            agent.update()
            total_reward += reward
            if done:
                break
            state = next_state

        agent.sync_target(ep)
        if (ep + 1) % 50 == 0:
            print(f"Episode {ep+1:4d} | Reward: {total_reward:6.1f} | "
                  f"Buffer: {len(agent.buffer):5d} | "
                  f"Eps: {agent.eps_end + (agent.eps_start - agent.eps_end) * np.exp(-agent.steps / agent.eps_decay):.3f}")


if __name__ == "__main__":
    train_cartpole()
```

**Key design points:**
- **Architecture B** (s → Q for all actions): single forward pass for action selection
- **Huber loss** instead of MSE: reduces impact of large TD errors
- **Gradient clipping** (`clip_grad_norm_`): prevents exploding gradients
- **Target network** synced every 50 episodes: stabilizes learning targets
- **Experience replay buffer**: breaks temporal correlation in sequential data

---

## 11. When to Use Neural Networks vs Linear FA

The choice between linear and neural function approximation is not always obvious.
Neural networks are not universally better — their instabilities can outweigh their
representational advantages in many settings.

### 11.1 Decision Framework

| Criterion | Linear FA | Neural Network FA |
|-----------|:---------:|:-----------------:|
| **State dimension** | Low to moderate ($\leq 20$) | Any (scales to images) |
| **Feature engineering** | Must design features manually | Learns features automatically |
| **Convergence guarantees** | TD fixed point theorem; bounded error | No theoretical guarantees |
| **Sample efficiency** | High (few parameters) | Low (many parameters, needs replay) |
| **Computational cost** | $O(d)$ per update | $O(\text{params})$ per update + backprop |
| **Representational power** | Limited to $\text{span}(\boldsymbol{\phi})$ | Universal approximation |
| **Debugging ease** | Easy: inspect weights and features | Hard: opaque representations |
| **Stability** | Stable with on-policy TD | Requires replay, target nets, clipping |
| **Domain expertise needed** | Feature design requires expertise | Architecture tuning requires expertise |
| **Recommended for** | Small–medium MDPs, when features are known | Large state spaces, complex value surfaces |

### 11.2 Rules of Thumb

1. **Start linear.** If tile coding or RBFs work, don't add complexity.
2. **Go neural when features fail.** If no hand-crafted feature set captures the value
   function (e.g., raw pixel input), neural networks are necessary.
3. **More data = more benefit from neural nets.** Neural networks shine when data is
   abundant and diverse; they struggle with limited, correlated data.
4. **Always add stabilization.** Never use a raw neural network with bootstrapping in
   RL — always add experience replay and target networks at minimum.

---

## 12. Summary

```
╔═══════════════════════════════════════════════════════════════════════╗
║  CHAPTER 3 — KEY TAKEAWAYS                                          ║
╠═══════════════════════════════════════════════════════════════════════╣
║  1. Neural networks learn features end-to-end — no hand-crafting     ║
║  2. V̂(s, w) = f_w(s) — the network IS the value function            ║
║  3. Q-networks: (s,a)→Q for continuous, s→Q(·,a) for discrete       ║
║  4. Gradients via backpropagation — automatic differentiation        ║
║  5. Deadly Triad: FA + bootstrapping + off-policy → divergence       ║
║  6. Correlated samples violate i.i.d. → experience replay            ║
║  7. Non-stationary targets → target network with frozen weights      ║
║  8. Catastrophic forgetting → replay revisits old transitions        ║
║  9. Gradient clipping + Huber loss → stable gradient magnitudes      ║
║ 10. Start linear, go neural when features fail or states are large   ║
╚═══════════════════════════════════════════════════════════════════════╝
```

| Concept | Key Point |
|---------|-----------|
| Nonlinear FA motivation | Linear FA limited to $\text{span}(\boldsymbol{\phi})$; neural nets learn their own features |
| Universal approximation | One hidden layer of sufficient width can approximate any continuous function |
| State-value network | $\hat{V}(s, \mathbf{w}) = f_\mathbf{w}(s) : \mathbb{R}^n \to \mathbb{R}$ |
| Q-network (Architecture B) | $f_\mathbf{w}(s) \to [\hat{Q}(s, a_1), \dots, \hat{Q}(s, a_{|\mathcal{A}|})]$ — one pass for all actions |
| Gradient computation | $\nabla_\mathbf{w} \hat{V}$ via backpropagation (chain rule through all layers) |
| Deadly Triad | FA + bootstrapping + off-policy → potential divergence |
| Correlated samples | Sequential RL data violates SGD's i.i.d. assumption |
| Non-stationary targets | Bootstrapped targets shift as weights update |
| Experience replay | Random sampling from buffer breaks correlation |
| Target network | Frozen $\mathbf{w}^-$ stabilizes the regression target |
| Gradient clipping | Bounds $\|\mathbf{g}\|$ to prevent exploding updates |

---

## Revision Questions

1. **Explain why a neural network can represent value functions that linear FA cannot.**
   Give a concrete example of a value surface that requires nonlinear approximation, and
   state the universal approximation theorem precisely.

2. **Compare the two Q-network architectures.** For a discrete action space with
   $|\mathcal{A}| = 18$ (Atari), how many forward passes does each architecture need to
   select a greedy action? Which is used in DQN and why?

3. **Derive the gradient $\nabla_{\mathbf{W}_2} \hat{V}$ for a two-layer ReLU network.**
   Why is automatic differentiation preferred over manual derivation in practice?

4. **State and explain the Deadly Triad.** Why is each element individually safe but
   their combination dangerous? Give Baird's counterexample as an illustration.

5. **Describe three instability issues in neural network RL and their solutions.** For
   each problem (correlated samples, non-stationary targets, catastrophic forgetting),
   explain the mechanism and the corresponding stabilization technique.

6. **When should you prefer linear FA over neural networks?** Suppose you have a
   4-dimensional continuous state space with known physics. Argue for linear FA with
   domain-specific features vs. a neural network, considering convergence, sample
   efficiency, and debuggability.

---

## Navigation

| ← Previous | ↑ Up | Next → |
|:-----------:|:----:|:------:|
| [Linear Function Approximation](./02-linear-function-approximation.md) | [Unit 6: Function Approximation](./README.md) | [Gradient Descent Methods](./04-gradient-descent-methods.md) |
