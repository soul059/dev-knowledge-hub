# Chapter 6: Eligibility Traces and TD(λ)

> **Unit 5: Temporal Difference Learning** — *Unifying TD and MC through a continuous spectrum of algorithms.*

---

## Overview

Eligibility traces are one of the most fundamental mechanisms in reinforcement learning, providing a bridge between TD learning and MC methods. Rather than choosing between one-step bootstrapping of TD(0) and full-return estimation of MC, eligibility traces smoothly interpolate between these extremes through a single parameter $\lambda \in [0, 1]$. The resulting family of algorithms, **TD(λ)**, often achieves better performance than either extreme by combining the low variance of TD with the low bias of MC.

```
╔══════════════════════════════════════════════════════════════════════════╗
║                 ELIGIBILITY TRACES & TD(λ) AT A GLANCE                 ║
╠══════════════════════════════════════════════════════════════════════════╣
║                                                                        ║
║   Idea:  unify TD(0) and MC through a continuous parameter λ ∈ [0,1]  ║
║                                                                        ║
║   Forward view (theoretical):                                          ║
║     G_t^λ = (1−λ) Σ_{n=1}^{T−t−1} λ^{n−1} G_t^{(n)} + λ^{T−t−1}G_t  ║
║                                                                        ║
║   Backward view (practical):                                           ║
║     e_t(s) = γλ e_{t−1}(s) + 𝟙{S_t = s}                              ║
║     V(s) ← V(s) + α δ_t e_t(s)   for all s                           ║
║                                                                        ║
║   λ = 0  →  TD(0):  pure bootstrapping, only current state updated    ║
║   λ = 1  →  TD(1):  equivalent to every-visit MC                      ║
║   0 < λ < 1  →  blend of TD and MC (best of both worlds)              ║
║                                                                        ║
╚══════════════════════════════════════════════════════════════════════════╝
```

The elegance of eligibility traces lies in their dual interpretation: the **forward view** defines what we want to compute (the λ-return), while the **backward view** tells us how to compute it efficiently online. In the offline case, these two views produce identical updates — the **forward-backward equivalence theorem**.

---

## 1. Motivation — Beyond One-Step and Full Returns

In previous chapters, we saw two extremes for estimating value:

| Method | Target | Steps Used | Bias | Variance |
|---|---|---|---|---|
| TD(0) | $R_{t+1} + \gamma V(S_{t+1})$ | 1 | Higher | Lower |
| MC | $G_t = R_{t+1} + \gamma R_{t+2} + \cdots + \gamma^{T-t-1} R_T$ | All | None | Higher |

The **n-step return** generalizes both:

$$G_t^{(n)} = R_{t+1} + \gamma R_{t+2} + \cdots + \gamma^{n-1} R_{t+n} + \gamma^n V(S_{t+n})$$

- When $n = 1$: this is the TD(0) target
- When $n = T - t$: this is the MC return $G_t$ (no bootstrapping)

But which $n$ should we choose? **Don't choose — average over all of them.** This is exactly what the λ-return does.

```
    n-step returns for different n:
    
    n=1:  S_t → R_{t+1} → [V(S_{t+1})]          ← TD(0) target
    n=2:  S_t → R_{t+1} → R_{t+2} → [V(S_{t+2})]
    n=3:  S_t → R_{t+1} → R_{t+2} → R_{t+3} → [V(S_{t+3})]
     ⋮
    n=T-t: S_t → R_{t+1} → R_{t+2} → ⋯ → R_T    ← MC return
    
    λ-return: weighted combination of ALL the above
```

---

## 2. The Forward View — The λ-Return

### 2.1 Definition

The **λ-return** is a weighted average of all possible n-step returns, where the weight on the n-step return is $(1 - \lambda)\lambda^{n-1}$, with the remaining weight going to the full Monte Carlo return:

$$G_t^\lambda = (1 - \lambda) \sum_{n=1}^{T-t-1} \lambda^{n-1} G_t^{(n)} + \lambda^{T-t-1} G_t$$

Where:
- $G_t^{(n)}$ is the n-step return from time $t$
- $G_t = G_t^{(T-t)}$ is the full MC return
- $\lambda \in [0, 1]$ controls the weighting decay
- $(1 - \lambda)$ is a normalization factor ensuring weights sum to 1

### 2.2 Weight Distribution

The weighting scheme assigns geometrically decaying weights to n-step returns:

```
    Weight on G_t^(n):
    
    (1−λ)  │ ██████████
  (1−λ)λ  │ ██████████  ███████
 (1−λ)λ²  │ ██████████  ███████  █████
 (1−λ)λ³  │ ██████████  ███████  █████  ███          λ^{T-t-1}
           │ ██████████  ███████  █████  ███   ···   ██
           ├──────────┬────────┬───────┬─────────────┬──→ n
           │   G^(1)  │ G^(2)  │ G^(3) │    ...      │G_t
           
    Weights sum to 1:  (1−λ)[1 + λ + λ² + ⋯ + λ^{T-t-2}] + λ^{T-t-1} = 1
```

### 2.3 Special Cases

**When $\lambda = 0$:**

$$G_t^\lambda = (1 - 0) \cdot G_t^{(1)} = G_t^{(1)} = R_{t+1} + \gamma V(S_{t+1})$$

All weight is placed on the 1-step return — this is exactly TD(0).

**When $\lambda = 1$:**

$$G_t^\lambda = G_t = R_{t+1} + \gamma R_{t+2} + \cdots + \gamma^{T-t-1} R_T$$

All weight shifts to the full MC return — this is every-visit MC.

### 2.4 The Forward View Update

Using the λ-return as the target:

$$V(S_t) \leftarrow V(S_t) + \alpha \left[ G_t^\lambda - V(S_t) \right]$$

This is theoretically clean but **impractical** for online learning: computing $G_t^\lambda$ requires all future rewards up to episode termination. This is where the backward view comes in.

---

## 3. The Backward View — Eligibility Traces

### 3.1 The Key Idea

Instead of looking forward to compute the λ-return, the backward view maintains a **memory variable** for each state — the **eligibility trace** — that records how recently and frequently each state has been visited. When a TD error occurs, credit is assigned to all states in proportion to their eligibility.

### 3.2 Eligibility Trace Update (Tabular)

For each state $s \in \mathcal{S}$, the **accumulating eligibility trace** is updated at every time step:

$$e_t(s) = \begin{cases} \gamma \lambda \, e_{t-1}(s) + 1 & \text{if } s = S_t \\ \gamma \lambda \, e_{t-1}(s) & \text{if } s \neq S_t \end{cases}$$

Or equivalently using the indicator function:

$$e_t(s) = \gamma \lambda \, e_{t-1}(s) + \mathbb{1}\{S_t = s\}$$

With initialization $e_0(s) = 0$ for all $s$.

### 3.3 Eligibility Trace with Function Approximation

When using a parameterized value function $V(s, \mathbf{w})$, the eligibility trace becomes a vector:

$$\mathbf{e}_t = \gamma \lambda \, \mathbf{e}_{t-1} + \nabla V(S_t, \mathbf{w}_t)$$

### 3.4 Intuition — Trace Decay Over Time

The trace rises sharply when a state is visited and decays exponentially by $\gamma\lambda$ per step:

```
    Eligibility trace e_t(s) over time for a single state s:
    
    e(s)
    1.0 │              ╱╲
    0.8 │            ╱    ╲
        │           ╱      ╲          ╱╲
    0.6 │          ╱        ╲        ╱  ╲
        │  ╱╲     ╱          ╲      ╱    ╲
    0.4 │ ╱  ╲   ╱            ╲    ╱      ╲
        │╱    ╲ ╱              ╲  ╱        ╲
    0.2 │      ╳                ╲╱          ╲
    0.0 │────────╲────────────────────────────╲───→ time
        t₀       t₁        t₂       t₃       t₄
        visit    decay    visit    visit     decay
    
    ► Bumps UP by +1 on visit, decays by γλ each step
```

### 3.5 The TD(λ) Value Update

At each time step $t$, the TD error is computed as usual:

$$\delta_t = R_{t+1} + \gamma V(S_{t+1}) - V(S_t)$$

Then **all** states are updated in proportion to their eligibility:

$$V(s) \leftarrow V(s) + \alpha \, \delta_t \, e_t(s) \quad \text{for all } s \in \mathcal{S}$$

This is the essence of the backward view: a single TD error is distributed backward through the eligibility trace, with recently visited states receiving the most credit.

---

## 4. The λ Spectrum — From TD(0) to MC

The parameter $\lambda$ provides a continuous dial between pure bootstrapping and pure Monte Carlo estimation:

```
    The λ Spectrum
    
    λ = 0                    λ ∈ (0,1)                     λ = 1
    TD(0)                  TD(λ)                          TD(1) ≈ MC
    ┣━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┫
    │                         │                              │
    │  Only current state     │  Blend: recently visited     │  All visited states
    │  is updated             │  states share credit         │  updated equally
    │                         │                              │
    │  e_t(s) = 𝟙{S_t=s}     │  e_t(s) decays by γλ        │  e_t(s) decays by γ only
    │                         │                              │
    │  Pure bootstrapping     │  Partial bootstrapping       │  No bootstrapping
    │  High bias, low var     │  Balanced bias-variance      │  No bias, high var
    │  No trace memory        │  Fading trace memory         │  Full trace memory
    │                         │                              │
    │  Best for:              │  Best for:                   │  Best for:
    │  Markov environments    │  Most practical problems     │  Non-Markov or when
    │  fast online learning   │  tunable performance         │  unbiased estimates needed
    │                         │                              │
    ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
```

### 4.1 TD(0): λ = 0

When $\lambda = 0$, the eligibility trace reduces to $e_t(s) = \mathbb{1}\{S_t = s\}$. Only the current state has a non-zero trace, so the update becomes:

$$V(S_t) \leftarrow V(S_t) + \alpha \, \delta_t$$

This is exactly the standard TD(0) update.

### 4.2 TD(1): λ = 1

When $\lambda = 1$, the trace decays only by $\gamma$ (no additional decay from $\lambda$). Over a complete episode, the accumulated updates are equivalent to the **every-visit Monte Carlo** update:

$$\sum_{t=0}^{T-1} \alpha \, \delta_t \, e_t(s) = \alpha \left[ G_t - V(S_t) \right] \quad \text{(summed over visits to } s \text{)}$$

### 4.3 Intermediate λ

For $0 < \lambda < 1$, TD(λ) blends the strengths of both methods. In practice, $\lambda \approx 0.8\text{–}0.9$ often outperforms both extremes because it propagates credit backward through multiple steps (unlike TD(0)) while maintaining lower variance than full MC returns (unlike TD(1)).

---

## 5. Forward-Backward Equivalence

**Theorem:** In the **offline** case — where updates are accumulated throughout an episode but only applied at episode end — the total update to each state under the backward-view TD(λ) equals the forward-view update:

$$\sum_{t: S_t = s} \alpha \left[ G_t^\lambda - V(S_t) \right] = \sum_{t=0}^{T-1} \alpha \, \delta_t \, e_t(s)$$

### 5.2 Proof Intuition

The proof expands the λ-return as a sum of TD errors weighted by powers of $\gamma\lambda$:

$$G_t^\lambda - V(S_t) = \sum_{k=t}^{T-1} (\gamma\lambda)^{k-t} \delta_k$$

Substituting into the forward-view update and re-indexing the double sum shows that each TD error $\delta_k$ contributes to earlier states through exactly the eligibility trace mechanism.

### 5.3 Online vs Offline

The equivalence is exact only in the **offline** case. In the **online** case, the backward view produces slightly different updates because the changing value function affects subsequent TD errors. However, the difference is typically small and the online backward view often performs comparably.

---

## 6. Accumulating Traces vs Replacing Traces

The standard **accumulating trace** increments by $+1$ on each visit:

$$e_t(s) = \gamma\lambda \, e_{t-1}(s) + \mathbb{1}\{S_t = s\}$$

The trace can grow above 1 if the state is visited repeatedly in quick succession.

The **replacing trace** resets the trace to 1 upon each visit instead of adding:

$$e_t(s) = \begin{cases} 1 & \text{if } s = S_t \\ \gamma \lambda \, e_{t-1}(s) & \text{if } s \neq S_t \end{cases}$$

### Comparison

| Trace Type | Update Rule | Trace Range | Best For |
|---|---|---|---|
| Accumulating | $e(s) = \gamma\lambda \, e(s) + 1$ | $[0, \infty)$ | General use, function approx |
| Replacing | $e(s) = 1$ if visited, else decay | $[0, 1]$ | Tabular, avoids trace explosion |

Replacing traces prevent the trace from growing unboundedly when a state is revisited frequently. Accumulating traces are more common with function approximation because the gradient-based trace update naturally handles this.

---

## 7. Algorithm — Tabular TD(λ) for Prediction

```
Algorithm: Tabular TD(λ) — Policy Evaluation (Backward View)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Input: policy π to evaluate, step size α ∈ (0, 1], trace decay λ ∈ [0, 1]
Initialize: V(s) arbitrarily for all s ∈ S, V(terminal) = 0

Repeat (for each episode):
│
│   Initialize e(s) = 0 for all s ∈ S
│   Initialize S (first state of episode)
│
│   Repeat (for each step of episode, until S is terminal):
│   │
│   │   A ← action given by π for S
│   │   Take action A, observe R, S'
│   │
│   │   δ ← R + γ V(S') − V(S)           // TD error
│   │   e(S) ← e(S) + 1                   // increment trace for current state
│   │                                      // (for replacing traces: e(S) ← 1)
│   │
│   │   For all s ∈ S:                     // update all states
│   │   │   V(s) ← V(s) + α δ e(s)        // scale update by eligibility
│   │   │   e(s) ← γ λ e(s)               // decay all traces
│   │
│   │   S ← S'
```

> **Note:** In practice, only states with non-negligible traces need updating. Many implementations maintain a list of "active" states to avoid iterating over the entire state space.

---

## 8. Python Implementation — TD(λ) on Random Walk

The classic **random walk** environment consists of a chain of states where the agent moves left or right with equal probability.

```python
import numpy as np
import matplotlib.pyplot as plt

# 19-state Random Walk
# States: 0 (left terminal) | 1..19 | 20 (right terminal)
NUM_STATES = 19
START_STATE = 10
LEFT_TERMINAL, RIGHT_TERMINAL = 0, NUM_STATES + 1
GAMMA = 1.0
TRUE_VALUES = np.array([i / (NUM_STATES + 1) * 2 - 1
                        for i in range(1, NUM_STATES + 1)])


def random_walk_episode():
    """Generate a complete episode under the random policy."""
    state = START_STATE
    trajectory = []
    while True:
        next_state = state + np.random.choice([-1, 1])
        if next_state == LEFT_TERMINAL:
            trajectory.append((state, -1.0, next_state)); break
        elif next_state == RIGHT_TERMINAL:
            trajectory.append((state, 1.0, next_state)); break
        else:
            trajectory.append((state, 0.0, next_state))
            state = next_state
    return trajectory


def td_lambda(lam, alpha, num_episodes=10, num_runs=100):
    """Run TD(λ) prediction on the 19-state random walk."""
    errors = np.zeros(num_episodes)

    for run in range(num_runs):
        V = np.zeros(NUM_STATES + 2)

        for ep in range(num_episodes):
            e = np.zeros(NUM_STATES + 2)          # eligibility traces
            trajectory = random_walk_episode()

            for (s, r, s_next) in trajectory:
                delta = r + GAMMA * V[s_next] - V[s]    # TD error
                e[s] += 1                                 # accumulating trace

                V[1:NUM_STATES+1] += alpha * delta * e[1:NUM_STATES+1]
                e *= GAMMA * lam                          # decay all traces

            rmse = np.sqrt(np.mean(
                (V[1:NUM_STATES+1] - TRUE_VALUES) ** 2))
            errors[ep] += rmse

    return errors / num_runs


# Experiment: RMSE vs episodes for different λ values
lambdas = [0.0, 0.4, 0.8, 0.9, 0.95, 1.0]
alpha = 0.02

plt.figure(figsize=(10, 6))
for lam in lambdas:
    errors = td_lambda(lam, alpha, num_episodes=50, num_runs=50)
    plt.plot(errors, label=f'λ = {lam}')

plt.xlabel('Episodes')
plt.ylabel('RMSE (averaged over runs)')
plt.title('TD(λ) on 19-State Random Walk')
plt.legend(); plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('td_lambda_random_walk.png', dpi=150)
plt.show()
```

### Key Observations from the Experiment

1. **TD(0)** ($\lambda = 0$) learns slowly — it only propagates information one step per episode
2. **Intermediate λ** ($\lambda \approx 0.8$) often converges fastest, achieving the best bias-variance trade-off
3. **TD(1)** ($\lambda = 1$) behaves like MC — higher variance causes more noisy learning curves
4. The **optimal λ** depends on the step size $\alpha$ and the problem structure

---

## 9. Effect of λ on Learning Performance

The classic Sutton (1988) experiment showed:

```
    RMSE at end of training vs λ (for different α values):
    
    RMSE
    0.6 │
        │ ╲                                               ╱
    0.5 │  ╲  α = 0.1                                   ╱
        │   ╲                                          ╱
    0.4 │    ╲          ╱──────╲                      ╱  α = 0.04
        │     ╲        ╱        ╲                   ╱
    0.3 │      ╲──────╱          ╲     α = 0.02   ╱
        │                         ╲    ╱────╲   ╱
    0.2 │                          ╲──╱      ╲╱
        │                                  optimal
    0.1 │                                  region
        │
    0.0 ├──────┬──────┬──────┬──────┬──────┬──────→ λ
        0.0   0.2    0.4    0.6    0.8    1.0
              TD(0)                       MC
    
    ► The optimal λ is typically in the range [0.7, 0.95]
    ► Performance degrades at both extremes for most α values
    ► The optimal α also varies with λ
```

**Key findings:**

1. For any given $\alpha$, intermediate $\lambda$ values outperform both extremes
2. The best $(\lambda, \alpha)$ pair outperforms the best pure TD(0) or MC configuration

---

## 10. Mathematical Properties

TD(λ) with tabular representation converges to $v_\pi$ under standard stochastic approximation conditions: (1) $\sum_t \alpha_t = \infty$ and (2) $\sum_t \alpha_t^2 < \infty$.

With linear function approximation $V(s, \mathbf{w}) = \mathbf{w}^\top \mathbf{x}(s)$, TD(λ) converges to a fixed point whose error bound depends on $\lambda$:

$$\| V_{\mathbf{w}} - v_\pi \|_\mu^2 \leq \frac{1 - \gamma\lambda}{1 - \gamma} \min_{\mathbf{w}} \| V_{\mathbf{w}} - v_\pi \|_\mu^2$$

This bound improves (tightens) as $\lambda \to 1$, confirming the bias-variance intuition.

### Computational Complexity

| Operation | Per-step Cost (Tabular) | Per-step Cost (Linear FA) |
|---|---|---|
| TD(0) | $O(1)$ | $O(d)$ |
| TD(λ) | $O(|\mathcal{S}|)$ | $O(d)$ |
| MC | $O(1)$ per step, $O(T)$ at episode end | $O(d)$ per step |

Where $d$ is the feature vector dimension. TD(λ) with function approximation is $O(d)$ per step — same as TD(0) — because the trace vector matches the weight vector dimension.

---

## 11. SARSA(λ) and Q(λ) — Control Extensions

Eligibility traces extend naturally to **control** algorithms by maintaining traces over state-action pairs.

### 11.1 SARSA(λ)

SARSA(λ) applies eligibility traces to the SARSA on-policy control algorithm:

$$\delta_t = R_{t+1} + \gamma Q(S_{t+1}, A_{t+1}) - Q(S_t, A_t)$$

$$e_t(s, a) = \gamma\lambda \, e_{t-1}(s, a) + \mathbb{1}\{S_t = s, A_t = a\}$$

$$Q(s, a) \leftarrow Q(s, a) + \alpha \, \delta_t \, e_t(s, a) \quad \text{for all } s, a$$

### 11.2 Watkins's Q(λ)

Watkins's Q(λ) extends Q-learning with eligibility traces, but **cuts the trace** whenever an exploratory (non-greedy) action is taken, ensuring only greedy trajectory segments propagate credit. This maintains the off-policy nature of Q-learning. An alternative, **Peng's Q(λ)**, does not cut the trace, blending on-policy and off-policy updates at the cost of some bias.

---

## 12. Practical Considerations

### 12.1 Choosing λ

| λ Range | Characteristics | When to Use |
|---|---|---|
| 0.0 | Pure TD, fastest per-step, highest bias | Highly Markov environments, large state spaces |
| 0.3–0.7 | Moderate credit propagation | Environments with short-range dependencies |
| 0.8–0.95 | Strong credit propagation, commonly optimal | Most practical RL problems |
| 1.0 | Equivalent to MC, no bias, highest variance | Strongly non-Markov, need unbiased estimates |

### 12.2 Implementation Tips

1. **Sparse trace updates:** Only update states/features with non-negligible traces
2. **Trace threshold:** Set traces below $\epsilon \approx 10^{-4}$ to zero for efficiency
3. **Replacing traces** often work better in tabular settings with revisited states
4. **Experience replay:** eligibility traces are challenging to combine with replay buffers since traces depend on sequential order

### 12.3 Eligibility Traces in the RL Landscape

```
╔══════════════════════════════════════════════════════════════════╗
║             ELIGIBILITY TRACES IN THE RL LANDSCAPE              ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║   TD(0) ────── n-step TD ────── TD(λ) ────── MC                ║
║     │              │               │            │                ║
║   SARSA ──── n-step SARSA ──── SARSA(λ) ─── MC Control         ║
║     │              │               │            │                ║
║   Q-learn ── n-step Q ──────── Q(λ) ──────── (off-policy MC)   ║
║                                                                  ║
║   ► Traces add a "memory dimension" to any TD method            ║
║   ► λ interpolates between single-step and full-return           ║
║   ► Computationally efficient: same order as base algorithm      ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

---

## Summary

| Concept | Key Point |
|---|---|
| **λ-Return** | $G_t^\lambda = (1-\lambda)\sum_{n=1}^{T-t-1}\lambda^{n-1}G_t^{(n)} + \lambda^{T-t-1}G_t$ — weighted average of all n-step returns |
| **Forward View** | Theoretical: compute λ-return as update target (requires future knowledge) |
| **Backward View** | Practical: maintain eligibility trace $e_t(s)$, distribute TD error to all states |
| **Eligibility Trace** | $e_t(s) = \gamma\lambda\,e_{t-1}(s) + \mathbb{1}\{S_t=s\}$ — memory of recent visits |
| **TD(λ) Update** | $V(s) \leftarrow V(s) + \alpha\,\delta_t\,e_t(s)$ for all $s$ — credit assignment via trace |
| **TD(0)** | $\lambda=0$: only current state updated, pure bootstrapping, higher bias |
| **TD(1)** | $\lambda=1$: equivalent to every-visit MC, no bootstrapping, higher variance |
| **Optimal λ** | Typically $\lambda \in [0.8, 0.95]$ — balances bias and variance |
| **Forward-Backward Equivalence** | Offline forward-view and backward-view produce identical total updates |
| **Accumulating vs Replacing** | Accumulating adds +1 per visit; replacing resets to 1; replacing avoids trace explosion |
| **Convergence Bound** | $\frac{1-\gamma\lambda}{1-\gamma}\min_\mathbf{w}\|V_\mathbf{w}-v_\pi\|^2$ — tighter as $\lambda \to 1$ |
| **SARSA(λ) / Q(λ)** | Control extensions: traces over $(s,a)$ pairs; Q(λ) cuts trace on exploration |

---

## Revision Questions

1. **Define the λ-return $G_t^\lambda$ and show that it reduces to the TD(0) target when $\lambda = 0$ and the MC return when $\lambda = 1$.** Explain why the weights $(1-\lambda)\lambda^{n-1}$ sum to 1 (together with the terminal weight).

2. **Write the eligibility trace update rule for both the accumulating and replacing variants.** Explain intuitively why the eligibility trace can be seen as a "short-term memory" that records which states deserve credit for recent TD errors.

3. **State the forward-backward equivalence theorem.** Under what conditions is this equivalence exact? Why does the backward view produce slightly different updates in the online case?

4. **Explain why intermediate values of $\lambda$ (e.g., 0.8) often outperform both TD(0) and MC.** How does this relate to the bias-variance trade-off? Describe how the optimal $\lambda$ depends on the step size $\alpha$.

5. **In SARSA(λ), how are eligibility traces maintained over state-action pairs?** Why does Watkins's Q(λ) cut the trace when a non-greedy action is taken, and what problem does this solve?

6. **Derive the per-step computational cost of tabular TD(λ) and compare it with TD(0).** Why is TD(λ) with linear function approximation only $O(d)$ per step despite maintaining traces? Describe one practical optimization for reducing the cost in the tabular case.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [TD vs MC vs DP](./05-td-vs-mc-vs-dp.md) | [Unit 5: TD Learning](./README.md) | [Why Function Approximation](../06-Function-Approximation/01-why-function-approximation.md) |
