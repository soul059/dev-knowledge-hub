# Chapter 1: Why Function Approximation?

> **Unit 6: Function Approximation** — *Scaling RL beyond tabular methods to real-world problems.*

---

## Overview

Every method we have studied so far — Dynamic Programming, Monte Carlo, and Temporal
Difference learning — stores values in a **table** with one entry per state (or
state-action pair). This works beautifully when the state space is small and discrete,
but it collapses spectacularly when confronted with the vast, continuous state spaces
that dominate real-world problems. Function approximation is the bridge that carries
reinforcement learning from toy grid-worlds to robotics, game-playing, and autonomous
driving.

```
 ╔══════════════════════════════════════════════════════════════════════╗
 ║              WHY FUNCTION APPROXIMATION?                            ║
 ╠══════════════════════════════════════════════════════════════════════╣
 ║  • Real-world state spaces are enormous or continuous               ║
 ║  • Tabular methods cannot store, visit, or generalise over them     ║
 ║  • Function approximation learns a compact parameterised model      ║
 ║    V̂(s, w) ≈ V^π(s)  with  |w| << |S|                             ║
 ║  • Enables generalisation: updating one state improves estimates    ║
 ║    for similar states automatically                                 ║
 ╚══════════════════════════════════════════════════════════════════════╝
```

---

## 1. The Curse of Dimensionality

The term *curse of dimensionality* was coined by Richard Bellman — the same Bellman
behind the Bellman equations. It refers to the phenomenon whereby the number of states
grows **exponentially** with the number of dimensions describing the state.

### 1.1 Exponential Growth

Consider a state described by $d$ continuous variables, each discretised into $n$ bins.
The total number of states is:

$$|S| = n^d$$

Even with a modest resolution of $n = 10$ bins per dimension:

| Dimensions $d$ | States $n^d$ ($n{=}10$) | Memory (8 bytes/entry) |
|:--------------:|:-----------------------:|:----------------------:|
| 1              | $10$                    | 80 B                   |
| 2              | $100$                   | 800 B                  |
| 3              | $1{,}000$               | 8 KB                   |
| 6              | $10^{6}$                | 8 MB                   |
| 10             | $10^{10}$               | 80 GB                  |
| 20             | $10^{20}$               | 800 Exabytes           |
| 50             | $10^{50}$               | More than atoms in Earth|

By the time we reach 20 dimensions — common in robotics — the table is larger than
all storage ever manufactured. And 50 dimensions is routine in modern control systems.

### 1.2 Mathematical Formulation

For a general state vector $\mathbf{s} = (s^{(1)}, s^{(2)}, \dots, s^{(d)})$ with each
component taking values in a continuous range $[a_i, b_i]$:

$$|S| = \prod_{i=1}^{d} \left\lfloor \frac{b_i - a_i}{\Delta_i} \right\rfloor$$

where $\Delta_i$ is the discretisation resolution for dimension $i$. As $\Delta_i \to 0$
(finer resolution), $|S| \to \infty$.

For continuous state spaces: $S \subseteq \mathbb{R}^d$, so $|S|$ is **uncountably
infinite** — a table literally cannot represent it.

---

## 2. Real-World State Spaces

To appreciate the scale of the problem, consider concrete examples:

### 2.1 State Space Sizes Across Domains

| Problem                | State Dimensions   | Approx. State Space Size  | Tabular Feasible? |
|------------------------|--------------------|---------------------------|:-----------------:|
| Tic-Tac-Toe            | 9 cells × 3 values| $\approx 5{,}478$         | ✅ Yes            |
| 4×4 Grid World         | position (x, y)   | $16$                      | ✅ Yes            |
| Blackjack              | 3 variables        | $\approx 200$             | ✅ Yes            |
| Backgammon             | board config.      | $\approx 10^{20}$         | ❌ No             |
| Chess                  | board config.      | $\approx 10^{47}$         | ❌ No             |
| Go (19×19)             | board config.      | $\approx 10^{170}$        | ❌ No             |
| Mountain Car           | 2 continuous       | $\infty$ (continuous)      | ❌ No             |
| Cart-Pole              | 4 continuous       | $\infty$ (continuous)      | ❌ No             |
| Robotic Arm (7-DOF)    | 14 (pos + vel)     | $\infty$ (continuous)      | ❌ No             |
| Autonomous Driving     | 100+ sensors       | $\infty$ (continuous)      | ❌ No             |
| Atari (raw pixels)     | 210×160×3          | $256^{100{,}800}$          | ❌ No             |

### 2.2 Concrete Examples

**Robotics — 7-DOF Arm:** Each joint has a position $\theta_i$ and velocity
$\dot{\theta}_i$, giving a 14-dimensional continuous state space. Discretising each
into just 100 bins yields $100^{14} = 10^{28}$ states.

**Autonomous Driving:** The state includes vehicle position, velocity, heading, plus
positions and velocities of surrounding vehicles, pedestrians, lane geometry, traffic
signals — easily 100+ continuous dimensions.

**Atari Games (from pixels):** A single frame is $210 \times 160 \times 3 = 100{,}800$
pixel values, each in $[0, 255]$. The raw state space has $256^{100{,}800}$ states —
a number so large it dwarfs the number of atoms in the observable universe.

---

## 3. Why Tabular Methods Fail

Tabular methods maintain a lookup table $V[s]$ or $Q[s, a]$ with one entry per state
(or state-action pair). They fail in large or continuous state spaces for three
fundamental reasons:

### 3.1 Memory

Storing $10^{20}$ floating-point entries requires roughly $10^{20} \times 8$ bytes =
$800$ Exabytes. For continuous spaces, the table is infinitely large.

### 3.2 Data Efficiency

Even if we could store the table, the agent must visit **every state** at least once to
learn its value. With $10^{20}$ states and $10^6$ updates per second, this would take:

$$\frac{10^{20}}{10^6} = 10^{14} \text{ seconds} \approx 3.2 \text{ million years}$$

In practice, most states are visited **zero** times.

### 3.3 No Generalisation

This is the most critical failure. In a tabular method, updating $V[s_1]$ tells us
**absolutely nothing** about $V[s_2]$, even if $s_1$ and $s_2$ are nearly identical.

```
  ┌────────────────────────────────────────────────────────────────────┐
  │                TABULAR: NO GENERALISATION                         │
  │                                                                   │
  │  State s₁ = (position: 2.51, velocity: 0.03)  →  V[s₁] = -45.2  │
  │  State s₂ = (position: 2.52, velocity: 0.03)  →  V[s₂] = ???    │
  │                                                                   │
  │  These states are almost identical, but the tabular method        │
  │  treats them as completely independent entries.                   │
  │                                                                   │
  │  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐          │
  │  │  s₁     │   │  s₂     │   │  s₃     │   │  s₄     │          │
  │  │ V=-45.2 │   │ V=???   │   │ V=???   │   │ V=???   │          │
  │  │ visited │   │ never   │   │ never   │   │ never   │          │
  │  └─────────┘   └─────────┘   └─────────┘   └─────────┘          │
  │      ↑                                                            │
  │      └── Only THIS entry was updated. Nothing else changed.      │
  └────────────────────────────────────────────────────────────────────┘
```

---

## 4. Generalisation — The Key Insight

The central motivation for function approximation is **generalisation**: the ability
to transfer knowledge from visited states to unvisited ones.

### 4.1 Similar States Should Have Similar Values

Consider a robot navigating a room. If the robot learns that being 1 metre from a wall
is bad, it should *automatically* infer that being 1.01 metres from the wall is also
(approximately) bad — without needing a separate experience at that exact distance.

$$s_1 \approx s_2 \implies V^\pi(s_1) \approx V^\pi(s_2)$$

This is a **smoothness assumption** on the value function: small changes in state lead
to small changes in value. Most real-world problems exhibit this structure.

### 4.2 How Generalisation Works

With function approximation, the value function is parameterised by a weight vector
$\mathbf{w}$. Updating $\mathbf{w}$ after visiting state $s_1$ changes the estimated
value for **all** states, with nearby states affected more than distant ones:

```
  ┌────────────────────────────────────────────────────────────────────┐
  │           FUNCTION APPROXIMATION: GENERALISATION                  │
  │                                                                   │
  │  Update at s₁ propagates to similar states:                       │
  │                                                                   │
  │  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐          │
  │  │  s₁     │   │  s₂     │   │  s₃     │   │  s₄     │          │
  │  │ V=-45.2 │   │ V≈-44.9 │   │ V≈-43.1 │   │ V≈-20.5 │          │
  │  │ visited │   │ inferred│   │ inferred│   │ inferred│          │
  │  └────┬────┘   └────┬────┘   └────┬────┘   └────┬────┘          │
  │       │              │              │              │               │
  │       └──────────────┴──────────────┴──────────────┘               │
  │                  All share parameter vector w                      │
  │                                                                   │
  │  Effect of update at s₁:                                          │
  │                                                                   │
  │  ΔV   ▲                                                           │
  │       │  *                                                        │
  │       │   * *                                                     │
  │       │      * * *                                                │
  │       │            * * * * *                                      │
  │       │                      * * * * * * *                        │
  │       └──────────────────────────────────────▶ distance from s₁   │
  │       s₁    s₂      s₃               s₄                          │
  └────────────────────────────────────────────────────────────────────┘
```

---

## 5. The Function Approximation Framework

### 5.1 Core Idea

Instead of a table, we approximate the value function using a **parameterised function**:

$$\hat{V}(s, \mathbf{w}) \approx V^\pi(s)$$

where:

- $s \in S$ is the state
- $\mathbf{w} \in \mathbb{R}^d$ is a **parameter vector** (also called weight vector)
- $\hat{V}$ is our **approximation** of the true value function $V^\pi$

The critical property is:

$$|\mathbf{w}| \ll |S|$$

A compact vector of, say, $d = 1{,}000$ weights represents the value of an infinite
number of states. This is the power of function approximation.

### 5.2 Notation Conventions

| Symbol                        | Meaning                                              |
|-------------------------------|------------------------------------------------------|
| $V^\pi(s)$                    | True value of state $s$ under policy $\pi$           |
| $\hat{V}(s, \mathbf{w})$     | Approximate value (parameterised by $\mathbf{w}$)    |
| $\mathbf{w}$                 | Parameter / weight vector, $\mathbf{w} \in \mathbb{R}^d$ |
| $d$                           | Number of parameters                                  |
| $\mathbf{x}(s)$              | Feature vector for state $s$                          |
| $\hat{Q}(s, a, \mathbf{w})$  | Approximate action-value function                     |

### 5.3 Tabular vs Function Approximation Side by Side

```
  ┌──────────────────────────────┐     ┌──────────────────────────────┐
  │      TABULAR METHOD          │     │   FUNCTION APPROXIMATION     │
  │                              │     │                              │
  │   State    Value             │     │   ┌─────────────────────┐    │
  │  ┌──────┬────────┐           │     │   │                     │    │
  │  │ s₁   │ -12.5  │           │     │   │   V̂(s, w) = wᵀx(s) │    │
  │  │ s₂   │  +3.8  │           │     │   │                     │    │
  │  │ s₃   │  -7.1  │           │     │   │   w = [w₁, w₂, w₃] │    │
  │  │ s₄   │ +15.2  │           │     │   │     3 parameters    │    │
  │  │ ...  │  ...   │           │     │   │     ∞ states        │    │
  │  │ sₙ   │  +1.0  │           │     │   └─────────────────────┘    │
  │  └──────┴────────┘           │     │                              │
  │   n entries for n states     │     │   d parameters for ∞ states  │
  │   |w| = |S| = n             │     │   |w| = d  <<  |S|           │
  │   No generalisation          │     │   Automatic generalisation   │
  │   Exact (given enough data)  │     │   Approximate (always)       │
  └──────────────────────────────┘     └──────────────────────────────┘
```

---

## 6. Types of Function Approximators

The choice of $\hat{V}(s, \mathbf{w})$ determines the expressiveness and trainability
of the approximation. Here are the major families:

### 6.1 Linear Function Approximation

The simplest and most theoretically understood form:

$$\hat{V}(s, \mathbf{w}) = \mathbf{w}^\top \mathbf{x}(s) = \sum_{i=1}^{d} w_i \, x_i(s)$$

where $\mathbf{x}(s) = (x_1(s), x_2(s), \dots, x_d(s))^\top$ is a **feature vector**
that encodes the state. The quality of the approximation depends entirely on the choice
of features.

**Pros:** Convergence guarantees, simple gradient, well-understood theory.
**Cons:** Limited to functions that are linear in the features.

### 6.2 Polynomial Basis

Features are polynomial combinations of state variables:

$$\mathbf{x}(s) = (1, \; s, \; s^2, \; s^3, \; \dots, \; s^k)^\top$$

For multi-dimensional states $s = (s_1, s_2)$, this includes cross-terms:

$$\mathbf{x}(s) = (1, \; s_1, \; s_2, \; s_1 s_2, \; s_1^2, \; s_2^2, \; \dots)^\top$$

### 6.3 Radial Basis Functions (RBFs)

Each feature responds to a specific region of state space:

$$x_i(s) = \exp\!\left( -\frac{\|s - c_i\|^2}{2\sigma_i^2} \right)$$

where $c_i$ is the centre and $\sigma_i$ is the width of the $i$-th basis function.
The value function becomes a weighted sum of Gaussian bumps spread across the state space.

### 6.4 Neural Networks (Deep RL)

A neural network with parameters $\mathbf{w}$ (weights and biases of all layers):

$$\hat{V}(s, \mathbf{w}) = f_L(\dots f_2(f_1(s; \mathbf{w}_1); \mathbf{w}_2) \dots; \mathbf{w}_L)$$

**Pros:** Can approximate any continuous function (universal approximation theorem),
automatic feature learning.
**Cons:** Non-linear optimisation, no convergence guarantees in general, sensitive to
hyperparameters.

### 6.5 Summary of Approximator Types

| Approximator       | Parameters $d$ | Expressiveness | Convergence Guarantee | Feature Engineering |
|--------------------|--------------:|:--------------:|:---------------------:|:-------------------:|
| Linear             | Low           | Low            | ✅ Yes (under conditions) | Required          |
| Polynomial         | Medium        | Medium         | ✅ Yes (linear in features) | Partial           |
| RBF                | Medium–High   | Medium–High    | ✅ Yes (linear in features) | Partial (centres) |
| Neural Network     | High          | Very High      | ❌ No (in general)     | Not required       |
| Decision Tree      | Variable      | Medium         | ❌ No                  | Not required       |

---

## 7. Generalisation vs Discrimination Trade-off

Function approximation introduces a fundamental tension:

### 7.1 The Trade-off

- **Generalisation:** The ability to infer values for states never visited. This is
  the primary motivation for function approximation.
- **Discrimination:** The ability to assign *different* values to states that truly
  require different values.

With too few parameters, the approximator **over-generalises** — it cannot distinguish
states that require different values. With too many parameters, it **over-fits** —
memorising visited states without generalising to new ones.

$$\text{Under-fitting} \xleftarrow{\quad \text{fewer parameters} \quad} \text{Sweet Spot} \xrightarrow{\quad \text{more parameters} \quad} \text{Over-fitting}$$

### 7.2 Bias-Variance in RL Context

| Regime           | Parameters | Generalisation | Discrimination | Risk         |
|------------------|:----------:|:--------------:|:--------------:|:------------:|
| Too simple       | $d \ll |S|$ | Over-generalises | Poor          | High bias    |
| Just right       | Optimal $d$| Good           | Good           | Balanced     |
| Too complex      | $d \to |S|$ | Over-fits      | Excellent      | High variance|
| Tabular (extreme)| $d = |S|$  | None           | Perfect        | Max variance |

---

## 8. From Tabular to Function Approximation

An elegant insight: **tabular methods are a special case of function approximation**.

### 8.1 One-Hot Encoding

For a discrete state space $S = \{s_1, s_2, \dots, s_n\}$, define the feature vector
as a **one-hot encoding**:

$$\mathbf{x}(s_i) = (\underbrace{0, \dots, 0}_{i-1}, \; 1, \; \underbrace{0, \dots, 0}_{n-i})^\top$$

Using linear function approximation:

$$\hat{V}(s_i, \mathbf{w}) = \mathbf{w}^\top \mathbf{x}(s_i) = w_i$$

Each weight $w_i$ corresponds exactly to the table entry $V[s_i]$! The gradient is:

$$\nabla_\mathbf{w} \hat{V}(s_i, \mathbf{w}) = \mathbf{x}(s_i)$$

So the SGD update only modifies $w_i$ — exactly like a tabular update. The tabular
method has $|\mathbf{w}| = |S|$, meaning $d = n$ — no compression and no generalisation.

### 8.2 The Spectrum

```
  ◀─────────────── Increasing Generalisation ──────────────────▶
  ◀─────────────── Decreasing Parameters ──────────────────────▶

  ┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
  │ TABULAR  │    │  COARSE  │    │  LINEAR  │    │ NEURAL   │
  │          │    │  CODING  │    │          │    │ NETWORK  │
  │ d = |S|  │    │ d < |S|  │    │ d << |S| │    │ d << |S| │
  │ No gen.  │    │ Some gen.│    │ Good gen.│    │ Best gen.│
  │ Exact    │    │ Approx.  │    │ Approx.  │    │ Approx.  │
  └──────────┘    └──────────┘    └──────────┘    └──────────┘
       ▲                                               ▲
  Special case                                    Most general
  of FA                                           (Deep RL)
```

---

## 9. The Value Error Objective

With function approximation, we cannot represent $V^\pi$ exactly. We need a formal
objective that defines what "good approximation" means.

### 9.1 Mean Squared Value Error

The **Value Error** (also called Mean Squared Value Error, $\overline{\text{VE}}$) is:

$$\overline{\text{VE}}(\mathbf{w}) = \sum_{s \in S} \mu(s) \left[ V^\pi(s) - \hat{V}(s, \mathbf{w}) \right]^2$$

This is a weighted mean squared error across all states, where the weight $\mu(s)$
captures how "important" each state is.

### 9.2 The State Distribution $\mu(s)$

The weighting $\mu(s) \geq 0$ with $\sum_s \mu(s) = 1$ is a **distribution over
states**. The natural choice in RL is the **on-policy distribution** — the fraction of
time spent in each state while following policy $\pi$.

For **episodic tasks:**

$$\mu(s) = \frac{\eta(s)}{\sum_{s'} \eta(s')}$$

where $\eta(s)$ is the expected number of visits to state $s$ per episode (including
the contribution of start states).

For **continuing tasks:**

$$\mu(s) = \lim_{t \to \infty} P(S_t = s \mid A_{0:t-1} \sim \pi)$$

which is the **stationary distribution** under $\pi$.

### 9.3 Why $\mu(s)$ Matters

Not all states are equally important. States the agent visits frequently should have
accurate values, while states rarely visited matter less:

$$\overline{\text{VE}}(\mathbf{w}) = \mathbb{E}_{S \sim \mu}\!\left[\left(V^\pi(S) - \hat{V}(S, \mathbf{w})\right)^2\right]$$

This means we are minimising the **expected** squared error under the distribution of
states the agent actually encounters.

### 9.4 The Ideal Goal

We seek:

$$\mathbf{w}^* = \arg\min_\mathbf{w} \; \overline{\text{VE}}(\mathbf{w})$$

In practice, we approximate this using **stochastic gradient descent (SGD)**. On each
step, the agent visits a state $s_t$, observes (or estimates) a target value $v_t$,
and updates:

$$\mathbf{w}_{t+1} = \mathbf{w}_t + \alpha \left[ v_t - \hat{V}(s_t, \mathbf{w}_t) \right] \nabla_\mathbf{w} \hat{V}(s_t, \mathbf{w}_t)$$

where $\alpha > 0$ is the step size. Because the agent samples states according to
$\mu(s)$ (by following $\pi$), this SGD naturally minimises $\overline{\text{VE}}$.

---

## 10. Practical Motivation — Mountain Car Example

The **Mountain Car** environment is a classic RL problem with continuous state space
that illustrates why function approximation is necessary.

### 10.1 Problem Description

A car is stuck at the bottom of a valley. Its engine is too weak to drive directly up
the hill — it must learn to rock back and forth to build momentum.

```
  ┌──────────────────────────────────────────────────────────────┐
  │                   MOUNTAIN CAR                               │
  │                                                              │
  │                                              🏁 Goal         │
  │         ╱                                   ╱                │
  │        ╱                                  ╱                  │
  │       ╱                                 ╱                    │
  │      ╱                                ╱                      │
  │     ╱                               ╱                        │
  │    ╱                              ╱                          │
  │   ╱           🚗               ╱                             │
  │  ╱            car            ╱                               │
  │ ╱  ╲                      ╱                                  │
  │╱    ╲                   ╱                                    │
  │      ╲               ╱                                      │
  │       ╲            ╱                                         │
  │        ╲─────────╱                                           │
  │         valley bottom                                        │
  │                                                              │
  │   State: s = (position, velocity)                            │
  │   position ∈ [-1.2, 0.6]                                    │
  │   velocity ∈ [-0.07, 0.07]                                  │
  │   Actions: {push left, no push, push right}                  │
  └──────────────────────────────────────────────────────────────┘
```

### 10.2 Why Tabular Fails Here

The state space is $S \subseteq \mathbb{R}^2$ — a 2D continuous space. Any
discretisation loses information and requires arbitrary resolution choices:

```python
import numpy as np

# --- Mountain Car: Tabular vs FA Comparison ---

# State bounds
pos_range = (-1.2, 0.6)     # position
vel_range = (-0.07, 0.07)   # velocity

# Attempt 1: Coarse discretisation
n_bins_coarse = 20
n_states_coarse = n_bins_coarse ** 2
print(f"Coarse (20×20):  {n_states_coarse:>10,} states")

# Attempt 2: Fine discretisation
n_bins_fine = 100
n_states_fine = n_bins_fine ** 2
print(f"Fine   (100×100): {n_states_fine:>10,} states")

# Attempt 3: Very fine discretisation
n_bins_vfine = 1000
n_states_vfine = n_bins_vfine ** 2
print(f"V.Fine (1000²):  {n_states_vfine:>10,} states")

# With function approximation (e.g., linear with tile coding)
n_tilings = 8
tiles_per_tiling = 9 * 9  # 9 tiles per dimension
n_params = n_tilings * tiles_per_tiling
print(f"\nFA (tile coding): {n_params:>10,} parameters")
print(f"Compression ratio: {n_states_fine / n_params:.1f}x")

# Key insight: the value function is smooth!
# Nearby (position, velocity) pairs have similar values.
# FA exploits this structure; tabular methods ignore it.

# Example: two nearly identical states
s1 = np.array([-0.50, 0.02])
s2 = np.array([-0.51, 0.02])
print(f"\nState s1: {s1}")
print(f"State s2: {s2}")
print(f"Distance: {np.linalg.norm(s1 - s2):.4f}")
print("Tabular: treats these as COMPLETELY INDEPENDENT entries")
print("FA:      these share parameters → similar values guaranteed")
```

**Expected output:**

```
Coarse (20×20):         400 states
Fine   (100×100):    10,000 states
V.Fine (1000²):   1,000,000 states

FA (tile coding):       648 parameters
Compression ratio: 15.4x

State s1: [-0.5   0.02]
State s2: [-0.51  0.02]
Distance: 0.0100
Tabular: treats these as COMPLETELY INDEPENDENT entries
FA:      these share parameters → similar values guaranteed
```

### 10.3 What FA Achieves

With just 648 parameters (using tile coding) instead of 10,000+ table entries:

1. **Memory:** 648 × 8 bytes ≈ 5 KB instead of 80 KB+
2. **Learning speed:** Each update improves values for many similar states
3. **Continuous states:** No need to discretise — features handle it naturally
4. **Better performance:** The agent solves Mountain Car in fewer episodes

---

## 11. Summary

```
╔═══════════════════════════════════════════════════════════════════════╗
║  CHAPTER 1 — KEY TAKEAWAYS                                          ║
╠═══════════════════════════════════════════════════════════════════════╣
║  1. State spaces grow exponentially with dimensions (curse of        ║
║     dimensionality) — tabular methods become infeasible              ║
║  2. Real-world RL problems have continuous, high-dimensional         ║
║     state spaces (robotics, games, driving)                          ║
║  3. Tabular methods fail on: memory, data efficiency,                ║
║     and (critically) lack of generalisation                          ║
║  4. Function approximation: V̂(s, w) ≈ V^π(s) with |w| << |S|       ║
║  5. Updating w at one state generalises to similar states            ║
║  6. Types: linear, polynomial, RBF, neural networks                 ║
║  7. Tabular is a special case of FA (one-hot features, d = |S|)     ║
║  8. Objective: minimise VE(w) = Σ μ(s)[V^π(s) - V̂(s,w)]²           ║
║  9. μ(s) = on-policy distribution (states visited under π)          ║
║  10. SGD on sampled states naturally minimises VE                    ║
╚═══════════════════════════════════════════════════════════════════════╝
```

| Concept                          | Key Point                                                      |
|----------------------------------|----------------------------------------------------------------|
| Curse of Dimensionality          | $n^d$ states — exponential growth with dimensions              |
| Continuous State Spaces          | $|S| = \infty$ — tables are impossible                         |
| Generalisation                   | Transfer learning from visited to unvisited states             |
| Function Approximation           | $\hat{V}(s, \mathbf{w}) \approx V^\pi(s)$                     |
| Compact Representation           | $|\mathbf{w}| \ll |S|$ — thousands of params for infinite states|
| Linear FA                        | $\hat{V} = \mathbf{w}^\top \mathbf{x}(s)$ — simple, convergent|
| Neural Network FA                | Deep RL — learns features automatically                        |
| Tabular as Special Case          | One-hot features → $w_i = V[s_i]$, no compression             |
| Value Error $\overline{\text{VE}}$ | $\sum_s \mu(s)[V^\pi(s) - \hat{V}(s, \mathbf{w})]^2$        |
| On-policy Distribution $\mu(s)$ | Weight errors by how often states are visited                  |
| SGD Update                       | $\mathbf{w} \leftarrow \mathbf{w} + \alpha \delta \nabla \hat{V}$ |
| Generalisation vs Discrimination | Too few params → bias; too many → variance                     |

---

## Revision Questions

1. **Explain the curse of dimensionality in the context of RL.** If a state has 15
   continuous dimensions discretised into 10 bins each, how many table entries are
   needed? Is this feasible?

2. **Why is generalisation the most important advantage of function approximation over
   tabular methods?** Give a concrete example using the Mountain Car environment.

3. **Write the function approximation equation and explain each symbol.** What does it
   mean that $|\mathbf{w}| \ll |S|$?

4. **Show that tabular value estimation is a special case of linear function
   approximation.** What feature vector achieves this? What is $d$ relative to $|S|$?

5. **Define the Mean Squared Value Error $\overline{\text{VE}}(\mathbf{w})$.** Why do
   we weight states by $\mu(s)$ rather than weighting them uniformly? What distribution
   does $\mu(s)$ typically represent?

6. **Compare linear function approximation and neural network function approximation.**
   Which has convergence guarantees? Which can learn features automatically? When would
   you choose each one?

---

## Navigation

| ← Previous | ↑ Up | Next → |
|:-----------:|:----:|:------:|
| [Eligibility Traces](../05-Temporal-Difference-Learning/06-eligibility-traces.md) | [Unit 6: Function Approximation](./README.md) | [Linear Function Approximation](./02-linear-function-approximation.md) |
