# Chapter 1: Monte Carlo Prediction

> **Unit 4: Monte Carlo Methods** — *Estimating value functions from sampled episodes without a model.*

---

## Overview

Monte Carlo (MC) prediction solves the same **prediction problem** as iterative policy
evaluation — compute $V^\pi(s)$ for a given policy $\pi$ — but takes a fundamentally
different approach. Instead of using the Bellman equation and sweeping through all states
with known transition dynamics, MC prediction **learns from complete episodes** of actual
(or simulated) experience. The agent follows policy $\pi$, collects a full trajectory
from start to termination, computes the return $G_t$ for each visited state, and averages
those returns across many episodes. No model of the environment is needed: no transition
probabilities $P(s' \mid s,a)$ and no reward function $R(s,a,s')$. This makes Monte Carlo
the first **model-free** prediction method in our study.

```
 ╔══════════════════════════════════════════════════════════════════╗
 ║              MC PREDICTION AT A GLANCE                          ║
 ╠══════════════════════════════════════════════════════════════════╣
 ║  • Input : A policy π, access to environment (or simulator)    ║
 ║  • Output: The value function V^π                               ║
 ║  • Method: Average the returns observed after visiting states   ║
 ║  • Type  : Prediction (not control)                             ║
 ║  • Model : Not required (model-free)                            ║
 ║  • Key   : Requires complete episodes (must reach termination)  ║
 ╚══════════════════════════════════════════════════════════════════╝
```

---

## 1. From DP to Monte Carlo: Motivation

Dynamic programming requires a **complete model** of the environment — the full transition
probability distribution $P(s' \mid s,a)$ and the reward function $R(s,a,s')$. In most
real-world problems, this model is unknown. Even when it is known, DP requires sweeping
over the entire state space on every iteration, which is infeasible for large problems
(see [Limitations of DP](../03-Dynamic-Programming/04-limitations-of-dp.md)).

Monte Carlo methods bypass both issues:

| Limitation of DP | How MC Addresses It |
|---|---|
| Requires complete model $P(s' \mid s,a)$ | Learns from sampled experience; no model needed |
| Must sweep all states every iteration | Can focus on states of interest |
| Computational cost of full-width backups | Uses sample backups (single trajectory) |
| Bootstraps (relies on other estimates) | Uses actual returns — no bootstrapping |

> **Key Insight:** MC prediction replaces the Bellman equation's **expected value** (over
> all transitions) with a **sample average** (over observed returns), trading computational
> exactness for model-free generality.

---

## 2. The Return $G_t$

The foundation of MC prediction is the **return** — the total discounted reward from time
step $t$ onward until the episode terminates at step $T$:

$$\boxed{G_t = R_{t+1} + \gamma R_{t+2} + \gamma^2 R_{t+3} + \cdots + \gamma^{T-t-1} R_T = \sum_{k=0}^{T-t-1} \gamma^k R_{t+k+1}}$$

where:
- $R_{t+1}$ is the reward received after leaving state $S_t$
- $\gamma \in [0, 1]$ is the discount factor
- $T$ is the terminal time step of the episode

### Recursive Computation

Rather than computing $G_t$ from scratch, we can exploit the recursive relationship. Working
**backwards** from the terminal state:

$$G_T = 0 \qquad \text{(no reward after termination)}$$

$$G_t = R_{t+1} + \gamma \, G_{t+1} \quad \text{for } t = T{-}1, T{-}2, \ldots, 0$$

This backward computation is $O(T)$ and avoids recomputing sums from scratch.

### Worked Example

Consider an episode with $\gamma = 0.9$ and rewards $[R_1, R_2, R_3, R_4] = [+1, -2, +3, +1]$
(episode length $T = 4$):

```
  Time step:     t=0       t=1       t=2       t=3       t=4
  State:         S₀        S₁        S₂        S₃      Terminal
  Reward:            R₁=+1     R₂=-2     R₃=+3     R₄=+1
                     │          │          │          │
  Backward:          │          │          │          └─ G₃ = +1
                     │          │          └──── G₂ = +3 + 0.9(+1) = +3.9
                     │          └───────── G₁ = -2 + 0.9(+3.9) = +1.51
                     └──────────────────── G₀ = +1 + 0.9(+1.51) = +2.359
```

So $V^\pi(S_0) \approx G_0 = 2.359$ based on this single episode. With more episodes,
we average all the $G_0$ values observed to get a better estimate.

---

## 3. The MC Prediction Idea

The value of a state under policy $\pi$ is defined as the **expected return** starting
from that state:

$$V^\pi(s) = \mathbb{E}_\pi[G_t \mid S_t = s]$$

MC prediction estimates this expectation by replacing it with a **sample mean**. Given
$N$ episodes, each time state $s$ is visited we record the subsequent return. The MC
estimate is:

$$\hat{V}^\pi(s) = \frac{1}{N(s)} \sum_{i=1}^{N(s)} G_t^{(i)}$$

where $N(s)$ is the number of times state $s$ was visited (across all episodes) and
$G_t^{(i)}$ is the return observed on the $i$-th visit to $s$.

By the **law of large numbers**, this sample mean converges to the true expected value:

$$\hat{V}^\pi(s) \xrightarrow{N(s) \to \infty} V^\pi(s)$$

Each state's estimate is **independent** — the estimate for $V^\pi(s)$ depends only on
returns observed from $s$, not on estimates of other states. This is in stark contrast
to DP, where the estimate for $V^\pi(s)$ is bootstrapped from estimates of successor
states.

---

## 4. First-Visit vs Every-Visit MC

When a state $s$ appears multiple times within a single episode, we must decide which
visits to count. This gives rise to two variants:

| Variant | Rule | Bias | Use Case |
|---|---|---|---|
| **First-visit MC** | Only count the return from the **first** time $s$ is visited in each episode | Unbiased | Most common; standard in textbooks |
| **Every-visit MC** | Count the return from **every** time $s$ is visited in each episode | Slightly biased (asymptotically unbiased) | Sometimes simpler to implement |

```
  ╔═══════════════════════════════════════════════════════════════════╗
  ║  FIRST-VISIT vs EVERY-VISIT — Illustration                      ║
  ╠═══════════════════════════════════════════════════════════════════╣
  ║                                                                  ║
  ║  Episode: S₀ → S₁ → S₂ → S₁ → S₃ → Terminal                   ║
  ║                      ↑         ↑                                 ║
  ║                  1st visit  2nd visit to S₁                      ║
  ║                                                                  ║
  ║  First-visit MC for S₁:  uses only G at 1st visit (t=1)         ║
  ║  Every-visit MC for S₁:  uses G at t=1 AND G at t=3             ║
  ║                                                                  ║
  ║  Both converge to V^π(S₁) as episodes → ∞                       ║
  ╚═══════════════════════════════════════════════════════════════════╝
```

Both variants converge to $V^\pi(s)$ as the number of episodes goes to infinity. The
detailed theoretical comparison (variance, convergence rate, MSE trade-offs) is covered
in [Chapter 5: First-Visit vs Every-Visit MC](./05-first-visit-vs-every-visit.md).

For this chapter, we focus on **first-visit MC prediction**, which is the standard approach.

---

## 5. First-Visit MC Prediction Algorithm

```
 ╔═══════════════════════════════════════════════════════════════════════╗
 ║  FIRST-VISIT MC PREDICTION                                          ║
 ╠═══════════════════════════════════════════════════════════════════════╣
 ║                                                                      ║
 ║  Input : policy π, number of episodes N, discount factor γ           ║
 ║  Output: V ≈ V^π                                                    ║
 ║                                                                      ║
 ║  1.  Initialize:                                                     ║
 ║        V(s) ← 0            for all s ∈ S                            ║
 ║        Returns(s) ← []     for all s ∈ S   (empty lists)            ║
 ║                                                                      ║
 ║  2.  For episode = 1, 2, ..., N:                                     ║
 ║  3.      Generate episode following π: S₀,A₀,R₁,S₁,A₁,R₂,...,S_T   ║
 ║  4.      G ← 0                                                      ║
 ║  5.      For t = T−1, T−2, ..., 0:         (loop backward)          ║
 ║  6.          G ← γ G + R_{t+1}                                      ║
 ║  7.          If S_t not in {S₀, S₁, ..., S_{t-1}}:  (first visit?)  ║
 ║  8.              Append G to Returns(S_t)                            ║
 ║  9.              V(S_t) ← mean(Returns(S_t))                        ║
 ║ 10.  Return V                                                        ║
 ║                                                                      ║
 ╚═══════════════════════════════════════════════════════════════════════╝
```

### Step-by-Step Walkthrough

1. **Generate a complete episode** by following policy $\pi$ until termination.
2. **Walk backward** through the episode. At each step, accumulate the discounted return
   using $G \leftarrow \gamma G + R_{t+1}$.
3. **Check first-visit condition:** if $S_t$ has not appeared earlier in this episode
   ($S_t \notin \{S_0, \ldots, S_{t-1}\}$), record $G$ in the returns list for $S_t$.
4. **Update the value estimate** as the mean of all recorded returns.

---

## 6. Incremental Mean Update

Storing all returns and recomputing the mean each time is memory-intensive. The
**incremental mean** formula allows us to update the mean online:

$$\mu_n = \mu_{n-1} + \frac{1}{n}(x_n - \mu_{n-1})$$

Applied to MC prediction, after the $n$-th return $G_n$ for state $s$:

$$\boxed{V(s) \leftarrow V(s) + \frac{1}{N(s)}\Big(G_t - V(s)\Big)}$$

This is equivalent to the running average but requires only $O(1)$ memory per state
(just store $V(s)$ and the count $N(s)$).

### Generalization: Constant Step-Size

For non-stationary environments (where the policy or dynamics may change), we replace
$\frac{1}{N(s)}$ with a constant step size $\alpha$:

$$V(s) \leftarrow V(s) + \alpha \Big(G_t - V(s)\Big)$$

This gives **exponentially decaying** weight to older returns, allowing the estimate to
track changes. The quantity $(G_t - V(s))$ is the **MC error** — the difference between
the observed return and the current estimate.

```
  INCREMENTAL UPDATE VISUALIZATION

  Current estimate:  V(s) = 7.0
  Observed return:   G_t  = 10.0
  Visit count:       N(s) = 4

  Error:             G_t - V(s) = 10.0 - 7.0 = 3.0
  Step size:         1/N(s) = 1/4 = 0.25
  Update:            V(s) ← 7.0 + 0.25 × 3.0 = 7.75

  ──────────────────────────────────────────────────
  V(s)  ●────────────────────●─────────────● G_t
        7.0                 7.75          10.0
                             ↑
                        New estimate
                     (moved 1/4 toward G_t)
```

---

## 7. Backup Diagrams: MC vs DP

A **backup diagram** shows the information flow used to update a state's value. The
backup diagrams for MC and DP reveal their fundamental differences:

```
  DP BACKUP (Policy Evaluation)           MC BACKUP (Monte Carlo)

         V(s)                                   V(s)
        / | \                                    |
     π(a₁) π(a₂) π(a₃)                         |
      /    |    \                                |
    a₁    a₂    a₃                              S₁   (sampled)
   /|\   /|\   /|\                               |
  s s s s s s s s s                              S₂   (sampled)
  ↑ ↑ ↑ ↑ ↑ ↑ ↑ ↑ ↑                             |
  All successors weighted                        S₃   (sampled)
  by P(s'|s,a)                                   |
                                                 ·
  Width: FULL                                    ·
  Depth: ONE STEP                                ·
  Needs: Model (P, R)                            |
  Bootstrap: YES                                S_T  (terminal)
  (uses estimated V(s'))
                                            Width: SINGLE SAMPLE
                                            Depth: FULL EPISODE
                                            Needs: Experience only
                                            Bootstrap: NO
                                            (uses actual return G_t)
```

### Key Differences

| Property | DP Backup | MC Backup |
|---|---|---|
| **Width** | Full (all actions, all successors) | Sample (one trajectory) |
| **Depth** | One step (bootstraps from $V(s')$) | Full episode (to termination) |
| **Model required** | Yes ($P$ and $R$) | No (model-free) |
| **Bootstrapping** | Yes (relies on estimated $V(s')$) | No (uses actual return $G_t$) |
| **Bias** | Biased by initial $V_0$ until convergence | Unbiased (returns are true samples) |
| **Variance** | Low (averages over all transitions) | High (single trajectory is noisy) |
| **Requires episodes?** | No (can run on continuing tasks) | Yes (must reach terminal state) |

> **Key Insight:** DP backs up from **all** possible successors weighted by their
> probabilities (full-width, shallow backup). MC backs up along **one** sampled trajectory
> all the way to termination (narrow, deep backup). This is why MC does not need a model
> but has higher variance.

---

## 8. Convergence Properties

### Law of Large Numbers

The fundamental convergence guarantee for MC prediction comes from the **strong law of
large numbers**: if each state $s$ is visited infinitely often, then:

$$\hat{V}^\pi(s) = \frac{1}{N(s)} \sum_{i=1}^{N(s)} G_t^{(i)} \xrightarrow{\text{a.s.}} V^\pi(s) \quad \text{as } N(s) \to \infty$$

This holds for first-visit MC because the returns $G_t^{(1)}, G_t^{(2)}, \ldots$ are
**independent, identically distributed** (i.i.d.) samples from the distribution of
returns starting from state $s$ under policy $\pi$.

### Convergence Rate

By the **central limit theorem**, the estimation error decreases as:

$$\text{Standard error} = \frac{\sigma_s}{\sqrt{N(s)}}$$

where $\sigma_s$ is the standard deviation of returns from state $s$. This means:
- To halve the error, we need **4× more episodes**
- Convergence is $O(1/\sqrt{N})$, which is slower than DP's geometric convergence

### Conditions for Convergence

1. **Episodic task:** Episodes must terminate with probability 1
2. **Sufficient visits:** Every state must be visited infinitely often (in the limit)
3. **Finite variance:** The returns must have finite variance

For first-visit MC with $\frac{1}{N(s)}$ step sizes, both conditions (1) and (2) are
sufficient. For every-visit MC, convergence also holds but the proof is more involved
because the returns within a single episode are not independent.

---

## 9. Advantages and Disadvantages

### Advantages of MC Prediction

1. **No model required:** Works without knowledge of $P(s' \mid s,a)$ or $R(s,a,s')$.
   The agent only needs to interact with the environment (or a simulator).

2. **Unbiased estimates:** Since MC uses actual returns (not bootstrapped estimates), the
   value estimates are unbiased. This is important for convergence guarantees.

3. **Can focus on states of interest:** Unlike DP, which must sweep all states, MC can
   estimate $V^\pi(s)$ for a single state $s$ without computing values for all other
   states. Just generate episodes that start from (or pass through) $s$.

4. **Works with black-box simulators:** Only needs to sample trajectories. No access to
   the environment's internal dynamics is required.

5. **Handles large state spaces better than tabular DP:** Since we only update visited
   states, the per-episode cost depends on episode length, not $|\mathcal{S}|$.

### Disadvantages of MC Prediction

1. **High variance:** Returns can vary enormously across episodes, especially with long
   episodes or stochastic environments. This leads to slow convergence.

2. **Requires complete episodes:** Cannot update values until the episode terminates. This
   is a problem for long episodes or continuing (non-episodic) tasks.

3. **Slow convergence:** The $O(1/\sqrt{N})$ convergence rate means many episodes are
   needed for accurate estimates, especially in high-variance environments.

4. **Inefficient use of data:** Each return only updates the value of one state (or a few
   states in the episode). DP, by contrast, propagates information to all states each sweep.

5. **Exploration dependency:** If policy $\pi$ never visits certain states, MC cannot
   estimate their values. Ensuring sufficient exploration can be challenging.

```
  ╔══════════════════════════════════════════════════════════════════════╗
  ║  TRADE-OFF SUMMARY                                                  ║
  ╠══════════════════════════════════════════════════════════════════════╣
  ║                                                                     ║
  ║  DP:  Low variance, biased (until converged), needs model           ║
  ║  MC:  High variance, unbiased, model-free                           ║
  ║                                                                     ║
  ║        Bias ──────────────────────── Variance                       ║
  ║         DP ●━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━● MC                     ║
  ║      (bootstraps)                    (actual returns)               ║
  ║                                                                     ║
  ║  TD methods (Unit 5) will occupy the middle ground.                 ║
  ╚══════════════════════════════════════════════════════════════════════╝
```

---

## 10. Python Implementation — Blackjack

We implement first-visit MC prediction for the Blackjack environment from Gymnasium
(Farama Foundation). In Blackjack, the player's goal is to obtain cards summing as close
to 21 as possible without exceeding it.

### State Space

A Blackjack state is a tuple: (player sum, dealer showing card, usable ace).

### Implementation

```python
import numpy as np
from collections import defaultdict
import gymnasium as gym


def generate_episode(env, policy):
    """
    Generate one complete episode following the given policy.

    Returns:
        List of (state, action, reward) tuples.
    """
    episode = []
    state, _ = env.reset()
    done = False

    while not done:
        action = policy(state)
        next_state, reward, terminated, truncated, _ = env.step(action)
        episode.append((state, action, reward))
        state = next_state
        done = terminated or truncated

    return episode


def simple_policy(state):
    """
    Simple fixed policy for Blackjack:
    Hit (1) if player sum < 20, else Stick (0).
    """
    player_sum, _, _ = state
    return 1 if player_sum < 20 else 0


def first_visit_mc_prediction(env, policy, num_episodes, gamma=1.0):
    """
    First-visit Monte Carlo prediction.

    Args:
        env:          Gymnasium environment
        policy:       Function mapping state -> action
        num_episodes: Number of episodes to sample
        gamma:        Discount factor

    Returns:
        V: Dictionary mapping state -> estimated value
    """
    V = defaultdict(float)
    N = defaultdict(int)  # Visit counts

    for ep in range(num_episodes):
        # Step 1: Generate a complete episode
        episode = generate_episode(env, policy)

        # Step 2: Compute returns backward
        G = 0.0
        visited = set()

        for t in range(len(episode) - 1, -1, -1):
            state, action, reward = episode[t]
            G = gamma * G + reward

            # Step 3: First-visit check
            if state not in visited:
                visited.add(state)
                N[state] += 1
                # Incremental mean update
                V[state] += (G - V[state]) / N[state]

    return V, N


# ========================================
# Run MC Prediction on Blackjack
# ========================================
env = gym.make("Blackjack-v1", sab=True)

print("Running first-visit MC prediction on Blackjack...")
print("Policy: Hit if player_sum < 20, else Stick\n")

V, N = first_visit_mc_prediction(env, simple_policy, num_episodes=500000, gamma=1.0)

# Display some state values
print(f"Total unique states visited: {len(V)}\n")
print("Sample state values (player_sum, dealer_card, usable_ace):")
print("-" * 60)

sample_states = [
    (21, 10, False),  # Blackjack-like hand vs dealer 10
    (20, 1, False),   # Strong hand vs dealer Ace
    (18, 6, False),   # Good hand vs weak dealer
    (15, 10, False),  # Weak hand vs strong dealer
    (12, 3, True),    # Low hand with usable ace
    (20, 5, False),   # Strong hand vs weak dealer
]

for s in sample_states:
    if s in V:
        print(f"  V{s} = {V[s]:+.4f}  (visited {N[s]} times)")
    else:
        print(f"  V{s} = not visited")

# Compute average value across all states
all_values = list(V.values())
print(f"\nValue statistics across all states:")
print(f"  Mean V(s):  {np.mean(all_values):.4f}")
print(f"  Std  V(s):  {np.std(all_values):.4f}")
print(f"  Min  V(s):  {np.min(all_values):.4f}")
print(f"  Max  V(s):  {np.max(all_values):.4f}")

env.close()
```

**Expected output (approximate):**

```
Running first-visit MC prediction on Blackjack...
Policy: Hit if player_sum < 20, else Stick

Total unique states visited: 280

Sample state values (player_sum, dealer_card, usable_ace):
------------------------------------------------------------
  V(21, 10, False) = +0.8724  (visited 14230 times)
  V(20, 1, False)  = +0.3548  (visited 12456 times)
  V(18, 6, False)  = +0.1215  (visited 8923 times)
  V(15, 10, False) = -0.4562  (visited 7841 times)
  V(12, 3, True)   = -0.1234  (visited 3215 times)
  V(20, 5, False)  = +0.6832  (visited 9102 times)

Value statistics across all states:
  Mean V(s):  -0.0548
  Std  V(s):   0.3812
  Min  V(s):  -0.7231
  Max  V(s):   +0.9167
```

### How the Code Connects to the Theory

| Code Element | Theory Concept |
|---|---|
| `generate_episode()` | Follows $\pi$ to get complete trajectory |
| `G = gamma * G + reward` | Backward return computation: $G_t = R_{t+1} + \gamma G_{t+1}$ |
| `if state not in visited` | First-visit check: $S_t \notin \{S_0, \ldots, S_{t-1}\}$ |
| `V[state] += (G - V[state]) / N[state]` | Incremental mean: $V(s) \leftarrow V(s) + \frac{1}{N(s)}(G_t - V(s))$ |
| `num_episodes=500000` | Law of large numbers: more episodes → better estimates |

---

## 11. Convergence Demonstration — Grid World

To visualize convergence behavior, here is a simpler example using a 4×4 grid world.
This demonstrates how MC estimates approach the true values as the number of episodes
increases.

```python
import numpy as np
from collections import defaultdict


class SimpleGridWorld:
    """
    4×4 grid world. Terminal states at (0,0) and (3,3).
    All transitions give reward -1. Random policy.
    """

    def __init__(self, size=4):
        self.size = size
        self.terminals = {(0, 0), (size - 1, size - 1)}

    def reset(self):
        # Start at a random non-terminal state
        while True:
            state = (
                np.random.randint(self.size),
                np.random.randint(self.size),
            )
            if state not in self.terminals:
                return state

    def step(self, state, action):
        """action: 0=N, 1=S, 2=E, 3=W"""
        if state in self.terminals:
            return state, 0.0, True

        moves = {0: (-1, 0), 1: (1, 0), 2: (0, 1), 3: (0, -1)}
        dr, dc = moves[action]
        nr = max(0, min(self.size - 1, state[0] + dr))
        nc = max(0, min(self.size - 1, state[1] + dc))
        next_state = (nr, nc)
        done = next_state in self.terminals
        return next_state, -1.0, done


def mc_prediction_grid(num_episodes, gamma=1.0):
    """First-visit MC prediction on 4×4 grid world."""
    env = SimpleGridWorld()
    V = defaultdict(float)
    N = defaultdict(int)

    for _ in range(num_episodes):
        # Generate episode
        state = env.reset()
        episode = []
        done = False
        while not done:
            action = np.random.randint(4)  # Random policy
            next_state, reward, done = env.step(state, action)
            episode.append((state, reward))
            state = next_state

        # Compute returns and update (first-visit)
        G = 0.0
        visited = set()
        for t in range(len(episode) - 1, -1, -1):
            s, r = episode[t]
            G = gamma * G + r
            if s not in visited:
                visited.add(s)
                N[s] += 1
                V[s] += (G - V[s]) / N[s]

    return V, N


# Show convergence at different episode counts
for n_ep in [100, 1000, 10000, 100000]:
    V, N = mc_prediction_grid(n_ep)
    print(f"\nAfter {n_ep:>6} episodes:")
    for r in range(4):
        row_vals = []
        for c in range(4):
            s = (r, c)
            if s in {(0, 0), (3, 3)}:
                row_vals.append(f"{'0.0':>7s}")
            elif s in V:
                row_vals.append(f"{V[s]:>7.1f}")
            else:
                row_vals.append(f"{'?':>7s}")
        print("  " + " ".join(row_vals))
```

**Expected convergence (approximate):**

```
After    100 episodes:
      0.0   -13.2   -19.5   -20.8
    -14.1   -17.3   -19.1   -19.7
    -19.8   -19.6   -17.8   -13.5
    -21.5   -20.2   -14.6     0.0

After 100000 episodes:
      0.0   -14.0   -20.0   -22.0     ← Matches DP result
    -14.0   -18.0   -20.0   -20.0
    -20.0   -20.0   -18.0   -14.0
    -22.0   -20.0   -14.0     0.0
```

This confirms that MC prediction converges to the same values as DP policy evaluation,
but without requiring the transition model.

---

## 12. Why MC is Model-Free

Consider what each method needs to compute its update:

```
  ┌──────────────────────────────────────────────────────────────────┐
  │              MODEL-BASED vs MODEL-FREE                           │
  │                                                                  │
  │  DP Update:                                                      │
  │    V(s) ← Σ_a π(a|s) Σ_{s'} P(s'|s,a) [R(s,a,s') + γ V(s')]   │
  │                       ^^^^^^^^^^^^                               │
  │                       Needs to know P (the model)                │
  │                                                                  │
  │  MC Update:                                                      │
  │    V(s) ← V(s) + (1/N(s)) × (G_t - V(s))                       │
  │                                ^^^^^                             │
  │                                G_t comes from experience         │
  │                                No P needed!                      │
  │                                                                  │
  │  DP needs:  P(s'|s,a), R(s,a,s'), π(a|s), γ                     │
  │  MC needs:  Sampled episodes from π, γ                           │
  └──────────────────────────────────────────────────────────────────┘
```

The key distinction is that MC replaces the model-dependent **expected value** computation
(summing over all transitions weighted by $P$) with a model-free **sample average**
(averaging observed returns). The environment acts as its own "model" — we sample from
$P$ implicitly by interacting with the environment, even if we don't know $P$ explicitly.

---

## 13. Practical Considerations

### When to Use MC Prediction

- **Unknown transition dynamics:** The environment is a black box or simulator
- **Episodic tasks:** Tasks naturally terminate (games, robot trials, medical treatments)
- **Focus on specific states:** You only need values for a subset of states
- **Correctness matters:** You need unbiased estimates (e.g., for theoretical analysis)

### When NOT to Use MC Prediction

- **Continuing tasks:** No natural episode boundary (use TD methods instead)
- **Very long episodes:** Returns have extreme variance; updates are infrequent
- **Online learning needed:** Cannot learn during an episode, only after it ends
- **Fast convergence needed:** DP or TD methods converge faster in many settings

### Variance Reduction Techniques

Several practical tricks can reduce the variance of MC estimates:

1. **Use $\gamma < 1$:** Discounting reduces the influence of far-future rewards,
   lowering variance at the cost of a shorter effective horizon.

2. **Baseline subtraction:** Subtract a baseline from returns before averaging
   (more relevant for MC control and policy gradients).

3. **Increase the number of episodes:** Variance decreases as $O(1/N)$.

4. **Episode truncation:** Cap episode length at some maximum (introduces bias but
   reduces variance in practice).

---

## 14. Summary

| Concept | Key Point |
|---|---|
| MC prediction | Estimate $V^\pi$ by averaging returns from sampled episodes |
| Return $G_t$ | $G_t = R_{t+1} + \gamma R_{t+2} + \cdots + \gamma^{T-t-1} R_T$ |
| Recursive return | $G_t = R_{t+1} + \gamma G_{t+1}$, computed backward from $G_T = 0$ |
| First-visit MC | Count only the first visit to $s$ in each episode (unbiased) |
| Every-visit MC | Count every visit to $s$ (slightly biased, asymptotically unbiased) |
| Incremental mean | $V(s) \leftarrow V(s) + \frac{1}{N(s)}(G_t - V(s))$ |
| Constant step-size | $V(s) \leftarrow V(s) + \alpha(G_t - V(s))$ for non-stationary problems |
| Convergence | By law of large numbers; rate $O(1/\sqrt{N})$ |
| Backup diagram | Narrow and deep (single trajectory to terminal) vs DP's wide and shallow |
| Model-free | No $P(s' \mid s,a)$ needed; learns from experience |
| Unbiased | Uses actual returns, not bootstrapped estimates |
| High variance | Single trajectories are noisy; many episodes needed |
| Requires episodes | Must reach terminal state; unsuitable for continuing tasks |

---

## Revision Questions

1. **Write the formula for the return $G_t$ in both summation and recursive forms.** Compute $G_0$ for an episode with rewards $[+2, -1, +3]$ and $\gamma = 0.5$.

2. **Explain the difference between first-visit and every-visit MC prediction.** Given an episode $S_A \to S_B \to S_A \to S_C \to \text{Terminal}$ with rewards $[+1, -1, +2, +3]$ and $\gamma = 1$, what return does first-visit MC assign to $S_A$? What about every-visit MC?

3. **Draw the backup diagrams for DP policy evaluation and MC prediction side by side.** Label the width, depth, model requirement, and bootstrapping property of each.

4. **Derive the incremental mean formula** starting from $\mu_n = \frac{1}{n}\sum_{i=1}^{n} x_i$. Show that $\mu_n = \mu_{n-1} + \frac{1}{n}(x_n - \mu_{n-1})$.

5. **Why is MC prediction said to be "model-free"?** Identify exactly which quantities in the DP update rule are replaced by sample-based quantities in MC, and explain why this eliminates the need for a model.

6. **A colleague claims MC is always better than DP because it is unbiased.** Critique this claim. Under what conditions would DP be preferable despite its initial bias? Discuss the bias-variance trade-off.

---

## Navigation

| Previous | Up | Next |
|---|---|---|
| [Limitations of DP](../03-Dynamic-Programming/04-limitations-of-dp.md) | [Unit 4: Monte Carlo Methods](./README.md) | [MC Control](./02-mc-control.md) |
