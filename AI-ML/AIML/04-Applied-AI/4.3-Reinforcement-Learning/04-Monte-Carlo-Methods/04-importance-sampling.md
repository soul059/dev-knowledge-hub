# Chapter 4: Importance Sampling

> **Unit 4: Monte Carlo Methods** — *Correcting for the difference between behavior and target policies.*

---

## Overview

Off-policy learning promises extraordinary power — learning about one policy while following
another — but it introduces a fundamental statistical problem: the data was generated under
the **behavior policy** $b$, yet we need expectations under the **target policy** $\pi$.
The distributions are mismatched. **Importance sampling (IS)** is the mathematical tool that
bridges this gap, re-weighting returns so that averages under $b$ converge to expectations
under $\pi$. Without importance sampling, off-policy Monte Carlo methods would be impossible.

This chapter develops importance sampling from first principles, derives two key estimators —
**ordinary IS** (unbiased, high variance) and **weighted IS** (biased, lower variance) —
analyzes their variance properties, provides incremental implementations suitable for
practical algorithms, and builds a complete off-policy MC prediction algorithm.

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║            IMPORTANCE SAMPLING AT A GLANCE                      ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║  • Problem : Data comes from b, but we want expectations of π  ║
 ║  • Solution: Multiply returns by the IS ratio ρ = ∏ π/b        ║
 ║  • Ordinary IS : Average ρG — unbiased, high (even ∞) variance ║
 ║  • Weighted IS : Weighted average ρG/Σρ — biased, low variance ║
 ║  • Key Req. : Coverage — b(a|s) > 0 wherever π(a|s) > 0       ║
 ║  • Use Case : Off-policy MC prediction and control              ║
 ╚══════════════════════════════════════════════════════════════════╝
```

---

## 1. Why Importance Sampling Is Needed

### 1.1 The Distribution Mismatch Problem

In on-policy Monte Carlo, we estimate $V^\pi(s)$ by averaging returns $G_t$ observed after
visiting $s$ while following $\pi$. By the law of large numbers, this average converges to
the true expectation:

$$V^\pi(s) = \mathbb{E}_\pi[G_t \mid S_t = s]$$

In off-policy learning, the agent follows behavior policy $b$, not $\pi$. The episodes
are sampled from $b$, so the returns $G_t$ reflect the **distribution of trajectories under
$b$**, not under $\pi$. Naively averaging these returns gives us $V^b(s)$, not $V^\pi(s)$.

```
  THE DISTRIBUTION MISMATCH
  ─────────────────────────────────────────────────────────────────

  What we HAVE:                    What we WANT:
  ┌──────────────────────┐         ┌──────────────────────┐
  │  Episodes from b     │         │  Expectation under π │
  │                      │         │                      │
  │  S₀ →ᵇ A₀ → R₁ →   │   ???   │  V^π(s) = E_π[G_t]  │
  │  S₁ →ᵇ A₁ → R₂ →   │ ══════► │                      │
  │  ...                 │         │  But we never follow  │
  │  S_{T-1} →ᵇ A_{T-1} │         │  π directly!         │
  └──────────────────────┘         └──────────────────────┘

  Solution: Re-weight each return by the importance sampling ratio ρ
            so that E_b[ρ · G_t] = E_π[G_t]
```

### 1.2 The Core Idea

Importance sampling is a general statistical technique. Given a function $f(x)$ and two
distributions $p$ and $q$ over $x$, we can express an expectation under $p$ in terms of
samples from $q$:

$$\mathbb{E}_p[f(x)] = \sum_x p(x) f(x) = \sum_x q(x) \frac{p(x)}{q(x)} f(x) = \mathbb{E}_q\!\left[\frac{p(x)}{q(x)} f(x)\right]$$

The ratio $\frac{p(x)}{q(x)}$ is the **importance sampling ratio**. It re-weights each
sample to correct for the fact that it was drawn from $q$ instead of $p$.

> **Key Insight:** By multiplying each sample by the ratio of the target density to the
> proposal density, we transform expectations under the proposal distribution into
> expectations under the target distribution — without ever sampling from the target.

---

## 2. The Importance Sampling Ratio

### 2.1 Trajectory Probability

Consider a trajectory starting from state $S_t$ and continuing to the terminal state at
step $T$:

$$S_t, A_t, R_{t+1}, S_{t+1}, A_{t+1}, R_{t+2}, \ldots, S_{T-1}, A_{T-1}, R_T, S_T$$

The probability of this trajectory under policy $\pi$ depends on two factors at each step:
the **policy** (which selects $A_k$ given $S_k$) and the **environment dynamics** (which
determines $S_{k+1}$ and $R_{k+1}$ given $S_k$ and $A_k$):

$$\Pr\{\text{trajectory} \mid S_t, \pi\} = \prod_{k=t}^{T-1} \pi(A_k \mid S_k) \cdot p(S_{k+1}, R_{k+1} \mid S_k, A_k)$$

Similarly under the behavior policy $b$:

$$\Pr\{\text{trajectory} \mid S_t, b\} = \prod_{k=t}^{T-1} b(A_k \mid S_k) \cdot p(S_{k+1}, R_{k+1} \mid S_k, A_k)$$

### 2.2 Derivation of the IS Ratio

The importance sampling ratio for the trajectory from $t$ to $T-1$ is:

$$\rho_{t:T-1} = \frac{\Pr\{\text{trajectory} \mid S_t, \pi\}}{\Pr\{\text{trajectory} \mid S_t, b\}} = \frac{\prod_{k=t}^{T-1} \pi(A_k \mid S_k) \cdot p(S_{k+1}, R_{k+1} \mid S_k, A_k)}{\prod_{k=t}^{T-1} b(A_k \mid S_k) \cdot p(S_{k+1}, R_{k+1} \mid S_k, A_k)}$$

The environment dynamics $p(S_{k+1}, R_{k+1} \mid S_k, A_k)$ appear in both numerator and
denominator and **cancel exactly**:

$$\boxed{\rho_{t:T-1} = \prod_{k=t}^{T-1} \frac{\pi(A_k \mid S_k)}{b(A_k \mid S_k)}}$$

> **Critical Result:** The IS ratio depends **only on the two policies and the sequence of
> actions actually taken**, not on the environment dynamics. This is why importance sampling
> works even when we have no model of the environment — the MDP dynamics are irrelevant.

### 2.3 Properties of the IS Ratio

| Property | Value |
|---|---|
| Range | $\rho_{t:T-1} \geq 0$ (always non-negative) |
| Expected value under $b$ | $\mathbb{E}_b[\rho_{t:T-1}] = 1$ |
| When $\pi = b$ | $\rho_{t:T-1} = 1$ (reduces to on-policy) |
| When $\pi(A_k \mid S_k) = 0$ for any $k$ | $\rho_{t:T-1} = 0$ (trajectory impossible under $\pi$) |
| Multiplicative structure | Product of per-step ratios; grows/shrinks exponentially |

### 2.4 The Coverage Assumption

For the IS ratio to be well-defined, we require $b(A_k \mid S_k) > 0$ whenever
$\pi(A_k \mid S_k) > 0$. This is the **coverage assumption**:

$$\pi(a \mid s) > 0 \implies b(a \mid s) > 0 \quad \forall\, s \in \mathcal{S},\; a \in \mathcal{A}(s)$$

If coverage fails, there exist state-action pairs that $\pi$ would visit but $b$ never
explores, making it impossible to estimate $\pi$'s value from $b$'s data alone.

---

## 3. Ordinary Importance Sampling

### 3.1 Definition

Let $\mathcal{T}(s)$ denote the set of time steps at which state $s$ is visited (across all
episodes). For **first-visit** MC, $\mathcal{T}(s)$ includes only the first visit to $s$
in each episode. Let $|\mathcal{T}(s)| = n$. The **ordinary importance sampling** estimator
is the simple average of the IS-weighted returns:

$$\boxed{V_{\text{ordinary}}(s) = \frac{1}{|\mathcal{T}(s)|} \sum_{t \in \mathcal{T}(s)} \rho_{t:T(t)-1}\, G_t = \frac{1}{n} \sum_{i=1}^{n} \rho_i\, G_i}$$

where $T(t)$ is the terminal time of the episode containing time step $t$.

### 3.2 Proof of Unbiasedness

We show that ordinary IS is an unbiased estimator of $V^\pi(s)$. Consider a single
IS-weighted return $\rho_{t:T-1} G_t$ where the trajectory is generated under $b$:

$$\mathbb{E}_b[\rho_{t:T-1}\, G_t \mid S_t = s] = \sum_{\text{traj}} \Pr\{\text{traj} \mid S_t = s, b\} \cdot \frac{\Pr\{\text{traj} \mid S_t = s, \pi\}}{\Pr\{\text{traj} \mid S_t = s, b\}} \cdot G(\text{traj})$$

$$= \sum_{\text{traj}} \Pr\{\text{traj} \mid S_t = s, \pi\} \cdot G(\text{traj}) = \mathbb{E}_\pi[G_t \mid S_t = s] = V^\pi(s)$$

Since each term $\rho_i G_i$ is an unbiased estimate, their average is also unbiased:

$$\mathbb{E}_b[V_{\text{ordinary}}(s)] = \frac{1}{n}\sum_{i=1}^{n} \mathbb{E}_b[\rho_i G_i] = V^\pi(s)$$

### 3.3 Variance Problem

Despite being unbiased, ordinary IS can have **extremely high — even infinite — variance**.
The IS ratio is a product of $T - t$ terms:

$$\rho_{t:T-1} = \prod_{k=t}^{T-1} \frac{\pi(A_k \mid S_k)}{b(A_k \mid S_k)}$$

If any factor $\frac{\pi(A_k \mid S_k)}{b(A_k \mid S_k)}$ is large (e.g., $\pi$ strongly
favors an action that $b$ rarely takes), the product can become enormous. Conversely, if
$b$ frequently takes actions that $\pi$ avoids, the ratio is often near zero. This creates a
distribution with **heavy tails**: most IS-weighted returns are near zero, but occasionally
one is extremely large, dominating the average.

> **Variance Result (Ordinary IS):** For a single IS-weighted return,
> $\text{Var}_b[\rho_{t:T-1} G_t]$ can be **infinite** even when $V^\pi(s)$ is finite,
> because $\mathbb{E}_b[\rho_{t:T-1}^2]$ can diverge.

### 3.4 When Variance Is Infinite

The second moment of the IS ratio satisfies:

$$\mathbb{E}_b[\rho_{t:T-1}^2] = \prod_{k=t}^{T-1} \mathbb{E}_b\!\left[\frac{\pi(A_k \mid S_k)^2}{b(A_k \mid S_k)^2}\right]$$

Each factor $\mathbb{E}_b\!\left[\frac{\pi(A_k \mid S_k)^2}{b(A_k \mid S_k)^2}\right] = \sum_a \frac{\pi(a \mid s)^2}{b(a \mid s)}$,
which can exceed 1 even when $\mathbb{E}_b[\rho] = 1$. Since the product of terms $> 1$
grows exponentially in $T - t$, the variance grows **exponentially with episode length**.

---

## 4. Weighted Importance Sampling

### 4.1 Definition

**Weighted importance sampling** normalizes the IS-weighted returns by the sum of the
IS ratios, producing a **weighted average** rather than a simple average:

$$\boxed{V_{\text{weighted}}(s) = \frac{\sum_{t \in \mathcal{T}(s)} \rho_{t:T(t)-1}\, G_t}{\sum_{t \in \mathcal{T}(s)} \rho_{t:T(t)-1}} = \frac{\sum_{i=1}^{n} \rho_i\, G_i}{\sum_{i=1}^{n} \rho_i}}$$

If the denominator is zero (all ratios are zero), we define $V_{\text{weighted}}(s) = 0$.

### 4.2 Bias Analysis

Weighted IS is a **ratio estimator** — a ratio of two random variables. The ratio of
expectations equals $V^\pi(s)$:

$$\frac{\mathbb{E}_b[\rho_{t:T-1} G_t]}{\mathbb{E}_b[\rho_{t:T-1}]} = \frac{V^\pi(s)}{1} = V^\pi(s)$$

However, $\mathbb{E}[X/Y] \neq \mathbb{E}[X]/\mathbb{E}[Y]$ in general (Jensen's
inequality), so weighted IS is **biased**. The bias is $O(1/n)$ and vanishes
asymptotically — weighted IS is **consistent** and **asymptotically unbiased**.

For first-visit MC, with only a **single observation** ($n = 1$), weighted IS gives:

$$V_{\text{weighted}}(s) = \frac{\rho_1 G_1}{\rho_1} = G_1$$

The IS ratio cancels entirely! The estimate is simply the observed return, which has
expected value $V^b(s)$, not $V^\pi(s)$. This confirms the bias for small $n$. By contrast,
ordinary IS gives $\rho_1 G_1$, which is unbiased for any $n$.

### 4.3 Variance Advantage

The critical advantage of weighted IS is **bounded variance**. Since the weights
$w_i = \rho_i / \sum_j \rho_j$ sum to 1, the weighted average is a **convex combination**
of the observed returns $G_i$. Therefore:

$$\min_i G_i \leq V_{\text{weighted}}(s) \leq \max_i G_i$$

The estimate is always bounded by the range of observed returns, regardless of how extreme
the IS ratios become. This is a dramatic improvement over ordinary IS, whose estimates can
be arbitrarily large.

> **Variance Result (Weighted IS):** The variance of weighted IS is bounded and converges
> at rate $O(1/n)$. It cannot produce estimates outside the range of observed returns.

---

## 5. Ordinary vs Weighted IS: Detailed Comparison

```
  BIAS–VARIANCE TRADE-OFF
  ─────────────────────────────────────────────────────────────────

  Error
  │
  │  ╲   Ordinary IS (high variance, zero bias)
  │   ╲
  │    ╲                    ╱ Weighted IS (low variance, small bias)
  │     ╲                 ╱
  │      ╲              ╱
  │       ╲           ╱
  │        ╲        ╱
  │         ╲     ╱
  │          ╲  ╱
  │           ╳       ← Crossover point
  │         ╱  ╲
  │       ╱     ╲───────────────────────  Weighted IS MSE
  │     ╱
  │   ╱
  │  ╱──────────────────────────────────  Ordinary IS MSE (if finite)
  │
  └──────────────────────────────────────────────────► n (episodes)
        Few                              Many

  • For small n : Weighted IS wins (lower MSE despite bias)
  • For large n : Both converge, but ordinary IS has no bias
  • In practice : Weighted IS almost always preferred
```

| Property | Ordinary IS | Weighted IS |
|---|---|---|
| **Bias** | Unbiased (zero bias) | Biased ($O(1/n)$, vanishes asymptotically) |
| **Variance** | Can be infinite; grows exponentially with horizon | Always bounded by range of returns |
| **MSE for small $n$** | Often very high due to variance | Low; bias is small |
| **MSE for large $n$** | Converges (if variance is finite) | Converges; often faster in practice |
| **Single observation** | $\rho G$ (unbiased, high variance) | $G$ (biased toward $V^b$, zero variance) |
| **Sensitivity to outliers** | Highly sensitive to extreme $\rho$ values | Self-normalizing; robust to extremes |
| **Convergence rate** | $O(1/n)$ if variance is finite | $O(1/n)$ |
| **Practical preference** | Rarely used alone | Strongly preferred in practice |

---

## 6. Incremental Implementation of Weighted IS

### 6.1 Motivation

Storing all returns and IS ratios to compute the weighted average at the end is memory-
inefficient. We want an **incremental update rule** that processes one return at a time.

### 6.2 Derivation

Let $C_n = \sum_{i=1}^{n} W_i$ be the cumulative sum of weights (where $W_i = \rho_i$),
and let $V_n$ be the weighted IS estimate after $n$ returns. Then:

$$V_n = \frac{\sum_{i=1}^{n} W_i G_i}{\sum_{i=1}^{n} W_i} = \frac{\sum_{i=1}^{n} W_i G_i}{C_n}$$

After observing the $(n+1)$-th return:

$$V_{n+1} = \frac{\sum_{i=1}^{n+1} W_i G_i}{C_{n+1}} = \frac{C_n V_n + W_{n+1} G_{n+1}}{C_n + W_{n+1}}$$

$$= \frac{C_n V_n + W_{n+1} G_{n+1}}{C_{n+1}}$$

$$= V_n + \frac{W_{n+1}}{C_{n+1}}\left(G_{n+1} - V_n\right)$$

$$\boxed{V_{n+1} = V_n + \frac{W_{n+1}}{C_{n+1}}\Big(G_{n+1} - V_n\Big), \qquad C_{n+1} = C_n + W_{n+1}}$$

This has the same form as the standard incremental mean update, but with the step size
$\frac{W_{n+1}}{C_{n+1}}$ replacing $\frac{1}{n+1}$. The step size naturally decreases as
more data accumulates and is proportional to the current weight.

### 6.3 Comparison with Standard Incremental Mean

| Standard Mean | Weighted IS |
|---|---|
| $\mu_{n+1} = \mu_n + \frac{1}{n+1}(x_{n+1} - \mu_n)$ | $V_{n+1} = V_n + \frac{W_{n+1}}{C_{n+1}}(G_{n+1} - V_n)$ |
| Step size: $\frac{1}{n+1}$ (fixed schedule) | Step size: $\frac{W_{n+1}}{C_{n+1}}$ (data-dependent) |
| Equal weight to all samples | Weight proportional to IS ratio |

---

## 7. Off-Policy MC Prediction Algorithm

### 7.1 Complete Pseudocode

```
Algorithm: Off-Policy MC Prediction (Weighted Importance Sampling)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Input:  Target policy π, Behavior policy b (with coverage of π)
Output: Value function V ≈ V^π

Initialize:
    V(s) ← arbitrary, for all s ∈ S
    C(s) ← 0,         for all s ∈ S

Repeat forever (for each episode):
│
│  Generate episode using b:
│      S₀, A₀, R₁, S₁, A₁, R₂, ..., S_{T-1}, A_{T-1}, R_T
│
│  G ← 0                          // Return accumulator
│  W ← 1                          // Importance sampling weight
│
│  For t = T-1, T-2, ..., 0:      // Loop backward through episode
│  │
│  │  G ← γ · G + R_{t+1}         // Accumulate discounted return
│  │  C(S_t) ← C(S_t) + W         // Update cumulative weight
│  │  V(S_t) ← V(S_t) + (W / C(S_t)) · (G - V(S_t))  // Update value
│  │  W ← W · π(A_t | S_t) / b(A_t | S_t)             // Update IS weight
│  │
│  │  If W = 0:                    // π would never take this action
│  │      break                    // Earlier steps contribute nothing
│  │
│  └──
└──
```

### 7.2 Key Implementation Details

1. **Backward iteration:** We process the episode backwards to efficiently accumulate both
   $G_t$ and $\rho_{t:T-1}$ using the recursive relationships $G_t = R_{t+1} + \gamma G_{t+1}$
   and $\rho_{t:T-1} = \frac{\pi(A_t \mid S_t)}{b(A_t \mid S_t)} \cdot \rho_{t+1:T-1}$.

2. **Early termination:** If $\pi(A_t \mid S_t) = 0$ at any step, then $W$ becomes zero
   and all preceding steps have $\rho = 0$. We can break out of the inner loop early, saving
   computation.

3. **Weight accumulation order:** Note that $W$ is updated **after** the value update for
   step $t$. At the time of the value update, $W = \rho_{t+1:T-1}$ (not including step $t$
   itself). This is intentional for weighted IS — see Sutton & Barto for the subtle reason
   this gives the correct weighted estimator.

4. **First-visit variant:** To implement first-visit IS, track which states have been
   visited in the current episode and skip updates for revisited states.

---

## 8. Python Implementation

### 8.1 Both Estimators Compared

```python
import numpy as np
from collections import defaultdict


def generate_episode(env, behavior_policy):
    """Generate an episode following the behavior policy."""
    episode = []
    state = env.reset()
    done = False
    while not done:
        action = behavior_policy(state)
        next_state, reward, done = env.step(action)
        episode.append((state, action, reward))
        state = next_state
    return episode


def importance_sampling_ratio(episode, target_policy, behavior_policy):
    """Compute the IS ratio for a full trajectory starting at each time step."""
    T = len(episode)
    ratios = np.ones(T)

    # Build per-step ratios
    per_step = np.array([
        target_policy(s, a) / behavior_policy(s, a)
        for s, a, _ in episode
    ])

    # ρ_{t:T-1} = product of per-step ratios from t to T-1
    for t in range(T):
        ratios[t] = np.prod(per_step[t:])

    return ratios


def off_policy_mc_prediction(
    env,
    target_policy,
    behavior_policy,
    num_episodes,
    gamma=1.0,
):
    """
    Off-policy MC prediction using both ordinary and weighted IS.

    Returns two value dictionaries: V_ordinary, V_weighted.
    """
    # Storage for ordinary IS
    returns_sum = defaultdict(float)   # Σ ρ_i G_i
    count = defaultdict(int)           # n(s)

    # Storage for weighted IS (incremental)
    V_weighted = defaultdict(float)
    C = defaultdict(float)             # cumulative weights

    for _ in range(num_episodes):
        episode = generate_episode(env, behavior_policy)
        T = len(episode)

        G = 0.0
        W = 1.0

        # Process episode backwards
        for t in range(T - 1, -1, -1):
            state, action, reward = episode[t]
            G = gamma * G + reward

            # --- Ordinary IS update ---
            rho_full = W  # W accumulates ρ_{t:T-1} as we go backward
            returns_sum[state] += rho_full * G
            count[state] += 1

            # --- Weighted IS update (incremental) ---
            C[state] += W
            if C[state] != 0:
                V_weighted[state] += (W / C[state]) * (G - V_weighted[state])

            # Update IS weight for the next (earlier) time step
            pi_prob = target_policy(state, action)
            b_prob = behavior_policy(state, action)

            if b_prob == 0:
                break
            W *= pi_prob / b_prob

            if W == 0:
                break  # π would never take this action

    # Compute ordinary IS estimates
    V_ordinary = {
        s: returns_sum[s] / count[s] if count[s] > 0 else 0.0
        for s in set(list(returns_sum.keys()) + list(count.keys()))
    }

    return V_ordinary, dict(V_weighted)
```

### 8.2 Example: Estimating a Simple Blackjack State Value

```python
def run_blackjack_example():
    """
    Demonstrate IS on a simplified scenario.

    Target policy π: stick on 20 or 21, hit otherwise (deterministic).
    Behavior policy b: random (hit or stick with equal probability).
    """
    # Target policy: deterministic — π(hit|s)=1 if sum<20, π(stick|s)=1 if sum≥20
    def target_policy(state, action=None):
        player_sum = state[0]
        preferred = 0 if player_sum < 20 else 1  # 0=hit, 1=stick
        if action is None:
            return preferred
        return 1.0 if action == preferred else 0.0

    # Behavior policy: random — b(hit|s) = b(stick|s) = 0.5
    def behavior_policy(state, action=None):
        if action is None:
            return np.random.choice([0, 1])
        return 0.5

    # Run both estimators
    V_ord, V_wt = off_policy_mc_prediction(
        env=BlackjackEnv(),         # a gym-like environment
        target_policy=target_policy,
        behavior_policy=behavior_policy,
        num_episodes=10000,
        gamma=1.0,
    )

    print("Ordinary IS estimates (sample):")
    for s in list(V_ord.keys())[:5]:
        print(f"  V({s}) = {V_ord[s]:.4f}")

    print("\nWeighted IS estimates (sample):")
    for s in list(V_wt.keys())[:5]:
        print(f"  V({s}) = {V_wt[s]:.4f}")
```

---

## 9. Variance Analysis: A Deeper Look

### 9.1 Second Moment of the IS Ratio

To understand variance, we compute $\mathbb{E}_b[\rho_{t:T-1}^2]$. Assuming actions at
different time steps are conditionally independent given the states:

$$\mathbb{E}_b[\rho_{t:T-1}^2] = \mathbb{E}_b\!\left[\prod_{k=t}^{T-1}\frac{\pi(A_k \mid S_k)^2}{b(A_k \mid S_k)^2}\right]$$

For a fixed state sequence (conditioning on $S_t, S_{t+1}, \ldots, S_{T-1}$):

$$= \prod_{k=t}^{T-1} \sum_{a} b(a \mid S_k) \cdot \frac{\pi(a \mid S_k)^2}{b(a \mid S_k)^2} = \prod_{k=t}^{T-1} \sum_{a} \frac{\pi(a \mid S_k)^2}{b(a \mid S_k)}$$

### 9.2 Numerical Example

Consider a two-action problem at each step with $\pi(a_1 \mid s) = 0.9$, $b(a_1 \mid s) = 0.5$:

$$\sum_a \frac{\pi(a \mid s)^2}{b(a \mid s)} = \frac{0.9^2}{0.5} + \frac{0.1^2}{0.5} = \frac{0.81 + 0.01}{0.5} = 1.64$$

For a trajectory of length $H$:

$$\mathbb{E}_b[\rho^2] = 1.64^H$$

| Horizon $H$ | $\mathbb{E}_b[\rho^2]$ | $\text{Var}_b[\rho] \approx$ |
|---|---|---|
| 1 | 1.64 | 0.64 |
| 5 | 11.9 | 10.9 |
| 10 | 141.4 | 140.4 |
| 20 | 19,987 | 19,986 |
| 50 | $5.6 \times 10^{10}$ | $5.6 \times 10^{10}$ |

The variance grows **exponentially** with horizon length, making ordinary IS impractical
for long episodes.

### 9.3 Why Weighted IS Controls Variance

Weighted IS avoids the explosion because it normalizes by $\sum_i \rho_i$. When one $\rho_i$
is extremely large, it dominates both the numerator and denominator, and the estimate stays
near $G_i$ for that particular sample rather than blowing up. Formally, the effective
sample size of weighted IS is:

$$n_{\text{eff}} = \frac{\left(\sum_{i=1}^n \rho_i\right)^2}{\sum_{i=1}^n \rho_i^2}$$

This measures how many "equally weighted" samples the weighted IS estimate is effectively
based on. When $n_{\text{eff}}$ is small, the estimate is dominated by a few high-weight
samples, but the estimate itself remains bounded.

---

## 10. Practical Considerations

### 10.1 When to Use Each Estimator

| Scenario | Recommended Estimator | Reason |
|---|---|---|
| Short episodes, $\pi \approx b$ | Either (both work well) | IS ratios are moderate |
| Long episodes, $\pi \neq b$ | Weighted IS | Ordinary IS variance explodes |
| Need strict unbiasedness guarantee | Ordinary IS | Weighted IS has $O(1/n)$ bias |
| Single or very few episodes | Weighted IS | More stable; avoids outlier-driven estimates |
| Combining with function approximation | Weighted IS | Gradient-based methods prefer bounded updates |

### 10.2 Improving IS in Practice

1. **Per-decision importance sampling:** Instead of weighting the full return $G_t$,
   decompose $G_t = R_{t+1} + \gamma R_{t+2} + \cdots$ and weight each reward term by
   only the ratios up to that step. This reduces variance by avoiding multiplication of
   ratios beyond what each reward requires.

2. **Truncated IS ratios:** Cap $\rho$ at some maximum value $\bar{\rho}$ to prevent
   extreme weights. This introduces bias but dramatically reduces variance.

3. **Adaptive behavior policies:** Design $b$ to be close to $\pi$ while maintaining
   coverage. The closer $b$ is to $\pi$, the closer the IS ratios are to 1, reducing
   variance.

4. **Multiple importance sampling:** When data comes from multiple behavior policies
   $b_1, b_2, \ldots$, use the **balance heuristic** to optimally combine estimates.

### 10.3 Connection to Other Methods

```
  IMPORTANCE SAMPLING IN THE RL LANDSCAPE
  ─────────────────────────────────────────────────────────────────

  ┌─────────────────────────┐
  │   Off-Policy Methods    │
  │                         │
  │  ┌───────────────────┐  │
  │  │ Importance         │  │     ┌──────────────────────────┐
  │  │ Sampling          │──┼────►│ MC Off-Policy Prediction │
  │  │                   │  │     └──────────────────────────┘
  │  │ • Ordinary IS     │  │
  │  │ • Weighted IS     │  │     ┌──────────────────────────┐
  │  │ • Per-decision IS │──┼────►│ MC Off-Policy Control    │
  │  │                   │  │     └──────────────────────────┘
  │  └───────────────────┘  │
  │                         │     ┌──────────────────────────┐
  │  ┌───────────────────┐  │     │ Off-Policy TD            │
  │  │ Other Corrections │──┼────►│ (IS for TD targets)      │
  │  │ • Q-learning      │  │     └──────────────────────────┘
  │  │   (implicit IS)   │  │
  │  └───────────────────┘  │     ┌──────────────────────────┐
  │                         │     │ Policy Gradient           │
  └─────────────────────────┘────►│ (IS for off-policy PG)   │
                                  └──────────────────────────┘
```

---

## Summary

| Concept | Key Point |
|---|---|
| Distribution mismatch | Data from $b$ cannot directly estimate $V^\pi$ without correction |
| IS ratio $\rho_{t:T-1}$ | $\prod_{k=t}^{T-1} \pi(A_k \mid S_k) / b(A_k \mid S_k)$; depends only on policies, not dynamics |
| Coverage assumption | $\pi(a \mid s) > 0 \implies b(a \mid s) > 0$; required for IS to be valid |
| Ordinary IS | $V(s) = \frac{1}{n}\sum \rho_i G_i$ — unbiased but potentially infinite variance |
| Weighted IS | $V(s) = \sum \rho_i G_i / \sum \rho_i$ — biased ($O(1/n)$) but bounded variance |
| Variance growth | Ordinary IS variance grows exponentially with episode length |
| Weighted IS bound | Estimates always lie within the range of observed returns |
| Incremental update | $V \leftarrow V + \frac{W}{C}(G - V)$, $C \leftarrow C + W$ |
| Early termination | If $\pi(A_t \mid S_t) = 0$, all earlier steps contribute $\rho = 0$ |
| Per-decision IS | Weight each reward by only the ratios relevant to it; reduces variance |
| Practical choice | Weighted IS preferred in nearly all practical settings |

---

## Revision Questions

1. **Derive the importance sampling ratio from first principles.** Start with the
   probability of a trajectory under $\pi$ and under $b$, and show that the environment
   dynamics cancel. Why is this cancellation critical for model-free methods?

2. **Prove that ordinary IS is unbiased.** Show that $\mathbb{E}_b[\rho_{t:T-1} G_t \mid
   S_t = s] = V^\pi(s)$. Then explain why weighted IS is biased and compute the weighted IS
   estimate when only a single episode is available.

3. **Consider a two-action MDP with $\pi(a_1 \mid s) = 1.0$ (deterministic) and
   $b(a_1 \mid s) = 0.1$.** Compute the IS ratio for a single step. If the episode has 10
   steps, compute $\mathbb{E}_b[\rho^2]$ and discuss the practical implications for ordinary
   IS estimation.

4. **Derive the incremental update rule for weighted importance sampling.** Starting from
   $V_n = \sum_{i=1}^n W_i G_i / \sum_{i=1}^n W_i$, show that
   $V_{n+1} = V_n + \frac{W_{n+1}}{C_{n+1}}(G_{n+1} - V_n)$. How does this compare to the
   standard incremental mean update?

5. **In the off-policy MC prediction algorithm, why does the inner loop iterate backward
   through the episode?** Explain how this enables efficient computation of both the return
   $G_t$ and the IS ratio $\rho_{t:T-1}$. Also explain the early termination condition.

6. **A colleague argues that ordinary IS should always be preferred because it is unbiased.**
   Construct a counter-argument using the bias-variance-MSE framework. Under what (rare)
   conditions might ordinary IS actually be preferable?

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [On-Policy vs Off-Policy](./03-on-policy-vs-off-policy.md) | [Unit 4: Monte Carlo Methods](./README.md) | [Every-Visit vs First-Visit](./05-every-visit-vs-first-visit.md) |
